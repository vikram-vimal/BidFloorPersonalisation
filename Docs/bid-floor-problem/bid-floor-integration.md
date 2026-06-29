# GSL SmartFloors — Integration Specification

This document defines what is required to integrate **GSL SmartFloors** (GSL BidFloor) into a mobile game. It is written for engineering, monetisation, ad operations, and release stakeholders who need a single reference for integration behaviour end to end.

{% hint style="info" %}
**Related pages**
- [Integration testing](bid-floor-integration-testing.md) — verification workflow, testability requirements, and release readiness
- [Risks and challenges](bid-floor-risks-and-challenges.md) — known integration risks and mitigations
- [Problem definition](problem_defination.md) — business context for bid floor personalisation
{% endhint %}

---

## Scope

This specification covers:

| In scope | Out of scope |
| --- | --- |
| SDK initialization and ad lifecycle routing | Gameplay QA and general user-facing quality |
| Ad event emission through the GSL SDK | GSL floor optimisation logic internals |
| Holdout assignment and verification behaviour | Mediation platform setup beyond access prerequisites |
| Platform parity across Unity, Android, and iOS | Post-launch A/B analysis methodology |

---

## Integration summary

The game must satisfy four obligations for a valid GSL BidFloor integration:

1. **Initialize early** — GSL BidFloor SDK is active before any monetised ad flow can begin.
2. **Route through GSL** — Ad load and show for supported placements pass through the GSL integration layer, not a parallel native path.
3. **Emit events** — Ad lifecycle events (load, show, revenue) are logged through the GSL SDK.
4. **Remain verifiable** — At least one deterministic, reviewer-accessible impression path exists per supported ad format.

---

## Preconditions

Before SDK work begins, the following must be in place.

### Client responsibilities

| Requirement | Detail |
| --- | --- |
| **GSL registration** | Game registered in GSL data tracking and BidFloor SDK systems |
| **Ad unit configuration** | Relevant placements (e.g. interstitial, rewarded) configured in the mediation stack |
| **Mediation access** | GSL granted access to the MAX, AdMob, or LevelPlay account used for monetisation operations |
| **Placement selection** | Product and monetisation teams define which placements participate in bid floor control |
| **Ad operations** | Selected placements exist and are correctly configured in mediation |

### Dependency chain

```text
Placement selection (product/monetisation)
        ↓
Ad unit setup in mediation (ad operations)
        ↓
GSL account access granted
        ↓
SDK integration (engineering)
        ↓
Integration verification (engineering + GSL)
        ↓
Release
```

Bid floor personalisation depends on both application-side integration and partner-side visibility into the live monetisation configuration. GSL access must be granted before verification so platform behaviour can be inspected against the production mediation setup.

---

## Integration stages

Integration follows a six-stage flow. Each stage is a hard requirement — not a recommendation.

| Stage | Requirement | Owner |
| --- | --- | --- |
| **1. Ad unit provisioning** | Create GSL-specific ad units. Unit count depends on the game's ad impression volume and DAU. GSL reserves the right to set floors on these units through mediation APIs. | Client + GSL |
| **2. SDK initialization** | Initialize the GSL BidFloor SDK early in the app lifecycle, before monetised sessions begin. | Client (engineering) |
| **3. Ad routing** | Request and display ads through the GSL SDK integration path. Do not bypass GSL with direct native mediation SDK calls for supported placements. | Client (engineering) |
| **4. Event logging** | Log ad-related events through the GSL SDK so GSL BidFloor receives the monetisation signals required for validation and operation. | Client (engineering) |
| **5. Holdout assignment** | Configure a seed to split users into holdout and GSL BidFloor cohorts. Holdout may be 0–50%. Holdout behaviour must function correctly when GSL uses a designated test user ID during verification. | Client + GSL |
| **6. Build submission** | Submit a stable standard build for GSL review. Release proceeds only after GSL confirms the integration is valid. | Client (engineering + release) + GSL|

