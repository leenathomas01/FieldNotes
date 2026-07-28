# Waypoint: From Token Reasoning to Persistent Computational State

## Context

This exploration began with recent discussions around the limitations of Chain-of-Thought (CoT) reasoning, particularly the observation that forcing language models to externalize every intermediate reasoning step into text is computationally expensive.

Rather than asking *"How do we replace transformers?"*, the discussion gradually shifted toward a narrower engineering question:

> **How can we stop repeatedly paying compute for reasoning that has already been performed?**

---

# Core Observation

Current autoregressive language models repeatedly perform three expensive operations:

1. Reconstruct context from language.
2. Rebuild intermediate reasoning.
3. Discard almost all of it after producing the answer.

This differs from most mature computing systems, which aggressively reuse previous computation.

Examples include:

* CPU caches
* Compiler intermediate representations
* Operating system process tables
* Incremental builds
* Git delta storage
* Database query caches

LLMs currently behave much closer to:

```
Prompt
    ↓
Reason
    ↓
Forget
```

than

```
Prompt
    ↓
Reuse previous work
    ↓
Compute only the novel portion
```

---

# Important Distinction

A recurring theme was separating three concepts that are often blended together.

## Representation

How information is encoded.

Examples:

* embeddings
* latent vectors
* compressed features

Question answered:

> "How do I describe something?"

---

## Memory

How information persists.

Examples:

* vector databases
* workspaces
* caches
* state stores

Question answered:

> "How do I preserve something?"

---

## Computation

How information changes.

Examples:

* inference
* optimization
* planning
* constraint solving

Question answered:

> "How does the state evolve?"

A vector embedding is not computation.

A vector database is not a reasoning engine.

It is possible to externalize memory without externalizing computation.

---

# Compression Revisited

Grant Sanderson's information theory discussion prompted an important clarification.

Compression is not itself intelligence.

Rather:

> Discovering stable regularities allows information to be compressed.

The emphasis shifts from:

> Better compression

toward

> Better discovery of invariants.

Example:

```
Triangle angle sum = 180°
```

Once discovered, thousands of future computations disappear.

The compute savings come from discovering reusable structure—not merely encoding text more compactly.

---

# The Workspace Idea

Instead of continuously reconstructing state from language, maintain an external structured workspace.

Possible contents:

```
Current objective

Known invariants

Resolved constraints

Open variables

Assumptions

Evidence

Confidence

Execution history
```

The model no longer needs to repeatedly infer these from previous tokens.

The workspace becomes persistent while the transformer performs localized computation.

This resembles a modernized form of classical blackboard architectures combined with learned representations.

---

# State vs. Trajectory

An important refinement emerged.

There are multiple kinds of persistent knowledge.

## Knowledge

Facts.

```
Paris → France
```

---

## State

Current situation.

```
Objective

Constraints

Unknowns
```

---

## Trajectory

How similar problems have previously been solved.

Not answers.

Not documents.

Reasoning topology.

Example:

```
Measure

↓

Localize bottleneck

↓

Verify

↓

Optimize
```

The proposal is not to replay previous reasoning but to initialize future reasoning from compressed trajectory priors.

---

# Compressed Execution Priors

Rather than storing complete reasoning traces, store reusable reasoning archetypes.

Conceptually:

```
Thousands of reasoning traces

↓

Cluster similar trajectories

↓

Compress

↓

Reusable execution priors
```

Instead of regenerating an entire reasoning chain, the model begins from an established reasoning topology and adapts only where necessary.

This is distinct from Retrieval-Augmented Generation (RAG).

RAG retrieves information.

Trajectory priors retrieve reasoning structure.

---

# Compute Perspective

The discussion increasingly focused on compute efficiency rather than intelligence.

Potential areas where compute may be unnecessarily spent include:

* repeated reconstruction of stable plans
* repeated narration of intermediate reasoning
* serialization of internal state into language
* maintaining identical context across many decoding steps

Possible engineering interventions include:

* persistent structured workspaces
* invariant tracking
* delta-based state updates
* selective reuse of reasoning templates
* modular latent computation for specialized subtasks

These ideas aim to reduce repeated work without replacing the transformer architecture.

---

# Open Questions

Several questions remain unresolved and represent promising experimental directions.

### 1. What portions of Chain-of-Thought are actually computational?

Distinguish between:

* reasoning
* narration
* explanation
* verification

Rather than assuming every generated token contributes equally.

---

### 2. What is the smallest persistent state representation that eliminates repeated computation?

Possible representations include:

* structured JSON
* graphs
* symbolic constraints
* latent vectors
* hybrid systems

---

### 3. Can reusable reasoning trajectories be learned and compressed?

Instead of storing facts, store reusable cognitive pathways.

---

### 4. Which invariants are worth externalizing?

For example:

* architecture constraints
* mathematical identities
* workflow rules
* dependency relationships

Externalizing stable invariants may reduce unnecessary inference.

---

### 5. Where should cognition live?

Rather than asking whether transformers should become more intelligent, ask:

> Which responsibilities belong inside the neural network, and which belong in the surrounding computational environment?

---

# Working Hypothesis

A practical middle path between current LLMs and entirely new architectures may not require replacing transformers.

Instead, improvements may come from externalizing stable computational artifacts:

* persistent task state
* invariants
* delta updates
* compressed execution priors
* reusable reasoning trajectories

The transformer remains responsible for flexible pattern transformation, while the surrounding workspace increasingly manages continuity, organization, and reuse.

---

# Provisional Takeaway

The central engineering intuition emerging from this exploration is remarkably simple:

> **Do not repeatedly recompute stable work.**

Modern computing systems have repeatedly advanced by introducing mechanisms that preserve and reuse prior computation.

Current LLM inference reconstructs large amounts of stable reasoning from scratch on every interaction.

If persistent state, invariants, and trajectory priors can be externalized without sacrificing adaptability, substantial compute savings may be achievable while remaining compatible with existing transformer architectures.

Whether this ultimately becomes a significant architectural direction remains an open research question, but it offers a concrete and experimentally approachable middle path between incremental optimization and wholesale architectural replacement.
