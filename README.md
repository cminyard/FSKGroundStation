# FSKGroundStation

This is a hardware design for a low-cost PacSat satellite ground
station, but it is also general two-way FSK commmunication for the 2
meter and 70 centimeter bands.  It should be possible to build these
in the US $60 range.  It can do FSK variants ([G]MSK, [G]FSK, OOK, ASK).
Notably, it cannot do AFSK, the normal 1200bps communication used over
FM radios.

Basically, this is two radios, one for around 136-175MHz, and one for
around 430-450MHz.  Each of these radios is a simplex two-way radio
using a basic ISM radio chip.  It then has an RF power amplifier (PA)
on the TX side to boost the power to the 4-5 watt range and an RF
switch after than to switch between transmit and receive.

## Host Communication

Communication with the host is through a SPI/UART to USB converter on
J3.

There is also a 2.4GHz radio, so a bluetooth connection could be done.

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

You can use either of the two USB connectors on the board to supply
power.  To use the host connector (J3), install the jumper on J8.  A
second USB connector J10 is a power-only USB connector that can be
used to power the board if the host connection cannot supply enough
current.  To use that install the J11 jumper.

There is also J25, a standard 6.0mm OD 2.5mm ID barrel connector on
the board, center pin is +5V and the outside is ground, as usual.  You
must remove the other power jumpers (J8 and J11) to use this.

All power connectors must supply at least 3A for transmit.

## Heat Dissipation and Measurement

The PAs produce a lot of heat.  At full power they will be dissipating
around 4 watts.  The heat mostly comes out through the bottom of the
board on a 2.5mm square copper pad under each PA.

There are a bunch of vias on the edge of the board and three
unobstructed ground planes between these vias and the PAs.  With edge
plating, this should be able to move heat from the ground planes into
the chassis from the edge.  This is experimental.

In case the edge vias and plating don't work, the 2.5mm square copper
pad under the PAs is left bare.  That way a thermally conductive pad
or a heat sink can be put on that to move heat away.

There are two thermsistors on the board, one mounted by each PA, for
measuring temperature.

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

## A Box

This was designed to go into a 100mm x 88mm box like:

    JIUWU Surface Drawing Split Aluminum Enclosure Project Box
	Electronic Enclosure Case for PCB Board DIY, 88x38x100mm (WHL),
	Black

on Amazon.  The actual board dimensions are 83mm x 100mm, it should
slide into the slots in the chassis, which is supposed to be 84mm
wide.

## TODO list

* Figure out how to handle heat from the PAs.

  * Screw mount on the bottom.  This would probably require putting
    the external connectors on the bottom of the board to give space
    in a box.  It would also require ventilation of some type and
    maybe a fan.
	
  * Connect a heat transfer device to the box surface.  This would
    require mounting in a box.  If you use the box referenced above,
    there is a 5.2mm gap between the bottom of the circuit board and the
    inside of the box.  I'm not sure how to conduct the head, though.

  * Wurth has some "Graphite Foam Gaskets" that have high thermal
    conductivity and are not electrically conductive, from what I can
    tell (1KV/mm dialectric strength), and not too expensive.  Laird
    has "Thermal Transfer Gaskets," which look similar.  Also
    Bergquist thermally conductive gap pads.  You could put that
    between the PA pad and the chassis.  I can't find one that's the
    right height, though, it needs to be 5.2mm.  Maybe it can be cut?
    The custom make them, but that would be expensive.

  * It would be possible to use 2oz copper on the inner and/or outer
    layers.  This will conduct a lot more heat, but limits the board
    stacks that can be used and requires some impedance adjustments.

  * The edges of the board could be plated, allowing heat transfer out
    of the edges of the board into the chassis.

* The 2m connection to the CC1312R is actually for 169MHz.  A proper
  design for 145MHz needs to be found, or it will need to be tweaked.
