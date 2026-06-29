# GSL SmartFloors — Raw Data Collection

This document lists the **raw events and JSON payload fields** the client must emit through the GSL SDK to support bid floor optimisation.

It covers source data only. Derived features — interval IDs, outcome labels, latency calculations, error bucketing, rolling averages, hour/weekend flags, and forward-filled counters — are built downstream and are documented in [Data requirements](data-input-ml.md).

{% hint style="info" %}
**Prerequisite:** [Ad lifecycle](../ads-concept/ad-lifecycle.md) — event names and sequencing.
{% endhint %}

---

## Event inventory

| Event | Required | Purpose |
| --- | --- | --- |
| `new_user` | Yes | Player install timestamp |
| `session_start` | Yes | Session boundaries and session numbering |
| `ad_load_request` | Yes | Floor at request time, player/session context, request counters |
| `ad_load_success` | Yes | Load completion timestamp |
| `ad_load_fail` | Yes (on failure) | Load failure timestamp and error message |
| `ad_watch` | Yes (on show) | Watch start timestamp and game context |
| `ad_watch_success` | Yes (on completion) | Watch completion timestamp |
| `ad_watch_fail` | Yes (on failure) | Watch failure timestamp |
| `ad_revenue` | Yes (on monetised impression) | Revenue, network, placement, ad format |

