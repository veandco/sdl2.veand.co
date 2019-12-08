+++
title = "Input"
weight = 2

[taxonomies]
categories = ["go"]
tags = ["pong"]
+++

## Moving our paddles

Let's update our `Game.HandleInput()` function to handle key presses for moving paddles!

```go
// game.go
...

func (game *Game) HandleInput() {
	// Handle input received from events such as window close or keyboard presses
	for event := sdl.PollEvent(); event != nil; event = sdl.PollEvent() {
		switch ev := event.(type) {
		// Quit the game if window is closed
		case *sdl.QuitEvent:
			game.running = false
		case *sdl.KeyboardEvent:
			// Get the keyboard press event's virtual key
			sym := ev.Keysym.Sym

			// Move the paddles if their corresponding keys are pressed
			if ev.State == sdl.PRESSED {
				// Paddle 1
				if sym == sdl.K_w {
					game.paddle1.velocity.Y = -PaddleSpeed
				} else if sym == sdl.K_s {
					game.paddle1.velocity.Y = PaddleSpeed

					// Paddle 2
				} else if sym == sdl.K_UP {
					game.paddle2.velocity.Y = -PaddleSpeed
				} else if sym == sdl.K_DOWN {
					game.paddle2.velocity.Y = PaddleSpeed
				}

				// Stop the paddles if their corresponding keys are released
			} else if ev.State == sdl.RELEASED {
				// Paddle 1
				if sym == sdl.K_w && game.paddle1.velocity.Y < 0 || sym == sdl.K_s && game.paddle1.velocity.Y > 0 {
					game.paddle1.velocity.Y = 0

					// Paddle 2
				} else if sym == sdl.K_UP && game.paddle2.velocity.Y < 0 || sym == sdl.K_DOWN && game.paddle2.velocity.Y > 0 {
					game.paddle2.velocity.Y = 0

					// Quit the game if the Escape key is released
				} else if sym == sdl.K_ESCAPE {
					game.running = false
				}
			}
		}
	}
}

...
```