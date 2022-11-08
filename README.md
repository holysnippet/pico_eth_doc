# pico_eth
>RP2040 Minimal hardware (PIO) Ethernet compatible interface + lwIP TCP/IP stack driver - ROBIN Guillaume, 2022


This library allows you to add Ethernet 10Base-T compatible connectivity to your Pico at the cost of some passive components. **It uses the proven TCP/IP lwIP stack provided in the latest RP2040 SDK.** It is therefore compatible with programs written for the Pico W or existing lwIP apps and code.


- Do not connect the electronic assemblies described on this page to equipment that uses or provides **Power over Ethernet (PoE).**

- Avoid connecting to Ethernet in a non-isolated way (without a transformer). This is possible but for testing purposes only. **Isolation guarantees the safety of the goods and the integrity of the data. Don't be surprised if you lose frames, your Pico fries or worse...🔥**
