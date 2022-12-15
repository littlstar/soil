Simple OpenGL Image Library (SOIL)
==================================

## Introduction

SOIL is a tiny C library used primarily for uploading textures into OpenGL.
It is based on `stb_image` version 1.16, the public domain code from Sean
Barrett (found here). It has been extended to load TGA and DDS files, and
to perform common functions needed in loading OpenGL textures. SOIL can
also be used to save and load images in a variety of formats.


## Installation

### With [clib](https://github.com/clibs/clib):

```sh
$ clib install littlstar/soil --save
```

### With automake

```sh
$ ./configure
$ make
$ make install
```

### With CMake

```sh
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=<SOIL_INSTALL_PREFIX> # /usr or /usr/local by default
$ cmake --build . --target install
```

## Build CMake-based downstream projects with SOIL

Where are two ways:
+ build/install SOIL and descover SOIL's CMake target (`SOIL::libsoil`) with `find_package(SOIL)` within your project
+ build SOIL as part of your project with `add_subdirectory(soil)` and automatically have `SOIL::libsoil` inside

### First way

#### System-wide installation (Unix only)

Within SOIL project's top directory:
```sh
$ mkdir build
$ cd build
$ cmake .. # use -DBUILD_SHARED_LIBS=TRUE to build SOIL as shared library (static is default)
$ cmake --build . --target install
```
This will build and install SOIL system-wide and your CMake-based project can now consume SOIL like that:
```cmake
...
find_package(SOIL 0.1.1)
...
add_executable(my_exe main.cpp)
target_link_libraries(my_exe ... PRIVATE SOIL::libsoil ...)
...
```

#### Installation with custom prefix

Within SOIL project's top directory:
```sh
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=<SOIL_INSTALL_PREFIX>
$ cmake --build . --target install
```
Then You can discover and link against SOIL library target inside of your project CMakeLists.txt
just like with system-wide installation:
```cmake
...
find_package(SOIL 0.1.1)
...
add_executable(my_exe main.cpp)
target_link_libraries(my_exe ... PRIVATE SOIL::libsoil ...)
...
```
Unlike the system-wide installation You should now configure your project like this:
```sh
cmake <PROJECT_SOURCE_DIR> -DSOIL_DIR=<SOIL_INSTALL_PREFIX>
```

#### Without installation

Within SOIL project's top directory:
```sh
$ mkdir build
$ cd build
$ cmake ..
$ cmake --build .
```
This will build SOIL target and configs inside of `<SOIL_PROJECT_DIR>/build`.

Then discover and link against SOIL just as usual:
```cmake
...
find_package(SOIL 0.1.1)
...
add_executable(my_exe main.cpp)
target_link_libraries(my_exe ... PRIVATE SOIL::libsoil...)
...
```
And configure your project like this:
```sh
cmake <PROJECT_SOURCE_DIR> -DSOIL_DIR=<SOIL_PROJECT_DIR>/build
```

### Second way

Copy SOIL project dir into your project dir or add it as git submodule and add it
as subdirectory in your project's CMakeLists.txt and link against SOIL::libsoil:
```cmake
...
add_subdirectory(soil)
...
add_executable(my_exe main.cpp)
...
target_link_libraries(my_exe ... PRIVATE SOIL::libsoil ...)
```
Additionally You can configure SOIL with cutom OpenGL includes and/or libraries.

In order to provide SOIL with your custom OpenGL header
(assuming include directory for that header is accessible project-wide)
add these lines before `add_subdirectory(soil)`:
```cmake
set(SOIL_CUSTOM_GL_HEADERS TRUE)
set(SOIL_CUSTOM_GL_HEADERS_DEFS
"#include <my_ogl_header.h>"
)
```

