---
name: plan-review
description: Critically review an implementation plan before any code is written — whether it solves the right problem, with a solution that fits the existing codebase, in a form an executor can carry out without deciding anything. Use this whenever a plan or a proposed set of steps exists and the user wants it reviewed, validated, challenged or sanity-checked — `plans/<slug>/plan.md`, `.ai/features/<n>-<slug>/plan.md`, plan-mode output, a spec, an issue with proposed steps — including when they only say "revisa o plano", "critica esse plano", "esse plano está de pé?", and especially right before implementation starts or a task is dispatched to an executor. For code that already exists — a diff, a branch, a PR — this is the wrong skill; use the PR/diff review skill instead.
argument-hint: "<plan path | empty for the plan in context>"
---

# Plan Review

Review a plan before it becomes code. Two things have to hold, and they fail differently:

- **The plan is right.** It solves the actual requirement with a solution that fits the existing domain model. A plan can be internally consistent and still rest on the wrong abstraction, duplicate state that already exists, or solve the implementation proposed in the issue instead of the requirement behind it.
- **The plan is executable.** Each task can be carried out by someone who decides nothing, and its acceptance criterion would fail if the task were done wrong. A plan that is conceptually right but vague in one task buys a wasted execution round.

Check the first before the second — a task spec is not worth sharpening if the decision above it is wrong.

## Ground rules

- **Read-only.** Do not edit files, do not rewrite the plan, do not start implementing. The report goes in the chat; folding items back into the plan is the user's call.
- **"Nothing here" is a real answer — and so is "this is serious".** Every question below can be answered in one line and left behind, and a review that finds nothing on a sound plan is a successful review. But restraint applies to *whether* you raise something, never to how hard you call what you did find: a real defect softened into a mild question is a missed review, and it costs more than a blunt one. Say plainly what you believe, and let the category carry the weight.
- **Requirements and declared scope are binding.** An explicit requirement outranks a simpler alternative you prefer. So does a documented out-of-scope decision.
- **Verify, don't recall.** If the plan was written in this same context, you already believe its claims — re-open the code for each one instead of answering from memory, and say in the report that it is a self-review.

## Sources of authority

Different questions have different authoritative sources. Match the source to the question:

| Question | Authoritative source |
|---|---|
| What *should* the system do? | Business requirement, acceptance criteria, explicit constraints, domain rules. |
| What does the system *currently* do? | Code, schema, migrations, queries, tests. Documentation describes intent and goes stale — never infer current behavior from it, or from the plan. |
| *Why* was it built this way? | Domain and decision docs, ADRs, `.ai/context/`, `.ai/guidelines/`, design docs. Code shows how it works, not why that choice was made. |

Two rules cut across all three:

- **No silent redefinition.** The requirement outranks the solution proposed in the issue, which outranks the plan. A plan that redefines the requirement, or an issue whose proposed solution has quietly become the requirement, is a finding in itself.
- **Existing behavior is not a requirement.** That the system does something today is evidence about the present, never proof that it is correct or must stay. Equally, "the code doesn't do this today" is not an argument against a requirement.

When sources disagree, report the conflict. Do not resolve it silently by picking whichever source you read first.

## Scale the review

Reach and reversibility set the depth. A local change with no new persisted state and no contract change needs the requirement, the affected paths and the acceptance criteria confirmed — a few lines of report. New persistent state, migrations or backfills, API/event/queue contracts, cross-cutting changes or concurrency need every step below.

Deciding "light" because the plan is short is the failure mode here. Judge by what the change touches and what it would cost to undo, not by plan length.

## Workflow

### 1. Locate the inputs

Find, in this order: the plan, the requirement it came from, the project's own rules, and the code it touches.

The requirement is usually one pointer away. Follow it rather than inferring:

- `Origem: Owner/repo#355` → `gh issue view 355 --repo Owner/repo`. The issue often lives in a different repository than the work.
- `Spec: specs/<area>/<slug>.md` → read it, including its state (draft/approved matters).
- Project rules live in tracked docs — `.ai/guidelines/*.md`, `.ai/context/*.md`, `specs/`, `design/`, ADRs. Be careful with `CLAUDE.md` and `AGENTS.md`: in some repos they are generated by tooling and git-ignored, so they are a copy, not the source. When a tracked rules file and a generated one disagree, the tracked one wins.

Then map the plan's own fields **by role, not by name** — the names differ per repo, the roles don't:

| Role | e.g. `plans/<slug>/plan.md` (pt-BR) | e.g. `.ai/features/<n>-<slug>/plan.md` (en) |
|---|---|---|
| Requirement pointer | `Origem:` / `Spec:` header bullets | issue number in the folder name |
| Decisions, with the why | `## Decisões`, numbered | `### Architecture decisions` |
| Tasks | `### N.` + `Arquivos` / `O que fazer` / `Aceite` / `Commit` | `#### Task N` + `Goal` / `Change` / `Non-goals` / `Feature test` / `Depends on` / `Done when` |
| Declared out of scope | `## Fora de propósito` | `Non-goals` per task |
| Feature-level acceptance | inside each `Aceite` | `### Acceptance criteria`, Given/When/Then |

A plan missing one of these roles entirely is itself worth a line: a plan with no recorded decisions and no declared scope boundary gives a reviewer nothing to check and gives the executor nothing to stay inside.

### 2. Open every locator the plan cites

Plans worth reviewing cite their own evidence inline — `SomeService:516`, `SomeTrait:61`, `.gitignore` line 78. Open each one and check the claim holds.

This is mechanical, fast, and where the highest-signal findings live: a decision resting on a locator that says something else is wrong at the root, not at the edge. Line numbers drift, so match on the symbol — if the line moved but the claim holds, that is not a finding. A locator that no longer exists, or that says the opposite, is.

**A name in the plan is not a name in the tree.** A plan is mostly symbols, fields and methods that do not exist yet — that is what makes it a plan. So every claim you make about what the repository *already* does is a claim you must have opened, and the trap is quiet: you read a field name in the spec's contract table, carry it forward as an existing method on the model, and end up recommending "keep the derived read you already have" — an alternative built on code nobody has written. Before writing that something already exists, search the tree for it. If it turns up only in the plan or the spec, it is proposed, not present.

Do this before forming any opinion on the design. It is cheap and it frequently changes what the rest of the review is about.

### 3. Restate the requirement

Separate the business requirement, the acceptance criteria, the constraints (technical, architectural, compatibility, migration, security, operational), the implementation merely *suggested* in the issue, and the assumptions the plan introduced on its own. An implementation suggested in the issue is not a requirement unless it is explicitly constrained as one.

Then restate it in one or two sentences **without naming the proposed solution**. If you cannot do that, you do not yet have the requirement — you have the plan's summary of it.

If the pointer is dead or absent, say so, state the requirement you inferred, and mark findings that depend on it as lower confidence. Never infer the requirement from the plan's own solution.

### 4. Six questions about the solution

Answer each one. One line for "nothing here" is fine and expected on a sound plan.

1. **Was the requirement understood?** Does the plan solve what was asked, or a neighbouring problem that is easier to state? Any silent redefinition between requirement, issue and plan.
2. **Does this concept need to exist?** Is it already carried by an existing model, state, relationship, service, query, policy or lifecycle rule? Is the plan compensating for a misreading of the existing design?
3. **Does it create a second source of truth?** Could the fact be derived instead of stored? If new state is genuinely needed: who creates it, who updates it, who invalidates it — and which write paths bypass that logic (bulk updates, raw queries, imports, scripts, console commands, another service). Persisted state plus a write path that skips the hook is the most common way these plans go wrong.
4. **Which invariant does it move?** Mutually exclusive states, required relationships, lifecycle transitions, uniqueness, idempotency, ordering, historical rows. Does the design make invalid states harder or easier to represent — and can an invalid state exist mid-operation or mid-deploy?
5. **Is the abstraction at the right level?** Trace how far the concept has to travel: schema, models, DTOs, services, jobs, events, controllers, API, UI, imports, tests. Wide propagation is not automatically wrong, but it is the usual symptom of solving the problem far from its source of truth.
6. **What breaks in production?** Only where it applies: concurrency, retries, duplicate execution, partial failure, stale reads, data volume, deploy order between schema and code, rollback, existing rows. Do not invent operational complexity the requirement does not ask for.

