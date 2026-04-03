# AutoChess

An autonomous chess board project that combines:
- **Mechanical piece movement** (XY stepper gantry + electromagnet)
- **Board state sensing** (reed switches via multiplexers)
- **On-board chess engine** (MicroMax-based logic)
- **Dual-microcontroller control flow** (Arduino Mega + ESP-based board)

![AutoChess banner](EXMO/main_banner.png)

---

## Overview

This repository contains firmware, PCB design files, and project assets for a physical chess board that:
1. Detects the human player’s move from sensors,
2. Validates and evaluates the move with an embedded chess engine,
3. Computes the AI response,
4. Physically moves pieces on the board.

The project appears structured for **Human vs Computer (HvsC)** gameplay with game-state handling (stalemate, invalid move, wins, punishment flow).

---

## Repository Structure

```text
autochess/
├── node_game &mega_game_algo/
│   ├── mega_game_algo/
│   │   ├── mega_game_algo.ino
│   │   └── global.h
│   └── node_game_algo/
│       └── node_game_algo.ino
├── Mux_codes/
│   └── module_testing/
│       └── module_testing.ino
├── pcb/
│   ├── autonomous chess playing machine/
│   └── libraries/
├── pcb_to_entc/
│   ├── acpm-arduino_motor_part/
│   ├── acpm-mux_part/
│   └── pre_final_to_entc/
├── EXMO/
│   └── main_banner.png
├── Datasheets/
├── AI Group 10 - Project Report.pdf
└── Autunomous Chess Board.pptx
```

---

## System Architecture

### 1) Arduino Mega firmware (`mega_game_algo.ino`)
- Handles the **main game loop/state machine**
- Reads board sensors through multiplexers
- Controls:
  - Stepper motors (X/Y movement)
  - Electromagnet pickup/drop
  - LCD display
  - Buttons and limit switches
- Sends player move to AI controller and receives AI result/move through `Serial2`

### 2) ESP/Node firmware (`node_game_algo.ino`)
- Runs a **MicroMax-style chess engine**
- Validates incoming human move
- Computes the computer move
- Returns:
  - status codes (stalemate/invalid/win/etc.), or
  - a 4-char move coordinate (e.g. `e2e4`)

### 3) Sensor subsystem
- Reed switch matrix read through mux lines (`s0..s3`) and mux inputs (`m0..m3`)
- Board occupancy stored in `arr[8][8]` and compared against backups to detect move origin/destination

### 4) Motion subsystem
- Two stepper channels (white/black motor pins in code naming)
- Calibrates against limit switches
- Supports:
  - straight moves
  - diagonal moves
  - knight movement path handling
  - castling handling
  - capture handling with resting zones

---

## Communication Protocol (Mega ↔ Node)

### Mega → Node
- Sends player move as **4 ASCII chars + newline**
- Example: `e2e4\n`
- Special restart signal: `R`

### Node → Mega
- Returns either a move (`a1b2` format) or status code.
- Mega checks first character:
  - `S` → stalemate
  - `N` → no valid move
  - `P` → player wins
  - `C` → computer wins
  - `I` → punishment
  - otherwise parse as move string

---

## Firmware Entry Points

- Mega setup/loop:
  - `/home/runner/work/autochess/autochess/node_game &mega_game_algo/mega_game_algo/mega_game_algo.ino`
- Node setup/loop:
  - `/home/runner/work/autochess/autochess/node_game &mega_game_algo/node_game_algo/node_game_algo.ino`
- Shared globals/constants:
  - `/home/runner/work/autochess/autochess/node_game &mega_game_algo/mega_game_algo/global.h`
- Multiplexer test sketch:
  - `/home/runner/work/autochess/autochess/Mux_codes/module_testing/module_testing.ino`

---

## Hardware Pin Mapping (from firmware)

### Mega-side key pins
- Multiplexer select: `s0=24`, `s1=26`, `s2=28`, `s3=30`
- Multiplexer read inputs: `m0=36`, `m1=38`, `m2=40`, `m3=42`
- Motors:
  - black step/dir: `29` / `27`
  - white step/dir: `47` / `45`
- Electromagnet: `53`
- Buttons: white `12`, black `11`
- Limit switches: white `23`, black `25`
- Reset pin to AI board: `50`

### Node-side key pins
- Serial + engine logic only in provided sketch (no board actuator pins)

> Confirm physical wiring against PCB schematics before powering hardware.

---

## Game Flow (HvsC)

1. System starts and displays startup screen.
2. Calibration homes trolley and moves to start position.
3. White (human) move is detected from reed sensor changes.
4. On white button press, move is sent to AI controller.
5. AI responds with status or computer move.
6. If move provided, Mega performs physical black/computer movement.
7. Repeat until game-over state.

---

## PCB and Manufacturing Files

Primary PCB resources are under:
- `/home/runner/work/autochess/autochess/pcb/`
- `/home/runner/work/autochess/autochess/pcb_to_entc/`

Includes:
- `.kicad_sch`, `.kicad_pcb`, `.kicad_pro` design files
- `gerbers/` output folders
- related custom libraries for DRV8825 modules
- printable board PDFs in `pcb_to_entc/pre_final_to_entc/`

---

## How to Build / Flash Firmware

No automated build scripts are included in this repository. Use the Arduino toolchain.

### Suggested process
1. Open `.ino` files in Arduino IDE / Arduino CLI.
2. Select the correct board/port for each firmware target:
   - Mega sketch: `mega_game_algo.ino`
   - Node/ESP sketch: `node_game_algo.ino`
3. Install required libraries (as referenced in source), including:
   - `LiquidCrystal_I2C`
   - standard Arduino core libraries
4. Compile and upload each sketch to its target board.
5. Open Serial Monitor(s) at `9600` baud for diagnostics.

---

## Testing and Validation

### Available in-repo validation artifact
- `Mux_codes/module_testing/module_testing.ino` for basic mux signal testing.

### Manual validation checklist
- Calibration reaches and respects both limit switches.
- Sensor scan detects a single move correctly (from/to squares).
- Serial protocol roundtrip works between boards.
- Electromagnet toggles reliably during pickup/drop.
- Piece movement logic works for:
  - straight, diagonal, knight, castling, captures.

---

## Known Notes / Limitations

- Repository currently has no CI, unit tests, or automated lint/build pipeline.
- Some directories include `desktop.ini` files from Windows environments.
- File and folder naming is historical and includes spaces/special characters.

---

## Documentation Assets

- Project report: `AI Group 10 - Project Report.pdf`
- Presentation: `Autunomous Chess Board.pptx`

These likely contain additional mechanical/electrical background and design rationale.

---

## Contributing

If you plan to contribute:
1. Keep firmware changes separated by module (Mega vs Node).
2. Document pin changes and protocol changes in this README.
3. Validate hardware safety (limit switches, motor directions, current limits) before long runs.

---

## License

No top-level project license file is currently present in this repository.
Some bundled PCB library components include their own license files under `pcb/libraries/.../License.txt`.
