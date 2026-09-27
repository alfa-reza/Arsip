# HAVK — Jcode Subscription Debloat Specification

> **Artifact type:** implementation specification for an autonomous coding agent  
> **Target:** HAVK fork of Jcode  
> **Audited baseline:** Jcode 0.88.0 source  
> **Policy owner:** HAVK  
> **Scope:** remove Jcode/Solo Systems first-party subscription, account, hosted-model, and subscription-dependent service coupling  
> **Status:** implementation-ready specification; re-cross-checked against Jcode 0.88.0 implementation and in-tree `docs/`, and still subject to revalidation against the repository's current `HEAD`

---

## 1. Purpose

This artifact defines exactly what must be removed, what must be edited, and what must be preserved when debloating **Jcode Subscription** from HAVK.

The target is not merely to hide a pricing command or disable one login option. The target is to remove the active product and code paths that make HAVK depend on the Jcode/Solo Systems first-party commercial account and hosted Subscription API.

The intended end state is:

```text
HAVK
├── third-party provider OAuth
├── third-party provider API keys
├── OpenAI-compatible provider profiles
├── local/self-hosted models
├── provider usage reporting that belongs to retained providers
├── generic account switching for retained providers
├── generic remote daemon/client operation
├── Memory and Browser functionality
├── JEV through retained BYOK providers
├── native/non-Jcode voice functionality if retained elsewhere
└── future provider-neutral cloud sandbox support

NO active Jcode Subscription provider
NO Jcode commercial account login
NO Jcode hosted-model catalog/router
NO Jcode subscription billing/tier/entitlement client
NO Jcode subscription-backed remote compilation
NO Jcode-managed Cloud activation path
NO Jcode-backed JEV route
NO Jcode subscription-backed voice route
NO Jcode subscription identity in support diagnostics
```

This artifact deliberately does **not** authorize broad removal of unrelated Jcode-hosted services or unrelated features.

---

## 2. Product decision

HAVK will not ship the Jcode/Solo Systems hosted-model Subscription API as a supported provider or commercial account system.

Jcode's public service boundary distinguishes the open-source software from hosted services. The Jcode Subscription API is a hosted, metered model-routing service. Jcode can independently operate with user-owned API credentials, supported provider OAuth access, and local models. HAVK will retain those independent paths while removing the first-party Jcode subscription path.

### 2.1 Remove

Remove active functionality for:

```text
Jcode Subscription
Jcode commercial account identity
Jcode subscription device login
Jcode subscription API key persistence
Jcode hosted-model curated catalog
Jcode hosted-model router identity
Jcode tier/status/usage/billing client
Jcode subscription onboarding and pricing nudges
Jcode subscription-specific provider runtime state
Jcode Remote Compile
Jcode-managed Cloud activation
Jcode-backed JEV route
Jcode subscription-backed voice transcription route
Jcode subscription identity in support diagnostics
subscription-specific telemetry residue, if telemetry remains
```

### 2.2 Preserve

Preserve:

```text
Claude OAuth / Claude subscription login
OpenAI / ChatGPT / Codex OAuth
Gemini login
GitHub Copilot login
Azure/provider API-key paths
normal xAI provider
Grok Build unless separately removed
OpenRouter BYOK
other named OpenAI-compatible profiles
Ollama / LM Studio / local models
generic provider login/account infrastructure
generic /account switching for retained providers
generic /usage and provider quota reporting
generic protocol Subscribe messages
Memory
Browser
JEV abstraction and retained BYOK JEV providers
native/non-Jcode voice paths unless separately removed
generic remote daemon/client architecture
generic WebSocket gateway and device pairing
SSH/socket forwarding
session persistence and reconnect
future Modal/E2B/provider-neutral sandbox integration
Telemetry as a whole unless separately debloated
Partner Tool Discovery unless separately debloated
Self-Dev unless separately debloated
```

---

## 3. Source-of-truth rules for the implementing agent

### 3.1 Current `HEAD` wins

The paths in this artifact are **known high-confidence hotspots**, not an immutable line-by-line patch plan.

Before editing:

1. read the repository root `AGENTS.md` and any nearer agent instructions;
2. inspect the current `HEAD` for each named module;
3. re-run the residue searches in this document;
4. confirm whether code has moved, been renamed, or already been removed;
5. preserve unrelated changes in the working tree.

If the current source contradicts a path or implementation detail recorded here, the current source is authoritative for implementation mechanics. The **product boundaries and acceptance criteria in this artifact remain authoritative for intent**.

### 3.2 Do not implement by keyword deletion

Search results are discovery inputs, not deletion instructions.

Every match containing words such as:

```text
subscription
subscribe
account
jcode
JCODE_API_KEY
JCODE_OPENROUTER_*
managed
cloud
remote
usage
```

must be classified as one of:

```text
SUBSCRIPTION-OWNED       -> remove
SHARED-INTEGRATION       -> surgically remove Jcode branch only
LEGACY-COMPATIBILITY     -> retain only if required for safe migration/readability
FALSE-POSITIVE           -> preserve
HISTORICAL-RECORD        -> normally preserve
```

A global search-and-replace is forbidden.

---

## 4. Mandatory warnings

> [!WARNING]
> **Do not remove third-party provider subscriptions.** "Subscription" in Claude, ChatGPT/OpenAI, Gemini, Copilot, or Grok Build does not mean the Jcode Subscription API.

> [!WARNING]
> **Do not remove generic protocol `Subscribe`.** `Request::Subscribe` is a client/server event subscription mechanism and has nothing to do with billing.

> [!WARNING]
> **Do not remove generic `/account`.** The TUI account manager is shared by retained provider accounts. Remove only the Jcode-specific account branch and Jcode commercial-account actions.

> [!WARNING]
> **Do not remove normal OpenRouter support.** Jcode Subscription currently reuses parts of the OpenRouter-compatible runtime. Remove the Jcode transport identity and subscription shim; preserve OpenRouter BYOK and generic OpenAI-compatible transports.

> [!WARNING]
> **Do not blanket-delete `JCODE_OPENROUTER_*` controls.** Some environment controls belong to the generic OpenRouter/OpenAI-compatible runtime. Delete only subscription-owned assignments, aliases, or state after proving they have no retained consumer.

> [!WARNING]
> **Do not remove Memory or Browser because they can use Jcode-backed JEV.** Remove `JevProvider::Jcode` and its entitlement/credential path while preserving retained JEV providers and the calling features.

> [!WARNING]
> **Do not remove the entire Voice stack from this artifact.** This task removes only subscription-backed transcription. Native Nari/audio/dictation/voice-intent removal is a separate feature-debloat decision.

> [!WARNING]
> **Do not remove Telemetry or Partner Discovery wholesale.** Jcode's public service terms classify Subscription API, Telemetry, and Partner Discovery as separate hosted services. This task may remove subscription-specific telemetry fields/events but does not authorize deleting the telemetry or discovery subsystems.

> [!WARNING]
> **Do not remove generic remote operation.** Jcode-managed Cloud activation is commercial subscription coupling; SSH, socket forwarding, WebSocket gateway, pairing, remote daemon/client operation, reconnect, and session persistence are independent capabilities.

> [!WARNING]
> **Do not silently reroute old Jcode Subscription sessions to a different paid provider.** Legacy route metadata must fail safely and visibly. A removed Jcode route must never silently spend a user's OpenRouter, OpenAI, Anthropic, or other BYOK balance.

> [!WARNING]
> **Do not hand-edit `Cargo.lock` to erase strings.** Let Cargo resolve legitimate dependency changes. A transitive or historical string is not evidence that active subscription functionality still exists.

> [!WARNING]
> **Do not rewrite changelogs or historical release notes merely to erase references to Jcode Subscription.** Historical facts are not active product coupling.

---

## 5. Definition of "Jcode Subscription" in the audited source

In the audited source, the first-party subscription is not one file. It spans several layers.

### 5.1 Credential and account identity

Primary concepts include:

```text
JCODE_API_KEY
JCODE_API_BASE
JCODE_ACCOUNT_ID
JCODE_ACCOUNT_EMAIL
JCODE_TIER
JCODE_SUBSCRIPTION_ACTIVE
jcode-subscription.env
```

The subscription catalog persists commercial-account credentials and account metadata and exposes a Jcode-specific provider/runtime identity.

### 5.2 Account and billing API

The typed client covers concepts including:

