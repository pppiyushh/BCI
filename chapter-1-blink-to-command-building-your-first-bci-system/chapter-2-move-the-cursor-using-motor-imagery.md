# 📘 Chapter 2: Move the Cursor Using Motor Imagery

### 🎯 **Project Objective**

Use EEG signals generated by **motor imagery**—the imagination of left or right-hand movement—to control cursor direction on a computer screen.

This project marks the shift from detecting **muscle artifacts** (like blinks) to decoding **actual brain activity**, enabling true hands-free control systems used in:

* Neurorehabilitation
* Assistive devices for ALS/paralysis
* Brain-to-robot interfaces

### 🧠 **Scientific & Clinical Background**

Motor imagery activates the **sensorimotor cortex**, specifically the **C3 and C4** regions.

* Imagining **right hand** → activates **left motor cortex (C3)**
* Imagining **left hand** → activates **right motor cortex (C4)**

The activity is detectable by **Event-Related Desynchronization (ERD)** in:

* **μ band**: 8–12 Hz
* **β band**: 13–30 Hz

These frequency band powers **decrease** (desynchronize) during imagined movement.

<div align="right"><figure><img src="../.gitbook/assets/somatosensory-cortex-damage.avif" alt=""><figcaption></figcaption></figure></div>

### 🧰 **What You Need**

#### 🧪 Hardware

* EEG headset with electrodes over **C3 and C4**
  * E.g., OpenBCI Ganglion or Cyton
* Optional EEG cap or electrode adhesive

💻 Software

```bash
pip install brainflow pyautogui numpy scipy matplotlib scikit-learn joblib
```

📍 **Channel Mapping**

| Channel   | Brain Region            | Purpose                  |
| --------- | ----------------------- | ------------------------ |
| Channel 1 | C3 (Left Motor Cortex)  | Imagined right-hand move |
| Channel 2 | C4 (Right Motor Cortex) | Imagined left-hand move  |
| Reference | Mastoid/Earlobe         | Voltage baseline         |
| Ground    | Fpz                     | Noise suppression        |

🔹 **Step 1: Record Training Data**

Save this as `record_motor_imagery_data.py`

```python
import numpy as np
from brainflow.board_shim import BoardShim, BrainFlowInputParams, BoardIds
import time

params = BrainFlowInputParams()
params.serial_port = '/dev/ttyUSB0'
board = BoardShim(BoardIds.GANGLION_BOARD.value, params)

board.prepare_session()
board.start_stream()
data = []
labels = []

def collect_trial(label, instruction):
    print(f"\nIMAGINE: {instruction} for 5 seconds...")
    time.sleep(1)
    board.get_board_data()  # flush buffer
    time.sleep(5)
    trial_data = board.get_board_data()
    data.append(trial_data)
    labels.append(label)

for i in range(10):
    collect_trial(0, "Left hand movement")
    time.sleep(2)
    collect_trial(1, "Right hand movement")
    time.sleep(2)

board.stop_stream()
board.release_session()
np.savez("motor_imagery_data.npz", data=data, labels=labels)
```

### 🔹 **Step 2: Train the Classifier**

Save this as `train_motor_classifier.py`

```python
import numpy as np
from scipy.signal import welch
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
import joblib

def extract_features(eeg, fs=256):
    f, psd = welch(eeg, fs, nperseg=fs)
    mu_band = np.logical_and(f >= 8, f <= 12)
    beta_band = np.logical_and(f >= 13, f <= 30)
    return [np.mean(psd[mu_band]), np.mean(psd[beta_band])]

data = np.load("motor_imagery_data.npz", allow_pickle=True)
X, y = [], []

for trial, label in zip(data['data'], data['labels']):
    ch_c3 = trial[1]
    ch_c4 = trial[2]
    features = extract_features(ch_c3) + extract_features(ch_c4)
    X.append(features)
    y.append(label)

X = np.array(X)
y = np.array(y)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
clf = RandomForestClassifier().fit(X_train, y_train)
print("Accuracy:", accuracy_score(y_test, clf.predict(X_test)))

joblib.dump(clf, 'motor_model.pkl')
```

### 🔹 **Step 3: Live Cursor Control**

Save this as `live_cursor_control.py`

```python
import numpy as np
import pyautogui
from scipy.signal import welch
from brainflow.board_shim import BoardShim, BrainFlowInputParams, BoardIds
import joblib
import time

def extract_features(eeg, fs=256):
    f, psd = welch(eeg, fs, nperseg=fs)
    mu_band = np.logical_and(f >= 8, f <= 12)
    beta_band = np.logical_and(f >= 13, f <= 30)
    return [np.mean(psd[mu_band]), np.mean(psd[beta_band])]

params = BrainFlowInputParams()
params.serial_port = '/dev/ttyUSB0'
board = BoardShim(BoardIds.GANGLION_BOARD.value, params)
board.prepare_session()
board.start_stream()

clf = joblib.load('motor_model.pkl')
print("Control started... Imagine movement now.")

try:
    while True:
        data = board.get_current_board_data(256)
        ch_c3 = data[1]
        ch_c4 = data[2]
        features = extract_features(ch_c3) + extract_features(ch_c4)
        pred = clf.predict([features])[0]

        if pred == 0:
            print("Left!")
            pyautogui.moveRel(-50, 0)
        else:
            print("Right!")
            pyautogui.moveRel(50, 0)

        time.sleep(1)
except KeyboardInterrupt:
    board.stop_stream()
    board.release_session()
```

🧪 **Verification & Validation**

| Condition            | Expected Result                  |
| -------------------- | -------------------------------- |
| Imagine left hand    | Cursor moves left                |
| Imagine right hand   | Cursor moves right               |
| Do nothing           | No movement                      |
| Physically move hand | Ideally no action (filter noise) |

📈 **Signal Characteristics**

| Feature         | Value                     |
| --------------- | ------------------------- |
| Frequency Bands | μ (8–12 Hz), β (13–30 Hz) |
| Type            | ERD                       |
| Brain Regions   | C3 (left), C4 (right)     |

### ✅ **Key Takeaways**

* You’ve progressed from muscular control (blinks) to cortical control (motor imagery).
* You’ve built a **complete BCI loop**:
  * Signal acquisition
  * Feature extraction
  * Machine learning classification
  * Cursor control
* You are now ready to build spellers, prosthetic control apps, or adaptive neurofeedback tools.
