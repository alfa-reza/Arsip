# HAVK OS Support Debloat — Adversarial Review & Cross-Check

> **Document type:** independent review / implementation-readiness audit  
> **Reviewed artifact:** `HAVK_OS_SUPPORT_DEBLOAT.md`  
> **Review posture:** adversarial — the goal is to find wrong assumptions, unsafe instructions, omissions, and ambiguous requirements rather than defend the existing document.  
> **Review date:** 2026-09-27  
> **Overall result:** **VALID STRATEGY, NOT YET SAFE TO TREAT AS IMPLEMENTATION-READY WITHOUT CORRECTIONS**

---

## 1. Executive verdict

The central platform strategy in `HAVK_OS_SUPPORT_DEBLOAT.md` is technically sound:

- Linux x86_64 and Linux ARM64 can reasonably be HAVK's primary supported platforms.
- macOS Apple Silicon and Intel can be retained as lower-priority/best-effort targets.
- native Windows support can be removed and Windows users can be directed to WSL2.
- FreeBSD can lose dedicated HAVK CI/release maintenance without deliberately breaking generic Unix portability.
- Windows-specific Rust dependencies and code paths can be removed without requiring Windows-named transitive crates to disappear from `Cargo.lock`.
- generic `#[cfg(unix)]` code must not be narrowed merely to make FreeBSD fail.

However, the artifact currently overstates its readiness with:

```yaml
status: "implementation-ready"
```

That status is **not yet justified**.

The source cross-check found several important omissions and ambiguities. The most serious one is a Linux CI script that actively tests Windows installer parity. An agent following the document can correctly delete the PowerShell installer and the Git Bash Windows branch, yet leave that Linux CI script unchanged and make CI fail.

### 1.1 Readiness verdict by category

| Area | Verdict | Notes |
|---|---|---|
| Overall platform direction | **Valid** | The policy makes architectural sense for HAVK. |
| Native Windows removal | **Valid** | Windows support is broad enough that removal meaningfully reduces code and maintenance. |
| WSL2 recommendation | **Valid with wording correction** | WSL2 is a Linux runtime path, but HAVK should avoid promising Windows-host integration. |
| FreeBSD maintenance removal | **Valid** | Dedicated FreeBSD cost is mainly CI/release rather than a large separate runtime implementation. |
| Preserve generic Unix portability | **Correct and important** | Supported by Rust `cfg` semantics. |
| macOS best-effort policy | **Valid but underspecified** | Release-gating semantics are unresolved. |
| Cargo dependency cleanup | **Mostly correct** | Lockfile regeneration guidance is too vague and can cause unrelated dependency churn if implemented poorly. |
| CI removal inventory | **Incomplete** | Two explicit Windows jobs and one Linux job with Windows parity checks need clearer treatment. |
| Documentation cleanup | **Incomplete / unsafe if interpreted literally** | Deleting `docs/WINDOWS.md` can create broken links. Rewrite-in-place is safer. |
| Guardrail/budget handling | **Incomplete** | At least one ratchet baseline contains paths to files planned for deletion. |
| Validation strategy | **Operationally valid, correctness proof incomplete** | Avoiding heavy local builds is fine, but remote compilation/CI must be a required validation gate before merge. |
| Current `implementation-ready` label | **Not valid yet** | Change after the corrections in this review are incorporated. |

### 1.2 Bottom line

**Do not hand the current artifact to an autonomous coding agent unchanged.**

It is close, but it needs targeted corrections before it is safe. The problem is not the high-level OS policy. The problem is the dependency surface surrounding that policy: CI scripts, guardrail baselines, documentation links, release finalization semantics, and lockfile handling.

---

## 2. What was cross-checked

This review checked three kinds of evidence:

1. the full `HAVK_OS_SUPPORT_DEBLOAT.md` artifact;
2. the Jcode source tree, including CI, release, Rust manifests, installers, tests, SDK packages, updater logic, quality-guardrail scripts, and documentation references;
3. current official documentation for Rust, Cargo, and WSL2.

The review deliberately distinguishes:

- **product policy** — what HAVK intends to support;
- **source implementation** — what files and branches currently implement inherited support;
- **build/release policy** — what CI and release jobs require;
- **portability** — code that happens to work on more systems but does not create a support promise;
- **historical references** — references that should not be deleted merely because they contain the word `Windows` or `FreeBSD`.

---

## 3. Official documentation cross-check

## 3.1 Rust conditional compilation: the document is correct about `unix`

The Rust Reference documents both `target_os` and `target_family`.

Relevant official behavior:

- `target_os` values include `windows`, `macos`, `linux`, and `freebsd`.
- `target_family` can include `unix` and `windows`.
- the bare `unix` configuration is set when `target_family = "unix"` is set.

Official reference:

- <https://doc.rust-lang.org/reference/conditional-compilation.html>

Therefore this warning in the artifact is **correct and should remain**:

> Do not replace generic `#[cfg(unix)]` code with a Linux/macOS-only condition merely to exclude FreeBSD.

That would be an artificial portability regression rather than useful debloat.

### Review conclusion

**VALID. KEEP.**

---

## 3.2 Rust FreeBSD target support: the rationale is valid

Current Rust platform documentation states:

- `x86_64-unknown-freebsd` is Tier 2 with host tools;
- `aarch64-unknown-freebsd` is Tier 2 with host tools;
- `i686-unknown-freebsd` is Tier 2 without host tools;
- other FreeBSD targets are Tier 3.

Official references:

- <https://doc.rust-lang.org/rustc/platform-support.html>
- <https://doc.rust-lang.org/rustc/platform-support/freebsd.html>

This does **not** mean HAVK must support FreeBSD. Rust's platform tier and HAVK's product support policy are separate concepts.

It does support the artifact's core conclusion that keeping low-cost generic Unix portability is reasonable even after HAVK stops publishing and continuously testing FreeBSD.

### Review conclusion

**VALID**, but the HAVK status label should be refined. See Finding M-5.

---

## 3.3 Cargo target-specific dependencies: the document is correct

Cargo officially supports target-specific dependency tables such as:

