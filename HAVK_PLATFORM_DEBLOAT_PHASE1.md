---
title: "HAVK Platform Debloat Plan — Phase 1"
source_archive: "jcode-0.88.0.tar.gz"
source_version: "Jcode 0.88.0"
artifact_status: "Draft / living plan"
scope: "Platform support only"
last_audited: "2026-09-26"
---

# HAVK Platform Debloat Plan — Phase 1

> **Purpose**
>
> This artifact records the first agreed debloat scope for HAVK, based on a static audit of the uploaded `jcode-0.88.0.tar.gz` source archive.
>
> This phase is intentionally limited to **operating-system / architecture support cleanup**. Other HAVK debloat decisions will be added in later phases.

## 1. Source audited

The audit in this document is based on:

```text
jcode-0.88.0.tar.gz
└── jcode-0.88.0/
```

No GitHub checkout was used for this audit.

The archive does not need to be compiled to establish the platform-removal surface documented here. The findings below come from static inspection of source files, Cargo manifests, scripts, SDK packages, documentation, tests, and GitHub Actions workflows.

---

# 2. Platform policy for HAVK

This policy is the fixed baseline for the platform debloat.

| Platform | Architecture | HAVK support policy | Priority |
|---|---|---:|---:|
| Linux | `x86_64` / x64 | **Fully supported** | Tier 1 / primary |
| Linux | `aarch64` / ARM64 | **Fully supported** | Tier 1 / primary |
| macOS | `x86_64` / Intel | **Supported** | Tier 2 / non-priority |
| macOS | `aarch64` / Apple Silicon | **Supported** | Tier 2 / non-priority |
| Windows | `x86_64` / x64 | **Removed / unsupported** | None |
| Windows | `aarch64` / ARM64 | **Removed / unsupported** | None |
| FreeBSD | `x86_64` | **Removed / unsupported** | None |

## 2.1 Interpretation

For HAVK, "remove Windows support" means removing the complete native Windows support surface, not merely hiding Windows from documentation.

That includes, where applicable:

- Rust `#[cfg(windows)]` implementations;
- Rust `#[cfg(not(unix))]` branches whose only practical supported target was Windows;
- Windows-specific dependencies such as `windows-sys`;
- Windows named-pipe transport;
- Windows filesystem / ACL / process / terminal behavior;
- Windows installers and uninstallers;
- Windows setup hints and hotkeys;
- Windows CI and smoke tests;
- Windows release builds and signing;
- Windows TypeScript SDK runtime packages;
- Windows-specific tests;
- current documentation advertising Windows support.

For HAVK, "remove FreeBSD support" means:

- no FreeBSD CI;
- no FreeBSD release artifact;
- no FreeBSD smoke workflow;
- no release gating/status/reporting for FreeBSD;
- no current documentation or developer instructions claiming FreeBSD support;
- no FreeBSD-specific compatibility work maintained for HAVK.

Generic Unix code that is already required by Linux and macOS **must not be removed merely because it also happens to compile on FreeBSD**.

---

# 3. Non-negotiable execution constraints

These constraints apply when an implementation agent later performs this debloat.

## 3.1 Do not compile or run heavy CI locally

The local development machine is not intended to perform a full Jcode/HAVK build during this cleanup.

**Do not run:**

```text
cargo build --workspace
cargo test --workspace
cargo check --workspace
full release builds
cross-platform builds
local CI/CD equivalents
```

Do not attempt to reproduce GitHub Actions locally.

The implementation should be based on careful static edits, dependency cleanup, formatting where safe, and source inspection.

Validation requiring compilation belongs to GitHub Actions after the commit reaches the remote branch / pull request.

## 3.2 Commit behavior

For a future coding-agent implementation task:

- make the requested source changes;
- inspect the resulting diff carefully;
- do not perform a local full build just to validate it;
- create the commit when the requested scope is complete;
- **do not push unless explicitly instructed**;
- the user can push the commit and let GitHub Actions perform the expensive validation.

---

# 4. Audit summary from Jcode 0.88.0

Static inspection found a substantial Windows surface and a much smaller explicit FreeBSD surface.

Approximate static inventory from the archive:

