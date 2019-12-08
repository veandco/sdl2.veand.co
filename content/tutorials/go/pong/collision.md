+++
title = "Collision"
weight = 3

[taxonomies]
categories = ["go"]
tags = ["pong"]
+++

## Making the entities interact

We have made the paddles and ball appear and they can move. However, the ball goes through the paddles and even the window!
The same with the paddles: if you move them high or low enough, they start to go through the window.

Let's make a list of all the interactions that we need to implement:

+ Ball should bounce when it hits a paddle
+ Ball should bounce when it hits the top and bottom walls
+ Player would score if the ball hits the left or right walls
+ Player should not be able to move beyond the window

We will implement each of these interactions in the `Game` type since it has been where we have been modifying our entities' behaviours (e.g. for their movement via keyboard input). Let's start with the first interaction!

## Ball bounces off paddles

```go
// game.go
...

func (game *Game) runBallAndPaddlesCollision() {
	// Run ball-paddle1 collision. Ball should bounce to the right.
	if _, intersects := game.ball.rect.Intersect(&game.paddle1.rect); intersects {
		game.ball.randomizeVelocity(BallDirectionRight)
	}

	// Run ball-paddle2 collision. Ball should bounce to the left.
	if _, intersects := game.ball.rect.Intersect(&game.paddle2.rect); intersects {
		game.ball.randomizeVelocity(BallDirectionLeft)
	}
}
```

Next we will do the next two interactions between the ball and the walls. We will also implement scoring when the ball hit the left or right wall while we are at it since it is pretty simple to implement.

## Ball bounces off walls

```go
// game.go
...

func (game *Game) runBallAndWallsCollision() {
	// Check if any of the ball's corners is outside the window
	ballIsOutsideWindow := game.ball.rect.X < 0 ||
							game.ball.rect.X+game.ball.rect.W >= 1 ||
							game.ball.rect.Y < 0 ||
							game.ball.rect.Y+game.ball.rect.H >= 1

	if ballIsOutsideWindow {
		// Check if the ball is outside the left-right walls
		// If the ball's Y-position is not outside the window, one of the players gain score and the ball is reset
		if game.ball.rect.Y >= 0 && game.ball.rect.Y+game.ball.rect.H < 1 {

			// The ball hit the right wall so player 1 should score
			if game.ball.rect.X > 0.5 {
				game.score1++

			// The ball hit the left wall so player 2 should score
			} else {
				game.score2++
			}

			// Create a new ball to put it back to the center
			game.ball = NewBall(rand.Intn(2))

		// Check if the ball is outside the top-bottom walls
		// If the ball's X-position is not outside the window, bounce the ball
		} else if game.ball.rect.X >= 0 && game.ball.rect.X+game.ball.rect.W < 1 {
			game.ball.velocity.Y = -game.ball.velocity.Y
		}
	}
}
```

Since our `Game` is currently missing `score1` and `score2`, let's add them!

```go
// game.go
...

type Game struct {
	running bool    // Whether the game is running or not
	paddle1 *Paddle // Paddle for player 1
	paddle2 *Paddle // Paddle for player 2
	ball    *Ball
	score1  int // Score for player 1
	score2  int // Score for player 2
}

...
```

## Making the top and bottom walls block paddles

```go
// game.go
...

func (game *Game) runPaddlesAndWallsCollision() {
	// Make sure paddle 1 doesn't go out of window
	if game.paddle1.rect.Y+game.paddle1.rect.H > 1 {
		game.paddle1.rect.Y = 1 - game.paddle1.rect.H
	} else if game.paddle1.rect.Y < 0 {
		game.paddle1.rect.Y = 0
	}

	// Make sure paddle 2 doesn't go out of window
	if game.paddle2.rect.Y+game.paddle2.rect.H > 1 {
		game.paddle2.rect.Y = 1 - game.paddle2.rect.H
	} else if game.paddle2.rect.Y < 0 {
		game.paddle2.rect.Y = 0
	}
}
```

## Putting the collisions to work

```go
...

func (game *Game) Update(deltaTime float32) {
	...

	game.runBallAndPaddlesCollision()
	game.runBallAndWallsCollision()
	game.runPaddlesAndWallsCollision()
}

...
```

## Making sure ball-paddle collision more robust

Now we pretty much have all the basic interactions implemented! There is one minor but important thing to do though. We need to make sure that the our ball-paddle collision doesn't continuously trigger. This sometimes happens when the ball hits the paddle on the top or bottom side as the ball is not fast enough to escape the paddle in the X-axis sometimes, it continuously reverses its direction until it can escape the paddle.

So! To fix that we just need a little book-keeping in our `Game` type. First, we keep track of whether ball is currently colliding with the paddles inside a couple of `bool` variables.

```go
// game.go
...

type Game struct {
	running bool    // Whether the game is running or not
	paddle1 *Paddle // Paddle for player 1
	paddle2 *Paddle // Paddle for player 2
	ball    *Ball
	score1  int // Score for player 1
	score2  int // Score for player 2
	ballCollidingPaddle1 bool
	ballCollidingPaddle2 bool
}

...
```

Then we set the variables and only triggers the direction reversal in the ball-paddle collision function:

```go
// game.go
...

func (game *Game) runBallAndPaddlesCollision() {
	// Run ball-paddle1 collision. Ball should bounce to the right.
	if _, intersects := game.ball.rect.Intersect(&game.paddle1.rect); intersects && !game.ballCollidingPaddle1 {
		game.ball.randomizeVelocity(BallDirectionRight)
		game.ballCollidingPaddle1 = true
	} else {
		game.ballCollidingPaddle1 = false
	}

	// Run ball-paddle2 collision. Ball should bounce to the left.
	if _, intersects := game.ball.rect.Intersect(&game.paddle2.rect); intersects && !game.ballCollidingPaddle2 {
		game.ball.randomizeVelocity(BallDirectionLeft)
		game.ballCollidingPaddle2 = true
	} else {
		game.ballCollidingPaddle2 = false
	}
}

...
```

Alright, that's basically all core game logic implemented! Next we need to show the scores so the players can see what their current scores are during the game. That is where we will learn how to use SDL2_ttf for drawing text!