```toml
[target.'cfg(windows)'.dependencies]
...

[target.'cfg(unix)'.dependencies]
...
```

Official reference:

- <https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#platform-specific-dependencies>

The source contains direct native-Windows dependency blocks in:

```text
Cargo.toml
crates/jcode-app-core/Cargo.toml
crates/jcode-base/Cargo.toml
crates/jcode-core/Cargo.toml
crates/jcode-setup-hints/Cargo.toml
crates/jcode-transport/Cargo.toml
```

This matches the existing artifact.

### Review conclusion

**VALID.**

The direct Windows target dependencies should be removed after the corresponding consumers are removed.

---

## 3.4 `Cargo.lock`: the warning is correct, but the procedure is incomplete

The artifact correctly warns:

> Do not use `Cargo.lock` as a "no Windows words" test.

This is important. A Unix-only HAVK build can still retain packages with names such as `windows-sys`, `windows-targets`, or architecture-specific Windows support crates because third-party cross-platform dependencies may include them in the resolved graph.

Cargo's own metadata format includes target-specific dependency information and, without platform filtering, resolves dependencies across target platforms.

Official reference:

- <https://doc.rust-lang.org/cargo/commands/cargo-metadata.html>

`cargo metadata --locked` is particularly useful as a consistency check because Cargo documents that it fails when the current dependency resolution would require changing `Cargo.lock`.

However, there is a major trap that should be documented explicitly:

```text
cargo generate-lockfile
```

is **not** an appropriate default "minimal refresh" command for this task. Cargo documents that if a lockfile already exists, `cargo generate-lockfile` rebuilds it using the latest available version of every package. That can generate unrelated dependency churn.

Official reference:

- <https://doc.rust-lang.org/cargo/commands/cargo-generate-lockfile.html>

### Review conclusion

**PARTIALLY VALID. REVISE.**

The artifact should explicitly prohibit `cargo generate-lockfile` as a blind lockfile repair mechanism for this debloat.

Recommended procedure is documented in Finding H-3.

---

## 3.5 WSL2: the main recommendation is valid

Microsoft documents that WSL2:

- uses an actual Linux kernel;
- runs it in a managed lightweight utility VM;
- provides full Linux system-call compatibility;
- is the current default WSL architecture for newly installed Linux distributions.

Official reference:

- <https://learn.microsoft.com/windows/wsl/wsl2-about>

Therefore it is technically reasonable for HAVK to say:

> Native Windows is unsupported. Windows users should use HAVK from a Linux distribution under WSL2.

This avoids maintaining a native Windows Rust/API/PowerShell/release surface merely to serve Windows-host users.

However, the artifact should not imply that WSL2 erases all host/runtime differences. Filesystem placement, Windows/Linux interop, device access, kernel modules, virtualization, networking, and privileged CTF workflows may behave differently.

### Review conclusion

**VALID WITH WORDING REFINEMENT.**

WSL2 should remain a deployment recommendation, not a separate compile target.

---

## 4. Source-level reality: Windows and FreeBSD are not equivalent

## 4.1 Windows is a real code-deletion target

The source audit confirms that native Windows support is spread across many layers, not only release YAML.

Examples include:

- `#[cfg(windows)]` / Windows API conditionals throughout Rust crates;
- direct `windows-sys` target dependencies;
- Windows named-pipe IPC;
- Windows process and power behavior;
- Windows filesystem/ACL/replacement behavior;
- native Windows setup hints and global-hotkey implementation;
- PowerShell install/uninstall paths;
- Git Bash/MSYS native-Windows logic inside shell installers;
- native Windows updater asset selection;
- Windows x86_64 and ARM64 CI;
- Windows cross-compilation validation;
- Windows lifecycle tests;
- Windows release packaging and signing;
- npm runtime packages for `win32-x64` and `win32-arm64`;
- TypeScript Windows named-pipe/runtime-launch behavior.

Six Cargo manifests alone contain direct Windows target dependency blocks.

This means native Windows removal is not cosmetic. It can materially reduce:

- source complexity;
- conditional compilation paths;
- platform-specific test burden;
- installer complexity;
- signing/release infrastructure;
- runtime package matrix;
- CI time and failure surface.

### Review conclusion

The decision to remove native Windows support is **technically coherent and meaningfully debloating**.

---

## 4.2 FreeBSD is mainly a maintenance/release target

The audit found very little dedicated FreeBSD runtime implementation compared with Windows.

Important FreeBSD references include:

- dedicated FreeBSD smoke/release workflows;
- release helper/status text;
- a TypeScript test confirming no FreeBSD npm runtime package;
- generic Unix portability comments/behavior;
- a test using `FreeBSD` as an "other host" value for fallback behavior.

Examples of code that should **not** be deleted simply because the comments mention FreeBSD:

```text
crates/jcode-base/src/platform.rs
crates/jcode-base/src/auth/cursor.rs
scripts/test_dev_cargo_jobs.sh
```

These are not expensive FreeBSD-specific implementations comparable to Windows named pipes or PowerShell installation.

### Review conclusion

The document is correct to treat FreeBSD as:

> remove dedicated support burden, do not deliberately damage generic Unix portability.

That is safer than attempting a broad source purge of the word `freebsd`.

---

# 5. Findings by severity

---

## BLOCKER-1 — `scripts/setup_friction_eval.sh` is missing from the artifact

### Severity

**BLOCKER / high confidence**

### What the source does

The Linux-executed script:

```text
scripts/setup_friction_eval.sh
```

contains a complete section explicitly named:

```text
Section D: Windows parity
```

It currently depends on native-Windows installer behavior, including:

- `scripts/install.ps1`;
- native Windows behavior inside `scripts/install.sh`;
- `MINGW64_NT-10.0` emulation;
- a mocked `powershell.exe`;
- `WM_SETTINGCHANGE` handling;
- `jcode-windows-x86_64.exe`;
- `jcode-windows-x86_64.tar.gz`;
- user-PATH persistence checks;
- cross-checking Git Bash and PowerShell installer behavior.

The main CI workflow has a `setup-friction` job that executes this script.

### Why this is dangerous

The current artifact tells the agent to remove:

