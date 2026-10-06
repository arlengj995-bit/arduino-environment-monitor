# Project 2 — Environment Monitor

An Arduino Uno environment monitor that measures temperature, relative humidity, and ambient light level. A DHT11 provides temperature and humidity, while a light-dependent resistor (LDR) provides a light measurement through a voltage divider. A 16×2 LCD displays the readings, and a Python logger records the Serial output and generates a graph.

This project develops sensor reading, error handling, display, and data logging skills for a future automated aquarium controller.

## Features

- Two sensors measure three environmental quantities.
- LCD readings update approximately every 2 seconds.
- Serial output sends temperature, humidity, and raw light readings at 9600 baud.
- An `isnan()` guard prevents invalid DHT11 readings from entering the normal display and numeric output.
- Python collects timestamped readings, saves a CSV after collection ends, and generates three plots.

## Components

| Component | Purpose |
| --- | --- |
| Arduino Uno / compatible Uno R3 | Reads sensors and controls the display |
| DHT11 module | Temperature and relative humidity |
| LDR | Responds to ambient light |
| 10 kΩ fixed resistor | Forms the LDR voltage divider |
| LCD 1602 with parallel interface | Displays measurements |
| 10 kΩ potentiometer | Adjusts LCD contrast |
| Breadboard, jumper wires, and USB cable | Connections and power |

## Wiring

| Signal | Arduino connection |
| --- | --- |
| DHT11 data | D7 |
| LDR divider midpoint | A0 |
| LCD RS | D12 |
| LCD E | D11 |
| LCD D4 | D5 |
| LCD D5 | D4 |
| LCD D6 | D3 |
| LCD D7 | D2 |
| Module supply | 5V |
| Common ground | GND |

The divider is wired as `5V → LDR → A0 midpoint → 10 kΩ resistor → GND`. More light lowers the LDR resistance and raises the midpoint voltage. LCD R/W is connected to GND. The contrast potentiometer outer terminals connect to 5V and GND; its wiper connects to LCD V0. See the diagrams for the remaining LCD power and backlight connections.

![Breadboard wiring](Project%202_Environment%20Monitor_Breadboard.jpg)

[View the schematic](Project%202_Environment%20Monitor_Schema.jpg).

## Software and operation

The source files are in the parent project folder:

- [Arduino sketch](../Environment_Monitor_3/Environment_Monitor_3.ino)
- [Python logger](../logger.py)

These links require the parent project folder; the submission folder alone contains the report and evidence, rather than separate source files.

1. Build the circuit using the wiring table and diagrams.
2. In Arduino IDE, install the Adafruit DHT sensor library and its required dependencies. The sketch also uses `LiquidCrystal`.
3. Open `Environment_Monitor_3.ino`, select the Uno and its serial port, and upload the sketch.
4. Adjust the contrast potentiometer until the LCD text is readable.
5. Open Serial Monitor at 9600 baud and confirm numeric lines such as `24.8,49.0,448`.
6. Close Serial Monitor before starting Python so the logger can open the port.
7. With Python installed, install the logger dependencies:

   ```sh
   python -m pip install pyserial matplotlib
   ```

8. Set `PORT` in `logger.py` to the Arduino's port. The current setting is `COM6`.
9. Run the logger from the parent project folder:

   ```sh
   python logger.py
   ```

The logger is configured for 1,800 seconds (30 minutes). Ctrl+C stops collection early and allows the collected data to be saved and plotted. It writes `environment_log.csv` and `environment_graph.png` in the working directory. The submitted evidence files have been renamed for this project.

## Data format and light conversion

Arduino sends three comma-separated fields:

```text
temperature_c,humidity_pct,light_raw
24.8,49.0,448
```

The logger adds a computer timestamp and calculated voltage:

```csv
time,temp_c,humid_pct,light_raw,light_volts
23:49:01,25.2,47.0,401,1.96
```

Voltage is calculated as `light_raw × (5.0 / 1023.0)`, assuming a 5V ADC reference. Light readings are divider voltage, not calibrated lux. The CSV timestamps contain time of day only; this session crosses midnight.

If the DHT11 reading fails, the sketch displays `DHT11 Error!`, sends an error message, waits, and retries. The logger skips that message because it is not a three-field numeric CSV line.

## Submitted results

The submitted CSV contains **400 readings**, from **23:49:01 to 00:02:36**, spanning **13 minutes 35 seconds** across midnight.

| Quantity | Recorded minimum | Recorded maximum |
| --- | --- | --- |
| Temperature | 24.2°C | 27.6°C |
| Relative humidity | 46% RH | 73% RH |
| Light divider voltage | 0.024V | 2.253V |

Three deliberate actions produced visible responses:

- **Covering the LDR:** the main covered interval near 23:54 drops to approximately 0.13–0.16V, then recovers.
- **Breathing on the DHT11:** temperature and humidity rise near 23:59, reaching session peaks of 27.6°C and 73% RH. Humidity declines to 49% RH by the end; temperature ends at 26.5°C.
- **Turning the room light off:** light voltage falls close to zero near 00:01 and returns to approximately 2V before recording ends.

These responses demonstrate that the monitor reacts to environmental changes. They do not establish absolute measurement accuracy. Smaller fluctuations cannot all be assigned to specific actions without a timestamped event diary.

**Graph title note:** the image below retains the title “30 Minute Log,” which reflects the logger's configured duration. The submitted recording spans 13 minutes 35 seconds.

![Temperature, humidity, and light graph](Project%202_Environment%20Monitor_Graph.png)

## Submission files

| File | Description |
| --- | --- |
| [Report R1](Project%202_Environment%20Monitor_R1.docx) | Explanation, code, troubleshooting, and results |
| [CSV log](Project%202_Environment%20Monitor_Log.csv) | 400 timestamped readings |
| [Graph](Project%202_Environment%20Monitor_Graph.png) | Temperature, humidity, and light plots |
| [Breadboard diagram](Project%202_Environment%20Monitor_Breadboard.jpg) | Circuit layout |
| [Schematic](Project%202_Environment%20Monitor_Schema.jpg) | Electrical connections |
| [Image 1](Project%202_Environment%20Monitor_Image%201.jpeg) | Earlier build stage showing a startup message |
| [Image 2](Project%202_Environment%20Monitor_Image%202.jpeg) | Intermediate stage showing temperature and humidity |
| [Image 3](Project%202_Environment%20Monitor_Image%203.jpeg) | Complete build showing all three measurements |

## Troubleshooting and lessons

- **Blank LCD:** adjust contrast, then check the RS and E wires against the sketch. This build required RS on D12 and E on D11.
- **Labels instead of CSV:** upload the final sketch; the Arduino runs the last uploaded program.
- **Logger cannot access the port:** close Serial Monitor and check the port setting.
- **DHT11 error:** check module power, ground, and data wiring. The sketch waits 2 seconds before retrying.

The main lessons were keeping wiring consistent with code, checking sensor data before using it, and comparing reported results with the actual CSV rather than the planned logging duration.
