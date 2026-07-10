# Step 4: Beneficial Owner & Personal Information

**Step Number:** 4  
**Section:** Ownership Details  
**Required Fields:** 16+ (Owner 1) + 0-16 (Owner 2)  
**Dependent Fields:** 4  
**Previous Step:** Step 3  
**Next Step:** Step 5

---

## Overview

Step 4 collects beneficial owner information with conditional dual-owner support. Owner 2 section appears when Owner 1 ownership < 51%. Each owner includes personal, financial, address, and driver's license information.

---

## Fields

### Group 1: Owner 1 - Personal Information (4 fields - Required)

| Field Name | Field ID | Type | Max Length | Validation | Required |
|------------|----------|------|-----------|-----------|----------|
| First Name | `owner[1][first_name]` | Text | 60 | Max 60 chars | ✓ |
| Last Name | `owner[1][last_name]` | Text | 60 | Max 60 chars | ✓ |
| Email | `owner[1][email]` | Email | 255 | RFC 5322 + uniqueness check | ✓ |
| Phone | `owner[1][phone]` | Phone | 20 | intlTelInput validation | ✓ |

**Phone Behavior:**
- Uses intlTelInput library
- Returns full E.164 format via getNumber()
- Country persistence in local storage

---

### Group 2: Owner 1 - Identification (3 fields - Required)

| Field Name | Field ID | Type | Max Length | Format | Validation | Required |
|------------|----------|------|-----------|--------|-----------|----------|
| Job Title | `primary_contact_job_title` | Dropdown | - | From reference | - | ✓ |
| SSN / SIN | `owner[1][ssn]` | Text | 11 | 3-2-4 (Cleave) | 9+ digits | ✓ |
| % Company Owned | `owner[1][ownership_percentage]` | Number | 3 | 1-100 | Numeric, max 100 | ✓ |

**SSN/SIN Format (Cleave.js):**
- US/Canada/PR: "3-2-4" format (XXX-XX-XXXX)
- Masked input for security
- Server strips hyphens for storage

**Ownership Percentage Logic:**
- Min: 1, Max: 100
- If Owner 1 < 51%: Owner 2 section appears and becomes required
- If Owner 1 ≥ 51%: Owner 2 section hidden
- Change handler validates and shows/hides Owner 2

---

### Group 3: Owner 1 - Personal Details (2 fields - Required)

| Field Name | Field ID | Type | Min Age | Max Age | Validation | Required |
|------------|----------|------|---------|---------|-----------|----------|
| Date of Birth | `owner[1][dob]` | Date | 18 years | 100 years | YYYY-MM-DD, age validation | ✓ |
| Year Started Business | `owner[1][started_business_year]` | Year | 1900 | Current | Numeric | ✗ |

**DOB Date Picker:**
- Min: 100 years ago
- Max: 18 years ago (age validation)
- Format: YYYY-MM-DD
- Custom validator: validate_date

---

### Group 4: Owner 1 - Financial History (3 fields - Conditional)

| Field Name | Field ID | Type | Options | Triggers | Required |
|------------|----------|------|---------|----------|----------|
| Bankruptcy Filed | `owner[1][bankruptcy_filed]` | Radio | Yes, No | Shows "Discharged" if Yes | ✓ |
| Bankruptcy Discharged | `owner[1][bankruptcy_discharged]` | Radio | Yes, No | Shows "Discharge Date" if Yes | ✓ (if filed) |
| Bankruptcy Discharge Date | `owner[1][bankruptcy_discharge_date]` | Date | - | Shown if discharged=Yes | ✓ (if discharged) |

**Bankruptcy Logic:**
- Initially hidden
- Shows when bankruptcy_filed = "1" (Yes)
- If discharged = "1": discharge date field appears
- Date picker: Min 100 years ago, Max today

---

### Group 5: Owner 1 - Home Address (6 fields - Required)

| Field Name | Field ID | Type | Max Length | Validation | Required |
|------------|----------|------|-----------|-----------|----------|
| Street Number | `owner[1][street_number]` | Text | 100 | Google Places validated | ✓ |
| Street Address | `owner[1][street_address]` | Text | 100 | Google Places validated | ✓ |
| City | `owner[1][city]` | Text | 50 | Google Places validated | ✓ |
| State / Province | `owner[1][state]` | Dropdown | - | Google Places validated | ✓ (if required) |
| Postal Code | `owner[1][postal_code]` | Text | 20 | Google Places validated | ✓ |
| Country | `owner[1][country]` | Dropdown | - | From reference, owner_country1 | ✓ |

**Address Auto-Complete:**
- Google Places API integration
- All 6 fields required if any populated
- Returns structured address data

---

### Group 6: Owner 1 - Driver's License (4 fields - Conditional)