- `scripts/install.ps1`;
- native Windows handling from `scripts/install.sh`;
- Windows release assets.

But it does **not** identify `scripts/setup_friction_eval.sh`.

An agent can therefore implement the requested Windows debloat correctly and then get a red **Linux CI** job because this script still expects the deleted Windows paths.

This is exactly the type of hidden dependency an implementation artifact must prevent.

### Required correction

Add `scripts/setup_friction_eval.sh` to the mandatory edit list.

The instruction should say approximately:

> Remove Section D's native-Windows parity checks and any Windows-only fixtures/mocks from `scripts/setup_friction_eval.sh`. Preserve the Linux/macOS/general setup-friction checks. Do not delete the entire setup-friction job merely because one section tests Windows.

### Verdict

**Must be fixed before implementation.**

---

## BLOCKER-2 — two concrete CI jobs are not explicitly named

### Severity

**HIGH**

### Current source

`.github/workflows/ci.yml` contains at least these Windows-oriented jobs:

```text
windows-build-test
powershell-syntax
windows-cross-check
```

The artifact explicitly names `windows-build-test`, and it mentions generic PowerShell syntax checks, but it does not explicitly name:

```text
powershell-syntax
windows-cross-check
```

### Why it matters

`windows-cross-check` is especially easy to miss because it runs on **Ubuntu**, not on `windows-latest`.

It installs Windows MSVC targets and uses `cargo xwin` to check:

```text
x86_64-pc-windows-msvc
aarch64-pc-windows-msvc
```

A simplistic search for Windows runners would not identify it as a Windows support job.

`powershell-syntax` is also a dedicated native Windows support gate.

### Required correction

The artifact should explicitly state:

```text
Delete from .github/workflows/ci.yml:
- windows-build-test
- powershell-syntax
- windows-cross-check
```

Then review any surviving `needs`, comments, cache keys, target setup, and conditional logic.

### Verdict

**Must be explicit.**

---

## BLOCKER-3 — deleting `docs/WINDOWS.md` can create broken documentation links

### Severity

**HIGH**

### Current inbound references

The source contains active links to `docs/WINDOWS.md` from at least:

```text
README.md
docs/README.md
docs/MULTI_SESSION_CLIENT_ARCHITECTURE.md
```

The current artifact says `docs/WINDOWS.md` may be removed or replaced with a concise WSL policy page.

### Risk

If an agent chooses the deletion path but only follows the artifact's minimum docs list, links can become stale/broken.

### Safer design

For this repository, **rewrite `docs/WINDOWS.md` in place** instead of deleting it.

The page can become something like:

```text
# Windows / WSL2

HAVK does not support native Windows binaries.
Windows users should install and run HAVK inside WSL2 as a Linux application.
...
```

Advantages:

- stable links remain valid;
- existing users following old links receive a migration path instead of 404;
- search engines and old references land on the new policy;
- less documentation churn;
- no need to hunt every historical inbound link merely to avoid breakage.

Then update the labels in README/docs index from "Windows Support" to "Windows / WSL2" or equivalent.

### Required correction

Change the original artifact from:

> delete or replace `docs/WINDOWS.md`

into:

> **prefer rewriting `docs/WINDOWS.md` in place as the native-Windows deprecation/WSL2 policy page; delete it only if all active inbound links are intentionally updated and verified.**

Also add:

```text
docs/README.md
docs/MULTI_SESSION_CLIENT_ARCHITECTURE.md
```

to the mandatory documentation review list.

---

## HIGH-1 — lockfile repair guidance is too vague

### Severity

**HIGH**

### Existing text

The artifact currently says:

> use a resolver-only/non-compiling Cargo operation if available and permitted in the environment.

This leaves too much interpretation to an autonomous agent.

### Dangerous interpretation

An agent may choose:

```bash
cargo generate-lockfile
```

Cargo's official documentation states that when a lockfile already exists this command rebuilds it with the latest available version of **every package**.

That can create a large unrelated dependency update in what should be a platform-debloat PR.

### Required policy

Add a warning:

> **Do not use `cargo generate-lockfile` as a blind lockfile repair command for this task.** It may update unrelated package versions.

Recommended non-compiling sequence when the manifest change makes the lockfile stale:

```bash
cargo metadata --format-version 1 > /dev/null
```

Then:

1. inspect the `Cargo.lock` diff carefully;
2. reject unrelated package/version churn;
3. verify consistency with:

```bash
cargo metadata --locked --format-version 1 > /dev/null
```

Important nuance:

- `cargo metadata --no-deps` is insufficient as the final dependency-graph check because the resolved graph is omitted with `--no-deps`.
- `--offline` should only be used if the required index/crates are already available locally; Cargo documents that offline resolution can behave differently depending on what is cached.
- never hand-delete package records simply because they contain `windows`.

### Verdict

The existing lockfile principle is correct; the operational recipe needs tightening.

---

## HIGH-2 — “no heavy local build” is an execution constraint, not a correctness acceptance criterion

### Severity

**HIGH**

### Current artifact

It includes:

```text
- [ ] No heavy local compilation/CI was performed.
```

as an acceptance criterion.

### Why that is conceptually wrong

The user's low-resource local environment is a valid **execution constraint**.

But not compiling code does not prove the implementation is correct.

This refactor removes many `#[cfg(windows)]` branches, modules, imports, dependencies, tests, and workflow paths. Static grep and formatting cannot prove that retained Linux/macOS targets still compile.

The artifact should distinguish:

### Execution constraint

```text
Do not run heavy workspace builds/tests locally.
```

from:

### Correctness gate

```text
Remote CI must compile/test the retained supported paths successfully before the change is considered merge-ready.
```

### Required correction

Move "no heavy local compilation" out of product acceptance criteria and place it under an **Execution Environment Constraint** section.

Then make remote CI a mandatory completion gate:

> Static validation can make the patch ready to push, but not ready to merge. The platform-debloat change is not considered validated until the applicable remote CI checks for retained platforms are green.

If CI has not run yet, the agent's report should say:

> implementation complete; static validation passed; remote compilation validation pending.

It should **not** claim the change is fully validated.

---

