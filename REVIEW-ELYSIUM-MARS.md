# Technical & Cultural Review: ELYSIUM MARS MUSIFICATION 

**Project:** ELYSIUM MARS MUSIFICATION (v22)  
**Author / Lineage:** SHOOK (`@ALLSH00K`) · SE1STR0N1CA · `seis.monster` lineage (successor to *SOS* and *SEISCLAW MARS*)  
**Runtime Stack:** Single-file Vanilla HTML5 / CSS / ES6+ / Web Audio API / Canvas 2D (Zero dependencies, offline-capable)  
**Data Sources:** NASA InSight SEIS VBB waveforms (`XB.ELYSE`, 2019 — *note: 2019, not 2029*) & MOLA `MarsTopo719` topography  
**License:** MIT (Code) · CC BY 4.0 (InSight SEIS) · BSD-3-Clause (SHTOOLS / MarsTopo719 derivative)<br>
**Review by:** Gemini 4 Hydrogen
**Date:** Nov 20, 2026

---

## Executive Summary

**ELYSIUM MARS** is an unusually cohesive piece of scientific instrument-making, generative music, and planetary cartography packed into a single self-contained HTML file. Rather than treating Martian seismic data as an abstract trigger for generic ambient sound, it builds a **physical-to-musical-to-visual pipeline** rooted in real seismology (non-orthogonal UVW rotation, glitch-robust scaling, Mars24 solar time), minimalist composition (Terry Riley–inspired additive cells, polyperiodic realignment, dual-tape time-lag accumulation), and spaceflight visual history (Mariner 4 paint-by-numbers, Mariner 9 dust-storm clearing, MOLA laser altimetry).

---

## 1. Technical Review

### 1.1 Single-File Packaging & Offline Resilience
- **Zero-dependency footprint:** The entire application—MiniSEED decoder, DSP pipeline, Web Audio synthesizer/effects graph, 4-language UI (`EN`, `中文`, `RU`, `FR`), 9 embedded InSight MiniSEED traces (`window.ELYSIUM_BUNDLE`), two MOLA terrain grids (`window.ELYSIUM_TERRAIN`), and base64 MOLA favicons—lives in one portable `.html` file.
- **Data encoding efficiency:**
  - Regional MOLA tile (`0.125°`, `288 × 360`, `0–36°N, 125–170°E`): quantized to 16-bit unsigned integers (`u16`) and base64-encoded.
  - Global MOLA sphere (`1°`, `180 × 360`): quantized to 8-bit unsigned integers (`u8`).
  - Both grids avoid runtime spherical-harmonic expansion while preserving metric elevation bounds (`-5582.3 m` to `+14012.3 m` regionally; `-6931.6 m` to `+22332.5 m` globally).

### 1.2 Seismological Parsing & Signal Conditioning
- **Custom MiniSEED 2.x Parser (`MiniSEED`):**
  - Automatically infers header endianness from BTIME year plausibility (`1900–2100`).
  - Walks Blockette 1000 (data encoding, word order, record length) and Blockette 100 (actual sample rate), and respects the SEED time-correction activity flag (`act & 0x02`).
  - Decodes `INT16`, `INT32`, `FLOAT32`, `FLOAT64`, `STEIM1`, and `STEIM2` (with forward/reverse integration constants `x0`/`xn` and integrity verification).
  - `assemble()` sorts multi-record traces chronologically, bridges gaps via linear interpolation, and trims overlaps.
- **Physical Triad Inversion (`uvwToZ`):**
  - Correctly treats InSight SEIS VBB channels `BHU`, `BHV`, `BHW` as a non-orthogonal Galperin-style triad (`VBB_ORIENT`) and inverts the 3×3 orientation matrix (`invert3`) to synthesize a true pseudo-vertical `Z` channel.
- **Glitch-Robust Conditioning (`reason`):**
  - Applies demean → 30 s cosine taper → zero-phase 2nd-order Butterworth high-pass (`0.08 Hz`, removing long-period thermal/tilt drift) → zero-phase low-pass (`1.2 Hz`, isolating the marsquake band).
  - `normalizeRobust()` scales by the **99.5th percentile** of `|x|` followed by `tanh(0.9 * x / s)`, preventing InSight's notorious thermal "donk" glitches from crushing the dynamic range of genuine seismic energy.
