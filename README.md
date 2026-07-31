# Orange Pi Zero 3W, PCIe-to-M.2 Carrier Board

Open-source KiCad design for a PCIe-to-M.2 carrier board for the Orange Pi Zero 3W (Allwinner A733). Breaks out the board's 18-pin PCIe FPC connector to an M.2 M-Key slot for NVMe storage, with an eye toward AI accelerator cards (e.g. Hailo-10H) as a longer-term stretch goal.

## Status

- Schematic: complete, ERC-clean (0 errors)
- PCB layout: complete, fully routed including differential pairs, DRC-clean (0 errors; remaining warnings are accepted and expected: 0.5mm-pitch connector solder mask clearances, deliberately downgraded to warnings, plus two harmless orphaned-geometry exclusions)
- Gerbers: exported and validated (confirmed parsing correctly on JLCPCB's quote tool, 65x32mm 2-layer board detected correctly)
- BOM: fully sourced via LCSC/JLCPCB part numbers (see below)
- **PCIe enumeration on this exact board is not yet confirmed.** As of writing, I could not find any public documentation of a PCIe device enumerating over the Zero 3W's own FPC connector specifically. That said, PCIe/NVMe has been documented working on other Allwinner A733 boards that share the same SoC and the same vendor PCIe driver, including the Orange Pi 4 Pro and the near-identical Radxa Cubie A7Z/A7A/A7S, subject to a known Gen3 enumeration bug in the vendor `sunxi-pcie` driver (Gen3 link trains but many Phison-controller NVMe drives fail to enumerate; forcing Gen1 or using a WD drive tends to work). This board exists to produce the first Zero 3W-specific test data.

**I'm not a professional electrical engineer,** this is a learning project. I've been careful and it's passed every check I know to run, but "passed DRC/ERC" is not the same as "guaranteed to work." If you can read a schematic, please review it. If something's wrong, tell me. If you have the budget to fab and test this before I do, please go for it. I'd genuinely rather someone else get the real answer sooner than have this sit unbuilt.

## Board summary

- 2-layer, 32x65mm (matches Zero 3W's own footprint and mounting hole spacing)
- Single TPS62086 buck regulator (fixed 3.3V/3A) powering the M.2 slot from the FPC connector's shared 5V rail, with added ferrite-bead input filtering
- PCIe x1 Gen1, single lane broken out (CLK, RX, TX, PERSTn, CLKREQn, WAKEn)
- GND/3V3 routed via copper pour and stitching vias; differential pairs routed same-layer via arc routing to resolve a J1/J2 pin-order mismatch

## Bill of Materials

| Ref | Part | LCSC/JLCPCB ID | Notes |
|---|---|---|---|
| U1 | TI TPS62086RLTR | C2071344 | Buck regulator, 2.5 to 6V in, fixed 3.3V/3A out |
| J1 | Molex 52746-1871 (alias of 52746-1870) | C114114 | 18-pin FPC ZIF, mates to Zero 3W's PCIe connector |
| J2 | Amphenol MDT670M01501 | C4523168 | M.2 M-Key slot, 75-pin |
| L1 | SHOU HAN CYA0630-1.0UH | C5189744 | 1uH inductor, buck stage |
| L2 | TAI-TECH HCB3216KV-501T30 | C5139915 | Ferrite bead, 5V input filtering |
| C1, C3 | Samsung CL21A106KAYNNNE | C15850 | 10uF, input and bulk |
| C2 | Samsung CL21A226MAQNNNE | C45783 | 22uF, output |

**Known supply issues at time of writing:** J1 was out of stock at JLCPCB (pre-order only). J2 had only single-digit stock remaining. Worth checking current availability before ordering, or sourcing these two from Molex/Amphenol's standard distributors (DigiKey, Mouser) directly if JLCPCB is short.

## Known open questions / issues

- **PCIe enumeration on the Zero 3W specifically is unconfirmed.** See Status above for what is and isn't known on sibling A733 boards.
- Kernel/BSP PCIe root-complex support is on the vendor Allwinner BSP kernel only; mainline Linux has no A733 PCIe controller driver yet.
- The vendor `sunxi-pcie` driver has a known Gen3 speed-change/equalization bug affecting some NVMe drives (notably Phison-controller models). Forcing the link to Gen1 via device tree, or a PCIe COMMAND-register remove/rescan fix, are the documented workarounds on sibling boards.
- GPIO 40-pin 5V power injection (for potentially powering an accelerator card back through the header) is documented as officially supported in Xunlong's manual, but not independently confirmed for stability under real accelerator load.
- Real-world total board power draw: one community data point suggests around 6.5W at 100% CPU load (untested under GPU load), against the board's rated 5V/3A (15W) ceiling. Suggests headroom exists but isn't confirmed exhaustively.

Discussion thread with more context and community replies: https://www.reddit.com/r/OrangePI/comments/1va42e7/zero_3w_building_a_custom_pcietom2_adapter/

## Repo contents

- `kicad files/`: full KiCad project (schematic, PCB, project file, design rules, library tables)
- `libraries/`: symbols, footprints, and 3D models for every part used, pre-linked to the project
- `Gerber and drill/`: exported, validated Gerber and Excellon drill files, ready for fab

## License

Released under the **CERN Open Hardware Licence v2, Permissive (CERN-OHL-P)**. This is the most open standard license for open hardware: anyone can use, build, modify, or sell this design, including for commercial purposes, without having to release their changes back. I have no plans to monetize this myself, but I'm not restricting others from building on it or selling assembled boards either. Full license text: https://ohwr.org/cern_ohl_p_v2.txt

## Contributing

Not trying to monetize or gatekeep this. If you build it, test it, find a bug, or improve on it, please open an issue or a pull request, or just reply on the Reddit thread above.