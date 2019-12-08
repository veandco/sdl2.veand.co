+++
title = "Font"
weight = 5

[taxonomies]
categories = ["go"]
tags = ["pong"]
+++

## Loading fonts

We already have the score data so all that is left to do for drawing the scoreboard is defining the draw function inside `Game`!

But first we have to initialize SDL2_ttf and load our fonts in `main.go`:
```go
// main.go
package main

import (
	...

	"github.com/veandco/go-sdl2/ttf" // Import the SDL2_ttf binding
)

const (
	...
	
	FontPath = "assets/fonts/SatellaRegular-ZVVaz.ttf"
	FontSize = 64
)

var (
	...
	
	// Make font available globally
	font *ttf.Font
)

// Create run() function that returns an exit code
func run() error {
	var err error
	
	// Initialize ttf package
	if err = ttf.Init(); err != nil {
		return err
	}
	
	// Load font
	if font, err = ttf.OpenFont(FontPath, FontSize); err != nil {
		return err
	}
	defer font.Close()
	
	...
}

...
```

## Drawing the scoreboard

After that, we can start drawing our scoreboard! For this tutorial, I have chosen [this font](https://www.fontspace.com/satella-font-f40960) (credits to [Pixel Kitchen](https://www.fontspace.com/pixel-kitchen)) but you can use whichever font you'd like to use!

```go
// game.go
package main

import (
	"log"
	"math/rand"
	"strconv"
	"time"
	
	...
)

...

func (game *Game) Draw(surface *sdl.Surface) {
	...
	
	game.drawScoreboard(surface)
}

func (game *Game) drawScoreboard(surface *sdl.Surface) {
	// Convert our score numbers into strings
	score1 := strconv.Itoa(game.score1)
	score2 := strconv.Itoa(game.score2)

	// Create score 1 surface with white color
	score1Surface, err := font.RenderUTF8Solid(score1, sdl.Color{R: 255, G: 255, B: 255, A: 255})
	if err != nil {
		log.Println(err)
	}

	// Create score 2 surface with white color
	score2Surface, err := font.RenderUTF8Solid(score2, sdl.Color{R: 255, G: 255, B: 255, A: 255})
	if err != nil {
		log.Println(err)
	}

	// Draw score 1 on the window surface
	score1Rect := sdl.Rect{
		X: 0.2 * WindowWidth,
		Y: 0.1 * WindowHeight,
		W: 0.2 * WindowWidth,
		H: 0.1 * WindowHeight,
	}
	score1Surface.Blit(nil, surface, &score1Rect)

	// Draw score 2 on the window surface
	score2Rect := sdl.Rect{
		X: 0.8 * WindowWidth,
		Y: 0.1 * WindowHeight,
		W: 0.2 * WindowWidth,
		H: 0.1 * WindowHeight,
	}
	score2Surface.Blit(nil, surface, &score2Rect)
}
...
```
