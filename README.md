# SwimBladder

Power distribution board (PDB) project files for the SwimBladder project.

Silkscreen tag: `SOAR UWU | 8 x T200 | V1.1`

> [!WARNING]
> This board is an unvalidated, untested work in progress (WIP).
> It has not completed electrical, thermal, or in-water system validation.
> Do not treat this design as production-ready hardware.

## Project Status

- Current state: WIP (design in progress)
- Revision: V1.1
- Validation status: not electrically validated
- Test status: not thermally validated, not full-load validated, not field-tested
- Use at your own risk until formal verification is completed

## Repository Contents

- `KiCad Project Files v1.1/` - **Canonical** KiCad project, PCB, schematic, and the
  `SwimBladder Custom Footprints` library. Open `KiCad Project Files v1.1/UWU PDB.kicad_pro`.
- `KiCad Project Files/` - Deprecated earlier project set, kept for history. Do not edit.
- `SwimBladder PDB.kicad_sch` (repo root) - Stale May snapshot, superseded by the schematic
  inside `KiCad Project Files v1.1/`. Do not use.
- `Screenshots/` - Board and schematic screenshots. `v1.1 *.png` are current; the unprefixed
  files are the earlier V1.0 captures.
- `SwimBladderV1.0.step` - 3D board model (V1.0 geometry).
- `Swim_Bladder_Doc_v3_0.pdf` - Full engineering reference and calculations. Component values
  in this PDF predate the V1.1 schematic changes listed below.

## Project Overview

This board is designed as a compact 8-thruster central star-node PDB for Blue Robotics T200
systems with 4S battery input. It is currently in a pre-validation state and should be treated
as a reference design draft.

- System topology: battery -> cable-side main fuse -> optional e-stop/contactor -> dual XT90 input -> central high-current node -> 8 fused ESC branches -> XT60 outputs
- Nominal battery voltage: 14.8 V (4S), full charge: 16.8 V
- Thruster design current: about 25 A each
- Total design current: 200 A (with burst targets up to around 280-300 A)
- Per-branch fuse target: 30 A

## Schematic Summary (V1.1)

### Input / central node
- Dual XT90 input connectors (`J1`, `J19`) feeding one common +VBAT star node, symmetrical wiring for current sharing.
- Bulk capacitance at the node: `C_MAIN1`, `C_MAIN2` = 4700 uF each.
- Input TVS: `D1` (`D_TVS`), 22-24 V standoff class for 4S. Recommended part: SMDJ22A
  (unidirectional, 3 kW, DO-214AB / SMC footprint, `Diode_SMD:D_SMC` or `D_SMC_HandSolder`).
  See "Known issues" - placement needs correction in the current layout.
- Rail bleed / discharge: `R_BLEED1` = 100 Ohm. **Placeholder value, not final** - 100 Ohm
  draws ~168 mA / ~2.8 W continuously at full charge. A true bleed resistor should be in the
  tens of kOhm (e.g. 22k-47k); confirm the intent before fabrication.
- Power/status LED: `D2` (`LED`) with series resistor `R_LED1` = 10 kOhm.

### Per-ESC branch (x8)
- Branch fuse: `F_ESC1..F_ESC8` = 30 A.
- Branch bulk cap: `C_ESC1..C_ESC8` = 1000 uF.
- Branch high-frequency cap: `C_HF1..C_HF8` = 1 uF.
- Output: `J3..J10`, XT60 per branch.
- Branch decoupling is now two caps (1000 uF + 1 uF). The earlier third per-branch cap
  (1 nF high-frequency) has been removed - on a high-current motor branch it contributes
  negligible benefit and can create anti-resonance with the larger caps.

### PWM signal
- 16-pin IDC header `J2` (`Conn_02x08_Odd_Even`, 2x8, 2.54 mm), alternating PWM / GND pins.
  Suggested part footprint: `Connector_IDC:IDC-Header_2x08_P2.54mm_Vertical`.
- Individual PWM channel inputs: `J11..J18` (PWM1..PWM8).
- Per-channel series resistor: `R_PWM1..R_PWM8` = 100 Ohm.
- Per-channel pulldown resistor: `R_PD1..R_PD8` = 10 kOhm.

## Passive Footprint Notes

- PWM channel series resistors (`R_PWM*`): 0402 hand-solder land pattern
  (`Resistor_SMD:R_0402_1005Metric_Pad0.72x0.64mm_HandSolder`) for IDC header pitch.
- All other signal passives (`R_BLEED1`, `R_LED1`, `R_PD*`) and branch HF caps: 0603
  hand-solder (`R_0603_1608Metric_Pad0.98x0.95mm_HandSolder` /
  `C_0603_1608Metric_Pad1.08x0.95mm_HandSolder`).
- Custom / project-specific footprints live in `KiCad Project Files v1.1/SwimBladder Custom Footprints`.

## Key Engineering Notes (from the PDF)

