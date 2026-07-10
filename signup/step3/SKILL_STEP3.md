# Step 3: Product & Transaction Information

**Step Number:** 3  
**Section:** Business Operations  
**Required Fields:** 9/13  
**Dependent Fields:** 2  
**Previous Step:** Step 2  
**Next Step:** Step 4

---

## Overview

Step 3 captures detailed product/service information and transaction processing methods. This step includes a unique dual-range slider for transaction entry method percentages and conditional fulfillment vendor field.

---

## Fields

### Group 1: Transaction Entry Methods (3 fields - Required)

| Field Name | Field ID | Type | Range | Sum Rule | Required |
|------------|----------|------|-------|----------|----------|
| Card Swiped at POS | `card_swiped` | Hidden Input | 0-100 | Σ = 100 | ✓ |
| Customer Enters Online | `customer_entered` | Hidden Input | 0-100 | Σ = 100 | ✓ |
| Staff Manually Enters | `staff_entered` | Hidden Input | 0-100 | Σ = 100 | ✓ |

**Slider Display:**
- Visual: ionRangeSlider (type: "double")
- Shows 3 colored sections with percentages
- Grid enabled
- Step: 20
- Min: 0, Max: 100
- Callback updates hidden input fields

**Slider Configuration:**
```javascript
{
  skin: "round",
  type: "double",
  grid: true,
  min: 0,
  max: 100,
  step: 20,
  hide_from_to: true,
  onStart: updateSliderInputs,
  onChange: updateSliderInputs
}
```

**Visual Updates:**
- Each section shows flex-basis percentage
- Live percentage labels update on slider change
- CSS classes: `.is-first`, `.is-second`, `.is-third`
- Hidden inputs: `.is-first-hidden`, `.is-second-hidden`, `.is-third-hidden`

---

### Group 2: Product Description (2 fields - Required)

| Field Name | Field ID | Type | Max Length | Min Length | Validation | Required |
|------------|----------|------|-----------|-----------|-----------|----------|
| Highest Priced Product/Service | `describe_highest_transaction` | Textarea | 1000 | 15 | Min 15 chars | ✓ |
| Products/Services Description | `describe_services` | Dropdown | - | - | From reference data | ✓ |

---

### Group 3: Transaction Amounts (2 fields - Required)

| Field Name | Field ID | Type | Max Length | Format | Validation | Required |
|------------|----------|------|-----------|--------|-----------|----------|
| Highest Transaction Amount | `highest_transaction_amount` | Currency | 16 | Thousands grouping | Max 16 chars, numeric | ✓ |
| Average Transaction Amount | `average_transaction_amount` | Currency | 16 | Thousands grouping | Max 16 chars, numeric | ✓ |

**Currency Formatting:**
- Cleave.js configuration:
  ```javascript
  {
    numeral: true,
    numeralThousandsGroupStyle: 'thousand',
    numeralDecimalScale: 0,
    numeralPositiveOnly: true
  }
  ```
- Input: "1234567" → Display: "1,234,567"
- Stored: Integer without commas

---

### Group 4: Customer & Fulfillment (4 fields - Required)

| Field Name | Field ID | Type | Options | Validation | Required |
|------------|----------|------|---------|-----------|----------|
| Customer Service Time | `customer_service_time` | Dropdown | From reference | - | ✓ |
| Refund Policy | `refund_policy` | Dropdown | From reference | - | ✓ |
| Fulfillment Method | `fulfillment_by` | Dropdown | In-house, Vendor, Other | - | ✓ |
| Fulfillment Vendor Name | `fullfillment_company` | Text | 100 | Max 100 chars | ✓ (if Vendor/Other) |

**Fulfillment Dropdown Values:**
- "1" = In-house (no vendor required)
- "2" = Vendor (triggers `fullfillment_company`)
- "3" = Other (triggers `fullfillment_company`)

---

### Group 5: Shopping & Checkout (2 fields - Required)

| Field Name | Field ID | Type | Options | Conditional | Required |
|------------|----------|------|---------|-------------|----------|
| Shopping Cart / CRM | `shopping_cart` | Dropdown | From reference | Always | ✓ |
| Shopping Cart (Other) | `shopping_cart_other` | Text | 100 | If cart = "8" or "58" | ✓ (conditional) |

**Shopping Cart Values:**
```
Endpoint: GET https://emap.epd.dev/api/shopping-carts
Example responses:
{id: 1, name: "Shopify"}
{id: 2, name: "WooCommerce"}
{id: 8, name: "Other Sales Platform"}
{id: 58, name: "API / Custom Integration"}
```

**Conditional Labels:**
- If value = "8": Label = "Other Sales Platform", Placeholder = "Instagram DMs, Custom CRM, etc."
- If value = "58": Label = "API / Custom Integration Name", Placeholder = "API / Custom Integration Name"

---

### Group 6: Additional (1 field - Required)

| Field Name | Field ID | Type | Options | Validation | Required |
|------------|----------|------|---------|-----------|----------|
| Leave Deposit Required | `leave_deposit` | Checkbox | - | - | ✓ |

**Behavior:**
- Single checkbox, required
- User must check to confirm requirement
- No value validation, presence only

---

## Dependent Fields (2 Rules)

### Rule 1: Fulfillment Method → Vendor Name
```json
{
  "trigger": "#fulfillment_by_change",
  "target": "#fullfillment_company",
  "targetWrapper": ".fulfillment_by_field",
  "value": ["2", "3"],
  "message": [
    "Required because your Fulfillment Method is Vendor.",
    "Required because your Fulfillment Method is Other."
  ]
}
```

