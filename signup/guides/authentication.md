# Authentication & Security

How to authenticate with the EPD eMerchant Signup API and manage security.

---

## Authentication Methods

### 1. Bearer Token (OAuth 2.0)

Standard method for all API requests.

**Obtaining a Token:**
- Contact EPD Admin
- Use OAuth 2.0 authorization flow
- Token typically valid for 24 hours

**Usage:**

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://emap.epd.dev/step/1/{uuid}
```

**In Code:**

```javascript
const headers = {
  'Authorization': `Bearer ${token}`,
  'Content-Type': 'application/json'
};

fetch('https://emap.epd.dev/step/1/{uuid}', { headers });
```

### 2. CSRF Token (For POST Requests)

Required for all POST/PUT/DELETE requests.

**Location:** HTML meta tag or response header

```html
<meta name="csrf-token" content="...">
```

**Usage:**

```bash
curl -X POST \
  -H "X-CSRF-TOKEN: {csrf_token}" \
  https://emap.epd.dev/signup/user
```

**Token Expiration:**
- Valid for 24 hours
- Expires on logout
- If expired, returns 419 status

**Refresh Strategy:**

```javascript
if (response.status === 419) {
  // Refresh CSRF token
  const newToken = document.querySelector('meta[name="csrf-token"]')?.content;
  
  // Retry request with new token
  headers['X-CSRF-TOKEN'] = newToken;
  return fetch(url, { headers, ...options });
}
```

---

## Security Best Practices

### 1. Token Storage

**✅ DO:**
- Store in secure HTTP-only cookies
- Store in memory for SPAs
- Refresh before expiration

**❌ DON'T:**
- Store in localStorage (XSS vulnerable)
- Embed in URLs
- Log tokens
- Share in client-side code

### 2. HTTPS Only

All requests must use HTTPS:

```
✅ https://emap.epd.dev/...
❌ http://emap.epd.dev/...
```

### 3. Request Validation

Always include:
- `Authorization` header with valid token
- `X-CSRF-TOKEN` header for POST/PUT/DELETE
- `Content-Type: application/x-www-form-urlencoded` or `multipart/form-data`

### 4. Sensitive Data

Never log or display:
- API tokens
- CSRF tokens
- SSN/SIN numbers
- Bank routing numbers
- Passwords

### 5. Rate Limiting

Respect rate limits:

```
Limit: 100 requests/minute per token
Status: 429 (Too Many Requests)
Retry-After: 60 seconds
```

If you hit limit:
- Wait recommended seconds
- Implement exponential backoff
- Cache responses when possible

---

## Error Responses

### 401 Unauthorized

Invalid or missing token.

```json
{
  "status": 401,
  "message": "Unauthorized",
  "error": "Invalid token"
}
```

**Fix:**
- Verify token is valid
- Check token hasn't expired
- Regenerate if needed

### 419 CSRF Token Expired

CSRF token expired or invalid.

```json
{
  "status": 419,
  "message": "CSRF token mismatch"
}
```

**Fix:**
- Refresh CSRF token from meta tag
- Retry request with new token
- Implement auto-refresh before expiration

### 429 Rate Limited

Too many requests.

```json
{
  "status": 429,
  "message": "Too Many Requests",
  "retry_after": 60
}
```

**Fix:**
- Wait `retry_after` seconds
- Implement backoff strategy
- Reduce request frequency

---

## Implementing Authentication

### JavaScript/Fetch

```javascript
class EPDSignupAPI {
  constructor(token, csrfToken) {
    this.token = token;
    this.csrfToken = csrfToken;
  }

  async request(url, options = {}) {
    const headers = {
      'Authorization': `Bearer ${this.token}`,
      'X-CSRF-TOKEN': this.csrfToken,
      'Content-Type': options.body ? 'application/x-www-form-urlencoded' : 'application/json',
      ...options.headers
    };

    let response = await fetch(url, { ...options, headers });

    // Handle CSRF token expiration
    if (response.status === 419) {
      this.csrfToken = document.querySelector('meta[name="csrf-token"]')?.content;
      headers['X-CSRF-TOKEN'] = this.csrfToken;
      response = await fetch(url, { ...options, headers });
    }

    return response;
  }

  async getStep(stepNumber, uuid) {
    return this.request(`https://emap.epd.dev/step/${stepNumber}/${uuid}`)
      .then(r => r.json());
  }

  async submitStep(stepNumber, data, uuid) {
    return this.request(`https://emap.epd.dev/signup/companies`, {
      method: 'POST',
      body: new URLSearchParams(data)
    }).then(r => r.json());
  }
}

// Usage
const api = new EPDSignupAPI(token, csrfToken);
const step1Data = await api.getStep(1, uuid);
```

### cURL

```bash
# Get step data
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://emap.epd.dev/step/1/{uuid}

# Submit step data
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "X-CSRF-TOKEN: YOUR_CSRF_TOKEN" \
  -d "first_name=John&last_name=Doe&..." \
  https://emap.epd.dev/signup/user
```

---

## Session Management

### Session Lifecycle

1. **Create Session** - Step 1 generates UUID
2. **Persist UUID** - Store in cookie or session
3. **Use UUID** - Reference in all subsequent requests
4. **Cleanup** - Clear session on completion/logout

### Session Storage

```javascript
// Store UUID
sessionStorage.setItem('signup_uuid', uuid);

// Retrieve UUID
const uuid = sessionStorage.getItem('signup_uuid');

// Clear session
sessionStorage.removeItem('signup_uuid');
```

### Session Timeout

- Inactivity: 30 minutes
- Hard limit: 24 hours
- On timeout: Start new signup session

---

## Compliance

### PCI DSS
- No sensitive payment data stored locally
- Bank details collected in Step 5 only
- No data logging for PCI fields

### GDPR
- All user data encrypted in transit
- User data deletable on request
- No tracking without consent

---

## Need Help?

- [Error Codes](./error-handling.md)
- [Getting Started](./getting-started.md)
- [API Reference](../reference/endpoints.md)
