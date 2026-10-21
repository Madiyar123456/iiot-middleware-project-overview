# IIoT Middleware with Service Discovery, Fault Isolation and Verified Health Monitoring

## Overview

This project implements an Industrial Internet of Things (IIoT) middleware layer positioned between MQTT sensor nodes and a real-time web dashboard.

Unlike a simple MQTT-to-dashboard pipeline, the middleware acts as an active software layer responsible for service discovery, service registration, message normalization, fault isolation, health monitoring, and REST-based data publishing.

The system is designed as a runtime model of the sensor network, where the middleware maintains knowledge about connected devices, their types, and their operational state.

---

## Architecture

Sensor Nodes (MQTT)

* Temperature Sensor
* Humidity Sensor
* Pressure Sensor

↓

MQTT Broker (Mosquitto)

↓

IIoT Middleware

* Registry Actor
* Normalizer Actor
* Health Monitor Actor
* REST API Publisher

↓

Web Dashboard

---

## Main Features

### Service Discovery

Sensor nodes automatically become visible to the middleware when they start publishing MQTT messages.

### Service Registry

The middleware maintains a runtime registry of active devices, including:

* Sensor ID
* Sensor Type
* Current Status
* Last Seen Timestamp

### Message Normalization

All incoming sensor messages are transformed into a unified internal format before being exposed through the REST API.

Example normalized structure:

```json
{
  "sensor_id": "temp_1",
  "sensor_type": "temperature",
  "reading": 25.4,
  "unit": "C",
  "timestamp": 1780338178
}
```

### Health Monitoring

The middleware continuously monitors sensor activity.

If a sensor stops sending data for more than the configured timeout period, it is marked as:

```text
unavailable
```

When the sensor starts publishing again, it automatically returns to:

```text
active
```

### Data Sanitization

The middleware validates incoming sensor readings before exposing them through the REST API.

Accepted ranges:

* Temperature: -50°C to 100°C
* Humidity: 0% to 100%
* Pressure: 800 hPa to 1200 hPa

Out-of-range values are automatically isolated and discarded by the normalizer.

Example fault injection:

```text
Temperature = 500°C
```

The middleware detects the invalid value, isolates the message, and prevents propagation to the dashboard.

The normalizer actor continues processing subsequent messages after isolating invalid data. The dashboard remains operational and continues displaying the last valid reading received from the sensor.

### Fault Isolation

Unavailable sensors are isolated from dashboard output and API responses, preventing stale or invalid data from being displayed.

### REST API

The middleware exposes the following endpoints:

```text
GET /sensors
GET /readings
GET /health
```

### Real-Time Dashboard

A web dashboard retrieves data from the REST API and displays:

* Active sensors
* Sensor status information
* Last valid sensor readings
* System Integrity metric

### System Integrity Metric

The dashboard provides a real-time System Integrity metric.

The metric is calculated as:

```text
(active sensors / registered sensors) × 100
```

Examples:

* 3 active / 3 registered = 100%
* 2 active / 3 registered = 66.67%

This visual health check allows operators to immediately identify sensor failures and monitor the overall health of the system.

---

## Asyncio Actor-Based Middleware

The middleware is implemented using independent asyncio actors communicating through asyncio queues.

### Registry Actor

Responsible for:

* Service discovery
* Service registration
* Status updates

### Normalizer Actor

Responsible for:

* Message normalization
* Internal data transformation
* Data sanitization
* Fault isolation

### Health Monitor Actor

Responsible for:

* Timeout detection
* Failure detection
* Recovery tracking

Communication between actors is performed using:

```python
asyncio.Queue
```

---

## Sensor Simulators

Three independent MQTT sensor simulators were implemented using Paho MQTT:

### Temperature Sensor

```text
temperature_sensor.py
```

### Humidity Sensor

```text
humidity_sensor.py
```

### Pressure Sensor

```text
pressure_sensor.py
```

---

## UPPAAL Verification

The health monitoring component was formally modelled in UPPAAL.

The final version of the model includes a heartbeat communication channel and a dedicated Sensor automaton. Heartbeat events represent message arrivals from sensor nodes and allow the Health Monitor to react to sensor activity in a way that more closely reflects the implemented middleware.

### States

* Monitoring
* MissedOnce
* MissedThreshold
* Unavailable
* Recovering

### Verified Properties

#### Unavailable State Reachability

```text
E<> Process.Unavailable
```

Result:

```text
True
```

The model can reach the Unavailable state when heartbeat messages stop arriving.

#### Recovery Reachability

```text
E<> Process.Recovering
```

Result:

```text
True
```

The system can recover after heartbeat communication resumes.

#### Monitoring Reachability

```text
E<> Process.Monitoring
```

Result:

```text
True
```

The system can return to normal monitoring operation after recovery.

---

## Technologies

* Python 3
* Python asyncio
* Paho MQTT
* Mosquitto Broker
* FastAPI
* HTML
* JavaScript
* UPPAAL

---

## Project Structure

```text
IIoT Project/
│
├── middleware.py
├── temperature_sensor.py
├── humidity_sensor.py
├── pressure_sensor.py
├── dashboard.html
├── README.md
│
└── uppaal/
    └── health_monitor.xml
```

---

## Author

Madiyar Yermagan

University of Messina

Industrial Internet of Things Project
