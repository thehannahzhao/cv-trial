# Candidate guide

The [README](README.md) says **what** we're asking. This guide is the **how**: getting set
up, planning your three hours, working with the metric, and what to send back.

---

## 1. Timeline

| Step | When |
|---|---|
| You receive the repo link + the two clips | Day 0 |
| Submit | Within [7] days of receiving the clips — pick any 3 hours that suit you |
| Walkthrough call (30 min) | We'll book it within [3] working days of your submission |

Questions at any point: **[contact email]**. Asking questions never costs you points —
clarifying an ambiguous problem is part of the job.

---

## 2. Before the clock starts (not part of your 3 hours)

Setup shouldn't eat into your time. Do this first, at your own pace (~15–20 min):

```bash
git clone <repo link> cv-trial && cd cv-trial
python3.12 -m venv .venv && source .venv/bin/activate     # Python 3.10–3.12
pip install -r requirements.txt
# copy the two clips you were sent into clips/
python make_targets.py     # box the target player on frame 0 of each clip -> targets.json
python baseline.py clips/<clip>.mp4 out/<clip>_baseline.mp4 --log out/<clip>_baseline.json
```

**Marking the target.** The README's "Who is the target?" table says which player to follow
in each clip. `make_targets.py` opens frame 0: drag a tight box around that player's whole
body and press Enter. Re-run it on one clip (`python make_targets.py clips/<clip>.mp4`)
if you want to redo a box. Not sure which player we mean? Email us before you start.

Then run the baseline on **both** clips and watch the outputs. You should see it do fine on
one and drift onto the wrong person on the other — that's the problem you're solving.

**If something goes wrong**

| Symptom | Fix |
|---|---|
| `No module named 'pkg_resources'` | `pip install "setuptools<81"` (already pinned in `requirements.txt`) |
| Model download hangs or fails | First run needs internet to fetch MoveNet from TF-Hub; retry, or check a proxy/VPN |
| `make_targets.py` window doesn't appear | It needs a desktop session (not SSH/headless). On macOS, check the window isn't behind your terminal |
| `pip` can't find `tensorflow` | Check your Python version is 3.10–3.12 |
| Output `.mp4` won't play | Open it in VLC, or re-encode: `ffmpeg -i in.mp4 -c:v libx264 out.mp4` |

Still stuck after 15 minutes? Email us — that's a bug in our setup, not a test of yours.

---

## 3. Pick a track

Both tracks are the same 3 hours, and neither is "the real one".

- **Build track** if you'd enjoy getting a tracker working on real footage and measuring it.
- **Design track** if you'd rather show how you'd frame the problem, choose metrics and
  design experiments before anyone writes much code.
- **Blend** if you want: build a little, design more. Just say which way you leaned.

---

## 4. A suggested 3-hour plan

These are suggestions, not rules. The point is to leave time to measure and write —
the most common mistake is coding until minute 175.

**Build track**

| Time | Focus |
|---|---|
| 0:00–0:20 | Watch both clips frame by frame. Note *when* and *why* the baseline fails. |
| 0:20–1:50 | Make the crop follow the target. Get the crowded clip working before polishing. |
| 1:50–2:30 | Metric before vs after on both clips. Hunt for one failure pattern of the metric. |
| 2:30–3:00 | Time a run, write the README, stop. |

**Design track**

| Time | Focus |
|---|---|
| 0:00–0:30 | Watch both clips and the baseline outputs. List every way the output can be "wrong". |
| 0:30–1:20 | Problem definition: which failures matter most to a viewer, and what's out of scope. |
| 1:20–2:30 | Experimental design + metric critique. Optionally, one small experiment as evidence. |
| 2:30–3:00 | Tighten the write-up to 1–2 pages, stop. |

---

## 5. Working with the metric

**Crop-centre error** for frame *i*:

```
crop_centre  = crop_x + crop_w / 2          # from your pipeline (baseline: --log JSON)
target_centre = ref_x + ref_w / 2           # from a reference box for the target
error = |crop_centre - target_centre| / crop_w
```

