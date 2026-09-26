# HAVK Debloat Plan — Phase 3
## Remove Jcode Remote Compile + Remove Self-Dev, Preserve Future Cloud Sandbox Capability

> **Source baseline:** `jcode-0.88.0.tar.gz`  
> **Target fork:** HAVK  
> **Status:** Approved removal scope  
> **Decision date:** 2026-09-26  
> **Scope:** Jcode-owned remote compile/cloud-compute path and Jcode Self-Dev infrastructure  
> **Explicitly out of scope:** Overnight mode, Swarm, generic remote server use, future Modal/E2B/cloud-sandbox integration, generic server reload/reconnect

---

# 1. Executive decision

Two debloat candidates are now approved for HAVK:

| Component | Decision | Removal safety | Difficulty | Reason |
|---|---|---:|---:|---|
| Jcode `compile_remote` / Jcode cloud-compute path | **REMOVE** | **5/5** | **2/5** | It is a Jcode account/subscription-backed build service, not a generic sandbox abstraction |
| Jcode Self-Dev | **REMOVE** | **4/5** | **5/5** | HAVK prioritizes runtime stability and CTF tool use rather than editing/building/reloading itself |

The two decisions share one important architectural principle:

> **HAVK should remove upstream Jcode-owned development infrastructure without removing the ability to execute tools inside external cloud sandboxes.**

The intended long-term direction is:

```text
HAVK harness
    |
    +-- local orchestration / TUI / model / swarm
    |
    +-- tool execution target
          |
          +-- future Modal Sandbox adapter
          +-- future E2B Sandbox adapter
          +-- future other cloud-sandbox adapter
          +-- optional user-controlled VPS/container runtime
```

Not:

```text
HAVK
  -> Jcode account
  -> Jcode subscription / compute credits
  -> api.jcode.sh
  -> Jcode backend
  -> Jcode-selected E2B sandbox
```

This distinction is mandatory during implementation.

---

# 2. Research basis

This plan is based on direct inspection of the uploaded Jcode 0.88.0 source archive and current official documentation.

## 2.1 Jcode source/documentation inspected

Primary files:

```text
docs/REMOTE_COMPILE.md
docs/REMOTE_COMPILE_VALIDATION.md
docs/SERVER_ARCHITECTURE.md
README.md
AGENTS.md

crates/jcode-app-core/src/tool/compile_remote.rs
crates/jcode-app-core/src/tool/compile_remote/source.rs
crates/jcode-app-core/src/tool/compile_remote/tests.rs
crates/jcode-app-core/src/agent_tests/compile_remote.rs

src/cli/selfdev.rs
src/cli/selfdev_tests.rs
crates/jcode-app-core/src/tool/selfdev/
crates/jcode-app-core/src/tool/desktop_selfdev.rs
crates/jcode-selfdev-types/
crates/jcode-build-support/
```

Jcode's public README describes Self-Dev as infrastructure that allows Jcode to modify its own source, build/test itself, reload its own binary, and continue existing sessions.

Jcode's server architecture documentation separately describes generic `/reload`, reconnect, client/server behavior, and Self-Dev session behavior. This separation is important: generic reload/reconnect is not synonymous with Self-Dev.

## 2.2 Modal official documentation

Official references:

- https://modal.com/docs
- https://modal.com/docs/sdk/py/latest/Sandbox
- https://modal.com/docs/sdk/js/latest/Sandbox
- https://modal.com/docs/guide/useful-snippets

Modal documents `Sandbox` as an isolated environment for arbitrary/untrusted code and exposes primitives useful to HAVK such as:

- selectable CPU/memory;
- images;
- environment variables/secrets;
- filesystem operations;
- command execution;
- working directories;
- process stdout/stderr;
- timeout/lifecycle management;
- optional GPU resources;
- networking controls.

Modal also documents GitHub Actions/CI deployment workflows.

Therefore removing Jcode's `compile_remote` does **not** remove HAVK's ability to use Modal later.

## 2.3 E2B official documentation

Official references:

- https://docs.e2b.dev/
- https://e2b.dev/security

E2B documents its Sandbox as an on-demand Linux VM for agent code/tool execution, with SDK-controlled command execution and filesystem access. Its documentation also explicitly includes a GitHub Actions CI/CD use case.

This is directly compatible with the planned HAVK direction where CTF tools can run remotely instead of on the local machine.

## 2.4 Important upstream finding: Jcode Remote Compile already uses a third-party sandbox backend

`docs/REMOTE_COMPILE.md` states that Jcode's backend executes remote compilation through an isolated third-party sandbox provider and identifies the **initial adapter as E2B**.

That means current Jcode architecture is effectively:

```text
Jcode client
    -> Jcode authenticated API
        -> entitlement / credit ledger / capacity admission
            -> third-party sandbox provider (initially E2B)
```

