import turtle

# Set up the screen
screen = turtle.Screen()
screen.bgcolor("white")

# Create a turtle object
pen = turtle.Turtle()
pen.shape("turtle")
pen.color("red")
pen.speed(3)

# Move to the starting position
pen.penup()
pen.goto(0, -200)
pen.pendown()

# Start drawing the heart shape
pen.begin_fill()

pen.left(50)
pen.forward(133)
pen.circle(50, 200)
pen.right(140)
pen.circle(50, 200)
pen.forward(133)

pen.end_fill()

# Hide the turtle after drawing
pen.hideturtle()

# Keep the window open
turtle.done()

