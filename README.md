# 🐍 From Snake Robots to Surgical Standards: The Architectural Origins of the da Vinci Research Kit

This repository preserves the original hardware control architecture for the [da Vinci Research Kit (dVRK)](https://www.intuitive-foundation.org/dvrk/), designed as part of my graduate research at Johns Hopkins University. The codebase has since moved to JHU's [`mechatronics-firmware`](https://github.com/jhu-cisst/mechatronics-firmware) repository, with the final commit here leading into the initial commit [(`a38917c`)](https://github.com/jhu-cisst/mechatronics-firmware/tree/a38917c2ce4880597da052d8d369eaeaf47659b4) there. This control architecture came from a completely separate robotic system: **a high-degree-of-freedom surgical snake robot.**

### 1. The Snake Robot Cabling Crucnh

<p align="center"><img width="438" height="328" alt="Under a conventional control architecture, robot joints are wired directly to the workstation bus for processing, resulting in a complex cabling scenario." src="https://github.com/user-attachments/assets/4927e4fb-b4fe-4480-857b-854357ff21d5" /> <img height="328" alt="The original snake robot actuation unit also suffered from cable drag." src="https://github.com/user-attachments/assets/bc741eae-0082-45da-822e-0fd8371acf59" /></p>

During my early days in the LCSR, the lab was making strides in dexterity with a snake robot designed for remote telesurgery of the upper airway. However, standard control architectures involved running bulky wiring bundles from every control axis to a centralized electronics rack. The dozens of sensors and actuators needed to articulate such a high-dof robot created a physical bottleneck. The system became increasingly error prone and difficult to work on.

To solve this, we designed a [hardware controller architecture](https://www.researchgate.net/publication/224353229_Centralized_processing_and_distributed_IO_for_robot_control) optimized for high-degree-of-freedom systems:
* **Centralized Processing, Distributed I/O:** Rather than routing a bundles of wires from all joints to the workstation individually, the sensor and actuator signals were digitized via a nearby FPGA and serialized for transfer.
* **The FireWire Bus:** Real-time, low-latency motor commands and sensor readings were multiplexed over a single, high-speed IEEE 1394 (FireWire) serial bus.
* **The Codebase:** This implementation of the architecture for the snake robot became the codebase [`SnakeFPGA-rev2`](https://github.com/deepsurgical/SnakeFPGA-rev2).

To top it all off (so to speak), we built a next-gen snake robot for the new controller and got [exciting results](https://www.researchgate.net/publication/254025466_Design_of_a_scalable_real-time_robot_controller_and_application_to_a_dexterous_manipulator).
<p align="center"><img height="583" alt="We built a next-gen snake robot for the new controller." src="https://github.com/user-attachments/assets/f4c8e592-6f26-42aa-ae15-6304806d144b" /></p>

### 2. Porting it to the da Vinci Robot
When Intuitive Surgical later provided retired first-generation da Vinci systems for research, labs faced a similar engineering hurdle. The da Vinci manipulators needed a new, high-bandwidth, low-latency motion control interface. Because the underlying FireWire communication protocols and packet definitions of `SnakeFPGA-rev2` were decoupled from robot kinematics, the snake robot controller was a perfect match for the da Vinci interface boards (the Quad Linear Amplifiers or QLAs).

<p align="center"><img height="218" alt="First-generation da Vinci Research Kit controller pictured on the left a year before launch, next to its predecessor the snake robot controller on the right" src="https://github.com/user-attachments/assets/ee1c27f2-e760-4bae-828f-442b6b8efd25" /> <img height="218" alt="The da Vinci Research Kit and snake robot controllers were developed in parallel so changes in one could be tested on the other immediately. This was at an advanced stage where robotic mechanisms and motors were being integrated and it was getting real." src="https://github.com/user-attachments/assets/01a7c7e6-b790-4ac5-a861-352e2342024d" />
</p>

This repository was thus created to port the Altera-based snake robot firmware to the new Xilinx-based da Vinci hardware. Development on the two controllers continued in parallel to ensure consistency and generalizability. Since the architecture was designed with generic, multi-axis data distribution in mind, the adaptation was seamless and the first functional systems were brought up in no time.

[First known recorded power-on of the da Vinci Research Kit (dVRK) motion controller](https://github.com/user-attachments/assets/3037465c-1144-416a-8ffe-dfd9f047238d)

### 🔍 The Architectural Lineage

`daVinci1394FPGA` was eventually released from our internal university servers as open source to GitHub. The exact steps the that architecture took along the way can be retraced through the file comments and other clues.

#### Intuitive Research Kit
Before the open-source rollout and rebrand to the _da Vinci Research Kit (dVRK)_, the platform was provisionally referred to as the _Intuitive Research Kit_ reflecting its initial donation status. The name persists in the [dVRK GitHub repository](https://github.com/jhu-dvrk/sawIntuitiveResearchKit) as well as in my [PhD thesis](http://jhir.library.jhu.edu/handle/1774.2/37924) ([pdf](https://rose.mepaul.com/w/images/b/b1/Pault_thesis-136-final.pdf#page=218)):
> _Besides enabling use of the Snake Robot (Section 3.4.2), the outcomes of this effort formed the basis for JHU Open Source Mechatronics [155, 156], which publicly hosts a set of electronics design files, FPGA code, and basic software for a FireWire-based motion controller. This in turn is a component of the **Intuitive Research Kit** [157,158]._

#### History Hiding in the Headers
The [official FireWire.v source code](https://github.com/jhu-cisst/mechatronics-firmware/blob/main/FPGA1394_QLA/Verilog/FireWire.v) contains a migration log etched in the header comments. The it shows that file was initially written for the snake robot in 2008, ported to the da Vinci about 2.5 years later in 2010, then received sporadic updates before moving to GitHub in Sep 2012:

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
Later artifacts of this porting sequence show up in this [prehistoric commit](https://github.com/jhu-cisst/mechatronics-firmware/commit/b6e0830884eb06d80bc08d6f908d80e43b5e1405) to the official repository. A simple refactor shows the physical translation between the two platforms:

> Paul Thienphrapa, Oct 16, 2012, FPGA1394_QLA: Increase maximum number of axes from 7 to 8

* **The Refactor:** Modifying the hardcoded system parameters to increase the number of supported axes from 7 to 8.
* **The Adaptation:** The snake robot had only seven (7) control axes in `SnakeFPGA-rev2`, while the corresponding dVRK controller was designed for 8.

#### The dVRK's Timeline
* **Apr 2008:** Hardware control architecture implemented for the snake robot: `SnakeFPGA-rev2`
* **Oct 2010:** Snake robot architecture ported to the dVRK: `daVinci1394FPGA` (Altera → Xilinx FPGA)
* **Nov 2011:** Both codebases updated and maintained in parallel
* **Sep 2012:** `daVinci1394FPGA` released as open source to GitHub as `mechatronics-firmware`
* **Oct 2012:** Expanded 7-axis snake robot to 8-axis da Vinci Research Kit (dVRK)
* **Present:** Legacy name remains in some repositories: `sawIntuitiveResearchKit`

#### Designed for Growth
What started off as a modest set of .v files has expanded to support a wide array of hardware variants and technical features. These are just a few of the many [interesting facts](https://www.mepaul.com/wiki/10-background-facts-da-vinci-research-kit-dvrk) about the dVRK!

### Systems Engineering Takeaway
This repository highlights the power of modular digital system design. A hardware control architecture abstracted enough to handle multi-axis distributed I/O over a unified serial interface can easily outlive its original physical form factor, jumping from a miniature articulated snake robot to one of the most popular medical robotics research platforms.