HAVK does not need the Jcode commercial middle layer merely to gain cloud sandbox execution.

A future HAVK architecture can instead be:

```text
HAVK
    -> user-selected sandbox adapter
        -> Modal / E2B / other provider
```

This is a major reason why deleting `compile_remote` is safe for the long-term cloud-execution goal.

---

# 3. Approved removal A — Jcode Remote Compile / Jcode Cloud Compute

## 3.1 What `compile_remote` actually is

The official source documentation states that `compile_remote`:

1. checks Jcode account/subscription access;
2. reads a local Git checkout;
3. creates a bounded source snapshot;
4. uploads it to Jcode's authenticated API;
5. submits a compile command;
6. Jcode's backend reserves cloud-compute credits;
7. the backend provisions an isolated Linux sandbox;
8. compiler stdout/stderr/exit status and metered usage are returned.

The tool does **not** directly expose E2B or a generic cloud sandbox provider to the Jcode client.

The client talks to Jcode's own API endpoints, including flows equivalent to:

```text
GET  /v1/me
GET  /v1/compute/usage
POST /v1/compile
```

This is therefore a hosted product integration rather than a reusable cloud-execution abstraction.

---

# 4. Why it should be removed from HAVK

HAVK's intended runtime model is different.

CTF workloads may need:

```text
binwalk
volatility
exiftool
steghide
gdb
radare2 / rizin
Ghidra helper automation
gcc / clang
Python exploit tooling
network analysis tools
custom challenge binaries
```

The requirement is **isolated compute**, not specifically Jcode's paid remote compiler.

A generic sandbox is more useful than a compiler-only product because HAVK needs arbitrary tool execution.

For example:

```text
Jcode compile_remote
    command = cargo check
    source snapshot = Git repo
```

is narrower than:

```text
HAVK cloud sandbox
    upload challenge artifact
    install/use CTF image
    execute arbitrary approved tools
    inspect output/artifacts
    persist or destroy environment
```

Deleting `compile_remote` therefore does not close an architectural door. It removes the wrong abstraction for HAVK.

---

# 5. Source footprint — Remote Compile

The direct implementation/documentation audited totals approximately **1,979 lines**:

```text
450  crates/jcode-app-core/src/tool/compile_remote.rs
586  crates/jcode-app-core/src/tool/compile_remote/source.rs
696  crates/jcode-app-core/src/tool/compile_remote/tests.rs
 70  crates/jcode-app-core/src/agent_tests/compile_remote.rs
114  docs/REMOTE_COMPILE.md
 63  docs/REMOTE_COMPILE_VALIDATION.md
```

Only about **9 files** contain direct `compile_remote`/remote-compile references in the audited source tree.

This relative isolation supports the earlier difficulty rating of **2/5**.

---

# 6. Remote Compile — high-confidence deletion list

## Delete

```text
crates/jcode-app-core/src/tool/compile_remote.rs
crates/jcode-app-core/src/tool/compile_remote/source.rs
crates/jcode-app-core/src/tool/compile_remote/tests.rs
crates/jcode-app-core/src/agent_tests/compile_remote.rs
docs/REMOTE_COMPILE.md
docs/REMOTE_COMPILE_VALIDATION.md
```

## Edit

```text
crates/jcode-app-core/src/tool/mod.rs
crates/jcode-app-core/src/agent_tests.rs
crates/jcode-app-core/src/agent/turn_execution.rs
```

### `tool/mod.rs`

Remove:

```text
mod compile_remote;
```

Remove registration of:

```text
compile_remote
```

Remove the dynamic helper that refreshes subscription-aware tool definitions, currently represented by behavior such as:

```text
remote_compile_definition()
compile_remote::refresh_access()
```

That dynamic schema refresh exists because account/subscription state changes the tool guidance. Once the tool is gone, the machinery is unnecessary.

### `agent/turn_execution.rs`

Remove the special case that refreshes the locked `compile_remote` definition after account entitlement changes.

Do not disturb generic locked tool definitions or MCP behavior.

### `agent_tests.rs`

Remove module wiring for:

```text
agent_tests/compile_remote.rs
```

Any generic custom-tool schema test that happens to use the string `compile_remote` merely as a dummy tool name should be renamed to a neutral dummy name rather than deleted if the test still validates generic SDK behavior.

---

# 7. Remove Jcode cloud-compute commercial coupling

Remote Compile is tied to upstream commercial/account concepts such as:

```text
Jcode subscription
Jcode account login
cloud-compute credits
compute usage ledger
paid entitlement
Jcode account API
Jcode backend capacity admission
```

Phase 2 already removes Jcode hosted-model subscription infrastructure.

Phase 3 should ensure no Remote Compile-specific references keep those concepts alive solely for compilation.

Examples to remove if they become unreferenced:

```text
remote_compile entitlement checks
remote_compile compute-credit guidance
compile-specific account status text
compile-specific pricing links
compile-specific telemetry/status fields
```

