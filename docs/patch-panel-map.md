# Patch Panel and I-PAC Assignment Map

The physical assignment sheet: which patch panel port carries which control, which I-PAC board and
input it lands on, and which LED channels light it. [wiring.md](wiring.md) defines the jack types and
the architecture; this document assigns the positions.

Print this one and keep it with the cabinet.

## Conventions

- **Two I-PAC Ultimate I/O boards, one per panel row.** Row 1 (ports 1-24) is board 1: P1, P2, the
  fixed panel, trackball 1 and the spinner. Row 2 (ports 25-48) is board 2: P3, P4 and trackball 2.
  Every label still carries its board: `B1 port 12`.
- **Each board is wired as a generic 2-player board** and software assigns which physical players it
  serves. Board 1's own player 1 and 2 regions serve P1 and P2; board 2's serve P3 and P4. This
  leaves the player 4 region free on **both** boards, which is where the trackball and spinner
  connectors live, so pointing devices never displace a joystick.
- **Port numbers** run 1 to 48 on a 48-port panel, read as two rows of 24. Row 1 is ports 1 to 24,
  row 2 is ports 25 to 48.
- **Jack type and color** follow [wiring.md](wiring.md): J is blue (joystick), P is red (RGB
  pushbutton), Q is green (pointing device).
- **I-PAC input names** use Ultimarc's logical labels. **Confirm every name against the board
  silkscreen and WinIPAC before punching down**, since this map was written from the documented input
  list rather than from the boards in hand.
- **LED-only P-jacks** carry lighting for a control whose own jack has no room for it. Pins 2-5 are
  wired, switch pins 1 and 8 are left empty. **A lit control's LED jack sits immediately to its
  right**, without exception.
- **RGB LED n** occupies that board's LED channels `3n-2`, `3n-1`, `3n`. Numbering restarts per
  board, so `B2 LED 1` is board 2's channels 1 to 3.

## Port map

**Row 1 is board 1, row 2 is board 2.** The panel's two physical rows are the two boards, so a port
number tells you its board without a lookup and the board-side looms never cross rows.

Within a row, ports are grouped **one player per block**, each block running joystick, joystick LED,
six buttons, then a spare. Pulling a player's controls means pulling a contiguous run.

### Row 1, ports 1-24: board 1 (P1, P2, trackball 1, spinner)

| Port | Type | Control | I-PAC input | RGB LED | LED ch |
| ---- | ---- | ------- | ----------- | ------- | ------ |
| 1 | J | P1 joystick | `1UP` `1DOWN` `1LEFT` `1RIGHT` | none | ServoStik board A |
| 2 | P | P1 joystick LED (LED only) | none | 13 | 37-39 |
| 3 | P | P1 button 1 | `1SW1` | 1 | 1-3 |
| 4 | P | P1 button 2 | `1SW2` | 2 | 4-6 |
| 5 | P | P1 button 3 | `1SW3` | 3 | 7-9 |
| 6 | P | P1 button 4 | `1SW4` | 4 | 10-12 |
| 7 | P | P1 button 5 | `1SW5` | 5 | 13-15 |
| 8 | P | P1 button 6 | `1SW6` | 6 | 16-18 |
| 9 | - | spare (P1 block) | none | none | none |
| 10 | J | P2 joystick | `2UP` `2DOWN` `2LEFT` `2RIGHT` | none | ServoStik board A |
| 11 | P | P2 joystick LED (LED only) | none | 14 | 40-42 |
| 12 | P | P2 button 1 | `2SW1` | 7 | 19-21 |
| 13 | P | P2 button 2 | `2SW2` | 8 | 22-24 |
| 14 | P | P2 button 3 | `2SW3` | 9 | 25-27 |
| 15 | P | P2 button 4 | `2SW4` | 10 | 28-30 |
| 16 | P | P2 button 5 | `2SW5` | 11 | 31-33 |
| 17 | P | P2 button 6 | `2SW6` | 12 | 34-36 |
| 18 | - | spare (P2 block) | none | none | none |
| 19 | Q | Trackball 1 | `XA` `XB` `YA` `YB`, top-left connector | none | none |
| 20 | P | Trackball 1 LED (LED only) | none | 15 | 43-45 |
| 21 | Q | Spinner | `A` `B`, top-right connector | none | none |
| 22 | P | Spinner LED (LED only, provisioned) | none | 16 | 46-48 |
| 23-24 | - | spare (aux block) | none | none | none |

