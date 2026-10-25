# Team-ReFlex
Upper Arm Rehabilation
# AXIS: Cable-Driven Upper-Limb Rehabilitation Prototype

AXIS is an engineering project that explores cable-driven movement of the forearm, wrist and elbow. The proposed system combines four motors with a Raspberry Pi Pico H, a serial-bus servo adapter and a local browser dashboard.

This repository brings together the CAD models, electronics schematics, bench-control software and design calculations.

**Project stage:** CAD and schematic development with a single detached-servo bench controller. The complete four-axis mechanism has not been integrated or tested under load. Rehabilitation effectiveness and human-use safety have not been demonstrated.

![AXIS integrated mechanical, power and control diagram](assets/axis-system-diagram.png)

## Movement modules

| Module | Intended movement | Proposed motors | Cable runs | Current status |
| --- | --- | ---: | ---: | --- |
| Forearm | Pronation and supination | 1 | 2 | Original Fusion/STEP CAD and a printed assembly reported to rotate freely by hand |
| Wrist, axis 1 | Flexion and extension | 1 | 2 | Reinforced soft-glove concept |
| Wrist, axis 2 | Radial and ulnar deviation | 1 | 2 | Same glove, with a second opposing cable pair |
| Elbow | Flexion and extension | 1 | 2 | Combined elbow/forearm STEP demonstrator, with physical corrections pending |
| **Planned total** | **4 movement axes** | **4** | **8** | **Integration pending** |

Each opposing cable pair is intended to wind and unwind through a motor-driven spool. Reversing motor direction reverses the cable exchange. Actual motion depends on cable anchoring, pretension, housing reaction mounts and a complete torque path.

### Forearm CAD

![Original forearm CAD rendering](assets/forearm-cad.png)

The original forearm model contains 151 closed solids. It includes a split support structure, rotating ring, sliding bearing interfaces, hand support and a remote cable-drive pod concept.

- [Original Fusion archive](AXIS_Forearm_Rotation_M.f3d)
- [Original STEP assembly](AXIS_Forearm_Rotation_M.step)
- [Dimensions and component guide](AXIS_Component_Dimensions.md)
- [Design notes](AXIS_Design_Notes.md)

The nominal ±60° travel is a CAD demonstration setting. It is not a prescribed rehabilitation range or a verified user-fit specification.

### Elbow attachment

![Combined elbow and forearm CAD rendering](assets/elbow-forearm-cad.png)

One motor and two opposing cables form the proposed elbow drive. A 40 mm elbow-drum pitch radius and 20 mm motor-spool pitch radius give a 2:1 ideal relationship between motor angle and elbow angle.

The exported assembly contains the 151 original forearm solids plus 76 new solids. STEP reimport checks report 227 valid solids at each of the three exported poses:

- [0° elbow pose](ElbowFusion/deliverables/AXIS_Elbow_Forearm_000deg.step)
- [55° elbow pose](ElbowFusion/deliverables/AXIS_Elbow_Forearm_055deg.step)
- [110° elbow pose](ElbowFusion/deliverables/AXIS_Elbow_Forearm_110deg.step)
- [STEP verification record](ElbowFusion/deliverables/STEP_roundtrip_check.json)
- [Physical feasibility review and unresolved defects](ElbowFusion/PHYSICAL_FEASIBILITY_REVIEW.md)

The motion viewer demonstrates ideal movement. It does not establish that the current assembly transmits torque or supports a physical load. The elbow has no newly generated, verified native Fusion F3D file.

### Wrist concept

![Wrist soft-glove concept illustration](concepts/AXIS_simple_glove_four_cables.png)

The latest wrist concept uses a reinforced soft glove, four cables and two motors. This concept illustration is not a photograph, completed wrist CAD assembly or tested prototype. Cable attachment, alignment, fit and calibration remain development tasks.

## Control and electronics

The proposed control path is:

**Laptop dashboard → Pico H → UART adapter → compatible serial-bus motors → cable mechanisms**

The laptop USB connection powers the Pico and carries dashboard commands. Motor power follows a separate battery, fuse, supply and adapter path, with common ground. The connected MPU6050 remains uncalibrated and is not part of a validated joint-angle control loop.

