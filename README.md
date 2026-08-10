# Modular Control Panel

**Game-specific control panels for an arcade cabinet — each one recreating the controls of the
original machine as closely as possible.**

A single cabinet can't do justice to every game: Smash TV wants twin sticks, a fighter wants a
6-button cluster, Centipede wants a trackball, Tempest wants a spinner. Rather than compromise on
one fixed layout, this project builds **a dedicated control panel per game (or per control family)**
and makes them interchangeable — each panel snaps into the same cabinet and plugs into the same
wiring seam. Swap the whole deck by unplugging a handful of patch cables. No soldering, no
re-wiring the cabinet.

Modularity is the means; the goal is the **closest-to-original experience for each game**.

Inspired by [this arcade build post][inspiration].

## Two halves of the project

### 1. The electrical / wiring design

A solderless, RJ45-based patch system built around an **Ultimarc I-PAC Ultimate I/O**. Each
control (joystick, RGB button, trackball/spinner) rides its own Cat5e jack; whole panels
unplug at a fixed patch field mounted near the I-PAC. Supports up to **4 players / 4 joysticks**
(including twin-stick), **ServoStik** 4-way/8-way restrictors, and an aux **trackball + spinner**
slot.

→ **Full build plan: [docs/wiring.md](docs/wiring.md)** — board sizing, typed jack pinouts,
architecture, and a bench-first verification checklist.

### 2. The physical / 3D-printed design

The mounting panels themselves are **3D printed**, so each control layout is a printable,
version-controlled part. The goal is **multi-color panels** (labels, player-color accents, and
control zones printed in-material rather than stickered) — targeting the **Prusa Core One L**
with **INDX** multi-material once it ships.

→ **CAD source: [modular_control_panel.FCStd](modular_control_panel.FCStd)** (FreeCAD) — the
cabinet control panel housing, the interchangeable panel blanks, per-game control bodies, control
mount test pieces, and build jigs all live in this one document.

## CAD model

### Orientation convention

**The control panel surface lies flush with the XY plane** (Z = 0 at the panel top). This is
deliberate: control layouts are drawn and edited as flat 2D sketches on XY, so positioning a
joystick, a button cluster, or a trackball cutout is a plain 2D exercise — no working on a tilted
datum plane, no compound angles in the sketch.

The **cabinet housing bodies are modeled at an angle** around that flat panel, so the panel ends up
at a comfortable playing tilt in the finished cabinet — **currently 6°**. In other words, the tilt
lives in the housing geometry, not in the panel sketches: the model is authored flat and presents
angled.

### Parameters

Driving dimensions live in a single **`Control Panel Adjustables`** VarSet (in the `Variables`
group) rather than being hard-coded in sketches, so the design re-solves when a number changes.

Dimensions are designed in **inches** (FreeCAD stores them internally as mm, so the property
editor shows the converted value). Notable ones:

| Variable | Value | Meaning |
| -------- | ----- | ------- |
| `ControlPanelAngle` | **6°** | Playing tilt of the panel in the cabinet |
| `ControlPanelHousingWidth` | 24″ (609.6 mm) | Overall cabinet control panel width |
| `ControlPanelDepth` | 9″ (228.6 mm) | Front-to-back panel depth |
| `ControlPanelThickness` | 15/32″ (11.91 mm) | Panel stock thickness |
| `ControlPanelHousingMaterialThickness` | 23/32″ (18.26 mm) | Plywood housing stock |
| `ControlPanelBlank3/4/6/8/10` | 3″/4″/6″/8″/10″ (76.2 – 254 mm) | Widths of the interchangeable panel blanks — the suffix *is* the width in inches |
| `PatchPanelWidth` / `PatchPanelHeight` | 17″ / 3.5″ (431.8 / 88.9 mm) | RJ45 patch field (ties into [docs/wiring.md](docs/wiring.md)) |

The rest are control-specific mounting dimensions (Ultimarc J-Stik and U-Trak, Happ buttons,
Williams 2-way sticks, Paradise black-top sticks) plus threaded-insert sizes. Vendor-spec
dimensions that are natively metric (e.g. the Ultimarc parts, M4/M5 inserts) are kept in mm.

### Changing the angle

Edit `ControlPanelAngle` in the VarSet and recompute — the housing bodies follow. The panel
sketches are unaffected, since they're authored flat on XY.

## Design goals

- **Original-accurate** — each panel reproduces its game's real control layout: stick type,
  button count and cluster, and any special control (trackball, spinner, twin-stick). Which input
  maps to which function is handled in software (I-PAC profiles / emulator keymaps), so accuracy is
  a design + mapping concern, not a re-wiring one.
- **Modular** — swap an entire game's control panel at a single pluggable seam.
- **Solderless** — patch cables at the seam; the I-PAC is punched down once and never touched again.
- **Bench-testable** — every panel is self-contained and can be verified off the cabinet.
- **Reproducible** — printable parts + documented pinouts + a laminated one-page reference that
  lives with the cabinet.

## Repository layout

```
modular_control_panel.FCStd   FreeCAD source — housing, panel blanks, control bodies, jigs
docs/                         Build & wiring documentation
  wiring.md                   RJ45/I-PAC modular wiring build plan (start here)
models/                       Exported printables (3MF/STL)
  build-jigs/                 Setup & practice prints (angle calibration, heated inserts)
  control-mount-tests/        Fit-test mounts for individual controls
```

Planned additions:

```
print/         Slicer profiles, INDX multi-color setups, material notes
```

## Key hardware

| Part | Role |
|------|------|
| [Ultimarc I-PAC Ultimate I/O][ipac] | 48 switch inputs + 96 LED channels (24 RGB) — the brains |
| [Ultimarc ServoStik][servostik] | Software-controlled 4-way/8-way joystick restrictors |
| U-Trak trackball / SpinTrak spinner | Aux-slot pointing devices (quadrature, read as a mouse) |
| Cat5e + RJ45 keystones / patch panel | The modular interconnect (colored, typed jacks) |
| Prusa Core One L + INDX *(planned)* | Multi-color 3D printing of the mounting panels |

## Status

Design phase. The wiring architecture is worked out, and the CAD model is underway: the cabinet
housing, the panel blank sizes, and the first per-game control bodies (Q*Bert, Joust, Toobin')
are modeled, with control-mount fit tests printed for the Ultimarc J-Stik and U-Trak and the
Williams 2-way stick. Slicer/print profiles are the next milestone.

[inspiration]: https://www.facebook.com/groups/2370133573/posts/10166784545748574
[ipac]: https://www.ultimarc.com/control-interfaces/i-pacs/i-pac-ultimate-i-o/
[servostik]: https://www.ultimarc.com/arcade-controls/joysticks/servostik/
