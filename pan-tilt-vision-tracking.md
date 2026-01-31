# Pan-Tilt Vision Tracking Platform

*Author: Percival Segui*  

*Prepared as an independent engineering project record documenting system architecture, implementation, and platform capability.*

---

This document presents the system architecture, design decisions, and engineering artifacts associated with the development of a pan–tilt vision tracking platform.

It is intended to preserve technical context, reasoning, and platform structure to support continued development, extension, and future reference.

---

### At a Glance 

**System Concept:** Embedded pan–tilt control platform $\rightarrow$ vision-based tracking (next integration step)  
**Plant:** 2-DOF pan–tilt gimbal (hobby servo actuation)  
**MCU/Sensing:** STM32 + MPU-6050 (gyro rate instrumentation)  

> This document is an engineering note for the current state of Project P1 as an in-progress embedded controls/mechatronics platform.  
> It is written to communicate what is implemented, what is validated on hardware, and what the platform is designed to support next (vision tracking).

---

## 1.0 Project Overview

Project P1 is an in-progress pan–tilt vision tracking platform built on electromechanical hardware and embedded firmware.

The core objective of the project is to develop a system capable of autonomous target tracking, where a perception system (camera or equivalent vision sensor) determines *where the platform should point*, and an embedded control system executes that motion safely, deterministically, and repeatably.

At its foundation, P1 is a physical system:

* a two-degree-of-freedom pan–tilt mechanism
* servo-based actuation
* an embedded real-time control platform
* supporting sensors used to improve motion behavior and robustness

The project is intentionally structured as a platform, not a single demonstration.
Its purpose is to support experimentation, integration, and extension of perception-driven control behaviors rather than to implement a fixed algorithm.

### Perception-driven control as the primary design goal

The defining characteristic of Project P1 is that perception drives control.

The system is designed such that:

* the perception layer (vision) defines pointing error or target offset
* the control layer translates that error into pan–tilt motion intent
* the actuation layer executes motion within known physical and safety limits

This architecture mirrors real robotic and mechatronic systems, where sensing, control, and actuation are intentionally separated to allow each layer to evolve independently.

While the vision system is the primary source of pointing information, the platform includes additional sensing and instrumentation to improve motion quality, system stability, and degraded-mode behavior.

### Role of supporting sensors

Supporting sensors - including inertial rate sensing - are incorporated not to define the tracking objective, but to improve how the system behaves while pursuing it.

These sensors contribute information about the platform’s own motion, enabling:

* smoother actuator response
* reduced oscillation and overshoot
* improved behavior during rapid slews
* controlled transitions when vision data is degraded or temporarily unavailable

Importantly, supporting sensors do not replace vision as the authoritative reference for where the system should point.

Vision remains the primary driver of system intent.

### Platform-first development approach

Project P1 follows a platform-first development philosophy.

Rather than beginning with a single end-to-end algorithm, the project focuses on building a reliable foundation consisting of:

* a physical mechanical plant
* deterministic real-time execution
* safe and bounded actuator control
* observability for tuning and diagnosis
* clearly defined subsystem boundaries

This approach ensures that future integration of vision-based tracking logic occurs on top of a stable, well-understood motion platform rather than being tightly coupled to early experimental code.

### Current project status

At its current stage, Project P1 has established the physical and embedded control foundation required for vision-based tracking.

The next phase of work focuses on integrating a vision system to generate target error inputs and exercising closed-loop tracking behaviors on the existing pan–tilt platform.

The project is actively in progress.

---

## 2.0 Design Philosophy

Project P1 is guided by a perception-first, platform-oriented design philosophy.

Rather than optimizing a single control law or pursuing a narrow algorithmic goal, the project emphasizes system structure, clear subsystem boundaries, and alignment between physical capability and control objectives.

This philosophy reflects how mechatronic systems are developed: by first building a reliable motion platform, and then layering increasingly intelligent behavior on top of it.

### Perception defines intent

In P1, intent originates from perception.

The role of the sensing layer is to answer the question:

> *Where should the system point right now?*

