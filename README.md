# VarunPLL

# Phase-Locked-Loop (PLL) -Design-using-SKY130nm-Technology

2-Day Workshop, 31st July 2021 and 1st August 2021

# Overview

The two day workshop on designing Phase Locked Loop (PLL) using SKY130nm Technology comprised of the following things:

Day 1 : Basics of PLL and lab setup

Day 2 : PLL simulation (Prelayout and Postlayout)

# Table of Contents
- [DAY 1 - Basics of PLL and Steps for lab setup](#day-1---basics-of-pll-and-steps-for-lab-setup)
  * [Basics of PLL](#basics-of-pll)
    + [PLL](#pll)
    + [Important terms used in PLL](#important-terms-used-in-pll)
  * [Lab setup](#lab-setup)
    + [Using tools](#using-tools)
    + [Development Flow](#development-flow)
    + [Ngspice setup](#ngspice-setup)
    + [Magic setup](#magic-setup)
  * [PLL Specifications](#pll-specifications)
-  [DAY 2 : PLL simulation (Prelayout and Postlayout)](https://github.com/Varun4498/VarunPLL/edit/main/READ.md#-PLL-simulation-(-Prelayout-and-Postlayout-)-)
    * [Pre-layout simulations](#pre-layout-simulations)
    + [Phase Frequency Detector](#phase-frequency-detector)
    + [Charge Pump](#charge-pump)
    + [VCO](#vco)
    + [Frequency Divider](#frequency-divider)
    + [PLL](#pll-1)
  * [Troubleshooting steps](#troubleshooting-steps)
  * [Layout](#layout)
    + [Phase Frequency Detector](#phase-frequency-detector-1)
    + [Charge Pump](#charge-pump-1)
    + [VCO](#vco-1)
    + [Frequency Divider](#frequency-divider-1)
    + [MUX](#mux)
    + [Integrated PLL](#integrated-pll)
  * [Post-layout Simulation](#post-layout-simulation)
    + [Phase Frequency Detector](#phase-frequency-detector-2)
    + [Charge Pump](#charge-pump-2)
    + [VCO](#vco-2)
    + [Frequency Divider](#frequency-divider-2)
    + [PLL](#pll-2)
  * [About Tapeout](#about-tapeout)
- [Acknowledgement](#acknowledgement)




# DAY 1 - Basics of PLL and Steps for lab setup

The day 1 covers the basics of PLL and steps to install ngspice and magic for designing of PLL.

## Basics of PLL

### PLL

Phase locked loop is a feedback system which generates a high frequency, low noise clock.

Crystal Oscillators and Voltage controlled oscillators are the two ways to generate the clock signal. 

Crystal oscillators are the purest form of clock with excellent phase noise. The drawback of crystal oscillators is that it has very low frequency and have very limited tuning range. Hence, Voltage controlled oscillators (VCOs) are used which can generate high frequency clock, but it has poor phase noise. To get a low phase noise, high frequency clock, we use phase locked loop which makes the VCO mimic the crystal to achieve low phase noise, while still getting high frequency clock.

Phase noise is the random fluctuation of the zero crossing of a periodic waveform , that is, untimely arrival of data. 

The block diagram of PLL is as shown below:

![Digital PLL]()

PLL has five blocks: 

1. Phase Frequency Detector (PFD) 

It detects the phase (and frequency) error between the input reference clock IN (that is, crystal oscillator) and feedback clock FB. It generates two output signals, UP and DN, whose pulse width is proportional to the phase error.

For, detecting the phase error between the input reference clock and the feedback clock, we donot use XOR phase detector as it doesnt correct the frequency errors. Instead, we use Phase Frequency Detector , which will correct both, phase and frequency differences between the input reference clock and the feedback clock.

The basic block diagram of PFD is as shown below:

![PFD_block]()

When falling edge of reference clock leads feedback clock, the UP signal goes high and DN signal remains low. UP signal remains high until it detects the falling edge of the feedback clock. As soon as feedback clock falling edge arrives, UP signal goes low. 

When falling edge of feedback clock leads reference clock, the DN signal goes high and UP signal remains low. DN signal remains high until it detects the falling edge of the reference clock. As soon as reference clock falling edge arrives, DN signal goes low. 

Dead zone is one of the problem of PFD. The phase difference below which PFD output is not able to reach the desired logical level and fails to turn on the charge pump switches is called the dead zone. Above precision PFDs are present, which overcomes this issues.

2. Charge Pump (CP)

The loop filer takes input as voltage/current. Hence, the pulse width modulates signal at the output of PFD have to be transformed to volateg/current. Therefore, we use charge pump circuit. 

It generates the error current (ICP) which is proportional to the phase difference od pulse width between UP and DN which inturn is proportional to the input phase error. 

  Charge pump chagres or discharges the loop filter based upon the UP and DN signals. If UP signal is high, charge pump charges the loop filter. If DN signal is high, charge pump discharges the loop filter.
  
  Charge pump circuit have of charge leakage, which should be reduced as much as possible.

3. Loop Filter (LF)

It converts the error current into error voltage. It also controls the loop dynamics and stability of the system. 

For the stability purpose, C2 capacitor should be one-tenth of the C1 capacitor. Loop filter bandwidth should be one-tenth of the highes output frequency.
4. Voltage Controlled Oscillator (VCO)

VCO is the heart of PLL and it generates the desired high frequency clock output. 

The most common on-chip VCO used is the ring oscillators which consists of odd number of inverters. The period will be twice the number of inverter delays multiplied be the inverter delay. One of the way, to control the frequency of the ring oscillator is to vary the current using the current starving technique. It is important to note that, we should design the VCO is such a way that the range of frequencies we want for the PLL is generated properly by the VCO.

5. Frequency Divider (N)

It divides the VCO clock to generate the feedback clock with same frequency as input reference clock.

A toggle flip flop generates the clock which has twice the time period of the input clock given to it, that is we obtain the clock having half the frequency of the input clock provided to the frequency divider. For 8x clock multiplier, we should divide the output clock by 8 to generate feedback clock. For obtaining one-eigth of frequency, we should cascade three toggle flip flops. 

### Important terms used in PLL

1. Lock-in range

Range of freuencies which PLL can stay in lock in steady state. It mainly depends on the range of frequencies VCO can produce and is limited by the dead zone of PFD.

2. Capture range 

Range of frequencies which PLL can lock when started from unlocked condition. Capture range is smaller than the lock-in range. It depends on the loop filter bandwidth.

3. Settling time

The time taken by the PLL to lock from an unlocked state is called the settling time. It depends on how quickly charge pump rises to the stable value.

## Lab setup

We should use the source code to build any software tool as it gives the latest updates and fixes. 

Here, we will use two tools. They are:

Ngspice : for transistor level simulation

Magic : for layout design and parasitic extraction.

### Using tools

From the ubuntu package , we will be downloading Ngspice directly.  Magic we will compile it with the source code because we need the latest version of Magic to support Skywater130nm technology node.

Ngspic: ngsice<circuit_file_name> 

It plots the results based upon the simulation instruction given in the circuit file after simulating the given circuit file. 

Magic: magic - T <Technology_file_from_PDK><the_layout_file_to_open> 

It opens the layout file, where we can view the layout  and make modifications to it. Magic has many features, out of which we will be using parasitic extraction and gds features.

### Development Flow

With the PLL specifications, we will perform SPICE-level circuit development and pre-layout simulation. After the pre-layout simulation we develop layout. Before layout phase we donot know the interconnection or proximity or size of the different circuits. After layout, we can extract the capacitive effect which is called a s the parasitic extraction. After this, we have spice netlist which is more closer to the real worls. Then, we do the post simulation which is more accurate than thr pre-layout simulation.

Its is important to note that, after each simulation step, we should make several changes to bring the working of the circuit much closer to the required specification.

### Ngspice setup

We use the following command to isntall Ngspice:

sudo apt-get install ngspice

Then we fetch the sky130 library which consists of the transistor models which is needed for the simulation.
### Magic setup

We will use the source code from opendesigncircuit.com. We can do the git clone and intall the Magic. Then, we should compile it, for that we should install the dependencies form the opendesigncircuit.com. Then we run the ./configure command. The we run the make command. Then we run the command

sudo make install

Then we should install the sky130nm technology information which Magic needs for desiging the layout.

## PLL Specifications

Corner = TT

Supply voltage = 1.8V

Temperature = Room temperature

Input Fmin= 5MHz , Fmax= 12.5MHz

Multiplier = 8x

Jitter (RMS) < 20ns

Duty cycle = 50%

# DAY 2 - PLL simulation (Prelayout and Postlayout)

A spice file is a text file with .cr extenstion.

For example, if we want to write code for frequency divider circuit, we write the following command in terminal:

touch FD.cir

and then page will be created where we should write the code for the desired circuit. Don't forget to include the SPICE library file in it.

Similarly we do it for all the blocks of PLL.

After creating the .cr files we run the Ngspice simulation.

## Pre-layout simulations

### Phase Frequency Detector 

#### The command to be run to obtain the PFD waveform is as shown below:

![PD_ngspice]()

Similarly, we run this command for different PLL blocks by appropriately changing the name of the block as per the requirement.

#### The PFD waveform is as shown below:

![PD_wave]()

<b>Red:</b> Clock 1 <br>
<b>Blue:</b> Clock 2 <br>
<b>Orange:</b> Up Signal <br>
<b>Green:</b> Down Signal

### Charge Pump 

#### CP response when "UP" signal is high:

![CP_charging]()

<b>Red:</b> Charge pump output voltage <br>

#### CP output rise due to charge leakage:

![cp_wave]()

<b>Red:</b> Charge pump output voltage <br>
<b>Leakage:</b> 40uV increase every 1us <br>

### VCO

#### VCO waveform is as shown below:

![VCO_wave]()

<b>Red:</b> Control Voltage <br>
<b>Blue:</b> Output Clock <br>

### Frequency Divider 

#### The Frequency Divider waveform is as shown below:

![Fd_wave]()

<b>Red:</b> Output Clock <br>
<b>Blue:</b> Input Clock <br>

### PLL

#### The PLL waveform is as shown below: 

![PLL_wave]()

<b>Red:</b> Reference Clock <br>
<b>Blue:</b> Output Clock Divided by 8 <br>
<b>Yellow:</b> Down Signal <br>
<b>Brown:</b> Up Signal <br>
<b>Pink (top) :</b>Charge pump output <br>

## Troubleshooting steps

If output doesn't match or mimic properly, first is to observe what kinds of issues you are facing. Always try to debug individual circuits beore simulating the whole circuit.

If signals are coming flat up or the simulation is crashing then check whether the connectivity is done properly or any issues like wrong net names, capitaliszation issues or parameter value issues. 

If the signals are coming as expected but mimicing of signal is not happening then verify the following:

1. VCO is working within the required range or not.

2. Whether the PFD is able to detect small phase differences or not.

3. How is the response of charge pump? Is it fast or slow. Too much fluctuations in charging or discharging, then capacitor sizing is the thing where we have to pay the attention to. Also, check if there is charge leakage. If the charge pump is charging when the input is zero, then there is charge leakage issue.

4. If nothing works out, then try adjusting the loop filter by using the thumb rule.

## Layout

In layout, these are the following different colour codes for different components used in layout: 

1. p-diffusion - plane orange colour

2. n-diffusion - plane green colour

3. polysilicon - plane red

4. n-well - slash lines

5. metal1 layer - purple

6. local interconnect layer - blue

To connect two transistors, we use interconnect layer. To connect two metal layers, we use the contact/via.

### Phase Frequency Detector 

#### The command to be run to obtain the PFD layout is as shown below:

![PFD_lay_term]()

Similarly, we run this command for different PLL blocks by appropriately changing the name of the block as per the requirement.

#### The PFD layout is as shown below:

![PFD_lay]()

<b>Area:</b> 49.09um square <br>

### Charge Pump

#### The CP layout is as shown below:

![CP_lay]()

<b>Area:</b> 132.29um square <br>

### VCO

#### The VCO layout is as shown below:

![VCO_lay]()

<b>Area:</b> 57.73um square <br>

### Frequency Divider

#### The Frequency Divider layout is as shown below:

![fd_lay]()

<b>Area:</b> 49.09um square <br>

### MUX

#### The MUX layout is as shown below:

![mux_lay]()

<b>Area:</b> 12.12um square <br>

### Integrated PLL

#### The PLL layout is as shown below:

![PLL_lay]()

<b>Area:</b> 496.03um square <br>

## Post-layout Simulation

### Phase Frequency Detector

#### The command to be run to obtain the PFD layout is as shown below:

![PFD_postlay_term]()

Similarly, we run this command for different PLL blocks by appropriately changing the name of the block as per the requirement.

![PFD_postlay]()

<b>Red:</b> Clock 1 <br>
<b>Blue:</b> Clock 2 <br>
<b>Orange:</b> Up signal <br>
<b>Green:</b> Down signal <br>

### Charge Pump

#### CP response when "UP" signal is high:

![CP_postlaywave]()

<b>Orange:</b> Charge Pump Output Voltage <br>
<b>Red:</b> Up Signal <br>
<b>Blue:</b> Down Signal <br>

#### CP output rise due to charge leakage: 

![Cp_leakage_wave]()

<b>Orange:</b> Charge Pump Output Voltage <br>
<b>Red:</b> Up Signal <br>
<b>Blue:</b> Down Signal <br>
<b>Leakage:</b> < 0.05V in 100us <br>

### VCO

![VCO_postlay]()

<b>Red:</b> Output Clock <br>
<b>Blue:</b> Control Voltage <br>

### Frequency Divider

![FD_postlaywave]()

<b>Red:</b> Input Clock <br>
<b>Blue:</b> Output Clock <br>

### PLL

![PLL_postlay]()

<b>Red:</b> Reference Clock <br>
<b>Blue:</b> Output Clock Divided by 8 <br>
<b>Yellow:</b> Down Signal <br>
<b>Brown:</b> Up Signal <br>
<b>Pink (top) :</b>Charge pump output <br>

## About Tapeout

Tapeout means to send our design to the fab after we prepare it with all the additional support we require. 

We should first connect our silicon wafer with the real world. For that we use I/O pads. Then for any kind of serial connectivity like I2C, UART and other peripherals, we should have their designs. Memory also has to be incorporated which takes a lot of space. Testing mechanisms should also be added. Taking care of all of this becomes complicated hence, we should choose a driver to enable our IP to meet the desired requirements to undergo fabrication process. For this we can use Efabless Caravel SoC template.

It will provide the user project area to add our design, and we need not bother about other things on SOC.

# Acknowledgement

1. I would like to thank Mr. Kunal Ghosh, co-founder [VSD](vlsisystemdesign), for providing me with this wonderful 2-day workshop.

2. I would like to thank Ms. Lakshmi S, for guiding throughout the workshop about how to design PLL.

