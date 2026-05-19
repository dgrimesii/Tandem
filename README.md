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
Raw activity minutes are converted to effort points using personal and activity multipliers. This accounts for differences in baseline fitness so each person's contribution reflects their relative effort, not absolute output.

### Quests
Time-bound goals that can be individual, shared (both contribute to one pool), parallel (same goal completed independently), or long-arc milestones. Quests are always winnable and always forward-looking.

### Streaks
Two-tier streak system: a daily Activity Streak (with Streak Shields earned through consistent weeks) and a weekly Journey Streak that reflects the longer pattern. One bad day never ends the story.

### Quest Awareness Engine
Proactive pace tracking for every active quest. Surfaces risk state (On Track / Heads Up / At Risk / Critical) before a window closes — not after.

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
| **Phase 1** | Profiles, Personal Plans, Activity Tagging, Quest Engine (manual logging), Streak System |
| **Phase 2** | Strava OAuth integration, automated sync |
| **Phase 3** | LLM layer — quest suggestions, weekly summaries, motivational copy |
| **Phase 4** | Peloton + Health Connect integration |
| **Phase 5** | Native Android wrapper if needed |

---

## Project Status

🟡 Pre-build — design and planning phase
