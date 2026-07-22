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
| multiple_merchant_accounts | select | No | Static: Yes(1), No(0) |
| **transaction_device** | **select** | **No** | **Hardcoded Static Options** - See details below |
| bad_experience | select | No | Static: Yes(1), No(0) |
| bad_experience_provider | text | Conditional | Show if bad_experience=1, max 300 chars |
| other_interests_capital | checkbox array | No | API: `/api/partner/interest-details`, multiple selection |
| accept_terms | checkbox | Yes | Required to submit, value must be 1 |

---

## Transaction Device Field - Hardcoded Static Options

**Field ID**: `id="transaction_device"`  
**Name**: `name="transaction_device"`  
**Type**: SELECT dropdown  
**Required**: No  

**Static Options** (slug => name key-value pairs):
```javascript
{
  "Easy-Pay-Direct-Gateway": "Easy-Pay-Direct-Gateway",
  "Another-Payment-Gateway": "Another-Payment-Gateway",
  "Need-terminal/POS-system": "Need-terminal/POS-system",
  "Own-terminal/POS-system": "Own-terminal/POS-system",
  "iPhone/Android": "iPhone/Android",
  "Other": "Other"
}
```

**HTML Example**:
```html
<select name="transaction_device" id="transaction_device" class="form-control custom-select">
  <option value="">Please Select</option>
  <option value="Easy-Pay-Direct-Gateway">Easy-Pay-Direct-Gateway</option>
  <option value="Another-Payment-Gateway">Another-Payment-Gateway</option>
  <option value="Need-terminal/POS-system">Need-terminal/POS-system</option>
  <option value="Own-terminal/POS-system">Own-terminal/POS-system</option>
  <option value="iPhone/Android">iPhone/Android</option>
  <option value="Other">Other</option>
</select>
```

**JavaScript Example** (for population):
```javascript
const transactionDevices = {
  "Easy-Pay-Direct-Gateway": "Easy-Pay-Direct-Gateway",
  "Another-Payment-Gateway": "Another-Payment-Gateway",
  "Need-terminal/POS-system": "Need-terminal/POS-system",
  "Own-terminal/POS-system": "Own-terminal/POS-system",
  "iPhone/Android": "iPhone/Android",
  "Other": "Other"
};

// Build dropdown
Object.entries(transactionDevices).forEach(([slug, name]) => {
  $('#transaction_device').append(`<option value="${slug}">${name}</option>`);
});

// Or pre-populate if value is known
const savedValue = "iPhone/Android";
$('#transaction_device').val(savedValue);
```

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
