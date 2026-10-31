# ATtiny Programmer and Toolchain Setup Guide

This guide provides step-by-step instructions for setting up programmer drivers, the build environment, and toolchains to compile and flash firmware to the ATtiny85.

It covers setup for both **USBtiny** (including **Little Wire**) and **USBasp** hardware programmers.

---

## 1. USB Programmer Drivers

Unlike an Arduino Uno or FTDI serial adapter, USBasp and USBtiny do not create a virtual `COM` port in Windows or your IDE. Instead, `avrdude` communicates with them directly as generic USB devices via `libusb`. You do not need to figure out what COM port happened to be created and flashing commands do not require a `-P COMx` port parameter.

### Windows
Windows does not ship with built-in drivers for USBasp or USBtiny/Little Wire. You must assign a `libusb` driver using **Zadig**:

1. Install Zadig via PowerShell:
   ```powershell
   winget install akeo.ie.Zadig
   ```
   *(Or download it directly from [zadig.akeo.ie](https://zadig.akeo.ie/)).*
2. Plug in your programmer.
3. Open **Zadig**:
   * **For Little Wire / USBtiny**: Select **USBtiny**, **USBtinyISP**, or **Little Wire**, and set the target driver (green arrow) to **`libusb-win32`**.
   * **For USBasp**: Select **USBasp**, and set the target driver to **`libusbK`**.
4. Click **Install Driver** (or **Replace Driver**).

### macOS
* **No drivers required**: macOS natively recognizes both USBasp and USBtiny through its built-in IOKit USB stack. AVRDUDE communicates with them directly via `libusb` without any third-party driver installation.

### Linux
* **No kernel drivers required**: Linux handles both devices out of the box via `libusb`.
* **udev rules (Permissions)**: By default, non-root users may not have permission to access raw USB devices. If `avrdude` fails with a permission error, a udev rule may help, e.g. for Little Wire:
  ```bash
  sudo tee /etc/udev/rules.d/99-avr-programmers.rules << 'EOF'
  # USBasp
  SUBSYSTEM=="usb", ATTR{idVendor}=="16c0", ATTR{idProduct}=="05dc", MODE="0666"
  # USBtiny / Little Wire
  SUBSYSTEM=="usb", ATTR{idVendor}=="1781", ATTR{idProduct}=="0c9f", MODE="0666"
  EOF

  sudo udevadm control --reload-rules && sudo udevadm trigger
  ```

---

## 2. Install the Build Environment (MSYS2 on Windows)

Windows does not ship with a native C compiler or AVR toolchain. The recommended distribution is provided via MSYS2.

### Step 2.1: Install MSYS2
Open Windows Terminal or PowerShell and run:
```powershell
winget install MSYS2.MSYS2
```
*(Alternatively, download the standalone installer from [msys2.org](https://www.msys2.org) and install to the default `C:\msys64` location).*

### Step 2.2: Update MSYS2 & Install AVR Toolchain
1. Open **`MSYS2 UCRT64`** from the Windows Start Menu.
2. Update the system package database:
   ```bash
   pacman -Syu
   ```
   *(If prompted to close the terminal window, close it, reopen **`MSYS2 UCRT64`**, and rerun `pacman -Syu` until the system is completely up to date).*
3. Install the AVR GCC compiler, C runtime library, AVRDUDE, and Make:
   ```bash
   pacman -S --needed make mingw-w64-ucrt-x86_64-avr-gcc mingw-w64-ucrt-x86_64-avr-libc mingw-w64-ucrt-x86_64-avrdude
   ```
4. Verify the installation:
   ```bash
   avr-gcc --version
   avrdude -v
   make --version
   ```

*(On Linux, install via your package manager: `sudo apt install gcc-avr avr-libc avrdude make`. On macOS: `brew install avrdude avr-gcc`).*

---

## 3. Add Tools to Windows System PATH (Optional)

To compile and flash directly from standard Windows Command Prompt (`cmd.exe`) or PowerShell without opening the MSYS2 shell:

1. Press <kbd>Win</kbd> + <kbd>R</kbd>, type `sysdm.cpl`, and press **Enter**.
2. Go to the **Advanced** tab $\rightarrow$ click **Environment Variables**.
3. Under **User variables** (or **System variables**), select **`Path`** and click **Edit**.
4. Click **New** and add:
   ```text
   C:\msys64\ucrt64\bin
   ```
5. Click **OK** to apply.
6. Restart any open terminals or start a new command prompt.

---

## 4. Test Programmer Communication

Wire the ISP header of your programmer to the target ATtiny85 socket/pins (VCC, GND, RESET, MOSI, MISO, SCK) and test the connection:

### For Little Wire / USBtiny:
```bash
avrdude -c usbtiny -p t85
```

### For USBasp:
```bash
avrdude -c usbasp -p t85
```

**Expected output:**
```text
Avrdude done.  Thank you.
```

USBasp programmers may also output the following message that can be ignored.
```text
Error: cannot set sck period; please check for usbasp firmware update
```
