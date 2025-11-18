# Smart Car Parking System using ESP8266 & Blynk IoT

A complete IoT-based smart parking management system that monitors parking slot availability in real-time, controls entry/exit gates automatically, and provides remote monitoring through Blynk mobile and web applications.

## Features

- **Real-time Slot Monitoring**: Track 3 parking slots with IR sensors
- **Automatic Gate Control**: Entry and exit gates open automatically when vehicles are detected
- **IoT Dashboard**: Monitor and control the system remotely via Blynk (Web & Mobile)
- **LCD Display**: Local 16x2 I2C LCD shows available slots and system status
- **Manual Override**: Control gates manually through Blynk buttons
- **Debounced Sensors**: Stable sensor readings with noise filtering

## Hardware Requirements

### Components
- 1x NodeMCU ESP8266 (ESP-12E Module)
- 5x IR Obstacle Avoidance Sensors
- 1x Servo Motor (SG90 or similar)
- 1x 16x2 LCD with I2C Module
- Jumper wires
- Breadboard
- Power supply (5V, 2A recommended)

### Optional
- 100µF capacitor (for power stabilization)
- External 5V power supply for servo motor

## Circuit Diagram

### Pin Connections

#### NodeMCU ESP8266

| Component | Pin | NodeMCU Pin | Description |
|-----------|-----|-------------|-------------|
| **LCD I2C** | VCC | 3.3V or VIN | Power |
| | GND | GND | Ground |
| | SDA | D2 (GPIO4) | I2C Data |
| | SCL | D1 (GPIO5) | I2C Clock |
| **Entry IR Sensor** | VCC | 3.3V | Power |
| | GND | GND | Ground |
| | OUT | D3 (GPIO0) | Signal |
| **Exit IR Sensor** | VCC | 3.3V | Power |
| | GND | GND | Ground |
| | OUT | D4 (GPIO2) | Signal |
| **Slot 1 IR Sensor** | VCC | 3.3V | Power |
| | GND | GND | Ground |
| | OUT | D5 (GPIO14) | Signal |
| **Slot 2 IR Sensor** | VCC | 3.3V | Power |
| | GND | GND | Ground |
| | OUT | D6 (GPIO12) | Signal |
| **Slot 3 IR Sensor** | VCC | 3.3V | Power |
| | GND | GND | Ground |
| | OUT | D7 (GPIO13) | Signal |
| **Servo Motor** | Red (VCC) | VIN | Power (5V) |
| | Brown/Black (GND) | GND | Ground |
| | Orange/Yellow (Signal) | D8 (GPIO15) | Control Signal |

### Connection Notes
- All sensor VCC pins can be connected to a common 3.3V rail
- All GND pins must be connected together (common ground)
- If LCD is dim, connect its VCC to VIN (5V) instead of 3.3V
- For better servo performance, use external 5V power supply

## Software Requirements

### Arduino IDE Setup

1. **Install Arduino IDE** (v1.8.19 or newer)
   - Download from: https://www.arduino.cc/en/software

2. **Add ESP8266 Board Support**
   - File → Preferences → Additional Board Manager URLs
   - Add: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`
   - Tools → Board → Boards Manager → Search "ESP8266" → Install

3. **Install Required Libraries**
   - Sketch → Include Library → Manage Libraries
   - Install the following:
     - **Blynk** by Volodymyr Shymanskyy
     - **LiquidCrystal I2C** by Frank de Brabander
     - **Servo** (built-in with ESP8266 board package)

### Blynk Setup

1. **Create Blynk Account**
   - Web: https://blynk.cloud
   - Or download Blynk IoT app (iOS/Android)

2. **Create New Template**
   - Name: Car Parking
   - Hardware: ESP8266
   - Connection: WiFi

3. **Configure Datastreams**

| Virtual Pin | Name | Data Type | Min | Max | Default |
|-------------|------|-----------|-----|-----|---------|
| V0 | Entry Gate | Integer | 0 | 1 | 0 |
| V1 | Exit Gate | Integer | 0 | 1 | 0 |
| V2 | Slot 1 Status | Integer | 0 | 255 | 0 |
| V3 | Slot 2 Status | Integer | 0 | 255 | 0 |
| V4 | Slot 3 Status | Integer | 0 | 255 | 0 |

4. **Design Dashboard**
   - Add 2x **Button** widgets (V0, V1) for gate control
   - Add 3x **LED** widgets (V2, V3, V4) for slot status
   - Set buttons to "Switch" mode for visual feedback

5. **Create Device**
   - Devices → New Device → From Template
   - Copy the **Auth Token**

## Installation

### 1. Clone Repository
```bash
git clone https://github.com/yourusername/smart-parking-system.git
cd smart-parking-system
```

### 2. Configure Code
Open the `.ino` file and update:

```cpp
// Blynk credentials
#define BLYNK_AUTH_TOKEN "YOUR_AUTH_TOKEN_HERE"

// WiFi credentials
char ssid[] = "YOUR_WIFI_SSID";
char pass[] = "YOUR_WIFI_PASSWORD";

// LCD I2C Address (usually 0x27 or 0x3F)
LiquidCrystal_I2C lcd(0x27, 16, 2);  // Change if needed
```

### 3. Find LCD I2C Address (if display doesn't work)
Upload this I2C scanner code:
```cpp
#include <Wire.h>

void setup() {
  Serial.begin(115200);
  Wire.begin(D2, D1);
  
  Serial.println("\nI2C Scanner");
  byte count = 0;
  for (byte i = 1; i < 127; i++) {
    Wire.beginTransmission(i);
    if (Wire.endTransmission() == 0) {
      Serial.print("Found address: 0x");
      Serial.println(i, HEX);
      count++;
    }
  }
}

