This guide describes how to use Visual Studio Code with the Espressif ESP32 build environment.  While written assuming it is running on Windows, it can be adapted to other operating systems or installation folders by updating the pathnames.

# Setup

Install [Microsoft Visual Studio Code](https://code.visualstudio.com/).
- install the [Microsoft's C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
- install the [WebFreak's Native Debug extension](https://marketplace.visualstudio.com/items?itemName=webfreak.debug)

Espressif IDF build environment
- install the GNU toolchain for *xtensa-esp32-elf* in `C:\Espressif\ESP32\msys32"`
   - download [MSYS2 and Toolchain](https://dl.espressif.com/dl/esp32_win32_msys2_environment_and_toolchain-20181001.zip) and unzip to `C:\Espressif\ESP32`
   - refer to [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#setup-toolchain) for details.
- install the ESP32 API in `/opt/esp-idf`
  - from a MINGW32 terminal (`C:\Espressif\ESP32\msys32\mingw32.exe`)
    - `cd /opt ; git clone -b v3.2 --recursive https://github.com/espressif/esp-idf.git`
    - `echo export IDF_PATH="C:/Espressif/ESP32/msys32/opt/esp-idf" > /etc/profile.d/export_idf_path.sh ; source /etc/profile.d/export_idf_path.sh`
    - `python -m pip install --user -r $IDF_PATH/requirements.txt`
    - refer to [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#get-started-get-esp-idf) for detail.

Optionally, disable Windows Defender's real-time scanning of `C:\Espressif` to speed up compile times.

# Compile and upload

Start with a simple example
- from a MINGW32 terminal (*C:\Espressif\ESP32\msys32\mingw32.exe*)
  - `cd ~ ; cp -r $IDF_PATH/examples/get-started/blink . ; cd blink`

Copy the `.json` files from [GitHub](https://github.com/cvonk/vscode-starters/blob/master/ESP32/) to a `.vscode` directory in your project workspace (`~/blink/.vscode`)
- The configuration files in `~/blink/.vscode` are
  - `c_cpp_properties.json`, specifies the compiler executable and where it looks for '#include' files. Details at [mingw.org](http://mingw.org/wiki/IncludePathHOWTO).
  - `tasks.json`, describes the build tasks such as `make flash`, `make clean`.  It uses the `-j4` option to enables parallel compilation on 4 cores.
  - `settings.json`, changes the default terminal shell to `bash`
  - `launch.json`, describes how to start the application (`monitor`, `openocd`)

Open vscode for folder `C:\Espressif\ESP32\msys32\YourName\blink`, where `YourName` is your user name. From vscode:
- Terminal » New Terminal, and give it the permission sought. You may have to open it once more. Make sure it is running the MINGW32 bundeled with ESP32 IDF.
- connect an ESP32 board, and note the port number from Windows Device manager
- Terminal » Run Task » `make menuconfig`, and specify your serial port (e.g. `COM4`)
- Terminal » Run Task » `make app`.  Compile errors will result in messages in the *problems* panel, from where you can jump to the location in the source.
- Terminal » Run Task » `make flash`, or (ctrl-shift-b).  Thils will build the app and flash it to the target.
- Terminal » Run Task » `make monitor`.  A manual is located [here](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/tools/idf-monitor.html).

# Debugging

By default, the panic handler prints relevant registers and the stack dump over the serial port. To use the GDB debugger instead, set `CONFIG_ESP32_PANIC` to `Invoke GDBStub` using `make menuconfig`.  Details [here](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/tools/idf-monitor.html#idf-specific-features).

# JTAG Debugging

The ESP32 supports the [Open On-Chip Debugger](http://openocd.org/).  Say goodbye to `printf`, and explore the world that was once the exclusive domain of in-circuit emulators.  Using special pins on the ESP32, your computer can set break points, inspect variables and single step instructions.

## Tools

ESP32 board with JTAG adapter, some of the options are
- ESP-WROVER-KIT, one board with both a ESP32 and a combined JTAG/UART adapter (FT2232HL)
- Wemos LOLIN D32 connected to a ESP-PROG as the JTAG/UART adapter: 
  - RESET/RS to TRST_N
  - GPIO15 to TDO
  - GPIO12 to TDI
  - GPIO13 to TCK
  - GPIO14 to TMS
  - GND to GND

OpenOCD and driver
- install OpenOCD `/opt/openocd-esp32`
  - download the latesest win32 release from [github.com/espressif](https://github.com/espressif/openocd-esp32/releases) and extract in the `/opt` directory.
  - for details refer to [espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/#jtag-debugging-setup-openocd)
- in MINGW32 use the package manager to install *libusb* as in `pacman -S mingw-w64-i686-libusb`
- When using Windows, make sure it uses the `libusb` for the JTAG adapter
  - verify that Window's Devices shows the JTAG as RS232-HS under USB Devices
  - to correct, use zadig-2.4 to change the driver for FT232HL to *libusb*.  Details are available at [Virgilia's writeup](https://github.com/VirgiliaBeatrice/esp32-devenv-vscode/blob/master/tutorial.md)

Compile and debug

- compile with at most an optimalization of -Og (`make menuconfig`).  Personally, I turn optimalizations off (`-O0`).
- from the debug side bar (ctrl-shift-d), click on the green arrow at the top and select `GDB/JTAG` to connect to the target


Notes:
see https://higaski.at/vscode-esp32-debugging/
compile with at least -Og (make menuconfig)
Espressif doc at https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/using-debugger.html#jtag-debugging-using-debugger-command-line
cd ~/esp/openocd-esp32 ; cp -r ./share/ /mingw32/share/openocd ; cp ./bin/openocd.exe /mingw32/bin/
- 