Do not remove unrelated provider usage accounting merely because the word `usage` appears.

---

# 8. Git dependency removed by Remote Compile

The Remote Compile source snapshotter is explicitly Git-based.

It uses commands equivalent to:

```text
git rev-parse --show-toplevel
git ls-files --cached --others --exclude-standard
```

The official docs require `path` to identify a Git repository root.

The snapshot includes tracked files and nonignored untracked files while excluding `.git`, common build outputs, symlinks, and common credential files.

Therefore deleting Remote Compile removes one concrete reason HAVK needs Git at runtime.

This is strategically useful because the broader Git debloat is intentionally postponed until later.

We should **not** implement a replacement filesystem snapshotter for `compile_remote` now, because the entire tool is being removed.

---

# 9. Critical boundary: do NOT delete cloud sandbox capability

The following product capability is explicitly preserved as a future HAVK direction:

> **HAVK may run CTF tools in external isolated cloud sandboxes instead of the local machine.**

Potential providers include:

```text
Modal Sandbox
E2B Sandbox
other compatible cloud sandbox services
user-managed VPS/container infrastructure
```

## Do NOT interpret this phase as

```text
remove all things named "sandbox"
remove remote execution
remove shell abstraction
remove file upload/download capability
remove client-server remote usage
forbid future cloud execution
```

That would be incorrect.

---

# 10. Important false positives containing the word `sandbox`

The Jcode source uses `sandbox` for several unrelated concepts.

Do not delete these merely because Jcode Cloud Compile is removed.

Examples:

```text
auth test sandbox
onboarding sandbox
sandboxed JCODE_HOME
provider auth test fixtures
project/local safety sandbox terminology
browser sandbox terminology
Google sandbox OAuth endpoints
```

For example:

```text
docs/ONBOARDING_SANDBOX.md
crates/jcode-base/src/auth/test_sandbox.rs
```

are test/state-isolation mechanisms, not the Jcode cloud compile backend.

They should remain unless independently debloated later.

---

# 11. Preserve generic remote Jcode/HAVK operation

Jcode's official docs describe a client-server architecture where the daemon can run on another machine and the TUI connects via SSH/socket forwarding.

That is useful independently of `compile_remote`.

Do not delete generic functionality such as:

```text
serve
Unix socket client/server
remote working directory
SSH socket forwarding compatibility
session persistence
reconnect logic
WebSocket gateway / paired client infrastructure
```

Removing Jcode Cloud Compile must not turn HAVK into a local-only application.

This is especially important because a future HAVK sandbox architecture may run the actual tool-execution daemon inside a remote/cloud environment.

---

# 12. Future cloud-sandbox architecture — preservation requirements

No Modal/E2B adapter needs to be implemented in this debloat phase.

However, implementation must avoid making future integration unnecessarily hard.

A future provider-neutral interface could conceptually look like:

```text
SandboxProvider
    create(spec) -> SandboxHandle
    exec(handle, command, cwd, env, timeout)
    upload(handle, files)
    download(handle, paths)
    status(handle)
    terminate(handle)
```

A `SandboxSpec` might eventually describe:

```text
CPU
RAM
GPU optional
image/template
network policy
timeout
working directory
secrets
persistent volume optional
CTF tool profile
```

This is intentionally **not** part of current implementation scope.

The only rule now is:

> Do not retain Jcode `compile_remote` merely because HAVK will later use cloud sandboxes.

The future abstraction should be designed for HAVK's arbitrary CTF tools, not inherited from Jcode's compile-only subscription API.

---

# 13. Why Modal/E2B fit the future direction

## Modal

Modal's official Sandbox API exposes arbitrary command execution in isolated containers and allows explicit resource selection such as CPU and memory. It also supports images, filesystem operations, secrets, working directories, lifecycle controls, and optional GPU resources.

This is more appropriate for HAVK than a hard-coded remote compiler because different CTF categories have different runtime requirements.

Examples:

```text
forensics -> memory-heavy image analysis
reversing -> compiler/debugger/tool image
pwn -> gcc/gdb/pwntools environment
crypto -> Python/Sage-like environment
AI-assisted tooling -> optional GPU workload
```

## E2B

E2B's official documentation describes on-demand Linux sandbox VMs for agents, command execution, filesystem access, templates, persistence/pause-resume, and GitHub Actions CI/CD scenarios.

Jcode's own remote compile documentation already names E2B as the initial backend adapter on the Jcode server side.

Therefore direct/provider-neutral E2B integration in HAVK is a realistic future direction without preserving Jcode's account/billing proxy.

---

# 14. Approved removal B — Self-Dev

## 14.1 Product decision

HAVK will **not support a mode where HAVK autonomously modifies, builds, tests, publishes, reloads, and resumes itself from its own source checkout.**

The priority is:

