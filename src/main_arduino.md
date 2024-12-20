---

## Main_arduino Code Explanation

This code collects environmental data using a **soil moisture sensor**, **light intensity sensor**, and **DHT11 temperature and humidity sensor** and outputs the data to the serial monitor.  
It also includes averaging processes to ensure stable sensor readings and reduce noise.

---

### Sensors and Pin Definitions

1. **Soil Moisture Sensor**  
   - **Pin Connection**: Analog pin `A1`  
   - **Function**: Reads soil moisture values and converts them to percentages (0–100%).  
   - **Calibration Values**:  
      - Dry condition: `520`  
      - Wet condition: `260`  

2. **Light Intensity Sensor**  
   - **Pin Connection**: Analog pin `A0`  
   - **Function**: Measures light intensity, with noise reduction via sampling averages and removal of outlier values.  

3. **DHT11 Temperature and Humidity Sensor**  
   - **Pin Connection**: Digital pin `D2`  
   - **Function**: Reads temperature and humidity values and outputs them.  
   - **Library**: `SimpleDHT.h`  

---

### Library Installation

The **`SimpleDHT.h` library** is required to read data from the DHT11 sensor.  
Follow these steps to install the library in Arduino IDE:

1. Open Arduino IDE and select **Sketch -> Include Library -> Manage Libraries**.  
2. In the search bar, type **"SimpleDHT"** and install the library.  

> **Note**: The SimpleDHT library is compatible with the DHT11 sensor.

---

### Key Features

1. **Soil Moisture Data Collection**  
   - Averaging sensor readings (10 measurements and calculating the average).  
   - Percentage conversion with calibration adjustments.  

2. **Light Intensity Data Collection**  
   - Collecting 10 readings, removing extreme values, and calculating a stable average.  

3. **Temperature and Humidity Data Collection**  
   - Reads and outputs temperature and humidity values using the DHT11 sensor.  

4. **Serial Output**  
   - Outputs all sensor values to the serial monitor every 2 seconds.  

---

### Hardware Connections

| Sensor             | VCC    | GND    | Data Pin       |
|--------------------|--------|--------|----------------|
| Soil Moisture Sensor   | VCC    | GND    | A1 (Analog)       |
| Light Intensity Sensor | VCC    | GND    | A0 (Analog)       |
| DHT11 Sensor           | VCC    | GND    | D2 (Digital)      |

---

### Arduino Sensor Measurement Code

```cpp
#include <SimpleDHT.h>

// Pin definitions
const int soilMoisturePin = A1; // Soil moisture sensor connected to analog pin
const int lightSensorPin = A0;  // Light sensor connected to analog pin
const int dhtPin = 2;           // DHT11 connected to digital pin

// Constants for soil moisture sensor
const int dryValue = 520;  // Calibrated dry soil value
const int wetValue = 260;  // Calibrated wet soil value

// DHT11 sensor initialization
SimpleDHT11 dht11;

// Function to get average sensor value for soil moisture
int getAverageSensorValue(int pin, int samples) {
  long total = 0;
  for (int i = 0; i < samples; i++) {
    total += analogRead(pin);
    delay(10); // Small delay between readings
  }
  return total / samples;
}

// Function to get stable and averaged light sensor reading
int getStableLightReading() {
  const int NUM_SAMPLES = 10;
  int samples[NUM_SAMPLES];

  // Collect samples
  for (int i = 0; i < NUM_SAMPLES; i++) {
    samples[i] = analogRead(lightSensorPin);
    delay(10);
  }

  // Sort samples
  for (int i = 0; i < NUM_SAMPLES - 1; i++) {
    for (int j = i + 1; j < NUM_SAMPLES; j++) {
      if (samples[i] > samples[j]) {
        int temp = samples[i];
        samples[i] = samples[j];
        samples[j] = temp;
      }
    }
  }

  // Remove extreme values and calculate the average
  long sum = 0;
  for (int i = 2; i < NUM_SAMPLES - 2; i++) {
    sum += samples[i];
  }
  return sum / (NUM_SAMPLES - 4);
}

void setup() {
  Serial.begin(9600);
  Serial.println("Starting Combined Sensor Test!");
}

void loop() {
  // 1. Soil Moisture Sensor
  int soilValue = getAverageSensorValue(soilMoisturePin, 10);
  int soilMoisturePercent = map(soilValue, dryValue, wetValue, 0, 100);
  soilMoisturePercent = constrain(soilMoisturePercent, 0, 100);

  // 2. Light Sensor
  int lightValue = getStableLightReading();
  int lightPercent = map(lightValue, 0, 1023, 0, 100);

  // 3. DHT11 Sensor
  byte temperature = 0;
  byte humidity = 0;
  int dhtErr = dht11.read(dhtPin, &temperature, &humidity, NULL);

  // Output all sensor data in one line
  Serial.print("Soil Moisture:");
  Serial.print(soilMoisturePercent);
  Serial.print(", Light Intensity:");
  Serial.print(lightPercent);
  Serial.print(", ");

  if (dhtErr == SimpleDHTErrSuccess) {
    Serial.print("Humidity:");
    Serial.print(humidity);
    Serial.print(", Temperature:");
    Serial.println(temperature);
  } else {
    Serial.println("Humidity:Error, Temperature:Error");
  }

  delay(2000); // Wait for 2 seconds before the next reading
}
```

---

### Execution Example

```
Soil Moisture: 45, Light Intensity: 78, Humidity: 60, Temperature: 24
Soil Moisture: 43, Light Intensity: 80, Humidity: 59, Temperature: 23
...
``` 
