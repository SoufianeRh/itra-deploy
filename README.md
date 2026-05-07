<div align="center">

<img src="docs/screenshots/logo.png" alt="Itra Logo" width="120" />

# Itra — Moroccan Craftsmen Marketplace

### A production-ready, full-stack platform connecting craftsmen with customers across Morocco

[![Live Platform](https://img.shields.io/badge/🌍_Live-itra.ma-8B5E2A?style=for-the-badge)](https://itra.ma)
[![Android APK](https://img.shields.io/badge/📱_Android-APK_Download-3DDC84?style=for-the-badge)](https://itra.ma/downloads/itra-app.apk)
[![React 19](https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev)
[![Quarkus](https://img.shields.io/badge/Quarkus-3.22-4695EB?style=for-the-badge&logo=quarkus&logoColor=white)](https://quarkus.io)
[![Java 21](https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://openjdk.org)
[![PostgreSQL 17](https://img.shields.io/badge/PostgreSQL-17+pgvector-316192?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgresql.org)

> **Built entirely solo over ~8 months. Production-deployed. Real users.**

</div>

---

## What is Itra?

Itra is a **MyHammer / Thumbtack-style marketplace** for Morocco — customers post projects, craftsmen submit bids. Think of it as the professional network that Morocco's construction and service industry has been missing.

**Why is this technically interesting?**

- Moroccan Arabic (Darija) has almost **no NLP tooling** — so I built a custom multilingual search engine from scratch that understands Darija slang, Franco-Arab (Arabizi), and mixes all four languages.
- Full platform: web app, Android app, backend API, AI server — all maintained by one developer.
- Production-running at [itra.ma](https://itra.ma) with real users and data.

---

## Screenshots

| Home / Services | Project Wizard | Craftsman Profile |
|:---:|:---:|:---:|
| ![Services](docs/screenshots/services.png) | ![Wizard](docs/screenshots/wizard.png) | ![Profile](docs/screenshots/craftsman.png) |

| Admin Dashboard | Real-time Chat | Mobile App |
|:---:|:---:|:---:|
| ![Admin](docs/screenshots/admin.png) | ![Chat](docs/screenshots/messages.png) | ![Mobile](docs/screenshots/mobile.png) |

---

## Key Features

### 🔍 4-Language Hybrid Search Engine
Supports French, Arabic, Moroccan Darija, and Franco-Arab (Arabizi like *"kahrabaji"* → *"كهربائي"*). Built from scratch — no off-the-shelf library does this.

**How it works:**
- **Layer 1** — Stop-word filter (removes filler phrases like *"bghit"*, *"je cherche"*, *"ana bgha"*)
- **Layer 2** — Per-token fuzzy search with Arabic normalization, broken plural mapping, and digit-to-Arabic transliteration
- **Layer 3** — Automatic fallback to AI semantic search (pgvector + Snowflake Arctic Embed) when fuzzy results are insufficient
- Final results ranked by match type: exact → prefix → fuzzy → semantic

---

### 🧙 Project Wizard — Smart Service-Aware Skill Resolution
A multi-step guided wizard for customers to post projects. 29 questions are shared across 306 services — the same question about "vehicle type" means different things depending on context (Transport vs. Bicycle Workshop).

The `ServiceSkillResolver` maps `(serviceId, selectedOptions)` → skill assignments at wizard completion, not at question-definition time. This keeps the question bank DRY while supporting complex many-to-many service/skill logic.

---

### 🔔 Real-time Notifications via SSE (without WebSocket)
**The challenge:** The browser's `EventSource` API cannot send `Authorization` headers. Putting JWT tokens in URLs is a security risk (browser history, server logs).

**The solution:** A one-time SSE ticket system:
1. Client authenticates and gets a short-lived (30s) Redis ticket
2. Client opens the SSE stream using only the ticket — no JWT in the URL
3. Backend validates and consumes the ticket (single-use), then opens the persistent stream

---

### 🔐 Custom HMAC-JWT Auth — No Keycloak, No Auth0
Built a complete authentication system from scratch:
- Dual JWT secrets for separate User and Admin auth realms
- Refresh token rotation with **family tracking** (if a stolen token is used, the whole family is invalidated)
- HttpOnly cookie for the refresh token (invisible to JavaScript)
- Axios interceptor with request queue — prevents parallel refresh races when multiple requests expire simultaneously
- Social auth: Google, Facebook, Apple OAuth
- Session inactivity timeout with warning dialog

---

### 🗺️ Geospatial Search with PostGIS
- Customers can filter projects by city and radius (e.g., "show all plumbers within 25 km")
- Backend uses `ST_DWithin` for database-level geo filtering — no Java Haversine loops
- GiST indexes on geometry columns for sub-50ms geo queries
- GeoIP auto-detection for city pre-selection

---

### 📊 Admin Dashboard — 25 Tabs
A complete backoffice for managing the entire platform:

| Section | What it does |
|:---|:---|
| Overview | Live stats, user growth charts, revenue tracking |
| Users | Full CRUD, bulk actions, soft-delete with trash/restore |
| Services | 306 services, skill assignment, bulk operations |
| Skills | 96 skills across categories |
| Wizard CMS | Manage all 29 wizard questions and options — without code deploys |
| Translations | In-app translation editor for all 4 languages — without code deploys |
| Verification | Review craftsman identity and credential documents |
| Disputes | Mediation workflow with resolution tracking |
| Notifications | Push notifications to users or segments |
| Monitoring | System health, Redis stats, cache hit rates, error rates |

---

### 📱 Mobile App (React Native + Expo)
Full feature parity with the web app, built with the latest React Native stack:
- **New Architecture** (Fabric Renderer) + Hermes Engine + **React Compiler**
- **FlashList** instead of FlatList — up to 10× faster rendering for large lists
- **MMKV** instead of AsyncStorage — up to 10× faster storage reads
- Firebase Cloud Messaging for push notifications
- Google Sign-In native integration

---

## Tech Stack

### Backend

| Layer | Technology |
|:---|:---|
| Framework | **Quarkus 3.22** (Java 21), Hibernate Panache ORM |
| Database | **PostgreSQL 17** + pgvector (AI search) + PostGIS (geo) + Flyway migrations |
| Cache | **Redis 7** — sessions, SSE tickets, rate-limiting, API cache |
| Realtime | **Server-Sent Events (SSE)** with one-time ticket auth |
| Auth | Custom HMAC-SHA256 JWT + Google / Facebook / Apple OAuth |
| SQL | **jOOQ 3.19** — type-safe SQL for complex joins and batch operations |
| Email | Quarkus Mailer (SMTP SSL), async via CDI Events |
| GeoIP | MaxMind GeoLite2 (61 MB binary DB, 10k IP in-memory LRU cache) |
| Architecture | BCI: Boundary → Controller → Interactor → Repository |

### Frontend

| Layer | Technology |
|:---|:---|
| Framework | **React 19.1** + Vite 7.3, React Compiler |
| UI | **TailwindCSS 4** + DaisyUI 5.5 + Framer Motion animations |
| Data | **TanStack React Query 5.96** — stale-while-revalidate caching |
| Routing | React Router v7 |
| i18n | i18next — 4 languages, RTL auto-switch for Arabic |
| Charts | Recharts 3.7 (Admin analytics) |
| Search | Fuse.js (client-side fuzzy) + backend pgvector semantic fallback |

### AI / Embedding Server

| Layer | Technology |
|:---|:---|
| Embeddings | ONNX Runtime + **Snowflake Arctic Embed L** (1024-dim, ~31 ms/query) |
| Darija translation | Atlas-Chat 2B (via Ollama) |
| French / English | qwen3:1.7b (via Ollama) |
| Arabic | NLLB-200 (via CTranslate2) |
| API | **FastAPI** (Python 3.11), API-Key authentication |

### Mobile

| Layer | Technology |
|:---|:---|
| Framework | **React Native 0.83** + Expo 55, TypeScript, React 19.2 |
| Navigation | Expo Router (file-based routing) |
| Rendering | Hermes Engine + **New Architecture (Fabric)** |
| Lists | **FlashList 2.0** (up to 10× faster than FlatList) |
| Storage | **MMKV 4.3** (up to 10× faster than AsyncStorage) |
| Push | Firebase Cloud Messaging + expo-notifications |

### Infrastructure

| Layer | Technology |
|:---|:---|
| Hosting | **Plesk** (Nginx reverse proxy), Let's Encrypt SSL |
| Containers | **Docker Compose** (multi-stage builds, non-root containers) |
| CI/CD | **GitHub Actions** — backend build, frontend lint+build, secrets check |
| Monitoring | **Prometheus 2.51 + Grafana 10.4** |
| APK distribution | Gradle build → self-hosted APK at itra.ma |

---

## Architecture

```
                    ┌─────────────────────────────┐
                    │   Nginx / Plesk  (SSL/TLS)   │
                    │   itra.ma  — Let's Encrypt   │
                    └────────┬──────────┬──────────┘
                             │          │
                  ┌──────────▼──┐  ┌───▼──────────────┐
                  │  React 19   │  │  Quarkus Java 21  │
                  │  (Vite SPA) │  │  :8080 internal   │
                  └─────────────┘  └────────┬──────────┘
                                            │
                              ┌─────────────┼──────────────┐
                              │             │              │
                     ┌────────▼──────┐ ┌───▼──────┐ ┌────▼──────┐
                     │ PostgreSQL 17 │ │ Redis 7  │ │  Mailer   │
                     │  + pgvector  │ │ Sessions │ │  (SMTP)   │
                     │  + PostGIS   │ │ SSE keys │ │           │
                     │  + Flyway    │ │ Rate lim │ └───────────┘
                     └───────────────┘ └──────────┘
                              │
                    ┌─────────▼──────────────────────┐
                    │   AI / Embedding Server (S2)   │
                    │   FastAPI + ONNX + Ollama       │
                    │   Darija / FR / EN / AR models  │
                    └────────────────────────────────┘

  📱 Android APK  ──────────────────────────────────────────────────────►
  React Native + Expo 55, New Architecture, FlashList, MMKV, FCM push
```

---

## Security Highlights

- Backend runs on `127.0.0.1:8080` — **never directly exposed** to the internet
- Nginx enforces **HSTS** (1 year + preload), CSP, X-Frame-Options, X-Content-Type-Options
- **CSRF protection** — Origin/Referer validation filter on all state-changing requests
- **Refresh token family tracking** — stolen token use triggers full family invalidation
- File uploads — path traversal protection, MIME type detection, SVG blocked (XSS risk)
- Docker containers run as **non-root UID 10000**
- Redis — FLUSHDB / FLUSHALL / CONFIG / DEBUG commands disabled, requirepass enforced
- **`deny-unannotated-endpoints=true`** — every endpoint requires an explicit auth annotation
- All secrets via environment variables — no hardcoded credentials anywhere
- Security validated against **OWASP Top 10** checklist

---

## Roadmap

### ✅ Completed

| Feature | Details |
|:---|:---|
| FlashList (mobile) | FlatList replaced in 10 screens |
| MMKV (mobile) | AsyncStorage replaced in 7 files |
| TanStack React Query | Stale-while-revalidate caching on web |
| React Compiler | Auto-memoization enabled on web |
| PostGIS geo search | `ST_DWithin` + GiST index, replaces Java Haversine |
| Redis cache | `@CacheResult` on Services and Skills endpoints |
| Prometheus + Grafana | JVM, DB pool, Redis monitoring |
| Unit + Integration tests | GeoServiceTest, ProjectLocationIT |
| Full CI/CD pipeline | GitHub Actions: build, lint, secrets check |
| Secrets hygiene | `.env.example` template, all credentials via ENV |

### 🟡 In Progress

| Feature | Details |
|:---|:---|
| SSE refactoring | Migrate to `@Context SseEventSink` per connection with clean lifecycle |
| Redis cache stampede lock | `SETNX` pattern before cache rebuild for top-list endpoints |
| GitHub Actions integration tests | Spin up PostGIS in CI for full DB-level test coverage |

### 🔵 Planned

| Feature | Details |
|:---|:---|
| Quarkus Reactive (Mutiny) | Full async stack — worthwhile at >10k concurrent users |
| GraalVM Native Image | Sub-50ms cold start — pending Hibernate reflection compatibility work |
| Read replicas | PostgreSQL streaming replication, auto-routed reads |
| E2E testing (Playwright) | Full user journey tests for wizard + bidding flow |
| Stripe payment integration | In-app escrow for project payments |
| iOS App | Expo EAS build + App Store submission |

---

## Why This Project?

This is not a tutorial clone. Every technical decision here came from a real constraint:

- **Darija search** — No library does this. I built one.
- **SSE auth without headers** — Browsers don't allow it. I invented a Redis ticket system.
- **Service-aware skill resolution** — 306 services × 29 questions × context-dependent answers. I built a resolver graph.
- **4-language JWT dual-realm auth** — I didn't want Keycloak dependency for a startup. I built a secure HMAC system with refresh token theft detection.

Everything here is production-running, user-tested, and maintained by a single developer.

---

<div align="center">

**[🌍 Live Platform](https://itra.ma)** &nbsp;·&nbsp; **[📱 Download APK](https://itra.ma/downloads/itra-app.apk)**

---

*Full-stack development by [Amir](https://github.com/AmirItra)*  
*Built in Morocco 🇲🇦 — for Morocco*

</div>
