# HAVK Debloat Plan — Phase 2: Remove Jcode Hosted-Model Subscription

> **Artifact type:** implementation/debloat specification  
> **Source baseline:** `jcode-0.88.0.tar.gz`  
> **Archive SHA-256:** `967e5a825f29b1ed3ab9649fe55966545ba4eba8e0d44e2897b015d2ada43b96`  
> **Target fork:** HAVK  
> **Status:** approved direction; implementation not yet performed  
> **Scope:** remove Jcode's own hosted-model subscription / metered inference product from HAVK.

---

## 1. Decision

HAVK will **not ship Jcode's first-party hosted-model subscription service**.

In upstream Jcode this product appears as:

- provider display name `Jcode Subscription`
- provider id `jcode`
- route/API method `jcode-subscription`
- `JCODE_API_KEY`
- `JCODE_API_BASE`
- `JCODE_ACCOUNT_ID`
- `JCODE_ACCOUNT_EMAIL`
- `JCODE_TIER`
- `JCODE_SUBSCRIPTION_ACTIVE`
- local credential file `jcode-subscription.env`
- account/billing API at `https://api.jcode.sh/v1`
- pricing at `https://jcode.sh/pricing`
- account management at `https://jcode.sh/account`
- device/account login
- curated hosted-model catalog
- server-managed model routing
- subscription usage/budget state
- hosted-model nudges and onboarding
- subscription lifecycle telemetry

All of that first-party **Jcode model-subscription path is a debloat target**.

HAVK should instead use normal model integrations chosen by the user: provider OAuth, provider API keys, OpenRouter BYOK, OpenAI-compatible APIs, Bedrock/Azure where retained, and local/self-hosted models.

---

## 2. Public upstream documentation verified

Audit sources:

- `https://jcode.sh/pricing`
- `https://jcode.sh/terms`
- `https://jcode.sh/docs`

At audit time, Jcode's pricing page describes an optional hosted inference product with recurring subscription billing, monthly inference credit, metered usage and a user-selected hard cap. The terms page separately identifies the **Subscription API** as a hosted metered model-routing service. The general docs separately describe provider subscriptions/OAuth such as Claude, OpenAI, Gemini and Copilot.

That distinction is essential for HAVK:

> **Remove Jcode's own subscription product. Do not remove third-party provider subscriptions simply because the word “subscription” appears.**

---

## 3. Scope boundary

### Remove

These are in scope:

- `Jcode Subscription`
- `jcode-subscription`
- Jcode hosted models / hosted inference
- Jcode-hosted model routing via `JCODE_API_KEY`
- Jcode subscription tiers and entitlements
- Jcode curated hosted-model catalog
- Jcode account/device login for this product
- Jcode billing/usage/budget status
- hosted-model sales/nudge UX
- `/hosted`
- `/subscribe` when it means the Jcode hosted-model pitch
- `/subscription` when it means Jcode hosted-model account status
- subscription analytics events tied to the product

### Preserve

Do **not** remove these as part of this phase:

- Claude OAuth / Claude Pro-Max access
- OpenAI/ChatGPT/Codex OAuth
- Gemini OAuth/entitlement login
- GitHub Copilot
- provider-specific paid plans
- OpenRouter BYOK
- generic provider API keys
- generic account switching
- generic `/usage`
- token usage tracking
- context usage
- generic model pricing metadata
- OpenAI/Anthropic service-tier logic
- server/client `subscribe` operations used for session event streaming
- WebSocket/Unix-socket `Subscribe` requests
- background watch/subscribe semantics

A global search-and-delete for the word `subscription` would be wrong.

---

## 4. Target behavior after debloat

After implementation:

1. `havk login` must not list **Jcode Subscription**.
2. `havk login --provider jcode` must not select a Jcode hosted-model provider.
3. Aliases `subscription` and `jcode-subscription` must not resolve to a Jcode model provider.
4. HAVK must not create/read `jcode-subscription.env` for model routing.
5. HAVK must not use `JCODE_API_KEY` to send model inference through Jcode's router.
6. HAVK must not show Jcode subscription tiers or entitlement errors.
7. HAVK must not expose Jcode hosted-model catalog entries as a synthetic provider.
8. `/hosted`, `/subscribe`, and `/subscription` hosted-model commands must disappear.
9. No model onboarding should open `jcode.sh/pricing` or `jcode.sh/account`.
10. No hosted-model upsell/nudge should appear.
11. OpenRouter must continue to work as a normal user-owned/BYOK provider.
12. Generic `/usage`, token accounting and provider usage views should remain.
13. Third-party OAuth subscriptions should remain.
14. Generic TUI provider-account switching should remain.

---

## 5. Architecture being removed

Upstream effectively has this first-party path:

```text
User
  |
  +-- jcode login --provider jcode
  |      |
  |      +-- device authorization
  |      +-- api.jcode.sh/v1/auth/device
  |      +-- browser/account approval
  |      +-- JCODE_API_KEY
  |      +-- hosted billing activation
  |
  +-- Jcode Subscription provider
         |
         +-- curated model catalog
         +-- tier/entitlement checks
         +-- managed route identity
         +-- OpenRouter-compatible transport slot
         +-- server-managed upstream routing
         |
         +--> Jcode hosted router
               +--> upstream model provider
```

