# Tandem — Data Model

> This document reflects the current canonical data model. All entities are organized by pillar.

---

## Pillar: Identity

### User
```
id
name
display_name
fitness_tier               # Beginner / Intermediate / Active / Athlete
personal_multiplier        # effort scalar — invisible to users, set from fitness_tier
onboarding_complete
created_at
```

### UserHeartRateProfile
```
id
user_id
resting_heart_rate         # bpm — measured, not estimated
max_heart_rate             # bpm — measured or calculated (220 - age)
hr_zones                   # JSON — custom zone boundaries (optional)
source                     # manual / garmin / apple_health / health_connect / estimated
updated_at
```

### UserCondition
```
id
user_id
body_system                # see Body System Vocabulary below
concern_level              # monitor / caution / avoid
notes                      # private free text (e.g. "patellar tendinitis")
active                     # boolean — false when condition resolved
created_at
updated_at
```

**Body System Vocabulary:**
- `lumbar_spine` — lower back, herniated discs, sciatica
- `hip` — hip joint, referred pain from lumbar
- `knee` — knee joint, patellar tendon, meniscus
- `ankle_foot` — ankle joint, plantar fascia, Achilles
- `shoulder` — shoulder joint, rotator cuff
- `wrist_hand` — wrist, carpal tunnel
- `neck_cervical` — cervical spine, neck
- `cardiovascular` — heart rate stress (intensity awareness)
- `general_joint` — broad joint loading

### Partnership
```
id
user_id_a
user_id_b
created_at
```

### PartnershipSettings (per user)
```
id
partnership_id
user_id
show_partner_activity            # boolean, default false
allow_partner_quest_visibility   # boolean, default false
updated_at
```

### UserMultiplierHistory
```
id
user_id
previous_multiplier
new_multiplier
previous_fitness_tier
new_fitness_tier
reason                     # manual / milestone_triggered
created_at
```

---

## Pillar: Library

### ActivityType
```
id
name                       # "Resistance Band Circuit"
display_name               # "Resistance Bands"
category                   # "Strength"
broad_tags[]               # ["strength", "low-impact"]
subtype_tag                # "strength:functional"
flags[]                    # computed — see note below
intensity_default          # low / medium / high — estimation input only
min_qualifying_minutes     # minimum duration to count as a qualifying session
equipment_required[]       # ["resistance bands"]
suggested_for[]            # ["beginner", "intermediate"]
is_custom                  # boolean
created_at
```

**Note on flags:** `back-safe`, `knee-safe`, `pf-safe`, `indoor`, `outdoor`, `equipment-free`
Safety flags (`back-safe`, `knee-safe`, `pf-safe`) are **computed** from `body_system_impacts`
at query time — not manually set. Other flags (`indoor`, `outdoor`, `equipment-free`) are
stored directly on the ActivityType.

### ActivityBodySystemImpact
```
id
activity_type_id           → ActivityType
body_system                # see Body System Vocabulary in Identity
stress_level               # low / medium / high
notes                      # optional — "impact reduced with proper form"
```

One row per body system per activity type. A single activity can have impacts
across multiple body systems. This replaces the former `back_stress_level` field.

**Example — Running:**
```
knee:           high
ankle_foot:     medium
lumbar_spine:   medium
cardiovascular: high
```

**Example — Peloton:**
```
knee:           medium
lumbar_spine:   low
cardiovascular: high
```

### ActivitySuitability (runtime — not stored)
```
Computed at query time by joining:
  ActivityBodySystemImpact (Library)
  UserCondition (Identity — active conditions only)

Result per activity per user:
  suitable  — no overlapping active conditions
  flag      — overlap exists, stress_level low/medium
  caution   — overlap exists, stress_level high, concern_level monitor
  avoid     — overlap exists, stress_level high, concern_level avoid
```

---

## Pillar: Record

### Activity
```
id
user_id
activity_type_id           → ActivityType (Library)
plan_id                    → Plan (optional)
duration_minutes
effort_multiplier          # resolved value used in effort_points calculation
effort_source              # user_override / biometric / estimated
effort_points              # duration × personal_multiplier × effort_multiplier
intensity_override         # Easy / Moderate / Hard / Maximum (optional — user set)
body_comfort               # Good / Neutral / Discomfort (optional — replaces back_comfort)
body_system_flagged        # nullable — which body system the comfort signal refers to
pain_day                   # boolean
notes                      # free text (optional)
source                     # manual / strava / peloton / health_connect
logged_at
created_at
```

