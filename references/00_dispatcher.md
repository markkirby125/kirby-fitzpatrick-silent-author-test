# Silent Author Test — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [I coached 25 writers to learn THIS](https://www.youtube.com/watch?v=gaqIcDkTakA)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Author Is an Unpinned Runtime Dependency

Every technical artifact is a program that executes inside a reader's head and produces exactly one output: the correct action. Like any program it has a dependency list, and that list is almost never written down. One entry on it is always the same: **the author**.

That dependency has unusual properties. It is unpinned — no version, no digest. It is unresolvable by the reader's tooling. It exists at exactly one address, with no failover, no retry, and a TTL measured in tenure plus memory. It has no registry entry, so nothing in the repository records that a call to it ever happened. When it resolves, the artifact appears to work — which is precisely why the defect survives review, CI, and merge.

**The Silent Author Invariant**: *write technical artifacts assuming the author will never be available to clarify or explain.* Not as style guidance — as a build target. Formally: for every fact `F` a reader needs in order to act correctly, `F` must have an address inside the artifact, inside the repository, or inside a durable escalation route (a role, a queue, a pager policy — never a person). Facts with no address are unresolved symbols: the artifact still parses, it simply cannot link.

The lecture's coaching observation, translated: Fitzpatrick coached 25 writers, and the recurring failure was never bad sentences. It was drafts that ran only while the writer stood next to them — the writer's spoken explanation silently supplied the missing context, so the text looked complete in the room and collapsed on the desk. Read a draft without its writer and the omitted step becomes visible in seconds. Software artifacts fail the same way, one degree worse: the writer is *present at review time*, so the reviewer's clarifying questions get answered in the thread and the answer never lands in the artifact. The artifact passes. The gap propagates.

The author becomes silent in four distinct ways, and only one of them involves leaving the company:

- **Absence** — the 03:00 reader, the new hire, the archived repo, the upstream consumer's team. The author is not on the call.
- **Latency** — the author exists, but asking costs an hour of theirs and a day of yours. Under incident pressure that call is skipped, and the reader improvises.
- **Fidelity decay** — the author is reachable and has forgotten. Six months on, the rationale for the constant is gone even for the person who typed it. The comment was the only witness, and it narrates the code.
- **Attrition** — the ordinary case, and the only one nobody plans for.

```text
[ANTI-PATTERN: the artifact is a client of a human API]

   reader ──▶ artifact ──▶ decision ──▶ action
                 ▲
                 │  oracle call: "ask the author"
                 │  latency      minutes → never
                 │  availability 1 (no failover, no queue, no retry)
                 │  fidelity     decays with the author's own memory
                 └──── author (single point of failure, unversioned, unpinned)

   Boundary behaviour
     A0  author is typing          → works. This is the trap.
     A1  author is reachable       → latency is moved inside the incident timeline
     A2  author is on another team → the call now routes through a stranger
     A3  author has left           → unresolved symbol, no diagnostic message
   Nothing in the repository records WHICH fact was borrowed, so the gap stays
   invisible to every check that runs while the author is still around.
```

```text
[PROTOCOL: the artifact is a closed program]

   reader ──▶ artifact ──▶ decision ──▶ action
                 │
                 ├── contract  what it does, and what it assumes
                 ├── evidence  the command, the expected output, the tested version
                 ├── failure   what a broken run looks like, and what to do next
                 └── bounds    when it does NOT apply + the trigger to re-verify

   Author edges: 0. Every fact the reader needs resolves inside the artifact,
   the repository, or a durable route (role + queue + expected response time).

   Verification is a measurement, not an opinion:
     silent read → time-to-first-correct-action, plus the list of ambiguities.
     Ambiguities are patched into the ARTIFACT, never into the reader.
```

Five load-bearing definitions:

