# Assignment 5 — Recursive Self-Improvement (RSI) on a Real Problem: Flaky-Test Repair

**Course:** RSI Masterclass (October 2026 edition) · **Section:** 10 · Read and Rebuild October 2026 Research
**Student:** _<your name>_ · **Repo:** _<your GitHub repo URL>_ · **Submission:** _<website / repo URL>_

> Each part below has its own code, run log and YouTube video. Replace every `PASTE_VIDEO_LINK_HERE` with your link.

---

## 1. Problem area

**A self-improving agent that repairs flaky tests.**

A flaky test passes and fails on the same code. The causes are timing, shared state, test order and randomness. Repairing one is a good RSI problem because:

- **Edits are cheap to propose and expensive to check.** Each candidate fix needs many repeated runs, so there is a real case for scoring edits before evaluating them (Budget RSI).
- **Failures repeat in structured ways.** The agent re-applies the same fix, flips between two fixes, or sees a result that contradicts what it expected. These are the anomalies second-order RSI looks for (SO-RSI).
- **The best workflow changes as bottlenecks change.** Early on, diagnosis is the slow step. Later, verification is (RSIFlow).

The task is small enough to run on a laptop with synthetic data, in line with the course rule that laptop adaptations preserve the idea, not the scale or headline result of the paper.

**What is improved:** the _improver_ (the research and repair procedure), not only the repaired tests. This matches the course's three objects: task model, research skill and improver.

---

## 2. The three RSI methods demonstrated

| # | Method | Paper (course reference) | Object that changes | Core idea |
|---|--------|--------------------------|---------------------|-----------|
| A | **Budget RSI** | arXiv:2610.29121 | The edit-selection step | A linear readout predicts an edit's held-out performance from a small steering split, so most edits are never fully evaluated. |
| B | **Second-Order RSI (SO-RSI)** | arXiv:2610.05701 | The diagnostic procedure | Investigate _why_ failures happen before patching, using probes for recurrence, opposing edits and expectation mismatch, plus an inquiry memory. |
| C | **RSIFlow** | arXiv:2610.21791 | The workflow itself | A meta-agent re-orchestrates the workflow as the bottleneck moves. |

Self-HPO (arXiv:2610.22791) is the fallback if you would rather use alternating harness/policy optimization for method C.

How they fit together on one problem:

```
flaky test ──► [B] SO-RSI diagnoses the failure pattern
          └──► [A] Budget RSI scores candidate fixes cheaply, picks the best
          └──► [C] RSIFlow rewires the pipeline as the bottleneck shifts
```

---

## 3. Codelabs implemented (Section 10 of COURSE-MAP.md)

The course map lists these labs under theme 10. Foundation themes 00–09 come first.

| Part | Labs | Method | Video |
|------|------|--------|-------|
| 0 | 09.01–09.07 (foundation: three objects, fixed vs revised improver, bounded generations, honest claim) | Setup | PASTE_VIDEO_LINK_HERE |
| 1 | 10.03 Run the Budget RSI demo · 10.04 Analyze the scoring mechanism · 10.05 Extend to your own edits | Budget RSI | PASTE_VIDEO_LINK_HERE |
| 2 | 10.06 Run the SO-RSI demo · 10.07 Analyze diagnostic probes · 10.08 Apply to a debugging task | SO-RSI | PASTE_VIDEO_LINK_HERE |
| 3 | 10.12 Compare workflow adaptation · 10.13 Simulate the RSIFlow meta-agent | RSIFlow | PASTE_VIDEO_LINK_HERE |
| 4 | 11.02 Recursive comparison + 10.01–10.02 claim audit (combined result) | All three | PASTE_VIDEO_LINK_HERE |

Part 0 and Part 4 are optional extras. The assignment requires one video per RSI method.

---

## 4. Repository layout

```
.
├── README.md
├── requirements.txt
├── common/
│   ├── flaky_env.py          # synthetic flaky-test environment (seeded, laptop-scale)
│   ├── edits.py              # candidate repair edits + 8 diff features
│   └── metrics.py            # pass-rate, cost, matched-budget comparison
├── part0_foundation/         # three-object diagram, fixed-improver baseline
├── part1_budget_rsi/         # Labs 10.03 – 10.05
│   ├── demo.py
│   ├── scorer.py             # linear readout on steering split
│   └── results/
├── part2_so_rsi/             # Labs 10.06 – 10.08
│   ├── demo.py
│   ├── probes.py             # recurrence, opposing-edits, mismatch
│   ├── inquiry_memory.py
│   └── results/
├── part3_rsiflow/            # Labs 10.12 – 10.13
│   ├── meta_agent.py
│   ├── workflow.py
│   └── results/
├── part4_combined/           # recursive comparison, matched resources
└── docs/                     # diagrams, claim statement, limits
```

---

## 5. Part-by-part walkthrough

### Part 1 — Budget RSI (Labs 10.03–10.05)

**Idea.** Don't fully evaluate every edit. Reserve a steering split, extract features of each edit's diff, and fit a linear model:

```
score = wᵀ · [parent_rate, diff_features]
```