**Note on body_comfort:** Renamed from `back_comfort` to `body_comfort` to reflect
that either user may flag discomfort in any body system, not just the back.
`body_system_flagged` optionally identifies which system (from the Body System Vocabulary).

### ActivityBiometric
```
id
activity_id                → Activity
biometric_type             # see Biometric Type Vocabulary below
value                      # numeric
unit                       # "bpm", "steps", "kcal", "meters", "watts", etc.
aggregation                # avg / max / min / total — how value was derived
source                     # manual / heart_rate_monitor / strava / health_connect / peloton
recorded_at                # nullable — point-in-time vs. session aggregate
```

**Biometric Type Vocabulary:**
- `heart_rate_avg` — bpm, avg
- `heart_rate_max` — bpm, max
- `heart_rate_min` — bpm, min
- `heart_rate_zones` — % time in each zone (JSON value)
- `steps` — count, total
- `cadence_avg` — rpm/spm, avg
- `power_avg` — watts, avg (Peloton, cycling)
- `power_max` — watts, max
- `calories` — kcal, total
- `distance` — meters, total
- `elevation_gain` — meters, total
- `pace_avg` — sec/km, avg
- `spo2_avg` — %, avg (future)
- `hrv` — ms (future)

---

## Effort Level Resolution

Effort level resolves via a three-priority hierarchy. Always produces a value.
See `docs/design/effort-calculation.md` for full specification.

```
Priority 1 — USER OVERRIDE
  intensity_override set → maps to effort_multiplier directly
  Easy: 0.8x | Moderate: 1.0x | Hard: 1.2x | Maximum: 1.5x

Priority 2 — BIOMETRIC (heart rate)
  heart_rate_avg biometric available + UserHeartRateProfile populated
  HRR% = (HR_avg - HR_resting) / (HR_max - HR_resting) × 100
  Zone → multiplier: <50%: 0.7x | 50-60%: 0.85x | 60-70%: 1.0x |
                     70-80%: 1.2x | 80-90%: 1.4x | >90%: 1.6x

Priority 3 — ESTIMATED
  base = intensity_default mapped to 0.7–1.2
  duration_ratio = min(duration / 45, 1.4)
  tier_factor = Beginner:1.0 | Intermediate:0.95 | Active:0.9 | Athlete:0.85
  effort_multiplier = base × duration_ratio × tier_factor
```

**Final effort_points formula:**
```
effort_points = duration_minutes × personal_multiplier × effort_multiplier
```

---

## Pillar: Plans

### Plan
```
id
user_id
name
description
status                     # active / on_hold / retired
frequency_target           # sessions per week
typical_days[]             # ["monday", "wednesday", "friday"] — for projection accuracy
activity_types[]           # qualifying activity type IDs from Library
tags[]                     # cardio / strength / flexibility / etc.
min_duration_minutes       # minimum qualifying session duration
fallback_activity_type_id  # → ActivityType — indoor/weather contingency
progression_enabled        # boolean
notes                      # private context
created_at
```

### PlanProgressionLevel
```
id
plan_id
level_number               # 1, 2, 3...
frequency_target           # sessions per week at this level
description                # "Bodyweight only" / "Add light resistance" / etc.
activated_at               # nullable — when user stepped up to this level
```

### PlanEvent
```
id
plan_id
event_type                 # activated / retired / progressed / put_on_hold
notes
created_at
```

---

## Pillar: Momentum

### Quest
```
id
title
description
type                       # individual / shared / parallel / milestone
target_value               # effort points or session count
target_unit                # points / sessions
tags[]                     # which activity tags contribute
window_days
difficulty                 # easy / medium / hard
status                     # active / complete / missed / archived
starts_at
ends_at
created_at
```

### QuestProgress
```
id
quest_id
user_id
current_value
last_updated
risk_state                 # on_track / heads_up / at_risk / critical / complete / missed
```

### Streak
```
id
user_id
type                       # activity (daily) / journey (weekly)
current_count
best_count
shields_available          # 0–3
last_activity_date
recovery_window_expires    # nullable
status                     # active / recovery / broken
updated_at
```

### Milestone
```
id
title
description
type                       # individual / shared
condition_type             # activity_count / effort_total / streak_length /
                           # plan_activated / consistency_score / pain_trend
condition_value            # numeric threshold
user_id                    # nullable — null for shared milestones
earned_at                  # nullable — set when condition met
```

### Notification
```
id
user_id
type                       # quest_heads_up / quest_at_risk / quest_critical /
                           # streak_recovery / shield_consumed / partner_logged /
                           # summary_ready / quest_complete / milestone_earned
content                    # generated message
sent_at
read_at                    # nullable
```
