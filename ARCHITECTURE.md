# Architecture Overview

This document describes the technical architecture of the Oinbox platform.

## System Design

### High-Level Architecture

```
                    ┌──────────────────┐
                    │   Cloudflare     │
                    │   CDN + WAF      │
                    └────────┬─────────┘
                             │
         ┌───────────────────┴───────────────────┐
         │                                       │
         ▼                                       ▼
┌─────────────────────┐             ┌─────────────────────┐
│  Cloudflare Pages   │             │  Cloudflare Workers │
│  (Static Frontend)  │             │  (API Backend)      │
│                     │             │                     │
│  • React SPA        │  ────────►  │  • Hono.js          │
│  • Vite build       │   REST API  │  • JWT Auth         │
│  • TailwindCSS      │             │  • Tenant Isolation │
└─────────────────────┘             └──────────┬──────────┘
                                               │
                    ┌──────────────────────────┼──────────────────────────┐
                    │                          │                          │
                    ▼                          ▼                          ▼
           ┌───────────────┐          ┌───────────────┐          ┌───────────────┐
           │ Cloudflare D1 │          │ Cloudflare R2 │          │ External APIs │
           │   (SQLite)    │          │   (Storage)   │          │               │
           │               │          │               │          │ • Stripe      │
           │ • Users       │          │ • Images      │          │ • Evolution   │
           │ • Properties  │          │ • Documents   │          │ • Resend      │
           │ • Leads       │          │ • Exports     │          │ • Gemini AI   │
           └───────────────┘          └───────────────┘          └───────────────┘
```

## Component Details

### Frontend (React SPA)

**Technology Stack:**
- React 18 with TypeScript
- Vite 5 for build tooling
- TailwindCSS for styling
- React Router v6 for navigation

**Key Features:**
- Code splitting for optimal loading
- Responsive design (mobile-first)
- PWA support with service worker
- Lazy loading for heavy components (Map, Charts)

**Bundle Optimization:**
```javascript
// vite.config.ts - Manual chunks
manualChunks: {
  'vendor-react': ['react', 'react-dom'],
  'vendor-ui': ['lucide-react', 'clsx'],
  'vendor-map': ['leaflet', 'react-leaflet'],
  'vendor-stripe': ['@stripe/stripe-js'],
}
```

### Backend (Cloudflare Workers)

**Technology Stack:**
- Hono.js v4 (Express-like API framework)
- TypeScript
- Cloudflare D1 (SQLite edge database)
- Cloudflare R2 (object storage)

**Request Flow:**
```
Request → CORS → Logging → Auth → Route Handler → Response
```

**Multi-Tenancy:**
Every API request includes `x-tenant-id` header. Middleware validates JWT and ensures tenant isolation at the database query level.

### Database Schema (Simplified)

```sql
-- Core Tables
tenants (id, name, owner_name, email, plan, status, trial_ends_at)
users (id, tenant_id, name, email, password_hash, role)
clients (id, tenant_id, name, email, phone, status, lead_score)
properties (id, tenant_id, title, price, location, features, image_url)

-- Messaging
conversations (id, tenant_id, contact_name, platform, last_message)
messages (id, conversation_id, sender_id, text, timestamp, type)

-- CRM
deals (id, tenant_id, conversation_id, property_id, value, stage)

-- Lead Capture
leads (id, google_place_id, name, type, score, status, source)
trial_fingerprints (id, email, phone, device_id, tenant_id)
```

## Integration Architecture

### WhatsApp (Evolution API)

```
User Message → Evolution API Webhook → Worker
                                         │
                                         ▼
                                   AI Processing
                                         │
                                         ▼
                             Store in D1 + Send Response
                                         │
                                         ▼
                                   Evolution API → WhatsApp
```

### Payments (Stripe)

```
Checkout → Stripe Session → Success/Cancel Redirect
                │
                ▼ (Webhook)
        Worker /api/billing/webhook
                │
                ▼
        Update tenant plan in D1
                │
                ▼
        Send welcome email via Resend
```

## Security Architecture

### Authentication Flow

```
1. User submits credentials
2. Server validates against D1
3. Generate JWT with claims: {sub, tenantId, role, name}
4. Client stores token in localStorage
5. All API requests include: Authorization: Bearer <token>
6. Middleware validates token + extracts context
```

### CORS Configuration

```typescript
cors({
  origin: (origin) => {
    const allowed = [
      'https://oinbox.oconnector.tech',
      'https://www.oinbox.oconnector.tech',
      'http://localhost:5173',
    ];
    return allowed.includes(origin) ? origin : null;
  },
  credentials: true,
})
```

## Observability

### Logging

Custom Datadog integration for structured logging:

```typescript
const logger = createDatadogLogger(env);
await logger.info('Request processed', {
  path: '/api/properties',
  tenantId: 'tenant-123',
  duration: 45,
});
```

### Health Checks

```
GET /api/health → { status: 'ok', version: '1.0.0' }
```

## Scalability Considerations

1. **Edge Computing**: Workers run in 300+ data centers globally
2. **Database**: D1 with automatic replication
3. **Storage**: R2 with unlimited scaling
4. **Stateless**: No server-side sessions

---

## Contact

**Developer**: [dev@oconnector.tech](mailto:dev@oconnector.tech)
