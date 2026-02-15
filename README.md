# Artificial-Intelligencer
The Artificial Intelligencer — A single-admin editorial website for publishing AI-assisted creative works under multiple persona identities, built with React, TypeScript, and Firebase.

# The Artificial Intelligencer

A single-admin editorial website for publishing AI-assisted creative works under multiple persona identities, built with React, TypeScript, and Firebase.

## Overview

The Artificial Intelligencer presents as a professional multi-author news and opinion platform. Behind the scenes, a single administrator creates and manages AI-generated personas — reporters, commentators, and contributors — each publishing content developed through conversations with various LLMs (Claude, Grok, Gemini, Copilot). All content is manually curated and uploaded through a secure admin dashboard.

This is not an automated content generation tool. It is a showcase platform for human-AI collaborative creative output.

## Features

### Public Site
- Responsive article feed with category filtering and client-side search
- Individual article pages with shareable URLs
- Persona profile pages with biography and published article listings
- Mobile-first responsive design (3 → 2 → 1 column grid)
- Global site privacy toggle (take the whole site offline with one switch)

### Admin Dashboard
- Secure email/password authentication (single admin user)
- Full CRUD for personas (name, bio, role, profile image, external links)
- Full CRUD for articles (title, body, excerpt, featured image, category, tags, visibility)
- Image upload to Firebase Storage with preview
- Per-article public/private visibility controls
- Site-wide settings management

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 19+, TypeScript, Vite |
| Styling | Tailwind CSS, shadcn/ui |
| Routing | React Router 6+ |
| Backend | Firebase (Auth, Firestore, Storage) |
| Hosting | Firebase Hosting |

## Project Structure

```
src/
├── components/        # Reusable UI components
│   ├── ArticleCard    # Article preview card for feed
│   ├── PersonaCard    # Persona display component
│   ├── Header         # Site navigation header
│   └── AdminBar       # Persistent admin navigation
├── pages/             # Route-level page components
│   ├── HomePage       # Public article feed
│   ├── ArticlePage    # Full article view
│   ├── PersonaPage    # Persona profile view
│   ├── LoginPage      # Admin authentication
│   └── AdminDashboard # Content management interface
├── lib/
│   ├── firebase.ts    # Firebase configuration and initialization
│   └── db.ts          # Firestore read/write operations
├── context/
│   └── AuthContext.tsx # Authentication state management
└── App.tsx            # Root component with routing
```

## Prerequisites

- Node.js 18+
- A Firebase project with Authentication, Firestore, and Storage enabled
- An admin user created in Firebase Console → Authentication → Users

## Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/[your-username]/artificial-intelligencer.git
   cd artificial-intelligencer
   ```

2. Install dependencies:
   ```bash
   pnpm install
   ```

3. Configure Firebase — update `src/lib/firebase.ts` with your project credentials:
   ```typescript
   const firebaseConfig = {
     apiKey: "your-api-key",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project-id",
     storageBucket: "your-project.firebasestorage.app",
     messagingSenderId: "your-sender-id",
     appId: "your-app-id"
   };
   ```

4. Start the development server:
   ```bash
   pnpm dev
   ```

5. Log in at `/admin/login` with the credentials you created in Firebase Console.

## Firebase Security Rules

### Firestore
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

### Storage
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

## Deployment

### Firebase Hosting (Primary)
```bash
npm install -g firebase-tools
firebase login
firebase init hosting    # Public directory: dist, SPA: Yes
pnpm build
firebase deploy --only hosting
```

### Netlify (Alternative)
Drag the `dist` folder to [app.netlify.com/drop](https://app.netlify.com/drop). Add your Netlify URL to Firebase → Authentication → Authorized domains.

## Design Specification

The full design spec (data schemas, edge cases, behavioral rules, visual identity, and build strategy) is maintained in `AIer_Design_Specification_v2.docx` in this repository.

## Status

🚧 **In Development** — Building from consolidated design spec (v2.0, February 2026).

## License

This project is personal/portfolio work. All persona-generated content is original AI-assisted creative output by the site administrator.