```text
device authorization
device-token polling
GET /v1/me
account status
subscription tier
usage/budget/billing information
manage/account URL
credential revocation
paid activation polling
optional capabilities
```

The typed capability structure in the audited source explicitly contains:

```text
voice_transcription
remote_compile
```

Other capability checks, such as `memory_jev` and `browser_jev`, are read dynamically by the JEV path and must not be falsely described as fields of that typed structure.

### 5.3 Hosted-model provider

The source represents Jcode Subscription as a model provider with:

```text
provider display: Jcode Subscription
route method: jcode-subscription
runtime identity: JcodeSubscription
curated model catalog
tier-gated model availability
server-managed routing
```

### 5.4 OpenRouter-compatible runtime reuse

The subscription route reuses parts of an OpenRouter-compatible HTTP/runtime slot but is explicitly distinguished from user-owned OpenRouter billing.

This is a **shared implementation detail**, not permission to delete OpenRouter.

### 5.5 Subscription-dependent secondary capabilities

The Jcode commercial account also gates or supplies credentials for:

```text
Remote Compile / cloud-compute credits
Jcode-managed Cloud activation
Jcode-backed JEV Decisions
subscription-backed WAV transcription
support diagnostic account identity
subscription-specific analytics events
```

Those tails are part of this artifact only to the extent described in later sections.

---

## 6. Target architecture after removal

### Before

```text
                              ┌──────────────────────┐
                              │ Jcode commercial     │
                              │ account / API key    │
                              └──────────┬───────────┘
                                         │
                              ┌──────────▼───────────┐
                              │ api.jcode.sh / v1    │
                              └──────────┬───────────┘
                                         │
          ┌──────────────────────────────┼───────────────────────────────┐
          │                              │                               │
┌─────────▼─────────┐        ┌───────────▼──────────┐        ┌───────────▼──────────┐
│ hosted model      │        │ Remote Compile /     │        │ JEV / voice / cloud │
│ router/catalog    │        │ cloud-compute credit │        │ account-backed tails│
└───────────────────┘        └──────────────────────┘        └──────────────────────┘
```

### After

```text
HAVK provider layer
├── provider OAuth
├── provider API keys
├── OpenRouter BYOK
├── OpenAI-compatible profiles
└── local providers

HAVK feature layer
├── Memory ──> retained JEV provider(s)
├── Browser ─> retained JEV provider(s)
├── Voice ───> retained non-Jcode path(s), if feature remains
├── Remote ──> self-hosted daemon/gateway/SSH/socket paths
└── Sandbox -> future provider-neutral Modal/E2B/etc. design

No Jcode account or Subscription API sits in the middle.
```

---

## 7. Scope matrix

| Area | Required action | Notes |
|---|---|---|
| Subscription catalog | **DELETE** | Dedicated first-party account/catalog state |
| Subscription API client | **DELETE** | Dedicated account/billing/entitlement client |
| Jcode account-login wrapper | **DELETE** | Dedicated subscription login lifecycle |
| Jcode hosted-model provider | **DELETE** | Dedicated subscription provider |
| Jcode device-login CLI module | **DELETE** | Dedicated subscription auth flow |
| Jcode commercial `account` CLI | **DELETE** | Do not confuse with generic TUI `/account` |
| Subscription TUI nudge | **DELETE** | Dedicated product upsell/onboarding |
| Remote Compile | **DELETE** | Entire feature is subscription/cloud-credit backed |
| Provider metadata | **EDIT** | Remove Jcode login/provider descriptor only |
| Subscription model gating | **EDIT** | Remove active Jcode tier/catalog enforcement from shared model-selection code |
| Provider runtime identities | **EDIT** | Remove/inert legacy Jcode route safely |
| Remote model-picker synthesis | **EDIT** | Remove Jcode route injection and subscription-poison repair logic only |
| OpenRouter runtime | **EDIT** | Remove JcodeSubscription transport state only |
| Generic auth framework | **EDIT** | Remove Jcode target/branches only |
| Global `AuthStatus` | **EDIT** | Remove the dedicated Jcode auth slot/probe/assessment branches only |
| TUI auth/account | **EDIT** | Remove Jcode-specific commands and state only |
| `/remote` | **EDIT** | Remove managed Jcode Cloud branch; preserve generic remote |
| JEV | **EDIT** | Remove Jcode provider only |
| Voice | **EDIT** | Remove subscription-backed route only |
| Support diagnostics | **EDIT** | Remove Jcode commercial identity only |
| Harness API credential binding | **EDIT** | Remove Jcode credential-file mapping only |
| Rust SDK auth surface | **EDIT** | Remove Jcode Subscription from `AuthClient` provider/login exposure only |
| TypeScript SDK docs/comments | **EDIT** | Remove subscription-specific claims; preserve generic credential inheritance |
| CI quality-ratchet baselines | **REVIEW / RATCHET** | Remove stale subscription-file baseline entries only after the code removal is correct |
| Telemetry | **EDIT IF PRESENT** | Remove subscription-specific events/fields only |
| Browser | **PRESERVE** | Rewrite Jcode-JEV docs/tests to retained routes |
| Memory | **PRESERVE** | Rewrite Jcode-JEV docs/tests to retained routes |
| Third-party OAuth | **PRESERVE** | Not Jcode Subscription |
| OpenRouter BYOK | **PRESERVE** | Shared runtime must survive |
| Generic `Subscribe` protocol | **PRESERVE** | Unrelated session/client operation |
| Self-Dev | **OUT OF SCOPE** | Separate debloat |
| Partner Discovery | **OUT OF SCOPE** | Separate hosted service/debloat |
| Full telemetry removal | **OUT OF SCOPE** | Separate debloat |
| Full Voice removal | **OUT OF SCOPE** | Separate debloat |
| Grok Build | **OUT OF SCOPE** | Separate xAI runtime/subscription decision |

---

## 8. High-confidence deletion set

The following files are dedicated enough that they are expected to be deleted after consumers are removed.

### 8.1 Core subscription/account

```text
crates/jcode-base/src/subscription_catalog.rs
crates/jcode-base/src/subscription_api.rs
crates/jcode-base/src/account_login.rs
crates/jcode-base/src/account_login/tests.rs
crates/jcode-base/src/provider/jcode.rs
crates/jcode-base/src/provider/tests/catalog_subscription.rs
src/cli/login/jcode_device.rs
src/cli/login/jcode_device/tests.rs
src/cli/account.rs
crates/jcode-tui/src/tui/app/subscribe_nudge.rs
```

### 8.2 Remote Compile

```text
crates/jcode-app-core/src/tool/compile_remote.rs
crates/jcode-app-core/src/tool/compile_remote/source.rs
crates/jcode-app-core/src/tool/compile_remote/tests.rs
crates/jcode-app-core/src/agent_tests/compile_remote.rs
docs/REMOTE_COMPILE.md
docs/REMOTE_COMPILE_VALIDATION.md
```

### 8.3 Subscription-only design/development documentation

Review current contents first. In the audited source these documents are dedicated to the Jcode commercial account or Jcode Cloud contract and are expected to be removed rather than rewritten as HAVK behavior:

```text
docs/dev/ACCOUNT_CONTRACT_CONFORMANCE_TESTS.md
docs/dev/ACCOUNT_FLOWS_OBSERVABILITY_PRIVACY.md
docs/JCODE_CLOUD_AWS.md
```

Do not delete a document solely by filename if current `HEAD` has since been repurposed for generic provider-account behavior.

---

## 9. Core subscription module removal

### 9.1 Remove module exports

Known module wiring includes:

```text
crates/jcode-base/src/lib.rs
```

Remove exports for dedicated subscription modules after their consumers are removed:

```text
account_login
subscription_api
subscription_catalog
```

Do not remove generic auth/account-store modules.

### 9.2 Remove subscription constants and persistence

Eliminate active handling for:

```text
JCODE_API_KEY
JCODE_API_BASE
JCODE_ACCOUNT_ID
JCODE_ACCOUNT_EMAIL
JCODE_TIER
JCODE_SUBSCRIPTION_ACTIVE
jcode-subscription.env
jcode-subscription cache/state namespace
Jcode pricing/account URLs used only for the commercial account
```

#### Safety rule

A residue string may remain in:

```text
historical changelog
migration history
compatibility test fixture
explicit legacy decoder
```

if it has a documented purpose and cannot activate a network/account route.

### 9.3 Remove tier and curated hosted-model policy

Delete Jcode-specific concepts such as:

```text
JcodeTier
curated hosted model list
minimum subscription tier per model
cached Jcode tier
default Jcode subscription model
server-managed Jcode route policy
Jcode subscription catalog augmentation
```

Do not remove generic provider model catalogs or model metadata shared by retained providers.

### 9.4 Remove subscription-only model gating from shared model selection

Cross-check against the audited implementation confirms an active shared-model hook in:

```text
crates/jcode-base/src/provider/models.rs
crates/jcode-base/src/provider/mod.rs
crates/jcode-base/src/provider/jcode.rs
```

The production helper:

```text
ensure_model_allowed_for_subscription(...)
```

is called by both the dedicated Jcode provider and the shared `set_model_on_jcode_subscription(...)` route. Its only purpose is to enforce the curated Jcode Subscription catalog and `JcodeTier` entitlement while subscription runtime mode is active.

After the Jcode provider and `JcodeSubscription` runtime are removed, remove:

```text
ensure_model_allowed_for_subscription(...)
its calls from the removed Jcode provider/runtime path
subscription-only error messages about catalog membership/tier upgrades
```

Do **not** generalize this function into a new HAVK paywall or tier system.

#### Important correction from source cross-check

The similarly named:

```text
filtered_display_models(...)
```

is `#[cfg(test)]` in the audited source and is referenced only by subscription catalog tests. It is **not** a production model-picker filter.

Therefore:

- remove it together with subscription-only tests if it becomes unused;
- do not describe it as an active production gating mechanism;
- do not remove unrelated generic model catalog/filtering logic by association.

---

## 10. Provider and routing cleanup

This is one of the highest-risk shared areas. The Jcode subscription provider must disappear without damaging retained providers.

### 10.1 Provider metadata

Known hotspot:

```text
crates/jcode-provider-metadata/src/catalog.rs
crates/jcode-provider-metadata/src/lib.rs
```

Remove the Jcode subscription login descriptor and associated target/state variants, including concepts equivalent to:

```text
JCODE_LOGIN_PROVIDER
LoginProviderTarget::Jcode
LoginProviderAuthStateKey::Jcode
aliases: jcode / subscription / jcode-subscription
menu entry: Jcode Subscription
```

Adjust static arrays/counts and tests accordingly.

Preserve descriptors for all retained providers.

### 10.2 Provider implementation selection

Known hotspot:

```text
src/cli/provider_init.rs
src/cli/provider_init_tests.rs
src/cli/commands/report_info.rs
```

Remove:

```text
ProviderChoice::Jcode
construction of provider::jcode::JcodeProvider
choice mapping for JCODE_LOGIN_PROVIDER
subscription runtime activation/clearing code that exists only for that provider
Jcode-specific provider-init tests
```

#### Important

The source currently clears subscription runtime state when switching to other providers. Once the subscription runtime is fully removed, remove that cleanup scaffolding only after proving no retained path needs it.

Do not alter provider-selection behavior for retained providers as incidental cleanup.

### 10.3 Model route identity

Known hotspots:

```text
crates/jcode-provider-core/src/lib.rs
crates/jcode-base/src/provider/selection.rs
crates/jcode-base/src/provider/activation.rs
crates/jcode-base/src/provider/catalog_routes.rs
crates/jcode-base/src/provider/mod.rs
crates/jcode-protocol/src/protocol_tests/misc_events.rs
```

Active generation/use of identities equivalent to these must end:

```text
RuntimeKey::JcodeSubscription
ModelRouteApiMethod::JcodeSubscription
"jcode-subscription"
"Jcode Subscription"
```

But see the legacy compatibility requirement in Section 19 before removing deserialization support blindly.

### 10.4 OpenRouter-compatible runtime

Known hotspots:

```text
crates/jcode-provider-openrouter-runtime/src/lib.rs
crates/jcode-provider-openrouter-runtime/src/openrouter_tests.rs
crates/jcode-base/src/provider/openrouter.rs
```

Remove only the Jcode-specific transport identity/branches, such as:

```text
OpenRouterTransportState::JcodeSubscription
runtime-provider alias "jcode"
transport-state aliases "jcode-subscription" / "subscription"
Jcode subscription display-name overrides
Jcode subscription route-method overrides
special checks against Jcode API base/key
subscription-only tests
```

#### Must survive

```text
OpenRouterApiKey
DirectApiKey
DirectNoAuth
normal OpenRouter endpoint/auth
OpenAI-compatible named provider profiles
local/no-auth compatible endpoints
```

### 10.5 Remove Jcode Subscription synthesis from the remote model picker

Cross-check against the audited TUI implementation confirms that remote model hydration has dedicated Jcode Subscription logic in:

```text
crates/jcode-tui/src/tui/app/inline_interactive.rs
crates/jcode-tui/src/tui/app/inline_interactive_placeholder_routes.rs
```

High-confidence subscription-owned logic includes concepts equivalent to:

```text
append_jcode_subscription_routes_static(...)
provider_is_jcode_subscription
poisoned_by_jcode_subscription
curated Jcode route injection based on cached tier/credentials
Jcode subscription route-method/display-name checks
```

The `poisoned_by_jcode_subscription` branch exists to repair an older Jcode-specific failure mode in which a mixed remote catalog could become an all-Jcode route set. Once active Jcode Subscription routes no longer exist, this repair machinery has no active product purpose.

Remove active Jcode route synthesis and Jcode-specific repair branches.

Preserve:

```text
remote model catalog hydration
remote_model_routes_fallback(...)
non-Jcode provider route reconstruction
remote provider/model discovery
generic route de-duplication
remote session/model picker behavior
```

The placeholder-route helper also contains a `ModelRouteApiMethod::JcodeSubscription` branch. Remove that active branch when the enum/route is removed, or keep only the minimum inert decode compatibility required by Section 19. A legacy value must never create a new active Jcode Subscription route.

Known subscription-specific TUI tests include:

```text
test_remote_jcode_subscription_catalog_is_not_augmented_with_local_auth_routes
test_remote_mixed_catalog_keeps_jcode_subscription_separate_from_other_providers
test_remote_hydrated_catalog_adds_entitled_jcode_subscription_routes
test_remote_non_jcode_catalog_repairs_poisoned_all_jcode_routes
```

Delete or rewrite these according to whether the test subject is the removed subscription behavior or a generic remote-catalog invariant.

---

## 11. Authentication and account UX cleanup

### 11.1 CLI provider login

Known hotspots:

```text
src/cli/login.rs
src/cli/login/jcode_device.rs
src/cli/auth_test/probes.rs
crates/jcode-base/src/auth/mod.rs
crates/jcode-base/src/auth/integration.rs
crates/jcode-base/src/auth/lifecycle.rs
crates/jcode-provider-doctor/src/provider_e2e.rs
```

Remove Jcode-specific login target handling, credential probes, route expectations, status, lifecycle, and provider-doctor logic.

Preserve the generic provider login framework.

### 11.2 Dedicated commercial account CLI

The audited source contains a top-level CLI surface for the first-party Jcode account, with actions such as:

```text
jcode account login
jcode account status
jcode account manage
jcode account logout
```

Known hotspots:

```text
src/cli/account.rs
src/cli/args.rs
src/cli/dispatch.rs
src/cli/mod.rs
src/cli/proctitle.rs
src/cli/args/tests.rs
```

Remove this dedicated Jcode-commercial-account command and its dispatch/tests.

#### Do not confuse with

```text
TUI /account
jcode login --provider <retained-provider>
provider account switching
provider OAuth stores
```

The generic retained-provider account UX must remain.

### 11.3 TUI Jcode account actions

Known hotspots:

```text
crates/jcode-tui/src/tui/app/auth.rs
crates/jcode-tui/src/tui/app/auth_account_commands.rs
crates/jcode-tui/src/tui/app/auth_tests.rs
```

Remove branches for:

```text
/account jcode login
/account jcode status
/account jcode manage
/account jcode logout
Jcode device login
Jcode paid activation polling
Jcode credential revoke/clear
Jcode subscription account status cards
```

Preserve shared overlays, account picker, provider login, logout, and retained-provider account switching.

### 11.4 Remove Jcode as a first-class slot in global `AuthStatus`

Cross-check against the audited auth implementation confirms that Jcode Subscription is represented directly in the global authentication snapshot:

```text
crates/jcode-base/src/auth/status_types.rs
crates/jcode-base/src/auth/mod.rs
crates/jcode-base/src/auth/tests.rs
```