void loop() {}
```

### 4. Upload Code
1. Connect NodeMCU via USB
2. Tools → Board → NodeMCU 1.0 (ESP-12E Module)
3. Tools → Port → Select your COM port
4. Upload → Wait for "Done uploading"

## Usage

### Initial Setup
1. Power on the system
2. LCD shows "Smart Parking" → "Initializing..."
3. System connects to WiFi (dots appear on LCD)
4. "WiFi Connected!" → Shows IP address
5. "Blynk Connected" → System ready

### Normal Operation

**LCD Display:**
```
Available: 2/3
S1:E S2:F S3:E
```
- E = Empty slot
- F = Full/Occupied slot

**Automatic Mode:**
- Vehicle approaches entry → Gate opens automatically for 3 seconds
- Vehicle approaches exit → Gate opens automatically for 3 seconds

**Manual Mode (Blynk):**
- Press "Entry Gate" button → Gate opens
- Press "Exit Gate" button → Gate opens
- LEDs show real-time slot status (Red = Occupied, Off = Available)

### LED Indicators on IR Sensors
- **Power LED (Red)**: Always ON when powered
- **Detection LED (Green/Blue)**: ON when object detected

## Troubleshooting

### Issue: LCD Not Working
**Solution:**
- Check I2C address (use scanner code above)
- Try different address: `0x27` or `0x3F`
- Ensure VCC connected to proper voltage (try VIN if 3.3V doesn't work)

### Issue: IR Sensors Flickering
**Solution:**
1. Adjust potentiometer on sensor (turn clockwise to reduce sensitivity)
2. Code includes debouncing (200ms) to filter noise
3. Add 100µF capacitor between VCC and GND of sensors
4. Keep sensor wires away from servo motor wires

### Issue: Servo Not Moving
**Solution:**
- Check if servo is connected to D8
- Ensure servo gets enough power (use external 5V supply if needed)
- Check Serial Monitor for "Gate opening..." messages

### Issue: Blynk Offline
**Solution:**
- Verify WiFi credentials are correct
- Check Auth Token matches your Blynk device
- Ensure Template ID is correct
- Check Serial Monitor for connection errors

### Issue: Gates Opening Randomly
**Solution:**
- Adjust IR sensor sensitivity using potentiometer
- Ensure sensors are stable (not loose)
- Check for electrical noise interference
- Code includes 5-second cooldown between operations

## System Behavior

### Gate Control Logic
- **Cooldown Period**: 5 seconds between gate operations
- **Opening Duration**: 3 seconds (servo rotates 90°)
- **Debounce Time**: 200ms for entry/exit sensors
- **Auto-close**: Gate automatically closes after 3 seconds

### Sensor Logic
- **HIGH (1)**: No object detected / Slot empty
- **LOW (0)**: Object detected / Slot occupied
- Entry/Exit sensors use **INPUT_PULLUP** for stability

### Update Intervals
- **Blynk Updates**: Every 1 second
- **LCD Updates**: Every 500ms
- **Serial Monitor**: Every 1 second

## Code Structure

```
├── setup()
│   ├── Initialize LCD
│   ├── Connect WiFi
│   ├── Connect Blynk
│   └── Setup sensors & servo
├── loop()
│   ├── Blynk.run()
│   ├── Update slot status
│   ├── Read sensors with debouncing
│   └── Trigger gate operations
├── Functions
│   ├── updateLCD() - Display slot status
│   ├── showGateMessage() - Show gate operation
│   ├── operateGate() - Control servo motor
│   └── debounceRead() - Filter sensor noise
└── Blynk Handlers
    ├── BLYNK_WRITE(V0) - Entry button
    ├── BLYNK_WRITE(V1) - Exit button
    └── BLYNK_CONNECTED() - Sync on connect
```

## Serial Monitor Output

### Normal Operation
```
=== Car Parking System Starting ===
Hardware initialized
Connecting WiFi...........
WiFi Connected!
IP: 192.168.1.100
Blynk Connected!
Entry:1 Exit:1 | Slots: 1 0 1
```

### When Vehicle Enters
```
!!! Entry sensor TRIGGERED (stable) !!!
>>> Gate opening (triggered by: Entry Sensor)
>>> Gate closed
```

## Future Enhancements

- [ ] Add camera for license plate recognition
- [ ] SMS/Email notifications when slots are full
- [ ] Historical data logging
- [ ] Multiple gate support
- [ ] Mobile app push notifications
- [ ] Payment integration
- [ ] Reserved parking slots
- [ ] Vehicle counting system

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a new branch (`git checkout -b feature/improvement`)
3. Make your changes
4. Commit your changes (`git commit -am 'Add new feature'`)
5. Push to the branch (`git push origin feature/improvement`)
6. Create a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Credits

- **Hardware**: NodeMCU ESP8266, IR Sensors, Servo Motor
- **Software**: Arduino IDE, Blynk IoT Platform
- **Libraries**: Blynk, LiquidCrystal_I2C, Servo

## Contact

For questions or support:
- GitHub: [@syedtazzi](https://github.com/syedtazzi)
- Email: kamal2305171025@diu.edu.bd

## Acknowledgments

- Thanks to the Arduino and ESP8266 community
- Blynk IoT platform for easy cloud connectivity
- All contributors and testers

---

**⭐ If you found this project helpful, please give it a star!**

## Project Status

✅ **Active Development** - Fully functional and tested

**Last Updated**: November 2025