HAVK target:

```text
User
  +--> selected provider login / BYOK / local runtime
         +--> Anthropic
         +--> OpenAI
         +--> Gemini
         +--> Copilot
         +--> OpenRouter BYOK
         +--> Bedrock / Azure if retained
         +--> OpenAI-compatible provider
         +--> local/self-hosted model
```

Do **not** merely rename the Jcode subscription to a HAVK subscription.

---

# 6. Dedicated files to delete

These files are high-confidence first-party subscription code.

## 6.1 Subscription catalog

### DELETE

`crates/jcode-base/src/subscription_catalog.rs`

Approximate size in this archive: **756 lines**.

It owns:

- Jcode account/model env names
- `jcode-subscription.env`
- `jcode-subscription` cache namespace
- Jcode API/pricing/account URLs
- `Jcode Subscription` identity
- `jcode-subscription` route method
- Jcode tiers
- curated hosted models
- tier/model entitlement checks
- managed router environment setup

This module should not exist in HAVK after this phase.

## 6.2 Subscription/account API

### DELETE

`crates/jcode-base/src/subscription_api.rs`

Approximate size: **734 lines**.

Its own source description is a typed client for the Jcode account and hosted-model billing API. It implements:

- subscription `/me`
- used USD / budget USD
- billed amount
- next charge threshold
- reset time
- plan status
- device authorization
- API-key issuance
- activation polling
- account API errors
- paid-plan checks
- remote key revocation

This is not generic model usage accounting.

## 6.3 Jcode account-login orchestration

### DELETE

- `crates/jcode-base/src/account_login.rs`
- `crates/jcode-base/src/account_login/tests.rs`

These depend on the Jcode subscription API/catalog and are not the generic multi-provider account infrastructure.

## 6.4 Dedicated Jcode model provider

### DELETE

`crates/jcode-base/src/provider/jcode.rs`

Approximate size: **421 lines**.

It implements `JcodeProvider`, curated subscription models, managed subscription routes and tier-filtered model availability.

## 6.5 Dedicated subscription provider tests

### DELETE

`crates/jcode-base/src/provider/tests/catalog_subscription.rs`

Approximate size: **768 lines**.

These tests validate a product HAVK intentionally removes.

## 6.6 CLI Jcode device flow

### DELETE

- `src/cli/login/jcode_device.rs`
- `src/cli/login/jcode_device/tests.rs`

This flow requests device authorization, opens the Jcode account page, obtains `JCODE_API_KEY`, persists account identity and waits for hosted billing/spending-limit activation.

## 6.7 Jcode-specific account CLI

### DELETE, subject to the coupling note below

`src/cli/account.rs`

The current top-level `jcode account ...` CLI is Jcode commercial-account specific:

- `jcode account login`
- `jcode account status`
- `jcode account manage`
- `jcode account logout`

It reports Jcode plan, usage, budget and account-management URL.

This is different from the TUI's generic provider account picker.

## 6.8 Hosted-model nudge

### DELETE

`crates/jcode-tui/src/tui/app/subscribe_nudge.rs`

Approximate size: **368 lines**.

It contains the hosted-model pitch and `/hosted`/`/subscribe` nudges.

---

# 7. Base module cleanup

Edit:

`crates/jcode-base/src/lib.rs`

Remove:

```rust
pub mod account_login;
pub mod subscription_api;
pub mod subscription_catalog;
```

Keep generic modules such as:

```rust
pub mod model_pricing;
pub mod model_usage;
```

---

# 8. Provider metadata cleanup

## 8.1 Remove descriptor

Edit:

`crates/jcode-provider-metadata/src/catalog.rs`

Remove `JCODE_LOGIN_PROVIDER`, currently defined with:

- id `jcode`
- display name `Jcode Subscription`
- aliases `subscription`, `jcode-subscription`
- target `LoginProviderTarget::Jcode`

Remove it from `LOGIN_PROVIDERS` and adjust any fixed array size.

## 8.2 Remove enum variants

Edit:

`crates/jcode-provider-metadata/src/lib.rs`

Remove, once all call sites are gone:

```rust
LoginProviderTarget::Jcode
LoginProviderAuthStateKey::Jcode
```

Update exhaustive matches.

Do not remove provider-specific subscription integrations such as Claude/OpenAI/Copilot.

---

# 9. Provider-core route identity cleanup

Edit:

`crates/jcode-provider-core/src/lib.rs`

Remove:

```rust
RuntimeKey::JcodeSubscription
ModelRouteApiMethod::JcodeSubscription
```

Also remove their handling from:

- `RuntimeKey::from_api_method`
- `RuntimeKey::stable_id`
- `RouteSelection::routed_model_spec`
- route parser for `jcode-subscription`
- UI/display label mapping for this method
- serde tests specific to this route

Keep generic `SetRoute` functionality.

---

# 10. Provider activation cleanup

Edit:

`crates/jcode-base/src/provider/activation.rs`

Remove Jcode-hosted activation state including:

- `RuntimeProviderId::Jcode`
- display name `Jcode Subscription`
- `ProviderActivation::jcode_subscription(...)`
- `JCODE_OPENROUTER_TRANSPORT_STATE=jcode-subscription`
- environment setup whose only purpose is the managed Jcode route