1. **Author Dependency (AD)** — a fact required for the correct use of an artifact that exists only in a person's head, a private thread, or a meeting. Not "missing documentation" in the vague sense: a specific, countable fact with no address.
2. **Silent Read** — the falsification procedure. Freeze the artifact at a version, hand it to a reader with repository access and no author contact, and require the artifact's job to be performed to completion. The count of questions the reader cannot answer from the artifact alone *is* the AD count.
3. **Ambiguity Budget** — how many unevidenced decisions a silent reader is allowed to make. Zero for critical-path artifacts (runbooks, money and data paths, restore procedures, public SDK docs). Non-zero budgets must be *enumerated* in the artifact, because an implicit budget is indistinguishable from a defect.
4. **Oracle Call Cost** — latency + availability + fidelity of asking a human. Once the artifact sits on a critical path, this cost is paid inside an incident, in a context where the reader holds production rights and no context.
5. **Blast-Radius Scaling** — required self-sufficiency scales with the cost of the reader acting wrongly. A scratch script may keep an author dependency; a restore runbook may not, in any field, at any level of detail.

Why this binds hardest in software: artifacts are executed out of order, under stress, by strangers who hold production credentials, and the failure mode is not confusion — it is a wrong mutation. Prose that needs its author wastes a reader's afternoon; a runbook that needs its author corrupts data. Software artifacts also decay silently: the dashboard is renamed, the version moves, the feature flag flips, and the text keeps rendering perfectly while its references rot. The silent author test is the one check that catches all three — omission, latency, and drift — because it asks the question no linter can: *with the author gone, does this still produce the right action?*

---

## 2. Core Transformation Protocols

1. **Write for A4; verify at A0.** Draft and audit against the archived, authorless state of the artifact; use the author's presence only as a test harness to answer questions the artifact should have answered itself.
2. **Give every fact an address.** Before shipping, list the facts a reader needs and confirm each resolves to the artifact, a versioned repository path, or a durable route. A fact that resolves only to a person is a blocker, not a nicety.
3. **Route to roles, never to people.** `page the storage on-call (rotation storage-primary, 15 min target)` replaces `ask Dave`. Rotations decay gracefully; handles 404.
4. **Ban conversational provenance.** *as we discussed*, *per our call*, *as agreed in standup*, *see the thread above* (collapsed) — replace with the durable artifact (`ISSUE-1043`, `ADR-021`, `commit 3a77e02`) or delete the claim, because an unaddressable claim is not information.
5. **Ban deictic pointers.** *this*, *that*, *the above*, *the old config*, *the previous behaviour* — name the symbol, path, or commit explicitly. A pronoun is a promise that the referent lives in the reader's head.
6. **Resolve coined vocabulary.** Internal codenames, project nicknames, and tribal acronyms appear only with a one-line definition or a link to the type, module, or table that names them canonically.
7. **State units, magnitudes, and the consequence of crossing them.** `timeout: 30` → *`30s`: the gateway aborts the request (not the downstream call) after this; raising it past the client's 2s budget makes the value unreachable.* Numbers without units and without consequences are the most expensive AD class, because they look like knowledge.
8. **Declare preconditions and environment inside the artifact.** Credential scope, region, cluster, dataset size, tool and dependency versions, feature-flag state — plus the command that reveals each. *It works on my machine* is the author dependency stated out loud.
9. **Document the failure path, not only the happy path.** Every step carries expected output, normal duration, what a failure looks like, and whether the reader stops, retries, rolls back, or escalates.
10. **State the reversal condition.** The artifact names the signal that makes it obsolete (version, scale threshold, violated assumption) and who re-checks it. An instruction that cannot be wrong is an instruction nobody can audit.
11. **Make decay visible.** Pin or date anything that rots — versions, sample output, dashboard names, links, thresholds — as *verified `<date>` on `<version>`*, plus the trigger that forces re-verification (schema bump, major upgrade, incident).
12. **Do not narrate what the reader can read.** Comments and docstrings carry what the code cannot express — the invariant, the external constraint, the cost of removal — never a restatement of the adjacent lines. Restated code is the author's voice reading the diff aloud.
13. **Prefer structural fixes to prose fixes.** If three paragraphs are needed to explain the order in which to do things, reorder the artifact instead. If a step is too dangerous to explain, remove it from the runbook path rather than clarifying it.
14. **Run the silent read as a gate for critical-path artifacts, and patch the artifact.** Ambiguities found in the read are defects with owners, not questions for the author. The reader is the instrument; the artifact is the thing under test.

