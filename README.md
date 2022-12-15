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

# 1) Temperature Sensor Generator
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



### 2) PLL Generator
This section will explain how PLL is generated using OpenFasoc.
In its most basic configuration, a phase-locked loop compares the phase of a reference signal  to the phase of an adjustable feedback signal.When the comparison is in steady-state, and the output frequency and phase are matched to the incoming frequency and phase of the error detector, we say that the PLL is locked.  

Mainly,there are four blocks in PLL
1) Phase Detector
2) Frequency Detector
3) Voltage Control Oscilator
4) Charge Pump

## 1) Phase Detector:
Phase detector produces two DC voltages namely UP and DOWN, which is proportional to the phase difference between the input signal(Vref) and feedback (output) signal(Vout). If the Vref phase is lagging with respect to Vout then UP signal remains high to the duration of their phase difference and the DOWN signal remains low. If the Vref phase is leading with respect to Vout then DOWN signal remains high to the duration of their phase difference and the UP signal remains low.Generally we create Phase detector using two negetive edge triggered D-fliplops and a AND gate.

                
<img width="445" alt="Screenshot 2022-11-21 at 7 25 32 PM" src="https://user-images.githubusercontent.com/110079631/203073018-1c06be90-d496-4347-851c-4c3b80aae57c.png">

<img width="589" alt="Screenshot 2022-11-21 at 7 26 24 PM" src="https://user-images.githubusercontent.com/110079631/203073555-9091d92f-1198-4d12-86f2-4ea532928ddf.png">

*2. Charge Pump:-**
                Charge Pump is used to convert the digital measure of the phase difference done in the Phase Detector into a analog control signal, which is used to control the Voltage Controlled Oscillator in the next stage.
                In the construction of the Charge Pump we use a current steering model which makes it a Analog block. Here when the UP signal is high the current flows from Vdd to output which charges the load capacitance. When the DOWN signal is high the current flows from load capacitance to ground which is discharging of the current.

<img width="336" alt="Screenshot 2022-11-21 at 5 43 39 PM" src="https://user-images.githubusercontent.com/110079631/203073124-c2d463ae-6064-4208-9792-2ce74ebab4af.png">

```
Avg Active time of UP   > Avg Active time of DOWN = Charging of Capacitance     [0-->1]--> Speeds up the VCO
Avg Active time of DOWN > Avg Active time of UP   = Dis-Charging of Capacitance [1-->0]--> Slows down the VCO
```

<img width="346" alt="Screenshot 2022-11-21 at 5 41 39 PM" src="https://user-images.githubusercontent.com/110079631/203073306-3ec3e940-5bbc-4895-82d6-011fe26713cf.png">

**3. Voltage Controlled Oscillator:-**
                  The Output of the Charge Pump acts as a Control signal to the Voltage Controlled Oscillator.The VCO generates a DC signal, the amplitude of which is proportional to the amplitude of output of Charge Pump Control Signal. Here the adjustment in the output frequency/phase of VCO is made until it shows equivalency with the input signal frequency/phase. 
                  The VCO is contructed using two current mirrors and a ring Oscillator which makes it a Analog block. The control signal is used as an input to these current source(mirrors) to control the current supply which in turn control the delay of the circuit. By controlling the delay we are basically controlling the frequency of the Oscillator which makes it frequency flexible.
    
<img width="568" alt="Screenshot 2022-11-21 at 7 30 50 PM" src="https://user-images.githubusercontent.com/110079631/203073888-bce2a8db-3aff-4842-a218-147715ec789a.png">

**4. Frequency Divider:-**
                  Frequency Divider is used to divide the frequency which is otherwise a multiplier in time of the Output voltage from the Voltage Controlled Oscillator and is feedback as an input to the Phase Detector, which is then compared with the Vref input signal in the Phase Detector stage.
                  The Frequency Divider is constructed using a series of Toggle Flip-Flops, which makes it a complete Digital block.

<img width="394" alt="Screenshot 2022-11-21 at 7 26 57 PM" src="https://user-images.githubusercontent.com/110079631/203073982-c36fa793-bf28-4de6-9e77-74e61333a34f.png">

## Phase Detector(PFD)
Phase Detector is AuxCell.To generate its verilog we have to follow few steps.
To generate first git clone the repositary and copy PLL-Gen and paste it to generator in openfasoc tool.
```
https://github.com/ishan-desai64/OpenFasoc.git
```
After that write this command in PLL-Gen folder.
```
make sky130hd_pll_verilog
```
![OpenfasocResult1](https://user-images.githubusercontent.com/70513539/207919983-dae71641-4a83-4218-8565-29aba650984d.png)

After Generating Verilog file I have write testbench and got the follwing result.

![Result2](https://user-images.githubusercontent.com/70513539/207920191-98f71507-adbc-419e-aba7-58a42f8452e4.png)



### Author
 - **Ishan Desai**

### Contributors
 - Ishan Desai` 1`  1q


