I use Windows, but this guide applies to other operating systems equally well.  I use absolute path names for the tools, not to be dependent on the search `PATH` of your operating system.

# Tools

- [Microsoft Visual Studio Code](https://code.visualstudio.com/) installed
  - with [Microsoft's C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
  - with [WebFreak's Native Debug extension](https://marketplace.visualstudio.com/items?itemName=webfreak.debug)
- The Espressif IDF build environment installed
  - GNU toolchain for *xtensa-esp32-elf* in `C:\Espressif\ESP32\msys32"`
     - download [MSYS2 and Toolchain](https://dl.espressif.com/dl/esp32_win32_msys2_environment_and_toolchain-20181001.zip) and unzip to `C:\Espressif\ESP32`
     - refer to [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#setup-toolchain) for details.
  - ESP32 API in `/opt/esp-idf`
    - from a MINGW32 terminal (*C:\Espressif\ESP32\msys32\mingw32.exe*)
      - `cd /opt ; git clone -b v3.2 --recursive https://github.com/espressif/esp-idf.git`
      - `echo export IDF_PATH="C:/Espressif/ESP32/msys32/opt/esp-idf" > /etc/profile.d/export_idf_path.sh ; `source /etc/profile.d/export_idf_path.sh`
      - `python -m pip install --user -r $IDF_PATH/requirements.txt`
      - refer to [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#get-started-get-esp-idf) for detail.
  - optionally, disable Windows Defender's real-time scanning of `C:\Espressif` to speed up compile times.

# Usage

- Copy a simple example
  - from a MINGW32 terminal (*C:\Espressif\ESP32\msys32\mingw32.exe*)
    - `cd ~ ; cp -r $IDF_PATH/examples/get-started/blink . ; cd blink`
- Copy the .vscode directory from [GitHub](https://github.com/cvonk/vscode-starters/blob/master/ESP32/) to your project workspace (`~/blink/`)
  - The configuration files in `~/blink/.vscode`
  - `c_cpp_properties.json`, specifies the compiler executable and where it looks for '#include' files. Details at [mingw.org](http://mingw.org/wiki/IncludePathHOWTO).
  - `tasks.json`, describes the build tasks such as `make flash`, `make clean`.  It uses the `-j4` option to enables parallel compilation on 4 cores.
  - `settings.json`, changes the default terminal shell to `bash`
  - `launch.json`, describes how to start the application (`monitor`, `openocd`)
- Open vscode for folder `C:\Espressif\ESP32\msys32\YourName\blink`, where `YourName` is your user name. From vscode:
  - Terminal » New Terminal, and give it the permission sought. Make sure it is running the MINGW32 bundeled with ESP32 IDF.
  - Terminal » Run Task » `make menuconfig`, and specify your serial port (e.g. `COM5`)
  - Terminal » Run Task » `make main`.  Compile errors will result in messages in the *problems* panel, from where you can jump to the location in the source.
  - connect an ESP32 board
  - Terminal » Run Task » `make flash`, or (ctrl-shift-b)
  - from the debug side bar (ctrl-shift-d), click on the green arrow at the top and select `monitor` to connect to the UART

# Debugging

The ESP32 supports the [Open On-Chip Debugger](http://openocd.org/).  Say goodbye to `printf`, and explore the world that was once the exclusive domain of in-circuit emulators.  Using special pins on the ESP32, your computer can set break points, inspect variables and single step instructions.

## Tools

- ESP32 board with JTAG adapter, some of the options are
  - ESP-WROVER-KIT, one board with both a ESP32 and a combined JTAG/UART adapter (FT2232HL)
  - Wemos LOLIN D32 connected to a ESP-PROG as the JTAG/UART adapter: 
    - RESET/RS to TRST_N
    - GPIO15 to TDO
    - GPIO12 to TDI
    - GPIO13 to TCK
    - GPIO14 to TMS
    - GND to GND
- OpenOCD installed in `/opt/openocd-esp32`, installatio instructions at [espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/#jtag-debugging-setup-openocd)
  - in MINGW32 use the package manager to install *libusb* as in `pacman -S mingw-w64-i686-libusb`
  - When using Windows, make sure it uses the `libusb` for the JTAG adapter
    - verify that Window's Devices shows the JTAG as RS232-HS under USB Devices
    - to correct, use zadig-2.4 to change the driver for FT232HL to *libusb*.  Details are available at [Virgilia's writeup](https://github.com/VirgiliaBeatrice/esp32-devenv-vscode/blob/master/tutorial.md)

    // CJV see https://higaski.at/vscode-esp32-debugging/
    // CJV compile with at least -Og (make menuconfig)
    // Espressif doc at https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/using-debugger.html#jtag-debugging-using-debugger-command-line


cd ~/esp/openocd-esp32 ; cp -r ./share/ /mingw32/share/openocd ; cp ./bin/openocd.exe /mingw32/bin/
- 
