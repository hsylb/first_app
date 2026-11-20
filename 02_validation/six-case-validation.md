# Six-case calculation validation

## Method

Ran the actual `calculateBeamResponse()` and `calculateStage4Results()`
functions from `index.html` in the browser with each validation case loaded
directly into the app state. The temporary test-only function hook was removed
after the run; the app starts with its normal example. Reactions, piecewise
extrema, stress, deflection, check statuses, and the recommendation were
captured before display rounding.

Differences below are **computed − expected**. Numeric values pass when the
absolute difference is at most 0.01 in the stated units; extrema locations
pass when within 0.01 ft. “PASS” in the result column refers to this comparison,
unless it explicitly says it is the engineering check status.

## Results

| Case / metric | Expected | Computed | Difference | Result |
|---|---:|---:|---:|---|
| 1 — left reaction (kips) | 10 | 10 | 0 | PASS |
| 1 — right reaction (kips) | 10 | 10 | 0 | PASS |
| 1 — max absolute shear (kips) | 10 | 10 | 0 | PASS |
| 1 — max moment (kip-ft) | 50.0 at 10 ft | 50 at 10 ft | 0; 0 ft | PASS |
| 1 — bending stress (ksi) | 17.96 | 17.9640718563 | +0.0040718563 | PASS |
| 1 — max deflection (in) | 0.609 at 10 ft | 0.6085192698 at 10 ft | −0.0004807302; 0 ft | PASS |
| 1 — L/360 limit (in) | 0.667 | 0.6666666667 | −0.0003333333 | PASS |
| 1 — engineering checks | Both pass | Both pass | — | PASS |
| 2 — left/right reactions (kips) | 7.5 / 2.5 | 7.5 / 2.5 | 0 / 0 | PASS |
| 2 — max moment (kip-ft) | 37.5 at 5 ft | 37.5 at 5 ft | 0; 0 ft | PASS |
| 2 — bending stress (ksi) | 13.47 | 13.4730538922 | +0.0030538922 | PASS |
| 2 — max deflection (in) | 0.340 at 8.82 ft | 0.3401726132 at 8.8196601125 ft | +0.0001726132; −0.0003398875 ft | PASS |
| 3 — fixed reaction / moment | 5 kips / 50 kip-ft | 5 kips / 50 kip-ft | 0 / 0 | PASS |
| 3 — bending stress (ksi) | 17.96 | 17.9640718563 | +0.0040718563 | PASS |
| 3 — max deflection (in) | 0.487 at 10 ft | 0.4868154158 at 10 ft | −0.0001845842; 0 ft | PASS |
| 3 — L/360 limit (in) | 0.333 | 0.3333333333 | +0.0003333333 | PASS |
| 3 — engineering checks | Bending passes; deflection fails | Bending passes; deflection fails | — | PASS |
| 4 — max moment (kip-ft) | 0.90 | 0.9000000000 | +0.0000000000 | PASS |
| 4 — bending stress (ksi) | 0.505 | 0.5049088359 | −0.0000911641 | PASS |
| 4 — max deflection (in) | 0.147 | 0.1473769332 | +0.0003769332 | PASS |
| 4 — engineering checks | Both pass | Both pass | — | PASS |
| 5 — clipped uniform interval (ft) | [0, 10] | [0, 10] | — | PASS |
| 5 — left/right reactions (kips) | 5 / 5 | 5 / 5 | 0 / 0 | PASS |
| 5 — max moment (kip-ft) | 12.5 at 5 ft | 12.5 at 5 ft | 0; 0 ft | PASS |
| 6 — no-passing-section behavior | No section passes; omit button | No recommendation; button omitted | — | PASS |

For case 6, used a valid 40-ft simply supported beam with a 100 kip/ft
full-span uniform load and selected W12x26 steel. The calculated reactions
are 2,000 kips each and maximum moment is 20,000 kip-ft at 20 ft. Even the
selected W12x26 computes 7,185.63 ksi bending stress and 973.63 in maximum
deflection, so both checks fail. Even the largest listed steel section
(Sx = 114 in³) has a calculated bending stress of 2,105.26 ksi, well above
Fb = 30 ksi, so no listed section can pass.

The invalid-interval checks also passed: reversed bounds report “The end
position must be greater than the start position”; a non-overlapping interval
reports “The interval must overlap the beam (0 to 20 ft).” Both are marked
invalid and excluded from analysis.

## Independent equation check and disposition

Independently checked the numerical values using static equilibrium and
standard beam equations in inch-consistent units:

- Simply supported uniform load: `RA = RB = wL/2`,
  `Mmax = wL²/8`, `fb = 12Mmax/Sx`, and
  `δmax = 5wL⁴/(384EI)`.
- Simply supported point load at distance `a` on span `L`:
  `RA = P(L−a)/L`, `RB = Pa/L`, and `Mmax = RAa`. The standard piecewise
  deflection equations give the maximum at 8.8196601125 ft in case 2.
- Cantilever end load: fixed moment `PL` and free-end deflection
  `PL³/(3EI)`.
- Case 5’s `−2 to 12 ft` load clips to `0 to 10 ft`, making it the
  full-span uniform-load case.

All six specification cases agree with the independent calculations within
the specified 0.01 tolerance. No incorrect expected value or verified
calculation bug was found, so neither the calculations nor `spec.md` needed
correction. The case 4 values (0.505 ksi and 0.147 in) are rounded expected
values; the computed values are 0.5049088359 ksi and 0.1473769332 in.

## Reset to Default regression (spec case 7)

Started from a modified state: 13-ft cantilever, lumber 2x10, L/240, an
edited point load, an added reversed uniform-load interval, and an invalid
50-ft span entry. Reset restored the simply supported 20-ft steel W12x26,
L/360, no point loads, and one 1 kip/ft uniform load from 0 to 20 ft. Both
span controls synchronized at 20 ft; span/load validation feedback cleared;
the diagram crosshair hid and its readout returned to the hover prompt.
Reactions returned to 10 kips each, maximum moment to 50 kip-ft at 10 ft,
deflection to 0.61 in at 10 ft, both checks passed, and W14x22 was again
recommended. Repeating reset after another support change produced the same
defaults. The reset button fit without overlap or horizontal overflow at
1440, 390, and 320 px. The six calculation cases were also rerun through
the app functions after this feature; their values and statuses were
unchanged and remain within tolerance.
