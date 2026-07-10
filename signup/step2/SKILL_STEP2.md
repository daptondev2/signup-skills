# Step 2: Company Information & Business Details

**Step Number:** 2  
**Section:** Company Registration  
**Required Fields:** 14/16  
**Dependent Fields:** 5  
**Previous Step:** Step 1  
**Next Step:** Step 3

---

## Overview

Step 2 captures detailed company information including legal structure, financial details, and complete business address information. This is the most complex step with multiple dependent field rules based on country and business type selections.

---

## Fields

### Group 1: Company Registration (7 fields - Required)

| Field Name | Field ID | Type | Max Length | Validation | Required |
|------------|----------|------|-----------|-----------|----------|
| Legal Company Name | `name` | Text | 60 | Max 60 chars | ✓ |
| Company Name / DBA | `name` | Text (duplicate) | 60 | Max 60 chars | ✓ |
| Industry Type | `industry_type` | Dropdown | - | From reference data | ✓ |
| Industry Type (Other) | `industry_type_other` | Text | 100 | Max 100 chars | ✓ (if industry=other) |
| Company Country | `country` | Dropdown | - | Country ID | ✓ |
| Business Location Type | `business_location_type` | Dropdown | - | From reference data | ✓ |

**Industry Dropdown:**
```
Endpoint: GET https://emap.epd.dev/api/industry-types
Response: [
  {id: 1, name: "Retail"},
  {id: 2, name: "Services"},
  {id: 3, name: "E-Commerce"},
  {id: 50, name: "Other", triggersOther: true}
]
```

---

### Group 2: Business Structure (4 fields - Required)

| Field Name | Field ID | Type | Options | Validation | Required |
|------------|----------|------|---------|-----------|----------|
| Company Formation Date | `business_formed` | Date | - | YYYY-MM-DD, min 100 years ago, max today | ✓ |
| Business Legal Structure | `business_structure` | Dropdown | Sole Prop, LLC, Corp, Partnership | - | ✓ |
| Federal Tax ID | `federal_tax_id` | Text | 11 chars | Formatted based on country | ✓ |
| Business Registration # | `business_registration_number` | Text | 50 | - | ✓ (if non-US) |

**Federal Tax ID Rules:**
- **US/Canada/PR:** Format "3-2-4" (XXX-XX-XXXX) via Cleave.js
- **Other countries:** Format "2-7" (XX-XXXXXXX)
- Server validates via company SSN validation
- Sole proprietorship may use personal SSN

**Date Picker:**
- Min: 100 years ago
- Max: Today
- Format: YYYY-MM-DD
- Uses daterangepicker library

---

### Group 3: Revenue Model (3 fields - Conditional)

| Field Name | Field ID | Type | Options | Validation | Required | Visible When |
|------------|----------|------|---------|-----------|----------|--------------|
| Marketing Model | `marketingModel[]` | Checkbox | One-time, Recurring/Subscription | At least 1 selected | ✓ | Always |
| Subscription Frequency | `subscription_frequency` | Dropdown | Monthly, Quarterly, Annual, Other | - | ✓ (if Recurring selected) | marketingModel includes "2" |
| Subscription Frequency (Other) | `subscription_frequency_other` | Text | 100 | Max 100 chars | ✓ (if Other selected) | subscription_frequency = "3" |

**Marketing Model:**
- Value "1" = One-time purchase
- Value "2" = Recurring/Subscription
- Multiple selections allowed
- Checkboxes, not radio buttons

---

### Group 4: Company Address (6 fields - Required)

| Field Name | Field ID | Type | Max Length | Validation | Required |
|------------|----------|------|-----------|-----------|----------|
| Legal Address - Street | `street_number` | Text | 100 | Google Places validated | ✓ |
| Legal Address - Street Address | `street_address` | Text | 100 | Google Places validated | ✓ |
| Legal Address - City | `city` | Text | 50 | Google Places validated | ✓ |
| Legal Address - State/Province | `state` | Text | 50 | Google Places validated, Conditional | ✓ (if required) |
| Legal Address - Postal Code | `postal_code` | Text | 20 | Google Places validated | ✓ |
| Legal Address - Country | `address_country` | Dropdown | - | From reference data | ✓ |

**Address Auto-Complete:**
- Uses Google Places API
- Endpoint: Autocomplete predictions
- Returns: street_number, street_address, city, state, postal_code, country
- All 6 fields required if any field filled

---

### Group 5: Physical Address (6 fields - Conditional)

| Field Name | Field ID | Type | Max Length | Condition | Required |
|------------|----------|------|-----------|-----------|----------|
| Same as Legal | `is_physical_address_same_as_legal_address` | Radio | - | Always shown | N/A |
| Physical Address - Street | `physical_street_number` | Text | 100 | If different from legal | ✓ (if differs) |
| Physical Address - Street Address | `physical_street_address` | Text | 100 | If different from legal | ✓ (if differs) |
| Physical Address - City | `physical_city` | Text | 50 | If different from legal | ✓ (if differs) |
| Physical Address - State/Province | `physical_state` | Text | 50 | If different from legal | ✓ (if differs) |
| Physical Address - Postal Code | `physical_postal_code` | Text | 20 | If different from legal | ✓ (if differs) |

**Physical Address Visibility:**
- Radio toggle: "Yes" (same as legal) / "No" (different)
- If "Yes": Physical address fields hidden, values copied from legal address
- If "No": Physical address fields required

---

### Group 6: Customer Service (1 field - Required)

| Field Name | Field ID | Type | Max Length | Validation | Required |
|------------|----------|------|-----------|-----------|----------|
| Customer Service Phone | `customer_service_telephone_number` | Phone | 20 | intlTelInput validation | ✓ |