```text
stable harness
predictable release binary
external development workflow
CI-validated changes
```

not:

```text
HAVK edits HAVK
HAVK compiles HAVK
HAVK swaps its own executable
HAVK hot-reloads itself
HAVK resumes agent work on its new build
```

---

# 15. What upstream Self-Dev actually does

Jcode public documentation describes Self-Dev as a major customization feature rather than a small developer flag.

The audited implementation includes behavior such as:

1. detect whether Jcode is being run from a Jcode repository;
2. automatically enable Self-Dev mode unless disabled;
3. clone Jcode source if a local checkout is unavailable;
4. create/mark a canary Self-Dev session;
5. inject Self-Dev-specific prompts/tools;
6. compile the Jcode binary using a dedicated `selfdev` Cargo profile;
7. fingerprint the source tree using Git;
8. publish versioned/current/canary binaries;
9. coordinate build queues/locks;
10. reload the shared server;
11. reconnect existing clients;
12. validate a pending activation;
13. roll back failed Self-Dev activations;
14. expose Self-Dev status/debug socket operations;
15. support a Desktop-specific Self-Dev workflow.

This is a substantial self-hosting subsystem.

---

# 16. Source footprint — Self-Dev

Dedicated Self-Dev files audited total approximately **6,657 lines**.

More importantly, direct textual references to Self-Dev concepts occur across approximately **216 files** in the source/docs tree.

The largest reference concentrations are approximately:

```text
69 files  crates/jcode-app-core
38 files  crates/jcode-tui
18 files  src/cli
18 files  crates/jcode-base
 9 files  crates/jcode-setup-hints
 5 files  crates/jcode-telemetry-core
 5 files  crates/jcode-protocol
 5 files  crates/jcode-build-support
```

This confirms the **5/5 difficulty** rating.

The feature is not hard because one algorithm is complex. It is hard because Self-Dev state crosses many architectural layers.

---

# 17. High-confidence Self-Dev files to delete

## CLI

Delete:

```text
src/cli/selfdev.rs
src/cli/selfdev_tests.rs
```

Remove the corresponding module declaration and command dispatch.

## Core Self-Dev tool

Delete:

```text
crates/jcode-app-core/src/tool/selfdev/mod.rs
crates/jcode-app-core/src/tool/selfdev/build_queue.rs
crates/jcode-app-core/src/tool/selfdev/launch.rs
crates/jcode-app-core/src/tool/selfdev/reload.rs
crates/jcode-app-core/src/tool/selfdev/setup.rs
crates/jcode-app-core/src/tool/selfdev/status.rs
crates/jcode-app-core/src/tool/selfdev/tests.rs
```

## Desktop Self-Dev

Delete:

```text
crates/jcode-app-core/src/tool/desktop_selfdev.rs
crates/jcode-app-core/src/tool/desktop_selfdev_tests.rs
crates/jcode-app-core/src/agent_tests/desktop_selfdev.rs
```

## Self-Dev prompt assets

Delete:

```text
crates/jcode-base/src/prompt/selfdev_mode.txt
crates/jcode-base/src/prompt/desktop_selfdev_mode.txt
crates/jcode-base/src/prompt/selfdev_focus_tui.txt
```

Remove `include_str!` declarations and prompt assembly branches referring to them.

## Dedicated type crate

Candidate for full deletion after all consumers are removed:

```text
crates/jcode-selfdev-types/
```

Remove it from:

```text
workspace members
workspace dependencies
crate manifests
```

Do not delete it first. Remove consumers, then confirm that no generic non-Self-Dev lifecycle types still need to be moved elsewhere.

---

# 18. Remove Self-Dev CLI surface

Edit:

```text
src/cli/args.rs
src/cli/dispatch.rs
src/cli/mod.rs
```

Remove commands/flags such as:

```text
jcode self-dev
jcode selfdev
--no-selfdev
```

Remove auto-detection logic such as:

```text
if current directory is a Jcode repo
    enable self-dev mode
```

Remove environment state such as:

```text
JCODE_CLIENT_SELFDEV_MODE
JCODE_REPO_DIR
```

where the variable is only serving Self-Dev.

Do not remove a generic working-directory option or ordinary project directory handling.

---

# 19. Remove automatic Git clone behavior

`src/cli/selfdev.rs` contains explicit source acquisition behavior equivalent to:

```text
git clone https://github.com/1jehuang/jcode.git
```

into Jcode-owned state when a local checkout is not found.

HAVK should not autonomously clone its own source merely to enter a self-modification mode.

Delete this behavior completely.

Do not replace it with another downloader.

---

# 20. Remove Self-Dev Git/source-state machinery

Self-Dev heavily depends on Git-derived source identity.

`jcode-build-support` contains logic around:

```text
git rev-parse --git-common-dir
git rev-parse --short HEAD
git rev-parse HEAD
git status --porcelain
git diff --binary HEAD
git ls-files --others --exclude-standard
git log -1 --format=%s
```

