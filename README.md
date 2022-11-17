# pico_eth
>RP2040 Minimal hardware (PIO) Ethernet compatible interface + lwIP TCP/IP stack driver - ROBIN Guillaume, 2022


This library allows you to add Ethernet 10Base-T compatible connectivity to your Pico at the cost of some passive components. **It uses the proven TCP/IP lwIP stack provided in the latest RP2040 SDK.** It is therefore compatible with programs written **in polling mode** for the Pico W or existing lwIP apps and code.


- Do not connect the electronic assemblies described on this page to equipment that uses or provides **Power over Ethernet (PoE).**

### Physical interface
**This "Ethernet to 3.3V" interface is electrically incorrect. We shamefully rely on the Pico's input protection devices.** But the picture is not so black, the limiting resistor prevents the current from reaching significant values. Ethernet does not have so high levels (+2.5V to -2.5V).

The Pico is connected to the Ethernet bus via a standard isolation transformer. You can get them from Ethernet devices (motherboards, network cards, hubs & switches ...) that are going to the trash. Don't forget to get the hardware MAC address. It's free and will be useful soon.
The positive side of the receiving pair is connected to the Pico by a limiting resistor. The negative side of the receiving pair is shifted by a resistor network in order to take advantage of the differential nature of the signal.

I have tested many differential receivers (DIY, with standard 2N2222 or 2N3904 transistors) but the result is not good because they don't work very well in 3.3V and they are a bit slow: they distort the signal more than a direct reception. 10Base-T Ethernet is wider than 20MHz!

With the right transistors or differential receiver chip, one could make a very clean setup! Feel free to help if you know of a good configuration or reference available! Ideally, the goal is to keep the assembly simple, without components that are too specific, expensive or complicated to procure. I put this simple and incorrect configuration so that people can simply test without investing too much.


### lwIP stack tuning
The choices I have made may not be appropriate for your application (max number of connections vs good throughput). I can't go into details here. lwIP configuration options are located in the provided file lwipopts.h. With only a few kilobytes of memory, you will have to make some compromises. There are many guides :

https://lwip.fandom.com/wiki/Tuning_TCP

https://lwip.fandom.com/wiki/Maximizing_throughput
