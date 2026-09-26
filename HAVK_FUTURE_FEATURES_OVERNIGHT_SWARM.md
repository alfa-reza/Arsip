# HAVK Future Features — Overnight & Swarm

> **Artifact type:** future-feature / product-direction specification  
> **Target:** HAVK  
> **Source baseline audited:** Jcode `0.88.0` source archive  
> **Status:** **DEFERRED / RETAIN FOR FUTURE — DO NOT DEBLOAT NOW**  
> **Decision date:** 2026-09-26  
> **Scope:** Overnight and Swarm only  
> **Important:** Overnight and Swarm are two separate future features. They are intentionally documented independently in this artifact.

---

# 1. Decision record

HAVK will **not remove Overnight or Swarm at this stage**.

They are moved out of the current debloat queue and into the **future HAVK feature roadmap**.

Current decision:

```yaml
future_features:
  overnight:
    status: deferred_keep
    debloat_now: false
    implementation_now: false
    future_direction: adapt_for_long_running_ctf_missions

  swarm:
    status: deferred_keep
    debloat_now: false
    implementation_now: false
    future_direction: adapt_for_parallel_ctf_specialist_agents
```

This does **not** mean HAVK must preserve the current Jcode UX or coding-specific semantics forever.

It means:

1. do not delete these subsystems during the current debloat;
2. avoid unnecessary invasive modifications while other core cleanup is ongoing;
3. revisit them later as dedicated HAVK features;
4. redesign coding-specific assumptions only when their future HAVK architecture is ready.

---

# 2. Why these features are being deferred instead of deleted

HAVK is moving away from Jcode's primary identity as a coding agent toward a tool-oriented security/CTF harness.

Many Jcode development features are therefore good candidates for removal. However, Overnight and Swarm contain general orchestration capabilities that are potentially much more useful than their current coding-specific presentation suggests.

The reusable concepts are:

```text
Overnight
  -> long-running autonomous work
  -> progress/lifecycle tracking
  -> automatic continuation
  -> deadline / cancellation
  -> structured final report

Swarm
  -> parallel agents
  -> coordinator/worker relationships
  -> task decomposition
  -> direct agent communication
  -> structured completion artifacts
  -> persistence/recovery
```

Those concepts map naturally to complex CTF/security tasks.

Therefore these features should not be deleted merely because upstream Jcode currently frames them around software repositories.

---

# PART A — OVERNIGHT

# 3. Overnight status

**Current HAVK decision:**

```text
KEEP IN SOURCE FOR NOW
DO NOT DEBLOAT
DO NOT PRIORITIZE IMPLEMENTATION/REWRITE YET
REVISIT AS A FUTURE HAVK LONG-RUNNING MISSION FEATURE
```

Overnight remains distinct from Swarm.

HAVK may eventually use Overnight without Swarm, Swarm without Overnight, or integrate them at a higher orchestration layer later.

No architectural dependency between the two should be assumed merely because they can complement each other.

---

# 4. What Jcode Overnight currently is

The main implementation surface audited includes:

```text
crates/jcode-overnight-core/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── prompts.rs
    └── helper_tests.rs

crates/jcode-app-core/src/overnight.rs
crates/jcode-tui/src/tui/app/commands_overnight.rs
```

Direct dedicated implementation is approximately:

```text
3,458 lines
```

Approximately 25 source/documentation files in the audited archive contain direct Overnight references.

Current commands include the equivalent of:

```text
/overnight <duration> [mission]
/overnight status
/overnight log
/overnight review
/overnight cancel
```

The core accepts a duration from one minute up to 72 hours in the audited version.

---

# 5. Valuable generic mechanisms inside Overnight

The important part of Overnight is not the word "overnight" and not its Git integration.

The reusable engine provides concepts such as:

## 5.1 Long-running autonomous lifecycle

A task can remain active for many turns rather than ending after a single model response.

Conceptually:

```text
start mission
    ↓
agent works
    ↓
turn ends
    ↓
engine decides work remains
    ↓
automatic continuation / poke
    ↓
agent continues
    ↓
repeat until completion, cancellation, fatal failure, or deadline
```

