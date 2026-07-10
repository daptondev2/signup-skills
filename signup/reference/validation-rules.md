# Validation Rules Reference

Complete field validation rules for all steps.

---

## Step 1: Account Information

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| first_name | Text | ✓ | Max 60 chars |
| last_name | Text | ✓ | Max 60 chars |
| email | Email | ✓ | RFC 5322 format, unique |
| phone | Phone | ✓ | Valid international format (E.164) |
| name | Text | ✓ | Max 60 chars (company name) |
| website | URL | ✓ | Must start with http:// or https://, valid domain |
| business_formed_in | Dropdown | ✓ | Valid country ID from reference |
| business_state | Dropdown | ✓ if US | Valid state code, US only |
| annual_cc_sales | Currency | ✗ | Max 12 digits, numeric only |
| promo_code | Text | ✗ | Max 100 chars, alphanumeric |

---

## Step 2: Company Details

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| name | Text | ✓ | Max 60 chars |
| industry_type | Dropdown | ✓ | Valid ID from reference |
| industry_type_other | Text | ✓ if Other | Max 100 chars |
| country | Dropdown | ✓ | Valid country ID |
| business_location_type | Dropdown | ✓ | Valid ID from reference |
| business_formed | Date | ✓ | YYYY-MM-DD, 100 years ago to today |
| business_structure | Dropdown | ✓ | Valid structure ID |
| federal_tax_id | Text | ✓ | Format depends on country: US/CA/PR="XXX-XX-XXXX", Others="XX-XXXXXXX" |
| business_registration_number | Text | ✓ if non-US | Max 50 chars |
| marketingModel[] | Checkbox | ✓ | At least 1 selected |
| subscription_frequency | Dropdown | ✓ if Recurring | Valid ID |
| subscription_frequency_other | Text | ✓ if Other freq | Max 100 chars |
| street_number | Text | ✓ | Google Places validated |
| street_address | Text | ✓ | Google Places validated |
| city | Text | ✓ | Google Places validated |
| state | Text | ✓ if US | Google Places validated |
| postal_code | Text | ✓ | Google Places validated |
| address_country | Dropdown | ✓ | Valid country ID |
| physical_street_* | Text | ✓ if different | Same as legal address rules |
| customer_service_telephone_number | Phone | ✓ | Valid international format |

---

## Step 3: Product Information

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| card_swiped | Hidden | ✓ | 0-100, sum must = 100 |
| customer_entered | Hidden | ✓ | 0-100, sum must = 100 |
| staff_entered | Hidden | ✓ | 0-100, sum must = 100 |
| describe_highest_transaction | Textarea | ✓ | Min 15 chars, max 1000 |
| describe_services | Dropdown | ✓ | Valid ID from reference |
| highest_transaction_amount | Currency | ✓ | Max 16 chars, numeric only |
| average_transaction_amount | Currency | ✓ | Max 16 chars, numeric only |
| customer_service_time | Dropdown | ✓ | Valid ID from reference |
| refund_policy | Dropdown | ✓ | Valid ID from reference |
| fulfillment_by | Dropdown | ✓ | Valid ID (1, 2, or 3) |
| fullfillment_company | Text | ✓ if Vendor/Other | Max 100 chars |
| shopping_cart | Dropdown | ✓ | Valid ID from reference |
| shopping_cart_other | Text | ✓ if Other | Max 100 chars |
| leave_deposit | Checkbox | ✓ | Must be checked |

---

## Step 4: Beneficial Owners

### Owner 1 (Always Required)

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| first_name | Text | ✓ | Max 60 chars |
| last_name | Text | ✓ | Max 60 chars |
| email | Email | ✓ | RFC 5322, unique, different from Owner 2 |
| phone | Phone | ✓ | Valid international format |
| job_title | Dropdown | ✓ | Valid ID from reference |
| ssn | Text | ✓ | Format XXX-XX-XXXX (9+ digits) |
| ownership_percentage | Number | ✓ | 1-100, if < 51% Owner 2 required |
| dob | Date | ✓ | 18+ years old, max 100 years old |
| bankruptcy_filed | Radio | ✓ | Yes/No |
| bankruptcy_discharged | Radio | ✓ if filed | Yes/No |
| bankruptcy_discharge_date | Date | ✓ if discharged | YYYY-MM-DD |
| street_number | Text | ✓ | Google Places validated |
| street_address | Text | ✓ | Google Places validated |
| city | Text | ✓ | Google Places validated |
| state | Text | ✓ | Google Places validated |
| postal_code | Text | ✓ | Google Places validated |
| country | Dropdown | ✓ | Valid country ID |
| driver_license_state_input | Dropdown | ✓ if US | Valid state code |
| document_number | Text | ✗ | Max 50 chars |
| expiration_date | Date | ✓ if US | YYYY-MM-DD |