| Finding | Static count | Meaning |
|---|---:|---|
| Rust/Cargo files with direct Windows cfg/target logic | **62 files** | Windows support is spread across multiple core crates, not isolated to one module |
| Rust files containing `not(unix)` platform branches | **33 files** | A second cleanup surface that should be reviewed after Windows removal |
| Files explicitly mentioning FreeBSD | **14 files** | FreeBSD support is concentrated primarily in release/CI plus a few portability comments/tests |
| Windows-named source/workflow/test paths | **10 paths** | High-confidence direct deletion candidates |
| Windows npm runtime package directories | **2 directories** | `win32-x64` and `win32-arm64` |

These counts are useful as an audit baseline, not as acceptance criteria by themselves. A blind search-and-delete would be unsafe because words such as "window" can refer to GUI windows rather than Microsoft Windows.

---

# 5. Desired end state

After this platform debloat, HAVK should conceptually look like this:

```text
HAVK
├── Linux
│   ├── x86_64        FULL SUPPORT
│   └── aarch64       FULL SUPPORT
│
├── macOS
│   ├── x86_64        SUPPORTED, NON-PRIORITY
│   └── aarch64       SUPPORTED, NON-PRIORITY
│
├── Windows           UNSUPPORTED / NO NATIVE BUILD
│   ├── x86_64        REMOVED
│   └── aarch64       REMOVED
│
└── FreeBSD
    └── x86_64        REMOVED
```

The important architectural simplification is that the HAVK CLI/runtime effectively becomes a **Unix-oriented Linux + macOS project** rather than a project maintaining two separate local-IPC/process/filesystem families for Unix and Windows.

---

# 6. Windows removal plan

Windows is the large part of this phase.

The work should be performed by layer rather than by blindly deleting every occurrence of the word `windows`.

## 6.1 GitHub Actions — remove Windows CI

### Delete

```text
.github/workflows/windows-smoke.yml
```

This workflow contains dedicated Windows x64 and Windows ARM64 smoke jobs.

### Edit

```text
.github/workflows/ci.yml
```

Remove the dedicated Windows build/test job and any Windows-only CI steps, comments, conditions, cache keys, PowerShell syntax checks, or warning-budget language that only exists because Windows is supported.

Known Windows-specific areas include the job beginning around:

```text
windows-build-test:
  name: Build & Test (windows-latest)
```

Also review conditions such as:

```text
if: runner.os == 'Windows'
if: runner.os != 'Windows'
```

After Windows support is gone, Linux/macOS CI should no longer contain commentary or exceptions describing a separate supported Windows surface.

## 6.2 GitHub Actions — remove Windows release production

### Edit

```text
.github/workflows/release.yml
```

Remove the complete Windows release pipeline, including:

```text
build-windows
publish-windows
```

Remove both targets:

```text
x86_64-pc-windows-msvc
aarch64-pc-windows-msvc
```

Remove Windows artifacts:

```text
jcode-windows-x86_64
jcode-windows-aarch64
windows-unsigned-x86_64
windows-unsigned-aarch64
```

Remove:

- Windows runner entries;
- ARM Windows runner entries;
- Windows executable packaging;
- `.exe` release files;
- `.tar.gz` Windows release bundles;
- Windows Authenticode signing;
- signing prerequisites/secrets/status handling;
- Windows artifact upload jobs;
- Windows artifact availability variables;
- Windows status lines in release notes;
- Windows artifacts from checksum generation;
- Windows jobs from `needs:` dependency chains.

The final release workflow should only aggregate Linux + macOS platform artifacts for the platform scope currently retained by HAVK.

## 6.3 Delete Windows release/install verification helper

### Delete

```text
.github/scripts/verify_windows_install.ps1
```

It exists solely to validate the native Windows installation path.

---

# 7. FreeBSD removal plan

FreeBSD support is much more concentrated than Windows support.

## 7.1 Delete dedicated FreeBSD smoke CI

### Delete

```text
.github/workflows/freebsd-smoke.yml
```

Jcode currently boots a FreeBSD VM under QEMU through `vmactions/freebsd-vm` to perform a real FreeBSD build/smoke test.

HAVK should not carry this workflow.

## 7.2 Remove FreeBSD from release production

### Edit

```text
.github/workflows/release.yml
```

Delete the job:

```text
build-freebsd
```

Delete the target/artifact handling around:

```text
x86_64-unknown-freebsd
jcode-freebsd-x86_64
jcode-freebsd-x86_64.tar.gz
```

Remove FreeBSD from:

- `needs:` dependencies;
- artifact downloads;
- checksum inputs;
- availability detection;
- release-note status output;
- final platform summaries.