Keep `RuntimeProviderId::OpenRouter` and normal OpenRouter activation.

---

# 11. Provider module cleanup

Edit:

`crates/jcode-base/src/provider/mod.rs`

Remove:

```rust
pub mod jcode;
```

Remove all code that:

- instantiates `JcodeProvider`
- selects `Jcode Subscription`
- calls `set_model_on_jcode_subscription`
- maps Jcode provider identity to the OpenRouter runtime
- special-cases Jcode hosted model selection

Do not damage normal OpenRouter or OpenAI-compatible selection.

---

# 12. Catalog/routes/model filtering

Edit:

- `crates/jcode-base/src/provider/catalog_routes.rs`
- `crates/jcode-base/src/provider/models.rs`
- `crates/jcode-base/src/provider/selection.rs`

Remove Jcode subscription-only behavior:

- `is_jcode_subscription`
- provider label `Jcode Subscription`
- method `jcode-subscription`
- detail `jcode subscription routing · managed server-side`
- curated subscription catalog filtering
- Jcode tier checks
- Jcode upgrade errors
- synthetic/fallback Jcode hosted routes

After removal, catalog visibility should derive only from remaining providers.

---

# 13. Auth subsystem cleanup

## 13.1 Auth integration registry

Edit:

`crates/jcode-base/src/auth/integration.rs`

Remove:

```rust
LoginProviderTarget::Jcode => Some(RuntimeProviderId::Jcode)
```

## 13.2 Auth lifecycle

Edit:

`crates/jcode-base/src/auth/lifecycle.rs`

Remove Jcode-hosted aliases/mappings:

```text
jcode
subscription
jcode-subscription
```

when they normalize to the first-party Jcode provider.

Remove Jcode env-file mappings:

```text
JCODE_API_KEY
JCODE_API_BASE
jcode-subscription.env
```

Remove matching logic for `Jcode Subscription` / `jcode-subscription` route identity.

## 13.3 Auth status

Edit:

- `crates/jcode-base/src/auth/mod.rs`
- `crates/jcode-base/src/auth/status_types.rs`
- corresponding tests

Remove:

- Jcode auth-state field/branch
- Jcode credential probe
- `subscription_catalog::has_credentials()`
- Jcode router-base status
- `JCODE_API_KEY` account-source reporting
- `jcode-subscription.env` account-source reporting

Do not remove the generic provider auth framework.

---

# 14. CLI cleanup

## 14.1 Login dispatch

Edit:

`src/cli/login.rs`

Remove:

```rust
mod jcode_device;
```

and the `LoginProviderTarget::Jcode` branch, `login_jcode_flow(...)`, `run_jcode_account_login(...)`, plus text such as:

```text
Starting jcode subscription sign-in...
```

## 14.2 Remove commercial account CLI

Edit:

- `src/cli/args.rs`
- `src/cli/dispatch.rs`
- `src/cli/mod.rs`
- `src/cli/args/tests.rs`

Remove the top-level Jcode-commercial account command and `AccountCommand` if nothing else uses it.

### Do not confuse it with TUI `/account`

The TUI account center supports normal provider accounts and is not automatically a debloat target.

## 14.3 Provider initialization

Edit:

- `src/cli/provider_init.rs`
- `src/cli/provider_init_tests.rs`

Remove first-party Jcode provider selection and notices such as:

```text
Using Jcode subscription provider
```

---

# 15. TUI command cleanup

Remove hosted-model registration, dispatch and help for:

```text
/hosted
/hosted status
/subscribe
/subscription
/subscription status
```

Known locations include:

- `crates/jcode-tui/src/tui/app/auth_account_commands.rs`
- `crates/jcode-tui/src/tui/app/commands.rs`
- `crates/jcode-tui/src/tui/app/commands_dispatch.rs`
- `crates/jcode-tui/src/tui/app/state_ui_input_helpers.rs`
- `crates/jcode-tui/src/tui/app/input_help.rs`
- `crates/jcode-tui/src/tui/ui_overlays.rs`
- Jcode subscription command tests

Remove help text mentioning:

- Jcode hosted models
- monthly spending limit
- Jcode router
- `/login jcode`
- pricing/subscription setup

---

# 16. TUI onboarding cleanup

Edit:

- `crates/jcode-tui/src/tui/app/onboarding_flow.rs`
- `crates/jcode-tui/src/tui/app/onboarding_flow_control.rs`
- `crates/jcode-tui/src/tui/ui_onboarding.rs`
- related onboarding tests/goldens

Remove UX that:

- offers “Jcode subscription” as an onboarding choice
- preselects Jcode subscription
- treats Jcode subscription as the default alternative to imported credentials
- opens Jcode pricing
- waits for Jcode account/billing activation

Suggested HAVK onboarding categories:

```text
1. Import existing provider credentials
2. Log in to a supported provider
3. Configure API key / OpenAI-compatible endpoint
4. Configure local model
```

---

# 17. OpenRouter runtime cleanup — do not remove OpenRouter

This is a critical surgical edit.

Known file:

`crates/jcode-provider-openrouter-runtime/src/lib.rs`

Upstream contains:

```rust
OpenRouterTransportState::JcodeSubscription
```

and parsing for:

```text
jcode
jcode-subscription
subscription
```

It also detects Jcode runtime identity via:

- default `api.jcode.sh` base
- `JCODE_API_KEY`
- Jcode display name and route method

### Remove

- `OpenRouterTransportState::JcodeSubscription`
- parsing of Jcode subscription transport aliases
- Jcode runtime display identity
- Jcode route identity
- Jcode endpoint/key detection
- Jcode-specific OpenRouter tests

### Keep

- OpenRouter BYOK
- `OPENROUTER_API_KEY`
- OpenRouter model catalog
- OpenRouter endpoint/pricing cache
- direct OpenAI-compatible modes
- no-auth local compatible endpoints

**Jcode Subscription reused OpenRouter transport code; it is not the same thing as OpenRouter support.**

---

# 18. Protocol cleanup

Edit:

`crates/jcode-protocol/src/protocol_tests/misc_events.rs`

Remove the Jcode subscription `SetRoute` test after `RuntimeKey::JcodeSubscription` is deleted.

Do not remove generic `SetRoute` or generic session subscription requests.

---

# 19. Harness API / key import cleanup

Edit:

- `crates/jcode-harness-api-server/src/translate.rs`
- `crates/jcode-harness-api-server/src/translate_tests.rs`

Remove provider translation that maps:

```text
jcode
subscription
jcode-subscription
```

to:

```text
JCODE_API_KEY
jcode-subscription.env
```

Keep generic provider-key import logic.

---

# 20. SDK cleanup

Edit:

`crates/jcode-sdk/src/auth.rs`

Remove Jcode-subscription-specific documentation/behavior such as the claim that Jcode subscription is API-key-only in the SDK.

If the SDK provider list is generated from metadata, removing `JCODE_LOGIN_PROVIDER` should eliminate most of this automatically; verify the public API does not retain a `jcode` hosted provider promise.

---

# 21. Provider doctor cleanup

Edit:

`crates/jcode-provider-doctor/src/provider_e2e.rs`

Remove Jcode subscription diagnostics including:

- label `Jcode Subscription`
- auth source `JCODE_API_KEY`
- login hint `jcode login --provider jcode`
- missing/valid Jcode subscription credential messages

Preserve doctor coverage for remaining providers.

---

# 22. Generic `/usage` must remain

This phase removes **Jcode commercial billing usage**, not generic usage observability.

Keep:

- `crates/jcode-base/src/usage.rs` except small Jcode-specific branches
- `crates/jcode-usage-types`
- `crates/jcode-tui-usage-overlay`
- `/usage`
- provider usage/quota fetching where supported
- token accounting
- session token counts
- local spend estimates
- context usage

A source comment mentioning `jcode subscription` inside generic usage code is a reason to remove only that branch, not the subsystem.

---

# 23. Generic pricing must remain

Do not delete merely because filenames contain `pricing`:

- `crates/jcode-base/src/model_pricing.rs`
- `crates/jcode-base/src/provider/pricing.rs`
- `crates/jcode-provider-core/src/pricing.rs`

These support provider price calculations, local estimates and service tiers independently of Jcode billing.

---

# 24. Generic TUI account switching must remain

Do not delete generic account-picker code solely because the first-party subscription also uses the term “account”.

Preserve provider account switching for normal providers where supported.

Important distinction:

```text
CLI: jcode account ...   -> Jcode commercial account in this snapshot
TUI: /account            -> broader multi-provider account functionality
```

Treat them separately.

---

# 25. Telemetry cleanup

Explicit subscription events exist in the telemetry worker:

```text
subscription_login
subscription_activated
subscription_budget_exhausted
subscription_router_error
account_linked
```

Known locations:

- `telemetry-worker/src/worker.js`
- `telemetry-worker/test/worker.test.mjs`
- `telemetry-worker/migrations/0016_web_subscription_analytics.sql`
- `telemetry-worker/README.md`
- `TELEMETRY.md`

Remove generation/handling/docs for events whose sole purpose is the deleted hosted-model business.

At minimum remove:

- `subscription_login`
- `subscription_activated`
- `subscription_budget_exhausted`
- `subscription_router_error`

Review `account_linked`: if HAVK has no Jcode commercial account identity after this phase, it should likely disappear too.

### Migration caution

Do not rewrite already-deployed upstream database history blindly. If HAVK starts with its own new telemetry backend, the Jcode subscription-specific migration does not need to be part of the HAVK deployment chain. If reusing an existing DB, handle migrations as an infrastructure decision.

---

# 26. Stale docs to delete or rewrite

Subscription/account references appear in docs such as:

- `docs/REMOTE_COMPILE.md`
- `docs/BROWSER_FAST_AGENT.md`
- `docs/JCODE_CLOUD_AWS.md`
- `docs/DESKTOP_AUTH_SDK.md`
- `docs/MEMORY_ARCHITECTURE.md`
- `docs/dev/ACCOUNT_CONTRACT_CONFORMANCE_TESTS.md`
- `docs/dev/ACCOUNT_FLOWS_OBSERVABILITY_PRIVACY.md`
- `docs/DISCOVERY_CONVERSION_ANALYSIS.md`
- `TELEMETRY.md`

