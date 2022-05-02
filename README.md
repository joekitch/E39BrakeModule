# BrakeModule

This project has been created to solve the need of make an MY99 BMW 540i (E39) to brake with use of semiautonoumous driving software OPENPILOT.

---

### Disclaimer

The software and hardware of this project has been done by fellow that have no knowledge of anykind on automotive, software or hardware developing stuff nor electrical engineering. So if you use it as is keep in mind that there is some change that the BrakeModule will behave unwantetly or will damage your cars electrical system!
If you have better knowledge than I, please contribute on developing this.

---

## BACKROUND

This module has been developed with use of car which uses BOSCH ABS 5.7 Dynamic Stability Control system 3 (DSCIII). This system is used on several different cars so it could possibly be used with these car only software modification. The description of the system is what I have found on my system, yours may vary. I also make a lot of assumptions of how the system works on based on what I have seen but I don't have near close to 100% certainty of it. There might be a change this could be used on BOSCH 5.3 system if DSCIII implemented.

Some list of of cars that use BOSCH 5.7 system found on the internet.

<p align="left">
  <img src="Pics/Bosch57cars.PNG?raw=true">
</p>

I'm not sure if these are compatible cars, eg. is BOSCH 5.7 system == charge pump system eg. DSCIII (or similar), that needs to be in place to brake the car?

The SOFTWARE has been designed to read braking demand values from CAN bus that are sent from OPENPILOT. OPENPILOT (OP) is opensource semiautonomous system (software) that can possibly be retrofitted to your car to have LEVEL 2 autonomous system. The OPENPILOT needs to be able to control longnitudinal (acceleration and deceleratio) and lateral (steering) via CAN bus. 

Video of OPENPILOT.

