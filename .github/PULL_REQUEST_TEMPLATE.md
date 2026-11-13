name: Evidence packet / ledger update
description: Submit an experiment packet, reproduction, correction, or documentation change (see CONTRIBUTING.md)
body:
  - type: markdown
    attributes:
      value: |
        Open the experiment **issue** first if this PR publishes a measurement
        (see [CONTRIBUTING.md](../blob/main/CONTRIBUTING.md) §2 and §6). Documentation
        corrections with proportionate verification may go straight to a PR.
  - type: input
    attributes:
      label: Linked experiment issue
      description: 'issue number and status (PROPOSED/CLAIMED/MEASURING/BLOCKED/READY FOR REVIEW/CLOSED)'
    validations:
      required: true
  - type: textarea
    attributes:
      label: Problem and resulting behavior
      description: What changed in the ledger or what the evidence shows
    validations:
      required: true
  - type: dropdown
    attributes:
      label: Evidence verdict
      description: One scoped verdict per CONTRIBUTING.md §6
      options:
        - PASS
        - NEGATIVE
        - INCONCLUSIVE
        - VOID
        - BLOCKED
        - documentation-only (no verdict)
    validations:
      required: true
  - type: textarea
    attributes:
      label: Evidence packet path and reproduction status
      description: 'e.g. experiments/issue-123/ (README, manifest, commands, results.csv, analysis) — or state "reported, not independently reproduced" and list what is missing'
    validations:
      required: true
  - type: textarea
    attributes:
      label: Correctness and engagement anchors
      description: Independent reference, A-A/drift controls, engagement proof for both arms
  - type: textarea
    attributes:
      label: Raw rows, failed runs, exclusions, hashes, CI method
      description: Where reviewers can reproduce the ratios; large traces as linked artifacts with hashes
  - type: textarea
    attributes:
      label: Limitations, affected workloads, deployment status, reopening condition
      description: 'Single-machine scope where applicable; documentation acceptance is not adoption'
    validations:
      required: true
  - type: checkboxes
    attributes:
      label: Reviewer checklist (CONTRIBUTING.md §7)
      options:
        - label: Nearest prior work linked; the changed premise explains any retry
        - label: Executed baseline and candidate identified; engagement demonstrated
        - label: Correctness anchored independently; quality/state contract preserved or tradeoff explicit
        - label: Same work and regime compared; A-A/drift controls adequate for the claimed gain
        - label: Raw rows, failed runs, exclusions, commands, hashes, analysis available
        - label: Ratios and CIs reproducible; end-to-end claims supported by uninstrumented timing
        - label: Memory, latency, stability, held-out regressions disclosed
        - label: Reproduction status, deployment status, reopening condition explicit
    validations:
      required: true