## HIGH-3 — macOS “best-effort” has unresolved release-gating semantics

### Severity

**HIGH / policy ambiguity**

### Current source behavior

The release workflow currently builds:

```text
x86_64-unknown-linux-gnu
aarch64-unknown-linux-gnu
aarch64-apple-darwin
x86_64-apple-darwin
```

plus Windows and FreeBSD.

The build matrix uses `continue-on-error`, but the final release job later validates a complete expected asset set. If an expected artifact is absent, finalization exits and keeps the GitHub release as a draft.

Therefore the current practical release contract is:

> every listed release artifact is required before the release becomes public.

### Conflict with HAVK wording

The HAVK artifact calls macOS:

> best-effort / non-priority

but also says its release artifacts are retained.

That leaves an important question unanswered:

**If macOS Intel fails but both Tier-1 Linux releases succeed, does HAVK block the entire public release?**

Two interpretations are both possible:

#### Model A — supported but low engineering priority

- macOS remains a mandatory release artifact.
- release is blocked until macOS is fixed or upstream/community supplies a fix.
- "best-effort" only describes developer priority, not release gating.

#### Model B — Linux-gated release

- Linux x86_64 + ARM64 are the only mandatory public-release gates.
- macOS artifacts are published when successful.
- macOS failure does not block Linux publication.
- Homebrew update can remain conditional on the required macOS/Linux assets.

The user's stated philosophy appears closer to **Model B**, but the implementation specification should not force an AI agent to infer that.

### Required correction

Choose and state one model explicitly before the implementation agent edits `release.yml`.

Do **not** leave release-gating semantics implicit.

---

## HIGH-4 — repository-level signing secrets cannot be “deleted” by ordinary source editing

### Severity

**MEDIUM-HIGH safety boundary**

### Current artifact wording

The release section says to remove:

> Windows signing secrets/settings and emergency override handling.

### Problem

A source-code agent can remove workflow references to variables/secrets such as:

```text
AZURE_CLIENT_ID
AZURE_TENANT_ID
AZURE_SUBSCRIPTION_ID
WINDOWS_SIGNING_ENDPOINT
WINDOWS_SIGNING_ACCOUNT
WINDOWS_SIGNING_CERTIFICATE_PROFILE
WINDOWS_SIGNING_REQUIRED
```

But repository/org-level GitHub Actions secrets and variables are not source files.

An autonomous agent with GitHub administration permissions should not infer that this document authorizes account/repository secret deletion.

### Required correction

Use precise language:

> Remove Windows-signing **workflow references, configuration consumption, documentation, and code paths** from the repository. Do not delete repository/org secrets or variables as part of this source-code task. They may remain inert and can be manually cleaned up by a repository maintainer in a separate administrative action.

Similarly:

- do not delete old release assets from historical GitHub releases;
- do not attempt to unpublish previously published npm packages;
- only stop future production/publication from the current source and workflows.

---

## MEDIUM-1 — `crates/jcode-app-core/src/update.rs` contains a Windows asset test fixture

### Current source

A checksum parser test contains an asset name similar to:

```text
jcode-windows-x86_64.exe
```

This test is not necessarily testing Windows runtime support. It is primarily exercising checksum-file parsing, including binary marker syntax/line formatting.

### Risk

The current post-edit semantic scan says Windows references in test fixtures may remain if Windows is merely data. That is technically safe.

However, leaving this exact asset name can confuse future auditors into thinking a Windows release asset is still expected.

### Recommendation

Do **not** delete the parser test.

Either:

1. keep it and explicitly classify it as a neutral parser fixture; or preferably
2. replace the Windows release filename with a neutral or retained asset name while preserving the syntax being tested.

For example, retain the `*filename` parsing behavior without implying that HAVK publishes `.exe` assets.

### Required artifact addition

Add:

```text
crates/jcode-app-core/src/update.rs
```

to mixed-file review hotspots.

---

## MEDIUM-2 — quality guardrail baselines contain deleted Windows file paths

### Current source

`scripts/swallowed_error_budget.json` contains tracked entries for at least:

```text
crates/jcode-setup-hints/src/windows_setup.rs
crates/jcode-transport/src/windows.rs
```

Both are intended deletion candidates.

The quality checker appears to treat deleted tracked files as an improvement rather than an immediate failure, so this is **not necessarily a blocker**.

However, the baseline becomes stale.

### Bigger safety concern

The agent must not respond by blindly rebaselining every ratchet.

A blanket baseline update can accidentally bless unrelated regressions elsewhere in the tree.

### Required policy

Add a guardrail section:

> Inspect all quality-budget/baseline files that mention deleted paths. Do not run blanket ratchet rebaselining or `--fix` operations without reviewing the resulting diff. Updating a baseline is allowed only when the diff is limited to paths/counts legitimately removed or changed by this task.

Known file to review:

```text
scripts/swallowed_error_budget.json
```

Also inspect the other budget/config files for path references at current HEAD rather than assuming the static list is exhaustive.

---

## MEDIUM-3 — stale comment in `scripts/check_guardrails.sh`

### Current source

The script says, in substance, that only Windows CI jobs pass `--locked` and therefore a stale lockfile otherwise survives most jobs.

But the same guardrail script now runs:

```bash
cargo metadata --locked --format-version 1
```

and current CI also contains metadata validation.

After Windows CI is removed, that comment becomes actively misleading.

### Recommendation

Keep the Cargo.lock gate, but rewrite/remove the obsolete Windows-specific explanation.

Add:

```text
scripts/check_guardrails.sh
```

to mandatory mixed-file review.

---

## MEDIUM-4 — the artifact does not explicitly name every documentation link affected by the Windows policy

At minimum, active references should be checked in:

```text
README.md
docs/README.md
docs/MULTI_SESSION_CLIENT_ARCHITECTURE.md
CONTRIBUTING.md
RELEASING.md
docs/WINDOWS.md
AGENTS.md
```

`AGENTS.md` is especially relevant because it currently documents native Windows install paths and the PowerShell installer.

An AI coding agent frequently reads `AGENTS.md` as authoritative repository guidance. Leaving native Windows installation instructions there after removing the implementation creates contradictory agent instructions.