**Steps to show on video**
1. Run the course demo (`12_october_2026_papers/01_budget_rsi/demo.py`, under 5 minutes) to see the mechanism.
2. Walk through `scorer.py`: the steering split, the 8 diff features, and the linear fit.
3. Apply it to flaky-test fixes. Candidate edits include adding a `sleep`, adding a retry, isolating state, fixing a seed and mocking time. The scorer ranks them.
4. Compare cost: full evaluation of all edits vs. scored selection.

**What to report:** correlation between predicted and true pass-rate, the evaluations used by each approach, and which diff features matter most.
**Show execution:** `python part1_budget_rsi/demo.py`

**Video:** PASTE_VIDEO_LINK_HERE

---

### Part 2 — Second-Order RSI (Labs 10.06–10.08)

**Idea.** Before patching again, ask why the last patches failed. Three structural anomalies trigger probes:

| Anomaly | Meaning in flaky-test repair | Probe |
|---------|------------------------------|-------|
| Recurrence | The same fix (e.g. "add sleep") is tried repeatedly and still fails | Look for the hidden constraint, such as a race on shared state |
| Opposing edits | The agent alternates between "add retry" and "remove retry" | Find the two constraints that conflict |
| Expectation mismatch | The test passes locally but fails in CI | Update assumptions about the environment |

**Inquiry memory** accumulates causal evidence across rounds, so later edits are structural instead of symptom patches.

**Steps to show on video**
1. Run the course demo (`12_october_2026_papers/02_second_order_rsi/demo.py`, under 2 minutes).
2. Walk through `probes.py` and `inquiry_memory.py`.
3. Run first-order patching vs. second-order inquiry on the same flaky tests.
4. Show a trace in which a probe fires and changes the next edit.

**Show execution:** `python part2_so_rsi/demo.py`

**Video:** PASTE_VIDEO_LINK_HERE

---

### Part 3 — RSIFlow (Labs 10.12–10.13)

**Idea.** An evolving meta-agent adapts workflow orchestration to the current bottleneck.

**Steps to show on video**
1. Define the workflow stages: detect → diagnose → propose → score → verify.
2. Measure per-stage time and failure share each round to find the bottleneck.
3. The meta-agent re-orders, merges, skips or parallelizes stages. For example, if verification dominates, it leans more on the Budget RSI scorer.
4. Compare a fixed workflow against the adaptive one at matched budget.

**Show execution:** `python part3_rsiflow/meta_agent.py`

**Video:** PASTE_VIDEO_LINK_HERE

---

### Part 4 — Combined recursive comparison (optional)

Compare two improvers under identical conditions (Lab 09.05 / 11.02):

- **I0:** fixed improver (plain generate-and-test repair).
- **I1:** revised improver (Budget RSI scoring + SO-RSI diagnosis + RSIFlow orchestration).

Controls: same seeds, same total evaluation budget, same test set, and a hard cap on generations (Lab 09.06).

**Video:** PASTE_VIDEO_LINK_HERE

---

## 6. How to run

```bash
git clone <your-repo-url>
cd <your-repo>
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

python part1_budget_rsi/demo.py
python part2_so_rsi/demo.py
python part3_rsiflow/meta_agent.py
python part4_combined/compare.py
```

Each script prints its results and writes them to its `results/` folder. Keep the logs for the videos.

---

## 7. Results

_Fill this in after you run the code. Do not copy numbers from the papers._

| Method | Metric | Baseline | With method | Notes |
|--------|--------|----------|-------------|-------|
| Budget RSI | Evaluations used | | | |
| Budget RSI | Predicted vs true pass-rate (r) | | | |
| SO-RSI | Tests repaired within N rounds | | | |
| RSIFlow | Time to fix (fixed vs adaptive) | | | |
| Combined | Fixed improver I0 vs revised I1, same budget | | | |

---

## 8. Honest claim and limits

- The environment is **synthetic and seeded**, so results show the mechanism and not real-world performance.
- This is a **laptop adaptation** of each paper's idea. It does not reproduce the paper's scale, benchmarks or headline numbers.
- Untested assumptions: transfer to real CI systems, other languages, and larger edit spaces.
- _Add your own limits after you see the results._

---

## 9. AI-assistance disclosure

AI coding tools were used to help write code. All codelabs were walked through and explained step by step in the videos. _<Name the tools you used.>_

---

## 10. Video index

| Part | Topic | Link |
|------|-------|------|
| 0 | Setup and foundations | PASTE_VIDEO_LINK_HERE |
| 1 | Budget RSI | PASTE_VIDEO_LINK_HERE |
| 2 | Second-Order RSI | PASTE_VIDEO_LINK_HERE |
| 3 | RSIFlow | PASTE_VIDEO_LINK_HERE |
| 4 | Combined comparison | PASTE_VIDEO_LINK_HERE |

## 11. References

- Course map: https://github.com/dlmastery/simple-coding-harness/blob/main/COURSE-MAP.md
- Research studio (theme 10): `10_research_studio/README.md` in the course repo
- Budget RSI — arXiv:2610.29121
- Second-Order RSI — arXiv:2610.05701
- RSIFlow — arXiv:2610.21791
- Self-HPO — arXiv:2610.22791
