# ESP32-Based Home Automation Board

This project describes the hardware architecture and system specification for an ESP32-based, mains-capable home automation controller. The board provides secure wireless connectivity (Wi‑Fi and Bluetooth), telemetry for power monitoring, and galvanically-isolated relay actuation for switching AC mains loads.

## Table of Contents
- System overview and capabilities
- Total system architecture
- Power supply subsystem
- Telemetry and sensing
  - AC current sensing
  - AC voltage sensing
- Actuation and isolation subsystem
- BOM summary
- Safety notes
- Next steps & repository contents
- License

## 1. System overview and capabilities
The PCB is designed as a centralized, high-voltage home automation controller built around an ESP32-DEVKITC-32E development board. Main capabilities:
- Independent actuation: control up to ten separate AC mains channels via electromechanical relays.
- Real-time current monitoring: non-intrusive AC current telemetry using Hall-effect sensing.
- Robust isolation: galvanic isolation between high-voltage AC domains and low-voltage logic domains for safety and reliability.

## 2. Total system architecture
The board is organized into four electrically distinct subsystems:
1. Processing and communications: ESP32 module — logic, state management, Wi‑Fi/BLE connectivity.
2. Power management: converts AC mains to regulated DC rails used by logic and sensors.
3. Telemetry and sensing: analog sensors capture load voltage and current, conditioned for the ESP32 ADCs.
4. Isolated actuation: switches high-power AC loads under control of the ESP32 with galvanic isolation.

## 3. Power supply subsystem
- Input: 3-pin Phoenix terminal block (J2) for mains (LIVE, NEUTRAL, EARTH).
- Protection: LIVE routed through a 10 A cylindrical fuse (F1, 5×20 mm). A master toggle switch (S1) provides manual power isolation.
- AC→DC: HLK-5M05 (PS1) SMPS module supplies isolated, regulated 5 V DC from 100–240 V AC.
- Distribution: 5 V rail powers relay coils, analog sensor circuitry, and the ESP32-DEVKITC-32E (which uses its onboard LDO to generate 3.3 V).
- Status LED: D1 with R1 (1 kΩ) indicates presence of the 5 V rail.

## 4. Telemetry and sensing capabilities
To support energy management, the board measures both current and voltage on monitored circuits.

### 4.1 AC current sensing
- Sensor: ACS712 Hall-effect linear current sensor (U2).
- Placement: The ACS712 sits in series with the AC path (using a copper conduction path) so load current creates a proportional Hall voltage.
- Conditioning: The sensor output (VIOUT) is filtered with C2 (1 nF) — a low-pass to reduce high-frequency noise — then fed to an ESP32 ADC pin.

### 4.2 AC voltage sensing
- Sensor: ZMPT101B isolated micro-voltage transformer (U4) steps down and isolates mains voltage.
- Primary protection: High-value resistor R2 (820 kΩ) in series with the primary to limit current.
- Conditioning: The transformer's secondary (micro AC signal) is amplified and DC-shifted using an LM358 dual op‑amp (U3A and U3B) to produce a centered waveform within the ESP32 ADC input range (0–3.3 V). A 10 kΩ trim potentiometer (RV1) allows amplitude calibration.
- Filtering: Passive and op-amp stage filtering (C4/C6, etc.) used to remove HF noise and stabilize readings.

## 5. Actuation and isolation subsystem
- Relay channels: Five independent relay channels in the referenced design (K6–K10). The top-level description supports up to ten channels if duplicated as required.
- Isolation: ESP32 GPIOs drive PC817 optocouplers (U7–U11) providing galvanic isolation between logic and relay drive stages.
- Coil driving: Optocoupler outputs drive 2N3904 NPN transistors (Q6–Q10) which sink the relay coils to ground from the 5 V rail.
- Flyback protection: Each relay coil has a reverse-biased 1N4148 diode (D17–D21) to clamp inductive voltage spikes when coils are turned off.
- Status LEDs: Each channel includes an indicator LED (D12–D16) in series with the optocoupler input to show logic state; series resistors (R17–R21 or R12–R16 where noted) set LED currents.

## 6. Component breakdown (BOM summary)
| Category | Reference | Part / Value | Function |
|---|---:|---|---|
| Microcontroller | U1 | ESP32-DEVKITC-32E | Core processing, Wi‑Fi/BLE, logic |
| Power module | PS1 | HLK-5M05 | Mains → isolated 5 V DC |
| Current sensor | U2 | ACS712 | Hall-effect AC current telemetry |
| Voltage sensor | U4 | ZMPT101B | Isolated AC voltage step-down |
| Op amp | U3 | LM358 | Signal conditioning for ZMPT101B |
| Relays | K6–K10 | 5 V SPDT relays | High-voltage load switching |
| Optocouplers | U7–U11 | PC817 | Galvanic isolation |
| Transistors | Q6–Q10 | 2N3904 | Relay coil drivers |
| Diodes (flyback) | D17–D21 | 1N4148 | Relay coil transient protection |
| LEDs | D1, D12–D16 | 3–5 mm LEDs | Power & channel status indicators |
| Potentiometer | RV1 | 10 kΩ (3296W-1-103LF) | Voltage sensor amplitude calibration |
| Fuse | F1 | 10 A (5×20 mm) | Input overcurrent protection |
| Resistors | R1 etc. | R1=1 kΩ, R2=820 kΩ, others per schematic | Current limiting, scaling |
| Capacitors | C1..C6 etc. | 1 µF, 10 nF, 1 nF | Decoupling and signal filtering |
| Connectors | J2, J3 | Phoenix MKDS 1×03 | Mains input and distribution |

(Adjust quantities and values to match your final schematic and number of relay channels.)

## 7. Safety notes
- This design interfaces directly with AC mains. High voltages are lethal and can cause fire: do not power or test the PCB without appropriate safety precautions and institutional/experienced oversight.
- Maintain proper creepage and clearance on the PCB between mains and low-voltage circuitry.
- The HLK-5M05 module is an enclosed isolated module — still follow safety rules for mains wiring.
- Ensure fuses, separation, safety grounding, and line filtering are applied per local electrical and regulatory standards.

## 8. Next steps / suggested repository contents
Suggested files and folders for the repo:
- README.md (this file)
- schematics/ — eagle/PCB or KiCad schematic files and PDFs
- board_layout/ — PCB layout files and Gerbers
- firmware/ — ESP32 firmware source (Arduino, Espressif-IDF, or PlatformIO)
- docs/ — datasheets, calibration procedures, BOM (CSV), test procedures
- safety/ — detailed safety & compliance notes, test reports
- LICENSE (choose license)
- .gitignore

If you want, I can:
- Create README.md in an existing repository you name (provide owner/repo).
- Generate suggested schematic templates (KiCad netlist or example files).
- Draft an initial PlatformIO project with a stub ESP32 firmware to read ADCs and toggle relays.

## 9. License
Add a LICENSE file of your choice (MIT, Apache‑2.0, etc.). Tell me which license you prefer and I can add it.

---
Prepared from your hardware specification. If you want me to add this README.md to a GitHub repository, please provide the repository in owner/repo format (for example: Senyoj/home-automation-board) and confirm you want the README created there. If you prefer, I can instead provide the git commands to create the repo and push this README from your machine.