For the intended use case, that information is provided by a vision system in the form of target offset, centroid error, or pose estimate.

Vision therefore defines what the system is trying to do, not how it executes that motion.

This distinction prevents control logic from becoming entangled with sensing implementation and allows different perception strategies to be evaluated without restructuring the platform.

### Control translates intent into motion

The control layer exists to translate abstract intent into physical action.

Its responsibilities include:

* mapping error signals into pan–tilt motion commands
* shaping actuator response to respect physical limits
* enforcing safety and bounded behavior
* maintaining stable motion under disturbances

The control system does not own the target definition.
It owns motion quality, safety, and repeatability.

By separating these responsibilities, the platform avoids the common failure mode where control logic becomes tightly coupled to a specific sensor or experiment.

### Actuation is treated as a constrained physical interface

Actuators are treated as physical devices with finite authority, quantization, and dynamic limitations.

Rather than assuming ideal behavior, the platform explicitly acknowledges that:

* commanded motion is discrete
* response is not perfectly linear
* bandwidth is limited
* small-signal behavior may not be expressible

The firmware architecture is therefore designed to work *with* these constraints rather than attempt to mathematically eliminate them.

This design choice ensures that control behavior remains predictable and defensible when exercised on hardware.

### Supporting sensors improve behavior, not objectives

Supporting sensors - such as inertial rate sensing - are included to improve how the system moves, not what it tries to achieve.

These sensors contribute information about the platform’s own dynamics, enabling:

* damping of self-induced motion
* reduction of overshoot during slews
* smoother transitions between behaviors
* graceful handling of degraded perception

They do not redefine the tracking objective and are not used as primary references for pointing.

This separation avoids conceptual ambiguity and keeps system intent clearly owned by perception.

### Platform evolution over algorithm fixation

Project P1 intentionally prioritizes platform evolution over early algorithm fixation.

Rather than committing prematurely to a specific tracking method, filter structure, or estimation strategy, the project establishes a foundation that can support:

* multiple vision approaches
* different target representations
* varying levels of autonomy
* incremental increases in system intelligence

This approach reduces the risk of rework and ensures that future capability is added through integration rather than reinvention.

### Engineering clarity over visual novelty

A core principle throughout P1 is that visible behavior must be explainable.

The project avoids behaviors that “look impressive” but cannot be traced back to clear system structure or measurable signals.

Every motion should be attributable to:

* a defined input
* a known control pathway
* a bounded actuator response

This discipline is critical when transitioning from prototype behavior to reliable autonomous operation.

---

## 3.0 System Architecture Overview

At a high level, Project P1 follows a layered architecture:

```
Perception (Vision)
        ↓
Pointing Error / Intent
        ↓
Motion-Control Platform
        ↓
Actuator Conditioning
        ↓
Pan–Tilt Mechanical Plant
```

Supporting sensors provide additional information about system dynamics and feed into the control layer to improve motion quality and robustness.

This separation ensures that sensing, control, and actuation remain loosely coupled and independently evolvable.


---

## 4.0 Mechanical plant: what exists and is frozen

P1 is a 2-DOF pan-tilt mechanism.
- Roll is not part of the plant. (This is a mechanical truth, not a firmware limitation.)
- Pan and tilt axes are mechanically independent
- The structure is designed to support repeatable motion

The mechanical assembly is treated as a fixed boundary condition for control design.

The mechanical assembly is treated as “frozen enough” to move the project forward as a controls/tracking platform.

### CAD / mass properties (referenced artifacts)
The project includes CAD models and Fusion-derived mass properties that are used to reason about actuator authority, expected response, and disturbance sensitivity:
- Center of Mass (COM) for assemblies
- Moments of Inertia (MOI) about relevant axes
- Axis geometry and lever arms used for torque reasoning and expected disturbance sensitivity

- `[Image] Pan–tilt assembly - front/side/isometric`
- `[Table] Fusion mass properties: COM + MOI per assembly`
- `[Image] Coordinate frames / axis definitions (pan axis, tilt axis)`

---

## 5.0 Actuation subsystem (what is implemented)
Actuation is performed using PWM position commands to DS3218 servos driving the pan and tilt axes.

