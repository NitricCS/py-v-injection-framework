# Modified Py-V

This project is a modification of [Py-V](https://github.com/kyaso/py-v), a cycle-accurate RISC-V simulator and RTL modeling library written in Python.

The modification adds instruction-based entropy extraction and simulation-time fault injection. It adds an entropy-capable core while retaining the regular core and the ability to execute ordinary binaries.

## Requirements

- Python 3.13 or newer
- Python packages listed in `requirements.txt`
- GNU Make
- A standard RISC-V GNU toolchain using the `riscv64-unknown-elf-` prefix

Install Python dependencies:

```
python3 -m pip install -r requirements.txt
```

The RISC-V toolchain is only required to compile source programs. It is not required to run an already compiled `.bin` file.

## Compile example programs

From the `programs` directory:

```
make
```

The Makefile builds example programs for RV32I.

Source programs and generated binaries may be replaced with your own. The repository also contains precompiled regular and entropy-carrying example binaries.

## Run a simulation

Configure and run one example call in `main.py`:

```
python3 main.py
```

`execute_bin()` accepts the core type and binary path:

```python
execute_bin(
    core_type="single",
    program_name="fibonacci",
    path_to_bin="programs/fibonacci/fibonacci.bin",
    num_cycles=140,
)
```

Available core types:

- `single`: regular Py-V core
- `single_entropy`: modified core with entropy extraction

The example helpers in `main.py` demonstrate both core types, including `fibonacci()`, `fibonacci_entropy()`, `memset()`, `memset_entropy()`, `strcpy()`, and `strcpy_entropy()`.

Enable only one top-level example call in `main()` at a time.

## Core and binary compatibility

| Core | Binary | Supported behavior |
| --- | --- | --- |
| `single` | Regular | Normal execution |
| `single_entropy` | Regular | Backward-compatible normal execution; no entropy is extracted, so extraction remains inactive and the result is unchanged |
| `single_entropy` | Entropy-carrying | Entropy extraction and normal program execution after the mode switch |
| `single` | Entropy-carrying | Not supported; the regular core does not implement the modified ISA use |

Backward compatibility means that regular binaries can run on the modified core.

## Creating entropy-carrying binaries

Entropy-carrying binaries are created outside the simulator using the entropy-injection utility:

<https://github.com/owenmillwood/riscv-entropy_injection>

A typical flow is:

```bash
make -C programs fibonacci
python3 riscv-binutils.py inject --image programs/fibonacci/fibonacci.out
riscv64-unknown-elf-objcopy -O binary \
  programs/fibonacci/fibonacci.out \
  programs/fibonacci/fibonacci_e.bin
```

See [Entropy-carrying binaries](docs/entropy-binaries.md) for the complete workflow.

## Fault injection

The modified core supports instruction-level fault injection during simulation. Pass a fault-injection cycle to `execute_bin()` or to an example helper:

```python
fibonacci_entropy(core_type="single_entropy", fi_cycle=30)
```

The fault is applied to fetched instructions during the selected cycle. `main.py` also contains helpers for collecting fault outcomes and plotting results.

## Documentation

- [Modification implementation](docs/modifications.md)
- [Creating entropy-carrying binaries](docs/entropy-binaries.md)
- [Original Py-V project](https://github.com/kyaso/py-v)

## Tests

Run the test suite from the repository root using `pytest`.