### 5. The plan as an executable artifact

A plan is a decision record handed to an executor who must decide nothing. These five checks are cheap and falsifiable, and each one maps to a concrete wasted round when it fails:

1. **Does each acceptance criterion bind?** Ask it as a test: *if this criterion passes, could the task still have been done wrong?* A task that adds an error chip, a success chip, a row highlight and two tooltips, accepted by "the build passes", accepts the task entirely not done. Name the behaviour the criterion fails to hold.
2. **Can the task be executed without deciding?** The files must be named and the change must be stated. If the executor has to infer *how* — which class, which signature, which existing pattern to imitate — the task is incomplete, and it will come back either wrong or as a question.
3. **Is each decision recorded with its why?** The plan's job is to carry what the diff cannot show. A decision stated without its reason cannot be reviewed later, and a decision made during planning but left out of the plan makes the plan lie about what was done.
4. **Is it still a decision record, or has it become a script?** Pasted final file contents, step-by-step editing instructions and inlined code belong in the executor's prompt, not in the committed plan. A plan running much past a third of the diff it directs has absorbed spec that should live elsewhere.
5. **Does it fit one PR, with one commit per task?** Tasks ordered so earlier ones unblock later ones, dependencies stated honestly, each task a coherent commit. Work that does not fit one PR is several plans with a declared stack order, not one long plan.

Checks 1 and 2 are where the value is. Do not let 3–5 turn into style complaints — raise them only when something concrete is missing or misplaced.

### 6. Implementation and verification

Only once the solution stands, check whether the plan changes the right components, reuses existing abstractions and project conventions, misses affected code paths, adds layers or coupling that buy nothing, handles migrations and existing data safely, preserves backwards compatibility where required, and handles failure.

On verification, look past whether tests are mentioned to whether they would catch a regression: behaviour tests that bind the requirement, coverage for the invariants from question 4, the affected paths, integration points, and production-safe validation for migrations and backfills. Tests that only mirror implementation details verify nothing unless those details are contractual.

Challenge steps that exist only because of an earlier questionable decision — they disappear when it is fixed.

## Priorities

Do not let a pile of small observations bury a fundamental one. In order: misunderstood requirement; wrong domain model; duplicated source of truth or unnecessary state; broken invariant; unbinding acceptance criterion or task the executor cannot run; architectural and lifecycle problems; missing paths and operational risk; then everything else.

## Do not raise

- Anything the plan explicitly declared out of scope, with a reason. That boundary is a decision, and re-opening it is noise. If the *reason* is wrong, that is a finding about the reason — say so in those terms, and do not smuggle the whole item back in.
- Naming, formatting or style already enforced by tooling.
- Refactors unrelated to the requirement; speculative extensibility; abstractions for elegance.
- Theoretical edge cases with no realistic path to happening.
- Infrastructure for hypothetical needs: an event where a direct call is enough, a generic mechanism for a single use case, derived state persisted "in case", synchronization before a second source exists.
- Alternatives whose only merit is being different.

## Findings

### Categories

Classify by what the reader has to *do*, not by how serious it sounds.

