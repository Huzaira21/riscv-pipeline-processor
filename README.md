# RISC-V 5-Stage Pipeline Processor

A fully implemented 5-stage pipelined RISC-V processor in Verilog.
Developed as part of Computer Architecture course at
Namal University, Mianwali.

## Pipeline Stages
- **IF** — Instruction Fetch
- **ID** — Instruction Decode
- **EX** — Execute
- **MEM** — Memory Access
- **WB** — Write Back

## Modules Implemented
| Module | Description |
|--------|-------------|
| Program Counter | PC with clock and reset |
| PC Adder & Mux | Next PC calculation |
| Instruction Memory | Fetch instructions |
| IF/ID, ID/EX, EX/MEM, MEM/WB | Pipeline registers |
| Register File | 32 general purpose registers |
| Main Control Unit | Instruction decoding |
| ALU + ALU Control | Arithmetic & logic operations |
| Hazard Detection Unit | Detects pipeline hazards |
| Forwarding Unit | Data forwarding (3x1 Mux) |
| Branch & Jump Control | Branching logic |
| Data Memory | Load/Store operations |
| Write Back Mux | Result write back |
| Immediate Generator | Immediate value extraction |

## Supported Instructions
- R-type, I-type, U-type, J-type
- Load/Store Operations
- Branch & Jump
- Memory Read/Write

## Features
- Complete hazard handling
- Data forwarding
- Branch resolution
- Clock & reset handling

## How to Run
Use any Verilog simulator:
- ModelSim
- Vivado
- EDA Playground (free, online — no installation needed)

## Skills Used
Verilog · Computer Architecture · Digital Logic Design ·
Pipeline Design · Hazard Detection · Data Forwarding
