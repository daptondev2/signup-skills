# Error Handling

How to handle errors and implement recovery flows.

---

## HTTP Status Codes

### 2xx Success

| Code | Meaning |
|------|---------|
| 200 | Request succeeded |
| 201 | Resource created |

### 4xx Client Errors

| Code | Meaning | Action |
|------|---------|--------|
| 400 | Bad Request (validation failed) | Fix data & retry |
| 401 | Unauthorized (token invalid) | Refresh token & retry |
| 419 | CSRF token expired | Refresh CSRF & retry |
| 422 | Unprocessable Entity (logic error) | Review error details |
| 429 | Rate Limited | Wait & retry |

### 5xx Server Errors

| Code | Meaning | Action |
|------|---------|--------|
| 500 | Internal Server Error | Retry with backoff |
| 502 | Bad Gateway | Retry with backoff |
| 503 | Service Unavailable | Retry with backoff |

---

## Error Response Format

### Validation Error (400)

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "email": ["The email has already been taken."],
    "phone": ["Please enter a valid phone number."],
    "business_formed_in": ["Please select your formation country."]
  }
}
```

**Recovery:**
1. Display error messages next to fields
2. Allow user to correct data
3. Resubmit same step

### Authentication Error (401)

```json
{
  "status": 401,
  "message": "Unauthorized",
  "error": "Invalid token"
}
```

**Recovery:**
1. Refresh token from OAuth provider
2. Retry request with new token
3. If still fails, redirect to login

### CSRF Error (419)

```json
{
  "status": 419,
  "message": "CSRF token mismatch"
}
```

**Recovery:**
1. Fetch fresh CSRF token from meta tag
2. Retry request with new token
3. If persists, clear cookies and start fresh

### Rate Limited (429)

```json
{
  "status": 429,
  "message": "Too Many Requests",
  "retry_after": 60
}
```

**Recovery:**
1. Wait `retry_after` seconds
2. Implement exponential backoff
3. Reduce request frequency

### Server Error (5xx)

```json
{
  "status": 500,
  "message": "Internal Server Error",
  "request_id": "req_12345"
}
```

**Recovery:**
1. Log error with request_id
2. Wait 1-2 seconds
3. Retry with exponential backoff
4. After 3 failures, show user message
5. Provide support contact with request_id

---

## Common Errors by Step

### Step 1 Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Email already taken | Duplicate signup | Use different email |
| Invalid phone number | Wrong format | Use E.164 format |
| Invalid URL | Missing protocol | Include http:// or https:// |
| State required | Country is US but no state | Select US state |

### Step 2 Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Address validation failed | Invalid address | Use Google Places autocomplete |
| Industry other required | Selected "Other" industry | Enter custom industry |
| Tax ID invalid | Wrong format | Use correct country format |
| Subscription freq required | Selected "Recurring" | Choose frequency |

### Step 4 Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Email already taken | Duplicate owner email | Use different email |
| SSN invalid format | Wrong format | Use XXX-XX-XXXX format |
| Ownership < 51% no Owner 2 | Missing second owner | Add Owner 2 details |
| Ownership percentages ≠ 100% | Calculation error | Adjust percentages |

### Step 5 Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Routing number invalid | Wrong length/value | Check 9-digit US routing |
| Account number length | Wrong length | Use 8-17 characters |
| Processor name required | Current processing = Yes | Enter processor name |

---

## Error Recovery Flow

```
User Submits Form
        ↓
Validation Success?
   ├─ No → Display Errors
   │        ↓
   │    User Corrects
   │        ↓
   │    Resubmit
   │
   └─ Yes → API Request
               ↓
            Status 200?
            ├─ No (4xx/5xx)
            │   ↓
            │ Specific Error?
            │ ├─ 400 (validation) → Display field errors
            │ ├─ 401 (auth) → Refresh token & retry
            │ ├─ 419 (CSRF) → Refresh CSRF & retry
            │ ├─ 429 (rate) → Wait & retry
            │ └─ 5xx (server) → Backoff & retry
            │
            └─ Yes → Advance to Next Step
```

---

## Implementation Examples

### JavaScript Error Handler

```javascript
async function submitStep(stepData, stepNumber) {
  try {
    const response = await fetch(
      `https://emap.epd.dev/signup/companies`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${token}`,
          'X-CSRF-TOKEN': csrfToken
        },
        body: new URLSearchParams(stepData)
      }
    );

    const data = await response.json();

    if (!response.ok) {
      switch (response.status) {
        case 400:
          // Validation error
          displayFieldErrors(data.errors);
          return { success: false, errors: data.errors };

        case 401:
          // Token expired
          token = await refreshToken();
          return submitStep(stepData, stepNumber); // Retry

        case 419:
          // CSRF expired
          csrfToken = document.querySelector('meta[name="csrf-token"]').content;
          return submitStep(stepData, stepNumber); // Retry

        case 429:
          // Rate limited
          await wait(data.retry_after * 1000);
          return submitStep(stepData, stepNumber); // Retry

        case 500:
          // Server error
          showError(`Server error. Reference: ${data.request_id}`);
          return { success: false };
      }
    }

    return { success: true, data };

  } catch (error) {
    console.error('Network error:', error);
    showError('Network error. Please check your connection.');
    return { success: false };
  }
}

function displayFieldErrors(errors) {
  Object.entries(errors).forEach(([field, messages]) => {
    const element = document.querySelector(`[name="${field}"]`);
    if (element) {
      element.classList.add('error');
      element.parentElement.querySelector('.error-message').textContent = 
        messages[0];
    }
  });
}

function wait(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

### Exponential Backoff

```javascript
async function requestWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s
      console.log(`Retry in ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
await requestWithBackoff(
  () => submitStep(data, stepNumber)
);
```

---

## Complete Error Reference

See [Error Codes Reference](../reference/error-codes.md) for:
- All error messages
- Status codes
- Root causes
- Resolution steps

---

## Support

If error persists after implementing recovery:
1. Note error code and request ID
2. Check [Error Codes](../reference/error-codes.md)
3. Review [Validation Rules](../reference/validation-rules.md)
4. Contact support with request ID
