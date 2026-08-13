repo roadmap and notes


RCS v0.1 — Local Execution Roadmap
Phase 0 — Create the repository skeleton

Set up:

runtime-control-system/
├── README.md
├── requirements.txt
├── rcs/
│   ├── __init__.py
│   ├── config.py
│   ├── environment.py
│   ├── generators.py
│   ├── telemetry.py
│   ├── governor.py
│   ├── memory.py
│   └── evaluation.py
├── tests/
│   ├── test_telemetry.py
│   ├── test_governor.py
│   ├── test_memory.py
│   └── test_invariants.py
├── run_p1_constraints.py
├── run_p2_stochastic.py
├── run_p3_adversarial.py
├── visualize.py
└── data/
    ├── raw/
    └── processed/

Don't build YAML yet.

Don't build an LLM adapter.

Don't build fancy abstractions.

Phase 1 — Build the deterministic world
environment.py

Implement the actual ground-truth environment.

Two topologies:

Topology A

asymmetric
differentiated paths
obvious G1 failure
G2 drift loop
G3 legitimate detour
G4 hidden-resource trajectory

Topology B

symmetric/interchangeable paths
deliberately weaker structural differentiation

The environment should know things the controller doesn't.

Especially G4's hidden resource state.

Critical invariant

Telemetry must receive an explicitly constructed observation packet.

It must never receive the environment object.

That makes accidental leakage much harder.

Phase 2 — Implement frozen generators
generators.py

Implement:

G1 — Obvious Failure

Immediately violates a hard constraint.

G2 — Gradual Drift

Enters a locally valid but globally unproductive attractor.

G3 — Legitimate Detour

Temporarily worsens objective distance while preserving structural integrity and eventually succeeds.

G4 — Hidden Resource

Trajectory remains structurally valid while an unobservable resource is consumed.

Important: generators don't know:

Governor state
α
τ
checkpoints
Historical State
telemetry
rollback decisions

They simply generate transitions.

Freeze them after the deterministic suite is validated.

Phase 3 — Telemetry
telemetry.py

Implement:

ObservationPacket
TelemetrySnapshot
observation stream
packet loss
current observation status
window estimate
α
current-step stability
structural metrics

Initially, use full observability.

Don't add stochastic loss yet.

Unit tests first

Test:

normal observation
UNKNOWN packet
all-UNKNOWN window
mixed observation window
current UNKNOWN + stable historical window
α calculation
current-step recovery predicate

Especially:

UNKNOWN must never enter α/velocity history.

Phase 4 — Governor
governor.py

Implement the frozen FSM:

GREEN
  ↓
YELLOW
  ↓
RED
  ↓
BLACK
  ↓
ORANGE
  ↓
GREEN

with uncertainty states:

GREEN_UNCERTAIN
YELLOW_HOLD
RED_HOLD

Core invariants:

Escalation

Dynamic τ responds to dα/dt.

Failure

τ is captured at the RED transition.

Restoration

recovery_tau is immutable until recovery succeeds.

Recovery

Only:

OBSERVED + current_stable

increments recovery_streak.

UNKNOWN
doesn't affect α history
doesn't affect velocity
doesn't count toward recovery
cannot constitute recovery evidence
Phase 5 — Historical State + Checkpoints
memory.py

Implement two completely separate things:

Checkpoint Store

checkpoint_id
state
trajectory position
metadata

Historical State

TransitionSignature
veto ledger

Exact signature equality only.

No similarity.

No embeddings.

No fuzzy matching.

No generator modification.

The Governor can ask:

“Is this branch signature vetoed?”

The Generator itself remains oblivious.

Phase 6 — Write the invariant tests

This is the real gate.

Before running experiments, all of these should pass.

Telemetry
UNKNOWN doesn't pollute history.
stale window doesn't override current UNKNOWN.
α is deterministic.
current stability is component-wise.
Governor
GREEN → YELLOW.
YELLOW → RED.
RED → BLACK.
BLACK → ORANGE.
ORANGE recovery requires fresh evidence.
recovery τ is latched.
recovery τ cannot move.
UNKNOWN freezes recovery.
UNKNOWN cannot create recovery evidence.
rapid degradation produces shorter escalation τ.
Memory
exact signature matches.
different signature doesn't match.
invalidated branch doesn't modify generator.
Recovery Trap

The old stable history must not allow premature ORANGE → GREEN.

Moving Goalpost Test

Recovery τ must not change during recovery.

Leakage Test

G4's hidden resource must be completely inaccessible to telemetry.

This one is particularly important.

Phase 7 — Deterministic baseline

Now run Phase P0, essentially a sanity experiment.

No packet loss.

No Monte Carlo.

No stochasticity.

Run:

G1
G2
G3
G4

on:

Topology A
Topology B

with Governor OFF and ON.

The question isn't yet “does RCS win?”

It's:

Does the apparatus behave exactly as specified?

You should be able to inspect individual trajectories manually.

Phase 8 — P1: Hidden Constraints

