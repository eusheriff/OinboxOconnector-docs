# API Reference

Complete API documentation for the Oinbox platform.

**Base URL**: `https://api.oinbox.oconnector.tech/api`

---

## Authentication

All endpoints (except `/auth/login` and `/auth/register`) require authentication via JWT Bearer token.

```http
Authorization: Bearer <token>
x-tenant-id: <tenant-uuid>
```

---

## Endpoints

### Auth

#### POST /auth/login
Authenticate user and receive JWT token.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "secret123"
}
```

**Response (200):**
```json
{
  "user": {
    "id": "uuid",
    "name": "John Doe",
    "email": "user@example.com",
    "role": "admin"
  },
  "tenantId": "tenant-uuid",
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "plan": "Pro",
  "trialEndsAt": null
}
```

#### POST /auth/register
Create new tenant and admin user.

**Request:**
```json
{
  "name": "John Doe",
  "email": "john@company.com",
  "password": "secure123",
  "companyName": "Real Estate Corp",
  "phone": "11999999999"
}
```

**Response (200):**
```json
{
  "success": true,
  "tenantId": "new-tenant-uuid",
  "userId": "new-user-uuid",
  "trialDays": 14
}
```

---

### Properties

#### GET /properties
List all properties for current tenant.

**Response (200):**
```json
[
  {
    "id": "prop-uuid",
    "title": "Luxury Apartment",
    "price": 500000,
    "location": "Downtown, São Paulo",
    "image": "https://r2.../image.jpg",
    "features": ["Pool", "Gym", "Parking"],
    "status": "Active",
    "listingType": "sale"
  }
]
```

#### POST /properties
Create new property listing.

**Request:**
```json
{
  "title": "Modern House",
  "price": 750000,
  "location": "Jardins, SP",
  "features": ["Garden", "3 Bedrooms"],
  "listingType": "sale"
}
```

#### GET /properties/:id
Get single property by ID.

#### DELETE /properties/:id
Remove property listing.

---

### Clients

#### GET /clients
List all clients.

**Response (200):**
```json
[
  {
    "id": "client-uuid",
    "name": "Maria Silva",
    "email": "maria@email.com",
    "phone": "11988887777",
    "status": "Em Atendimento",
    "leadScore": 75,
    "temperature": "Hot",
    "registeredAt": "2024-01-15T10:00:00Z"
  }
]
```

#### POST /clients
Create new client.

#### POST /clients/:id/analyze
Run AI analysis on client to update lead score.

**Response (200):**
```json
{
  "score": 82,
  "summary": "High-intent buyer with budget for premium properties. Responded quickly to messages."
}
```

---

### AI

#### POST /ai/generate-description
Generate property description using AI.

**Request:**
```json
{
  "type": "apartment",
  "location": "Pinheiros, SP",
  "features": {
    "bedrooms": 3,
    "bathrooms": 2,
    "area": 120,
    "extras": ["pool", "gym"]
  }
}
```

**Response (200):**
```json
{
  "description": "Stunning 3-bedroom apartment in the heart of Pinheiros..."
}
```

#### POST /ai/chat
Interactive AI consultant chat.

**Request:**
```json
{
  "messages": [
    { "role": "user", "content": "What properties do you recommend for a family?" }
  ]
}
```

---

### WhatsApp

#### GET /whatsapp/status
Check WhatsApp connection status.

**Response (200):**
```json
{
  "connected": true,
  "instanceName": "oinbox-main"
}
```

#### POST /whatsapp/send
Send WhatsApp message.

**Request:**
```json
{
  "number": "5511999999999",
  "message": "Hello! How can I help you?",
  "mediaUrl": "https://...",  // optional
  "mediaType": "image"        // optional
}
```

#### GET /whatsapp/messages
Retrieve message history.

**Query Params:**
- `limit` (default: 50)
- `remoteJid` (optional, filter by contact)

---

### Billing

#### POST /stripe/create-checkout-session
Create Stripe checkout session.

**Request:**
```json
{
  "planName": "Pro",
  "interval": "monthly"
}
```

**Response (200):**
```json
{
  "url": "https://checkout.stripe.com/c/pay/..."
}
```

---

## Error Responses

All errors follow this format:

```json
{
  "error": "Error message description",
  "code": "ERROR_CODE"  // optional
}
```

**Common HTTP Status Codes:**
- `400` - Bad Request (validation error)
- `401` - Unauthorized (invalid/missing token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `500` - Internal Server Error

---

## Rate Limiting

Rate limits are enforced by Cloudflare:
- 100 requests per minute per IP
- 1000 requests per hour per tenant

---

## Contact

**Developer**: [dev@oconnector.tech](mailto:dev@oconnector.tech)
