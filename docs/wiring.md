# Modular RJ45 Arcade Control Wiring: Build Plan

## Context

Goal: wire arcade controls to an **Ultimarc I-PAC Ultimate I/O** so that entire control panels are
modular and pluggable. Swapping between game-accurate layouts (up to **4 players / 4 joysticks**,
including twin-stick like Smash TV, plus an optional **trackball/spinner** in the aux slot) means
unplugging patch cables, with no soldering. The interconnect uses **RJ45 (Cat6a) connectors through
a patch panel**.

The one exception to solderless swaps is converting the P4 aux slot between a 4th joystick and the
trackball/spinner, which re-terminates one or two jacks. See [Trackball / spinner](#trackball--spinner-aux-slot-separate-wiring).

### Board facts that drive the design

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
  cramming many buttons onto one cable.
- **The LED rides the button's own jack**, with no separate LED patch domain. A lone switch wastes
  an 8-conductor cable; adding that button's RGB LED fills it (switch signal + GND, LED anode +
  R/G/B = 6 conductors). A switch and its lighting are one self-contained plug.
- **Up to 4 players / 4 joysticks.** Four J-jack positions cover 4 single-stick players or 2
  twin-stick players like Smash TV. Layouts vary per game, so swaps happen at the **control-panel**
  level. At 4 players the board is essentially full (see [Sizing](#sizing-4-players-against-a-48-input--24-rgb-board)),
  and the fixed field is capped accordingly.
- **Trackball and spinner are in scope**, on separate wiring (their own Q-jack), occupying the **P4
  aux slot** in place of a 4th joystick player rather than in addition to one. The UIO has no free
  trackball bus, so it **borrows 6 of the 48 inputs**: trackball XA/XB/YA/YB on the top-left
  connector (4) and spinner A/B on the top-right (2). The **P4 stick is wired to those top-left
  pins**, so the "in place of P4" constraint is physical, not just a budget choice. A single I-PAC
  pin feeds one jack, so the top-connector pins cannot be both "P4 stick" and "trackball" at once,
  and the P4 region is **re-terminated to convert** between the two. This is a build-time rewire of
  one or two jacks, not a hot-swap.
- **Typed jacks**, with distinct joystick and button pinouts keyed by color so you cannot mis-plug.
  There is no universal interchangeable jack.
- **ServoStik 4-way/8-way restrictor support** on the joysticks. Each stick uses a **2-wire**
  connection to Ultimarc's **USB ServoStik control board** (1 board drives 2 sticks). Software
  (MAME/RetroFE) commands the restriction per game; the I-PAC UIO does not drive it. The 2 wires
  **ride the J-jack's spare pins**, so a joystick stays a single clean plug.
- **Asymmetric button counts.** P1/P2 get **6 buttons** each (buttons 7 and 8 dropped), P3/P4 get
  **4 buttons** each, for **20 buttons total** and 20 P-jacks. The 4 dropped buttons free 4 RGB LEDs
  and 4 inputs, which are reassigned to light coin and start.
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
 I-PAC Ultimate I/O terminals
        |  solid-core Cat6a, punched down ONCE (never touched again)
        v
 +------------------------------+
 |  BOARD PATCH FIELD           |  Mounted near the I-PAC. Each jack splits to its
 |  +--------------------------+|  destinations on this fixed side:
 |  | single integrated field  ||    J-jack: pins 1-4,8 to I-PAC in; pins 5-6 to ServoStik board
 |  | (J + P; Q in Config B)   ||    P-jack: pins 1,8 to I-PAC in; pins 2-5 to I-PAC LED outs
 |  +--------------------------+|    Q-jack: to I-PAC top-left (trackball) / top-right (spinner)
 +------------------------------+
        |  standard Cat6a patch cables: THE PLUGGABLE SEAM (unplug to swap a panel)
        v
 +------------------------------+
 |  CONTROL-PANEL JACKS         |  RJ45 keystones on each removable control panel
 +------------------------------+
        |  short pigtails
        v
   joysticks (switch + servo) / RGB buttons (switch + LED) / trackball + spinner (Q)
   one control per cable
```

Swapping a whole control panel means unplugging a handful of patch cables at the seam. Each control
panel is self-contained and bench-testable, and the I-PAC is never re-wired.

## Typed jack definitions (T568B color order on the wire)

There are **three** jack types, distinguished by **keystone color** so you cannot mis-plug. For a
hard mechanical guarantee you can use keyed RJ45 (such as Panduit keyed jacks and plugs), but color
coding is normally sufficient. All three carry +5V (LED anode, servo, or device power), so treat the
whole field as live. The Q-jack is only present in the trackball/spinner build config.

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
unpopulated and use the same jack type.

### Q-jack: trackball + spinner (quadrature), color GREEN, aux slot only, carries +5V

| Pin | Signal |
| --- | ------ |
| 1 | Trackball XA |
| 2 | Trackball XB |
| 3 | Trackball YA |
| 4 | Trackball YB |
| 5 | Spinner A |
| 6 | Spinner B |
| 7 | +5V (device power) |
| 8 | GND |

One Q-jack carries **both** devices. Trackball pins (1-4) land on the UIO's **top-left** connector
and spinner pins (5-6) on the **top-right** connector, because Ultimarc requires trackball-left and
spinner-right for the U-Trak and SpinTrak special connectors. These 6 signals are 6 of the 48 inputs,
and the board reads them as a mouse. The Q-jack occupies the **P4 aux slot** and is mutually
exclusive with a 4th joystick. Trackball and spinner *action buttons* use ordinary P-jacks.

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

The coin and start switches stay direct-wired to their named I-PAC inputs, with a direct-wired RGB
LED lead running from each to the I-PAC LED section. None of these are patch-panel jacks. Escape,
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

Config A commits all 20 P-jacks to game buttons, so there is no spare jack to carry a P3 or P4 start
button across the seam. Four-player start buttons are not wired.

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
in [Sizing](#sizing-4-players-against-a-48-input--24-rgb-board) move when one is added. The trade is
that such a control cannot be lit from the I-PAC LED section and cannot be driven by a ServoStik
board. Whatever the device itself provides is what you get.

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

## Trackball / spinner (aux slot, separate wiring)

- **No free bus on the UIO.** Trackball (XA/XB/YA/YB) and spinner (A/B) consume **6 of the 48
  inputs**, on the **top-left** (trackball) and **top-right** (spinner) connectors. The board reads
  them as a **mouse** (quadrature), and RetroFE and emulators use mouse X/Y plus a spinner axis.
- Wired to a **dedicated Q-jack** in its own zone and color, which is the "separate wiring."
- **Occupies the P4 aux slot and is mutually exclusive with a 4th joystick.** The P4 J-jack is wired
  to the top-left connector and 2 of P4's button jacks to the top-right. **Converting** the slot
  between "4th stick + buttons" and "trackball/spinner" means **re-terminating those one or two
  jacks**, a build-time change rather than a hot-swap, chosen deliberately for clean separate wiring.
- U-Trak and SpinTrak devices expect trackball-left and spinner-right, so keep that assignment for
  their special connectors to work. Trackball and spinner **action buttons** use ordinary P-jacks.

## Sizing: 4 players against a 48-input / 24-RGB board

The I-PAC UIO has **48 inputs** and **24 RGB LEDs (96 LED channels)**. The board supports two build
configs, and the P4 aux slot is re-terminated to switch between them.

**Config A: full 4-player (4 sticks + 20 buttons, for example Smash TV).** The 4 joysticks are just
4 jacks; the rest of the jack count is the 20 button jacks. RGB is the binding ceiling and is full.

| Item | RJ45 jacks | I-PAC inputs | RGB LEDs |
| ---- | ---------- | ------------ | -------- |
| 4x J-jack (joysticks + servo) | 4 | 16 | none |
| P1 buttons (6) + P2 buttons (6) | 12 | 12 | 12 |
| P3 buttons (4) + P4 buttons (4) | 8 | 8 | 8 |
| coin1/2 + start1/2 (direct, RGB-lit) | none | 4 | 4 |
| escape + pause + menu (direct, single-color) | none | 3 | none |
| **Total** | **4 J + 20 P = 24** | **43 / 48** | **24 / 24** |

**Config B: trackball/spinner in the P4 slot.** Drops the 4th stick and P4 buttons, adds the Q-jack.

| Item | RJ45 jacks | I-PAC inputs | RGB LEDs |
| ---- | ---------- | ------------ | -------- |
| 3x J-jack (P1/P2/P3 sticks + servo) | 3 | 12 | none |
| P1 (6) + P2 (6) + P3 (4) buttons | 16 | 16 | 16 |
| 1x Q-jack (trackball 4 + spinner 2) | 1 | 6 | none |
| coin1/2 + start1/2 (direct, RGB-lit) | none | 4 | 4 |
| escape + pause + menu (direct, single-color) | none | 3 | none |
| **Total** | **3 J + 16 P + 1 Q = 20** | **41 / 48** | **20 / 24** |

Converting A to B means re-terminating the P4 J-jack (to trackball, top-left) plus 2 P4 button jacks
(to spinner, top-right) onto the Q-jack, so one or two jacks.

- **Jack count:** 24 live jacks in Config A (4 J + 20 P), plus **1 more Q-jack position** provisioned
  for Config B, giving **25 positions**, which fits a **32-port patch panel** with comfortable
  spares. A 24-port panel plus 1 keystone also works. The Q-jack and the P4 J-jack are never live at
  the same time.
- **Inputs: 43/48.** Dropping P1/P2 buttons 7 and 8 freed 4 inputs. Modular controls use 36 (16
  sticks + 20 buttons) and the fixed panel uses 7 (coin1/2, start1/2, escape, pause, menu). **5 spare
  inputs** remain, including one unused aux input (`2b`), free for future controls.
- **RGB is the binding ceiling at 24/24, full.** 20 lit buttons plus coin1/2 and start1/2 is 24 RGB,
  which is 72 of the 96 LED channels. **24 single-color channels remain**; escape, pause and menu
  take 3, leaving 21 for marquee accents and similar. Marquee and underglow strips run on the 1A
  MOSFET drivers, hard-wired.
- **USB-attached controls sit outside every count above.** They enumerate to the PC on their own and
  use no inputs, no RGB, and no jacks, so adding one never pressures the board.
- **ServoStik:** no dedicated jacks. Servo wires ride J-jack pins 5-6, and on the board side those
  pins land on the **2 ServoStik USB control boards** (1 per 2 sticks; 4 sticks fill both).
- **RGB (24/24) is the hard ceiling, not inputs (43/48).** More *lit* controls would need a second
  LED controller, and more *inputs* beyond the 5 spare would need a second encoder. Both are out of
  scope unless required. Any button can stay unlit (leave P-jack pins 2-5 empty) to save RGB.

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
- **Board to panel (fixed side):** solid-core Cat6a, punched down onto a 110/keystone patch panel.
- **Seam (loom):** stranded Cat6a patch cables, which flex and survive repeated plugging.
- **Control-panel side:** panel-mount keystone jacks, colored per type, with short pigtails to each
  control. Add a service loop and strain relief.
- **USB hub:** powered, mounted in the control panel housing near the patch field, with spare ports.
  See [USB-attached controls](#usb-attached-controls).
- **Labeling:** label both ends, color-code patch cables and boots by player (P1/P2/P3/P4), and keep
  a laminated one-page pinout and mapping doc with the cabinet.

## Verification (bench-first, before cabinet install)

1. **Continuity and pinout test.** Check each terminated jack and patch cable end-to-end with a cable
   tester or multimeter against the tables above. Confirm no crossed pairs, GND on pin 8, and on the
   P-jack, +5V on pin 2 with switch signal on pin 1.
2. **Switch functional test.** With the I-PAC connected to a PC, open a key or gamepad test (WinIPAC
   test mode or an OS input tester) and actuate each control through the full seam: control, pigtail,
   panel jack, patch cable, board panel, I-PAC. Confirm each maps to the expected input, including
   the seven direct-wired fixed panel buttons. While here, confirm the shift behavior: `start1`
   pressed alone sends its start code on release, and `start1` held plus another button sends the
   shifted code.
3. **LED functional test (low current first).** Verify P-jack +5V and anode polarity with a meter
   *before* energizing. Then drive each RGB channel with Ultimarc's LED test utility and confirm
   R/G/B per button, checking for shorted or swapped channels. Include the coin and start RGB leads
   and the three single-color admin channels.
4. **ServoStik test.** Command 4-way and then 8-way (via the ServoStik utility or a MAME per-game
   switch) and confirm each restrictor plate physically rotates and that ganged sticks move together.
   Confirm J-jack pins 5-6 land on the control board, not the I-PAC.
5. **Trackball and spinner test (Config B).** With the Q-jack wired to top-left (trackball) and
   top-right (spinner), confirm the board enumerates as a **mouse**. Roll the trackball and watch X/Y
   move, spin the spinner and watch its axis. Verify the trackball-left and spinner-right assignment
   so the U-Trak and SpinTrak special connectors work.
6. **Swap and conversion test.** Unplug one control panel's loom (its J-jacks and P-jacks) and plug a
   second panel into the same board field. Confirm it enumerates, maps, lights, and restricts
   correctly, which is the core modularity guarantee. Pull a single P-jack to confirm one-button
   hot-swap. Then exercise the A-to-B re-termination: move the P4 top-connector wires to the Q-jack,
   load the trackball profile, and confirm the aux slot switches roles.
7. **USB control test.** For any panel carrying a USB control, confirm it enumerates through the hub
   and maps correctly. Then unplug and replug it as part of a swap, and move it to a different hub
   port, confirming the mapping survives both. If it does not, the front end is binding by
   enumeration order rather than device identity (see
   [USB-attached controls](#usb-attached-controls)).

## Sources

- [I-PAC Ultimate I/O (Ultimarc)](https://www.ultimarc.com/control-interfaces/i-pacs/i-pac-ultimate-i-o/)
- [ServoStik (Ultimarc)](https://www.ultimarc.com/arcade-controls/joysticks/servostik/)
- [I-PAC install guide, trackball/spinner use 6 of 48 inputs, top-left/right connectors](https://www.t-molding.com/media/products/ultimarc-ipac-installation.pdf)
- [I-PAC Ultimate I/O with full wiring harness (Paradise Arcade Shop)](https://paradisearcadeshop.com/products/i-pac-ultimate-io-with-full-wiring-harness)
- [9 LED connection pack / LED power notes (Ultimarc)](https://www.ultimarc.com/control-interfaces/accessories/9-led-single-color-connection-pack-for-i-pac-ultimate-i-o-interface/)
