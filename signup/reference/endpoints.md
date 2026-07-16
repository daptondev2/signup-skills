# API Endpoints Reference

All form submission and data endpoints used by the signup form.

---

## Form Submission Endpoints

### 1. POST https://emap.epd.dev/api/v1/signup
**Purpose**: Initial account signup (Step 1)

**When to Call**: User submits Step 1 form

**Full URL**: `https://emap.epd.dev/api/v1/signup`

**Payload**:
```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "phone": "+1-555-1234",
  "name": "Company Name",
  "website": "https://example.com",
  "country": "1",
  "business_state": "CA",
  "annual_sales": "500000"
}
```

**Success Response** (201):
```json
{
  "success": true,
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 1,
  "message": "Account created successfully"
}
```

**Store UUID**: `localStorage.setItem('signup_uuid', uuid)`

**Navigate To**: `/step/2/{uuid}`

---

### 2. POST https://emap.epd.dev/api/v1/application/step
**Purpose**: Submit data for steps 2-6

**When to Call**: User submits any step (2, 3, 5, 6)

**Full URL**: `https://emap.epd.dev/api/v1/application/step`

**Payload Structure**:
```json
{
  "step_count": 2,
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "section": "company_info",
  "... step specific fields ..."
}
```

**Step 2 Payload** (section: "company_info"):
```json
{
  "step_count": 2,
  "uuid": "...",
  "section": "company_info",
  "legal_name": "ABC Inc.",
  "name": "ABC",
  "industry_type": "retail",
  "customer_service_telephone_number": "+1-555-5678",
  "business_location": "2",
  "business_formed": "2020-01-15",
  "business_organized": "3",
  "federal_tax_id": "12-3456789"
}
```

**Step 3 Payload** (section: "product_info"):
```json
{
  "step_count": 3,
  "uuid": "...",
  "section": "product_info",
  "card_swiped": "25",
  "customer_entered": "50",
  "staff_entered": "25",
  "average_transaction_amount": "200",
  "highest_transaction_amount": "1000",
  "describe_services": "Retail products",
  "describe_highest_transaction": "Premium package"
}
```

**Step 5 Payload** (section: "banking_info"):
```json
{
  "step_count": 5,
  "uuid": "...",
  "section": "banking_info",
  "institution_number": "001",
  "customer_pay_currency": "USD",
  "routing_number": "123456789",
  "account_number": "9876543210"
}
```

**Step 6 Payload** (section: "interest_details"):
```json
{
  "step_count": 6,
  "uuid": "...",
  "section": "interest_details",
  "howdidyouhear": "50",
  "hear_about_us_other": "Friend",
  "multiple_merchant_accounts": "1",
  "transaction_device": "terminal",
  "bad_experience": "0",
  "other_interests_capital": ["capital", "resources"]
}
```

**Success Response** (200):
```json
{
  "success": true,
  "step_count": 2,
  "next_step_url": "/step/3/550e8400-e29b-41d4-a716-446655440000",
  "message": "Step 2 saved successfully"
}
```

**Step 6 Final Response**:
```json
{
  "success": true,
  "message": "Application submitted",
  "redirect": "/dashboard/merchant"
}
```

**Navigate To**: `response.next_step_url` or `response.redirect`

---

### 3. POST https://emap.epd.dev/api/v1/ownership
**Purpose**: Submit owner information (Step 4)

**When to Call**: User submits Step 4 form

**Full URL**: `https://emap.epd.dev/api/v1/ownership`

**Payload**:
```json
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "owner": {
    "1": {
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "phone": "+1-555-1234",
      "ssn": "123-45-6789",
      "job_title": "1",
      "ownership_percentage": 100,
      "dob": "1980-05-15"
    },
    "2": null
  }
}
```

**If Owner 2 Required** (when owner 1 < 50%):
```json
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "owner": {
    "1": { ... owner 1 data ... },
    "2": {
      "first_name": "Jane",
      "last_name": "Smith",
      "email": "jane@example.com",
      "phone": "+1-555-5678",
      "ssn": "987-65-4321",
      "job_title": "2",
      "ownership_percentage": 50,
      "dob": "1985-03-20"
    }
  }
}
```