All events must include the common envelope fields in [Common envelope](#common-envelope).

---

## Common envelope

Every event payload must include these top-level fields.

| Field | Type | Description |
| --- | --- | --- |
| `event_name` | string | Event type (e.g. `ad_load_request`) |
| `event_timestamp` | string (ISO 8601) | Wall-clock time the event occurred |
| `player_id` | string | Unique player identifier |
| `session_id` | string | Session UUID (from `session_start` or first event in session) |
| `device_os` | string | Raw platform value from the device (e.g. `Android`, `iOS`) |
| `country_code` | string | ISO country code (e.g. `US`) |
| `time_zone` | string | timezone name |
| `device_ram` | int | RAM size in GB |
| `device_make_model` | string | Device Model |

**Example envelope fragment:**

```json
{
  "event_name": "ad_load_request",
  "event_timestamp": "2026-05-20T18:42:11.000Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "ad_unit_id": "app2352235-interstitial-GSL",
  "device_make_model" : "iPhone 17",
  "device_RAM" : 4,
  "time_zone" : "Asia/Kolkata"
}
```

---

## Raw payload fields by event

### `new_user`

Emitted once per player at first app open / install.

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `install_timestamp` | string (ISO 8601) | `install_timestamp` | Time of first app open / install |

```json
{
  "event_name": "new_user",
  "event_timestamp": "2026-05-18T09:15:00.000Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "install_timestamp": "2026-05-18T09:15:00.000Z"
}
```

---

### `session_start`

Emitted at the start of each player session.

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `session_number` | integer | `session_number` | Ordinal session count for the player |
| `tot_sessions` | integer | `tot_sessions` | Total sessions to date (fallback if `session_number` unavailable) |

```json
{
  "event_name": "session_start",
  "event_timestamp": "2026-05-20T18:30:00.000Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "session_number": 7,
  "tot_sessions": 7
}
```

---

### `ad_load_request`

Emitted when the game requests an interstitial ad. One row per load attempt.

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `floor` | number | `floor` | Active bid floor at request time (e.g. `0`, `6`, `8`, `9`, `10`) |
| `ad_request_number_lifetime` | integer | `ad_request_number_lifetime` | Lifetime interstitial load-request count for the player |
| `ad_request_number_session` | integer | `ad_request_number_session` | Load-request count within the current session |
| `time_since_session_start` | number | `time_since_session_start` | Seconds elapsed since `session_start` |
| `ad_format` | string | `ad_format` | Ad format requested (e.g. `interstitial`) |
| `ad_unit_id` | string | `ad_unit_id` | Mediation ad unit identifier |
| `placement` | string | `placement` | In-game placement triggering the request (e.g. `level_complete`) |

```json
{
  "event_name": "ad_load_request",
  "event_timestamp": "2026-05-20T18:42:11.000Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "floor": 8,
  "ad_request_number_lifetime": 48,
  "ad_request_number_session": 3,
  "time_since_session_start": 731,
  "ad_format": "interstitial",
  "ad_unit_id": "max_interstitial_001",
  "placement": "level_complete"
}
```

---

### `ad_load_success`

Emitted when the ad creative loads successfully.

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `ad_format` | string | `ad_format` | Ad format loaded |
| `ad_unit_id` | string | `ad_unit_id` | Mediation ad unit identifier |

```json
{
  "event_name": "ad_load_success",
  "event_timestamp": "2026-05-20T18:42:14.800Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "ad_format": "interstitial",
  "ad_unit_id": "max_interstitial_001"
}
```

---

### `ad_load_fail`

Emitted when the ad fails to load.

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `error_message` | string | `error_message` | Raw error string from the mediation SDK |
| `ad_format` | string | `ad_format` | Ad format requested |
| `ad_unit_id` | string | `ad_unit_id` | Mediation ad unit identifier |

```json
{
  "event_name": "ad_load_fail",
  "event_timestamp": "2026-05-20T18:42:13.500Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "ad_format": "interstitial",
  "ad_unit_id": "max_interstitial_001",
  "error_message": "No ads meet eCPM floor. Unit ID: max_interstitial_001"
}
```

---

### `ad_watch`

Emitted when the ad begins playing / is shown to the user.

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `level_number` | integer | `level_number` | Game level at time of ad show |
| `ad_format` | string | `ad_format` | Ad format shown |
| `placement` | string | `placement` | In-game placement (e.g. `level_complete`) |

```json
{
  "event_name": "ad_watch",
  "event_timestamp": "2026-05-20T18:42:20.000Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "ad_format": "interstitial",
  "placement": "level_complete",
  "level_number": 42
}
```

---

### `ad_watch_success`

Emitted when the user completes the ad (or minimum view threshold is met).

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `ad_format` | string | `ad_format` | Ad format watched |
| `placement` | string | `placement` | In-game placement |

```json
{
  "event_name": "ad_watch_success",
  "event_timestamp": "2026-05-20T18:42:35.000Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "ad_format": "interstitial",
  "placement": "level_complete"
}
```

---

### `ad_watch_fail`

Emitted when the ad show fails or the user dismisses before completion.

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `ad_format` | string | `ad_format` | Ad format |
| `placement` | string | `placement` | In-game placement |

```json
{
  "event_name": "ad_watch_fail",
  "event_timestamp": "2026-05-20T18:42:25.000Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "ad_format": "interstitial",
  "placement": "level_complete"
}
```

---

### `ad_revenue`

Emitted when a monetised impression is recorded. Required for interstitial optimisation; banner and rewarded revenue events provide cross-format context.

| Field | Type | JSON path | Description |
| --- | --- | --- | --- |
| `revenue` | number | `revenue` | Raw impression revenue in USD (not scaled) |
| `ad_network` | string | `ad_network` | Winning ad network (e.g. `Unity (Android mediation)`) |
| `placement` | string | `placement` | In-game placement (e.g. `level_complete`) |
| `ad_format` | string | `ad_format` | Ad format (`interstitial`, `banner`, `rewarded`) |
| `ad_unit_id` | string | `ad_unit_id` | Mediation ad unit identifier |
| `currency` | string | `currency` | Revenue currency code (e.g. `USD`) |

**Interstitial example:**

```json
{
  "event_name": "ad_revenue",
  "event_timestamp": "2026-05-20T18:42:35.100Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "ad_format": "interstitial",
  "ad_unit_id": "max_interstitial_001",
  "placement": "level_complete",
  "ad_network": "Unity (Android mediation)",
  "revenue": 0.0229,
  "currency": "USD"
}
```

**Banner / rewarded example (same schema, different `ad_format`):**

```json
{
  "event_name": "ad_revenue",
  "event_timestamp": "2026-05-20T18:40:00.000Z",
  "player_id": "usr_8f3a2b1c",
  "session_id": "sess_4e91d7a0-3c2f-4b1a-9e8d-7f6a5b4c3d2e",
  "device_os": "Android",
  "country_code": "US",
  "ad_format": "rewarded",
  "ad_unit_id": "max_rewarded_001",
  "placement": "extra_lives",
  "ad_network": "AdMob (Android)",
  "revenue": 0.0450,
  "currency": "USD"
}
```

---

## Master reference — raw fields only

Single table mapping every raw collection field to its source event and JSON path.

| Raw field | Source event | JSON path | Notes |
| --- | --- | --- | --- |
| `player_id` | All events | `player_id` | Common envelope |
| `session_id` | All events | `session_id` | Common envelope; originates from `session_start` |
| `event_timestamp` | All events | `event_timestamp` | Per-event wall-clock time |
| `device_os` | All events | `device_os` | Raw platform string from device |
| `country_code` | All events | `country_code` | ISO country code |
| `install_timestamp` | `new_user` | `install_timestamp` | Player install time |
| `session_number` | `session_start` | `session_number` | Primary session counter |
| `tot_sessions` | `session_start` | `tot_sessions` | Fallback session counter |
| `floor` | `ad_load_request` | `floor` | Active bid floor at request time |
| `ad_request_number_lifetime` | `ad_load_request` | `ad_request_number_lifetime` | Lifetime interstitial request count |
| `ad_request_number_session` | `ad_load_request` | `ad_request_number_session` | Session interstitial request count |
| `time_since_session_start` | `ad_load_request` | `time_since_session_start` | Seconds since session start |
| `ad_format` | Ad lifecycle events | `ad_format` | Format identifier |
| `ad_unit_id` | `ad_load_request`, `ad_revenue` | `ad_unit_id` | Mediation ad unit ID |
| `placement` | `ad_load_request`, `ad_watch`, `ad_revenue` | `placement` | In-game placement name |
| `error_message` | `ad_load_fail` | `error_message` | Raw SDK error string |
| `level_number` | `ad_watch` | `level_number` | Game level at ad show |
| `revenue` | `ad_revenue` | `revenue` | Raw impression revenue (USD) |
| `ad_network` | `ad_revenue` | `ad_network` | Winning network name |
| `currency` | `ad_revenue` | `currency` | Revenue currency code |

---

## Explicitly out of scope (derived downstream)

The following fields from [Data requirements](data-input-ml.md) are **not** collected at source. They are computed by GSL during feature engineering:

| Derived field | How it is built |
| --- | --- |
| `ad_interval_id` | `{player_id}_{sequence}` via `add_ad_interval_id()` |
| `ad_request_timestamp` | Alias of `ad_load_request.event_timestamp` (timezone-normalised downstream) |
| `request_hour` | Extracted from `event_timestamp` |
| `weekend` | Derived from `event_timestamp` (ET, Sat/Sun flag) |
| `result` | Interval outcome via `_interval_result()` state machine |
| `ad_revenue_number_lifetime` | Forward-filled count of revenue events per player |
| `ad_revenue_number_session` | Forward-filled count of revenue events per session |
| `session_depth` | Alias of `time_since_session_start` |
| `days_since_install` | `ad_load_request.event_timestamp` − `install_timestamp` |
| `gap_days` | Days between consecutive `session_start` events |
| `ad_latency` | `ad_load_success` or `ad_load_fail` timestamp − `ad_load_request` timestamp |
| `ad_watch_time` | `ad_watch_success` timestamp − `ad_watch` timestamp |
| `level_number_watched` | Alias of `ad_watch.level_number` |
| `ad_revenue` (eCPM) | `revenue` × 1000 |
| `avg_banner_ad_revenue` | Expanding mean of prior banner `ad_revenue` events |
| `avg_rewarded_revenue` | Expanding mean of prior rewarded `ad_revenue` events |
| `error_message_type` | Bucketed via `classify_error_message()` |

---

## Related pages

{% content-ref url="data-input-ml.md" %}
[data-input-ml.md](data-input-ml.md)
{% endcontent-ref %}

{% content-ref url="bid-floor-integration.md" %}
[bid-floor-integration.md](bid-floor-integration.md)
{% endcontent-ref %}