### Recommendation

Explicitly require repository-agent documentation to be updated when it describes removed support.

Do not remove unrelated development instructions.

---

## MEDIUM-5 — “Unofficial / community-compatible” can overpromise FreeBSD compatibility

### Problem

The phrase:

> community-compatible

can be interpreted as a positive compatibility promise.

But the intended policy is actually:

- not tested by HAVK;
- not released by HAVK;
- no maintenance SLA;
- generic Unix code is not deliberately sabotaged;
- low-cost community fixes may be accepted.

Those are not the same as guaranteeing compatibility.

### Safer label

Prefer one of:

```text
Unsupported — portability preserved
```

or:

```text
Unofficial — may work via generic Unix portability
```

Then state:

> FreeBSD compatibility is not a release contract. Community patches may be accepted when they are low-risk and do not add a dedicated maintenance burden or regress supported targets.

### Verdict

Policy remains valid; terminology should be tightened.

---

## MEDIUM-6 — WSL-specific prohibition is too absolute for future maintenance

### Current intent

The artifact correctly says not to create a new `cfg(wsl)` architecture just to compensate for Windows removal.

### Problem

A permanent statement like:

> never add WSL-specific behavior

can become wrong if a real future WSL2 bug requires a narrow workaround.

### Better wording

> Do not add WSL-specific Rust/configuration branches **as part of this debloat merely to replace removed native Windows functionality**. Future WSL-specific fixes require separate evidence, scope, and justification.

This preserves architectural cleanliness without creating an unreasonable permanent prohibition.

---

## MEDIUM-7 — Linux “fully supported” needs exact target boundaries everywhere

The artifact does define the exact GNU targets in its detailed text, which is good:

```text
x86_64-unknown-linux-gnu
aarch64-unknown-linux-gnu
```

But some summary language says simply:

> Linux x86_64 / Linux ARM64 fully supported

For an implementation agent or future contributor, "Linux ARM64" might be misread to include:

- musl variants;
- Android/Termux as a native target;
- other ARM ABIs;
- distro-specific binary compatibility promises.

### Recommendation

In the support table itself, make the exact release targets explicit:

```text
Linux x86_64: x86_64-unknown-linux-gnu
Linux ARM64: aarch64-unknown-linux-gnu
```

Then separately document any Termux/runtime compatibility behavior as inherited special handling rather than a third official target triple.

---

## MEDIUM-8 — current per-PR CI does not fully mirror the declared release target matrix

Current primary build/test CI includes:

```text
ubuntu-latest -> x86_64-unknown-linux-gnu
macos-latest  -> aarch64-apple-darwin
```

Linux ARM64 and macOS Intel are exercised in release production rather than the ordinary main build matrix.

This does not make the HAVK policy invalid, but the artifact should avoid wording that implies every declared target is currently tested on every PR.

### Recommendation

State separately:

- **support target**;
- **per-PR CI target**;
- **release-build target**.

Do not expand the debloat task into new CI architecture unless that is an explicit separate requirement.

---

## MEDIUM-9 — SDK removal boundary should distinguish local Windows runtime from generic client protocol

The SDKs contain Windows-local runtime behavior such as:

- Win32 runtime packages;
- executable naming/mapping;
- named-pipe paths;
- local process launch behavior.

Those are legitimate removal targets.

But generic SDK protocol/client APIs may still be platform-neutral and useful for:

- remote sessions;
- SSH/custom transports;
- connecting to a HAVK server running elsewhere.

### Required clarification

> Remove native-Windows **local runtime, launcher, packaged-binary, and IPC behavior**. Do not remove protocol/client abstractions solely because a client process could theoretically execute on Windows if those abstractions are otherwise platform-neutral and used by retained remote/custom transports.

This prevents an agent from over-deleting shared SDK functionality.

---

## LOW-1 — upstream `docs/WINDOWS.md` contains a stale architecture path

Current upstream Windows documentation says named-pipe transport lives under a path equivalent to:

```text
crates/jcode-base/src/transport/windows.rs
```

but the audited implementation is under:

```text
crates/jcode-transport/src/windows.rs
```

This is not a HAVK artifact error by itself; it is evidence that inherited documentation can already be stale.

### Lesson for the artifact

Do not copy inherited path statements blindly. Treat current HEAD as source of truth.

The existing artifact already follows this philosophy in general, so only a minor note is needed.

---

## LOW-2 — current upstream release comments/documentation are internally inconsistent

The Windows documentation describes platform publication as independent and implies a Windows failure does not block successful Unix assets.

The current release finalization script, however, checks an expected set that includes Linux, macOS, Windows, and FreeBSD and refuses to publish the draft if any required asset is missing.

This is another reason HAVK should define its own release policy explicitly rather than inheriting prose assumptions from Jcode.

---

# 6. Revised source-action inventory

This section is the most important implementation-oriented correction to the original artifact.

## 6.1 High-confidence DELETE targets

Delete when they still exist at current HEAD and are still Windows/FreeBSD-owned:

```text
.github/workflows/windows-smoke.yml
.github/workflows/freebsd-smoke.yml
.github/scripts/verify_windows_install.ps1

crates/jcode-setup-hints/src/windows_hotkeys.rs
crates/jcode-setup-hints/src/windows_setup.rs
crates/jcode-terminal-launch/src/windows_portable_tests.rs
crates/jcode-transport/src/windows.rs

tests/e2e/windows_lifecycle.rs

scripts/install.ps1
scripts/uninstall.ps1
scripts/test_windows_launcher_install.ps1
scripts/test_windows_setup_evaluation.ps1

sdk/typescript/npm/win32-x64/
sdk/typescript/npm/win32-arm64/
```

Delete only if they are still dedicated to the removed support surface. Current HEAD always wins over this list.

---

## 6.2 Mandatory EDIT / REDUCE targets

These are more dangerous than deletion-only files because retained functionality coexists with removed functionality.

### CI / release

```text
.github/workflows/ci.yml
.github/workflows/release.yml
.github/workflows/publish-typescript-sdk.yml
```

In `ci.yml`, explicitly handle:

