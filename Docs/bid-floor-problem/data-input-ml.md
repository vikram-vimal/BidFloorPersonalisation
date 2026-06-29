# ML Model Input Features

This page defines the derived feature set passed into the GSL bid floor ML model — one row per ad interval, built from raw SDK events collected during integration.

The schema below is scoped to **interstitial** ads: request counters, revenue counters, and interval outcomes all refer to interstitial load and impression events. Cross-format context (`avg_banner_ad_revenue`, `avg_rewarded_revenue`) is included as supplementary player-level signals. The same feature structure can be replicated for **rewarded** and **banner** ads by swapping the primary ad format in event filters and counters while keeping the column definitions unchanged.

| Column | Description |
| --- | --- |
| **player_id** | Unique player identifier. |
| **ad_interval_id** | One ad attempt per player, in time order. |
| **session_id** | Session UUID for the ad load request. |
| **ad_request_timestamp_gmt** | Ad load request time in GMT. |
| **request_hour_gmt** | Hour of day (0–23) in GMT. |
| **weekend_gmt** | `1` if Saturday or Sunday in GMT, else `0`. |
| **ad_request_timestamp_local** | Ad load request time converted to the user's local timezone. |
| **request_hour_local** | Hour of day (0–23) in the user's local timezone. |
| **weekend_local** | `1` if Saturday or Sunday in the user's local timezone, else `0`. |
| **timezone** | User timezone used for local time correction (e.g. `America/New_York`, `Asia/Kolkata`). |
| **floor** | Active bid floor at request time. |
| **ad_request_number_lifetime** | Lifetime interstitial load-request count for the player at request time. |
| **ad_request_number_session** | Interstitial load-request count within the current session at request time. |
| **ad_revenue_number_lifetime** | Count of interstitial revenue events for the player up to this interval. |
| **ad_revenue_number_session** | Count of interstitial revenue events within the current session up to this interval. |
| **session_depth** | Seconds since session start at request time. |
| **country_code** | ISO country code for the user. |
| **days_since_install** | Days between player install and ad request. |
| **gap_days** | Days since the previous session start. |
| **session_number** | Ordinal session count for the player. |
| **device_os** | Device platform: `Android`, `iOS`, or `other`. |
| **device_ram** | Device RAM in GB. |
| **device_age** | Device model age in years since release. |
| **device_price** | Estimated device retail price in USD. |
| **avg_banner_ad_revenue** | Player's mean banner eCPM (revenue × 1000) strictly before this interval. |
| **avg_rewarded_revenue** | Player's mean rewarded eCPM (revenue × 1000) strictly before this interval. |
| **ad_latency** | Seconds from ad load request to load success or load fail. |
| **ad_watch_time** | Seconds from ad watch start to ad watch success. Null when no successful watch. |
| **level_number_watched** | Game level at ad watch. Null when no watch event. |
| **placement** | In-game ad placement (e.g. `level_complete`). |
| **ad_network** | Winning ad network on revenue. Null when no revenue. |
| **error_message_type** | Load failure category: `floor_fill`, `other`, or null when load did not fail. |
