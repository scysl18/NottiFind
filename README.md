# NottFind 🎯

> AI-powered internship and campus event assistant for UNNC students.

---

## Table of Contents

- [What is NottFind?](#what-is-nottfind)
- [Who is it for?](#who-is-it-for)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Project Structure](#project-structure)
- [Five-Dimension Matching Algorithm](#five-dimension-matching-algorithm)
- [Frontend Architecture](#frontend-architecture)
- [Backend Architecture](#backend-architecture)
- [API Reference](#api-reference)
- [Data Sources & Scrapers](#data-sources--scrapers)
- [Authentication & Security](#authentication--security)
- [Quick Start](#quick-start)
- [Deployment](#deployment)
- [User Guide](#user-guide)
- [FAQ](#faq)

---

## What is NottFind?

NottFind is an intelligent platform built for **University of Nottingham Ningbo China (UNNC)** students that combines AI-driven internship matching with campus event aggregation. Instead of spending 5–10 hours weekly scrolling through 10+ WeChat accounts and multiple recruitment platforms, students can:

- **Chat with AI** to define their five-dimensional profile (Skills, Time, Interests, Competency, Company Fit)
- **Get matched** with internships ranked by a mathematically rigorous TOPSIS + Entropy Weight algorithm
- **Discover events** — lectures, job fairs, teach-ins, and WeChat articles — aggregated into one unified view
- **Visualize everything** on a smart calendar with class-conflict detection and AI-curated recommendations

### The Problem

| Pain Point | Description |
|------------|-------------|
| **信息碎片化** | Internship info scattered across Shixiseng, Nowcoder, 10+ WeChat accounts, and UNNC Careers |
| **筛选效率低** | 5–10 hours/week browsing and filtering across platforms |
| **匹配度不透明** | No way to know how well a position fits — students apply by gut feeling |
| **活动信息遗漏** | Campus lectures, job fairs, and teach-ins easily missed across channels |
| **课表冲突** | Manual checking required before attending any event |

---

## Who is it for?

| Persona | Description | Core Need |
|---------|-------------|-----------|
| **Goal-oriented students** | Know their target industry/company, have project or internship experience | Efficient matching, explainable scores |
| **Exploratory students** | Still figuring out their direction, want to build skills | Event recommendations, skill-building paths |

> Registration is restricted to `@nottingham.edu.cn` emails to ensure UNNC student identity.

---

## Key Features

| Feature | Description |
|---------|-------------|
| 🤖 **AI Internship Matching** | Conversational AI (DeepSeek) collects a 5-dimensional profile; aggregates postings from Shixiseng & Nowcoder; displays ranked results with radar charts |
| 📅 **Smart Calendar** | AI-curated event recommendations from UNNC official events, Careers lectures, job fairs, and teach-ins; marks class conflicts from imported timetable |
| 🎯 **Dual-Entry Design** | Start with internships OR start with events — the product adapts to your stage |
| 🔁 **Connected Experience** | Internship skill gaps drive event recommendations; build skills for roles you want |
| 🔐 **User Authentication** | JWT + HttpOnly cookie auth, restricted to `@nottingham.edu.cn` emails |
| 📊 **5D Radar Visualization** | Recharts-powered radar chart shows your match score breakdown across all five dimensions |
| 📰 **Campus News Aggregation** | WeChat articles, UNNC official events, Careers lectures, job fairs, teach-ins — all in one place |
| 📤 **iCal Export** | Export merged calendar to `.ics` for Apple Calendar, Google Calendar, Outlook |

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| **Frontend** | Next.js 14 (App Router), React 18, TypeScript, Tailwind CSS 3.4, Recharts, Framer Motion, Axios, Lucide React |
| **Backend** | FastAPI 0.111, Uvicorn 0.29, Pydantic v2, APScheduler 3.10 |
| **Database** | SQLAlchemy 2 + SQLite (user accounts) + JSON file caches (jobs & events) |
| **AI / ML** | DeepSeek (via OpenAI-compatible API), sentence-transformers (paraphrase-multilingual-MiniLM-L12-v2), LangChain + LangGraph |
| **Scraping** | Requests, BeautifulSoup 4, lxml — Shixiseng, Nowcoder, UNNC official, Sogou WeChat |
| **Auth** | JWT (PyJWT), bcrypt, HttpOnly Cookies |
| **Calendar** | icalendar, recurring-ical-events |
| **Deployment** | Vercel (frontend) + Render (backend) |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Browser (Client)                    │
│           Next.js App (React 18 + TypeScript)            │
└─────────────────────┬───────────────────────────────────┘
                      │  /api/* requests
                      ▼
┌─────────────────────────────────────────────────────────┐
│               Next.js Server (BFF Layer)                 │
│     Route Handlers (/api/match, /api/chat)               │
│     next.config.js rewrites → backend                    │
└─────────────────────┬───────────────────────────────────┘
                      │  HTTP proxy
                      ▼
┌─────────────────────────────────────────────────────────┐
│                 FastAPI Backend Server                    │
│                                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │   Auth   │ │  Match   │ │  Events  │ │ Calendar │   │
│  │  Router  │ │  Router  │ │  Router  │ │  Router  │   │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘   │
│       └─────────────┴────────────┴─────────────┘         │
│                         │                                │
│  ┌──────────────────────▼───────────────────────────┐   │
│  │               Core Services                       │   │
│  │  matcher · embedder · conversation · advisor      │   │
│  │  schedule_parser · explainer · smart_calendar     │   │
│  └───────────┬──────────────────┬────────────────────┘   │
│              │                  │                         │
│  ┌───────────▼───────┐ ┌───────▼──────────┐             │
│  │  SQLite (Users)   │ │ JSON File Cache  │             │
│  │  SQLAlchemy ORM   │ │ jobs / events    │             │
│  └───────────────────┘ └──────────────────┘             │
│              │                                           │
│  ┌───────────▼───────────────────────────────┐          │
│  │          External Services                 │          │
│  │  DeepSeek API · HuggingFace Models        │          │
│  │  Shixiseng · Nowcoder · UNNC · Sogou      │          │
│  └────────────────────────────────────────────┘          │
└──────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
intern-match/
├── backend/
│   ├── main.py                          # FastAPI entry + lifespan + scheduler + CORS
│   ├── requirements.txt                 # Python dependencies
│   ├── runtime.txt                      # Python 3.11.9 (for PaaS)
│   ├── .env.example                     # Environment variable template
│   ├── ical_import.py                   # Scientia iCal import (SSRF-protected)
│   ├── core/
│   │   ├── matcher.py                   # 5D matching + TOPSIS + Entropy Weight
│   │   ├── embedder.py                  # sentence-transformers wrapper
│   │   ├── conversation.py              # DeepSeek multi-turn dialogue + profile extraction
│   │   ├── schedule_parser.py           # Natural language / iCal timetable parsing
│   │   ├── explainer.py                 # AI recommendation reason generation
│   │   ├── advisor.py                   # Skill gap analysis + weak dimension advice
│   │   ├── campus_smart_calendar_agent.py  # Smart calendar (LangChain + DeepSeek)
│   │   └── security.py                  # JWT generation/verification, cookie security
│   ├── routers/
│   │   ├── auth.py                      # Register / login / me / logout
│   │   ├── match.py                     # Matching pipeline + schedule parsing + feedback
│   │   ├── jobs.py                      # Job listing / count / refresh
│   │   ├── chat.py                      # Conversation session management
│   │   ├── events.py                    # Campus events (multi-source)
│   │   └── calendar.py                  # Merged calendar + iCal export + smart calendar
│   ├── scraper/
│   │   ├── shixiseng.py                 # Shixiseng + Nowcoder merged cache
│   │   ├── nowcoder.py                  # Nowcoder API
│   │   ├── unnc_events.py              # UNNC official website events
│   │   ├── careers_lectures.py          # Careers lectures
│   │   ├── careers_jobfairs.py          # Job fairs
│   │   ├── careers_teachins.py          # Teach-ins
│   │   ├── wechat_articles.py           # Sogou WeChat articles
│   │   └── campus_refresh.py            # Unified refresh entry
│   ├── db/
│   │   ├── database.py                  # SQLAlchemy + SQLite init
│   │   └── models.py                    # User table model
│   ├── models/
│   │   ├── schemas.py                   # Match request/response/feedback schemas
│   │   ├── calendar_schemas.py          # Calendar event schemas
│   │   └── smart_calendar_schemas.py    # Smart calendar request/response
│   └── data/                            # Runtime cache (auto-generated, do NOT commit)
│       ├── jobs_cache.json
│       ├── unnc_events.json
│       ├── careers_lectures.json
│       ├── careers_jobfairs.json
│       ├── careers_teachins.json
│       ├── wechat_articles.json
│       ├── feedback.json
│       └── app.db                       # SQLite database
├── frontend/
│   ├── package.json
│   ├── next.config.js                   # API rewrites + image domains
│   ├── tailwind.config.js
│   ├── tsconfig.json
│   ├── app/
│   │   ├── layout.tsx                   # Root layout: AuthProvider + SmartCalendarProvider
│   │   ├── page.tsx                     # Homepage + AI chat entry (?match=1)
│   │   ├── results/page.tsx             # Match results: radar chart + job list + advice
│   │   ├── campus/
│   │   │   ├── page.tsx                 # Campus hub: merged calendar + events + WeChat
│   │   │   └── smart-calendar/page.tsx  # AI-curated event recommendations
│   │   ├── login/page.tsx
│   │   ├── register/page.tsx
│   │   ├── me/page.tsx
│   │   ├── saved/page.tsx               # Saved jobs
│   │   └── api/                         # BFF Route Handlers
│   │       ├── match/route.ts           # Proxy (maxDuration: 120s)
│   │       └── chat/route.ts
│   ├── components/
│   │   ├── SiteHeader.tsx               # Global nav, backend health indicator
│   │   ├── SiteFooter.tsx
│   │   └── campus/                      # Calendar UI, event cards, timetable import
│   ├── hooks/
│   │   ├── useAuth.tsx                  # Auth context (axios withCredentials)
│   │   └── useSmartCalendarStore.tsx     # Smart calendar global request state
│   └── lib/
│       ├── storage.ts                   # Per-userId localStorage (profile, saved, drafts)
│       ├── timetableStorage.ts          # Timetable local cache
│       ├── customCampusEventsStorage.ts # Custom events storage
│       └── matchToCalendarPrefill.ts    # Prefill calendar from match results
└── render.yaml                          # Render deployment blueprint
```

---

## Five-Dimension Matching Algorithm

NottFind's core matching uses a mathematically rigorous five-dimension scoring system with **TOPSIS + Entropy Weight** aggregation.

### Overview

```
Input:  User Profile + Job Database
  │
  ├── D1: IDF-weighted skill matching + hard-skill penalty
  ├── D2: Sigmoid time fit + internship-type awareness
  ├── D3: SIF-weighted interest alignment (Jaccard + semantic)
  ├── D4: Logistic competency assessment + project bonus
  └── D5: RBF kernel company fit
  │
  ├── Static weights  W_static  = [0.30, 0.25, 0.20, 0.15, 0.10]
  ├── Entropy weights W_entropy = (data-driven dynamic weights)
  └── W_final = 0.6 × W_static + 0.4 × W_entropy
  │
  └── TOPSIS ranking → Top-N recommendation list
```

### Dimension Details

| Dimension | Weight | Formula | Description |
|-----------|--------|---------|-------------|
| **D1** Skills Match | 30% | `D1 = [Σⱼ idf(rⱼ) · maxᵢ cos(uᵢ, rⱼ)] / Σⱼ idf(rⱼ) × ∏ₖ (1 - α·I(hardₖ ∉ U))` | IDF-weighted bipartite optimal coverage with hard-skill multiplicative penalty |
| **D2** Time Fit | 25% | `D2 = σ(κ·(ratio - θ)), κ=8, θ=0.8` | Sigmoid-smoothed hour coverage ratio; remote jobs → `max(D2, 0.9)`; expired jobs → `D2 *= 0.15` |
| **D3** Interest Alignment | 20% | `D3 = 0.35·Jaccard_SIF + 0.65·cos(v_user, v_job)` | SIF-weighted (`α=0.001`) Jaccard + semantic cosine similarity |
| **D4** Competency Level | 15% | `D4 = clamp(σ(1.5·ΔL) + 0.3·exp(-ΔL²/2)·I(project), 0, 1)` | Logistic grade mapping + Gaussian project experience bonus |
| **D5** Company Fit | 10% | `K(x,y) = exp(-\|x-y\|²/2σ²), σ=1.0` | RBF kernel similarity on company size/industry/culture attributes |

### TOPSIS + Entropy Weight Aggregation

```
d⁺ = √[Σ wᵢ · (vᵢ - 1)²]     # Distance to positive ideal solution
d⁻ = √[Σ wᵢ · vᵢ²]            # Distance to negative ideal solution
TOPSIS = d⁻ / (d⁺ + d⁻)        # Closer to 1 = better match
```

**Academic references**: Hwang & Yoon (1981) — TOPSIS; Shannon (1948) — Entropy Weight; Arora et al. (2017) — SIF Embeddings; Robertson & Jones (1976) — IDF.

---

## Frontend Architecture

### Pages

| Route | Page | Description |
|-------|------|-------------|
| `/` | Homepage | Brand showcase + AI chat entry (`?match=1`) |
| `/results` | Match Results | 5D radar chart + ranked job list + AI advice |
| `/campus` | Campus Hub | Merged calendar + categorized events + WeChat articles + timetable import |
| `/campus/smart-calendar` | Smart Calendar | AI-curated event picks with conflict detection |
| `/login` | Login | Email + password |
| `/register` | Register | UNNC email registration |
| `/me` | Profile | User info |
| `/saved` | Saved Jobs | Bookmarked positions |

### State Management

| Data | Storage | Details |
|------|---------|---------|
| Auth state | React Context (`useAuth`) | JWT cookie auto-attached via `withCredentials` |
| Smart calendar requests | React Context (`useSmartCalendarStore`) | Survives page navigation |
| Profile drafts | localStorage (per userId) | Resume interrupted conversations |
| Saved jobs | localStorage (per userId) | Offline viewable |
| Timetable | localStorage | Parsed iCal cached locally |
| Custom events | localStorage | User-added events |

### API Proxy Strategy

`next.config.js` rewrites all `/api/*` to the backend. Long-running requests (matching) use Next.js Route Handlers with `maxDuration: 120` seconds for Vercel Serverless compatibility.

---

## Backend Architecture

### Router Modules

| Router | Prefix | Responsibilities |
|--------|--------|-----------------|
| `auth.py` | `/api/auth` | Register, login, get current user, logout |
| `match.py` | `/api` | Full matching pipeline, schedule parsing preview, user feedback |
| `jobs.py` | `/api/jobs` | Job listing, count, manual cache refresh |
| `chat.py` | `/api/chat` | Multi-turn conversation, greeting, session snapshot/restore |
| `events.py` | `/api/campus` | Events, lectures, job fairs, teach-ins, WeChat articles |
| `calendar.py` | `/api/calendar` | Merged calendar, iCal export, timetable import, smart calendar |

### Core Services

| Module | Function |
|--------|----------|
| `matcher.py` | 5D scoring + TOPSIS + entropy weight ranking |
| `embedder.py` | sentence-transformers vectorization (multilingual MiniLM) |
| `conversation.py` | DeepSeek multi-turn dialogue + JSON profile extraction |
| `schedule_parser.py` | Natural language / iCal → busy/free time slots |
| `explainer.py` | Per-job AI recommendation reason |
| `advisor.py` | Skill gap analysis + weak dimension advice |
| `campus_smart_calendar_agent.py` | LangChain + DeepSeek event filtering with conflict marking |
| `security.py` | JWT lifecycle, cookie security flags, email domain validation |

### Scheduled Tasks

| Task | Time | Description |
|------|------|-------------|
| Job cache refresh | Daily 03:00 | Scrape Shixiseng + Nowcoder, dedupe, write `jobs_cache.json` |
| Campus event refresh | Daily 04:00 | Refresh all campus data source caches |

Powered by APScheduler `BackgroundScheduler` + `CronTrigger`.

---

## API Reference

### Authentication (`/api/auth`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/register` | Register (`@nottingham.edu.cn` only) |
| `POST` | `/login` | Login (returns HttpOnly JWT cookie) |
| `GET` | `/me` | Get current user info |
| `POST` | `/logout` | Logout (clear cookie) |

### Matching (`/api`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/match` | Full matching pipeline (schedule → profile → rank → advice) |
| `POST` | `/parse-schedule` | Schedule parsing preview |
| `POST` | `/feedback` | Submit match feedback |

### Jobs (`/api/jobs`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | List cached job postings |
| `GET` | `/count` | Total job count |
| `POST` | `/refresh` | Trigger background scraper refresh |

### Conversation (`/api/chat`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/` | Send message, get AI reply |
| `GET` | `/greeting` | Get greeting message |

### Campus Events (`/api/campus`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/events` | UNNC official events |
| `GET` | `/lectures` | Careers lectures |
| `GET` | `/jobfairs` | Job fairs |
| `GET` | `/teachins` | Teach-ins |
| `GET` | `/articles` | WeChat articles |
| `POST` | `/refresh-all` | Refresh all campus caches |

### Calendar (`/api/calendar`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/merged` | Merged calendar (all sources) |
| `GET` | `/export.ics` | iCal format export |
| `POST` | `/import/ical` | Import Scientia timetable |
| `POST` | `/smart-plan` | AI-curated event recommendations |

---

## Data Sources & Scrapers

| Source | Content | Scraper Module | Cache File | Refresh |
|--------|---------|---------------|------------|---------|
| Shixiseng (实习僧) | Ningbo internship postings | `shixiseng.py` | `jobs_cache.json` | Daily 03:00 + manual |
| Nowcoder (牛客) | National internship postings | `nowcoder.py` | Merged into `jobs_cache.json` | Daily 03:00 + manual |
| UNNC Official | Campus events | `unnc_events.py` | `unnc_events.json` | Daily 04:00 + manual |
| Careers Office | Lectures | `careers_lectures.py` | `careers_lectures.json` | Daily 04:00 + manual |
| Careers Office | Job fairs | `careers_jobfairs.py` | `careers_jobfairs.json` | Daily 04:00 + manual |
| Careers Office | Teach-ins | `careers_teachins.py` | `careers_teachins.json` | Daily 04:00 + manual |
| Sogou WeChat | WeChat articles | `wechat_articles.py` | `wechat_articles.json` | On-demand |
| Scientia iCal | Student timetable | `ical_import.py` | — (per-request) | User-initiated |

---

## Authentication & Security

| Mechanism | Implementation |
|-----------|---------------|
| Password hashing | bcrypt (irreversible) |
| Token | JWT (PyJWT), stored in HttpOnly Cookie |
| Email restriction | `@nottingham.edu.cn` only |
| CORS | `ALLOWED_ORIGINS` env var whitelist |
| Cookie security | `Secure` + `SameSite` in production (`AUTH_COOKIE_SECURE=1`) |
| iCal SSRF protection | Only `*.scientia.com.cn` domains allowed |

---

## Quick Start

### Prerequisites

| Tool | Version |
|------|---------|
| Python | 3.11+ |
| Node.js | 18+ |
| npm | 9+ |

### 1. Backend

```bash
cd backend

# Create & activate virtual environment
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS / Linux

# Install dependencies (~3-5 min first time)
pip install -r requirements.txt

# Configure environment
copy .env.example .env          # Windows
# cp .env.example .env          # macOS / Linux
# Edit .env → fill in DEEPSEEK_API_KEY

# Start server
uvicorn main:app --reload --port 8000
```

API docs available at http://localhost:8000/docs

#### Hugging Face Download Issues (Mainland China)

If you see `huggingface.co ConnectTimeout`, add to `.env`:

```env
# Option A: Use official mirror
USE_HF_MIRROR=1

# Option B: Custom mirror endpoint
HF_ENDPOINT=https://hf-mirror.com

# Temporary: skip preload, download on first match request
SKIP_EMBEDDER_WARMUP=1
```

### 2. Frontend

```bash
cd frontend
npm install
npm run dev
```

Visit http://localhost:3000

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DEEPSEEK_API_KEY` | ✅ | — | DeepSeek API key |
| `JWT_SECRET` | ✅ | — | JWT signing secret (use long random string in production) |
| `AUTH_COOKIE_SECURE` | — | `0` | Set to `1` in production (HTTPS) |
| `ALLOWED_ORIGINS` | — | `*` | CORS whitelist (comma-separated) |
| `DATABASE_URL` | — | `sqlite:///./data/app.db` | Database connection string |
| `SKIP_EMBEDDER_WARMUP` | — | — | `1` to skip model preload on startup |
| `USE_HF_MIRROR` | — | — | `1` to use HuggingFace China mirror |
| `NEXT_PUBLIC_API_URL` | — | `http://localhost:8000` | Backend URL for frontend |

---

## Deployment

### Backend → Render

1. Push code to GitHub
2. Go to [Render Dashboard](https://dashboard.render.com/) → **New → Blueprint** (auto-detects `render.yaml`)
3. Or manually create **Web Service**:
   - Root Directory: `backend`
   - Build Command: `pip install -r requirements.txt`
   - Start Command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
4. Set environment variables: `DEEPSEEK_API_KEY`, `JWT_SECRET`, `AUTH_COOKIE_SECURE=1`, `SKIP_EMBEDDER_WARMUP=1`, `ALLOWED_ORIGINS`
5. Recommended: Mount **Disk** (1GB, path `/opt/render/project/src/data`) to persist SQLite and cache files

### Frontend → Vercel

1. Go to [Vercel Dashboard](https://vercel.com/dashboard) → **Import** GitHub repo
2. Set Root Directory to `frontend`
3. Set env var: `NEXT_PUBLIC_API_URL` = your Render backend URL
4. Deploy, then copy the Vercel URL back to Render's `ALLOWED_ORIGINS`

### Deployment Notes

- Render free tier sleeps after ~15 min of inactivity; first request takes 30–60s cold start
- sentence-transformers model is ~400MB, downloaded on first match request when `SKIP_EMBEDDER_WARMUP=1`
- SQLite data is lost on redeploy without a Disk mount
- APScheduler cron jobs may not fire during Render sleep; frontend has a compensating first-visit auto-refresh

---

## User Guide

### AI Internship Matching

1. Click **开始匹配** on the homepage (or visit `?match=1`)
2. Chat with the AI assistant — it will ask about your:

| Dimension | What to share | Example |
|-----------|--------------|---------|
| **Skills** | Programming languages, tools, soft skills | "I know Python, Java, and have used PyTorch" |
| **Time** | Class schedule, free hours, internship timing | "Classes Mon–Wed, looking for summer internship" |
| **Interests** | Industry and role preferences | "Interested in data analysis and fintech" |
| **Competency** | Year, project experience | "Junior year, built an ML project" |
| **Company Fit** | Company size, industry, culture | "Prefer large companies, tech industry" |

3. After profile collection, the system runs matching (~20-30s) and shows:
   - **Radar chart** — 5D score visualization
   - **Ranked job list** — with total score and per-dimension breakdown
   - **AI recommendation reasons** — personalized for each job
   - **Improvement advice** — based on your weak dimensions

4. You can **save** jobs, **provide feedback**, or **prefill calendar** with deadlines

### Campus Hub

- Browse events from UNNC, Careers lectures, job fairs, teach-ins, and WeChat articles in one unified view
- **Import timetable** from Scientia iCal to auto-detect class conflicts
- Use **Smart Calendar** for AI-curated event picks based on your profile
- **Export** merged calendar as `.ics` for external calendar apps
- **Add custom events** manually

---

## FAQ

**Q: Why is the first match so slow?**
A: The first match takes 20–30s because it parses your schedule, builds profile vectors, runs matching against all jobs, and generates AI explanations. Subsequent matches are faster.

**Q: Why are there few job listings?**
A: Jobs are scraped in real-time from Shixiseng and Nowcoder. The count depends on current market postings. The system auto-refreshes daily at 03:00 and supports manual refresh.

**Q: Can I use it on mobile?**
A: Yes. NottFind has a fully responsive design that works on phones and tablets.

**Q: Is my data safe?**
A: Passwords are bcrypt-hashed (irreversible). Auth uses HttpOnly cookies (XSS-resistant). Only UNNC emails are accepted. Conversation and profile data is used only for matching, never shared with third parties.

**Q: What if the HuggingFace model won't download?**
A: Set `USE_HF_MIRROR=1` or `HF_ENDPOINT=https://hf-mirror.com` in `.env`. Or set `SKIP_EMBEDDER_WARMUP=1` to defer download to the first match request.

---

## License

This project is developed for the UNNC Hackathon. All rights reserved.

## Team

Built with ❤️ by UNNC students.
