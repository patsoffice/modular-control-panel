# Modular RJ45 Arcade Control Wiring — Build Plan

## Context

Goal: wire arcade controls to an **Ultimarc I-PAC Ultimate I/O** so that entire control
panels are **modular and pluggable** — swap between game-accurate layouts (up to **4 players /
4 joysticks**, incl. twin-stick like Smash TV, plus an optional **trackball/spinner** in the aux
slot, "as close to original as possible") by unplugging patch cables, no soldering. The
interconnect uses **RJ45 (Cat5e) connectors through a patch panel**. (The one exception to
solderless swaps is converting the P4 aux slot between a 4th joystick and trackball/spinner, which
re-terminates ~1–2 jacks — see Trackball/spinner.)

This is a physical/electrical build plan (not a change to the RetroFE codebase).

### Board facts that drive the design
- **48 switch inputs**, each a dedicated pin, **common ground** (Ultimarc supplies per-input
  ground wiring; every switch closes between its pin and a shared GND). → only **one ground
  conductor per cable** is needed.
- **96 LED channels = 24 RGB LEDs**, constant-current **sinks** fed from **+5V**. Each RGB LED
  shares a **+5V anode**; the board pulls R/G/B to ground. No per-LED resistors needed.
- Switch closures are logic-level milliamps → **wire gauge, length, and EMI are non-issues**;
  ordinary Cat5e 24 AWG is far more than enough.

### Decisions locked in (from discussion)
- **One control per RJ45** — low wire density per jack (easy to wire and label), rather than
  cramming many buttons onto one cable.
- **LED rides the button's own jack** (no separate LED patch domain). A lone switch wastes an
  8-conductor cable; adding that button's RGB LED fills it (switch sig+GND + LED anode+R/G/B =
  6 conductors). Switch and its lighting are one self-contained plug.
- **Up to 4 players / 4 joysticks** (4 J-jack positions cover 4 single-stick players OR 2
  twin-stick players like Smash TV); layouts vary per game → swaps at the **control-panel** level.
  At 4 players the board is essentially full (see Sizing) — the fixed field is capped accordingly.
- **Trackball + spinner are IN scope**, on **separate wiring** (their own Q-jack) and occupying the
  **P4 aux slot** — *in place of* a 4th joystick-player, not in addition. The UIO has no free
  trackball bus: it **borrows 6 of the 48 inputs** (trackball XA/XB/YA/YB on the top-left connector
  = 4; spinner A/B on the top-right = 2). The **P4 stick is wired to those top-left pins**, so the
  reason it's "in place of P4" is physical, not just budget: a single I-PAC pin feeds one jack, so
  the top-connector pins can't be both "P4 stick" and "trackball" at once → the P4 region is
  **re-terminated to convert** between the two (build-time rewire of ~1–2 jacks, not a hot-swap).
- **Typed jacks** (distinct joystick vs. button pinouts, keyed by color so you cannot mis-plug),
  NOT one universal interchangeable jack.
- **ServoStik 4-way/8-way restrictor support** on the joysticks. Each stick uses a **2-wire**
  connection to Ultimarc's **USB ServoStik control board (1 board drives 2 sticks)** — software
  (MAME/RetroFE) commands the restriction per-game; the I-PAC UIO does **not** drive it. The
  2 wires **ride the J-jack's spare pins** — a joystick stays a single clean plug.
- **Asymmetric button counts:** P1/P2 = **6 buttons** each (buttons 7 & 8 dropped), P3/P4 = **4
  buttons** each = **20 buttons total** (20 P-jacks). The 4 dropped buttons free 4 RGB LEDs (and 4
  inputs) — reassigned to light coin/start (next line).
- **Direct-wired (NON-modular) cabinet controls = the I-PAC's 8 named inputs:** `coin1`, `coin2`,
  `start1`, `start2`, `1a`, `1b`, `2a`, `2b`. The board only provides **2-player** coin/start
  labels; the four aux inputs (`1a/1b/2a/2b`) absorb P3/P4 start + a hardware admin button (or
  shared coins). **coin1/2 + start1/2 are RGB-illuminated** — switch stays direct-wired to its named
  input, and a direct-wired RGB LED lead runs to the I-PAC LED section (they are NOT patch-panel
  jacks). `1a/1b/2a/2b` stay unlit. All hard-wired to fixed cabinet locations. See Sizing.

