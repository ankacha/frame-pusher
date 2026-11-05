# Frame Pusher

**Push a structure and watch it respond.** Frame Pusher is a live 2D frame analyser that runs entirely in your browser. You draw a beam, frame, truss, arch or cable structure, support it, then push and pull it with loads. The deflected shape, bending moment, shear, axial force and reactions update as you drag.

It was built for two jobs:

- **Checking behaviour.** Structural engineers can confirm how a structure works, such as where moment goes, which way a frame sways, or what a settled support does, before or alongside a formal analysis.
- **Teaching the finite element method.** Every step of the direct stiffness method can be inspected, from element stiffness matrices to the assembled global matrix, the solution and the checks on it.

Frame Pusher is analysis software only. It does no design-code checks and has no load cases or load combinations.

**Current version: v2.11 "Ghost"**

---

## Using it

Open `frame-pusher.html` in any modern browser (Chrome, Edge, Firefox or Safari). Nothing to install, no server, no internet connection needed: the whole app is one self-contained HTML file with no external dependencies.

A welcome guide opens on start-up and can be reopened at any time from **? Guide** in the top bar. You can also start from the **Gallery**, which holds the worked examples and head-to-head comparisons.

### Quick start

1. **Draw:** use the Member tool (`B`) and click points in turn. Use Curved (`A`) for arcs and Cable (`C`) for tension-only members.
2. **Support:** choose Pin, Roller, Fixed or Spring (`1`–`6`) and click a node.
3. **Load:** use Point load (`P`) and drag from the structure, Moment (`O`), UDL (`U`) or Var. UDL (`W`).
4. **Read the response:** the diagrams update live. Select a member to interrogate it along its length.
5. **Check:** open **Verification tests** in the top bar.

`Esc` returns to the Select tool, `F` fits the view and `Ctrl+Z` / `Ctrl+Y` undo and redo. **Shortcuts** in the top bar lists every key.

---

## Features

### Modelling
- Straight, curved (circular arc) and tension-only cable members
- Pinned, roller, fixed and spring supports (vertical, horizontal and rotational), with a menu of typical ground stiffnesses
- Imposed support settlement
- Rigid and hinged joints, plus moment releases at individual member ends
- Real sections: a British Standard library (UB, UC, SHS, RHS, CHS), a shape calculator or direct EI / EA entry, with materials
- Tapered and haunched (non-prismatic) members
- Point loads, moments, and uniform and varying distributed loads. Distributed loads can act normal to the member or in global X / Y, per true or projected length, with optional self-weight.
- Annotations: text, boxes, ellipses, lines, arrows and dimensions

### Results
- Deflected shape, bending moment, shear and axial force diagrams, and reaction arrows
- Peak values labelled on the canvas, plus a member inspector with values along the member
- Animated response: the load grows from zero to full and back
- Ghosted outlines of supports that move with the deflected shape (rollers, springs)
- Toggle for the initial (undeformed) structure, so you can view the results on their own
- Head-to-head examples that put several structures on one canvas at a common scale

### Inside the method
- **Stiffness matrix inspector:** a live heatmap of **K** with **u**, **F** and **R** alongside. Hovering a cell shows each member's local matrix, its rotation to global axes and the exact term that feeds that cell. You can also step through the assembly member by member.
- **Matrix diagnostics:** symmetry and rigid-body tests, pivots, condition-number estimates, backward error, work balance and global equilibrium, each with an explanation of how it is calculated
- **Verification suite:** 351 automated tests plus textbook-style closed-form worked examples (beams, continuous beams, portal frames, trusses, arches, cables, springs, settlement). Each worked example compares the hand solution with the analysis engine.

### Output
- Save and open models as `.json` files (or drag a file onto the app), with autosave in the browser
- A4 calculation-sheet reports as vector PDF, with a title block, sheet numbering and selectable contents

---

## Method in brief

- Linear-elastic 2D analysis by the direct stiffness method: Euler–Bernoulli beam elements with 3 DOF per node and small displacements
- Members without a section are a generic reference member with EI = 10 000 kNm² and EA = 1 000 000 kN, so deflections are real values
- Equations are solved by symmetric Gaussian elimination (K = L·D·Lᵀ). A pivot below 1e-11 of its original diagonal is reported as a mechanism.
- Cables are solved by an active-set iteration that removes members in compression
- Springs add stiffness to the diagonal of K. Settlements are prescribed displacements moved to the right-hand side.
- Units are kN, kN/m, kNm and m

The full assumptions are listed under **Method** in the app.

> Results are for exploration and teaching. They are not a substitute for a checked design calculation.

---

## Building from source

The app is developed as separate modules that a small build script combines into the single distributable HTML file.

```
src/
  page.html      page shell with /*@include path*/ markers in one <style> and one <script>
  css/           13 style sheets, in cascade order
  js/            23 script modules, run in the order listed in page.html (shared global scope)
  build.mjs      replaces each marker with the file's contents, verbatim
```

Build with Node.js (no packages needed):

```
node src/build.mjs frame-pusher.html
```

The build fails if the version number is inconsistent. The version is set in three places that must agree: `VER` at the end of `js/09-help.js`, the `<title>` in `page.html`, and the top-bar label in `page.html`. The changelog lives in `js/10-changelog.js` and is shown in the app under **About**.

### Workflow for a change

1. Edit a module (or add a new one and its include marker in `page.html`).
2. Bump the version in the three places and add a changelog entry.
3. Rebuild, open the result and run **Verification tests**. All tests should pass.
4. Commit both the changed sources and the rebuilt HTML.

Later modules often extend earlier ones by wrapping their functions (for example `draw`, `runTests`, `buildPop`). Keep the include order in `page.html`, because it is the order the code runs in.

---

## Version history

The app has grown from v1.0 "First Push" through matrix inspection (v1.20), springs and settlement (v2.0), closed-form verification (v2.1), PDF reports (v2.3), the modular build (v2.4), example gallery (v2.7), model files (v2.8), tapered members (v2.9), member end releases (v2.10) and moving-support ghosts (v2.11). The full changelog is in the app under **About**.

---

## Author

Ashley Kacha · [ankacha.github.io](https://ankacha.github.io/)

Developed with Claude (Anthropic) over an iterative design process.
