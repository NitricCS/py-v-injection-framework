# Creating entropy-carrying binaries

Entropy-carrying binaries are regular RISC-V binaries with additional six-bit entropy values placed in unused `funct7` bits of R-type instructions.

The entropy-injection tool is [riscv-entropy_injection](https://github.com/owenmillwood/riscv-entropy_injection).

## Prerequisites

- A RISC-V source program
- A standard RISC-V GNU toolchain
- GNU Make
- Python 3.13 or newer
- The `riscv-binutils.py` entropy-injection utility

This repository uses the `riscv64-unknown-elf-` toolchain prefix and the RV32I/ILP32 ABI.

## 1. Build the ELF output

Use the existing Makefile for an included example:

```bash
cd programs
make fibonacci
```

This creates `fibonacci/fibonacci.out`. The Makefile compiles for RV32I with `-nostdlib -nostartfiles`.

For a source program of your own, compile it in the same way and produce an ELF output:

```bash
riscv64-unknown-elf-gcc \
  -march=rv32i \
  -mabi=ilp32 \
  -nostdlib \
  -nostartfiles \
  common/crt.S \
  your_program.c \
  -o your_program.out
```

## 2. Inject entropy

Run the injection utility on the ELF output:

```bash
python3 riscv-binutils.py inject --image strcpy.out
```

The exact utility syntax may vary with the released tool version. The command above is the repository's documented example.

The injected binary must contain the STOP instruction `0xffffffff` at the end of the entropy-carrying region. This instruction tells the modified core to stop extraction and return to normal execution.

## 3. Convert to a raw binary

Convert the injected ELF output to the raw `.bin` format expected by `main.py`:

```bash
riscv64-unknown-elf-objcopy -O binary \
  your_program.out \
  your_program_e.bin
```

For an existing example:

```bash
riscv64-unknown-elf-objcopy -O binary \
  programs/fibonacci/fibonacci.out \
  programs/fibonacci/fibonacci_e.bin
```

## 4. Run with the entropy core

Pass the raw binary and `single_entropy` core to `execute_bin()`:

```python
execute_bin(
    core_type="single_entropy",
    program_name="your program with entropy",
    path_to_bin="your_program_e.bin",
    num_cycles=3000,
)
```

Or add the program to an example helper in `main.py` and enable only that helper in `main()`.

## Compatibility notes

- Use `single_entropy` for entropy-carrying binaries.
- Do not use `single` for entropy-carrying binaries.
- A regular binary can be run on `single_entropy`, but it will extract no entropy.
- Extracted entropy is stored at memory address `1024`.
- The STOP instruction is `0xffffffff`.
