# Tandem 🚴‍♂️🚴‍♀️

> A shared fitness and longevity companion for two.

Tandem is a couples fitness app built around the idea that two people can pursue their own individual goals while working toward shared ones. It draws inspiration from Duolingo's Friends Quests model — collaborative, not competitive — and is designed from the ground up for consistency over performance, encouragement over criticism, and the long game of aging well together.

---

## The Why

Tandem exists for two reasons:

1. **Right now** — support an active weight loss journey with motivation, habit-building, and shared accountability
2. **Long game** — build the physical foundation (muscle mass, balance, endurance, mobility) that protects quality of life in the decades ahead

---

## Core Concepts

### Personal Plans
Each person has their own set of plans reflecting individual goals, physical constraints, and preferred activities. Plans are private but their completions contribute to shared outcomes.

### Activity Tagging
Every logged activity carries tags (e.g. `cardio`, `strength`, `flexibility`). Quests are defined against tags — not specific activities — so a bike commute and a Peloton ride both contribute to the same cardio quest.

### Effort Points
Raw activity minutes are converted to effort points that reflect how hard a session actually was — for that person, on that day. Effort level resolves through a three-priority hierarchy: an explicit user rating (Easy / Moderate / Hard / Maximum) takes precedence; heart rate data from a monitor produces a precise biometric-based value; and when neither is available, an estimate is derived from the activity type, duration, and the user's fitness baseline. A personal multiplier accounts for differences in fitness level between users, so contributions to shared quests feel genuinely equitable.

### Activity Suitability
Each activity type carries objective stress impact data across multiple body systems (knee, lumbar spine, hip, ankle/foot, and others). Each user's active physical conditions are stored separately in their profile. These two facts are joined at query time to determine whether an activity is suitable, flagged, or to be avoided for that specific person today — without hardcoding any user's concerns into the activity library itself.

### Quests
Time-bound goals that can be individual, shared (both contribute to one pool), parallel (same goal completed independently), or long-arc milestones. Quests are always winnable and always forward-looking.

### Streaks
Two-tier streak system: a daily Activity Streak (with Streak Shields earned through consistent weeks) and a weekly Journey Streak that reflects the longer pattern. One bad day never ends the story.

### Quest Awareness Engine
Proactive pace tracking for every active quest. Surfaces risk state (On Track / Heads Up / At Risk / Critical) before a window closes — not after. Projections use each user's planned activity schedule, not just historical pace.

---

## Five Pillars

| Pillar | What it owns |
|---|---|
| **Identity** | Who you are — profile, fitness tier, physical conditions, heart rate baseline, partnership |
| **Library** | What you can do — activity catalog, tags, body system impacts, classification rules |
| **Record** | What you did — activity logging, biometrics, effort calculation, comfort signals |
| **Plans** | Where you're going — weekly targets, progressions, weather fallbacks, plan intent |
| **Momentum** | What keeps you going — quests, streaks, milestones, awareness engine, celebrations |

See `docs/design/pillars.md` for full definitions and `docs/design/effort-calculation.md` for the effort point specification.

---

## Users

| User | Goals | Primary Activities |
|---|---|---|
| D | Muscle retention, endurance rebuild, back-safe routines | Bike commute, strength training, TBD endurance |
| A | Consistency habit, muscle base, balance/mobility, longevity | Peloton, yoga, walking |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React PWA |
| Backend | Node.js or FastAPI on Raspberry Pi |
| Database | SQLite (PostgreSQL when needed) |
| Networking | Tailscale |
| Version Control | GitHub |
| CI/CD | GitHub Actions |
| AI Layer | Gemini Pro (data tasks) + Claude API (language/copy) |
| Fitness Integrations | Strava (Phase 2), Health Connect (Phase 4), Peloton (Phase 4) |

---

## Roadmap

| Phase | Focus |
|---|---|
| **Phase 1** | Profiles, Personal Plans, Activity Library, Quest Engine (manual logging), Streak System |
| **Phase 2** | Strava OAuth integration, automated sync |
| **Phase 3** | LLM layer — quest suggestions, weekly summaries, motivational copy |
| **Phase 4** | Peloton + Health Connect integration |
| **Phase 5** | Native Android wrapper if needed |

---

## Project Status

🟡 Pre-build — design and planning phase
