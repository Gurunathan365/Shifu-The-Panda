# Shifu the Panda

A projectile puzzle game about **inverse functions**. The player never gets to fire a clean shot — every launcher module distorts the numbers it is handed. The job is to measure the distortion from a handful of test shots, work out its inverse, and pre-compensate the input so the distorted output lands on target.

Named for Shifu adapting his teaching to Po: you don't fix the student, you change what you feed them.

## Premise

Weapons engineers outsourced the hardcoded modules that set projectile speed and launch angle. A batch came back defective, each with a different drift induced in its transfer function. Scrapping them isn't an option, so the engineers write a software workaround: compare each module's output against the expected result for known inputs, identify the error, and invert it.

## How a round works

1. **Solve the physics.** Pick a speed or an angle, fix one, and solve the 2-D motion equations for the other so the trajectory passes through the target. A volunteer checks this working before the player is allowed to touch the editor.
2. **Fire the naive shot.** Enter the value pair you just computed. It will miss — that is the point.
3. **Characterise the drift.** Spend test inputs probing the module. Each stage announces its *category* of drift (zero, span, zonal, dead zone), so you are solving for the constants, not the shape.
4. **Write the inverse.** Set `inverseu` / `inversecostheta` (or `inverseux` / `inverseuy`) so that the module's output equals the values you actually want.

Launcher sits at the origin. The target is a single point at `(D, h_t)`.

```
x = u·cos(θ)·t
y = u·sin(θ)·t − ½gt²

time of flight  t = 2u·sin(θ)/g
apex            H = u²sin²(θ)/(2g)
range           R = u²sin(2θ)/g
```

## Drift types

| Stage | Category | Module computes | Levels |
|---|---|---|---|
| 1 | Zero drift | `taken = in + c` | 1–4 |
| 2 | Span drift | `taken = m · in` | 5–8 |
| 3 | Zonal drift | `taken = m · in + c` | 9–12 |
| 4 | Dead zone | `taken = k` for `in` inside a band, pass-through elsewhere | 13–14 |
| 5 | Component inputs | any of the above, applied to `ux` / `uy` | 15–18 |

Stages 1–4 take `u` and `costheta`. Stage 5 switches the input space to `ux` and `uy`, which changes the algebra even when the drift is familiar.

Stage 1 tells the player which of the two variables is drifting. From stage 2 on, finding that out is part of the work.

## Levels

`D` is the target's horizontal distance, `h_t` its height. Only one variable drifts per level.

**Stage 1 — zero drift**

| # | D | h_t | Module | Player writes |
|---|---|---|---|---|
| 1 | 111 | 0 | `u_taken = u_in − 7` | `inverseu = u + 7` |
| 2 | 194 | 30 | `u_taken = u_in + 23` | `inverseu = u − 23` |
| 3 | 116 | 0 | `cos_taken = cos_in − 0.1` | `inversecostheta = costheta + 0.1` |
| 4 | 142 | 40 | `cos_taken = cos_in + 0.05` | `inversecostheta = costheta − 0.05` |

**Stage 2 — span drift** (constraint: `cos(θ) < 1` after the drift is applied)

| # | D | h_t | Module | Player writes |
|---|---|---|---|---|
| 5 | 136 | 0 | `u_taken = u_in × 1.25` | `inverseu = u × 0.8` |
| 6 | 154 | 0 | `cos_taken = cos_in × 0.5` | `inversecostheta = costheta × 2` |
| 7 | 152 | 60 | `u_taken = u_in × 0.8` | `inverseu = u × 1.25` |
| 8 | 179 | 25 | `cos_taken = cos_in × 0.75` | `inversecostheta = costheta × 4/3` |

**Stage 3 — zonal drift.** These levels pin the trajectory down completely: range *and* apex are both given, which fixes the angle and the speed.

| # | Range | Apex | cos θ | Answer u | Module | Player writes |
|---|---|---|---|---|---|---|
| 9 | 150 | 45 | 0.64 | ≈39.05 | `2u + 3` | `inverseu = (u − 3)/2` |
| 10 | 150 | 45 | 0.64 | ≈39.05 | `3u − 2` | `inverseu = (u + 2)/3` |
| 11 | 180 | 65 | 0.57 | ≈43.85 | `1.5u + 3` | `inverseu = (u − 3)/1.5` |
| 12 | 180 | 65 | 0.57 | ≈43.85 | `3u − 1.5` | `inverseu = (u + 1.5)/3` |

**Stage 4 — dead zone, with obstacles.** A trap stage: the dead zone looks threatening but the correct answer sits outside it, so no correction code is needed. After twelve levels of inverting things, that is the joke.

| # | Range | cos θ | Wall | Dead zone | Answer |
|---|---|---|---|---|---|
| 13 | 117.6 | 0.6 | x = 58.8, height 39 | `30 < u < 40 → 30` | `u = 35` |
| 14 | 185.856 | 0.8 | x = 92.928, height 34.5 | `40 < u < 45 → 50` | `u = 44` |

Both walls sit at exactly half the range, so the shot has to clear them at its apex — 39.2 m over a 39 m wall, and 34.85 m over a 34.5 m wall. The margins are deliberately thin.

**Stage 5 — component inputs**

