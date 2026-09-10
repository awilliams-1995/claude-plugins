# Plan criteria

One file, two readers. `developing-plans` writes against it; `plan-reviewer` fails plans on it.
Kept in one place so the writer's standard and the reviewer's standard cannot drift apart.

Every criterion below is stated in **checkable** form. "The schema should be obvious" is not a
criterion -- nobody can fail a plan on it. "Every data structure is defined exactly once, with
field types" is. If you find yourself adding a criterion you could not fail a plan on, it belongs
in the skill's prose, not here.

The `Why` lines are not decoration. Each criterion exists because a specific piece of
`plan-execution` breaks without it, and a reviewer who knows which piece breaks can rank the
finding correctly.

## The document skeleton

```markdown
# <Project> Implementation Plan

**Spec:** docs/specs/<file>.md
**Plan review:** passed YYYY-MM-DD, round N -- X findings closed, Y parked
**Executor:** use plan-execution's orchestrating-plans skill

## End state

<observable, checkable conditions -- one per line>

## Global constraints

- Language: Python 3.11.9 (resolved: /opt/homebrew/bin/python3.11)
- Framework: <name and version>
- <one line each, exact values -- these are copied verbatim into every dispatch>

## Sources

| Source | What was taken from it | Verified against installed tool |
|---|---|---|

## Parked review findings

| Finding | Why parked | What it costs if it matters |
|---|---|---|

## Milestone M1: <deliverable>

<one line: what exists when this milestone is done>

### Step M1.1: <name>

**Estimate:** 1.5h
**Files:** create / modify (with line ranges) / test -- exact paths
**Interfaces:** consumes <exact signatures> / produces <exact signatures>
**Data structures:** <spec definitions this step touches, by name>
**Success criteria:** <observable and checkable>
**Design:** <artboard URL -- only when the step is UI>

- [ ] a. <sub-task>
- [ ] b. <sub-task>
```

## A. Spec and traceability

**A1. The plan names a reachable spec.**
Test: the `**Spec:**` path exists and describes this work.
Why: `orchestrating-plans` treats the spec as the binding authority and marks every ruling made
without one as provisional. A spec-less plan makes the whole execution provisional.

**A2. Every spec requirement maps to at least one step.**
Test: walk the spec's requirements; name the step that implements each. Any requirement with no
step is a finding.
Why: this is the gap that costs most. It is discovered at the end state check, after the work is
built, or not at all.

**A3. Every step traces to a spec requirement or to a named non-goal exception.**
Test: walk the steps; name the requirement each serves. A step serving nothing is scope creep.
Why: work nobody asked for still consumes six-plus dispatches and a review loop.

**A4. The spec names non-goals.**
Test: a non-goals section exists and is specific enough to exclude something.
Why: without it A3 cannot distinguish scope creep from an unstated requirement.

## B. End state

**B1. The end state is a list of observable conditions, not prose about intent.**
Test: each line names something a person or a command could check. "Users can export their data
as CSV from the settings page" passes. "The export experience is solid" does not.
Why: `orchestrating-plans` finishes with a whole-branch review on the strongest model. Without
this section that review has no rubric and grades the diff against its own taste.

**B2. The last milestone reaches the end state.**
Test: every end-state condition is satisfied by the deliverable of some milestone, and the final
milestone leaves none outstanding.
Why: a plan that stops short of its own end state has a missing milestone, not a missing step.

## C. Structure the executor requires

Each of these breaks a named piece of the plugin. They are cheap to check and cheap to fix, and
a plan failing them cannot be executed at all.

**C1. Milestones exist, each with a one-line deliverable.**
Test: at least one `## Milestone <id>:` heading; each followed by a line saying what exists when
it is done.
Why: `plan-progress-reporter` reports a milestone rollup as Table 1 and expands "the current
milestone and the next one only" in Table 2. A flat step list breaks its contract.

**C2. Every step carries an estimate in hours.**
Test: each step has `**Estimate:** <X>h`.
Why: the reporter's estimate, actual and ratio columns, and `orchestrating-plans`'
`Step <id> actual: <X>h` ledger line, all compare against it. Without estimates every ratio is
`--` and the plan's pace is unmeasurable.

**C3. A step is a heading; its sub-tasks are checkbox items.**
Test: steps are `### Step <id>: <name>`; sub-tasks are `- [ ] a. ...` inside the step's section.
Why: `task-brief` ends a checkbox step at the next checkbox item or the next heading, whichever
comes first. A checkbox step with checkbox sub-tasks truncates the brief at sub-task (a), then
fails the empty-body guard. Steps as headings resolve to the next same-or-higher heading, so
sub-tasks travel with the step.

**C4. Each step ID appears in exactly one heading, and no ID is a prefix of another at a word
boundary.**
Test: grep the ID; one heading hit. `M1.1` and `M1.10` are both fine -- the matcher excludes
digits and dots at the boundary -- but a step ID also appearing in a table of contents is not.
Why: `task-brief` refuses an ambiguous identifier rather than briefing the wrong step. That is a
hard stop mid-execution.