| Field Name | Field ID | Type | Max Length | Format | Condition | Required |
|------------|----------|------|-----------|--------|-----------|----------|
| License State | `owner[1][driver_license_state_input]` | Dropdown | - | State code | Country = US | ✓ (US only) |
| License Number | `owner[1][document_number]` | Text | 50 | Alphanumeric | - | ✗ |
| Expiration Date | `firstExpirationDate` | Date | - | YYYY-MM-DD | Country = US | ✓ (US only) |
| Country | `owner_country1` | Dropdown | - | From reference | - | Triggers license fields |

**License State:**
- Disabled for non-US countries
- Dropdown, state codes only
- Cleared when country changes to non-US

**Expiration Date:**
- Only visible if country = "1" (US)
- Date picker: Min 100 years ago, Max today
- Custom validator: validate_date

---

## Owner 2 Section (Conditional - appears if Owner 1 < 51%)

### Group 7: Owner 2 - Personal Information (4 fields - Required if shown)

| Field Name | Field ID | Type | Max Length | Validation | Required |
|------------|----------|------|-----------|-----------|----------|
| First Name | `owner[2][first_name]` | Text | 60 | Max 60 chars | ✓ (if Owner 2) |
| Last Name | `owner[2][last_name]` | Text | 60 | Max 60 chars | ✓ (if Owner 2) |
| Email | `owner[2][email]` | Email | 255 | Uniqueness check vs Owner 1 | ✓ (if Owner 2) |
| Phone | `owner[2][phone]` | Phone | 20 | intlTelInput validation | ✓ (if Owner 2) |

**Email Uniqueness:**
- AJAX call to `/signup/checkEmail`
- Must differ from Owner 1 email
- Real-time validation on blur

---

### Group 8: Owner 2 - Identification (3 fields - Required if shown)

| Field Name | Field ID | Type | Max Length | Format | Required |
|------------|----------|------|-----------|--------|----------|
| Job Title | `owner[2][job_title]` | Dropdown | - | From reference | ✓ (if Owner 2) |
| SSN / SIN | `owner[2][ssn]` | Text | 11 | 3-2-4 (Cleave) | ✓ (if Owner 2) |
| % Company Owned | `owner[2][ownership_percentage]` | Number | 3 | 1-100 | ✓ (if Owner 2) |

**Ownership Percentage Constraint:**
- Max value: 100 - Owner 1 percentage
- If Owner 1 = 60%, Owner 2 max = 40%
- Dynamic validation based on Owner 1 input

---

### Group 9: Owner 2 - Personal Details (2 fields - Required if shown)

| Field Name | Field ID | Type | Min Age | Max Age | Required |
|------------|----------|------|---------|---------|----------|
| Date of Birth | `owner[2][dob]` | Date | 18 years | 100 years | ✓ (if Owner 2) |
| Bankruptcy Filed | `owner[2][bankruptcy_filed]` | Radio | Yes, No | - | ✓ (if Owner 2) |

---

### Group 10: Owner 2 - Financial History (2 fields - Conditional)

| Field Name | Field ID | Type | Condition | Required |
|------------|----------|------|-----------|----------|
| Bankruptcy Discharged | `owner[2][bankruptcy_discharged]` | Radio | If bankruptcy_filed=Yes | ✓ (conditional) |
| Discharge Date | `owner[2][bankruptcy_discharge_date]` | Date | If discharged=Yes | ✓ (conditional) |

---

### Group 11: Owner 2 - Address & License (Same as Owner 1)
- 6 home address fields
- 4 driver's license fields
- Identical formatting and validation

---

## Dependent Fields (4 Rules)

### Rule 1: Owner 1 Country → Driver License State
```json
{
  "trigger": "#owner_country1",
  "target": "#owner_1_driver_license_state_input",
  "targetWrapper": ".owner1_license_state",
  "value": "1",
  "message": "Required because your country is United States."
}
```

### Rule 2: Owner 1 Country → Expiration Date
```json
{
  "trigger": "#owner_country1",
  "target": "#firstExpirationDate",
  "targetWrapper": ".owner1_expiration_date",
  "value": "1",
  "message": "Required because your country is United States."
}
```

### Rule 3: Owner 2 Country → Driver License State
```json
{
  "trigger": "#owner_country2",
  "target": "#owner_2_driver_license_state_input",
  "targetWrapper": ".owner2_license_state",
  "value": "1",
  "message": "Required because your country is United States."
}
```

### Rule 4: Owner 2 Country → Expiration Date
```json
{
  "trigger": "#owner_country2",
  "target": "#secondExpirationDate",
  "targetWrapper": ".owner2_expiration_date",
  "value": "1",
  "message": "Required because your country is United States."
}
```

---

## API Endpoints

