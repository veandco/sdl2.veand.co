+++
title = "Images"
weight = 6

[taxonomies]
categories = ["go"]
tags = ["pong"]
+++

## Loading our images

Previously, we have been drawing the game entities using simple white rectangles. However, we can do a little better! We can make the paddles and ball glow to give them a little less bland look.

```go
// main.go
...

const (
	...


	PaddleGlowRedImagePath = "assets/images/paddle-glow-red.png"
	BallGlowYellowImagePath = "assets/images/ball-glow-yellow.png"
)

var (
	...

	paddleGlowRedImage *sdl.Surface
	ballGlowYellowImage *sdl.Surface
)

// Create run() function that returns an exit code
func run() error {
	var err error

	// Initialize img package
	if err = img.Init(img.INIT_PNG); err != nil {
		return err
	}

	// Load images
	if paddleGlowRedImage, err = img.Load(PaddleGlowRedImagePath); err != nil {
		return err
	}
	defer paddleGlowRedImage.Free()

	if ballGlowYellowImage, err = img.Load(BallGlowYellowImagePath); err != nil {
		return err
	}
	defer ballGlowYellowImage.Free()

	...
}

...
```

## Drawing the images

We now have the images loaded. All we need to do now is just add our little image drawing code to our `Paddle.Draw()` and `Ball.Draw()` methods!

```go
// paddle.go
...

func (paddle *Paddle) Draw(surface *sdl.Surface) {
	...

	// Create another rectangle for the paddle glow image
	rect.X = int32((paddle.rect.X - 0.005) * WindowWidth)
	rect.Y = int32((paddle.rect.Y - 0.005) * WindowHeight)
	rect.W = int32((paddle.rect.W + 0.01) * WindowWidth)
	rect.H = int32((paddle.rect.H + 0.01) * WindowHeight)

	// Draw the glowing image on top the rectangle
	paddleGlowRedImage.BlitScaled(nil, surface, &rect)
}
```

```go
// ball.go
...

func (ball *ball) Draw(surface *sdl.Surface) {
	...

	// Create another rectangle for the ball glow image
	rect.X = int32((ball.rect.X - 0.005) * WindowWidth)
	rect.Y = int32((ball.rect.Y - 0.005) * WindowHeight)
	rect.W = int32((ball.rect.W + 0.01) * WindowWidth)
	rect.H = int32((ball.rect.H + 0.01) * WindowHeight)

	// Draw the glowing image on top the rectangle
	ballGlowYellowImage.BlitScaled(nil, surface, &rect)
}
```