- **Dual-Scale Feature Extraction (`shareScale`):**
  - Computes **own-scale** envelopes (`M`, smoothed `Ms`, `slope`) for per-event structural phase detection (`FLOOR`, `ONSET`, `PEAK`, `CODA`, `DRIFT`) with 6-frame hysteresis.
  - Computes **shared-scale** envelopes (`Mx`, `Ax`) normalized across *all* loaded events simultaneously, preserving the real physical contrast between noisy midday wind (`mqs2019drgb`, Sol 85) and quiet dusk quakes (`mqs2019fddj`, Sol 105; `mqs2019fxyu`, Sol 116).

### 1.3 Audio Engine, DSP & WebKit / iOS Hardening
- **880× Speed Standard & Wavetable Grain Synthesis:**
  - Pre-renders buffers at `880×` (`1 Hz` on Mars → `880 Hz` / `A5`, the 9th of `Gadd9`), so pitched grains (`note()`) loop short waveform slices (`loopLen = rate / f0`) directly out of the Martian ground motion. The quake waveform *is* the oscillator timbre.
- **V22 Synchronous Resampling for iOS / WebKit:**
  - A standout engineering detail in v22: instead of running 15 async `OfflineAudioContext` renders before starting (which causes Safari/WebKit to drop the user-activation token and block `AudioContext.resume()`), `makeBuffers()` resamples synchronously in JS using 4-point Catmull-Rom cubic interpolation (`resampleCubic`).
  - Sets `navigator.audioSession.type = 'playback'` (iOS 17+ silent-switch bypass), fires a 1-frame silent unlock buffer, and attaches `onstatechange` + `visibilitychange`/`pointerup` recovery (`tryResume`) for lock-screen and phone-call interruptions.
- **Terry Riley Time-Lag Accumulator (`tlaA`, `tlaB`):**
  - Implements two cross-coupled long delays (`3 bars` and `5 bars`) with per-pass tape HF loss (`3800 Hz` / `3100 Hz` LPFs), soft saturation (`WaveShaperNode` `tanh(1.6x)/1.6`), independent slow wow LFOs (`0.31 Hz`, `0.23 Hz`), and stereo panning (`-0.55`, `+0.55`).
  - Feedback matrix gain is mathematically bounded (`fb <= 0.85`, wave-shaper slope `<= 1`), guaranteeing unconditional stability even as feedback swells during `CODA` phases and Martian night.
- **Precision Lookahead Scheduler:**
  - Uses a 25 ms `setInterval` feeding a 150 ms Web Audio `ctx.currentTime` lookahead (`LOOKAHEAD = 0.15`), with backlog skipping (`S.nextStep < now - 0.2`) to prevent burst-fire glitches after tab throttling.

### 1.4 Visual Engine (`ELYSIUM VISION`)
- **Four Real-Topography Projections:**
  - `RISE`: Custom CPU voxel-space raycaster (`FOVK = 0.62`, `VE = 5` vertical exaggeration) with exponential distance fog and solar azimuth/elevation hillshading.
  - `PLAINS`: Top-down `0.125°` regional relief map with stylized Cerberus Fossae fissures and animated pulse rays travelling to InSight.
  - `CALDERA`: Bilinearly interpolated summit close-up marking the ~14 km caldera rim.
  - `ORBIT`: Orthographic 1° global projection rotating smoothly under a sol-clock-driven subsolar illumination vector.
- **Dithering & Paint-by-Numbers Atlas:**
  - `Pixel` and `Hybrid` modes apply a 4×4 Bayer ordered dither to the low-res framebuffer (`300–420 px` wide).
  - `ASCII` mode pre-renders a multi-luminance glyph atlas (`0123456789.`) across three colour families (`MINT`, `RUST`, `DUSKC`) and renders elevation quantized to digits `0–9`.