Rule:

```text
if the document is dedicated to Jcode subscription/account architecture:
    delete/archive it from HAVK docs
else:
    remove/rewrite only the Jcode subscription sections
```

Historical changelog entries should normally remain historical unless the fork has a separate changelog-cleanup policy.

---

# 27. Critical coupling: `JCODE_API_KEY` is reused outside model inference

This is the main implementation hazard found in the audit.

## 27.1 Remote compilation

Known file:

`crates/jcode-app-core/src/tool/compile_remote.rs`

It calls the subscription account API and checks:

```text
capabilities.remote_compile
```

It tells users to subscribe at `jcode.sh/pricing`, run `jcode account login`, and manage credits at `jcode.sh/account`.

### Consequence

Once the Jcode account/subscription API is removed, this hosted remote compile path cannot continue unchanged.

### Rule

Do not leave a dead advertised tool.

For HAVK, either:

- remove/disable the Jcode-hosted remote compile integration, or
- explicitly defer it to a later hosted-services debloat phase and ensure it is not presented as functional.

Do not preserve subscription code merely to keep this upstream commercial service alive.

## 27.2 Voice transcription

Known file:

`crates/jcode-base/src/voice.rs`

It uses:

- `subscription_api`
- `subscription_catalog`
- `JCODE_API_KEY`
- `/v1/me`
- `voice_transcription` entitlement

Remove the Jcode subscription entitlement/backend path. If an independent BYOK/local transcription backend exists, preserve that. Otherwise the Jcode-hosted transcription capability becomes a separate removal candidate.

## 27.3 JEV/browser decision service

Known file:

`crates/jcode-base/src/jev.rs`

The Jcode path recognizes:

```text
JCODE_API_KEY
jcode-subscription.env
https://api.jcode.sh/v1/decisions
```

The subsystem also has non-Jcode provider paths.

Remove the **Jcode subscription credential/provider option**, not the entire abstraction if BYOK alternatives remain useful.

## 27.4 Browser-fast/helper paths

Some browser/decision tests accept either Jcode subscription credentials or another BYOK credential. Preserve the independent path and remove only the Jcode account dependency.

---

# 28. Separate services are not automatically authorized for deletion

This phase does **not** authorize indiscriminate removal of every network service under the Jcode brand.

Separate decisions are still needed for:

- sponsor/discovery backend
- telemetry as a whole
- cloud agents
- remote compile as a whole
- voice as a whole
- browser/JEV as a whole
- email notifications
- update service
- tool discovery partnerships

A dependency on `JCODE_API_KEY` means the auth path must be resolved; it does not automatically prove the entire feature is unwanted.

---

# 29. Discovery API is not the model subscription

The repository also contains:

```text
https://api.jcode.sh/v1/discovery
```

Do not remove it in this phase merely because it shares the same origin.

Sponsored/discovery services should have their own HAVK debloat decision.

---

# 30. Session `Subscribe` protocol is unrelated

Many tests/files use:

```rust
client.subscribe()
Request::Subscribe
subscribe_events
```

These are client/server event-stream operations.

Examples include transport, disconnect, resume and multiclient tests.

**Keep them.** They are unrelated to paid model subscriptions.

---

# 31. Third-party OAuth subscriptions are explicitly preserved

Provider metadata includes descriptions such as:

```text
requires Claude Pro or Max subscription
requires ChatGPT Plus or Pro subscription
Coding Plan subscription API key
```

These refer to provider-owned plans, not Jcode hosted inference.

Do not remove them under this artifact.

---

# 32. Grok Build is a separate decision

Provider metadata also contains a separate `Grok Build` integration described as a subscription managed by Jcode.

It is related to Jcode-managed services but is **not the same provider as `Jcode Subscription`**.

For Phase 2:

- do not accidentally remove it via broad text replacement
- record it for a later “Jcode-hosted services” audit

If HAVK later adopts the stronger rule “zero Jcode/Solo Systems paid backends,” Grok Build should be evaluated then.

---

# 33. Recommended implementation order

## Step 1 — Remove user-facing entry points

Remove:

- Jcode subscription login provider
- `/hosted`
- `/subscribe`
- `/subscription`
- hosted nudges
- hosted onboarding choices
- Jcode commercial account CLI

Goal: users can no longer enter the feature.

## Step 2 — Remove dedicated provider route

Delete:

- `provider/jcode.rs`
- subscription catalog
- Jcode provider activation
- Jcode runtime/API-method variants

Goal: the model system cannot construct a hosted Jcode route.

## Step 3 — Remove auth/account implementation

Delete:

- subscription API
- account login
- device flow
- billing/status logic
- model-use key persistence

## Step 4 — Remove OpenRouter special case

Remove only the Jcode subscription transport identity while preserving OpenRouter BYOK and generic compatible transports.

## Step 5 — Remove tests/protocol remnants

Update/remove:

- Jcode subscription tests
- auth tests
- onboarding tests
- provider doctor
- SDK/harness mapping tests
- protocol Jcode route test

## Step 6 — Resolve coupled hosted services

For remote compile, voice and JEV/browser:

```text
if independent auth/provider exists:
    remove Jcode subscription path and keep alternative
else:
    remove/disable Jcode-hosted capability
```

