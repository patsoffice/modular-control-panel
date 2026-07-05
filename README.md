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

> **Status:** the mechanical/CAD side is not modeled yet. This repo starts from the wiring
> design; panel models, mounting interface, and print profiles come next.

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
docs/          Build & wiring documentation
  wiring.md    RJ45/I-PAC modular wiring build plan (start here)
```

Planned additions:

```
cad/           Panel models & the cabinet mounting interface (source + STL/3MF)
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

Early / design phase. The wiring architecture is worked out; CAD and print files are the
next milestone.

[inspiration]: https://www.facebook.com/groups/2370133573/posts/10166784545748574
[ipac]: https://www.ultimarc.com/control-interfaces/i-pacs/i-pac-ultimate-i-o/
[servostik]: https://www.ultimarc.com/arcade-controls/joysticks/servostik/
