# 🐍 From Snake Robots to Surgical Standards: The Architectural Origins of the da Vinci Research Kit

This repository preserves the original hardware control architecture for the [da Vinci Research Kit (dVRK)](https://www.intuitive-foundation.org/dvrk/), designed as part of my graduate research at Johns Hopkins University. The codebase has since moved to JHU's [`mechatronics-firmware`](https://github.com/jhu-cisst/mechatronics-firmware) repository, with the final commit here flowing into the initial commit [(`a38917c`)](https://github.com/jhu-cisst/mechatronics-firmware/tree/a38917c2ce4880597da052d8d369eaeaf47659b4) there. This control architecture came from a completely separate robotic system: **a high-degree-of-freedom surgical snake robot.**

### 1. The Snake Robot Cabling Crunch

<p align="center"><img width="438" height="328" alt="Under a conventional control architecture, robot joints are wired directly to the workstation bus for processing, resulting in a complex cabling scenario." src="https://github.com/user-attachments/assets/4927e4fb-b4fe-4480-857b-854357ff21d5" /> <img height="328" alt="The original snake robot actuation unit also suffered from cable drag." src="https://github.com/user-attachments/assets/bc741eae-0082-45da-822e-0fd8371acf59" /></p>

During my early days in the LCSR, the lab was making strides in dexterity with a snake robot designed for remote telesurgery of the upper airway. However, standard control architectures involved running bulky wiring bundles from every control axis to a centralized electronics rack. The dozens of sensors and actuators needed to articulate such a high-dof robot created a physical bottleneck. The system became increasingly error prone and difficult to work with.

To solve this, we designed a [hardware controller architecture](https://www.researchgate.net/publication/224353229_Centralized_processing_and_distributed_IO_for_robot_control) optimized for high-degree-of-freedom systems:
* **Centralized Processing, Distributed I/O:** Rather than routing bundles of wires from each joint to the workstation individually, sensor and actuator signals are digitized and serialized by a nearby FPGA.
* **The FireWire Bus:** The motor commands and sensor readings are then multiplexed over a single, high-speed IEEE 1394 (FireWire) serial bus, maintaining real-time performance.
* **The Codebase:** The implementation of this architecture for the snake robot became the [`SnakeFPGA-rev2`](https://github.com/deepsurgical/SnakeFPGA-rev2) codebase.

To top it all off (so to speak), we built a next-gen snake robot for the new controller and got [exciting results](https://www.researchgate.net/publication/254025466_Design_of_a_scalable_real-time_robot_controller_and_application_to_a_dexterous_manipulator).
<p align="center"><img height="583" alt="We built a next-gen snake robot for the new controller." src="https://github.com/user-attachments/assets/f4c8e592-6f26-42aa-ae15-6304806d144b" /></p>

### 2. Porting from the Snake to the da Vinci
When Intuitive Surgical later provided retired first-generation da Vinci systems for research, labs faced a similar engineering hurdle: The donated manipulators needed new motion control electronics. Because the communication protocols and packet definitions implemented in `SnakeFPGA-rev2` were decoupled from robot kinematics, porting it to the da Vinci interface boards was relatively seamless.

<p align="center"><img height="218" alt="First-generation da Vinci Research Kit controller pictured on the left a year before launch, next to its predecessor the snake robot controller on the right" src="https://github.com/user-attachments/assets/ee1c27f2-e760-4bae-828f-442b6b8efd25" /> <img height="218" alt="The da Vinci Research Kit and snake robot controllers were developed in parallel so changes in one could be tested on the other immediately. This was at an advanced stage where robotic mechanisms and motors were being integrated and it was getting real." src="https://github.com/user-attachments/assets/01a7c7e6-b790-4ac5-a861-352e2342024d" />
</p>

This repository was created to port the Altera-based snake robot firmware to the Xilinx-based da Vinci Research Kit. Development on the two controllers continued in parallel to ensure consistency and generalizability. The generalized multi-axis data distribution design allowed the first functional systems to be brought up in no time.

[First known recorded power-on of the da Vinci Research Kit (dVRK) motion controller](https://github.com/user-attachments/assets/3037465c-1144-416a-8ffe-dfd9f047238d)

### 🔍 The Architectural Migration

We can retrace the steps leading from this then-internal `daVinci1394FPGA` codebase to its eventual open source release as `mechatronics-firmware` on GitHub.

#### History Hiding in the Headers
Like a primitive cave painting, there's an old school migration log etched in the header comments of the official [FireWire.v source code](https://github.com/jhu-cisst/mechatronics-firmware/blob/main/FPGA1394_QLA/Verilog/FireWire.v), showing how it was initially written for the snake robot in 2008, ported to the da Vinci about 2.5 years later in 2010, and received sporadic updates until moving to GitHub in Sep 2012:

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

#### Adding Another Axis as an Afterthought
Mechanically adding a robot axis might require a complete redesign, but adding it to the controller can be as simple as... adding. This [prehistoric commit](https://github.com/jhu-cisst/mechatronics-firmware/commit/b6e0830884eb06d80bc08d6f908d80e43b5e1405) shows another step in 'transforming' the snake robot into the dVRK:

> Paul Thienphrapa, Oct 16, 2012, FPGA1394_QLA: Increase maximum number of axes from 7 to 8

The fact that snake robot had 7 control axes while the dVRK controller had 8 was hardcoded into the firmware in `SnakeFPGA-rev2` and `daVinci1394FPGA` respectively.

#### Intuitive Research Kit
Before the open-source rollout and rebrand to the _da Vinci Research Kit (dVRK)_, the platform was provisionally referred to as the _Intuitive Research Kit_. The name can still be found in the [dVRK GitHub repository](https://github.com/jhu-dvrk/sawIntuitiveResearchKit) and in some [transient documents](http://jhir.library.jhu.edu/handle/1774.2/37924) ([pdf](https://rose.mepaul.com/w/images/b/b1/Pault_thesis-136-final.pdf#page=218)):
> _Besides enabling use of the Snake Robot (Section 3.4.2), the outcomes of this effort formed the basis for JHU Open Source Mechatronics [155, 156], which publicly hosts a set of electronics design files, FPGA code, and basic software for a FireWire-based motion controller. This in turn is a component of the **Intuitive Research Kit** [157,158]._

<!--
#### Summary of the dVRK's Timeline Leading up to its Launch
* **Apr 2008:** Hardware control architecture implemented for the snake robot: `SnakeFPGA-rev2`
* **Oct 2010:** Snake robot architecture ported to the dVRK: `daVinci1394FPGA` (Altera → Xilinx FPGA)
* **Nov 2011:** Both codebases updated and maintained in parallel
* **Sep 2012:** `daVinci1394FPGA` released as open source to GitHub as `mechatronics-firmware`
* **Oct 2012:** Expanded 7-axis snake robot to 8-axis da Vinci Research Kit (dVRK)
* **Present:** Legacy name remains in some repositories: `sawIntuitiveResearchKit`
-->

#### Designed for Growth
What started off as a modest set of .v files has expanded to support a wide array of hardware variants and technical features. These are just a few of the many [interesting facts](https://www.mepaul.com/wiki/10-background-facts-da-vinci-research-kit-dvrk) about the dVRK!

### Systems Engineering Takeaway
This repository highlights the power of modular digital system design. A hardware control architecture abstracted enough to handle multi-axis distributed I/O over a high-speed serial link can easily outlast its physical form, jumping from a miniature articulated snake robot to a medical robotics research platform found in labs all around the world.
