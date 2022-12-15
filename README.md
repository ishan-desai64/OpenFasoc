### OpenFasoc

Fully Open-Source Autonomous SoC Synthesis using Customizable Cell-Based Synthesizable Analog Circuits The FASoC Program is focused on developing a complete system-on-chip (SoC) synthesis tool from user specification to GDSII with fully open-sourced tools.Fully Open-Source Autonomous SoC Synthesis using Customizable Cell-Based Synthesizable Analog Circuits The FASoC Program is focused on developing a complete system-on-chip (SoC) synthesis tool from user specification to GDSII with fully open-sourced tools.Fully Open-Source Autonomous SoC Synthesis using Customizable Cell-Based Synthesizable Analog Circuits The FASoC Program is focused on developing a complete system-on-chip (SoC) synthesis tool from user specification to GDSII with fully open-sourced tools.

## Prerequisites
Please build the following tools:

Magic https://github.com/RTimothyEdwards/magic

Netgen https://github.com/RTimothyEdwards/netgen

Klayout https://github.com/KLayout/klayout Please use this command to build preferably: ./build.sh -option '-j8' -noruby -without-qt-multimedia -without-qt-xml -without-qt-svg

Yosys https://github.com/The-OpenROAD-Project/yosys

OpenROAD https://github.com/The-OpenROAD-Project/OpenROAD (commid id: 7ff7171)

open_pdks https://github.com/RTimothyEdwards/open_pdks

open_pdks is required to run drc/lvs check and the simulations
After open_pdks is installed, please update the open_pdks key in common/platform_config.json with the installed path, down to the sky130A folder
Other notice:

Python 3.7 is used in this generator.
All the required tools need to be loaded into the environment before running this generator.

## Design Generation

# Temperature Sensor Generator
This page gives an overview of how the Temperature Sensor Generator (temp-sense-gen) works internally in OpenFASoC.

# Circuit
This generator creates a compact mixed-signal temperature sensor based on the topology from this paper <https://ieeexplore.ieee.org/document/9816083>.

It consists of a ring oscillator whose frequency is controlled by the voltage drop over a MOSFET operating in subthreshold regime, where its dependency on temperature is exponential.
![tempsense_ckt](https://user-images.githubusercontent.com/110079631/199317479-67f157c5-6934-470b-8552-5451b1361b9c.png)

Block diagram of the temperature sensor’s circuit

The physical implementation of the analog blocks in the circuit is done using two manually designed standard cells:

* HEADER cell, containing the transistors in subthreshold operation;
* SLC cell, containing the Split-Control Level Converter.

The gds and lef files of HEADER and SLC cells are pre-created before the start of the Generator flow.
The layout of the HEADER cell is shown below:

Block Diagram of the flow:
--------------------------

![photo_2022-11-02 10 43 24](https://user-images.githubusercontent.com/110079631/199403863-d02e56c8-40ad-417f-9002-d7130cf23770.jpeg)



### PLL Generator
This section will explain how PLL is generated using OpenFasoc.
In its most basic configuration, a phase-locked loop compares the phase of a reference signal  to the phase of an adjustable feedback signal.When the comparison is in steady-state, and the output frequency and phase are matched to the incoming frequency and phase of the error detector, we say that the PLL is locked.  

Mainly,there are four blocks in PLL
1) Phase Detector
2) Frequency Detector
3) Voltage Control Oscilator
4) Charge Pump

## 1) Phase Detector:
Phase detector produces two DC voltages namely UP and DOWN, which is proportional to the phase difference between the input signal(Vref) and feedback (output) signal(Vout). If the Vref phase is lagging with respect to Vout then UP signal remains high to the duration of their phase difference and the DOWN signal remains low. If the Vref phase is leading with respect to Vout then DOWN signal remains high to the duration of their phase difference and the UP signal remains low.Generally we create Phase detector using two negetive edge triggered D-fliplops and a AND gate.

https://user-images.githubusercontent.com/110079631/203073018-1c06be90-d496-4347-851c-4c3b80aae57c.png


