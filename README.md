# Nutzung der SDL-Library unter CMake

// Maximilian Kerst 12/2019

// Yannis Becker 06/2020

## Installieren von SDL2

Wir brauchen die Pakete sdl2 und sdl2_image.

Installation auf Linux: distributionsspezifisch, sollte aber schon installiert sein

Installation auf Windows:

- Herunterladen der **32 Bit** Binaries von der [offiziellen Seite](https://www.libsdl.org/download-2.0.php)
- die .tar.gz mit einer Archivsoftware öffnen (ich empfehle [7-zip](https://7-zip.org))
- SDL2 entweder in die MinGW Ordnerstruktur integrieren ODER irgendwohin entpacken und Pfad merken

Sollte SDL2 irgendwo hin entpackt worden sein, funktioniert ``FindPkgConfig`` eventuell nicht mehr

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

#### Direkt einbinden

```PKG_SEARCH_MODULE``` muss unter MinGW erst installiert werden. Einfacher ist es, SDL2 über einen Pfad direkt einzubinden.

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

#### mit FindPkgConfig

Voraussetzung: eine installierte MinGW Entwicklungsumgebung

[PkgConfig](http://ftp.gnome.org/pub/gnome/binaries/win32/dependencies/pkg-config_0.26-1_win32.zip)

[GLib](http://ftp.gnome.org/pub/gnome/binaries/win32/glib/2.28/glib_2.28.8-1_win32.zip)

[GetText Runtime](http://ftp.gnome.org/pub/gnome/binaries/win32/dependencies/gettext-runtime_0.18.1.1-2_win32.zip)

-----

[Quelle: Stackoverflow](https://stackoverflow.com/questions/1710922/how-to-install-pkg-config-in-windows)

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