From the firmware configuration (snapshot in repo):
- PWM generation uses a hardware timer configured for servo rate:
  - TIM3 configured to 1 MHz tick (`Prescaler=83`) and 20 ms period (`Period=19999`) $\rightarrow$ 50 Hz servo frame
  - PWM outputs on TIM3 CH1 (pan) and TIM3 CH2 (tilt)

Design boundary (important): The servo inner loop is treated as a black box position device with known constraints:
- finite resolution
- limited bandwidth
- internal closed-loop behavior not directly accessible

 The platform owns:
- commanded PWM microseconds
- clamping, slew limiting, quantization
- deterministic update cadence
- safety semantics and observability (telemetry)

This approach ensures repeatable motion and avoids reliance on assumptions that cannot be validated experimentally.

---

## 6.0 Firmware Motion-Control Platform

The firmware provides a general-purpose motion-control platform responsible for converting abstract motion intent into safe actuator commands.

The firmware is intentionally designed as infrastructure rather than a fixed application. The firmware executes on hardware and is actively used for motion control, but is structured as reusable infrastructure rather than a single-purpose application.

### Core responsibilities

The firmware provides:

* deterministic real-time execution
* actuator command conditioning
* safety and bounded behavior
* observability for tuning and diagnosis

It is structured to accept motion intent from multiple upstream sources without architectural change.

### Control spine

At the center of the firmware is a control spine that:

* receives abstract error or motion intent
* applies shaping and constraints
* produces actuator commands within known limits

The control spine is agnostic to how intent is generated, allowing vision, inertial sensing, or test excitation to drive the same motion pipeline.

### Safety and observability

Safety semantics are treated as first-class design elements.

The firmware supports:

* defined behavior under degraded sensing
* prevention of runaway outputs
* graceful transitions between operating states

Instrumentation and telemetry exist to support engineering analysis rather than user-facing features.

### Publication status

The firmware is under active refactor and cleanup.

Public documentation therefore describes capability and architecture, not implementation details, ensuring long-term consistency as the codebase evolves.

---

## 7.0 Vision System - Primary Sensing Layer

Vision is the primary sensing modality for Project P1.

The vision system is responsible for determining:

* target location relative to the platform
* pointing error in pan and tilt
* confidence or validity of perception data

This information defines system intent.

Vision processing may be implemented externally or on a dedicated vision module, but in all cases the output presented to the motion-control platform is a well-defined error signal.

The control platform does not depend on the internal details of vision processing.

---

## 8.0 IMU Subsystem - Role in a Vision-Based Tracking Platform

The pan–tilt platform includes an inertial measurement unit (IMU) providing angular rate information.
The IMU is not used to define the primary control objective of the system, and it is not intended to perform inertial stabilization of the gimbal.

Instead, the IMU exists to provide rate information that is fundamentally unavailable from vision alone, and to improve motion quality and robustness of the tracking system.

### IMU as a supporting sensor, not a primary reference

In the P1 architecture, vision defines *where* the system should point.

The IMU does not define target orientation, absolute pose, or pointing reference.
Those quantities originate exclusively from the vision system (e.g., pixel error, centroid offset, or marker pose).

The IMU plays a different role:

> It provides real-time angular rate information describing how the platform is currently moving.

This distinction is critical.

Vision sensors are well suited for estimating position error, but they are inherently limited in their ability to provide clean, low-latency rate information.

---

### 8.1 Why vision alone is insufficient for motion control

A vision pipeline introduces several unavoidable characteristics:

- discrete frames
- non-zero processing latency
- variable computation time
- dropped or delayed frames
- exposure-dependent noise

As a result, vision measurements are best treated as low-bandwidth positional guidance, not as a high-bandwidth motion signal.

In contrast, an IMU provides:

- continuous angular rate measurement
- very low latency
- high update rate
- deterministic timing
- independence from lighting or visual features

These two sensing modalities are complementary rather than redundant.

