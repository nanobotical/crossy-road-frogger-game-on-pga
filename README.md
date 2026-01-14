# crossy-road-frogger-game-on-DE1-SOC


16×16 LED-matrix “Crossy Road” style game for the DE1-SoC board.  
**Sprite** (player) moves on a grid while **SPIKE** (traffic) scrolls across lanes. Overlap ⇒ **hit/freeze**.  
**Score** is the *highest row reached* (bottom=0, top=15). Pure RTL, ModelSim-friendly, Quartus-ready.

---

## Features
- **Sprite**: one-hot 16×16 position; up/down/left/right pulses; bounds-checked; no wraparound.
- **SPIKE**: per-row traffic with 0–4 cars, lengths 1/2/3; random direction & speed via 16-bit LFSR.
- **Freeze on hit**: outputs come from registered state → true freeze when `hit=1`.
- **Scoring**: tracks **max row** reached; no increment on a hit frame.
- **HUD**:
  - HEX1..HEX0 → score (hex)
  - HEX5..HEX2 → status: `LOL` (safe) or `DEATH` (on hit)
- **Beeper**: simple 3-note cycle; tone always plays, notes advance only when **not** hit.
- Works on hardware using the provided `LEDDriver.sv` (no changes needed).

---

## Hardware / Tools
- **Board**: DE1-SoC (Cyclone V)
- **Display**: 16×16×2 LED matrix via `GPIO_1`
- **Clock**: 50 MHz base + `clock_divider` (32-bit ripple)
- **Controls**:
  - `SW[9]` = Reset (active-HIGH)
  - `KEY[...]` = movement pulses (map to up/down/left/right)
  - One `GPIO_0[x]` routed to buzzer
- **Software**: Quartus Prime Lite 17.0, ModelSim Intel 10.5b

---

## Repo Structure

/src
LEDDriver.sv        # given 16x16 display driver (unchanged)
clock_divider.sv    # 32-bit ripple divider
Sprite.sv           # player grid (one-hot row/col → 2D grid)
SPIKE.sv            # traffic lanes + LFSR + speeds + freeze-on-hit
Playtime.sv         # overlap detect + score (max row) + hit
Beeper.sv           # 3-note FSM; advances only when not hit
DisplayText.sv      # 7-seg text driver (LOL / DEATH) + hex font
DE1_SoC.sv          # top: wiring, clocks, LEDDriver, HEX, buzzer
/test
*_testbench.sv      # unit testbenches (Sprite, SPIKE, Playtime, Beeper)
/sim
*.do                # ModelSim scripts (e.g., SPIKE_lab.do, playtime_lab.do)

---

## How It Works (short)
- **Sprite**  
  Two 4-bit enums (row_state/col_state). Case-maps to one-hot masks; `sprite_grid[r] = col_mask` on the active row bit. No wraparound.
- **SPIKE**  
  Each row keeps a 16-bit lane mask, direction, speed bucket (move every 1/2/4/8 ticks), and a countdown. On reset, lanes are randomized with a 16-bit LFSR; 0–4 cars of length 1–3 placed per row. Rows **0, 1, 15** are reserved empty. When `hit=1`, registered state is held (display freezes).
- **Playtime**  
  Per row: `(sprite_grid[r] & spike_grid[r]) != 0` ⇒ **hit**. Score increments only when sprite moves **up** (to a larger row index) **and** there’s no hit that cycle. `prev_row` is seeded on reset from the current sprite row to avoid a false +1.
- **Beeper**  
  3-state FSM (A→B→C→A). **Note clock** is fast (divider tap), **state clock** is slower (another divider tap). Output tone keeps playing; FSM advances notes only when not hit.

---

## Simulation (ModelSim)
Quick start (adjust file paths as needed):
```tcl
vlib work
vlog ./src/clock_divider.sv \
     ./src/Sprite.sv ./src/SPIKE.sv ./src/Playtime.sv ./src/Beeper.sv
vlog ./test/Sprite_testbench.sv
vsim -voptargs="+acc" Sprite_testbench
run -all

