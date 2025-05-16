# 📘 Chapter 5: BCI-Controlled Robot Arm

### 🎯 Project Objective

Control a physical robot arm using **EEG-based brain signals**.

This project showcases:

* Real-world actuation
* Brain-to-machine command flow
* How BCI can assist motor-impaired users

### 🧠 Scientific & Engineering Background

This system detects **motor imagery** or **eye blinks** and sends movement commands to a **robot arm** through an **Arduino** interface.

Inspired by:

* Neuroprosthetics
* Brain-controlled assistive devices
* Rehabilitation robotics

### 🧰 What You Need

#### Hardware

* **Arduino UNO or Nano**
* **4–6 DOF robot arm** (SunFounder, Adeept, or any servo-based kit)
* Jumper wires, USB cable, power supply
* EEG headset with at least 2 channels (e.g., OpenBCI Ganglion)

Software

```bash
pip install brainflow pyserial numpy scipyBash
```

Arduino IDE to upload servo control code

📍 Channel Mapping & Signal

| Channel   | Brain Area | Signal Type                |
| --------- | ---------- | -------------------------- |
| Channel 1 | C3         | Motor imagery (right hand) |
| Channel 2 | C4         | Motor imagery (left hand)  |
| Ref       | Mastoid    | Baseline signal            |
| Ground    | Fpz        | Noise suppression          |

Alternatively, you can use Fp1/Fp2 if building a blink-triggered arm instead.

### 🧠 Arduino Code (`robot_arm.ino`)

This controls the robot arm using simple serial commands from the Python script.

```cpp
#include <Servo.h>

Servo base;  // Create servo object to control the base rotation

void setup() {
  base.attach(9);         // Connect servo signal pin to digital pin 9
  Serial.begin(9600);     // Start serial communication at 9600 bps
}

void loop() {
  if (Serial.available()) {
    char cmd = Serial.read();  // Read incoming character

    if (cmd == 'L') {
      base.write(0);           // Rotate arm to LEFT position
    }

    if (cmd == 'R') {
      base.write(180);         // Rotate arm to RIGHT position
    }
  }
}
```

### 🧠 Python Code (`bci_robot_arm_control.py`)

This script reads EEG data via BrainFlow, calculates μ-band power, and sends control commands over serial to the Arduino.

```python
import serial
import time
import numpy as np
from brainflow.board_shim import BoardShim, BrainFlowInputParams, BoardIds
from scipy.signal import welch

# --- EEG Setup ---
params = BrainFlowInputParams()
params.serial_port = '/dev/ttyUSB0'  # Replace with your EEG COM port
board = BoardShim(BoardIds.GANGLION_BOARD.value, params)

board.prepare_session()
board.start_stream()
print("[INFO] EEG stream started...")

# --- Arduino Setup ---
arduino = serial.Serial('/dev/ttyACM0', 9600)  # Replace with your Arduino COM port
time.sleep(2)  # Let Arduino boot up

try:
    while True:
        data = board.get_current_board_data(256)  # Get 1 second of data
        ch_c3 = data[1]  # Channel 1: C3 (left motor cortex)
        ch_c4 = data[2]  # Channel 2: C4 (right motor cortex)

        f, psd_c3 = welch(ch_c3, fs=256)
        _, psd_c4 = welch(ch_c4, fs=256)

        mu_c3 = np.mean(psd_c3[(f >= 8) & (f <= 12)])
        mu_c4 = np.mean(psd_c4[(f >= 8) & (f <= 12)])

        if mu_c3 < mu_c4:
            arduino.write(b'L')  # Move arm LEFT
            print("[ACTION] Move Left")
        else:
            arduino.write(b'R')  # Move arm RIGHT
            print("[ACTION] Move Right")

        time.sleep(1)

except KeyboardInterrupt:
    print("[INFO] Stopping stream and closing connections...")
    board.stop_stream()
    board.release_session()
    arduino.close()
```

🧪 Verification & Testing

| Test Condition       | Expected Result         |
| -------------------- | ----------------------- |
| Imagine right hand   | Arm moves left          |
| Imagine left hand    | Arm moves right         |
| Physical hand move   | Should not trigger      |
| Check Arduino Serial | Confirm `L` or `R` sent |

### 📈 Signal Characteristics

* **Motor Imagery** = ERD in μ band (8–12 Hz)
* **C3** = Left motor cortex → right hand
* **C4** = Right motor cortex → left hand
* We compare μ suppression between C3 and C4 to infer intent

***

### ✅ Key Takeaways

* You've closed the loop: **Thought → Signal → Robot Action**
* This is a foundation for:
  * Brain-controlled prosthetics
  * Assistive robotics
  * Neural control of exoskeletons or drones
