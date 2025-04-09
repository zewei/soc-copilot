
# Welcome to SoC Copilot!


The SoC Copilot framework provides a convenient and efficient infrastructure to create SoCs.

**SoC builder framework quick tour/overview: [Wiki](https://github.com/enjoy-digital/litex/wiki)!**

SoC Copilot provides all the common components required to easily create an SoC:
 - :heavy_check_mark: Buses and Streams (Wishbone, AXI, Avalon-ST) and their  interconnect.
 - :heavy_check_mark: Simple cores: RAM, ROM, Timer, UART, JTAG, etc….
 - :heavy_check_mark: Complex cores through the ecosystem of cores: [LiteDRAM](https://github.com/enjoy-digital/litedram), [LitePCIe](https://github.com/enjoy-digital/litepcie), [LiteEth](https://github.com/enjoy-digital/liteeth), [LiteSATA](https://github.com/enjoy-digital/litesata), etc...
 - :heavy_check_mark: Various CPUs & ISAs: RISC-V, OpenRISC, LM32, Zynq, X86 (through a PCIe), etc...
 - :heavy_check_mark: Mixed languages support with VHDL/Verilog/(n)Migen/Spinal-HDL/etc... integration capabilities.
 - :heavy_check_mark: Powerful debug infrastructure through the various [bridges](https://github.com/enjoy-digital/litex/wiki/Use-Host-Bridge-to-control-debug-a-SoC) and [Litescope](https://github.com/enjoy-digital/litescope).
 - :heavy_check_mark: Direct/Fast simulation through [Verilator](https://www.veripool.org/verilator/).
 - :heavy_check_mark: Build backends for open-source and vendors toolchains.
 - :heavy_check_mark: And a lot more... :)

SoC Copilot was initially developed by [Enjoy-Digital](http://enjoy-digital.fr/) to create projects for clients (and we are still using it for that :)) and trying to take the different clients' requirements/needs consideration made, we think, the framework very flexible:
 - Some users only want to use it to easily interconnect their existing VHDL/Verilog/SV cores.
 - Some users are only interested to reuse the PCIe/Ethernet/SATA/etc cores as regular core and just integrate them in their traditional flow.
 - Some users with a hardware background start with the above approaches and then switch later to the full Python flow since find it more efficient.
 - Some users with a software background and fluent with Python start playing with FPGAs while they would probably never touch FPGA otherwise :)
 - Etc...

# Typical LiteX design flow:
```
                                      +---------------+
                                      |FPGA toolchains|
                                      +----^-----+----+
                                           |     |
                                        +--+-----v--+
                       +-------+        |           |
                       | Migen +-------->           |
                       +-------+        |           |        Your design
                                        |   LiteX   +---> ready to be used!
                                        |           |
              +----------------------+  |           |
              |LiteX Cores Ecosystem +-->           |
              +----------------------+  +-^-------^-+
               (Eth, SATA, DRAM, USB,     |       |
                PCIe, Video, etc...)      +       +
                                         board   target
                                         file    file
```
LiteX already supports various softcores CPUs: VexRiscv, Rocket, LM32, Mor1kx, PicoRV32, BlackParrot and is compatible with the LiteX's Cores Ecosystem:

| Name                                                         | Build Status                                                                                                                       | Description               |
| ------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| [LiteX-Boards](http://github.com/litex-hub/litex-boards)     | [![](https://github.com/litex-hub/litex-boards/workflows/ci/badge.svg)](https://github.com/litex-hub/litex-boards/actions)         | Boards support            |
| [LiteDRAM](http://github.com/enjoy-digital/litedram)         | [![](https://github.com/enjoy-digital/litedram/workflows/ci/badge.svg)](https://github.com/enjoy-digital/litedram/actions)         | DRAM                      |
| [LiteEth](http://github.com/enjoy-digital/liteeth)           | [![](https://github.com/enjoy-digital/liteeth/workflows/ci/badge.svg)](https://github.com/enjoy-digital/liteeth/actions)           | Ethernet                  |
| [LitePCIe](http://github.com/enjoy-digital/litepcie)         | [![](https://github.com/enjoy-digital/litepcie/workflows/ci/badge.svg)](https://github.com/enjoy-digital/litepcie/actions)         | PCIe                      |
| [LiteSATA](http://github.com/enjoy-digital/litesata)         | [![](https://github.com/enjoy-digital/litesata/workflows/ci/badge.svg)](https://github.com/enjoy-digital/litesata/actions)         | SATA                      |
| [LiteSDCard](http://github.com/enjoy-digital/litesdcard)     | [![](https://github.com/enjoy-digital/litesdcard/workflows/ci/badge.svg)](https://github.com/enjoy-digital/litesdcard/actions)     | SD card                   |
| [LiteICLink](http://github.com/enjoy-digital/liteiclink)     | [![](https://github.com/enjoy-digital/liteiclink/workflows/ci/badge.svg)](https://github.com/enjoy-digital/liteiclink/actions)     | Inter-Chip communication  |
| [LiteJESD204B](http://github.com/enjoy-digital/litejesd204b) | [![](https://github.com/enjoy-digital/litejesd204b/workflows/ci/badge.svg)](https://github.com/enjoy-digital/litejesd204b/actions) | JESD204B                  |
| [LiteSPI](http://github.com/litex-hub/litespi)               | [![](https://github.com/litex-hub/litespi/workflows/ci/badge.svg)](https://github.com/litex-hub/litespi/actions)                   | SPI/SPI-Flash               |
| [LiteScope](http://github.com/enjoy-digital/litescope)       | [![](https://github.com/enjoy-digital/litescope/workflows/ci/badge.svg)](https://github.com/enjoy-digital/litescope/actions)       | Logic analyzer            |

# Papers, Presentations, Tutorials, Links
**FPGA lessons/tutorials:**
- https://github.com/enjoy-digital/fpga_101

**Migen tutorial:**
- https://m-labs.hk/migen/manual

**OSDA 2019 paper/slides:**
- https://osda.gitlab.io/19/1.1.pdf
- https://osda.gitlab.io/19/1.1-slides.pdf

**Linux on LiteX-Vexriscv:**
- https://github.com/litex-hub/linux-on-litex-vexriscv

**RISC-V Getting Started Guide:**
- https://risc-v-getting-started-guide.readthedocs.io/en/latest/

# Sub-packages
**litex.gen**
Provides specific or experimental modules to generate HDL that are not integrated in Migen.

**litex.build:**
Provides tools to build FPGA bitstreams (interface to vendor toolchains) and to simulate HDL code or full SoCs.

**litex.soc:**
Provides definitions/modules to build cores (bus, bank, flow), cores and tools to build a SoC from such cores.

# Quick start guide
1. Install Python 3.6+ and FPGA vendor's development tools and/or [Verilator](http://www.veripool.org/).
2. Install Migen/LiteX and the LiteX's cores:

```sh
$ wget https://raw.githubusercontent.com/enjoy-digital/litex/master/litex_setup.py
$ chmod +x litex_setup.py
$ ./litex_setup.py --init --install --user (--user to install to user directory) --config=(minimal, standard, full)
```
  Later, if you need to update all repositories:
```sh
$ ./litex_setup.py --update
```

> **Note:** On MacOS, make sure you have [HomeBrew](https://brew.sh) installed. Then do, ``brew install wget``.

> **Note:** On Windows, it's possible you'll have to set `SHELL` environment variable to `SHELL=cmd.exe`.

3. Install a RISC-V toolchain (Only if you want to test/create a SoC with a CPU):
```sh
$ pip3 install meson ninja
$ ./litex_setup.py --gcc=riscv
```

4. Build the target of your board...:

Go to litex-boards/litex_boards/targets and execute the target you want to build.

5. ... and/or install [Verilator](http://www.veripool.org/) and test LiteX directly on your computer without any FPGA board:

On Linux (Ubuntu):
```sh
$ sudo apt install libevent-dev libjson-c-dev verilator
$ litex_sim --cpu-type=vexriscv
```

On MacOS:
```sh
$ brew install json-c verilator libevent
$ brew cask install tuntap
$ litex_sim --cpu-type=vexriscv
```

6. Run a terminal program on the board's serial port at 115200 8-N-1.

  You should get the BIOS prompt like the one below.