This solves a real problem for security research: the user should not have to manually type "continue" every time a model turn ends during a multi-hour investigation.

## 5.2 Duration and deadline control

The subsystem tracks:

- start time;
- requested duration;
- target wake/report time;
- elapsed time;
- progress;
- end-of-run behavior;
- cancellation.

## 5.3 Preflight checks

Current upstream preflight records:

- provider/model usage state;
- projected quota risk;
- host system resources;
- Git state.

The mechanism is useful even though the data model should change for HAVK.

## 5.4 Structured progress and review

Upstream maintains run artifacts such as:

```text
task cards
validation results
event log
run log
review output
final/morning report
```

This is valuable for long-running investigations where the user needs a concise explanation of what happened while they were away.

---

# 6. Why upstream Overnight is not ready for HAVK as-is

The current Jcode prompts are explicitly coding/repository oriented.

They instruct the coordinator to perform work such as:

```text
inspect repo/session state
inspect git status
find bugs
reproduce failures
fix bugs
write regression tests
perform bounded code-quality improvements
validate code changes
```

Current task-card structures contain concepts including:

```text
problem
change
files_changed
validation.commands
regression-style evidence
```

This is appropriate for an autonomous coding agent but is not an ideal native model for a CTF harness.

Therefore the future HAVK work should be a **semantic adaptation**, not simply exposing Jcode's existing `/overnight` unchanged.

---

# 7. Future HAVK Overnight direction

Working concept only — naming is **not yet decided**.

Possible names include:

```text
Overnight
Mission
Mission Mode
Long Run
Investigation
```

Do not rename the subsystem during the current debloat just to anticipate this future decision.

The future abstraction should be closer to:

```text
HAVK long-running mission
        │
        ├── objective
        ├── evidence
        ├── hypotheses
        ├── tool executions
        ├── cloud sandbox jobs
        ├── findings
        ├── validation
        ├── blockers
        └── final report
```

---

# 8. Suggested future Overnight task model for CTF/security

Instead of upstream's coding-oriented task card:

```text
problem
files changed
code change
regression test
```

HAVK could eventually use something like:

```yaml
mission_task:
  objective: "Recover the hidden payload from image.png"
  category: "forensics"
  status: "completed"

  evidence:
    - "Unexpected bytes after PNG IEND"
    - "binwalk identifies embedded ZIP"

  hypotheses:
    - text: "Payload is appended after IEND"
      confidence: high

  actions:
    - tool: exiftool
    - tool: binwalk
    - tool: unzip

  artifacts:
    - recovered/payload.zip
    - recovered/note.txt

  validation:
    result: confirmed
    evidence:
      - "ZIP extracts without error"

  findings:
    - "Embedded archive recovered"

  open_questions: []
```

This is only a future design direction, not a current implementation requirement.

---

# 9. Future cloud-sandbox relationship for Overnight

HAVK's planned security-tool execution model should allow external isolated compute such as Modal, E2B, or another provider-neutral cloud sandbox.

Future Overnight should therefore distinguish:

```text
orchestrator state
        vs
execution environment state
```

Possible future architecture:

```text
HAVK Overnight / Mission Coordinator
              │
              ├── model/provider budget
              ├── mission lifecycle
              ├── evidence graph
              ├── progress/reporting
              │
              └── sandbox execution abstraction
                     ├── E2B
                     ├── Modal
                     └── future providers
```

The current Jcode local resource monitor is not sufficient for that architecture.

A future HAVK preflight could eventually track:

- available cloud execution provider;
- sandbox CPU/memory limits;
- sandbox lifetime/timeout;
- cloud budget/credits;
- number of running sandboxes;
- artifact storage availability;
- network policy;
- tool/image availability.

This work belongs to a later cloud-sandbox architecture phase.

---

# 10. Overnight and Git

Overnight does currently contain direct Git integration.

The audited implementation invokes Git for snapshot information, including behavior equivalent to:

```text
git branch --show-current
git status --short
```

and stores a structure similar to:

```rust
GitSnapshot {
    branch,
    dirty_count,
    dirty_summary,
    error,
}
```

However, Git is **not a hard requirement for the fundamental Overnight lifecycle engine**.

If Git is removed from HAVK later, this portion can be replaced with a generic workspace/mission snapshot or removed.

Possible future HAVK equivalent:

```rust
WorkspaceSnapshot {
    working_directory,
    artifacts,
    active_sandboxes,
    evidence_count,
    last_activity,
}
```

Therefore:

```text
KEEP Overnight
+
FUTURE Git debloat
```

is considered architecturally compatible.

Overnight must **not** be used as a reason to retain Git automatically.

---

# 11. Overnight future acceptance goals

When HAVK eventually revisits this feature, desirable goals include:

- [ ] long-running missions survive ordinary UI disconnects;
- [ ] mission state is persistent;
- [ ] automatic continuation is bounded and observable;
- [ ] user can inspect status/logs while work continues;
- [ ] mission can be cancelled safely;
- [ ] no requirement for Git;
- [ ] no assumption that the target is source code;
- [ ] no default regression-test / GitHub-issue language;
- [ ] structured evidence and findings replace code-change-centric reporting;
- [ ] cloud sandbox execution can be tracked separately from the coordinator;
- [ ] expensive/unsafe CTF tools can run outside the local host;
- [ ] final report clearly distinguishes confirmed findings from hypotheses;
- [ ] future Swarm integration remains optional rather than mandatory.

---

# PART B — SWARM

# 12. Swarm status

**Current HAVK decision:**

```text
KEEP IN SOURCE FOR NOW
DO NOT DEBLOAT
DO NOT TREAT AS CURRENT IMPLEMENTATION PRIORITY
REVISIT AS A FUTURE PARALLEL CTF AGENT FEATURE
```

Swarm is documented separately from Overnight because it solves a different problem.

```text
Overnight = duration/lifecycle/autonomous continuation
Swarm     = parallel multi-agent coordination
```

They can complement one another later, but neither should be defined as a required implementation detail of the other.

---

# 13. What Jcode Swarm currently is

Upstream Jcode describes Swarm as a server-managed mechanism for running multiple collaborating agents in parallel.

Official Jcode material describes capabilities including:

- multiple agents working concurrently;
- coordinator/worker relationships;
- direct messages;
- subtree broadcast;
- structured task graph nodes;
- completion artifacts;
- file-change awareness;
- shared server state.

The audited source contains a significant implementation surface.

Representative files include:

```text
crates/jcode-swarm-core/
crates/jcode-app-core/src/server/swarm.rs
crates/jcode-app-core/src/server/swarm_channels.rs
crates/jcode-app-core/src/server/swarm_mutation_state.rs
crates/jcode-app-core/src/server/swarm_persistence.rs
crates/jcode-app-core/src/server/debug_swarm_read.rs
crates/jcode-app-core/src/server/debug_swarm_write.rs
crates/jcode-tui/src/tui/app/remote/swarm_plan_core.rs
crates/jcode-tui/src/tui/app/remote/swarm_status_core.rs
crates/jcode-tui/src/tui/info_widget_swarm_*.rs
crates/jcode-tui/src/tui/swarm_plan_graph.rs
crates/jcode-tui-render/src/swarm_gallery.rs
crates/jcode-tui-render/src/swarm_tiles.rs
```

A narrow count of the dedicated Swarm-named core/server/UI files audited is already approximately:

```text
13,496 lines
```

and direct `swarm` references occur across roughly:

```text
277 source/documentation files
```

This means Swarm is a major Jcode subsystem, not a small wrapper around spawning sessions.

---

# 14. Important reusable Swarm concepts

## 14.1 Coordinator and worker hierarchy

Upstream supports a root coordinator and spawned workers.

In deep mode, descendants can themselves spawn further workers subject to bounds.

