# Hermes
![3D Model](./Images/3D_Model.png)
A small form factor, low power, RF module with a programmable STM32.  The goal of this project was to make a small board to mess around with RF and some basic DSP.

## Stack-up
![PCB Stack-up](./Images/Stackup.png)
I planned to manufacture this board through JLC PCB.  The service allows for cost-effective manufacturing with reasonable accuracy and constraints.  I decided to use an impedance-controlled, 4-layer stack-up because of the RF section (JLC04161H-7628).  This board is 1.6 mm thick with an outer copper weight of 1oz and inner copper weight of 0.5oz.  The stack-up information is important in determining the USB differential and RF antenna trace widths and/or spacing. 

## Design Considerations
![Schematic](./Images/Schematic.png) 
![Layout](./Images/Layout.png)

### USB Connection
I decided to use a micro-USB connection for this board.  I chose this connector mainly for its size.  I did consider using USB-C since it's becoming more prevalent in modern electronics and similar in size.  However, USB-C has many connections which would not be useful in this case.  I did not need anything incredibly high-speed or with high power demand.  If one wanted to use USB-C with this board, it's possible to basically just swap out the connector and terminate it as per the USB-C specifications for USB-C to micro-USB.  If this is done, it may be wise to add current protection.  I left current protection out mainly to reduce the number of components, and the likelihood of micro-USB (due to the shape) having reverse current is incredibly low.  I did still include ESD protection though.

USB uses differential signals.  The specification for the USB protocol is to have 90 Ohm impedance traces.  To do this, I assumed a trace spacing of 8 mils.  The spacing could be reduced, but I chose this for ease of manufacturing.  Using the stack-up information and the differential spacing, trace widths are calculated to be 10.28 mil.

### Power
The USB connection to a host device will provide the board with 5V.  Most of the components on the board are designed for 3V3, so a voltage regulator is needed.  I chose the XC6206P332MR.  This is an efficient, low cost, and low noise LDO regulator. 

### Microcontroller
I chose to use the STM32L432KBU6 for this board.  It's low power and has a built in USB module.  This MCU doesn't have a ton of memory, but it will be used mainly for passing the RX/TX signal along to the host and do basic processing.  So, it has more than enough memory for its intended use.  Additionally, the core speed is more than enough to handle the transceiver's data.  

Assuming a 1ns rise time, which is faster than expected for this setup, the critical length is around 41.1 mm.  This means as long as the traces are less than 41.1 mm, impedance effects can largely be ignored for the MCU's digital signals (the board isn't even that long!).  Furthermore, assuming worst-case propagation delay, based on the PCB materials, of 180 ps/in, the slight differences in trace length won't make a huge difference.  The SPI traces are short and the fastest setup/hold times are around 1.5 ns, so the slight difference in SPI trace lengths (the skew in delay for the SPI signals is approximately 37.4 ps) will have virtually no effect on performance.  If the MCU was operating at GHz frequencies, problems would likely arise, but this length difference is okay for the fastest signals supported by this system. 

### Transceiver
I chose the nRF24L01+ which is a 2.4GHz transceiver (using [GFSK](https://en.wikipedia.org/wiki/Frequency-shift_keying#Gaussian_frequency-shift_keying) modulation).  This transceiver contains basically everything needed to perform the functions of most consumer electronics, so it is more than enough for experimenting with.  It has good documentation, a decent amount of customizability, and doesn't use a ton of power.  Additionally, The IC supports a variety of different antenna configurations, but I planned to connect a 2.4GHz passive, single-ended antenna.  Thus, I included the necessary matching and bias networks (assuming a 50 Ohm antenna) to meet the specifications as described by the data sheet.

To meet the impedance matching requirements, the antenna traces are to be 50 Ohms.  Using the stack-up information, the calculated trace width is 11.55mil.


## Datasheets
- [STM Datasheet](https://www.st.com/content/ccc/resource/technical/document/datasheet/24/01/9f/59/f0/83/47/fc/DM00257205.pdf/files/DM00257205.pdf/jcr:content/translations/en.DM00257205.pdf)
- [Transceiver Datasheet](https://www.sparkfun.com/datasheets/Components/SMD/nRF24L01Pluss_Preliminary_Product_Specification_v1_0.pdf)
- [ Linear and low-dropout regulator](https://www.mouser.com/datasheet/2/760/TOSL_S_A0007229533_1-2575067.pdf)