### Row 2, ports 25-48: board 2 (P3, P4, trackball 2)

| Port | Type | Control | I-PAC input | RGB LED | LED ch |
| ---- | ---- | ------- | ----------- | ------- | ------ |
| 25 | J | P3 joystick | `1UP` `1DOWN` `1LEFT` `1RIGHT` | none | ServoStik board B |
| 26 | P | P3 joystick LED (LED only) | none | 13 | 37-39 |
| 27 | P | P3 button 1 | `1SW1` | 1 | 1-3 |
| 28 | P | P3 button 2 | `1SW2` | 2 | 4-6 |
| 29 | P | P3 button 3 | `1SW3` | 3 | 7-9 |
| 30 | P | P3 button 4 | `1SW4` | 4 | 10-12 |
| 31 | P | P3 button 5 | `1SW5` | 5 | 13-15 |
| 32 | P | P3 button 6 | `1SW6` | 6 | 16-18 |
| 33 | - | spare (P3 block) | none | none | none |
| 34 | J | P4 joystick | `2UP` `2DOWN` `2LEFT` `2RIGHT` | none | ServoStik board B |
| 35 | P | P4 joystick LED (LED only) | none | 14 | 40-42 |
| 36 | P | P4 button 1 | `2SW1` | 7 | 19-21 |
| 37 | P | P4 button 2 | `2SW2` | 8 | 22-24 |
| 38 | P | P4 button 3 | `2SW3` | 9 | 25-27 |
| 39 | P | P4 button 4 | `2SW4` | 10 | 28-30 |
| 40 | P | P4 button 5 | `2SW5` | 11 | 31-33 |
| 41 | P | P4 button 6 | `2SW6` | 12 | 34-36 |
| 42 | - | spare (P4 block) | none | none | none |
| 43 | Q | Trackball 2 | `XA` `XB` `YA` `YB`, top-left connector | none | none |
| 44 | P | Trackball 2 LED (LED only) | none | 15 | 43-45 |
| 45-48 | - | spare (aux block) | none | none | none |

Port 22 is provisioned but unpopulated, since an illuminated spinner may not exist. Everything else
in both tables is populated.

**All four players have 6 buttons.** The UIO allots 4 directions plus **8 buttons** to its player 1
and 2 positions, but only 4 buttons to its player 3 and 4 positions. On a single board that is what
capped P3 and P4 at four. Because each board here serves two players out of its **player 1 and 2
regions**, all four cabinet players sit in 8-button-capable positions, and the old limit simply
disappears.

Six is the practical maximum for the games this cabinet targets, so `SW7` and `SW8` stay free on all
four positions. **Each block's spare port is that player's 7th button**, already wired to a live
input, so adding one is populating a keystone rather than renumbering the panel.

## RGB allocation

48 RGB LEDs across the two boards, 35 assigned and 13 in reserve. **Every control is lit.** The two
boards use the same internal numbering, which is why every label carries its board.

### Board 1

| RGB LED | Channels | Assigned to | Port |
| ------- | -------- | ----------- | ---- |
| 1-6 | 1-18 | P1 buttons 1-6 | 3-8 |
| 7-12 | 19-36 | P2 buttons 1-6 | 12-17 |
| 13 | 37-39 | P1 joystick LED | 2 |
| 14 | 40-42 | P2 joystick LED | 11 |
| 15 | 43-45 | Trackball 1 LED | 20 |
| 16 | 46-48 | Spinner LED (provisioned) | 22 |
| 17-20 | 49-60 | **Reserve** | none |
| 21-24 | 61-72 | coin 1, coin 2, start 1, start 2 | fixed panel |

Board 1 uses **20 of 24**, with 4 in reserve.

### Board 2

| RGB LED | Channels | Assigned to | Port |
| ------- | -------- | ----------- | ---- |
| 1-6 | 1-18 | P3 buttons 1-6 | 27-32 |
| 7-12 | 19-36 | P4 buttons 1-6 | 36-41 |
| 13 | 37-39 | P3 joystick LED | 26 |
| 14 | 40-42 | P4 joystick LED | 35 |
| 15 | 43-45 | Trackball 2 LED | 44 |
| 16-24 | 46-72 | **Reserve** | none |

