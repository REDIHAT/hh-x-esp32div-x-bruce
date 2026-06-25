# HALEHOUND X ESP32DIV X BRUCE

This repository contains the ported source code of the **[ESP32-DIV](https://github.com/cifertech/ESP32-DIV)** firmware configured to run on the standard **2.8" Cheap Yellow Display (CYD, model `esp32-2432s028`)**, utilizing the specific hardware pinout from the **[HaleHound-CYD](https://github.com/JesseCHale/HaleHound-CYD)** project. Also included are precompiled, merged binaries for both the ported ESP32-DIV and Bruce firmware.

It also integrates automatic control of the **Ebyte E07 (CC1101 + PA + LNA)** module's Transmit/Receive enable pins for long-range SubGHz attacks.

---

## 💾 Pre-Compiled Merged Binaries

We provide precompiled, unified binaries in the **`firmware/`** folder. These are ready to flash directly to offset **`0x0`** (containing the bootloader, partitions, boot_app, and the application):

### ESP32-DIV Firmware (Ported)
*   **[`esp32_div_cyd_16mb_full.bin`](firmware/esp32_div_cyd_16mb_full.bin)**: For CYD boards with 16MB Flash (uses `app3M_fat9M_16MB` partition scheme).
*   **[`esp32_div_cyd_4mb_full.bin`](firmware/esp32_div_cyd_4mb_full.bin)**: For CYD boards with 4MB Flash (uses `huge_app` partition scheme).

### Bruce Firmware (Ported)
*   **[`Bruce-CYD-2432S028-HaleHound-pins-merged.bin`](firmware/Bruce-CYD-2432S028-HaleHound-pins-merged.bin)**: Bruce Firmware for CYD (model `esp32-2432s028`) with HaleHound pins (CC1101 GDO0/GDO2, NRF24 CSN/CE/IRQ, PN532 SS, GPS RX/TX, and E07 PA TX_EN/RX_EN).

### HaleHound-CYD Firmware
*   **[`HaleHound-CYD-FULL.bin`](firmware/HaleHound-CYD-FULL.bin)**: Official HaleHound-CYD portrait firmware (v3.5.5) compiled and merged into a single-file flashable binary for 2.8" CYD boards.

---

## 📌 Pinout & Wiring Diagram

The external modules connect to the CYD's breakout connectors (**CN1**, **P3**, and **J1 JST**) and the repurposed RGB LED pins as follows:

### 1. CC1101 SubGHz Module (Standard & Ebyte E07)
Connect standard SPI pins (sharing the VSPI bus with the SD card) and the following control lines:
*   **SCK**: GPIO 18
*   **MISO**: GPIO 19
*   **MOSI**: GPIO 23
*   **CS (Chip Select)**: GPIO 27 (available on the **CN1** connector)
*   **GDO0 (TX)**: GPIO 22 (available on both **CN1** and **P3** connectors)
*   **GDO2 (RX)**: GPIO 35 (available on the **P3** connector)

#### ⚡ Ebyte E07 Amplified Module pins:
If you are using the Ebyte E07 (CC1101 + PA + LNA), the firmware automatically drives the Power Amplifier control pins:
*   **TX_EN (Transmit Enable)**: **GPIO 4** (repurposed RGB Red pin)
*   **RX_EN (Receive Enable)**: **GPIO 0** (available on the Boot/J1 header)
*   *Note: Power the Ebyte module from an independent 5V→3.3V buck regulator as the onboard regulator cannot supply enough current.*

### 2. NRF24L01+ 2.4GHz Module
Connect standard SPI pins and the following control lines (which repurpose the CYD's RGB LED pins):
*   **SCK**: GPIO 18
*   **MISO**: GPIO 19
*   **MOSI**: GPIO 23
*   **CSN (Chip Select)**: **GPIO 4** (RGB Red pin)
*   **CE (Chip Enable)**: **GPIO 16** (RGB Green pin)
*   **IRQ**: **GPIO 17** (RGB Blue pin)

### 3. PN532 NFC/RFID Module (SPI Mode)
Set PN532 DIP switches to SPI mode (CH1=OFF, CH2=ON) and connect:
*   **SCK**: GPIO 18
*   **MISO**: GPIO 19
*   **MOSI**: GPIO 23
*   **SS (Slave Select)**: **GPIO 17** (RGB Blue pin)

### 4. GPS Module (NEO-6M / GT-U7)
Connects to the **J1 (P1)** JST header:
*   **GPS TX** → **GPIO 1** (labeled TX on the J1 header)
*   **GPS RX** → **GPIO 3** (labeled RX on the J1 header, optional/unused)

### 5. IR Transmitter & Receiver Modules
*   **IR Receiver**: Connect to **GPIO 35** (on P3 header, input-only).
    *   *Alternative:* Connect to **GPIO 3** (on J1 header) if GPS is not connected.
*   **IR Transmitter**: Connect to **GPIO 4** (RGB Red pin).
    *   *Alternative:* Connect to **GPIO 26** (buzzer pin) to avoid NRF24 CSN conflicts (the buzzer will click/buzz during transmission).
    *   *Alternative:* Connect to **GPIO 1** (on J1 header) if GPS is not connected.

---

## ⚡ Flashing Instructions

Flash the `.bin` files directly to offset **`0x0`**.

### Method 1: Web Flasher (Easiest)
1. Open **[esp.huhn.me](https://esp.huhn.me)** in a Web Serial-compatible browser (Chrome/Edge).
2. Connect your CYD to your PC.
3. Select your serial port, choose the appropriate `full.bin` file, set the address to **`0x0`**, and click **Program**.

### Method 2: Command Line (esptool)
```bash
esptool.py --chip esp32 write_flash 0x0 esp32_div_cyd_16mb_full.bin
```

---

## 🏗️ Compiling from Source

If you wish to compile the code yourself:
1. Open the project in the **Arduino IDE**.
2. Install the following libraries:
   * **`TFT_eSPI`** (replace its `User_Setup.h` with the `User_Setup cyd.h` file included in this repository's `libraries/` directory).
   * **`SmartRC-CC1101-Driver-Lib`** (use the modified version from this repository's `libraries/` folder which includes E07 PA control).
   * **`NimBLE-Arduino`** (version **`1.4.1`** required).
   * **`arduinoFFT`** (version **`1.6.2`** required).
   * **`ArduinoJson`**, **`TinyGPSPlus`**, **`Adafruit PN532`**, **`IRremoteESP8266`**, **`rc-switch`**, **`PCF8574`**, and **`XPT2046_Touchscreen`**.
3. Select board **ESP32 Dev Module** and configure settings:
   * *Upload Speed:* `921600`
   * *Partition Scheme:* `Huge APP` (for 4MB boards) or `16M Flash (3M APP/9.9M FATFS)` (for 16MB boards).
4. Compile and upload.
