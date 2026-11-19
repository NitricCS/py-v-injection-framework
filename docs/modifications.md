# Entropy and fault-injection modifications

This project modifies [Py-V](https://github.com/kyaso/py-v) to support instruction-based entropy extraction and instruction-level fault injection.

## General idea

Entropy is injected into unused bits of the RISC-V `funct7` field in R-type instructions. The entropy-enabled core extracts six bits from each applicable instruction, collects 16 values into a 96-bit buffer, and stores them in memory while normal execution is paused.

An entropy-carrying binary ends with the STOP instruction `0xffffffff`. The STOP instruction starts the mode switch: the program counter is reset and the normal binary instructions begin execution.

## Extractor

`pyv/extractor.py` introduces the `Extractor` module and the interfaces:

- `IFXT_t`: instruction supplied from the fetch stage
- `XTIF_t`: extracted entropy and active, ready, and flush signals
- `TXT_t`: flush-complete feedback from the memory stage

The extractor:

1. Accepts fetched instructions.
2. Checks for R-type instructions.
3. Extracts six entropy bits from `funct7`.
4. Buffers up to 16 values.
5. Emits a flush signal for a full or final partial buffer.
6. Detects `0xffffffff` and signals the fetch stage to switch modes.

## Fetch stage

`pyv/stages_entropy.py` adds an entropy-aware instruction-fetch stage.

During extraction, it:

- Supplies the original instruction to the extractor.
- Supplies NOP instruction `0x00000013` to the decode stage.
- Increments the program counter through the entropy-carrying instructions.
- Stops branching and program-counter progression while entropy is being flushed.
- Resets the program counter after the STOP instruction and returns to normal execution.

## Decode stage

The modified decode stage recognizes the STOP instruction and clears entropy-carrying `funct7` bits for R-type instructions during normal execution. The functional bit remains available to the ALU so ordinary instruction decoding continues correctly.

This keeps the modified implementation separate from the regular decode path.

## Memory stage

The modified memory stage receives entropy and flush signals from the fetch/extractor path.

When flushing begins:

- The partial or full entropy buffer is padded to 16 six-bit values if necessary.
- Entropy is written as three consecutive 32-bit values starting at address `1024`.
- Flush feedback is sent to the extractor.
- The extractor can continue only after the memory stage has completed the flush.

## Core model

`pyv/models/singlecycle_entropy.py` adds `SingleCycleEntropy` and `SingleCycleEntropyModel`.

The entropy core retains the 8 KiB memory and five-stage single-cycle structure of the regular core, with the extractor connected to fetch and memory stages.

`pyv/models/singlecycle.py` remains the regular core. Its binary-loading behavior and ISA handling are unchanged.

## Backward compatibility

A regular binary has no injected entropy and no STOP instruction. When run on `single_entropy`, the extractor finds no entropy values and uses zero-filled extraction data. The program should then execute and return the normal result.

Entropy-carrying binaries are not supported by `single` because that core does not implement the modified ISA use.

## Fault injection

The simulator supports selecting a fault-injection cycle through `setFICycle()`. `main.py` can run an example repeatedly at different cycles, compare its result and extracted entropy with expected values, and plot the outcome.

Faults are modeled by clearing selected instruction bits while instructions are fetched and before they reach the decode stage.

## Tests

The extraction tests cover:

- Entropy extraction from R-type instructions
- STOP-instruction handling
- Fetch-stage pipeline blocking
- Full and partial entropy flushing
- Regular and entropy-enabled core behavior
- Fault-injection and exception handling

See `test/test_extractor.py` and `test/test_stages_entropy.py`.
