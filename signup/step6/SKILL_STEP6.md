# Step 6: Final Details & Account Preferences

**Step Number:** 6  
**Section:** Application Completion  
**Required Fields:** 4/12  
**Dependent Fields:** 2  
**Previous Step:** Step 5  
**Next Step:** Dashboard / CCBill

---

## Overview

Step 6 is the final step before application submission. It captures referral source, account setup preferences, and business interests. Upon completion, users are redirected to the dashboard or CCBill payment flow.

---

## Fields

### Group 1: Referral Source (2 fields - Required)

| Field Name | Field ID | Type | Options | Condition | Required |
|------------|----------|------|---------|-----------|----------|
| How Did You Find Us | `howdidyouhear` | Dropdown | From reference | Always | ✓ |
| Additional Information | `hear_about_us_other` | Text | 100 | Specific referral values | ✓ (conditional) |

**Referral Source Dropdown:**
```
Endpoint: GET https://emap.epd.dev/api/referral-sources
Example values:
{id: 1, name: "Google Search"}
{id: 8, name: "Live Event / Trade Show"}
{id: 14, name: "Friend"}
{id: 50, name: "Other"}
```

**Additional Information (Conditional):**
- Shown if howdidyouhear = "50" (Other), "14" (Friend), or "8" (Live Event/Trade Show)
- Max 100 characters
- Dynamic placeholder based on referral type:
  - "Other" → Placeholder: "Other / Friend"
  - "Friend" → Placeholder: "Friend Name"
  - "Live Event" → Placeholder: "Event Name"

---

### Group 2: Account Setup Preferences (4 fields - Required)

| Field Name | Field ID | Type | Options | Condition | Required |
|------------|----------|------|---------|-----------|----------|
| Multiple Merchant Accounts | `multiple_merchant_accounts` | Dropdown | Yes (1), No (0) | Always | ✓ |
| Transaction Device | `transaction_device` | Dropdown | From reference | Always | ✗ |
| Bad Experience | `bad_experience` | Dropdown | Yes (1), No (0) | Always | ✓ |
| Bad Provider Name | `bad_experience_happened` | Text | 100 | If bad_experience = 1 | ✓ (conditional) |

**Multiple Merchant Accounts:**
- Dropdown: Yes/No options
- Indicates if merchant needs multiple MIDs
- Affects onboarding complexity

**Transaction Device:**
- Optional dropdown
- Examples: POS Terminal, Mobile Reader, Online Only
- Helps identify use case

**Bad Experience:**
- Dropdown: Yes/No options
- Indicates churn from previous processor
- Triggers provider name field if Yes

**Bad Provider Name (Conditional):**
- Text input, max 100 characters
- Only required if bad_experience = "1" (Yes)
- Examples: "Square", "Stripe", "PayPal"

---

### Group 3: Business Interests & Terms (2 fields - Required)

| Field Name | Field ID | Type | Options | Validation | Required |
|------------|----------|------|---------|-----------|----------|
| Interest Checkboxes | `other_interests_capital[]` | Checkbox | Capital, Credit Line, etc. | At least 1 checked | ✗ |
| Terms & Conditions | `terms_and_conditions_agreed` | Checkbox | - | Must be checked | ✓ |

**Interest Checkboxes:**
- Multiple selection allowed
- Options loaded from API endpoint
- Not required but captured if selected
- Helps with follow-up offers

**API Endpoint for Interest Options:**
```
GET https://emap.epd.dev/api/interest-details
Authorization: Bearer {token}

Response:
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Capital Infusion",
      "interest_group": {"id": 1, "name": "capital"}
    },
    {
      "id": 2,
      "name": "Equipment Financing",
      "interest_group": {"id": 1, "name": "capital"}
    },
    {
      "id": 3,
      "name": "Marketing Assistance",
      "interest_group": {"id": 2, "name": "marketing"}
    }
  ]
}
```

