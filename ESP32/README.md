I use Windows, but this guide applies to other operating systems equally well.  I use absolute path names for the tools, not to be dependent on the search `PATH` of your operating system.

# Toools

- [Microsoft Visual Studio Code](https://code.visualstudio.com/) installed
  - with [Microsoft's C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
  - with [WebFreak's Native Debug extension](https://marketplace.visualstudio.com/items?itemName=webfreak.debug)
- The Espressif IDF build environment installed
  - toolchain in `C:\Espressif\ESP32\msys32"`. Installation instruction can be found at [espressif](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#setup-toolchain)
  - API in `/opt/`.  Instruction are available at [espressif](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#get-started-get-esp-idf)
  - optionally, disable Windows Defender's real-time scanning of `C:\Espressif` to speed up compile times.

# Usage

- Copy an example `cd ~ ; cp -r $IDF_PATH/examples/get-started/blink . ; cd blink`
- Copy the .vscode directory from [GitHub](https://github.com/cvonk/vscode-starters/blob/master/ESP32/) to your project workspace
  - `.vscode/c_cpp_properties.json`, specifies the compiler executable and where it looks for '#include' files. Details at [mingw.org](http://mingw.org/wiki/IncludePathHOWTO).
  - `.vscode/tasks.json`, describes the build tasks such as `make flash`, `make clean`.  It uses the `-j4` option to enables parallel compilation on 4 cores.
  - `.vscode/settings.json`, changes the default terminal shell to `bash`
  - `.vscode/launch.json`, describes how to start the application (`monitor`, `openocd`)
- Open vscode at `C:\Espressif\ESP32\msys32\YourName\blink`, where `YourName` is your user name
  - open a terminal, give it the permission sought, and make sure it is running the MINGW32 bundeled with ESP32 IDF.
  - run the task `make menuconfig` and specify your serial port (e.g. `COM5`)
  - run the task `make main`.  Compile errors would result in messages in the *problems* tab, from where you can click them to jump to the location in the source.
  - connect an ESP32 board
  - flash the code using `make flash` (ctrl-shift-b)
  - from the debug side bar (ctr-shift-d), click on the green arrow at the top and select `monitor` to connect to the UART

# Debugging

The ESP32 supports the [Open On-Chip Debugger](http://openocd.org/).  Say goodbye to `printf`, and explore the world that was once reserved for in-circuit emulators.  Using special pins on the ESP32, your computer can set break points, inspect variables and single step instructions.

## Tools

- JTAG adapter, such as..
- ESP32 board with the .. pins exposed, such as ..
- OpenOCD installed in `/opt/openocd`, installatio instructions at [espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/#jtag-debugging-setup-openocd)
- in MINGW32 use the package manager to install *libusb* as in `pacman -S mingw-w64-i686-libusb`
- When using Windows, make sure it uses the `libusb` for the JTAG adapter
  - use zadig-2.4 to change the driver for FT232HL to *libusb*, so that Window's Devices shows the JTAG as RS232-HS under USB Devices.  For details refer to [Virgilia's writeup](https://github.com/VirgiliaBeatrice/esp32-devenv-vscode/blob/master/tutorial.md)

- cd ~/esp/openocd-esp32 ; cp -r ./share/ /mingw32/share/openocd ; cp ./bin/openocd.exe /mingw32/bin/
- 
