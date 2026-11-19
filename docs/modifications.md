# Entropy and fault-injection modifications

This project modifies [Py-V](https://github.com/kyaso/py-v) to support instruction-based entropy extraction and instruction-level fault injection.

## General idea

Entropy is placed in unused bits of the RISC-V `funct7` field of R-type instructions. The entropy-enabled core extracts six bits from each applicable instruction, buffers 16 values, and stores 96 bits in memory while normal execution is paused.

The STOP instruction `0xffffffff` ends extraction. The fetch stage then resets the program counter and normal execution begins.

## Extractor

`pyv/extractor.py` introduces:

- `Extractor`
- `IFXT_t`, the instruction interface from fetch
- `XTIF_t`, the entropy and control-signal interface
- `TXT_t`, the flush-complete feedback interface

The extractor identifies R-type instructions, collects six bits from `funct7`, buffers up to 16 values, emits flush signals, and detects the STOP instruction.

## Fetch stage

`pyv/stages_entropy.py` provides the entropy-enabled fetch stage.

During extraction it:

- sends the original instruction to the extractor
- sends NOP instruction `0x00000013` to decode
- increments the program counter in instruction order
- pauses the program counter during buffer flushing
- resets to normal execution after the STOP instruction

The regular fetch implementation remains in `pyv/stages.py`.

## Decode and memory stages

The modified decode stage recognizes the STOP instruction and clears entropy-carrying `funct7` bits during normal R-type decoding. The functional bit remains available to the ALU.

The modified memory stage:

- pads an incomplete entropy buffer to 16 values
- writes three 32-bit values starting at address `1024`
- reports flush completion to the extractor

## Core model

`pyv/models/singlecycle_entropy.py` connects the extractor to the entropy-enabled core.

The regular core remains in `pyv/models/singlecycle.py`. A regular binary can run on the entropy core without extracted entropy. A regular core cannot execute an entropy-carrying binary correctly.

## Root-level organization

- `main.py` selects the top-level example
- `programs.py` implements `execute_bin()` and program helpers
- `fi_test.py` runs fault-injection sweeps and stores results
- `pyv/` implements the simulator and cores

See [Fault-injection testing](fault-injection.md) for fault parameters and runner behavior.
