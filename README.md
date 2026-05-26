# Offshore wind grid design — KIRO challenge with RTE

Simulated-annealing heuristic for an **offshore wind grid topology design**
problem, solved during a **KIRO** academic optimisation challenge in
partnership with **RTE** (Réseau de Transport d'Électricité, the French
transmission system operator). The task is to choose substation locations,
substation types, and inter- / onshore cable types so that the total cost
(capital + expected production loss from failure scenarios) is minimised.

> Also submitted as part of the *Recherche Opérationnelle* (Operations
> Research) course.

Problem statement: [`KIRO sujet.pdf`](KIRO%20sujet.pdf).

---

## 1. The problem

Each JSON instance (`toy.json`, `small.json`, `medium.json`, `large.json`,
`huge.json`) describes:

- **`wind_turbines`** — turbines with `(x, y)` coordinates.
- **`substation_locations`** — candidate offshore substation sites
  `(x, y)` to choose from.
- **`substation_types`** — substation hardware options, each with
  `(cost, rating, probability_of_failure)`.
- **`substation_substation_cable_types`** — cable options for
  substation-to-substation links: `(rating, variable_cost, fixed_cost)`.
- **`land_substation_cable_types`** — cable options for the
  substation-to-shore links: same fields + `probability_of_failure`.
- **`wind_scenarios`** — discrete scenarios `(power_generation,
  probability)` modelling stochastic production.
- **`general_parameters`** — global cost weights, curtailment penalty, etc.

A **solution** must decide:

1. Which subset of `substation_locations` to actually build (and which
   `substation_type` for each).
2. Which `substation_substation_cable_type` connects each substation
   to a hub (and the routing).
3. Which `land_substation_cable_type` brings each hub to shore.
4. Which turbine connects to which substation.

The objective is **expected total cost** = capital + expected
curtailment-loss across `wind_scenarios` weighted by their probability
and accounting for cable / substation failure probabilities.

## 2. The approach: simulated annealing

The decision space is mixed-discrete: subset selection + categorical
choices + assignments. Simulated annealing handles all three with a
single move set:

1. **Initial solution**: greedy — connect each turbine to its nearest
   chosen substation, use the cheapest cable type that satisfies the
   rating constraint.
2. **Neighbourhood**: at each step, randomly choose one of:
   - swap two turbines between substations,
   - upgrade / downgrade a substation type,
   - upgrade / downgrade a cable type,
   - toggle a substation location on/off (with reassignment of its
     turbines to a neighbouring substation).
3. **Acceptance**: Metropolis criterion
   `accept(Δcost) = min(1, exp(-Δcost / T))`.
4. **Cooling**: geometric `T ← α · T`.

The implementation is in
[`probing_code.ipynb`](probing_code.ipynb). The notebook loads an
instance, builds the cost evaluator (including the
expected-curtailment Monte-Carlo sum over scenarios), runs SA, and
emits the best topology found.

## 3. Why simulated annealing here

The cost function is **non-convex and non-decomposable** — turbine
assignments couple substation choices, which couple cable choices, which
couple expected-curtailment-loss across stochastic scenarios. MILP
formulations exist but become slow on the `large` / `huge` instances
under the contest time budget. SA gives a good cost / time trade-off
and is robust to the multiple neighbourhood moves above.

## 4. Running

```bash
pip install jupyterlab tqdm numpy
jupyter lab
```

Open `probing_code.ipynb`. The first cell hardcodes the instance file
name; change `'KIRO-medium.json'` to any of the bundled `*.json`
files. The notebook has cell outputs committed.

## 5. What this is *not*

- **Not a production grid planner** — real offshore wind grid design
  involves AC vs DC trade-offs, marine-route constraints, environmental
  impact assessments, etc. None of those are in this problem.
- **Not a research contribution** — SA is textbook (Kirkpatrick et al.,
  1983); the contribution is the cost evaluator and the move-set design.

## 6. Acknowledgements

Problem statement, JSON instances, and scoring rules belong to the KIRO
organisers and RTE. The repo contains only my solver code, the public
instances, and the public problem-statement PDF.