Now introduce partial constraint visibility.

Keep transitions deterministic.

Something like:

100%
80%
60%
40%
20%

But the generator/environment stays unchanged.

Only what Telemetry can see changes.

For every condition record:

detection
divergence latency
recovery latency
false rollback
compute steps
observation coverage
outcome class

This gives you the first observability envelope.

Phase 9 — P2: Stochastic Observation

Now introduce packet loss.

Two missingness processes:

Uniform

Independent Bernoulli observation:

P(observe)=p

Burst

Same mean observation probability, but correlated missingness.

For example:

OBS OBS DROP DROP DROP OBS OBS

versus:

OBS DROP OBS OBS DROP OBS DROP

Sweep:

p = 1.0
0.9
0.8
...
0.1

And use many seeds.

Don't compare unmatched observation rates.

Phase 10 — Fixed vs Dynamic τ

Run the same trajectories under:

Controller A

Fixed recovery τ.

Controller B

Dynamic escalation τ + latched recovery τ.

Everything else identical.

This is where we test whether adaptive τ actually improves the tail, rather than simply producing more aggressive intervention.

Record:

P50
P90
P95
P99
false rollback
detection rate
indeterminate rate
compute cost
Phase 11 — P3: G4 adversarial experiment

Now isolate G4.

Run:

T1

Revisit + oscillation

T2

T1 + observable constraint churn

T3

T2 + temporal progress

T4

Full telemetry

And separately test:

G4-linear
G4-quadratic
G4-exponential

The crucial question:

When does the hidden failure first become observable?

And if it never becomes observable:

that's a result.

Phase 12 — Topology permutation

Run identical controller configurations against:

Topology A
Topology B

Don't tune B.

Don't “fix” thresholds for B.

Compare:

detection frontier
false rollback frontier
indeterminate region
P95 latency
governable/ambiguous/blind regions

This tests whether the operating boundary is topology-dependent.

Phase 13 — Raw data first

Every trial writes one immutable record.

Something like:

seed
topology
generator
p_observe
missingness_profile
tau_mode
theta_degrade
theta_halt
v_max
lambda
detected_before_failure
detected_after_failure
false_rollback
indeterminate
detection_latency
recovery_latency
unknown_fraction
max_unknown_run
compute_steps
outcome

And ideally a separate step-level log:

trial_id
step
true_state
observed
alpha
d_alpha_dt
tau
governor_state
recovery_streak
recovery_tau

The aggregate CSV is for analysis.

The step log is for forensic debugging.

Phase 14 — Statistical analysis

Only after raw data is frozen.

Generate:

Pareto frontier

Detection vs false rollback.

Latency distributions

P50/P90/P95/P99.

Missingness comparison

Uniform vs burst at matched p.

Topology comparison

A vs B.

Operating envelope

Governable / ambiguous / blind.

G4 impossibility boundary

Which telemetry configuration first detects each hidden-failure regime.

Confidence intervals

Bootstrap them from trial-level results, not from already-aggregated percentages.

Phase 15 — Adversarial QA of the experiment itself

This is where I'd spend disproportionate effort.

Ask:

Could the simulator be leaking the answer?

Test:

Can Telemetry access hidden resource?
Can Generator access Governor?
Can Generator access veto history?
Does packet loss alter the true trajectory?
Does changing the random observation seed alter generator behaviour?
Does topology metadata leak into protocol signatures?
Does UNKNOWN affect α?
Does recovery τ change?
Does changing λ alter runtime behaviour?

If any answer is unexpectedly “yes,” stop and fix it before interpreting results.

Phase 16 — Documentation

Only now populate the README.

Structure:

1. Hypothesis
2. What RCS is / isn't
3. Formal architecture
4. Experimental protocol
5. Invariants
6. Generator definitions
7. Telemetry definitions
8. Experimental conditions
9. Results
10. Failure boundaries
11. Negative results
12. Limitations
13. Reproduction instructions

And importantly:

Separate three things

Observed result

What the simulation actually showed.

Interpretation

What we think it means.

Speculation

What might happen in real reasoning systems.

Don't let those collapse into each other.

The actual order I'd use during your downtime

If you want the minimum viable path, don't try to build the whole thing at once:

Session 1

Environment + G1/G2/G3

Session 2

Telemetry + tests

Session 3

Governor + invariant tests

Session 4

Checkpoint/Historical State + recovery tests

Session 5

Deterministic P0

Session 6

P1

Session 7

P2 uniform loss

Session 8

P2 burst loss

Session 9

Fixed vs dynamic τ

Session 10

G4 + T1–T4

Session 11

Topology A/B

Session 12

Analysis + plots + README

And don't worry about making it pretty.

The first milestone I'd aim for is simply:

pytest passes → one G2 trajectory visibly enters drift → Governor detects it → RED → BLACK → ORANGE → checkpoint restoration → fresh stable evidence → GREEN.

If that single loop works cleanly, then we earn the right to throw 1,000 seeds at it.
