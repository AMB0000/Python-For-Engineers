
ARDUINO


void setup() {
  Serial.begin(115200);
}

void loop() {
  int value = analogRead(A0);
  Serial.println(value);
  delay(5);  //5ms sampling rate
}



----------------------
VSCODE


import serial
import matplotlib.pyplot as plt
import time 

ser = serial.Serial('/dev/tty.usbmodem14201', 115200)  # Replace with your actual port

try:
    while True:
        line_bytes = ser.readline()

        try:
            value = int(line_bytes.decode('utf-8', errors = 'ignore').strip())
        except:
            continue

        print(value)

        time.sleep(0.001)
       
except KeyboardInterrupt:
    print("Exiting")

finally:
    ser.close()
    
--------------------

THREADING


import sys
import serial
import threading
from queue import Queue
from collections import deque
import numpy as np
from PyQt5 import QtWidgets, QtCore
import pyqtgraph as pg


# shared data ques 

q = Queue()
running = True

# Serial reading thread
def read_serial(ser, q):
    while running:
        line_bytes = ser.readline()

        try:
            value = int(line_bytes.decode('utf-8', errors = 'ignore').strip())
            q.put(value)
        except:
            continue


# Qt application
app = QtWidgets.QApplication([])
win = pg.GraphicsLayoutWidget(show=True, title="Real-time Serial Plot")

#btime-domain plot
plot_time = win.addPlot(title="Time Domain")
curve_raw = plot_time.plot(pen=pg.mkPen((200,200,200)))
#curve_filtered = plot_time.plot(pen=pg.mkPen('y', width = 2))
plot_time.setYRange(-10, 1100)

# Data buffers
data = []

# Serial setup
ser = serial.Serial('/dev/tty.usbmodem14201', 115200)  # Adjust the port and baud rate as needed

thread = threading.Thread(target=read_serial, args=(ser, q))
thread.start()

fs = 200  # Sampling frequency

def update():
    global data
    
    max_per_update =50
    count = 0
   
    while not q.empty() and count < max_per_update:
        value = q.get()
        data.append(value)
        count += 1

    # keep last 100 samples 
    if len(data) > 500:
        data = data[-500:]

    if len(data) > 0:
        curve_raw.setData(data)

    #timer 
timer = QtCore.QTimer()
timer.timeout.connect(update)
timer.start(20)  # Update every 20 ms

# cleanup on close
def on_close():
    global running
    running = False
    thread.join()
    ser.close()

app.aboutToQuit.connect(on_close)

# Run 
sys.exit(app.exec_())

