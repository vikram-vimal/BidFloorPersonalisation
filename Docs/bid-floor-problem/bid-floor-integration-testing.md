# GSL SmartFloors — Integration Testing

This document defines how GSL BidFloor integration is tested, what the client must build to support verification, and the criteria for release readiness.

{% hint style="info" %}
**Prerequisite:** [Integration specification](bid-floor-integration.md) — architecture requirements and integration stages.
{% endhint %}

---

## Testing model

Integration testing has three phases. GSL verification is the gate before release; client QA runs in parallel.

| Phase | Owner | Purpose |
| --- | --- | --- |
| **1. Internal preflight** | Client (engineering) | Catch integration defects before external review |
| **2. GSL verification** | GSL | Confirm integration correctness on real devices |
| **3. Release readiness** | Client (engineering + release) | Confirm production build meets all integration conditions |

GSL evaluates **integration correctness only** — not gameplay quality or broad user-facing QA. The client team is still expected to run its own QA in parallel to validate placement behaviour and ensure there are no runtime errors in the ad flow.

---

## What the client must build for integration testing

The following are build requirements, not optional QA conveniences. Without them, GSL cannot complete verification.

### 1. Deterministic ad trigger paths

For each supported ad format (e.g. interstitial, rewarded), the build must include at least one impression path that is:

- Reachable without deep gameplay progression
- Triggerable repeatedly on a real device
- Free of special account states or environment flags (beyond the designated GSL test user ID flow)

**Acceptable patterns:** button-triggered ad, short level flow that reliably surfaces an ad opportunity.

**Unacceptable patterns:** impressions gated behind hours of progression, paywalls, A/B flags that block the ad route, or routes only accessible in non-production environments.

### 2. Complete event chain

Every ad lifecycle triggered through a test path must emit the full event sequence through the GSL SDK:

```text
SDK initialized
    → ad load requested (via GSL path)
    → ad load success / fail
    → ad shown
    → ad revenue recorded
```

GSL verifies this chain through log inspection and SDK-level analysis. Missing events fail verification regardless of whether ads display correctly to the user.

### 3. Holdout testability

The build must support holdout verification without embedded shortcuts:

| Requirement | Rationale |
| --- | --- |
| Holdout split active (0–50%) | GSL validates cohort branching behaviour |
| Designated test user ID routes correctly through holdout logic | Confirms assignment is not bypassed |
| No hardcoded user IDs in the binary | Hardcoded overrides invalidate verification results |

### 4. Production-representative build

GSL requires **one standard build** that reflects normal production behaviour. The build must be:

- Stable — no crash-prone early-development quality
- Representative — integration paths match what will ship to users
- Free of debug-only ad routing that differs from production

---

## Internal preflight

Before submitting a build to GSL, the client team should run the **SDK Integration Test tool**.

This is a preflight checkpoint to catch integration defects — initialization failures, missing events, routing bypasses — before external review begins.

{% hint style="warning" %}
**Open item:** Document the SDK Integration Test tool setup, invocation steps, and pass/fail criteria once GSL provides tooling documentation. This page will be updated when that material is available.
{% endhint %}

### Preflight checklist

- [ ] Cold start: GSL BidFloor SDK initializes before any ad opportunity on all platforms
- [ ] Ad load and show route through GSL integration layer for all supported placements
- [ ] Ad load, show, and revenue events appear in GSL SDK logs for each test trigger path
- [ ] Holdout behaviour responds correctly to the designated test user ID
- [ ] No hardcoded user IDs present in the build
- [ ] SDK Integration Test tool passes (when available)

---

## GSL verification

GSL verifies the integration on **real devices** through log inspection and SDK-level analysis.

### What GSL checks

| Check | Pass criteria |
| --- | --- |
| **SDK initialization** | GSL BidFloor SDK initializes successfully on app start |
| **Ad routing** | Ad requests and shows pass through the GSL integration layer |
| **Event emission** | Expected SDK events (load, show, revenue) fire for verified impression paths |
| **Holdout behaviour** | Holdout logic applies correctly for the designated test user ID flow |

### What GSL does not check

- Gameplay quality or user experience beyond the ad flow
- Broader monetisation performance (eCPM uplift, fill rate impact)
- Non-GSL ad placements or formats outside the agreed scope

### Submission requirements

| Requirement | Detail |
| --- | --- |
| **Build type** | One standard build per verification cycle |
| **Build quality** | Stable; mirrors normal production behaviour |
| **User IDs** | No hardcoded user IDs in the binary |
| **Preflight** | SDK Integration Test tool run recommended before submission |

---

## Client QA (parallel)

While GSL verifies integration correctness, the client team should run parallel QA focused on:

- Placement behaviour — correct ad format, timing, and dismissal flow
- Runtime stability — no crashes or hangs in the ad flow across supported devices
- Cross-platform parity — identical integration behaviour on Unity, Android, and iOS builds
- Edge cases — backgrounding during ad playback, network loss during load, rapid re-trigger

Client QA results do not replace GSL verification. Both must pass before release.

---

## Release readiness

Release proceeds only after GSL confirms the integration is valid and provides go-ahead.

### Release gate conditions

| Condition | Verified by |
| --- | --- |
| GSL integration verification passed | GSL |
| SDK Integration Test tool passed | Client |
| Client parallel QA completed with no blocking ad-flow defects | Client |
| Holdout split configured and behaving correctly | GSL + Client |
| Submitted build matches the build approved for release | Client (release) |

### Post-approval rollout

After GSL go-ahead, rollout follows the client's preferred deployment process — internal QA, staged rollout, or full release. GSL does not mandate a specific rollout strategy.

---

## What's next

{% content-ref url="bid-floor-risks-and-challenges.md" %}
[bid-floor-risks-and-challenges.md](bid-floor-risks-and-challenges.md)
{% endcontent-ref %}

{% content-ref url="bid-floor-integration.md" %}
[bid-floor-integration.md](bid-floor-integration.md)
{% endcontent-ref %}