Board 2 uses **15 of 24**, with 9 in reserve.

## Aux region

There are **no exclusions**. Because each board serves only two players, the player 4 region on each
board is unused, and that is exactly where the trackball (top-left) and spinner (top-right)
connectors live. Every joystick, every button and every pointing device can be live at once.

| Aux port | Board | Uses |
| -------- | ----- | ---- |
| 19 | B1 | top-left connector, 4 inputs |
| 21 | B1 | top-right connector, 2 inputs |
| 43 | B2 | top-left connector, 4 inputs |

**Two trackballs need two boards.** Each Ultimate I/O has exactly one trackball connector, so this is
the reason both boards are in the cabinet, independent of whether P3 and P4 ever get wired. Each
board enumerates as its own mouse, which is the two-mouse arrangement Marble Madness wants.

Board 2's top-right connector is unused and free for a second spinner if one is ever wanted.

## Direct-wired fixed panel

These seven buttons are **not** patch panel ports. They are hard-wired from the fixed panel to
**board 1**, with the RGB leads running directly to that board's LED section. See
[Fixed cabinet controls](wiring.md#fixed-cabinet-controls).

| Button | I-PAC input | RGB LED | LED ch |
| ------ | ----------- | ------- | ------ |
| Coin 1 | `coin1` | B1 21 | 61-63 |
| Coin 2 | `coin2` | B1 22 | 64-66 |
| Start 1 (shift key) | `start1` | B1 23 | 67-69 |
| Start 2 | `start2` | B1 24 | 70-72 |
| Escape | `1a` | none | B1 73 (single-color) |
| Pause | `1b` | none | B1 74 (single-color) |
| Menu | `2a` | none | B1 75 (single-color) |

## Spare capacity

| Resource | Spare | Which |
| -------- | ----- | ----- |
| Patch panel positions | 10 | ports 9, 18, 23-24, 33, 42, 45-48 |
| Board 1 inputs | 15 | `1SW7` `1SW8` `2SW7` `2SW8`, the unused player 3 region, 2 player 4 buttons, `2b` |
| Board 2 inputs | 24 | `1SW7` `1SW8` `2SW7` `2SW8`, the unused player 3 region, 4 player 4 buttons, all 8 named inputs |
| RGB LEDs | 13 | 4 on board 1, 9 on board 2 |
| Single-color LED channels | ample | board 1 has 21 free after admin; board 2 has all of its own |

USB-attached controls consume none of these. See
[USB-attached controls](wiring.md#usb-attached-controls).

## Panel headroom

38 positions are populated, leaving **10 spare** distributed through the blocks rather than pooled at
the end, so a player can gain a control without renumbering anything downstream.

Budget two positions for every lit control, one for the control and one for its LED. The panel fills
roughly twice as fast as the control count suggests, which is why the 10 spares are worth less than
they look.

USB-attached controls consume no panel positions, so the hub absorbs growth the patch field cannot.

## Build phases

Board 1 is a complete, playable cabinet on its own. Wire it first and verify it before starting
board 2.

| Phase | Ports | What it gets you |
| ----- | ----- | ---------------- |
| 1 | **row 1** (1-24) | P1, P2, the fixed panel, a trackball and a spinner. Most of the library. |
| 2 | **row 2** (25-48) | P3, P4 and the second trackball. Four-player games and Marble Madness. |

The phases are the two rows, which is also the two boards.

If phase 2 never happens, board 2 still earns its place by hosting trackball 2, which no single board
can provide.

## Verification

Work through this map alongside the checklist in
[wiring.md](wiring.md#verification-bench-first-before-cabinet-install). Three checks specific to this
document:

- **Confirm the I-PAC input names against each board's silkscreen** before punching down anything.
- **Confirm the LED channel numbering** by lighting channel 1 and channel 96 on each board from the
  Ultimarc utility and noting which physical output moves, before trusting the `3n-2, 3n-1, 3n`
  grouping.
- **Confirm the two boards send non-overlapping key codes** and enumerate as two distinct devices
  before wiring the second one into the panel.