### GET - Retrieve Step 4 Data
```
GET /step/4/{uuid}
Headers: 
  - Authorization: Bearer {token}

Response (200 OK):
{
  "applicationInfo": {...},
  "company": {...},
  "owners": [
    {
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "phone": "+1-555-123-4567",
      "job_title": "1",
      "ssn": "123-45-6789",
      "ownership_percentage": "100",
      "dob": "1980-05-15",
      "bankruptcy_filed": "0",
      "street_number": "123",
      "street_address": "Main St",
      "city": "San Francisco",
      "state": "CA",
      "postal_code": "94105",
      "country": "1",
      "driver_license_state": "CA",
      "document_number": "D1234567",
      "expiration_date": "2028-05-15"
    }
  ]
}
```

### POST - Submit Step 4 Data
```
POST /signup/handleOwnership
Headers:
  - X-CSRF-TOKEN: {csrf_token}
  - Content-Type: application/x-www-form-urlencoded

Body (Owner 1 only - 100%):
{
  "owner": [
    {
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "phone": "+1-555-123-4567",
      "job_title": "1",
      "ssn": "123-45-6789",
      "ownership_percentage": "100",
      "dob": "1980-05-15",
      "bankruptcy_filed": "0",
      "street_number": "123",
      "street_address": "Main St",
      "city": "San Francisco",
      "state": "CA",
      "postal_code": "94105",
      "country": "1",
      "driver_license_state": "CA",
      "document_number": "D1234567",
      "expiration_date": "2028-05-15"
    }
  ],
  "step_count": 4,
  "uuid": "550e8400...",
  "user_id": "user_123"
}

Response (200 OK):
{
  "message": "Saved successfully",
  "uuid": "550e8400...",
  "redirect": "/step/5/{uuid}"
}
```

---

## Validation Rules

### Ownership Percentage
- Owner 1: 1-100
- Owner 2: 1 to (100 - Owner 1 %), hidden if Owner 1 ≥ 51%
- Combined must be exactly 100 if both owners present

### SSN/SIN
- US/Canada/PR: Format 3-2-4 via Cleave
- Custom validator: validate_first_owner_ssn, validate_second_owner_ssn
- 9+ digits required

### Phone
- intlTelInput validation
- Must be valid international number
- Custom validators: ownerOneValidatePhone, ownerTwoValidatePhone

### DOB
- Age must be 18+
- Max 100 years old
- Custom validator: validate_date

### Email
- Unique per owner
- Different from other owner if both present
- AJAX validation to /signup/checkEmail

---

## Error Scenarios

| Scenario | Status | Error |
|----------|--------|-------|
| Owner 1 ownership + Owner 2 ≠ 100% | 400 | Combined ownership must equal 100% |
| Owner 1 < 51% but Owner 2 missing | 400 | Second owner information required |
| Email already exists | 400 | The email has already been taken |
| SSN invalid format | 400 | Invalid SSN format |
| Age < 18 | 400 | Owner must be at least 18 years old |

---

## Integration Checklist

- [ ] Implement ownership percentage change handler
- [ ] Create Owner 2 conditional display logic (< 51%)
- [ ] Set up 4 dependent field rules (license/expiration by country)
- [ ] Implement intlTelInput for both owner phones
- [ ] Configure Cleave.js for SSN/SIN fields (2 instances)
- [ ] Set up date pickers for DOB, license expiration, bankruptcy discharge
- [ ] Implement email uniqueness check via AJAX
- [ ] Create bankruptcy field show/hide logic
- [ ] Implement Google Places autocomplete for both addresses
- [ ] Test ownership percentage constraints and Owner 2 conditional logic

---

## Key Implementation Details

### Owner 2 Visibility Logic
```javascript
if (owner1Percentage < 51) {
  showOwner2Section();
  makeOwner2FieldsRequired();
  owner2Max = 100 - owner1Percentage;
} else {
  hideOwner2Section();
  clearOwner2Fields();
  removeOwner2Requirements();
}
```

### Percentage Constraint
```javascript
$('#owner_2_percentage').rules('add', {
  max: (100 - parseInt($('#owner_1_percentage').val()))
});
```

### Bankruptcy Handler
```javascript
$('#bankruptcy_filed').on('change', function() {
  if (this.value === '1') {
    showBankruptcyDischargedFields();
  } else {
    hideBankruptcyFields();
    clearBankruptcyData();
  }
});
```

---

## Field Count Summary
- **Owner 1 Total Fields:** 24
- **Owner 1 Required:** 16 (always)
- **Owner 1 Conditional:** 4 (license/expiration if US)
- **Owner 2 Total Fields:** 24 (0 if Owner 1 ≥ 51%)
- **Owner 2 Required:** 16 (if shown)
- **Owner 2 Conditional:** 4 (license/expiration if US)
- **Dependent Fields:** 4 rules

---

## Next Steps
After successful Step 4 submission, user proceeds to [Step 5: Banking Information](../step5/SKILL_STEP5.md)
