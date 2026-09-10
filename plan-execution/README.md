# plan-execution

Turns an idea into a reviewed plan, and that plan into delivered, verified work.

Two skills run in your session and stay there. Five agents run in their own contexts and are
thrown away. The split is deliberate: anything that judges the work must not share the reasoning
of whoever asked for it, and anything that needs continuity across the whole plan cannot be an
agent.

| Piece | Kind | Job |
|---|---|---|
| `developing-plans` | skill | gates on whether the intent is concrete enough to build, elicits it into a spec, derives the plan, has it reviewed, and hands off |
| `plan-reviewer` | agent | judges a plan against its spec and against the structure the executor requires, before any of it runs |
| `orchestrating-plans` | skill | owns the ledger, the rulings, the dispatch decisions, the convergence calls, and the conversation with the human |
| `step-preflight` | agent | checks the step's premises against the tree as it actually is, immediately before dispatch. Reports; never decides |
| `harsh-critic` | agent | independent adversarial review of one step's diff |
| `fix-verifier` | agent | reproduces each finding against the current tree; the fixer's account is not evidence |
| `plan-progress-reporter` | agent | reconciles plan, ledger and git history into three tables, and names their disagreements |

## The problem it solves

**Nothing produced the plan.** The executor's pieces require structure a hand-written plan rarely
has: milestones, because the progress reporter rolls up by milestone; per-step estimates, because
the ledger records actuals against them; version pins with resolved paths, because pre-flight
checks the installed toolchain against them; a reachable spec, because without one every ruling
made during execution is provisional; and step IDs that resolve to exactly one heading, because
`task-brief` refuses an ambiguous identifier rather than briefing the wrong step. That format is
now written down in `skills/developing-plans/plan-criteria.md`, one skill writes against it, and
one agent fails plans on it.

**And a plan is written before the code exists.** By the twelfth step several of its premises are false:
signatures it invented, directories it expected, tool versions it assumed. Executing a stale
premise costs a full review loop to find and a full fix loop to undo.

On the project this was extracted from, steps that got a pre-flight pass ran at about 1.35x their
estimate. Steps that did not ran at about 3.25x.

## Install

```sh
/plugin marketplace add awilliams-1995/claude-plugins
/plugin install plan-execution@awilliams-1995-plugins
```

Adding the marketplace is a once-per-machine step. To switch the plugin on for everyone working
in a given repo, commit this to that repo's `.claude/settings.json`:

```json
"enabledPlugins": {
  "plan-execution@awilliams-1995-plugins": true
}
```

Then, with an idea and no plan:

```
Use developing-plans to plan <what you are building>
```

Or, in a repo that already has a plan:

```
Use orchestrating-plans to execute docs/plans/<the-plan>.md
```

`orchestrating-plans` will not execute a plan that has not been reviewed. Given one without a
review stamp in its header it dispatches `plan-reviewer` once before step 1; given no plan at all
it sends you to `developing-plans`.

## Workspace

Everything for one plan lives in `<repo-root>/.plan-execution/<plan-basename>/`: the ledger, the
step briefs, worker reports, review packages, pre-flight tables. It is added to the repository's
local git exclude, so it never reaches a commit. The ledger is the recovery map -- conversation
memory does not survive compaction, and the ledger and `git log` do.

## Scripts

| Script | Use |
|---|---|
| `plan-workspace PLAN_FILE` | prints (creating if needed) the plan's workspace directory |
| `task-brief PLAN_FILE STEP_ID` | extracts one step to its own file so no worker ever reads the whole plan |
| `review-package PLAN_FILE BASE HEAD` | writes the commit list, stat summary and full diff to one file for the reviewer |

The spec and the plan are committed to the repository -- `docs/specs/` and `docs/plans/` by
default -- because they outlive the workspace. Only review reports and execution artifacts live
in the git-ignored workspace.

`task-brief` resolves a step by markdown heading or by a `- [ ] **Step M1.4: ...**` checkbox
item, matches at a word boundary so `M1.1` does not also hit `M1.10`, and refuses an ambiguous
identifier rather than briefing the wrong step. Plans written by `developing-plans` use the
heading form, because a checkbox step containing checkbox sub-tasks would end at the first
sub-task. `review-package` refuses an empty range: a step
that produced no commits has not been implemented.

## Where the content came from

The lessons encoded here are measured, not theorised. They come from executing the first
milestone of an eleven-milestone embedded firmware rewrite: 19 fix rounds across 8 steps, 83
commits, and a catalogue of the defects that survived ordinary review. The defect list in the
skill -- above all "the gate that cannot fail", found 26 times in that one milestone -- is that
catalogue.

## Credits

The two-phase shape of `developing-plans`, its one-question-at-a-time elicitation and the plan
skeleton are derived from the `brainstorming` and `writing-plans` skills in Jesse Vincent's
[superpowers](https://github.com/obra/superpowers), MIT licensed, (c) 2025 Jesse Vincent. The
granularity rule, the executor-required structure, the adversarial plan review and the gates are
this plugin's.

## Licence

MIT. See [LICENSE](../LICENSE).