The active surface includes concepts equivalent to:

```text
AuthStatus::jcode
probe_jcode_status(...)
"jcode" auth probe timing
"jcode" auth_status_snapshot field
Jcode contribution to has_any_available()
LoginProviderAuthStateKey::Jcode -> status.jcode
LoginProviderTarget::Jcode state/credential assessment
JCODE_API_KEY / jcode-subscription.env credential-source reporting
```

Remove these Jcode-specific fields and branches after the provider metadata target is removed.

Preserve the `AuthStatus` framework and all retained-provider fields/probes.

Because `AuthStatus` is serializable, verify any real persistence/protocol consumers before changing its serialized shape. Do not keep an **active** Jcode credential probe merely for compatibility; if compatibility is actually required, handle old serialized data with the narrowest inert/defaulted migration behavior and add a focused test.

Update auth tests surgically. For example, a cache-expiry test that currently uses `status.jcode` merely as a convenient field should be rewritten to use a retained auth field rather than deleted if the invariant is generic.

---

## 12. Remove hosted-model subscription commands and nudges

Known hotspots:

```text
crates/jcode-tui/src/tui/app/auth_account_commands.rs
crates/jcode-tui/src/tui/app/state_ui_input_helpers.rs
crates/jcode-tui/src/tui/ui_overlays.rs
crates/jcode-tui/src/tui/app/onboarding_flow.rs
crates/jcode-tui/src/tui/app/onboarding_flow_control.rs
crates/jcode-tui/src/tui/ui_onboarding.rs
crates/jcode-tui/src/tui/mod.rs
crates/jcode-tui/src/tui/app/subscribe_nudge.rs
```

Remove active commands/UI equivalent to:

```text
/hosted
/hosted status
/subscribe
/subscription
/subscription status
subscription pricing pitch
subscription activation CTA
subscription onboarding pill
subscription nudge state
```

Remove module/state wiring for `subscribe_nudge` after all references are removed.

Update command registries, completions, help, overlays, onboarding snapshots, and golden tests.

### Preserve

```text
/model
/login
/account
/usage
/info
provider onboarding for retained providers
```

---

## 13. Remove Jcode Remote Compile completely

### 13.1 Why full deletion is required

The audited tool identifies itself as **subscription-backed remote compilation**.

Its active flow depends on the Jcode commercial account for:

```text
JCODE_API_KEY
Jcode API base
GET /v1/me
remote_compile capability
cloud_compute entitlement
compute usage / credits
Jcode account login/pricing guidance
remote compile request submission
```

The source snapshotter is a feature implementation detail of this product, not a provider-neutral cloud-sandbox abstraction.

Therefore the correct action is full deletion rather than a disabled stub.

### 13.2 Delete implementation and docs

Delete the files listed in Section 8.2.

### 13.3 Remove tool registry and dynamic entitlement refresh

Known hotspots:

```text
crates/jcode-app-core/src/tool/mod.rs
crates/jcode-app-core/src/agent/turn_execution.rs
crates/jcode-app-core/src/agent_tests.rs
```

Remove:

```text
mod compile_remote
built-in compile_remote registration
remote_compile_definition()
compile_remote::refresh_access()
special locked-tool refresh for compile_remote
subscription-aware tool schema/guidance refresh
agent test module wiring
```

### 13.4 Preserve generic compilation and remote execution primitives

Do **not** remove:

```text
shell tool
local cargo/gcc/clang execution
generic background process execution
remote daemon/client operation
future provider-neutral sandbox integration
file transfer primitives used elsewhere
```

### 13.5 Dummy test-name trap

If a generic tool-schema test merely uses the text `compile_remote` as an arbitrary custom-tool name, do not delete the generic test. Rename the fixture to a neutral dummy name if necessary.

---

## 14. Remove Jcode-managed Cloud activation only

The audited source distinguishes Jcode-managed Cloud activation from generic self-hosted remote controls.

Known hotspots:

```text
crates/jcode-base/src/gateway/control.rs
crates/jcode-tui/src/tui/app/commands_remote.rs
docs/JCODE_CLOUD_AWS.md
```

### Remove

```text
RemoteCommand::Cloud
/remote cloud
/remote setup
managed Jcode Cloud activation URL
Jcode subscription credential/tier check in cloud activation
Jcode Cloud account-opening behavior
Jcode Cloud commercial documentation
```

### Bare `/remote`

After removing the commercial default, bare `/remote` must not open a deleted account page.

Preferred retained behavior:

```text
/remote -> show local remote status/help
```

or another non-commercial generic remote help surface consistent with current command architecture.

### Preserve

```text
/remote status
/remote on
/remote off
/remote pair
/remote revoke <device>
WebSocket gateway
paired-device registry
SSH/socket forwarding
remote daemon/client protocol
session persistence
reconnect
```

---

## 15. JEV: remove the Jcode route, not JEV

The audited JEV subsystem supports several provider routes. Jcode Subscription is one route, not the abstraction itself.

Known hotspot:

```text
crates/jcode-base/src/jev.rs
```

Related consumers/docs include:

```text
crates/jcode-app-core/src/tool/browser_jev.rs
crates/jcode-app-core/src/tool/browser_fast_live_tests.rs
crates/jcode-app-core/src/tool/memory.rs
docs/BROWSER_FAST_AGENT.md
docs/MEMORY_ARCHITECTURE.md
```

### Remove

```text
JevProvider::Jcode
JCODE_API_KEY lookup for JEV
jcode-subscription.env lookup for JEV
Jcode gateway /decisions endpoint construction
Jcode /v1/me capability preflight
memory_jev Jcode entitlement check
browser_jev Jcode entitlement check
selector aliases: jcode / subscription / jcode-subscription
Jcode-specific request constraints/tests
subscription-first auto-selection behavior
```

### Preserve

Retained provider routes in current source, including the applicable combination of:

```text
OpenRouter
TypeSafe
AI/ML API
```

Preserve:

```text
JEV request/response validation
Memory relevance selection
Browser handoff decision logic
provider-specific safety bounds
no-silent-fallback semantics
```

### Auto-selection after removal

Update `auto` behavior so it considers only retained JEV providers.

Do not invent a new paid default.

Do not silently move a request from the removed Jcode route to another provider after an auth/network/billing failure. Explicit provider selection must remain predictable.

---

## 16. Voice: remove only subscription-backed transcription

Known hotspot:

```text
crates/jcode-base/src/voice.rs
```

The file contains both non-Jcode native voice functionality and a Jcode subscription-backed WAV transcription path.

### Remove from this artifact

```text
subscription_api import/use for voice
subscription_catalog import/use for voice
subscription_voice_available()
JCODE_API_KEY voice credential path
Jcode /v1/me voice_transcription entitlement check
Jcode /audio/transcriptions upload path
Jcode subscription voice errors/tests
```

### Preserve unless separately debloated

```text
Nari/provider-key path
microphone capture
resampling/audio utilities
voice intent
external dictation command
generic TUI voice UX
```

If another approved HAVK artifact has already removed native voice entirely, treat this section as satisfied by that removal. Do not reintroduce voice merely to satisfy this artifact.

---

## 17. Remove Jcode subscription identity from diagnostics

Known hotspot:

```text
crates/jcode-tui/src/tui/app/support.rs
```

The audited support diagnostics conditionally collect subscription-specific identity:

```text
account_id
account_email
tier
```

from Jcode subscription state.

Remove the Jcode commercial-account lookup and any fields that exist solely for that identity.

### Preserve

```text
version
build information
OS/architecture
provider/model
last error
telemetry identifier if telemetry itself remains
other generic support diagnostics
```

Do not remove `/support` as a feature merely because one payload section belonged to Jcode Subscription.

---

## 18. Harness API / SDK credential bridge cleanup

Known hotspot:

```text
crates/jcode-harness-api-server/src/translate.rs
crates/jcode-harness-api-server/src/translate_tests.rs
```

Remove mappings that interpret provider identifiers such as:

```text
jcode
subscription
jcode-subscription
```

as:

```text
JCODE_API_KEY
jcode-subscription.env
```

### Test-preservation rule

If a test is fundamentally testing generic credential-file security, ownership, symlink handling, or API-key translation but happens to use Jcode as its fixture provider, **rewrite the fixture using a retained provider** rather than deleting the generic security test.

### 18.1 Remove Jcode Subscription from the Rust SDK auth surface

Cross-check against:

```text
crates/jcode-sdk/src/auth.rs
docs/DESKTOP_AUTH_SDK.md
```

confirms that the Rust SDK currently exposes Jcode Subscription as a supported API-key login target. In the audited source, `LoginProviderTarget::Jcode` maps to `LoginMethod::ApiKey`, and the source documentation explicitly says that Jcode Subscription supports API-key entry through this SDK surface.

After removing the Jcode login provider descriptor/target:

- remove the `Jcode` match arm from SDK login-method resolution;
- ensure `AuthClient::providers()` no longer returns Jcode Subscription;
- remove Jcode-specific SDK auth tests if present;
- rewrite `docs/DESKTOP_AUTH_SDK.md` so the supported method list contains only retained providers.

Preserve the Rust SDK auth framework, `AuthClient`, generic API-key/OAuth/device-code flows, and retained-provider support.

### 18.2 TypeScript SDK: remove subscription-specific documentation, preserve generic env-file inheritance

Cross-check against:

```text
sdk/typescript/src/launch.ts
sdk/typescript/test/launch.test.ts
sdk/typescript/README.md
crates/jcode-base/src/subscription_catalog.rs
```

shows two distinct concerns:

1. the TypeScript SDK has **generic** logic that inherits `*.env` provider files into a private launched instance; and
2. comments/README text specifically advertise a Jcode Subscription credential.

The generic inheritance is not owned by Jcode Subscription and must remain.

In particular, the audited TypeScript test deliberately uses an arbitrary file named:

```text
n.env
```

to verify that only env files are linked and that credential directories are not symlinked wholesale. That test is generic security/shape coverage; do **not** delete it merely because the adjacent source comment calls `n.env` a Jcode subscription example.

The actual Jcode Subscription env filename in the Rust source is:

```text
jcode-subscription.env
```

Therefore:

- rewrite the subscription-specific comment in `launch.ts` to describe generic provider env-file inheritance accurately;
- remove the README claim that API-key provisioning supports the Jcode Subscription key;
- preserve `inheritCredentials(...)`, generic `*.env` linking, owner/symlink safety checks, and their tests.

Do not replace the generic `n.env` test fixture with a real Jcode subscription filename unless there is an independent testing reason to do so.

---

## 19. Legacy session and route compatibility

This section is mandatory because route identities can cross persistence and protocol boundaries.

Known subscription route identities include concepts equivalent to:

```text
RuntimeKey::JcodeSubscription
ModelRouteApiMethod::JcodeSubscription
provider_key = "jcode-subscription"
route_api_method = "jcode-subscription"
```

Some of these values can appear in:

```text
serialized protocol messages
saved session route metadata
old test fixtures
resume/session history
```

### 19.1 Required behavior for legacy data

A session created before the debloat that references Jcode Subscription should, where practical:

1. remain readable;
2. clearly report that its former provider route is unavailable;
3. require explicit user/provider reselection before the next model request;
4. make **no request** to the Jcode Subscription API;
5. never silently route to another paid provider.

### 19.2 Implementation choices

The agent may choose the smallest safe mechanism supported by the current architecture, for example:

```text
A. remove active enum variant and add legacy string decoding to an unavailable route;
B. preserve a deprecated/inert decode-only enum variant temporarily;
C. migrate persisted route metadata on load to an explicit unavailable/unknown state.
```

The artifact does not mandate one mechanism because the current serialization shape must be inspected first.

### 19.3 Forbidden compatibility behavior

Do not keep functional Jcode credentials, catalog, routing, or network clients solely to make old sessions work.

Compatibility may preserve **readability**, not the removed service.

---

## 20. Telemetry residue: remove subscription-specific semantics only

Telemetry is a separate hosted service and is not deleted by this artifact.

If the telemetry subsystem remains in HAVK, remove active subscription-only analytics concepts such as applicable producers/schema/query/docs for:

```text
subscription_login
subscription_activated
subscription_budget_exhausted
subscription_router_error
account_linked
Jcode subscription plan/tier funnel fields
Jcode hosted-model subscription conversion data
```

Known areas to inspect include:

```text
crates/jcode-telemetry-core/
telemetry-worker/
```

### Migration safety

Do not delete historical database migrations merely to remove a word unless the telemetry database history is intentionally being reset by a separate approved change.

An old migration can remain as historical schema history if:

```text
it is not an active producer;
it does not cause HAVK to contact the Subscription API;
and deleting it would damage migration ordering/upgrade safety.
```

If telemetry is separately removed in another HAVK change, this section is automatically satisfied.

---

## 21. Documentation cleanup

### 21.1 Delete subscription-only docs

Use the high-confidence list in Section 8.

### 21.2 Rewrite shared docs

Known shared documents requiring surgical edits include:

```text
docs/BROWSER_FAST_AGENT.md
docs/MEMORY_ARCHITECTURE.md
docs/DESKTOP_AUTH_SDK.md
sdk/typescript/README.md
```

The in-tree source documentation was re-read during cross-check:

- `docs/MEMORY_ARCHITECTURE.md` explicitly documents `auto` as preferring Jcode, then OpenRouter, TypeSafe, and AI/ML API, plus live `/v1/me` `memory_jev` entitlement checks.
- `docs/BROWSER_FAST_AGENT.md` explicitly documents Jcode account login, `browser_jev` entitlement, `/v1/decisions`, and `JCODE_BROWSER_JEV_PROVIDER=jcode`.
- `docs/DESKTOP_AUTH_SDK.md` explicitly documents Jcode Subscription API-key entry through the Rust SDK.
- `docs/REMOTE_COMPILE.md` explicitly describes Remote Compile as subscription-backed and tied to Jcode cloud-compute access.
- `docs/JCODE_CLOUD_AWS.md` explicitly says managed Cloud access is bundled into paid Jcode subscriptions while self-hosted `/remote status/on/pair/revoke` remains separate.
- `docs/dev/ACCOUNT_CONTRACT_CONFORMANCE_TESTS.md` and `docs/dev/ACCOUNT_FLOWS_OBSERVABILITY_PRIVACY.md` are dedicated to the Jcode subscription account contract and remain high-confidence deletion candidates listed in Section 8.

These files are source-grounding for the implementation boundary; do not infer a broader deletion from similarly named unrelated documentation.

Also re-scan all docs for active instructions such as:

```text
jcode account login
Jcode Subscription
jcode-subscription
JCODE_API_KEY
Jcode pricing/account URL
api.jcode.sh subscription routes
Jcode Cloud subscription activation
subscription-first JEV selection
```

Rewrite surviving feature docs to describe retained providers only.

### 21.3 Historical documents

Do not rewrite historical release notes/changelogs solely for branding cleanliness.

If a historical document is presented as current operational guidance rather than history, update or archive it so an agent/user cannot follow a dead Jcode account path.

---

## 22. Tests: delete owned tests, preserve generic invariants

### 22.1 Delete tests whose subject is the removed product

Examples include tests for:

```text
subscription catalog tiering
Jcode device login
Jcode account API parsing
Jcode hosted provider routing
Jcode subscription transport state
Remote Compile entitlement/source upload
Jcode Cloud activation
Jcode JEV entitlement
Jcode subscription voice backend
subscription commands/nudges
```

### 22.2 Rewrite generic tests that use Jcode only as a fixture

Examples of generic behaviors that should survive:

```text
custom tool schema locking
credential-file security
provider route serialization
account picker mechanics
protocol event serialization
OpenRouter transport behavior
Browser handoff validation
Memory decision validation
```

Replace the fixture/provider name with a retained neutral provider when necessary.

### 22.3 Cross-checked subscription test hotspots

Additional verified test hotspots include:

```text
crates/jcode-tui/src/tui/app/tests/remote_startup_input_02/part_01.rs
crates/jcode-tui/src/tui/app/tests/onboarding_flow.rs
crates/jcode-tui/src/tui/app/tests/onboarding_golden.rs
crates/jcode-tui/src/tui/app/tests/commands_accounts_01/part_02.rs
crates/jcode-base/src/auth/tests.rs
crates/jcode-base/src/memory_agent_tests.rs
src/cli/commands_tests.rs
```

Classification rules:

- tests whose subject is Jcode tiering, Jcode route synthesis, Jcode account login, or Jcode entitlement should be deleted with the feature;
- tests of generic auth-cache behavior, env isolation, memory/Browser semantics, or CLI behavior should be rewritten using retained providers/fields when Jcode was merely the fixture;
- `memory_agent_tests.rs` contains an end-to-end loopback test that intentionally selects the Jcode JEV route; rewrite that coverage to a retained JEV route if the same generic recall invariant is still valuable;
- `src/cli/commands_tests.rs` includes `JCODE_API_KEY` in a credential-clearing test setup; remove the dead key from the fixture without deleting the generic memory CLI test.

### 22.4 Golden/snapshot updates

Update TUI onboarding/command snapshots only for intended changes.

Do not regenerate broad snapshot sets without reviewing the diff.

---

## 23. Dependency and Cargo cleanup

### 23.1 Do not assume a dedicated dependency disappears

The subscription and Remote Compile code uses common crates such as:

```text
reqwest
serde
serde_json
sha2
tokio
anyhow
```

These are heavily shared. Their presence after the debloat is expected.

### 23.2 Remove only dependencies proven unused

After code deletion:

1. inspect the affected crate manifests;
2. use the repository's unused-dependency guard (`cargo machete`) if available;
3. remove only direct dependencies proven unused by the affected crate;
4. do not remove workspace dependencies still used elsewhere;
5. do not edit `Cargo.lock` manually.

The repository CI includes an unused-dependency check, so manifest cleanup must be part of completion when removal makes dependencies unused.

### 23.3 Lockfile rule

Use Cargo tooling to update/validate the lockfile if manifest changes require it.

Do not run a broad dependency update as part of this debloat.

The PR should not contain unrelated version churn.

### 23.4 Quality-ratchet baseline cleanup

The audited repository currently tracks:

```text
crates/jcode-base/src/subscription_api.rs
crates/jcode-base/src/subscription_catalog.rs
```

inside:

```text
scripts/swallowed_error_budget.json
```

Cross-check of `scripts/check_swallowed_error_budget.py` confirms that removing a tracked source file is treated as an **improvement**, not a regression:

```text
swallowed-error-like usage removed: <path> (... -> 0)
```

Therefore stale entries are **not a reason to keep subscription files** and do not need to be edited preemptively just to make the checker pass.

After the code removal is correct and unrelated regressions are absent, review whether the repository expects the ratchet baseline to be refreshed. If an intentional baseline refresh is performed, do it only after the final code shape is known and inspect the diff carefully.

Do not use a blanket baseline update to hide new swallowed-error usage elsewhere.

---

## 24. Network endpoint acceptance boundary

After the change, HAVK must have no **active** Subscription-owned code path that performs Jcode commercial account/model/entitlement requests.

The agent must classify active references to:

```text
api.jcode.sh/v1
jcode.sh/account
jcode.sh/pricing
```

A reference is unacceptable when it exists to:

```text
log into the Jcode commercial account
fetch Jcode subscription state
route hosted model inference
check Jcode subscription entitlements
submit Remote Compile jobs
activate Jcode managed Cloud
call Jcode-backed JEV
upload Jcode subscription-backed voice transcription
```

A reference may be acceptable when it belongs to a **separate retained service** outside this artifact, such as Partner Discovery, if that feature has not been independently removed.

Therefore **do not assert that every `api.jcode.sh` string must disappear**. Classify by endpoint and ownership.

---

## 25. False-positive catalog — explicitly preserve

The following are common traps.

### 25.1 Protocol subscription

```text
Request::Subscribe
wire type "subscribe"
client event subscription
swarm channel subscribe
```

**Preserve.**

### 25.2 Third-party subscriptions

```text
Claude Pro/Max OAuth
ChatGPT/OpenAI/Codex OAuth
GitHub Copilot subscription
Grok Build subscription
```

**Preserve unless another artifact explicitly removes them.**

### 25.3 Generic account management

```text
/account provider switching
account picker
generic auth state
provider credential stores
```

**Preserve.**

### 25.4 Generic usage

```text
/usage
token accounting
provider quota display
model cost estimates
usage types
```

**Preserve.**

### 25.5 Jcode-managed OAuth wording

Some code may call locally stored/imported provider OAuth credentials "Jcode-managed". That does not automatically mean the Solo Systems Subscription API.

**Classify before editing.**

### 25.6 Generic remote/cloud capability

```text
remote daemon
remote working directory
SSH
socket forwarding
WebSocket gateway
future sandbox provider
```

**Preserve.**

### 25.7 Generic TypeScript credential inheritance

The TypeScript SDK links provider `*.env` files generically for private launched instances.

**Preserve the mechanism and its security tests.** Remove only Jcode Subscription-specific documentation/examples.

### 25.8 Historical provider-activity display labels

`crates/jcode-base/src/provider_activity.rs` contains a display mapping equivalent to:

```text
"jcode" -> "Jcode subscription"
```

This can describe historical local usage records in `provider_activity.json`; the mapping itself does not activate Jcode Subscription or perform network access.

Do not delete historical local activity solely for branding cleanliness. Prefer either:

- retaining an inert legacy label, optionally clarified as `Legacy Jcode subscription`; or
- performing an explicit, tested migration if there is a strong reason to remove it.

The default for this artifact is **preserve inert historical readability**.

### 25.9 Generic `/usage`

`crates/jcode-base/src/usage.rs` contains wording mentioning Jcode Subscription among providers that may have local usage activity.

The `/usage` subsystem is generic and serves retained providers. **Do not remove it.** Remove only active Jcode-specific report branches if any survive after provider deletion, and update stale comments where useful.

---

## 26. Recommended implementation sequence

The order below reduces compile fallout and makes reviews easier.

### Stage A — inventory and baseline

1. Read repository instructions.
2. Record the current branch/working tree without modifying unrelated files.
3. Run the static residue searches in Section 28.
4. Identify all current consumers of the dedicated subscription modules.
5. Confirm whether another HAVK debloat already removed Voice, Telemetry, Self-Dev, or other shared code.

### Stage B — remove user entry points

1. Remove Jcode login-provider descriptor.
2. Remove Jcode provider selection option.
3. Remove dedicated commercial `account` CLI.
4. Remove Jcode device-login flow.
5. Remove TUI `/hosted`, `/subscribe`, `/subscription` surfaces.
6. Remove `/account jcode ...` actions.
7. Remove subscription nudge/onboarding UI.

This prevents new user state from entering the removed path while deeper cleanup proceeds.

### Stage C — remove hosted-model provider core

1. Remove `JcodeProvider`.
2. Remove curated subscription catalog.
3. Remove subscription API/account client.
4. Remove Jcode runtime/route generation.
5. Remove Jcode OpenRouter-compatible transport state.
6. Update provider metadata/auth lifecycle/doctor logic.

### Stage D — remove dependent products/tails

1. Delete Remote Compile.
2. Remove Jcode-managed Cloud activation.
3. Remove Jcode JEV provider route.
4. Remove subscription-backed voice route.
5. Remove support diagnostic identity.
6. Remove harness API Jcode credential mapping.
7. Remove subscription-specific telemetry events if telemetry remains.

### Stage E — compatibility and persistence

1. Inspect saved route/session serialization.
2. Add the minimum safe legacy handling for removed Jcode route IDs.
3. Ensure old sessions do not silently reroute.
4. Add/update tests for safe unavailable-route behavior.

### Stage F — docs/tests/dependencies

1. Delete dedicated tests/docs.
2. Rewrite generic tests using subscription fixtures.
3. Rewrite shared current docs.
4. Remove newly unused direct dependencies.
5. Update lockfile only as required.

### Stage G — residue audit

Re-run all searches and classify every remaining match.

Do not stop at "it compiles" if active Jcode subscription semantics still remain.

---

## 27. Validation policy

### 27.1 Static/light local validation

Use low-cost validation first:

```bash
cargo fmt --all -- --check
cargo metadata --locked --format-version 1 > /dev/null
cargo machete
```

`cargo machete` is conditional on the repository/tooling having it installed; CI already enforces the unused-dependency guard.

Run focused source searches from Section 28.

### 27.2 Targeted tests when practical

If the local environment can run targeted tests safely, prefer affected crates/modules rather than a full workspace build.

Potential targets include the retained/shared areas touched by the patch, for example:

```text
provider core
provider metadata
OpenRouter runtime
base auth/provider/JEV tests
TUI account/command tests
protocol route tests
harness API translation tests
```

Exact commands must follow current repository instructions and crate names.

### 27.3 Heavy validation

Do not claim the work is fully validated solely from static inspection.

For the HAVK workflow, full compilation, broad test matrices, Clippy, and platform CI should be validated by the configured remote/GitHub CI when local resources are intentionally constrained.

