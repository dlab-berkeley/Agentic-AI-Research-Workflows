# Workshop Spec: Agentic AI for Research Workflows

**Status:** Draft v2, revised after review. This is the internal spec — order of operations, timing, and design decisions — not the participant-facing materials. Curriculum files get written from this after sign-off.

**Format:** 2-hour hands-on workshop. Participants follow along on their own machines using the **Codex app** (not the CLI), free tier. Instructor demos from the front. Slides for the conceptual sections (1 and 3); everything else is live.

---

## Locked decisions

- **Dataset:** BTS On-Time Performance data, filtered to Bay Area departures (Origin ∈ {SFO, OAK, SJC}). Source: `https://transtats.bts.gov/PREZIP/` monthly zips. One month ≈ 17–19K Bay Area flights after filtering.
- **Participants build their own dataset live** — the agent downloads a raw month and filters it (section 4). No pre-staged repo clone; participants create a fresh project during the workshop.
- **Target:** `ArrDel15` (BTS's official flag: arrival ≥15 min late). Base rate 12–17% depending on month.
- **Metric: accuracy only.** No AUC in participant-facing material. Every accuracy is always reported next to the majority-class baseline ("always guess on-time"), which is what makes accuracy honest.
- **Common model (M1):** logistic regression; features = carrier, origin, dest, day-of-week, scheduled departure hour, distance; 75/25 stratified split, seed 0.

**Validated reference numbers** (single months, must be re-verified on the final chosen month):

| Month | Delay rate | M1 accuracy | M1 + weather | M1 + DepDelay | Majority baseline |
|---|---|---|---|---|---|
| Jan 2025 (dry) | 12.3% | 0.877 | 0.876 | 0.960 | 0.877 |
| Mar 2025 (wet) | 16.5% | 0.835 | 0.840 | 0.947 | 0.835 |

**⚠ Open design risk (section 7):** with accuracy as the only metric and weather as the only enhancement, the honest improvement is ~0–0.5 points — weather shifts delay probabilities but rarely pushes flights across the 50% line. Options, for discussion:

1. **Lean into it (recommended):** the flat number becomes the lesson — "the agent did exactly what we asked and the score didn't move; is the model useless or is the metric blunt?" Follow immediately with the risk-score framing: have the agent report the delay rate among the 100 flights it's most worried about (~40–50% vs. ~15% overall — this number moves impressively with weather and is fully intuitive). Accuracy stays the official metric; the risk-list is just a plain-language view of the same model.
2. **Choose a wet month:** the effect is real but weather-dependent (see Jan vs Mar). Pick the workshop month after checking precipitation; still expect ≤1 point of accuracy lift.
3. **Reintroduce the inbound-aircraft feature** (previously cut): "was this plane's incoming flight late?" gave the only big honest accuracy jump (0.835 → 0.881 in March). The concept is completely intuitive even though the wrangling is technical — and the wrangling is the agent's job. Cut for scope, but it remains the strongest fix if 1 feels too subtle.

---

## Pre-workshop (email, not workshop time)

- Install the Codex app + create an account (free tier); link the ChatGPT education-license page.
- Assume 30–50% won't have done it; section 2 absorbs them.
- Materials (prompt sheets, schema text, links) distributed via a short URL participants open in a browser — nothing to clone.

---

## Workshop flow

### 1. Hook + what is agentic programming? — slides (10 min)

**Goal:** Motivation and mental model before anyone touches the app.

- Slides (to be made later; keep general here): what agentic coding is; chatbot vs. coding agent (local file access, runs commands, persistent state); when to use which; the inspect → plan → approve → edit → run → verify loop.
- **Hook: TBD.** The "show the finished dashboard first" opening is on hold until a prototype dashboard exists that is genuinely good-looking and meaningful — build the prototype during materials development, then decide. Fallback hook: a 60-second live "most interesting plot in this data" ask, which lands almost as well and requires nothing.

### 2. Setup: app install, new project, AGENTS.md, permissions (15 min)

**Goal:** Everyone has a working agent in a fresh project of their own.

- (5–8 min) Codex **app** installed and signed in; helpers circulate. Early finishers get a warm-up prompt card ("ask the agent what it can and can't do in this folder").
- (3 min) Interface walkthrough: chat pane, how the agent proposes actions, where files land. **Permissions:** show the approval settings, explain the default (agent asks before running commands / editing files), and set the workshop norm — stay on ask-before-acting until you understand what it's doing.
- (4 min) **Create a new project:** make an empty folder, open it in Codex. Then create `AGENTS.md` via copy-paste text (provided in materials). Contents — the project norms that make the rest of the workshop deterministic:
  - Environment rule: which Python and how packages get installed (see open question below).
  - Data rules: data lives in `data/`; never modify raw downloads; state row counts after any filter.
  - Modeling rules: fixed seed, report accuracy alongside the always-on-time baseline.
  - Workflow rules: propose a plan before editing files; show the exact command run.
- **Open question (resolve during dry-run, before materials):** how does the agent handle package management on a clean machine? Does it reach for `uv`, `pip`, `venv`, or conda unprompted, and does that vary run to run? Recommended resolution: pin it in `AGENTS.md` (e.g., "use `uv run` with inline script dependencies" or "use pip install into a project venv") and verify on a fresh machine. The full install list must be known ahead of time — expected: pandas, scikit-learn, plotting library, streamlit (if chosen for dashboards) — so the venue wifi hit is predictable and front-loaded.

### 3. Motivate the problem: the dataset — slides (5 min)

**Goal:** Participants care about the question before seeing any data.

- "Will my flight out of SFO be delayed?" Establish the prediction target up front: classify arrival ≥15 min late.
- Introduce BTS: government data, real codebook, cryptic columns, lookup tables — real research data, on purpose.
- Scope: departures from SFO/OAK/SJC, one month.

### 4. Getting the data: manual vs. agent (10 min)

**Goal:** First live demonstration that the agent takes on real work — and that its work is inspectable.

- (3 min) **Manual first.** Instructor walks the BTS site: the download form, the 110-column selector, the zip, the codebook. Enumerate what a human would do: download, unzip, decipher columns, filter, save. Tedious, error-prone, undocumented.
- (5 min) **Delegate.** Participants use a provided prompt: download the month's prezip, unzip, filter to SFO/OAK/SJC departures, save to `data/`. Agent asks permission along the way (reinforces section 2).
- (2 min) **Expand the agent's steps.** Open the agent's action log and walk it: the exact `curl`/download step, the unzip, the filtering code it wrote, row counts before and after. Two points: the agent's work is *inspectable* — every step is recorded, unlike your own undocumented clicking — and *this is what you'll later turn into a pipeline*.
- **Wifi risk:** everyone pulling a ~29 MB zip from BTS simultaneously. Mitigation: instructor hosts a mirror of the exact zip at a short URL; the provided prompt tries the mirror first. Verify at dry-run.

### 5. EDA: guided plots, then a challenge (15 min)

**Goal:** Meet the target variable, see conversational analysis, then let participants loose — and converge everyone's data before modeling.

- (2 min) **Vague-prompt opener:** "Show me the most interesting plot in this data." A few volunteers screen-share results — everyone got something different. First stochasticity seed; also demonstrates the agent handles a hopelessly vague ask.
- (6 min) **Two guided plots** (only two; each earns its place):
  1. **Arrival delay distribution** (histogram of `ArrDelay`). Meets the raw target: heavy right tail, most flights early or on time. Motivates the 15-minute binarization — where `ArrDel15` comes from.
  2. **Delay rate by scheduled departure hour** (bar/line). The classic cascade result — 6 am flights are safe, 6 pm flights aren't. Intuitive, actionable, and quietly introduces M1's best feature.
- (5 min) **Challenge:** "Make one plot of your own that tells you something about delays." Share-outs. (Natural directions participants will find: carrier comparisons, destination effects, cancellation patterns, the delay-cause columns.)
- (2 min) **Data contract.** Copy-paste prompt (materials provide verbatim) that puts everyone's data in an identical format before modeling — the convergence point after the divergent EDA. Draft:
  > Create `data/flights_clean.csv` with exactly these columns: `FlightDate, Reporting_Airline, Origin, Dest, DayOfWeek, dep_hour` (defined as `CRSDepTime // 100`)`, Distance, DepDelay, ArrDelay, ArrDel15, Cancelled, Diverted`. Keep only departures from SFO, OAK, SJC. Do not drop any rows beyond that filter. Print the final row count and the delay rate (`mean of ArrDel15`).
  - Row count on screen: everyone should match the instructor's number exactly. First reproducibility checkpoint of the day.

### 6. Modeling I: open exploration → enforced common model (20 min)

**Goal:** Experience agent stochasticity, then converge and interpret accuracy honestly.

- (7 min) **Open-ended:** "I want to predict whether a flight will be delayed 15+ minutes. Ask your agent what approach and which variables it would use." Deliberately non-prescriptive.
- (5 min) **Share-out / discussion:** collect proposals from the room. Different variables, different models; some agents will propose `DepDelay` or other post-departure information. Two lessons: (a) same prompt, different answers — stochasticity is inherent; (b) the agent will cheerfully propose using information you couldn't actually have at prediction time — the human is the check. Don't fully resolve the leakage point yet; it returns with force in section 7's "maximize accuracy" beat.
- (5 min) **Enforce the common model.** Verbatim copy-paste prompt (draft; final text set during materials development):
  > Using `data/flights_clean.csv`, fit a logistic regression predicting `ArrDel15`. Use exactly these features and no others: `Reporting_Airline, Origin, Dest, DayOfWeek, dep_hour, Distance`. One-hot encode the categorical features. Drop rows with missing values in the features or target. Split the data 75/25 with stratification and `random_state=0`. Report: (1) test-set accuracy, (2) the accuracy of a model that always predicts "not delayed." Save the script as `model_v1.py` and do not change it afterward.
  - Room converges to the reference number (e.g., 0.835 for March). Expect small spread despite the prescriptive prompt — preprocessing latitude remains; materials should say "within ~0.01" and treat spread as a teachable reproducibility moment. Instructor keeps a reference implementation for arbitration.
- (3 min) **Group interpretation:** "83.5% accuracy — good, right?" Reveal the baseline: always guessing "on time" also scores 83.5%. The model learned nothing usable yet. Framing stays ML-free: "if you always guess on-time, you're right 5 times out of 6 — did we beat that?"

### 7. Modeling II: weather, and the maximize-accuracy stress test (20 min)

**Goal:** Improve the model with domain reasoning (weather); then let the agent off the leash and watch what it does to the metric.

- (3 min) **Group brainstorm, ML-free:** "What would you want to know to guess whether your flight will be late?" Weather will come up immediately. (If someone says "whether the incoming plane is late" — acknowledge it's excellent and note it as a take-home extension.)
- (8 min) **Weather join.** Agent pulls hourly weather for SFO/OAK/SJC from the Open-Meteo historical API (free, no key, works live) and joins on airport + date + scheduled departure hour; refit with weather features added. This is the workshop's "agent does real multi-source data work conversationally" moment: an API the participants have never heard of, discovered, fetched, and joined in one prompt.
- (3 min) **Confront the result:** accuracy barely moved (see open design risk, top of spec — resolve which framing before materials; recommended: the risk-score follow-up, where the agent reports the delay rate among its 100 highest-risk flights, which roughly triples the base rate and shows the weather model genuinely works even though "accuracy" won't say so).
- (6 min) **Stress test — "make the accuracy as good as possible."** Participants give the agent exactly that instruction, unconstrained, and watch. Expected behaviors to harvest from the room: grabbing `DepDelay` (accuracy leaps to ~0.95 — but now the model needs to know the flight already left late), or other post-departure columns if present; possibly threshold or evaluation tricks. Debrief: the agent optimized exactly what we asked for; nothing it did was malicious; *the metric was the misconception, and the agent baked it in*. This is the workshop's central cautionary lesson about agentic research, staged safely.

### 8. Deliverable: interactive dashboard (10 min)

**Goal:** The endpoint of agentic analysis can be a living research product, not a static PNG.

- Participants prompt the agent to assemble the session's plots + model into an interactive dashboard.
- **Format decision pending the prototype** (see section 1): self-contained Plotly HTML (zero install, double-click to open, easily shared) vs. Streamlit (more "real" dashboard, needs `streamlit` in the install list from section 2 and a local server — viable since installs are now front-loaded). Build one of each during materials development, pick the better-looking one, and that same prototype settles the section-1 hook question.
- Recommended interactivity (pick 2–3): destination dropdown → delay profile for that route; SFO/OAK/SJC toggle on the delay-by-hour chart ("should I fly Oakland?"); hour × day-of-week heatmap; (stretch) carrier + destination + hour → predicted delay probability from the fitted model.
- Everyone's dashboard will differ — by now that's an expected property of the process, not a surprise.

### 9. Reproducibility: spec sheet + fresh-thread rebuild (20 min)

**Goal:** The workshop's thesis — conversational decisions must be transformed into a reproducible artifact.

- (5 min) **Spec sheet.** Prompt the agent: write a detailed `WORKFLOW.md` documenting everything — data source and filters, the data contract, every cleaning decision, feature definitions (including the weather join keys), model specification, evaluation protocol, dashboard contents. Push participants to check it against what they learned in section 7: are the silent decisions written down?
- (12 min) **Fresh-thread rebuild.** Start a **new** Codex thread — make it land that the agent that did the work is gone; only the files remain. Prompt: "Using `WORKFLOW.md`, build this analysis as a reproducible pipeline of sequentially numbered Python scripts." Target shape, deliberately unfancy — no package managers beyond the section-2 norm, no software-engineering ceremony:
  1. `01_download.py` — fetch raw data
  2. `02_clean.py` — filter + data contract
  3. `03_weather.py` — fetch weather, join
  4. `04_train.py` — fit the models, fixed seed
  5. `05_evaluate.py` — accuracy vs. baseline, written to a results file
  6. `06_dashboard.py` — regenerate the dashboard
- Run it end to end; check the numbers match the session. If they don't, *that's the lesson*: the spec was incomplete — which is exactly what happens when analysis decisions live only in a chat transcript. (`AGENTS.md` from section 2 quietly pays off here: the fresh thread still obeys the project norms.)
- (3 min) **Wrap-up + exit ticket:** one task where an agent would help your research; one where you shouldn't use one without approval; one instruction that belongs in your own project's AGENTS.md; one dataset-specific danger your agent should check.

---

## Timing summary

| # | Section | Est. |
|---|---|---|
| 1 | Hook + concepts (slides) | 10 |
| 2 | Setup: app install, project, AGENTS.md, permissions | 15 |
| 3 | Dataset motivation (slides) | 5 |
| 4 | Getting the data (manual → agent → inspect steps) | 10 |
| 5 | EDA: 2 guided plots + challenge + data contract | 15 |
| 6 | Modeling I: exploration → common model | 20 |
| 7 | Modeling II: weather + maximize-accuracy stress test | 20 |
| 8 | Dashboard | 10 |
| 9 | Spec sheet + fresh-thread rebuild + wrap-up | 20 |
| | **Total** | **125** |

~5–15 min over a true 2-hour room after realistic slippage (mostly section 2). Trims in preference order:

1. Launch the dashboard build as a background task during section 7's debrief discussion — agents work while humans talk (–5).
2. Cut the EDA challenge share-out to two volunteers (–3).
3. Make the dashboard instructor-demo-only, prompt provided as take-home (–5, only if badly behind).
4. Protect sections 7 and 9 — they carry the thesis. Cut spectacle before reflection or reproducibility.

---

## Open items before materials are written

1. **Resolve the section-7 accuracy-flatness risk** (options at top of spec). This decision shapes the section-7 materials more than anything else.
2. **Build the dashboard prototype** (Plotly HTML and/or Streamlit) — settles both the section-8 format and the section-1 hook.
3. **Dry-run the package-management question** on a clean machine; pin the answer in the `AGENTS.md` starter text, and finalize the ahead-of-time install list.
4. **Pick the workshop month** (prefer a wet one; verify reference numbers on it) and set up the download mirror.
5. **Record the Codex model/version at dry-run time** and re-verify the reference numbers the week of the workshop — agent behavior drifts across model updates.
6. Take-home extensions to draft alongside materials: the inbound-aircraft feature ("was your plane's incoming flight late?" — the single strongest honest feature, cut for scope), and the scramble-the-data debugging exercise.
