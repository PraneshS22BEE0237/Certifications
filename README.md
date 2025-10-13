# Functional Block Diagrams - Dual IR Sensor Parking System

## 1. HARDWARE FUNCTIONAL BLOCK DIAGRAM

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            PARKING SLOT HARDWARE ARCHITECTURE                   │
└─────────────────────────────────────────────────────────────────────────────────┘

                           ┌─────────────────────┐
                           │    POWER SUPPLY     │
                           │    5V/3.3V DC       │
                           └──────────┬──────────┘
                                      │
                ┌─────────────────────┼─────────────────────┐
                │                     │                     │
                ▼                     ▼                     ▼
    ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐
    │   MICROCONTROLLER   │ │    IR SENSOR 1      │ │    IR SENSOR 2      │
    │                     │ │   (Entry Side)      │ │   (Exit Side)       │
    │  ┌───────────────┐  │ │                     │ │                     │
    │  │   ESP32/ESP8266│  │ │  ┌─────────────┐   │ │  ┌─────────────┐   │
    │  │   or Arduino   │  │ │  │   IR LED    │   │ │  │   IR LED    │   │
    │  │                │  │ │  │ Transmitter │   │ │  │ Transmitter │   │
    │  │  Digital Pins: │  │ │  └─────────────┘   │ │  └─────────────┘   │
    │  │  Pin 2 ────────┼──┼──┤                   │ │                     │
    │  │  Pin 3 ────────┼──┼──┼─────────────────────┤  │                   │
    │  │  Pin 13────────┼──┤  │  ┌─────────────┐   │ │  ┌─────────────┐   │
    │  │                │  │  │  │   IR Photo  │   │ │  │   IR Photo  │   │
    │  │  WiFi Module   │  │  │  │   Detector  │   │ │  │   Detector  │   │
    │  │  (ESP32/ESP8266│  │  │  └─────────────┘   │ │  └─────────────┘   │
    │  │   built-in)    │  │  │                     │ │                     │
    │  └───────────────┘  │  │   Digital Output    │ │   Digital Output    │
    └─────────────────────┘  │   LOW = Object      │ │   LOW = Object      │
                │            │   HIGH = No Object  │ │   HIGH = No Object  │
                │            └─────────────────────┘ └─────────────────────┘
                ▼                         │                         │
    ┌─────────────────────┐              │                         │
    │     STATUS LED      │              │                         │
    │   (Pin 13/Built-in) │              │                         │
    │                     │              │                         │
    │  ● ON = Both Active │              │                         │
    │  ● BLINK = Not Both │              │                         │
    └─────────────────────┘              │                         │
                │                         │                         │
                ▼                         ▼                         ▼
    ┌─────────────────────────────────────────────────────────────────────┐
    │                        DETECTION ZONE                              │
    │                                                                     │
    │  IR1 Beam ●━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━● IR1 Detector │
    │            ▲                                                       │
    │            │              VEHICLE                                  │
    │            │         ┌─────────────────┐                          │
    │            │         │                 │                          │
    │  IR2 Beam ●━━━━━━━━━━│     🚗 CAR      │━━━━━━━━━━━● IR2 Detector │
    │                      │                 │                          │
    │                      └─────────────────┘                          │
    │                                                                     │
    │  Logic: Vehicle Detected = (IR1 == ACTIVE) AND (IR2 == ACTIVE)    │
    └─────────────────────────────────────────────────────────────────────┘
