# 🌱 **Project Name: Smart Environmental Data Monitoring System**

## 📖 **Project Overview**
This project leverages **Jetson Nano** and **Arduino sensors** to monitor and manage environmental data.  
It collects real-time data on soil moisture, temperature, humidity, and light intensity, and alerts users of water shortages through **Discord notifications**. Additionally, it stores sensor data and provides easy access to users through a **Gradio UI-based chatbot**.

## 🚀 **Key Features**
- **Data Collection and Storage**:
  - **Soil Moisture Sensor**: Measures soil moisture levels and collects data in real-time.
  - **DHT11 Sensor**: Collects real-time temperature and humidity data.
  - **Light Intensity Sensor**: Collects real-time ambient light intensity data.
  - All collected data is stored in **CSV files**.

- **Automated Notification System**:
  - Sends **Discord alerts** to notify users immediately in case of water shortages.

- **Gradio UI-Based Chatbot**:
  - Provides an interactive **chatbot interface** for users to check sensor information.
  - Visualizes data through Gradio UI for enhanced accessibility.

## 🛠️ **Tech Stack**
- **Programming Languages**: Python, C++
- **Platforms/Frameworks**: Jetson Nano, Arduino, Gradio
- **Hardware**:
  - Jetson Nano
  - Arduino
  - Sensors: Soil Moisture Sensor, DHT11 (Temperature and Humidity Sensor), Light Intensity Sensor
- **Other Tools**: Discord API, CSV data storage

## 📂 **Project Structure**
```plaintext
📁 Smart Environmental Data Monitoring System/
├── README.md             # Project description
├── src/                  # Source code
│   ├── main_arduino.md   # Main Arduino sensor measurement code
│   ├── sensor_data.md    # Sensor data collection code
│   ├── discord_alert.md  # Discord alert system code
│   └── chatbot_ui.md     # Gradio UI-based chatbot code
├── failed_attempts/
│   └── gradio_graph_attempt.md  # Attempted Gradio UI graph code
├── data/                 # Folder for CSV data storage
│   └── sensor_data.csv   # Collected sensor data **sensor data 20241213.csv**
└── requirements.txt      # Dependency list
```
