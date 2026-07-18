# P3 EV Bike Charger PLC

A PLC-based lithium battery charger controller built on a Siemens S7-1516-3 PN/DP with a Siemens MTP1500 Unified Comfort HMI. The system implements a full CC/CV charging algorithm with thermal derating and fault handling. It was designed for a 13S lithium-ion e-bike battery pack (48V nominal) and tested in simulation using TIA Portal PLCSIM and a real HMI.

## Demo

[Watch the demo video](https://github.com/Rayan1092/P3-ev-bike-charger-plc/blob/main/demo_video.txt)

## What it does

The charger runs a full CC/CV sequence automatically after the operator sets a target voltage and current on the HMI and presses START.

CC phase pushes constant current into the battery until voltage hits the target. CV phase then holds the voltage steady and lets current taper naturally. A trickle phase tops off the last fraction at 5% of target current. The system moves to DONE when both voltage and current conditions are met.

Thermal derating reduces the charge current proportionally if battery temperature rises between 35C and 45C. Above 45C it faults out.

All five faults latch and need a manual reset from the HMI after the condition clears.

## Hardware

- PLC: Siemens SIMATIC S7-1516-3 PN/DP
- HMI: Siemens MTP1500 Unified Comfort
- Software: TIA Portal V20

No physical battery was used. The system was designed for a 13S lithium-ion e-bike pack (48V nominal, 54.6V full charge) and all testing was done in simulation with manually forced analog values.

## Software structure

Three function blocks called from OB1 each scan.

`FB_ChargeController` runs the state machine and PID loop. It takes target voltage, target current, PID gains and the fault flag as inputs. It handles all six states and outputs the PWM command.

`FB_ThermalDerating` computes the derated current setpoint based on battery temperature. It outputs the adjusted setpoint to FB_ChargeController.

`FB_FaultHandler` checks all five fault conditions each scan. It latches the fault flag and clears it only when all conditions are gone and the operator resets.

OB1 scales the analog inputs, calls the three FBs in order and writes the output.

## States

| State | Value |
|---|---|
| IDLE | 0 |
| CC | 1 |
| CV | 2 |
| TRICKLE | 3 |
| DONE | 4 |
| FAULT | 5 |

## Faults

| Fault | Condition |
|---|---|
| Overvoltage | Battery voltage exceeds target x 1.05 |
| Overtemperature | Battery temp above 45C |
| Charge timeout | Session exceeds 8 hours |
| Overcurrent | Current exceeds target x 1.10 |
| E-Stop | HMI E-Stop button pressed |

## Repository structure

```
P3-ev-bike-charger-plc/
├── src/
│   ├── FB_ChargeControl.scl
│   ├── FB_FaultHandler.scl
│   ├── FB_ThermalDerating.scl
│   └── Main.scl
├── EBike charging station Final Rayan 2026.zap20
├── FDS_EBikeCharger_v1.2.docx
├── HMI Display Image.jpg
├── demo_video.txt
└── README.md
```

The `.zap20` file is the full TIA Portal project archive. Open it with Project -> Retrieve in TIA Portal V20. The SCL files in `/src` are the raw source code exported from each block.

The FDS was generated with Claude since I needed something to follow as a design structure.
