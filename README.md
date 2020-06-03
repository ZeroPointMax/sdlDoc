# Nutzung der SDL-Library unter CMake

Dieses Dokument wird doppelt versioniert:

- [Github](https://github.com/ZeroPointMax/sdlDoc)
- [ZPMs Kallithea](https://hg.hobbyist-overclock.de/DHGE/sdlsetup)

// Maximilian Kerst 12/2019

// Yannis Becker 06/2020

## Installieren von SDL2

Wir brauchen die Pakete sdl2 und sdl2_image.
Installation auf Linux: distributionsspezifisch, sollte aber schon installiert sein

## Einbinden von SDL2 in CMake am Beispiel von CLion

### Linux

folgenden Block in die CMakeLists.txt einfügen:

```cmake
INCLUDE(FindPkgConfig)

PKG_SEARCH_MODULE(SDL2 REQUIRED sdl2)
PKG_SEARCH_MODULE(SDL2IMAGE REQUIRED SDL2_image>=2.0.0)

INCLUDE_DIRECTORIES(${SDL2_INCLUDE_DIRS} ${SDL2IMAGE_INCLUDE_DIRS})
TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${SDL2_LIBRARIES} ${SDL2IMAGE_LIBRARIES})
```

die Datei sollte etwa so aussehen: 

```cmake
cmake_minimum_required(VERSION 3.15)
project(sdltest C)

set(CMAKE_C_STANDARD 11)

add_executable(sdltest main.c)

INCLUDE(FindPkgConfig)

PKG_SEARCH_MODULE(SDL2 REQUIRED sdl2)
PKG_SEARCH_MODULE(SDL2IMAGE REQUIRED SDL2_image>=2.0.0)

INCLUDE_DIRECTORIES(${SDL2_INCLUDE_DIRS} ${SDL2IMAGE_INCLUDE_DIRS})
TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${SDL2_LIBRARIES} ${SDL2IMAGE_LIBRARIES})
```

<a href=https://stackoverflow.com/questions/23850472/how-to-use-sdl2-and-sdl-image-with-cmake>Quelle: Stackoverflow</a>


Zum testen kann man folgenden Code ausführen; nicht schön, aber erfüllt seinen Zweck:

```c
#include <stdio.h>
#include <SDL.h>
int main(int argc, char* argv []) {
    if (SDL_Init(SDL_INIT_EVERYTHING) != 0) {
        printf("Error initializing SDL!\n");
        return 1;
    }
    SDL_Quit();
    return 0;
}
```

Wenn der Returncode 0 ist, funktioniert SDL2.

### Windows

```PKG_SEARCH_MODULE``` ist unter Windows nicht verfügbar. Daher ist es notwendig SDL2 über einen Pfad direkt einzubinden.

```cmake
cmake_minimum_required(VERSION 3.15)
project(sdltest C)

set(CMAKE_C_STANDARD 11)

add_executable(sdltest main.c)

set(SDL2_DIR C:/dev/SDL2)
set(SDL2_LIB_DIR ${SDL2_DIR}/lib)

include_directories(${SDL2_DIR}/include)
target_link_libraries(${PROJECT_NAME} ${SDL2_LIB_DIR}/libSDL2.dll.a ${SDL2_LIB_DIR}/libSDL2main.a -mwindows)
```

## Einbinden von C(++) Quell/-Headerdateien in ein CMake-Projekt am Beispiel von sdlinterf

- die gewünschten Dateien in einen neuen Ordner innerhalb des Projektes ablegen, z.B. in lib für Quelldateien oder include für Header-Dateien
- in der CMakeLists.txt...
  - ``INCLUDE_DIRECTORIES`` um ``${PROJECT_SOURCE_DIR}/include`` erweitern, damit CMake die Header-Dateien berücksichtigt. Bei abweichenden Pfaden muss diese Zeile angepasst werden
  - ``add_executable [projektname] [quelldateien.c]`` um die hinzugefügten Quelldateien, z.B. ``lib/sdlinterf.c`` erweitern, damit CMake diese auch baut

Die CMakeLists.txt auf einem Linux-System sollte damit wie folgt aussehen (Windows analog):

```cmake
cmake_minimum_required(VERSION 3.15)
project(sdltest C)

set(CMAKE_C_STANDARD 11)

add_executable(sdltest main.c lib/sdlinterf.c)

INCLUDE(FindPkgConfig)

PKG_SEARCH_MODULE(SDL2 REQUIRED sdl2)
PKG_SEARCH_MODULE(SDL2IMAGE REQUIRED SDL2_image>=2.0.0)

INCLUDE_DIRECTORIES(${SDL2_INCLUDE_DIRS} ${SDL2IMAGE_INCLUDE_DIRS} ${PROJECT_SOURCE_DIR}/include)
TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${SDL2_LIBRARIES} ${SDL2IMAGE_LIBRARIES})
```

