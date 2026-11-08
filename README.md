# Frame Pusher – source modules

Frame Pusher is a live 2D frame analyser that runs in the browser: draw a structure, support it, push and pull it, and watch the deflected shape, bending moment, shear, axial force and reactions update as you go. It is for checking structural behaviour and for teaching the finite element (direct stiffness) method. It is analysis software only: there are no design-code checks, load cases or load combinations.

Current version: **v2.13 "Nodes"**.

## Layout (since v2.4)

The published page is a single self-contained HTML file. It is built from the source files below, which are kept in the "Frame Pusher" Project's files under `frame-pusher-src/`. They are not in the artifact, so the artifact downloads as one HTML file. Each version is published as its own artifact.

| Path | What it holds |
|---|---|
| `page.html` | page shell: head, markup, and one `<style>` and one `<script>` element holding `/*@include path*/` markers |
| `css/` | fifteen style sheets, in cascade order: base, v1.21 theme, v1.24 neon, v1.26 layout, v2.2 popups/flyouts, v2.5 welcome guide, v2.6 loads, v2.7 example gallery, v2.8 save / open files, v2.9 tapered members, v2.9.1 member inspector, v2.10 member end releases, v2.11 unstable warning and home button, v2.12 Labels control, v2.13 node inspector, inspector tab and theme button |
| `js/` | twenty-six script modules, in the run order listed in `page.html`: 01 setup … 13 report, 13b report decluttering (v2.12), 13a load direction / self-weight, 14 start-up, 15 welcome guide, 16 example gallery, 17 save / open files, 18 joining collinear members when a node is removed, 19 tapered and haunched members, 20 member inspector, 21 member end releases, 22 moving-support ghosts, 23 decluttering (Labels control, load picking, tests), 24 node inspector, inspector tab, theme button and surprising-behaviour examples. They share one global scope, so order matters. |
| `build.mjs` | replaces each marker with the file's contents, verbatim |

## Build

```
node src/build.mjs frame-pusher.html
```

## Workflow

Edit a module, rebuild, run the in-app tests (Verification tests in the top bar), then publish the built file as the artifact page (on its own) and save the changed module(s) back to the Project under `frame-pusher-src/`. The built file is what gets downloaded and shared; it needs nothing else. The artifact declares the `downloads` capability (saving model files and PDF reports); a new artifact needs that declaration too.

The version lives in three places, and `build.mjs` checks that they agree: `VER` at the end of `js/09-help.js`, the `<title>` and the top-bar label in `page.html`. The changelog is `js/10-changelog.js`; the About build date is in `js/11-undo-sections-solver.js`.

If the Project copies are ever out of step with the published page, the modules can be recovered from the published page itself: the build inserts each file verbatim, so splitting the page's `<style>` and `<script>` at each module's first line gives the modules back (v2.13 was rebuilt this way, byte for byte, from the v2.12 page).

## Notes by version

- **v2.4:** rebuilding the v2.3 modules reproduced the v2.3 page byte for byte.
- **v2.7:** the gallery (`js/16-gallery.js`) reads its groups and order from the Example structures drop-down; give a new example a one-line entry in `GAL_DESC` there.
- **v2.8:** model files are `{app:'Frame Pusher', format:1, version, saved, name, model}`, where `model` is `dump()` (`js/09-help.js`); `loadDump()` reads it back. `js/17-files.js` also accepts the bare `dump()` text from earlier versions.
- **v2.8.1:** `js/18-merge.js` wraps `delSel()` (js/05) for the node join and wraps `runTests()` (js/06) to append its own verification tests.
- **v2.9:** a member's profile is `m.np`, stored by `dump()` as member field 10. Sections keep their plate dimensions in `m.sec.g = {t, d, w}`. `solveCore()` (js/11) divides a non-prismatic member at `segPts(m)` and gives each element `npSeg()` stiffness and `npLd()` self-weight; these and the other solver helpers in `js/19-taper.js` are function declarations because start-up solves before js/19 runs. js/19 also wraps `split`, `swW`, `memStats`, `shapeFill`, `applySec`, `secCalc`, `buildPop`, `rpBlocks` and `runTests`.
- **v2.9.1:** `m.np` ends may be plate dimensions, `dA` / `dB` (same section type as `m.sec.g`): a taper is `{k:'taper', dB}` with end A = the member's own section; a haunch is `{k:'haunch', dA, la, dB, lb}`. v2.9 depth-ratio fields `ra` / `rb` (and `n` for generic members) are still read. `js/20-inspector.js` adds the member drawing, end conditions, loads and the section editor to the side panel (wraps `sideUpdate`, `buildPop`, `runTests`).
- **v2.10:** member end releases are `m.ha` (end A) and `m.hb` (end B), true or absent, stored by `dump()` as member field 11 (`[A, B]` as 0 / 1, or 0 for none). `mRel(m, 'a' | 'b')` in js/11 is the one test the solver uses: released when the member flag is set or its node is a hinge (`n.hinge` still means "release every end here"). Each released end gets its own rotation DOF (`m.ra` / `m.rb`, as hinges did); a node's own rotation DOF is fixed only when no unreleased moment-carrying member end meets there. `solveCore` returns `KM.RD`, a map of released-end DOF → `[member index, 'A' | 'B', node]`, and `dg.rnote` (fixed supports that act as pins). `js/21-releases.js` adds the End release tool (Q) after Hinge in the left panel, the inspector toggles (it replaces `insEnd` and wraps `insDiag` and `insUpdate`), the drawing on the canvas (wraps `draw`), report (wraps `rpStructure`, `rpBlocks`) and stiffness matrix labels (wraps `hmLab`), keeps releases through `split`, `mgCheck` / `mgJoin` and the node popup, adds two head-to-head examples and wraps `runTests`. js/09 copies them in `copySel` / `pasteClip`; js/16 draws them in the gallery cards.
- **v2.11 "Ghost":**
  - *Moving supports* — `js/22-ghost.js` wraps `drawSpr` (js/12) to call `drawGhosts()`. `ghostList(ds)` lists every support that moves visibly on the current deflection scale `dscale()` (so it follows Diagram scale and Animate). Settled supports are skipped (js/12 already draws them at the settled level). Ghosts are canvas-only; the PDF report is unchanged.
  - *Initial structure* — the bottom-bar checkbox `#sI` (in `page.html`, before Deflection). `draw()` in js/12 reads it as `SI` and skips the undeformed members, their section labels and the node dots when it is off.
  - *Unstable warning* — js/12 writes the warning as `<b class="unst"><i class="unstI">⚠</i>…</b>`, and the collapsed Info panel button gets class `unst`. `css/13-v2.11-ghost.css` makes both red with a throbbing icon (off under `prefers-reduced-motion`).
  - *Home button* — `<a id="home">` before the title in `page.html` links to https://ankacha.github.io/. When the page runs inside a frame, js/22 sets it to open in a new tab.
