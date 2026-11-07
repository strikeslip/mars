The three datasets are three marsquake events that InSight's SEIS seismometer recorded in early 2019, each as three VBB components (BHU, BHV, BHW), so nine MiniSEED files in all:

| | Event 1 | Event 2 | Event 3 |
|---|---|---|---|
| **MQS event ID** | `mqs2019drgb` | `mqs2019fddj` | `mqs2019fxyu` |
| **Recorded (UTC)** | 22 Feb 2019, 02:51 | 14 Mar 2019, 21:28 | 26 Mar 2019, 06:07 |
| **Sol · Mars local time** | Sol 85 · 13:12 (midday) | Sol 105 · 18:29 (dusk) | Sol 116 · 19:50 (dusk) |
| **Stream** | XB.ELYSE.03 | XB.ELYSE.03 | XB.ELYSE.02 |
| **Sample rate** | 10 sps | 10 sps | 20 sps |
| **Length** | 60.0 min | 14.8 min | 53.2 min |
| **Catalogue name** | not determined | probably **S0105a** (Mw 3.0; unconfirmed) | not determined |
| **What the data show** | Mostly Martian wind; weak quake signal | One clear burst in the 0.1–1 Hz band around 21:30–21:31 | Several swells around 06:10, 06:18–19 and 06:29–30 |
| **Role in ELYSIUM** | Room (pedal, pad, wind) | Singer (the melody) | Rhythm (bass, kick, hats) |

The nine files:
- `xb.elyse.03.bh{u,v,w}.mqs2019drgb.1.mseed`
- `xb.elyse.03.bh{u,v,w}.mqs2019fddj.1.mseed`
- `xb.elyse.02.bh{u,v,w}.mqs2019fxyu.1.mseed`

**One correction on timing.** These weren't recorded just before SEIS stopped working; they're from the **early** part of the mission, within the first four months after the November 2018 landing. InSight kept recording for nearly four more years. It detected about 1,300 marsquakes in total, including the largest, S1222a (about magnitude 5) in May 2022, before dust ended the mission in December 2022.

If the statement or README describes them as the lander's last recordings, it should say *early-mission* recordings from InSight's first months on Mars. That's accurate, and arguably more poetic: they are among the first quakes Mars was ever properly recorded making.

**Also unconfirmed:** the events behind `drgb` and `fxyu` aren't confirmed as catalogued marsquakes. The windy midday record in particular may be largely atmospheric. Describe them as "three seismic event recordings" rather than "three confirmed marsquakes" unless you check the MQS catalogue.
