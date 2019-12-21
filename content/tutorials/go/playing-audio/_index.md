+++
title = "Playing audio"
template = "section-tutorials-lang-snippet.html"
sort_by = "weight"
weight = 4
+++

## Playing audio

```go
package main

// typedef unsigned char Uint8;
// void SineWave(void *userdata, Uint8 *stream, int len);
import "C"
import (
	"log"
	"math"
	"os"
	"os/signal"
	"reflect"
	"syscall"
	"unsafe"

	"github.com/veandco/go-sdl2/sdl"
)

const (
	DefaultFrequency = 16000
	DefaultFormat    = sdl.AUDIO_S16
	DefaultChannels  = 2
	DefaultSamples   = 512

	toneHz = 440
	dPhase = 2 * math.Pi * toneHz / DefaultSamples
)

//export SineWave
func SineWave(userdata unsafe.Pointer, stream *C.Uint8, length C.int) {
	n := int(length) / 2
	hdr := reflect.SliceHeader{Data: uintptr(unsafe.Pointer(stream)), Len: n, Cap: n}
	buf := *(*[]C.ushort)(unsafe.Pointer(&hdr))

	var phase float64
	for i := 0; i < n; i++ {
		phase += dPhase
		sample := C.ushort((math.Sin(phase) + 0.999999) * 32768)
		buf[i] = sample
	}
}

func main() {
	var dev sdl.AudioDeviceID
	var err error

	// Initialize SDL2
	if err := sdl.Init(sdl.INIT_AUDIO); err != nil {
		log.Println(err)
		return
	}
	defer sdl.Quit()

	// Specify the configuration for our default playback device
	spec := sdl.AudioSpec{
		Freq:     DefaultFrequency,
		Format:   DefaultFormat,
		Channels: DefaultChannels,
		Samples:  DefaultSamples,
		Callback: sdl.AudioCallback(C.SineWave),
	}

	// Open default playback device
	if dev, err = sdl.OpenAudioDevice("", false, &spec, nil, 0); err != nil {
		log.Println(err)
		return
	}
	defer sdl.CloseAudioDevice(dev)

	// Start playback audio
	sdl.PauseAudioDevice(dev, false)

	// Listen to OS signals
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)

	// Run infinite loop until we receive SIGINT or SIGTERM!
	running := true
	for running {
		select {
		case sig := <-c:
			log.Printf("Received signal %v. Exiting.\n", sig)
			running = false
		}
	}
}
```

## Playing WAV file

```go
package main

// typedef unsigned char Uint8;
// void OnAudioPlayback(void *userdata, Uint8 *stream, int len);
import "C"
import (
	"log"
	"os"
	"os/signal"
	"reflect"
	"syscall"
	"unsafe"

	"github.com/veandco/go-sdl2/sdl"
)

var (
	audio  []byte
	offset int // We use this to keep track of which part of audio to play
)

//export OnAudioPlayback
func OnAudioPlayback(userdata unsafe.Pointer, stream *C.Uint8, length C.int) {
	n := int(length)
	hdr := reflect.SliceHeader{Data: uintptr(unsafe.Pointer(stream)), Len: n, Cap: n}
	buf := *(*[]byte)(unsafe.Pointer(&hdr))
	for i := 0; i < n; i++ {
		buf[i] = audio[offset]
		offset = (offset + 1) % len(audio) // Increase audio offset and loop when it reaches the end
	}
}

func main() {
	var dev sdl.AudioDeviceID
	var err error

	// Initialize SDL2
	if err := sdl.Init(sdl.INIT_AUDIO); err != nil {
		log.Println(err)
		return
	}
	defer sdl.Quit()

	// Load WAV audio file
	tmpaudio, spec := sdl.LoadWAV("../../assets/test.wav")
	if spec == nil {
		log.Println(sdl.GetError())
		return
	}
	audio = tmpaudio

	spec.Callback = sdl.AudioCallback(C.OnAudioPlayback)

	// Open default playback device
	if dev, err = sdl.OpenAudioDevice("", false, spec, nil, 0); err != nil {
		log.Println(err)
		return
	}
	defer sdl.CloseAudioDevice(dev)

	// Start playback audio
	sdl.PauseAudioDevice(dev, false)

	// Listen to OS signals
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)

	// Run infinite loop until we receive SIGINT or SIGTERM!
	running := true
	for running {
		select {
		case sig := <-c:
			log.Printf("Received signal %v. Exiting.\n", sig)
			running = false
		}
	}
}
```

## Playing MP3 and WAV with SDL2_mixer

```go
package main

import "C"
import (
	"log"
	"os"
	"os/signal"
	"syscall"

	"github.com/veandco/go-sdl2/mix"
	"github.com/veandco/go-sdl2/sdl"
)

func main() {
	// Initialize SDL2
	if err := sdl.Init(sdl.INIT_AUDIO); err != nil {
		log.Println(err)
		return
	}
	defer sdl.Quit()

	// Initialize SDL2 mixer
	if err := mix.Init(mix.INIT_MP3); err != nil {
		log.Println(err)
		return
	}
	defer mix.Quit()

	// Open default playback device
	if err := mix.OpenAudio(mix.DEFAULT_FREQUENCY, mix.DEFAULT_FORMAT, mix.DEFAULT_CHANNELS, mix.DEFAULT_CHUNKSIZE); err != nil {
		log.Println(err)
		return
	}
	defer mix.CloseAudio()

	// Load compressed audio file (like MP3) with long duration as *mix.Music
	music, err := mix.LoadMUS("../../assets/test.mp3")
	if err != nil {
		log.Println(err)
	}
	defer music.Free()

	// Load WAV file with short duration as *mix.Chunk
	chunk, err := mix.LoadWAV("../../assets/test.wav")
	if err != nil {
		log.Println(err)
	}
	defer chunk.Free()

	// Play the music once. Change 0 to -1 for infinite playback!.
	if err := music.Play(0); err != nil {
		log.Println(err)
	}

	// Play the chunk once. Change 0 to -1 for infinite playback!
	if _, err := chunk.Play(-1, 0); err != nil {
		log.Println(err)
	}

	// Listen to OS signals
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)

	// Run infinite loop until we receive SIGINT or SIGTERM!
	running := true
	for running {
		select {
		case sig := <-c:
			log.Printf("Received signal %v. Exiting.\n", sig)
			running = false
		}
	}
}
```