- **v2.12 "Clear":** (canvas and report decluttering)
  - *Canvas labels* — `txt()` in js/12 queues labels while a frame is drawn (`LBL`), with priorities in `LBL.pri`; `drawLoads()` calls `lblFlush()`, which places them with `lblPlace(items, mode, W, H)` (pure, tested). `LBL.mode` is `'all' | 'key' | 'none'`, set by `#lblM` (js/23) and kept in `localStorage['fp.labels']`.
  - *Point loads* — `plLen(F)` / `plTip(n)` in js/12; js/23 replaces `hitLoad` (js/05) to pick along the drawn arrow.
  - *Springs* — `sprSym(n)`, `sprKeys()`, `sprLink()` in js/12 draw the compound spring symbol.
  - *Section names* — `secLab(m)` / `secRuns()` in js/12.
  - *Report* (`js/13b-report-v2.12.js`, loaded straight after js/13, whose functions of the same name it replaces) — `rpSupMark()`, `rpAmp(F, key)`, `rpGap()`, `rpReac()` / `rpBlock()`, the Output column (wraps `rpBlocks`) and `rpSVG()` text widths.
  - js/23 appends 21 verification tests (372 in total).
- **v2.13 "Nodes":** (`js/24-nodes.js`, `css/15-v2.13-nodes.css`)
  - *Node inspector* — `nodeData(n, r)` (pure, tested) returns, for a solved result `r`, the node's displacements, one entry per element end at the node (`ends`: member, end A / B, internal N, V, M from `e.at()`, the global force the element puts on the joint `g = −Tᵀf` at that end from `nodeElF(e, n)`, the end rotation and whether it is a released-end DOF), the applied load `P`, the support reaction `R` (moment only for a fixed support), the spring force `S`, their sum `sum` and its relative residual `res`, and the node's 3 × 3 block of `KM.K` with the fixed DOFs. `nodeHTML(n)` builds the panel (free-body SVG `nodeSvg`, displacement, equilibrium table, member-end table, K block). It goes in `#sideNode`, inserted before `#sideMain`; `sideUpdate` is wrapped to show it when `interr` is on and a node is selected, to set the panel title (Member / Node inspector) and to show or hide the tab. `buildPop` is wrapped to add the Node inspector tick box to the node popup.
  - *Inspector tab* — `#sideTab`, a button in `#wrap`, visible while `#side` is hidden; `#wrap.tabOn` moves the dock left to make room. `sideOpen(on)` opens or toggles the panel; key I toggles it (ignored while the guide or gallery is open); `showKeys` is wrapped to list it.
  - *Theme* — the existing `#theme` button (its click handler is still in js/11) is moved to the end of `#top`, given class `thm` and a sun / moon icon by `thmSync()`; the choice is kept in `localStorage['fp.theme']` and applied at start-up.
  - *Surprising behaviour* — nine presets `sur_*` in a new optgroup before Head to head, with `GAL_DESC` entries; `galTitle` is wrapped to drop the "Surprise:" prefix in the gallery.
  - js/24 appends 43 verification tests (415 in total, all passing at release), including joint equilibrium at every node of every example.