---

## Dependent Fields (5 Rules)

### Rule 1: Industry Type → Other
```json
{
  "trigger": "#industry_type",
  "target": "#industry_type_other",
  "targetWrapper": ".industry_type_other_container",
  "value": "50",
  "message": "Required because your Industry Type is Other."
}
```

### Rule 2: Marketing Model → Subscription Frequency
```json
{
  "trigger": "input[name='marketingModel[]']",
  "target": "#subscription_frequency",
  "targetWrapper": ".subscription_frequency_container",
  "value": "2",
  "message": "Required because you selected Recurring/Subscription."
}
```

**Behavior:** Only show subscription frequency when marketing model includes value "2"

### Rule 3: Subscription Frequency → Other
```json
{
  "trigger": "#subscription_frequency",
  "target": "#subscription_frequency_other",
  "targetWrapper": ".subscription_frequency_other_container",
  "value": "3",
  "message": "Required because you selected Other for subscription frequency."
}
```

### Rule 4: Country → State
```json
{
  "trigger": "#country",
  "target": "#state",
  "targetWrapper": ".state_container",
  "value": "1",
  "message": "Required because your country is United States."
}
```

**Note:** State field may be optional for non-US countries

### Rule 5: Country → Institution Number (Canada Only)
```json
{
  "trigger": "#country",
  "target": "#institution_number",
  "targetWrapper": ".institution_number_container",
  "value": "2",
  "message": "Required because your country is Canada."
}
```

---

## API Endpoints

### GET - Retrieve Step 2 Data
```
GET /step/2/{uuid}
Headers: 
  - Authorization: Bearer {token}

Response (200 OK):
{
  "applicationInfo": {...},
  "company": {
    "name": "Acme Corp",
    "country": "1",
    "state": "CA",
    "industry_type": "1",
    "business_structure": "2",
    "business_formed": "2020-05-15",
    "federal_tax_id": "12-3456789",
    "customer_service_telephone_number": "+1-555-987-6543",
    "street_address": "123 Business Ave",
    "city": "San Francisco",
    "postal_code": "94105",
    "is_physical_address_same_as_legal_address": 1,
    "marketingModel": ["1"],
    "subscription_frequency": null
  },
  "referenceData": {
    "industries": [...],
    "countries": [...],
    "businessStructures": [...]
  }
}
```

### POST - Submit Step 2 Data
```
POST /signup/companies
Headers:
  - X-CSRF-TOKEN: {csrf_token}
  - Content-Type: application/x-www-form-urlencoded

Body:
{
  "name": "Acme Corp",
  "country": "1",
  "state": "CA",
  "industry_type": "1",
  "industry_type_other": "",
  "business_structure": "2",
  "business_formed": "2020-05-15",
  "federal_tax_id": "12-3456789",
  "business_registration_number": "",
  "business_location_type": "1",
  "customer_service_telephone_number": "+1-555-987-6543",
  "annual_cc_sales": "500000",
  "marketingModel": ["1"],
  "subscription_frequency": null,
  "subscription_frequency_other": "",
  "street_number": "123",
  "street_address": "Business Ave",
  "city": "San Francisco",
  "state": "CA",
  "postal_code": "94105",
  "address_country": "1",
  "is_physical_address_same_as_legal_address": "1",
  "step_count": 2,
  "section": "company_info",
  "id": "company_123",
  "uuid": "550e8400..."
}

Response (200 OK):
{
  "message": "Saved successfully",
  "uuid": "550e8400...",
  "redirect": "/step/3/{uuid}"
}
```

---

## Validation Rules

### Form-Level Validation
- At least one marketing model checkbox selected
- If marketing model includes "2", subscription frequency required
- If subscription frequency = "3", other text required
- All 6 address fields required if any filled

### Field-Level Rules
- **Federal Tax ID:** 
  - US format (3-2-4): 11 chars total with hyphens
  - Server validates with companySSN validation
  - Formatted automatically via Cleave.js

- **Business Registration Number:**
  - Only required for non-US countries
  - Max 50 characters

- **State Field:**
  - Only shown if country = "1" (US)
  - Required if shown
  - Disabled for other countries

---

## Error Scenarios

| Scenario | Status | Error |
|----------|--------|-------|
| No marketing model selected | 400 | Please select at least one marketing model |
| Subscription frequency missing | 400 | Please select subscription frequency |
| Federal tax ID invalid format | 400 | Invalid federal tax ID format |
| Address validation failed | 400 | Please provide a valid address |
| State required but empty (US) | 400 | Please select your state |

---

## Integration Checklist

- [ ] Fetch industry types via GET /api/industry-types
- [ ] Implement 5 dependent field rules
- [ ] Set up Google Places autocomplete for addresses
- [ ] Configure Cleave.js for federal_tax_id (2 format types)
- [ ] Set up daterangepicker for business_formed
- [ ] Implement phone validation with intlTelInput
- [ ] Handle marketing model checkbox selection
- [ ] Implement subscription frequency conditional logic
- [ ] Test physical address toggle
- [ ] Verify all 6 dependent field rules trigger correctly

---

## Field Count Summary
- **Total Fields:** 15 (4 optional/conditional)
- **Always Required:** 11
- **Conditionally Required:** 4 (industry_type_other, subscription_frequency[_other], business_registration_number, physical address fields if different)
- **Dependent Fields:** 5 rules
- **Address Groups:** 2 (legal, physical)

---

## Next Steps
After successful Step 2 submission, user proceeds to [Step 3: Product Information](../step3/SKILL_STEP3.md)
