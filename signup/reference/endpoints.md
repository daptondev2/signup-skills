# API Endpoints Reference

Complete reference for all EPD eMerchant Signup API endpoints.

**Base URL:** `https://emap.epd.dev`

---

## Table of Contents

- [Step 1: Account Information](#step-1-account-information)
- [Step 2: Company Details](#step-2-company-details)
- [Step 3: Product Information](#step-3-product-information)
- [Step 4: Beneficial Owners](#step-4-beneficial-owners)
- [Step 5: Banking Information](#step-5-banking-information)
- [Step 6: Final Details](#step-6-final-details)
- [Reference Data](#reference-data)
- [Validation Endpoints](#validation-endpoints)

---

## Step 1: Account Information

Account owner personal information and basic company details.

### GET /step/1/{uuid}

Retrieve Step 1 data (pre-filled from previous submission).

```
GET https://emap.epd.dev/step/1/{uuid}
Headers:
  Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "applicationInfo": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "step_count": 1
  },
  "data": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone": "+1-555-123-4567"
  },
  "company": {
    "name": "Acme Corp",
    "website": "https://acme.com",
    "business_formed_in": "1",
    "business_state": "CA",
    "annual_cc_sales": "500000"
  }
}
```

**Fields:** 8 total
- first_name, last_name, email, phone (4 required)
- name, website, business_formed_in (3 required)
- business_state (1 conditional - if country US)
- annual_cc_sales (1 optional)
- promo_code (1 optional)

**Dependent Fields:** 1 rule
- Country = "1" (US) → state becomes required

[Full Step 1 Documentation →](../step1/SKILL_STEP1.md)

### POST /signup/user

Submit Step 1 data and create signup session.

```
POST https://emap.epd.dev/signup/user
Headers:
  Authorization: Bearer {token}
  X-CSRF-TOKEN: {csrf_token}
  Content-Type: application/x-www-form-urlencoded

Body:
  first_name=John
  last_name=Doe
  email=john@example.com
  phone=%2B15551234567
  name=Acme%20Corp
  website=https%3A%2F%2Facme.com
  business_formed_in=1
  business_state=CA
  annual_cc_sales=500000
  promo_code=WELCOME2024
  step_count=1
```

**Response (200 OK):**
```json
{
  "message": "Success",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "url": "/step/2/{uuid}",
  "companyExist": false,
  "verificationLink": false
}
```

**Response (400 Bad Request):**
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

---

## Step 2: Company Details

Company legal structure, addresses, and business model.

### GET /step/2/{uuid}

Retrieve Step 2 data.

```
GET https://emap.epd.dev/step/2/{uuid}
Headers:
  Authorization: Bearer {token}
```

**Fields:** 15 total (12 required + 3 conditional)

[Full Step 2 Documentation →](../step2/SKILL_STEP2.md)

### POST /signup/companies (Step 2)

```
POST https://emap.epd.dev/signup/companies
Headers:
  Authorization: Bearer {token}
  X-CSRF-TOKEN: {csrf_token}

Body:
  name=Acme%20Corp
  country=1
  business_formed=2020-05-15
  industry_type=1
  business_structure=2
  federal_tax_id=12-34-56789
  customer_service_telephone_number=%2B15559876543
  street_number=123
  street_address=Business%20Ave
  city=San%20Francisco
  state=CA
  postal_code=94105
  address_country=1
  is_physical_address_same_as_legal_address=1
  marketingModel=1
  step_count=2
  section=company_info
  uuid={uuid}
```

**Response (200 OK):**
```json
{
  "message": "Saved successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "redirect": "/step/3/{uuid}"
}
```

---

## Step 3: Product Information

Transaction entry methods, product descriptions, and fulfillment.

### GET /step/3/{uuid}

Retrieve Step 3 data.

```
GET https://emap.epd.dev/step/3/{uuid}
Headers:
  Authorization: Bearer {token}
```

**Fields:** 13 total (9 required + 2 conditional)

[Full Step 3 Documentation →](../step3/SKILL_STEP3.md)

### POST /signup/companies (Step 3)

```
POST https://emap.epd.dev/signup/companies
Headers:
  Authorization: Bearer {token}
  X-CSRF-TOKEN: {csrf_token}

Body:
  card_swiped=40
  customer_entered=35
  staff_entered=25
  describe_highest_transaction=Premium%20consulting
  highest_transaction_amount=50000
  average_transaction_amount=5000
  describe_services=1
  customer_service_time=1
  refund_policy=1
  fulfillment_by=1
  shopping_cart=1
  leave_deposit=1
  step_count=3
  section=product_info
  uuid={uuid}
```

**Response:** Redirect to Step 4

---

## Step 4: Beneficial Owners

Owner personal information, identification, addresses, and ownership percentages.

### GET /step/4/{uuid}

Retrieve Step 4 data.

```
GET https://emap.epd.dev/step/4/{uuid}
Headers:
  Authorization: Bearer {token}
```

**Fields:** 24-48 total (16 Owner 1 always + 24 Owner 2 if < 51%)

[Full Step 4 Documentation →](../step4/SKILL_STEP4.md)

### POST /signup/handleOwnership

Submit Step 4 owner data.

```
POST https://emap.epd.dev/signup/handleOwnership
Headers:
  Authorization: Bearer {token}
  X-CSRF-TOKEN: {csrf_token}

Body:
  owner[1][first_name]=John
  owner[1][last_name]=Doe
  owner[1][email]=john@example.com
  owner[1][phone]=%2B15551234567
  owner[1][job_title]=1
  owner[1][ssn]=123-45-6789
  owner[1][ownership_percentage]=100
  owner[1][dob]=1980-05-15
  owner[1][bankruptcy_filed]=0
  owner[1][street_number]=123
  owner[1][street_address]=Main%20St
  owner[1][city]=San%20Francisco
  owner[1][state]=CA
  owner[1][postal_code]=94105
  owner[1][country]=1
  owner[1][driver_license_state_input]=CA
  owner[1][document_number]=D1234567
  owner[1][expiration_date]=2028-05-15
  step_count=4
  uuid={uuid}
```

**Response:** Redirect to Step 5

---

## Step 5: Banking Information

Bank account details and payment processor information.

### GET /step/5/{uuid}

Retrieve Step 5 data.

```
GET https://emap.epd.dev/step/5/{uuid}
Headers:
  Authorization: Bearer {token}
```

**Fields:** 8 total (4 required + 2 conditional by country)

[Full Step 5 Documentation →](../step5/SKILL_STEP5.md)

### POST /signup/companies (Step 5)

```
POST https://emap.epd.dev/signup/companies
Headers:
  Authorization: Bearer {token}
  X-CSRF-TOKEN: {csrf_token}

Body:
  bank_routing_number=123456789
  bank_account_number=98765432
  bank_name=Chase%20Bank
  current_processing=1
  processor_name=Square
  customer_pay_currency=USD
  step_count=5
  section=product_info
  is_plaid=0
  uuid={uuid}
```

**Response:** Redirect to Step 6

---

## Step 6: Final Details

Referral source, account preferences, and interests.

### GET /step/6/{uuid}

Retrieve Step 6 data.

```
GET https://emap.epd.dev/step/6/{uuid}
Headers:
  Authorization: Bearer {token}
```

**Fields:** 12 static + 0-N dynamic

[Full Step 6 Documentation →](../step6/SKILL_STEP6.md)

### POST /signup/companies (Step 6)

```
POST https://emap.epd.dev/signup/companies
Headers:
  Authorization: Bearer {token}
  X-CSRF-TOKEN: {csrf_token}

Body:
  howdidyouhear=1
  multiple_merchant_accounts=0
  transaction_device=1
  bad_experience=0
  other_interests_capital=1&other_interests_capital=2
  terms_and_conditions_agreed=on
  step_count=6
  section=interest_details
  uuid={uuid}
```

**Response (200 OK - No Dynamic Fields):**
```json
{
  "message": "Application submitted successfully",
  "redirect": "/dashboard/merchant?landing=1"
}
```

**Response (200 OK - With Dynamic Fields):**
```json
{
  "message": "Additional information needed",
  "redirect": "dynamicFields",
  "dynamicFields": [
    {
      "id": "b2b_sales_percentage",
      "name": "B2B Sales Percentage",
      "type": "number",
      "value": "",
      "rule": [...]
    }
  ]
}
```

### POST /signup/dynamic_fields

Submit dynamic fields from Step 6 modal.

```
POST https://emap.epd.dev/signup/dynamic_fields
Headers:
  Authorization: Bearer {token}
  X-CSRF-TOKEN: {csrf_token}

Body:
  textBoxFields[b2b_sales_percentage]=60
  textBoxFields[b2c_sales_percentage]=40
  uuid={uuid}
```

**Response (200 OK):**
```json
{
  "message": "Dynamic fields saved",
  "redirect": "/dashboard/merchant?landing=1"
}
```

---

## Reference Data

Fetch dropdown values for form population.

### GET /api/countries

```
GET https://emap.epd.dev/api/countries
```

**Response:**
```json
[
  {"id": 1, "name": "United States"},
  {"id": 2, "name": "Canada"},
  ...
]
```

### GET /api/states

```
GET https://emap.epd.dev/api/states
```

### GET /api/industry-types

```
GET https://emap.epd.dev/api/industry-types
```

### GET /api/shopping-carts

```
GET https://emap.epd.dev/api/shopping-carts
```

### GET /api/referral-sources

```
GET https://emap.epd.dev/api/referral-sources
```

### GET /api/pricing-templates

```
GET https://emap.epd.dev/api/pricing-templates
```

### GET /api/files-types

```
GET https://emap.epd.dev/api/files-types
```

---

## Validation Endpoints

### POST /signup/validateRoutingNumber

Validate US bank routing number (Step 5).

```
POST https://emap.epd.dev/signup/validateRoutingNumber
Headers:
  Authorization: Bearer {token}

Body:
  routing_number=123456789
  uuid={uuid}
```

**Response (200 OK):**
```json
{
  "valid": true,
  "bankName": "Chase Bank"
}
```

### POST /signup/checkEmail

Check if email address already registered (Step 4).

```
POST https://emap.epd.dev/signup/checkEmail
Headers:
  Authorization: Bearer {token}

Body:
  email=john@example.com
  uuid={uuid}
```

**Response (200 OK):**
```json
{
  "exists": false
}
```

---

## See Also

- [Error Codes](./error-codes.md) - All error messages
- [Validation Rules](./validation-rules.md) - Field requirements
- [Dependent Fields](./dependent-fields.md) - Conditional logic
- [Data Types](./data-types.md) - Field formats
- [Enums](./enums.md) - Dropdown values
