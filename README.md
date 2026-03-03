# FSKGroundStation

This is a hardware design for a low-cost PacSat satellite ground
station, but it is also general two-way FSK commmunication for the 2
meter and 70 centimeter bands.  It should be possible to build these
in the US $50 range.

Basically, this is two radios, one for around 136-175MHz, and one for
around 420-460MHz.  Each of these radios is a simplex two-way radio
using a basic ISM radio chip.  It then has a PA on the TX side to
boost the power to the 4-5 watt range and an RF switch after than to
switch between transmit and receive.

## Host Communication

Communication with the host is through a SPI/UART to USB converter on
J3.

## RF Connections

RF input/output is through SMA connectors on the end of the board.
One connector is for 2 meters only, one is for 70cm only, and one has
a diplexer on it and can be used with a dual-band antenna.  You lose
about 1dB in the diplexer, so it's slightly better to have two
antennas.

There are two open-collector (open drain, really) outputs on a 3.5mm
audio-type jack that the CPU can drive for controlling external
amplifiers or for anything else, really.

Note that you really can't transmit on both bands at the same time.
Power draw would go somewhat above 4A and heat dissipation would be a
problem.  If you have a power source that can supply that it might be
ok, but it's pushing things on the heat side.

## Powering the Board

Two USB connectors on the board can both supply power to the board.
To use the host connector (J3), install the jumper on J8.  A second
USB connector J10 is a power-only USB connector that can be used to
power the board if the host connection cannot supply enough current.
To use that install the J11 jumper.

There is also J25, a standard 6.0mm OD 2.5mm ID barrel connector on
the board, center pin is +5V and the outside is ground, as usual.  You
must remove the other power jumpers (J8 and J11) to use this.

All power connectors must supply at least 3A for transmit.

## Heat Dissipation

The PAs produce a lot of heat.  At full power they will be dissipating

## Design Variants

Lots of things can be removed if not required, which will decrease the
price and allow slight performance improvements in some cases:

* The 3.5mm external PA control jack and associated circuits can be
  removed if not necessary.

* PacSat satellites transmit on 70cm and receive on 2m, so the ground
  station for that really only has to transmit on 2m and receive on
  70cm.  The 70cm transmit circuits could be removed and the 2m
  receive circuits could be removed.  That means you can remove the RF
  switches on the, gaining about 1dB.  That, of course, makes the
  board less flexible for other purposes.
  
* The diplexer and connector can be removed if not needed.

* If only the diplexer is used, the TX and RX connectors can be removed.

* The radios can be modified to different bands up to the capabilities
  of the various parts.  The limiting factor is the bands supported by
  the radios (TI CC1312R and CC1125 chips) and the capabilities of the
  PA (Guerrilla RF GRF5604, 100-600MHz).

* The power connectors that aren't used can be removed.