## Step 7 — Remove subscription analytics/docs

Remove dead billing/account telemetry and stale product docs.

## Step 8 — Static residue audit

Run the searches below.

---

# 34. Static residue searches

First-party identities:

```bash
rg -n \
  'Jcode Subscription|jcode-subscription|JCODE_SUBSCRIPTION_ACTIVE|JCODE_TIER|JCODE_ACCOUNT_ID|JCODE_ACCOUNT_EMAIL' \
  .
```

Account/router values:

```bash
rg -n \
  'JCODE_API_KEY|JCODE_API_BASE|jcode-subscription\.env|api\.jcode\.sh/v1|jcode\.sh/pricing|jcode\.sh/account' \
  .
```

TUI/CLI:

```bash
rg -n \
  '"/hosted|"/subscribe|"/subscription|login --provider jcode|account login|account status|account manage' \
  src crates
```

Types:

```bash
rg -n \
  'JcodeSubscription|LoginProviderTarget::Jcode|LoginProviderAuthStateKey::Jcode|RuntimeProviderId::Jcode|JcodeProvider' \
  src crates
```

Telemetry:

```bash
rg -n \
  'subscription_login|subscription_activated|subscription_budget_exhausted|subscription_router_error|account_linked' \
  telemetry-worker TELEMETRY.md crates src
```

### Classify remaining hits

Every remaining hit must be classified as:

1. stale Jcode hosted-subscription code → remove
2. historical changelog → usually keep
3. third-party subscription → keep
4. generic event-stream subscribe → keep
5. other Jcode hosted service → separate decision
6. intentionally retained migration/compatibility artifact → document why

---

# 35. Acceptance criteria

## Provider/UI

- [ ] `Jcode Subscription` no longer appears in provider/login menus.
- [ ] provider id `jcode` no longer selects Jcode hosted models.
- [ ] aliases `subscription` / `jcode-subscription` no longer select Jcode hosted models.
- [ ] `/hosted` is gone.
- [ ] `/subscribe` hosted-model alias is gone.
- [ ] `/subscription` hosted billing/status command is gone.
- [ ] hosted-model nudges are gone.
- [ ] onboarding does not recommend Jcode hosted models.
- [ ] no UI asks the user to purchase Jcode model usage.

## Runtime

- [ ] `JcodeProvider` no longer exists.
- [ ] `RuntimeKey::JcodeSubscription` no longer exists.
- [ ] `ModelRouteApiMethod::JcodeSubscription` no longer exists.
- [ ] `OpenRouterTransportState::JcodeSubscription` no longer exists.
- [ ] no model request is routed through the Jcode hosted router.
- [ ] Jcode subscription tier entitlement logic is gone.
- [ ] curated Jcode hosted catalog is gone.

## Auth/account

- [ ] no Jcode hosted-model device login remains.
- [ ] no hosted billing activation polling remains.
- [ ] model-provider setup no longer persists `jcode-subscription.env`.
- [ ] no model login opens Jcode pricing/account pages.
- [ ] no hosted-model path requires `JCODE_API_KEY`.

## Billing/usage

- [ ] Jcode USD usage/budget structures are gone from model subscription runtime.
- [ ] billing tranche/next-charge behavior is gone.
- [ ] monthly spending-limit UX specific to Jcode is gone.
- [ ] Jcode plan/tier names are gone from model routing.

## Must still work conceptually

- [ ] Claude OAuth remains.
- [ ] OpenAI OAuth remains.
- [ ] Gemini remains.
- [ ] Copilot remains.
- [ ] OpenRouter BYOK remains.
- [ ] direct API providers remain.
- [ ] local/OpenAI-compatible providers remain.
- [ ] generic `/usage` remains.
- [ ] token accounting remains.
- [ ] generic provider pricing remains.
- [ ] generic TUI provider account switching remains.
- [ ] client/server session `Subscribe` behavior remains.

## Coupled services

- [ ] remote compile no longer advertises an unusable Jcode subscription route.
- [ ] Jcode-hosted voice entitlement is removed/replaced.
- [ ] JEV/browser Jcode subscription credentials are removed while independent alternatives remain where applicable.

---

# 36. Tests to delete rather than “fix”

Delete tests whose sole purpose is preserving the removed commercial product:

- Jcode subscription catalog tests
- Jcode account device-auth tests
- hosted billing activation tests
- subscription tier tests
- Jcode OpenRouter identity tests
- Jcode provider-doctor tests
- `/subscribe` sales-pitch tests
- `/subscription` billing-status tests
- onboarding tests requiring Jcode subscription
- telemetry tests for deleted subscription lifecycle events
- protocol tests for `RuntimeKey::JcodeSubscription`

Do not keep dead product behavior just to satisfy old tests.

---

# 37. Tests to preserve

Keep tests for:

- OpenRouter BYOK
- Claude/OpenAI OAuth
- API-key providers
- OpenAI-compatible endpoints
- model switching
- remaining route selection
- token usage
- generic provider usage
- generic account switching
- transport subscribe/resume
- WebSocket/Unix transport behavior
- provider failover independent of Jcode hosted routing

---

# 38. Legacy configuration handling

