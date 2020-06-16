# ESP32 + VSCode

This guide describes how to use Visual Studio Code with the Espressif ESP32 build environment.  This assumes that the host OS is Windows 10 x64.

![VSCode + Wemos LOLIN D32](../media/VSCode%20+%20Wemos%20LOLIN%20D32.JPG)

## Setup ESP-IDF 4.0

I was happy to see Espressif embracing [VSCode for ESP-IDF 4.0](https://github.com/espressif/vscode-esp-idf-extension).  For the older ESP-IDF 3.2, refer to the corresponding section further down in this document. 

If you have not already, install [Git](https://git-scm.com/download/win) (globally) and [Python x64](https://www.python.org/downloads/windows/) in `%APPDATA%\..\local`.

Install [Microsoft Visual Studio Code](https://code.visualstudio.com/) (VSCode).
- Add the [Microsoft's C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
- Add the [Espressif IDF extension](https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension) in `User`
    - It should find `git`.  If not install it and restart VSCode.
    - `python.exe` is probably in `C:\Users\your-name\AppData\Local\Programs\Python\Python38\`
    - ESP-IDF 4.0.1, in `C:\Users\your-name\espressif` (goes to subdir `esp-idf`)
    - GNU Tools in `C:\Users\your-name\espressif\bin` (not `.espressif`)
- Optionally, disable Windows Defender's real-time scanning of `C:\Espressif` to speed up compile times.
    
### Start simple

 - From VSCode, open an empty folder and copy a simple example project such as `blink`
   - [F1] » ESP-IDF: Show ESP-IDF Example Projects
     - Get-started » Blink 
 - Connect the ESP32 board with your computer.  In Device Manager, the UART (COM) will show up under the Ports.  In VSCode, select this port
   - [F1] » ESP-IDF: Device configuration
      - Device Target = ESP32 (idf.adapterTargetName)
      - Device Port = COM4 (idf.portWin)
      - Baud Rate = 115200 (idf.baudRate)
 - Start compile/flash/monitor
   - [F1] » ESP-IDF: Build, flash and monitor (`ctrl-e` `d`)(https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/tools/idf-monitor.html).
     - Error `esptool.py failed with exit code 2` hints that the baudrate is to high.  Try a different/shorter cable.
     - Error: `no such option: -b`.  Change `idf.py monitor` line in `.vscode\tasks.json` to use `--baud` instead of `-b`
     - Error `CMake Error: Unable to open check cache file for write.` means it the directory for the cache file doesn't exist.  Instead create it by hand.  Some notes:
     - Warning ` IDF_PATH environment variable is different from inferred IDF_PATH.`.  Likely because `$IDF_PATH` is not set in Windows' system/user environment.  Ignore.
     - Warning `Could NOT find Git (missing: GIT_EXECUTABLE) `.  Odd because `${env:PATH}` does include `C:\Program Files\Git\cmd`
(https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/build-system.html)

### Debugging using GDB over JTAG

![VSCode + ESP-WROVER-KIT](../media/VSCode%20+%20ESP-WROVER-KIT.JPG)

The ESP32 supports the [Open On-Chip Debugger](http://openocd.org/).  Say goodbye to `printf`, and explore the world that was once the exclusive domain of in-circuit emulators.  Using special pins on the ESP32, your computer can set break points, inspect variables and single step instructions.

#### ESP32 board and JTAG adapter

The JTAG interface hooks directly to the CPU.  That allows it do stop the CPU, set breakpoints and have access the CPU has access to.

To use this JTAG interface, we need exclusive access to specific pins on the ESP32.  Two different approaches:
- Some development boards (e.g. ESP-WROVER-KIT) have a USB interface with two channels.  The first is used for JTAG while the other is used for a serial port.  With the ESP-WROVER-KIT, make sure to connect the JTAG jumpers on JP2 to prevent `Error: libusb_open() failed with LIBUSB_ERROR_NOT_SUPPORTED`.
- Other boards bring out the raw JTAG interface using a 10 pin header that should be connected to a JTAG/USB interface, such as the [ESP-PROG](https://github.com/espressif/esp-iot-solution/blob/master/documents/evaluation_boards/ESP-Prog_guide_en.md).  With the interface, make sure to set the `JTAG PWR SEL` jumper for 3.3V.  Also try to keep the connection short (<10 cm)
- If the board has neither, and GPIO12-15 are left you can connect them yourself to a JTAG/USB interface as follows:
    - RESET/RS to TRST_N
    - GPIO15 to TDO
    - GPIO12 to TDI
    - GPIO13 to TCK
    - GPIO14 to TMS
    - GND to GND

#### OpenOCD and driver

Under Windows, install the 64-bit [FTDI D2xx Driver](https://www.ftdichip.com/Drivers/D2XX.htm) setup executable;
- Connect the ESP32 using both JTAG/USB and UART/USB interfaces to the computer.
- Use [Zadig](https://zadig.akeo.ie/) » Options » List All Devices » `Dual RS232-HS (Interface 0)` = `WinUSB v6`; press `Replace Driver`
- Windows Device Manager should reveal:
  - the name of the UART (COM) port.
  - the JTAG will not show up, because WinUSB is a user-space driver

#### Compile and debug

Add configuration information for OpenOCD in `.vscode\settings.json`.
- For WROVER modules use `board/esp32-wrover.cfg`, has 1.8V SPI flash (because of PSRAM limitations)
- For WROOM modules use `board/esp-wroom-32.cfg`, has 3.3V SPI flash.
The flash voltage is important, because GPIO12 is [shared with SPI flash](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/jtag-debugging/tips-and-quirks.html#why-to-set-spi-flash-voltage-in-openocd-configuration)]. Selecting the wrong voltage may cause flash uploads to fail.

```javascript
    "idf.openOcdConfigs": [
      "interface/ftdi/esp32_devkitj_v1.cfg",
      "board/esp32-wrover.cfg"   // update "board/esp32-wrover.cfg" or "board/esp-wroom-32.cfg"
    ],
```    
Debugging requires symbolic data and gets easier when the code is not optimized for runtime or size.  Use `[F1] » ESP-IDF: Launch gui configuration tool` to specify the `Debug (-Og)` compiler optimalization level.

Press `ctrl-e` `d` to Built, upload and monitor over the serial port.  Note that you can also upload the binary over JTAG (`program_esp filename.bin 0x10000 verify`).

From the VSCode debug side bar (ctrl-shift-d), click on the green arrow at the top and select `GDB/JTAG` to connect to the target
 
#### Notes

- There are only two hardware breakpoints
- After hitting a break point, you may still have to select the corresponding task
- To determine the image location in flash, OpenOCD uses the address of the first `app` in the partition table.  When using OTA updates, this will be the `factory` image instead of the OTA downloaded application.  To work around this, specify `esp32 appimage_offset <offset>` (see [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/jtag-debugging/tips-and-quirks.html))
guides/jtag-debugging/index.html)
- You may only want to debug the 1st core (`set ESP32_ONLYCPU 1`)
- More info in [Espressif JTAG Debugging](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-

## Setup ESP-IDF 3.2 (older version)

I started this repository while using ESP-IDF 2.3.  There was no Espressif support for VSCode at the time.  Continue reading to learn how to configure this environment for VSCode.

This guide describes how to use Visual Studio Code with the Espressif ESP32 build environment.  While written for Windows, it can be adapted to other operating systems or installation folders by updating the pathnames.

![VSCode + Wemos LOLIN D32](../media/VSCode%20+%20Wemos%20LOLIN%20D32.JPG)

Install [Microsoft Visual Studio Code](https://code.visualstudio.com/).
- install the [Microsoft's C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

Espressif IDF build environment
- install the GNU toolchain for *xtensa-esp32-elf* in `C:\Espressif\ESP32\msys32"`
   - download [MSYS2 and Toolchain](https://dl.espressif.com/dl/esp32_win32_msys2_environment_and_toolchain-20181001.zip) and unzip to `C:\Espressif\ESP32`
   - refer to [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#setup-toolchain) for details.
- install the ESP32 API in `/opt/esp-idf`
  - from a MINGW32 terminal (`C:\Espressif\ESP32\msys32\mingw32.exe`)
    - `cd /opt ; git clone -b release/v3.3 --recursive https://github.com/espressif/esp-idf.git`
    - `echo export IDF_PATH="C:/Espressif/ESP32/msys32/opt/esp-idf" > /etc/profile.d/export_idf_path.sh ; source /etc/profile.d/export_idf_path.sh`
    - `python -m pip install --user -r $IDF_PATH/requirements.txt`
    - refer to [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/index.html#get-started-get-esp-idf) for detail.
- ((optionally switch to release branch v3.2))
  - from a MINGW32 terminal (`C:\Espressif\ESP32\msys32\mingw32.exe`)
    - `cd /opt/esp-idf`
    - `git fetch`
    - `git checkout release/v3.2`
    - `git pull`
    - `git submodule update --init --recursive`
    - `git branch -a`

Optionally, disable Windows Defender's real-time scanning of `C:\Espressif` to speed up compile times.

## Compile and upload

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
- connect an ESP32 board, and note the port number from Windows Device manager (Properties » Port Settings » Avanced, lets you change the port number)
- Terminal » Run Task » `make menuconfig`, and specify your serial port (e.g. `COM4`)
- Terminal » Run Task » `make app`.  Compile errors will result in messages in the *problems* panel, from where you can jump to the location in the source.
- Terminal » Run Task » `make flash`, or (ctrl-shift-b).  Thils will build the app and flash it to the target.
- Terminal » Run Task » `make monitor`.  A manual is located [here](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/tools/idf-monitor.html).

## Debugging

By default, the panic handler prints relevant registers and the stack dump over the serial port. To use the GDB debugger instead, set `CONFIG_ESP32_PANIC` to `Invoke GDBStub` using `make menuconfig`.  Details [here](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/tools/idf-monitor.html#idf-specific-features).

## JTAG Debugging

![VSCode + ESP-WROVER-KIT](../media/VSCode%20+%20ESP-WROVER-KIT.JPG)

The ESP32 supports the [Open On-Chip Debugger](http://openocd.org/).  Say goodbye to `printf`, and explore the world that was once the exclusive domain of in-circuit emulators.  Using special pins on the ESP32, your computer can set break points, inspect variables and single step instructions.

### Tools

#### ESP32 board and JTAG adapter

Some of the options are
- ESP-WROVER-KIT, one board with both a ESP32 and a combined JTAG/UART adapter (FT2232HL)
  - remember to connect the JTAG jumpers on JP2, otherwise you get "Error: libusb_open() failed with LIBUSB_ERROR_NOT_SUPPORTED".  Details [here](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/get-started-wrover-kit.html#setup-options).
- Wemos LOLIN D32 connected to a ESP-PROG as the JTAG/UART adapter: 
  - RESET/RS to TRST_N
  - GPIO15 to TDO
  - GPIO12 to TDI
  - GPIO13 to TCK
  - GPIO14 to TMS
  - GND to GND

#### OpenOCD and driver

Install OpenOCD in `/opt/openocd-esp32`
- download the latesest win32 release from [github.com/espressif](https://github.com/espressif/openocd-esp32/releases) and extract in the `/opt` directory.
- for details refer to [espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/#jtag-debugging-setup-openocd)

in Windows install the [FTDI D2xx Driver](https://www.ftdichip.com/Drivers/D2XX.htm)
- connect the target using both UART and JTAG to the computer
- use [Zadig](https://zadig.akeo.ie/) » Options » List All Devices » use driver `WinUSB v6` for `Dual RS232-HS (Interface 0)`. ([or libusbk](https://gnu-mcu-eclipse.github.io/arch/riscv/ftdi-jtag-drivers/))
- Device Manager should reveal
  - the JTAG on interface 0 will not show up, because WinUSB is a user-space driver
  - the UART on interface 1 as "Ports (COM & LPT) » COM8"

Compile and debug

- user the compiler flag `-Og` (`make menuconfig`) for minimal optimalizations and symbolic data.
- from the debug side bar (ctrl-shift-d), click on the green arrow at the top and select `GDB/JTAG` to connect to the target

Note

- To determine the image location in flash, OpenOCD uses the address of the first `app` in the partition table.  When using OTA updates, this will be the `factory` image instead of the OTA downloaded application.  To work around this, specify `esp32 appimage_offset <offset>` (see [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/jtag-debugging/tips-and-quirks.html))