The server tracks parent/report-back relationships, allowing it to reconstruct ancestry and worker ownership.

This is valuable for HAVK because specialist security agents could be spawned under a controlling coordinator without exposing every low-level interaction to the user.

## 14.2 Agent lifecycle

Upstream defines explicit lifecycle states broadly equivalent to:

```text
spawned
ready
running
blocked
completed
failed
stopped
crashed
```

A security-oriented swarm also needs these states.

## 14.3 Structured completion reports

Workers are expected to return useful final reports including:

- outcome/status;
- findings or changes;
- validation performed;
- blockers/follow-ups.

The completion is automatically delivered to the owner/coordinator.

This maps well to CTF/research workers.

## 14.4 Agent communication

The architecture supports:

- direct messages;
- subtree broadcasts;
- topic channels;
- shared context mechanisms;
- lifecycle notifications.

Notifications can be injected as safe interrupts while another agent is active.

## 14.5 Shared plans / task graph

Modern upstream Swarm increasingly uses a DAG/task-graph model rather than simply "N agents in a repo".

The audited core includes concepts such as:

```text
assign task
expand node
complete node
inject gap
verification/critique gates
typed completion artifact
confidence
what-was-not-checked
```

This is particularly interesting for HAVK because CTF investigations naturally branch into parallel hypotheses.

---

# 15. Why Swarm is potentially valuable to HAVK

Security/CTF problems are frequently search-space problems.

One agent following a single hypothesis can spend a long time going down the wrong path.

A future HAVK Swarm can deliberately explore different hypotheses in parallel.

Example:

```text
HAVK Coordinator
│
├── Forensics worker
│   ├── metadata
│   ├── embedded files
│   └── steganography
│
├── Reverse worker
│   ├── strings/imports
│   ├── static analysis
│   └── decompilation
│
├── Crypto worker
│   ├── cipher identification
│   └── key-material analysis
│
└── Network worker
    ├── PCAP analysis
    └── protocol reconstruction
```

The coordinator could then compare completion artifacts rather than asking one model to serially explore every branch.

---

# 16. Future HAVK Swarm should not simply preserve Jcode's coding assumptions

Current upstream documentation frequently frames Swarm around:

```text
same repository
code changes
file conflicts
Git worktrees
merge/integration
```

For HAVK the more important shared objects should eventually be:

```text
mission
challenge/evidence workspace
artifacts
hypotheses
findings
sandbox jobs
shared observations
```

Source-code collaboration may still be useful for exploit scripting or PoCs, but it should not define the whole Swarm architecture.

---

# 17. Suggested future HAVK specialist roles

The exact roles should remain dynamic, but possible examples include:

```text
recon
web
pwn
reverse-engineering
forensics
crypto
network/pcap
malware
OSINT
cloud
binary triage
artifact validation
```

A coordinator should create only the workers useful for the current challenge rather than always spawning a fixed team.

For example:

```text
PCAP challenge
    ↓
Coordinator decides:
    ├── Network analyst
    ├── Protocol analyst
    └── Artifact extraction worker
```

A crypto-only challenge may require only two competing hypothesis workers.

---

# 18. Future Swarm + cloud sandbox model

A future HAVK Swarm should be compatible with the planned provider-neutral sandbox architecture.

A useful separation is:

```text
Agent logical identity
        ≠
Sandbox runtime identity
```

For example:

```text
Coordinator
│
├── Agent A
│    └── E2B sandbox A
│
├── Agent B
│    └── Modal sandbox/job B
│
└── Agent C
     └── isolated sandbox C
```

or several read-only analysis agents could share a safe artifact store while dangerous execution remains isolated.

The Swarm coordinator should eventually understand:

- worker model/provider;
- worker specialization;
- sandbox allocation;
- resource budget;
- artifact ownership;
- cross-worker evidence sharing;
- task status;
- completion confidence.

None of this needs to be implemented during the current debloat.

---

# 19. Swarm and Git

This is important for HAVK's later Git-debloat discussion.

The upstream **Swarm Architecture** explicitly describes Git worktrees as **optional**.

