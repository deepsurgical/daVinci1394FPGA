# 🐍 From Snake Robots to Surgical Standards: The Architectural Origins of the da Vinci Research Kit

Welcome! I am **Paul Thienphrapa**, a medical robotics and autonomous systems scientist-engineer serving as head advisor for the Autonomous Interventions and Robotics (AIR) program at the Advanced Research Projects Agency for Health (ARPA-H).

This repository (`daVinci1394FPGA`) preserves the foundational hardware control architecture designed as part of my graduate research at Johns Hopkins University. While the active codebase for the [da Vinci Research Kit (dVRK)](https://www.intuitive-foundation.org/dvrk/) has since transitioned to the institutional [JHU CISST Mechatronics Firmware Repository](https://github.com/jhu-cisst/mechatronics-firmware) to support collaboration, the structural backbone of that hardware interface layer originates directly from a completely separate robotic platform: **a high-degree-of-freedom surgical snake robot.**

### 1. The Design Baseline: The Snake Robot Cabling Crisis
During my early days in the LCSR, the lab had built a highly dexterous, remote telesurgery snake robot designed for upper airway interventions. Managing dozens of sensors and actuators for a miniature, articulated robot created a cumbersome physical bottleneck. Standard architectures required running separate, bulky wiring bundles back to a centralized electronics rack for every single axis.

To solve this, I helped design a hardware controller architecture optimized for high-degree-of-freedom systems:
* **Centralized Processing, Distributed I/O:** Rather than routing a thick bundle of wires from every each and every joint back to the workstation for processing, sensor and actuator signals were digitized and serialized by a nearby FPGA.
* **The FireWire Bus:** Real-time, low-latency motor commands and sensor readings were multiplexed over a single, high-speed IEEE 1394 (FireWire) serial bus.
* **The Codebase:** This implementation was built as the now legacy codebase [`SnakeFPGA-rev2`](https://github.com/deepsurgical/SnakeFPGA-rev2).

### 2. Migration to `daVinci1394FPGA`
When Intuitive Surgical later provided first-generation retired da Vinci systems for research, labs faced a similar engineering hurdle. The da Vinci arms needed an entirely new high-bandwidth, low-latency motor controller interface. Because the underlying FireWire communication protocols and data packets of `SnakeFPGA-rev2` were decoupled from robot kinematics, the snake robot controller was a perfect match for the da Vinci interface boards (the Quad Linear Amplifiers or QLAs).

I created this (`daVinci1394FPGA`) repository and ported the Altera-based snake robot core to the new Xilinx-based da Vinci hardware. Because the data transfer interface handled generic multi-axis distributed I/O, the architecture adapted seamlessly, bringing up the first functional test hardware in no time.

### 🔍 Architectural Lineage & Implementation Details

This early codebase was eventually migrated from our internal university servers to GitHub by other lab contributors, so the architecture's lineage is best understood by looking at the inline file headers and early porting commits.

#### Naming Timeline
Prior to the wider open-source rollout and subsequent rebranding to the da Vinci Research Kit (dVRK), the platform was provisionally referred to as the **"Intuitive Research Kit"** reflecting its initial donation status. The name persists in the [dVRK GitHub repository](https://github.com/jhu-dvrk/sawIntuitiveResearchKit) as well as in my [PhD thesis](http://jhir.library.jhu.edu/handle/1774.2/37924) ([pdf](https://rose.mepaul.com/w/images/b/b1/Pault_thesis-136-final.pdf#page=218)):
> _Besides enabling use of the Snake Robot (Section 3.4.2), the outcomes of this effort formed the basis for JHU Open Source Mechatronics [155, 156], which publicly hosts a set of electronics design files, FPGA code, and basic software for a FireWire-based motion controller. This in turn is a component of the **Intuitive Research Kit** [157,158]._

#### Inline Header History
Comparing the low-level Verilog files preserved in this archival repository against the [official FireWire.v source code](https://github.com/jhu-cisst/mechatronics-firmware/blob/main/FPGA1394_QLA/Verilog/FireWire.v), the historical migration events can be found etched in the comment logs:

```verilog
/*
 * ...
 * Revision history
 *     04/24/08    Paul Thienphrapa    Initial revision
 *     10/13/10    Paul Thienphrapa    Copied from SnakeFPGA-rev2 and tweaked
 *                                       for Xilinx
 *     10/31/11    Paul Thienphrapa    React to rx packets only when addressed
 *     11/11/11    Paul Thienphrapa    Happy 111111!!11!
 *                                     Fixed mixed blocking/non-blocking issues
 *     10/16/13    Zihan Chen          Modified to support hub capability
 * ...
 */
```

#### The 7-to-8 Axis Adaptation
Later artifacts of this porting sequence is visible in [one of my early commits](https://github.com/jhu-cisst/mechatronics-firmware/commit/b6e0830884eb06d80bc08d6f908d80e43b5e1405) to the official repository, before I had a GitHub account. A simple refactor shows the physical translation of the code between the two platforms:
* **The Refactor:** Modifying the hardcoded system parameters to increase the number of supported axes from **7 to 8**.
* **The Constraint:** While the original snake robot had only seven (7) control axes in `SnakeFPGA-rev2`, the corresponding controller board on the dVRK platform was designed to handle 8 axes.

#### Foundations for Growth
What started off as a humble set of 16 Verilog FPGA code files has expanded to support a wide array of hardware variants and technical features.

### Systems Engineering Takeaway
This repository stands as a case study in the power of modular digital system design. It demonstrates that a communications and computing architecture abstracted to handle arbitrary multi-axis distributed I/O over a unified serial interface can cleanly outlive its original physical form factor, scaling from a miniature articulated snake robot to one of the most visible research platforms in medical robotics history.