Self-Dev then derives concepts such as:

```text
repo_scope
worktree_scope
short_hash
full_hash
dirty
source fingerprint
version label
changed paths
commit message
```

Types include:

```text
SourceState
BuildInfo
CanaryStatus
CrashInfo
PendingActivation
DevBinarySourceMetadata
SelfDevBuildCommand
SelfDevBuildTarget
```

This subsystem should be reduced aggressively once Self-Dev is removed.

However, **do not delete the entire `jcode-build-support` crate blindly** because the crate also contains stable/update/install path helpers that may still be needed.

---

# 21. `jcode-build-support`: split Self-Dev from stable release behavior

Current `jcode-build-support` mixes at least two responsibilities:

```text
A. Self-Dev / source checkout / canary build lifecycle
B. installed binary / stable channel / launcher / reload candidate lifecycle
```

HAVK wants to remove A while preserving the useful parts of B.

## Likely Self-Dev-specific removal candidates

Review/remove functions and types such as:

```text
SELFDEV_CARGO_PROFILE
selfdev_binary_path
selfdev_build_command
selfdev_build_command_for_target
run_selfdev_build
current_source_state
ensure_source_state_matches
repo_build_version
current_git_hash
current_git_hash_full
current_git_diff
is_working_tree_dirty
get_commit_message
repo_scope_key
worktree_scope_key
SelfDevBuildTarget
SelfDevBuildCommand
Self-Dev canary build state
Self-Dev pending activation state
```

Also review repository discovery helpers:

```text
get_repo_dir
find_repo_in_ancestors
is_jcode_repo
```

Do not delete them solely based on name until the later Git audit determines whether another feature still consumes them.

## Preserve until proven unused

Generic stable/update behavior such as:

```text
stable binary paths
versioned build paths
launcher symlink handling
installed version markers
stable update candidate
shared server binary resolution
generic running-binary payload resolution
```

These are not inherently Self-Dev.

---

# 22. Canary terminology requires classification, not blind deletion

Self-Dev uses session/build concepts named `canary` extensively.

Examples include:

```text
session.is_canary
set_canary("self-dev")
canary build
canary_session
canary_status
```

But some `canary` concepts may have acquired generic test/deployment meaning elsewhere.

Implementation rule:

```text
if canary state exists only to identify Self-Dev sessions/builds:
    remove it
else if another independent subsystem genuinely uses it:
    preserve or rename/refactor that independent concept
```

Do not perform a global deletion of the word `canary`.

---

# 23. Preserve generic server reload/reconnect

This is one of the most important safety constraints.

Jcode's official server architecture documents generic reload behavior separately from Self-Dev:

```text
/reload
server execs a new binary
clients reconnect
session state persists
```

Self-Dev uses this infrastructure, but does not necessarily own all of it.

HAVK should preserve generic capabilities if used by normal operation, updates, or recovery:

```text
server reconnect loop
session persistence across disconnect
reload handoff state
generic process restart
client reconnect
normal update/restart commands
shared daemon lifecycle
```

The goal is:

```text
remove "agent builds and activates its own source"
```

not:

```text
remove robust process restart/reconnect behavior
```

This distinction is what makes the 4/5 safety rating realistic rather than 5/5.

---

# 24. Preserve stable update channel

HAVK still needs a normal way to receive/install known release binaries.

Do not remove stable release/update infrastructure merely because Self-Dev also uses binary channels.

Target conceptual state:

```text
HAVK release
    -> CI builds
    -> published artifact
    -> stable update/install
    -> normal restart/reload if required
```

Remove:

```text
local source edit
-> selfdev compile
-> canary publish
-> shared server candidate
-> self validation
```

The two lifecycles should no longer be conflated.

---

# 25. Remove Self-Dev tool registration and agent special cases

Known dedicated native-tool identities include:

```text
selfdev
desktop_selfdev
```

Remove them from native tool registration and capability lists.

Review files including:

```text
crates/jcode-app-core/src/agent.rs
crates/jcode-app-core/src/tool/mod.rs
crates/jcode-app-core/src/agent/turn_streaming_mpsc.rs
```

Remove special handling such as:

```text
if tool call == selfdev then reload is expected
```

Do not disturb generic tool interruption/recovery semantics.

---

# 26. Remove Self-Dev TUI behavior

The TUI has significant Self-Dev awareness.

Audit/remove Self-Dev-specific:

```text
status labels
canary/session badges
selfdev command help
selfdev build/reload progress
selfdev support diagnostics
selfdev-specific update candidate selection
selfdev prompts/tool schema
```

Known consumer areas include many files under:

```text
crates/jcode-tui/src/tui/
```

Do not remove generic:

```text
progress display
background task UI
reconnect UI
session picker
update UI
```

unless those components are proven to exist only for Self-Dev.

---

