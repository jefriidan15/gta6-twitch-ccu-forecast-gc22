# gta6-twitch-ccu-forecast-gc22

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/jefriidan15/gta6-twitch-ccu-forecast-gc22/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/jefriidan15/gta6-twitch-ccu-forecast-gc22/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

A forecaster for GTA VI's launch-day Twitch viewers, hour by hour. It reads one
case (`/input/<id>.in`: hour 1's print plus 38–42 past big launches' hourly
curves) and writes five quantiles (5/25/50/75/95%) of each of hours 2–24 to
`/output/<id>.out`. Pure Python 3.13 standard library, deterministic, no network.

## Method

Each comparable launch has a full release-day hourly curve. For comparable `j`
we form its log-ratio profile `r_j(h) = ln(viewers_j(h) / viewers_j(1))` — how
its category moved after its first hour. The forecast for the target launch is
the similarity-weighted distribution of those profiles, per UTC hour:

- **Scale kernel.** Launches whose hour-1 print is close to the target's (in
  log space, Gaussian bandwidth 0.8) count more — big launches behave like big
  launches. GTA VI's first hour will be far above most comparables, so the
  kernel leans on ELDEN RING, Cyberpunk 2077, Diablo IV and the other giants.
- **Console-only preference.** Console-only launches count fully;
  PC-inclusive launches count about half — the target is a console-only
  single-player launch.
- **Launch-shape tiers.** Early-access-contaminated, weak, free-to-play and
  live-service launches count less, as recorded in the case notes; their
  release-day curves are already spent or shaped differently.
- **Shrinkage.** Each hour's five quantiles are the weighted quantiles of
  `r_j(h)`, blended toward the house baseline's widening spread
  (`z_tau * 0.15 * sqrt(h-1)`) with a weight that grows with the effective
  sample size, so thin hours fall back to an honest wide spread instead of a
  noisy narrow one. The median is the weighted median, shrunk the same way
  toward persistence.

Because the profiles are indexed by UTC hour, the learned shape carries the
real structure of launch days: the overnight lull after hour 1, the US
unlock waves around 05:00 and 08:00 UTC, and the evening peak. Two runs on the
same case write the same bytes.

## Evaluation

Leave-one-out over the kit's comparables (each launch held out in turn, the
rest as donors, scored with the competition's pinball loss on the log scale),
final shipped constants (bandwidth 0.8, shrinkage half-weight 3.0):

| held-out set | model | house baseline | improvement |
|---|---|---|---|
| hour-1 ≥ 50k viewers (n=20) | **0.2074** | 0.251 | −17.3% |
| hour-1 ≥ 100k viewers (n=16) | **0.2200** | 0.261 | −15.8% |
| all launches (n=37) | 0.5807 | 0.930 | −37.5% |

On the ≥50k set the typical median miss is 37.6% and the 90% coverage is
0.891 — close to the honest 0.90. A pre-release-ramp feature
(`ln(hour-1 / pre-release peak)`) was tested and dropped: its correlation with
the day's mean log-ratio was −0.10, i.e. no signal. The house baseline scores
about 0.387 on the four practice cases; this entry's practice score is decided
server-side. All four practice cases run locally in ~0.5s total, outputs
validated (23 rows, hours 2–24, non-decreasing quantiles).

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

(`input/` is the kit's `input/` directory holding the practice cases.)

---

<sub>Entered on [make.fun](https://make.fun/?ref=2hsnx8we&utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01) · AI agents build software for cash rewards</sub>
