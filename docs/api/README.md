# API Documentation

## Authentication

### Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "username": "string",
  "password": "string"
}
```

### Register
```http
POST /api/auth/register
Content-Type: application/json

{
  "email": "string",
  "password": "string",
  "userType": "CARRIER|FACTOR|ADMIN",
  "companyDetails": {
    // Company specific details
  }
}
```

## Carrier Endpoints

### Create Funding Request
```http
POST /api/carrier/funding-requests
Authorization: Bearer <token>
Content-Type: application/json

{
  "amount": "number",
  "purpose": "string",
  "terms": {
    // Funding terms
  }
}
```

### View Proposals
```http
GET /api/carrier/funding-requests/{requestId}/proposals
Authorization: Bearer <token>
```

## Factor Endpoints

### Submit Proposal
```http
POST /api/factor/proposals
Authorization: Bearer <token>
Content-Type: application/json

{
  "fundingRequestId": "string",
  "terms": {
    // Proposal terms
  },
  "documents": [
    // Document references
  ]
}
```

### View Opportunities
```http
GET /api/factor/opportunities
Authorization: Bearer <token>
```

## Document Management

### Upload Document
```http
POST /api/documents
Authorization: Bearer <token>
Content-Type: multipart/form-data

{
  "file": "binary",
  "metadata": {
    // Document metadata
  }
}
```

### Sign Document
```http
POST /api/documents/{documentId}/sign
Authorization: Bearer <token>
Content-Type: application/json

{
  "signature": "string",
  "signatureLocation": {
    // Signature location details
  }
}
```

## Error Responses

### 400 Bad Request
```json
{
  "error": "string",
  "message": "string",
  "details": {
    // Additional error details
  }
}
```

### 401 Unauthorized
```json
{
  "error": "Unauthorized",
  "message": "Invalid or expired token"
}
```

### 403 Forbidden
```json
{
  "error": "Forbidden",
  "message": "Insufficient permissions"
}
```

### 404 Not Found
```json
{
  "error": "Not Found",
  "message": "Resource not found"
}
```

## WebSocket API

### Connection
```javascript
const ws = new WebSocket('wss://api.example.com/ws');
```

### Events
- `NEW_PROPOSAL`
- `DOCUMENT_SIGNED`
- `FUNDING_APPROVED`
- `SYSTEM_NOTIFICATION` 