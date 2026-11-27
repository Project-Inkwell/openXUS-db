# openXUS-db


This repository holds an open, and permissively licensed device database for the Xilinx
UltraScale+ family of FPGAs. Currently, the only (experimentally) supported device is
the xcku3p, however we hope to have full support for this chip, and to expand the database
to include more UltraScale+ chips over the next couple of months. 

> NOTE: This database is not yet verified to be correct. Use with caution.

For each supported device family, this repository will hold the device's routing graph
and its sites, timing information, which bits in the bitstream configures which unit
(pip, LUT, etc), and all other necessary artifacts.

The repository layout is similar to that of the Project X-Ray's databases, so integration
this integrates nicely with himbaechel.

The version of [nextpnr](https://github.com/YosysHQ/nextpnr) that we use, is itself a
fork of [openXC7](https://github.com/openXC7/nextpnr), and can be found here:  
https://github.com/Project-Inkwell/nextpnr

> NOTE: You must check out the correct branch (`ultrascale`) in that repo!

We work on top of openXC7, since openXC7 provides some features that make adding support
for large FPGAs a lot easier, and because development of openXC7 moves a lot faster than
nextpnr.

## Provenance

The data in this repository was generated with Kraken, our custom bitstream documentation tool,
which we hope to publish some time in the future. The device structure (tiles, sites, wires,
pips, nodes and package pins) was queried from the vendor toolchain (Vivado). The configuration
bits (`segbits`, `defaults`, `functions`) were found by bitstream fuzzing: Kraken generates
hypotheses, proposes experiments and verifies the results (e.g. through HiTL). This lets it run
continuously and massively cuts down the time needed to generate a device database.

## Using it

To build a nextpnr chipdb from this database:

    zstd -d xcku3p/xcku3p/nodes.txt.zst
    PYTHONPATH=<openXUS>/himbaechel python3 <openXUS>/himbaechel/uarch/xilinx/gen/xilinx_gen.py \
        --xray xcku3p --metadata xcku3p --device xcku3p --bba ku3p.bba


## What works so far

On the card, openXUS places and routes from this database a blinky, the SERV RISC-V core and
the PicoRV32 RISC-V core running its program from block RAM. The behavior of our openXUS-generated
bitstreams are verified to behave identical with ones generated with the vendor toolchain.


## Development and useful scripts

Most of the development work in this project happens here: https://github.com/Project-Inkwell/openXUS-flow


## Layout

| file                                         | holds |
| -------------------------------------------- | --- |
| `xcku3p/tile_type_<type>.json`               | all 186 tile types (this includes wires, pips, sites, pin wiring) |
| `xcku3p/site_type_<type>.json`               | each site type and its alternatives: bels, bel pins, site internal pips |
| `xcku3p/xcku3p/tilegrid.json`                | all 106821 tiles (this includes place, type/sites, configuration frames, bit offsets |
| `xcku3p/xcku3p/nodes.txt.zst`                | all 8606711 nodes, a line each of `<tile>/<wire>`, analogous to X-Ray's `tileconn.json`. this is zstd compressed because of github file size limits |
| `xcku3p/wire_intents.json`                   | wire intents/kinds (`NODE_SINGLE`, `NODE_HLONG`, ...) by tile type |
| `xcku3p/xcku3p-ffvb676-2-e/package_pins.csv` | all 362 package pins (bank, site, tile and function) |
| `xcku3p/xcku3p-ffvb676-2-e/frames.json`      | configuration frames (their addresses and lengths) |
| `xcku3p/segbits_<type>.db`                   | the corresponding configuration bits of each tile type. analogous to X-Ray's segbits |
| `xcku3p/defaults_<type>.db`                  | default/template configuration bits |
| `xcku3p/functions_<type>.json`               | configuration bits that dont only rely on a single value, e.g. BRAM content and width, the clock tile buffers, etc. config is expressed as a function the tile settings |

## Not there yet

- Timing: nextpnr times nothing on this part yet, so it neither meets setup   nor fixes hold.
- The bits of most hard blocks' settings (the transceivers, the PCIe block, the MMCMs and PLLs,
  the DSPs), and of the IO logic.


## License

The data in this repository is dedicated to the public domain under
[CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/). See [LICENSE](LICENSE).

## Disclaimer

This project is not affiliated with, endorsed by, or sponsored by AMD or Xilinx. AMD, Xilinx,
UltraScale+, Kintex and Vivado are trademarks of Advanced Micro Devices, Inc. They are used here
only to describe the devices this database is intended for.
