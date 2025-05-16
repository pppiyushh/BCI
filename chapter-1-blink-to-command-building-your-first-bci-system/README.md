# Chapter 1: Blink to Command — Building Your First BCI System



### 🎯 Project Objective

Use a non-invasive EEG headset to **detect eye blinks** and trigger a basic action (e.g., pressing the spacebar) on your computer.

This forms the foundation for real-world BCI systems, including:

* **Hands-free interfaces** for disabled users
* **Calibration steps** in more advanced neurotech pipelines
* A gentle on-ramp into EEG signal processing

### 🧠 Understanding the Signal: Eye Blinks & the Frontal Lobe

#### What causes the signal?

Eye blinks generate **strong muscle artifacts** due to the **orbicularis oculi muscle** activating around the eyes. These artifacts:

* Are **non-cortical** in origin
* Appear clearly on **prefrontal electrodes**
* Can spike to **±100–300 µV** in amplitude

#### Where do they show up?

**Electrode locations** based on the international **10–20 system**:

| Electrode | Region           | Signal Role          |
| --------- | ---------------- | -------------------- |
| **Fp1**   | Left prefrontal  | Primary blink signal |
| **Fp2**   | Right prefrontal | Secondary            |
| **Ref**   | Earlobe/mastoid  | Reference voltage    |
| **GND**   | Forehead (Fpz)   | Electrical ground    |

These are not “brain commands” but intentional **muscle-based triggers**.

#### &#x20;What You Need:

### 🧰 What You Need

#### Hardware

* EEG Headset (choose one):
  * **OpenBCI Ganglion** (preferred, 4–8 channels)
  * **NeuroSky MindWave** (1 channel at Fp1)
  * **Emotiv Insight** (multiple prefrontal channels)
* Bluetooth/USB interface to PC
* EEG cap (optional) or dry-contact electrodes

Software Setup

```bash
pip install brainflow pyautogui numpy matplotlib
```

* **BrainFlow SDK** — unified interface for EEG headsets
* **pyautogui** — lets Python simulate keyboard/mouse
* **NumPy** — handles numerical signal processing

🔌 Channel Mapping (Critical)

| Device            | EEG Channel | Physical Location | Notes                 |
| ----------------- | ----------- | ----------------- | --------------------- |
| OpenBCI Ganglion  | Channel 1   | Fp1               | Good blink visibility |
| OpenBCI Ganglion  | Channel 2   | Fp2               | Optional secondary    |
| NeuroSky MindWave | Only 1 ch   | Fp1               | Limited but usable    |
| Emotiv Insight    | AF3 / AF4   | Near Fp1/Fp2      | Acceptable substitute |

Make sure your **electrode placement** and **channel mapping** match the software script.

### 🧪 Blink Detection Strategy

We’ll use **thresholding** — a basic but reliable method. A blink appears as a **sharp voltage spike** in Fp1/Fp2 that exceeds ±100 µV.

> ✅ Pros: Simple and effective\
> ❌ Cons: Doesn’t distinguish between blinks and other facial artifacts

You’ll improve this in later chapters using band-power, machine learning, or SSVEP detection.

Full Code: `blink_detector.py`

```python
from brainflow.board_shim import BoardShim, BrainFlowInputParams, BoardIds
import numpy as np
import pyautogui
import time

# Set up board connection
params = BrainFlowInputParams()
params.serial_port = '/dev/ttyUSB0'  # ⚠️ Replace with your EEG device port
board_id = BoardIds.GANGLION_BOARD.value  # Or your specific headset
board = BoardShim(board_id, params)

# Prepare EEG session
board.prepare_session()
board.start_stream()
print("[INFO] EEG Stream started. Blink intentionally to trigger events...")

try:
    while True:
        data = board.get_current_board_data(256)  # Get ~1s of data at 256 Hz
        eeg_channel = data[1]  # ⚠️ Use Fp1, usually Channel 1 on Ganglion

        # Blink detection (µV threshold)
        if np.max(np.abs(eeg_channel)) > 100:
            print("[DETECTED] Blink!")
            pyautogui.press('space')
            time.sleep(1)  # Avoid retriggering too fast
except KeyboardInterrupt:
    print("[STOP] KeyboardInterrupt received. Shutting down.")
    board.stop_stream()
    board.release_session()

```

🧪 Calibration & Testing

| Test Task            | What to Look For                             |
| -------------------- | -------------------------------------------- |
| Blink once slowly    | Spike over 100 µV on console print           |
| Move eyes left/right | Should not trigger if threshold is good      |
| Shake head           | May show spikes — adjust threshold higher    |
| Blink rapidly        | System should wait due to cooldown (`sleep`) |

If false triggers occur:

* Increase the threshold (e.g., 150 µV)
* Add a refractory period (`time.sleep(1.5)`)
* Use multiple channels for consensus logic

🧠 Signal Characteristics

| Parameter | Typical Value                              |
| --------- | ------------------------------------------ |
| Amplitude | 100–300 µV                                 |
| Duration  | 200–400 ms                                 |
| Frequency | Low freq (<4 Hz) — transient, not periodic |

### 📌 Key Takeaways

* You built a **working BCI** with nothing but an EEG headset and Python.
* You learned about:
  * Blink signal physiology
  * Frontal lobe placement (Fp1/Fp2)
  * Threshold-based signal recognition
  * Real-world interface automation

**Chapter 2 will now move toward true cortical control**:

> Imagining left vs. right hand movement and moving a cursor accordingly.
