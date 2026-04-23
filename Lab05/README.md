# 📡 Real-Time Arduino Serial Plotter

A real-time signal visualization tool that reads analog data from an Arduino over serial and plots it live using Python, PyQt5, and pyqtgraph.

---

## 🗂 Project Structure

```
.
├── arduino/
│   └── serial_sender.ino       # Arduino sketch
├── python/
│   ├── simple_reader.py        # Basic serial reader (no GUI)
│   └── realtime_plotter.py     # Threaded real-time plotter (PyQt5)
└── README.md
```

---

## ⚙️ Arduino Sketch

Reads from analog pin `A0` at ~200 Hz and sends values over serial.

```cpp
void setup() {
  Serial.begin(115200);
}

void loop() {
  int value = analogRead(A0);
  Serial.println(value);
  delay(5);  // 5ms → 200 Hz sampling rate
}
```

**Upload to your Arduino board via the Arduino IDE before running any Python script.**

---

## 🐍 Python Scripts

### 1. Simple Serial Reader

Minimal script — reads and prints serial values to the console with no GUI.

```python
import serial
import time

ser = serial.Serial('/dev/tty.usbmodem14201', 115200)  # Replace with your port

try:
    while True:
        line_bytes = ser.readline()
        try:
            value = int(line_bytes.decode('utf-8', errors='ignore').strip())
        except:
            continue
        print(value)
        time.sleep(0.001)

except KeyboardInterrupt:
    print("Exiting")
finally:
    ser.close()
```

---

### 2. Real-Time Threaded Plotter

Full GUI plotter using PyQt5 + pyqtgraph. Serial reading runs on a background thread to avoid blocking the UI.

```python
import sys
import serial
import threading
from queue import Queue
import numpy as np
from PyQt5 import QtWidgets, QtCore
import pyqtgraph as pg

# Shared state
q = Queue()
running = True

# --- Serial Thread ---
def read_serial(ser, q):
    while running:
        line_bytes = ser.readline()
        try:
            value = int(line_bytes.decode('utf-8', errors='ignore').strip())
            q.put(value)
        except:
            continue

# --- Qt App Setup ---
app = QtWidgets.QApplication([])
win = pg.GraphicsLayoutWidget(show=True, title="Real-time Serial Plot")

# Time-domain plot
plot_time = win.addPlot(title="Time Domain")
curve_raw = plot_time.plot(pen=pg.mkPen((200, 200, 200)))
plot_time.setYRange(-10, 1100)

# Data buffer
data = []

# --- Serial Setup ---
ser = serial.Serial('/dev/tty.usbmodem14201', 115200)  # Adjust port
thread = threading.Thread(target=read_serial, args=(ser, q))
thread.start()

fs = 200  # Sampling frequency (Hz)

# --- Update Callback ---
def update():
    global data

    max_per_update = 50
    count = 0

    while not q.empty() and count < max_per_update:
        value = q.get()
        data.append(value)
        count += 1

    if len(data) > 500:
        data = data[-500:]  # Keep last 500 samples

    if len(data) > 0:
        curve_raw.setData(data)

timer = QtCore.QTimer()
timer.timeout.connect(update)
timer.start(20)  # Refresh every 20ms

# --- Cleanup ---
def on_close():
    global running
    running = False
    thread.join()
    ser.close()

app.aboutToQuit.connect(on_close)
sys.exit(app.exec_())
```

---

## 📦 Requirements

Install dependencies via pip:

```bash
pip install pyserial pyqtgraph PyQt5 numpy
```

---

## 🔌 Finding Your Serial Port

| OS      | Example Port              |
|---------|---------------------------|
| macOS   | `/dev/tty.usbmodem14201`  |
| Linux   | `/dev/ttyACM0`            |
| Windows | `COM3`                    |

To list available ports:

```bash
# macOS / Linux
ls /dev/tty.*

# Windows (PowerShell)
[System.IO.Ports.SerialPort]::getportnames()
```

Update the port string in whichever script you run.

---

## 🚀 Usage

1. Flash `serial_sender.ino` to your Arduino
2. Connect Arduino via USB
3. Update the serial port in your chosen Python script
4. Run the plotter:

```bash
python realtime_plotter.py
```

---

## 📊 How It Works

```
Arduino (A0)
    │
    │  Serial @ 115200 baud
    ▼
read_serial() thread  ──→  Queue  ──→  update() @ 20ms  ──→  pyqtgraph plot
```

- **Sampling rate:** 200 Hz (5ms delay on Arduino)
- **Buffer size:** 500 samples (~2.5 seconds of data)
- **Plot refresh:** Every 20ms (50 FPS)
- **Thread-safe:** Serial I/O is isolated on a background thread

---

## 📝 Notes

- `analogRead()` returns values in the range `0–1023` (10-bit ADC)
- The Y-axis is fixed to `-10` to `1100` to cover the full ADC range
- Extend the plotter by adding an FFT plot on a second `win.addPlot()` row