If CI fails, fix only failures caused by this debloat; do not opportunistically rewrite unrelated code.

---

## 28. Static residue searches

These searches are intentionally broad. Every match must be classified rather than blindly deleted.

### 28.1 Core subscription identity

```bash
rg -n 'Jcode Subscription|jcode-subscription|JCODE_SUBSCRIPTION_ACTIVE' .
rg -n 'JCODE_API_KEY|JCODE_API_BASE|JCODE_ACCOUNT_ID|JCODE_ACCOUNT_EMAIL|JCODE_TIER' .
rg -n 'subscription_catalog|subscription_api|account_login' src crates
```

### 28.2 Provider/runtime identity

```bash
rg -n 'JcodeSubscription|JcodeProvider|JCODE_LOGIN_PROVIDER|LoginProviderTarget::Jcode' src crates
rg -n 'ModelRouteApiMethod::JcodeSubscription|RuntimeKey::JcodeSubscription' src crates
rg -n 'OpenRouterTransportState::JcodeSubscription' src crates
```

### 28.3 User-facing commands

```bash
rg -n '/hosted|/subscribe|/subscription|account jcode|jcode account' src crates docs
```

### 28.4 Jcode account/model endpoints

```bash
rg -n 'jcode\.sh/(pricing|account)|api\.jcode\.sh/v1' src crates docs scripts
```

Classify discovery/telemetry or other separate hosted-service endpoints before touching them.

### 28.5 Remote Compile

```bash
rg -n 'compile_remote|remote compile|Remote compilation|cloud-compute credits' src crates docs
```

### 28.6 Managed Cloud

```bash
rg -n 'RemoteCommand::Cloud|/remote cloud|/remote setup|Jcode Cloud' src crates docs
```

### 28.7 JEV

```bash
rg -n 'JevProvider::Jcode|memory_jev|browser_jev|jcode-subscription' crates/jcode-base/src/jev.rs crates/jcode-app-core docs
```

Do not interpret `memory_jev`/`browser_jev` alone as deletion targets.

### 28.8 Voice

```bash
rg -n 'subscription_voice_available|voice_transcription|audio/transcriptions|JCODE_API_KEY' crates/jcode-base/src/voice.rs
```

### 28.9 Model gating and remote picker residue

```bash
rg -n 'ensure_model_allowed_for_subscription|filtered_display_models' crates/jcode-base/src
rg -n 'append_jcode_subscription_routes_static|provider_is_jcode_subscription|poisoned_by_jcode_subscription' crates/jcode-tui/src
rg -n 'ModelRouteApiMethod::JcodeSubscription' crates/jcode-tui/src/tui/app/inline_interactive_placeholder_routes.rs
```

Expected result:

- no active subscription model-gating or Jcode route-synthesis path remains;
- `filtered_display_models` should disappear with subscription tests if unused;
- any retained `JcodeSubscription` reference must be justified strictly by the Section 19 legacy-decoding policy.

### 28.10 Auth/SDK residue

```bash
rg -n 'pub jcode: AuthState|probe_jcode_status|status\.jcode|LoginProviderAuthStateKey::Jcode' crates/jcode-base/src/auth
rg -n 'LoginProviderTarget::Jcode|Jcode subscription is API-key-only|Jcode subscription supports API-key' crates/jcode-sdk docs/DESKTOP_AUTH_SDK.md
rg -n -i 'supports the jcode subscription key|working credential is a jcode subscription' sdk/typescript
```

Expected result: no active Jcode auth slot or SDK login exposure remains.

Do **not** fail the audit merely because a generic TypeScript env-file inheritance test still uses an arbitrary filename such as `n.env`.

### 28.11 Quality-ratchet residue

```bash
rg -n 'subscription_api\.rs|subscription_catalog\.rs' scripts/swallowed_error_budget.json
```

A remaining baseline entry is cleanup debt, not evidence that the deleted source must be restored. Refresh the baseline only according to the repository ratchet policy after verifying the implementation.

### 28.12 Protocol false positive

```bash
rg -n '\bSubscribe\b|"subscribe"' crates src
```

Expected result: generic protocol/session subscription matches remain.

---

## 29. Acceptance criteria

The implementation is complete only when all applicable criteria below are satisfied.

### 29.1 Core provider/account

- [ ] `subscription_catalog` no longer exists as an active module.
- [ ] `subscription_api` no longer exists as an active module.
- [ ] Jcode commercial `account_login` no longer exists.
- [ ] `JcodeProvider` is removed.
- [ ] Jcode Subscription is absent from provider/login selection.
- [ ] Jcode Subscription is absent from model picker/catalog routing.
- [ ] active `ensure_model_allowed_for_subscription(...)` gating is removed with the subscription runtime.
- [ ] Jcode remote model-picker route synthesis/injection is removed.
- [ ] Jcode-specific `poisoned_by_jcode_subscription` repair logic is removed unless a narrowly documented inert migration still requires it.
- [ ] Jcode commercial account device login is removed.
- [ ] `jcode account login/status/manage/logout` commercial flow is removed.
- [ ] `jcode-subscription.env` is no longer created or read by active HAVK subscription code.
- [ ] HAVK no longer persists Jcode account ID/email/tier for this service.
- [ ] global `AuthStatus` no longer probes or exposes a first-class Jcode Subscription slot.
- [ ] the Rust SDK auth provider list no longer exposes Jcode Subscription.
- [ ] TypeScript SDK documentation no longer claims Jcode Subscription key support, while generic credential inheritance remains intact.

### 29.2 User-facing subscription UX

- [ ] `/hosted` subscription behavior is gone.
- [ ] `/subscribe` is gone.
- [ ] `/subscription` is gone.
- [ ] `/account jcode ...` actions are gone.
- [ ] subscription onboarding/nudge/pricing CTA is gone.
- [ ] generic `/account` remains functional for retained providers.
- [ ] retained provider login flows remain available.

### 29.3 Routing/runtime

- [ ] No active route is produced with `jcode-subscription`.
- [ ] No active `JcodeSubscription` runtime remains.
- [ ] OpenRouter BYOK still has its own normal transport identity.
- [ ] OpenAI-compatible provider profiles remain supported.
- [ ] local/no-auth compatible providers remain supported.
- [ ] legacy Jcode route metadata cannot silently trigger another paid provider.

### 29.4 Remote Compile

- [ ] built-in `compile_remote` tool is gone.
- [ ] Remote Compile source snapshotter is gone.
- [ ] no Remote Compile entitlement refresh exists.
- [ ] no compile request is submitted to the Jcode Subscription backend.
- [ ] no Jcode cloud-compute credit guidance remains for Remote Compile.
- [ ] generic local compilation/tool execution remains.
- [ ] future provider-neutral sandbox support has not been structurally prohibited.

### 29.5 Managed Cloud vs generic remote

- [ ] Jcode-managed Cloud activation is gone.
- [ ] bare `/remote` no longer opens a Jcode commercial account page.
- [ ] `/remote status/on/off/pair/revoke` or their current equivalents remain if still supported by HAVK.
- [ ] generic gateway, pairing, SSH/socket forwarding, reconnect, and session persistence are not removed by this task.

### 29.6 JEV / Memory / Browser

- [ ] `JevProvider::Jcode` is gone or decode-only/inert for a documented compatibility reason.
- [ ] Jcode JEV credentials and `/decisions` route are gone.
- [ ] Jcode JEV entitlement preflight is gone.
- [ ] Memory remains functional with retained JEV configuration.
- [ ] Browser remains functional with retained JEV configuration.
- [ ] no failure silently spends a different provider balance.

### 29.7 Voice

- [ ] subscription-backed Jcode voice transcription is gone.
- [ ] Jcode voice entitlement check is gone.
- [ ] Jcode voice upload endpoint is gone.
- [ ] retained non-Jcode voice functionality is not removed by this artifact unless separately approved.

### 29.8 Diagnostics / telemetry / docs

- [ ] support diagnostics no longer report Jcode commercial account ID/email/tier.
- [ ] active subscription-specific telemetry producers/schema are removed if telemetry remains.
- [ ] current user-facing docs do not instruct HAVK users to buy/login to Jcode Subscription.
- [ ] current Browser/Memory docs describe retained JEV routes.
- [ ] historical records are left intact unless they masquerade as current instructions.

### 29.9 Quality gates

