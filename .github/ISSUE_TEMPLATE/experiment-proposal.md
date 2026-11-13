name: Experiment proposal
description: Record a hypothesis/measurement per the evidence-ledger protocol (CONTRIBUTING.md §2–§3)
labels: []
body:
  - type: markdown
    attributes:
      value: |
        Use this template for `[experiment]`, `[reproduction]`, `[correction]`, or `[bug]` records
        (put the tag in the title). Read [CONTRIBUTING.md](../blob/main/CONTRIBUTING.md) first —
        especially §1 (start here), §3 (choose from evidence), and the proposal block below.
        A historical rejection is scoped to its implementation, stack, and workload: it is evidence
        against repeating that experiment unchanged, not proof the mechanism is exhausted.
        This repository is an **evidence ledger**, not the inference engine — engine changes belong
        in the implementation repository; link exact commits and bring the evidence here.
  - type: input
    id: title_details
    attributes:
      label: Title tag and workload
      description: 'e.g. "[experiment] IQ3_S expert dispatch / 94K single-request decode"'
    validations:
      required: true
  - type: input
    attributes:
      label: Owner and next checkpoint
      description: Who owns this, and when progress posts. Without-hardware owners may take analysis/fixtures/review.
    validations:
      required: true
  - type: textarea
    id: ledger_review
    attributes:
      label: Ledger commit reviewed and related adopted/rejected results
      description: 'git rev-parse HEAD of the ledger you read; nearest prior experiments and issues; what changed since'
    validations:
      required: true
  - type: textarea
    attributes:
      label: Observed bottleneck (source symbol / trace / raw measurement)
      description: Link the primary evidence. If attribution is missing, propose the measurement instead of inventing headroom.
    validations:
      required: true
  - type: textarea
    attributes:
      label: Hypothesis and cheapest falsifying measurement
  - type: textarea
    attributes:
      label: Baseline fingerprint
      description: Engine commit, binary hash, dispatch/config, model fingerprint (per tensor group), GPU/host, fixture hashes
  - type: dropdown
    attributes:
      label: Classification
      options:
        - semantics-preserving
        - numerical change (declared tolerances)
        - quality-changing (separate tradeoff evaluation required)
        - capability fix
    validations:
      required: true
  - type: textarea
    attributes:
      label: Correctness oracle and predeclared tolerances
  - type: textarea
    attributes:
      label: Ceiling estimate (or unknown)
      description: 'For exclusive exposed fraction f and speedup s: total = 1/((1-f) + f/s). Kernel durations are not f.'
  - type: textarea
    attributes:
      label: Measurement design
      description: Warm-up, A-A control, pairing, repetitions, CI method/seed, token-matched/trailing-twin needs
  - type: textarea
    attributes:
      label: Success gate, regression limits, exclusions, budget, stop conditions
      description: Predeclare before running. Historical example gate: median paired ratio ≥ 1.020, 95% CI excluding 1.00.
  - type: dropdown
    attributes:
      label: Status (edit the body as this progresses)
      options:
        - PROPOSED
        - CLAIMED
        - MEASURING
        - BLOCKED
        - READY FOR REVIEW
        - CLOSED
    validations:
      required: true
  - type: textarea
    attributes:
      label: Reopening condition (what new evidence would change the prior verdict)
