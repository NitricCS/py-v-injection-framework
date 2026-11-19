# Modified Py-V

This project is a fork/modification of [Py-V](https://github.com/kyaso/py-v), a cycle-accurate RISC-V simulator and RTL modeling library written in Python.

The modification adds instruction-based entropy extraction and configurable instruction-level fault injection.

## Requirements

- Python 3.13 or newer
- Python packages listed in `requirements.txt`
- GNU Make
- A standard RISC-V GNU toolchain using the `riscv64-unknown-elf-` prefix

Install dependencies:

```bash
python3 -m pip install -r requirements.txt
```

The RISC-V toolchain is only required to compile source programs. It is not required to run an already compiled `.bin` file.

## Repository layout

- `main.py` - top-level entry point. It selects which program helper should run.
- `programs.py` - binary execution helpers and core selection.
- `fi_test.py` - fault-injection test runner, result collection, SQLite storage, and plotting.
- `pyv/` - simulator and core implementation.
- `programs/` - source programs, build rules, and precompiled example binaries.
- `test/` - pytest-based unit tests.
- `docs/` - implementation and usage documentation.
- `data/` - generated SQLite fault-injection results.
- `figs/` - generated fault-injection plots.

The `data/` and `figs/` directories are runtime output locations. They should be created before running result collection or plotting.

## Compile example programs

From the `programs` directory:

```bash
make
```

The Makefile builds `loop_acc`, `fibonacci`, and `endless_loop` for RV32I.

Source programs and generated binaries may be replaced with your own.

## Run a simulation

There is no separate CLI. Configure one call in `main.py`, then run:

```bash
python3 main.py
```

The default entry point currently calls `programs.atoi()`.

Program helpers and binary execution are implemented in `programs.py`. `execute_bin()` selects the core and optionally configures fault-injection parameters:

```python
fi_params = (30, 12, 2, "clear")

programs.execute_bin(
    core_type="single_entropy",
    program_name="fibonacci with entropy",
    path_to_bin="programs/fibonacci/fibonacci_e.bin",
    num_cycles=178,
    fi_params=fi_params,
)
```

Available core types:

- `single` - regular core
- `single_entropy` - core with entropy extraction

Example helpers include:

- `fibonacci()` / `fibonacci_entropy()`
- `atoi()` / `atoi_entropy()`
- `strcpy()` / `strcpy_entropy()`
- `memset()` / `memset_entropy()`

Enable or call only the example you want to run.

## Core and binary compatibility

| Core | Binary | Behavior |
| --- | --- | --- |
| `single` | Regular | Normal execution |
| `single_entropy` | Regular | Backward-compatible normal execution; no entropy is extracted |
| `single_entropy` | Entropy-carrying | Entropy extraction followed by normal execution |
| `single` | Entropy-carrying | Not supported |

Backward compatibility means that regular binaries can run on the modified core.

Entropy-carrying binaries require the STOP instruction `0xffffffff`. The modified core uses NOP instruction `0x00000013` while extraction is active and stores extracted entropy at memory address `1024`.

## Creating entropy-carrying binaries

Use [riscv-entropy_injection](https://github.com/owenmillwood/riscv-entropy_injection) to place entropy into unused `funct7` bits:

```bash
python3 riscv-binutils.py inject --image strcpy.out
riscv64-unknown-elf-objcopy -O binary strcpy.out strcpy_e.bin
```

See [Creating entropy-carrying binaries](docs/entropy-binaries.md).

## Fault injection

Fault-injection parameters are passed as a tuple:

```text
(fi_cycle, fi_index, num_bits, fi_type)
```

Supported injection types are:

- `flip` - invert the selected bits
- `set` - set the selected bits to one
- `clear` - clear the selected bits to zero

The fetch stage applies the fault before the instruction reaches the decode stage in both the regular and entropy-enabled cores.

The high-level test runner is separate from normal program execution:

```bash
python3 fi_test.py
```

`fi_test.py`:

- runs a program for a range of cycles
- varies bit position and injection type
- categorises the result
- stores results in `data/fi.db`
- saves result plots to `figs/`

The database and required tables must be prepared for the selected program before the runner can insert results.

See [Fault-injection testing](docs/fault-injection.md) for parameters, result categories, and runner configuration.

## Tests

Run the unit test suite:

```bash
pytest
```

## Documentation

- [Entropy and core modifications](docs/modifications.md)
- [Creating entropy-carrying binaries](docs/entropy-binaries.md)
- [Fault-injection testing](docs/fault-injection.md)
- [Original Py-V project](https://github.com/kyaso/py-v)
