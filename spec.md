# Beam Designer

## Purpose
A preliminary beam-analysis tool for engineers and students. The user sets
up a beam, places loads on it, and picks a steel or lumber section. The app
shows the shear, moment, and deflection diagrams and checks the selected
section for bending stress and deflection, updating instantly as anything
changes.

## Users
A civil engineering student checking a homework beam, or an engineer doing
preliminary screening of a floor beam or deck joist. The app is not for
final design or construction approval.

## Inputs
- Support type: Simply supported or Cantilever (fixed at the left end).
- Span: 4 to 40 ft, set with a slider and a number box that stay in sync.
- Loads, listed in a panel where each can be edited or removed:
  - Loads are downward only; magnitudes must be nonnegative.
  - Point loads: magnitude (kips) and position (ft). The position must be
    within the beam, from 0 to the span inclusive. A new point load starts
    at midspan (at the free end of a cantilever).
  - Uniform loads: magnitude (kips/ft) and start and end positions (ft).
    The start must be less than the end, and the interval must overlap the
    beam. Portions outside the beam are clipped to the beam ends. Invalid
    bounds are reported and are not used in calculations.
  - "Add point load" and "Add uniform load" buttons.
  - Point loads can also be dragged along the beam in the drawing,
    snapping to 0.25 ft and staying within the beam.
- Material and section, from these tables:
  - The user selects a material and then a section from that material's
    table. The selected section is analyzed; the section recommendation
    searches the selected material only.

  Steel W-shapes (E = 29,000 ksi, allowable bending stress Fb = 30 ksi):

  | Section | Ix (in^4) | Sx (in^3) | Weight (lb/ft) |
  |---------|-----------|-----------|----------------|
  | W8x10   | 30.8      | 7.81      | 10 |
  | W10x12  | 53.8      | 10.9      | 12 |
  | W12x26  | 204       | 33.4      | 26 |
  | W14x22  | 199       | 29.0      | 22 |
  | W16x26  | 301       | 38.4      | 26 |
  | W18x35  | 510       | 57.6      | 35 |
  | W21x44  | 843       | 81.6      | 44 |
  | W24x55  | 1350      | 114       | 55 |

  Douglas Fir-Larch No. 2 lumber (E = 1,600 ksi, Fb = 0.900 ksi):

  | Section | Ix (in^4) | Sx (in^3) |
  |---------|-----------|-----------|
  | 2x8     | 47.63     | 13.14     |
  | 2x10    | 98.93     | 21.39     |
  | 2x12    | 177.98    | 31.64     |

- Deflection limit: L/240 or L/360 (default L/360).
- Self-weight is ignored.

## Calculations
- Reactions from static equilibrium. Downward loads and deflection are
  positive; upward reactions are positive. Internal shear is positive
  upward on the left face of a cut, and positive moment is sagging.
- Use linear-elastic, small-deflection beam theory with the selected
  section and constant E and I over the span.
- Shear V(x) and moment M(x) along the beam, at 1 ft spacing or finer,
  plus exactly at every load point and load start/end. Compute extrema
  piecewise, including at load boundaries and internal stationary points;
  do not use the largest value from the plotting samples as the design
  maximum.
- Deflection y(x), positive downward, by integrating the beam equation
  y'' = -M/EI with the support boundary conditions (y = 0 at both supports;
  or y = 0 and slope = 0 at the fixed end). Include support and load
  boundaries and internal points where the slope is zero when finding
  maximum deflection.
- Bending stress fb = max(|M(x)|) * 12 / Sx (ksi), with M in kip-ft.
  Bending check passes if fb <= Fb.
- Deflection check passes if max(|y(x)|) <= L/240 or L/360, with L in
  inches.
- Lightest passing section: among sections in the selected material, use
  the lightest passing steel section (if two weigh the same, use the one
  with larger Ix). For lumber, use the smallest listed size that passes;
  the lumber table does not provide weights.
- Convert loads and lengths to consistent inch-based units for stress and
  deflection calculations (kip/ft to kip/in and ft to in as applicable).
  Show kips, ft, and kip-ft on screen; show deflection in inches.

## Outputs
- A beam drawing with the supports, load arrows (point loads as single
  arrows, uniform loads as a row of small arrows), and the reactions.
- Three diagrams stacked under the beam and lined up with it on the same
  x scale: shear (kips), moment (kip-ft), and deflected shape (in). Each
  diagram is filled, shows its signed values around a zero reference, and
  labels the largest absolute value with its sign and location.
- Moving the mouse over any diagram shows a vertical line across all three
  with the values of V, M, and y at that x.
- Two check cards: Bending and Deflection. Each shows the maximum absolute
  actual value, the allowable value, a bar for actual divided by allowable
  (green under 70%, amber 70-100%, red over 100%), and PASS or FAIL. A check
  passes when actual is less than or equal to allowable.
- A line suggesting the lightest passing steel section or smallest passing
  lumber size in the selected material, with a button to use it. If no
  listed section passes, say so and do not show a use-it button.
- Everything updates immediately whenever any input changes.

## Look and feel
- Opens with an example already loaded (20 ft simply supported W12x26
  with a 1 kip/ft uniform load over the full span), so it is useful the
  moment it opens.
- Clean and professional, like a commercial engineering web app: light
  background, white cards with soft shadows, one accent color, a modern
  system font, generous spacing. No clutter.
- Inputs in a panel on the left; drawing, diagrams, and checks on the
  right. On a phone, the panels stack.
- Every input has a short label with units. Hovering over a check card
  explains the formula used.

## Validation
1. Simply supported, 20 ft, W12x26, uniform 1 kip/ft over the full span:
   reactions 10 k and 10 k; max shear 10 k; max moment 50.0 kip-ft at
   10 ft; fb = 17.96 ksi; max deflection 0.609 in at 10 ft; L/360 =
   0.667 in; both checks pass.
2. Simply supported, 20 ft, W12x26, point load 10 k at 5 ft: reactions
   7.5 k (left) and 2.5 k (right); max moment 37.5 kip-ft at 5 ft;
   fb = 13.47 ksi; max deflection 0.340 in at 8.82 ft.
3. Cantilever, 10 ft, W12x26, point load 5 k at the free end: reaction
   5 k and fixed-end moment 50 kip-ft; fb = 17.96 ksi, so bending passes;
   max deflection 0.487 in at the free end (PL^3/3EI); L/360 = 0.333 in,
   so deflection fails.
4. Simply supported 12 ft 2x10 joist, uniform 0.05 kip/ft: max moment
   0.90 kip-ft; fb = 0.505 ksi; max deflection 0.147 in; both pass.
5. Simply supported, 10 ft, W12x26, uniform 1 kip/ft from -2 ft to 12 ft:
   the load is clipped to the 0-to-10 ft span and gives the same results as
   a 1 kip/ft full-span uniform load: reactions 5 k and 5 k, and max moment
   12.5 kip-ft at 5 ft.
6. With a valid input for which every section in the selected material
   fails at least one check, show that no listed section passes and omit
   the use-it button. Reversed or non-overlapping uniform-load bounds must
   be rejected with a clear validation message rather than silently used.
- Compare validation values within 0.01 in their stated units and extrema
  locations within 0.01 ft.

## Out of scope
- Self-weight, shear stress, lateral-torsional buckling, load
  combinations, code-specific design factors, and continuous beams. Results
  are preliminary screening only, not code-compliant final design or
  construction approval.
- Saving designs.
