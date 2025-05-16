# 📘 Chapter 4: EEG-Based Adaptive Attention Trainer

### 🎯 Project Objective

Build a **real-time attention trainer** that monitors EEG activity and provides **feedback** based on your mental focus.\
This is the foundation of neurofeedback techniques used in:

* **ADHD therapy**
* **Peak cognitive performance training**
* **Self-regulation biofeedback systems**

### 🧠 Scientific & Clinical Background

We measure attention using **EEG frequency bands**:

* **Beta (13–30 Hz)**: Associated with **focus, concentration**, alertness
* **Theta (4–8 Hz)**: Increases during **drowsiness, distraction**, or mind-wandering

👉 The **Beta/Theta Ratio** is a widely studied marker for attentional control.\
Higher ratios = More focus.\
Lower ratios = Mind drift or cognitive fatigue.

### 🧰 What You Need

#### Hardware

* EEG headset with a channel over the **frontal** or **central cortex**
  * e.g., **Fz** (frontal midline), **Cz** (central midline)

#### Software

```bash
pip install brainflow numpy matplotlib scipy pygame
```

📍 Channel Mapping

| Channel   | Region          | Purpose               |
| --------- | --------------- | --------------------- |
| Channel 1 | Cz or Fz        | Core attention signal |
| Reference | Earlobe/mastoid | Baseline voltage      |
| Ground    | Fpz             | Stability/noise floor |

💻 Step 1: Run the Attention Trainer

```python
import numpy as np
import pygame
import time
from brainflow.board_shim import BoardShim, BrainFlowInputParams, BoardIds
from scipy.signal import welch

# --- 1. Setup EEG Connection ---
params = BrainFlowInputParams()
params.serial_port = '/dev/ttyUSB0'  # ⚠️ Change this to match your device port
board = BoardShim(BoardIds.GANGLION_BOARD.value, params)

# Initialize EEG session
board.prepare_session()
board.start_stream()
print("[INFO] EEG stream started...")

# --- 2. Setup Visual Feedback Window using Pygame ---
pygame.init()
screen = pygame.display.set_mode((400, 300))
pygame.display.set_caption("EEG Attention Trainer")
font = pygame.font.SysFont(None, 36)

# --- 3. Main Loop ---
try:
    while True:
        # Acquire latest EEG data (1 second window at 256Hz)
        data = board.get_current_board_data(256)
        eeg_channel = data[1]  # ⚠️ Use correct channel (e.g., Fz or Cz)

        # Compute Power Spectral Density
        freqs, psd = welch(eeg_channel, fs=256, nperseg=256)

        # Extract frequency band powers
        beta_power = np.mean(psd[(freqs >= 13) & (freqs < 30)])
        theta_power = np.mean(psd[(freqs >= 4) & (freqs < 8)])
        attention_ratio = beta_power / (theta_power + 1e-6)

        # --- 4. Visual Feedback Based on Attention Ratio ---
        screen.fill((255, 255, 255))

        if attention_ratio > 2.0:
            color = (0, 200, 0)      # Green = focused
            message = "Focused!"
        else:
            color = (200, 0, 0)      # Red = needs focus
            message = "Focus Needed"

        # Draw feedback bar and status text
        pygame.draw.rect(screen, color, (50, 100, min(int(attention_ratio * 100), 300), 50))
        text = font.render(f"Beta/Theta Ratio: {attention_ratio:.2f} - {message}", True, (0, 0, 0))
        screen.blit(text, (20, 200))
        pygame.display.flip()

        # Allow for safe quit
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                raise KeyboardInterrupt

        time.sleep(1)

# --- 5. Cleanup ---
except KeyboardInterrupt:
    print("[INFO] Stopping EEG stream and closing window...")
    board.stop_stream()
    board.release_session()
    pygame.quit()

```

🧪 Verification & Validation

| Task                             | Expected Result          |
| -------------------------------- | ------------------------ |
| Solve a mental puzzle            | Ratio ↑ → "Focused!"     |
| Let your mind wander or daydream | Ratio ↓ → "Focus Needed" |
| Deep breathing or mindfulness    | Ratio may remain steady  |

📈 Signal Characteristics

| Frequency Band | Meaning              |
| -------------- | -------------------- |
| **Beta**       | Alertness, attention |
| **Theta**      | Drowsiness, fatigue  |
| **Ratio**      | Cognitive engagement |

### ✅ Key Takeaways

* You’ve built a **live cognitive performance visualizer**
* This tool gives **real-time biofeedback** for attention training
* Can be used in:
  * ADHD therapy sessions
  * Peak performance routines
  * Meditation or focus apps