Potential stale upstream values:

```text
JCODE_API_KEY
JCODE_API_BASE
JCODE_ACCOUNT_ID
JCODE_ACCOUNT_EMAIL
JCODE_TIER
JCODE_SUBSCRIPTION_ACTIVE
jcode-subscription.env
provider = "jcode"
```

Recommended behavior:

### Environment variables

Do not consume them for HAVK model routing. HAVK does not need to mutate the user's shell environment.

### Old `jcode-subscription.env`

Stop loading it. It can remain an unused upstream legacy file until a separate migration/uninstall cleanup.

### Old default provider `jcode`

Do **not** silently map it to OpenRouter or another provider; that could unexpectedly change billing/credentials.

Prefer a clear migration error telling the user the old Jcode hosted provider is no longer supported and that they must choose a provider explicitly.

---

# 39. Security/privacy benefit

Removing this subsystem removes a large upstream-specific account surface:

- device authorization
- Jcode bearer key storage
- account-id/email persistence
- hosted billing status polling
- remote key revocation
- commercial usage/budget state
- account-to-telemetry linking
- server-managed model routing through an upstream service

The desired model is simpler:

```text
HAVK = harness
model access = user's chosen provider/key/OAuth/local runtime
```

---

# 40. Maintenance benefit

This removes maintenance obligations for:

- Jcode plan tiers
- hosted model curation
- billing contract compatibility
- spend limits
- account activation
- managed router identity
- hosted upsell UI
- subscription analytics
- Jcode commercial credential lifecycle

HAVK does not operate the Jcode/Solo Systems account/router backend, so retaining this code would produce dead or misleading product surfaces.

---

# 41. Approximate dedicated code identified

Dedicated files already identified account for at least roughly:

```text
subscription_catalog.rs                         ~756 lines
subscription_api.rs                             ~734 lines
account_login.rs                                ~270 lines
account_login/tests.rs                          ~320 lines
provider/jcode.rs                               ~421 lines
provider/tests/catalog_subscription.rs          ~768 lines
src/cli/login/jcode_device.rs                   ~225 lines
src/cli/account.rs                              ~136 lines
subscribe_nudge.rs                              ~368 lines
telemetry migration 0016                         ~40 lines
----------------------------------------------------------
subtotal                                      ~4,038 lines
```

This does **not** include:

- `src/cli/login/jcode_device/tests.rs`
- Jcode branches in shared auth/provider files
- onboarding code
- OpenRouter transport special cases
- provider-core variants
- SDK/harness/provider-doctor remnants
- telemetry handlers/tests/docs

Therefore the real reduction is larger than 4,000 lines.

The objective is subsystem removal, not maximizing deleted line count.

---

# 42. Consolidated file checklist

## Dedicated delete candidates

- [ ] `crates/jcode-base/src/subscription_catalog.rs`
- [ ] `crates/jcode-base/src/subscription_api.rs`
- [ ] `crates/jcode-base/src/account_login.rs`
- [ ] `crates/jcode-base/src/account_login/tests.rs`
- [ ] `crates/jcode-base/src/provider/jcode.rs`
- [ ] `crates/jcode-base/src/provider/tests/catalog_subscription.rs`
- [ ] `src/cli/login/jcode_device.rs`
- [ ] `src/cli/login/jcode_device/tests.rs`
- [ ] `src/cli/account.rs`
- [ ] `crates/jcode-tui/src/tui/app/subscribe_nudge.rs`

## Shared source edit candidates

- [ ] `crates/jcode-base/src/lib.rs`
- [ ] `crates/jcode-base/src/auth/integration.rs`
- [ ] `crates/jcode-base/src/auth/lifecycle.rs`
- [ ] `crates/jcode-base/src/auth/mod.rs`
- [ ] `crates/jcode-base/src/auth/status_types.rs`
- [ ] `crates/jcode-base/src/auth/tests.rs`
- [ ] `crates/jcode-base/src/provider/mod.rs`
- [ ] `crates/jcode-base/src/provider/activation.rs`
- [ ] `crates/jcode-base/src/provider/catalog_routes.rs`
- [ ] `crates/jcode-base/src/provider/models.rs`
- [ ] `crates/jcode-base/src/provider/openrouter.rs`
- [ ] `crates/jcode-base/src/provider/selection.rs`
- [ ] `crates/jcode-base/src/provider/tests.rs`
- [ ] `crates/jcode-base/src/usage.rs` — Jcode branch only
- [ ] `crates/jcode-base/src/voice.rs` — remove Jcode entitlement/backend dependency
- [ ] `crates/jcode-base/src/jev.rs` — remove Jcode credential/provider path
- [ ] `crates/jcode-provider-core/src/lib.rs`
- [ ] `crates/jcode-provider-metadata/src/lib.rs`
- [ ] `crates/jcode-provider-metadata/src/catalog.rs`
- [ ] `crates/jcode-provider-openrouter-runtime/src/lib.rs`
- [ ] `crates/jcode-provider-openrouter-runtime/src/openrouter_tests.rs`
- [ ] `crates/jcode-provider-doctor/src/provider_e2e.rs`
- [ ] `crates/jcode-sdk/src/auth.rs`
- [ ] `crates/jcode-harness-api-server/src/translate.rs`
- [ ] `crates/jcode-harness-api-server/src/translate_tests.rs`
- [ ] `crates/jcode-protocol/src/protocol_tests/misc_events.rs`

