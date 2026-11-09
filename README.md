# elevenset — Computer Vision Technical Trial

**Role:** Computer Vision / Applied ML Intern
**Product:** elevenset — AI tennis coaching from phone videos
**Time:** Please cap this at **2 hours**. We mean it — see "How we respect your time" below.
**Environment:** CPU-only inference. Pretrained models welcome; no training needed.
**Tools:** Python + OpenCV / NumPy / FFmpeg, or anything equivalent you prefer.

---

## How we respect your time

We know open-ended take-home work is a big ask — especially if you have a
full-time job or family. So:

- **It's capped at 2 hours.** Do the first two hours and stop. We'd rather see two
  focused hours and an honest "here's what I'd do next" than a polished weekend project.
- **Prefer to skip the take-home?** Tell us — we'll do a **60-minute live paired
  session** instead, where we work through the same problem together. No disadvantage
  either way; some of our strongest candidates choose this.
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

**Do one thing well, in ~2 hours:**

1. **Follow the right player.** Extend or replace the baseline so the 9:16 crop stays
   on the target identified in `targets.json`, including on the clip with other people
   in frame. Handle the target briefly leaving frame or being occluded, in whatever
   way you think is reasonable.
2. **Measure it honestly.** Define at least one quantitative metric for "is the right
   player well-framed?" (crop-centre error is a natural start; stability and
   identity-correctness are good too). Report before vs after on both clips.
   > Important: scoring your crop against *your own tracker's* boxes is circular — wrong
   > tracking still scores perfectly. Tell us how you'd get an independent reference, and
   > use one if you have time (even a few hand-checked frames counts).
3. **Note the cost.** Roughly how long does it run per minute of video, on what CPU?

If two hours runs out before it all works, that's expected — submit what you have and
tell us what you'd try next. **We read that as a strong signal, not a failure.**

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

- Your code + one-command run instructions.
- A 9:16 output for each clip.
- Your metric results (numbers; a plot is a bonus), before vs after.
- A short **README** (~half a page): what you did, what you measured and why,
  the CPU cost, what failed, and what you'd do with another day.

Please don't include the clips themselves in your submission.

---

## What we're evaluating

| Dimension | Weight |
|---|---:|
| Following the **correct** player, robustly | 30% |
| Measurement — including not grading yourself | 30% |
| CPU-cost awareness | 15% |
| Code clarity & reproducibility | 15% |
| Judgement & honest communication | 10% |

We care more about how you reason about a messy problem than about a perfect result.

---

## Notes

- The clips are for this assessment only — please don't publish or share them.
- This is a hiring assessment, not production work.
- After you submit (or do the live session), we'll have a short walkthrough to talk
  through your approach and where you'd take it next.

Thanks for your time — we're genuinely looking forward to seeing how you think.
