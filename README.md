# ELYSIUM MARS MUSIFICATION

**Three prototype musification instruments built from NASA InSight marsquake data, Elysium Planitia, Mars**

| | |
|---|---|
| Project | ELYSIUM MARS (adapted to Martian seismic data) |
| Lineage | Sounds of Seismic (SOS) → ELYSIUM MARS |
| Author | SHOOK (@ALLSH00K · allshookup.org) |
| Data | NASA InSight SEIS, network XB, station ELYSE, events `mqs2019drgb`, `mqs2019fddj`, `mqs2019fxyu` |
| Runtime | One self-contained HTML file; Web Audio API only; no libraries; runs offline |
| Document date | 3 November 2026 |

---

## Contents

1. [Summary](#1-summary)
2. [Mars context](#2-mars-context)
   - 2.1 NASA InSight lander
   - 2.2 Elysium volcanic province
   - 2.3 Marsquakes, and the Viking-2 precedent
3. [The source data](#3-the-source-data)
4. [Instrument architecture](#4-instrument-architecture)
5. [PERCEIVE: MiniSEED parser](#5-perceive--miniseed-parser)
6. [REASON: signal conditioning and features](#6-reason--signal-conditioning-and-features)
7. [PLAN: event-phase classifier](#7-plan--event-phase-classifier)
8. [ACT: three-layer granular engine](#8-act--three-layer-granular-engine)
9. [REFLECT: decision log](#9-reflect--decision-log)
10. [Interface and visualization](#10-interface-and-visualization)
11. [The three prototypes](#11-the-three-prototypes)
12. [Verification and testing](#12-verification-and-testing)
13. [Usage](#13-usage)
14. [Limitations and caveats](#14-limitations-and-caveats)
15. [References and data sources](#15-references-and-data-sources)

---

## 1. Summary

ELYSIUM MARS is a browser instrument. It turns real ground-motion recordings from Mars into electronica. It reads the raw MiniSEED files that NASA's InSight lander recorded in Elysium Planitia, decodes them in the browser, and analyzes the signal: envelope, RMS, derivative and zero crossings. Those features drive a three-layer granular synthesizer (Texture, Tone and Rhythm) on the Web Audio API.

It adapts the ELYSIUM MARS architecture to offline planetary data. ELYSIUM MARS is an agent system that renders live terrestrial seismic feeds as electronica through its **SeisConduct** loop: Perceive → Reason → Plan → Act → Reflect. The Mars version swaps the live network feed for local files, and it handles the problems specific to InSight data:

- **Steim2 compression.** The raw samples are compressed, not stored as 32-bit integers.
- **A tilted sensor.** The three seismometer axes (U, V, W) are not orthogonal and do not point north, east and up.
- **Low sample rates.** The data are 10–20 samples per second, so the usable band tops out at 5–10 Hz.
- **Instrument glitches.** Spikes much larger than the marsquakes themselves.
- **A strong day/night noise cycle.**

We built three prototypes in sequence, one per marsquake event. Each new data set exposed something the previous version did not handle. The final build loads any InSight VBB event (or any three-component terrestrial set) and adapts to its sample rate and length.

**Deliverables**

| File | Purpose |
|---|---|
| `v1-elysium-mars.html` | The instrument. Load any three BH* MiniSEED files by drag-and-drop or file picker. |
| `v1-elysium-mars-bundled.html` | Same code with the current event (`mqs2019fxyu`) embedded as base64. Plays as soon as it opens. |

---

## 2. Mars context

### 2.1 NASA InSight lander

**InSight** stands for *Interior Exploration using Seismic Investigations, Geodesy and Heat Transport*. It was the first mission dedicated to the deep interior of another planet: its job was to measure the crust, mantle and core of Mars.

| Item | Detail |
|---|---|
| Launch | 5 May 2018, Atlas V 401, Vandenberg SLC-3E, California |
| Landing | 26 November 2018, 19:52:59 UTC, Elysium Planitia |
| Landing site | ≈ 4.502° N, 135.623° E |
| Builder / management | Lockheed Martin Space / NASA JPL |
| Principal investigator | W. Bruce Banerdt |
| Primary mission | ~709 sols planned; operated **1,440 sols** (4 years, 19 days) |
| End of mission | Last contact 15 Dec 2022; declared over 21 Dec 2022 after dust cut its solar power |
| Cost | ~US$830 million total |

**Instruments**

- **SEIS** (Seismic Experiment for Interior Structure). The seismometer used in this project, provided by the French space agency CNES with IPGP, ETH Zurich, MPS, Imperial College, ISAE and JPL. It had two sensor sets:
  - **VBB** (Very Broad Band): a 3-axis sensor, and the source of the BHU/BHV/BHW channels used here.
  - **SP** (Short Period): a separate 3-axis sensor.

  The lander's arm placed SEIS directly on the ground, and a dome-shaped **Wind and Thermal Shield** covered it.
- **HP³** (Heat Flow and Physical Properties Package, DLR). A self-hammering heat probe meant to reach ~5 m. It stalled at ~0.35 m and was declared failed in January 2021.
- **RISE** (Rotation and Interior Structure Experiment). Radio tracking of Mars's rotation and wobble, used to infer core properties.
- **Weather and environment sensors:** TWINS wind and temperature sensors (Spanish Astrobiology Center), a magnetometer (UCLA), a pressure sensor (JPL) and a laser retroreflector (ASI).
- **Instrument Deployment Arm:** the 1.8 m arm that placed SEIS and HP³ on the ground.

**Key science results**

- **Seismicity:** about 1,300 marsquakes recorded over the mission. The largest, **S1222a** on 4 May 2022, was about magnitude 5, and its waves reverberated for roughly six hours.
- **Core:** liquid, with a radius of about **1,810–1,860 km**. That is larger than many pre-mission models predicted, which means a less dense core containing light elements.
- **Crust:** thinner than expected and layered, with two or three sub-layers under the lander.
- **Water (2024):** a PNAS study led by Vashan Wright (Scripps) modeled InSight's seismic wave speeds. It found that fractured igneous rock **~11–20 km** below the lander is best explained by pores holding liquid water, about 17% of the rock volume. The authors note large uncertainties.

### 2.2 Elysium volcanic province

InSight sat on the flat plains of **Elysium Planitia**, just south of the **Elysium volcanic province**, Mars's second-largest volcanic region after Tharsis.

| Feature | Detail |
|---|---|
| Location | Elysium and Cebrenia quadrangles; centered near 24.7° N, 150° E |
| Structure | Three major volcanoes on a broad dome about **1,700 × 2,400 km** |
| Volcanoes, north to south | **Hecates Tholus**, **Elysium Mons**, **Albor Tholus** |
| Activity span | At least ~3.9 billion years; peak ~2.2 billion years ago; youngest activity ~2 million years ago |
| Age contrast | The south-east is ~0.85 billion years younger than the north-west |
| Hecates Tholus | Eruption dated ~350 million years ago; caldera glacial deposits 5–24 million years old |
| Young lava | Flow units across Elysium Planitia, including Rahway Valles and Marte Vallis, are attributed to lava from the province |
| Cerberus Fossae | A system of young fissures overlapping the south-east of the province, linked to volcanic and lava–water processes. InSight detected marsquakes coming from this region. |

**Elysium Mons**

| Item | Detail |
|---|---|
| Coordinates | 25.02° N, 147.21° E |
| Height | ~12.6 km above its base; ~14.1 km above the Martian datum |
| Diameter | ~240 km |
| Summit caldera | ~14 km across, with sinuous channels past the rim |
| Flanks | Slopes of ~7–7.5°, about 1.5× steeper than other Martian shield volcanoes |
| Ranking | Third-tallest Martian mountain by relief, fourth-highest by elevation |
| Discovery | Found in 1972 in images from **Mariner 9**, the first spacecraft to orbit another planet |
| Distance from InSight | ≈ **1,380 km** great-circle (our calculation from the coordinates above, Mars radius 3,389.5 km) |

**Why it matters for this project.** Elysium is one of the youngest volcanic regions on Mars. The Cerberus Fossae fissures at its edge turned out to be the main source of InSight's located marsquakes, so the ground motion in these instruments was recorded on the doorstep of a recently active volcanic province. The 2026 re-analysis of Viking-2 signals (§2.3) suggests the same broad region may already have been active in 1976.

### 2.3 Marsquakes, and the Viking-2 precedent

Three Mars landers have carried seismometers: **Viking 1** and **Viking 2** in 1976, and **InSight** in 2018.

**Viking 1.** Its seismometer never released its launch lock (it "failed to uncage") and returned no useful data.

**Viking 2** (Utopia Planitia) returned the only Martian seismic data before InSight:
- **The instrument:** a 3-axis short-period sensor, 2.2 kg, 3.5 W, useful from about 0.1 to 10 Hz with peak sensitivity near 3 Hz.
- **Placement:** bolted to the lander deck rather than coupled to the ground, so it was dominated by wind. About half its data allocation was unusable, and the noise floor only dropped when winds fell below ~2 m/s.
- **Volume:** about 89 days (2,100+ hours) of low-noise data before it was switched off.
- **Sampling modes:**
  - *High Data Rate:* 20.2 samples per second per axis, usable only briefly.
  - *Event mode*, the most-used: **average amplitude and positive zero-crossing counts** at 1 sample per second per axis.
  - *Normal mode:* a 12.7 s running average sampled every 14.85 s.

**Re-analyses**

- **Lorenz et al. (2017)** documented the Viking-2 data archive in the PDS and showed how to use it as a meteorological record of the wind.
- **Lazarewicz (2023), "Viking Marsquakes 1976 — Seismic Archaeology"** re-examined the two candidate events, Sol 53 and Sol 80. Wikipedia's Marsquake article reports that this 2023 re-evaluation confirmed both as marsquakes.
- **"Fifty Years Later: Revisiting Viking 2 Seismic Signals Through InSight" (2026).** Eos and Universe Magazine (October 2026) report an *AGU Advances* study by **Xiao et al.** that compares the two Viking events with InSight data. It concludes that one of them could be a real marsquake, from the same region where InSight later recorded nearly 1,000 events. Wind remains a possible explanation. (We could not confirm that this is the same paper as the user-supplied PDF; see §15.)

> **Design note.** Viking's event mode reduced the Martian ground to two numbers per second: **amplitude** and **zero-crossing count**. ELYSIUM MARS's Reason stage computes the same two quantities (RMS envelope and zero-crossing density) as its main control signals. The instrument is, in a sense, a 2019 VBB re-reading of Viking's 1976 measurement idea.

**InSight event naming.** The Marsquake Service (MQS, ETH Zurich) catalogues each event two ways:
- a **sol-based name** such as `S0105a`: sol 105, first event of that sol;
- an internal **MQS event ID** such as `mqs2019fddj`, which is what names the event-waveform folders in the PDS.

Events are sorted into frequency classes (low-frequency, broadband, high-frequency, very-high-frequency and others).

---

## 3. The source data

### 3.1 Where the files come from

The event waveforms come from the NASA Planetary Data System (PDS) Geosciences Node, InSight SEIS archive bundle:

```
https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_seis/data/xb/event_waveform/elyse/<mqs-event-id>/
```

Each event folder holds many streams (VBB and SP sensors, several location codes and rates). Each stream comes as a `.mseed` file plus a PDS4 `.xml` label and an ASCII `.a.csv` export. For example, the `mqs2019fxyu` folder contains 17 streams across location codes 02, 03, 67 and 68. The same data are also served through EarthScope (formerly IRIS) FDSN web services and the IPGP SEIS data portal.

### 3.2 File naming

```
xb.elyse.03.bhu.mqs2019drgb.1.mseed
│  │     │  │   │           └─ file sequence number
│  │     │  │   └─ MQS event ID
│  │     │  └─ channel: B = broadband sample-rate band, H = high-gain seismometer, U = VBB axis U
│  │     └─ location code (02 = 20 sps VBB velocity, 03 = 10 sps VBB velocity, in these files)
│  └─ station: ELYSE (Elysium)
└─ network: XB
```

The PDS label for `xb.elyse.02.bhu.mqs2019fxyu.1` gives the signal as **"vbb 1 velocity high gain science mode"** at 20.00 Hz.

### 3.3 Format observed in all nine files

| Property | Value |
|---|---|
| Container | MiniSEED (SEED 2.x data-only records), big-endian headers |
| Record length | 512 bytes (from Blockette 1000) |
| Encoding | **STEIM2** (encoding 11). Not raw 32-bit integers, so a Steim decoder is required. |
| Blockettes | 1000 (encoding, word order, record length) and 1001 |
| Gaps | None in any of the three events |

### 3.4 The three events

Sol numbers and local mean solar time (LMST) at InSight were computed from the UTC start times with the Mars24 algorithm (longitude 135.623° E). Our method gives Sol 116 for `mqs2019fxyu`, which matches its PDS label, and puts the landing at ~14:17 LMST.

| | **Event 1** | **Event 2** | **Event 3** |
|---|---|---|---|
| MQS ID | `mqs2019drgb` | `mqs2019fddj` | `mqs2019fxyu` |
| Stream | XB.ELYSE.**03**.BH{U,V,W} | XB.ELYSE.**03**.BH{U,V,W} | XB.ELYSE.**02**.BH{U,V,W} |
| Sample rate | 10 sps | 10 sps | **20 sps** |
| UTC window (aligned) | 2019-02-22 02:51:56 → ~03:51:57 | 2019-03-14 21:28:39 → ~21:43:29 | 2019-03-26 06:07:17 → 07:00:26 |
| Aligned length | 36,015 samples · **60.0 min** | 8,902 samples · **14.8 min** | 63,782 samples · **53.2 min** |
| Samples U / V / W | 36,015 / 36,225 / 36,181 | 9,090 / 8,902 / 9,291 | 63,782 / 63,921 / 63,817 |
| Records U / V / W | 103 / 117 / 110 | 21 / 22 / 22 | 152 / 153 / 152 |
| Sol · LMST | Sol 85 · ~13:12 (midday) | Sol 105 · ~18:29 (evening) | Sol 116 (PDS label) · ~19:50 (evening) |
| Catalogue name | Not determined | Probably **S0105a** (see note) | Not determined |
| Character observed | Low signal-to-noise; broadband energy dominated by long-period drift; no clear onset | Clear 0.1–1 Hz burst around 21:30–21:31 over a steady 1–4 Hz noise floor | Several 0.1–1 Hz swells near 06:10, 06:18–19 and 06:29–30; brighter 1–10 Hz floor |

> **Note on catalogue names.** `mqs2019fddj` falls on Sol 105, and the MQS v14 changelog lists **S0105a** (Mw revised to 3.0 in v14). That makes S0105a the probable match, but we could not read the full catalogue in this session, so treat it as unconfirmed. We did not determine catalogue names for the other two events. The ETH catalogue page and the EarthScope Mars event service could not be reached from our environment.

> **Why the timing matters.** Daytime on Mars (Event 1) is windy and turbulent, and the VBB picks that up. Evening is much quieter, which is when InSight detected most of its events. This matches what the instruments show: Event 1 is mostly atmosphere, and Events 2 and 3 carry clearer quake energy.

---

## 4. Instrument architecture

```
 ┌───────────────────────────── SeisConduct loop (offline) ─────────────────────────────┐
 │                                                                                       │
 │  PERCEIVE            REASON                  PLAN                ACT                  │
 │  MiniSEED  ──►  condition + features  ──►  phase classifier ──►  granular engine ──┐  │
 │  parser          (env, RMS, dx/dt,          FLOOR/ONSET/PEAK/     Texture (V,W)     │  │
 │  (Steim1/2)       zero crossings,           CODA/DRIFT +          Tone    (U)       │  │
 │  align UVW        pseudo-Z)                 layer multipliers     Rhythm  (U)       │  │
 │                                                                                    │  │
 │                                            REFLECT ◄── decision log ◄──────────────┘  │
 └───────────────────────────────────────────────────────────────────────────────────────┘

 Signal chain
 Texture ─► HP 140 Hz ─┐
 Tone ────► LP (res.) ─┼─► sum ─► DC-block 28 Hz ─► master LP ─┬─► dry ─────────────┐
 Rhythm ───────────────┘                                       ├─► reverb send ─► conv ─┤
                                                               └─► delay send ─► dotted-8th ─┤
                                                   compressor ◄────────────────────────────┘
                                                        └─► make-up ×2.4 ─► master ─► analyser ─► out
```

**Timing model.** Two speed controls are separate, which is the core trick of granular synthesis:

- **Speed** (100–1600×, default 300×) is the spectral speed-up. It sets the grains' playback rate, so it decides which part of the seismic band becomes audible. At 300×, 0.1–10 Hz of ground motion maps to 30 Hz–3 kHz.
- **Scan** (1–60× event-seconds per audio-second) is how fast the playhead moves through the event. It sets the length of the performance, independent of pitch.

**Scheduler.** A 25 ms look-ahead scheduler (`setInterval`) schedules grains up to 140 ms ahead against `AudioContext.currentTime`. The playhead's sample index at audio time *t* is `idx0 + (t − t0) · scan · rate`, wrapped to loop. Changing Scan re-anchors `idx0` and `t0` so the playhead never jumps. If the tab stalls (for example, in the background), the scheduler skips the backlog instead of firing it all at once.

**Play buffers.** Each normalized component is written to an `AudioBuffer` at `rate × baseSpeed` (10 sps × 800 = 8 kHz; 20 sps × 800 = 16 kHz). An `OfflineAudioContext` then upsamples it to the device rate with the browser's high-quality resampler. Grains read from these buffers at `playbackRate = speed / baseSpeed`.

---

## 5. PERCEIVE — MiniSEED parser

The parser is a standalone script block (`<script id="mseed-parser">`) with no browser dependencies, so it can also run in Node for testing.

**Record walk**

1. **Header detection.** A record starts with a 6-character sequence number (digits or spaces) followed by a quality code `D/R/Q/M`.
2. **Byte order.** The header is taken as big-endian if the BTIME year field at offset 20 is plausible (1900–2100); otherwise little-endian.
3. **Fixed header fields:**
   - station / location / channel / network codes;
   - BTIME (year, day-of-year, h:m:s, 0.0001 s);
   - sample count (offset 30);
   - sample-rate factor and multiplier (32, 34), resolved per the SEED manual's four sign cases;
   - activity flags and time correction (applied unless flag bit 0x02 says it is already in);
   - data offset (44) and first-blockette offset (46).
4. **Blockettes.** The walk follows the blockette chain:
   - **1000** gives encoding, word order and record length (2ⁿ);
   - **100** gives the actual sample rate.
   - A guard counter stops malformed chains.
5. **Record length fallback.** If Blockette 1000 is missing, the parser probes power-of-two offsets (256 B–64 KiB) for the next valid header.

**Decoders**

| Code | Encoding | Method |
|---|---|---|
| 1 | INT16 | `DataView.getInt16` |
| 3 | INT32 | `DataView.getInt32` |
| 4 / 5 | FLOAT32 / FLOAT64 | `getFloat32` / `getFloat64` |
| 10 | STEIM1 | 64-byte frames; 2-bit nibbles → 4×8, 2×16 or 1×32-bit differences |
| 11 | **STEIM2** | As Steim1, plus dnib sub-codes → 1×30, 2×15, 3×10, 5×6, 6×5 or 7×4-bit sign-extended differences |

How the Steim decode works:
- Frame 0, words 1 and 2 hold the forward integration constant **X₀** and the reverse constant **Xₙ**.
- Samples rebuild as `x[0] = X₀`, `x[i] = x[i−1] + d[i]`. The first difference refers to the previous record, so it is ignored.
- An integrity check confirms that `x[n−1] == Xₙ`.

**Assembly**

- Records are grouped by `NET.STA.LOC.CHA`, sorted by start time and joined.
- Gaps are bridged with a linear ramp; overlaps are trimmed.
- The trace's encoding, record count and gap count are kept for the interface.

**Role assignment and alignment**

- If U, V and W are all present (Mars mode), U goes to Tone/Rhythm, V to Texture-left and W to Texture-right.
- Otherwise the parser falls back to Z/N/E (or 1/2), then to the first three channels. This lets the same instrument play terrestrial BHZ data.
- The three VBB channels start up to ~15 s apart (Event 1: V at 02:51:41.49, W at 02:51:50.49, U at 02:51:56.59). They are cropped to their common time window so the same sample index means the same moment on all three.

**Input paths**

- `<input type="file" multiple>` or drag-and-drop anywhere on the page;
- `ElysiumMarsV1.loadArrayBuffers([ab1, ab2, ab3])` or `[{name, buffer}, …]` for data fetched ahead of time;
- `window.ELYSIUM_V1_BUNDLE` (base64), loaded automatically by the bundled build.

---

## 6. REASON — signal conditioning and features

### 6.1 Conditioning (Mars-specific)

| Step | Why |
|---|---|
| Demean | Remove the DC offset. Raw counts sit at offsets in the thousands. |
| 30 s cosine taper at both ends | Stops filter edge transients. Without it, a high-pass on Event 3 creates a large false "burst" in the first minute. |
| **Zero-phase 2nd-order Butterworth high-pass, 0.08 Hz** (biquad run forward then backward) | Removes the VBB's long-period thermal and tilt drift. In Event 1, that drift made the broadband envelope a U-shaped curve across the hour. |
| **UVW → pseudo-Z rotation** | The VBB axes tilt ~29.5° up from horizontal at azimuths ~120° apart. A 3×3 matrix of the axis direction vectors is inverted, and its vertical row combines U, V and W into a pseudo-vertical stream: the closest equivalent to the BHZ channel used in SOS. |
| **Robust normalization** | Scale by the **99.5th percentile** of \|x\| instead of the peak, then soft-clip with `tanh(0.9·x/s)` into [−1, 1]. InSight "glitches" are larger than the quake signal, and peak normalization would bury the event. |
| **Event-band analysis copy** (added in Prototype 2) | A second copy, low-passed at **1.2 Hz** (zero-phase), drives every analysis feature. The audio keeps the full band. Marsquake energy sits mostly in 0.1–1 Hz, while the 1–4 Hz lander and wind noise is steady and would hide the event from the phase classifier. |

VBB orientations used (approximate published values; [azimuth°, dip°], SEED convention, negative dip = pointing up): **U [135.1, −29.4] · V [15.0, −29.2] · W [255.0, −29.7]**. Raw counts are treated as equal-gain, so the pseudo-Z is a musical approximation, not a calibrated vertical velocity.

### 6.2 Features

Frames use a 0.5 s hop and a 4 s window in event time. Running sums keep each frame O(1).

| Feature | Definition | Used for |
|---|---|---|
| `envU`, `envV`, `envW` | Windowed RMS per component, normalized by its 98th percentile | Layer amplitude and density, stereo lean |
| `M` | Combined envelope √((U²+V²+W²)/3), normalized | Master filter, reverb amount, kick gating |
| `Ms`, `slope` | `M` smoothed over ±8 s; slope = Ms(t) − Ms(t − 8 s) | Phase classification (Plan) |
| `derU` | Mean \|dx/dt\| of U, normalized | Texture grain length, Tone filter brightness, Tone lift |
| `zcrZ`, `zcrZn` | Zero-crossing density of pseudo-Z (per event-second), normalized at the 95th percentile | Tone pitch (scale degree), Texture octave sparkle |
| Zero-crossing list (U) | Positions with sub-sample interpolation, signed slope at each crossing, and the **rank** of each crossing's slope | Rhythm triggers |

**Zero crossings use hysteresis.** The state flips only when the signal passes ±0.04, so small jitter around zero does not register. Each crossing's position is interpolated back to the true sign change.

**Rank normalization** turns crossing slopes into a uniform 0–1 scale. A Rhythm threshold therefore maps directly to a hit probability, however loud or quiet the event is.

**Rhythm uses full-band crossings.** From Prototype 2 onward, Rhythm crossings come from the full-band U trace, while all density and phase features come from the event band. Counts per event:

| Event | Full-band U crossings | Event-band U crossings |
|---|---|---|
| drgb | 12,437 | n/a (Prototype 1 had no event band) |
| fddj | 1,702 | 188 (too sparse to drive Rhythm) |
| fxyu | 25,400 | n/a |

---

## 7. PLAN — event-phase classifier

Every 250 ms of audio time, Plan reads the smoothed envelope `Ms` and its slope at the playhead:

| Phase | Condition (checked in this order) | Musical intent |
|---|---|---|
| **ONSET** | slope > 0.06 | Energy arriving: Rhythm ×1.4, Texture ×1.3, dry and open |
| **PEAK** | Ms > 0.55 | Full activity |
| **CODA** | slope < −0.02 and Ms ≥ 0.18 | Long-tailed attenuation: Tone ×1.3, most reverb, filter closing |
| **FLOOR** | Ms < 0.18 | Noise floor: sparse Texture, Rhythm ×0.45, dark |
| **DRIFT** | anything else | Neutral |

| Multiplier | FLOOR | ONSET | PEAK | CODA | DRIFT |
|---|---|---|---|---|---|
| Texture | 0.60 | 1.30 | 1.20 | 0.80 | 1.00 |
| Tone | 1.00 | 0.80 | 1.00 | 1.30 | 1.00 |
| Rhythm | 0.45 | 1.40 | 1.20 | 0.70 | 1.00 |
| Space | 0.85 | 0.35 | 0.45 | 1.00 | 0.60 |
| Filter | 0.55 | 1.00 | 1.00 | 0.70 | 0.85 |

- **Hysteresis.** A new phase must hold for 3 consecutive Plan ticks before it takes over.
- **Smoothing.** Multipliers glide toward their targets (8% per tick), so changes never step audibly.
- **Continuous modulation**, also set each Plan tick:
  - Master LP cutoff = `base(Filter) · (0.3 + 0.7·M^0.6) · mul.filter`, where base runs from 120 Hz to 14 kHz on an exponential curve.
  - Tone LP cutoff = 500 + 2600·derU Hz.
  - Reverb send = `Space · (0.25 + 0.9·mul.space·(1.1 − M))`, so reverb grows as energy falls.
  - Delay send scales with Space and the Rhythm multiplier; dry level = 1 − 0.35·Space.

This is the "electronica attenuation" behavior: as the coda decays, the sound does not just get quieter. It gets darker, longer and more reverberant.

---

## 8. ACT — three-layer granular engine

Every grain is one `AudioBufferSourceNode` → `GainNode` (envelope) → `StereoPannerNode` → layer bus, and it disconnects itself when it ends. Smooth grains use a 64-point sin² (Hann) envelope via `setValueCurveAtTime`. Percussive grains get a 2 ms attack and an exponential decay.

### 8.1 Texture — BHV + BHW (stereo field)

| Parameter | Mapping |
|---|---|
| Source | V or W, chosen by their relative local envelope |
| Pan | V −0.65, W +0.65, plus 0.3 × (eW − eV)/(eV + eW) toward the louder component, ±0.2 jitter |
| Read position | Playhead ± 3 s of event time (spray) |
| Duration | 30–160 ms; a higher U derivative gives shorter grains |
| Pitch | speed/baseSpeed × 2^(±0.3 st jitter); jumps an octave with probability 0.15 + 0.4·zcrZn |
| Amplitude | 0.08 + 0.32·max(eV, eW) |
| Density | (3 + 68·e^1.2) × (0.25 + Texture slider) × Plan multiplier: about 3–70 grains/s |
| Bus | High-pass 140 Hz |

### 8.2 Tone — BHU wavetable grains

The Tone layer turns noise-like seismic data into **pitched** sound. Each grain loops a tiny slice of the BHU waveform (`loop = true`, `loopEnd − loopStart = r / f₀`), so one slice of Martian ground motion becomes one cycle of an oscillator:

- **Pitch** is set by the loop length, so it stays exact at any Speed.
- **Speed sets the timbre.** At 300×, one 110 Hz cycle holds ~3.4 s of 10 sps seismic data.

| Parameter | Mapping |
|---|---|
| Scale | A minor pentatonic over two octaves from A2 (110 Hz): 0, 3, 5, 7, 10, 12, 15, 17, 19, 22, 24 semitones |
| Scale degree | round(zcrZn × 8). A busier signal climbs; a quiet coda sinks toward the root. 35% chance of the chord tone two degrees below. A derivative spike lifts it up to 2 degrees. |
| Duration | 0.35 + 1.3·(1 − envU) s. Quieter means longer, overlapping drones. |
| Amplitude | 0.06 + 0.16·√envU |
| Pan | Slow sine LFO (0.13 rad/s, ±0.35) plus jitter |
| Density | (1.2 + 5·envU) × (0.3 + Tone slider) × Plan multiplier |
| Bus | Resonant low-pass (Q 4), cutoff tracking the U derivative |

### 8.3 Rhythm — BHU zero crossings on a 16th grid

The grid runs at 100 BPM in 16th notes (150 ms). On each step:

1. Map the step's audio-time span to its span of event samples.
2. Find the BHU zero crossing nearest the step boundary within that span (binary search).
3. Fire if that crossing's slope rank ≥ 1 − p, where p = Rhythm × 0.85 × Plan multiplier + accent. The accent is +0.15 on beats, +0.05 on 8ths and −0.05 on off-beats.
4. Choose the voice:
   - **Seismic kick** (beat steps, rank > 0.6, M > 0.2): BHU at 0.22× the current rate, 220 ms, through a 180 Hz low-pass.
   - **Click/hat:** the crossing itself at 3–6× rate, 15–65 ms, panned ±0.4 by the crossing's direction (rising or falling).

A dotted-8th feedback delay (450 ms, 38% feedback, 2.4 kHz low-pass in the loop) turns sparse hits into echoing patterns.

### 8.4 Master processing

| Stage | Settings |
|---|---|
| DC block | High-pass 28 Hz |
| Master filter | Low-pass, Q 0.9, modulated by Plan |
| Reverb | Generated stereo impulse response, 5.5 s with 25 ms pre-delay. Its one-pole low-pass darkens over the tail: a nod to Mars's thin CO₂ atmosphere (~0.6% of Earth's surface pressure), which strongly absorbs high frequencies. |
| Compressor | −20 dB threshold, 12 dB knee, 4:1, 6 ms attack, 280 ms release |
| Make-up gain | ×2.4 (added after Prototype 1 measured −29 dBFS RMS) |
| Master | Volume slider, 0.3 s fade on Begin and Stop |

---

## 9. REFLECT — decision log

Reflect writes to the on-screen log (newest first, 80 lines kept) and to the console (`[ELYSIUM MARS]`). Entries are stamped with the event-time UTC at the playhead. It logs:

- system events: parse summary, conditioning, audio setup, begin/stop, seek, loop;
- every committed phase change, with envelope and slope;
- a summary every 6 s: phase, envelope, pseudo-Z zero-crossing rate, chosen Tone frequency and interval, Texture multiplier, Rhythm hit count, master LP cutoff.

Example from Prototype 3 (`mqs2019fxyu`):

```
[06:07:17] perceived 3 channel(s) · 63782 aligned samples @ 20 sps · 53.2 min from 2019-03-26 06:07:17 UTC
[06:07:17] reasoned: 25400 BHU zero crossings · UVW→pseudo-Z rotation applied · audio HP 0.08 Hz · features 0.08–1.2 Hz
[06:07:17] begin · speed 300× · scan 21.5× (2.5 min per pass)
[06:07:52] PHASE → ONSET  (env 0.45, slope +0.096)
[06:09:05] PHASE → CODA  (env 0.40, slope -0.023)
[06:10:29] PHASE → ONSET  (env 0.91, slope +0.101)
[06:11:32] ONSET env 0.38 · zcr 1.18/s → tone 294 Hz (+17st) · tex ×1.14 · rhythm 24 hits · LP 2.11 kHz
```

---

## 10. Interface and visualization

- **Header:** "ELYSIUM MARS", plus a subtitle with mission, sensor, place, station ID and event ID. The station and event IDs come from the loaded files.
- **Intake:** a drop zone and one chip per channel showing code, sample count, rate, encoding, record count, gaps and assigned role.
- **Transport:** a Begin/Stop button and a SeisConduct indicator that lights the active stage. The readout shows event-time UTC at the playhead, current phase, grains per second, position and BPM.
- **Canvas:**
  - three lanes (BHU in rust, BHV and BHW in Martian-sunset blue), drawn as per-pixel min/max of the **0.08–1.2 Hz event band**;
  - the combined envelope as a dust-colored backdrop;
  - time ticks spaced 1, 2, 5, 10 or 30 minutes to suit the record length;
  - a playhead with its ±3 s spray window;
  - fading dots for each grain on the lane it came from;
  - a log-frequency spectrum strip (30 Hz–16 kHz) from an `AnalyserNode`.

  **Click the canvas to seek.**
- **Controls:** Texture, Tone, Rhythm, Filter, Speed, Scan, Space, Volume. Scan sets itself on load to give about 2.5 minutes per pass.
- **Styling:** a dark monospace theme. It adapts to phone width (the canvas shrinks to 280 px, the readout wraps).

---

## 11. The three prototypes

The prototypes were developed in sequence on one codebase. Each event's data exposed a limitation that the next version fixed. The current `v1-elysium-mars.html` includes all three rounds of changes and can load any of the three events.

### Prototype 1 — `mqs2019drgb` (Sol 85, midday, 10 sps, 60 min)

**Brief:** build a self-contained ELYSIUM MARS-style instrument for InSight VBB U/V/W data, with a MiniSEED parser, the SeisConduct loop, three granular layers, filter, compressor, reverb and delay, and a minimal interface.

**Findings and decisions**

- **Parser.** The files turned out to be **Steim2**, not raw int32, so we wrote a full Steim1/Steim2 decoder (keeping int16/int32/float support).
- **Drift.** The broadband envelope was dominated by long-period drift, hence the 0.08 Hz zero-phase high-pass.
- **Low SNR.** The 1–4 Hz band showed no clean onset, hence robust percentile normalization and a phase classifier with hysteresis.
- **Tone layer.** Transposing noisy seismic grains does not produce pitch, so we used loop-based wavetable grains.
- **Rhythm layer.** Raw crossings would fire on almost every step, so we added slope-rank selection on a 16th grid.
- **Defaults:** Speed 300×, Scan 24× (2.5 min per pass).
- **Level fix.** First measurement was −29 dBFS RMS; adding ×2.4 make-up gain brought peaks to about −5 dBFS.

**Result:** about 50 grains/s. Sparse, atmospheric and mostly a "noise floor" piece, which suits a midday recording dominated by wind.

### Prototype 2 — `mqs2019fddj` (Sol 105, evening, 10 sps, 15 min)

**Brief:** replace the data with event 2.

**Findings and decisions**

- **Shorter record.** A quarter of the length, so Scan defaulted to 6× (2.5 min per pass) and the time axis switched to 2-minute ticks.
- **Hidden burst.** A clear 0.1–1 Hz burst sat under a steady 1–4 Hz noise floor that the classifier could not see past, so we added the **event-band analysis copy** (0.08–1.2 Hz) for all features. The audio stays full band.
- **Rhythm too sparse.** The event band gave only 188 crossings in 15 minutes, so Rhythm crossings switched back to the full-band trace (1,702).
- **Labeling.** The event ID in the subtitle now comes from the filenames.

**Result:** the classifier tracked FLOOR → ONSET (21:28:56) → CODA (21:29:21) → DRIFT (21:29:50). About 17–22 grains/s, with 5–11 Rhythm hits per 6 s. The most defined, single-gesture piece of the three.

### Prototype 3 — `mqs2019fxyu` (Sol 116, evening, **20 sps**, 53 min)

**Brief:** replace the data with event 3.

**Findings and decisions**

- **Different stream.** Location code **02** at **20 sps**: double the bandwidth (Nyquist 10 Hz). Rate-dependent values already adapt: the play buffer is built at 16 kHz and the feature low-pass is min(1.2 Hz, 0.45 × rate).
- **False opening burst.** An apparent energy burst in the first minute turned out to be a filter edge artifact (0.2 relative once tapered), and the existing 30 s taper already handles it. The real swells are near 06:10, 06:18–19 and 06:29–30 UTC.
- **Scan sets itself** to about 2.5 min per pass for any record length (21.5× here).
- **Station ID** in the subtitle now comes from the files (XB.ELYSE.02).
- **Display.** At 20 sps the full-band waveform lanes turned into flat noise bands, so the canvas now draws the **event band**, which shows the quake.

**Result:** about 30–37 grains/s, peaks −3 to −4 dBFS. Three separate swell and coda cycles per pass: the busiest and brightest of the three pieces.

### Comparison

| | Prototype 1 · drgb | Prototype 2 · fddj | Prototype 3 · fxyu |
|---|---|---|---|
| Record | 60 min @ 10 sps | 15 min @ 10 sps | 53 min @ 20 sps |
| Local time | Midday (windy) | Evening | Evening |
| Audible band at 300× | 30 Hz–1.5 kHz | 30 Hz–1.5 kHz | 30 Hz–3 kHz |
| Default Scan | 24× | 6× | 21.5× (auto) |
| Grains/s (tested) | ~52 | ~17–22 | ~30–37 |
| Character | Diffuse, drifting texture | One clear rise and long coda | Several swells, brighter, busier rhythm |

---

## 12. Verification and testing

| Test | Method | Result |
|---|---|---|
| Parser correctness | The parser script was extracted from the HTML and run in Node on all 9 files. Output was compared with **ObsPy** (`obspy.read`): sample counts, start times, first and last 50 samples, and integer sum. | **Exact match** on all 9 channels; start time difference 0.000 ms; no gaps |
| Steim integrity | X₀/Xₙ check inside the decoder | No warnings |
| Runtime | Headless Chromium (Playwright): load by file input and by bundle, click Begin, run 9–15 s, seek, change Scan, stop and restart | No page errors or console errors/warnings |
| Output level | `AnalyserNode` time-domain sampling, 60–100 frames | Peak −2.9 to −7.3 dBFS across builds and events; median RMS about −21 to −30 dBFS; no NaNs; light compression (−1.6 to −2.7 dB gain reduction) |
| Behavior | Phase sequence, grain rate and Rhythm hit counts read from the instrument state | As described in §11 |
| Visual | Screenshots at 1100 px wide | Lanes, playhead, grain dots, spectrum and log rendering correctly |

Not tested: listening on real speakers or headphones, and Safari or Firefox. Testing used headless Chromium only.

---

## 13. Usage

1. Open `v1-elysium-mars.html` in a current Chrome, Edge, Firefox or Safari. No server or network connection is needed.
2. Drop the three files (`xb.elyse.LL.bhu|bhv|bhw.<mqs-id>.1.mseed`) onto the page, or click the drop zone to pick them. The channel chips confirm decoding.
   - Or open `v1-elysium-mars-bundled.html`, which has its event built in.
3. Press **Begin**; browsers require a click before audio can start.
4. Play:
   - **Texture / Tone / Rhythm:** layer levels and densities.
   - **Filter:** master low-pass ceiling; the envelope opens it further.
   - **Speed:** pitch and timbre of the seismic band (100–1600×).
   - **Scan:** how fast the event passes; set automatically on load.
   - **Space:** reverb and delay.
   - **Click the waveform** to jump to a moment in the event.
5. To load another event, drop a new set of three files at any time.

**From code**

```js
const bufs = await Promise.all(urls.map(u => fetch(u).then(r => r.arrayBuffer())));
await ElysiumMarsV1.loadArrayBuffers(bufs);   // or [{ name, buffer }, …]
ElysiumMarsV1.begin();
ElysiumMarsV1.params.speed = 500;             // live parameter object
```

**Getting more events.** Browse `…/data/xb/event_waveform/elyse/` in the PDS SEIS bundle and download the three `bh{u,v,w}` `.mseed` files for one location code (02 or 03). Or request `XB.ELYSE.<loc>.BH?` for a time window from the EarthScope FDSN dataselect service.

---

## 14. Limitations and caveats

- **Not instrument-corrected.** Samples are raw counts, high-passed and percentile-normalized. They are not converted to m/s, and the instrument response is not removed. Amplitudes are relative, for musical use only.
- **Pseudo-Z is approximate.** It uses nominal published VBB orientations and assumes equal gain on all three axes. For science-grade rotation, use the dataless SEED / StationXML response and orientation metadata.
- **Glitches are softened, not removed.** Soft-clipping keeps glitches from dominating the sound, but they are not detected or subtracted.
- **Phase labels are musical states.** FLOOR, ONSET, PEAK, CODA and DRIFT come from envelope heuristics. They are not seismological P/S picks or MQS event classes.
- **Event identification is incomplete.** Only `mqs2019fddj` has a probable catalogue name (S0105a), and that match is not confirmed.
- **Randomness.** Grain scheduling uses `Math.random`, so no two performances are sample-identical. Seeding a PRNG would make renders reproducible.
- **Browser support.** Requires `StereoPannerNode` and `OfflineAudioContext`. Background tabs throttle timers, and the scheduler skips the backlog rather than playing it all at once.

---

## 15. References and data sources

**About the PDFs.** Three PDFs were described as attached earlier in this thread, but none were present in this session's files. The citations below use the public pages and links supplied; content attributed to those papers comes from those pages and from secondary coverage.

1. Lorenz, R. D., et al. (2017). *Viking-2 Seismometer Measurements on Mars: PDS Data Archive and Meteorological Applications.* Earth and Space Science. https://agupubs.onlinelibrary.wiley.com/doi/10.1002/2017EA000306
2. *Fifty Years Later: Revisiting Viking 2 Seismic Signals Through InSight* (2026). https://www.researchgate.net/publication/415048583_Fifty_Years_Later_Revisiting_Viking_2_Seismic_Signals_Through_InSight. Related coverage: Francis, M. R. (21 Oct 2026), "InSight into Seismology on the Red Planet," *Eos*, reporting Xiao et al., *AGU Advances*, doi:10.1029/2026AV002319. https://eos.org/?p=251816
3. *InSight-Seismic-Data-Mars-China-Analysis-2026.pdf*: user-supplied. Not available in this session; not cited for content.
4. Lazarewicz, A. R. (2023). *Viking Marsquakes 1976 — Seismic Archaeology.* JGR Planets. https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2022JE007660
5. *Basic processing of the InSight seismic data from Mars for further seismological research* (2025). https://www.sciencedirect.com/science/article/pii/S1674451925000400
6. EarthScope (2024). *InSight data may have revealed water deep below the surface of Mars* (Wright et al., PNAS, doi:10.1073/pnas.2409983121). https://www.earthscope.org/news/insight-data-may-have-revealed-water-deep-below-the-surface-of-mars/
7. Viking Lander Seismometry Final Report (PDS). https://pds-geosciences.wustl.edu/viking/vl2-m-seis-5-rdr-v1/vl_9020/document/viking_seismometry_final_report.pdf
8. *Analysis of Data from Viking 2 Helped Identify a Previously Unknown Marsquake.* Universe Magazine (Oct 2026). https://universemagazine.com/en/analysis-of-data-from-viking-2-helped-identify-a-previously-unknown-marsquake/

**Data and catalogues**

- PDS Geosciences Node — InSight SEIS: https://pds-geosciences.wustl.edu/missions/insight/seis.htm
- Event waveform folder used for Event 3: https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_seis/data/xb/event_waveform/elyse/mqs2019fxyu/
- MQS catalogue v14 (PDS derived data): https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_seis/data_derived/mqs_catalog_v14/
- Mars Seismic Catalogue v14 (ETH Zurich): https://www.insight.ethz.ch/en/seismicity/catalog/v14/
- EarthScope / IRIS — InSight data access: https://www.iris.edu/hq/sis/insight#DMC
- IPGP — SEIS data releases: https://www.seis-insight.eu/en/science/seis-data/seis-data-releases
- InSight Analyst's Notebook — SEIS data products: https://an.rsl.wustl.edu/help/Content/About%20the%20data/Data%20products/INS/SEIS%20data%20products.htm

**General background**

- InSight: https://en.wikipedia.org/wiki/InSight
- Elysium volcanic province: https://en.wikipedia.org/wiki/Elysium_(volcanic_province)
- Elysium Mons: https://en.wikipedia.org/wiki/Elysium_Mons
- Mariner 9: https://en.wikipedia.org/wiki/Mariner_9
- Marsquake: https://en.wikipedia.org/wiki/Marsquake

**Software**

- ObsPy, used only for verification: https://www.obspy.org
- Mars24 time algorithm (Allison & McEwen), used for the sol and LMST calculations: https://www.giss.nasa.gov/tools/mars24/

---

*ELYSIUM MARS is part of the SE1STR0N1CA / ELYSIUM MARS family of seismic sonification instruments by SHOOK.*
