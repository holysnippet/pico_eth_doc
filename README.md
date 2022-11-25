# pico_eth
>RP2040 Minimal hardware (PIO) Ethernet compatible interface + lwIP TCP/IP stack driver - ROBIN Guillaume, 2022

This library allows you to add Ethernet 10Base-T compatible connectivity to your Pico at the cost of some passive components. **It uses the proven TCP/IP lwIP stack provided in the latest RP2040 SDK.** It is therefore compatible with programs written **in polling mode** for the Pico W or existing lwIP apps and code.

- It is not advisable to connect the electronic assemblies described on this page to equipment that uses or provides **Power over Ethernet (PoE).**
- It is not advisable to connect to Ethernet without an isolation transformer, it's electrically insecure and will result in more lost frames!

**Sample project and code is in this repository: https://github.com/holysnippet/pico_eth**

- [What to expect from such a set-up?](#wte)

- [Physical interface, cheap DIY version](#phy)

- [Principle of passive reception](#pri)

- [Assembly guidelines](#ass)

- [Hardware troubleshooting guide](#tro)

- [Using the UF2 test image](#usi)

- [Create your own Pico E project](#cre)

- [Using the source code](#src)

- [Using the stack](#stk)

- [Known software issues](#kno)

- [lwIP stack tuning](#lwi)

## Caution, early release

The software and hardware interfaces provided on this page are at an advanced stage of development but are not finished. They are provided free of charge and the author cannot be held responsible for their use.

Please consider a Github ⭐ or a donation if you find my work useful, if you like it and if you wish to encourage its completion and maintenance.

![alt text](https://github.com/holysnippet/pico_eth_doc/blob/main/images/both.png "Two Pico E")

<a name="wte"></a>
## What to expect from such a set-up?

As mentioned above, this software is under development. The code published in the main branch is stable. The implementation of the lwIP stack is correctly done. The dialog has been checked with Wireshark, there is little or no TCP retransmission, the examination of the lwIP logs suggests a healthy behavior of the IP stack.

The UF2 test image and the test program provided with the source code embeds an iperf **version 2** server that supports TCP. **Beware, iperf version 3 clients are not supported!**

![alt text](https://github.com/holysnippet/pico_eth_doc/blob/main/images/speeiperf.png "iperf 2 TCP test")


We can measure throughputs ranging from more than **5Mbit/s** to more than **7Mbit/s!** This is a respectable performance, it's an early software, it runs on a single core (including lwIP) the electrical interface doesn't cost much and we have to remember that it's TCP: the flow control costs bandwidth. The UDP throughput should be close to the theoretical maximum throughput of the interface, i.e. not far from 10Mbit/s.

<a name="phy"></a>
## Physical interface, cheap DIY version
**This "Ethernet to 3.3V" interface is electrically incorrect. We shamefully rely on the Pico's input protection devices.** However, in practice; the limiting resistor prevents the current from reaching significant values. Ethernet does not have so high levels (+2.5V to -2.5V).

![alt text](https://github.com/holysnippet/pico_eth_doc/blob/main/images/boardclassic.png "Early interface version")

The Pico is connected to the Ethernet bus via a standard isolation transformer. You can get them from Ethernet devices (motherboards, network cards, hubs & switches ...) that are going to the trash. Don't forget to get the hardware MAC address. It's free and will be useful soon.
The positive side of the receiving pair is connected to the Pico by a limiting resistor. The negative side of the receiving pair is shifted by a resistor network in order to take advantage of the differential nature of the signal.

I have tested many differential receivers (DIY, with standard 2N2222 or 2N3904 transistors) but the result is not good because they don't work very well in 3.3V; they are a bit slow: they distort the signal more than a direct reception. 10Base-T Ethernet is wider than 20MHz!

With the right transistors or differential receiver chip, one could make a very clean setup! Feel free to help if you know of a good configuration or reference available! Ideally, the goal is to keep the assembly simple, without components that are too specific, expensive or complicated to procure. I put this simple and incorrect configuration so that people can simply test without investing too much. I've left some tracks further down (Physical interface possible improvements) if you want to dig deeper.

![alt text](https://github.com/holysnippet/pico_eth_doc/blob/main/images/eliface.png "Electrical interface")

(The values in the diagram are tested, **prefer them**, I advance intervals in the text)

The electrical diagram is simple:

**Common:**

- **T1** Is a standard 10/100 Ethernet transformer, it provides galvanic isolation between the Pico and the Ethernet bus.
- **C1** is a power supply smoothing capacitor. It should be present if your setup is wire fed or if the power lines have significant resistance. Its value is not critical and can range from 100nF to 10uF.

**Transmitter:**

- **R5 & R6** limit the output impedance of the transmitter to about 100 ohms. You can use resistors from 47 to 68 ohms.
- **R7** limits the amplitude of returned reflections from the line. You can use resistors from 500 to 1k ohms.
- **C3 & C4** They block the DC component of the signal at the output of the Pico. They are mandatory for Ethernet transformers with common winding center taps (RX & TX Pico side center taps connected together). It is better to put them on if you are not sure of the type of your transformer. Omit them at your own risk, you won't damage your Pico but it won't work! If you populate them even though there is no need for them, it won't make any difference. You can try from 6.8nF to 68nF.

**Receiver:**

- **R4** sets the impedance at the output of the transformer in order to present a correct impedance to the line. 82 to 120 ohms should be suitable.
- **R1 & R2** form a voltage divider bridge. Its division factor is 0.45 = R2 / (R2 + R1) = 2700 / (2700 + 3300).
- **C2** buffers the voltage delivered by the divider bridge R1 & R2. 100pF to 1nF should work but more investigation could be beneficial.
- **R3** limits the current flowing to the Pico input. The Ethernet signal is a high frequency signal. It can therefore cause a significant current to flow (in the worst case) through the small capacitance (a few pF) of the Pico's input port. A slightly lower value of **R3** would distort the signal a little less and allow a lower error rate. 800 ohms to 1.6k ohms seem to work. Prefer the value in the diagram (1k). More research could also be beneficial here.

***R1, R2, R3 and C2** were determined empirically. I wanted to keep the receiver simple so that this project would be accessible to as many people as possible. I think there are better solutions. Mine consumes little current and takes advantage of the differential nature of the signal. The disadvantage is that we expose the Pico to the Ethernet voltages and depend on its internal protections. (And we also accept to potentially lose some frames compared to an active differential receiver). But the voltages presented by Ethernet are low, if we keep a sufficient value for **R3** (but not too high either) it is unlikely that the Pico will suffer from latch up events and even more unlikely that it will be damaged.*

**Physical interface possible improvements:**

- Evaluate the reception quality by disabling the Schmitt trigger on the GPIO input.
- Evaluate the reception quality by coupling the signal differently or with a different biasing method.
- An active differential transceiver (ISL3177) would provide an extremely low error rate, out of the box.
- Someone talented could probably arrange a pair of 2N3904s into a cheap and fast 3.3V differential receiver.

<a name="pri"></a>
## Principle of passive reception

The purpose of this setup is to shift the voltage applied to the negative side of the Ethernet transformer (RD-). It is important to set this voltage at the right threshold so that the Pico input gate switches at the right level, i.e. preserving the symmetry and temporal properties of the Ethernet signal.

The positive side (RD+) will thus be continuously shifted by about half the supply voltage (but not necessarily exactly). Therefore, when a negative pulse occurs at the input of the transformer, it will drop a little below zero at its output on the positive side of the transformer. Conversely, it is just as simple, when a positive pulse occurs at the input, it will slightly exceed the supply voltage at the output. So we take advantage of the differential nature of the Ethernet signal!

<a name="ass"></a>
## Assembly guidelines

The use of a rapid prototyping board called "Breadboard" can eventually work but will give poor results (you will definitely have frame drops). This is due to the fact that Ethernet is a fast signal (its spectrum is at least 20MHz wide) which means that the parasitic capacities presented by a Breadboard will significantly tend to "smooth" this fast signal. Without mentioning possible crosstalk effects due to the construction of the Breadboard.

The use of a "Perfboard" rapid soldering circuit gives satisfactory results if the realization is careful; the legs of the components must be cut and the Ethernet wires (if there are any) must remain short.

![alt text](https://github.com/holysnippet/pico_eth_doc/blob/main/images/boardmagjack.png "DIY Pico E")

To make a quick first try you should try to recover the transformer of a used Ethernet device. It should be a 10/100 Base-T transformer. They are abundant. Also think (if you can) about getting the MAC address of the donor device: you will be able to assign it to your shiny new Pico-E !

![alt text](https://github.com/holysnippet/pico_eth_doc/blob/main/images/mbtra.png "Ethernet transformer")

![alt text](https://github.com/holysnippet/pico_eth_doc/blob/main/images/mb-transformer-ds.png "Ethernet transformer datasheet")

There is also another option, the "MagJack Ethernet" type plugs (this is a registered trademark). These are shielded Ethernet sockets that contain the transformer. This is a more expensive option but has the advantage of integrating the connector, the transformer, LEDs, HV capacitors and a metallic shield.

![alt text](https://github.com/holysnippet/pico_eth_doc/blob/main/images/magjack.png "Magjack integrated Ethernet transformer")

The other components are standard passive. You can buy them or desolder them if you have access to "electronic waste".

<a name="tro"></a>
## Hardware troubleshooting guide

This section has not yet been written due to lack of experimentation. It is possible that some Ethernet transformers are not suitable. If your setup does not work with the UF2 image provided then you most likely have an Ethernet signal integrity problem. You can open a GitHub issue but don't expect quick help.

The biasing voltage is the most important parameter and is the only thing you can check in case of a problem. It should be 0.45 times the supply voltage of the Pico.

As an indication, a measurement on one of my interfaces gives 1.45V (for a supply voltage of 3.25V).

If you find a value close to zero volts then you have (as I had) a transformer with the center point of the windings (RX & TX) connected together. You must populate **C3 & C4**.

<a name="usi"></a>
## Using the UF2 test image

This image is provided to allow you to quickly test your interface without having to compile the source code. It embeds a TCP iperf server version 2 only **(version 3 does not seem to be supported)**. There is also a HTTP test server. An NTP demo client runs permanently on the board. It queries an NTP server every 30 seconds and displays the result on the USB serial port.

#### Program pins: (See electrical diagram)

>TX_NEG is on Pico GPIO 16

>TX_POS is on Pico GPIO 17

>RX_POS is on Pico GPIO 18

The MAC address is as follows (**remember to change it** in your images, **if you use two Picos with the same MAC on the same network you will have trouble!**)

>MAC: 00:01:02:03:04:05

If you do not have a DHCP server, the default settings are as follows:

>IP: 192.168.1.110

>Netmask: 255.255.255.0

>Gateway: 192.168.1.1

The hostname of the card (which you can use if your network infrastructure allows it) is the following:

>lwIP_Pico

<a name="cre"></a>
## Source code compilation

The source code is not finished. This version is stable and can be used in your own projects. Some things are missing and some things can be improved, but the calls to initialize Ethernet will (in all likelihood) remain the same. Everything else (development and deployment of your application) depends on the lwIP library and its documentation. This is the same IP stack as the one used by the Pico W.

### Software requirements:

- The latest version of pico-sdk for RP2040 which is **1.4.0 is mandatory**.
- Currently compiles with GCC 10.3.1 but compiles with earlier versions (at your own risk).
- Visual Studio Code is used as IDE. But it is still possible to compile it in a classical way on the command line using cmake & make. 

### Steps:

- Get the code, you can make a clone of the GitHub repository of the source code, navigate to your development folder, then:
```bash
git clone https://github.com/holysnippet/pico_eth.git
```
Or download the zip archive directly from GitHub (Green button, download ZIP):

https://github.com/holysnippet/pico_eth

- Using the method of your choice (VS Code or other) make sure that the **PICO_SDK_PATH** environment variable is correctly set.

- The project should compile.

<a name="src"></a>
## Using the source code

For the moment this project does not take advantage of CMake to facilitate its distribution. You can start from the delivered sample project and customize the lwIP options there.

All files required for Ethernet support are in the "includes" directory of the sample project. This is very little because everything else is already in the Pico SDK!

If you want to add Ethernet support to an existing project the contents of the CMakeLists.txt file will be of great help. It contains everything you need. The Pico SDK has to find the lwipopts.h file of our project; if you look carefully I use the line : 

```cmake
# lwIP : Not sure, allows LwIP to find lwipopts.h
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_LIST_DIR}/includes)
```

This may not be the most sensible solution... You will also need to make sure that CMake generates the PIO program correctly. You can refer to the content of the provided CMakeLists.txt.

<a name="stk"></a>
## Using the stack

lwIP is currently running in NO_SYS=1 mode, which means that it must be initialized at the beginning of the program and then we must periodically pass the execution flow to it. An interrupt version (NO_SYS=0) is in progress and a version taking advantage of FreeRTOS is planned.

Most of the work is done in the network_init() function in main.c.

You have to configure your network interface, remember to change the MAC address! You can specify if you want to use DHCP or not, a default address, a hostname. There are some things missing like an Ethernet LED emulation. 

You can also configure the pins of the Pico to use for the Ethernet port.

### Important detail !
You can use any pins for the Ethernet port. The only condition is that the two pins of the TX must follow each other. For example **if TX_NEG is 16 then TX_POS WILL be 17.** There is no condition for the RX pin; try to choose a pin that is far from potentially noisy devices.

Then the applications are initialized before entering the main loop which starts with the "while(1)".

You will then have to call eth_pio_arch_poll() periodically between the processing of your application.

If you need the "**netif**" structure to use the lwIP features you just have to include ethpio_arch.h which publishes the instance of this structure.

### Pico resources used
1 DMA interrupt

1 PIO interrupt

3 pins

Only 1 of the 2 PIO modules

4 DMA channels (Could be reduced)

RAM : TBD (Could be reduced)

FLASH : TBD

<a name="kno"></a>
## Known software issues

You should only run this code at a system clock of 120MHz or 240MHz (overclock). Without going into details it is because of the fractional divisor of the PIO clock. It is quite possible that it will still work fine at other speeds, but strictly no testing has been done!

The code should work as well on core 0 as on core 1 but strictly no testing has been done on core 1.

Same for the PIO module, the code has only been tested on the PIO 0.

The PIO code needs to be reworked, there is no NLP detection on the RX side.

lwIP only works in NO_SYS=1 for the moment.

No method of unloading the code at the moment (deInit()).

No RAM/FLASH profiling performed at this time.

Ethernet LEDs are not emulated yet.

A software diagnostic bench would be welcome to test and fine-tune the Ethernet transceiver.

<a name="lwi"></a>
## lwIP stack tuning
The choices I made may not be suitable for your application as I torture test the stack to see how it works with the interface. **I can't go into details here.** lwIP configuration options are located in the provided file lwipopts.h. With only a few kilobytes of memory, you will have to make some compromises (i.e. max number of connections vs good throughput). There are many guides:

https://lwip.fandom.com/wiki/Tuning_TCP

https://lwip.fandom.com/wiki/Maximizing_throughput
