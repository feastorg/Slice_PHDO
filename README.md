# 2-Channel Atlas EZO Carrier (ACAR)

A BREAD Slice that carries two Atlas Scientific EZO circuits, each on an
Atlas Electrically Isolated EZO Carrier Board Gen 2. Any EZO circuit the
carrier supports fits a channel: pH, ORP, dissolved oxygen, conductivity,
RTD temperature and others. The carrier provides the galvanic isolation
and the probe's SMA connector.

The repository keeps its original name, Slice_PHDO, from the 2022 pH and
dissolved-oxygen board it grew out of.

Each carrier is screwed flat on its own standoffs (A2, A3: four holes,
no pins) and connects through the 5-wire Dupont cable it ships with, to a
5-pin male header on the Slice (J3, J4). The header pins follow the
carrier's own header: VCC, OFF, GND, TX/SDA, RX/SCL.

Use 6 mm standoffs (#4-40 or M3), not the 11 mm ones in the Atlas kit.
On 6 mm the carrier with its EZO stands about 18.8 mm above the Slice,
inside the 21.6 mm the slice case leaves above the board. On 11 mm it
would stand about 23.8 mm and hit the case roof.

> **Status:** schematic only (first revision, G3 r1). The board has not
> been laid out, built or tested.

## Modes

Each channel has a slide switch with two positions. The one switch sets
both the carrier's supply and where its two data lines go.

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

### Changing a channel's mode

1. On the bench, set the EZO's protocol: `I2C,n` for direct mode, or
   `Baud,n` for proxy mode (Atlas EZO datasheets).
2. Power the Slice off.
3. Move the channel's switch.

An EZO left in UART mode on the slot bus drives SDA from its TX line and
can hang the bus.

Two direct-mode carriers on one bus add two 4.7 kOhm pull-ups to the
host's own. Keep to at most two direct-mode carriers per bus.

## Channel pin map (Arduino Nano)

| Nano pin | Net | Use |
| --- | --- | --- |
| D2 | E_STOP | slot bus |
| D3 | INT | slot bus |
| D4 | SYNC | slot bus |
| A4 / A5 | I2C_DAT / I2C_CLK | slot bus I2C |
| D5 | CH1_OFF | channel 1 carrier OFF |
| D6 | CH2_OFF | channel 2 carrier OFF |
| D7 | CH1_TX | Nano TX to channel 1 carrier RX (proxy) |
| D8 | CH1_RX | channel 1 carrier TX to Nano RX (proxy) |
| D9 | CH2_TX | Nano TX to channel 2 carrier RX (proxy) |
| D10 | CH2_RX | channel 2 carrier TX to Nano RX (proxy) |
| D12 | LED | WS2812 status LED |

D5 to D10 keep the 2022 board's pin choices. The status LED, on D5 in the
Slice template, moves to D12.

OFF has no pull-up on this board; the carrier pulls it up itself. Drive it
open-drain: output LOW turns the carrier and its EZO off, input (high
impedance) turns them on. Do not drive it high: a 5 V high would reach a
3.3 V carrier in direct mode.

## Power

- U1 (AP63205) makes +5V from the bus +12V: the Nano, the LED, proxy-mode
  carriers and U2.
- U2 (AP7361C-33E LDO) makes +3V3 from +5V: direct-mode carriers and the
  Qwiic jack.

Per the Atlas carrier datasheet, a carrier with an EZO draws 53 to 82 mA
at 5 V and 84 to 97 mA at 3.3 V, depending on the circuit.

## Provenance

The circuit is re-entered from **SLC_PHDO R0 2022**, the BREAD pH and
dissolved-oxygen carrier published on OSF:
[osf.io/zbg98](https://osf.io/zbg98/). That board had two carrier sockets
with I2C/UART jumpers and ran both carriers from 5 V. This revision keeps
its socket order and Nano pins and adds the switched supply, the 3.3 V
rail, local decoupling and the G2 S-format template (12 V buck, slot bus,
mounting).

## Documentation

Please refer to the [docs](/docs) for more info.

---

<div align="center">
  <p>
    <img src="https://img.shields.io/badge/Developed_by-UWO--FAST--et--al-orange" alt="Developed by UWO-FAST et al.">
    <img src="https://img.shields.io/badge/Powered_by-Open_Source-blue" alt="Powered by Open Source">
  </p>
</div>
