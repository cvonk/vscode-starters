I use Windows, but this guide applies to other operating systems equally well.

# Getting Started

- [Microsoft Visual Studio Code](https://code.visualstudio.com/) installed
  - with [Microsoft's C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
  - with [WebFreak's Native Debug extension](https://marketplace.visualstudio.com/items?itemName=webfreak.debug)
- The Espressif IDF build environment installed
  - toolchain in `C:\Espressif\ESP32\msys32"`, install details can be found at [espressif](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#setup-toolchain)
  - API in in `/opt/`, details at [espressif](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#get-started-get-esp-idf)
  - optionally, disable Windows Defender's real-time scanning of `C:\Espressif` to speed up compile times.

For the serial port specify COM5 (or whatever) instead of e.g. /dev/ttyS4

Visual Code
- Copy the .vscode directory from "\\nas.vonk\Coert Vonk\Masters\Electronics and Math\ESP32" to your project workspace
- Install the extension C/C++ by Microsoft
- Wait until it is done loading (otherwise you get a MINGW64 or whatever your default terminal is). When opening a Terminal, give it the permission sought

JTAG
- https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/#jtag-debugging-setup-openocd
- cd ~/esp/openocd-esp32 ; cp -r ./share/ /mingw32/share/openocd ; cp ./bin/openocd.exe /mingw32/bin/
- pacman -S mingw-w64-i686-libusb
- use zadig-2.4 to change the driver for FT232HL to libusb, so that Win Devices shows the JTAG as RS232-HS under USB Devices
- (see https://github.com/VirgiliaBeatrice/esp32-devenv-vscode/blob/master/tutorial.md)
