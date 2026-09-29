# Smart Plant Monitor — Complete Build Guide

A smart plant monitoring and control system using **ESP8266, sensors, a water pump, servo motor, Node.js, MongoDB, and a React frontend**.

---

## STEP 1: Understand Your Components

| Component | What it does |
|---|---|
| **ESP8266 NodeMCU** | The brain — runs WiFi, reads sensors, and controls actuators |
| **Capacitive Soil Sensor** | Measures soil wetness (0–100%) |
| **DHT22** | Measures air temperature and humidity |
| **IRLZ44N MOSFET** | Acts as a switch — ESP controls the water pump |
| **1N4007 Diode** | Protects the circuit from voltage spikes when the pump stops |
| **1kΩ Resistor** | Limits current into the MOSFET gate |
| **10kΩ Resistor** | Pulls MOSFET gate to GND when not active |
| **SG90 Servo** | Opens/closes the shade (0° = closed, 90° = open) |
| **Water Pump** | 3V–6V submersible mini pump |
| **Breadboard** | Used to build the circuit without soldering |

---

## STEP 2: Wire the Soil Moisture Sensor

The capacitive soil sensor has 3 pins:

- VCC
- GND
- AOUT

### Wiring

```text
Sensor VCC  → Breadboard +rail → ESP 3V3 pin
Sensor GND  → Breadboard -rail → ESP GND pin
Sensor AOUT → ESP A0 pin
```

> **WHY A0?** ESP8266 has only one analog input, called **A0**. The sensor sends an analog voltage that changes according to soil moisture.

---

## STEP 3: Wire the DHT22 Temperature & Humidity Sensor

The DHT22 has 3 active pins when facing the grid side:

- Pin 1 = VCC
- Pin 2 = DATA
- Pin 4 = GND
- Pin 3 = Not connected

### Wiring

```text
DHT22 Pin 1 (VCC)  → ESP 3V3
DHT22 Pin 2 (DATA) → ESP D1
DHT22 Pin 4 (GND)  → ESP GND
```

> **IMPORTANT:** Use the required pull-up resistor between DATA and VCC according to your DHT22 module/sensor configuration.

---

## STEP 4: Wire the MOSFET + Water Pump

This is one of the most important parts of the circuit.

For the **IRLZ44N**, when viewed from the front/flat side, the typical pin order is:

```text
Left   = GATE (G)
Middle = DRAIN (D)
Right  = SOURCE (S)
```

### Wiring

```text
ESP D5 → 1kΩ resistor → MOSFET GATE
MOSFET GATE → 10kΩ resistor → GND
MOSFET SOURCE → GND

5V power → Pump (+)
Pump (-) → MOSFET DRAIN

1N4007 Diode → across the pump
Stripe end → Pump (+)
Other end   → Pump (-)
```

> **WHY THE DIODE?** A motor/pump can create a voltage spike when switched off. A flyback diode provides a path for that energy and helps protect the switching circuit.

> **WHY THE 10kΩ RESISTOR?** It keeps the MOSFET gate at a defined LOW state when the ESP8266 is not actively driving it.

---

## STEP 5: Wire the SG90 Servo Motor

The servo has three wires:

```text
Red wire    → 5V power
Brown wire  → GND
Orange wire → ESP D6 (signal)
```

> **WARNING:** The servo can require more current than an ESP8266 power pin should provide. Use an appropriate 5V supply and connect its ground to the ESP8266 ground.

---

## STEP 6: Power Everything

A common-ground arrangement is important when using separate power sources.

```text
5V Power Supply (+) → Pump (+)
5V Power Supply (+) → Servo red wire
5V Power Supply (-) → Common GND

ESP GND             → Common GND
Servo GND           → Common GND
MOSFET Source       → Common GND
```

> **KEY CONCEPT:** All parts that communicate with the ESP8266 need a common ground reference.

---

## STEP 7: Flash the ESP8266

### 1. Install Arduino IDE

Download and install **Arduino IDE**.

### 2. Add ESP8266 Board Support

Open:

```text
Arduino IDE
→ File
→ Preferences
```

In **Additional Board Manager URLs**, add:

```text
http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

### 3. Install the ESP8266 Board

Go to:

```text
Tools
→ Board
→ Board Manager
```

Search for:

```text
esp8266
```

Then install the ESP8266 package.

### 4. Install Required Libraries

Go to:

```text
Tools
→ Manage Libraries
```

Install:

- **DHT sensor library** by Adafruit
- **ArduinoJson** by Benoit Blanchon
- **ESP8266WebServer** — built into the ESP8266 core

### 5. Open the ESP8266 Code

Open:

```text
esp8266/plant_monitor.ino
```

Edit the required configuration values such as:

```text
WiFi name
WiFi password
Backend/server IP address
```

### 6. Select the Board

Go to:

```text
Tools
→ Board
→ ESP8266 Boards
→ NodeMCU 1.0 (ESP-12E)
```

### 7. Select the Port

Go to:

```text
Tools
→ Port
```

Select the COM port connected to your ESP8266.

### 8. Upload

Click the **Upload** button.

### 9. Open Serial Monitor

Open the Serial Monitor and use:

```text
115200 baud
```

You can then view the ESP8266 debug messages.

---

## STEP 8: Run the Backend

### Prerequisites

Make sure you have:

- Node.js
- MongoDB

installed and configured.

### Open the Backend

From the project root:

```powershell
cd backend
npm install
npm run dev
```

The backend runs on:

```text
http://localhost:5000
```

> **IMPORTANT:** Keep your real `.env` file private. Never commit database passwords, API keys, or other secrets to GitHub.

---

# STEP 9: API Reference for the Frontend

## Sensor Data

| Method | URL | Purpose |
|---|---|---|
| `GET` | `/api/sensor-data/latest` | Current raw reading |
| `GET` | `/api/sensor-data/history?hours=24` | Past readings for charts |

---

## Plant Intelligence

| Method | URL | Frontend use |
|---|---|---|
| `GET` | `/api/plant/summary` | Main dashboard — everything |
| `GET` | `/api/plant/health` | Health score gauge |
| `GET` | `/api/plant/conditions` | Per-sensor status cards |
| `GET` | `/api/plant/classify` | Plant category match grid |
| `GET` | `/api/plant/alerts` | Notification panel |
| `GET` | `/api/plant/recommendations` | Actuator advice |
| `GET` | `/api/plant/trends?hours=24` | Chart data |
| `GET` | `/api/plant/types` | All plant type definitions |
| `GET` | `/api/plant/ideal-ranges/:type` | Ranges for one plant type |

---

## Control Commands

| Method | URL | Body | Purpose |
|---|---|---|---|
| `POST` | `/api/commands` | `{ "command": "pump_on" }` | Turn pump on |
| `POST` | `/api/commands` | `{ "command": "pump_off" }` | Turn pump off |
| `POST` | `/api/commands` | `{ "command": "servo", "value": 90 }` | Move servo |
| `POST` | `/api/commands` | `{ "command": "manual_on" }` | Switch to manual mode |
| `POST` | `/api/commands` | `{ "command": "manual_off" }` | Switch to auto mode |

---

## Socket.IO Events

| Event | Direction | Data |
|---|---|---|
| `sensor_update` | Server → Frontend | Latest SensorReading |
| `command_queued` | Server → Frontend | Command that was queued |
| `send_command` | Frontend → Server | `{ command, value }` |

---

# STEP 10: Frontend Function Map

The frontend should implement functions that communicate with the backend API.

## Dashboard & Plant Data

```javascript
fetchDashboardSummary()     → GET /api/plant/summary
fetchHealthScore()          → GET /api/plant/health
fetchConditions()           → GET /api/plant/conditions
fetchPlantClassification()  → GET /api/plant/classify
fetchAlerts()               → GET /api/plant/alerts
fetchRecommendations()      → GET /api/plant/recommendations
fetchTrends(hours)          → GET /api/plant/trends?hours={hours}
fetchPlantTypes()           → GET /api/plant/types
```

## Actuator Controls

```javascript
sendPumpOn()                → POST /api/commands { command: "pump_on" }

sendPumpOff()               → POST /api/commands { command: "pump_off" }

sendServoAngle(angle)       → POST /api/commands {
                                command: "servo",
                                value: angle
                              }

enableManualMode()          → POST /api/commands {
                                command: "manual_on"
                              }

enableAutoMode()            → POST /api/commands {
                                command: "manual_off"
                              }
```

## Real-Time Data

```javascript
subscribeToLiveData(cb)     → Socket.IO listen 'sensor_update'

subscribeToCommands(cb)     → Socket.IO listen 'command_queued'
```

---

# Project File Structure

```text
HACKZION.V3__Event-Horizon/
│
├── esp8266/
│   └── plant_monitor.ino
│
├── backend/
│   ├── server.js
│   ├── package.json
│   ├── .gitignore
│   │
│   ├── models/
│   │   ├── SensorReading.js
│   │   └── Command.js
│   │
│   ├── routes/
│   │   ├── sensor.js
│   │   ├── commands.js
│   │   └── plant.js
│   │
│   └── utils/
│       └── plantAnalysis.js
│
├── frontend/
│   └── frontend/
│       ├── public/
│       ├── src/
│       │   ├── assets/
│       │   ├── App.css
│       │   ├── App.jsx
│       │   ├── Background3D.jsx
│       │   ├── index.css
│       │   └── main.jsx
│       ├── package.json
│       └── vite.config.js
│
├── .gitignore
├── package-lock.json
└── README.md
```

---

# How to Run

## 1. Start the Backend

Open one terminal:

```powershell
cd backend
npm install
npm run dev
```

---

## 2. Start the Frontend

Open another terminal:

```powershell
cd frontend\frontend
npm install
npm run dev
```

---

## 3. Open the Application

After Vite starts, open the local URL shown in the terminal, usually similar to:

```text
http://localhost:5173
```

---

# Smart Plant Monitor 🌱

A complete IoT-based plant monitoring system that combines:

- 🌱 Soil moisture monitoring
- 🌡️ Temperature monitoring
- 💧 Humidity monitoring
- 💦 Automatic water pump control
- 🌤️ Servo-controlled shade
- 📊 Plant health analysis
- 🚨 Plant alerts
- 💡 Plant recommendations
- 📡 ESP8266 connectivity
- 🗄️ MongoDB data storage
- ⚡ Real-time Socket.IO communication
- 🖥️ React dashboard

---

## Built With

```text
ESP8266
Arduino
Node.js
Express.js
MongoDB
Socket.IO
React
Vite
```

---

## Project Goal

The goal of the Smart Plant Monitor is to create an automated plant-care system that can monitor environmental conditions, analyze plant health, and control connected devices based on the collected data.

🌱 **Monitor → Analyze → Decide → Act**