## Architecture: fixed board side ↔ swappable control side

```
 I-PAC Ultimate I/O terminals
        │  solid-core Cat5e, punched down ONCE (never touched again)
        ▼
 ┌──────────────────────────────┐
 │  BOARD PATCH FIELD           │  mounted near the I-PAC. Each jack SPLITS to its
 │  ┌──────────────────────────┐│  destinations on this fixed side:
 │  │ single integrated field  ││   • J-jack: pins1-4,8 → I-PAC in; pins5-6 → ServoStik bd
 │  │ (J + P; Q in Config B)   ││   • P-jack: pin1,8 → I-PAC in; pins2-5 → I-PAC LED outs
 │  └──────────────────────────┘│   • Q-jack: → I-PAC top-left (TB) / top-right (spin)
 └──────────────────────────────┘
        │  standard Cat5e patch cables  ← THE PLUGGABLE SEAM (unplug to swap a panel)
        ▼
 ┌──────────────────────────────┐
 │  CONTROL-PANEL JACKS         │  RJ45 keystones on each removable control panel
 └──────────────────────────────┘
        │  short pigtails
        ▼
   joysticks (switch+servo) / RGB buttons (switch+LED) / trackball+spinner (Q)  — one per cable
```

Payoff: swapping a whole control panel = unplug a handful of patch cables at the seam. Each
control panel is self-contained and bench-testable. The I-PAC is never re-wired.

## Typed jack definitions (T568B color order on the wire)

**Three** jack types, distinguished by **keystone color** so you cannot mis-plug. (Optional hard
guarantee: mechanically-keyed RJ45 such as Panduit keyed jacks/plugs — color coding is normally
sufficient.) All carry +5V (LED anode / servo / device power), so the whole field is treated as live.
The Q-jack is only present in the trackball/spinner build config (see below).

### J-jack — Joystick (switches + ServoStik)  (color: BLUE)
| Pin | Signal |
|-----|--------|
| 1 | Up |
| 2 | Down |
| 3 | Left |
| 4 | Right |
| 5 | ServoStik wire A |
| 6 | ServoStik wire B |
| 7 | spare |
| 8 | Common GND |

→ Pins 1–4 + 8 land on the I-PAC; pins 5–6 land on a **ServoStik USB control board** (1 board per
2 sticks). One RJ45 carries the whole "smart joystick," splitting to its two destinations on the
fixed board side; the control-panel side stays a single plug.

### P-jack — RGB Pushbutton (switch + LED, ONE button)  (color: RED)  ⚠ carries +5V
| Pin | Signal |
|-----|--------|
| 1 | Switch signal → I-PAC input |
| 2 | LED +5V common anode |
| 3 | LED R |
| 4 | LED G |
| 5 | LED B |
| 6–7 | spare |
| 8 | Switch GND (common) |

→ **One RGB button per jack.** Pins 1 + 8 land on an I-PAC switch input; pins 2–5 land on the
I-PAC LED-output section. Low density = trivially easy to wire and label, and every button becomes
individually hot-swappable. (Confirm anode-common/current-sink polarity against the Ultimarc LED
wiring diagram before crimping; set pin 2 accordingly. For a non-lit button, just leave pins 2–5
unpopulated — same jack type.)

### Q-jack — Trackball + Spinner (quadrature)  (color: GREEN, aux slot only)  ⚠ carries +5V
| Pin | Signal |
|-----|--------|
| 1 | Trackball XA |
| 2 | Trackball XB |
| 3 | Trackball YA |
| 4 | Trackball YB |
| 5 | Spinner A |
| 6 | Spinner B |
| 7 | +5V (device power) |
| 8 | GND |

→ One Q-jack carries **both** devices. Trackball pins (1–4) land on the UIO's **top-LEFT** connector,
spinner pins (5–6) on the **top-RIGHT** connector (Ultimarc requires trackball-left / spinner-right
to use the U-Trak / SpinTrak special connectors). These 6 signals **are** 6 of the 48 inputs — the
board reads them as a mouse. Occupies the **P4 aux slot**; mutually exclusive with a 4th joystick
(see Trackball/Spinner section). Trackball/spinner *action buttons* use ordinary P-jacks.

## ServoStik 4/8-way restrictor control

