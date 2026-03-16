Notes on the FSK Ground Station
===============================

This keeps track of history, general information, things that need to
be done, and things that have been done.

The general information will probably make it into another document at
some point.

# TODO

When a smaller version of the MSPM0G5187 comes out, switch to it.

# Done

Possibly replace the USB chip with a MSPM0G5187.  It's a
microprocessor with a USB interface, and it's actually cheaper than
the CY7C65215-32LTXI and it frees us from using lousy libraries from
the vendor and proprietary programming tools.

# Not going to do

# Chip Selection and Possibilities

PA final: PD55015 15W 15dB gain, 12.5V, could be pushed a bit

PA stage 1: GRF5030, GRF5040, GRF5112, GRF5115, etc.  Some Qorvo chips
would work.

GRF5604 is capable of 6 watts (38dBm, 37dB gain) and it is a two-stage
design, so only one chip is needed to get to 6 watts.  This was chosen
for now.

For the radios, a large number of ISM chips are available, but few
will do 144-149 MHz.  The TI CC1312R and C1352R are Cortex M4F CPUs
with an attached radio.  They cannot do HDLC, though.

The TI CC12xx series of radios are similar to the CC1312R but stand
alone radios.  They will not directly do HDLC or the G3RUH whitening,
but they have something called "Synchronous serial mode" that appears
to put/get the FSK directly on GPIO pins, so you could do HDLC on an
external chip.  It also has something called "Transparent mode" that
appears to give direct access to ADC info and the TX synthesizer.

The Silicon Labs 446x series of radio chips is very similar to the TI
CC12xx series.  It has "direct mode" which inputs/outputs FSK and "raw
data mode" which is the ADC and TX synthesizer access.

For bit handling (HDLC and whitening), the Lattice iCE40 LP/HX series
of FPGAs looks good.  Probably the LP640 or the LP1K.

For a USB interface, FTDI has a bunch of chips that look workable.
They are on the expensive side, though.

Infineon has some chips, too, and CY7C65215-32LTXI looks good.  It has
a dual interface that can do UART, SPI, and I2C.  The plan is to do
UART and SPI for the host interface.  One problem is that the various
chips (from FTDI, Microchip, Infineon) have very poor documentation on
how to program them; they expect you to use their libraries

TI and some others make a chip with USB that could be used, too.  It's
cheaper, and it doesn't tie us to proprietary tools.

If we use the CC1352R, it can also do Bluetooth, which might be nice
for a host interface.  It doesn't cost a lot more than the CC1312R.

If we have two independent radio chips, then other non-radio
processors can be used.  The Cortex-M0+ processors could be used,
though they are limited in RAM, but they cost less than $1.
Cortex-M4F processors would be better.  It turns out it's cheaper to
buy the CC1312R or CC1352R than just a plain Cortex-M4 from TI.
And the radio chips have more flash and RAM.

But even better, the TI CC2755 chips are cheaper and have even more
FLASH and RAM.  And they can do bluetooth.

# History

## 2026-03-06

Initial basic design was done a few days ago.  Original design was a
CC1312R processor+radio for 144MHz and a CC1125 radio for 440MHz, and
two GRF5604 PAs.  The host interfaced is a CY7C65215-32LTXI.

However, I have discovered that the ISM chips that are cheap and easy
to use will not do HDLC or the G3RUH whitening.  I've also learned
some information about G3RUH:

I had always been a little confused about g3ruh modulation based on my
understanding of how modulation works and what g3ruh does.  I had
heard that g3ruh was GMSK.  That's not true, it's GFSK.  I had
wondered how it knew where the zero crossings were.  There's actually
a note in the PacSat software saying that GMSK didn't work on receive
and they had to use GFSK to receive.  And now I know why.  You can
transmit GMSK and a GFSK receiver will receive it, but you cannot
transmit GFSK and have a GMSK receiver receive it.  Unfortunately,
there's no really good explanation about g3ruh, I had to reverse
engineer it from the hardware design.

I then spent some time looking at the ISM radios.  It turns out they
don't support HDLC and lack a compatible whitening algorithm.  If you
remember I said that MSK doesn't require periodic bit shifts, you can
use zero crossings to keep the receiver in alignment.  And that's what
they have done.  They generally transmit a 32-bit sequence to start a
packet, and if the receiver sees that it assumes the start of a packet
has happened.  Not as robust as HDLC, really.  So they won't talk to a
g3ruh radio.  In fact, I don't think they will talk to an ax5043 at
all in any mode.

However, all is not lost.  I did find some radios that will
input/output a raw FSK stream to a pin.  One is already in my current
board design.  You would need to add an FPGA to take that data stream
and do the whitening and HDLC.  It will raise the cost of the board a
bit, $3 or so, but that's not a big deal.  You will also need another
separate radio chip.

I have some open questions out to TI on this, but it looks doable.

So looking at a redesign, it's the CC2755R10 chip for the main
processor, two CC1125 chips for radios, and an iCE40 LP1K for an FPGA.
Still two GRF5604 PAs and a CY7C65215-32LTXI host interface.

## 2026-03-10

Finished the rework for two radio chips and the FPGA.

TI confirmed that the synchronous output of the CC115 can be used to
provide our own whitening and HDLC.  The chip does not do differential
coding, so that will have to be accounted for in the FPGA, too.

## 2026-03-11

Change the voltage regulator to a diode for 2.5V to the FPGA.

Switch to a CPU with a USB interface for the USB connection The part
that was picked had proprietary tools and configuration and it didn't
support interrupts.  The CPU is cheaper and more capable.
Unfortunately it's a lot bigger.  A newer smaller part may come out at
some time, switch then.

## 2026-03-12

Add a clock line from the USB chip to the FPGA so the FPGA can have a
clock.

Split the reset lines so that the USB chips controls the main CPUs
reset line.  This way they don't share the same reset, which should
simplify things.

## 2026-03-15

Add DNP resistors on the unused pins of the CPU and the USB chip.