```text
windows-build-test
powershell-syntax
windows-cross-check
setup-friction
```

`setup-friction` should normally remain, but its Windows parity section must be removed from the called script.

### Installer / release scripts

```text
scripts/install.sh
scripts/uninstall.sh
scripts/setup_friction_eval.sh
scripts/test_install_conversion.sh
scripts/prepare_sdk_runtime_packages.sh
scripts/quick-release.sh
scripts/check_guardrails.sh
```

### Rust manifests

```text
Cargo.toml
crates/jcode-app-core/Cargo.toml
crates/jcode-base/Cargo.toml
crates/jcode-core/Cargo.toml
crates/jcode-setup-hints/Cargo.toml
crates/jcode-transport/Cargo.toml
```

### Rust mixed-platform code

Review all existing `#[cfg(windows)]`, `target_os = "windows"`, Windows API imports, Windows asset mapping, process/filesystem behavior, and setup dispatch.

Mandatory specifically re-check:

```text
crates/jcode-app-core/src/update.rs
crates/jcode-base/src/auth/grok_build.rs
crates/jcode-update-core/src/lib.rs
crates/jcode-transport/src/lib.rs
```

as well as every current production hit found by the pre-edit semantic scan.

### TypeScript SDK

Review at least:

```text
sdk/typescript/package.json
sdk/typescript/package-lock.json
```

plus current source files implementing:

- platform runtime-package mapping;
- executable naming;
- local binary launch;
- Windows named-pipe/socket derivation.

### Quality guardrails

```text
scripts/swallowed_error_budget.json
```

and any other current baseline/config that names files being removed.

Do not globally regenerate ratchets without reviewing scope.

### Documentation / agent instructions

```text
README.md
RELEASING.md
CONTRIBUTING.md
AGENTS.md
docs/WINDOWS.md
docs/README.md
docs/MULTI_SESSION_CLIENT_ARCHITECTURE.md
```

Prefer rewriting `docs/WINDOWS.md` in place.

---

## 6.3 PRESERVE unless independently proven dead

These should not be deleted merely because a broad search finds Windows/FreeBSD-related text.

### Generic Unix portability

Preserve:

```rust
#[cfg(unix)]
std::os::unix::...
Unix domain socket implementation
POSIX permissions/metadata logic used by Linux/macOS
libc portability handling
XDG-style generic Unix fallback behavior
```

### FreeBSD portability examples

Preserve unless a separate retained-target simplification proves otherwise:

```text
crates/jcode-base/src/platform.rs
crates/jcode-base/src/auth/cursor.rs
scripts/test_dev_cargo_jobs.sh
```

A test using the string `FreeBSD` as an "other platform" fixture is not automatically official FreeBSD support.

### UI terminology

Do not delete references where `window` / `windows` means graphical/UI windows rather than Microsoft Windows.

### Environment sanitization fixtures

Tests may mention environment variables such as:

```text
LOCALAPPDATA
SYSTEMROOT
```

simply to sanitize inherited environments. Such references are not necessarily Windows runtime support and require semantic classification.

### Build-directory cleanup

A cleanup helper that knows how to remove old `*-pc-windows-*` target directories is not, by itself, an active Windows product-support promise. Removing it is optional cleanup, not a success criterion.

### Historical records

Preserve truthful historical references in:

- changelogs;
- old reports;
- archived design notes;
- historical release records.

Do not retroactively rewrite history to pretend Jcode/HAVK never supported Windows or FreeBSD.

---

# 7. CI/release corrections in detail

## 7.1 CI end state

After debloat, current Windows-specific CI coverage should no longer exist.

Expected removals include:

```text
windows-build-test
powershell-syntax
windows-cross-check
```

Also remove:

- MSVC target installation used solely for Windows checks;
- `cargo xwin` installation/use if nothing retained requires it;
- Windows cache keys;
- Windows test selectors;
- native Windows lifecycle tests;
- PowerShell syntax validation for deleted scripts;
- comments describing Windows job exceptions;
- dead `runner.os` branches whose only remaining purpose was Windows exclusion.

Preserve:

- Linux CI;
- macOS CI;
- quality guardrails;
- setup-friction testing after its Windows-only section is removed;
- SDK tests that remain platform-neutral;
- release-script parsing/tests for retained releases.

---

## 7.2 Release end state

Remove future production of:

```text
jcode/havk Windows x86_64 artifacts
jcode/havk Windows ARM64 artifacts
FreeBSD official artifact
```

Remove from release workflow:

- native Windows build chain;
- Windows signing chain;
- Windows upload/download artifact steps;
- FreeBSD VM build chain;
- Windows/FreeBSD entries from final asset validation;
- Windows/FreeBSD availability environment variables;
- Windows/FreeBSD release summary lines;
- Windows/FreeBSD checksum inputs generated solely from removed artifacts;
- obsolete `needs:` relationships.

Do **not** delete historical release assets.

Do **not** delete repository-level secrets/variables automatically. Remove only source references; administrative cleanup is separate.

---

## 7.3 Release finalization policy must be decided

Before editing the final expected-asset set, the artifact needs one explicit answer:

### Option A

Required final artifacts:

```text
Linux x86_64
Linux ARM64
macOS Apple Silicon
macOS Intel
```

Any of the four failing keeps the release draft.

### Option B

Mandatory release gate:

```text
Linux x86_64
Linux ARM64
```

macOS artifacts are best-effort additions. Their failure does not block a Linux release.

The existing artifact should not be considered implementation-ready until this policy is fixed in writing.

---

# 8. Cargo and Rust safety rules — corrected version

## 8.1 Direct dependency removal

For each direct Windows target dependency:

1. identify the production/test code that consumes it;
2. remove the consumer as part of native Windows support removal;
3. confirm the dependency is unused by retained target paths;
4. remove the corresponding direct manifest declaration.

Do not remove dependencies only because their names look Windows-specific if a retained path still uses them.

---

## 8.2 Lockfile update rule

### Never do this blindly

```bash
cargo generate-lockfile
```

It may refresh unrelated packages to newer compatible versions.

### Never do this manually

Do not hand-delete package entries from `Cargo.lock`.

