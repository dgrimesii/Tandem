# Tandem — Architecture

## Deployment Overview

```
[Pixel 8 Pro]  [Fold 10]             (Android clients)
      |              |
      +----Tailscale VPN---------+
                                 |
                         [Raspberry Pi]        (home network)
                         ├── PWA server (React build)
                         ├── API server (Node/FastAPI)
                         ├── SQLite database
                         ├── Sync engine (Strava, Health Connect)
                         └── AI orchestration layer
                                 |
                         [External APIs]
                         ├── Strava API
                         ├── Google Health Connect
                         ├── Peloton (unofficial)
                         ├── Gemini Pro API
                         └── Anthropic Claude API
```

---

## Networking

**Tailscale** connects both phones to the Pi from anywhere. No port forwarding, no dynamic DNS, no security exposure. Both phones and the Pi join the same Tailscale network.

---

## AI Layer

| Task | Model | Reason |
|---|---|---|
| Quest suggestion engine | Gemini Pro | Structured output, Google ecosystem |
| Activity classification | Gemini Pro | Fast, cheap, structured |
| Motivational copy | Claude API | Nuanced, emotionally calibrated language |
| Weekly wins summary | Claude API | Tone sensitivity — Amy-specific language |
| Streak risk messaging | Claude API | High-stakes copy, warm and non-critical |
| Back comfort pattern analysis | Either | Simple correlation task |

---

## Fitness Integrations

| Platform | API Type | Priority | Notes |
|---|---|---|---|
| Strava | Official OAuth + Webhooks | Phase 2 | Cleanest integration, real-time |
| Google Health Connect | Android SDK | Phase 4 | Requires native or WebView wrapper |
| Peloton | Unofficial | Phase 4 | Check if data flows to Health Connect first |

---

## CI/CD

GitHub Actions deploys to Pi on push to `main`. Pi runs a pull-and-restart script via SSH over Tailscale.