The audited dedicated Swarm core/server code did not show a direct `Command::new("git")` invocation analogous to Overnight's explicit Git snapshot command.

Swarm does have concepts tied to coding repositories, such as:

- repo-scoped swarm identity;
- file-read/file-touch awareness;
- conflict notification;
- optional Git worktree managers;
- code integration language in documentation.

But those concepts are separable from the fundamental multi-agent coordination engine.

Therefore the current HAVK direction is:

```text
DO NOT keep Git merely because Swarm exists.
```

During a future Swarm redesign, Git/worktree-specific logic can be:

1. removed;
2. made an optional adapter;
3. retained only for exploit/code-development workloads where useful.

The core future HAVK Swarm should rely on mission/task/artifact identity rather than requiring a Git repository.

---

# 20. Future Swarm task artifact model

The existing deep-swarm design already requires typed artifacts with concepts such as findings, evidence, validation, open questions, confidence, and explicitly stating what was not checked.

That is unusually compatible with security work.

A future HAVK worker artifact could evolve toward:

```yaml
worker_result:
  task_id: reverse.entrypoint
  specialist: reverse
  status: completed

  findings:
    - "Binary validates a 32-byte token before decrypting payload"

  evidence:
    - artifact: challenge.bin
      location: "function sub_4018A0"
    - artifact: strings.txt
      observation: "AES-related constants present"

  validation:
    - "Confirmed control-flow branch in disassembler"

  confidence: high

  open_questions:
    - "Key derivation source not yet identified"

  what_i_did_not_check:
    - "Dynamic behavior under debugger"
```

This type of explicit uncertainty is preferable to workers returning only free-form chat.

---

# 21. Future Swarm guardrails

Parallelism is useful but can also waste model tokens and sandbox resources.

A future HAVK implementation should preserve or strengthen bounds such as:

- [ ] configurable maximum live workers;
- [ ] global mission resource budget;
- [ ] maximum recursion/depth;
- [ ] coordinator authority over shared plan changes;
- [ ] worker timeout;
- [ ] explicit stop/cancel;
- [ ] deduplication of identical work;
- [ ] typed completion artifact requirement;
- [ ] clear confidence and validation reporting;
- [ ] cloud sandbox concurrency limit;
- [ ] graceful worker crash recovery;
- [ ] no uncontrolled exponential agent spawning.

---

# 22. Overnight and Swarm — possible future composition

Although they are separate features, future HAVK could optionally compose them.

Example:

```text
Overnight / Mission Engine
    │
    │ owns:
    │ - duration
    │ - deadline
    │ - progress
    │ - persistence
    │ - final report
    │
    └── may request Swarm
            │
            │ owns:
            │ - parallel task graph
            │ - worker lifecycle
            │ - communication
            │ - completion artifacts
            │
            ├── Forensics worker
            ├── Reverse worker
            └── Crypto worker
```

This relationship should remain optional.

Examples:

```text
short parallel task
    -> Swarm only

single-agent six-hour investigation
    -> Overnight only

six-hour multi-discipline CTF investigation
    -> Overnight + Swarm
```

This separation prevents either subsystem from becoming unnecessarily coupled to the other.

---

# 23. What NOT to do during current debloat

Until these future features are actively redesigned:

## Overnight

Do not:

- delete the Overnight crate;
- remove command wiring merely because it is coding-oriented;
- heavily rewrite prompts as part of an unrelated debloat;
- make Git debloat contingent on keeping the current Git snapshot implementation;
- rename it prematurely.

## Swarm

Do not:

- delete `jcode-swarm-core`;
- remove server swarm persistence/state machinery;
- remove worker/coordinator lifecycle support;
- remove communication/task graph primitives;
- delete it merely because upstream documentation talks about repositories;
- assume optional Git worktrees mean Git must remain in HAVK.

Changes needed purely for compilation after another approved debloat should be minimal and should preserve the feature boundary where practical.

---

# 24. Interaction with approved debloat decisions