**Success Response** (200):
```json
{
  "success": true,
  "message": "Ownership information saved"
}
```

**After Success**: Call `POST https://emap.epd.dev/api/v1/application/step` with step_count=4

---

## Data Fetch Endpoints

### GET https://emap.epd.dev/api/partner/countries
**Used For**: Formation Country (Step 1), Company Country (Step 2)

**Response**:
```json
{
  "data": [
    { "slug": "1", "name": "United States" },
    { "slug": "2", "name": "Canada" }
  ]
}
```

---

### GET https://emap.epd.dev/api/partner/states
**Used For**: Formation State (Step 1, conditional on country=US)

**Response**:
```json
{
  "data": [
    { "slug": "CA", "name": "California" },
    { "slug": "NY", "name": "New York" }
  ]
}
```

---

### GET https://emap.epd.dev/api/partner/industry-types
**Used For**: Industry Type (Step 2)

**Response**:
```json
{
  "data": [
    { "slug": "retail", "name": "Retail" },
    { "slug": "ecommerce", "name": "E-commerce" },
    { "slug": "other", "name": "Other" }
  ]
}
```

---

### GET https://emap.epd.dev/api/partner/referral-sources
**Used For**: How did you find us (Step 6)

**Response**:
```json
{
  "data": [
    { "slug": "1", "name": "Google Search" },
    { "slug": "50", "name": "Other" },
    { "slug": "14", "name": "Friend" },
    { "slug": "8", "name": "Live Event / Trade Show" }
  ]
}
```

---

### GET https://emap.epd.dev/api/partner/shopping-carts
**Used For**: Transaction Device (Step 6)

**Response**:
```json
{
  "data": [
    { "slug": "terminal", "name": "Card Terminal" },
    { "slug": "software", "name": "Software / Web" }
  ]
}
```

---

### GET https://emap.epd.dev/api/partner/interest-details
**Used For**: Help Your Company Grow (Step 6)

**Response**:
```json
{
  "data": [
    { "slug": "capital", "name": "Capital" },
    { "slug": "resources", "name": "Resources" },
    { "slug": "marketing", "name": "Marketing" }
  ]
}
```

---

## Error Response Format

### Validation Errors (400/422)
```json
{
  "success": false,
  "message": "Validation failed",
  "errors": {
    "email": "Email is already registered",
    "phone": "Phone number is invalid",
    "name": "Company name is required"
  }
}
```

### Server Error (500)
```json
{
  "success": false,
  "message": "Internal server error"
}
```

### Session Expired (419)
```json
{
  "success": false,
  "message": "Your session has expired. Please refresh and try again."
}
```

---

## Request Headers (Required for All Requests)

```javascript
headers: {
  'Content-Type': 'application/json',
  'X-CSRF-TOKEN': csrfToken,
  'Accept': 'application/json'
}
```

---

## Complete Submission Flow

```
User Opens Step 1
  ↓
User Fills Form
  ↓
User Clicks Next
  ↓
Validate Form (Client-side)
  ↓
POST https://emap.epd.dev/api/v1/signup
  ↓
Store UUID in localStorage
  ↓
Redirect to /step/2/{uuid}
  ↓
Load Step 2 (uuid from URL)
  ↓
Fetch dropdown data (https://emap.epd.dev/api/partner/*)
  ↓
User Fills Form
  ↓
User Clicks Next
  ↓
Validate Form
  ↓
POST https://emap.epd.dev/api/v1/application/step
  ↓
Redirect to /step/3/{uuid}
  ↓
... Repeat Steps 2-5 ...
  ↓
Step 4 Special:
  ├─ POST https://emap.epd.dev/api/v1/ownership (owner details)
  └─ POST https://emap.epd.dev/api/v1/application/step (step marker)
  ↓
... Continue Steps 5-6 ...
  ↓
Step 6 Final:
  ├─ POST https://emap.epd.dev/api/v1/application/step
  └─ Redirect to /dashboard/merchant (success)
```

---

**All endpoints ready for implementation** ✅