### 2.1 Author dependency classes and their fixes

| AD class | Where it hides | Silent-author fix |
|---|---|---|
| Oracle reference | *ask X*, *ping #platform*, *check with the team* | Route with role, queue, and expected response time |
| Conversational provenance | *as discussed*, *we decided*, references to collapsed threads | Durable artifact ID (issue, ADR, commit) or delete the claim |
| Deictic pointer | *this*, *that*, *the above*, *the old behaviour* | Named symbol, `path:line`, or commit SHA |
| Coined vocabulary | Project codenames, tribal acronyms, internal nicknames | One-line definition or canonical type/module reference |
| Unstated unit | `30`, *retry 3 times*, *large*, *fast* | Unit, semantics, and the effect of exceeding it |
| Unstated precondition | Credential scope, region, dataset size, flag state, versions | Enumerated in the artifact, plus a command that checks each |
| Unstated failure policy | Happy-path-only step lists | Expected error, interpretation, and the next action |
| Unstated reversal condition | *we chose X because it's faster* | Measured baseline plus the signal that would reverse the choice |
| Decayed fact | Renamed dashboards, stale versions, pasted sample output | Pin + `verified <date>` + re-verification trigger |
| Implicit ordering | Step 4 assumes an action performed elsewhere | Numbered steps plus explicit entry state |
| Author-tacit rationale | Why the constant, cap, or guard exists | Invariant, external constraint, cost of removal |
| First-person residue | *I*, *my*, *we* where no lasting owner exists | Role-based subject, anchored to the artifact carrying the decision |

### 2.2 Transformation table: anti-patterns and clean replacements

| Anti-pattern | Author dependency | Clean replacement |
|---|---|---|
| `// retry 3 times` | Unit, rationale, and the source of the number | `// Bounds the CALLER's 2s deadline, not the downstream service: 3 attempts x 400ms backoff + 2 x 200ms jitter = 1.6s. The deadline check runs before each sleep, so a slow attempt consumes the budget instead of stacking retries on top of it. Do not raise the cap without re-running budget_test.go:88.` |
| `// This is a workaround` | What it works around, and when it may be deleted | `// Works around upstream bug GH-4412 (v2.3–v2.5): the loader drops the final record when the batch is closed by timeout. Remove once the pinned upstream version is >= 2.6; see pins.lock.` |
| `// See the design doc` | Which doc, at which version, discoverable by whom | The invariant in one sentence, then the durable link: `ADR-021 §Decision` |
| `"""Returns the result."""` | Precondition, error semantics, side effects | `"""Failure is absorbing: the first error is returned and the stream is closed, so callers must treat this handle as single-use. Reads at most one page; callers needing full results loop until EOF."""` |
| `timeout: 30` in config | Unit and consequence | `timeout_s: 30  # the gateway aborts AFTER this, not the downstream call; must stay below the client's 2s budget or the value is unreachable` |
| Runbook: *check the logs* | Which logs, which pattern, what normal looks like | `kubectl -n payments logs -l app=reconciler --since=10m` then filter for `reconcile:` — expect one `scanned=<n>` line per minute; zero lines means the worker is not consuming at all. |
| Runbook: *ask the platform team if it fails* | Route, scope, expected latency | `If step 3 has not cleared within 15 min, page platform-primary (rotation platform-primary, 15 min target). Do not DM individual engineers.` |
| PR body: *fixes the bug we discussed* | The bug, the mechanism, the verification | Issue ID, one-line mechanism, verify command, evidence table, rollback path |
| Doc: *setup is straightforward* | Every hidden prerequisite | Enumerated steps with versions, per-step expected output, and the state the reader must start from |
| *We chose X because it is faster* | Measured baseline and the condition that reverses it | *X: p95 480 → 210 ms, workload bench-recon-100k, baseline 4f9c1ab, 5 runs, ±6 ms. Reverses if items per page grow past 5k, where the constant-factor win disappears.* |
| *TODO: clean this up* | Who, and the trigger | `TODO(schema-v3): delete the legacy path once all tenants are migrated; tracked in ISSUE-1043.` |
| Commit body: *Misc fixes* | Anything a future reader can use | The invariant, the blast radius, and why the change is safe |
| Onboarding doc: *follow the usual process* | A process that exists only as habit | The numbered procedure, plus the artifact that enforces it (CI job, template, script) |

