# The Artificial Intelligencer (AIer) — Consolidated Design Specification v2.0

> **Build-Ready Specification** | February 2026
>
> This document consolidates the best ideas from over ten separate design conversations
> conducted between August 2025 and February 2026 across multiple AI platforms.
> It represents the definitive, build-ready specification for The Artificial Intelligencer website.

---

## Document Purpose

- This is the **single source of truth** for building the AIer website.
- It **replaces all previous specs**, partial builds, and planning documents.
- Any AI coding agent should be able to build from this spec alone.
- Designed for **iterative, phased construction** — not a single-shot generation.

---

## Table of Contents

1. [Project Overview & Vision](#1-project-overview--vision)
2. [Core Concept & Strategic Purpose](#2-core-concept--strategic-purpose)
3. [Technology Stack](#3-technology-stack)
4. [Database Schema](#4-database-schema)
5. [Authentication & Security](#5-authentication--security)
6. [Public Pages — Frontend Features](#6-public-pages--frontend-features)
7. [Admin Dashboard — Backend Features](#7-admin-dashboard--backend-features)
8. [Design System & Visual Identity](#8-design-system--visual-identity)
9. [Edge Cases & Behavioral Specification](#9-edge-cases--behavioral-specification)
10. [Phased Build Strategy](#10-phased-build-strategy)
11. [Deployment & Infrastructure](#11-deployment--infrastructure)
12. [Success Criteria](#12-success-criteria)
13. [Future Roadmap (v2.0+)](#13-future-roadmap-v20)
14. [Appendix A: Firebase Configuration Checklist](#appendix-a-firebase-configuration-checklist)
15. [Appendix B: Lessons Learned from Past Builds](#appendix-b-lessons-learned-from-past-builds)

---

## 1. Project Overview & Vision

### Project History

The AIer concept originated as a fictional news aggregator for a movie concept, then evolved into a real platform. Multiple build attempts were made with Claude (at least 3 using React + Firebase), Gemini Code Assist, and Famous.AI. One build achieved deployment to Netlify (chat c6ffb9ba) using React 19 + TypeScript + Firebase, but was never fully operationalized due to context-limit issues during builds and gaps in the builder's ability to debug deployment problems.

### Key Lessons from Past Builds

- Builds that tried to generate everything at once hit context limits and failed
- Firebase was successfully set up (project: `artificial-intelligencer-c1`) and should be reused
- The gap between "I can describe what I want" and "I can debug why it broke" is the primary risk
- Phased, incremental builds with testing checkpoints are essential
- The spec needs behavioral definitions (edge cases), not just feature lists
- Music player and rich-text editor were identified as highest-complexity features

---

## 2. Core Concept & Strategic Purpose

### What It Is

A manually-managed editorial website that presents as a professional multi-author news and opinion platform. The sole administrator (Chaz) publishes AI-assisted creative works under various persona identities. All content is generated through sophisticated conversations with multiple LLMs (Grok, Gemini, Claude, Copilot) and manually uploaded.

### What It Is NOT

- **NOT** an automated content generation platform (no API-injected content in v1)
- **NOT** a multi-user CMS (single admin only, no registration flow)
- **NOT** a monetized site (no ads, subscriptions, or paywalls)
- **NOT** a social platform (no comments, likes, or user accounts)

### Strategic Purpose

1. Portfolio demonstration of human-AI collaborative methodology
2. Showcase platform for AI persona creative output (articles, commentary, music)
3. Digital footprint building for credibility in the AI/LLM community
4. Foundation for future Substack analysis of process and methodology
5. Proof-of-concept for the "penname protocol" system developed with Grok
6. Connection point to broader creative ecosystem (Safety! The Musical, four creative personas)

### Persona Architecture

| Slot | Role | Content Type | Description |
|------|------|-------------|-------------|
| 1-4 | Reporter | Factual news-style | Objective reporting voice, various beats |
| 5-8 | Commentator | Opinion/analysis | Editorial voice, perspective pieces |
| 9 | New Contributor | Experimental | Guest or new persona testing slot |

Start with 1 persona for initial build. Admin can create additional personas (up to 9+ slots) through the dashboard. The persona count is not hard-coded; the architecture supports unlimited personas.

---

## 3. Technology Stack

### Stack Decision Rationale

After extensive discussion across multiple conversations, the consensus is:

> **Use Firebase for v1** — it is already set up, already proven in a working build,
> and matches the simple document-oriented data model of this project.
> Supabase (suggested by Gemini) is better for v2.0 if relational data or SQL is needed.
> Firebase + React is the path of least resistance for getting v1 live.

### Frontend

| Technology | Version | Purpose |
|-----------|---------|---------|
| React | 19+ | UI framework — component-based, well-documented |
| TypeScript | 5.x | Type safety, better error catching |
| Vite | 7+ | Build tool — fast dev server, optimized production builds |
| Tailwind CSS | 3.x | Utility-first styling — rapid development |
| shadcn/ui | Latest | Pre-built UI components — professional appearance |
| React Router | 6+ | Client-side routing for SPA |

### Backend (Firebase — Backend-as-a-Service)

| Service | Purpose | Notes |
|---------|---------|-------|
| Firebase Auth | Admin login (email/password) | Single user, no registration flow |
| Cloud Firestore | Document database | Personas, articles, site settings |
| Firebase Storage | Image hosting | Persona avatars, article images |
| Firebase Hosting | Primary deployment | CDN, HTTPS, custom domain support |
| Google Analytics | Passive tracking (optional) | Enabled but no integration needed |

### Existing Firebase Project

- **Project ID:** `artificial-intelligencer-c1`
- **Hosting URL:** https://artificial-intelligencer-c1.web.app
- **Status:** Auth, Firestore, Storage, and Hosting already enabled from previous builds.

### Rich Text Editor (Phase 2+)

For v1, use a simple textarea for article body text. Plain text with line breaks is sufficient for launch. In Phase 2, integrate **Tiptap** for rich text editing with the understanding that it is the single highest-complexity feature and should be isolated as its own build sprint.

### Music Player Decision

> **DEFER TO v2.0.** The music player with auto-pause on YouTube embed requires
> cross-component state management that adds significant complexity.
> For v1, if music is needed, link to a dedicated "Listen" page or embed
> a SoundCloud/Bandcamp player — 80% of the experience at 20% of the cost.

---

## 4. Database Schema

Firestore is a NoSQL document database. Collections hold documents, and each document is a flat key-value structure. The schema below incorporates feedback from the February 2026 design review addressing missing timestamps, sort ordering, and tag support.

### Collection: `personas`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string (auto) | Yes | Auto-generated document ID |
| name | string | Yes | Persona display name |
| bio | string | Yes | Short biography (1-3 paragraphs) |
| role | string | Yes | `"reporter"` \| `"commentator"` \| `"contributor"` |
| profileImageUrl | string | No | Firebase Storage URL |
| moreInfoText | string | No | Extended lore/background (v1: plain text) |
| externalLinks | map | No | `{ youtube: url, social: url, etc. }` |
| displayOrder | number | No | Manual sort order (0-100) |
| isActive | boolean | Yes | Soft-delete flag (default: true) |
| createdAt | timestamp | Yes | Auto-set on creation |
| updatedAt | timestamp | Yes | Auto-set on edit |

### Collection: `articles`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string (auto) | Yes | Auto-generated document ID |
| title | string | Yes | Article headline |
| body | string | Yes | Full article text (plain text v1, HTML v2) |
| excerpt | string | Yes | Preview text (150-200 chars, auto or manual) |
| featuredImageUrl | string | No | Firebase Storage URL |
| personaId | string | Yes | Reference to persona document |
| personaName | string | Yes | Denormalized for display performance |
| category | string | No | Primary category (US News, Sports, etc.) |
| tags | array\<string\> | No | Additional tags for future filtering |
| style | string | No | Content style (satire, commentary, analysis) |
| isPublic | boolean | Yes | Visibility toggle (default: true) |
| publishedAt | timestamp | No | Scheduled/backdated publish date |
| createdAt | timestamp | Yes | Auto-set on creation |
| updatedAt | timestamp | Yes | Auto-set on edit |

### Collection: `settings` (single document: `"site"`)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| isPublic | boolean | true | Global site visibility toggle |
| siteName | string | The Artificial Intelligencer | Configurable site title |
| tagline | string | (editable) | Site subtitle/tagline |

### Storage Structure

```
/persona-images/{personaId}-{timestamp}.{ext}
/article-images/{articleId}-{timestamp}.{ext}
```

### Key Schema Decisions

- `personaName` is denormalized on articles for display performance — update it if persona name changes
- Soft-delete via `isActive` on personas prevents orphaned articles
- `publishedAt` is separate from `createdAt` to allow backdating and scheduling
- `tags` is an array for future multi-category support; `category` remains the primary single category
- `displayOrder` on personas allows manual control of listing order

---

## 5. Authentication & Security

### Authentication Model

- Firebase Authentication with email/password provider
- Single admin user only — **NO registration flow, NO password reset UI**
- Admin user is created manually in Firebase Console before deployment
- Simple login page at `/admin/login` with email and password fields
- Successful login redirects to `/admin/dashboard`
- Session persists across browser tabs (Firebase default behavior)

### Route Protection

- All `/admin/*` routes are protected by an `AuthContext` wrapper
- Unauthenticated access to `/admin/*` redirects to `/admin/login`
- Public routes (`/`, `/article/:id`, `/persona/:id`) are always accessible
- When `isPublic = false` in site settings: public routes show a "Site is currently private" message, admin routes still function normally

### Firestore Security Rules

Public read access for personas and public articles. Admin-only write access for everything. Settings readable by all (to check site visibility), writable only by authenticated user. Full rules are provided in [Appendix A](#appendix-a-firebase-configuration-checklist).

### Storage Security Rules

Public read for all images (required for public display). Authenticated write only (admin uploads). File validation enforced client-side: JPG/PNG only, max 5MB.

---

## 6. Public Pages — Frontend Features

### Homepage / Article Feed

**Route:** `/`

- Static header with site logo/title and navigation
- Scrollable feed of article cards in a responsive grid:
  - **Desktop (1024px+):** 3 columns
  - **Tablet (768px+):** 2 columns
  - **Mobile (<768px):** 1 column
- Each article card displays: featured image, title, excerpt, persona name, publish date
- Cards link to full article page
- Category filter bar (sticky below header) for filtering visible articles
- Client-side search: simple text filter on titles and excerpts (no backend search)
- Articles sorted by `publishedAt` descending (newest first)
- Pagination or infinite scroll for large article counts

### Article Detail Page

**Route:** `/article/:id`

- Full article body text
- Featured image displayed prominently
- Persona attribution with clickable link to persona profile
- Category and style tags displayed
- Publication date
- Shareable URL (each article has a unique, permanent link)
- Optional: Related articles section (same persona or category)

### Persona Profile Page

**Route:** `/persona/:id`

- Profile image
- Name and role badge (Reporter / Commentator / Contributor)
- Biography text
- Dynamic list of published articles by this persona
- Optional: "More Info" section for extended lore/background
- Optional: External links (YouTube, social media, etc.)

### Site-Wide Privacy Behavior

When site settings `isPublic = false`: All public routes display a simple branded "The Artificial Intelligencer is currently offline. Check back soon." page. No article data is exposed. Admin routes continue to function normally.

---

## 7. Admin Dashboard — Backend Features

**Route:** `/admin/dashboard`

Single-page admin dashboard with tab navigation. Persistent admin bar at top of all admin pages with link to view public site and logout button.

### Tab 1: Personas

- List all personas with name, role, image thumbnail, article count
- **Create new persona:** name (required), bio (required), role (required), profile image (upload), display order
- **Edit existing persona:** all fields editable
- **Delete persona:** only allowed if persona has zero articles. Show error with article count if attempted. Soft-delete via `isActive = false` is preferred over hard delete.

### Tab 2: Articles

- List all articles with title, persona, category, publish date, public/private status
- **Create new article:** title (required), body (required), excerpt (auto-generated or manual), featured image (upload), persona author (dropdown of active personas), category, style, tags, publishedAt date picker, isPublic toggle
- **Edit existing article:** all fields editable
- **Delete article:** confirmation dialog required
- Cannot create article if no personas exist — show message directing admin to create a persona first

### Tab 3: Site Settings

- Global site visibility toggle (public/private)
- Site name and tagline (editable)
- Preview: link to open public site in new tab

### Admin UX Requirements

- Form validation on all required fields with clear error messages
- Image upload with preview before submission
- Upload progress indicator
- Loading states during all async operations
- Success/error toasts for all CRUD operations
- Confirmation dialogs for all delete operations

---

## 8. Design System & Visual Identity

### Design Philosophy

Clean, professional editorial aesthetic. Newspaper-inspired with modern web sensibilities. The site should look like a legitimate multi-author publication, not a student project or AI demo. High contrast for readability. Minimal but polished.

### Typography

| Use | Font | Fallback | Notes |
|-----|------|----------|-------|
| Display/Headlines | Playfair Display | Georgia, serif | Elegant, editorial feel |
| Body Text | Crimson Pro | Georgia, serif | Readable, warm editorial text |
| Technical/Meta | DM Mono | Courier, monospace | Dates, categories, tags, metadata |

### Color Palette

| Name | Hex | Use |
|------|-----|-----|
| Ink Black | `#0a0a0a` | Primary text, headers, hero backgrounds |
| Charcoal | `#1a1a1a` | Secondary backgrounds, footer |
| Parchment | `#f8f6f0` | Page backgrounds, card backgrounds |
| Warm Gray | `#6b6b6b` | Secondary text, metadata |
| Slate | `#3a3a3a` | Borders, dividers |
| Accent Rust | `#b85c38` | Primary accent — links, CTAs, hover states |
| Deep Crimson | `#8b1e3f` | Secondary accent — avatars, badges |
| Antiqued Gold | `#c9a961` | Tertiary accent — persona roles, highlights |
| Editorial Teal | `#2d6a6a` | Optional accent — tags, categories |

### Layout Principles

- Maximum content width: 1200px, centered
- Consistent spacing scale using CSS custom properties
- Sticky header with site navigation
- Category filter bar sticky below header
- Responsive grid: 3 -> 2 -> 1 columns at standard breakpoints
- Cards have subtle hover effects (translateY, border color change)
- Smooth scroll-triggered fade-in animations (respect `prefers-reduced-motion`)
- Print styles: hide header, footer, navigation

### Glassmorphism Header (Optional)

The previous spec included a glassmorphism (`backdrop-filter` blur) header. This is visually attractive but inconsistent on mobile Safari. **Recommendation:** implement with a solid dark fallback for browsers that don't support `backdrop-filter`.

---

## 9. Edge Cases & Behavioral Specification

> This section was identified as a critical gap in the February 2026 design review.
> The unglamorous edge cases are where low-code builds tend to fall apart.

### Empty States

- **No articles yet:** Homepage shows "No articles yet. Check back soon!" with site branding
- **No personas yet:** Admin article form disables persona dropdown, shows "Create a persona first" message
- **Persona with no articles:** Profile page shows "No articles published yet"
- **Search with no results:** Show "No articles match your search" with option to clear filter

### Error States

- **Image upload fails:** Show error message, preserve form data, allow retry
- **Article fails to save:** Show error toast, preserve form data, do not navigate away
- **Article not found (bad URL):** Show 404 page with link to homepage
- **Firebase connection lost:** Show offline banner, disable write operations
- **Image fails to load on public page:** Show placeholder/fallback image

### Deletion Logic

- **Delete persona with articles:** BLOCK. Show error "This persona has X articles. Delete or reassign them first."
- **Delete article:** Require confirmation dialog. Hard delete from Firestore. Delete associated image from Storage.
- **Preferred:** Soft-delete personas (`isActive = false`). Hard-delete articles.

### Data Integrity

- If persona name changes, update `personaName` on all associated articles
- Excerpt auto-generation: first 150-200 characters of body text, trimmed to last complete word
- Slug generation for URLs: lowercase, hyphenated, stripped of special characters, conflict-checked

### Mobile-Specific

- Touch-friendly tap targets (minimum 44px)
- Swipe-friendly card layouts
- Admin dashboard must be functional (not just pretty) on tablet
- Image upload works from mobile camera roll

---

## 10. Phased Build Strategy

### Critical Build Philosophy

> **DO NOT** attempt to generate the entire application in one session.
> Each phase should be built, tested, and verified before moving to the next.
> Each phase should produce a working (if incomplete) application.
> This approach was identified as the #1 success factor from past build attempts.

### Phase 1: Foundation (Get something rendering)

1. Initialize React + TypeScript + Vite project
2. Install Tailwind CSS and shadcn/ui
3. Set up React Router with placeholder pages
4. Create Firebase configuration file (`firebase.ts`) using existing project credentials
5. Build basic app shell: header, routing, footer

**Test checkpoint:** App renders in browser with navigation between empty pages.

### Phase 2: Public Pages with Static Data

1. Build `ArticleCard` component with hardcoded sample data
2. Build `HomePage` with responsive article grid
3. Build `ArticlePage` with full article display
4. Build `PersonaPage` with profile and article list
5. Implement responsive layout (3/2/1 columns)
6. Style everything to match design system

**Test checkpoint:** Public pages render beautifully with static data. All navigation works.

### Phase 3: Firebase Integration

1. Wire Firestore reads to `HomePage` (fetch articles)
2. Wire Firestore reads to `ArticlePage` and `PersonaPage`
3. Add loading states and error handling
4. Seed Firestore with test data via Firebase Console
5. Implement client-side search filter

**Test checkpoint:** Public pages display data from Firestore. Search works.

### Phase 4: Authentication & Admin Shell

1. Build `AuthContext` with Firebase Auth listener
2. Build `LoginPage` with email/password form
3. Build `ProtectedRoute` component
4. Build `AdminDashboard` shell with tab navigation
5. Build persistent admin bar

**Test checkpoint:** Can log in, see admin dashboard, and navigate tabs. Unauthenticated users are redirected.

### Phase 5: Admin CRUD Operations

1. Build `PersonaForm` (create/edit) with image upload to Firebase Storage
2. Build `PersonasSection` with list, edit, delete
3. Build `ArticleForm` (create/edit) with image upload and persona dropdown
4. Build `ArticlesSection` with list, edit, delete
5. Implement all form validation and error handling
6. Implement deletion logic with safeguards

**Test checkpoint:** Full CRUD works. Can create persona, create article, edit both, delete article.

### Phase 6: Settings & Polish

1. Build Settings tab with global privacy toggle
2. Implement site privacy behavior on public routes
3. Add per-article privacy toggles
4. Add category filter bar to homepage
5. Add success/error toasts for all operations
6. Cross-browser testing and mobile responsiveness audit

**Test checkpoint:** All v1 features work. Site looks professional on all devices.

### Phase 7: Deployment

1. Production build (`vite build`)
2. Deploy to Firebase Hosting (primary) or Netlify (backup)
3. Verify Firebase authorized domains
4. Verify all security rules in production
5. Final smoke test of all features on live URL

**Test checkpoint:** Site is live, admin can log in, create content, and public users can browse.

### Phase 8+ (Future)

*Separate build sprints:*

- Rich text editor (Tiptap) integration
- Music player with SoundCloud/Bandcamp embed
- SEO optimization (meta tags, sitemap, RSS)
- YouTube embed support in articles
- Analytics dashboard for admin
- Custom domain setup

---

## 11. Deployment & Infrastructure

### Primary: Firebase Hosting

Already configured in project `artificial-intelligencer-c1`. Previous builds have been successfully deployed here.

- Automatic HTTPS/SSL
- Global CDN distribution
- Custom domain support
- Integrated with Firebase project
- Deploy command: `firebase deploy --only hosting`
- URL: https://artificial-intelligencer-c1.web.app

### Backup: Netlify

Previous successful deployment was made to Netlify via drag-and-drop of the `dist` folder. Free tier provides subdomain (`*.netlify.app`), HTTPS, and CDN. Remember to add Netlify URL to Firebase authorized domains.

### Backup: Vercel

Third option. Good developer experience, fast builds, Git integration. Similar setup to Netlify.

### Environment Variables

Firebase configuration values can be hardcoded in the `firebase.ts` config file (they are not secret — Firebase security is enforced by security rules, not by hiding config keys). However, for cleanliness, a `.env` file can store them.

---

## 12. Success Criteria

The website is successfully built when **ALL** of the following are true:

### Admin Functions

- Admin can log in securely with email/password
- Admin can create, edit, and delete personas
- Admin can create, edit, and delete articles
- Admin can upload images for both personas and articles
- Admin can toggle individual article visibility
- Admin can toggle global site visibility
- Admin can easily switch between dashboard and public view
- All forms validate required fields
- All delete operations require confirmation

### Public Experience

- Public users can view article feed on homepage
- Public users can read full articles at unique URLs
- Public users can view persona profiles
- Public users can filter by category
- Public users can search article titles/excerpts
- Site is mobile responsive and looks professional
- Images display correctly on all pages
- All navigation links work

### Technical Quality

- No console errors in normal operation
- Loading states shown during data fetching
- Error states handled gracefully
- Site loads in under 3 seconds on broadband
- Production build is under 1MB (gzipped under 300KB)

---

## 13. Future Roadmap (v2.0+)

These features were discussed across multiple conversations and represent the vision for the platform beyond v1. They are explicitly **OUT OF SCOPE** for the initial build.

### Content Enhancements

- Rich text editor (Tiptap) with YouTube embed support
- Music player (persistent, auto-pause on video embed)
- Image gallery/media library in admin
- Article scheduling (future publish dates)
- Draft/revision history for articles
- Code block syntax highlighting for technical pieces

### Discovery & SEO

- Full-text search (Algolia or Supabase PostgreSQL search)
- SEO metadata per article (title tags, Open Graph images)
- Sitemap generation
- RSS feed (personas as "columnists")
- Related articles recommendation
- Archive/chronological navigation

### Platform Evolution

- API-injected content generation (connect LLMs directly)
- Cross-platform persona calibration tools
- Track which AI system generated which content
- Multi-admin support
- Migration to Supabase (PostgreSQL) for relational queries
- Custom domain and branding
- Email newsletter integration
- Export/backup tools for content

### Creative Ecosystem

- Substack integration for methodology analysis
- Connection to Safety! The Musical project
- Persona "calibration" interface
- A/B testing different persona prompts
- Guest submission portal

---

## Appendix A: Firebase Configuration Checklist

Complete this checklist **BEFORE** starting any build session. All of these should already be done from previous builds, but verify each one.

### Firebase Console (console.firebase.google.com)

#### 1. Authentication

- [ ] Email/Password provider is enabled
- [ ] Admin user is created (Users tab -> Add user)
- [ ] Authorized domains include `localhost` AND deployment domain

#### 2. Firestore Database

- [ ] Database is created (production mode)
- [ ] Security rules allow public read for personas and public articles
- [ ] Security rules allow authenticated write for everything
- [ ] Rules include settings document access

#### 3. Firebase Storage

- [ ] Storage is enabled
- [ ] Rules allow public read for all files
- [ ] Rules allow authenticated write only

#### 4. Firebase Config Values

Retrieve from Project Settings -> Your Apps -> Web app config:

- [ ] `apiKey`
- [ ] `authDomain`
- [ ] `projectId`: `artificial-intelligencer-c1`
- [ ] `storageBucket`
- [ ] `messagingSenderId`
- [ ] `appId`
- [ ] `measurementId`

### Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /personas/{personaId} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    match /articles/{articleId} {
      allow read: if resource.data.isPublic == true || request.auth != null;
      allow write: if request.auth != null;
    }
    match /settings/site {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

### Storage Security Rules

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

---

## Appendix B: Lessons Learned from Past Builds

These lessons were collected across 3+ build attempts and 10+ planning conversations. They represent hard-won knowledge about what works and what doesn't when building this specific project with AI assistance.

### What Worked

1. Phased, incremental builds with clear testing checkpoints at each stage
2. Providing Firebase credentials upfront so the AI doesn't build unnecessary registration flows
3. Using a simplified specification that removes ambiguity
4. Deploying to Netlify via drag-and-drop for fast verification
5. Starting with static data before wiring up the database

### What Didn't Work

1. Attempting to generate the entire application in a single chat session (hit context limits every time)
2. Leaving the search feature in v1 scope (added unnecessary complexity)
3. Not specifying edge cases (empty states, error states, deletion logic) led to broken UI
4. Mixing Firebase and Supabase suggestions across conversations created confusion
5. Overly detailed aesthetic specs without behavioral specs (every hover state defined, but no empty state defined)

### Recommendations for the Build Agent

1. Read this entire spec before writing any code
2. Build one phase at a time and verify it works before proceeding
3. Use the Firebase project that already exists (`artificial-intelligencer-c1`)
4. Start with Phase 1 and do NOT skip ahead
5. When in doubt about a design decision, choose the simpler option
6. If a feature is not in this spec, do not add it
7. Test every CRUD operation before declaring a phase complete

---

*END OF SPECIFICATION*
*The Artificial Intelligencer v2.0 Design Spec — Build Ready*
