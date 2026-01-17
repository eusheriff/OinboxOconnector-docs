# Oinbox - Real Estate CRM Platform

> **Live Demo**: [oinbox.oconnector.tech](https://oinbox.oconnector.tech)  
> **Contact**: [dev@oconnector.tech](mailto:dev@oconnector.tech)

---

## Overview

Oinbox is a comprehensive **Real Estate CRM SaaS Platform** designed to streamline property management, client interactions, and sales pipeline tracking for real estate agencies and independent brokers in Brazil.

### Key Features

| Feature | Description |
|---------|-------------|
| **AI-Powered Assistant** | Gemini-based consultant for property recommendations and client analysis |
| **Multi-Channel Inbox** | Unified messaging via WhatsApp, Email, and Portal integrations |
| **Lead Scoring** | Automatic qualification with temperature tracking (Cold/Warm/Hot) |
| **Property Management** | Listings, photos, portal syndication (OLX, ZAP Imóveis, VivaReal) |
| **CRM Pipeline** | Kanban-style deal tracking from lead to closing |
| **Marketing Studio** | Social media content generation and campaign management |
| **Client Portal** | White-label portal for property browsing and document exchange |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND                                │
│  React 18 + TypeScript + Vite + TailwindCSS                    │
│  Deployed: Cloudflare Pages                                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         BACKEND                                 │
│  Hono.js (Cloudflare Workers) + TypeScript                     │
│  Database: Cloudflare D1 (SQLite Edge)                         │
│  Storage: Cloudflare R2 (S3-compatible)                        │
│  AI: Cloudflare Workers AI + Google Gemini                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      INTEGRATIONS                               │
│  • Evolution API (WhatsApp)                                    │
│  • Stripe (Payments & Subscriptions)                           │
│  • Resend (Transactional Email)                                │
│  • Google Places API (Lead Capture)                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

### Frontend
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite 5
- **Styling**: TailwindCSS + custom design tokens
- **State**: React Context + custom hooks
- **Routing**: React Router v6
- **Testing**: Vitest + React Testing Library

### Backend
- **Runtime**: Cloudflare Workers (Edge)
- **Framework**: Hono.js v4
- **Database**: Cloudflare D1 (SQLite at the edge)
- **Object Storage**: Cloudflare R2
- **Authentication**: JWT (HS256)
- **Testing**: Vitest

### Infrastructure
- **CDN/Hosting**: Cloudflare (Workers + Pages)
- **CI/CD**: GitHub Actions
- **Monitoring**: Datadog (custom logger)
- **Payments**: Stripe (Checkout, Webhooks)

---

## Project Structure

```
oinbox/
├── src/                    # Frontend source
│   ├── components/         # React components
│   ├── pages/             # Route pages (admin, client)
│   ├── services/          # API service layer
│   ├── contexts/          # React contexts
│   ├── layouts/           # Page layouts
│   └── types.ts           # TypeScript definitions
├── backend/
│   └── src/
│       ├── routes/        # Hono route handlers
│       ├── middleware/    # Auth, logging, CORS
│       ├── services/      # Business logic
│       └── utils/         # Helpers (email, datadog)
├── shared/
│   └── types/             # Shared type definitions
├── docs/
│   └── _consolidated/     # Technical documentation
└── dist/                  # Production build output
```

---

## API Endpoints

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | User authentication |
| POST | `/api/auth/register` | New tenant registration |
| POST | `/api/auth/client/login` | Client portal login |

### Properties
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/properties` | List tenant properties |
| POST | `/api/properties` | Create new property |
| GET | `/api/properties/:id` | Get property details |
| DELETE | `/api/properties/:id` | Remove property |

### CRM
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/clients` | List clients |
| POST | `/api/clients` | Create client |
| POST | `/api/clients/:id/analyze` | AI lead scoring |

### AI
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/ai/generate-description` | Property description generation |
| POST | `/api/ai/chat` | AI consultant chat |

---

## Security

### Implemented Measures
- **JWT Authentication** with tenant isolation
- **CORS Allow-List** (explicit origin validation)
- **Anti-Fraud Trial Protection** (fingerprint tracking)
- **Password Hashing** (bcrypt)
- **Rate Limiting** (Cloudflare)

### Multi-Tenancy
Every request includes `x-tenant-id` header, enforced by middleware. Database queries are automatically scoped to tenant context.

---

## Development

### Prerequisites
- Node.js 20+
- npm 9+
- Wrangler CLI (Cloudflare)

### Setup
```bash
# Install dependencies
npm install

# Start development servers
npm run dev          # Frontend (Vite)
wrangler dev         # Backend (Workers)

# Run tests
npm test

# Build for production
npm run build
```

---

## Deployment

### Cloudflare Workers (Backend)
```bash
wrangler deploy
```

### Cloudflare Pages (Frontend)
Automatic deployment via GitHub push to `main` branch.

---

## Testing

```bash
# Run all tests
npm test

# Test coverage
npm test -- --coverage
```

### Test Files
- `src/services/apiService.test.ts` - API client tests
- `backend/test/auth.test.ts` - Authentication flows
- `backend/test/billing.test.ts` - Stripe webhook handling
- `backend/test/portals.test.ts` - XML feed generation

---

## License

Proprietary. All rights reserved.

---

## Contact

**Developer**: [dev@oconnector.tech](mailto:dev@oconnector.tech)  
**Website**: [oconnector.tech](https://oconnector.tech)
