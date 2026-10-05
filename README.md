# IoT Final Project — Smart Washing Machine Monitor

Final project for the Internet of Things course at Universidade Politécnica de Bragança (UPB).

## Overview

A smart water volume monitoring system for washing machines using an ESP8266 microcontroller. The device reads a potentiometer (simulating a water level sensor), publishes the data via MQTT, and supports both automatic and remote-triggered refills. A Node-RED dashboard provides real-time visualization and manual control.

## Architecture

```
ESP8266 ──MQTT──► Node-RED ──► InfluxDB
   ▲                │
   └────────────────┘
      (refill command)
```

- **ESP8266** reads the sensor, publishes volume to MQTT, and listens for refill commands
- **Node-RED** formats the data, feeds an InfluxDB batch, and serves a UI dashboard
- **InfluxDB** stores historical volume readings

## Hardware

| Component | Description |
|---|---|
| ESP8266 | Wi-Fi microcontroller (NodeMCU) |
| Potentiometer | Analog water level sensor (pin A0) |
| LED | Refill status indicator (pin D0 / GPIO 16) |

## MQTT Topics

| Topic | Direction | Description |
|---|---|---|
| `IoT/ESP8266/WashMachine` | ESP8266 → Broker | Current volume (0–100 mL) |
| `IoT/ESP8266/Response` | Broker → ESP8266 | Refill command (`true` / `false`) |

**Broker:** `broker.emqx.io:1883` (public)

## Refill Logic

- **Auto refill:** triggered automatically when volume drops to ≤ 25 mL
- **Manual refill:** triggered via MQTT command (`true`) from the Node-RED dashboard button
- Volume increases gradually in 1 mL steps every 300 ms until it reaches 100 mL
- A 5-second cooldown prevents immediate re-triggering after a completed refill

## Node-RED Dashboard

The `flows.json` file contains the full Node-RED flow with:

- **Gauge** — real-time water volume display (0–100 mL)
- **Refill button** — sends `true`/`false` to the MQTT response topic
- **InfluxDB batch node** — persists readings for historical analysis

## Setup

### ESP8266

1. Install the following libraries in Arduino IDE:
   - `ESP8266WiFi`
   - `PubSubClient`
2. Update the credentials in `FinalProject_ESP8266.ino`:
```cpp
   const char* ssid = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
```
3. Flash the sketch to your ESP8266.

### Node-RED

1. Import `flows.json` into your Node-RED instance.
2. Configure the InfluxDB node with your own host/credentials.
3. Deploy the flow.

## Repository Structure

```
├── FinalProject_ESP8266/
│   └── FinalProject_ESP8266.ino   # Arduino sketch
├── flows.json                      # Node-RED flow export
├── FinalProject-IoTReport.pdf      # Full project report
├── FinalProject-Plan.pdf           # Initial project plan
└── FinalProject-Slides.pdf         # Presentation slides
```
