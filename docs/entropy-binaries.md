# Creating entropy-carrying binaries

Entropy-carrying binaries place entropy in unused `funct7` bits of R-type instructions.

The repository uses [riscv-entropy_injection](https://github.com/owenmillwood/riscv-entropy_injection) for this step.

## Prerequisites

- A RISC-V source program
- A standard RISC-V GNU toolchain
- GNU Make
- Python 3.13 or newer
- `riscv-binutils.py`

The repository uses the `riscv64-unknown-elf-` prefix and RV32I/ILP32 options.

## Build an ELF output

For an included example:

```bash
cd programs
make fibonacci
```

This creates `fibonacci/fibonacci.out`.

For another program:

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

## Inject entropy

```bash
python3 riscv-binutils.py inject --image strcpy.out
```

The resulting ELF output must include the STOP instruction `0xffffffff` at the end of the entropy-carrying region.

## Convert to a binary

```bash
riscv64-unknown-elf-objcopy -O binary \
  your_program.out \
  your_program_e.bin
```

## Run with the entropy core

```python
import programs

programs.execute_bin(
    core_type="single_entropy",
    program_name="your program with entropy",
    path_to_bin="your_program_e.bin",
    num_cycles=3000,
)
```

Use `single_entropy` for entropy-carrying binaries. A regular binary can run on the entropy core, but it extracts no entropy.
