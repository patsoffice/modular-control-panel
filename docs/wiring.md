# Modular RJ45 Arcade Control Wiring: Build Plan

## Context

Goal: wire arcade controls to **two Ultimarc I-PAC Ultimate I/O boards** so that entire control panels are
modular and pluggable. Swapping between game-accurate layouts (up to **4 players / 4 joysticks**,
including twin-stick like Smash TV, plus **two trackballs and a spinner**) means
unplugging patch cables, with no soldering. The interconnect uses **RJ45 (Cat6a) connectors through
a patch panel**.

The electronics ride on the **back of the control panel housing**, so the cabinet seam is a short
umbilical rather than a loom. See [Board-side wiring](#board-side-wiring).

### Board facts that drive the design

Per board, doubled across the two:

- **48 switch inputs**, each a dedicated pin, with a **common ground**. Ultimarc supplies per-input
  ground wiring, and every switch closes between its pin and a shared GND, so only **one ground
  conductor per cable** is needed.
- **96 LED channels = 24 RGB LEDs**, constant-current **sinks** fed from **+5V**. Each RGB LED
  shares a **+5V anode** and the board pulls R/G/B to ground. No per-LED resistors needed.
- Switch closures are logic-level milliamps, so **wire gauge, length, and EMI are non-issues**.
  Cat6a 23 AWG is far more than enough. Cat6a is chosen for build quality, keystone availability and
  headroom rather than for any electrical requirement, and its heavier construction has handling
  consequences (see [Hardware / materials](#hardware--materials)).

### Decisions locked in

- **One control per RJ45.** Low wire density per jack keeps wiring and labeling easy, rather than
  cramming many buttons onto one cable. Lighting that will not fit alongside its control takes a
  second, LED-only jack rather than being crammed in; the rule is one *thing* per cable, not one
  cable per control.
- **The LED rides the button's own jack**, with no separate LED patch domain. A lone switch wastes
  an 8-conductor cable; adding that button's RGB LED fills it (switch signal + GND, LED anode +
  R/G/B = 6 conductors). A switch and its lighting are one self-contained plug.
- **Illuminated joysticks use a second, LED-only P-jack.** The J-jack has no room for LED
  conductors, so a lit stick is two jacks: switches and servo on its J-jack, RGB on a P-jack
  populated for LED only. Costs 1 patch port and 1 RGB LED per stick. See
  [Lit joysticks and trackballs](#lit-joysticks-and-trackballs-a-second-led-only-p-jack).
- **Up to 4 players / 4 joysticks.** Four J-jack positions cover 4 single-stick players or 2
  twin-stick players like Smash TV. Layouts vary per game, so swaps happen at the **control-panel**
  level. At 4 players the board is essentially full (see [Sizing](#sizing-two-ultimate-io-boards)),
  and the fixed field is capped accordingly.
- **Two Ultimate I/O boards**, each wired as a generic 2-player board. Board 1 serves P1, P2, the
  fixed panel, trackball 1 and the spinner; board 2 serves P3, P4 and trackball 2. Each board has
  exactly **one trackball connector**, so two boards is the only way to read two trackballs natively,
  which Marble Madness needs. This holds even if P3 and P4 are never wired.
- **Trackball and spinner are in scope, and are separate from each other.** Each pointing device gets
  its **own Q-jack**, one device per jack. Games do not come in fixed trackball-plus-spinner pairs:
  Missile Command wants one trackball, Marble Madness wants two, Tempest wants a spinner and no
  trackball. A trackball takes 4 inputs on its board's top-left connector and a spinner 2 on the
  top-right. Because each board serves only two players, **its player 4 region is free**, so pointing
  devices never displace a joystick. See
  [Trackball and spinner](#trackball-and-spinner-the-aux-region).
- **Typed jacks**, with distinct joystick and button pinouts keyed by color so you cannot mis-plug.
  There is no universal interchangeable jack.
- **ServoStik 4-way/8-way restrictor support** on the joysticks. Each stick uses a **2-wire**
  connection to Ultimarc's **USB ServoStik control board** (1 board drives 2 sticks). Software
  (MAME/RetroFE) commands the restriction per game; the I-PAC UIO does not drive it. The 2 wires
  **ride the J-jack's spare pins**, so a joystick stays a single clean plug.
- **Six buttons for every player, symmetrically.** 24 buttons total on 24 P-jacks. The UIO allots its
  inputs as 4 directions plus **8 buttons** for its player 1 and 2 positions, but only 4 buttons for
  its player 3 and 4 positions. Because each board serves just two players out of its **player 1 and
  2 regions**, all four cabinet players sit in 8-button-capable positions. Six is the practical
  maximum for the games this cabinet targets, so `SW7` and `SW8` stay free on all four, which is
  where a 7th button would go with no renumbering.
- **USB-attached controls bypass the patch field entirely.** Some controls bring their own USB
  interface, starting with the Star Wars yoke repro, and more are expected. They connect to a USB hub
  in the control panel housing rather than to a typed jack, and consume no I-PAC inputs, no RGB, and
  no RJ45 jacks. See [USB-attached controls](#usb-attached-controls).
- **Direct-wired (non-modular) cabinet controls: seven buttons on a fixed panel.** `coin1`, `coin2`,
  `start1` and `start2` are RGB-illuminated. Escape, pause and menu take three of the four aux
  inputs (`1a`/`1b`/`2a`/`2b`) and use single-color channels. `start1` doubles as the I-PAC shift
  key. There are no P3/P4 start buttons. See [Fixed cabinet controls](#fixed-cabinet-controls).

## Architecture: fixed board side and swappable control side

```text
  BACK OF THE CONTROL PANEL HOUSING (everything below travels with the panel)
  +--------------------------------------------------------------+
  |  I-PAC UIO board 1        I-PAC UIO board 2                   |
  |  (P1, P2, fixed panel,    (P3, P4, trackball 2)               |
  |   trackball 1, spinner)                                       |
  |        |                        |                             |
  |        |  discrete conductors in 3 looms, plus a GND bus       |
  |        |  and a +5V anode bus                                  |
  |        v                        v                             |
  |  +--------------------------------------------------------+  |
  |  |  48-PORT PATCH FIELD  (J + P + Q, punched down ONCE)    |  |
  |  +--------------------------------------------------------+  |
  |        |  ServoStik boards A and B, USB hub mounted alongside  |
  +--------|-------------------------------------------------------+
           |  short Cat6a patch cables: THE PLUGGABLE SEAM
           v
  +------------------------------+
  |  CONTROL-PANEL JACKS         |  RJ45 keystones on each removable control panel
  +------------------------------+
           |  short pigtails
           v
   joysticks (switch + servo) / RGB buttons (switch + LED) / pointing devices (Q)
   one thing per cable

  To the cabinet: one USB lead per board, one from the hub, and power. That is the
  entire umbilical, so the panel lifts with everything still attached.
```

Swapping a whole control panel means unplugging a handful of patch cables at the seam. Each control
panel is self-contained and bench-testable, and the boards are never re-wired.

Mounting the electronics on the back of the housing means the patch field, both boards, the ServoStik
boards and the hub all travel together. Only power and a few USB leads cross to the cabinet, so the
panel can be lifted for service without disturbing any of the wiring described here.

This section defines the architecture. The port-by-port assignments (which panel position carries
which control, where it lands on the I-PAC, and which LED channels light it) live in
[patch-panel-map.md](patch-panel-map.md).

## Typed jack definitions (T568B color order on the wire)

There are **three** jack types, distinguished by **keystone color** so you cannot mis-plug. For a
hard mechanical guarantee you can use keyed RJ45 (such as Panduit keyed jacks and plugs), but color
coding is normally sufficient. All three carry +5V (LED anode, servo, or device power), so treat the
whole field as live. Q-jacks are live only when the aux region is configured for a pointing device.

### J-jack: joystick (switches + ServoStik), color BLUE

| Pin | Signal |
| --- | ------ |
| 1 | Up |
| 2 | Down |
| 3 | Left |
| 4 | Right |
| 5 | ServoStik wire A |
| 6 | ServoStik wire B |
| 7 | spare |
| 8 | Common GND |

Pins 1-4 and 8 land on the I-PAC; pins 5-6 land on a **ServoStik USB control board** (1 board per 2
sticks). One RJ45 carries the whole "smart joystick," splitting to its two destinations on the fixed
board side while the control-panel side stays a single plug.

### Lit joysticks and trackballs: a second, LED-only P-jack

Neither the J-jack nor the Q-jack has room for an RGB LED. The J-jack is full: pins 1-4 are
directions, 5-6 are ServoStik, 8 is ground, and only pin 7 is spare. The Q-jack leaves pins 6 and 7
spare, one short of the 3 channel wires an RGB LED needs even if its anode shares the device +5V on
pin 2. An RGB LED needs 4 conductors (+5V anode, R, G, B) and neither jack can supply them.

A lit joystick or a lit trackball therefore uses **two jacks**: its J-jack or Q-jack for the control
itself, plus a second **P-jack populated for LED only**, with pins 2-5 wired and switch pins 1 and 8
left empty.
This is the mirror image of a non-lit button, which is a P-jack with pins 2-5 empty, so it needs no
new jack type and no new keystone color.

Cost per lit control is **1 patch panel port and 1 RGB LED**. Panel positions are now the scarce
resource rather than RGB, so the port is the part worth counting. See
[Sizing](#sizing-two-ultimate-io-boards) for the allocation.

**Everything is lit.** With two boards there are 48 RGB LEDs against 31 assigned, so nothing has to
go dark: all 20 buttons, all 4 joysticks, both trackballs and the spinner each get one, alongside
coin and start. Lighting is allocated per board, following whichever board reads the control.

Because lighting is independent of how a control is *read*, an LED-only jack can also light a
**USB-attached** control that never touches an I-PAC input.

Any control can be left unlit by leaving its LED jack unpopulated, which returns that RGB LED to that
board's reserve.

### P-jack: RGB pushbutton (switch + LED, one button), color RED, carries +5V

| Pin | Signal |
| --- | ------ |
| 1 | Switch signal to I-PAC input |
| 2 | LED +5V common anode |
| 3 | LED R |
| 4 | LED G |
| 5 | LED B |
| 6-7 | spare |
| 8 | Switch GND (common) |

**One RGB button per jack.** Pins 1 and 8 land on an I-PAC switch input; pins 2-5 land on the I-PAC
LED-output section. The low density makes each jack easy to wire and label, and every button becomes
individually hot-swappable. Confirm anode-common and current-sink polarity against the Ultimarc LED
wiring diagram before crimping, and set pin 2 accordingly. For a non-lit button, leave pins 2-5
unpopulated and use the same jack type. For an LED-only use, such as an illuminated joystick, do the
reverse and leave pins 1 and 8 unpopulated (see
[Lit joysticks and trackballs](#lit-joysticks-and-trackballs-a-second-led-only-p-jack)).

### Q-jack: one pointing device (trackball or spinner), color GREEN, carries +5V

| Pin | Signal | Trackball | Spinner |
| --- | ------ | --------- | ------- |
| 1 | Signal A | XA | A |
| 2 | +5V device power | yes | yes |
| 3 | Signal B | XB | B |
| 4 | Signal C | YA | not used |
| 5 | Signal D | YB | not used |
| 6-7 | spare | | |
| 8 | GND | yes | yes |

**One pointing device per jack**, which puts pointing devices under the same one-control-per-RJ45
rule as everything else. A trackball populates all four signal pins; a spinner populates two and
leaves pins 4 and 5 empty.

The signal pins are 1, 3, 4 and 5 rather than a straight run of 1 to 4 so that **the Q-jack's
conductor layout matches the P-jack exactly**: signals on 1/3/4/5, +5V on 2, ground on 8. Terminating
a Q-jack is then the same physical operation as terminating a P-jack, and only the board-side
destination differs. The keystone stays **green** so a pointing device can never be plugged into a
button position, since the matching pinout removes the wiring-level protection that different
pinouts would give.

Trackball signals land on the UIO's **top-left** connector and spinner signals on the **top-right**,
because Ultimarc requires trackball-left and spinner-right for the U-Trak and SpinTrak special
connectors. The board reads them as a mouse. Trackball and spinner *action buttons* use ordinary
P-jacks.

## Fixed cabinet controls

Seven buttons are direct-wired to the cabinet and sit outside the modular seam. They are the only
controls whose identity stays constant across every panel swap.

| Button | I-PAC input | Lighting |
| ------ | ----------- | -------- |
| Coin 1 | `coin1` | RGB |
| Coin 2 | `coin2` | RGB |
| Start 1 (also the shift key) | `start1` | RGB |
| Start 2 | `start2` | RGB |
| Escape | aux (`1a`) | single-color |
| Pause | aux (`1b`) | single-color |
| Menu | aux (`2a`) | single-color |

The coin and start switches stay direct-wired to their named inputs on **board 1**, with a
direct-wired RGB LED lead running from each to that board's LED section. None of these are patch-panel jacks. Escape,
pause and menu draw from the 24 single-color LED channels that remain after the 24 RGB LEDs are
allocated, which keeps them from competing visually with the game controls.

Menu covers the MAME configuration menu and the front-end menu, so the cabinet can be retuned in
place (dip switches, per-game control mapping) without attaching a keyboard.

### Start 1 as the shift key

`start1` is designated as the I-PAC shift key, which is the board default. Holding it gives every
other input a second code, so volume, save and load state, quit-to-desktop and similar functions
need no dedicated hardware. Pressing it alone still sends its normal start code on release. Confirm
that behavior on the bench (verification step 2) before the panel is printed, since the whole
arrangement depends on it.

Map shifted functions onto the fixed panel's own buttons only, never onto game buttons:

- The fixed panel is the only hardware whose button identity survives a panel swap. A combo anchored
  to "P1 button 3" means something different on the Q\*Bert panel than on the Joust panel, which puts
  the admin map back into per-swap work.
- Shift is global. With P1 holding start, a P2 button press can land on a combo. Keeping combos on
  admin hardware means only a deliberate press triggers one.

Starting map: shift + coin1 and shift + coin2 for volume down and up, shift + escape for
quit-to-desktop.

### No start 3 or start 4

The base layout commits all 20 button P-jacks to game buttons, and the 4 remaining P-jack positions
go to joystick LEDs, so there is no spare jack to carry a P3 or P4 start button across the seam.
Four-player start buttons are not wired.

### Coin mech variant

If the cabinet later gets real coin mechs, the door switches land on the same `coin1` and `coin2`
inputs and the two panel coin buttons are deleted, leaving five fixed buttons. Input accounting is
unchanged. Coin door lamps run on their own 12V circuit, so this frees 2 RGB LEDs and drops the RGB
count from 24/24 to 22/24.

Model the coin button cutouts as a separate sketch or feature in the fixed panel body, so the
variant can be toggled without disturbing the escape, pause and menu positions.

### Deliberately not on the panel

- **Volume:** shift combos, or an analog knob on the amp. A knob costs no inputs and still works if
  the front end is wedged.
- **Service, test, and service credit:** inside the coin door, as on the original machines, which
  also keeps them away from players.
- **Power and shutdown:** a keyswitch or a door-mounted button, never next to Escape.
- **Pinball flipper and magnasave buttons:** out of scope. A dedicated pinball cabinet covers this,
  and it would be 4 buttons with magnasave.

Recess Escape, or place it well away from hand travel, and consider a hold-to-exit setting in the
front end. An accidental exit mid-game is the most disruptive failure this panel can produce.

## USB-attached controls

Some controls bring their own USB interface and do not use the patch field at all. The first is the
**Star Wars yoke**, a repro with a built-in USB interface, and more are expected.

These are a deliberate exception to the one-control-per-RJ45 rule. A USB control enumerates to the PC
on its own, so it consumes **no I-PAC input, no RGB LED, and no RJ45 jack**, and none of the counts
in [Sizing](#sizing-two-ultimate-io-boards) move when one is added. It cannot be
driven by a ServoStik board, and whatever the device itself provides for input is what you get.

**Lighting is the exception.** Only the device's *data* goes over USB. If the control exposes its LED
on a separate connector, as an illuminated trackball does, that LED can still ride an I-PAC LED
section on an ordinary LED-only P-jack and be animated with everything else. A USB-attached control
is therefore outside the input budget but still inside the RGB budget when it is lit.

Both trackballs are read natively by the two boards, so no pointing device currently uses this route.
It stays the escape hatch for anything past the cap.

### The USB hub is a second seam

A **USB hub mounted in the control panel housing** collects these controls, with a single upstream
cable to the PC running alongside the RJ45 loom. The cabinet therefore has two parallel seams:

```text
   removable control panel
      |                     |
      |  RJ45 patch cables  |  USB leads
      v                     v
   BOARD PATCH FIELD      USB HUB (in the control panel housing)
      |                     |
      v                     v
   I-PAC UIO             single upstream cable
      |                     |
      +-------- PC ---------+
```

A panel swap now means unplugging its patch cables **and** its USB leads. Mount the hub near the
patch field so both halves of a swap happen in one place, and give USB leads the same service loop
and strain relief as the pigtails.

- **Use a powered hub.** Bus power is shared and easy to exhaust as controls accumulate, and a
  control that is intermittently unreliable under load is far harder to diagnose than one that is
  simply dead. The hub supply is separate from the I-PAC's +5V.
- **Leave spare ports.** Size the hub for the controls you expect plus spares. It is buried in the
  housing and awkward to swap out later.
- **Label USB leads at both ends**, the same as the RJ45 loom.

### Bind by device identity, not enumeration order

USB devices do not enumerate in a stable order. Map each USB control in the emulator or front end by
its **VID/PID or persistent device path**, never by device index. Otherwise a panel swap, a hub port
change, or a cold boot can silently shuffle which physical control is which, and the symptom looks
like a broken mapping rather than a reordering.

This is the USB counterpart of keeping the shift map on fixed hardware: anything whose identity
changes across a swap is a bad thing to anchor a mapping to.

## ServoStik 4-way/8-way restrictor control

- Each ServoStik stick uses a **2-wire** connection to the control board, riding J-jack pins 5-6.
- Driver: **Ultimarc USB ServoStik control board, 1 board drives 2 sticks.** Up to 4 sticks needs
  **2 boards**, fully used.
- Sticks are normally **ganged**, all set to the same restriction per game. The control board plus
  software (MAME/RetroFE) handles per-game switching.
- Because the servo wires ride the J-jack, a joystick module unplugs completely with one cable,
  switches and restrictor together.

## Trackball and spinner: the aux region

A trackball (XA/XB/YA/YB) costs 4 inputs on its board's **top-left** connector and a spinner (A/B)
costs 2 on the **top-right**. The board reads them as a **mouse** (quadrature), and RetroFE and
emulators use mouse X/Y plus a spinner axis. U-Trak and SpinTrak expect trackball-left and
spinner-right, so keep that assignment for their special connectors to work.

### There are no exclusions

On a single board those connectors are the P4 stick's and P4 buttons' pins, which forced a choice
between a 4th player and a pointing device. With two boards that conflict disappears: each board
serves only **two** players, so **its player 4 region is free**, and that is exactly where the
trackball and spinner connectors live.

| Board | Top-left connector | Top-right connector |
| ----- | ------------------ | ------------------- |
| 1 | Trackball 1 | Spinner |
| 2 | Trackball 2 | free, available for a second spinner |

Every joystick, every button and both pointing devices can be live at once. Nothing has to be
re-terminated to change games, and each pointing device rides its own Q-jack so the control-panel
side stays one plug per device. Trackball and spinner **action buttons** use ordinary P-jacks.

### Two trackballs, natively

Each Ultimate I/O has exactly **one** trackball connector, so two boards is the only way to read the
two trackballs Marble Madness needs without falling back to USB. Each board enumerates as its own
mouse, which is precisely the two-mouse arrangement a 2-player trackball game wants.

**This is the standing justification for the second board**, independent of whether P3 and P4 are
ever wired.

Two trackballs and one spinner are the cap for this cabinet, so no further pointing device positions
are provisioned. Board 2's top-right connector stays free if that changes. Anything beyond that would
go over **USB** through the housing hub, with its lighting still landing on the patch panel; see
[USB-attached controls](#usb-attached-controls).

**Confirm the one-trackball-per-board limit against Ultimarc's documentation before wiring the aux
region.** It is the assumption here most likely to change the design if it turns out to be wrong.

## Sizing: two Ultimate I/O boards

Two boards give **96 switch inputs**, **48 RGB LEDs**, **2 trackball connectors** and **2 spinner
connectors**.

Each board is wired as a **generic 2-player board**, with software assigning which physical players
it serves. Board 1's own player 1 and 2 regions serve P1 and P2; board 2's serve P3 and P4. That
leaves the **player 4 region free on both boards**, which is exactly where the trackball (top-left)
and spinner (top-right) connectors live. Pointing devices therefore never displace a joystick, and
the aux exclusions that a single board forced are gone.

### Board 1: P1, P2, the fixed panel, trackball 1, spinner

| Item | RJ45 jacks | Inputs | RGB |
| ---- | ---------- | ------ | --- |
| P1 + P2 joysticks (J-jack + servo) | 2 | 8 | none |
| P1 + P2 joystick LEDs (LED-only P-jack) | 2 | none | 2 |
| P1 buttons (6) + P2 buttons (6) | 12 | 12 | 12 |
| Trackball 1 (Q-jack) + its LED | 2 | 4 | 1 |
| Spinner (Q-jack) + its LED (provisioned) | 2 | 2 | 1 |
| coin1/2 + start1/2 (direct, RGB-lit) | none | 4 | 4 |
| escape + pause + menu (direct, single-color) | none | 3 | none |
| **Total** | **2 J + 16 P + 2 Q = 20** | **33 / 48** | **20 / 24** |

### Board 2: P3, P4, trackball 2

| Item | RJ45 jacks | Inputs | RGB |
| ---- | ---------- | ------ | --- |
| P3 + P4 joysticks (J-jack + servo) | 2 | 8 | none |
| P3 + P4 joystick LEDs (LED-only P-jack) | 2 | none | 2 |
| P3 buttons (6) + P4 buttons (6) | 12 | 12 | 12 |
| Trackball 2 (Q-jack) + its LED | 2 | 4 | 1 |
| **Total** | **2 J + 15 P + 1 Q = 18** | **24 / 48** | **15 / 24** |

### Totals and what binds

- **Patch panel positions: 38 populated of 48**, leaving 10 spare. This is now **the binding
  constraint**, not RGB. Budget **two positions per lit control**, one for the control and one for
  its LED.
- **Inputs: 57 of 96.** Neither board is close to full. The spares are concentrated in each board's
  unused player 3 and 4 regions, plus `SW7` and `SW8` on all four live player positions.
- **RGB: 35 of 48, with 13 in reserve.** Every control in the cabinet is lit: all 24 buttons, all 4
  joysticks, both trackballs, the spinner (when a lit one exists), and coin plus start.
- **Two trackballs are the reason for two boards.** Each Ultimate I/O has exactly one trackball
  connector, so a second board is the only way to read two trackballs natively. That holds even if P3
  and P4 are never wired.
- **ServoStik:** no dedicated jacks. Servo wires ride J-jack pins 5-6, landing on the **2 ServoStik
  USB control boards** (1 per 2 sticks). Board 1's sticks use ServoStik board A, board 2's use B.
- **USB-attached controls sit outside every count above.** They enumerate to the PC on their own and
  use no inputs, no RGB, and no jacks.

### Two boards means two devices

Each Ultimate I/O enumerates as its own keyboard **and** its own mouse. Two consequences:

- **Program non-overlapping key codes** in WinIPAC. Two boards both sending the same code for
  different controls is the most likely way to lose a day.
- **Two mice is the point, not a problem.** A 2-player trackball game wants exactly that. Map by
  device identity, the same discipline the USB controls need. See
  [USB-attached controls](#usb-attached-controls).

## Per-game / per-original layout mapping

The physical wiring exposes *enough typed jacks*. **Which input maps to which game function is
handled in software**, via I-PAC profiles and RetroFE or emulator keymaps per panel, so
"close to original" layouts are a mapping exercise rather than a re-wiring exercise.

## Hardware / materials

- **Cable: Cat6a throughout.** Two practical consequences follow from the heavier construction,
  neither of them electrical:
  - Solid Cat6a is typically **23 AWG**, with a larger outside diameter and often a cross-filler
    spline, so it is **stiffer and wants a larger bend radius** than Cat5e. This matters most for the
    short pigtails inside a control panel cavity, where the run is shortest and the turns are
    tightest. Leave the pigtails a little longer than feels necessary, and don't crush the service
    loop against the panel underside when the panel seats.
  - Use **Cat6a-rated keystones**. Cat5e jacks are not reliably rated to terminate 23 AWG
    conductors, and this applies to the panel-mount keystones on every control panel as well as the
    board-side field.
- **Prefer UTP (unshielded).** Shielded F/UTP or S/FTP would need shielded keystones plus a
  deliberate single-point ground, and a shield bonded at both ends invites a ground loop against the
  common-ground switch scheme. There is no noise problem here that would justify it.
- **Board to panel (fixed side):** discrete 24 AWG solid conductors, not jacketed cable. See
  [Board-side wiring](#board-side-wiring).
- **Seam:** short stranded Cat6a patch cables, which flex and survive repeated plugging. Keep them as
  short as the layout allows; 48 full-length patch cables will not fit behind a control panel.
- **Control-panel side:** panel-mount keystone jacks, colored per type, with short pigtails to each
  control. Add a service loop and strain relief.
- **USB hub:** powered, mounted in the control panel housing near the patch field, with spare ports.
  See [USB-attached controls](#usb-attached-controls).
- **Labeling:** label both ends, color-code patch cables and boots by player (P1/P2/P3/P4), and keep
  a laminated one-page pinout and mapping doc with the cabinet.

## Board-side wiring

Everything here lives on the **back of the control panel housing**: both boards, the 48-port patch
field, the two ServoStik boards, the USB hub, and two bus bars. Only power and a few USB leads cross
to the cabinet, so the panel lifts for service with its wiring intact.

The naive approach, a jacketed Cat6a run from every keystone to a board terminal, produces about 138
conductors of round cable in a shallow cavity. Four things cut that down.

### Bus the grounds and the LED anodes

Every switch shares a common ground and every RGB LED shares a +5V anode, so those conductors never
need to reach a board individually. Land them on two short bus bars mounted behind the patch field.

| Conductor group | Naive | With buses |
| --------------- | ----- | ---------- |
| Switch signals | 42 | 42 |
| LED channels | 48 | 48 |
| Switch grounds, one per jack | 34 | **1 bus** |
| LED +5V anodes, one per lit control | 17 | **1 bus** |
| **Total reaching the boards** | **141** | **~92** |

That is a third of the wire gone before anything is routed.

### Do not run jacketed cable on the fixed side

The jacket and cross-filler spline are most of Cat6a's bulk, and over a run of inches its electrical
properties buy nothing (see [Board facts](#board-facts-that-drive-the-design)). Punch **discrete
24 AWG solid conductors** into the keystones and route them as flat combed harnesses. Confirm your
keystones are rated for the gauge; most accept 22-26 AWG. Cat6a stays where it earns its keep: the
patch cables at the seam and the pigtails on each control panel.

### Bundle by destination, not by port

The panel side is organised for the human swapping decks, one block per player. The board side should
be organised for the terminal strips, because each jack's conductors split anyway. That gives three
looms, each uniform in destination and therefore traceable by position:

- **Switch loom** to the input terminals
- **LED loom** to the LED sections
- **Servo loom** to the two ServoStik boards

### Build it as one assembly, on the bench

The control panel back is the assembly. Mount the patch field, both boards, the ServoStik boards, the
hub and the buses on it, wire it away from the cabinet, and test it there. Wire and verify **one loom
at a time**: testing a finished 92-conductor field is far worse than testing three looms as they go.

If the Ultimate I/O harness leads reach, punching those flying ends into the keystones removes the
board-side termination entirely. Check lead length and connector type against your boards first.

### Watch the depth

Joysticks and trackballs protrude well below the panel surface, so the electronics can only occupy
what is left. Confirm the clear depth behind the panel against the deepest control before committing
a layout, and keep the patch field where 48 cables can turn without fighting their bend radius.

## Verification (bench-first, before cabinet install)

1. **Continuity and pinout test.** Check each terminated jack and patch cable end-to-end with a cable
   tester or multimeter against the tables above. Confirm no crossed pairs, GND on pin 8, and on the
   P-jack, +5V on pin 2 with switch signal on pin 1.
2. **Switch functional test.** With both boards connected to a PC, open a key or gamepad test (WinIPAC
   test mode or an OS input tester) and actuate each control through the full seam: control, pigtail,
   panel jack, patch cable, board panel, I-PAC. Confirm each maps to the expected input, including
   the seven direct-wired fixed panel buttons on board 1. Confirm the two boards send
   **non-overlapping key codes** and enumerate as two distinct devices. While here, confirm the shift
   behavior: `start1`
   pressed alone sends its start code on release, and `start1` held plus another button sends the
   shifted code.
3. **LED functional test (low current first).** Verify P-jack +5V and anode polarity with a meter
   *before* energizing. Then drive each RGB channel with Ultimarc's LED test utility and confirm
   R/G/B per button, checking for shorted or swapped channels. Include the coin and start RGB leads
   and the three single-color admin channels.
4. **ServoStik test.** Command 4-way and then 8-way (via the ServoStik utility or a MAME per-game
   switch) and confirm each restrictor plate physically rotates and that ganged sticks move together.
   Confirm J-jack pins 5-6 land on the control board, not the I-PAC.
5. **Pointing device test.** Test each device on its own Q-jack. Confirm each board enumerates as a
   **mouse**, that rolling each trackball moves its own X/Y, and that the spinner axis moves. Then
   confirm the two trackballs register as **two distinct mice** and that a 2-player trackball game
   sees them separately, which is the whole reason for the second board. Verify the trackball-left
   and spinner-right assignment so the U-Trak and SpinTrak special connectors work.
6. **Swap and conversion test.** Unplug one control panel's loom (its J-jacks and P-jacks) and plug a
   second panel into the same board field. Confirm it enumerates, maps, lights, and restricts
   correctly, which is the core modularity guarantee. Pull a single P-jack to confirm one-button
   hot-swap. There is no aux re-termination to exercise: with two boards, joysticks and pointing
   devices coexist, so nothing has to be rewired between games.
7. **USB control test.** For any panel carrying a USB control, confirm it enumerates through the hub
   and maps correctly. Then unplug and replug it as part of a swap, and move it to a different hub
   port, confirming the mapping survives both. If it does not, the front end is binding by
   enumeration order rather than device identity (see
   [USB-attached controls](#usb-attached-controls)).

## Build phases

Board 1 is a complete, playable cabinet on its own. Wire and verify it before starting board 2.

| Phase | Ports | What it gets you |
| ----- | ----- | ---------------- |
| 1 | **row 1** (1-24) | P1, P2, the fixed panel, a trackball and a spinner. Most of the library. |
| 2 | **row 2** (25-48) | P3, P4 and the second trackball. Four-player games and Marble Madness. |

The phases are the two panel rows, which are also the two boards. Phase 1 is 20 jacks and roughly 60
conductors, a far more testable first pass than the whole field at once. If phase 2 never happens, board 2 still earns its place by hosting trackball 2,
which no single board can provide.

## Sources

- [I-PAC Ultimate I/O (Ultimarc)](https://www.ultimarc.com/control-interfaces/i-pacs/i-pac-ultimate-i-o/)
- [ServoStik (Ultimarc)](https://www.ultimarc.com/arcade-controls/joysticks/servostik/)
- [I-PAC install guide, trackball/spinner use 6 of 48 inputs, top-left/right connectors](https://www.t-molding.com/media/products/ultimarc-ipac-installation.pdf)
- [I-PAC Ultimate I/O with full wiring harness (Paradise Arcade Shop)](https://paradisearcadeshop.com/products/i-pac-ultimate-io-with-full-wiring-harness)
- [9 LED connection pack / LED power notes (Ultimarc)](https://www.ultimarc.com/control-interfaces/accessories/9-led-single-color-connection-pack-for-i-pac-ultimate-i-o-interface/)
