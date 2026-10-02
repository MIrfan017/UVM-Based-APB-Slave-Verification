# apb-slave-uvm-verification

UVM verification environment for an APB memory slave (64-bit data, byte strobes, PSLVERR).
Includes driver, monitor, scoreboard with a reference memory model, and constrained-random write/read sequences.

> Status: work in progress

## DUT Overview

| Block | File | Description |
|---|---|---|
| Wrapper | `design.sv` | Top module `apb_wrapper`, connects FSM, memory and error generator |
| FSM | `apb_fsm.sv` | APB slave FSM (IDLE, SETUP, ACCESS) |
| Memory | `generic_mem.sv` | Parameterized memory with byte strobes and 2-stage registered read |
| Memory cell | `mem_1024x32.sv` | 1024 x 32 RAM, sync write, async read |
| Error gen | `err_gen.sv` | Out-of-bounds and misaligned address detection |

Default config: `ADDR_W=32`, `DATA_W=64`, `MEM_SIZE_K=64`, `BASE_ADDR=0`

## Folder Hierarchy

```
apb-slave-uvm-verification/
├── rtl/
│   ├── design.sv              # apb_wrapper (top)
│   ├── apb_fsm.sv
│   ├── generic_mem.sv
│   ├── mem_1024x32.sv
│   └── err_gen.sv
├── tb/
│   ├── apb_interface.sv       # apb_if
│   ├── apb_transaction.sv
│   ├── apb_sequencer.sv
│   ├── apb_driver.sv
│   ├── apb_monitor.sv
│   ├── apb_agent.sv
│   ├── apb_scoreboard.sv      # reference memory + checks
│   ├── apb_env.sv
│   ├── apb_base_seq.sv        # constrained-random writes
│   ├── apb_wr_rd_seq.sv       # write then read-back
│   ├── apb_base_test.sv
│   ├── apb_test.sv            # apb_random_test, apb_wr_rd_test
│   └── apb_top.sv             # package + tb_top
├── sim/                       # run scripts (to be added)
└── README.md
```

## Testbench Architecture

```
Test -> Sequence -> Sequencer -> Driver -> [ DUT ] -> Monitor -> Scoreboard
                                                     (ref memory + PSLVERR check)
```

## Progress

### Done
- [x] DUT (wrapper, FSM, memory, error generator)
- [x] APB interface and transaction
- [x] Driver (SETUP/ACCESS phases, waits for PREADY, timeout)
- [x] Monitor (captures completed transfers)
- [x] Agent and environment
- [x] Scoreboard with byte-level reference memory and PSLVERR prediction
- [x] Constrained-random write sequence (valid addresses)
- [x] Write-then-read sequence (full and partial strobes)
- [x] Tests: `apb_random_test`, `apb_wr_rd_test`

### In progress / To do
- [ ] Error-injection sequence (out-of-bounds and misaligned addresses)
- [ ] Error-write aliasing test (invalid write must not corrupt memory)
- [ ] Back-to-back transfer tests
- [ ] Functional coverage
- [ ] SVA protocol assertions in the interface
- [ ] Run scripts (Makefile / VCS command)
- [ ] Regression results

## Known Notes
- Scoreboard parameters (`BASE_ADDR`, `MEM_SIZE_B`, `DATA_W`) are hardcoded to match `apb_top.sv`.
- Bytes that were never written are not compared (DUT memory is uninitialized).
