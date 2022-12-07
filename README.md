<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Inhaltsverzeichnis**

- [Nutzung der SDL-Library unter CMake](#nutzung-der-sdl-library-unter-cmake)
  - [Installieren von SDL2](#installieren-von-sdl2)
    - [Installation auf Linux:](#installation-auf-linux)
    - [Installation auf Windows:](#installation-auf-windows)
      - [Bestimmen der MinGW-Architektur](#bestimmen-der-mingw-architektur)
      - [herunterladen und entpacken](#herunterladen-und-entpacken)
  - [Einbinden von SDL2 in CMake am Beispiel von CLion](#einbinden-von-sdl2-in-cmake-am-beispiel-von-clion)
    - [Linux](#linux)
    - [Windows](#windows)
      - [Direkt einbinden](#direkt-einbinden)
      - [mit FindPkgConfig](#mit-findpkgconfig)
      - [voll-statisch linken](#voll-statisch-linken)
  - [Einbinden von C(++) Quell/-Headerdateien in ein CMake-Projekt am Beispiel von sdlinterf](#einbinden-von-c-quell-headerdateien-in-ein-cmake-projekt-am-beispiel-von-sdlinterf)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

![TOC Generator](https://github.com/ZeroPointMax/sdlDoc/workflows/TOC%20Generator/badge.svg)

# Nutzung der SDL-Library unter CMake

Jetzt auch auf GitHub Pages: https://zeropointmax.github.io/sdlDoc/

// Maximilian Kerst 12/2019

// Yannis Becker 06/2020

**Hinweis**:

In diesem Tutorial sind viele Pfade und andere Elemente, die auf verschiedenen Systemen abweichen könnten. Daher sind (hoffentlich) alle Pfade, welche ggf. angepasst werden müssen, *kursiv* geschrieben oder anderweitig gekennzeichnet. In diesen Fällen appellieren wir an die Aufmerksamkeit des Lesers, sollte etwas nicht mit Copy-Paste aus diesem Tutorial funktionieren.

Sollten wesentliche Teile des Tutorials fehlen / falsch oder unvollständig sein, bitten wir um ein Issue auf GitHub, damit wir eine Lösung finden können.

Viel Spaß und Erfolg :-)

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

Die erste Zeile der Ausgabe verrät, welche Architektur installiert ist. Beispiel:

``gcc.exe (x86_64-posix-seh-rev0, Built by MinGW-W64 project) 8.1.0`` --> hier handelt es sich um ein 64 Bit MinGW

``gcc.exe (MinGW.org GCC-8.2.0-5) 8.2.0`` --> hier handelt es sich um ein 32 Bit MinGW

| Architektur | benötigter Unterordner |
|-------------|------------------------|
| 32 Bit      | i686                   |
| 64 Bit      | x86_64                 |

#### herunterladen und entpacken

- Herunterladen der Development-Pakete von der [offiziellen SDL2-Seite](https://www.libsdl.org/download-2.0.php) für SDL2
- die .tar.gz mit einer Archivsoftware öffnen (ich empfehle [7-zip](https://7-zip.org))
- **einen** der folgenden Schritte ausführen:

Variante 1: den Inhalt des *benötigten Unterordners* (siehe oben) so in das MinGW-Installationsverzeichnis entpacken, dass die Ordner bin, lib, include usw. im Archiv mit denen von MinGW *verschmelzen*.

Variante 2: den Inhalt des *benötigten Unterordners* (siehe oben) in ein willkürlich gewähltes Verzeichnis entpacken (möglichst kurzer Pfad und ohne Sonderzeichen / Leerzeichen), z.B.: *``C:\SDL2\``*. Hier unbedingt den Pfad merken, wir brauchen ihn später wieder. Ebenso entfällt hier die Möglichkeit, **SDL2 via FindPkgConfig einzubinden**

**Welche Variante nehmen?**

TL;DR: Empfohlen für Programmier-Übungen in der Gruppe: Variante 1

- Der Vorteil von Möglichkeit 1 ist, dass SDL2 nun im gesamten MinGW bekannt und "installiert" ist - wie auf einem \*nix-System (Linux, macOS,...). Darüber hinaus kann man ein CMake-Projekt, welches auf einem Linux-Computer (und dem Linux-Tutorial) erstellt wurde, auch auf einem Windows-Rechner verwenden Der Nachteil: die Deinstallation ist komplizierter
- Der Vorteil von Möglichkeit 2 ist, dass SDL2 in einem separaten Ordner, getrennt vom MinGW ist. So kann man seinen MinGW-Ordner sauberer halten. Die Konsequenz: Man muss CMake erstmal beibringen, wo das SDL2 ist. Da jeder die Installation woanders haben könnte, kann man das Projekt nicht einfach auf einen anderen Rechner kopieren, insofern SDL2 nicht am genau gleichen Ort entpackt wurde.

## Einbinden von SDL2 in CMake am Beispiel von CLion

Hinweis: in diesem Abschnitt sollte das Projekt im CLion schon existieren. Sollten automatisch generierte Teile der CMakeLists.txt von den Beispielen unten abweichen, **sind die automatisch generierten zu verwenden**.

CLion fordert zum Reload der CMakeLists.txt auf, um die Änderungen zu übernehmen. Nach jeder Änderung sollte der Leser dem Nachkommen, da so Fehler in der Datei rechtzeitig bemerkt werden. Des Komforts halber kann auch Auto-Reload aktiviert werden.

### Linux

folgenden Block in die CMakeLists.txt einfügen:

```cmake
INCLUDE(FindPkgConfig)

PKG_SEARCH_MODULE(SDL2 REQUIRED sdl2)
PKG_SEARCH_MODULE(SDL2IMAGE REQUIRED SDL2_image>=2.0.0)

INCLUDE_DIRECTORIES(${SDL2_INCLUDE_DIRS} ${SDL2IMAGE_INCLUDE_DIRS})
TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${SDL2_LIBRARIES} ${SDL2IMAGE_LIBRARIES})
```

Was passiert hier? Zuerst laden wir "FindPkgConfig" - ein Tool, welches die richtigen Pfade für SDL2 automatisch herzaubert. Dann lassen wir es SDL2 und SDL2_image suchen, was uns die Variablen "SDL2\[...\]" liefert. Mit INCLUDE_DIRECTORIES und TARGET_LINK_LIBRARIES weisen wir CMake an, dem Linker SDL2 mitzugeben.

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

Weitere Erläuterungen zum CMakeLists.txt:

- add_executable legt fest, wie die ausführbare Datei (.exe bei Windows) heißen soll und welche Dateien kompiliert werden sollen. Im o.g. Beispiel bauen wir eine Datei namens ``sdltest`` aus der ``main.c``. Weiter unten im Tutorial kommt eine sdlinterf.c hinzu. Hier müssen also alle C-Files rein, aus denen das Projekt besteht. Sind C-Files in Unterordnern wie *``lib``*, müssen diese auch mit angegeben werden *``lib/sdlinterf.c``*

Zum testen kann man folgenden C-Code ausführen; nicht schön, aber erfüllt seinen Zweck:

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

Wenn der Returncode 0 ist, funktioniert SDL2 :-)

### Windows

Windows erfordert etwas mehr Handarbeit. Der Quick-And-Dirty Weg ist, die Pfade direkt einzutragen, dann ist das Projekt jedoch nicht mehr portabel.
Je nach Architektur von MinGW (siehe oben) kann man FindPkgConfig nachrüsten. Das ist der **empfohlene Weg**, da er auf allen kompatiblen Systemen gleich funktioniert und das Projekt portabel bleibt. (besonders gut für Projekte, die man später vielleicht noch brauchen könnte)

| MinGW-Architektur | verfügbare Methoden                     |
|-------------------|-----------------------------------------|
|32 Bit             | - direktes Einbinden<br>- FindPkgConfig |
|64 Bit             | direktes Einbinden                      |

Von den nachfolgenden Methoden ist also **eine** umzusetzen.

#### Direkt einbinden

**Hinweis**: alle ``set``-Zeilen **müssen mit dem von der Installation verwendeten Pfad angepasst werden**.
Sollte SDL2 in die MinGW-Ordnerstruktur integriert worden sein, muss der MinGW-Installationspfad verwendet werden.

Folgenden Block **angepasst** in die CMakeLists.txt eintragen:

```cmake
set(SDL2_DIR C:/dev/SDL2)
set(SDL2_LIB_DIR ${SDL2_DIR}/lib)

include_directories(${SDL2_DIR}/include)
target_link_libraries(${PROJECT_NAME} ${SDL2_LIB_DIR}/libSDL2.dll.a ${SDL2_LIB_DIR}/libSDL2main.a)
````

Was passiert hier? Die ``set``-Zeilen erstellen CMake-Variablen, die den Pfad von SDL2 speichern. Mit INCLUDE_DIRECTORIES und TARGET_LINK_LIBRARIES weisen wir CMake an, dem Linker SDL2 mitzugeben.

Die CMakeLists.txt sollte damit *etwa* so aussehen:

```cmake
cmake_minimum_required(VERSION 3.15)
project(sdltest C)

set(CMAKE_C_STANDARD 11)

add_executable(sdltest main.c)

set(SDL2_DIR C:/dev/SDL2)
set(SDL2_LIB_DIR ${SDL2_DIR}/lib)

include_directories(${SDL2_DIR}/include)
target_link_libraries(${PROJECT_NAME} ${SDL2_LIB_DIR}/libSDL2.dll.a ${SDL2_LIB_DIR}/libSDL2main.a)
```

Entgegen der offiziellen SDL-Doku sollte hier auf die Flag ```-mwindows``` verzichtet werden, um Funktionalität für das DOS-Fenster zu erhalten (```stdout```, ...)

Weitere Erläuterungen zum CMakeLists.txt:

- add_executable legt fest, wie die ausführbare Datei (.exe bei Windows) heißen soll und welche Dateien kompiliert werden sollen. Im o.g. Beispiel bauen wir eine Datei namens ``sdltest`` aus der ``main.c``. Weiter unten im Tutorial kommt eine sdlinterf.c hinzu. Hier müssen also alle C-Files rein, aus denen das Projekt besteht. Sind C-Files in Unterordnern wie *``lib``*, müssen diese auch mit angegeben werden *``lib/sdlinterf.c``*

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

Code zum testen:

```c
#include <stdio.h>
#include <SDL.h> // evtl. #include <SDL2/SDL.h>
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
- **SDL.h könnte in einem Unterordner liegen, z.B. "SDL2", wenn der Leser das direkte Einbinden verwendet**
- **die Main-Funktion heißt jetzt "WinMain"**

#### voll-statisch linken

Damit die erzeugten .exe files auch portabel sind, *sollte* man voll-statisch bauen, d.h. alle benötigten Libraries werden in die fertige .exe gepackt und müssen auf anderen Rechnen **nicht** im genau gleichen Ordner vorhanden sein.

Um das zu erreichen, muss folgender Block **über ``add_executable``** eingefügt werden:

````
link_libraries("-static -lmingw32 -lSDL2main -lSDL2  -Wl,--no-undefined -lm -ldinput8 -ldxguid -ldxerr8 -luser32 -lgdi32 -lwinmm -limm32 -lole32 -loleaut32 -lshell32 -lversion -luuid -static-libgcc -lhid -lsetupapi")
````

Quelle: Prof. Dr. Klaus Kusche

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
