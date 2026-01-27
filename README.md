# Pan–Tilt Vision Tracking Platform

**Project Status:** IN PROGRESS

This repository contains an engineering project record and supporting artifacts for a perception-driven pan–tilt motion platform.

The platform is developed as an embedded controls and mechatronics system intended to support vision-based tracking and related perception-driven behaviors.

---

## Overview

The project implements a layered architecture in which:

* perception defines pointing intent
* the embedded control system translates intent into motion
* actuator conditioning enforces bounded and repeatable behavior

Development prioritizes physical integration, system behavior, and architectural clarity over early publication of implementation details.

---

## Engineering Record

`pan-tilt-vision-tracking.md`

The engineering record documents the stabilized understanding of the platform, including:

* mechanical plant definition and axis structure
* actuation constraints and command semantics
* sensing roles and signal ownership
* motion conditioning, safety, and observability
* system-level architectural boundaries

The document reflects the current engineering model of the system rather than a complete snapshot of all implementation artifacts.

---

## Repository Contents

This repository includes supporting engineering materials such as:

* mechanical CAD models and assemblies
* center-of-mass and moment-of-inertia data
* part- and assembly-level properties derived from CAD
* supporting figures and diagrams referenced in the engineering record

These artifacts are published as the physical platform and system understanding stabilize.

---

## Firmware Status

The embedded firmware is undergoing active architectural evolution and refactoring.

To avoid publishing transient structure or misleading implementation details, firmware sources will be added once the control architecture and execution model have stabilized.

---

## Current Status

The mechanical platform and motion-control foundation are established.

Ongoing work focuses on continued system integration and preparation for perception-driven tracking experiments.

---

*Author: Percival Segui*
*Independent engineering project record*
