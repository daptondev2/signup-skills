# Step 5: Banking & Payment Information

**Step Number:** 5  
**Section:** Financial Details  
**Required Fields:** 4/8  
**Dependent Fields:** 2  
**Previous Step:** Step 4  
**Next Step:** Step 6

---

## Overview

Step 5 captures bank account information for processing payments. This step includes optional Plaid integration for automated bank connectivity and manual bank details entry. Currency selection is required for Canadian merchants.

---

## Fields

### Group 1: Bank Account Information (3 fields - Required)

| Field Name | Field ID | Type | Max Length | Format | Validation | Required |
|------------|----------|------|-----------|--------|-----------|----------|
| Bank Routing Number | `bank_routing_number` | Text | 9 | US only | 9 digits, API validation (US) | ✓ (US) |
| Bank Account Number | `bank_account_number` | Text | 17 | 8-17 chars | 8-17 digits | ✓ |
| Bank Name | `bank_name` | Text | 100 | Free text | - | ✗ |

**Routing Number Validation (US Only):**
- Must be exactly 9 digits
- Server-side validation via `/signup/validateRoutingNumber`
- Validation triggered on blur
- Skipped if Plaid connection active (is_plaid=1)
- Returns validation error if invalid

**Account Number:**
- Min 8 characters, Max 17 characters
- Numeric validation for US (digits rule)
- Blur validation: length check

**Bank Name:**
- Optional
- User can override value

---

### Group 2: Institution Information (1 field - Conditional)

| Field Name | Field ID | Type | Max Length | Condition | Required |
|------------|----------|------|-----------|-----------|----------|
| Institution Number | `institution_number` | Text | 3 | Country = Canada (2) | ✓ (CA only) |

**Institution Number:**
- Canada-specific (Transit Number)
- 3 digits
- Only shown if country = "2" (Canada)
- Required for Canadian merchant accounts

---

### Group 3: Processing Information (2 fields - Conditional)

| Field Name | Field ID | Type | Options | Condition | Required |
|------------|----------|------|---------|-----------|----------|
| Currently Processing | `current_processing` | Dropdown | Yes (1), No (0) | - | ✓ |
| Processor Name | `processor_name` | Text | 100 | If current_processing = 1 | ✓ (conditional) |

**Currently Processing:**
- Dropdown with Yes/No options
- Indicates if merchant currently has processor
- Triggers processor_name conditional visibility

**Processor Name:**
- Text input, max 100 characters
- Only required if current_processing = "1" (Yes)
- Examples: "Square", "Stripe", "PayPal"

---

### Group 4: Currency (1 field - Conditional)

| Field Name | Field ID | Type | Options | Condition | Required |
|------------|----------|------|---------|-----------|----------|
| Customer Pay Currency | `customer_pay_currency` | Radio | USD, CAD | Country = Canada | ✓ (CA only) |

**Currency Selection:**
- Radio buttons: USD or CAD
- Only shown if company country = "2" (Canada)
- Default: USD
- Affects settlement currency for transactions

---

### Group 5: Plaid Integration (1 field - Alternative)

| Field Name | Field ID | Type | Description | Required |
|------------|----------|------|-------------|----------|
| Plaid Link Button | N/A | UI Component | Opens Plaid modal for bank connection | ✗ |

**Plaid Behavior:**
- Alternative to manual bank entry
- When connected, sets `is_plaid=1` flag
- Auto-populates: routing_number, account_number, bank_name
- Skips routing number validation (Plaid handles it)
- Returns account details from Plaid API response

**Plaid Event:**
- Custom event: `plaidConnected` with `{accountConnected: true}`
- Sets `isPlaid` JavaScript variable to true
- Skips validation when is_plaid=1

---

## Dependent Fields (2 Rules)

### Rule 1: Currently Processing → Processor Name
```json
{
  "trigger": "#current_processing",
  "target": "#processor_name",
  "targetWrapper": ".processor_name_container",
  "value": "1",
  "message": "Required because you are currently Processing Credit Cards."
}
```

**Behavior:**
- When current_processing = "1", processor_name becomes required
- When changed to "0", processor_name cleared and disabled
- Processor name disabled by default if current_processing = "0"

### Rule 2: Country → Currency
```json
{
  "trigger": "#country",
  "target": "#customer_pay_currency",
  "targetWrapper": ".currency_container",
  "value": "2",
  "message": "Required because your country is Canada."
}
```

**Behavior:**
- Only visible if country = "2" (Canada)
- Required selection if shown
- Defaults to USD if not changed
- Stored in form submission

---

## API Endpoints

### GET - Retrieve Step 5 Data
```
GET /step/5/{uuid}
Headers: 
  - Authorization: Bearer {token}

Response (200 OK):
{
  "applicationInfo": {...},
  "company": {
    "country": "1",
    "bank_routing_number": "123456789",
    "bank_account_number": "98765432",
    "bank_name": "Chase Bank",
    "current_processing": "1",
    "processor_name": "Square",
    "institution_number": "",
    "customer_pay_currency": "USD",
    "is_plaid": 0
  }
}
```