### 2.3 The silent-read probes

| Probe | Question asked of the artifact | Pass condition |
|---|---|---|
| Substitution | If the author were replaced by a stranger with repository access, does the artifact still work? | No step requires knowledge absent from the artifact or a versioned path |
| Temporal | Read six months from now, with none of today's recall, does every claim still resolve? | Versions, links, and sample output carry a verification date and a re-verify trigger |
| Address | For each fact the reader needs, where does it resolve? | Every fact has an artifact address; every human route is a role with a queue and a target response |
| Failure path | When the instruction breaks, what does the reader see and do? | Expected error, interpretation, and the stop/retry/rollback/escalate decision are stated |
| Bounds | Does the artifact say when it does not apply? | An explicit *does not cover* section, plus the signal that invalidates the procedure |
| Latency | How long until the reader's first correct action, unaided? | Measured on a cold reader, recorded, and inside the incident budget |

### 2.4 Failure diagnostics

| Symptom | AD diagnosis | Fix |
|---|---|---|
| *"What does this mean?"* asked of a passing reviewer | Deictic pointer or coined vocabulary | Name the referent; define the term |
| Reader improvises a step and breaks production | Unstated failure policy in a critical-path artifact | Add expected output, timing, and the decision at each step |
| Incident resolved, cause unknown | Rationale lived only in the author | Record the invariant and the trigger, not the narrative |
| Runbook executed correctly, wrong result | Unstated precondition (scope, region, flag state) | Enumerate preconditions plus a command verifying each |
| Artifact "works" until the author leaves | Oracle reference never converted into a route | Replace with rotation, queue, and target response |
| Ticket reopened two quarters later with the same question | Answer given in a thread, never landed in the artifact | Patch the artifact; close the thread by reference |
| Doc reads fine but the reader cannot start | Entry state unstated | State the starting state, the command, and the observable output |
| Reviewer argues for a threshold change | Reversal condition absent | Bind the number to its baseline and to the signal that moves it |