- Architecture: single compact central +VBAT node, not dual banks, for a roughly 90 mm x 90 mm board.
- Safety: main fuse should be upstream on the battery cable (recommended 225-250 A), close to the battery.
- Copper strategy: use a short, wide, via-stitched central polygon (L1 + L3 for +VBAT), and strong common ground return (L2 + L4).
- Copper thickness for the 200 A / 300 A rating: 1 oz (35 um) is not sufficient. Spec 2 oz
  (70 um) minimum on the outer layers, 3 oz preferred, and carry +VBAT and GND on two layers
  each in parallel (~4 oz-equivalent per net) with dense inter-layer via stitching. Treat the
  central node as a plane, not a trace, and size pours/vias for <= 20-30 C rise. Per-branch
  (~25 A): 2 oz outer with ~10-15 mm pour width is adequate.
- Via stitching: use two via classes - large power vias (~1.2 mm pad / 0.6 mm drill or bigger)
  for the +VBAT node and ground return, and a smaller class (~0.6-0.8 mm pad / 0.3-0.4 mm drill)
  for signal-ground / thermal stitching around the PWM header.
- Branch geometry: practical branch widths around 12-15 mm (8-10 mm absolute compact minimum for very short sections).
- Connectors: dual XT90 input for current sharing (symmetrical wiring), XT60 per ESC branch preferred.
- Protection/passives: 22-24 V standoff TVS class for 4S input, bulk capacitance near input/node plus branch-level capacitors.
- Signal: 16-pin IDC PWM header with alternating PWM/GND and per-channel series + pulldown resistors.

## Known Issues / TODO (V1.1)

- **TVS placement:** `D1` currently sits at the bottom of the board next to the PWM header
  (`J2`), far from the battery input. It must move to the top-center input node, on the load
  side of the fuse/contactor, hard against the XT90 +VBAT pads and straddling the +VBAT and
  GND pours with local ground via stitching. Consider a second parallel SMDJ22A footprint at
  the same node if hot-disconnect under load is possible.
- **`R_BLEED1` value:** 100 Ohm is a placeholder - see Schematic Summary.
- **Input HF cap grouping:** keep any input high-frequency cap clustered at the XT90 pads with
  `C_MAIN1/2` and the TVS, not offset to one side.
- **`R_BLEED1` thermal:** relocate away from electrolytic bodies and the PWM header.
- **Indicator LEDs:** only `D2` is placed; the engineering notes call for more indicators.
- **Screenshots:** V1.1 captures added; regenerate again after the TVS relocation.

## Quick Calculation Snapshot

- Per thruster power at 16 V and 25 A: 400 W
- System power for 8 thrusters: 3200 W
- Example central copper section in the doc (50 mm width, 140 um thickness, 25 mm length, one layer):
	- Estimated R ~= 0.0616 mOhm
	- Vdrop at 200 A ~= 0.012 V
	- Copper loss at 200 A ~= 2.46 W

For complete assumptions, formulas, BOM guidance, and test plan, see `Swim_Bladder_Doc_v3_0.pdf`.

## Screenshots (V1.1)

### Schematic
<img src="Screenshots/v1.1%20schematic.png" alt="V1.1 schematic" width="760" />

### Routing
<img src="Screenshots/v1.1%20routing.png" alt="V1.1 routing" width="760" />

### 3D Render - Top
<img src="Screenshots/v1.1%20render%20top%20view.png" alt="V1.1 render, top view" width="760" />

### 3D Render - Isometric
<img src="Screenshots/v1.1%20render%20isometric%20view.png" alt="V1.1 render, isometric view" width="760" />

<sub>Earlier V1.0 captures (`Schematic.png`, `PCB Editor View.png`, `3D View.png`) remain in `Screenshots/` for reference.</sub>

## Getting Started

1. Install KiCad (recommended version 8+).
2. Open the project from `KiCad Project Files v1.1/UWU PDB.kicad_pro`.
3. Review schematic, layout, and engineering assumptions from the PDF before fabrication,
   noting the V1.1 changes and open TODO items above.

## Changelog

### V1.1 (in progress)
- Canonical project files moved to `KiCad Project Files v1.1/`; earlier `KiCad Project Files/`
  and the repo-root `SwimBladder PDB.kicad_sch` are deprecated.
- Added `SwimBladder Custom Footprints` library.
- Main bulk capacitance specified as `C_MAIN1/2` = 4700 uF each.
- Removed the 1 nF per-branch decoupling cap; branches now use 1000 uF + 1 uF only.
- PWM front-end defined: `R_PWM*` = 100 Ohm series, `R_PD*` = 10 kOhm pulldown, inputs `J11..J18`.
- Added rail bleed resistor `R_BLEED1` (value still a placeholder) and status LED `D2` / `R_LED1`.
- Input TVS `D1` present; placement correction pending (see Known Issues).

### V1.0
- Initial 8-branch central star-node PDB layout.

## License

This project is licensed under the MIT License. See the LICENSE file for details.
