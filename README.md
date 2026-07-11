Hermes Autoresearch Harness
===========================

A reusable, provider-agnostic Python harness for iterative experiment loops with strict
keep/revert controls.

What it provides
- Proposal loop with configurable commands (provider/agent-command agnostic)
- Evaluator contract parsing (JSON in stdout)
- Git-backed safety: keep/revert semantics with configurable strategy
- Concise TSV and JSONL experiment logs
- Configurable edit allowlist (path safety)
- Time/budget stop conditions and early-stop on score target

No `program.md` is included in this repository. You will supply your own proposal/evaluation
program later.

Quick start
1. Configure your proposal and evaluator commands in a JSON config.
2. Keep your project repository clean before running.
3. Run the harness.

Example config
-------------

```json
{
  "repo_path": ".",
  "proposal_command": ["python3", "examples/proposal_command_example.py"],
  "evaluator_command": ["python3", "examples/evaluator_contract_example.py"],
  "max_trials": 5,
  "stop_after_no_improve": 2,
  "min_improvement": 0.0,
  "allowlist_relative_paths": ["."]
}
```

Run
---

```bash
python -m hermes_autoresearch.cli --config examples/example_config.json
```

Evaluator contract
-----------------

Evaluator command should print JSON to stdout. Example:

```json
{"score": 12.3, "accepted": true, "reason": "tests pass", "metrics": {"coverage": 0.91}}
```

Required:
- `score` must be numeric.

Optional:
- `accepted` defaults to true
- `reason` human-readable note
- `metrics` object

Environment variables provided to proposal/evaluator
- `AR_TRIAL`: 1-based trial number
- `AR_PHASE`: `proposal` or `evaluate`
- `AR_REPO_PATH`: absolute repository path
- `AR_PREVIOUS_SCORE`: current best score or empty

Behavioral knobs
----------------

- `keep_on_improve` (default: true): keep only when the score improves by
  `min_improvement` over best.
- `revert_on_no_improve`: revert git state when proposal/eval fails or score does not
  meet acceptance rule.
- `revert_on_failure`: revert when proposal/evaluator command fails.
- `allowlist_relative_paths`: list of repo-relative paths that may be edited. Any change
  outside this list is reverted as a safety control.
- `max_trials`: upper limit of iterations.
- `max_seconds`: wall-time budget.
- `target_score`: stop early on reaching this score.
- `stop_after_no_improve`: stop after N consecutive no-improve trials.

Logging
-------

- TSV: `runs/experiments.tsv`
- JSONL: `runs/experiments.jsonl`

Each row includes status/decision, score, best_score, reason, elapsed_ms, changed paths.

Testing
-------

```bash
pytest -q
```

You should see pass/fail output and fixture-backed proofs of:
- kept vs reverted trials
- allowlist enforcement

Why provider-agnostic
---------------------

The harness treats proposal/evaluator as arbitrary shell commands. That makes it compatible with
Hermes, Claude, Codex, or any agent runtime without hard-coding provider APIs.
