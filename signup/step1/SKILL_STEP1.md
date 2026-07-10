# Step 1: Account Information & Ownership Details

**Step Number:** 1  
**Section:** Account Setup  
**Required Fields:** 5/8  
**Dependent Fields:** 1  
**Auto-Save:** Yes (from lander flows)  
**Previous Step:** N/A (Entry point)  
**Next Step:** Step 2

---

## Overview

Step 1 collects the account owner's personal information and basic company details. This is the entry point to the signup workflow and includes optional annual sales field for specific variants.

---

## Fields

### Group 1: Owner Details (4 fields - Required)

| Field Name | Field ID | Type | Max Length | Validation | Required |
|------------|----------|------|-----------|-----------|----------|
| First Name | `first_name` | Text | 60 | Max 60 chars | ✓ |
| Last Name | `last_name` | Text | 60 | Max 60 chars | ✓ |
| Email | `email` | Email | 255 | RFC 5322 regex | ✓ |
| Phone | `phone` | Phone | 20 | intlTelInput validation | ✓ |

**Phone Behavior:**
- Uses international tel input library
- Country priority: US, Canada
- Returns full number with country code
- Server validates via intlTelInput.isValidNumber()

---

### Group 2: Company Information (4 fields - Required)

| Field Name | Field ID | Type | Options | Max Length | Validation | Required |
|------------|----------|------|---------|-----------|-----------|----------|
| Company Name | `name` | Text | - | 60 | Max 60 chars | ✓ |
| Company Website | `website` | URL | - | 255 | validUrl (http/https required) | ✓ |
| Formation Country | `business_formed_in` | Dropdown | GET /api/countries | - | Country ID (1=US, 2=Canada, etc.) | ✓ |
| Formation State | `business_state` | Dropdown | GET /api/states | - | State code (e.g., "CA", "TX") | ✓ (if Country=US) |

**Country Dropdown Values:**
```
Endpoint: GET https://emap.epd.dev/api/countries
Response: [{id: 1, name: "United States"}, {id: 2, name: "Canada"}, ...]
```

**State Dropdown:**
- Only visible if `business_formed_in` = "1" (US)
- GET https://emap.epd.dev/api/states filters by country context
- Returns state code and full name

---

### Group 3: Additional Information (2 fields - Mixed)

| Field Name | Field ID | Type | Max Length | Validation | Required |
|------------|----------|------|-----------|-----------|----------|
| Annual CC Sales | `annual_cc_sales` | Currency | 12 digits | Max 12 digits, numeric | ✓ |
| Referral/Promo Code | `promo_code` | Text | 100 | Alphanumeric | ✗ |

**Annual Sales Format:**
- Cleave.js formatting: thousands grouping, no decimals
- Example: "1,234,567" → stored as 1234567
- Max value: $999,999,999,999 (12 digits)

---

## Dependent Fields

### Rule 1: Country → State
```json
{
  "trigger": "#business_formed_in",
  "target": "#business_state",
  "targetWrapper": ".usa_state",
  "value": "1",
  "message": "Required because your Formation Country is the United States."
}
```

**Behavior:**
- When `business_formed_in` changes to "1", `business_state` becomes required
- When changed to non-US country, `business_state` is hidden and cleared
- State field only accepts valid US state codes

---

## API Endpoints

### GET - Retrieve Step 1 Data
```
GET /step/1/{uuid}
Headers: 
  - Authorization: Bearer {token}
  - Accept: application/json

Response (200 OK):
{
  "applicationInfo": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "step_count": 1
  },
  "company": {
    "name": "Acme Corp",
    "website": "https://acme.com",
    "country": "1",
    "state": "CA",
    "annual_cc_sales": "500000"
  },
  "data": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@acme.com",
    "phone": "+1-555-123-4567"
  }
}
```