The hard part is `ref_x` — a reference that isn't your own tracker. The cheapest honest
option is to hand-check some frames. This snippet shows every Nth frame; drag a box
around the target and press Enter (press `c` to skip a frame where they're not visible):

```python
import cv2, json, sys
path, step = sys.argv[1], int(sys.argv[2]) if len(sys.argv) > 2 else 30
cap, refs, i = cv2.VideoCapture(path), [], 0
while True:
    ok, frame = cap.read()
    if not ok:
        break
    if i % step == 0:
        x, y, w, h = cv2.selectROI(f"frame {i}", frame)
        cv2.destroyAllWindows()
        if w and h:
            refs.append({"frame_index": i, "x": x, "y": y, "w": w, "h": h})
    i += 1
json.dump(refs, open(path.rsplit("/", 1)[-1] + ".refs.json", "w"), indent=1)
```

Twenty well-chosen frames is plenty. Include the hard moments — people crossing, the
target near the edge — not just easy ones, and say how you chose them.

Then **argue with the number**. A *failure pattern* is a case where the metric and a
viewer disagree. Find one, show it with a timestamp or frame, and propose what you'd
measure instead. That's the scoring point.

---

## 6. What good looks like

**Strong submissions usually:**
- Show the baseline's failure first, then their fix, with the same metric on both.
- Pick one approach and explain why over the obvious alternatives.
- Report a failure honestly ("the crop swaps to the opponent at 0:14 — here's why").
- Give CPU timing as *seconds of processing per second of video*, with the CPU model.

**Common pitfalls:**
- Spending all three hours on the tracker and reporting no numbers.
- Grading the crop against your own tracker's boxes (see "Circularity" in the README).
- Only reporting a mean — one bad 2-second stretch can hide inside a good average.
- Code you can't explain in the walkthrough. AI assistants are fine; unexplained code isn't.

---

## 7. FAQ

**Can I use any model or library?** Yes — any open-source pretrained model or library
(e.g. a person detector, a tracker, re-identification features). Note its licence in your README.

**Do I have to keep `baseline.py`?** No. Extend it, replace it, or ignore it — but
report its numbers as your "before".

**Can I use a GPU?** Develop however you like, but report timing on CPU.

**What should happen when the target leaves the frame or is hidden?** Your call. Choose a
behaviour, explain why, and show what it looks like.

**My `targets.json` box isn't pixel-perfect — does that matter?** No. It only tells your
tracker *who* to follow. We judge whether you follow the right person, not your box.

**The brief seems ambiguous about X.** Make a reasonable assumption, write it down, and
move on — or email us. Both are fine.

**What if I go over three hours?** Please try not to — we really do compare everyone at
about three hours. If you do, just tell us roughly how long you spent; honesty matters more.

---

## 8. Submission checklist

**Build track**
- [ ] Code + one command to reproduce your outputs
- [ ] Your `targets.json`
- [ ] A 9:16 output for each clip
- [ ] Metric results, before vs after, on both clips
- [ ] At least one metric failure pattern, with evidence
- [ ] CPU timing + the CPU model you ran on
- [ ] README (~half a page): approach, results, failures, next steps

**Design track**
- [ ] 1–2 page write-up: problem definition, experimental design, metric critique
- [ ] Any code / frames / numbers from your optional experiment

**Both**
- [ ] The clips are **not** included (`.gitignore` already excludes them — check before you push)
- [ ] Sent to **[contact email]** as a repo link (private repo: add **[GitHub username]**) or a zip

---

## 9. The walkthrough (30 min)

- **10 min** — you walk us through what you did and why. No slides needed; your README is enough.
- **15 min** — we discuss: what you'd change with more time, how it'd behave on other footage.
- **5 min** — your questions for us.

No live coding.

---

## 10. Data & privacy

- The clips are for this assessment only. Please don't publish, share, or commit them.
- Delete them when the process is finished.
