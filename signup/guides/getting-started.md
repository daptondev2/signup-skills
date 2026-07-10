# Getting Started

Learn how to integrate the EPD eMerchant Signup API into your application.

---

## Prerequisites

- HTTP client or SDK capability
- Valid API credentials (token)
- Understanding of REST APIs & JSON

---

## 5-Minute Quickstart

### 1. Get Your Token

Obtain a Bearer token from the EPD Admin panel or use OAuth 2.0.

```bash
Authorization: Bearer YOUR_TOKEN_HERE
```

### 2. Create a Signup Session

Start Step 1 by creating a new application record:

```bash
POST https://emap.epd.dev/signup/user
Headers:
  Authorization: Bearer {token}
  X-CSRF-TOKEN: {csrf_token}
  Content-Type: application/x-www-form-urlencoded

Body:
  first_name=John&last_name=Doe&email=john@example.com&phone=+15551234567&
  name=Acme Corp&website=https://acme.com&business_formed_in=1&business_state=CA&
  annual_cc_sales=500000&promo_code=WELCOME2024&step_count=1
```

### 3. Handle Response

Success response includes UUID (use for all subsequent steps):

```json
{
  "message": "Success",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "url": "/step/2/{uuid}",
  "companyExist": false
}
```

### 4. Continue to Step 2

Use the returned UUID to access next step:

```bash
GET https://emap.epd.dev/step/2/550e8400-e29b-41d4-a716-446655440000
Headers:
  Authorization: Bearer {token}
```

### 5. Repeat for Steps 3-6

Each step follows same pattern:
- GET to retrieve pre-filled data
- POST to submit & advance

---

## Key Concepts

### UUID (Session ID)
Unique identifier for each merchant signup session. Returned in Step 1, used in all subsequent requests.

**Storage:** Keep UUID in session/local storage for persistence across page refreshes.

### CSRF Token
Required for all POST requests. Refresh from meta tag if request returns 419.

```html
<meta name="csrf-token" content="...">
```

### Step Progression
Steps must be completed sequentially:
- Step 1 → Step 2 → Step 3 → Step 4 → Step 5 → Step 6 → Completion

Cannot skip steps or go backward.

### Dependent Fields
Some fields only appear when parent field has specific value.

**Example:** If country = "US", state dropdown appears.

See [Dependent Fields Guide](../reference/dependent-fields.md) for all 16 rules.

---

## Common Integration Patterns

### Pattern 1: Single Page Application (SPA)

```javascript
// Step 1: Create session
const response = await fetch('https://emap.epd.dev/signup/user', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'X-CSRF-TOKEN': csrfToken,
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: formData
});

const data = await response.json();
const uuid = data.uuid;

// Store UUID
localStorage.setItem('signup_uuid', uuid);

// Step 2: Load next step
const step2Data = await fetch(`https://emap.epd.dev/step/2/${uuid}`, {
  headers: { 'Authorization': `Bearer ${token}` }
}).then(r => r.json());

// Display Step 2 form with pre-filled data
```

### Pattern 2: Multi-Step Form (Traditional)

```html
<!-- Step 1 Form -->
<form id="step1" method="POST" action="/signup/user">
  <input name="first_name" type="text" required>
  <!-- ... other fields ... -->
  <input type="hidden" name="step_count" value="1">
  <button type="submit">Next</button>
</form>

<!-- On success, redirect to Step 2 with UUID in URL -->
```

### Pattern 3: Auto-Save (Lander Flow)

```javascript
// Auto-save when user arrives from marketing lander
fetch('https://emap.epd.dev/signup/user', {
  method: 'POST',
  headers: { /* ... */ },
  body: new URLSearchParams({
    first_name,
    last_name,
    email,
    phone,
    from_lander: '1'  // Enable auto-save
  })
});
```

---

## Fetching Reference Data

Before building your form, fetch dropdown values:

```bash
# Countries
GET https://emap.epd.dev/api/countries

# US States
GET https://emap.epd.dev/api/states

# Industry types
GET https://emap.epd.dev/api/industry-types

# Shopping carts/CRM
GET https://emap.epd.dev/api/shopping-carts

# Referral sources
GET https://emap.epd.dev/api/referral-sources
```

Cache these values (they change infrequently).

---

## Field Validation

All fields have client-side AND server-side validation. Always implement both:

### Client-Side (JavaScript)
- Required field checks
- Format validation (email, phone, etc.)
- Length constraints
- Numeric validation

### Server-Side (API)
- All validations re-run
- Additional business logic checks
- Cross-field validation (e.g., ownership percentages)
- Uniqueness checks (email, SSN)

See [Validation Rules](../reference/validation-rules.md) for complete list.

---

## Error Handling

If any step fails validation:

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "email": ["The email has already been taken."],
    "phone": ["Please enter a valid phone number."]
  }
}
```

Display errors to user and allow correction. Resubmit same step data.

For complete error reference, see [Error Codes Guide](./error-handling.md).

---

## Testing Your Integration

### 1. Test with Sample Data

Use test credentials to verify integration:

```
Email: test@example.com
Phone: +1-555-123-4567
Company: Test Corp
Website: https://testcorp.com
```

### 2. Test Dependent Fields

- Select country = "US" → verify state dropdown appears
- Select country = "CA" → verify state hidden
- Change ownership < 51% → verify Owner 2 appears

### 3. Test Error Scenarios

- Submit with missing required fields
- Submit with invalid email format
- Submit with email that already exists
- Submit with invalid phone number

### 4. Test Session Persistence

- Complete Step 1
- Refresh page
- Verify data still there via GET /step/2/{uuid}

---

## Next Steps

1. [Set up authentication](./authentication.md)
2. [Review error handling](./error-handling.md)
3. [Check API reference](../reference/endpoints.md)
4. [See code examples](../examples/)

---

## Need Help?

- [Error Codes](./error-handling.md) - Troubleshoot issues
- [Validation Rules](../reference/validation-rules.md) - Field requirements
- [Dependent Fields](../reference/dependent-fields.md) - Conditional logic
- [OpenAPI Spec](../openapi.yaml) - Machine-readable reference