**Behavior:**
- When fulfillment_by = "2" or "3", vendor name becomes required
- When changed to "1", vendor name disabled and cleared
- Error messages vary by selected value

### Rule 2: Shopping Cart → Other
```json
{
  "trigger": "#shopping_cart",
  "target": "#shopping_cart_other",
  "targetWrapper": ".shopping_cart_other_container",
  "value": ["8", "58"],
  "message": [
    "Required because your Shopping Cart / CRM is Other.",
    "Required because your Shopping Cart / CRM is API / Custom Integration."
  ]
}
```

**Behavior:**
- When shopping_cart = "8" or "58", other field becomes required
- Label and placeholder update based on specific value
- Field cleared when value changed away

---

## API Endpoints

### GET - Retrieve Step 3 Data
```
GET /step/3/{uuid}
Headers: 
  - Authorization: Bearer {token}

Response (200 OK):
{
  "applicationInfo": {...},
  "company": {
    "card_swiped": "40",
    "customer_entered": "35",
    "staff_entered": "25",
    "describe_highest_transaction": "Premium consulting packages",
    "highest_transaction_amount": "50000",
    "average_transaction_amount": "5000",
    "describe_services": "1",
    "customer_service_time": "1",
    "refund_policy": "1",
    "fulfillment_by": "1",
    "fullfillment_company": "",
    "shopping_cart": "1",
    "leave_deposit": "1"
  }
}
```

### POST - Submit Step 3 Data
```
POST /signup/companies
Headers:
  - X-CSRF-TOKEN: {csrf_token}
  - Content-Type: application/x-www-form-urlencoded

Body:
{
  "card_swiped": "40",
  "customer_entered": "35",
  "staff_entered": "25",
  "describe_highest_transaction": "Premium consulting packages",
  "highest_transaction_amount": "50000",
  "average_transaction_amount": "5000",
  "describe_services": "1",
  "customer_service_time": "1",
  "refund_policy": "1",
  "fulfillment_by": "1",
  "fullfillment_company": "",
  "shopping_cart": "1",
  "shopping_cart_other": "",
  "leave_deposit": "1",
  "step_count": 3,
  "section": "product_info",
  "uuid": "550e8400...",
  "id": "company_123"
}

Response (200 OK):
{
  "message": "Saved successfully",
  "uuid": "550e8400...",
  "redirect": "/step/4/{uuid}"
}
```

---

## Validation Rules

### Slider Validation
- Sum of card_swiped + customer_entered + staff_entered must = 100
- Each value must be 0-100
- Validation triggered on slider change

### Amount Fields
- Max 16 characters (including formatting)
- Numeric values only (Cleave.js enforces)
- No decimal places allowed

### Description Field
- Minimum 15 characters
- Required validation on first error

### Fulfillment Logic
- If fulfillment_by = "1": vendor name ignored
- If fulfillment_by = "2" or "3": vendor name required (max 100 chars)

### Shopping Cart Logic
- If shopping_cart = "8" or "58": other field required
- Otherwise: other field ignored

---

## Error Scenarios

| Scenario | Status | Error |
|----------|--------|-------|
| Slider sum ≠ 100 | 400 | Transaction percentages must sum to 100 |
| Description < 15 chars | 400 | Please provide at least 15 characters |
| Vendor name missing | 400 | Please enter your fulfillment vendor name |
| Amount > 16 chars | 400 | Please enter no more than 16 characters |

---

## Integration Checklist

- [ ] Implement ionRangeSlider with double type
- [ ] Create updateSliderInputs() callback function
- [ ] Fetch shopping cart options via GET /api/shopping-carts
- [ ] Implement Cleave.js for transaction amount formatting
- [ ] Set up 2 dependent field rules
- [ ] Handle fulfillment vendor conditional visibility
- [ ] Handle shopping cart other field conditional logic
- [ ] Implement slider validation (sum = 100)
- [ ] Verify amount field formatting and validation

---

## Key Implementation Details

### Slider Implementation
```javascript
$(".js-range-slider").ionRangeSlider({
  skin: "round",
  type: "double",
  grid: true,
  min: 0,
  max: 100,
  step: 20,
  from: parseInt($("#card_swiped").val() || 0),
  to: parseInt(($("#customer_entered").val() || 0)) + 
      parseInt(($("#card_swiped").val() || 0)),
  hide_from_to: true,
  onStart: updateSliderInputs,
  onChange: updateSliderInputs
});
```

### Update Function
```javascript
function updateSliderInputs(data) {
  const from = data.from;
  const to = data.to;
  
  const firstValue = from;
  const secondValue = to - from;
  const thirdValue = 100 - to;
  
  // Update visual and hidden fields
  // Trigger validation
}
```

---

## Field Count Summary
- **Total Fields:** 13
- **Always Required:** 9 (slider 3 fields + descriptions 2 + amounts 2 + service dropdowns 2)
- **Conditionally Required:** 2 (fulfillment_company, shopping_cart_other)
- **Optional:** 1 (leave_deposit - actually required but checkbox)
- **Dependent Fields:** 2 rules
- **Hidden Fields:** 3 (slider values)

---

## Next Steps
After successful Step 3 submission, user proceeds to [Step 4: Owner Information](../step4/SKILL_STEP4.md)