| Item | Existing bench configuration / proposed extension |
| --- | --- |
| Controller | Raspberry Pi Pico H, RP2040 |
| Firmware | MicroPython, recorded bench version 1.29.0 |
| Motor interface | Waveshare Bus Servo Adapter (A) V1.1, both selectors at A |
| Existing motor | Standard 12 V ST3215, ID 1, tested detached and unloaded |
| Added motors | IDs 2 and 3 for the wrist, ID 4 for the elbow, proposed |
| UART | UART0 at 1 Mbps, GP0 / physical pin 1 and GP1 / physical pin 2 |
| IMU connection | MPU6050 on I2C0, GP4 / GP5, 3.3 V supply |
| Existing switch inputs | GP2 / GP3 with internal pull-ups, normally-open contacts to ground |
| Existing motor power stage | 11.1 V nominal battery and XL4015, documented for the single-servo bench |

The recorded adapter TX-to-GP0 and RX-to-GP1 wiring follows the adapter's UART convention. Use the [detailed connection guide](electronics/AXIS_Electronics/README.md), rather than inferring connector pin order from an overview image.

### Schematics

![Proposed four-motor circuit schematic](electronics/AXIS_Electronics_4Motor_RevB/exports/AXIS_Electronics_4Motor.png)

- [Rev A: recorded single-servo bench schematic](electronics/AXIS_Electronics/AXIS_Electronics.kicad_sch)
- [Rev B: proposed four-motor schematic](electronics/AXIS_Electronics_4Motor_RevB/AXIS_Electronics_4Motor.kicad_sch)
- [Rev B PDF](electronics/AXIS_Electronics_4Motor_RevB/exports/AXIS_Electronics_4Motor.pdf)
- [Rev B assumptions and power requirements](electronics/AXIS_Electronics_4Motor_RevB/README.md)
- [Rev B connection list](electronics/AXIS_Electronics_4Motor_RevB/Connections.csv)
- [Rev B electrical-check record](electronics/AXIS_Electronics_4Motor_RevB/VERIFICATION.txt)

The KiCad projects are editable module-level schematics with a local symbol library. No completed PCB layout or fabricated PCB is claimed.

**Power expansion is unresolved.** Four standard ST3215 motors at the manufacturer's 2.7 A stall figure would total 10.8 A. This is a simultaneous-stall sizing case, not expected normal current. The existing 5 A XL4015 cannot cover that combined demand. Final actuator selection must inform the supply, battery/BMS, fuse, adapter power path, connectors and wiring. A different elbow motor may need a separate driver or supply.

## Design calculations

| Quantity | Forearm | Elbow |
| --- | ---: | ---: |
| Joint pulley pitch radius, R | 49.5 mm | 40 mm |
| Motor spool pitch radius, r | 12 mm | 20 mm |
| Motor angle / joint angle, R/r | 4.125:1 | 2:1 |
| CAD demonstration sweep | 120°, from −60° to +60° | 110°, from 0° to 110° |
| Required motor rotation | 495° | 220° |
| Cable exchange per tendon over that sweep | 103.67 mm | 76.79 mm |

For taut cables with no slip and constant pitch radii:

```text
motor_angle = (joint_radius / spool_radius) × joint_angle
cable_exchange = joint_radius × joint_angle_in_radians
ideal_joint_torque = (joint_radius / spool_radius) × motor_torque
```

These are ideal calculations from nominal CAD dimensions. Friction, compliance and an incomplete mechanical torque path prevent treating them as measured performance.

- [Calculation table image](Presentation/data_images/AXIS_Design_Calculations.png)
- [Editable calculation table](Presentation/data_images/AXIS_Design_Calculations.csv)
- [Calculated motion samples](Presentation/data_images/AXIS_Calculated_Motion_Samples.csv)
- [Digital design-check summary](Presentation/data_images/AXIS_Digital_Design_Checks.png)

## Explore the project

### View the CAD without Fusion

Download the repository ZIP and extract it. Open [the offline elbow motion viewer](ElbowFusion/deliverables/AXIS_Elbow_Motion_Viewer.html) in a desktop browser. GitHub's file view shows the HTML source rather than running the viewer. The viewer is self-contained and requires no server.

For a lighter preview, open the [motion GIF](ElbowFusion/deliverables/AXIS_Elbow_Bend_Straighten.gif). Import a STEP assembly into a compatible CAD tool to inspect the geometry. STEP does not preserve Fusion feature history or joints.

### Open the dashboard in simulation

With Node.js available, run:

```sh
cd electronics/bench_controller
node launcher.mjs --open
```