```

### Hardware Components Breakdown:

**Power Distribution:**
- Main Supply: 5V DC (Arduino) / 3.3V DC (ESP32)
- Current: ~100mA per IR sensor + 200mA for microcontroller
- Total: ~500mA for dual sensor setup

**IR Sensor Specifications:**
- Type: Obstacle Avoidance IR Sensors
- Detection Range: 2-30cm (adjustable)
- Output: Digital (HIGH/LOW)
- Response Time: <10ms

**Microcontroller I/O:**
- Digital Input Pins: 2 (for IR sensors)
- Digital Output Pins: 1 (for status LED)
- Communication: WiFi (ESP32/ESP8266) or Serial (Arduino UNO).

---

## 2. SOFTWARE FUNCTIONAL BLOCK DIAGRAM

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            SOFTWARE ARCHITECTURE                                │
└─────────────────────────────────────────────────────────────────────────────────┘

                               ┌─────────────────────┐
                               │    MAIN LOOP        │
                               │   (loop() function) │
                               └──────────┬──────────┘
                                          │
                    ┌─────────────────────┼─────────────────────┐
                    │                     │                     │
                    ▼                     ▼                     ▼
        ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐
        │   SENSOR READING    │ │   WEB SERVER        │ │   STATUS UPDATE     │
        │     MODULE          │ │   MODULE            │ │     MODULE          │
        │                     │ │   (ESP32/ESP8266)   │ │                     │
        └──────────┬──────────┘ └──────────┬──────────┘ └──────────┬──────────┘
                   │                       │                       │
                   ▼                       ▼                       ▼
        ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐
        │  checkParkingSlot() │ │   handleClient()    │ │   updateLEDs()      │
        │                     │ │                     │ │                     │
        │  ┌───────────────┐  │ │  ┌───────────────┐  │ │  ┌───────────────┐  │
        │  │Read IR Sensor1│  │ │  │   /sensor     │  │ │  │  LED Control  │  │
        │  │digitalRead(2) │  │ │  │   endpoint    │  │ │  │digitalWrite() │  │
        │  └───────────────┘  │ │  └───────────────┘  │ │  └───────────────┘  │
        │          │          │ │          │          │ │          │          │
        │          ▼          │ │          ▼          │ │          ▼          │
        │  ┌───────────────┐  │ │  ┌───────────────┐  │ │  ┌───────────────┐  │
        │  │Read IR Sensor2│  │ │  │   JSON        │  │ │  │ Serial Debug  │  │
        │  │digitalRead(3) │  │ │  │   Response    │  │ │  │   Output      │  │
        │  └───────────────┘  │ │  └───────────────┘  │ │  └───────────────┘  │
        │          │          │ │                     │ │                     │
        │          ▼          │ └─────────────────────┘ └─────────────────────┘
        │  ┌───────────────┐  │
        │  │   AND Logic   │  │
        │  │ S1 && S2 =    │  │
        │  │ carPresent    │  │
        │  └───────────────┘  │
        │          │          │
        │          ▼          │
        │  ┌───────────────┐  │
        │  │  Stability    │  │
        │  │   Check       │  │
        │  │(5 consistent  │  │
        │  │  readings)    │  │
        │  └───────────────┘  │
        └─────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                           DATA FLOW DIAGRAM                                    │
└─────────────────────────────────────────────────────────────────────────────────┘

Input Sensors → Processing Logic → Output Actions → External Communication

┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ IR Sensor 1 │    │             │    │             │    │             │
│   (Pin 2)   │───▶│   AND       │    │  Stability  │    │    LED      │
│ HIGH/LOW    │    │   Logic     │───▶│   Filter    │───▶│  Control    │
└─────────────┘    │             │    │             │    │             │
                   │ S1 && S2    │    │ 5 readings  │    │digitalWrit  │
┌─────────────┐    │     =       │    │ threshold   │    │e(LED_PIN)   │
│ IR Sensor 2 │    │ carPresent  │    │             │    │             │
│   (Pin 3)   │───▶│             │    └─────────────┘    └─────────────┘
│ HIGH/LOW    │    │             │           │                   │
└─────────────┘    └─────────────┘           │                   │
                                             ▼                   ▼
                                   ┌─────────────┐    ┌─────────────┐
                                   │   JSON      │    │   Serial    │
                                   │  Response   │    │   Debug     │
                                   │             │    │   Output    │
                                   │ {"car_      │    │             │
                                   │ present":   │    │ "VEHICLE    │
                                   │ true/false} │    │ DETECTED"   │
                                   └─────────────┘    └─────────────┘
                                           │                   │
                                           ▼                   ▼
                                   ┌─────────────┐    ┌─────────────┐
                                   │   WiFi      │    │  Hardware   │
                                   │Transmission │    │  Feedback   │
                                   │to HTML App  │    │             │
                                   └─────────────┘    └─────────────┘
```

