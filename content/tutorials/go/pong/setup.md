+++
title = "Setup"
weight = 0

[taxonomies]
categories = ["go"]
tags = ["pong"]
+++

## Introduction

Welcome to an SDL2 tutorial for Go! The goal of this tutorial is to provide a quick overview of SDL2 capabilities which includes the base SDL2, SDL2_ttf, SDL2_mixer, and SDL2_image through creation of a Pong-like game.

## Preparation

Let's first set up our working directory. Then, download the assets [here](/assets/tutorials/go/pong/assets.zip) and extract the content into the working directory. Credits to [Pixel Kitchen](https://www.fontspace.com/pixel-kitchen) for the [Satella Font](https://www.fontspace.com/satella-font-f40960) and [captaincrunch80](https://opengameart.org/users/captaincrunch80) for the [Pong sounds](https://opengameart.org/content/3-ping-pong-sounds-8-bit-style). The images were simply created in GIMP and inspired by [](https://opengameart.org/content/pong-graphics) which is authored by [Deathsbreed](https://opengameart.org/users/deathsbreed).

## Creating the window

We can start by creating our `main.go` file:

```go
// main.go
package main

import (
	"log" // For printing errors
	"os"  // For exiting program with code

	"github.com/veandco/go-sdl2/sdl" // Import the SDL2 binding
)

// Define several constants so we can change them later easily if we want
const (
	WindowTitle  = "Pong"
	WindowWidth  = 800
	WindowHeight = 600

	FPS = 60
)

// Create run() function that returns an error
func run() error {
	// Initialize sdl package
	if err := sdl.Init(sdl.INIT_EVERYTHING); err != nil {
		return err
	}
	defer sdl.Quit()

	// Create a window for displaying our game
	window, err := sdl.CreateWindow(WindowTitle, sdl.WINDOWPOS_UNDEFINED, sdl.WINDOWPOS_UNDEFINED, WindowWidth, WindowHeight, sdl.WINDOW_SHOWN)
	if err != nil {
		return err
	}
	defer window.Destroy() // Destroy window when this function returns

	// Run our game loop. We will define it in another file!
	NewGame().Run(window)

	return nil
}

// Where it all begins!
func main() {
	if err := run(); err != nil {
		log.Println(err)
		os.Exit(1)
	}
}
```

## Game skeleton

Now that we have a high-level view of how the game will run. Let's define the `Game` type next! We will simply define enough of the `Game` type so the program compiles and run, showing the window, and capable of quitting if the close button on the window is pressed or the Escape key is pressed.

```go
// game.go
package main

import (
	"time"

	"github.com/veandco/go-sdl2/sdl"
)

type Game struct {
	running bool // Whether the game is running or not
}

func NewGame() *Game {
	return &Game{
		running: true,
	}
}

// We will call most of our other game-related code inside this function
func (game *Game) Run(window *sdl.Window) {
	currentTimeNs := time.Now().UnixNano() // Current time in nanoseconds

	// Get the surface for our game to draw on
	surface, _ := window.GetSurface()

	for game.running {
		previousTimeNs := currentTimeNs // Keep tracks of previous time in nanoseconds
		currentTimeNs = time.Now().UnixNano()

		// Clear the window for new drawings
		surface.FillRect(nil, 0xFF000000)

		// Handle input such as window and keyboard events
		game.HandleInput()

		// Our game logic and rendering code here!
		game.Update(float32(currentTimeNs-previousTimeNs) / 1000000000) // Convert the delta time to seconds
		game.Draw(surface)

		// Tell SDL2 to update our window's surface
		window.UpdateSurface()

		// Let the processor sleep for awhile to match our FPS
		sdl.Delay(1000 / FPS)
	}
}

func (game *Game) HandleInput() {
	// Handle input received from events such as window close or keyboard presses
	for event := sdl.PollEvent(); event != nil; event = sdl.PollEvent() {
		switch ev := event.(type) {
		// Quit the game if window is closed
		case *sdl.QuitEvent:
			game.running = false

		// Quit the game if the Escape key is released
		case *sdl.KeyboardEvent:
			// Get the keyboard press event's virtual key
			sym := ev.Keysym.Sym
			if sym == sdl.K_ESCAPE {
				game.running = false
			}
		}
	}
}

func (game *Game) Update(deltaTime float32) {
	// TODO: We will update paddle, ball and other things here!
}

func (game *Game) Draw(surface *sdl.Surface) {
	// TODO: We will draw paddle, ball and other things here!
}
```

For now, we have finished our initial game skeleton and have general idea of how the game will run. Next, we shall draw our game entities which are the paddles and the ball!
