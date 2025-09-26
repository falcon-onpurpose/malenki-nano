# How to Flash Firmware on the Malenki-Nano HV Board

This guide documents the process for flashing firmware onto the Malenki-Nano HV board, which uses an ATtiny1616 microcontroller. The recommended method is to use an Arduino Uno (or a similar board) as a `jtag2updi` programmer. This approach is reliable, especially on systems like M1/M2 Macs where drivers for other USB-to-Serial adapters can be problematic.

## Required Hardware

*   An Arduino Uno board
*   The target Malenki-Nano HV board
*   A 4.7k Ohm resistor
*   Jumper wires
*   A USB cable for the Arduino Uno

## Required Software

*   [Arduino IDE](https://www.arduino.cc/en/software)
*   The `jtag2updi` firmware for the Arduino
*   The `avrdude` command-line utility
*   The Malenki-Nano HV firmware file (`hv.bin`) located in the `hv_flash/` directory of this project.

---

## Step 1: Prepare the Arduino Programmer

First, you need to turn your Arduino Uno into a dedicated UPDI programmer.

1.  **Download the Programmer Sketch:** The code is a separate project from `megaTinyCore`. Download it from its official repository:
    *   **https://github.com/SpenceKonde/jtag2updi**
    *   Click the green "Code" button, then "Download ZIP".

2.  **Upload to Uno:**
    *   Unzip the downloaded file.
    *   Navigate into the unzipped folder, then into the inner `jtag2updi` folder.
    *   Open the `jtag2updi.ino` file with the Arduino IDE.
    *   Connect your Arduino Uno to your computer.
    *   In the Arduino IDE, select **Tools > Board > Arduino AVR Boards > Arduino Uno**.
    *   Select the correct serial port for your Uno under **Tools > Port**.
    *   Click the "Upload" button to flash the `jtag2updi` sketch to your Uno.

## Step 2: Wire the Programmer to the Malenki-Nano

With the sketch uploaded, disconnect the Uno from your computer and wire it to the Malenki-Nano's J2 programming header as follows:

```
   Arduino Uno         Malenki-Nano J2 Header
  +-----------+       +----------------------+
  |      GND  |------>| Pin 3 (GND)          |
  |       5V  |------>| Pin 1 (VBAT)         |
  |      D6   |---+-->| Pin 6 (UPDI)         |
  +-----------+   |   +----------------------+
                  |
              [ 4.7kΩ ]
              [Resistor]
```

*   **Uno `GND` pin** → **Malenki J2 Pin 3** (or Pin 4, also GND)
*   **Uno `5V` pin** → **Malenki J2 Pin 1** (VBAT)
*   **Uno `Digital Pin 6`** → One leg of the **4.7kΩ Resistor**
*   The other leg of the **4.7kΩ Resistor** → **Malenki J2 Pin 6** (UPDI)

*Note: If you experience instability, a 10µF capacitor can be connected between the Uno's `RESET` and `GND` pins. This prevents the Uno from resetting when the serial connection is opened, but it is often not required.*

## Step 3: Install `avrdude`

This process uses the `avrdude` command-line tool. On macOS, the easiest way to install it is with [Homebrew](https://brew.sh/):

```bash
brew install avrdude
```

## Step 4: Flash the Firmware

1.  **Connect the Uno:** Plug the fully wired Arduino Uno programmer into your computer.

2.  **Find the Serial Port:** Open a terminal and run `ls /dev/tty.*` to list available serial ports. The port for your Uno will likely be named `/dev/tty.usbserial-XXXX` or `/dev/tty.usbmodemXXXX`. Note this port name.

3.  **Run the Command:** Navigate to the root directory of the `malenki-nano` project in your terminal and execute the following command. **Remember to replace `/dev/tty.usbserial-1140` with the actual port name for your Uno.**

    ```bash
    avrdude -p t1616 -c jtag2updi -P /dev/tty.usbserial-1140 -b 115200 -U flash:w:hv_flash/hv.bin:r
    ```

    *   `-p t1616`: Specifies the target microcontroller (ATtiny1616).
    *   `-c jtag2updi`: Specifies the programmer type (your Uno).
    *   `-P /dev/tty.usbserial-1140`: The serial port for your Uno.
    *   `-b 115200`: The communication speed.
    *   `-U flash:w:hv_flash/hv.bin:r`: The main operation: Write (`w`) to the `flash` memory with the contents of the `hv_flash/hv.bin` file.

If successful, you will see progress bars for writing and reading, ending with a "Thank you." message. The board is now flashed.