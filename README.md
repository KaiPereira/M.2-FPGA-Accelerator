# M.2 FPGA Hardware Accelerator Devboard

This is an M.2 2280 M-Key card I've designed that plugs into the SSD slot in your computer and provides an FPGA fabric you can interact with over PCIe. It also has a couple pins broken out and a couple connectors like HDMI and USB-C so you can create custom pipelines that go from peripherals, into the FPGA, directly into your computer through the high speed PCIe. 

<img max-width="2560" max-height="1407" alt="image" src="https://github.com/user-attachments/assets/77cba1c9-4b35-45fb-a068-d3ae5028eb26" />
<img max-width="2560" max-height="1404" alt="image" src="https://github.com/user-attachments/assets/676cc1cd-6085-4a40-867c-d37594a27ac8" />

## Custom Features
- M.2 M-Key 2280 SSD Form Factor for PCIe programming and interfacing
- Artix 7 with 50K LUTs and 256MB RAM
- USB-to-JTAG for programming and interfacing with peripherals
- HDMI and 10 pins broken out for extra functionality
- Functions as a standalone devboard or can interface with your computer
- 6 LEDs for debugging and programming
- TC2030 breakout under USB-C if you DNP USB-to-JTAG

## PCB Design

The M.2 FPGA Hardware Accelerator is an extremely high density 0.8mm, 6 layer board using the JLC06081H-2116 stackup and has a stackup of SIGNAL/GND/SIGNAL/PWR/GND/SIGNAL.

A V2 is likely going to use a custom or different stackup to reduce the trace width for the DDR3 traces so I can minimize the cross-talk further, which is an issue with the current board.

<img max-width="2560" max-height="1404" alt="image" src="https://github.com/user-attachments/assets/47e564b3-1be6-4827-a081-26e20712cb27" />
<img max-width="2560" max-height="1404" alt="image" src="https://github.com/user-attachments/assets/3c27cdb4-98ad-46a0-a430-d6244a5cbe2a" />
<img max-width="2560" max-height="1402" alt="image" src="https://github.com/user-attachments/assets/b163a9d5-e2fd-44cb-b5fc-3f52513e9f48" />
<img max-width="2560" max-height="1405" alt="image" src="https://github.com/user-attachments/assets/66b56f1a-82a5-42f6-895b-27b8eeeb1387" />
<img max-width="2560" max-height="1403" alt="image" src="https://github.com/user-attachments/assets/d4cad3cf-e495-428b-bb94-e0beff3f1256" />
<img max-width="2560" max-height="1399" alt="image" src="https://github.com/user-attachments/assets/7c44b159-f7a5-4b2a-a407-1c3497a4e905" />
<img max-width="2560" max-height="1403" alt="image" src="https://github.com/user-attachments/assets/6df61845-10e0-4c02-b489-567c03f4d54b" />

## Firmware

Firmware is unwritten because I haven't produced the boards, and it requires significant prototyping in Vivado but there is some basic skeleton software inside of the [/software](/software) folder.

## Production

I've based the board around JLCPCB production and I've included an LCSC/JLCPCB BOM inside of the [/PCB/production](/PCB/production/) folder.

It uses the JLC06081H-2116 stackup which you can find details about on the JLC site if you want to produce the board from a different fabricator.

You'll want some specific settings too like:
- Impedance controlled traces
- ENIG and select gold fingers to add hard gold onto the M.2 M-Key edge connector
- 0.8mm thick board
- X-Ray to confirm all the BGA placement

Produce this board at your own cost, it's an untested board and I haven't manufactured it myself yet, because I want to create a V2 with a different stackup, and it's an extremely constrained design so many problems can arise!  

## Credits

Thanks so much to the KiCad discord for the PCB reviews, and to everyone in Hack Club for the support and to Renran and Samliu for running [Fallout](https://fallout.hackclub.com/) which is the program that's sponsoring this project!