### Owner 2 (If Owner 1 < 51%)

Same fields as Owner 1, with:
- ownership_percentage: Max = 100 - Owner 1 %
- email: Must differ from Owner 1

---

## Step 5: Banking Information

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| bank_routing_number | Text | ✓ if US | Exactly 9 digits, valid routing number |
| bank_account_number | Text | ✓ | 8-17 digits |
| bank_name | Text | ✗ | Max 100 chars |
| current_processing | Dropdown | ✓ | Yes/No |
| processor_name | Text | ✓ if Yes | Max 100 chars |
| institution_number | Text | ✓ if Canada | 3 digits |
| customer_pay_currency | Radio | ✓ if Canada | USD or CAD |

---

## Step 6: Final Details

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| howdidyouhear | Dropdown | ✓ | Valid ID from reference |
| hear_about_us_other | Text | ✓ if specific | Max 100 chars |
| multiple_merchant_accounts | Dropdown | ✓ | Yes/No |
| transaction_device | Dropdown | ✗ | Valid ID from reference |
| bad_experience | Dropdown | ✓ | Yes/No |
| bad_experience_happened | Text | ✓ if Yes | Max 100 chars |
| other_interests_capital[] | Checkbox | ✗ | Multiple selection allowed |
| terms_and_conditions_agreed | Checkbox | ✓ | Must be checked |

### Dynamic Fields (if rule engine triggered)

Each field has rules array:
- `required` - Field is required
- `email` - Must be valid email
- `min` - Minimum value
- `max` - Maximum value
- `minlength` - Minimum string length
- `maxlength` - Maximum string length
- `maxPercentage` - B2B + B2C = 100

---

## Universal Validation Rules

### Email Format

```regex
^([a-zA-Z0-9_\.\-])+\@(([a-zA-Z0-9\-])+\.)+([a-zA-Z0-9]{2,4})+$
```

### Phone Format

- Must be valid international number
- Returns E.164 format: +[country code][number]
- Example: +1-555-123-4567

### URL Format

- Must start with http:// or https://
- Must have valid domain
- Can include port and path

### Date Format

- Always YYYY-MM-DD
- No other formats accepted

### Currency Format

- Numeric only, no symbols
- Thousands grouped with comma on display
- Stored as integer without comma

### SSN/SIN Format

- US/Canada/PR: XXX-XX-XXXX (9 digits with hyphens)
- Must pass Luhn algorithm (US only)
- Masked input for security

---

## Cross-Field Validation

### Ownership Percentages

```
Owner 1 + Owner 2 = 100% (if both present)
Owner 1 < 51% requires Owner 2
Owner 2 max = 100 - Owner 1
```

### Dependent Field Requirements

- Country = "1" (US) → State required
- Industry = "50" (Other) → industry_other required
- Marketing includes "2" → Frequency required
- Fulfillment in [2,3] → Vendor name required
- Shopping cart in [8,58] → Other name required
- Current processing = "1" → Processor name required
- Bad experience = "1" → Provider name required
- Ownership < 51% → Owner 2 section required

### Address Fields

- If any address field filled, all 6 required
- State only shown if country = US
- Google Places autocomplete must validate

---

## Tips

- **Server validates everything** - never trust client validation
- **Always validate dates** for age requirements (18+, max 100 years)
- **Phone validation** uses intlTelInput library
- **Email uniqueness** checked against database
- **Address validation** requires Google Places autocomplete
- **Numeric fields** should strip formatting before sending to API

---

## See Also

- [Error Codes](./error-codes.md) - Error messages for each rule
- [Data Types](./data-types.md) - Field data types
- [Dependent Fields](./dependent-fields.md) - Conditional logic
