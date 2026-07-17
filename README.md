# 🌿 EcoSort — Smart Waste Segregation & Bin Monitoring System

An Arduino UNO based smart dustbin that automatically opens on hand approach, classifies waste as **Wet** or **Dry** using a moisture sensor, routes it into the correct compartment with a servo-actuated flap, and continuously monitors both bins for fill-level with LED + buzzer alerts. Designed and functionally validated in **Tinkercad Circuits**.

![Platform](https://img.shields.io/badge/Platform-Arduino%20UNO-00979D?style=flat-square&logo=arduino)
![Simulation](https://img.shields.io/badge/Simulated%20in-Tinkercad-1BA94C?style=flat-square)
![Language](https://img.shields.io/badge/Language-C%2B%2B%20(Arduino)-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-Simulation%20Validated-success?style=flat-square)

---

## 📌 Overview

Manual waste segregation is inconsistent and unhygienic. **EcoSort** removes the human decision step entirely:

1. A hand/object near the bin triggers an ultrasonic sensor → the lid opens automatically.
2. A moisture sensor reads the wetness of the deposited item.
3. A servo-actuated flap routes the item into the **Wet** or **Dry** compartment.
4. Two independent ultrasonic sensors continuously monitor fill-level in both compartments.
5. A buzzer + red/green LED pair alert when either bin is full.

---

## ⚙️ Components Used

| # | Component | Qty | Role |
|---|---|---|---|
| 1 | Arduino UNO | 1 | Main controller |
| 2 | HC-SR04 Ultrasonic Sensor | 3 | Hand detection, Wet-bin level, Dry-bin level |
| 3 | SparkFun Soil Moisture Sensor | 1 | Wet/Dry waste classification |
| 4 | SG90 Micro Servo | 2 | Lid actuation + Wet/Dry flap routing |
| 5 | Piezo Buzzer | 1 | Audible bin-full alert |
| 6 | Red LED / Green LED | 1 each | Visual bin-full / bin-available indicator |
| 7 | 220 Ω Resistor | 2 | LED current limiting |
| 8 | Breadboard + Jumper Wires | — | Power & signal distribution |

---

## 🔌 Pin Mapping

| Signal | Arduino Pin | Component |
|---|---|---|
| `trigHand` / `echoHand` | D2 / D3 | Hand-detection HC-SR04 |
| `trigWet` / `echoWet` | D4 / D5 | Wet-bin level HC-SR04 |
| `trigDry` / `echoDry` | D6 / D7 | Dry-bin level HC-SR04 |
| `buzzer` | D8 | Piezo Buzzer |
| `lidPin` | D9 | Lid Servo (PWM) |
| `flapPin` | D10 | Flap Servo (PWM) |
| `greenLED` | D11 | Bin-available indicator |
| `redLED` | D12 | Bin-full indicator |
| `moisturePin` | A0 | Soil Moisture Sensor (analog) |

---

## 🧠 Working Principle

- **Hand Detection & Lid Actuation** — if `handDistance < 15 cm`, the lid servo opens to 90°.
- **Wet/Dry Classification** — moisture reading `> 550` → **Wet Waste** (flap → 30°); otherwise → **Dry Waste** (flap → 150°).
- **Bin Fill Monitoring** — if `wetDistance <= 5 cm` or `dryDistance <= 5 cm`, red LED + buzzer activate; otherwise green LED stays on.
- The full sensor/actuator state is streamed to the Serial Monitor (9600 baud) every loop for debugging.

---

## 🖼️ Circuit Diagram

Full wiring — three HC-SR04 sensors, moisture sensor, dual servos, buzzer, and LED indicators on Arduino UNO, built and simulated in Tinkercad Circuits.

`/images/circuit_full_wiring.png`

---

## 🧪 Simulation Results

| Scenario | Sensor Input | Console Output | System Response |
|---|---|---|---|
| Wet waste deposit | Moisture 95.28% (raw 975) | `Wet Waste` | Flap → 30° (wet compartment) |
| Dry waste deposit | Moisture 50.00% (raw 512) | `Dry Waste` | Flap → 150° (dry compartment) |
| Dry bin near capacity | Dry-bin distance 2 cm | `DRY BIN FULL` | Red LED ON, buzzer sounds |
| Wet bin near capacity | Wet-bin distance 2 cm | `WET BIN FULL` | Red LED ON, buzzer sounds |
| Normal operation | Dry-bin distance 12 cm | — | Green LED ON, buzzer silent |

Screenshots of each scenario are in `/images`.

---

## 💻 Firmware

Full source: [`EcoSort_WasteBin.ino`](./EcoSort_WasteBin.ino)

```cpp
#include <Servo.h>

Servo lidServo;
Servo flapServo;

const int trigHand = 2;
const int echoHand = 3;
const int trigWet = 4;
const int echoWet = 5;
const int trigDry = 6;
const int echoDry = 7;
const int buzzer = 8;
const int lidPin = 9;
const int flapPin = 10;
const int greenLED = 11;
const int redLED = 12;
const int moisturePin = A0;

long getDistance(int trig, int echo) {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);
  long duration = pulseIn(echo, HIGH);
  return duration * 0.034 / 2;
}

void setup() {
  pinMode(trigHand, OUTPUT); pinMode(echoHand, INPUT);
  pinMode(trigWet, OUTPUT);  pinMode(echoWet, INPUT);
  pinMode(trigDry, OUTPUT);  pinMode(echoDry, INPUT);
  pinMode(buzzer, OUTPUT);
  pinMode(redLED, OUTPUT);   pinMode(greenLED, OUTPUT);

  lidServo.attach(lidPin);
  flapServo.attach(flapPin);
  lidServo.write(0);
  flapServo.write(90);

  Serial.begin(9600);
}

void loop() {
  long handDistance = getDistance(trigHand, echoHand);
  long wetDistance  = getDistance(trigWet, echoWet);
  long dryDistance  = getDistance(trigDry, echoDry);
  int moistureValue = analogRead(moisturePin);

  if (handDistance > 0 && handDistance < 15) {
    lidServo.write(90);
    delay(1000);

    if (moistureValue > 550) {
      flapServo.write(30);   // Wet bin
    } else {
      flapServo.write(150);  // Dry bin
    }
    delay(2000);

    flapServo.write(90);
    delay(500);
    lidServo.write(0);
    delay(500);
  }

  if (wetDistance <= 5) {
    digitalWrite(redLED, HIGH); digitalWrite(greenLED, LOW);
    tone(buzzer, 1000); delay(1000); noTone(buzzer);
  } else if (dryDistance <= 5) {
    digitalWrite(redLED, HIGH); digitalWrite(greenLED, LOW);
    tone(buzzer, 1000); delay(1000); noTone(buzzer);
  } else {
    digitalWrite(redLED, LOW); digitalWrite(greenLED, HIGH);
    noTone(buzzer);
  }

  delay(200);
}
```

> Full commented version with Serial debug prints is in the `.ino` file in this repo.

---

## 🚀 Applications

- Smart homes & apartment complexes — touch-free, pre-segregated waste disposal
- Office campuses & educational institutions — improved recycling compliance
- Public spaces & transit hubs — optimized collection routing via fill alerts

## 🔭 Future Scope

- ESP32-based Wi-Fi/MQTT connectivity for remote bin-full alerts
- A third compartment for recyclables (metal/plastic) via inductive/capacitive sensing
- Calibrated, material-specific moisture classification model
- Solar-assisted power with battery backup for outdoor deployment

---

## 👤 Author

**Pathan Mohammad Shoaib Khan**
B.Tech, Electronics and Communication Engineering — SRM University AP
🔗 [GitHub](https://github.com/shoaib725) · [LinkedIn](https://linkedin.com/in/shoaibkhan725)
