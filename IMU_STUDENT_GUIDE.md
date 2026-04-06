# IMU Data Logger - Student Guide

Record accelerometer, gyroscope, magnetometer, and temperature data using the OpenLog Artemis data logger. No GPS or external hardware required.

---

## What You Need

- SparkFun OpenLog Artemis (has a built-in ICM-20948 IMU)
- MicroSD card (FAT32 formatted)
- USB-C cable
- Computer with Arduino IDE installed
- MATLAB (for plotting)

---

## 1. Arduino IDE Setup

### Install the Board Package

1. Open Arduino IDE
2. Go to **File > Preferences**
3. In "Additional Board Manager URLs", add:
   ```
   https://raw.githubusercontent.com/sparkfun/Arduino_Apollo3/main/package_sparkfun_apollo3_index.json
   ```
4. Go to **Tools > Board > Boards Manager**
5. Search for **SparkFun Apollo3** and install **version 2.2.1** (must be this exact version)

### Install Libraries

Open **Sketch > Include Library > Manage Libraries** and install:

- **SparkFun u-blox GNSS v3** by SparkFun Electronics
- **SparkFun 9DoF IMU Breakout ICM 20948 Arduino Library** by SparkFun Electronics
- **SdFat - Adafruit Fork** by Bill Greiman

### Board Settings

- **Tools > Board:** SparkFun Apollo3 > RedBoard Artemis ATP
- **Tools > Port:** Select the port that appears when you plug in the board

---

## 2. Upload the Firmware

1. Insert a FAT32-formatted microSD card into the OpenLog Artemis
2. Connect the OpenLog Artemis to your computer via USB-C
3. In Arduino IDE, open the sketch: `OpenLog_Artemis_GNSS_Logging_Modified/OpenLog_Artemis_GNSS_Logging.ino`
   - All `.ino` and `.h` files in the folder must be present -- Arduino compiles them together
4. Select the correct board and port (see above)
5. Click **Upload**
6. Open **Tools > Serial Monitor** and set baud rate to **115200**

---

## 3. What to Expect on Boot

You will see output like this:

```
Artemis OpenLog GNSS v3.2
Starting IMU initialization...
IMU online
SD card online
Data logging online
Created log file: dataLog00001.ubx
Opening IMU file: imuLog00001.csv
IMU file opened successfully
IMU CSV header written
GNSS offline
```

**"GNSS offline" is normal.** It just means no GPS module is connected. The IMU logs independently.

Take note of the IMU log file number (e.g., `imuLog00001.csv`) -- you will need this later.

---

## 4. Recording Data

Once you see "Returning to logging..." or the boot messages finish, the logger is recording IMU data to the SD card.

**Default settings:**
- IMU sample rate: 100 ms (10 Hz)
- All sensors enabled: accelerometer, gyroscope, magnetometer, temperature

### What the Terminal Output Looks Like

If terminal output is enabled, you will see lines like:

```
2026/04/06 17:33:52.88,IMU,-12.45,3.67,1021.33,-0.05,0.12,-0.03,25.60,-8.12,42.10,24.3
```

Each line is: `Timestamp,IMU,AccX,AccY,AccZ,GyrX,GyrY,GyrZ,MagX,MagY,MagZ,Temp`

### Collecting a Dataset

1. Place the logger on a flat surface (for a baseline), or hold it and move/rotate it
2. Note the start time
3. Record for at least 30 seconds
4. Note the end time

---

## 5. Stop Logging and Retrieve Data

1. Open the Serial Monitor (if not already open)
2. Type any character and press Enter -- this opens the menu
3. Press **q** then **y** to quit and safely close the log file
4. Unplug USB
5. Remove the microSD card
6. Insert the SD card into your computer and copy the `imuLogXXXXX.csv` file

**Important:** Always quit through the menu before removing the SD card. Pulling power without quitting can corrupt the file.

---

## 6. Plot the Data

### Using MATLAB

Open `matlab/simple_IMU_plotting.m` and change the filename on line 7:

```matlab
data = readtable('imuLog00001.csv');  % <-- your file number here
```

Run the script. You will get four subplots:

