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
- Reset to Default button in the Beam setup header: restore the initial
  20 ft simply supported steel W12x26 example, with no point loads, one
  valid 1 kip/ft uniform load from 0 to 20 ft, and L/360. Synchronize the
  inputs, clear validation feedback and diagram inspection, and refresh the
  beam drawing, results, checks, and recommendation.
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
7. After changing the support, span, material/section, deflection limit, and
   loads—including entering an invalid uniform-load interval—use Reset to
   Default. Confirm that the controls return to the initial 20 ft simply
   supported steel W12x26 setup, with no point loads, one valid 1 kip/ft
   uniform load over 0–20 ft, and L/360; validation feedback and diagram
   inspection clear, and results match case 1. Confirm reset works again
   after making further changes.
- Compare validation values within 0.01 in their stated units and extrema
  locations within 0.01 ft.

## Build stages

### 1. Creating the complete visual mockup

- **Goal:** Lay out the full page: input panel; beam, supports and example
  load drawing; shear, moment and deflection chart placeholders; bending and
  deflection cards; recommendation area. Apply the specified clean visual
  style and responsive panel stacking. Use static values for the default
  20 ft simply supported W12x26 example. Do not implement calculations.
- **Expect to see:** The complete page structure on desktop and a stacked
  layout on a narrow screen. Static example results are clearly mockup
  content, not responsive to input changes.
- **Test:** Open the root `index.html` in a browser at desktop and phone
  widths. Confirm every specified page region is present, readable, and
  arranged as described; confirm no calculation behavior is implied.

### 2. Adding inputs and beam/load interactions

- **Goal:** Add synchronized span slider/number input, support and
  material/section selection, deflection-limit selection, and editable
  point/uniform load lists. Implement add/remove, input validation,
  point-load dragging with 0.25 ft snapping, and the specified load-bound
  and clipping rules. Keep analysis outputs explicitly pending/mock until
  calculation stages are complete.
- **Expect to see:** Input changes immediately update the selected controls
  and beam drawing. Invalid uniform-load bounds show clear feedback; valid
  uniform loads are drawn only over their clipped span.
- **Test:** Check span limits and synchronization; add, edit and remove each
  load type; verify point-load positions stay within the beam and snap as
  specified; check uniform-load clipping, rejected reversed/non-overlapping
  bounds, support defaults, and material/section choices.

### 3. Implementing reactions, shear, and moment

- **Goal:** Calculate reactions by static equilibrium and signed V(x) and
  M(x) using the specified conventions. Evaluate load discontinuities and
  piecewise extrema accurately rather than selecting maxima from chart
  samples. Render the reaction arrows and shear/moment diagrams; retain
  clear placeholders for deflection and checks.
- **Expect to see:** Changing a valid load updates reactions and signed
  shear/moment diagrams and their labeled extrema, while deflection remains
  pending.
- **Test:** Compare reactions, shear, and moment against spec validation
  cases 1, 2, 4, and clipped-load case 5, including the stated tolerances.
  Check point-load jumps, uniform-load slopes, and signed diagram values.

### 4. Implementing deflection, checks, and section recommendation

- **Goal:** Integrate the specified beam equation with the support
  boundary conditions; locate maximum absolute deflection accurately.
  Calculate bending stress and deflection pass/fail ratios, color bands,
  and the best passing section in the selected material, including steel
  tie-breaking, the smallest-listed lumber rule, and the no-passing state.
- **Expect to see:** Both diagrams and check cards update when inputs or
  the selected section change. The recommendation button appears only when
  a listed section passes.
- **Test:** Compare stresses, deflections, limits, and check statuses
  against validation cases 1–4. Check the steel tie rule, lumber selection,
  and a load case where no section passes.

### 5. Integrating interactions and validating the finished page

- **Goal:** Connect the diagram pointer interaction across all three
  diagrams, formula explanations on the check cards, the recommendation
  button, and the responsive layout. Remove mock values and ensure the
  example loads on startup. Keep saving designs and all other out-of-scope
  items unimplemented.
- **Expect to see:** The complete app updates consistently from the initial
  example through load editing, section changes, checks, and recommendation;
  diagram inspection shows V, M, and y at a shared x position.
- **Test:** Run all six validation items in `spec.md` in the browser,
  including invalid-load feedback and no-passing-section behavior. Recheck
  desktop and phone layouts, the initial example, hover explanations, and
  that values and labels remain within the specified tolerances.

## Out of scope
- Self-weight, shear stress, lateral-torsional buckling, load
  combinations, code-specific design factors, and continuous beams. Results
  are preliminary screening only, not code-compliant final design or
  construction approval.
- Saving designs.