## 7.3 Remove FreeBSD assumptions from quick release scripts

### Edit

```text
scripts/quick-release.sh
```

Current release messages explicitly say that CI will add FreeBSD artifacts.

Those references should be removed so HAVK release instructions describe only the retained Linux/macOS targets.

## 7.4 Review FreeBSD-specific developer test

### Edit or remove the relevant test case from

```text
scripts/test_dev_cargo_jobs.sh
```

The archive contains a test using:

```text
TEST_UNAME_S=FreeBSD
```

If its only purpose is to maintain FreeBSD behavior, remove it.

If it is actually testing a generic unsupported-OS fallback that HAVK still intentionally wants, rewrite the case to a generic unsupported platform rather than preserving FreeBSD support implicitly.

## 7.5 TypeScript SDK FreeBSD test

### Review

```text
sdk/typescript/test/launch.test.ts
```

The file currently asserts that:

```text
platformBinaryPackage("freebsd", "x64") === undefined
```

This is not FreeBSD *support*; it verifies that the SDK does **not** provide a FreeBSD runtime package.

Therefore this line does **not** need deletion merely because FreeBSD support is being removed. It may remain as an explicit unsupported-platform regression test if useful.

---

# 8. Windows installers and PowerShell surface

HAVK should not maintain a native Windows installation path after Windows support is removed.

## 8.1 Delete native Windows installer

### Delete

```text
scripts/install.ps1
```

This is a full Windows installer with architecture detection, release selection, PATH setup, hotkey integration, upgrade handling, source-build handling, and Windows process logic.

It currently selects:

```text
jcode-windows-x86_64
jcode-windows-aarch64
```

All of that becomes unnecessary.

## 8.2 Delete native Windows uninstaller

### Delete

```text
scripts/uninstall.ps1
```

## 8.3 Delete Windows installer tests

### Delete

```text
scripts/test_windows_launcher_install.ps1
scripts/test_windows_setup_evaluation.ps1
```

## 8.4 Review remaining PowerShell utilities

The archive contains:

```text
scripts/check_powershell_syntax.ps1
scripts/invoke_cargo_with_timeout.ps1
```

If these scripts have no non-Windows consumer after the Windows CI jobs are removed, delete them as part of the same cleanup.

Do not retain PowerShell infrastructure solely for an operating system HAVK no longer supports.

## 8.5 Simplify the POSIX installer

### Edit

```text
scripts/install.sh
```

The shell installer currently detects Git Bash / MSYS / Cygwin:

```text
MINGW*|MSYS*|CYGWIN*)
```

and resolves either:

```text
jcode-windows-x86_64
jcode-windows-aarch64
```

Delete this Windows branch.

Retain only the HAVK-supported download paths:

```text
Linux x86_64
Linux aarch64
macOS x86_64
macOS aarch64
```

Any additional Linux environment support already implemented inside the Linux branch, such as Termux handling, should be evaluated separately and **must not be removed merely as part of Windows/FreeBSD cleanup**.

---

# 9. TypeScript SDK runtime packages

Jcode ships per-platform npm packages.

Current archive:

```text
sdk/npm/
├── darwin-arm64/
├── darwin-x64/
├── linux-arm64/
├── linux-x64/
├── win32-arm64/
└── win32-x64/
```

HAVK should retain:

```text
sdk/npm/darwin-arm64/
sdk/npm/darwin-x64/
sdk/npm/linux-arm64/
sdk/npm/linux-x64/
```

## 9.1 Delete Windows runtime packages

### Delete directories

```text
sdk/npm/win32-arm64/
sdk/npm/win32-x64/
```

## 9.2 Edit TypeScript runtime mapping

### Edit

```text
sdk/typescript/src/binary.ts
```

Remove:

```ts
"win32-x64"
"win32-arm64"
```

from the platform package map.

Retain:

```ts
"linux-x64"
"linux-arm64"
"darwin-x64"
"darwin-arm64"
```

## 9.3 Edit SDK runtime-package preparation

### Edit

```text
scripts/prepare_sdk_runtime_packages.sh
```

Remove preparation for:

```text
win32-x64
win32-arm64
```

including `.exe` handling.

## 9.4 Edit SDK publishing workflow

### Edit

```text
.github/workflows/publish-typescript-sdk.yml
```

Remove release download patterns for:

```text
jcode-windows-x86_64.tar.gz
jcode-windows-aarch64.tar.gz
```

Ensure only Linux/macOS runtime assets are required/prepared/published.

## 9.5 Remove Windows named-pipe handling from SDK sockets

### Edit

```text
sdk/typescript/src/sockets.ts
```

Current code has a Windows-specific username branch and `transportEndpoint()` implementation that maps a socket path to a Windows named pipe.

With Windows removed:

- remove `process.platform === "win32"` handling;
- remove named-pipe derivation;
- simplify `transportEndpoint(socketPath)` to the Unix socket path behavior, or remove the abstraction entirely if it becomes redundant.

### Delete or rewrite

```text
sdk/typescript/test/pipe-name.test.ts
```

This test exists specifically to ensure the TypeScript Windows pipe derivation matches `jcode-transport`.

Once Windows named pipes are removed from both sides, this test has no remaining purpose and should normally be deleted.

---

# 10. Rust transport layer

This is one of the cleanest architectural wins from removing Windows.

## 10.1 Current Jcode design

```text
crates/jcode-transport/
├── src/lib.rs
├── src/unix.rs
└── src/windows.rs
```

The crate currently abstracts:

```text
Unix socket on Unix
Windows named pipe on Windows
```

## 10.2 Delete Windows transport implementation

### Delete

```text
crates/jcode-transport/src/windows.rs
```

## 10.3 Simplify transport module wiring

### Edit

```text
crates/jcode-transport/src/lib.rs
```

Remove:

```rust
#[cfg(windows)]
mod windows;

#[cfg(windows)]
pub use windows::*;
```

The Unix implementation becomes the only transport implementation required by supported HAVK desktop/CLI platforms.

The module can be simplified so callers no longer need a cross-family IPC abstraction for Windows named pipes.

## 10.4 Remove Windows-only transport dependencies

### Edit

```text
crates/jcode-transport/Cargo.toml
```

Remove the Windows target dependency block and Windows-only dependencies that exist for pipe-name creation / Windows errors:

```text
sha2
hex
windows-sys
```

**Only remove `sha2` / `hex` from this crate if they are not used elsewhere in its retained Unix implementation.**

The static audit shows they are currently grouped under the Windows target block.

## 10.5 Update transport documentation/comments

The crate description currently says that it provides Unix sockets or Windows named pipes behind one API.

Rewrite it to describe the retained Unix socket transport instead.

---

# 11. Harness API bridge simplification

Windows named-pipe support leaks into the harness API bridge.

## 11.1 Edit

```text
crates/jcode-harness-api-server/src/lib.rs
```

Current code has separate listener binding semantics:

```rust
#[cfg(windows)]
let mut listener = ...;

#[cfg(unix)]
let listener = ...;
```

It also documents Windows named pipes vs Unix socket files.

After Windows removal:

- delete the Windows listener branch;
- retain the Unix socket lock/unlink/permissions behavior;
- simplify comments so they no longer explain Windows named-pipe exceptions.

## 11.2 Edit

```text
crates/jcode-harness-api/src/sockets.rs
```

The function `runtime_user_discriminator()` currently has both:

```rust
#[cfg(unix)]
...

#[cfg(not(unix))]
...
```

For the retained Linux/macOS surface, the non-Unix variant is unnecessary and should be removed unless the crate intentionally remains portable to another supported non-Unix consumer.

---

# 12. Windows-specific Cargo dependencies

At minimum, direct Windows dependency blocks exist in these manifests:

```text
Cargo.toml
crates/jcode-app-core/Cargo.toml
crates/jcode-base/Cargo.toml
crates/jcode-core/Cargo.toml
crates/jcode-setup-hints/Cargo.toml
crates/jcode-transport/Cargo.toml
```

They include `windows-sys` features for areas such as:

- Win32 foundation APIs;
- threading;
- power management;
- security;
- keyboard/mouse input;
- Windows messaging.

## Required action

After the corresponding Windows Rust branches are deleted, remove these target dependency blocks and regenerate the lockfile **through the normal repository workflow/CI-compatible method**, not through a resource-heavy full local compile.

Do not leave `windows-sys` direct dependencies in HAVK if no retained code requires them.

A later lockfile audit should determine whether `windows-sys` still appears transitively because of third-party crates. A transitive lockfile entry is not automatically evidence of remaining HAVK Windows support.

---

# 13. Setup hints and global hotkeys