# 27. Remove Self-Dev launch hotkeys/setup hints

`jcode-setup-hints` contains behavior for a global hotkey that opens a Self-Dev session in the last Jcode repository.

Examples include text equivalent to:

```text
Cmd+Shift+' -> self-dev session
last_repo -> Jcode repository used for self-dev
```

After Self-Dev removal:

- remove the Self-Dev-specific global launch shortcut;
- remove `last_repo` state if it exists only for that shortcut;
- simplify launch-hotkey metadata that carries a `self_dev` flag;
- preserve normal launch hotkeys such as opening HAVK in home or the last working project if those are retained.

This also reduces Git/repository detection pressure later.

---

# 28. Remove Self-Dev process title / telemetry flags

Audit/remove Self-Dev-only state in:

```text
src/cli/proctitle.rs
crates/jcode-telemetry-core/
crates/jcode-app-core/src/telemetry_state.rs
```

Remove provenance flags whose only meaning is:

```text
client is running in Self-Dev mode
session is a Self-Dev canary
```

Do not use this phase to remove telemetry as a whole; telemetry is a separate debloat decision unless already covered elsewhere.

---

# 29. Protocol cleanup

`jcode-protocol` currently depends on `jcode-selfdev-types` for at least reload recovery data.

Example audited export:

```text
ReloadRecoverySnapshot = jcode_selfdev_types::ReloadRecoveryDirective
```

This requires careful classification.

If the recovery structure exists only because Self-Dev build/reload needs to preserve work across self replacement:

```text
remove it
```

If some fields are needed for **generic server reload/reconnect**, move the minimal generic structure into an appropriate non-Self-Dev crate instead of keeping a Self-Dev crate alive.

Desired dependency direction:

```text
protocol -> generic lifecycle types
```

not:

```text
protocol -> selfdev-types
```

---

# 30. Desktop Self-Dev removal must not remove Desktop itself

`desktop_selfdev` is a separate self-development control path for Jcode Desktop.

Delete its:

```text
checkout detection
host build profile detection
paired UI build/reload logic
self-development preview/reload commands
```

Do not delete generic desktop functionality merely because it shares process/socket/reload helpers.

This phase removes Desktop **self-development**, not necessarily the Desktop client/product itself.

---

# 31. Documentation cleanup for Self-Dev

Update/remove current documentation that advertises Self-Dev as a supported product feature.

At minimum review:

```text
README.md
docs/SERVER_ARCHITECTURE.md
docs/SYSTEM_PROMPT_CONFIG.md
docs/SPAWN_HOOK.md
docs/WRAPPERS.md
docs/COMPILE_TIME_ISOLATION_REFACTOR.md
docs/plans/COMPILE_PERFORMANCE_PLAN.md
docs/plans/UNIFIED_SELFDEV_SERVER_PLAN.md
AGENTS.md
```

Rules:

### Current behavior docs

Rewrite to describe HAVK's actual architecture after removal.

### Self-Dev-only plans

Delete or archive if they are no longer relevant.

### Historical changelog entries

Do not rewrite historical records solely to erase Self-Dev references.

---

# 32. Self-Dev removal and future Git debloat

This phase is strategically important for the later Git removal discussion.

Self-Dev currently requires Git for:

```text
clone source
identify repo/worktree
read commit hashes
check dirty state
read diffs
fingerprint source
read commit messages
infer build state
```

Once Self-Dev is gone, many runtime Git consumers disappear.

That means the later Git audit can answer a simpler question:

> Which remaining HAVK features truly need Git for CTF/tool operation?

instead of needing to preserve Git simply so HAVK can build itself.

This is why Self-Dev should be removed **before** the final Git debloat.

---

# 33. What must explicitly survive both removals

The following are **not authorized for removal by this artifact**:

```text
Swarm
Overnight mode
generic shell/tool execution
MCP tools
generic background execution
generic server/client architecture
SSH remote use
Unix socket transport
WebSocket gateway
generic session persistence
generic reconnect logic
stable release updates
generic reload/restart where independently required
auth test sandboxes
onboarding test sandbox
future cloud sandbox support
future Modal integration
future E2B integration
user-managed cloud/VPS execution
```

Overnight remains undecided and will be discussed separately.

---

# 34. Future CTF cloud sandbox principle

HAVK's eventual execution architecture should optimize for **untrusted CTF workload isolation** rather than code-generation convenience.

Possible future execution policy:

```text
HAVK control plane / harness
        |
        | chooses execution target
        v
Sandbox provider
        |
        +-- create disposable environment
        +-- upload only required artifacts
        +-- execute CTF tool commands
        +-- stream stdout/stderr
        +-- collect result artifacts
        +-- destroy sandbox
```

Potential security controls:

```text
network policy per challenge
short default lifetime
resource limits
no host filesystem access
explicit secret injection
artifact size limits
separate workspace per session/challenge
provider-level audit/logging
optional persistence only when requested
```