### POST - Submit Step 1 Data
```
POST /signup/user
Headers:
  - Authorization: Bearer {token}
  - Content-Type: application/x-www-form-urlencoded
  - X-CSRF-TOKEN: {csrf_token}

Body:
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@acme.com",
  "phone": "+1-555-123-4567",
  "name": "Acme Corp",
  "website": "https://acme.com",
  "business_formed_in": "1",
  "business_state": "CA",
  "annual_cc_sales": "500000",
  "promo_code": "WELCOME2024",
  "step_count": 1,
  "from_lander": 0,
  "link_visit_id": "lvi_123",
  "partner_id": "partner_456",
  "utm_tracking": "{...}"
}

Response (200 OK):
{
  "message": "Success",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "url": "/step/2/{uuid}",
  "companyExist": false,
  "verificationLink": false
}

Response (400 Bad Request):
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "email": ["The email has already been taken."],
    "phone": ["Please enter a valid phone number."]
  }
}
```

---

## Auto-Save Behavior

### When Enabled
- Triggered when `from_lander=1`
- Only saves if all validated fields are populated
- Saves: first_name, last_name, email, phone, name, website, business_formed_in, business_state
- Endpoint: `/signup/user` with `from_lander=1`
- Debounce: 100ms after validation check

### Auto-Save Fields Validation
```javascript
const fieldsToValidate = [
  '#companyName',      // name field
  '#firstName',        // first_name field
  '#lastName',         // last_name field
  '#website',          // website field
  '#email'             // email field
];
```

---

## Validation Rules

### Field-Level Validation

**First Name / Last Name:**
- Required
- Max 60 characters
- No validation error if empty in secondary case

**Email:**
- Required
- Regex: `/^([a-zA-Z0-9_\.\-])+\@(([a-zA-Z0-9\-])+\.)+([a-zA-Z0-9]{2,4})+$/`
- Uniqueness check on server

**Phone:**
- Required
- Must validate with intlTelInput library
- getNumber() returns E.164 format

**Company Name:**
- Required
- Max 60 characters

**Website:**
- Required
- Regex: `/^(http:\/\/www\.|https:\/\/www\.|http:\/\/|https:\/\/)?.+\..{1,}(:[0-9]{1,5})?(\/.*)?$/`
- Must include protocol

**Country:**
- Required
- Must be valid country ID from reference data

**State:**
- Required if Country = 1 (US)
- Must be valid state code

---

## Error Scenarios

| Scenario | Status | Message |
|----------|--------|---------|
| Email already exists | 400 | The email has already been taken. |
| Invalid phone number | 400 | Please enter a valid phone number. |
| Invalid URL format | 400 | Please enter a valid URL. |
| Country not in reference data | 400 | Invalid country selection. |
| State required but missing (Country=US) | 400 | Please select your formation state. |
| Session expired | 419 | Your session has timed out. Please refresh the page and try again. |

---

## Response Examples

### Success - New Application
```json
{
  "message": "Success",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "url": "/step/2/550e8400-e29b-41d4-a716-446655440000",
  "companyExist": false,
  "verificationLink": false
}
```

### Success - Company Already Exists
```json
{
  "message": "Success",
  "url": "/dashboard/merchant",
  "companyExist": true,
  "verificationLink": false
}
```

### Success - Email Verification Required
```json
{
  "message": "Success",
  "url": "/verify-email",
  "companyExist": false,
  "verificationLink": true
}
```

---

## Integration Checklist

- [ ] Fetch country dropdown via GET /api/countries
- [ ] Implement dependent field rule: Country → State
- [ ] Set up phone field with intlTelInput library
- [ ] Configure Cleave.js for annual_cc_sales (if applicable)
- [ ] Implement form validation with custom rules
- [ ] Handle CSRF token in POST requests
- [ ] Track Step1Submitted event in analytics
- [ ] Implement auto-save for lander flows
- [ ] Test with both US and non-US countries
- [ ] Verify state dropdown only shows for US

---

## Common Issues

| Issue | Solution |
|-------|----------|
| State field not appearing | Check if country value is exactly "1" (string) |
| Phone validation failing | Ensure intlTelInput library is loaded and initialized |
| Auto-save not triggering | Verify from_lander parameter is set to 1 |
| CSRF token errors | Refresh page and fetch new token from meta tag |

---

## Field Count Summary
- **Total Fields:** 8
- **Required Fields:** 5 (always) + 1 conditional (state if US) + 1 annual sales
- **Optional Fields:** 1 (promo code)
- **Dependent Fields:** 1 rule

---

## Next Steps
After successful Step 1 submission, user proceeds to [Step 2: Company Information](../step2/SKILL_STEP2.md)
