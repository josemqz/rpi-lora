Raspi-LMIC library
====================
This repository is a subset of this one **https://github.com/hallard/arduino-lmic/tree/rpi** and provides the IBM LMIC (LoraMAC-in-C) library ready to be used with a **Raspberry Pi with a Dragino Hat**.

In general it could be used  with the SX1272, SX1276 tranceivers and compatible modules (such as some HopeRF RFM9x modules).

This library mostly exposes the functions defined by LMIC, it makes no attempt to wrap them in a higher level API. To find out how to use the library itself, see the examples, or see the PDF file in the doc subdirectory.


Installing for Raspberry PI
---------------------------

**1st step:**
You need 3 dependencies:
- build essential package `apt-get install build-essential`
- other tools packages `apt-get install git-core wget`
- [bcm2835_library](http://www.airspayce.com/mikem/bcm2835/):
  ```
  # download the latest version of the library (for example):
  wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.56.tar.gz
  # then:
  tar zxvf bcm2835-1.56.tar.gz
  cd bcm2835-1.xx
  ./configure
  make
  sudo make check
  sudo make install
  # and very important
  sudo reboot now
  ```

**2nd step:**
Clone branch repository
```shell
git clone https://github.com/pmanzoni/raspi-lmic.git
```


**3rd step:**
Run the examples....

> IMPORTANT! Check that the following two lines **must be** like this:
 ```
 void os_getDevEui (u1_t* buf) { memcpy_P(buf, DEVEUI, 8);}
 //void os_getDevEui (u1_t* buf) { getDevEuiFromMac(buf); }
 ```

> Check that this section is **un**commented:
```
 // Dragino Raspberry PI hat (no onboard led)
 // see https://github.com/dragino/Lora
 #define RF_CS_PIN  RPI_V2_GPIO_P1_22 // Slave Select on GPIO25 so P1 connector pin #22
 #define RF_IRQ_PIN RPI_V2_GPIO_P1_07 // IRQ on GPIO4 so P1 connector pin #7
 #define RF_RST_PIN RPI_V2_GPIO_P1_11 // Reset on GPIO17 so P1 connector pin #11
```


---

Related Infos
---------------------------

**Connection and pins definition**

Boards pins (Chip Select, IRQ line, Reset and LED) definition are set in the samples sketch when needed see [Raspberry PI](https://github.com/hallard/arduino-lmic/tree/rpi/examples/raspi) examples folder, for example [LoRasPI](https://github.com/hallard/LoRasPI) board V1


```cpp
// LoRasPi board 
// see https://github.com/hallard/LoRasPI
#define RF_LED_PIN RPI_V2_GPIO_P1_16 // Led on GPIO23 so P1 connector pin #16
#define RF_CS_PIN  RPI_V2_GPIO_P1_24 // Slave Select on CE0 so P1 connector pin #24
#define RF_IRQ_PIN RPI_V2_GPIO_P1_22 // IRQ on GPIO25 so P1 connector pin #22
#define RF_RST_PIN RPI_V2_GPIO_P1_15 // RST on GPIO22 so P1 connector pin #15

// Pin mapping
const lmic_pinmap lmic_pins = { 
    .nss  = RF_CS_PIN,
    .rxtx = LMIC_UNUSED_PIN,
    .rst  = RF_RST_PIN,
    .dio  = {LMIC_UNUSED_PIN, LMIC_UNUSED_PIN, LMIC_UNUSED_PIN},
};

```

Then in your code you'll have exposed RF_CS_PIN, RF_IRQ_PIN, RF_RST_PIN and RF_LED_PIN and you'll be able to do some `#ifdef RF_LED_LIN` for example. See [ttn-otaa](https://github.com/hallard/arduino-lmic/tree/rpi/examples/raspi/ttn-otaa/ttn-otaa.cpp) sample code.


Features
--------
The LMIC library provides a fairly complete LoRaWAN Class A and Class B
implementation, supporting the EU-868 and US-915 bands. Only a limited
number of features was tested using this port on Arduino hardware, so be
careful when using any of the untested features.

What certainly works:
 - Sending packets uplink, taking into account duty cycling.
 - Encryption and message integrity checking.
 - Receiving downlink packets in the RX2 window.
 - Custom frequencies and datarate settings.
 - Over-the-air activation (OTAA / joining).

What has not been tested:
 - Receiving downlink packets in the RX1 window.
 - Receiving and processing MAC commands.
 - Class B operation.

If you try one of these untested features and it works, be sure to let
us know (creating a github issue is probably the best way for that).

Configuration
-------------
A number of features can be configured or disabled by editing the
`config.h` file in the library folder. Unfortunately the Arduino
environment does not offer any way to do this (compile-time)
configuration from the sketch, so be careful to recheck your
configuration when you switch between sketches or update the library.

At the very least, you should set the right type of transceiver (SX1272
vs SX1276) in config.h, most other values should be fine at their
defaults.

Supported hardware
------------------
This library is intended to be used with plain LoRa transceivers, connecting to them using SPI. In particular, the SX1272 and SX1276 families are supported (which should include SX1273, SX1277, SX1278 and SX1279 which only differ in the available frequencies, bandwidths and spreading factors). It has been tested with both SX1272 and SX1276 chips, using the Semtech SX1272 evaluation board and the HopeRF RFM92 and RFM95 boards (which supposedly contain an SX1272 and SX1276 chip respectively).

This library contains a full LoRaWAN stack and is intended to drive these Transceivers directly. It is *not* intended to be used with full-stack devices like the Microchip RN2483 and the Embit LR1272E. These contain a transceiver and microcontroller that implements the LoRaWAN stack and exposes a high-level serial interface instead of the low-level SPI transceiver interface.

This library is intended to be used inside the Arduino environment. It should be architecture-independent, so it should run on "normal" AVR arduinos, but also on the ARM-based ones, and some success has been seen running on the ESP8266 board as well. It was tested on the Arduino Uno, Pinoccio Scout, Teensy LC and 3.x, ESP8266, Arduino 101.

This library an be quite heavy, especially if the fairly small ATmega 328p (such as in the Arduino Uno) is used. In the default configuration, the available 32K flash space is nearly filled up (this includes some debug output overhead, though). By disabling some features in `config.h` (like beacon tracking and ping slots, which are not typically needed), some space can be freed up. Some work is underway to replace the AES encryption implementation, which should free up another 8K or so of flash in the future, making this library feasible to run on a 328p microcontroller.

Connections
-----------
To make this library work, your Arduino (or whatever Arduino-compatible board you are using) should be connected to the transceiver. The exact connections are a bit dependent on the transceiver board and Arduino used, so this section tries to explain what each connection is for and in what cases it is (not) required.

Note that the SX1272 module runs at 3.3V and likely does not like 5V on its pins (though the datasheet is not say anything about this, and my transceiver did not obviously break after accidentally using 5V I/O for a few hours). To be safe, make sure to use a level shifter, or an Arduino running at 3.3V. The Semtech evaluation board has 100 ohm resistors in series with all data lines that might prevent damage, but I would not count on that.

### Power
The SX127x transceivers need a supply voltage between 1.8V and 3.9V. Using a 3.3V supply is typical. Some modules have a single power pin (like the HopeRF modules, labeled 3.3V) but others expose multiple power pins for different parts (like the Semtech evaluation board that has `VDD_RF`, `VDD_ANA` and `VDD_FEM`), which can all be connected together. Any *GND* pins need to be connected to the Arduino *GND* pin(s).

### SPI
The primary way of communicating with the transceiver is through SPI (Serial Peripheral Interface). This uses four pins: MOSI, MISO, SCK and SS. The former three need to be directly connected: so MOSI to MOSI, MISO to MISO, SCK to SCK. Where these pins are located on your Arduino varies, see for example the "Connections" section of the [Arduino SPI documentation](SPI).

The SS (slave select) connection is a bit more flexible. On the SPI slave side (the transceiver), this must be connect to the pin (typically) labeled *NSS*. On the SPI master (Arduino) side, this pin can connect to any I/O pin. Most Arduinos also have a pin labeled "SS", but this is only relevant when the Arduino works as an SPI slave, which is not the case here. Whatever pin you pick, you need to tell the library what pin you used through the pin mapping (see below).

[SPI]: https://www.arduino.cc/en/Reference/SPI

### DIO pins

Since now, a software feature has been added to remove needing DIO connections. Of course, you can continue to use DIO mapping has follow, but in case you're restricted in GPIO available, you can avoid using any GPIO connection ;-) to activate this feature, you just need to declare 3 .dio to LMIC_UNUSED_PIN, in your sketch as detailled in Pin mapping section.

If you want to use hardware IRQ but not having 3 IO pins, another trick is to OR DIO0/DOI1/DIO2 into one. This is possible because the stack check all IRQs, even if only one is triggered. Doing this is quite easy, just add 3 1N4148 diodes to each output and a pulldown resistor, see schematic example on [WeMos Lora shield](https://github.com/hallard/WeMos-Lora).

If you still have DIO connection, following is explaining how they work. The DIO (digitial I/O) pins on the transceiver board can be configured for various functions. The LMIC library uses them to get instant status information from the transceiver. For example, when a LoRa transmission starts, the DIO0 pin is configured as a TxDone output. When the transmission is complete, the DIO0 pin is made high by the transceiver, which can be detected by the LMIC library.

The LMIC library needs only access to DIO0, DIO1 and DIO2, the other DIOx pins can be left disconnected. On the Arduino side, they can connect to any I/O pin, since the current implementation does not use interrupts or other special hardware features (though this might be added in the feature, see also the "Timing" section).

In LoRa mode the DIO pins are used as follows:
 * DIO0: TxDone and RxDone
 * DIO1: RxTimeout

In FSK mode they are used as follows::
 * DIO0: PayloadReady and PacketSent
 * DIO2: TimeOut

Both modes need only 2 pins, but the tranceiver does not allow mapping them in such a way that all needed interrupts map to the same 2 pins. So, if both LoRa and FSK modes are used, all three pins must be connected.

The pins used on the Arduino side should be configured in the pin mapping in your sketch (see below).

### Reset
The transceiver has a reset pin that can be used to explicitely reset it. The LMIC library uses this to ensure the chip is in a consistent state at startup. In practice, this pin can be left disconnected, since the transceiver will already be in a sane state on power-on, but connecting it might prevent problems in some cases.

On the Arduino side, any I/O pin can be used. The pin number used must be configured in the pin mapping (see below).

### RXTX
The transceiver contains two separate antenna connections: One for RX and one for TX. A typical transceiver board contains an antenna switch chip, that allows switching a single antenna between these RX and TX connections.  Such a antenna switcher can typically be told what position it should be through an input pin, often labeled *RXTX*.

The easiest way to control the antenna switch is to use the *RXTX* pin on the SX127x transceiver. This pin is automatically set high during TX and low during RX. For example, the HopeRF boards seem to have this connection in place, so they do not expose any *RXTX* pins and the pin can be marked as unused in the pin mapping.

Some boards do expose the antenna switcher pin, and sometimes also the SX127x *RXTX* pin. For example, the SX1272 evaluation board calls the former *FEM_CTX* and the latter *RXTX*. Again, simply connecting these together with a jumper wire is the easiest solution.

Alternatively, or if the SX127x *RXTX* pin is not available, LMIC can be configured to control the antenna switch. Connect the antenna switch control pin (e.g. *FEM_CTX* on the Semtech evaluation board) to any I/O pin on the Arduino side, and configure the pin used in the pin map (see below). It is not entirely clear why would *not* want the transceiver to control the antenna directly, though.

### Pin mapping
As described above, most connections can use arbitrary I/O pins on the Arduino side. To tell the LMIC library about these, a pin mapping struct is used in the sketch file.

For example, this could look like this:

	lmic_pinmap lmic_pins = {
	    .nss = 6,
	    .rxtx = LMIC_UNUSED_PIN,
	    .rst = 5,
	    .dio = {2, 3, 4},
	};

The names refer to the pins on the transceiver side, the numbers refer to the Arduino pin numbers (to use the analog pins, use constants like `A0`). For the DIO pins, the three numbers refer to DIO0, DIO1 and DIO2 respectively. Any pins that are not needed should be specified as `LMIC_UNUSED_PIN`. The nss is required the others can potentially left out (depending on the environments and requirements, see the notes above for when a pin can or cannot be left out).

The name of this struct must always be `lmic_pins`, which is a special name recognized by the library.

If you don't have any DIO pins connected to GPIO (new software feature) you just need to declare 3 .dio to LMIC_UNUSED_PIN, in your sketch That's all, stack will do the job for you.

#### [WeMos Lora Shield](https://github.com/hallard/WeMos-Lora)
```arduino
// Example with NO DIO pin connected
const lmic_pinmap lmic_pins = {
    .nss = 16,
    .rxtx = LMIC_UNUSED_PIN,
    .rst = LMIC_UNUSED_PIN,
    .dio = {LMIC_UNUSED_PIN, LMIC_UNUSED_PIN, LMIC_UNUSED_PIN},
};
```

If you used 3 diodes OR hardware trick like in this [schematic](https://github.com/hallard/WeMos-Lora), just indicate which GPIO is used on DIO0 definition as follow:

```arduino
// Example with 3 DIO OR'ed on one pin connected to GPIO14
const lmic_pinmap lmic_pins = {
    .nss = 16,
    .rxtx = LMIC_UNUSED_PIN,
    .rst = LMIC_UNUSED_PIN,
    .dio = {15, LMIC_UNUSED_PIN, LMIC_UNUSED_PIN},
};
```

#### LoRa Nexus by Ideetron
This board uses the following pin mapping:

    const lmic_pinmap lmic_pins = {
        .nss = 10,
        .rxtx = LMIC_UNUSED_PIN,
        .rst = LMIC_UNUSED_PIN, // hardwired to AtMega RESET
        .dio = {4, 5, 7},
    };

Examples
--------
This library currently provides three examples:

 - `ttn-abp.ino` shows a basic transmission of a "Hello, world!" message
   using the LoRaWAN protocol. It contains some frequency settings and
   encryption keys intended for use with The Things Network, but these
   also correspond to the default settings of most gateways, so it
   should work with other networks and gateways as well. This example
   uses activation-by-personalization (ABP, preconfiguring a device
   address and encryption keys), and does not employ over-the-air
   activation.

   Reception of packets (in response to transmission, using the RX1 and
   RX2 receive windows is also supported).

 - `ttn-otaa.ino` also sends a "Hello, world!" message, but uses over
   the air activation (OTAA) to first join a network to establish a
   session and security keys. This was tested with The Things Network,
   but should also work (perhaps with some changes) for other networks.

 - `raw.ino` shows how to access the radio on a somewhat low level,
   and allows to send raw (non-LoRaWAN) packets between nodes directly.
   This is useful to verify basic connectivity, and when no gateway is
   available, but this example also bypasses duty cycle checks, so be
   careful when changing the settings.

Timing
------
Unfortunately, the SX127x tranceivers do not support accurate timekeeping themselves (there is a sequencer that is *almost* sufficient for timing the RX1 and RX2 downlink windows, but that is only available in FSK mode, not in LoRa mode). This means that the microcontroller is responsible for keeping track of time. In particular, it should note when a packet finished transmitting, so it can open up the RX1 and RX2 receive windows at a fixed time after the end of transmission.

This timing uses the Arduino `micros()` timer, which has a granularity of 4μs and is based on the primary microcontroller clock.  For timing events, the tranceiver uses its DIOx pins as interrupt outputs. In the current implementation, these pins are not handled by an actual interrupt handler, but they are just polled once every LMIC loop, resulting in a bit inaccuracy in the timestamping. Also, running scheduled jobs (such as opening up the receive windows) is done using a polling approach, which might also result in further delays.

Fortunately, LoRa is a fairly slow protocol and the timing of the receive windows is not super critical. To synchronize transmitter and receiver, a preamble is first transmitted. Using LoRaWAN, this preamble consists of 8 symbols, of which the receiver needs to see 4 symbols to lock on. The current implementation tries to enable the receiver for 5 symbol times at 1.5 symbol after the start of the receive window, meaning that a inacurracy of plus or minus 2.5 symbol times should be acceptable.

At the fastest LoRa setting supported by the tranceiver (SF5BW500) a single preamble symbol takes 64μs, so the receive window timing should be accurate within 160μs (for LoRaWAN this is SF7BW250, needing accuracy within 1280μs). This is certainly within a crystal's accuracy, but using the internal oscillator is probably not feasible (which is 1% - 10% accurate, depending on calibration). This accuracy should also be feasible with the polling approach used, provided that the LMIC loop is run often enough.

It would be good to properly review this code at some point, since it seems that in some places some offsets and corrections are applied that might not be appropriate for the Arduino environment. So if reception is not working, the timing is something to have a closer look at.

The LMIC library was intended to connect the DIO pins to interrupt lines and run code inside the interrupt handler. However, doing this opens up an entire can of worms with regard to doing SPI transfers inside interrupt routines (some of which is solved by the Arduino `beginTransaction()` API, but possibly not everything). One simpler alternative could be to use an interrupt handler to just store a timestamp, and then do the actual handling in the main loop (this requires modifications of the library to pass a timestamp to the LMIC `radio_irq_handler()` function).

An even more accurate solution could be to use a dedicated timer with an input capture unit, that can store the timestamp of a change on the DIO0 pin (the only one that is timing-critical) entirely in hardware. Unfortunately, timer0, as used by Arduino's `millis()` and `micros()` functions does not seem to have an input capture unit, meaning a separate timer is needed for this.

If the main microcontroller does not have a crystal, but uses the internal oscillator, the clock output of the transceiver (on DIO5) could be usable to drive this timer instead of the main microcontroller clock, to ensure the receive window timing is sufficiently accurate. Ideally, this would use timer2, which supports asynchronous mode (e.g. running while the microcontroller is sleeping), but that timer does not have an input capture unit. Timer1 has one, but it seems it will stop running once the microcontroller sleeps. Running the microcontroller in idle mode with a slower clock might be feasible, though. Instead of using the main crystal oscillator of the transceiver, it could be possible to use the transceiver's internal RC oscillator (which is calibrated against the transceiver crystal), or to calibrate the microcontroller internal RC oscillator using the transceiver's clkout. However, that datasheet is a bit vague on the RC oscillator's accuracy and how to use it exactly (some registers seem to be FSK-mode only), so this needs some experiments.

Downlink datarate
-----------------
Note that the datarate used for downlink packets in the RX2 window defaults to SF12BW125 according to the specification, but some networks use different values (iot.semtech.com and The Things Network both use SF9BW). When using personalized activate (ABP), it is your responsibility to set the right settings, e.g. by adding this to your sketch (after calling `LMIC_setSession`). `ttn-abp.ino` already does this.

     LMIC.dn2Dr = DR_SF9;

When using OTAA, the network communicates the RX2 settings in the join accept message, but the LMIC library does not currently process these settings. Until that is solved (see issue #20), you should manually set the RX2 rate, *after* joining (see the handling of `EV_JOINED` in the `ttn-otaa.ino` for an example.

License
-------
Most source files in this repository are made available under the Eclipse Public License v1.0. The examples which use a more liberal license. Some of the AES code is available under the LGPL. Refer to each individual source file for more details.