**Implementation Guide:**
1. Fetch interests from `/api/interest-details` on page load
2. Group by `interest_group.name`
3. Render as checkbox list grouped by category
4. Use interest `id` as checkbox value
5. Cache results locally (24-hour TTL recommended)
6. On form submit, send selected IDs: `other_interests_capital[]=1&other_interests_capital[]=3`

**JavaScript Example:**
```javascript
// Fetch and render interests
fetch('https://emap.epd.dev/api/interest-details', {
  headers: { 'Authorization': `Bearer ${token}` }
})
.then(r => r.json())
.then(response => {
  const grouped = {};
  response.data.forEach(interest => {
    const groupName = interest.interest_group?.name || 'Other';
    if (!grouped[groupName]) grouped[groupName] = [];
    grouped[groupName].push(interest);
  });
  
  // Render grouped checkboxes
  Object.entries(grouped).forEach(([groupName, items]) => {
    console.log(`<h4>${groupName}</h4>`);
    items.forEach(item => {
      console.log(`
        <label>
          <input type="checkbox" name="other_interests_capital[]" value="${item.id}">
          ${item.name}
        </label>
      `);
    });
  });
})
.catch(err => console.error('Failed to load interests:', err));
```

**Terms & Conditions:**
- Single checkbox, required
- Must be checked to submit
- Links to terms document
- Validation: `required: true`

---

## Dependent Fields (2 Rules)

### Rule 1: Referral Source → Additional Info
```json
{
  "trigger": "#howdidyouhear",
  "target": "#hear_about_us_other",
  "targetWrapper": ".hear_about_us_other_container",
  "value": ["50", "14", "8"],
  "message": [
    "Required because you selected Other for how you found us.",
    "Required because you selected Friend for how you found us.",
    "Required because you selected Live Event / Trade Show for how you found us."
  ]
}
```

### Rule 2: Bad Experience → Provider Name
```json
{
  "trigger": "#bad_experience",
  "target": "#bad_experience_happened",
  "targetWrapper": ".bad_experience_happened_container",
  "value": "1",
  "message": "Required because you selected Yes for bad experience."
}
```

---

## API Endpoints

### GET - Retrieve Step 6 Data
```
GET /step/6/{uuid}
Headers: 
  - Authorization: Bearer {token}

Response (200 OK):
{
  "applicationInfo": {...},
  "company": {
    "howdidyouhear": "1",
    "hear_about_us_other": "",
    "multiple_merchant_accounts": "0",
    "transaction_device": "1",
    "bad_experience": "0",
    "bad_experience_happened": ""
  },
  "referralSources": [...],
  "transactionDevices": [...],
  "step_count": 6
}
```

### POST - Submit Step 6 Form
```
POST /signup/companies
Headers:
  - X-CSRF-TOKEN: {csrf_token}
  - Content-Type: application/x-www-form-urlencoded

Body:
{
  "howdidyouhear": "1",
  "hear_about_us_other": "",
  "multiple_merchant_accounts": "0",
  "transaction_device": "1",
  "bad_experience": "0",
  "bad_experience_happened": "",
  "other_interests_capital": ["1", "2"],
  "terms_and_conditions_agreed": "on",
  "step_count": 6,
  "section": "interest_details",
  "uuid": "550e8400...",
  "id": "company_123",
  "user_id": "user_123"
}

Response (200 OK - No Dynamic Fields):
{
  "message": "Application submitted successfully",
  "redirect": "/dashboard/merchant?landing=1"
}

Response (200 OK - CCBill Redirect):
{
  "message": "Redirecting to payment",
  "redirect": "ccbill"
}
```


## Redirect Scenarios

After Step 6 submission, response determines next action:

| Redirect Value | Action | URL |
|---|---|---|
| `/dashboard/merchant?landing=1` | Success - Dashboard | Primary dashboard |
| `/c/ccbill/{uuid}` | CCBill payment flow | Payment processor |

---

## Validation Rules

### Form Validation
- terms_and_conditions_agreed: Required (must be checked)
- howdidyouhear: Required dropdown selection
- multiple_merchant_accounts: Required dropdown selection
- bad_experience: Required dropdown selection

