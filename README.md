# Prusa Firmware MK3/S/+ and MK2.5/S

This repository contains the source code and the development versions of the firmware running on the [Original Prusa i3](https://prusa3d.com/) MK3S/MK3/MK2.5S/MK2.5 line of printers.

The latest official builds can be downloaded from [Prusa Drivers](https://www.prusa3d.com/drivers/). Pre-built development releases are also [available here](https://github.com/prusa3d/Prusa-Firmware/releases).

The firmware for the Original Prusa i3 printers is proudly based on [Marlin 1.0.x](https://github.com/MarlinFirmware/Marlin/) by Scott Lahteine (@thinkyhead) et al. and is distributed under the terms of the [GNU GPL 3 license](LICENSE).

This repository contains _development material only!_

# Table of contents

<!--ts-->
   * [Linux build](#linux)
     * [Cmake](#cmake)
     * [PF-build](#pf-build)
   * [Windows, Linux and Mac](#windows-linux-and-mac)
     * [Visual Studio Code](#visual-studio-code)
   * [Arduino IDE (deprecated)](#arduino-ide-deprecated)
   * [Documentation](#documentation)
   * [Advanced](#advanced)
<!--te-->

## Linux
There are three ways to build Prusa-Firmware on Linux: using [CMake](#cmake), [VSCode](#visual-studio-code) (recommended for developers) or with [PF-build](#pf-build) which is more user-friendly for casual users.

### CMake
#### Quick-start
The workflow should be pretty straightforward for anyone with development experience. After installing git and a recent version of python 3 all you have to do is:

    # clone the repository
    git clone https://github.com/prusa3d/Prusa-Firmware Prusa-Firmware/master
    cd Prusa-Firmware/master

    # automatically setup dependencies
    ./utils/bootstrap.py

    # configure and build
    mkdir build
    cd build
    cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=../cmake/AvrGcc.cmake
    ninja


#### Detailed CMake guide
Building with cmake requires:

- cmake >= 3.22.5
- ninja >= 1.10.2 (optional, but recommended)

Python >= 3.6 is also required with the following modules:

- pyelftools (package `python3-pyelftools`)
- polib (package `python3-polib`)
- regex (package `python3-regex`)

Additionally `gettext` is required for translators.

Assuming a recent Debian/Ubuntu distribution, install the dependencies globally with:

    sudo apt-get install cmake ninja python3-pyelftools python3-polib python3-regex gettext

Prusa-Firmware depends on a pinned version of `avr-gcc` and the external `prusa3dboards` package. These can be setup using `./utils/bootstrap.py`:

    # automatically setup dependencies
    ./utils/bootstrap.py

which will download and unpack them inside the `.dependencies` directory. `./utils/bootstrap.py` will also install `cmake`, `ninja` and the required python packages if missing, although installing those through the system's package manager is usually preferred.

You can then proceed by creating a build directory, configure for AVR and build:

    # configure
    mkdir build
    cd build
    cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=../cmake/AvrGcc.cmake

    # build
    ninja

By default all variants are built. There are several ways to restrict the build for development. During configuration you can set:

- `cmake -DFW_VARIANTS=variant`: comma-separated list of variants to build. This is the file name as present in `Firmware/variants` without the final `.h`.
- `cmake -DMAIN_LANGUAGES=languages`: comma-separated list of ISO language codes to include as main translations.
- `cmake -DCOMMUNITY_LANGUAGES=languages`: comma-separated list of ISO language codes to include as community translations.

When building the following targets are available:

- `ninja ALL_MULTILANG`: build all multi-language targets (default)
- `ninja ALL_ENGLISH`: build all single-language targets
- `ninja ALL_FIRMWARE`: build all single and multi-language targets
- `ninja VARIANT_ENGLISH`: build the single-language version of `VARIANT`
- `ninja VARIANT_MULTILANG`: build the multi-language version of `VARIANT`
- `ninja check_lang`: build and check all language translations
- `ninja check_lang_ISO`: build and check all variants with language `ISO`
- `ninja check_lang_VARIANT`: build and check all languages for `VARIANT`
- `ninja check_lang_VARIANT_ISO`: build and check language `ISO` for `VARIANT`


#### Automated tests
Automated tests are built with cmake by configuring for the current host:

    # clone the repository
    git clone https://github.com/prusa3d/Prusa-Firmware Prusa-Firmware/master
    cd Prusa-Firmware/master

    # automatically setup dependencies
    ./utils/bootstrap.py

    # configure and build
    mkdir build
    cd build
    cmake .. -G Ninja
    ninja

    # run the tests
    ctest


### PF-build
PF-build is recommended for users without development experience. Download or clone the repository,
then run PF-build and simply follow the instructions:

    cd Prusa-Firmware/master
    ./PF-build.sh

PF-build currently assumes a Debian/Ubuntu (or derivative) distribution.


## Windows, Linux and Mac
### Visual Studio Code
#### Prerequisites

* [Visual Studio Code](https://code.visualstudio.com/)
* [CMake Tools plugin](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
* [Python](https://www.python.org/)
* `git` for Linux and MAC or
* [Git Bash for Windows](https://git-scm.com/downloads)

#### First time setup

Start by cloning the Prusa-Firmware repository

    git clone https://github.com/prusa3d/Prusa-Firmware Prusa-Firmware/master

Open the `Prusa-Firmware/master` folder in VScode.

Open a new terminal in VScode (Terminal→New Terminal) and run

    python .\utils\bootstrap.py

This will download all dependencies required to build the firmware. You should see a `.dependencies` folder in the Prusa-Firmware folder.

Reload VScode. If all works correctly you should see the VScode automatically configuring the CMake project for you. If this doesn't happen you likely need to set the CMake kit; This can be done in two ways:

1. Type `Ctrl+Shift+P` and search for `CMake: Select a Kit`. Select `avr-gcc`. If none appear, Scan for kits first.
2. If 1) does not work for some reason, as a last resort you can edit the CMake Tools settings. Search for "Additional Kits" and add `.vscode/cmake-kits.json` to the list.

After updating the kit, you may need to reload VScode.

#### Building

To start building a firmware, click the CMake Tools plugin icon on the far left side. You will get a very large list of targets to build. Find the firmware you'd like to build (like `MK3S-EINSy10a_ENGLISH`) and select the small icon which shows "Build" when hovered over.

The built .hex file can then be found in folder `Prusa-Firmware/master/build`


## Arduino IDE (deprecated)

Using Arduino IDE is still possible, but _no longer supported_. Prusa-Firmware requires a complex multi-step build process that cannot be done automatically with just the IDE. For a long time we provided instructions to use Arduino in combination with shell scripts, however starting with 3.13 the build system has been completely switched to `cmake`.

Building with Arduino IDE results in a *limited* firmware:

- Arduino IDE can only build a single, english-only variant at a time that you manually have to select
- The build will not be reproducible (meaning you will likely get a different binary every time you build the same sources)
- You need to download, patch and select the correct board definitions by hand

For these reasons, you should think twice before reporting issues for a firmware built with Arduino. If you find a bug in the firmware, building and testing using CMake should be your first thought. Issues regarding Arduino builds are answered by the community and are not officially supported.


### Environment preparation

Install "Arduino Software IDE" from the official website https://www.arduino.cc -> Software -> Downloads. Version 1.8.19 or higher is required.

Setup Arduino to install and use the Prusa board definitions:

- Open Arduino and navigate to File -> Preferences -> Settings
- To the text field "Additional Boards Manager URLs" add `https://raw.githubusercontent.com/prusa3d/Arduino_Boards/master/IDE_Board_Manager/package_prusa3d_index.json`
- Open Board manager (Tools -> Board -> Board manager)
- Install "Prusa Research AVR Boards by Prusa Research"


### Source code preparation

Clone or download this repository to your local drive.

In the subdirectory `Firmware/variants/` select the configuration file (.h) corresponding to your printer model and manually copy it to `Firmware/Configuration_prusa.h`

Run "Arduino IDE", then

- Open the file `Firmware/Firmware.ino`
- Select the target board with Tools -> Board -> "PrusaResearch Einsy RAMBo"
- Open `Firmware/config.h` and change `LANG_MODE` to 0.


### Compilation and upload

- Run the compilation: Sketch -> Verify/Compile
- Upload the result code into the connected printer: Sketch -> Upload

# Documentation
Run [doxygen](http://www.doxygen.nl/) in `Firmware` folder.
Visit for doxygen generated output
- [EEPROM Table](https://prusa3d.github.io/Prusa-Firmware-Doc/group__eeprom__table.html)
- [G-Code List](https://prusa3d.github.io/Prusa-Firmware-Doc/group__GCodes.html)
- [XFLASH Layout](https://prusa3d.github.io/Prusa-Firmware-Doc/group__xflash__layout.html)

# Advanced
This section is for advanced users.

<!--ts-->
   * [How-to prepare a Pull Request](#how-to-prepare-a-pull-request)
   * [MK404 simulator](#mk404-simulator)
   * [Tools](#tools)
   * [Translations](#translations)
 <!--te-->

## How-to prepare a Pull Request
Before submiting a Pull Request we would like you to check your PR.

PRs containing the steps below are easier to review and have a better chance to be merged.

PRs saving resources are more than welcome!

PRs consuming additional resources should have "strong" arguments to convince the developers as resources are a rare thing on an Atmel ATMEGA 2560.  

- [ ] Short description of PR
- [ ] Detailed description of PR
  - [ ] Bug-fix, enhancement of existing feature, new feature
    - [ ] Links to issues
- [ ] Test scenario
  - [ ] Describe old behavior
  - [ ] Describe new behavior
    - [ ] Expected results
- [ ] Tested
  - [ ] on MK404 simulator
    - [ ] MK3/S
      - [ ] with MMU
    - [ ] MK2.5/S
      - [ ] with MMU
  - [ ] on real printer (type)
    - [ ] with/without MMU
- [ ] Resource usage, how many additional flash and RAM are used or saved

## MK404 Simulator
Please visit [MK404 Sim](https://github.com/vintagepc/MK404) for more information.

### Linux
#### How-to prepare MK404 simulator
Please install needed packages following https://github.com/vintagepc/MK404/wiki/Supported-Operating-Systems#linux

You gonna need `sudo apt install libelf-dev gcc gcc-avr libglew-dev freeglut3-dev libsdl-sound1.2-dev libpng-dev cmake git build-essential lcov mtools` 

- Git clone the MK404 repository `git clone https://github.com/vintagepc/MK404 MK404/master`
- Change the folder `cd MK404/master`
- Get submodules `git submodule init` and `git submodule update`
- Create build folder `mkdir -p build`
- build MK404
  - `cmake -Bbuild -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=TRUE -DCMAKE_BUILD_TYPE=Release -G "Unix Makefiles"`
  - `cd build`
  - `make`
- build SD cards from `MK404/master/SDcard` folder
  - Change back to `MK404/master` folder `cd ..`
  - for MK3S `cmake --build build --config Release --target Prusa_MK3S_SDcard.bin`
  - for MK3 `cmake --build build --config Release --target Prusa_MK3_SDcard.bin`
  - for MK2.5 `cmake --build build --config Release --target Prusa_MK25_13_SDcard.bin`
  - for MK2.5S `cmake --build build --config Release --target Prusa_MK25S_13_SDcard.bin`
  - for MK3S/+ with MMU2 `cmake --build build --config Release --target Prusa_MK3SMMU2_SDcard.bin`
  - for MK3 with MMU2 `cmake --build build --config Release --target Prusa_MK3MMU2_SDcard.bin`

#### Run MK404
Please read the [MK404 Wiki](https://github.com/vintagepc/MK404/wiki)

You can use the [online configuration tool](https://vintagepc.github.io/MK404/MK404Exec.html) to prepare the command.

Please select/fill in at least following settings:
- `Printer model` (default is MK3S without MMU2)
- `Firmware file` including the path to your build hex file

Recommended settings:
- `Route printer UART to a PTY instead of stdio` to connect with a terminal like PrusaLink or OctoPrint to the MK404 sim. Use `/tmp/sim-avr-uart0` to connect.
- `Enable (experimental) scripting terminal` very useful if you want test things, see [details](https://github.com/vintagepc/MK404/wiki/Scripting)

#### Tips and Tricks

##### Keys to trigger events in the SIM
- `f` toggle Filament Sensor
- `S` make a screen shot of current LCD. Saved with timestamp in the build folder

##### Custom MMU2 firmware
To run a custom or development MMU2 firmware add `-F <path/mmu2-firmware-filname.hex>` to your `./MK404 ....` command

##### Examples
Run Prusa MK3S with latest self build firmware in M404
`./MK404 Prusa_MK3S -f ~/Prusa-Firmware/master/build/build_gen/MK3S-EINSy10a/MK3S-EINSy10a_MULTILANG.hex -s --terminal`

or

`./MK404 Prusa_MK3S -f ~/Prusa-Firmware/master/build/FW3.13.0+6914-MK3S-EINSy10a_MULTILANG.hex -s --terminal`

where `3.13.0+6914` and paths will differ depending when/where you build the firmware.

Run Prusa MK3S with custom MMU2 firmware
`./MK404 Prusa_MK3SMMU2 -f ~/Prusa-Firmware/master/build/build_gen/MK3S-EINSy10a/MK3S-EINSy10a_MULTILANG.hex -s --terminal -F ~/Prusa-Firmware-MMU/main/build/release/MMU_2.0.0+764.hex`

where `2.0.0+764` and paths will differ depending when/where you build MMU firmware.

## Tools
We have several tools available in the `/tools` folder

### Dump tools
Please read the [Dump tools readme](https://github.com/prusa3d/Prusa-Firmware/blob/MK3/tools/README.md)

### Thermal model analysis
Please read the [Thermal model analysis readme](https://github.com/prusa3d/Prusa-Firmware/tree/MK3/tools#thermal-model-analysis)

## Translations
Please read the [Translations readme](https://github.com/prusa3d/Prusa-Firmware/blob/MK3/lang/README.md) for more details

### Translation pull requests
Ensure that the Traslation releated pull request compiles without any issues and have been tested.

The translation pull request should contain next to the [regular checks](#How-to prepare a Pull Request) also
- [ ] Verified LCD output
- [ ] All multiple languages translations tested and reviewed

#### Translation tips and tricks
- Please use diacritics in the `.po` files.

  - At this moment we support ONLY Germanic diacritcs `äÄöÖüÜß` being show on display. Other language diacritics will be automatically replaced with `aA-zZ` characters.

- Review your changes
  - Sometimes it makes sense to split/shorten long words to fit messages on one screen instead of having one word on second page.
  
Original translation is split on LCD two screen     
```
[I]: MSG_BED_SKEW_OFFSET_DETECTION_FAILED_FRONT_BOTH_FAR c=20 r=6
 source text:
      ₀₁₂₃₄₅₆₇₈₉₀₁₂₃₄₅₆₇₈₉
   1 |XYZ calibration     |
   2 |failed. Front       |
   3 |calibration points  |
   4 |not reachable.      |
 translated text:
      ₀₁₂₃₄₅₆₇₈₉₀₁₂₃₄₅₆₇₈₉
   1 |XYZ-Kalibrierung    |
   2 |fehlgeschlagen.     |
   3 |Vordere             |
   4 |Kalibrirungsunkte   |
   5 |nicht erreichbar.   |
```

Modified translation fits on one LCD screen

```
[I]: MSG_BED_SKEW_OFFSET_DETECTION_FAILED_FRONT_BOTH_FAR c=20 r=6
 source text:
      ₀₁₂₃₄₅₆₇₈₉₀₁₂₃₄₅₆₇₈₉
   1 |XYZ calibration     |
   2 |failed. Front       |
   3 |calibration points  |
   4 |not reachable.      |
 translated text:
      ₀₁₂₃₄₅₆₇₈₉₀₁₂₃₄₅₆₇₈₉
   1 |XYZ-Kalibrierung    |
   2 |fehlgeschlagen.     |
   3 |Vordere Kal.-Punkte |
   4 |nicht erreichbar.   |
```

##### How-to verify the LCD output
- Build the multi-language firmware
  - `*_lang.map` files can be found in the build folder `build/build_gen/<Printer type>/lang/<Printer tpye>_lang.map`
- Change to `/lang` folder
- Execute `./lang-check.py --map <path and filename of _lang.map> po/Firmware_<language>.po`
  - Additonal arguments
    - `--no-suggest` removes warnings like same as original
    - `--information` outputs ALL messages as shown on LCD screen
- All `[W]`arnings and `[E]`rrors need to be solved

Example:
- Run German check for suggestions, warnings and errors `./lang-check.py --map ../build/build_gen/MK3S-EINSy10a/lang/MK3S-EINSy10a_lang.map po/Firmware_de.po`


- Output German translation in a text file for review. `./lang-check.py --map ../build/build_gen/MK3S-EINSy10a/lang/MK3S-EINSy10a_lang.map po/Firmware_de.po --no-suggest --information >~/20230208_German_translation.txt`