- **Two-Layer Composition:**
  - Separates the terrain framebuffer from a persistent glyph particle canvas (`glyph`) whose `destination-out` decay rate scales dynamically with the audio engine's Time-Lag Accumulator feedback (`S.fb`).

### 1.5 Code Audit: Potential Edge Cases & Polish Opportunities

| Area | Observation | Impact / Suggestion |
| :--- | :--- | :--- |
| **Short External MiniSEED Files** | In `grain()`, `off = Math.min(off, buf.duration - loopLen - 0.02)`. If a user drops a very short external trace (e.g., `< ~44 s` at 20 sps, yielding `< 0.05 s` at 880×), `buf.duration - loopLen - 0.02` can become negative. | Clamp `off = Math.max(0, ...)` and guard `buf.duration > loopLen + 0.01` before setting `s.loop = true`. |
| **Truncated MiniSEED Records** | In `MiniSEED.parse()`, if `reclen < r.dataOffset`, `len` becomes negative. Steim decoding is safe (`nframes <= 0`), but raw `INT16`/`INT32`/`FLOAT` loops iterate to `nsamp` regardless. | Add `if (len <= 0) continue;` and bound `nsamp` against remaining bytes in `decodeData()`. |
| **Drawer `#viz` Legacy Palette** | `drawStatic()` and `frame()` in the drawer waveform view still use ochre/brown hex values (`#090706`, `#7d6b5f`, `rgba(217,168,108,...)`) from an earlier aesthetic iteration, contrasting with the mint/obsidian CSS (`#0d1513`, `#bff1e5`). | Harmonize `#viz` background and sky-wash colours with `--bg` (`#0d1513`) and `--mint`/`--blue`. |
| **Mobile CPU / Battery in `ORBIT` & `RISE`** | `renderOrbit()` evaluates `Math.asin`, `Math.atan2`, and `Math.hypot` per pixel at 30 fps (`~99k px` desktop / `~50k px` mobile), plus `Math.random()` per pixel in `post()`. | Consider throttling terrain redraws to `15–20 fps` when `dust < 0.02` or precomputing the sphere's `(lat, lon)` map per rotation step / using a reduced `LW` on `devicePixelRatio` constrained mobile viewports. |
| **`ROOM` Slider vs. G Pedal Floor** | `room()` clamps pedal amplitude to `Math.max(0.35, P.room)`, though `G.room.gain` ramps to `P.room` (`0` at slider minimum), effectively muting it. | Intentional and safe, though `Math.max(0.35, P.room)` is redundant when `P.room === 0`. |
| **i18n Native Review** | Comments inside `<script id="i18n">` transparently note that Russian (`RU`) and French (`FR`) translations await native-speaker review. | Track native-speaker proofreading as a release milestone. |

---

## 2. Cultural, Musical & Scientific-Artistic Review

### 2.1 Philosophy of "Constrained Musification"
A recurring failure mode of naïve data sonification is **literalist mapping**: feeding raw seismic frequencies or amplitudes directly into unrestricted pitch and rhythm generators, producing fatiguing, random-sounding bleeps that obscure rather than illuminate the data.

**ELYSIUM MARS** rejects that approach with an explicit compositional thesis:
> *"The seismic data never chooses free pitch or timing. It chooses touch: velocity, density, which safe note, when the melody grows or shrinks, when a bent 'away' note appears, how bright each voice is."*

By locking the band into **G major / Gadd9** at **108 BPM** with a permanent G pedal, the seismic data is given a *musical room* in which to act as performers. The listeners' ears stay anchored while the real Martian physics—wind noise at midday, quiet high-Q bursts at dusk, coda decay—manifest as **expression, articulation, orchestration, and space**.