**C5. Global constraints are stated as one line each, with exact values.**
Test: no constraint requires reading another document to act on.
Why: `orchestrating-plans` copies this section verbatim into every dispatch as the worker's
attention lens. A constraint phrased as a reference is a constraint the worker cannot apply.

## D. Steps

**D1. A step is the smallest unit that carries its own test cycle and is worth a fresh
reviewer's gate.**
Test: could a reviewer meaningfully reject this step while approving its neighbours? If not, it
belongs inside a neighbour as a sub-task. Could a reviewer only reject it wholesale because it
does five unrelated things? Then it is five steps.
Why: each step costs pre-flight, an implementer, a review package, a critic, up to five fix
rounds, a verifier and a reporter. Granularity below the reviewable unit multiplies that
overhead without dividing the work.

**D2. Setup, configuration, scaffolding and documentation fold into the step whose deliverable
needs them.**
Test: no step whose entire deliverable is preparation for another step.
Why: such a step has nothing a critic can judge, so its review round is spent for nothing.

**D3. Each step's success criteria are observable and could not be satisfied by code that does
nothing.**
Test: imagine the step's implementation replaced by a stub. Do the stated criteria still pass? If
yes, the criteria are the defect.
Why: this is "the gate that cannot fail" -- the most common serious defect in this plugin's
measured experience -- planted at plan time rather than at implementation time.

**D4. No step consumes something a later step produces.**
Test: for each step's `consumes`, find the step that produces it and check its position.
Why: pre-flight will catch it one step at a time, at the cost of an amendment and a ruling
apiece. Catching it once, here, is free.

**D5. Interfaces are named with exact signatures, both directions.**
Test: `produces` names the function or type names and the parameter and return types later steps
will call; `consumes` names what this step calls, as it actually is.
Why: an implementer sees only its own brief. This block is the only way it learns the names its
neighbours use, and a guessed name is a rename in a later fix round.

## E. Data and interfaces

**E1. Every data structure is defined exactly once, with field types, in the spec.**
Test: grep the structure's name; one definition.
Why: two definitions become two implementations and the divergence surfaces at integration.

**E2. Steps cite data structures by name; they do not restate them.**
Test: no step contains a second copy of a structure the spec defines.
Why: a restatement drifts from the definition, and the worker follows the copy in front of it.

## F. Design

**F1. A step that introduces a new screen, a component with no existing precedent in the repo, or
a change to a user-visible flow, carries a `**Design:**` artboard URL.**
Test: for each such step, the URL is present and resolves to a specific artboard, not to the
canvas as a whole.
Why: a UI step with no design reference is a step whose implementer invents the design, and no
reviewer here can judge that -- see F2.

**F2. Copy changes, a field added to an existing form, and restyling inside an established design
system do not require a design reference.**
Test: applied as an exception to F1, and stated in the step when claimed.
Why: requiring a canvas for every visual change makes the requirement noise, and noise gets
ignored where it matters.

**F3. Design adequacy is not this reviewer's call.**
`plan-reviewer` is a text agent and cannot look at a canvas. It checks that the reference exists
and is specific. Whether the design is any good is settled by the human before the plan exists.

## G. Sources and version pins

**G1. Every external source consulted is listed, with what was taken from it.**
Test: the Sources table has a row per source and the row says what the plan relies on it for.
Why: a plan built on documentation nobody can re-find cannot be re-checked when it turns out to
be wrong.

**G2. Every claim taken from documentation is marked verified or unverified against the installed
tool.**
Test: the Sources table's third column, per row. Verified means something ran locally and the
output is quoted or the resolved path recorded.
Why: defect pattern 3 in `orchestrating-plans` -- "verify a new configuration key against the
installed tool, never against memory". Documentation describes some version; the machine has the
one that matters.

**G3. Language, framework and toolchain constraints carry version pins and resolved paths.**
Test: `Python 3.11.9 (resolved: /opt/homebrew/bin/python3.11)` passes; `Python 3` does not.
Why: `step-preflight` checks "a tool is installed at a version" and reports the resolved binary,
because a machine can carry more than one toolchain and a gate that passed with the wrong one
proved nothing. Given only a language name it has nothing to check against.

## H. Placeholders

**H1. No step defers its own content.**
Test: grep for `TBD`, `TODO`, `implement later`, `fill in`, `as appropriate`, `handle edge
cases`, `add error handling`, `add validation`, `similar to step`, `write tests for the above`.
Why: a worker sees only its brief. A deferred decision in the brief is a decision the worker
makes alone, in isolation, with no way to ask.

**H2. No step references a type, function or file that no step and no existing file defines.**
Test: for each identifier a step names as pre-existing, find it in the tree or in an earlier
step's `produces`.
Why: this is the plan's own version of a false premise, and it is the one thing pre-flight cannot
resolve by looking -- there is nothing to look at.

**H3. A step that repeats an earlier step's shape repeats its content.**
Test: no step whose body is a cross-reference in place of instructions.
Why: steps are extracted individually by `task-brief` and dispatched out of order. A worker never
sees the step being referenced.
