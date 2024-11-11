mkdir simple_framework
cd simple_framework

python -m venv venv
source venv/bin/activate  # On macOS/Linux
venv\Scripts\activate  # On Windows

simple_framework/
├── simple_framework/
│   ├── __init__.py
│   ├── server.py
│   └── request.py
├── examples/
│   └── app.py
├── tests/
│   └── test_framework.py
└── setup.py

import socket
import threading
import sys
from .request import Request

class SimpleFramework:
    def __init__(self, host='127.0.0.1', port=5000):
        self.host = host
        self.port = port
        self.routes = {}

    def route(self, path, method):
        def wrapper(func):
            if path not in self.routes:
                self.routes[path] = {}
            self.routes[path][method] = func
            return func
        return wrapper

    def handle_request(self, client_socket):
        request = client_socket.recv(1024).decode('utf-8')
        request_line = request.split('\r\n')[0]
        method, path, _ = request_line.split()

        # Create a Request object to parse further
        req = Request(method, path)

        if path in self.routes and method in self.routes[path]:
            response = self.routes[path][method](req)
        else:
            response = "HTTP/1.1 404 Not Found\r\n\r\nPage not found"
        
        client_socket.send(response.encode('utf-8'))
        client_socket.close()

    def run(self):
        server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server_socket.bind((self.host, self.port))
        server_socket.listen(5)
        print(f"Listening on {self.host}:{self.port}")

        while True:
            client_socket, _ = server_socket.accept()
            threading.Thread(target=self.handle_request, args=(client_socket,)).start()

class Request:
    def __init__(self, method, path):
        self.method = method
        self.path = path

from simple_framework.server import SimpleFramework

app = SimpleFramework()

@app.route('/hello', method='GET')
def hello(request):
    return "HTTP/1.1 200 OK\r\n\r\nHello, World!"

@app.route('/submit', method='POST')
def submit(request):
    return "HTTP/1.1 200 OK\r\n\r\nForm Submitted!"

if __name__ == '__main__':
    app.run()

python examples/app.py

curl http://127.0.0.1:5000/hello

curl -X POST http://127.0.0.1:5000/submit

Form Submitted!

import unittest
from simple_framework.server import SimpleFramework
from simple_framework.request import Request

class SimpleFrameworkTests(unittest.TestCase):

    def setUp(self):
        self.app = SimpleFramework()

    def test_route_definition(self):
        @self.app.route('/test', method='GET')
        def test_route(request):
            return "HTTP/1.1 200 OK\r\n\r\nTest Passed"
        
        request = Request('GET', '/test')
        response = self.app.routes['/test']['GET'](request)
        self.assertIn("Test Passed", response)

    def test_404_for_non_existent_route(self):
        @self.app.route('/hello', method='GET')
        def hello(request):
            return "HTTP/1.1 200 OK\r\n\r\nHello, World!"
        
        request = Request('GET', '/nonexistent')
        response = self.app.routes.get('/nonexistent', {}).get('GET')
        self.assertIsNone(response)

if __name__ == '__main__':
    unittest.main()


## Usage

```python
from simple_framework.server import SimpleFramework

app = SimpleFramework()

@app.route('/hello', method='GET')
def hello(request):
 return "HTTP/1.1 200 OK\r\n\r\nHello, World!"

if __name__ == '__main__':
 app.run()


### Conclusion

You've now created a simple Python framework for building web servers that can handle HTTP requests and routes. This framework provides a foundation for building RESTful APIs, similar to frameworks like Flask or FastAPI, but with minimal functionality.

This is just the start. You can expand this framework by adding more features like:
- Middleware for request processing.
- Support for other HTTP methods (PUT, DELETE).
- Better request parsing (e.g., form data, JSON).
- Template rendering for HTML responses.
