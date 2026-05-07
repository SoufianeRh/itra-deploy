<div align="center">

# Itra — Moroccan Craftsmen Marketplace

### A production-ready, full-stack platform connecting craftsmen with customers across Morocco

[![Live Platform](https://img.shields.io/badge/🌍_Live-itra.ma-8B5E2A?style=for-the-badge)](https://itra.ma)
[![Android APK](https://img.shields.io/badge/📱_Android-APK_Download-3DDC84?style=for-the-badge)](https://itra.ma/downloads/itra-app.apk)
[![React 19](https://img.shields.io/badge/React-19.1-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev)
[![Quarkus](https://img.shields.io/badge/Quarkus-3.22.3-4695EB?style=for-the-badge&logo=quarkus&logoColor=white)](https://quarkus.io)
[![Java 21](https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://openjdk.org)
[![PostgreSQL 17](https://img.shields.io/badge/PostgreSQL-17+pgvector+PostGIS-316192?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgresql.org)

> **Built entirely solo over ~8 months. Production-deployed. Real users.**

</div>

---

## What is Itra?

Itra is a **MyHammer / Thumbtack-style marketplace** for Morocco — customers post projects, craftsmen submit bids. Think of it as the professional network that Morocco's construction and service industry has been missing.

**Why is this technically interesting?**

- Moroccan Arabic (Darija) has almost **no NLP tooling** — so I built a custom multilingual search engine from scratch that understands Darija slang, Franco-Arab (Arabizi), and mixes all four languages.
- Full platform: web app, Android app, backend API, AI embedding server — all maintained by one developer.
- Production-running at [itra.ma](https://itra.ma) with real users and data.

---

---

## Key Features

### 🔍 4-Language Hybrid Search Engine
Supports French, Arabic, Moroccan Darija (ar-MA), and Franco-Arab (Arabizi like *"kahrabaji"* → *"كهربائي"*). Built from scratch — no off-the-shelf library does this.

**How it works:**
- **Layer 1** — Stop-word filter (removes filler phrases like *"bghit"*, *"je cherche"*, *"ana bgha"*)
- **Layer 2** — Per-token fuzzy search with Arabic normalization, broken plural mapping, and digit-to-Arabic transliteration (`pg_trgm` + `unaccent`)
- **Layer 3** — Automatic fallback to AI semantic search (pgvector + Snowflake Arctic Embed) when fuzzy results are insufficient
- Final results ranked by match type: exact → prefix → fuzzy → semantic

---

### 🧙 Project Wizard — Smart Service-Aware Skill Resolution
A multi-step guided wizard for customers to post projects. 29 questions are shared across 306 services in 35 subcategories — the same question about "vehicle type" means different things depending on context (Transport vs. Bicycle Workshop).

The `ServiceSkillResolver` maps `(serviceId, selectedOptions)` → skill assignments at wizard completion, not at question-definition time. This keeps the question bank DRY while supporting complex many-to-many service/skill logic.

---

### 🔔 Real-time Notifications via SSE (without WebSocket)
**The challenge:** The browser's `EventSource` API cannot send `Authorization` headers. Putting JWT tokens in URLs is a security risk (browser history, server logs).

**The solution:** A one-time SSE ticket system:
1. Client authenticates and gets a short-lived (30s) Redis ticket
2. Client opens the SSE stream using only the ticket — no JWT in the URL
3. Backend validates and consumes the ticket (single-use), then opens the persistent stream

Push notifications via **Firebase Cloud Messaging (FCM)** for mobile/background delivery.

---

### 🔐 Custom HMAC-JWT Auth — No Keycloak, No Auth0
Built a complete authentication system from scratch:
- **Dual JWT secrets** for separate User and Admin auth realms
- Refresh token rotation with **family tracking** (if a stolen token is used, the whole family is invalidated)
- **HttpOnly cookie** for the refresh token (invisible to JavaScript)
- Axios interceptor with request queue + **BroadcastChannel** — prevents parallel refresh races across browser tabs
- **Social auth:** Google, Facebook, Apple OAuth
- **TOTP 2FA** for additional account security
- Session inactivity timeout with warning dialog

---

### 🗺️ Geospatial Search with PostGIS
- Customers can filter projects by city and radius (e.g., "show all plumbers within 25 km")
- Backend uses `ST_DWithin` for database-level geo filtering — no Java Haversine loops
- **GiST indexes** on `geo_point` geometry columns for sub-50ms geo queries
- **GeoIP auto-detection** (MaxMind GeoLite2, 10,000-IP in-memory LRU cache) for city pre-selection

---

### 🤝 Subcontracting Marketplace (Craftsman-to-Craftsman)
A second marketplace layer where craftsmen post work they need to outsource:
- Separate `subcontracts` + `subcontract_bids` + `subcontract_photos` tables
- Full bid lifecycle: open → accepted → completed/cancelled
- Geospatial search for nearby subcontract opportunities
- Full admin management and moderation

---

### ⭐ Reviews, Portfolio & Disputes
- **Reviews** with photos, written responses from the craftsman, and verified-badge display
- **Portfolio system** — craftsmen showcase past work with photos per service category
- **Disputes** — structured mediation workflow: open → in review → resolved, with history tracking

---

### 📢 Ads & Monetization
A full ad management system for monetizing the platform:
- Multiple ad types: banners, inline list injections, sponsored cards, popups
- Zone-based delivery (`header-below`, `list-inline`, etc.) per page
- Impression and click tracking with daily stats aggregation
- Revenue models: free, CPM (per-mille), CPC (per-click), flat fee
- AdSense integration support
- Full admin CMS for creating/scheduling campaigns — no code deploy needed

---

### 📝 CMS System
- Content zones rendered on any page (announcements, banners, marketing blocks)
- Blog with multilingual articles
- All content manageable via admin dashboard without code deployments
- Announcement banners with scheduling

---

### 🛡️ Anti-Scraping & Rate Limiting
- IP-based rate limiting via Redis
- Dedicated admin tab for monitoring and managing scraping attempts
- Nginx blocks probes for `.env`, `.git`, `wp-admin`, and sensitive file extensions

---

### 📊 Admin Dashboard — 25+ Tabs, 3-Level Role System

Roles: **MODERATOR** → **ADMIN** → **SUPER_ADMIN**

| Section | What it does |
|:---|:---|
| Overview | Live stats, user growth charts, revenue tracking |
| Users | Full CRUD, bulk actions, soft-delete with trash/restore |
| Projects | Full project management with status lifecycle |
| Bids | Bid oversight across all projects |
| Subcontracts | Craftsman-to-craftsman subcontract management |
| Reviews | Moderation workflow with approval/rejection |
| Skills | 60+ skills with category assignment |
| Skill Categories | 11 skill categories |
| Services | 306 services, skill assignment, bulk operations |
| Wizard CMS | Manage all 29 wizard questions and options — without code deploys |
| Service Comparison | Side-by-side service/skill mapping review |
| Categories | Category and subcategory (35 subcategories) management |
| Translations | In-app translation editor for all 4 languages — without code deploys |
| CMS Content | Blog, announcements, content zones |
| Ads & Monetization | Campaign management, stats, revenue tracking |
| Verification | Review craftsman identity and credential documents |
| Disputes | Mediation workflow with resolution tracking |
| Admin Activity | Full audit log of all admin actions |
| User Activity | User behavior and session logs |
| Notifications | Push notifications to users or segments |
| Admin Messaging | Internal admin-to-admin and admin-to-user messaging via SSE |
| System Settings | Platform-wide config, Redis stats, cache hit rates |
| Anti-Scraping | IP monitoring and blocking |
| AI Search | Embedding pipeline config, threshold tuning, re-index triggers |
| Archive | Soft-deleted and archived records management |

---

### 📱 Mobile App (React Native + Expo)
Full feature parity with the web app, built with the latest React Native stack:
- **New Architecture** (Fabric Renderer) + Hermes Engine + **React Compiler**
- **FlashList 2.0** instead of FlatList — up to 10× faster rendering for large lists
- **MMKV 4.3** instead of AsyncStorage — up to 10× faster synchronous storage reads
- **Firebase Cloud Messaging** for push notifications
- **Voice Search** (expo-speech-recognition)
- **Dark / Light theme** with persistent preference
- **4-language RTL support** including Darija (ar-MA)
- Google Sign-In native integration
- Crash catcher with branded error screen
- CMS ad zones and sponsored cards integrated in lists

---

## Tech Stack

### Backend

| Layer | Technology |
|:---|:---|
| Framework | **Quarkus 3.22.3** (Java 21), Hibernate Panache ORM |
| Database | **PostgreSQL 17** + pgvector 1024-dim (AI search) + PostGIS (geo) + pg_trgm + unaccent |
| Migrations | **Flyway** — 29 versioned migrations (V1 baseline → V29) |
| Cache | **Redis 7** — sessions, SSE tickets, rate-limiting, API cache (`@CacheResult`) |
| Realtime | **Server-Sent Events (SSE)** with one-time Redis ticket auth |
| Push | **Firebase Cloud Messaging (FCM)** — mobile push notifications |
| Auth | Custom HMAC-SHA256 JWT + TOTP 2FA + Google / Facebook / Apple OAuth |
| SQL | **jOOQ 3.19.21** — 80 generated type-safe table classes for complex joins & batch ops |
| Email | Quarkus Mailer (SMTP SSL), async via CDI Events, project digest scheduler |
| GeoIP | MaxMind GeoLite2 (binary DB, 10k-IP in-memory LRU cache) |
| Architecture | BCI: Boundary → Controller → Interactor + Repository (26 domain modules) |
| Monitoring | Micrometer + **Prometheus** — JVM, DB pool, custom metrics |

### Frontend

| Layer | Technology |
|:---|:---|
| Framework | **React 19.1** + Vite 7.3.1, React Compiler (auto-memoization) |
| UI | **TailwindCSS 4.2.1** + DaisyUI 5.5.19 + Framer Motion 12.38 animations |
| Data | **TanStack React Query 5.96** — stale-while-revalidate caching |
| Routing | React Router v7, lazy-loaded with retry |
| i18n | i18next 25.8 — 4 languages (FR, EN, AR, AR-MA), RTL auto-switch, admin DB overrides |
| Charts | Recharts 3.7 (Admin analytics) |
| Search | Fuse.js (client-side fuzzy) + backend pgvector semantic fallback |
| State | Context API (AuthContext, AdminAuthContext), custom hooks (57+ states in useAdminDashboard) |

### AI / Embedding Server

| Layer | Technology |
|:---|:---|
| Model | **Snowflake Arctic Embed L** (1024-dim vectors) |
| Runtime | **ONNX Runtime INT8** — CPU inference, ~19ms/query, no PyTorch at inference time |
| Quantization | Auto-exported FP32 → INT8 dynamic quantization, pre-baked into Docker image |
| API | **FastAPI** (Python 3.11), API-Key authentication |
| Integration | Quarkus REST Client → embedding service, pgvector cosine similarity search |

### Mobile

| Layer | Technology |
|:---|:---|
| Framework | **React Native 0.83.6** + Expo 55.0.23, TypeScript, React 19.2 |
| Navigation | Expo Router 55.0.14 (file-based routing) |
| Rendering | Hermes Engine + **New Architecture (Fabric)** + React Compiler |
| Lists | **FlashList 2.0.2** (up to 10× faster than FlatList) |
| Storage | **MMKV 4.3** (up to 10× faster than AsyncStorage) |
| Push | Firebase Cloud Messaging 24 + expo-notifications |
| Voice | **expo-speech-recognition** for voice search |
| Animations | react-native-reanimated 4.2.1 + react-native-gesture-handler |

### Infrastructure

| Layer | Technology |
|:---|:---|
| Hosting | **Plesk** (Nginx reverse proxy, `127.0.0.1:8080` only), Let's Encrypt SSL |
| Containers | **Docker Compose** (multi-stage builds, non-root UID 10000) |
| DB Tuning | PostgreSQL tuned for 24GB server: 500 connections, 6GB shared_buffers, 16GB effective_cache |
| CI/CD | **GitHub Actions** — backend tests (19 test classes), frontend build, secrets check, OWASP dependency check |
| Monitoring | **Prometheus 2.51 + Grafana 10.4** — JVM, DB pool, Redis, custom metrics |
| APK distribution | Gradle build → self-hosted at itra.ma/downloads/itra-app.apk |

---

## Architecture

```
                    ┌─────────────────────────────┐
                    │   Nginx / Plesk  (SSL/TLS)   │
                    │   itra.ma  — Let's Encrypt   │
                    └────────┬──────────┬──────────┘
                             │          │
                  ┌──────────▼──┐  ┌───▼──────────────────┐
                  │  React 19   │  │  Quarkus 3.22 / JVM   │
                  │  Vite SPA   │  │  127.0.0.1:8080       │
                  └─────────────┘  └──────────┬────────────┘
                                              │
                              ┌───────────────┼──────────────────┐
                              │               │                  │
                     ┌────────▼──────┐ ┌──────▼───┐ ┌──────────▼──────┐
                     │ PostgreSQL 17 │ │ Redis 7  │ │  Quarkus Mailer │
                     │  + pgvector  │ │ Sessions │ │  (SMTP SSL)     │
                     │  + PostGIS   │ │ SSE keys │ └─────────────────┘
                     │  + pg_trgm   │ │ Rate lim │
                     │  + Flyway    │ │ API cache│
                     └───────┬───────┘ └──────────┘
                             │
                    ┌────────▼───────────────────────┐
                    │   AI / Embedding Server        │
                    │   FastAPI + ONNX Runtime INT8   │
                    │   Snowflake Arctic Embed L      │
                    │   1024-dim · ~19ms/query · CPU  │
                    └────────────────────────────────┘

  📱 Android APK ─────────────────────────────────────────────────────►
  React Native 0.83 + Expo 55, New Architecture, FlashList, MMKV, FCM
```

### Backend Domain Modules (26 total)
`admin` · `ads` · `archive` · `auth` · `bid` · `craftsman` · `customer` · `dispute` · `embedding` · `favorite` · `geo` · `message` · `notification` · `portfolio` · `project` · `question` · `review` · `scheduler` · `servicecatalog` · `skill` · `subcontract` · `trash` · `translation` · `user` · `shared`

---

## Security Highlights

- Backend binds to `127.0.0.1:8080` — **never directly exposed** to the internet
- Nginx enforces **HSTS** (1 year + preload), CSP with SHA-256 script hash, X-Frame-Options, X-Content-Type-Options
- **CSRF protection** — Origin/Referer validation filter on all state-changing requests
- **Refresh token family tracking** — stolen token use triggers full family invalidation
- File uploads — path traversal protection, MIME type detection, SVG blocked (XSS risk)
- Docker containers run as **non-root UID 10000**
- Redis — `FLUSHDB` / `FLUSHALL` / `CONFIG` / `DEBUG` disabled, requirepass enforced
- **`deny-unannotated-endpoints=true`** — every endpoint requires an explicit auth annotation
- All secrets via environment variables — `.env.example` template, no hardcoded credentials
- **Secrets scanner** (`scripts/check_secrets.sh`) runs in CI on every push
- OWASP Dependency Check runs on every merge to `main`
- Security validated against **OWASP Top 10**

---

## Scheduled Tasks

| Schedule | Task | What it does |
|:---|:---|:---|
| Daily 02:00 | TrashCleanupScheduler | Archives + permanently deletes trash items after retention period |
| Hourly | CancellationAutoApprovalScheduler | Auto-approves cancellation requests after 24h without a response |
| Daily 03:00 | AccountCleanupScheduler | Removes expired unverified accounts (cascaded delete) |
| Hourly | ProjectDigestScheduler | Sends daily/weekly digest emails to craftsmen about new projects in their radius |

---

## CI/CD Pipeline

**GitHub Actions** (3 jobs on every push):

| Job | Steps |
|:---|:---|
| `backend-tests` | Secrets check → PostGIS/Redis spin-up → JDK 21 → 19 test classes (Auth, Bids, Geo, Projects, …) |
| `frontend-build` | Node 20 → `npm install` → test suite → `npm run build` |
| `dependency-check` | OWASP check on merge to `main` |

---

## Roadmap

### ✅ Completed

| Feature | Details |
|:---|:---|
| Subcontracts system | Full craftsman-to-craftsman marketplace (V27 migration) |
| Service subcategories | 35 subcategories for 8 categories, all 283 services mapped (V23) |
| Skill categories | 11 categories with full skill assignment (V9) |
| FlashList (mobile) | FlatList replaced in all list screens |
| MMKV (mobile) | AsyncStorage replaced throughout |
| Voice Search (mobile) | expo-speech-recognition integration |
| Dark / Light theme (mobile) | Persistent theme preference |
| TanStack React Query | Stale-while-revalidate caching on web |
| React Compiler | Auto-memoization enabled on web and mobile |
| PostGIS geo search | `ST_DWithin` + GiST index, replaces Java Haversine |
| Redis cache | `@CacheResult` on Services, Skills endpoints |
| FCM Push Notifications | Firebase Cloud Messaging (V22 migration) |
| AI embedding upgrade | Snowflake Arctic Embed L 1024-dim ONNX INT8 (V13) |
| Admin AI Search Config | 28 configurable parameters: pipeline, boost, thresholds, cache (V15) |
| 4-language content | All 60+ skills and 296+ services described in FR/EN/AR/AR-MA (V18–V20) |
| TOTP 2FA | Two-factor authentication for accounts |
| Ads & Monetization | Full ad management: banners, inline, sponsored, popups, impression/click tracking |
| CMS system | Blog, content zones, announcements — admin-managed, no code deploys |
| Anti-scraping | IP-based rate limiting + admin monitoring tab |
| Favorites system | User bookmarks for projects and craftsmen |
| User activity logs | Full user behavior audit trail |
| jOOQ codegen | 80 generated type-safe table classes for complex SQL |
| Prometheus + Grafana | JVM, DB pool, Redis, custom metric dashboards |
| Full CI/CD pipeline | 19 backend test classes + frontend build + secrets check |
| Secrets hygiene | `.env.example` template, all credentials via ENV, scanner in CI |

### 🟡 In Progress

| Feature | Details |
|:---|:---|
| SSE refactoring | Migrate to per-connection `@Context SseEventSink` with clean lifecycle management |
| Redis cache stampede lock | `SETNX` pattern before cache rebuild for top-list endpoints |

### 🔵 Planned

| Feature | Details |
|:---|:---|
| E2E testing (Playwright) | Full user journey tests for wizard + bidding flow |
| Stripe payment integration | In-app escrow for project payments |
| iOS App | Expo EAS build + App Store submission |
| Quarkus Reactive (Mutiny) | Full async stack — worthwhile at >10k concurrent users |
| GraalVM Native Image | Sub-50ms cold start — pending Hibernate reflection compatibility |
| Read replicas | PostgreSQL streaming replication, auto-routed reads |

---

## Why This Project?

This is not a tutorial clone. Every technical decision here came from a real constraint:

- **Darija search** — No library does this. I built one.
- **SSE auth without headers** — Browsers don't allow it. I invented a Redis ticket system.
- **Service-aware skill resolution** — 306 services × 29 questions × context-dependent answers. I built a resolver graph.
- **Dual-realm JWT** — I didn't want Keycloak dependency for a startup. I built a secure HMAC system with refresh token theft detection.
- **ONNX embedding server** — No GPU, so PyTorch inference was too slow. I pre-quantize to INT8 at Docker build time for 19ms CPU inference.

Everything here is production-running, user-tested, and maintained by a single developer.

---

<div align="center">

**[🌍 Live Platform](https://itra.ma)** &nbsp;·&nbsp; **[📱 Download APK](https://itra.ma/downloads/itra-app.apk)**

---

*Full-stack development by [Amir](https://github.com/AmirItra)*  
*Built in Morocco 🇲🇦 — for Morocco*

</div>
