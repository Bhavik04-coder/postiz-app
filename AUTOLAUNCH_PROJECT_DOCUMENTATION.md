# AutoLaunch — Project Documentation

> **Tagline:** Ship content as fast as you ship code.
> **Version:** 1.0.0 | **Hackathon:** Commudle 2025
> **Team:** Yash · Bhavik · Akhilesh · Mrunmai

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Proposed Solution](#3-proposed-solution)
4. [How It Works — Full Flow](#4-how-it-works--full-flow)
5. [System Architecture](#5-system-architecture)
6. [Technology Stack](#6-technology-stack)
7. [Repository Structure](#7-repository-structure)
8. [Database Schema](#8-database-schema)
9. [API Design](#9-api-design)
10. [AI Pipeline — Caption Generation](#10-ai-pipeline--caption-generation)
11. [AI Pipeline — Image Generation](#11-ai-pipeline--image-generation)
12. [Scheduling Engine — How It Works](#12-scheduling-engine--how-it-works)
13. [Component Breakdown](#13-component-breakdown)
14. [Excalidraw Diagram Guide](#14-excalidraw-diagram-guide)
15. [Team Responsibilities](#15-team-responsibilities)
16. [Roadmap](#16-roadmap)

---

## 1. Project Overview

**AutoLaunch** is an AI-powered social media content automation platform built for developer communities like Commudle.

When a new feature, event, or community update is ready to go live, AutoLaunch automatically generates a complete content package — platform-specific captions + AI-generated images — and pushes everything into a visual calendar dashboard. The team then picks the date and time, previews the posts, and publishes with one click.

### What AutoLaunch Is

```
AutoLaunch = AI Content Generation Pipeline
           + Visual Calendar Scheduling Dashboard
           + Multi-Platform Publishing Engine
```

Everything is built and owned by AutoLaunch. One product. One repo. One brand.

---

## 2. Problem Statement

### The Core Pain

Every time Commudle ships a new event, feature, or update — someone has to manually write posts for LinkedIn, Instagram, X, and WhatsApp. That means 4 different captions in 4 different tones, 3 image size variants, approval back-and-forth, and manual scheduling across platforms.

### Quantified Problems

| Problem | Data |
|---|---|
| Time per feature launch | 6–10 hours manual work |
| Promotion delay | 2–3 days after feature ships |
| Content team cost | ₹70,000–₹1,45,000/month |
| Tool cost | ₹12,000–₹25,000/month |
| Approval cycle | 2–4 days of back-and-forth |
| Brand inconsistency | LinkedIn copy pasted to Instagram |
| Repetitive tasks | 40–60% of marketing time wasted |

---

## 3. Proposed Solution

### One Brief In → Four Platforms Out → Under 60 Seconds

User submits a feature brief. AutoLaunch's NestJS backend calls Groq Llama 3.2 to generate captions for all 4 platforms in parallel, then calls NanoBanana to generate 3 image variants. Everything lands in AutoLaunch's visual calendar dashboard as drafts. Team picks time, reviews, and schedules — everything publishes automatically.

### Input
```
Feature Name     : Community Dashboard 2.0
Description      : Real-time analytics for community managers
Target Audience  : Developer community leads and event organizers
Key Benefit      : See who's engaging, who's churning, what content performs
Campaign Goal    : Signups
Launch Date      : 2025-03-15
```

### Output (appears in calendar as drafts)
```
LinkedIn Post     → Professional long-form with data points and CTA
X Post            → Punchy thread with hooks and hashtags
Instagram Post    → Emoji-rich caption + 30 hashtags
WhatsApp Message  → Brief, friendly, direct link
Square Image      → 1080×1080 (LinkedIn, Instagram feed, X)
Story Image       → 1080×1920 (Instagram Stories, WhatsApp Status)
Landscape Image   → 1920×1080 (LinkedIn banner, X card)
Status            → Draft in AutoLaunch Calendar — ready to schedule
```

---

## 4. How It Works — Full Flow

```
STEP 1 — USER SUBMITS FEATURE BRIEF
  Via: AutoLaunch dashboard form
  Data: feature name, description, audience, benefit, goal, launch date
  Action: POST /api/launches → NestJS → saved to PostgreSQL

STEP 2 — NESTJS RUNS THE AI PIPELINE
  GroqService → Llama 3.2:
    Call 1 : Enrich brief (extract hooks, USPs, angles, power words)
    Call 2 : Generate LinkedIn caption   ┐
    Call 3 : Generate X caption          ├── all 4 run in parallel
    Call 4 : Generate Instagram caption  ┘    via Promise.all()
    Call 5 : Generate WhatsApp message

  NanoBananaService → NanoBanana API:
    Call 1 : Square image    1080×1080 (LinkedIn, Instagram, X)
    Call 2 : Story image     1080×1920 (Instagram Stories, WhatsApp)
    Call 3 : Landscape image 1920×1080 (LinkedIn header, X card)

  All captions + image URLs saved to PostgreSQL

STEP 3 — CONTENT APPEARS IN AUTOLAUNCH CALENDAR AS DRAFTS
  All 4 posts arrive in the visual calendar dashboard
  Status: DRAFT — not yet scheduled

STEP 4 — AUTOLAUNCH CALENDAR DASHBOARD
  Team opens AutoLaunch dashboard:
    → Sees all 4 draft posts on the visual calendar
    → Previews each post in platform-native format
    → Edits caption directly in the dashboard if needed
    → Drags post to desired date and time on calendar
    → Selects which platforms to publish to
    → Clicks Schedule

STEP 5 — TEMPORAL PUBLISHES AT SCHEDULED TIME
  AutoLaunch's scheduling engine (Temporal):
    → Creates a durable workflow that sleeps until scheduled time
    → Wakes at exact datetime and fires PostActivity
    → Each platform has its own task queue
    → Auto-retries on failure with backoff
    → Survives server restart — no post is ever lost

STEP 6 — POSTS GO LIVE ACROSS ALL PLATFORMS
  LinkedIn ✅   X ✅   Instagram ✅   WhatsApp ✅

STEP 7 — ANALYTICS BACK IN AUTOLAUNCH DASHBOARD
  Engagement, reach, and impressions per platform
  Visible in AutoLaunch's built-in analytics panel
```

---

## 5. System Architecture

### High-Level Overview

```
┌────────────────────────────────────────────────────────────────┐
│                     AUTOLAUNCH PLATFORM                        │
│                                                                │
│    Next.js Dashboard              Calendar + Analytics View    │
│    (Brief Submission +            (Scheduling + Publishing     │
│     Content Preview)               + Post Performance)         │
└──────────────┬─────────────────────────────┬───────────────────┘
               │                             │
               ▼                             ▼
┌──────────────────────────┐   ┌─────────────────────────────────┐
│    AUTOLAUNCH BACKEND    │   │    AUTOLAUNCH SCHEDULING ENGINE  │
│       (NestJS)           │   │                                 │
│                          │   │  Visual Calendar Dashboard      │
│  LaunchService           │──▶│  Drag-and-drop Scheduling       │
│  ContentService          │   │  Platform OAuth (20+ channels)  │
│  GroqService             │   │  Temporal Publishing Worker     │
│  NanoBananaService       │   │  Analytics Dashboard            │
│  SchedulerService        │   │  Draft + Edit + Publish Flow    │
│  BrandService            │   │                                 │
└──────────┬───────────────┘   └─────────────────────────────────┘
           │                                 │
           ▼                                 ▼
┌──────────────────────┐        ┌────────────────────────────────┐
│     AI SERVICES      │        │        SOCIAL PLATFORMS        │
│                      │        │                                │
│  Groq + Llama 3.2    │        │  LinkedIn  X  Instagram        │
│  NanoBanana          │        │  WhatsApp  Facebook  etc.      │
└──────────────────────┘        └────────────────────────────────┘
           │
           ▼
┌──────────────────────┐
│      DATA LAYER      │
│                      │
│  PostgreSQL          │
│  Redis               │
└──────────────────────┘
```

### Request Flow

```
User
 │  Submit Feature Brief
 ▼
AutoLaunch Next.js Dashboard
 │  POST /api/launches
 ▼
AutoLaunch NestJS Backend
 ├── Save to PostgreSQL
 ├── Fetch Brand Config from DB
 ├── GroqService
 │     ├── Enrich Brief (1 call — Llama 3.2)
 │     └── Generate Captions (4 parallel calls)
 │           LinkedIn · X · Instagram · WhatsApp
 ├── NanoBananaService
 │     └── Generate Images (3 calls)
 │           Square · Story · Landscape
 ├── Save all content to PostgreSQL
 └── Push to Scheduling Engine as Drafts
                    │
                    ▼
         AutoLaunch Calendar Dashboard
                    │
              Team reviews all 4 drafts
              Picks date + time per platform
              Drags to calendar slot
              Edits if needed
              Clicks Schedule
                    │
                    ▼
         Temporal Workflow Created
                    │
              Sleeps until scheduled time
              Fires at exact datetime
                    │
                    ▼
          Social Platforms Published ✅
                    │
                    ▼
          Analytics tracked in Dashboard
```

---

## 6. Technology Stack

### AutoLaunch (Everything We Build)

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | Next.js 14 App Router | Dashboard + calendar + analytics |
| Frontend | React 18 | UI components |
| Frontend | TypeScript | Type safety across entire codebase |
| Frontend | TailwindCSS | Styling |
| Frontend | shadcn/ui | Component library |
| Backend | NestJS | API server + AI pipeline orchestration |
| Backend | TypeScript | Language |
| Backend | Prisma ORM | Database queries + migrations |
| Scheduling | Temporal | Durable workflow engine for publishing |
| Database | PostgreSQL | Primary data store |
| Cache | Redis | Caching + rate limiting + task queues |
| AI Captions | Groq API + Llama 3.2 | Caption generation (500+ tokens/sec, free tier) |
| AI Images | NanoBanana | Image generation (3 format variants) |
| DevOps | Docker + Docker Compose | Local dev + deployment |
| DevOps | GitHub Actions | CI/CD pipeline |
| Hosting | Vercel | Frontend deployment |
| Hosting | Railway / Render | Backend + worker deployment |

### Platform OAuth Integrations

| Platform | Auth Method | Purpose |
|---|---|---|
| LinkedIn | OAuth 2.0 | Publish professional posts |
| X (Twitter) | OAuth 2.0 | Publish tweets and threads |
| Instagram | OAuth 2.0 | Publish feed posts and stories |
| WhatsApp | OAuth 2.0 | Publish business messages |
| Facebook | OAuth 2.0 | Publish page posts |

---

## 7. Repository Structure

```
autolaunch/
├── apps/
│   ├── frontend/                        # Next.js web dashboard
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── (dashboard)/
│   │   │   │   │   ├── launches/        # Feature brief form + history
│   │   │   │   │   ├── calendar/        # Visual scheduling calendar
│   │   │   │   │   ├── preview/         # Content package preview
│   │   │   │   │   ├── analytics/       # Post performance dashboard
│   │   │   │   │   └── settings/        # Brand guidelines + integrations
│   │   │   │   ├── auth/                # Login / register
│   │   │   │   └── layout.tsx
│   │   │   ├── components/
│   │   │   │   ├── brief-form/          # Feature brief input
│   │   │   │   ├── calendar/            # Drag-and-drop calendar UI
│   │   │   │   ├── content-preview/     # Platform post previews
│   │   │   │   ├── brand-settings/      # Brand voice config
│   │   │   │   └── ui/                  # Generic components
│   │   │   └── middleware.ts
│   │   └── next.config.js
│   │
│   ├── backend/                         # NestJS API server
│   │   └── src/
│   │       ├── main.ts                  # App entry point
│   │       ├── app.module.ts            # Root module
│   │       └── api/
│   │           └── routes/
│   │               ├── launches.controller.ts   # Launch CRUD + AI trigger
│   │               ├── content.controller.ts    # Generated content
│   │               ├── schedule.controller.ts   # Scheduling CRUD
│   │               ├── brands.controller.ts     # Brand guidelines
│   │               ├── analytics.controller.ts  # Post analytics
│   │               ├── auth.controller.ts       # Authentication
│   │               └── webhooks.controller.ts   # GitHub / Notion webhooks
│   │
│   └── worker/                          # Temporal worker (publishing engine)
│       └── src/
│           ├── main.ts                  # Worker entry point
│           ├── workflows/
│           │   ├── post.workflow.ts     # Main post publishing workflow
│           │   └── token.workflow.ts    # OAuth token refresh workflow
│           └── activities/
│               ├── post.activity.ts     # Calls social platform APIs
│               └── analytics.activity.ts # Fetches post performance data
│
├── libraries/
│   └── nestjs-libraries/
│       └── src/
│           ├── database/prisma/
│           │   ├── schema.prisma        # All DB tables + relations
│           │   ├── launches/            # Launch repository
│           │   ├── content/             # Content repository
│           │   ├── schedules/           # Schedule repository
│           │   ├── brands/              # Brand repository
│           │   └── users/               # User repository
│           ├── groq/
│           │   └── groq.service.ts      # Groq + Llama 3.2 wrapper
│           ├── nanobanana/
│           │   └── image.service.ts     # NanoBanana image gen wrapper
│           ├── integrations/
│           │   ├── linkedin.provider.ts # LinkedIn API
│           │   ├── twitter.provider.ts  # X / Twitter API
│           │   ├── instagram.provider.ts # Instagram API
│           │   └── whatsapp.provider.ts # WhatsApp Business API
│           └── temporal/
│               └── temporal.module.ts   # Temporal client config
│
├── docker-compose.yaml                  # Full stack
├── docker-compose.dev.yaml              # Dev (DB + Redis only)
├── pnpm-workspace.yaml                  # Monorepo config
└── package.json
```

---

## 8. Database Schema

```prisma
model User {
  id           String       @id @default(uuid())
  email        String       @unique
  name         String
  role         Role         @default(USER)
  organization Organization @relation(fields: [orgId], references: [id])
  orgId        String
  createdAt    DateTime     @default(now())
  @@index([email])
}

model Organization {
  id           String        @id @default(uuid())
  name         String
  slug         String        @unique
  users        User[]
  brand        Brand?
  launches     Launch[]
  integrations Integration[]
  createdAt    DateTime      @default(now())
}

model Brand {
  id              String       @id @default(uuid())
  organization    Organization @relation(fields: [orgId], references: [id])
  orgId           String       @unique
  toneRules       Json         // per-platform tone instructions
  ctaLibrary      Json         // array of approved CTAs
  bannedWords     String[]     // words AI must avoid
  hashtagStrategy Json         // per-platform hashtag sets
  visualStyle     String       // image generation style description
  brandColors     String[]     // hex color codes for image gen
  updatedAt       DateTime     @updatedAt
}

model Integration {
  id           String       @id @default(uuid())
  organization Organization @relation(fields: [orgId], references: [id])
  orgId        String
  platform     Platform
  accessToken  String
  refreshToken String?
  expiresAt    DateTime?
  profileName  String
  profileId    String
  createdAt    DateTime     @default(now())
  @@unique([orgId, platform])
}

model Launch {
  id             String       @id @default(uuid())
  organization   Organization @relation(fields: [orgId], references: [id])
  orgId          String
  featureName    String
  description    String
  targetAudience String
  keyBenefit     String
  launchDate     DateTime
  campaignGoal   CampaignGoal
  status         LaunchStatus @default(DRAFT)
  content        Content?
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  @@index([orgId, status])
}

model Content {
  id                String         @id @default(uuid())
  launch            Launch         @relation(fields: [launchId], references: [id])
  launchId          String         @unique
  linkedinCaption   String
  twitterCaption    String
  instagramCaption  String
  whatsappMessage   String
  squareImageUrl    String         // 1080×1080
  storyImageUrl     String         // 1080×1920
  landscapeImageUrl String         // 1920×1080
  status            ContentStatus  @default(GENERATING)
  scheduledPosts    ScheduledPost[]
  createdAt         DateTime       @default(now())
  updatedAt         DateTime       @updatedAt
}

model ScheduledPost {
  id              String      @id @default(uuid())
  content         Content     @relation(fields: [contentId], references: [id])
  contentId       String
  platform        Platform
  caption         String
  imageUrl        String?
  scheduledAt     DateTime
  publishedAt     DateTime?
  status          PostStatus  @default(DRAFT)
  platformPostId  String?     // ID returned by social platform API
  platformPostUrl String?     // live URL of the published post
  temporalWorkflowId String?  // Temporal workflow reference
  createdAt       DateTime    @default(now())
  updatedAt       DateTime    @updatedAt
  @@index([contentId, platform])
  @@index([status, scheduledAt])
}

enum Role          { ADMIN USER }
enum CampaignGoal  { AWARENESS SIGNUPS ENGAGEMENT RETENTION }
enum LaunchStatus  { DRAFT PROCESSING READY PUBLISHED }
enum ContentStatus { GENERATING READY SCHEDULED }
enum Platform      { LINKEDIN TWITTER INSTAGRAM WHATSAPP FACEBOOK }
enum PostStatus    { DRAFT SCHEDULED PUBLISHED FAILED CANCELLED }
```

---

## 9. API Design

### Launches

```
POST   /api/launches                  Create launch + trigger AI pipeline
GET    /api/launches                  List all launches (org scoped)
GET    /api/launches/:id              Get single launch with content
PATCH  /api/launches/:id              Update launch brief
DELETE /api/launches/:id              Delete launch
```

### Content

```
GET    /api/launches/:id/content      Get generated content package
POST   /api/launches/:id/regenerate   Re-run AI pipeline (replace content)
```

### Scheduling

```
POST   /api/schedule                  Create scheduled post (from calendar)
GET    /api/schedule                  List all scheduled posts
PATCH  /api/schedule/:id              Update scheduled time or caption
DELETE /api/schedule/:id              Cancel scheduled post
GET    /api/schedule/calendar         Get posts for calendar view (date range)
```

### Integrations

```
GET    /api/integrations              List connected social accounts
POST   /api/integrations/connect      Start OAuth flow for a platform
DELETE /api/integrations/:id          Disconnect a social account
```

### Analytics

```
GET    /api/analytics                 Overview — all platforms, all posts
GET    /api/analytics/:postId         Single post performance stats
```

### Brand

```
GET    /api/brand                     Get organization brand config
PUT    /api/brand                     Update brand config
```

### Auth

```
POST   /auth/register                 Register organization
POST   /auth/login                    Login
GET    /auth/me                       Current user
POST   /auth/refresh                  Refresh JWT token
```

### Webhooks

```
POST   /webhooks/github               GitHub release → auto-trigger pipeline
POST   /webhooks/notion               Notion DB update → auto-trigger pipeline
```

---

## 10. AI Pipeline — Caption Generation

### Model: Llama 3.2 via Groq API

**Why Groq + Llama 3.2:**
- Free tier supports hackathon scale
- 500+ tokens/second — fastest inference available
- Excellent instruction-following and tone control
- Low latency = near-instant content generation

### Pipeline Steps Inside NestJS ContentService

```
Step 1 — Brief Enrichment (1 Groq call)
  Input  : raw feature brief + brand config
  Output : enriched context object
    {
      primary_hook,       // most compelling angle
      pain_points,        // what problem this solves
      usp,                // unique selling proposition
      power_words,        // emotionally resonant vocabulary
      emotional_angle,    // excitement / relief / curiosity
      platform_hints      // per-platform content recommendations
    }

Step 2 — Caption Generation (4 parallel Groq calls via Promise.all)

  LinkedIn system prompt:
    Tone     : Professional, data-driven, thought leadership
    Audience : Founders, CTOs, community leads
    Format   : 3–5 insight paragraphs, bold hook opener
    Length   : 1200–1800 characters
    Ends with: one clear CTA

  X system prompt:
    Tone     : Punchy, hook-first, casual but sharp
    Audience : Developers, builders, tech enthusiasts
    Format   : Under 280 chars, line breaks, 2–4 hashtags
    Opens    : bold statement or question

  Instagram system prompt:
    Tone     : Conversational, community-first, emoji-friendly
    Audience : Community members, event attendees
    Format   : 150–300 chars caption + 20–30 hashtags below
    Tells    : micro-story, ends with engaging question

  WhatsApp system prompt:
    Tone     : Friendly, brief, like a message from a friend
    Audience : Community WhatsApp group members
    Format   : 100–200 chars, no hashtags, one direct link
    Simple CTA
```

### Groq API Call (TypeScript)

```typescript
// groq.service.ts
const response = await groq.chat.completions.create({
  model: 'llama-3.2-70b-versatile',
  messages: [
    { role: 'system', content: platformSystemPrompt },
    { role: 'user',   content: JSON.stringify(enrichedContext) }
  ],
  temperature: 0.7,
  max_tokens: 1000
});
return response.choices[0].message.content;
```

### Brand Injection

Every Groq call receives the brand config as part of the system prompt:
```
Brand: {brand_name}
Tone rules: {toneRules}
Approved CTAs: {ctaLibrary}
Never use: {bannedWords}
Always include: {hashtagStrategy}
Visual style: {visualStyle}
```

---

## 11. AI Pipeline — Image Generation

### Service: NanoBanana

**Why NanoBanana:**
- API-first, fast image generation
- Supports brand color injection
- Returns social-media-ready image URLs
- Cost-effective for startup / hackathon scale

### 3 Formats Generated Per Launch

| Format | Dimensions | Used On |
|---|---|---|
| Square | 1080 × 1080 | LinkedIn feed, Instagram feed, X |
| Story | 1080 × 1920 | Instagram Stories, WhatsApp Status |
| Landscape | 1920 × 1080 | LinkedIn header, X card preview |

### Image Prompt Assembly

Each prompt is built from:
- Feature name + key benefit (content layer)
- Brand visual style from Brand config (tone layer)
- Brand color hex codes (color layer)
- Platform layout hint (format layer)
- Text overlay instruction (copy layer)

### NanoBanana API Call (TypeScript)

```typescript
// image.service.ts
const image = await fetch('https://api.nanobanana.io/v1/generate', {
  method: 'POST',
  headers: { 'x-api-key': process.env.NANOBANANA_API_KEY },
  body: JSON.stringify({
    prompt: assembledImagePrompt,
    width: 1080,
    height: 1080,
    style: brand.visualStyle,
    brand_colors: brand.brandColors,
    quality: 'high',
    format: 'png'
  })
});
// returns: { url: 'https://cdn.nanobanana.io/...' }
```

---

## 12. Scheduling Engine — How It Works

### Overview

AutoLaunch has a built-in scheduling engine powered by **Temporal** — a durable workflow orchestrator. It handles all post scheduling, publishing, retries, and token management.

### Architecture

```
AutoLaunch Backend (NestJS)
    ↕ Triggers Temporal workflows
AutoLaunch Worker (Temporal Worker — apps/worker/)
    ↕ Executes at scheduled time
Social Platform APIs
    (LinkedIn, X, Instagram, WhatsApp, etc.)
```

### What the Calendar Dashboard Gives the Team

```
Visual calendar (monthly / weekly / daily views)
  → All draft posts appear on the calendar
  → Color-coded per platform
  → Drag and drop to any date + time slot
  → Platform-native post preview
  → Edit caption directly in the dashboard
  → Bulk scheduling support
  → Team collaboration
  → Optimal posting time suggestions
  → Post status: DRAFT → SCHEDULED → PUBLISHED
```

### How Temporal Works Inside AutoLaunch

When a post is scheduled via the calendar:

```
1. User clicks Schedule on a draft post in the calendar
      ↓
2. Backend creates a ScheduledPost record in PostgreSQL
      ↓
3. Backend calls Temporal Client to start a workflow:
   temporalClient.workflow.start('publishPost', {
     args: [scheduledPostId],
     taskQueue: 'autolaunch-publisher',
     workflowId: `post-${scheduledPostId}`,
     startDelay: timeUntilScheduledAt
   })
      ↓
4. Temporal workflow sleeps until scheduledAt datetime
      ↓
5. Temporal wakes at exact scheduled time
   → Calls PostActivity
      ↓
6. PostActivity fetches post details + OAuth token from DB
   → Calls correct social platform API
   → LinkedIn / X / Instagram / WhatsApp
      ↓
7. On SUCCESS:
   → Saves platformPostId + platformPostUrl to DB
   → Updates status to PUBLISHED
   → Fetches initial analytics
      ↓
   On FAILURE:
   → Temporal auto-retries (3 attempts, exponential backoff)
   → If all retries fail → status = FAILED, team notified
      ↓
8. Server restart or crash?
   → Temporal resumes from exactly where it stopped
   → No post is ever lost or skipped
```

### Token Refresh Workflow

OAuth tokens expire. AutoLaunch handles this automatically:
```
TokenRefreshWorkflow runs on a cron schedule
  → Checks all integrations for expiring tokens (< 24hr)
  → Calls platform refresh endpoint
  → Updates token in DB
  → No manual intervention needed
```

---

## 13. Component Breakdown

### Frontend Components (Next.js)

| Component | File Location | Purpose |
|---|---|---|
| `BriefForm` | `components/brief-form/` | Feature brief input with validation |
| `CalendarView` | `components/calendar/` | Visual drag-and-drop scheduling calendar |
| `ContentPreview` | `components/content-preview/` | All 4 platform previews |
| `PlatformCard` | `components/content-preview/` | Per-platform styled preview card |
| `ImagePreview` | `components/content-preview/` | 3-format image grid |
| `BrandSettings` | `components/brand-settings/` | Brand voice + tone + CTA configuration |
| `LaunchHistory` | `components/launches/` | Past launches table with status |
| `AnalyticsDashboard` | `components/analytics/` | Post performance charts |

### Backend Services (NestJS)

| Service | File | Purpose |
|---|---|---|
| `LaunchService` | `launches/launch.service.ts` | Launch CRUD + triggers ContentService |
| `ContentService` | `content/content.service.ts` | Orchestrates full AI pipeline |
| `GroqService` | `groq/groq.service.ts` | Groq API — enrichment + 4 captions |
| `NanoBananaService` | `nanobanana/image.service.ts` | Image generation — 3 variants |
| `ScheduleService` | `schedule/schedule.service.ts` | Creates + manages ScheduledPosts |
| `TemporalService` | `temporal/temporal.service.ts` | Starts + cancels Temporal workflows |
| `IntegrationService` | `integrations/integration.service.ts` | OAuth connect + token management |
| `BrandService` | `brand/brand.service.ts` | Brand config CRUD + AI injection |
| `AnalyticsService` | `analytics/analytics.service.ts` | Fetch + store post performance data |

### Worker Services (Temporal)

| Activity | File | Purpose |
|---|---|---|
| `PostActivity` | `activities/post.activity.ts` | Calls social platform API to publish |
| `AnalyticsActivity` | `activities/analytics.activity.ts` | Fetches post performance after publish |
| `post.workflow.ts` | `workflows/post.workflow.ts` | Waits until scheduledAt then fires PostActivity |
| `token.workflow.ts` | `workflows/token.workflow.ts` | Cron — refreshes expiring OAuth tokens |

---

## 14. Excalidraw Diagram Guide

Use these colors consistently across all diagrams:

| Color | Hex | Meaning |
|---|---|---|
| Indigo | `#3730A3` | AutoLaunch backend (our core code) |
| Emerald | `#0F9E6E` | Outputs / success / published posts |
| Orange | `#D97706` | External AI services (Groq, NanoBanana) |
| Purple | `#7C3AED` | AutoLaunch scheduling / Temporal engine |
| White | `#FFFFFF` | User-facing / inputs / frontend |
| Gray | `#6B7280` | Database / infrastructure |
| Coral | `#D84A30` | Problems / errors / failure paths |

---

### Diagram 1 — System Architecture (High Level)

Draw these rows of boxes:

```
Row 1 (white):   [AutoLaunch Next.js Dashboard]   [AutoLaunch Calendar + Analytics]

Row 2 (indigo):  [AutoLaunch NestJS Backend — AI Pipeline + API]

Row 3 (orange):  [Groq + Llama 3.2]          [NanoBanana]

Row 4 (purple):  [AutoLaunch Scheduling Engine — Temporal Worker]

Row 5 (emerald): [LinkedIn]  [X / Twitter]  [Instagram]  [WhatsApp]

Row 6 (gray):    [PostgreSQL]          [Redis]
```

Arrows:
```
Next.js Dashboard ────────────→ NestJS Backend (submit brief)
NestJS Backend ───────────────→ Groq Llama 3.2 (enrich + captions)
NestJS Backend ───────────────→ NanoBanana (images)
NestJS Backend ───────────────→ PostgreSQL (save content)
NestJS Backend ───────────────→ Scheduling Engine (create workflow)
Calendar Dashboard ───────────→ Scheduling Engine (pick date + time)
Scheduling Engine ────────────→ LinkedIn / X / Instagram / WhatsApp
```

---

### Diagram 2 — AI Pipeline (Detailed)

```
[Feature Brief Submitted]
         ↓
[Brand Config Fetched from DB]
         ↓
[Groq: Enrich Brief — 1 call]
         ↓
┌────────┬────────┬────────┬────────┬─────────────┐
│        │        │        │        │             │
▼        ▼        ▼        ▼        ▼             │
[LinkedIn][X]  [Instagram][WhatsApp][NanoBanana]  │
 Caption Caption  Caption   Message  Square+Story  │
                                     +Landscape    │
└────────┴────────┴────────┴────────┴─────────────┘
         ↓ (all complete via Promise.all)
[Assemble Content Package]
         ↓
[Save to PostgreSQL]
         ↓
[Appear in AutoLaunch Calendar as DRAFTS]
         ↓
[Team picks date + time → Clicks Schedule]
         ↓
[Temporal Workflow Created — sleeps until time]
         ↓
[PostActivity fires → Platform APIs called]
         ↓
[Posts Go Live ✅]
         ↓
[Analytics tracked in Dashboard]
```

---

### Diagram 3 — Scheduling Engine (Temporal Flow)

```
[User clicks Schedule on Calendar]
         ↓
[ScheduledPost saved to PostgreSQL]
         ↓
[Temporal Client starts publishPost workflow]
         ↓
[Workflow sleeps until scheduledAt datetime]
         ↓
[Temporal wakes at exact time]
         ↓
[PostActivity fires]
  ├── Fetches caption + image + OAuth token from DB
  └── Calls platform API (LinkedIn / X / Instagram / WhatsApp)
         ↓
  ┌──── SUCCESS ────┐      ┌──── FAILURE ───────────┐
  │                 │      │                        │
  ▼                 │      ▼                        │
[Save postId + URL] │   [Auto-retry (3 attempts)]   │
[Status → PUBLISHED]│   [If all fail → FAILED]      │
[Fetch analytics]   │   [Notify team]               │
  └─────────────────┘      └────────────────────────┘
```

---

### Diagram 4 — Database Entities (ERD)

```
Organization
  ├── has one    → Brand
  ├── has many   → Integration (LinkedIn, X, Instagram, WhatsApp)
  └── has many   → Launch
                     └── has one → Content
                                     ├── linkedinCaption
                                     ├── twitterCaption
                                     ├── instagramCaption
                                     ├── whatsappMessage
                                     ├── squareImageUrl
                                     ├── storyImageUrl
                                     ├── landscapeImageUrl
                                     └── has many → ScheduledPost
                                                       ├── platform
                                                       ├── scheduledAt
                                                       ├── status
                                                       └── temporalWorkflowId
User
  └── belongs to → Organization
```

---

### Diagram 5 — Full Sequence (Brief to Live Post)

Actors left to right:
```
User → Dashboard → NestJS → Groq → NanoBanana → PostgreSQL → Temporal → Platforms
```

Messages:
```
User          → Dashboard    Submit feature brief form
Dashboard     → NestJS       POST /api/launches
NestJS        → Groq         Enrich brief (Llama 3.2)
Groq          → NestJS       Enriched context returned
NestJS        → Groq         Generate 4 captions in parallel
Groq          → NestJS       4 platform captions returned
NestJS        → NanoBanana   Generate 3 image variants
NanoBanana    → NestJS       3 image URLs returned
NestJS        → PostgreSQL   Save full content package
NestJS        → Dashboard    Content ready — show preview
User          → Dashboard    Opens calendar, drags to date/time, clicks Schedule
Dashboard     → NestJS       POST /api/schedule
NestJS        → PostgreSQL   Create ScheduledPost records
NestJS        → Temporal     Start publishPost workflows
Temporal      → Platforms    Publish at exact scheduled time
Platforms     → Temporal     Post IDs + URLs returned
Temporal      → PostgreSQL   Update status to PUBLISHED
Dashboard     → User         Analytics visible in dashboard
```

---

## 15. Team Responsibilities

| Member | Slides | Technical Area |
|---|---|---|
| **Mrunmai** | Introduction | Project overview, Commudle context, user research |
| **Akhilesh** | Problem Statement, USP | Problem data, competitive analysis, differentiation |
| **Yash** | Proposed Solution, Impact & Benefits | AI pipeline, Groq integration, scheduling flow |
| **Bhavik** | Technical Approach, Architecture | NestJS backend, database schema, NanoBanana, Temporal |
| **All** | Architecture Diagram | Excalidraw — built together |

---

## 16. Roadmap

### V1 — Hackathon MVP
- Feature brief form (Next.js)
- NestJS AI pipeline (Groq + Llama 3.2)
- NanoBanana image generation (3 formats)
- Visual calendar scheduling dashboard
- Temporal publishing worker
- PostgreSQL + Redis data layer
- OAuth for LinkedIn, X, Instagram, WhatsApp
- Docker Compose full stack setup

### V2 — Post Hackathon (Month 1–2)
- Organization multi-tenancy
- Brand voice configuration UI
- GitHub / Linear / Notion webhook auto-triggers
- Content preview before scheduling
- Regenerate specific platform only
- Analytics dashboard

### V3 — Agency Product (Month 3–6)
- Multi-organization SaaS
- Campaign-level brand templates
- A/B caption variant generation
- Stripe billing integration
- White-label option

### V4 — SaaS Scale (Month 6–12)
- 10+ platform support
- Team collaboration + role-based approval
- AI analytics interpretation
- Public API + SDK for developers
- Enterprise SSO

---

## Quick Reference

| Resource | Detail |
|---|---|
| Groq Console | console.groq.com |
| Groq Model | llama-3.2-70b-versatile |
| NanoBanana | nanobanana.io |
| Temporal Docs | docs.temporal.io |
| Architecture Board | https://miro.com/app/board/uXjVGDtcCok=/ |

---

*AutoLaunch — From feature to feed, automatically.*
*Commudle Hackathon 2025 | Yash · Bhavik · Akhilesh · Mrunmai*
