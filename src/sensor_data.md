---

## Collecting Arduino Sensor Data (`collect_sensor_data.md`)

This script provides code for real-time collection of Arduino sensor data using **Python** and **Pandas**, storing the data in a **CSV file**.  
The script is designed to run in a **virtual environment** on **Jetson Nano** via **Jupyter Notebook**.

---

### **Full Code**

```python
import serial
import time
import pandas as pd

# Serial communication setup
port = "/dev/ttyUSB0"  # Arduino serial port
baudrate = 9600        # Baud rate must match Arduino's settings
arduino = serial.Serial(port, baudrate, timeout=1)

# List to store data
data = []

# Initialize current date
current_date = time.strftime("%Y-%m-%d")

try:
    print("Collecting data from Arduino... Press Ctrl+C to stop.")
    while True:
        if arduino.in_waiting > 0:  # Check if data is available on the serial port
            line = arduino.readline().decode("utf-8").strip()  # Read and decode data
            
            # Filter data: Process only valid sensor data
            if "Soil Moisture:" in line and "Light Intensity:" in line:
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")  # Record current time
                
                # Parse sensor data
                parts = line.split(", ")
                soil_moisture = parts[0].split(":")[1].strip()  # Soil Moisture value
                light_intensity = parts[1].split(":")[1].strip()  # Light Intensity value
                humidity = parts[2].split(":")[1].strip()  # Humidity value
                temperature = parts[3].split(":")[1].strip()  # Temperature value
                
                print(f"{timestamp}, Soil Moisture: {soil_moisture}, Light Intensity: {light_intensity}, Humidity: {humidity}, Temperature: {temperature}")
                
                # Store data
                data.append({
                    "Timestamp": timestamp,
                    "Soil Moisture (%)": soil_moisture,
                    "Light Intensity (%)": light_intensity,
                    "Humidity (%)": humidity,
                    "Temperature (C)": temperature
                })

                # Create a DataFrame
                df = pd.DataFrame(data)

                # Generate a file name based on the current date
                file_name = f"sensor_data_{current_date.replace('-', '')}.csv"

                # Save the DataFrame to a CSV file
                df.to_csv(file_name, index=False, encoding="utf-8")
                print(f"Data saved to {file_name}")

except KeyboardInterrupt:
    print("Data collection stopped.")

finally:
    # Save remaining data
    if data:
        df = pd.DataFrame(data)
        file_name = f"sensor_data_{current_date.replace('-', '')}.csv"
        df.to_csv(file_name, index=False, encoding="utf-8")
        print(f"Final data saved to {file_name}")

    # Close the serial port
    arduino.close()
```

---

### **Project Overview**

This script performs the following tasks:

1. **Set up Arduino serial communication**: Connects to the serial port to read data.  
2. **Filter and parse data**: Processes sensor data (soil moisture, light intensity, temperature, and humidity) from Arduino.  
3. **Real-time data storage**: Saves collected data to a CSV file in real-time.  
4. **Safe data preservation on exit**: Ensures all collected data is saved when the program is stopped.

---

### **How to Use**

1. Upload and run the Arduino code (`mainarduino.md`) on your Arduino device.  
2. Run this Python script:
   ```bash
   python collect_sensor_data.py
   ```
3. Press `Ctrl+C` to stop the program at any time.  
4. Collected data will be saved in a file named **`sensor_data_YYYYMMDD.csv`**.

---

### **Sample Output (Terminal)**

```
2024-06-01 14:23:45, Soil Moisture: 45, Light Intensity: 78, Humidity: 60, Temperature: 24
Data saved to sensor_data_20240601.csv
2024-06-01 14:23:47, Soil Moisture: 43, Light Intensity: 80, Humidity: 59, Temperature: 23
Data saved to sensor_data_20240601.csv
```

---

### **Sample Output (CSV File)**

| Timestamp           | Soil Moisture (%) | Light Intensity (%) | Humidity (%) | Temperature (C) |
|---------------------|-------------------|---------------------|-------------|-----------------|
| 2024-06-01 14:23:45 | 45                | 78                  | 60          | 24              |
| 2024-06-01 14:23:47 | 43                | 80                  | 59          | 23              |

---

### **Important Notes**

1. **Serial Port Settings**: Modify `/dev/ttyUSB0` (Linux/Mac) or `COM3` (Windows) according to your system's serial port.  
2. **Baud Rate**: Ensure the baud rate (`9600`) matches between the Arduino code and the Python script.  
3. **Data Preservation on Exit**: When you press `Ctrl+C`, all collected data will be saved to a CSV file.

---