**Related dispatchers.** Audit your own artifact as a zero-context stranger with the [Cold Reader PR Auditor](../../kirby-fitzpatrick-cold-reader-pr-auditor/SKILL.md); convert review dialogue into durable dispositions with the [Letter of Response Reviewer](../../kirby-fitzpatrick-letter-of-response-reviewer/SKILL.md) (same lecture — that skill governs the conversation that produces the artifact, this one governs whether it survives without it); strip pronouns, filler, and restated code with the [Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md) and the [Empty Verb Extractor](../../kirby-fitzpatrick-empty-verb-extractor/SKILL.md); make the surviving structure discoverable at skim speed with [Cathedral Taxonomy](../../kirby-fitzpatrick-cathedral-taxonomy/SKILL.md) and the [Skim-Test Outliner](../../kirby-fitzpatrick-skim-test-outliner/SKILL.md); ground the opening steps in primitives a stranger already holds with [Bilbo Simple-to-Complex](../../kirby-fitzpatrick-bilbo-simple-to-complex/SKILL.md); lead with the decision the reader must make and follow with the rationale using [Here's Why Inversion](../../kirby-fitzpatrick-heres-why-inversion/SKILL.md); and price the cost of a wrong action before shipping the artifact with the [3-Part Proposal Engine](../../kirby-fitzpatrick-3part-proposal-engine/SKILL.md).

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews — the last moment the author is still available

Review is the final window in which tacit knowledge can be converted into the artifact. Treat every question the reviewer has to ask as an unresolved symbol in the diff, not as a conversation to hold. The author's spoken answer is not the deliverable; the artifact is.

Three passes:

1. **Sweep for AD markers.** Deictic pointers (*this*, *that*, *the above*), oracle references (*ask X*), conversational provenance (*as discussed*), narration of adjacent code (`// increment the counter`), bare numbers, and TODOs with neither owner nor trigger.
2. **Probe every artifact the diff touches** — comments, docstrings, test names, commit body, PR body, and any runbook or config the change invalidates. Apply the substitution and temporal probes to all of them.
3. **Require the address.** If the answer to a reviewer's question is knowable, it belongs in the artifact in the same commit; if it is genuinely unknowable, it is an open question with an owner, written as such.

```go
// BEFORE — three author dependencies in five lines:
// no unit, no rationale, no failure path, no consumer for the number
// retry 3 times
if attempts > maxAttempts {
    return err
}

func TestRetry(t *testing.T) { /* ... */ }
```

```go
// AFTER — the artifact answers the reviewer's question inside the diff
// RetryPolicy bounds the CALLER's 2s deadline, not the downstream service.
// Budget: 3 attempts x 400ms backoff + 2 x 200ms jitter = 1.6s worst case. The
// deadline check runs before each sleep, so a slow attempt consumes the budget
// instead of stacking retries on top of it. Raising the cap does not raise the
// success rate; it converts a fast failure into a caller-visible timeout.
// Do not change maxAttempts without re-running budget_test.go:88 (deadline
// crossing) and retry_test.go:231 (attempt cap). verified 2025-09-17 against
// gateway/timeouts.go:14 (caller budget 2s).
if attempts > maxAttempts {
    return err
}

// TestRetryCapBoundsCallerDeadlineNotDownstreamHealth — the name is the durable
// oracle: when this fails in two years, the failure states the invariant
// without the author.
func TestRetryCapBoundsCallerDeadlineNotDownstreamHealth(t *testing.T) { /* ... */ }
```

Reviewer's pass on the annotation: *the numbers are unaddressed — which budget, measured where, and what happens if I raise the cap?* The author's pass closes all three: the budget, the arithmetic, the failure path, the artifact that breaks if it changes, and a dated verification. Nothing in the final block requires the author's presence, and the guard comment now has a consumer — the next engineer who wants a higher cap.

### 3.2 PR Descriptions — the artifact read at rollback time

The PR body is never read by the author. It is read by the person reverting at 03:00, the next engineer in six months, and the incident reviewer with no access to the discussion that shaped it. A body that reproduces the diff has performed the author dependency at document scale: the information that mattered was in the room, and the body recorded the part that was already machine-readable.

```markdown
## What changed
Write-batch flushing now triggers on `min(interval, 512 records)` in `batcher.go:88`
instead of on the ticker alone.

## Why
A flush trigger IS the durability contract. With a time-only trigger the window is
bounded in seconds and unbounded in records, so a burst converts a latency guarantee
into an unbounded buffer. This makes the bound a function of the buffer — the only
quantity the writer controls.

## How to verify
`make bench-batcher WORKLOAD=burst-4k RUNS=5` → expect p99 write lag < 800 ms and
peak rows buffered == 512 (before: 3,140 ms / 12,480 rows on main @ 9f21c0d).

| Metric | main @ 9f21c0d | PR head | Δ |
|---|---|---|---|
| p99 write lag | 3,140 ms | 610 ms | −81% |
| peak rows buffered | 12,480 | 512 | −96% |
| write txn / 10k rows | 42 | 59 | +40% (expected) |

## Blast radius and rollback
Affects the write path of `reconciler` only; readers see smaller batches, never
partial ones. Rollback is one commit (`git revert <sha>`); no schema, config, or
feature-flag change is involved. The startup banner `batch_flush_watermark=512`
disappears when the revert is live.

## What would reverse this
If storage cost per transaction grows past the measured +40%, the smaller batches
stop paying for themselves. Signal: `write_txn_per_row` on the `reconciler`
dashboard above 1.5x baseline for a full week. Owner: storage on-call
(rotation `storage-primary`).

## Does not cover
Compaction scheduling and the read path; the watermark is not a throughput dial.
```

Every section answers a question the author would otherwise be asked at a worse time. The closing *does not cover* block is not boilerplate: a reader who does not know the limits of a change cannot safely rely on it.

### 3.3 Architecture RFCs / ADRs — decisions that must outlive their author

An ADR is a bet that a future maintainer can re-evaluate the decision without re-litigating it with someone who has left. A document valid only while its author is around is a memo, not a decision record; the gate is one question — *could a silent maintainer re-decide this correctly, or overturn it knowingly?*

| ADR section | Silent-author obligation |
|---|---|
| Context | The system property or invariant at stake, in the system's own vocabulary — no codenames, no *we* |
| Decision | One falsifiable commitment, stated so a reader can tell whether the code still matches it |
| Rationale | Why this mechanism works, and why the rejected alternatives do not |
| Preconditions & environment | Scale band, topology, dependency versions, and the state that must hold for the decision to be valid |
| Evidence | Benchmarks, traces, query counts, fixtures — with command, baseline, and workload |
| Reversal trigger | The signal that proves the decision wrong, its threshold, and the role that watches it |
| Re-evaluation | Interval, owner role, and the artifact that forces the check (upgrade, incident review, load test) |
| Does not cover | Explicit outer bounds, so the reader does not extend the decision into territory the author never examined |

```markdown
# ADR-021 — Watermark-bounded write batching

## Context
A batcher's flush trigger IS its durability contract: a time-only trigger bounds
latency in seconds and leaves record count unbounded.

## Decision
Flush on `min(interval, 512 records)` in `batcher.go:88`.

## Preconditions
Valid for the current single-writer partition assignment and datasets <= 10M rows
per tenant. Not valid under multi-writer assignment: the watermark then bounds one
writer's buffer, not the dataset's in-flight records.

## Evidence
`make bench-batcher WORKLOAD=burst-4k RUNS=5` · main @ 9f21c0d vs PR head ·
p99 lag 3,140 → 610 ms · peak rows 12,480 → 512 · txn/row +40% (expected price).

## Reversal trigger
`write_txn_per_row` > 1.5x baseline for one week, or storage cost per transaction
rising past the measured +40%. Watched by the storage on-call (rotation
`storage-primary`); reviewed each quarter by the platform role, not by the author.

## Does not cover
Compaction scheduling and the read path. The watermark is not a throughput dial.
```

The same standard governs the critical-path artifacts sitting beside an ADR. A runbook gets one additional clause: its first section is scope and its last is what it does not cover, because an on-call engineer at 03:00 needs to know within ten seconds whether this page is even theirs. Escalation is always a route — rotation, queue, target response — never a name; and a runbook counts as verified only once it has been executed cold, in staging, by someone who is not its author, with time-to-first-correct-action recorded.

---

## 4. Verification Checklist

- [ ] **No unresolved facts.** Every fact a reader needs has an address inside the artifact, a versioned repository path, or a durable route; no step depends on a conversation, a collapsed thread, a private channel, or a named individual — escalation appears as a role with a queue and a target response time.
- [ ] **Contract is self-contained.** Preconditions, environment, units, magnitudes, and the consequence of exceeding them are stated in the artifact itself, and the reader can determine from the artifact alone whether they are in the state the procedure assumes.
- [ ] **Failure path and bounds are explicit.** Each step carries expected output, normal duration, and the failure interpretation with the stop/retry/rollback/escalate decision; the artifact separately states when it does not apply and what signal invalidates it.
- [ ] **Decay is dated and pinned.** Versions, sample output, dashboard names, links, and thresholds carry a verification date and a re-verification trigger; nothing that can rot is presented as a timeless fact.
- [ ] **Silent read executed and recorded.** A reader with repository access and no author contact performed the artifact's job end-to-end, cold, on a clean environment, and every ambiguity found was patched into the artifact — with ambiguity count and time-to-first-correct-action recorded for critical-path artifacts (required budget: zero).