---

## Architecture requirements

### SDK initialization

The GSL BidFloor SDK must initialize early in the application lifecycle:

| Platform | Initialization point |
| --- | --- |
| **Unity** | Before gameplay systems can trigger ad opportunities |
| **Android** | Earliest reliable app-start phase in the host application lifecycle |
| **iOS** | Earliest reliable app-start phase in the host application lifecycle |

The exact entry point name is less important than the guarantee: **no monetised ad flow begins before GSL is active**. Late initialization causes early-session impressions to bypass GSL BidFloor logic or fail verification because GSL cannot observe the complete ad request and event chain.

### Ad request and display ownership

GSL requires the app to load and show ads through the GSL BidFloor SDK — not merely integrate native platform ad SDKs in parallel.

For cross-platform games, a single monetisation abstraction should delegate to GSL-enabled ad loading and showing on every client runtime. Unity wrappers, Android modules, and iOS managers are platform adapters around one shared monetisation policy, not three independently designed integrations.

### Ad event logging

Ad-related events must be logged through the GSL SDK. This is an **event completeness** requirement, not a narrow analytics task.

The ad lifecycle must be observable from initialization through impression and monetisation outcome. GSL validates that expected events — ad load, ad shown, and ad revenue — are fired for every verified impression path.

| Event category | Purpose |
| --- | --- |
| **Load** | Confirms the GSL-routed request path is active |
| **Show** | Confirms the impression was displayed to the user |
| **Revenue** | Confirms monetisation outcome was captured |

### Predictable impression paths

GSL requires ad impressions that can be reached without complex gating — for example, a short level flow or a button-triggered ad.

Verification runs on real devices and depends on reproducible access to ad opportunities. Each supported ad format must expose at least one deterministic test route that QA and GSL reviewers can trigger repeatedly without progression blockers, special account states, or deep gameplay dependence.

### Holdout behaviour

GSL verifies that holdout behaviour is correctly applied when using the designated test user ID. Holdout logic includes controlled behavioural branching that must remain intact during testing and must not be overridden by build-specific shortcuts.

{% hint style="warning" %}
Verification builds must **not** contain hardcoded user IDs. Holdout logic must be externally testable and environment-safe.
{% endhint %}

---

## Platform parity

Business logic is identical across Unity, Android, and iOS. Platform differences are implementation mechanics only; the integration contract is the same.

| Integration area | Requirement (all platforms) |
| --- | --- |
| **Startup** | Initialize early, before monetised flows begin |
| **Ad mediation path** | Load and show via GSL integration layer |
| **Event coverage** | Ad load, show, and revenue events reach GSL |
| **Testability** | Easy-to-trigger impressions per supported format |
| **Holdout verification** | Correct behaviour for test user ID; no hardcoded IDs in build |

---

## Client deliverables

Before requesting GSL verification, the client team must confirm:

- [ ] GSL-specific ad units created; mediation access granted to GSL
- [ ] GSL BidFloor SDK initializes on every cold start, before any ad opportunity
- [ ] All supported placements route through the GSL-controlled integration path
- [ ] Ad load, show, and revenue events emit consistently for every verified ad lifecycle
- [ ] At least one reviewer-friendly ad trigger path exists per supported format
- [ ] Holdout split configured (0–50%); holdout behaviour testable without hardcoded identifiers
- [ ] Standard build submitted that mirrors normal production behaviour

Full verification and release criteria are documented in [Integration testing](bid-floor-integration-testing.md).

---

## What's next

{% content-ref url="bid-floor-integration-testing.md" %}
[bid-floor-integration-testing.md](bid-floor-integration-testing.md)
{% endcontent-ref %}

{% content-ref url="bid-floor-risks-and-challenges.md" %}
[bid-floor-risks-and-challenges.md](bid-floor-risks-and-challenges.md)
{% endcontent-ref %}