## CLI

- [ ] `src/cli/mod.rs`
- [ ] `src/cli/args.rs`
- [ ] `src/cli/args/tests.rs`
- [ ] `src/cli/dispatch.rs`
- [ ] `src/cli/login.rs`
- [ ] `src/cli/provider_init.rs`
- [ ] `src/cli/provider_init_tests.rs`

## TUI

- [ ] `crates/jcode-tui/src/tui/app/auth.rs`
- [ ] `crates/jcode-tui/src/tui/app/auth_account_commands.rs`
- [ ] `crates/jcode-tui/src/tui/app/auth_tests.rs`
- [ ] `crates/jcode-tui/src/tui/app/commands.rs`
- [ ] `crates/jcode-tui/src/tui/app/commands_dispatch.rs`
- [ ] `crates/jcode-tui/src/tui/app/commands_remote.rs`
- [ ] `crates/jcode-tui/src/tui/app/inline_interactive.rs`
- [ ] `crates/jcode-tui/src/tui/app/inline_interactive_placeholder_routes.rs`
- [ ] `crates/jcode-tui/src/tui/app/input_help.rs`
- [ ] `crates/jcode-tui/src/tui/app/onboarding_flow.rs`
- [ ] `crates/jcode-tui/src/tui/app/onboarding_flow_control.rs`
- [ ] `crates/jcode-tui/src/tui/app/state_ui_input_helpers.rs`
- [ ] `crates/jcode-tui/src/tui/mod.rs`
- [ ] `crates/jcode-tui/src/tui/ui_onboarding.rs`
- [ ] `crates/jcode-tui/src/tui/ui_overlays.rs`
- [ ] Jcode subscription-specific TUI tests/goldens

## Coupled hosted services

- [ ] `crates/jcode-app-core/src/tool/compile_remote.rs`
- [ ] `crates/jcode-app-core/src/tool/compile_remote/tests.rs`
- [ ] `crates/jcode-app-core/src/tool/browser_jev.rs`
- [ ] browser-fast live tests mentioning Jcode subscription
- [ ] credential lists that include `JCODE_API_KEY`

## Telemetry/docs

- [ ] `telemetry-worker/src/worker.js`
- [ ] `telemetry-worker/test/worker.test.mjs`
- [ ] `telemetry-worker/migrations/0016_web_subscription_analytics.sql`
- [ ] `telemetry-worker/README.md`
- [ ] `TELEMETRY.md`
- [ ] Jcode subscription/account architecture docs

---

# 43. Rebrand-sensitive rule

Wrong approach:

```text
Jcode Subscription -> HAVK Subscription
JCODE_API_KEY -> HAVK_API_KEY
api.jcode.sh -> api.havk...
```

That merely renames a commercial hosted service architecture HAVK does not operate.

Correct approach:

```text
remove first-party subscription provider entirely
```

A future HAVK hosted service would require a separate deliberate design.

---

# 44. Validation policy for HAVK

Do not run a full local workspace compile/test loop on the low-memory development machine solely for this debloat.

Implementation workflow:

1. edit source
2. run lightweight static residue searches (`rg`, source inspection, `git diff`)
3. inspect obvious Rust module/exhaustive-match fallout
4. commit changes
5. let GitHub Actions / PR CI perform the expensive build/test matrix
6. fix CI failures in follow-up commits

Avoid local full-workspace compilation that risks OOM.

---

# 45. Final HAVK model-access philosophy

```text
HAVK does not sell model usage.
HAVK does not proxy model traffic through a Jcode/HAVK subscription router.
HAVK connects to providers selected by the user.
Authentication/billing is between the user and those providers,
unless a future HAVK service is designed separately.
```

---

# 46. Phase boundary

This artifact authorizes:

> **Removal of the first-party Jcode hosted-model subscription and all client/runtime/UI/auth/billing machinery whose sole purpose is that product.**

It does **not yet authorize blanket removal** of:

- telemetry as a whole
- sponsored discovery
- cloud features as a whole
- voice as a whole
- Grok Build
- browser automation/JEV as a whole
- generic usage reporting
- generic provider pricing
- generic accounts
- third-party provider subscriptions/OAuth

Those should be evaluated as separate HAVK debloat phases.

---

## Audit conclusion

Jcode's hosted-model subscription is a **cross-cutting subsystem**, not a pricing button.

The core removal needs to eliminate:

- dedicated Jcode provider
- curated hosted-model catalog
- Jcode subscription route/runtime identity
- account/device login
- billing/status API
- plan tiers and usage budgets
- Jcode commercial account CLI
- hosted-model TUI commands/nudges/onboarding
- OpenRouter subscription transport special case
- provider metadata/auth registry entries
- subscription analytics
- stale hosted-model documentation

At the same time HAVK must deliberately preserve:

- third-party OAuth subscriptions
- direct/BYOK providers
- OpenRouter BYOK
- local providers
- generic token/cost/usage tracking
- generic account switching
- generic session `Subscribe` transport operations

That separation is the central safety requirement for implementing this phase.
