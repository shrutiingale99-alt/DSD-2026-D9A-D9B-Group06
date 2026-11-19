# 02 – FSM and Logisim

## FSM type and encoding
- **Model:** Moore (outputs depend on the present state only)
- **States:** 5. **Encoding:** 3-bit binary, using three D flip-flops S2 S1 S0
- **Reset:** synchronous, active-high, to IDLE

| State | S2 S1 S0 | Outputs (Moore) |
|---|---|---|
| IDLE | 000 | All OFF |
| CALM | 001 | CALM LED (green) |
| BREEZY | 010 | BREEZY LED (yellow) |
| WARNING | 011 | WARNING LED (orange) – **new in H1** |
| STORM | 100 | STORM LED (red) + BUZZER |

State diagram: `../01_Problem_Analysis/fsm_state_diagram_H1.png`

## State transition table
R = registered wind value. "x" = don't care.

| Present state | RESET | START | ACK | Wind (R) | Next state |
|---|---|---|---|---|---|
| any | 1 | x | x | x | IDLE |
| IDLE | 0 | 0 | x | x | IDLE |
| IDLE | 0 | 1 | x | 0–1 | CALM |
| IDLE | 0 | 1 | x | 2–4 | BREEZY |
| IDLE | 0 | 1 | x | 5 | WARNING |
| IDLE | 0 | 1 | x | 6–7 | STORM |
| CALM / BREEZY / WARNING / STORM | 0 | x | 0 | x | same state (hold) |
| CALM / BREEZY / WARNING / STORM | 0 | x | 1 | x | IDLE |

## Derived logic (implemented gate-by-gate in Logisim)
```
Range decoder:  CALM    = R2'·R1'
                BREEZY  = R2'·R1 + R2·R1'·R0'
                WARNING = R2·R1'·R0
                STORM   = R2·R1
Control:        IDLE = S2'·S1'·S0'     GO = IDLE·START     HOLD = IDLE'·ACK'
Next state:     D_S2 = RESET'·( GO·STORM            + HOLD·S2 )
                D_S1 = RESET'·( GO·(BREEZY+WARNING) + HOLD·S1 )
                D_S0 = RESET'·( GO·(CALM+WARNING)   + HOLD·S0 )
Input register: D_Ri = Wi·RESET',  enable = IDLE + RESET
Outputs:        CALM_LED = S2'·S1'·S0    BREEZY_LED = S2'·S1·S0'
                WARNING_LED = S2'·S1·S0  STORM_LED = BUZZER = S2
```

## Logisim implementation
- **Circuit file:** `weather_alert_H1.circ` (Logisim 2.7.1 format; it also opens in Logisim-evolution)
- **Sections on the canvas:** (1) user inputs, (2) 3-bit input register, (3) range decoder, (4) FSM next-state logic and state register, (5) Moore output logic with LEDs and output pins
- Signals are connected with labelled **tunnels** (e.g. `R2`, `nS1`, `GO`), so each block can be read separately.

### How to simulate by hand
1. Open the `.circ` file and select the Poke tool (hand icon).
2. Set RESET = 1 and tick the clock once (Ctrl+T, or enable ticks with Ctrl+K), then set RESET = 0.
3. Set W2 W1 W0 (e.g. 1 0 1 = 5) and let the clock tick at least once.
4. Set START = 1 for one tick, then back to 0. The WARNING LED (orange) turns on and stays on.
5. Set ACK = 1 for one tick to return to IDLE (all LEDs off).

## Simulation evidence
- `logisim_simulation_log.txt`: automated run of this exact `.circ` file on the Logisim 2.7 simulation engine. It covers all 8 wind values, all 5 states, RESET, START, ACK, hold behaviour and 600 random cycles. **Result: 647 checks, 647 passed, 0 failed.**
- Screenshots rendered from the simulated circuit:
  - `logisim_idle_wind4.png`: IDLE (wind = 4 loaded, no START)
  - `logisim_calm_wind1.png`: CALM
  - `logisim_breezy_wind3.png`: BREEZY
  - `logisim_warning_wind5.png`: WARNING (new H1 level)
  - `logisim_storm_wind6.png`: STORM (red LED and buzzer)
- `verification_scripts/`: the circuit generator and the test harness used for the automated run

> Tip: take one or two of your own screenshots in Logisim while presenting, to show the circuit running live.