[![](https://img.youtube.com/vi/NmBfgOanCyk/0.jpg)](https://youtu.be/NmBfgOanCyk)

This module could be used with other adaptive cruise control systems (ACC) with modifying the software.

---

## Working principle
Superabstaract of BOSCH 5.7/DSCIII system. In my mind the system can be roughly derived into few subsystems: ABS module which is the control unit (ECU), valves that control the brakefluid flow, ABS relief pump, charge pump (also known as pre-charge pump) and sensors that monitor the system and the trajectory of the car.
In ABS/DSC situation ABS module can reduce motors torque via CAN messages, relief brake pressure on individual brakes with valves and ABS relief pump or apply pressure to certain brakes by controlling the valves and running the charge pump.

Picture of BOSCH system with DSCIII.
<p align="left">
  <img src="Pics/DSCIII.PNG?raw=true">
</p>

As the charge pump can increase the brake pressure, if one can control it can control also cars deceleration (braking). The charge pump is controlled normally by 2 N-channel MOSFETs inside ABS module in halfbrige configuration.

<p align="left">
  <img src="Pics/PumpFETs.PNG?raw=true">
</p>

Handwritten schematic of the driving circuit of the charge pump.
<p align="left">
  <img src="Pics/PumpSch.PNG?raw=true">
</p>

If someone want's to deep dive closer on Bosch DSC 5.7 ABS Module Diagnosis and Repair read this great post: https://www.bimmerfest.com/threads/bosch-dsc-5-7-abs-module-diagnosis-and-repair.822139/#post-8854110 (the pics are stolen from it).

Main thesis of working principle of the BrakeModule is 1. to disconnect the ABS unit from the charge pump and connect the charge pump wires from ABS module with resistor so that the ABS module "thinks" that the pump is connected (if wires are disconnected ABS module throws an charge pump error). 2. Connect 12V to the pump and use N-channel power MOSFET to adjust to charge pump yield (brake pressure). 3. One has to manipulate also the cars brake light (pedal) switch because if car detects increased brake pressure in the system without detection of brake pedal beeing pressed, it throws an brake pressure sensor defekt error. Brake light switch has 4 wires: 12V, ground, signal LOW, signal 2 HIGH. Signal LOW is grounded and signal HIGH is floating when pedal not pressed. When pedal is pressed, signal LOW is floating and signal HIGH is connected to 12V.

Further knowledge on how BOSCH 5.7/DSCIII works, look at dsc_system.pdf on the repo.

<p align="left">
  <img src="Pics/ABSunit.PNG?raw=true">
</p>

ABS module/DSC hydraylic unit shown in M54 engine bay.

<p align="left">
  <img src="Pics/Pump.PNG?raw=true">
</p>

Charge pump shown in M62TU engine bay.


---

## BrakeModule HARDWARE

Brakemodule main functionalities are 3 relays for switching charge pump inline with ABS system or BrakeModule. 1 power MOSFET to control the charge pump. Temperature measurement to read the power MOSFET temperature. FAN + mosfet to cool down the power MOSFET. CAN module is used for communication. Voltage divider is used to measure ABS line voltage.

In this development stage the main components of the hardware is LGT8F328P LQFP32 MiniEVB (can work w Nano also), MCP2515 CAN module, 4 relay module, Infineon IRF40R207 N-chan power MOSFET, TC1413N gate driver IC, 2 2N7000 N-chan MOSFET, 2N3904 NPN transistor, BS250 P-chan MOSFET, DS18B20 temperature sensor.

Main principle of driving charge pump and brakelight switch with BrakeModule.
<p align="center">
  <img src="Pics/BMmain.PNG?raw=true">
</p>

Picture of PCB of BrakeModule v0.2.
<p align="center">
  <img src="Pics/BM0.2.PNG?raw=true">
</p>

BrakeModule PCB is mounted on 3D-printed case with 4 relay module, and attachments for the fan.
Picture of BrakeModule.
<p align="center">
  <img src="Pics/BM.jpg?raw=true">
</p>

The PCB layout is much improved on the next generation and should not give too much thought. Example there are not use of 2 power MOSFETs (TO220 and TO252) these slots are for testing purposes.

---

## Next generation

The next generation of the board is planned to have an Blue Pill development board accompanied with MCP2551 CAN module. 
- Integrate the relays (and use better ones) into the PCB. 
- Integrate the brakeswitch lines into the PCB, get the low current demand of the board from brake light switch lines.
- "Read" the brake pedal switch true state. 
- Change the temperature sensor from DS18B20 to LM36. 
- Integrate pump wire connections into PCB.
- Have USB to UART (FTDI) connector for updating the software and debugging.
- Maybe use of different kind of pwr MOSFET
- Add 12V to 5V buck converter module

<p align="center">
  <img src="Pics/BM03.PNG?raw=true">
</p>

---

## SOFTWARE

The software has been developed in arduino IDE to LGT8F328P board with Arduino Nano compability in mind. The next generation of the module most likely uses Blue Pill development board (STM32F103). Normally precharge pump is controlled via BOSCH ABS module (relays are on NC mode) but when decelaration demand from OP is detected on CAN msg 0x343 (ACC_CMD), it'll disconnects the ABS module from the pump and start controlling the pump with ~2 kHz PWM signal of the power MOSFET (relays state are switched OFF from NC mode). Also brake light swithes HIGH and LOW signal lines are driven so that the car detects brake pedal pressed event. DS18B20 temperature measurements from power MOSFET every 10 seconds and small fan turn on if over 45 degrees is detecteted. Also if temperature exceeds 80 degrees, brake module will disable OPENPILOT if it is on control and wont engage until temperature is below that.

BrakeModule is used to emulate TOYOTA corollas cruise controller because this is the car which is used on my OPENPILOT fork. This implementation is shown as sent data sent in 0x1D2 and 0x1D3 CAN messages which are originally used by TOYOTA cruise controller. The use of TOYOTA in OP is from legacy reasons because the first guy that used OP on older cars implement it on TOYOTA Celica and my code is just revision of that.

For discussion of "old" cars impelementation of OPENPILOT join discord: discord server link here.

Speed value is read on message 0x153 and send to OPENPILOT as ACC set speed when OP is engaged (set_speed). Brake pedal state is read on message 0x329 (BRK_ST) and sent to OP when OP is not braking and pedal press is detected to disengage OP (BRK_ST_OP). Also BMW cruise control steering wheel button presses (BTN_ST) are detected on same message. Cruise state (CC_ST_OP) is detected on 0x545 and if it is on OP wont engage to prevent original CC and OP to control longnitidinal simultatoneusly.

Filtering of CAN messages are used to give better change to not to lose wanted messages.

If DEBUG is #defined you can control the board via serial (look at the readSerial() function).

Note to self:
- Add DBC file that contains only E39 and TOYOTA corolla msgs.
- Add reference to how to read DBC's.
- Openpilot is disangaded rapidly if 80 degrees is detected, is there a better way?

---

### Considerations
What I have understand there are standards for automotive hardware and software design and this does project does not follow any of those.

The main worry point that I have is that if an stability control event should happen on the same time that BrakeModule is controlling the charge pump there is quite high change that the stability control system wont funtion as planned. This is an issue that I want to address after getting the next generation hardware done.

If you damage the ABS control unit it is quite hard to repair and if bought new also quite pricey. I have no promises that using the BrakeModule won't anything, I think even that it is somewhat likely that it could happen at least with this HARDWARE/SOFTWARE. Knock on wood, I haven't braken my unit even though it has had quite a rough love.

Good design would prolly be to install the module on a professional case so components wouldn't be exposed with integrated heat dispersion.

---
