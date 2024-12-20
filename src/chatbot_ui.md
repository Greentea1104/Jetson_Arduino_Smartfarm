---

# **Sensor Data Retrieval and Gradio UI Implementation Project**

This code processes **Arduino sensor data** in real-time or from a file and provides a user interface using **OpenAI API** and **Gradio UI**.

---

## **1. Project Setup**

### **Required Library Installation**

Install the following libraries:

```bash
!pip install openai
!pip install gradio
!pip install pyserial
```

### **Environment Variable Setup**

Set the `OpenAI API` key.

**Code**:
```python
import os
os.environ['OPENAI_API_KEY'] = 'please enter your key'
```

---

## **2. Full Code**

### **Basic Setup and Library Import**

```python
# Required Library Installation
!pip install openai
!pip install gradio
!pip install pyserial

# Import Libraries
import gradio as gr
import pandas as pd
import serial
import time
import json
from openai import OpenAI
import os

os.environ['OPENAI_API_KEY'] = 'please enter your key'

OpenAI.api_key = os.getenv("OPENAI_API_KEY")
```

---

### **Soil Moisture Sensor Function**

```python
def moisture_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    Processes and returns soil moisture sensor data.

    Parameters:
    - mode (str): 'real-time' or 'file'. Determines the data processing mode.
    - file_path (str): Path to the CSV file for 'file' mode.
    - port (str): Serial port for 'real-time' mode.
    - baudrate (int): Serial communication speed.

    Returns:
    - dict: Dictionary containing soil moisture data or error messages.
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Soil Moisture:" in data:
                        try:
                            soil_moisture = int(data.split(", ")[0].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Soil Moisture (%)': soil_moisture
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Soil Moisture (%)': latest_entry['Soil Moisture (%)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```

---

### **Light Sensor Function**

```python
def light_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    Processes and returns light intensity sensor data.
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Light Intensity:" in data:
                        try:
                            light_intensity = int(data.split(", ")[1].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Light Intensity (%)': light_intensity
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Light Intensity (%)': latest_entry['Light Intensity (%)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```

---

### **DHT11 Sensor Function**

```python
def dht_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    Processes and returns DHT11 sensor data (humidity and temperature).
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Humidity:" in data and "Temperature:" in data:
                        try:
                            humidity = int(data.split(", ")[2].split(":")[1].strip())
                            temperature = int(data.split(", ")[3].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Humidity (%)': humidity,
                                'Temperature (C)': temperature
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Humidity (%)': latest_entry['Humidity (%)'],
                'Temperature (C)': latest_entry['Temperature (C)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```

---

### **Gradio UI Implementation**

```python
def process(user_message, chat_history):
    proc_messages, ai_message = ask_openai("gpt-4o-mini", messages, user_message, functions=sensor_functions)
    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="Sensor Data Query")
    user_textbox = gr.Textbox(label="Input")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

Let me know if you'd like further refinements! 🌟