- [ ] formatting passes.
- [ ] Cargo metadata/lockfile validation passes.
- [ ] unused direct dependencies introduced by the removal are cleaned up.
- [ ] affected tests pass in the configured validation environment.
- [ ] quality-ratchet baselines do not retain misleading subscription-file allowances after any intentional final rebaseline.
- [ ] remote CI is green before declaring the implementation merge-ready.
- [ ] the final diff contains no unrelated dependency upgrades or feature redesigns.

---

## 30. Non-goals

The agent must not expand this task into any of the following without a separate approved artifact:

```text
remove all telemetry
remove Partner Tool Discovery
remove Self-Dev
remove Grok Build
remove third-party OAuth subscriptions
remove OpenRouter
remove Memory
remove Browser
remove all Voice/native audio
remove generic account switching
remove generic /usage
remove generic remote/server operation
remove WebSocket gateway/pairing
remove SSH/socket forwarding
remove all Jcode-hosted URLs by keyword
replace Jcode Subscription with a new HAVK-hosted paid router
implement Modal or E2B sandbox integration
perform broad provider architecture redesign
perform global branding cleanup
perform unrelated Git debloat
```

---

## 31. Agent decision rules for ambiguous code

When the agent encounters code not listed here, use this decision tree.

```text
Does this code require the Jcode commercial account,
JCODE_API_KEY/jcode-subscription.env, Jcode tier/billing,
or the Jcode hosted-model router?
    |
    +-- YES --> Is that dependency the entire purpose of the component?
    |              |
    |              +-- YES --> delete component after removing consumers
    |              |
    |              +-- NO  --> remove only Jcode branch and retain generic feature
    |
    +-- NO --> Is it only named "subscription", "account", "managed", or "subscribe"?
                   |
                   +-- YES --> classify ownership; do not delete by name
                   |
                   +-- NO  --> outside subscription scope unless another direct coupling exists
```

For uncertainty affecting a shared subsystem, prefer preservation plus a narrowly documented TODO over destructive guessing.

Do not preserve an active Jcode commercial network path merely because cleanup is inconvenient.

---

## 32. Expected final source shape

A successful implementation should conceptually reduce the source to:

```text
provider metadata
├── retained OAuth providers
├── retained API-key providers
├── retained OpenAI-compatible profiles
└── local providers

provider runtime
├── OpenAI / Anthropic / Gemini / Copilot / etc.
├── OpenRouter BYOK
├── compatible endpoints
└── no JcodeSubscription runtime

auth
├── retained provider auth
├── generic account switching
├── no Jcode field/probe in global AuthStatus
└── no Jcode commercial account flow

features
├── Memory -> retained JEV
├── Browser -> retained JEV
├── Voice -> non-Jcode route(s), if retained
├── Remote -> self-hosted/generic remote
└── no subscription-backed Remote Compile
```

The source should be simpler because the Jcode commercial provider no longer needs to be treated as a special model provider, login provider, transport mode, billing identity, entitlement authority, or secondary feature credential.

---

## 33. Review checklist for the final PR

A reviewer should answer all of these questions before merge:

1. **Did the patch remove the actual Jcode Subscription service, or only hide UI?**
2. **Can any active path still read `jcode-subscription.env`?**
3. **Can any active model request still produce the `jcode-subscription` route?**
4. **Can any UI/CLI path still ask the user to buy or log into Jcode hosted inference?**
5. **Can Remote Compile still contact the Jcode backend or consume Jcode cloud credits?**
6. **Can JEV still choose the Jcode gateway?**
7. **Can voice still use a Jcode subscription key/backend?**
8. **Can `/remote` still activate Jcode-managed Cloud?**
9. **Did the patch accidentally remove OpenRouter BYOK?**
10. **Did the patch accidentally remove Claude/OpenAI/Gemini/Copilot OAuth?**
11. **Did it remove generic `/account`, `/usage`, `Subscribe`, remote gateway, Memory, or Browser?**
12. **Do old sessions referencing Jcode Subscription fail visibly rather than silently spending another provider?**
13. **Were generic tests preserved when Jcode was merely their fixture?**
14. **Did the patch remove active shared-model subscription gating and remote-picker Jcode route synthesis without damaging generic catalogs?**
15. **Did it remove the Jcode slot/probe from `AuthStatus` while preserving generic auth state?**
16. **Do Rust/TypeScript SDK surfaces no longer advertise Jcode Subscription while generic credential inheritance remains?**
17. **Were newly unused direct dependencies removed without unrelated version churn?**
18. **Were quality-ratchet baselines reviewed only after correctness, without hiding new regressions?**
19. **Does CI validate the changed provider/auth/TUI/protocol surfaces?**

If any answer is unknown, the task is not complete.

---

## 34. Why this artifact is structured this way

This specification intentionally follows coding-agent instruction practices rather than a traditional product-only PRD format.

The design principles are:

- state the goal and completion boundary early;
- separate deletion targets from protected shared infrastructure;
- provide high-confidence source hotspots to reduce blind repository exploration;
- keep `HEAD` as implementation truth so the artifact survives refactors;
- use explicit warnings for likely false positives;
- define observable acceptance criteria instead of prescribing every edit;
- preserve room for the agent to choose the smallest safe implementation for compatibility details;
- require validation and a residue audit rather than trusting the first patch;
- avoid unrelated feature decisions in the same artifact.

This approach is consistent with current guidance for coding agents and prompt/context engineering: clear and specific instructions, relevant context, explicit boundaries, build/test/validation information, iterative verification, and avoiding bloated or conflicting instruction sets.

---

## 35. External references used to design this artifact

### Jcode product/service boundary

- Jcode Terms of Service — hosted service boundaries: `https://jcode.sh/terms`
- Jcode pricing — hosted inference versus BYOK/OAuth/local use: `https://jcode.sh/pricing`
- Jcode docs — provider login, remote operation, generic account/usage surfaces: `https://jcode.sh/docs`
- Jcode TypeScript SDK docs — generic launched-instance/provider behavior: `https://jcode.sh/sdk`

### Jcode 0.88.0 in-tree source documentation re-read for this cross-check

```text
docs/REMOTE_COMPILE.md
docs/REMOTE_COMPILE_VALIDATION.md
docs/JCODE_CLOUD_AWS.md
docs/MEMORY_ARCHITECTURE.md
docs/BROWSER_FAST_AGENT.md
docs/DESKTOP_AUTH_SDK.md
docs/dev/ACCOUNT_CONTRACT_CONFORMANCE_TESTS.md
docs/dev/ACCOUNT_FLOWS_OBSERVABILITY_PRIVACY.md
```

These in-tree documents are used to establish implementation intent and feature boundaries for the audited source. Current `HEAD` still wins if a later HAVK/Jcode revision has changed those contracts.

### Coding-agent / instruction design

- GitHub Docs — repository custom instructions for Copilot: `https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions`
- GitHub Docs — writing effective custom instructions: `https://docs.github.com/en/copilot/tutorials/customize-code-review`
- OpenAI Developers — Rethinking skills and prompts for GPT-6 Astra: `https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra`
- Anthropic — Claude Code Best Practices: `https://www.anthropic.com/engineering/claude-code-best-practices`
- Google AI for Developers — Prompt design strategies: `https://ai.google.dev/gemini-api/docs/prompting-strategies`
- IBM — Prompt engineering / context engineering guidance: `https://www.ibm.com/think/topics/prompt-engineering` and `https://www.ibm.com/think/topics/context-engineering`

---

## 36. Final implementation directive

**Remove the Jcode/Solo Systems first-party Subscription and commercial account system completely from HAVK, including the hosted-model provider and subscription-dependent Remote Compile path.**

For shared features, remove only the Jcode Subscription route:

```text
JEV             -> keep feature, remove Jcode provider
Voice           -> keep non-Jcode paths, remove Jcode subscription backend
Remote          -> keep self-hosted/generic remote, remove Jcode-managed Cloud activation
Diagnostics     -> keep diagnostics, remove Jcode commercial identity
Telemetry       -> keep unless separately debloated, remove subscription-specific semantics
OpenRouter      -> keep BYOK/runtime, remove Jcode subscription transport identity
Account/Auth    -> keep generic provider-account infrastructure, remove Jcode commercial account
```

The implementation must leave HAVK capable of operating entirely through user-controlled provider credentials, supported third-party OAuth, local/self-hosted models, and retained generic infrastructure **without any requirement for a Jcode commercial account or hosted-model subscription**.
