# Nutzung der SDL-Library unter CMake

Dieses Dokument wird doppelt versioniert:

- [Github](https://github.com/ZeroPointMax/sdlDoc)
- [ZPMs Kallithea](https://hg.hobbyist-overclock.de/DHGE/sdlsetup)

// Maximilian Kerst 12/2019

// Yannis Becker 06/2020

## Installieren von SDL2

Wir brauchen die Pakete sdl2 und sdl2_image.

### Installation auf Linux:

distributionsspezifisch, sollte aber schon installiert sein.

Beispiele:

```bash
sudo pacman -S sdl2 sdl2_image # Arch-Familie
sudo apt install libsdl2-dev libsdl2-2.0-0 libjpeg-dev libwebp-dev libtiff5-dev libsdl2-image-dev libsdl2-image-2.0-0 # Ubuntu-Familie ab 18.04
```

### Installation auf Windows:

#### Bestimmen der MinGW-Architektur

Die MinGW Architektur kann von der Windows-Architektur abweichen. Das wird für die Installation relevant, da eine falsche Architektur der Pakete nicht funktionieren wird (und man diese anschließend wieder rausfummeln darf).

Einfachster Fall: Windows ist 32 Bit - MinGW ist auch 32 Bit, doh.

Für 64 Bit Windows:

- Powershell oder CMD öffnen
- ``gcc --version`` ausführen

Die erste Zeile der Ausgabe verrät, welche Architektur installiert ist. Beispiel

``gcc.exe (x86_64-posix-seh-rev0, Built by MinGW-W64 project) 8.1.0`` --> hier handelt es sich um ein 64 Bit MinGW

``gcc.exe (MinGW.org GCC-8.2.0-5) 8.2.0`` --> hier handelt es sich um ein 32 Bit MinGW

| Architektur | benötigter Unterordner |
|-------------|------------------------|
| 32 Bit      | i686                   |
| 64 Bit      | x86_64                 |

#### herunterladen und entpacken

- Herunterladen der Development-Pakete von der [offiziellen SDL2-Seite](https://www.libsdl.org/download-2.0.php) für SDL2
- Herunterladen der Development-Pakete von der [offiziellen SDL2_image-Seite](https://www.libsdl.org/projects/SDL_image/)
- die .tar.gz mit einer Archivsoftware öffnen (ich empfehle [7-zip](https://7-zip.org))
- SDL2 und SDL2_image entweder in die MinGW Ordnerstruktur integrieren ODER irgendwohin entpacken und Pfad merken

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

Windows erfordert etwas mehr Handarbeit. Der Quick-And-Dirty Weg ist, die Pfade direkt einzutragen, dann ist das Projekt jedoch nicht mehr portabel.
Je nach Architektur von MinGW (siehe oben) kann man FindPkgConfig nachrüsten.

| MinGW-Architektur | verfügbare Methoden                     |
|-------------------|-----------------------------------------|
|32 Bit             | - direktes Einbinden<br>- FindPkgConfig |
|64 Bit             | direktes Einbinden                      |

Von den nachfolgenden Methoden ist also **eine** umzusetzen.

#### Direkt einbinden

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

Code zum testen:

```c
#include <stdio.h>
#include <SDL.h>
int WinMain(int argc, char* argv []) {
    if (SDL_Init(SDL_INIT_EVERYTHING) != 0) {
        printf("Error initializing SDL!\n");
        return 1;
    }
    SDL_Quit();
    return 0;
}
```

**zu beachten:**
- **SDL.h könnte in einem Unterordner liegen, z.B. "SDL2"**
- **die Main-Funktion heißt jetzt "WinMain"**

#### mit FindPkgConfig

Voraussetzung: eine installierte **32 Bit** MinGW Entwicklungsumgebung

Folgende Pakete sind herunterzuladen:

[PkgConfig](http://ftp.gnome.org/pub/gnome/binaries/win32/dependencies/pkg-config_0.26-1_win32.zip)

[GLib](http://ftp.gnome.org/pub/gnome/binaries/win32/glib/2.28/glib_2.28.8-1_win32.zip)

[GetText Runtime](http://ftp.gnome.org/pub/gnome/binaries/win32/dependencies/gettext-runtime_0.18.1.1-2_win32.zip)

-----

[Quelle: Stackoverflow](https://stackoverflow.com/questions/1710922/how-to-install-pkg-config-in-windows)

- die heruntergeladenen Zip-Files in den MinGW Installationspfad entpacken (z.B. ``C:\Mingw``), sodass die Verzeichnisse mergen (``bin`` mit ``bin``, usw.)
- SDL kann nun wie im Linux-Abschnitt beschrieben eingebunden werden

## Einbinden von C(++) Quell/-Headerdateien in ein CMake-Projekt am Beispiel von sdlinterf

- die gewünschten Dateien in einen neuen Ordner innerhalb des Projektes ablegen, z.B. in lib für Quelldateien oder include für Header-Dateien
- in der CMakeLists.txt...
  - ``INCLUDE_DIRECTORIES`` um ``${PROJECT_SOURCE_DIR}/include`` erweitern, damit CMake die eben hinzugefügten Header-Dateien berücksichtigt. **Bei abweichenden Pfaden muss diese Zeile angepasst werden**
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

