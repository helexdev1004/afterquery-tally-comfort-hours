# Tally comfort hours for a service-charge rebate

A harbor-format benchmark task, authored for the AfterQuery Experts programme as data
annotation for AI evaluation. Task slug `afterquery/tally-comfort-hours` — **Operations / Building Management**.

An AI agent gets a container, this brief and nothing else. It works alone for
4 hours. Then a sealed verifier it never sees scores the result `0` or `1`.

## What it asks for

Comfort hours for the tenants' service-charge rebate

The full brief is in [`instruction.md`](instruction.md).

| | |
| --- | --- |
| **Deliverable** | `/app/out_of_band.csv` |
| **Agent budget** | 4 hours (14400s) |
| **Expert estimate** | 4 hours |
| **Machine** | 1 CPU, 2048 MB RAM, 0 GPU |
| **Verifier budget** | 300s |
| **Tags** | `building-management`, `temperature-trends`, `lease-compliance`, `data-cleaning`, `python` |

## Why it is hard

Who does this: a building services engineer or a managing agent's energy analyst, settling a comfort clause in a lease from building management system trends. It is ordinary month-end work wherever a lease ties a rebate to zone temperature. Where the data comes from: it is synthetic, and was made truth first.

<details>
<summary>Full note</summary>

Five weeks were run for 10 zones, each with a wall sensor, a tenant keeping the hours in the lease and plant that mostly holds setpoint and sometimes slips; the 50 sealed figures count the two-minute readings of the sensor that is really in each zone. The trends and the point list were written afterwards. The figures are exact and nothing is computed when the work is marked. What is plainly wrong in the files: three of the ten trends are not in degrees Celsius as the agent's note says - one is in Fahrenheit, one in kelvin, one in hundredths of a degree written as whole numbers. Anyone who looks sees it, and each ignored costs a zone's five lines where only 4 may be out. With the note taken at its word on everything the score is 20 of 50. What is not plain, and decides the outcome: the point list is wrong for 4 trends. Since a controls refit, two pairs of zones have been trended under each other's names. Nothing in the files says so, and using the list as it stands fails without a sign - every zone still gets five believable weekly figures - at a cost of 20 lines. The usual summaries are no help. The plant runs to setpoint day and night, so a zone's mean inside its lease hours is within 0.27 degrees of its mean outside them, and its slips out of band come when the plant slips, not when the tenant is in, so the plain spread inside and outside lease hours differs by chance alone, in either direction. What occupation leaves is smaller and finer: doors, people and blinds make a reading restless from one two minutes to the next. Measured as the distance of each reading from the mean of its two neighbours, a trend is about four and a half times as restless inside the lease hours of the zone it is really in as outside them (a log ratio of at least 1.47), and under every other zone's hours the figure is lower by at least 0.16. The lease hours are needed for the count anyway; the note never says what occupation does to a reading. tests/lease_check.py measures all of this against the sealed figures and will not pass otherwise: across 16 readings of the files the right one scores 50 and the best other 45, against a bar of 46.

</details>

## How it is verified

The verifier reads /app/out_of_band.csv with the csv module and compares each zone-week's hours with /tests/sealed.json, accepting a quarter of an hour either way; at most 4 of the 50 lines may be out. None of the agent's code is imported or run. The sealed figures exist only in the verifier image, where /tests is mode 700, and the agent's container never mounts it.

<details>
<summary>Full note</summary>

The quarter hour covers rounding at the band's edges after a unit conversion and nothing more: with the right trend the reference matches every sealed figure, and with a crossed trend every one of the 20 lines concerned misses by more than that, so none comes right by chance. No figure is attested anywhere in the agent's container and nothing has to balance; a sheet made from the point list as it stands is as self-consistent as the right one. The reward is binary and is written to /logs/verifier/reward.txt on every path; an empty container scores 0 and the reference scores 1. The generator is seeded and shipped.

</details>

<details>
<summary>Reference approach (spoiler)</summary>

solution/rebate.py reads only /app and writes /app/out_of_band.csv using the standard library. It reads each outstation file, tells a trend's unit from the middle of its readings and brings it to Celsius. For each trend it computes how far every reading stands from the mean of its two neighbours, and, for each zone's lease hours in turn, the log ratio of the middle such distance inside those hours to the middle outside them; trends are then matched to zones one to one, best fit first. With each zone served by the trend so found, it counts the occupied readings outside 20.0 to 24.0 degrees, two minutes apiece, by week. In the task's own container it gets 50 of 50.

</details>

## Layout

```
instruction.md          the brief - the only thing the agent is told
task.toml               artifacts, timeouts, resources, metadata
environment/            Dockerfile and data for the agent's container
solution/solve.sh       reference solution; scores 1 by construction
tests/                  verifier image, grading logic and ground truth
```

## Running it

With [harbor](https://github.com/laude-institute/harbor) installed and Docker running:

```bash
uv tool install harbor

harbor run -p . -a oracle -e docker   # reference solution - must score 1
harbor run -p . -a nop    -e docker   # nothing happens    - must score 0
```

Those two runs are the calibration the bundle has to satisfy: the oracle run substitutes
`solution/solve.sh` for the agent and must score `1`, the nop run does nothing and must
score `0`. Anything in between means the verifier is measuring the wrong thing.

## Reading this

`tests/` and `solution/` hold the answers. To try the task honestly, read only
`instruction.md` and `environment/`.

---

Part of a set of 38 tasks indexed at
[afterquery-data-annotation](https://github.com/helexdev1004/afterquery-data-annotation).
