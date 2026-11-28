# 1-Channel Atlas EZO Carrier (ACAR S2L)

A BREAD Slice that carries one Atlas Scientific EZO circuit on an Atlas
Electrically Isolated EZO Carrier Board Gen 2. Any EZO circuit the
carrier supports fits: pH, ORP, dissolved oxygen, conductivity, RTD
temperature and others. The carrier provides the galvanic isolation and
the probe's SMA connector.

Slice_ACAR_S2L is the small (S, 70 x 100 mm), 2-layer, one-channel
board, with the Nano at the fleet's standard position. The two-channel
board is Slice_ACAR_L2L (L format). Both grew out of the 2022 pH and
dissolved-oxygen board SLC_PHDO.

> **Status:** laid out, not built or tested (first revision, G3 r1).

## Fitting the carrier

The carrier lies flat on four standoffs over the top right of the
Slice (A2: four holes, no pads), turned so its SMA jack points toward
the front panel and down, toward the Nano's USB side. The carrier is
modified before it goes on:

1. **Take the kit header off.** Desolder the right-angle 1x05 header
   the carrier ships with.
2. **Fit a new header on the bottom.** Solder a stock 1x05 2.54 mm
   right-angle male header to the carrier's **bottom** side, in the
   same holes, pins pointing inward (toward the middle of the
   carrier), with its plastic about 1 mm off the board.
3. **Cut the rear socket tails.** Cut the three tails of the rear EZO
   socket row flush with the carrier's bottom (0.5 mm or less), so
   the cable housing passes under them.
4. **Use 6 mm standoffs** (#4-40 or M3), not the 11 mm ones in the
   Atlas kit. On 6 mm the carrier with its EZO stands about 18.8 mm
   above the Slice, inside the 21.6 mm the slice case leaves above
   the board; on 11 mm it would stand about 23.8 mm and hit the roof.

The Dupont housing on the new header then lies under the carrier, in
the 6 mm gap. The five wires turn toward the carrier's DC/DC
converter side, pass under the converter and rise to J3 beside the
carrier. The board keeps the space under the carrier free: no part
inside the carrier's outline, Slice parts at most 2.1 mm tall under the
housing and at most 1.8 mm under the wire path.

### The cable

J3 is a vertical 1x05 male header beside the carrier, its row turned
to the carrier's angle, in the carrier header's order: VCC, OFF, GND,
TX/SDA, RX/SCL (pin 1 nearest the board's top edge).

Requirement: a 5-wire female-female jumper, 2.54 mm Dupont housings,
26 AWG or finer, about 3 inches (75 mm) long. Candidate: Pololu's
3-inch premium F-F jumper wires, or their 10-wire ribbon cable
(item 4560) peeled down to 5 wires. The kit's own cable is not used.

### Probe plug and the front panel

The SMA jack and the mated probe plug (about 21 mm long) stay inside
the case, behind the sliding front panel. The probe coax turns inside
the case on a radius of about 15 mm and leaves through a notch in the
front panel:

| Notch | Slot frame | Board frame (x from the bus edge, y from the top edge) |
| --- | --- | --- |
| centre | A -32.7 | y 82.7, at the front edge (x 70) |
| width | 5.5 mm along A | y 79.95 to 85.45 |
| height | open bottom to N 9.0 | about 10.3 mm above the top copper |

The slice case does not cut this notch yet. Its tooling
(bread-mechanical `slice-case/design/synth/tools/resolve.py`) makes a
front-panel notch only for a top-side part whose courtyard crosses the
board's front edge; this board has none (the jack and plug stay
inside), and there is no enclosure modifier for a panel notch (the
modifiers are `wall_notches`, `floor_reliefs` and `roof_windows`, for
the tray). The front-zone constraints, and a way for a board to declare
a cable notch, are pending
[bread-infra#43](https://github.com/feastorg/bread-infra/issues/43).
Until then, cut the notch above in a stock panel by hand.

## Modes

A slide switch (SW1) with two positions sets both the carrier's supply
and where its two data lines go.

| Switch | Carrier supply | Carrier TX/SDA, RX/SCL go to | Who talks to the EZO |
| --- | --- | --- | --- |
| A, direct | 3.3 V | the slot bus SDA and SCL | the host (Loaf controller), over I2C |
| C, proxy | 5 V | the Nano's UART pins | the Nano, over UART |

The carrier pulls its data lines up to its own supply through 4.7 kOhm. A
carrier on the slot bus therefore always runs at 3.3 V, so it never pulls
a 3.3 V host's SDA and SCL up toward 5 V. A pull-up on the switch's control
line selects the UART side if the switch is between positions, which keeps
a carrier off the slot bus.

Direct mode needs no Slice firmware: the host reads the EZO at its own I2C
address. Proxy mode needs Nano firmware that relays EZO traffic; it does
not exist yet.

### Changing the mode

1. On the bench, set the EZO's protocol: `I2C,n` for direct mode, or
   `Baud,n` for proxy mode (Atlas EZO datasheets).
2. Power the Slice off.
3. Move SW1.

An EZO left in UART mode on the slot bus drives SDA from its TX line and
can hang the bus.

A direct-mode carrier adds its 4.7 kOhm pull-ups to the host's own.
Keep to at most two direct-mode carriers on one bus, across all
Slices.

## Pin map (Arduino Nano)

| Nano pin | Net | Use |
| --- | --- | --- |
| D2 | E_STOP | slot bus |
| D3 | INT | slot bus |
| D4 | SYNC | slot bus |
| A4 / A5 | I2C_DAT / I2C_CLK | slot bus I2C |
| D5 | CH1_OFF | carrier OFF |
| D7 | CH1_TX | Nano TX to carrier RX (proxy) |
| D8 | CH1_RX | carrier TX to Nano RX (proxy) |
| D12 | LED | WS2812 status LED |

D5, D7 and D8 keep the 2022 board's channel-1 pins; D6, D9 and D10
(its channel 2) are free here and carry channel 2 on Slice_ACAR_L2L.
The status LED, on D5 in the Slice template, moves to D12.

OFF has no pull-up on this board; the carrier pulls it up itself. Drive it
open-drain: output LOW turns the carrier and its EZO off, input (high
impedance) turns them on. Do not drive it high: a 5 V high would reach a
3.3 V carrier in direct mode.

## Power

- U1 (AP63205) makes +5V from the bus +12V: the Nano, the LED, the carrier in
  proxy mode and U2.
- U2 (AP7361C-33E LDO) makes +3V3 from +5V: the carrier in direct mode
  and the Qwiic jack (J1).

Per the Atlas carrier datasheet, a carrier with an EZO draws 53 to 82 mA
at 5 V and 84 to 97 mA at 3.3 V, depending on the circuit.

## Provenance

The circuit is re-entered from **SLC_PHDO R0 2022**, the BREAD pH and
dissolved-oxygen carrier published on OSF:
[osf.io/zbg98](https://osf.io/zbg98/). That board had two carrier sockets
with I2C/UART jumpers and ran both carriers from 5 V. This revision keeps
its socket order and Nano pins and adds the switched supply, the 3.3 V
rail, local decoupling and the G3 S-format Nano template, r2 (12 V buck,
slot bus links, option-B Nano supply, mounting).

## Documentation

Please refer to the [docs](/docs) for more info.

---

<div align="center">
  <p>
    <img src="https://img.shields.io/badge/Developed_by-UWO--FAST--et--al-orange" alt="Developed by UWO-FAST et al.">
    <img src="https://img.shields.io/badge/Powered_by-Open_Source-blue" alt="Powered by Open Source">
  </p>
</div>
