+++
title = "Basic Graphics"
weight = 1

[taxonomies]
categories = ["go"]
tags = ["pong"]
+++

## Defining our game entities

Now that we have the program showing a window, we can proceed to define our paddles and ball types!

```go
// paddle.go
package main

import (
	"github.com/veandco/go-sdl2/sdl"
)

const (
	PaddleSpeed = 1
)

type Paddle struct {
	rect     sdl.FRect  // This stores the paddle's position and size. We will use this to draw rectangle as our paddle as well.
	velocity sdl.FPoint // This stores the paddle's velocity
}

func NewPaddle(x float32) *Paddle {
	paddle := &Paddle{}

	// Paddle's X-coordinate is set by the game
	paddle.rect.X = x

	// Paddle always starts at the middle of the window in Y-axis
	paddle.rect.Y = 0.5

	paddle.rect.W = 0.01
	paddle.rect.H = 0.06

	return paddle
}

func (paddle *Paddle) Update(deltaTime float32) {
	paddle.rect.Y += paddle.velocity.Y * deltaTime
}

func (paddle *Paddle) Draw(surface *sdl.Surface) {
	// We need to convert sdl.FRect to sdl.Rect as surface.FillRect() accepts sdl.Rect
	// This also maps our coordinate to match our window coordinate.
	rect := sdl.Rect{
		X: int32(paddle.rect.X * WindowWidth),
		Y: int32(paddle.rect.Y * WindowHeight),
		W: int32(paddle.rect.W * WindowWidth),
		H: int32(paddle.rect.H * WindowHeight),
	}

	// Specify rectangle color to white
	color := uint32(0xFFFFFFFF)

	// Draw the rectangle on the surface
	surface.FillRect(&rect, color)
}
```

```go
// ball.go
package main

import (
	"math"
	"math/rand"

	"github.com/veandco/go-sdl2/sdl"
)

const (
	BallSpeed          = 1
	BallDirectionRight = 0
	BallDirectionLeft  = 1
)

type Ball struct {
	rect     sdl.FRect  // This stores the ball's position and size. We will use this to draw a small rectangle as our ball as well.
	velocity sdl.FPoint // This stores the ball's velocity
}

func NewBall(direction int) *Ball {
	ball := &Ball{}

	// Ball always starts at the middle of the window
	ball.rect.X = 0.5
	ball.rect.Y = 0.5
	ball.rect.W = 0.02
	ball.rect.H = 0.02

	// Randomize the ball's initial velocity and direction (either BallDirectionRight or BallDirectionLeft)
	randomDirection := rand.Intn(2)
	ball.randomizeVelocity(randomDirection)

	return ball
}

func (ball *Ball) Update(deltaTime float32) {
	ball.rect.X += ball.velocity.X * deltaTime
	ball.rect.Y += ball.velocity.Y * deltaTime
}

func (ball *Ball) Draw(surface *sdl.Surface) {
	// We need to convert sdl.FRect to sdl.Rect as surface.FillRect() accepts sdl.Rect
	// This also maps our coordinate to match our window coordinate.
	rect := sdl.Rect{
		X: int32(ball.rect.X * WindowWidth),
		Y: int32(ball.rect.Y * WindowHeight),
		W: int32(ball.rect.W * WindowWidth),
		H: int32(ball.rect.H * WindowHeight),
	}

	// Specify rectangle color to white
	color := uint32(0xFFFFFFFF)

	// Draw the rectangle on the surface
	surface.FillRect(&rect, color)
}

// Select an angle for the ball to move toward
// NOTE: Make sure the ball start to move at angle between -45deg and 45deg in either direction
func (ball *Ball) randomizeVelocity(direction int) {
	randomRadian := (math.Pi/2*rand.Float64() - math.Pi/4) + math.Pi*float64(direction)

	ball.velocity = sdl.FPoint{
		X: float32(math.Cos(randomRadian)) * BallSpeed,
		Y: float32(math.Sin(randomRadian)) * BallSpeed,
	}
}
```

## Drawing our game entities

Let's go back to our `Game` type to create and draw our entities!

```go
// game.go
package main

import (
	"math/rand"
	"time"

	"github.com/veandco/go-sdl2/sdl"
)

type Game struct {
	running bool    // Whether the game is running or not
	paddle1 *Paddle // Paddle for player 1
	paddle2 *Paddle // Paddle for player 2
	ball    *Ball
}

func NewGame() *Game {
	return &Game{
		running: true,
		paddle1: NewPaddle(0.1),
		paddle2: NewPaddle(0.9),
		ball:    NewBall(rand.Intn(2)),
	}
}

...

func (game *Game) Update(deltaTime float32) {
	game.paddle1.Update(deltaTime)
	game.paddle2.Update(deltaTime)
	game.ball.Update(deltaTime)
}

func (game *Game) Draw(surface *sdl.Surface) {
	game.paddle1.Draw(surface)
	game.paddle2.Draw(surface)
	game.ball.Draw(surface)
}
```
