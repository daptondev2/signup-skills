---
name: signup-step-6
description: Step 6 Final Details - Referral sources, interests, and terms acceptance
---

# STEP 6: Final Details

Sixth step of the 7-step signup form. Collects referral information, interest details, and terms acceptance.

---

## Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| howdidyouhear | select | No | API: `/api/partner/referral-sources` |
| howdidyouhear_other | text | Conditional | Show if howdidyouhear="Other" or "Friend", max 300 chars |
| multiple_merchant_accounts | select | No | Options: Yes(1), No(0) |
| transaction_device | select | No | Static options or from API |
| bad_experience | select | No | Options: Yes(1), No(0) |
| bad_experience_provider | text | Conditional | Show if bad_experience=1, max 300 chars |
| other_interests_capital | checkbox array | No | API: `/api/partner/interest-details`, multiple selection |
| accept_terms | checkbox | Yes | Required to submit, value must be 1 |

---

## Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

- **howdidyouhear_other**: Show if howdidyouhear="Other" or "Friend"
- **bad_experience_provider**: Show if bad_experience=1

---

## Form Submission & Final Redirect

**On Success (HTTP 200/201)** - FINAL STEP:
```javascript
// Response example:
{
  "status": true,
  "message": "Signup completed successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 6,
  "redirect": "/dashboard/merchant?landing=1"
}

// Clear stored UUID after completion
localStorage.removeItem('signup_uuid');

// Redirect to Merchant Dashboard (FINAL STEP)
window.location.href = '/dashboard/merchant?landing=1';
```

**On Validation Error (HTTP 422)**:
```javascript
// Response example:
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "accept_terms": ["You must accept the terms and conditions"]
  }
}

// Display field errors
// DO NOT change URL - user stays on Step 6
for (const [field, messages] of Object.entries(response.errors)) {
  displayErrorForField(field, messages[0]);
}
```

---

## API Integration

**Dropdown Data**:
```
GET /api/partner/referral-sources
GET /api/partner/interest-details
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 6
  section: interest_details
  all Step 6 fields
  uuid: {uuid}
Response: { redirect: "/dashboard/merchant" }
```

---

## Field Summary

**Total Fields**: 11  
**Required Fields**: 2  
**Conditional Fields**: 2

---

**Production Ready** ✅