### Conditional Validation
- hear_about_us_other: Required if howdidyouhear = "50", "14", or "8"
- bad_experience_happened: Required if bad_experience = "1"


---

## Error Scenarios

| Scenario | Status | Error |
|----------|--------|-------|
| Terms not checked | 400 | Please agree to terms and conditions |
| Referral source empty | 400 | Please select how you found us |
| Additional info missing (if required) | 400 | Please provide additional information |
| Bad provider name missing | 400 | Please enter the provider name |
| Dynamic fields invalid (B2B+B2C ≠ 100) | 400 | The sum of B2B and B2C percentages should equal to 100 |

---

## Integration Checklist

- [ ] Fetch referral sources via GET /api/referral-sources
- [ ] Implement 2 dependent field rules
- [ ] Set up conditional label updates for referral info
- [ ] Create dynamic fields modal HTML structure
- [ ] Implement getDynamicForm() function
- [ ] Implement field-specific validation (email, min, max, etc.)
- [ ] Create B2B/B2C percentage validator
- [ ] Handle dynamic fields form submission
- [ ] Test all redirect scenarios
- [ ] Implement addRule() function for dynamic field rules
- [ ] Test conditional field visibility/requirement

---

## Key Implementation Details

### Dynamic Form Generation
```javascript
function getDynamicForm(fields) {
  let html = '';
  for (let field of fields) {
    const gridClass = field.type === 'textarea' ? 'col-sm-12' : 'col-sm-6';
    
    if (field.type === 'text') {
      html += `<input type="text" name="textBoxFields[${field.id}]" />`;
    } else if (field.type === 'number') {
      html += `<input type="number" name="textBoxFields[${field.id}]" />`;
    } else if (field.type === 'textarea') {
      html += `<textarea name="textarea[${field.id}]"></textarea>`;
    } else if (field.type === 'checkbox') {
      // Generate checkbox options
    } else if (field.type === 'radio') {
      // Generate radio options
    }
  }
  return html;
}
```

### Dynamic Rule Application
```javascript
function addRule(field) {
  if (!field.rule) return;
  
  let rules = {};
  field.rule.forEach(r => {
    if (r.id === 'required') rules.required = true;
    if (r.id === 'email') rules.validate_email = true;
    if (r.id === 'min') rules.min = parseInt(r.value);
    if (r.id === 'max') rules.max = parseInt(r.value);
  });
  
  // Apply rules to field selector
  $(`input[name="textBoxFields[${field.id}]"]`).rules('add', rules);
}
```

### B2B/B2C Percentage Validation
```javascript
jQuery.validator.addMethod('maxPercentage', function() {
  let b2b = parseInt($('#b2b_sales_percentage').val() || 0);
  let b2c = parseInt($('#b2c_sales_percentage').val() || 0);
  return (b2b + b2c) === 100;
}, "The sum of B2B and B2C percentages should equal to 100.");
```

---

## Field Count Summary
- **Static Fields:** 12
- **Always Required:** 4 (referral source, multiple accounts, bad experience, terms)
- **Conditionally Required:** 2 (referral info, bad provider)
- **Optional:** 3 (transaction device, interests)
- **Hidden:** 1 (section: interest_details)
- **Dependent Fields:** 2 rules
- **Dynamic Fields:** 0-N (rule engine driven)

---

## Completion Flow

1. User submits Step 6 form with all required fields
2. Server validates and checks rule engine
3. **If application approved:**
   - Redirect to `/dashboard/merchant?landing=1`
   - Application complete

4. **If CCBill required:**
   - Return response with `redirect: "ccbill"`
   - Redirect to `/c/ccbill/{uuid}` for payment

---

## Next Steps
Upon successful Step 6 submission:
- **Dashboard:** Redirect to `/dashboard/merchant?landing=1`
- **CCBill:** Redirect to payment flow at `/c/ccbill/{uuid}`