### Preferred non-compiling consistency flow

If Cargo manifests changed:

```bash
cargo metadata --format-version 1 > /dev/null
```

Then inspect:

```bash
git diff -- Cargo.lock
```

Reject unexplained/unrelated version churn.

Then validate:

```bash
cargo metadata --locked --format-version 1 > /dev/null
```

If that succeeds, the checked-in lockfile matches the dependency resolution Cargo expects.

If network/cache constraints prevent metadata resolution, do not fake the result. Report it and let remote CI perform the check.

### Do not use a string-purge success criterion

It is acceptable for `Cargo.lock` to still contain:

```text
windows-sys
windows-targets
windows_x86_64_*
windows_aarch64_*
```

when they are transitively referenced by retained third-party dependencies.

The success criterion is:

> no HAVK-owned direct native-Windows dependency or production code path remains unless separately justified.

---

## 8.3 `cfg` safety rules

Good removals:

```rust
#[cfg(windows)]
mod windows;
```

when `windows` exists only for removed native support.

Unsafe mechanical rewrite:

```rust
#[cfg(unix)]
```

becoming:

```rust
#[cfg(any(target_os = "linux", target_os = "macos"))]
```

without a concrete retained-target correctness reason.

Also do not add:

```rust
#[cfg(target_os = "freebsd")]
compile_error!(...);
```

merely to enforce the product policy.

Unsupported is not the same as intentionally broken.

---

# 9. Documentation policy — corrected version

## 9.1 Recommended support wording

### Linux

```text
Primary / fully supported
- x86_64-unknown-linux-gnu
- aarch64-unknown-linux-gnu
```

### macOS

```text
Best-effort / non-priority
- aarch64-apple-darwin
- x86_64-apple-darwin
```

State that fixes can come from:

- Jcode upstream;
- HAVK contributors;
- low-risk HAVK maintenance work when appropriate.

Do not promise the same proactive platform work as Linux.

### Windows

```text
Native Windows is unsupported.
Use WSL2 and run the Linux build inside the WSL2 distribution.
```

Do not publish `.exe` artifacts or maintain PowerShell install support.

### FreeBSD

Recommended wording:

```text
Unsupported — generic Unix portability is preserved where practical.
No official HAVK binary, dedicated CI, release guarantee, or maintenance SLA.
Community fixes may be accepted when low-risk and non-disruptive to supported targets.
```

---

## 9.2 Rewrite `docs/WINDOWS.md` instead of deleting it

Recommended purpose:

```text
# Windows / WSL2

HAVK does not support native Windows execution.
Windows users should install a Linux distribution with WSL2 and use the normal Linux HAVK installation path inside that distribution.

HAVK does not publish Windows .exe binaries and does not maintain a PowerShell installer.

WSL2 is recommended because it uses a Linux kernel and provides broad Linux system-call compatibility. Some CTF workflows requiring unusual hardware/device/kernel/network access may still work better on native Linux or a dedicated Linux VM.
```

Keep the page concise.

Update active link labels to match the new purpose.

---

# 10. Guardrail policy — new required section

The original artifact needs a section specifically for repository quality baselines.

Recommended rule:

> Deleting platform-owned files can interact with ratchet/budget files that store path/count baselines. Search these baselines before finishing. Deleted paths can usually be removed from the relevant baseline, but do not globally regenerate or relax budgets merely to make CI green.

Checklist:

```text
[ ] Search scripts/*budget*.json and guardrail configs for every deleted source path.
[ ] Inspect the checker behavior before modifying a baseline.
[ ] Update only entries directly affected by this task.
[ ] Do not increase unrelated limits.
[ ] Do not run a blanket --fix/--update and commit the output without diff review.
[ ] Keep cargo metadata --locked validation.
[ ] Remove stale comments that claim lock validation exists only in Windows CI.
```

Known current baseline needing review:

```text
scripts/swallowed_error_budget.json
```

---

# 11. Revised validation model

The original document should separate local execution constraints from correctness gates.

## 11.1 Stage A — local/static validation allowed

Examples:

```bash
git diff --check
cargo fmt --all -- --check
cargo metadata --locked --format-version 1 > /dev/null
```

plus semantic scans for:

```text
#[cfg(windows)]
target_os = "windows"
windows-sys direct declarations
std::os::windows
tokio Windows named pipes
windows-latest
pc-windows-msvc
cargo xwin
jcode/havk Windows release assets
install.ps1
uninstall.ps1
FreeBSD official release workflow/status
```

Every hit must be classified semantically; the search result does not dictate deletion by itself.

Also check:

```text
broken documentation links
references to deleted module/file paths
quality-budget references to deleted files
release needs/expected arrays
SDK optionalDependencies/package-lock consistency
```

---

## 11.2 Stage B — remote CI required

Because heavy local compilation is intentionally avoided, remote CI becomes a **required correctness gate**.

At minimum:

- retained Linux build/test CI must pass;
- retained macOS CI must pass according to the chosen support/release policy;
- quality guardrails must pass;
- setup-friction must pass after Windows parity removal;
- SDK checks must pass;
- no workflow references deleted jobs/files.

The agent may commit/push without local compilation if that is the project workflow, but it must not describe the change as fully validated until remote CI is green.

---

## 11.3 Stage C — retained release-target verification

The main PR CI currently does not exercise every release architecture in the same way.

Before the first HAVK release under the new policy, verify the retained release paths for:

```text
x86_64-unknown-linux-gnu
aarch64-unknown-linux-gnu
aarch64-apple-darwin
x86_64-apple-darwin
```

subject to the explicit macOS release-gating decision.

This can be done through the normal remote release workflow; it does not require heavy local compilation.

---

# 12. Proposed corrections to the original artifact

The following changes are recommended before changing its status back to implementation-ready.

## Correction 1 — status

Replace:

```yaml
status: "implementation-ready"
```

with something like:

```yaml
status: "reviewed-needs-corrections"
```

until all blocking corrections are incorporated.

---

## Correction 2 — add missing CI jobs

Explicitly list:

```text
windows-build-test
powershell-syntax
windows-cross-check
```

