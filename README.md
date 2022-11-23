# pico_eth
>RP2040 Minimal hardware (PIO) Ethernet compatible interface + lwIP TCP/IP stack driver - ROBIN Guillaume, 2022

This library allows you to add Ethernet 10Base-T compatible connectivity to your Pico at the cost of some passive components. **It uses the proven TCP/IP lwIP stack provided in the latest RP2040 SDK.** It is therefore compatible with programs written **in polling mode** for the Pico W or existing lwIP apps and code.

- It is not advisable to connect the electronic assemblies described on this page to equipment that uses or provides **Power over Ethernet (PoE).**
- It is not advisable to connect to Ethernet without an isolation transformer, it's electrically insecure and will result in more lost frames!

### Physical interface, cheap DIY version
**This "Ethernet to 3.3V" interface is electrically incorrect. We shamefully rely on the Pico's input protection devices.** However, in practice; the limiting resistor prevents the current from reaching significant values. Ethernet does not have so high levels (+2.5V to -2.5V).

The Pico is connected to the Ethernet bus via a standard isolation transformer. You can get them from Ethernet devices (motherboards, network cards, hubs & switches ...) that are going to the trash. Don't forget to get the hardware MAC address. It's free and will be useful soon.
The positive side of the receiving pair is connected to the Pico by a limiting resistor. The negative side of the receiving pair is shifted by a resistor network in order to take advantage of the differential nature of the signal.

I have tested many differential receivers (DIY, with standard 2N2222 or 2N3904 transistors) but the result is not good because they don't work very well in 3.3V; they are a bit slow: they distort the signal more than a direct reception. 10Base-T Ethernet is wider than 20MHz!

With the right transistors or differential receiver chip, one could make a very clean setup! Feel free to help if you know of a good configuration or reference available! Ideally, the goal is to keep the assembly simple, without components that are too specific, expensive or complicated to procure. I put this simple and incorrect configuration so that people can simply test without investing too much. I've left some tracks further down (Physical interface possible improvements) if you want to dig deeper.

![alt text](https://github.com/holysnippet/pico_eth/blob/main/images/eliface.png "Electrical interface")

(The values in the diagram are tested, **prefer them**, I advance intervals in the text)

The electrical diagram is simple:

**Common:**

- **T1** Is a standard 10/100 Ethernet transformer, it provides galvanic isolation between the Pico and the Ethernet bus.
- **C1** is a power supply smoothing capacitor. It should be present if your setup is wire fed or if the power lines have significant resistance. Its value is not critical and can range from 100nF to 10uF.

**Transmitter:**

- **R2 & R3** limit the output impedance of the transmitter to about 100 ohms. You can use resistors from 47 to 68 ohms.
- **R1** limits the amplitude of returned reflections from the line. You can use resistors from 500 to 1k ohms.

**Receiver:**

- **R4** sets the impedance at the output of the transformer in order to present a correct impedance to the line. 82 to 120 ohms should be suitable.

- **R6**, a 10k ohms potentiometer **preferably linear**, together with **C2** provide a variable voltage reference. It should be adjusted to a value close to half the supply voltage *(but not necessarily exactly)*. A fixed resistor network could work but I have not tested or determined values. **I managed to get less than one Ethernet frame wrong in a thousand (1/1000) with a potentiometer set in less than a minute!** (See the section on adjusting this potentiometer later). I have not tested other values for **R6** feel free to send me your test results.

- For **C2** you can try a value from 100pF to 1nF. Small values seem preferable.

- **R5** limits the current flowing to the Pico input. The Ethernet signal is a high frequency signal. It can therefore cause a significant current to flow (in the worst case) through the small capacitance (a few pF) of the Pico's input port. A slightly lower value of **R5** (and possibly a slightly different setting of potentiometer **R6**) would distort the signal a little less and allow a lower error rate.

***R6**, **R5** & **C2** have been determined empirically. I wanted to keep the receiver simple so that this project would be accessible to as many people as possible. I think there are better solutions. Mine consumes little current and takes advantage of the differential nature of the signal. The downside is that we expose the Pico to the Ethernet voltages and rely on its internal protections. The value of **R5** is consequent and the voltages presented by the Ethernet bus are low. It is more than unlikely that the Pico will suffer any latch-up event at these levels. It would be even more unlikely to damage it.*

**Physical interface possible improvements:**

- Evaluate the reception quality by disabling the Schmitt trigger on the GPIO input.
- Evaluate the reception quality by deactivating the pulldown resistor on the GPIO input.
- Evaluate the reception quality by coupling the signal differently (capacitively) in conjunction with one (or both) of the two options above.
- An active differential transceiver (ISL3177) would provide an extremely low error rate with no potentiometer to adjust.
- Someone talented could probably arrange a pair of 2N3904s into a cheap and fast 3.3V differential receiver.

### Assembly guidelines

The use of a rapid prototyping board called "Breadboard" can eventually work but will give poor results (you will definitely have frame drops). This is due to the fact that Ethernet is a fast signal (its spectrum is at least 20MHz wide) which means that the parasitic capacities presented by a Breadboard will significantly tend to "smooth" this fast signal. Without mentioning possible crosstalk effects due to the construction of the Breadboard.

The use of a "Perfboard" rapid soldering circuit gives satisfactory results if the realization is careful (the legs of the components must be cut and the Ethernet wires (if there are any) must remain as short as possible.

To make a quick first try you should try to recover the transformer of a used Ethernet device. It should be a 10/100 Base-T transformer. They are abundant. Also think (if you can) about getting the MAC address of the donor device: you will be able to assign it to your Pico.

There is also another option, the "MagJack Ethernet" type plugs (this is a registered trademark). These are shielded Ethernet sockets that contain the transformer. This is a more expensive option but has the advantage of integrating the connector, the transformer, LEDs, HV capacitors and a metallic shield.

The other components are standard passive. You can buy them or desolder them if you have access to "electronic waste". The potentiometer should ideally be linear, a logarithmic one will also work but will be less obvious to set. Prefer a potentiometer that is easy to adjust, if you can get a small multi-turn it is even better but not mandatory. The setting of the potentiometer is not so critical; but when it is very well set there are no or very few reception errors.

### Setting the biasing potentiometer

The purpose of the potentiometer setting is to adjust the voltage applied to the negative side of the Ethernet transformer. It is important to set this voltage at the right threshold in order to preserve the symmetry and temporal properties of the Ethernet signal. This is important to keep the signal synchronized and to remain synchronized throughout the duration of an Ethernet frame.

The positive side will thus be continuously shifted by about half the supply voltage (but not necessarily exactly). Therefore, when a negative pulse occurs at the input of the transformer, it will drop a little below zero at its output on the positive side of the pair. Conversely, it is just as simple, when a positive pulse occurs at the input, it will slightly exceed the supply voltage at the output. So we take advantage of the differential nature of the Ethernet signal!

The adjustment consists of several steps (which will be detailed). It's not hard but it's an important setting (if you want to keep a low frame dropping rate). You may have to be patient to tweak your setting the first few times.

**Always try to keep the potentiometer close to half, avoid going below 1/4 and above 3/4. It won't work and it exposes the Pico to slightly high (transient) levels.**



### lwIP stack tuning
The choices I have made may not be appropriate for your application (max number of connections vs good throughput). **I can't go into details here.** lwIP configuration options are located in the provided file lwipopts.h. With only a few kilobytes of memory, you will have to make some compromises. There are many guides :

https://lwip.fandom.com/wiki/Tuning_TCP

https://lwip.fandom.com/wiki/Maximizing_throughput