### Software Module Descriptions:

**1. Sensor Reading Module:**
- Function: `checkParkingSlot()`
- Frequency: Every 200ms
- Logic: Dual IR sensor AND operation
- Stability: 5 consistent readings before state change

**2. Web Server Module (ESP32/ESP8266):**
- Endpoints: `/sensor`, `/`, `/cors-proxy`
- Protocol: HTTP REST API
- Format: JSON responses
- CORS: Enabled for cross-origin requests

**3. Communication Module:**
- WiFi: ESP32/ESP8266 web server
- Serial: Arduino UNO USB communication
- Baud Rate: 115200 for ESP32, 9600 for UNO

---

## 3. OVERALL SYSTEM FUNCTIONAL BLOCK DIAGRAM

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         COMPLETE SYSTEM ARCHITECTURE                           │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                              PHYSICAL LAYER                                    │
└─────────────────────────────────────────────────────────────────────────────────┘

                    PARKING SLOT PHYSICAL INSTALLATION

            Entry Side                                    Exit Side
                │                                           │
                ▼                                           ▼
        ┌─────────────────┐                        ┌─────────────────┐
        │   IR SENSOR 1   │                        │   IR SENSOR 2   │
        │                 │                        │                 │
        │ ●─────────────────────────────────────────────────────────● │
        │    IR Beam 1                                   IR Beam 2    │
        └─────────────────┘                        └─────────────────┘
                │                                           │
                │              VEHICLE ENTRY                │
                │         ┌─────────────────────┐          │
                │         │                     │          │
                └─────────┤      🚗 VEHICLE     ├─────────┘
                          │                     │
                          └─────────────────────┘
                                     │
                                     ▼
                            ┌─────────────────┐
                            │  BOTH BEAMS     │
                            │    BROKEN       │
                            │ = OCCUPIED SLOT │
                            └─────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                            CONTROL LAYER                                       │
└─────────────────────────────────────────────────────────────────────────────────┘

        ┌─────────────────────────────────────────────────────────────────┐
        │                    ARDUINO/ESP32 CONTROLLER                    │
        │                                                                 │
        │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
        │  │   INPUT     │  │  PROCESSING │  │   OUTPUT    │            │
        │  │   LAYER     │  │    LAYER    │  │    LAYER    │            │
        │  │             │  │             │  │             │            │
        │  │ IR Sensor 1 │  │  AND Logic  │  │ Status LED  │            │
        │  │   Pin 2     │  │             │  │   Pin 13    │            │
        │  │             │  │ S1 && S2 =  │  │             │            │
        │  │ IR Sensor 2 │  │ carPresent  │  │ Serial Out  │            │
        │  │   Pin 3     │  │             │  │             │            │
        │  │             │  │ Stability   │  │ WiFi/HTTP   │            │
        │  │             │  │ Filtering   │  │ Server      │            │
        │  └─────────────┘  └─────────────┘  └─────────────┘            │
        └─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                          COMMUNICATION LAYER                                   │
└─────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────┐           ┌─────────────────┐           ┌─────────────────┐
    │   ARDUINO       │   WiFi/   │    NETWORK      │    HTTP   │   WEB CLIENT    │
    │   CONTROLLER    │   HTTP    │    ROUTER       │   Request │   (HTML APP)    │
    │                 │◄─────────►│                 │◄─────────►│                 │
    │ WiFi Enabled    │           │  192.168.1.x    │           │  Browser/App    │
    │ (ESP32/ESP8266) │           │                 │           │                 │
    └─────────────────┘           └─────────────────┘           └─────────────────┘
            │                                                           │
            │ JSON API Response                                         │
            │ {"car_present": true,                                    │
            │  "sensor1_reading": true,                                │
            │  "sensor2_reading": true,                                │
            │  "both_sensors_active": true}                            │
            └──────────────────────────────────────────────────────────┘

    Alternative: USB Serial Communication (Arduino UNO)
    ┌─────────────────┐   USB     ┌─────────────────┐   Bridge   ┌─────────────────┐
    │   ARDUINO UNO   │  Serial   │    COMPUTER     │  Software  │   WEB CLIENT    │
    │   CONTROLLER    │◄─────────►│    (Python/     │◄──────────►│   (HTML APP)    │
    │                 │           │     Node.js)    │            │                 │
    │ USB Connection  │           │  Serial Bridge  │            │  Browser/App    │
    └─────────────────┘           └─────────────────┘            └─────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                           APPLICATION LAYER                                    │