### POST - Submit Step 5 Data
```
POST /signup/companies
Headers:
  - X-CSRF-TOKEN: {csrf_token}
  - Content-Type: application/x-www-form-urlencoded

Body (Manual Entry):
{
  "bank_routing_number": "123456789",
  "bank_account_number": "98765432",
  "bank_name": "Chase Bank",
  "current_processing": "1",
  "processor_name": "Square",
  "customer_pay_currency": "USD",
  "step_count": 5,
  "section": "product_info",
  "is_plaid": 0,
  "bank_account_type": 1,
  "uuid": "550e8400...",
  "id": "company_123",
  "user_id": "user_123"
}

Body (Plaid Entry):
{
  "bank_routing_number": "123456789",
  "bank_account_number": "98765432",
  "bank_name": "Chase Bank",
  "current_processing": "0",
  "processor_name": "",
  "customer_pay_currency": "USD",
  "step_count": 5,
  "section": "product_info",
  "is_plaid": 1,
  "bank_account_type": 1,
  "uuid": "550e8400...",
  "id": "company_123",
  "user_id": "user_123"
}

Response (200 OK):
{
  "message": "Saved successfully",
  "uuid": "550e8400...",
  "redirect": "/step/6/{uuid}"
}
```

---

## Validation Rules

### Routing Number (US Only)
- Exactly 9 digits
- Server validates via API (country = "1" only)
- Validation occurs on blur
- Error: "Please enter 9 digit valid routing number."
- Skipped if is_plaid = 1

### Account Number
- Min 8 characters
- Max 17 characters
- Numeric digits only (US)
- Error: "The Account number must be between 8 to 17 characters."

### Processor Name
- Required if current_processing = "1"
- Max 100 characters
- Error: "Please enter a processor name"

### Routing Number API Validation (US Non-Plaid)
```
POST /signup/validateRoutingNumber
Body: {
  routing_number: "123456789",
  uuid: "550e8400..."
}

Response: {
  valid: true,
  bankName: "Chase Bank"
}
```

---

## Plaid Integration Details

### Plaid Link Configuration
- Environment: Production
- User: Merchant email address
- Public key: From config
- Account types: Checking, Savings
- Returns: Plaid access token + account details

### On Plaid Success
```javascript
$(document).trigger('plaidConnected', {
  accountConnected: true,
  routing: "123456789",
  account: "98765432",
  institutionName: "Chase Bank"
});

isPlaid = true;
// Populate fields with Plaid data
// Skip routing validation
```

---

## Error Scenarios

| Scenario | Status | Error |
|----------|--------|-------|
| Routing number not 9 digits | 400 | Please enter 9 digit valid routing number |
| Routing number invalid (API) | 400 | Invalid routing number |
| Account number < 8 chars | 400 | The Account number must be between 8 to 17 characters |
| Account number > 17 chars | 400 | The Account number must be between 8 to 17 characters |
| Processor name required but empty | 400 | Please enter a processor name |
| Currency missing (Canada) | 400 | Please select a currency |

---

## Integration Checklist

- [ ] Implement Plaid Link button with configuration
- [ ] Set up routing number blur validation (US only)
- [ ] Implement account number length validation
- [ ] Create plaidConnected event handler
- [ ] Implement 2 dependent field rules
- [ ] Set up routing number API validation endpoint
- [ ] Test Plaid connection
- [ ] Test manual entry validation
- [ ] Verify currency field only shows for Canada

---

## Key Implementation Details

### Routing Validation (US Non-Plaid Only)
```javascript
$('#bank_routing_number').blur(function() {
  const routingNumber = $(this).val();
  const country = companyCountry;
  
  if (routingNumber.length !== 9) {
    showError("Please enter 9 digit valid routing number.");
    return;
  }
  
  if (country === "1" && !isPlaid) {
    validateRoutingNumberAPI(routingNumber, uuid);
  }
});
```

### Account Number Validation
```javascript
$('#bank_account_number').blur(function() {
  const length = $(this).val().length;
  
  if (length < 8 || length > 17) {
    showError("The Account number must be between 8 to 17 characters.");
  }
});
```

### Plaid Success Handler
```javascript
$(document).on('plaidConnected', function(e, data) {
  if (data.accountConnected) {
    isPlaid = true;
    $('#bank_routing_number').val(data.routing);
    $('#bank_account_number').val(data.account);
    $('#bank_name').val(data.institutionName);
  }
});
```

---

## Conditional Field Display

### US Merchants
- Routing number: Required
- Institution number: Hidden
- Currency: Not shown (defaults to USD)

### Canadian Merchants
- Routing number: Not shown (N/A)
- Institution number: Required
- Currency: Required (USD or CAD selection)

### Other Countries
- Routing number: Hidden
- Institution number: Hidden
- Currency: Not shown

---

## Field Count Summary
- **Total Fields:** 8
- **Always Required:** 2-4 (account number + current_processing, routing if US)
- **Conditionally Required:** 2 (institution # if Canada, processor name if current=yes, currency if Canada)
- **Optional:** 1 (bank_name)
- **Dependent Fields:** 2 rules
- **Plaid Alternative:** Replaces manual routing/account entry

---

## Next Steps
After successful Step 5 submission, user proceeds to [Step 6: Final Details & Preferences](../step6/SKILL_STEP6.md)