The exact provider/API is intentionally deferred.

---

# 35. Implementation order

Because Self-Dev is cross-cutting, order matters.

## Stage A — Remote Compile

1. Remove `compile_remote` module files.
2. Remove tool registration.
3. Remove dynamic entitlement-aware tool refresh.
4. Remove dedicated tests/docs.
5. Remove leftover Remote Compile subscription/credit references.
6. Run static residue searches.

This should be a relatively contained change.

## Stage B — Self-Dev user entry points

1. Remove `self-dev` / `selfdev` CLI command.
2. Remove `--no-selfdev` because there is no longer anything to disable.
3. Remove auto-detection inside the HAVK repository.
4. Remove Self-Dev environment flags.
5. Remove Self-Dev global launch hotkey/setup hints.

## Stage C — Self-Dev tools/prompts

1. Remove `selfdev` tool.
2. Remove `desktop_selfdev` tool.
3. Remove Self-Dev prompt assets.
4. Remove agent special cases.
5. Remove TUI Self-Dev UI paths.

## Stage D — build/canary lifecycle

1. Classify `jcode-build-support` functionality.
2. Remove Git source-state and Self-Dev build commands.
3. Remove Self-Dev canary/pending activation state that has no generic use.
4. Preserve stable release/update paths.
5. Preserve generic reload/reconnect.

## Stage E — types/protocol/dependencies

1. Remove remaining `jcode-selfdev-types` consumers.
2. Move any genuinely generic lifecycle type elsewhere.
3. Delete `crates/jcode-selfdev-types`.
4. Remove Cargo dependencies/workspace entry.

## Stage F — docs/tests/residue

1. Remove Self-Dev-only tests.
2. Rewrite generic tests that only use Self-Dev terms as fixtures.
3. Clean current docs.
4. Run static search audit.
5. Commit and rely on GitHub Actions for heavy compilation/testing.

---

# 36. Static validation searches

## Remote Compile

```bash
rg -n 'compile_remote|remote compilation|Remote compilation|REMOTE_COMPILE' .
rg -n '/v1/compile|cloud-compute credits|remote_compile' .
```

Every remaining match must be reviewed.

## Self-Dev

```bash
rg -n 'self-dev|selfdev|SelfDev|desktop_selfdev' .
rg -n 'CLIENT_SELFDEV_ENV|JCODE_CLIENT_SELFDEV_MODE|JCODE_REPO_DIR' .
rg -n 'SelfDevBuild|SourceState|PendingActivation|canary_session|canary_status' .
```

## Git pressure removed by these features

```bash
rg -n 'git (clone|rev-parse|status|diff|ls-files|log)' src crates
```

Do **not** remove all remaining Git hits in this phase. Record them for the later dedicated Git audit.

## Sandbox false-positive audit

```bash
rg -n -i 'sandbox' src crates docs
```

Classify each occurrence before touching it.

`AuthTestSandbox`, onboarding sandbox, browser sandbox language, etc. are not cloud compile code.

---

# 37. Acceptance criteria — Remote Compile

The Remote Compile removal is complete when:

- [ ] `compile_remote` is no longer a built-in tool.
- [ ] `compile_remote.rs` and its source snapshot module are gone.
- [ ] Remote Compile-specific tests are gone.
- [ ] Remote Compile-specific docs are gone.
- [ ] No tool schema changes based on Jcode remote-compute entitlement.
- [ ] No compile request is submitted to Jcode's `/v1/compile` API.
- [ ] No local Git snapshot is created for Jcode remote compilation.
- [ ] No Remote Compile-specific Jcode credit/status guidance remains.
- [ ] Generic local shell compilation still works.
- [ ] Generic remote client/server operation still works.
- [ ] Future Modal/E2B integration has not been prohibited or structurally removed.

---

# 38. Acceptance criteria — Self-Dev

The Self-Dev removal is complete when:

- [ ] `havk self-dev` / `havk selfdev` does not exist.
- [ ] `--no-selfdev` does not exist.
- [ ] HAVK does not auto-enable special behavior merely because it runs inside its own source repository.
- [ ] HAVK does not clone its own source for self-development.
- [ ] Built-in `selfdev` tool is gone.
- [ ] Built-in `desktop_selfdev` tool is gone.
- [ ] Self-Dev-specific prompts are gone.
- [ ] Self-Dev build queue/compile lock is gone.
- [ ] Self-Dev Git source fingerprinting is gone.
- [ ] Self-Dev canary activation/rollback state is gone unless a documented independent consumer exists.
- [ ] Self-Dev global launch hotkey is gone.
- [ ] Self-Dev process title/telemetry flags are gone.
- [ ] `jcode-selfdev-types` is gone, or any remaining generic type has been moved out and the crate removed.
- [ ] Generic server reconnect still works.
- [ ] Generic session persistence still works.
- [ ] Stable update/install channel still works.
- [ ] Generic reload/restart remains if required independently of Self-Dev.

