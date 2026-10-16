# Batch Counter on a Conveyor: PLC Project

A PLC program that counts products on a conveyor, stops the belt when a batch is complete, sounds a buzzer for 3 seconds, and resets the count for the next batch. This repository also includes a browser simulator that runs the same logic, so the program can be tested without hardware.

**Live demo:** https://Sumit-108-1.github.io/batch-counter-plc/

## Features

- Start/Stop control with a latched run bit and interlock
- Product counting with a CTU counter (preset = 12)
- 3-second buzzer using a TON timer when the batch is complete
- Automatic reset after the buzzer, plus a manual reset button
- Two-digit count display (tens and ones digits split with DIV, MUL and SUB)
- Live simulator with an animated conveyor and a ladder view that lights up as contacts turn on

## How it works

1. Press **Start**. The run latch `%M0.0` turns on and the conveyor motor `%Q0.0` runs.
2. Each product passing the sensor `%I0.2` adds 1 to counter C0, but only while the conveyor is running.
3. At 12 products, C0 turns on. This breaks the run latch, so the conveyor stops and the count stays at 12.
4. The batch-done flag `%Q0.2` starts timer T20 (3 s) and the buzzer `%Q0.3` sounds until T20 finishes.
5. When T20 finishes, `%M0.1` resets the counter. The count returns to 0 and the line waits for **Start**.
6. **Stop** pauses the line without losing the count.

Because C0 is part of the run latch, pressing Start while the buzzer is sounding does not restart the conveyor.

## I/O map

| Address | Function | Type |
|---|---|---|
| `%I0.0` | Start button | Input |
| `%I0.1` | Stop button (NC contact in logic) | Input |
| `%I0.2` | Product sensor | Input |
| `%I0.3` | Manual count reset | Input |
| `%Q0.0` | Conveyor motor | Output |
| `%Q0.1` | Counter reset coil | Output |
| `%Q0.2` | Batch done flag | Output |
| `%Q0.3` | Buzzer | Output |
| `%M0.0` | Run latch | Memory |
| `%M0.1` | Auto reset request | Memory |
| `C0` | Batch counter, PV = 12, value in `%VW0` | Counter |
| `T20` | Buzzer timer, PT = 30 (3 s), value in `%VW6` | Timer |
| `%VW2` / `%VW4` | Tens digit / ones digit | Word |
| `%VW8` | Helper word (tens x 10) | Word |

## Ladder program (Networks 0 to 9)

| Network | Purpose |
|---|---|
| 0 | Conveyor run latch: Start latches, Stop or batch complete breaks it |
| 1 | Conveyor motor output |
| 2 | Counter reset: auto after buzzer or manual button |
| 3 | Batch counter: counts products while the conveyor runs, preset 12 |
| 4 | Buzzer timer: 3 s delay starts when the batch is complete |
| 5 | Buzzer output: on for 3 s after the batch is complete |
| 6 | Auto reset request when the buzzer timer finishes |
| 7 | Count display: tens digit = count / 10 |
| 8 | Count display: tens x 10 (helper for the ones digit) |
| 9 | Count display: ones digit = count - (tens x 10) |

## Testing the program

| Step | Action | Expected result |
|---|---|---|
| 1 | Pulse the sensor before Start | Nothing is counted |
| 2 | Press Start | `%M0.0` and `%Q0.0` turn on |
| 3 | Pulse the sensor 5 times | `%VW0` = 5 |
| 4 | Press Stop | Conveyor stops, count stays at 5 |
| 5 | Press Start and count to 12 | Conveyor stops, buzzer on, display holds 12 |
| 6 | Wait 3 s | Buzzer off, count resets to 0 |
| 7 | Press Start | New batch begins |
| 8 | Press manual reset mid-batch | Count returns to 0 |

## Running the simulator

Open `index.html` in any browser. No installation or build step is needed. Use **Start**, **Stop**, **Add product**, **Auto feed** and **Reset count** to operate it.

To host it on GitHub Pages, go to **Settings > Pages**, choose the `main` branch and the `/ (root)` folder, then save.

## Files

- `index.html`: the simulator and ladder viewer (single file)
- `README.md`: this document

## Tools

- PLC programming software: Kinco Builder (ladder logic)
- Simulator: HTML, CSS and JavaScript