- Each ServoStik stick uses a **2-wire** connection to the control board (ride J-jack pins 5–6).
- Driver: **Ultimarc USB ServoStik control board, 1 board drives 2 sticks.** Up to 4 sticks → **2
  boards** (fully used).
- Sticks are normally **ganged** (all set to the same restriction per game); the control board +
  software (MAME/RetroFE) handle per-game switching.
- Because the servo wires ride the J-jack, a joystick module unplugs completely with one cable —
  switches and restrictor together.

## Trackball / spinner (aux slot, separate wiring)

- **No free bus on the UIO** — trackball (XA/XB/YA/YB) + spinner (A/B) consume **6 of the 48
  inputs**, on the **top-left** (trackball) and **top-right** (spinner) connectors. The board reads
  them as a **mouse** (quadrature); RetroFE/emulators use mouse X/Y + spinner axis.
- Wired to a **dedicated Q-jack** (separate zone/color) — this is the "separate wiring."
- **Occupies the P4 aux slot, mutually exclusive with a 4th joystick.** The P4 J-jack is wired to
  the top-left connector and 2 of P4's button jacks to the top-right; **converting** the slot
  between "4th stick + buttons" and "trackball/spinner" = **re-terminating those ~1–2 jacks** onto
  the Q-jack (a build-time change, not a hot-swap — chosen deliberately for clean separate wiring).
- U-Trak / SpinTrak devices expect trackball-left / spinner-right; keep that assignment so their
  special connectors work. Trackball/spinner **action buttons** use ordinary P-jacks.

## Sizing — 4 players against a 48-input / 24-RGB board

The I-PAC UIO has **48 inputs** and **24 RGB LEDs (= 96 LED channels)**. The board supports two
build configs (the P4 aux slot is re-terminated to switch between them):

**Config A — full 4-player (4 sticks + 20 buttons; e.g. Smash TV).** The **4 joysticks are just 4
jacks**; the rest of the jack count is the 20 button jacks. RGB is the binding ceiling (full):

| Item | RJ45 jacks | I-PAC inputs | RGB LEDs |
|------|-----------|--------------|----------|
| 4× J-jack (joysticks + servo) | 4 | 16 | — |
| P1 buttons (6) + P2 buttons (6) | 12 | 12 | 12 |
| P3 buttons (4) + P4 buttons (4) | 8 | 8 | 8 |
| coin1/2 + start1/2 (direct, **RGB-lit**) | — | 4 | 4 |
| 1a/1b/2a/2b (direct, unlit) | — | 4 | — |
| **Total** | **4 J + 20 P = 24** | **44 / 48** | **24 / 24** |

→ 4 spare inputs (from dropping buttons 7/8) are now free for dedicated admin buttons or extras.

**Config B — trackball/spinner in the P4 slot.** Drops the 4th stick + P4 buttons, adds the Q-jack:

| Item | RJ45 jacks | I-PAC inputs | RGB LEDs |
|------|-----------|--------------|----------|
| 3× J-jack (P1/P2/P3 sticks + servo) | 3 | 12 | — |
| P1 (6) + P2 (6) + P3 (4) buttons | 16 | 16 | 16 |
| 1× Q-jack (trackball 4 + spinner 2) | 1 | 6 | — |
| coin1/2 + start1/2 (direct, **RGB-lit**) | — | 4 | 4 |
| 1a/1b/2a/2b (direct, unlit) | — | 4 | — |
| **Total** | **3 J + 16 P + 1 Q = 20** | **42 / 48** | **20 / 24** |

Conversion A↔B = re-terminate the P4 J-jack (→ trackball, top-left) + 2 P4 button jacks (→ spinner,
top-right) onto the Q-jack; ~1–2 jacks.

- **Jack count:** 24 live jacks in Config A (4 J + 20 P); provision **1 more Q-jack position** for
  Config B = **25 positions → a 32-port patch panel** (comfortable spares; a 24-port + 1 keystone
  also works). The Q-jack and the P4 J-jack are never live at the same time.
- **Inputs:** **44/48** — dropping P1/P2 buttons 7 & 8 freed 4 inputs. Modular = 16 sticks + 20
  buttons = 36; direct-wired = 8 named (coin1/2, start1/2, 1a/1b/2a/2b). The four aux inputs
  (1a/1b/2a/2b) are reassigned in software to **P3/P4 start + admin** (or shared coins). **4 spare
  inputs** remain — free for dedicated admin buttons or extra controls.
