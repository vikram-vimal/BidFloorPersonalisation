# GSL SmartFloors — Risks and Challenges

This document captures known integration risks, their impact, and mitigations. It is derived from the GSL BidFloor integration contract and intended for engineering, monetisation, and release stakeholders planning an integration.

{% hint style="info" %}
**Related pages**

- [Integration specification](bid-floor-integration.md) — architecture requirements
- [Integration testing](bid-floor-integration-testing.md) — verification workflow and testability requirements
{% endhint %}

---

## Risk summary

| Risk | Severity | Likelihood | Mitigation |
| --- | --- | --- | --- |
| Late SDK initialization | High | Medium | Enforce init-before-ads guard in monetisation abstraction |
| Parallel native SDK bypass | High | Medium | Single monetisation path; code review for direct MAX/AdMob calls |
| Incomplete event emission | High | Medium | Event completeness checklist per ad lifecycle; preflight tool |
| Non-reproducible impression paths | Medium | High | Dedicated test triggers per format; QA verification before submission |
| Hardcoded test user IDs | High | Low | Lint/review gate; no developer overrides in release builds |
| Holdout logic overridden by build flags | High | Medium | Environment-safe holdout; no debug shortcuts in verification build |
| Platform implementation divergence | Medium | Medium | Shared monetisation policy; platform parity table in spec |
| Verification build ≠ production build | High | Medium | Submit production-representative build; no debug-only ad routing |
| Insufficient GSL mediation access | High | Low | Grant access in preconditions phase before SDK work |
| Ad unit misconfiguration | Medium | Medium | Ad ops review before verification; GSL inspects live setup |

---

## Integration risks

### Late SDK initialization

**Risk:** GSL BidFloor SDK initializes after the first ad opportunity, causing early-session impressions to bypass GSL logic.

**Impact:** Verification fails. Early-session revenue is not subject to bid floor personalisation.

**Mitigation:** Initialize at the earliest reliable app-start phase on every platform. Add a guard in the monetisation layer that blocks ad requests until GSL reports ready.

---

### Parallel native SDK bypass

**Risk:** Engineering integrates GSL alongside existing direct MAX, AdMob, or LevelPlay calls. Some placements route through GSL; others bypass it.

**Impact:** GSL cannot observe or control bypassed impressions. Verification fails for affected placements. Floor personalisation is partial.

**Mitigation:** Treat GSL as the sole load/show path for all supported placements. Audit existing ad call sites before integration. Use one shared monetisation abstraction across Unity, Android, and iOS.

---

### Incomplete event emission

**Risk:** Ads load and display correctly, but load, show, or revenue events are not consistently logged through the GSL SDK.

**Impact:** GSL verification fails. GSL BidFloor lacks the signals required for operational behaviour and validation.

**Mitigation:** Map every ad lifecycle state to a GSL SDK event. Validate the full chain (init → load → show → revenue) on each test trigger path before submission. Run the SDK Integration Test tool as a preflight gate.

---

### Non-reproducible impression paths

**Risk:** Ad opportunities require deep gameplay progression, special account states, or environment-specific flags that GSL reviewers cannot reach on a real device.

**Impact:** Verification cannot complete. Review cycles extend.

**Mitigation:** Build at least one deterministic, button-or-short-flow trigger per supported ad format. Validate reachability on a clean device install before submitting to GSL.

---

## Verification and release risks

### Hardcoded test user IDs

**Risk:** Developers embed fixed user IDs in the build to force holdout or treatment behaviour during internal testing.

**Impact:** GSL verification is invalidated. Build must be resubmitted.

**Mitigation:** Prohibit hardcoded IDs in release and verification builds. Use the designated GSL test user ID flow for holdout verification. Add a release checklist item and code review gate.

---

### Holdout logic overridden by build flags

**Risk:** Debug flags, A/B test overrides, or build-specific shortcuts bypass holdout assignment during verification.

**Impact:** GSL cannot confirm holdout behaviour. Verification fails.

**Mitigation:** Holdout logic must be environment-safe and externally testable. Remove debug overrides from the verification build. Confirm holdout responds correctly to the designated test user ID in preflight.

---

### Verification build does not match production

**Risk:** The build submitted for GSL review uses debug ad routing, test-only placements, or integration paths that differ from the production release.

**Impact:** Verification results are not meaningful for the shipped product. Post-release integration may behave differently.

**Mitigation:** Submit one standard build that mirrors normal production behaviour. Avoid debug-only ad routes in the verification binary.

---

## Organisational and operational risks

### Insufficient GSL mediation access

**Risk:** SDK integration proceeds before GSL has access to the MAX, AdMob, or LevelPlay account.

**Impact:** GSL cannot inspect live monetisation configuration. Floor setting through mediation APIs cannot be validated. Verification is blocked.

**Mitigation:** Complete the preconditions dependency chain — ad unit setup and GSL access — before engineering begins SDK work.

---

### Ad unit misconfiguration

**Risk:** Placements selected for bid floor control are misconfigured in mediation, or GSL-specific ad units are not provisioned correctly.

**Impact:** Floor personalisation applies to the wrong inventory or not at all. Revenue impact is unpredictable.

**Mitigation:** Product/monetisation defines participating placements. Ad operations confirms configuration. GSL validates against the live setup during verification.

---

### Platform implementation divergence

**Risk:** Unity, Android, and iOS teams implement the integration independently with subtle behavioural differences — different init timing, missing events on one platform, or inconsistent holdout logic.

**Impact:** Verification passes on one platform and fails on another. Post-launch behaviour is inconsistent.

**Mitigation:** One shared monetisation policy with platform-specific adapters only where the host stack requires it. Use the platform parity table in the integration spec as a review checklist for each build.

---

### Scope creep during verification

**Risk:** Teams treat GSL verification as broad QA and delay submission while fixing unrelated gameplay or UX issues.

**Impact:** Integration timeline extends. GSL review is blocked on non-integration defects.

**Mitigation:** Separate GSL integration verification (integration correctness) from client parallel QA (placement behaviour, stability). GSL does not evaluate gameplay quality.

---


## What's next

{% content-ref url="bid-floor-integration.md" %}
[bid-floor-integration.md](bid-floor-integration.md)
{% endcontent-ref %}

{% content-ref url="bid-floor-integration-testing.md" %}
[bid-floor-integration-testing.md](bid-floor-integration-testing.md)
{% endcontent-ref %}