For example You build your project using SDL2 and glad for Android
and You want to provide SOIL with proper OpenGL headers. In this case You
also need to provide SOIL with proper libraries to interface with.
Add these lines before `add_subdirectory(soil)`:
```cmake
set(SOIL_GL_LINK_LIBRARIES "INTERFACE" SDL2::SDL2-static glad)
set(SOIL_GL_INCLUDE_DIRECTORIS "PRIVATE"
	$<TARGET_PROPERTY:SDL2::SDL2-static,INTERFACE_INCLUDE_DIRECTORIES>
	$<TARGET_PROPERTY:glad,INTERFACE_INCLUDE_DIRECTORIES>
)
set(SOIL_CUSTOM_GL_HEADERS TRUE)
set(SOIL_CUSTOM_GL_HEADERS_DEFS
"
#include <glad/gles2.h>
#include <SDL_opengles2.h>
#ifndef APIENTRY
#define APIENTRY GLAPIENTRY
#endif
"
)
```

## Usage

SOIL is meant to be used as a static library (as it's tiny and in the public domain).

Simply `#include <SOIL/SOIL.h>` in your C or C++ file, link in the static
library, and then use any of SOIL's functions. The file `<SOIL/SOIL.h>`
contains simple doxygen style documentation (which will be built as html in case of
CMake build).
If You use the static library, no other header files are needed besides `SOIL.h`.


## Features

* No external dependencies
* Tiny
* Cross platform (Windows, \*nix, Mac OS X)
* Public Domain
* Can load an image file directly into a 2D OpenGL texture
* Can generate a new texture handle, or reuse one specified
* Can automatically rescale the image to the next largest power-of-two size
* Can automatically create MIPmaps
* Can scale (not simply clamp) the RGB values into the "safe range" for NTSC displays (16 to 235, as recommended [here][1])
* Can multiply alpha on load (for more correct blending / compositing)
* Can flip the image vertically
* Can compress and upload any image as DXT1 or DXT5 (if `EXT_texture_compression_s3tc` is available), using an internal (very fast!) compressor
* Can convert the RGB to YCoCg color space (useful with DXT5 compression: see [this link][2] from NVIDIA)
* Will automatically downsize a texture if it is larger than `GL_MAX_TEXTURE_SIZE`
* Can directly upload DDS files (DXT1/3/5/uncompressed/cubemap, with or without MIPmaps). Note: directly uploading the compressed DDS image will disable the other options (no flipping, no pre-multiplying alpha, no rescaling, no creation of MIPmaps, no auto-downsizing)
* Can load rectangluar textures for GUI elements or splash screens (requires `GL_ARB/EXT/NV_texture_rectangle`)
* Can decompress images from RAM (e.g. via [PhysicsFS][3] or similar) into an OpenGL texture (same features as regular 2D textures, above)
* Can load cube maps directly into an OpenGL texture (same features as regular 2D textures, above)
* Can take six image files directly into an OpenGL cube map texture
* Can take a single image file where `width = 6 * height` (or vice versa), split it into an OpenGL cube map texture

### Readable Image Formats

* BMP - non-1bpp, non-RLE (from `stb_image` documentation)
* PNG - non-interlaced (from `stb_image` documentation)
* JPG - JPEG baseline (from `stb_image` documentation)
* TGA - greyscale or RGB or RGBA or indexed, uncompressed or RLE
* DDS - DXT1/2/3/4/5, uncompressed, cubemaps (can't read 3D DDS files yet)
* PSD - (from `stb_image` documentation)
* HDR - converted to LDR, unless loaded with *HDR* functions (RGBE or RGBdivA or RGBdivA2)

### Writeable Image Formats
* TGA - Greyscale or RGB or RGBA, uncompressed
* BMP - RGB, uncompressed
* DDS - RGB as DXT1, or RGBA as DXT5

## License

Public Domain

Originally sourced from https://github.com/paralin/soil

[1]: http://msdn2.microsoft.com/en-us/library/bb174608.aspx#NTSC_Suggestions
[2]: http://developer.nvidia.com/object/real-time-ycocg-dxt-compression.html
[3]: http://icculus.org/physfs/
