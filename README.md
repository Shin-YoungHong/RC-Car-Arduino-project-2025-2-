# Arduino Line Tracer Control

Arduino-based autonomous line-tracing RC car built for **Creative Engineering Design (창의공학설계)**.  
Implemented closed-loop sensor-to-motor driving logic, ultrasonic start trigger, and real-world PWM motor calibration.

## System Overview
- **MCU:** Arduino
- **Sensors:** 2-channel IR line sensors (Left/Right), HC-SR04 Ultrasonic sensor
- **Actuators:** Dual DC motors with PWM speed control (5V → 9V power upgrade)

### Pin Configuration
| Component | Signal | Arduino Pin |
| :--- | :--- | :--- |
| Left Motor | PWM Enable / Forward / Backward | `9` / `4` / `2` |
| Right Motor | PWM Enable / Forward / Backward | `10` / `7` / `11` |
| IR Sensors | Left / Right | `8` / `6` |
| Ultrasonic | Trig / Echo | `12` / `13` |

## Control Logic
1. **Start Trigger (`setup`):** Waits until an object is detected within `0 < distance < 10 cm` via the ultrasonic sensor, then applies a short `255` PWM kick (`250 ms`) to overcome static friction.
2. **Line Tracking (`loop`):**
   - `(0, 0)` White / White → Straight forward (`STRAIGHT_FORWARD = 200`)
   - `(0, 1)` Right black → Hard right turn (differential reverse wheel braking)
   - `(1, 0)` Left black → Hard left turn with asymmetric RPM compensation
   - `(1, 1)` Both black → Full stop (intersection / boundary safety stop)

## Hardware Troubleshooting & Control Tuning
Real-world driving exposed hardware non-idealities that required iterative software and hardware fixes:
- **Asymmetric Motor RPM Compensation:** Left and right DC motors exhibited different torque/RPM curves under identical PWM duty cycles. Added directional offsets (`STRAIGHT_SPEED + 25`, `REVERSE_SPEED + 55`) during left turns after track testing.
- **8-bit PWM Overflow Guard:** Kept base speed below `245` when applying additive offsets so `analogWrite()` never wraps around `255` and stalls the motor.
- **Sensor Fault Diagnosis:** Isolated a faulty right IR sensor by writing a dedicated `Serial.monitor` diagnostic sketch, confirming hardware failure, and replacing the module to complete full-track laps.
