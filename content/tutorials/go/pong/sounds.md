+++
title = "Sounds"
weight = 4

[taxonomies]
categories = ["go"]
tags = ["pong"]
+++

## Loading sound files

To play sounds, first we have to initialize SDL2_mixer and load our sounds in `main.go`:

```go
// main.go
package main

import (
	...

	"github.com/veandco/go-sdl2/mix" // Import the SDL2_mix binding
)

const (
	...

	BallPaddleSoundPath = "assets/sounds/ping_pong_8bit_beeep.ogg"
	BallWallSoundPath   = "assets/sounds/ping_pong_8bit_plop.ogg"
	BallScoreSoundPath  = "assets/sounds/ping_pong_8bit_peeeeeep.ogg"
)

var (
	ballPaddleSound *mix.Chunk
	ballWallSound   *mix.Chunk
	ballScoreSound  *mix.Chunk
)

// Create run() function that returns an exit code
func run() error {
	var err error

	// Initialize mix package with OGG support
	if err = mix.Init(mix.INIT_OGG); err != nil {
		return err
	}

	// Open our audio device
	if err = mix.OpenAudio(mix.DEFAULT_FREQUENCY, mix.DEFAULT_FORMAT, mix.DEFAULT_CHANNELS, 512); err != nil {
		return err
	}
	defer mix.CloseAudio()

	// Load the sound files
	if ballPaddleSound, err = mix.LoadWAV(BallPaddleSoundPath); err != nil {
		return err
	}
	defer ballPaddleSound.Free()

	if ballWallSound, err = mix.LoadWAV(BallWallSoundPath); err != nil {
		return err
	}
	defer ballWallSound.Free()

	if ballScoreSound, err = mix.LoadWAV(BallScoreSoundPath); err != nil {
		return err
	}
	defer ballScoreSound.Free()

	
	...
}

...
```

## Playing the sounds

Now that we have loaded the sound files, we can proceed to play them!

For the `ballPaddleSound`, we'll have it play when the ball hits the paddle:

```go
// game.go
...

func (game *Game) runBallAndPaddlesCollision() {
	// Run ball-paddle1 collision. Ball should bounce to the right.
	if _, intersects := game.ball.rect.Intersect(&game.paddle1.rect); intersects && !game.ballCollidingPaddle1 {
		game.ball.randomizeVelocity(BallDirectionRight)
		game.ballCollidingPaddle1 = true
		ballPaddleSound.Play(-1, 0)
	} else {
		game.ballCollidingPaddle1 = false
	}

	// Run ball-paddle2 collision. Ball should bounce to the left.
	if _, intersects := game.ball.rect.Intersect(&game.paddle2.rect); intersects && !game.ballCollidingPaddle2 {
		game.ball.randomizeVelocity(BallDirectionLeft)
		game.ballCollidingPaddle2 = true
		ballPaddleSound.Play(-1, 0)
	} else {
		game.ballCollidingPaddle2 = false
	}
}

...
```

And for the `ballWallSound`, we'll have it play when the ball hits the wall or the top and bottom sides of the window:

```go
// game.go
...

func (game *Game) runBallAndWallsCollision() {
	...

	if ballIsOutsideWindow {
		...

		if game.ball.rect.Y >= 0 && game.ball.rect.Y+game.ball.rect.H < 1 {
			...

		} else if game.ball.rect.X >= 0 && game.ball.rect.X+game.ball.rect.W < 1 {
			game.ball.velocity.Y = -game.ball.velocity.Y
			ballWallSound.Play(-1, 0)
		}
	}
}

...
```

Finally for the `ballScoreSound`, we'll have it play when the ball hits the sides of the window:

```go
// game.go
...

func (game *Game) runBallAndWallsCollision() {
	...

	if ballIsOutsideWindow {
		if game.ball.rect.Y >= 0 && game.ball.rect.Y+game.ball.rect.H < 1 {

			// The ball hit the right wall so player 1 should score
			if game.ball.rect.X > 0.5 {
				game.score1++
				ballScoreSound.Play(-1, 0)

			// The ball hit the left wall so player 2 should score
			} else {
				game.score2++
				ballScoreSound.Play(-1, 0)
			}

			...
		} else if game.ball.rect.X >= 0 && game.ball.rect.X+game.ball.rect.W < 1 {
			...
		}
}

...
```
