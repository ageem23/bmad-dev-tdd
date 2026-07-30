# Dev TDD (BMad Community Module)

A story-implementation workflow for the [BMad Method](https://github.com/bmad-code-org/BMAD-METHOD) that runs strict test-driven development.

Where the built-in `bmad-dev-story` mentions red-green-refactor as one step among several, this workflow makes the cycle the spine and adds two things it doesn't have:

- **Failure-mode analysis before any code.** Per behavior, the agent enumerates the ways it can plausibly break and classifies each as `GUARD` (must be handled), `PROPAGATE` (must escape with a defined error contract), or `OUT-OF-SCOPE` (recorded, with why). This is written into the story's Dev Agent Record before a single test is authored.
- **Defensive hardening tied back to tests.** Guards are added only where a test forces the failure. The gate is explicit: a guard with no test behind it means a failure mode was missed, and the workflow sends the agent back to the analysis step.

Two rules are enforced throughout that ordinary "write some tests" prompting does not give you: a red must fail *for the right reason* (a missing-symbol error is not a valid red), and no test may be weakened, skipped, or deleted to reach green.

## How this differs from what BMad already has

This module replaces **one step** of the story cycle — the dev step — and nothing else. It is interactive, and it is a methodology, not an orchestrator or a test generator.

| If you want to… | Use |
| --- | --- |
| Implement a story the standard way | `bmad-dev-story` (built into bmm) |
| Implement a story test-first, with failure modes enumerated and guards proven | **this module** |
| Run the whole cycle unattended — create, dev, QA, review, retro | `bmad-dev-auto` (bmm), or the `bmad-automator` module |
| Generate API and E2E tests for code that already exists | `bmad-qa-generate-e2e-tests` (bmm) |
| Set quality strategy, release gates, and a test architecture | the `tea` (Test Architect) module |

The last two operate on code that has already been written. This one runs *while* the code is being written, and the tests are how the code gets built rather than something applied to it afterwards. It sits exactly where `bmad-dev-story` sits in the cycle — preceded by `bmad-create-story:validate`, followed by `bmad-code-review`.

## Requirements

- BMad Method v6+ with the **`bmm` module installed**. This module is an extension of bmm, not a standalone: it reads bmm's `config.yaml` and discovers work through bmm's story files and `sprint-status.yaml`.

  Nothing enforces this at install time — `--custom-source` without `--modules bmm` installs the skill into a project that cannot run it. The workflow checks for bmm's config on activation and halts with an actionable message rather than proceeding with unresolved paths, but keeping `--modules bmm` in the install command is what avoids the situation.
- A project with a runnable test harness. The workflow establishes a test baseline before writing anything and halts if there is no way to run tests.

## Install

```bash
npx bmad-method install \
  --directory /path/to/your-project \
  --modules bmm \
  --custom-source https://github.com/ageem23/bmad-dev-tdd \
  --tools claude-code \
  --yes
```

To iterate on a local checkout, point `--custom-source` at the directory instead of the URL. Re-run the same command to pick up source edits — `--action quick-update` does not re-sync custom modules from their source.

## Adding it to the dev agent's menu

Installing the skill does not put it on Amelia's menu. Don't edit the installed `bmad-agent-dev/customize.toml` — BMad updates overwrite it. Use the override layer instead:

`_bmad/custom/bmad-agent-dev.toml`

```toml
[agent]

[[agent.menu]]
code = "DT"
description = "Write the next or specified story strictly test-first"
skill = "bmad-dev-tdd"
```

Menu items merge by `code`, so this appends without disturbing the built-in entries, and it survives updates. You can also invoke the skill by name and skip the menu entirely.

## Usage

```
TDD this story docs/stories/1-2-user-auth.md
```

Or, with the menu override in place, pick `DT` from the dev agent's menu. With no story path given, it discovers the first `ready-for-dev` story the same way `bmad-dev-story` does.

## Tuning the rigor

`customize.toml` exposes a `[workflow.tdd]` table. Override it per-project in `_bmad/custom/bmad-dev-tdd.toml` (team, committed) or `.user.toml` (personal, gitignored):

| Key | Default | Effect |
| --- | --- | --- |
| `min_failure_modes_per_behavior` | `3` | Minimum failure modes enumerated before any test is written |
| `require_inverse_or_crosscheck` | `true` | Data-transforming behaviors need an inverse or cross-check test, or a written justification |
| `verify_test_sensitivity` | `true` | Breaks the code deliberately to confirm the key test actually fails, then restores it |
| `defensive_style` | `"guard-clauses"` | Preferred shape for defensive code |

The standard `persistent_facts`, `activation_steps_prepend` / `_append`, and `on_complete` surfaces are available as well.

## Test design

The test list for each behavior is derived from the checklists [`src/bmad-dev-tdd/test-design-reference.md`](src/bmad-dev-tdd/test-design-reference.md).

## License

MIT. Not affiliated with or endorsed by the BMad Method maintainers.