Jcode contains substantial platform-specific setup guidance.

## 13.1 Delete Windows setup implementation

### Delete

```text
crates/jcode-setup-hints/src/windows_setup.rs
crates/jcode-setup-hints/src/windows_hotkeys.rs
```

## 13.2 Edit setup-hints module wiring

### Edit

```text
crates/jcode-setup-hints/src/lib.rs
```

Remove imports/modules/branches involving:

```rust
windows
#[cfg(windows)]
#[cfg(any(test, windows))]
#[cfg(any(..., windows))]
```

Preserve Linux and macOS behavior.

The retained setup-hints design should describe only:

- Linux behavior;
- macOS behavior.

## 13.3 Edit CLI launch hint logic

### Edit

```text
crates/jcode-setup-hints/src/cli_launch_hints.rs
```

Remove Windows-specific launch/setup paths and simplify conditions that currently have Linux/macOS/Windows branches.

## 13.4 Remove Windows-only dependency block

### Edit

```text
crates/jcode-setup-hints/Cargo.toml
```

Remove its `windows-sys` target dependency.

---

# 14. Storage, filesystem permissions, and ACL cleanup

Windows support is embedded in storage/security behavior, not only release scripts.

## 14.1 Primary file

```text
crates/jcode-storage/src/lib.rs
```

The file contains Windows-specific branches for secret/config hardening and ACL scheduling.

Examples include functions/branches around concepts such as:

```text
schedule_windows_path_hardening
harden_secret_file_permissions_windows
Windows ACL hardening worker
#[cfg(windows)]
#[cfg(not(windows))]
```

## Required action

Remove the Windows ACL implementation and make the retained Unix owner-only permission path the normal supported implementation.

This is a good candidate for real simplification rather than merely changing `#[cfg(not(windows))]` to another cfg.

Linux and macOS both use Unix-style permission behavior, so the supported code path can often become unconditional inside the HAVK-supported target set.

---

# 15. Core filesystem/process/platform branches

Windows cfg logic appears across many core modules.

High-value review areas include:

```text
crates/jcode-core/src/fs.rs
crates/jcode-core/src/console.rs
crates/jcode-core/src/stdin_detect.rs
crates/jcode-base/src/platform.rs
crates/jcode-base/src/power_inhibit.rs
crates/jcode-base/src/background.rs
crates/jcode-base/src/browser.rs
crates/jcode-base/src/browser_detect.rs
crates/jcode-app-core/src/server/socket.rs
crates/jcode-app-core/src/server/runtime.rs
crates/jcode-app-core/src/session_launch.rs
crates/jcode-terminal-launch/src/lib.rs
src/cli/terminal.rs
src/main.rs
```

## Required action pattern

For each branch:

1. determine whether the Windows side is the only non-Unix implementation;
2. delete the Windows implementation;
3. preserve Linux/macOS differences where they are real;
4. simplify the remaining Unix path when safe;
5. remove now-dead helpers/tests/imports/dependencies.

Do **not** mechanically replace every `#[cfg(not(windows))]` with nothing before confirming that macOS and Linux both support the retained behavior.

---

# 16. `#[cfg(not(unix))]` cleanup

The archive contains approximately **33 Rust files** with `not(unix)` branches.

Because the currently retained HAVK desktop/CLI platforms are Linux and macOS, both are Unix-family targets.

Therefore many of these branches become dead support code once Windows is removed.

Representative files include:

```text
src/cli/dispatch.rs
src/cli/auth_import.rs
src/cli/ssh.rs
src/cli/terminal.rs
src/cli/commands.rs
crates/jcode-harness-api/src/sockets.rs
crates/jcode-storage/src/active_pids.rs
crates/jcode-storage/src/lib.rs
crates/jcode-build-support/src/paths.rs
crates/jcode-sdk/src/client.rs
crates/jcode-sdk/src/launch.rs
crates/jcode-terminal-launch/src/lib.rs
crates/jcode-app-core/src/session_launch.rs
crates/jcode-app-core/src/server/socket.rs
crates/jcode-app-core/src/server/lifecycle.rs
crates/jcode-app-core/src/notifications.rs
crates/jcode-base/src/platform.rs
```

## Important rule

Treat `not(unix)` as a **review queue**, not an automatic deletion command.

Why:

- some branches are genuine Windows implementations and should disappear;
- some may be generic unsupported-platform fallbacks that are still useful for clear error reporting;
- tests may intentionally compile fallback helpers on Unix;
- library crates might have broader portability expectations than the HAVK CLI binary.

The goal is to remove real maintenance burden without breaking useful explicit "unsupported platform" diagnostics.

---

# 17. Terminal-launch Windows behavior

### Review

```text
crates/jcode-terminal-launch/src/lib.rs
```

The file contains non-Unix paths using Windows command execution, including logic around:

```text
cmd.exe
windows_arg_quote
windows_command_line
```

Once Windows is unsupported:

- remove native Windows terminal spawning;
- remove Windows command-line quoting helpers if no retained tests/use remain;
- remove associated non-Unix branches;
- retain macOS-specific Terminal/AppleScript behavior;
- retain Linux terminal behavior.

### Delete

```text
crates/jcode-terminal-launch/src/windows_portable_tests.rs
```

if it is only testing Windows portable behavior, as indicated by its name and current platform scope.

---

# 18. Windows end-to-end tests

### Delete

```text
tests/e2e/windows_lifecycle.rs
```

### Edit

```text
tests/e2e/main.rs
```

Remove module wiring such as:

```rust
#[cfg(windows)]
```

for Windows lifecycle tests.

Also remove Windows-only targeted test names from GitHub Actions once the associated implementation is deleted.

---

# 19. Documentation cleanup

Current-user documentation must accurately reflect the HAVK platform policy.

## 19.1 Delete dedicated Windows documentation

### Delete

```text
docs/WINDOWS.md
```

## 19.2 Edit root README

### Edit

```text
README.md
```

Remove:

- Windows PowerShell installation instructions;
- Windows x64/ARM64 support claims;
- Windows installer behavior;
- Defender/SmartScreen notes;
- Windows documentation links.

Update the platform table from Jcode's current model to HAVK's policy:

```text
Linux x86_64 / aarch64        Fully supported
macOS Intel / Apple Silicon   Supported, non-priority
Windows                       Unsupported
FreeBSD                       Unsupported
```

Whether unsupported platforms should be explicitly listed or simply omitted can be decided during final documentation polish. For development clarity, this plan recommends explicitly stating them at least once.

## 19.3 Edit releasing documentation

### Edit

```text
RELEASING.md
```

Remove:

- Windows release timing;
- Windows build matrix;
- signing documentation;
- Windows signing secrets;
- Windows artifact requirements;
- references to CI completing Windows artifacts.

Also remove FreeBSD from any release narrative if present or added elsewhere.

The resulting release document should describe Linux and macOS only.

## 19.4 Edit TypeScript SDK docs

### Edit

```text
sdk/typescript/README.md
sdk/typescript/RELEASING.md
```

Remove claims that runtime packages are shipped for Windows.

Retain Linux and macOS runtime-package documentation.

---

# 20. Historical changelog policy

**Do not rewrite history.**

The archive contains old changelog files that mention Windows and FreeBSD releases, bug fixes, or support milestones.

Examples include entries under:

```text
changelog/
```

These historical records describe what Jcode shipped at the time.

They should normally remain untouched during HAVK platform debloat unless the entire Jcode changelog is later intentionally replaced as part of the HAVK rebranding/history strategy.

Therefore:

```text
Historical Windows mention != active Windows support
Historical FreeBSD mention != active FreeBSD support
```

Do not inflate the debloat diff by editing historical release notes solely to erase platform names.

---

# 21. Critical false positives — do not delete blindly

## 21.1 `crates/jcode-app-core/src/tool/computer/win.rs`

**KEEP.**

Despite its filename, this is not Microsoft Windows support.

The file implements **macOS application/window management** using System Events / NSWorkspace / AppleScript/JXA concepts.

It is loaded under:

```rust
#[cfg(target_os = "macos")]
mod win;
```

Here `win` means **window**, not **Windows OS**.

Deleting it because of its filename would break retained macOS functionality.

## 21.2 Human-language uses of "window/windows"

Search results contain many references such as:

```text
terminal window
application windows
front window
browser window
```

These are not Windows OS support.

Never use a global text replacement/deletion for the word `windows` without understanding context.

## 21.3 Generic `cfg(unix)`

**Do not delete generic Unix implementations.**

Linux and macOS both require them.

FreeBSD may happen to use the same path today, but that is not a reason to duplicate or destroy Unix code that HAVK still needs.