- **Blocking** — do not implement until this is resolved. The plan may produce wrong behaviour, violate a requirement, set up the wrong source of truth, or create a problem whose fix changes the plan's shape. It needs a decision, not an edit.
- **Required change** — the approach is sound, but a concrete defect must be fixed first: a missed code path, an unsafe migration order, a missing constraint, an acceptance criterion that binds nothing. The fix is known and local.
- **Improvement** — optional. The plan is already valid and an alternative would be simpler or safer. An improvement never changes the verdict.
- **Open question** — you need information that is missing or ambiguous. Only ask what changes the solution or the implementation, then classify by what happens if the answer comes back the other way. If it would change the rule itself, the data model, a contract or a migration, it is **blocking**: you cannot validate a plan whose rule is still undecided. If it would change a local detail, it is a **required change**. Informational is the narrow residue where the answer changes nothing you would do differently and you are only confirming. When you cannot tell which it is, it is not informational — you are raising it precisely because you do not know. Asking whether some case belongs inside a business rule is a question about the rule, not a detail.
  Classify by the consequence of the answer, never by which answer you expect. You are in this category precisely because the premise is unconfirmed — if you could confirm it you would be filing a defect — so "I looked for the path and did not find it" is the reason you are asking, never a reason to file it lower. Existing defensive code that already handles the case (a null check, a guard, a filter written for this very incident) is evidence the case occurs, not a hint. And carry the severity in the label — **Open question (blocking)** — so a verdict that counts blockers cannot quietly drop it.

When torn between blocking and required change, try to state the fix in one sentence. If you can, it is a required change. If answering it could change the design, it is blocking.

### Format

Number the findings so the user can cite them when reworking the plan. For each one:

- **Finding** — what is wrong.
- **Consequence** — the concrete failure: which input or state produces which wrong result. "Totals come out at 938,44 instead of 469,22 — exactly double" is a consequence; "the total may be wrong" is a restatement.
- **Evidence** — the most precise locator you actually checked: file and symbol, test, query result, requirement, doc section. Mark it **Verified** (you opened it in the tree during this review) or **Inferred** (reasoned, not confirmed). Verified is a claim about the tree, never about the plan: finding a symbol in the plan or the spec confirms only that the plan proposes it. A named symbol you really opened beats an invented line number.
- **Direction** — only when the evidence supports one. Where both exist, give the preferred fix and the minimal one, and say what the minimal one leaves fragile: choosing between cost and durability is the user's call, and they need both options to make it.

### Calibration

Weak:

> Consider concurrency in the job.

Strong:

> **3. Blocking — `status_summary` diverges after bulk cancellation.**
> The plan persists `status_summary` and updates it from a model lifecycle hook, but `OrderService::bulkCancel()` writes with a bulk query that never fires those hooks, so the column keeps the pre-cancellation value. *(Verified — read `OrderService::bulkCancel`.)*
> **Direction:** preferred, derive it from the existing `payments` relationship — the requirement only needs it for display, and derived state cannot drift. Minimal, add the same write to `bulkCancel`; cheaper now, but the next bulk path that skips the hook reintroduces the divergence silently.

The strong version names the mechanism, the failure, the evidence, and what each fix costs.

## Output

Match the language of the conversation, not of the plan — a project can require plans in English while the conversation runs in Portuguese. Translate the headings along with the body; a Portuguese report under English headings reads like a template nobody filled in.

Use this structure, dropping empty sections:

```
## Requirement
Restated, independent of the proposed solution, and whether it came from the
original source or was inferred.

## Findings
Numbered, blocking first, then required changes, then open questions, then improvements.

## Simplification
Only when a materially simpler approach satisfies every stated requirement.

## Reviewed, sound
One line per area checked and found fine, so the user doesn't re-audit it.

## Verdict
Proceed | Proceed with changes | Rework — one sentence, grounded in the findings above.
```

- **Rework** — at least one blocking finding, and an open question you classified as blocking is one.
- **Proceed with changes** — required changes, nothing blocking. Name them in the sentence.
- **Proceed** — neither. Improvements and informational questions may still be listed; say plainly that they are optional.

Then stop. A blocking finding is a decision for the user to make; do not adjust the plan and carry on.

## Re-reviews

If the plan has been reviewed before — the user reworked it after a previous pass, or the plan records earlier review items — verify the previous findings first, and open the report with that status: **resolved** (name what in the plan now covers it), **partly resolved** (say what is left), or **untouched**. Never assume a finding was handled because the plan changed.

Only then review what is new, with the full pass above. Rework introduces its own defects — read the changed decisions as fresh material, not as trusted patches.
