# Tandem — Data Model (Draft)

## Core Entities

### User
```
id
name
display_name
personal_multiplier        # effort scalar (e.g. 0.75 for D, 1.25 for A)
fitness_tier               # Beginner / Intermediate / Active / Athlete
onboarding_complete
created_at
```

### Plan
```
id
user_id
name
description
status                     # active / on_hold / retired
frequency_target           # sessions per week
activity_types[]           # qualifying activity types
tags[]                     # cardio, strength, flexibility, etc.
min_duration_minutes       # minimum qualifying session length
progression_enabled        # boolean
notes                      # private context (back issues, etc.)
created_at
```

### Activity
```
id
user_id
plan_id                    # optional — which plan this contributes to
activity_type              # bike_commute, peloton, yoga, walk, strength, etc.
tags[]                     # inherited from activity type + plan
duration_minutes
effort_points              # duration × personal_multiplier × activity_multiplier
intensity_override         # Easy / Moderate / Hard (optional self-report)
back_comfort               # Good / Neutral / Discomfort (optional)
pain_day                   # boolean
source                     # manual / strava / health_connect / peloton
logged_at
created_at
```

### Quest
```
id
title
description
type                       # individual / shared / parallel / milestone
target_value               # effort points or session count
target_unit                # points / sessions
tags[]                     # which activity tags contribute
window_days                # quest duration
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
recovery_window_expires    # nullable — set when recovery window opens
status                     # active / recovery / broken
updated_at
```