---

# 22. FreeBSD and `cfg(unix)`: intentional unsupported-platform policy

A subtle point: removing the explicit FreeBSD workflow does not automatically stop every crate from compiling on FreeBSD because Rust's `cfg(unix)` also matches FreeBSD.

HAVK has two reasonable policies:

## Recommended policy

Treat Linux + macOS as the **supported platform contract**, while allowing shared Unix code to remain generic where that reduces complexity.

Do not spend effort breaking generic Unix portability merely to make FreeBSD compilation impossible.

Instead:

- stop testing FreeBSD;
- stop releasing FreeBSD;
- stop documenting FreeBSD as supported;
- do not accept FreeBSD-specific maintenance burden as part of the HAVK support contract.

If a strict compile-time guard is later desired for the HAVK binary, add an explicit supported-OS boundary at the binary/application layer rather than replacing every `cfg(unix)` throughout the workspace.

Conceptually:

```rust
// Example policy only — exact placement should be chosen during implementation.
#[cfg(not(any(target_os = "linux", target_os = "macos")))]
compile_error!("HAVK currently supports Linux and macOS only");
```

This is preferable to scattering Linux/macOS target lists through every shared Unix module.

---

# 23. macOS support must remain intact

macOS is **not** part of this removal phase.

Retain both current release architectures:

```text
aarch64-apple-darwin
x86_64-apple-darwin
```

Retain relevant macOS-only dependencies and functionality, including areas such as:

```text
global-hotkey
objc2
objc2-foundation
objc2-app-kit
macOS notification broker
macOS setup hints
macOS launcher/hotkeys
macOS terminal integration
macOS computer-use implementation
```

"Non-priority" means HAVK development is Linux-first; it does **not** mean macOS code should be removed in this phase.

---

# 24. Linux support must remain complete

Retain both Linux release architectures:

```text
x86_64-unknown-linux-gnu
aarch64-unknown-linux-gnu
```

Retain current Linux release artifacts conceptually equivalent to:

```text
havk-linux-x86_64
havk-linux-aarch64
```

after the later rebranding work is applied.

Platform cleanup must not accidentally remove Linux-specific code simply because it is adjacent to Windows branches.

Linux is HAVK's primary supported environment and should receive the simplest, clearest retained code path after this debloat.

---

# 25. Suggested implementation order

This ordering minimizes dangling references and makes review easier.

## Step 1 — remove dedicated platform-only files

Delete high-confidence Windows/FreeBSD-only files first:

```text
.github/workflows/windows-smoke.yml
.github/workflows/freebsd-smoke.yml
.github/scripts/verify_windows_install.ps1
scripts/install.ps1
scripts/uninstall.ps1
scripts/test_windows_launcher_install.ps1
scripts/test_windows_setup_evaluation.ps1
docs/WINDOWS.md
tests/e2e/windows_lifecycle.rs
crates/jcode-setup-hints/src/windows_hotkeys.rs
crates/jcode-setup-hints/src/windows_setup.rs
crates/jcode-transport/src/windows.rs
crates/jcode-terminal-launch/src/windows_portable_tests.rs
sdk/npm/win32-x64/
sdk/npm/win32-arm64/
```

Review and likely delete after confirming no remaining consumer:

```text
scripts/check_powershell_syntax.ps1
scripts/invoke_cargo_with_timeout.ps1
sdk/typescript/test/pipe-name.test.ts
```

## Step 2 — repair module/import references

Update Rust/TypeScript module wiring so deleted files are no longer referenced.

## Step 3 — simplify Cargo target dependencies

Remove direct Windows target dependency blocks after their users are gone.

## Step 4 — simplify Windows cfg branches in core code

Work crate-by-crate rather than repository-wide blind replacement.

Recommended order:

```text
jcode-transport
jcode-harness-api / jcode-harness-api-server
jcode-setup-hints
jcode-storage
jcode-core
jcode-base
jcode-terminal-launch
jcode-app-core
root CLI
```

## Step 5 — clean `not(unix)` fallbacks

Review the ~33 Rust files containing `not(unix)` and remove Windows-only dead paths.

## Step 6 — simplify installers and SDK packaging

Make supported platform mappings Linux/macOS-only.

## Step 7 — simplify CI/release workflows

Remove Windows and FreeBSD jobs/artifacts/dependencies/statuses.

This can be done earlier if preferred, but it should be reviewed after package names are stable.