On Windows, you can also double-click `Launch dashboard.cmd` in that folder. Select the clearly labelled **Simulation** mode to explore the controls without hardware. Real Web Serial operation uses a supported desktop browser such as Chrome or Edge.

Simulation does not validate mechanical movement, cable tension, torque or actual stop response. The current real-hardware firmware controls only servo ID 1. Read the [bench controller guide](electronics/bench_controller/README.md) before any hardware session.

### Inspect the electronics

Open either `.kicad_pro` file in the corresponding electronics folder using KiCad. Keep its local `AXIS_Modules.kicad_sym` and `sym-lib-table` alongside the project.

## Verification and evidence

| Evidence | Recorded result | Scope |
| --- | --- | --- |
| Forearm CAD | 151 closed solids | Geometry count, not a physical parts inventory |
| Elbow STEP reimport | 227 valid solids in each of 3 poses | Exported geometry validity |
| Four-motor schematic ERC | 0 errors and 0 warnings | Drawing rules, not power-system adequacy |
| Four-motor connection comparison | 15 named nets and 52 terminals | Connectivity against the documented specification |
| Controller automated tests | 25 firmware and 26 browser tests passed in the development record | Simulated devices and transports |
| Detached-servo bench checks | User-reported jog, software STOP and both switch-stop checks | One unloaded servo, separate from the mechanism |

See the [software validation record](electronics/bench_controller/VALIDATION.md) for evidence boundaries. The automated tests are distinct from the reported physical bench checks.

To run the existing software checks without connecting hardware:

```sh
cd electronics/bench_controller
node --test tests/dashboard.test.mjs
python -m unittest discover -s tests -p "test_*.py" -v
```

The recorded software runs used Node.js 24 and CPython 3.14. A laptop test pass does not establish MicroPython compatibility or physical motor safety.

## Open engineering work

1. **Forearm travel:** The complete CAD sweep requires 495° of motor rotation, beyond what the present single-turn firmware provides.
2. **Elbow torque connection:** Add and validate a positive connection between the drum, shaft and moving yoke.
3. **Elbow reaction mounts:** Secure the motor-side cable housings to a fixed supporting structure.
4. **Cable routing:** Resolve the elbow cable/housing wall intersections identified in the physical audit.
5. **Wrist integration:** Fabricate the glove concept and establish cable paths, anchoring, fit and pretension.
6. **Actuation and power:** Select the elbow motor and size the complete power-distribution path for the selected loads.
7. **Control and feedback:** Implement multi-motor firmware, add elbow limits and direct angle sensing, then calibrate joint motion.
8. **Physical validation:** Measure angle error, output torque, loaded current, temperature, stop response and mechanical retention before considering wearer trials.

The existing software output setting is not a calibrated force or torque limit. Normally-open switch wiring does not detect a broken wire as a pressed switch. An independent hardwired motor-power cutoff has not been confirmed installed.

## Repository layout

```text
.
├── README.md
├── REPOSITORY_DETAILS.md
├── assets/                         # Overview diagram and CAD previews
├── AXIS_Forearm_Rotation_M.f3d      # Original forearm native CAD
├── AXIS_Forearm_Rotation_M.step
├── AXIS_Component_Dimensions.md
├── AXIS_Design_Notes.md
├── concepts/                       # Latest wrist glove illustration
├── ElbowFusion/                    # Elbow STEP files, viewer and audit
├── electronics/
│   ├── AXIS_Electronics/           # Recorded single-servo bench circuit
│   ├── AXIS_Electronics_4Motor_RevB/# Proposed expansion and checks
│   └── bench_controller/           # Dashboard, firmware and software tests
└── Presentation/data_images/       # Diagrams, calculation tables and CSVs
```

## References

- [Waveshare ST3215 specifications](https://docs.waveshare.com/ST3215_Servo)
- [Waveshare Bus Servo Adapter (A) wiring](https://docs.waveshare.com/Bus_Servo_Adapter_A/Product-Wiring-Example)
- [Raspberry Pi Pico datasheet](https://datasheets.raspberrypi.com/pico/pico-datasheet.pdf)
- [Related single-motor cable-actuation research](https://arxiv.org/abs/2306.17427), background for the mechanism principle, not validation of AXIS

## License

A project license has not yet been selected. This repository includes no `LICENSE` file for the original AXIS material. Third-party software and referenced material retain their respective terms.
