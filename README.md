# elevenset — Computer Vision Technical Trial

**Role:** Computer Vision / Applied ML Intern
**Product:** elevenset — AI tennis coaching from phone videos
**Time:** Please cap this at **2 hours**. We mean it — see "How we respect your time" below.
**Environment:** CPU-only inference. Pretrained models welcome; no training needed.
**Tools:** Python + OpenCV / NumPy / FFmpeg, or anything equivalent you prefer.

> **New here?** This page is the brief. [CANDIDATE_GUIDE.md](CANDIDATE_GUIDE.md) walks you
> through setup, a suggested 2-hour plan, the metric, and the submission checklist.

---

## How we respect your time

We know open-ended take-home work is a big ask — especially if you have a
full-time job or family. So:

- **It's capped at 2 hours.** Do the first two hours and stop. We'd rather see two
  focused hours and an honest "here's what I'd do next" than a polished weekend project.
- **AI assistants are fine.** We only ask that you can explain your choices in a short
  follow-up call.

---

## Background

People film tennis on a phone — from behind the baseline, from the side, sometimes
in portrait. Other people are often in frame: the opponent, the next court, someone
walking past. We turn these clips into vertical (9:16) highlights that should stay on
**the player who uploaded**, and into stroke analysis.

Keeping the crop on the *right* player, reliably, across messy real-world footage is
the hard part. This trial is a small, real slice of that.

---

## The task

You'll receive:

- **Two short clips** (~20–30s). In one, the target player is mostly alone. In the
  other, **other people are also on court** — the opponent and a passer-by.
- **`targets.json`** — a bounding box marking the target player on the **first frame**
  of each clip.
- **`baseline.py`** — our current naïve approach: it centres the crop on whatever
  person it detects, with no notion of *which* person. It works on the easy clip and
  struggles on the crowded one.

### Choose your track — same 2 hours either way

- **Build track (default).** Make it work, measure it, note the cost — steps 1–3 below.
- **Design track (optional).** Spend the same 2 hours on **problem definition and
  experimental design** instead of a polished implementation. This is not extra work
  and not a lesser option — it's scored with a different emphasis (see the table under
  "What we're evaluating"). Pick it if you'd rather show how you frame a problem than
  how fast you can ship one.

You can also blend the two — build less, design more. Just tell us which you leaned towards.

**Build track — do one thing well:**

1. **Follow the right player.** Extend or replace the baseline so the 9:16 crop stays
   on the target identified in `targets.json`, including on the clip with other people
   in frame. Handle the target briefly leaving frame or being occluded, in whatever
   way you think is reasonable.
2. **Measure it — then argue with the measurement.** See "The metric" below. Report
   it before (baseline) vs after on both clips, and tell us where it lies.
3. **Note the cost.** Roughly how long does it run per minute of video, on what CPU?

**Design track — a 1–2 page write-up covering:**

1. **Problem definition.** What exactly does "follow the right player" mean? Which
   failures matter most to someone watching the highlight (wrong person, late crop,
   jitter, player cut off)? What's in scope, and what would you deliberately leave out?
2. **Experimental design.** Which metrics, against what reference, and how you'd get
   that reference cheaply. Which baselines and ablations. What result would change your mind.
3. **The metric** (below) — its failure patterns are scored on this track too.
4. *Optional:* one small experiment that tests a claim you make — e.g. run `baseline.py`,
   hand-check ten frames, and show one failure pattern on real footage.

If two hours runs out before it all works, that's expected — submit what you have and
tell us what you'd try next. **We read that as a strong signal, not a failure.**

---

## The metric — a starting point worth challenging

We'll give you a metric. It's deliberately simple, and it's deliberately flawed:

> **Crop-centre error** — per frame, the horizontal distance between the centre of the
> 9:16 crop and the centre of the target player's box, divided by the crop width.
> 0 means perfectly centred; above 0.5 means the player's centre is outside the crop.
> Report the mean over the clip.

Use it, and don't trust it. **The thing we score here is the metric's failure patterns.**

A **failure pattern** is a concrete situation where the metric and reality disagree:
the metric gives a good score to an output a viewer would call bad, or a bad score to
one a viewer would call good. A strong answer:

1. **Names** at least one failure pattern — of this metric, or of one you propose instead.
2. **Shows** it on the real clips — a timestamp, a frame, a number. Evidence beats argument.
3. **Proposes** what to measure instead or alongside, and what that costs (labels, compute, complexity).

One well-evidenced pattern beats a list of five guesses. Here's one for free, so you
know the shape we mean:

> **Circularity.** Score the crop against *your own tracker's* boxes and a tracker that
> follows the wrong person still scores perfectly — the metric is checking the system
> against itself. Tell us how you'd get an independent reference, and use one if you
> have time (even a few hand-checked frames counts).

---

## Getting started

```bash
python3.12 -m venv .venv && source .venv/bin/activate   # Python 3.10–3.12
pip install -r requirements.txt
# put the two clips you were sent into clips/ (next to targets.json)
python baseline.py clips/<clip>.mp4 out/<clip>_baseline.mp4 --log out/<clip>_baseline.json
```

The first run downloads the MoveNet model from TF-Hub, so you need internet once.
`--log` writes the baseline's per-frame crop position, which you can use as your
"before" numbers.

**`targets.json` format.** One entry per clip, keyed by the clip filename without `.mp4`.
Coordinates are integer pixels in the original video, origin top-left; `x, y` is the
box's top-left corner and `w, h` its size:

```json
{"<clip>": {"frame_index": 0, "x": 1312, "y": 402, "w": 118, "h": 296}}
```

---

## Submit

A repo (or zip) with:

**Build track**
- Your code + one-command run instructions.
- A 9:16 output for each clip.
- Your metric results (numbers; a plot is a bonus), before vs after.
- A short **README** (~half a page): what you did, what you measured and why,
  the metric's failure pattern(s) you found, the CPU cost, what failed, and what
  you'd do with another day.

**Design track**
- Your 1–2 page write-up (Markdown or PDF).
- Any code, frames or numbers behind your optional experiment.

Please don't include the clips themselves in your submission.

---

## What we're evaluating

| Dimension | Build track | Design track |
|---|---:|---:|
| Following the **correct** player, robustly | 30% | 10% |
| **Metric failure patterns** — named, evidenced, with a better alternative | 25% | 30% |
| Problem definition & experimental design | 10% | 35% |
| CPU-cost awareness | 15% | 10% |
| Code clarity & reproducibility | 10% | 5% |
| Judgement & honest communication | 10% | 10% |

We care more about how you reason about a messy problem than about a perfect result.

---

## Notes

- The clips are for this assessment only — please don't publish or share them.
- This is a hiring assessment, not production work.
- After you submit, we'll have a short walkthrough to talk
  through your approach and where you'd take it next.

Thanks for your time — we're genuinely looking forward to seeing how you think.