## Step 8 — update current documentation

README, releasing docs, SDK docs, contributor instructions.

## Step 9 — static residual audit

Without compiling locally, run lightweight searches such as:

```bash
rg -n 'cfg\(windows\)|target_os\s*=\s*"windows"|x86_64-pc-windows-msvc|aarch64-pc-windows-msvc'
rg -n 'jcode-windows|win32-x64|win32-arm64'
rg -n 'freebsd|FreeBSD|x86_64-unknown-freebsd'
rg -n '#\[cfg\(not\(unix\)\)'
```

Every remaining match should be reviewed and classified, not automatically deleted.

## Step 10 — commit; remote CI validates

No full local compile is required by this plan.

Commit the completed scope and allow GitHub Actions on the pushed branch/PR to expose compile/test issues.

---

# 26. Acceptance criteria for this phase

The platform debloat is complete when all of the following are true.

## Platform contract

- [ ] Linux x86_64 remains a supported release target.
- [ ] Linux aarch64 remains a supported release target.
- [ ] macOS Intel remains supported.
- [ ] macOS Apple Silicon remains supported.
- [ ] Windows x86_64 is no longer supported or released.
- [ ] Windows ARM64 is no longer supported or released.
- [ ] FreeBSD is no longer supported or released.

## CI / release

- [ ] No Windows smoke workflow remains.
- [ ] No FreeBSD smoke workflow remains.
- [ ] No Windows build/release/signing jobs remain.
- [ ] No FreeBSD release build remains.
- [ ] Release checksums/status summaries do not expect Windows or FreeBSD artifacts.

## Packaging

- [ ] No `win32-x64` npm runtime package remains.
- [ ] No `win32-arm64` npm runtime package remains.
- [ ] TypeScript binary selection only maps retained Linux/macOS packages.
- [ ] SDK publishing does not download or publish Windows runtime assets.

## Installers

- [ ] Native Windows PowerShell installer is removed.
- [ ] Native Windows uninstaller is removed.
- [ ] POSIX installer no longer contains Git Bash/MSYS/Cygwin Windows artifact installation logic.

## Rust

- [ ] Dedicated Windows transport implementation is removed.
- [ ] Direct Windows target dependency blocks are removed where no longer required.
- [ ] Windows setup/hotkey implementation is removed.
- [ ] Windows storage ACL implementation is removed.
- [ ] Windows-only terminal/process/filesystem branches are removed or simplified.
- [ ] Remaining `not(unix)` branches have been intentionally reviewed.

## Documentation

- [ ] Current README no longer advertises Windows support.
- [ ] `docs/WINDOWS.md` is removed.
- [ ] Current release documentation no longer describes Windows signing/builds.
- [ ] Current documentation does not advertise FreeBSD support.
- [ ] Historical changelogs are not rewritten merely to erase old platform references.

## Safety

- [ ] macOS `computer/win.rs` is preserved because it is macOS window management, not Windows OS code.
- [ ] Shared Unix code required by Linux/macOS is preserved.
- [ ] No full local build/CI was run as part of the debloat implementation unless the user explicitly changes this constraint.

---

# 27. Out of scope for Phase 1

This artifact intentionally does **not** yet decide whether HAVK should remove or retain other large Jcode features such as providers, telemetry, voice/dictation, browser/computer-use functionality, SDKs, iOS-related components, sponsorship/discovery components, memory systems, self-development tooling, remote compile, or other non-platform features.

Those will be evaluated separately so platform cleanup does not get mixed with unrelated architectural decisions.

---

# 28. Current decision record

```yaml
havk_platform_policy:
  linux:
    x86_64: fully_supported
    aarch64: fully_supported
    priority: primary

  macos:
    x86_64: supported
    aarch64: supported
    priority: non_priority

  windows:
    x86_64: remove
    aarch64: remove
    native_support: remove

  freebsd:
    x86_64: remove
    support: remove

implementation_constraints:
  local_full_compile: forbidden_by_current_plan
  local_full_test: forbidden_by_current_plan
  local_ci_emulation: forbidden_by_current_plan
  validation: github_actions_after_push
  agent_push: only_if_explicitly_requested
```

---

# 29. Phase status

**Phase 1 definition is ready.**

This file should remain a living artifact. The next debloat topics can be appended as additional phases while keeping this platform decision stable unless the user explicitly changes the support policy.