| # | D | h_t | Module | Player writes |
|---|---|---|---|---|
| 15 | 137 | 15 | `ux_taken = ux_in − 14` | `inverseux = ux + 14` |
| 16 | 182 | 55 | `uy_taken = uy_in + 8` | `inverseuy = uy − 8` |
| 17 | 169 | 65 | `ux_taken = ux_in × 1.75` | `inverseux = ux × 4/7` |
| 18 | 107 | 35 | `uy_taken = uy_in × 0.9` | `inverseuy = uy × 10/9` |

## Scoring

| Rule | Effect |
|---|---|
| Test input budget | 7 per level in stages 1–2, 10 in stages 3–5 |
| Buying more | 3 extra test inputs cost 10% of the level's points |
| Stage weighting | 1 : 1 : 2 : 2 : 2 across stages 1–5 |
| Passing without showing the inverse to a volunteer | 60% of the level's points |
| Skipping after finding the correct speed and angle | 20% of the level's points |
| Ties | broken on per-stage time |

## Volunteer briefing

Volunteers gate progress and supply theory; they never hand out answers.

- Know the player instructions cold.
- Check the player's hand-solved speed and angle before unlocking the editor. You are confirming they did the work, not re-deriving it.
- Supply the 2-D motion formulae on request.
- Explain drift theory in the abstract when asked: `u' = u + c`, `u' = mu`, `u' = mu + c`, and dead-zone behaviour. Announce the category in play for the current stage. For stage 1 only, also say whether `u` or `costheta` is the one drifting.
- Track test inputs and stop players exceeding their budget.
- Make sure the inverse function is actually written down. Passing by trial and error carries its own penalty, but catch it anyway.
- Past stage 2, be generous with extra test inputs if a player is genuinely stuck. Generous with attempts, never with answers.

## Running the game

The build is one self-contained HTML file with no dependencies or build step. Open it in a browser, or serve the folder:

```
python3 -m http.server 8000
```

## Implementation notes

- **Player code runs in a `Function` sandbox.** Both `u = 50;` and `let u = 50;` work: the first compilation pass pre-declares the four names so bare assignments stay local, and a `SyntaxError` from a redeclaration falls through to a second pass that runs the code as written.
- **The drift lives in the level table**, as a function `t(a, b) → [a', b']` applied to the *inverse* values. The player's stated `u` and `costheta` are only validated, never fired. This is what makes the inverse the actual deliverable.
- **Hit test is closest approach** to the target point, inside a 0.5 m radius. Testing range and height independently is wrong for elevated targets, where landing at `D` and arriving at height `h_t` are mutually exclusive.
- **Collision stepping is bounded in x**, not in time, so a fast shot can't step over a 1 m wall between samples.
- **Both the raw and the drifted `cos θ` are checked** against `0 < cos θ < 1`. Only checking the raw value lets a transform push it past 1 and turn the whole trajectory into `NaN`.

## Known issues

Items 1–3 mean the documented answers for levels 9–14 do not currently win those levels.

1. **Gravity mismatch.** The level design assumes `g = 10 m/s²`; the build uses `9.8`. Every documented answer lands exactly on target at 10 and misses by 2–3 m at 9.8 — `u = 35` gives a range of 117.6 m at `g = 10` and 120.0 m at 9.8, against a 117.6 m target. Set `G = 10` in the build, or rescale every `D` in the table. The first is far less work.
2. **Levels 9–12 encode the apex as the target height.** The design gives range 150 m *and* apex 45 m, with the target on the ground at the end of the range. The build places the target at `(150, 45)`. The levels are still solvable as built, but they are a different puzzle with a different answer, and the documented `cos θ = 0.64` / `u ≈ 39.05` no longer applies. Fix: set `h: 0` and present the apex as a constraint in the level brief.
3. **Level 14's dead-zone escape is off.** The answer `u = 44` is meant to pass through untouched, as `u = 35` does on level 13. The code maps 44 into the dead zone (→ 50) and then adds 4, giving 54. It should return 44. Level 13 works only because its escape happens to arrive back at the right number.
4. **Doc typo, level 2.** The expected implementation is listed as `inverseu = u + 7`, carried over from level 1. For `u_taken = u_in + 23` it should be `inverseu = u − 23`.
5. **Doc typo, trajectory equation.** The boxed equation in the stage 1 sheet reads `y = x·tanθ + gx²/(2u²cos²θ)`. The second term is subtracted.
6. **Not implemented in the build.** Test-input budgets, the points system, per-stage timing, and the volunteer verification gate are all run manually. The build tracks which levels have been cleared and nothing else.

## Adding a level

Append to the `levels` array:

```js
{
  d: 150,        // target distance, m
  h: 0,          // target height, m
  tol: 0.5,      // hit radius, m
  ox: 75,        // optional: wall at this x, 1 m thick
  oy: 40,        // optional: wall height, m
  t: (a, b) => [a * 2 + 3, b]   // the module's drift
}
```

`t` receives the player's inverse values and returns what the launcher actually fires. For levels past index 13 the pair is `(ux, uy)` instead of `(u, cos θ)`; the boundary is the `ADV_FROM` constant. The level selector, brief text and obstacle rendering all read from these fields, so nothing else needs touching.
