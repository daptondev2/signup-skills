---
name: signup-step-5
description: Step 5 Banking Information - Bank account details and payment routing information
---

# STEP 5: Banking Information

Fifth step of the 7-step signup form. Collects bank account and routing information for payment processing.

---

## Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| institution_number | text | Conditional | Show if country="CA" (Canada), 3 digits |
| customer_pay_currency | radio | Conditional | Show if country="CA", Options: USD, CAD |
| routing_number | text | Yes | Label varies by country |
| account_number | text | Yes | Alphanumeric |
| account_type | select | No | Static: Checking, Savings |
| currently_processing | select | No | Options: Yes(1), No(0) |
| processor_name | text | Conditional | Show if currently_processing=1, max 300 chars |

---

## Field Labels by Country

**Routing Number**:
- US: "Routing Number" (9 digits)
- Canada: "Transit Number" (5 digits)
- UK: "Sort Code" (6 digits)
- Australia: "BSB" (6 digits)

**Account Number**:
- US: "Account Number" (max 17 chars)
- Canada: "Account Number" (max 12 chars)
- Others: "Account Number" (max 34 chars)

---

## Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

- **institution_number**: Show if country="CA"
- **customer_pay_currency**: Show if country="CA"
- **processor_name**: Show if currently_processing=1

---

## Form Submission & Redirect

**On Success (HTTP 200/201)**:
```javascript
// Response example:
{
  "status": true,
  "message": "Banking information saved successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 5
}

// UUID already stored from Step 1, retrieve and use
const uuid = localStorage.getItem('signup_uuid');

// Redirect to Step 6
window.location.href = `/signup/step/6/${uuid}`;
```

**On Validation Error (HTTP 422)**:
```javascript
// Response example:
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "routing_number": ["Routing number is invalid"],
    "account_number": ["Account number is required"]
  }
}

// Display field errors
// DO NOT change URL - user stays on Step 5
for (const [field, messages] of Object.entries(response.errors)) {
  displayErrorForField(field, messages[0]);
}
```

---

## API Integration

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 5
  section: banking_info
  all Step 5 fields
  uuid: {uuid}
Response: { next_step_url, message }
```

---

## Field Summary

**Total Fields**: 10  
**Required Fields**: 5  
**Conditional Fields**: 3

---

**Production Ready** ✅