---

# 39. Validation constraints for HAVK development

As established in earlier HAVK debloat plans:

- do not run a full workspace compile on the constrained local machine merely to validate this phase;
- do not run full local CI emulation;
- use static inspection, `rg`, diffs, dependency inspection, and lightweight formatting checks;
- commit the coherent changes;
- use GitHub Actions / remote CI for full build and test validation;
- fix compile/test fallout based on CI results.

This is especially relevant to Self-Dev: do not use the subsystem being deleted to validate its own deletion.

---

# 40. Risk notes

## Remote Compile risk

**Low.**

It is a dedicated tool with a small integration surface.

Main mistakes to avoid:

```text
removing generic custom tool behavior because a test uses compile_remote as a dummy name
removing all sandbox-related code
removing remote server usage
```

## Self-Dev risk

**Moderate despite being product-safe to remove.**

The primary risk is deleting shared infrastructure by association.

Most dangerous categories:

```text
generic reload state
generic update candidate selection
stable binary channels
session reconnect
protocol lifecycle types
background task recovery
TUI progress/reconnect UI
```

The implementation should repeatedly ask:

> Does this code exist because HAVK can reload/recover normally, or only because Jcode can rebuild itself?

Only the second category is automatically authorized for deletion.

---

# 41. Expected architectural result

After Phase 3:

```text
HAVK
├── stable harness runtime
├── model/provider integrations
├── Swarm
├── MCP/tool execution
├── local shell/tool capability
├── remote client/server capability
├── generic session persistence/reconnect
│
├── NO Jcode Remote Compile product
├── NO Jcode cloud-compute credit integration
├── NO Jcode Self-Dev
├── NO self-build/self-canary lifecycle
│
└── future execution target
    ├── Modal Sandbox      [future]
    ├── E2B Sandbox        [future]
    └── other cloud/VPS    [future]
```

---

# 42. Decision record

```yaml
phase_3:
  remote_compile:
    decision: remove
    safety_rating: 5/5
    difficulty_rating: 2/5
    remove_jcode_cloud_api_path: true
    remove_jcode_compute_credit_path: true
    remove_git_snapshotter_for_remote_compile: true

  cloud_sandbox_capability:
    decision: preserve_as_future_architecture
    jcode_managed_sandbox: remove
    modal: future_candidate
    e2b: future_candidate
    other_provider: allowed
    arbitrary_ctf_tool_execution: desired

  self_dev:
    decision: remove
    safety_rating: 4/5
    difficulty_rating: 5/5
    self_build: remove
    self_test_orchestration: remove
    self_reload_activation: remove
    selfdev_canary: remove
    selfdev_git_source_state: remove

  preserve:
    swarm: true
    overnight: undecided
    generic_reload: true_if_independently_used
    generic_reconnect: true
    stable_updates: true
    remote_server_use: true
    generic_sandbox_future: true

  git:
    full_debloat_decision: deferred
    note: remove_these_git_consumers_first
```

---

# 43. Next discussion boundary

The next unresolved candidate from the previous research is **Overnight Development Mode**.

No decision about Overnight is encoded in this artifact.

It should be discussed separately because its current implementation is coding/repository-oriented, but its long-running autonomous execution concept could potentially be redesigned into a HAVK CTF mission mode rather than deleted.

---

# 44. Official references

## Jcode

- Jcode repository / README: https://github.com/1jehuang/jcode
- Jcode public docs: https://jcode.sh/docs
- Jcode server architecture: https://github.com/1jehuang/jcode/blob/master/docs/SERVER_ARCHITECTURE.md
- Jcode wrapper docs: https://github.com/1jehuang/jcode/blob/master/docs/WRAPPERS.md
- Source baseline inspected locally: `jcode-0.88.0.tar.gz`
- Source doc inspected locally: `docs/REMOTE_COMPILE.md`
- Source doc inspected locally: `docs/REMOTE_COMPILE_VALIDATION.md`

## Modal

- Modal docs: https://modal.com/docs
- Python Sandbox API: https://modal.com/docs/sdk/py/latest/Sandbox
- JavaScript Sandbox API: https://modal.com/docs/sdk/js/latest/Sandbox
- GitHub Actions / CI snippet: https://modal.com/docs/guide/useful-snippets

## E2B

- E2B documentation: https://docs.e2b.dev/
- E2B security/isolation: https://e2b.dev/security

---

## Final phase statement

**Approved:** remove Jcode Remote Compile and Jcode Self-Dev.

**Protected design constraint:** HAVK must retain the architectural freedom to run CTF tools in external isolated cloud sandboxes such as Modal, E2B, or similar providers. The removed Jcode Remote Compile service must not be mistaken for HAVK's future generic sandbox execution layer.
