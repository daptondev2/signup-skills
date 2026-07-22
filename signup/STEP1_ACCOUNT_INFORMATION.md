---
name: signup-step-1
description: Step 1 Account Information - First step of merchant signup with basic details and phone formatting
---

# STEP 1: Account Information

First step of the 7-step signup form. Collects basic merchant contact and company information.

---

## Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| first_name | text | Yes | Max 60 chars |
| last_name | text | Yes | Max 60 chars |
| email | email | Yes | Valid email format |
| phone | tel | Yes | Use intl-tel-input library |
| name | text | Yes | Company name, max 60 chars |
| website | url | Yes | Valid URL or social media profile |
| country | select | Yes | API: `/api/partner/countries` |
| business_state | select | Yes | Show only if country="US", API: `/api/partner/states` |
| annual_sales | text | No | Numeric, USD, variant-specific |
| promo_code | text | No | Optional referral code |

---

## Libraries Required

- `intl-tel-input@22.0.2` - Phone formatting with country selector
- `dependent-fields.js` - Conditional field visibility

---

## API Integration

**Dropdown Data**:
```
GET /api/partner/countries
GET /api/partner/states
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/signup
Headers: X-API-Key: {user.security_key}
Payload: all Step 1 fields + step_count: 1
Response: { uuid, step_count, message }
Store UUID in localStorage for all subsequent steps
```

---

## Validation

- First/Last name: required
- Email: valid format
- Phone: valid number (intl-tel-input validates)
- Website: valid URL
- Country: required
- Business State: required if country="1" (US)

---

## Dependent Fields

- **business_state**: Show if country="US" (hidden by default)

---

## Field Summary

**Total Fields**: 9  
**Required Fields**: 8  
**Conditional Fields**: 1 (business_state)  
**Phone Fields**: 1 (intl-tel-input)

---

**Production Ready** ✅