└─────────────────────────────────────────────────────────────────────────────────┘

                              WEB APPLICATION STACK

    ┌─────────────────────────────────────────────────────────────────────────┐
    │                         FRONTEND (HTML APP)                            │
    │                                                                         │
    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
    │  │    HTML     │  │     CSS     │  │ JavaScript  │  │   Parking   │    │
    │  │   Layout    │  │   Styling   │  │   Logic     │  │   Slots UI  │    │
    │  │             │  │             │  │             │  │             │    │
    │  │ - Slot Grid │  │ - Colors    │  │ - API Calls │  │ - Status    │    │
    │  │ - Controls  │  │ - Animation │  │ - Real-time │  │ - Timer     │    │
    │  │ - Status    │  │ - Layout    │  │   Updates   │  │ - Booking   │    │
    │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
    └─────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼ HTTP Requests
    ┌─────────────────────────────────────────────────────────────────────────┐
    │                         BACKEND (Node.js)                              │
    │                                                                         │
    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
    │  │   Express   │  │   Database  │  │    API      │  │   Arduino   │    │
    │  │   Server    │  │   (SQLite)  │  │  Endpoints  │  │  Interface  │    │
    │  │             │  │             │  │             │  │             │    │
    │  │ - Routing   │  │ - Slots     │  │ - /sensors  │  │ - WiFi      │    │
    │  │ - Middleware│  │ - Users     │  │ - /booking  │  │   Polling   │    │
    │  │ - CORS      │  │ - History   │  │ - /status   │  │ - Data      │    │
    │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
    └─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                            SYSTEM DATA FLOW                                    │
└─────────────────────────────────────────────────────────────────────────────────┘

1. SENSOR INPUT PHASE:
   IR Sensor 1 (Pin 2) ──┐
                          ├─► AND Logic ──► Stability Filter ──► State Change
   IR Sensor 2 (Pin 3) ──┘

2. PROCESSING PHASE:
   State Change ──► LED Update ──► JSON Creation ──► WiFi Transmission

3. COMMUNICATION PHASE:
   Arduino ──► WiFi/HTTP ──► Web Server ──► Database ──► HTML App

4. USER INTERFACE PHASE:
   HTML App ──► Visual Update ──► User Interaction ──► Booking/Timer

5. FEEDBACK PHASE:
   Booking Status ──► Database ──► API Response ──► Arduino Notification
```

## Key System Features:

### **Hardware Features:**
- **Dual Sensor Redundancy**: Eliminates false positive
- **Low Power Design**: <500mA total consumption
- **Modular Architecture**: Easy to scale for multiple slots
- **Weather Resistant**: Suitable for outdoor parking areas

### **Software Features:**
- **Real-time Processing**: 200ms sensor polling
- **Stability Filtering**: 5-reading confirmation
- **Web API Integration**: RESTful JSON endpoints
- **Cross-platform Support**: ESP32/ESP8266 and Arduino UNO

### **System Features:**
- **High Accuracy**: Dual sensor AND logic
- **Scalable Design**: Support for multiple parking areas.
- **Web Integration**: Direct HTML app communication
- **Remote Monitoring**: WiFi-enabled real-time updates

This comprehensive block diagram system provides a complete overview of how the dual IR sensor parking system operates from the physical hardware level through to the user interface application.