| Quantity                   | Vision              | IMU                 |
| -------------------------- | ------------------- | ------------------- |
| Direction / pointing error | ✓                   | ✗                   |
| Angular rate               | ✗                   | ✓                   |
| Latency                    | high / variable     | low / deterministic |
| Update rate                | frame-based         | continuous          |
| Failure mode               | occlusion, lighting | electrical / comms  |

Because of this separation, vision and inertial sensing naturally occupy different roles in the control architecture.

---

### 8.2 IMU contribution to motion quality

During tracking, the pan–tilt system generates motion through discrete servo commands.
This motion introduces secondary effects that vision alone cannot observe cleanly in real time:

- quantized step motion
- backlash and gear compliance
- inertial coupling between pan and tilt axes
- cable forces and external disturbances
- oscillatory response following large command changes

The IMU directly measures the platform’s angular response to these effects.

This enables the control system to:

- damp self-induced oscillations
- limit overshoot during rapid slews
- prevent chatter near the target
- shape motion to appear smooth and intentional

In this role, the IMU improves motion behavior, not pointing accuracy.

This distinction is intentional.

---

### 8.3 Robust behavior under vision degradation

Vision-based systems do not fail catastrophically; they degrade temporarily.

Common events include:

- partial occlusion
- glare or saturation
- motion blur
- momentary loss of target
- dropped frames

During these intervals, relying exclusively on vision can result in abrupt or unstable behavior.

The IMU provides a means to maintain controlled motion during such transitions:

- continued damping of motion
- prevention of runaway actuator commands
- smooth deceleration when vision input disappears
- graceful handoff between tracking and search/reacquisition states

This capability significantly improves system robustness and perceived intelligence.

---

### 8.4 What the IMU is deliberately *not* used for

The IMU is not used to:

- perform inertial stabilization as a primary objective
- estimate absolute orientation
- compensate gravity
- replace vision as the pointing reference
- implement full sensor fusion or state estimation

Those functions are intentionally excluded because they do not align with the physical capabilities of the plant or the project goals.

The IMU exists to support tracking - not to redefine it.

---

### 8.5 Summary of IMU role

In Project P1:

- Vision determines where to point
- The IMU informs how the system is moving
- The control system uses both to produce stable, well-behaved motion

This separation preserves clarity of purpose, avoids unnecessary complexity, and aligns the sensing strategy with what each sensor is fundamentally good at.

---

## 9.0 Robustness and Degraded-Mode Behavior

Project P1 is designed to behave predictably under non-ideal conditions.

Examples include:

* temporary loss of vision input
* degraded confidence in perception data
* disturbances introduced by motion or external interaction

Supporting sensors and safety semantics allow the system to:

* avoid abrupt or unstable motion
* maintain bounded actuator behavior
* transition cleanly between tracking and recovery states

This emphasis on robustness is central to the platform’s design.

---

## 10.0 Control and Output Conditioning Layer

Project P1 includes a dedicated control and output conditioning layer designed around the physical realities of the pan–tilt actuators.

This layer exists to ensure that motion behavior remains safe, bounded, and repeatable, independent of how motion intent is generated.

The control platform treats upstream inputs - whether originating from vision-based tracking, inertial sensing, or test excitation - as abstract motion intent rather than as tightly coupled control laws.

### 10.1 Motion-intent shaping capabilities

The platform provides mechanisms to shape and condition motion intent before it is applied to the actuators, including:

* rate-based shaping using angular velocity information (used during platform bring-up and motion characterization)
* optional signal smoothing to reduce high-frequency excitation
* deadband and hysteresis mechanisms to prevent limit-cycle chatter
* command-center deadband to stabilize behavior near target
* configurable slew-rate limiting
* explicit command quantization handling

These mechanisms allow the system to produce controlled and intentional motion despite actuator resolution limits and non-ideal dynamics.

Importantly, these behaviors are capabilities of the platform, not fixed algorithms.

They may be enabled, disabled, or reparameterized depending on the active operating mode (e.g., tracking, search, characterization).

### 10.2 Output gating and determinism

All actuator commands pass through an output gating stage prior to hardware update.

This stage enforces:

* deterministic update timing
* minimum effective command resolution
* suppression of micro-updates that would not produce physical motion
* explicit observability of applied actuator commands

This ensures that actuator behavior is predictable, measurable, and free from unnecessary chatter.

### 10.3 Architectural significance

A key design principle of Project P1 is that output safety and determinism are independent of the upstream error source.

Whether motion intent is produced by:

* vision-derived target error
* inertial-rate feedback used for motion-quality management
* synthetic excitation during characterization

the same output conditioning, clamping, and observability mechanisms remain in effect.

This separation allows perception and control strategies to evolve without compromising actuator safety or motion predictability.

---

## 11.0 Telemetry and Observability

Project P1 is instrumented to support repeatable engineering analysis and tuning.

The observability layer exists to expose system behavior in a form suitable for debugging, characterization, and performance evaluation.

Telemetry capabilities include:

* reporting of motion intent and conditioned output
* visibility into angular-rate measurements
* monitoring of sensor validity and failure states
* readback of hardware-applied actuator commands

Telemetry is structured to support offline analysis and correlation with experimental data rather than serving as a user-interface feature.

This emphasis on observability allows system behavior to be explained, measured, and refined systematically.

---

## 12.0 Characterization and Excitation Capabilities

The platform includes mechanisms to support controlled characterization of the mechanical plant and actuators.

These capabilities are used to establish operational boundaries and to inform safe control behavior.

Supported characterization functions include:

* deterministic step and sweep excitation
* controlled sequencing of motion patterns
* detection of actuator saturation and boundary conditions
* abort and recovery behavior during characterization

These tools allow the platform to be measured and bounded deliberately, rather than tuned by trial-and-error alone.

The resulting understanding of actuator limits, response characteristics, and effective resolution directly supports reliable vision-based tracking behavior.

---

## 13.0 Platform Capability Inventory

*This inventory reflects implemented and validated platform capabilities, not final application behavior.*

At its current stage of development, Project P1 provides the following platform-level capabilities:

### 13.1 Mechanical and Actuation

* two-degree-of-freedom pan–tilt mechanical plant
* servo-based actuation with bounded command interfaces
* explicit handling of actuator resolution and limits

### 13.2 Real-Time Execution

* deterministic timer-driven control execution
* separation of real-time control from non-real-time tasks

### 13.3 Sensing and Safety

* inertial rate sensing used for motion-quality management
* explicit detection of degraded sensing conditions
* defined safe behavior under sensor failure

### 13.4 Motion Conditioning

* shaping of motion intent prior to actuation
* deadband, slew limiting, and quantization handling
* deterministic actuator update semantics

### 13.5 Observability

* structured telemetry supporting analysis and tuning
* visibility into both commanded and applied actuator outputs

This inventory intentionally describes platform capability, not final application behavior.

It establishes that the mechanical plant, timing infrastructure, safety semantics, and observability required for vision-based tracking are real and operational.

---

## 14.0 Why the Platform Supports Vision-Based Tracking

Vision-based tracking places specific demands on the embedded back end:

* deterministic actuator update cadence
* bounded and predictable motion response
* safe behavior when perception degrades
* observability for tuning and validation

Project P1 already provides this foundation.

When vision is integrated, the perception system becomes the upstream generator of motion intent, while the existing control and output-conditioning layers ensure:

* safe motion
* repeatable behavior
* measurable performance

This allows vision-based tracking behavior to be added without restructuring the underlying motion-control platform.

---

## 15.0 Current Status and Next Integration Steps

The immediate next steps include:

* integration of a vision sensing system
* generation of pan–tilt error signals from perception
* closed-loop tracking experiments
* implementation of search and reacquisition behaviors

The platform is intentionally structured to support these additions without architectural redesign.

---

### Status

Project P1 is an active engineering effort focused on the development of a pan–tilt vision tracking platform.

The current work establishes the mechanical, actuation, sensing, and motion-control infrastructure required for perception-driven tracking behavior.

Subsequent work will integrate a vision system as the primary source of motion intent and exercise closed-loop tracking on the existing platform.

---
