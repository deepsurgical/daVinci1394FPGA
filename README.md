# 🐍 From Snake Robots to Surgical Standards: The Architectural Origins of the da Vinci Research Kit

Welcome! My name is **Paul Thienphrapa** and I'm a medical robotics and autonomous systems scientist-engineer serving as head advisor for the Autonomous Interventions and Robotics (AIR) program at the Advanced Research Projects Agency for Health (ARPA-H).

This repository (`daVinci1394FPGA`) preserves the foundational hardware control architecture designed as part of my graduate research at Johns Hopkins University. The active codebase for the [da Vinci Research Kit (dVRK)](https://www.intuitive-foundation.org/dvrk/) has since transitioned to the [JHU Mechatronics Firmware](https://github.com/jhu-cisst/mechatronics-firmware) repository, with the final commit here leading directly into the [initial commit `a38917c`](https://github.com/jhu-cisst/mechatronics-firmware/tree/a38917c2ce4880597da052d8d369eaeaf47659b4) there. The control architecture itself originates directly from a completely separate robotic system: **a high-degree-of-freedom surgical snake robot.**

### 1. Where It All Began: The Snake Robot Cabling Crisis

<p align="center"><img width="438" height="328" alt="Under a conventional control architecture, robot joints are wired directly to the workstation bus for processing, resulting in a complex cabling scenario." src="https://github.com/user-attachments/assets/4927e4fb-b4fe-4480-857b-854357ff21d5" /> <img height="328" alt="The original snake robot actuation unit also suffered from cable drag." src="https://github.com/user-attachments/assets/bc741eae-0082-45da-822e-0fd8371acf59" /></p>

During my early days in the LCSR, the lab had built a highly dexterous snake robot designed for remote telesurgery on the upper airways. However, standard architectures involved running bulky wiring bundles to a centralized electronics rack for every single control axis. The dozens of sensors and actuators needed to articulate this (ironically) miniature robot created a unmitigated cabling bottleneck--the system became brittle and impossible to troubleshoot.

To solve this, we designed a hardware controller architecture optimized for high-degree-of-freedom systems:
* **Centralized Processing, Distributed I/O:** Rather than routing a thick bundle of wires from each and every joint back to the workstation for processing, sensor and actuator signals were digitized and serialized on an FPGA on the robot [🔗](https://www.researchgate.net/publication/224353229_Centralized_processing_and_distributed_IO_for_robot_control).
* **The FireWire Bus:** Real-time, low-latency motor commands and sensor readings were multiplexed over a single, high-speed IEEE 1394 (FireWire) serial bus.
* **The Codebase:** This implementation was built as the now legacy codebase [`SnakeFPGA-rev2`](https://github.com/deepsurgical/SnakeFPGA-rev2).

To top it all off (so to speak), we built a next-gen snake robot for the new controller and got [exciting results](https://www.researchgate.net/publication/254025466_Design_of_a_scalable_real-time_robot_controller_and_application_to_a_dexterous_manipulator).
<p align="center"><img height="583" alt="We built a next-gen snake robot for the new controller." src="https://github.com/user-attachments/assets/f4c8e592-6f26-42aa-ae15-6304806d144b" /></p>

### 2. Porting to the da Vinci Robot
When Intuitive Surgical later provided retired first-generation da Vinci systems for research, labs faced a similar engineering hurdle. The da Vinci arms needed a brand new, high-bandwidth, low-latency motion control interface. Because the underlying FireWire communication protocols and packet definitions of `SnakeFPGA-rev2` were decoupled from robot kinematics, the snake robot controller was a perfect match for the da Vinci interface boards (the Quad Linear Amplifiers or QLAs).

<p align="center"><img height="218" alt="First-generation da Vinci Research Kit controller pictured on the left a year before launch, next to its predecessor the snake robot controller on the right" src="https://github.com/user-attachments/assets/ee1c27f2-e760-4bae-828f-442b6b8efd25" /> <img height="218" alt="The da Vinci Research Kit and snake robot controllers were developed in parallel so changes in one could be tested on the other immediately. This was at an advanced stage where robotic mechanisms and motors were being integrated and it was getting real." src="https://github.com/user-attachments/assets/01a7c7e6-b790-4ac5-a861-352e2342024d" />
</p>

I created this repository to port the Altera-based snake robot firmware to the new Xilinx-based da Vinci hardware. The two controllers were then refined in parallel to ensure consistency and generalizability. Since the architecture was designed around generic, multi-axis data distribution, the architecture adapted seamlessly, bringing up the first functional tests in no time.

[First known recorded power-on of the da Vinci Research Kit (dVRK) motion controller](https://github.com/user-attachments/assets/3037465c-1144-416a-8ffe-dfd9f047238d)

### 🔍 Architectural Lineage & Implementation Details

This `daVinci1394FPGA` codebase was eventually migrated from our internal university servers to GitHub by other lab contributors, so the architecture's lineage is best reconstructed by looking at file comments and other clues.

#### Naming Timeline
Before the open-source rollout and rebrand to the _da Vinci Research Kit (dVRK)_, the platform was provisionally referred to as the _Intuitive Research Kit_ reflecting its initial donation status. The name persists in the [dVRK GitHub repository](https://github.com/jhu-dvrk/sawIntuitiveResearchKit) and in my own [PhD thesis](http://jhir.library.jhu.edu/handle/1774.2/37924) ([pdf](https://rose.mepaul.com/w/images/b/b1/Pault_thesis-136-final.pdf#page=218)):
> _Besides enabling use of the Snake Robot (Section 3.4.2), the outcomes of this effort formed the basis for JHU Open Source Mechatronics [155, 156], which publicly hosts a set of electronics design files, FPGA code, and basic software for a FireWire-based motion controller. This in turn is a component of the **Intuitive Research Kit** [157,158]._

#### The History in the Headers
Comparing the low-level Verilog files preserved in this archival repository against the [official FireWire.v source code](https://github.com/jhu-cisst/mechatronics-firmware/blob/main/FPGA1394_QLA/Verilog/FireWire.v), the migration log can be found etched in the header comments. The file was initially written for the snake robot in 2008, ported to the da Vinci 2.5 years later in 2010, then received sporadic updates before moving to GitHub in Sep 2012:

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

#### The 7-to-8 Additional Axis Adaptation
Later artifacts of this porting sequence show up in [my prehistoric commit](https://github.com/jhu-cisst/mechatronics-firmware/commit/b6e0830884eb06d80bc08d6f908d80e43b5e1405) to the official repository, before I had a GitHub account. A simple refactor shows the physical translation between the two platforms:

> Paul Thienphrapa, Oct 16, 2012, FPGA1394_QLA: Increase maximum number of axes from 7 to 8

* **The Refactor:** Modifying the hardcoded system parameters to increase the number of supported axes from **7 to 8**.
* **The Adaptation:** The snake robot had only seven (7) control axes in `SnakeFPGA-rev2`, while the corresponding dVRK controller was designed for 8.

#### Designed for Growth
What started off as a modest set of .v files has expanded to support a wide array of hardware variants and technical features. These were just a few of the many [interesting facts](https://www.mepaul.com/wiki/10-background-facts-da-vinci-research-kit-dvrk) about the dVRK!

### Systems Engineering Takeaway
This repository stands as a case study in the power of modular digital system design. It demonstrates that a communications and computing architecture abstracted to handle arbitrary multi-axis distributed I/O over a unified serial interface can cleanly outlive its original physical form factor, scaling from a miniature articulated snake robot to one of the most visible research platforms in medical robotics history.
