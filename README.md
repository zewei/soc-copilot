
# Welcome to SoC Copilot!


The SoC Copilot framework provides a convenient and efficient infrastructure to create SoCs.The SoC is generated using a custom generator written in [Migen HDL](https://github.com/m-labs/migen), which pulls together the CPU, peripherals, and creates the address mapping and the platform-support files needed to compile software for the core.

Build a 100% Python-based SoC Design environment! SoC-Copilot seeks for new design methodologies, collaboration approaches and creation of reusable IPs and subsystems. The main focus is on large and complex SoCs in which integration expertise is essential. The main goals are 1) creation of a versatile SoC template and 2) development of agile SoC development design process. 

**SoC Copilot framework quick overview: [Wiki][(https://github.com/zewei/soc-copilot/wiki/SoC-Copilot-Tutorial)]!**

The file `soc/rtl/pifive.py` declares the SoC architecture:
- It begins by creating a memory map, where each peripheral is assigned a base address, an end address, an addressing-mode (some peripherals assume byte-level addressing while others assume word-level addressing), a custom translation function (if needed), and whether to include the peripheral in the platform-support header file.
- It then defines the ports of the SoC (including the I/O pins, which are all tristate, as well as the clock, reset, and optionally the debug UART).
- Based on the defined ports, the I/O config is created, which defines which peripherals each I/O port should be able to switch between (by default, each I/O functions only as a GPIO, but it can be configured such that the software is able to switch it between multiple different peripherals).
- The I/O controller and other peripherals are instantiated and added using `add_periph`, which adds them to the shared interconnect.
- The high-performance memories (data and instruction SRAMs) are instantiated and added using `add_mem`, which places them on the crossbar interconnect.
- The CPU itself is instantiated and its busses are connected to the crossbar
- The bus is actually generated - a crossbar is generated with each of the memories as peripherals, as well as a bridge to the shared interconnect (to reduce resource utilization).
- The `platform.h`, `platform_init.h`, and `platform.ld` support files are created. These files contain information about the supported I/O modes and the addresses of the MMIO peripherals, and must be used when compiling software for the CPU.
- Once the entire SoC has been constructed, it is transformed into verilog and written out to a file (to be used for the FPGA build).

Full directory structure of SoC:
```
soc
├── Makefile
├── memory-subsystem - Chisel3 project containing the wishbone -> HyperRAM bridge and instruction cache
│   ├── Makefile
│   ├── build.sbt
│   ├── project
│   ├── src
│   └── testcases
├── rtl
│   ├── bootloader.py - Bootloader for loading programs onto the CPU
│   ├── build.py - Top-level program to build the SoC and generate verilog
│   ├── bus
│   │   ├── wishbone_bridge.py
│   │   ├── wishbone_debug_bus.py
│   │   ├── wishbone_external.py
│   │   └── wishbone_utils.py
│   ├── cpu.py - Wrapper for the CPU itself (into a Migen class with a wishbone port)
│   ├── debug - Debug-bus related modules
│   │   ├── debug_mem.py
│   │   ├── debug_probe.py
│   │   └── inst_buffer.py
│   ├── io_control.py - Multiplexes I/Os between peripherals and allows software control of GPUO
│   ├── periphs
│   │   ├── i2c.py
│   │   ├── pwm.py
│   │   ├── ram_subsystem.py
│   │   ├── spi.py
│   │   ├── timer.py
│   │   └── uart.py
│   ├── pifive.py - Defines the actual SoC itself and connects all the peripherals
│   ├── soc.py - Generic SoC class - handles bus configuration given the list of peripherals
│   ├── third_party
│   │   └── wishbone.py - Wrapper for wishbone bus, SRAM, Arbiter, and Decoder (from LiteX)
│   ├── util.py
│   └── verilog - Verilog modules (for UART and DebugBus)
│       ├── sync_2ff.sv
│       ├── uart_fifo.sv
│       ├── uart_rx.sv
│       ├── uart_tx.sv
│       ├── wbdbgbus.sv
│       └── wbuart.sv
└── third_party - Third-party modules used as SoC peripherals
    ├── hyperram
    ├── spi-controller
    │   └── spi_controller.v
    └── verilog-i2c
```

To run a generic build of the SoC:
```bash
cd soc/
make build/top.v
```


# What is the LiteX?

LiteX provides all the common components required to easily create an SoC:[Wiki](https://github.com/enjoy-digital/litex/wiki)!
 - :heavy_check_mark: Buses and Streams (Wishbone, AXI, Avalon-ST) and their  interconnect.
 - :heavy_check_mark: Simple cores: RAM, ROM, Timer, UART, JTAG, etc….
 - :heavy_check_mark: Complex cores through the ecosystem of cores: [LiteDRAM](https://github.com/enjoy-digital/litedram), [LitePCIe](https://github.com/enjoy-digital/litepcie), [LiteEth](https://github.com/enjoy-digital/liteeth), [LiteSATA](https://github.com/enjoy-digital/litesata), etc...
 - :heavy_check_mark: Various CPUs & ISAs: RISC-V, OpenRISC, LM32, Zynq, X86 (through a PCIe), etc...
 - :heavy_check_mark: Mixed languages support with VHDL/Verilog/(n)Migen/Spinal-HDL/etc... integration capabilities.
 - :heavy_check_mark: Powerful debug infrastructure through the various [bridges](https://github.com/enjoy-digital/litex/wiki/Use-Host-Bridge-to-control-debug-a-SoC) and [Litescope](https://github.com/enjoy-digital/litescope).
 - :heavy_check_mark: Direct/Fast simulation through [Verilator](https://www.veripool.org/verilator/).
 - :heavy_check_mark: Build backends for open-source and vendors toolchains.
 - :heavy_check_mark: And a lot more... :)

LiteX was initially developed by [Enjoy-Digital](http://enjoy-digital.fr/) to create projects for clients (and we are still using it for that :)) and trying to take the different clients' requirements/needs consideration made, we think, the framework very flexible:
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
