# 📘 Chapter 3: EEG-based Meditation and Stress Visualizer

### 🎯 Project Objective

Build a real-time EEG visualizer that quantifies and displays **mental relaxation or stress** using **alpha** and **theta** band power.

This project introduces neurofeedback principles used in:

* Meditation apps
* Cognitive behavioral therapy tools
* Wellness and biohacking platforms

### 🧠 Scientific & Clinical Background

* **Alpha waves (8–12 Hz)**\
  Appear during **relaxed wakefulness**, especially with **eyes closed** or in **calm focus**.
* **Theta waves (4–8 Hz)**\
  Dominate in **deep meditation**, **creativity**, or **drowsiness**.
* **Beta waves (13–30 Hz)**\
  Correlate with **mental effort**, **alertness**, and **stress**.

👉 By comparing **alpha and theta** to **beta**, we can visualize real-time stress/relaxation.

### 🧰 What You Need

#### Hardware

* EEG headset with channels over:
  * **Pz**, **O1**, or **O2** (parietal/occipital lobes)
* Recommended:
  * OpenBCI Ganglion, Cyton, or Emotiv Insight

#### Software

```bash
pip install brainflow matplotlib numpy scipy
```

📍 Channel Mapping

| Channel   | Brain Region            | Role               |
| --------- | ----------------------- | ------------------ |
| Channel 1 | Pz or O1/O2             | Detect alpha/theta |
| Reference | Earlobe/mastoid         | Baseline signal    |
| Ground    | Fpz or neutral position | Noise suppression  |

⚠️ If Pz is unavailable, use the closest posterior location with good alpha response.

### 💻 Step 1: Run Live EEG Stress Visualizer

Save this script as: `eeg_relaxation_visualizer.py`

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.signal import welch
from brainflow.board_shim import BoardShim, BrainFlowInputParams, BoardIds

# Set up connection parameters
params = BrainFlowInputParams()
params.serial_port = '/dev/ttyUSB0'  # ⚠️ Update this based on your platform
board = BoardShim(BoardIds.GANGLION_BOARD.value, params)

# Prepare EEG session
board.prepare_session()
board.start_stream()
print("[INFO] EEG stream started...")

# Plot setup
plt.ion()
fig, ax = plt.subplots()
bars = ax.bar(['Alpha', 'Theta', 'Beta'], [0, 0, 0])
ax.set_ylim(0, 100)
ax.set_ylabel('Band Power (scaled)')
ax.set_title('Real-Time EEG Band Power')
ax.grid(True)

try:
    while True:
        # Collect 1 second of data (assuming 256 Hz sampling rate)
        data = board.get_current_board_data(256)
        eeg_channel = data[1]  # ⚠️ Replace with your actual channel index (e.g., Pz = Channel 1)

        # Compute power spectral density
        f, psd = welch(eeg_channel, fs=256, nperseg=256)

        # Calculate band powers
        alpha_power = np.mean(psd[(f >= 8) & (f <= 12)])
        theta_power = np.mean(psd[(f >= 4) & (f < 8)])
        beta_power  = np.mean(psd[(f >= 13) & (f < 30)])

        # Update bar graph (scaled for visibility)
        bars[0].set_height(alpha_power * 1000)
        bars[1].set_height(theta_power * 1000)
        bars[2].set_height(beta_power  * 1000)

        # Refresh plot
        plt.pause(0.05)
except KeyboardInterrupt:
    print("[INFO] Stopping...")
    board.stop_stream()
    board.release_session()

```

#### 🔧 Notes for Execution

* Replace `'/dev/ttyUSB0'` with your device’s serial port:
  * Windows: `'COM3'`, `'COM5'`, etc.
  * Linux: `'/dev/ttyUSB0'`
  * macOS: `'/dev/cu.SLAB_USBtoUART'` or similar
* `data[1]` is assuming Channel 1 maps to Pz or O1/O2. Adjust if needed based on your hardware.
* If plot is empty or values are too low/high, adjust the `* 1000` scaling or set `ax.set_ylim(...)` accordingly.

🧪 Verification & Validation

Try these real-world tasks and **observe the bar graph**:

| Task                     | Expected Change        |
| ------------------------ | ---------------------- |
| Close eyes               | ↑ Alpha                |
| Deep breathing           | ↑ Theta                |
| Solve math in your head  | ↑ Beta                 |
| Multitask under pressure | ↑↑ Beta, ↓ Alpha/Theta |

✅ This lets users visualize **real-time changes in mental state**.

📈 Signal Characteristics

| Band  | Frequency | Associated State         |
| ----- | --------- | ------------------------ |
| Alpha | 8–12 Hz   | Relaxation, calm, flow   |
| Theta | 4–8 Hz    | Meditation, drowsiness   |
| Beta  | 13–30 Hz  | Focus, anxiety, thinking |

### ✅ Key Takeaways

* You built your **first real-time neuro-feedback tool**
* Learned how to extract and plot **band power**
* Established the foundation for:
  * Wellness dashboards
  * Meditation coaches
  * Clinical EEG assessment tools