| Plot | Axes | Units | What It Shows |
|------|------|-------|---------------|
| Accelerometer | X, Y, Z | m/s^2 | Linear acceleration (gravity appears as ~9.81 on Z when flat) |
| Gyroscope | X, Y, Z | deg/s | Rotational rate (should be near zero when still) |
| Magnetometer | X, Y, Z | uT | Magnetic field (direction/magnitude changes with orientation) |
| Temperature | - | C | Board temperature |

### Using Python (alternative)

```python
import pandas as pd
import matplotlib.pyplot as plt

data = pd.read_csv('imuLog00001.csv')

fig, axes = plt.subplots(2, 2, figsize=(12, 8))

# Accelerometer (convert milli-g to m/s^2)
axes[0,0].plot(data['AccX']/1000*9.81, label='X')
axes[0,0].plot(data['AccY']/1000*9.81, label='Y')
axes[0,0].plot(data['AccZ']/1000*9.81, label='Z')
axes[0,0].set_title('Accelerometer (m/s^2)')
axes[0,0].legend()
axes[0,0].grid(True)

# Gyroscope (deg/s)
axes[0,1].plot(data['GyrX'], label='X')
axes[0,1].plot(data['GyrY'], label='Y')
axes[0,1].plot(data['GyrZ'], label='Z')
axes[0,1].set_title('Gyroscope (deg/s)')
axes[0,1].legend()
axes[0,1].grid(True)

# Magnetometer (uT)
axes[1,0].plot(data['MagX'], label='X')
axes[1,0].plot(data['MagY'], label='Y')
axes[1,0].plot(data['MagZ'], label='Z')
axes[1,0].set_title('Magnetometer (uT)')
axes[1,0].legend()
axes[1,0].grid(True)

# Temperature (C)
axes[1,1].plot(data['Temp'], color='purple')
axes[1,1].set_title('Temperature (C)')
axes[1,1].grid(True)

plt.tight_layout()
plt.show()
```

---

## 7. Menu Reference

Press any key in the Serial Monitor to open the menu during logging.

| Key | Action |
|-----|--------|
| **3** | Configure IMU -- enable/disable sensors, change sample rate |
| **1** | Configure logging -- toggle SD logging, terminal output |
| **q** | Quit -- close log file and power down safely |
| **f** | Start a new log file (without restarting) |
| **x** | Return to logging |

### IMU Settings (Menu 3)

```
1) IMU Logging           : Enabled/Disabled
2) Log Accelerometer     : Enabled/Disabled
3) Log Gyroscope         : Enabled/Disabled
4) Log Magnetometer      : Enabled/Disabled
5) Log Temperature       : Enabled/Disabled
6) IMU Log Rate (ms)     : 100
```

You can change the sample rate (option 6) to values between 10 ms (100 Hz) and 5000 ms (0.2 Hz). Settings are saved and persist across reboots.

---

## IMU CSV File Format

| Column | Description | Units |
|--------|-------------|-------|
| Timestamp | Date and time | yyyy/MM/dd HH:mm:ss.SS |
| Sensor | Always "IMU" | - |
| AccX, AccY, AccZ | Acceleration | milli-g |
| GyrX, GyrY, GyrZ | Angular velocity | deg/s |
| MagX, MagY, MagZ | Magnetic field | uT |
| Temp | Board temperature | C |

**Note:** Accelerometer values are in milli-g (1000 = 1g = 9.81 m/s^2). The MATLAB and Python scripts convert to m/s^2 for you.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "SD card offline" | Format your SD card as FAT32. Make sure it's seated properly. |
| "IMU initialization failed" | Try pressing the reset button on the board. If it persists, check that the board is a genuine OpenLog Artemis with ICM-20948. |
| Serial Monitor shows nothing | Check baud rate is set to 115200. Try pressing reset. |
| CSV file is empty | Did you quit properly (q, then y)? Data may not have been flushed. |
| "GNSS offline" | **This is expected.** You are not using GPS in this exercise. |
| IMU values show "%.2f" | Firmware is outdated. Re-upload the latest version from the repo. |
| Menu opened accidentally | Press **x** to return to logging. The menu times out after 15 seconds automatically. |