as Windows support jobs to remove.

---

## Correction 3 — add setup-friction dependency

Add:

```text
scripts/setup_friction_eval.sh
```

with an explicit instruction to remove only its native-Windows parity section while preserving retained setup-friction checks.

---

## Correction 4 — rewrite Windows doc in place

Prefer:

```text
docs/WINDOWS.md -> concise Windows/WSL2 policy page
```

rather than deletion.

Add active link reviewers:

```text
docs/README.md
docs/MULTI_SESSION_CLIENT_ARCHITECTURE.md
```

---

## Correction 5 — harden Cargo.lock instructions

Explicitly state:

```text
DO NOT blindly run cargo generate-lockfile.
DO NOT manually purge Windows-named package records.
```

Use Cargo metadata resolution + lockfile diff review + `cargo metadata --locked` validation when feasible.

---

## Correction 6 — define remote CI as validation gate

Move "no local heavy build" into an execution-constraint section.

Add:

> Remote CI is required before the implementation is considered validated/merge-ready.

---

## Correction 7 — choose macOS release-gating semantics

Choose either:

- all retained Linux+macOS assets are mandatory; or
- Linux Tier-1 assets are mandatory and macOS is optional/best-effort.

Do not leave this to the agent.

---

## Correction 8 — clarify signing-secret scope

Replace "remove signing secrets" with:

> Remove repository source references and workflow consumption of native-Windows signing configuration. Repository/org secret deletion is an administrative follow-up and is not authorized by this source-code task.

---

## Correction 9 — add guardrail baseline review

Add at minimum:

```text
scripts/swallowed_error_budget.json
scripts/check_guardrails.sh
```

and require a current-HEAD scan for other affected budgets.

---

## Correction 10 — add mixed Windows fixture review

Add:

```text
crates/jcode-app-core/src/update.rs
```

and instruct the agent to preserve parser coverage while removing misleading Windows-release semantics where practical.

---

## Correction 11 — tighten FreeBSD label

Prefer:

```text
Unsupported — portability preserved
```

instead of:

```text
Unofficial / community-compatible
```

unless HAVK intentionally wants to make a positive compatibility claim.

---

## Correction 12 — scope the WSL prohibition

Change a permanent prohibition into a task-scoped rule:

> Do not create WSL-specific compatibility branches merely to replace native Windows support during this debloat. Future evidence-backed WSL fixes belong in separate work.

---

# 13. Final safety judgment

## 13.1 Is the core idea correct?

**Yes.**

The source structure supports the strategy:

- Windows has a substantial native implementation and is a legitimate deep-debloat target.
- FreeBSD does not justify the same style of source purge; removing its dedicated CI/release contract while retaining generic Unix portability is cleaner and safer.
- WSL2 is a reasonable official recommendation for Windows users who need HAVK.
- macOS can remain a retained but lower-priority target.

---

## 13.2 Is the current artifact fully valid as written?

**Not fully.**

Its architectural rules are mostly correct, but implementation inventory and acceptance semantics have important gaps.

The largest concrete error is the omission of:

```text
scripts/setup_friction_eval.sh
```

which can make a Linux CI job fail immediately after native Windows installer removal.

Other material gaps are:

- unnamed `powershell-syntax` and `windows-cross-check` jobs;
- broken-link risk around `docs/WINDOWS.md`;
- vague lockfile repair instructions;
- missing guardrail baseline handling;
- unresolved macOS release gating;
- treating “no local build” too much like a correctness criterion;
- ambiguous wording around repository-level signing-secret deletion.

---

## 13.3 Is the debloat safe?

**Yes, if performed with the corrected boundaries in this review.**

It is **not** safe as a blind string-removal exercise.

The safe implementation rule is:

> Remove Windows-owned implementation and official FreeBSD maintenance infrastructure; preserve retained Linux/macOS behavior, shared protocol logic, and generic Unix portability; repair every direct CI/release/docs/guardrail dependency of deleted paths; then require remote compilation/CI validation.

---

## 13.4 Should the current spec be given to an AI agent now?

**No — revise it first.**

After the blocker/high-severity corrections are merged into `HAVK_OS_SUPPORT_DEBLOAT.md`, it can reasonably become an implementation artifact for an AI coding agent.

The corrected artifact should specifically eliminate the need for the agent to guess:

1. which CI jobs disappear;
2. what happens to `setup-friction`;
3. what happens to `docs/WINDOWS.md` and inbound links;
4. how `Cargo.lock` may be refreshed safely;
5. whether macOS failure blocks an official release;
6. whether repository-level secrets may be altered;
7. how quality-budget baselines are handled;
8. when validation is truly complete.

---

# 14. References

## Rust

- Rust Reference — Conditional compilation:  
  <https://doc.rust-lang.org/reference/conditional-compilation.html>
- rustc book — Platform support:  
  <https://doc.rust-lang.org/rustc/platform-support.html>
- rustc book — FreeBSD targets:  
  <https://doc.rust-lang.org/rustc/platform-support/freebsd.html>

## Cargo

- Cargo Book — Platform-specific dependencies:  
  <https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#platform-specific-dependencies>
- Cargo Book — `cargo metadata`:  
  <https://doc.rust-lang.org/cargo/commands/cargo-metadata.html>
- Cargo Book — `cargo generate-lockfile`:  
  <https://doc.rust-lang.org/cargo/commands/cargo-generate-lockfile.html>

## Microsoft

- Microsoft Learn — Comparing WSL versions / WSL2 architecture:  
  <https://learn.microsoft.com/windows/wsl/wsl2-about>

## Jcode upstream

- Repository:  
  <https://github.com/1jehuang/jcode>
- Native Windows support documentation:  
  <https://github.com/1jehuang/jcode/blob/master/docs/WINDOWS.md>

---

# 15. One-line reviewer conclusion

> **The HAVK platform-debloat strategy is defensible and technically valid, but the current implementation artifact needs targeted corrections before an autonomous agent can execute it safely; the highest-risk omissions are hidden Windows dependencies in Linux CI, incomplete CI job enumeration, documentation-link integrity, lockfile procedure, and release-gating ambiguity.**
