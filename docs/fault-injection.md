# Fault injection testing

The simulator supports instruction-level fault injection at the fetch stage. This follows the model described in the project dissertation: selected bits of a fetched RISC-V instruction are changed before the instruction reaches decode.

Fault injection is separate from normal program execution and is configured in `programs.py` or through `fi_test.py`.

## Fault parameters

The fault parameter tuple has the form:

```text
(fi_cycle, fi_index, num_bits, fi_type)
```

- `fi_cycle`: cycle at which the fetched instruction is modified
- `fi_index`: lowest/starting target bit
- `num_bits`: number of consecutive target bits
- `fi_type`: `flip`, `set`, or `clear`

The regular and entropy-enabled fetch-stage implementations support the same parameter types.

## Single run

Pass fault parameters to a program helper:

```python
import programs

fi_params = (30, 12, 2, "clear")
programs.fibonacci_entropy("single_entropy", fi_params)
```

The helper passes the tuple through `execute_bin()` to the model and simulator. The fetch stage applies the change only on the selected cycle.

## Test runner

`fi_test.py` contains the high-level test runner:

- `inject_faults()` runs one program for a cycle range and stores each result
- `run_fi_test()` iterates over configured bit positions
- `run_bad_bit_test()` tests `flip`, `set`, and `clear` across the configured bit range
- `Testbench` holds expected values and test-result helpers

Run the default configuration with:

```bash
python3 fi_test.py
```

Configure a run with a settings dictionary:

```python
settings = {
    "program": programs.memset,
    "core_type": "single_entropy",
    "cycle_start": 1,
    "cycle_end": 106,
    "fi_index": 0,
    "num_bits": 1,
    "fi_type": "flip",
}

fi_test.run_fi_test(settings)
```

The runner expects the database and output directories to exist and a matching SQLite table to have been created for the program.

## Result categories

The runner records:

- `no_effect` - the program result remains expected
- `target_meet` - the program output differs from its expected value
- `pc_out_of_bound` - execution reaches an invalid instruction address
- `seg_fault` - a memory access fails
- `misaligned_access` - an instruction or data access is misaligned
- `ill_inst` - an instruction is illegal or corrupted
- `funct_violation` - another processor exception is raised

Results are written to `data/fi.db`. Plots are saved below `figs/`.

## Faulting entropy-carrying binaries

For entropy-enabled tests, faults can affect both:

- instructions after the fetch stage
- instructions supplied to the entropy extractor

This makes cycle selection and target-bit selection significant. Faults before the STOP or mode switch can also affect extraction, while faults during normal execution affect the program pipeline.
