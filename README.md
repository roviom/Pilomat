# Pilomat III - Setup Guide

## Hardware Requirements
- ESP32 DevKit board (ESP32-WROOM-32 or similar)
- Traffic light LEDs (Red, Yellow, Green) + resistors
- Buzzer (optional - piezo or active buzzer)
- Power supply (5V USB or 7-12V barrel jack)

## Pin Connections (Suggested)
```
Red Light    -> GPIO 25
Yellow Light -> GPIO 26
Green Light  -> GPIO 27
Buzzer       -> GPIO 14
```

## Software Requirements
1. **Arduino IDE** (1.8.19 or newer) or **PlatformIO**
2. **ESP32 Board Support**
   - In Arduino IDE: File → Preferences → Additional Board URLs:
     ```
     https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
     ```
   - Tools → Board → Boards Manager → Search "ESP32" → Install

3. **Required Libraries** (Install via Library Manager):
   - `ESPAsyncWebServer` by me-no-dev
   - `AsyncTCP` by me-no-dev
   - `ArduinoJson` by Benoit Blanchon (v6.x)

## Installation Steps

### 1. Prepare SPIFFS
Create a folder structure:
```
Pilomat3/
├── Pilomat3.ino          (the ESP32 code)
└── data/
    ├── index.html        (the web interface)
    └── config.json       (your configuration)
```

### 2. Upload SPIFFS Files
**Arduino IDE:**
- Install "ESP32 Sketch Data Upload" plugin
- Tools → ESP32 Sketch Data Upload
- Wait for upload to complete

**PlatformIO:**
- Add to `platformio.ini`:
  ```ini
  [env:esp32dev]
  platform = espressif32
  board = esp32dev
  framework = arduino
  monitor_speed = 115200
  board_build.filesystem = spiffs
  ```
- Run: `pio run -t uploadfs`

### 3. Configure WiFi
Edit the ESP32 code:
```cpp
const char* ssid = "YourWiFiName";
const char* password = "YourWiFiPassword";
```

### 4. Upload Code
- Connect ESP32 via USB
- Select Board: "ESP32 Dev Module"
- Select Port: (your COM port)
- Click Upload
- Open Serial Monitor (115200 baud)
- Note the IP address shown

### 5. Access the Interface
- On your tablet/phone, connect to same WiFi
- Open browser and go to: `http://[ESP32_IP_ADDRESS]`
- Example: `http://192.168.1.100`

### 6. Load Configuration
- Click "📁 Load Configuration JSON"
- Select your configuration file
- System will automatically start

## Configuration File Format

Place your JSON configuration in `data/config.json`:

```json
{
  "name": "Your Configuration Name",
  "displayMode": "center",
  "maxCounter": 999,
  "states": [...],
  "events": [...],
  "buttons": [...]
}
```

## Tablet Optimization

### For iPad/Android tablets (9.7" / 10.1"):
- Add to home screen for fullscreen app experience
- iOS: Safari → Share → Add to Home Screen
- Android: Chrome → Menu → Add to Home Screen

### Display keeps the tablet awake
The interface prevents auto-sleep during operation.

## Network Setup Options

### Option 1: Connect to existing WiFi (default)
```cpp
const char* ssid = "YourWiFiName";
const char* password = "YourWiFiPassword";
```

### Option 2: Create Access Point (standalone)
```cpp
WiFi.softAP("Pilomat3", "pilomat123");
IPAddress IP = WiFi.softAPIP();
```
Then connect devices directly to "Pilomat3" WiFi.

## Troubleshooting

### Can't connect to ESP32
1. Check Serial Monitor for IP address
2. Verify tablet is on same WiFi network
3. Try pinging the ESP32 IP
4. Check firewall settings

### Configuration not loading
1. Verify JSON syntax (use JSONLint.com)
2. Check SPIFFS uploaded correctly
3. Monitor Serial output for errors

### Display not syncing
1. Check polling is working (console logs)
2. Verify ESP32 responding to `/state`
3. Try reducing number of connected clients

### Timers not accurate
- ESP32 clock drift is normal (<1s per hour)
- Consider using NTP time sync for long sessions
- Restart system between competition rounds

## Advanced: Hardware Outputs

To actually control physical traffic lights and buzzer:

```cpp
// In setup()
pinMode(25, OUTPUT);  // Red
pinMode(26, OUTPUT);  // Yellow
pinMode(27, OUTPUT);  // Green
pinMode(14, OUTPUT);  // Buzzer

// In executeActions()
if (actions.containsKey("trafficLight")) {
  String light = actions["trafficLight"].as<String>();
  digitalWrite(25, light == "red" ? HIGH : LOW);
  digitalWrite(26, light == "yellow" ? HIGH : LOW);
  digitalWrite(27, light == "green" ? HIGH : LOW);
}

if (actions.containsKey("buzzer")) {
  // Trigger buzzer pattern
  // Implementation depends on your buzzer circuit
}
```

## Production Recommendations

1. **Power Supply**: Use regulated 5V 2A supply, not USB from computer
2. **Enclosure**: Weatherproof case for outdoor use
3. **Status LED**: Add indicator for WiFi/operation status
4. **Reset Button**: Wire GPIO 0 to ground for physical reset
5. **SD Card**: Add SD card reader for easy config changes
6. **Backup Power**: Consider UPS for competitions

## System Architecture

```
┌─────────────┐
│   Tablets   │──┐
│  (Display)  │  │
└─────────────┘  │
                 │  WiFi
┌─────────────┐  │  100ms polling
│  Laptops    │──┼──────────► ┌──────────────┐
│  (Display)  │  │            │    ESP32     │
└─────────────┘  │            │  (Master)    │
                 │            │              │
┌─────────────┐  │            │ - Timer      │
│   Phones    │──┘            │ - State      │
│  (Display)  │               │ - Config     │
└─────────────┘               └──────┬───────┘
                                     │
                              ┌──────▼───────┐
                              │  Physical    │
                              │  - Lights    │
                              │  - Buzzer    │
                              └──────────────┘
```

All clients stay synchronized within 0.1 second!

## Legacy Note

**Pilomat III** (2024) - Web-based, JSON-configurable
**Pilomat II** (1992) - 8051 Microcontroller-based
**Pilomat I** (1977) - TTL logic, the original! 🏆

---

*For support and updates, see the GitHub repository or contact the developer.*
