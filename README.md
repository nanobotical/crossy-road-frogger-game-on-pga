# crossy-road-frogger-game-on-DE1-SOC
16×16 LED-matrix “Crossy Road” style game for the DE1-SoC board. Sprite (player) moves on a grid while SPIKE (traffic) scrolls across lanes. Overlap = hit/freeze. Score is the highest row reached. Designed in pure RTL, ModelSim-friendly, Quartus-ready.
Features
	•	Sprite: one-hot 16×16 position, up/down/left/right pulses, bounds-checked.
	•	SPIKE: per-row traffic with 0–4 cars, lengths 1/2/3, random direction & speed via 16-bit LFSR.
	•	Freeze on hit: display/state truly freeze (outputs come from registered state).
	•	Scoring: tracks max row reached (bottom=0, top=15), no increment on a hit frame.
	•	HUD:
	•	HEX1..HEX0 → score (hex)
	•	HEX5..HEX2 → status: LOL (safe) or DEATH (on hit)
	•	Beeper: simple 3-note cycle; keeps tone playing, advances notes only when not hit.
	•	Works on hardware using the provided LEDDriver.sv (no changes needed).

Hardware / Tools
	•	Board: DE1-SoC (Cyclone V)
	•	Display: 16×16×2 LED matrix via GPIO_1
	•	Clock: 50 MHz base + my clock_divider (32-bit ripple)
	•	SW/KEY:
	•	SW[9] = reset (active-HIGH)
	•	KEY[0]..(your mapping) = movement pulses
	•	One GPIO pin routed to buzzer (see pin plan)
	•	Software: Quartus Prime Lite 17.0, ModelSim Intel 10.5b