### 2.2 Polyperiodic Sol Numerology & Cyclical Time
The piece translates orbital/mission metadata directly into minimalist rhythmic structure:
- **Sol 85** (`5 × 17`) → `5`-bar chord blocks (`20`-bar harmonic cycle)
- **Sol 105** (`3 × 5 × 7`) → `7`-eighth-note naive melody cell
- **Sol 116** (`4 × 29`) → `4`-beat bass ostinato
- **Realignment (`lcm`):** Every **140 bars**, all three loops converge on `HOME`, closing a glowing *ensō* around the caldera.
- **Sets (`95` bars):** Every ~3.5 minutes, the band shifts between six idiomatic styles (`Band`, `Curved Air`, `Dub`, `Mariner`, `Broken`, `Drone`) while the three quakes **swap instruments** (`SET_ROLES`), rotating through 36 combinations (~2 hours 7 minutes) before repeating. Sets drift against the 140-bar `HOME` cycle, realigning only every **2,660 bars** (~98.5 minutes).
- **Sol Swing (`SOL_RATIO = 1.0274912517`):** Every fourth bar stretches by `+2.75%`—the exact fractional difference between an Earth day and a Martian sol—turning a geophysical constant into a subtle rhythmic *feel*.

### 2.3 Spaceflight & Minimalist Lineage
The work weaves together multiple strands of 20th-century electronic music and planetary exploration history:
1. **Terry Riley (*In C*, *A Rainbow in Curved Air*, Time-Lag Accumulator):**
   - Evident in the additive 7-note melody cell (`CELL_ORDER` expanding from 3 to 7 notes and retreating), the rapid 5-note high pentatonic *Curved Air* runs, and the dual-tape feedback accumulator.
2. **Mariner 4 (1965) "Paint-by-Numbers":**
   - The `ASCII` mode maps MOLA elevation to digits `0–9` in tribute to the famous 1965 moment at JPL when engineers, impatient for digital image processing of Mars's first close-up telemetry, printed the numeric strips and hand-coloured them with pastel chalks.
3. **Mariner 9 (1971–1972) Dust Storm:**
   - Every scene initializes inside a suspended dust veil (`dust = 0.62`), allowing only the highest summit of Elysium Mons to poke through until playback begins and the storm settles—mirroring Mariner 9's arrival during the 1971 planet-wide dust storm, when the Tharsis and Elysium volcanoes were the first features visible above the haze.
4. **The *Ensō* & *Huǒ* (`火`) Play Button:**
   - Framing the transport trigger as an unclosed Zen *ensō* surrounding the character `火` (fire / Mars—*Huǒxīng* / *Kasei*, "Fire Star" in East Asian astronomy) bridges minimalist visual contemplation with Martian planetary identity.
5. **Colour Discipline:**
   - On a CRT mint-phosphor (`#bff1e5`) and obsidian (`#0d1513`) ground, rust-orange (`#e0663a`) is **strictly rationed** to three semantically linked elements: the word `MARS`, the Elysium Mons volcanic edifice (`VW`), and the `SINGER` voice/melody rings. Blue (`#7fb2c9`) represents both Martian twilight (real scattering physics of fine Martian dust) and the `RHYTHM` / Cerberus Fossae fissure sparks.

### 2.4 Scientific Honesty & Cultural Accessibility
- **Transparency over hype:** The embedded `Readme` explicitly distinguishes between real MOLA data and artistic stylization (acknowledging that Elysium Mons' 14 km caldera and the narrow graben of Cerberus Fossae lie below the `0.125°` / ~7 km grid resolution).
- **Planetary Commons:** Offering full UI and explanatory notes in **English, Simplified Chinese (`中文`), Russian (`RU`), and French (`FR`)**—with no external trackers, CDNs, or accounts—positions Mars seismic data not as proprietary spectacle, but as shared human heritage.

---

## 3. Verdict

**ELYSIUM MARS** is an exemplary synthesis of **domain rigor** and **poetic restraint**. Its technical foundations—from Steim-2 MiniSEED decoding and Galperin triad inversion to synchronous Catmull-Rom resampling for WebKit audio compliance—are unusually solid for a browser-based art-science work. Culturally, it elevates data sonification into a genuine musical instrument: one that remembers Mariner 4's chalk pastels, Mariner 9's dust, Terry Riley's tape loops, and InSight's quiet evening listening on Elysium Planitia.