Current approved removals include upstream functionality such as Jcode Remote Compile and Self-Dev.

Those removals should **not** be interpreted as permission to remove Overnight or Swarm.

In particular:

```text
Remove Self-Dev
    != Remove Swarm
    != Remove Overnight

Remove Jcode Remote Compile
    != Remove cloud sandbox architecture
    != Remove Swarm workers
    != Remove Overnight missions
```

If shared code is encountered, refactor the shared primitive rather than deleting Overnight/Swarm functionality by accident.

---

# 25. Priority

Neither feature is a current HAVK MVP requirement.

Recommended high-level priority:

```text
Current phase
    1. finish core Jcode debloat
    2. stabilize HAVK
    3. establish provider-neutral cloud sandbox/tool execution
    4. remove/refactor remaining coding-only assumptions

Future phase
    5. revisit Swarm architecture for CTF/security
    6. revisit Overnight as long-running Mission orchestration
    7. optionally compose both
```

The exact ordering of steps 5 and 6 is not fixed by this artifact.

---

# 26. Research notes / upstream documentation

The following upstream materials were used to understand the intent of the retained features:

## Jcode official documentation

- Jcode documentation: `https://jcode.sh/docs`
  - exposes Swarm as a configurable feature;
  - documents `[agents]` as controlling swarm routing/spawn/concurrency behavior;
  - documents project/global `swarm-prompt.md` files;
  - documents the client/server model that allows headless work to survive client disconnects.

- Jcode Swarm page: `https://jcode.sh/swarm`
  - describes Swarm as parallel search of solution space;
  - describes server-managed multi-agent coordination;
  - describes DM/broadcast/task-graph coordination and file-change awareness.

## Source documentation

- `docs/SWARM_ARCHITECTURE.md`
- `docs/SWARM_TASK_GRAPH.md`
- `crates/jcode-overnight-core/src/lib.rs`
- `crates/jcode-overnight-core/src/prompts.rs`
- `crates/jcode-app-core/src/overnight.rs`
- `crates/jcode-swarm-core/src/lib.rs`
- `crates/jcode-app-core/src/server/swarm.rs`
- `crates/jcode-app-core/src/server/swarm_persistence.rs`

Upstream `SWARM_ARCHITECTURE.md` is particularly important because it explicitly states that Git worktrees are optional, while the newer task-graph direction separates coordination state from repository files.

---

# 27. Final future-feature classification

## Overnight

```yaml
name: Overnight
havk_classification: future_feature
keep_now: true
debloat_now: false
current_upstream_fit_for_ctf: low
future_engine_value_for_havk: high
major_future_work:
  - remove coding-centric mission semantics
  - replace Git-centric preflight
  - add CTF evidence/hypothesis model
  - integrate provider-neutral cloud sandbox telemetry
  - preserve persistence/cancel/progress/report lifecycle
```

## Swarm

```yaml
name: Swarm
havk_classification: future_feature
keep_now: true
debloat_now: false
current_upstream_fit_for_ctf: medium
future_engine_value_for_havk: very_high
major_future_work:
  - move from repo-centric to mission-centric coordination
  - keep task graph and structured completion artifacts
  - add CTF/security specialist roles
  - make Git/worktree behavior optional or removable
  - integrate provider-neutral sandbox allocation
  - enforce model/compute concurrency budgets
```

---

# 28. Decision summary

**Overnight is crossed off the current debloat candidate list.**

It remains in HAVK as a deferred feature because its long-running autonomous lifecycle may later become a CTF-oriented Mission Engine.

**Swarm is also classified as a deferred HAVK future feature.**

Its parallel-agent coordination, task graph, lifecycle state, communications, persistence, and structured completion reporting have strong potential for security/CTF workloads even though upstream Jcode presents them primarily as coding collaboration.

The two features must remain **separate architectural concepts**:

```text
Overnight = long-running autonomous lifecycle
Swarm     = parallel multi-agent coordination
```

They may be composed in the future, but neither is to be deleted, renamed, or deeply redesigned during the current debloat phase.
