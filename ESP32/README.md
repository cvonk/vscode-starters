I use Windows, but this guide applies to other operating systems equally well.

# Toools

- [Microsoft Visual Studio Code](https://code.visualstudio.com/) installed
  - with [Microsoft's C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
  - with [WebFreak's Native Debug extension](https://marketplace.visualstudio.com/items?itemName=webfreak.debug)
- The Espressif IDF build environment installed
  - toolchain in `C:\Espressif\ESP32\msys32"`, installation details at [espressif](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#setup-toolchain)
  - API in in `/opt/`, details at [espressif](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#get-started-get-esp-idf)
  - optionally, disable Windows Defender's real-time scanning of `C:\Espressif` to speed up compile times.

# Usage

- Copy an example `cd ~ ; cp -r $IDF_PATH/examples/get-started/blink . ; cd blink
- copy the .vscode directory from GitHub to your project workspace
  - `.vscode/c_cpp_properties.json`, the compiler configuration
  - `.vscode/tasks.json`, describes the build tasks such as `make main`.  Note that the `-j4` enables parallel compiles on 4 cores.
  - `.vscode/settings.json`, changes the default terminal shell to `bash`
  - `.vscode/launch.json`, specifies how to launch the application
- open vscode at `C:\Espressif\ESP32\msys32\YourName\blink`, where `YourName` is your user name
  - open a terminal, give it the permission sought, and make sure it is MSYS32.
  - run the task `make menuconfig` and specify your serial port (e.g. `COM5`)
  - run the task `make main`.  Compile errors would result in messages in the `problems` space, from where you can click them to jump to the location in the source.

# Debugging

JTAG
- https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/#jtag-debugging-setup-openocd
- cd ~/esp/openocd-esp32 ; cp -r ./share/ /mingw32/share/openocd ; cp ./bin/openocd.exe /mingw32/bin/
- pacman -S mingw-w64-i686-libusb
- use zadig-2.4 to change the driver for FT232HL to libusb, so that Win Devices shows the JTAG as RS232-HS under USB Devices
- (see https://github.com/VirgiliaBeatrice/esp32-devenv-vscode/blob/master/tutorial.md)