- **RGB (the binding ceiling): 24/24 — full.** 20 lit buttons + **coin1/2 + start1/2 lit** = 24
  RGB. That is 72 of the 96 LED channels; **24 single-color channels remain** for marquee accents
  etc. Marquee/underglow strips run on the 1A MOSFET drivers, hard-wired.
- **ServoStik:** no dedicated jacks — servo wires ride J-jack pins 5–6; on the board side those
  pins land on the **2 ServoStik USB control boards** (1 per 2 sticks; 4 sticks fill both).
- **RGB (24/24) is now the hard ceiling, not inputs (44/48).** More *lit* controls need a second
  LED controller; more *inputs* (beyond the 4 spare) need a second encoder — out of scope unless
  required. Any button can stay unlit (leave P-jack pins 2–5 empty) to save RGB.

## Per-game / per-original layout mapping

Physical wiring exposes *enough typed jacks*; **which input = which game function is handled in
software** (I-PAC profiles and/or RetroFE + emulator keymaps per panel), so "close to original"
layouts are a mapping concern, not a re-wiring concern.

## Hardware / materials

- **Board→panel (fixed side):** solid-core Cat5e, punched down onto a 110/keystone patch panel.
- **Seam (loom):** stranded Cat5e patch cables (flex + survive repeated plugging).
- **Control-panel side:** panel-mount keystone jacks (colored per type) with short pigtails to each
  control; add a service loop + strain relief.
- **Labeling:** both ends labeled; color-code patch cables/boots by player (P1/P2/P3/admin); keep a
  laminated one-page pinout + mapping doc with the cabinet.

## Verification (bench-first, before cabinet install)

1. **Continuity/pinout test** each terminated jack and patch cable end-to-end with a cable tester
   or multimeter against the tables above — confirm no crossed pairs, GND on pin 8, and (P-jack)
   +5V on pin 2 with switch signal on pin 1.
2. **Switch functional test:** with the I-PAC connected to a PC, open a key/gamepad test (e.g.
   WinIPAC test mode or an OS input tester) and actuate each control through the full seam
   (control → pigtail → panel jack → patch cable → board panel → I-PAC). Confirm each maps to the
   expected input. Include the direct-wired coin/start/1a-b/2a-b controls.
3. **LED functional test (LOW current first):** verify P-jack +5V/anode polarity with a meter
   *before* energizing; then drive each RGB channel via Ultimarc's LED test/utility and confirm
   R/G/B per button, checking for no shorted/swapped channels.
4. **ServoStik test:** command 4-way then 8-way (via the ServoStik utility / MAME per-game switch)
   and confirm each restrictor plate physically rotates and that ganged sticks move together;
   confirm J-jack pins 5–6 land on the control board, not the I-PAC.
5. **Trackball/spinner test (Config B):** with the Q-jack wired to top-left (trackball) / top-right
   (spinner), confirm the board enumerates as a **mouse**; roll the trackball and watch X/Y move,
   spin the spinner and watch its axis. Verify trackball-left / spinner-right assignment so U-Trak /
   SpinTrak special connectors work.
6. **Swap + conversion test:** unplug one control panel's loom (its J-jacks + P-jacks) and plug a
   second panel into the same board field; confirm it enumerates, maps, lights, and restricts
   correctly — the core modularity guarantee. Pull a single P-jack to confirm one-button hot-swap.
   Then exercise the **A↔B re-termination**: move the P4 top-connector wires to the Q-jack, load the
   trackball profile, and confirm the aux slot switches roles.

## Sources
- [I-PAC Ultimate I/O (Ultimarc)](https://www.ultimarc.com/control-interfaces/i-pacs/i-pac-ultimate-i-o/)
- [ServoStik (Ultimarc)](https://www.ultimarc.com/arcade-controls/joysticks/servostik/)
- [I-PAC install guide — trackball/spinner use 6 of 48 inputs, top-left/right connectors](https://www.t-molding.com/media/products/ultimarc-ipac-installation.pdf)
- [I-PAC Ultimate I/O with full wiring harness (Paradise Arcade Shop)](https://paradisearcadeshop.com/products/i-pac-ultimate-io-with-full-wiring-harness)
- [9 LED connection pack / LED power notes (Ultimarc)](https://www.ultimarc.com/control-interfaces/accessories/9-led-single-color-connection-pack-for-i-pac-ultimate-i-o-interface/)
