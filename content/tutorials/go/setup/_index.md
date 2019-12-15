+++
title = "Setup"
template = "section-tutorials-lang-snippet.html"
sort_by = "weight"
weight = 0
+++

### Linux

On __Ubuntu__, type:
```bash
apt install libsdl2{,-image,-mixer,-ttf,-gfx}-dev
```

On __Fedora__, type:
```bash
yum install SDL2{,_image,_mixer,_ttf,_gfx}-devel
```

On __Arch Linux__, type:
```bash
pacman -S sdl2{,_image,_mixer,_ttf,_gfx}
```

On __Gentoo__, type:
```bash
emerge -av libsdl2 sdl2-{image,mixer,ttf,gfx}
```

### macOS

On __macOS__, install SDL2 via [Homebrew](http://brew.sh) like so:
```bash
brew install sdl2{,_image,_mixer,_ttf,_gfx} pkg-config
```

### Windows

On __Windows__,
1. Install mingw-w64 from [Mingw-builds](http://mingw-w64.org/doku.php/download/mingw-builds)
    * Version: latest (at time of writing 6.3.0)
    * Architecture: x86_64
    * Threads: win32
    * Exception: seh
    * Build revision: 1
    * Destination Folder: Select a folder that your Windows user owns
2. Install SDL2 http://libsdl.org/download-2.0.php
    * Extract the SDL2 folder from the archive using a tool like [7zip](http://7-zip.org)
    * Inside the folder, copy the `i686-w64-mingw32` and/or `x86_64-w64-mingw32` depending on the architecture you chose into your mingw-w64 folder e.g. `C:\Program Files\mingw-w64\x86_64-6.3.0-win32-seh-rt_v5-rev1\mingw64`
3. Setup Path environment variable
    * Put your mingw-w64 binaries location into your system Path environment variable. e.g. `C:\Program Files\mingw-w64\x86_64-6.3.0-win32-seh-rt_v5-rev1\mingw64\bin` and `C:\Program Files\mingw-w64\x86_64-6.3.0-win32-seh-rt_v5-rev1\mingw64\x86_64-w64-mingw32\bin`
4. Open up a terminal such as `Git Bash` and run `go get -v github.com/veandco/go-sdl2/sdl`.
5. (Optional) You can repeat __Step 2__ for [SDL_image](https://www.libsdl.org/projects/SDL_image), [SDL_mixer](https://www.libsdl.org/projects/SDL_mixer), [SDL_ttf](https://www.libsdl.org/projects/SDL_ttf)
    * NOTE: pre-build the libraries for faster compilation by running `go install github.com/veandco/go-sdl2/{sdl,img,mix,ttf}`

* Or you can install SDL2 via [Msys2](https://msys2.github.io) like so:
```
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-SDL2{,_image,_mixer,_ttf,_gfx}
```

## Installation

To get the bindings, type:
```bash
go get -v github.com/veandco/go-sdl2/sdl`
go get -v github.com/veandco/go-sdl2/img`
go get -v github.com/veandco/go-sdl2/mix`
go get -v github.com/veandco/go-sdl2/ttf`
go get -v github.com/veandco/go-sdl2/gfx`
```

or type this if you use Bash terminal:
```bash
go get -v github.com/veandco/go-sdl2/{sdl,img,mix,ttf}
```