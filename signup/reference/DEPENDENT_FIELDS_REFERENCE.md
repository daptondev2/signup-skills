# Dependent Fields Reference Guide

**Version:** 1.0.0  
**Total Dependent Field Rules:** 16  
**Implementation Library:** DependentFields.js  
**Activation:** On DOM ready

---

## Overview

This reference document consolidates all conditional field dependencies across Steps 1-6. Dependent fields create a dynamic form experience where field visibility, requirement status, and enabled state change based on parent field values.

**Key Concept:** When a "trigger" field changes value, "target" field(s) become required/visible/disabled based on matching the specified value(s).

---

## Implementation Pattern

All dependent fields use the DependentFields.js library with consistent configuration:

```javascript
new DependentFields({
  rules: [
    {
      trigger: '#parentFieldSelector',
      target: '#childFieldSelector',
      targetWrapper: '.containerClass',
      value: 'triggerValue' or ['value1', 'value2'],
      message: 'Error message' or ['msg1', 'msg2']
    }
  ]
});
```

---

## Complete Dependency Map

### STEP 1: Account Information (1 Rule)

#### Rule 1.1: Country → State
```json
{
  "step": 1,
  "rule_id": "S1_R1",
  "rule_name": "Formation Country triggers State requirement",
  "trigger": "#business_formed_in",
  "triggerLabel": "Formation Country",
  "target": "#business_state",
  "targetLabel": "Formation State",
  "targetWrapper": ".usa_state",
  "value": "1",
  "valueName": "United States",
  "effect": "Makes state field visible and required",
  "clearOn": "When country changes away from US",
  "message": "Required because your Formation Country is the United States."
}
```

**Behavior Flow:**
```
Country = 1 (US) → State visible + required
Country ≠ 1 → State hidden + cleared + not required
```

**Auto-Save Impact:** State field included in auto-save if US selected

---

### STEP 2: Company Information (5 Rules)

#### Rule 2.1: Industry Type → Other Text
```json
{
  "step": 2,
  "rule_id": "S2_R1",
  "rule_name": "Industry Type Other triggers text input",
  "trigger": "#industry_type",
  "triggerLabel": "Industry Type",
  "target": "#industry_type_other",
  "targetLabel": "Industry Type (Other)",
  "targetWrapper": ".industry_type_other_container",
  "value": "50",
  "valueName": "Other",
  "effect": "Makes text input visible and required",
  "maxLength": 100,
  "message": "Required because your Industry Type is Other."
}
```

#### Rule 2.2: Marketing Model Recurring → Subscription Frequency
```json
{
  "step": 2,
  "rule_id": "S2_R2",
  "rule_name": "Recurring marketing model triggers subscription frequency",
  "trigger": "input[name='marketingModel[]']",
  "triggerLabel": "Marketing Model (checkbox)",
  "target": "#subscription_frequency",
  "targetLabel": "Subscription Frequency",
  "targetWrapper": ".subscription_frequency_container",
  "value": "2",
  "valueName": "Recurring/Subscription",
  "effect": "Makes dropdown visible and required if 'Recurring' checkbox is checked",
  "triggerType": "checkbox",
  "message": "Required because you selected Recurring/Subscription."
}
```

**Unique Aspect:** Trigger is checkbox, not single select. Rule activates when value "2" is checked.

#### Rule 2.3: Subscription Frequency Other → Other Text
```json
{
  "step": 2,
  "rule_id": "S2_R3",
  "rule_name": "Subscription Frequency Other triggers text input",
  "trigger": "#subscription_frequency",
  "triggerLabel": "Subscription Frequency",
  "target": "#subscription_frequency_other",
  "targetLabel": "Subscription Frequency (Other)",
  "targetWrapper": ".subscription_frequency_other_container",
  "value": "3",
  "valueName": "Other",
  "effect": "Makes text input visible and required",
  "maxLength": 100,
  "message": "Required because you selected Other for subscription frequency."
}
```

#### Rule 2.4: Country → State (Company Address)
```json
{
  "step": 2,
  "rule_id": "S2_R4",
  "rule_name": "Country triggers State requirement for address",
  "trigger": "#country",
  "triggerLabel": "Country",
  "target": "#state",
  "targetLabel": "State/Province",
  "targetWrapper": ".state_container",
  "value": "1",
  "valueName": "United States",
  "effect": "Makes state field required if US selected",
  "message": "Required because your country is United States."
}
```

#### Rule 2.5: Country Canada → Institution Number
```json
{
  "step": 2,
  "rule_id": "S2_R5",
  "rule_name": "Canada country triggers institution number",
  "trigger": "#country",
  "triggerLabel": "Country",
  "target": "#institution_number",
  "targetLabel": "Institution Number",
  "targetWrapper": ".institution_number_container",
  "value": "2",
  "valueName": "Canada",
  "effect": "Makes institution number field visible and required for Canadian merchants",
  "maxLength": "3",
  "message": "Required because your country is Canada."
}
```

---

### STEP 3: Product & Transaction Information (2 Rules)

#### Rule 3.1: Fulfillment Method → Vendor Name
```json
{
  "step": 3,
  "rule_id": "S3_R1",
  "rule_name": "Fulfillment method triggers vendor name field",
  "trigger": "#fulfillment_by_change",
  "triggerLabel": "Fulfillment Method",
  "target": "#fullfillment_company",
  "targetLabel": "Fulfillment Vendor Name",
  "targetWrapper": ".fulfillment_by_field",
  "value": ["2", "3"],
  "valueNames": ["Vendor", "Other"],
  "effect": "Makes vendor name visible and required",
  "maxLength": 100,
  "messages": [
    "Required because your Fulfillment Method is Vendor.",
    "Required because your Fulfillment Method is Others."
  ],
  "clearOn": "When fulfillment method changes to In-house"
}
```

**Multi-Value Behavior:** Rule triggers when value is EITHER "2" OR "3"

#### Rule 3.2: Shopping Cart Type → Other Text
```json
{
  "step": 3,
  "rule_id": "S3_R2",
  "rule_name": "Shopping cart type triggers other text field",
  "trigger": "#shopping_cart",
  "triggerLabel": "Shopping Cart / CRM",
  "target": "#shopping_cart_other",
  "targetLabel": "Shopping Cart (Other)",
  "targetWrapper": ".shopping_cart_other_container",
  "value": ["8", "58"],
  "valueNames": ["Other Sales Platform", "API / Custom Integration"],
  "effect": "Makes text input visible and required",
  "maxLength": 100,
  "messages": [
    "Required because your Shopping Cart / CRM is Other.",
    "Required because your Shopping Cart / CRM is API / Custom Integration."
  ],
  "labelUpdate": {
    "8": "Other Sales Platform",
    "58": "API / Custom Integration Name"
  },
  "placeholderUpdate": {
    "8": "Instagram DMs, Custom CRM, etc.",
    "58": "API / Custom Integration Name"
  }
}
```

**Special Feature:** Label and placeholder update based on specific selected value

---

### STEP 4: Beneficial Owner & Personal Information (4 Rules)

#### Rule 4.1: Owner 1 Country → Driver License State
```json
{
  "step": 4,
  "rule_id": "S4_R1",
  "rule_name": "Owner 1 country triggers driver license state field",
  "trigger": "#owner_country1",
  "triggerLabel": "Owner 1 Country",
  "target": "#owner_1_driver_license_state_input",
  "targetLabel": "Owner 1 Driver License State",
  "targetWrapper": ".owner1_license_state",
  "value": "1",
  "valueName": "United States",
  "effect": "Makes license state dropdown visible and required (US only)",
  "message": "Required because your country is United States."
}
```

#### Rule 4.2: Owner 1 Country → License Expiration Date
```json
{
  "step": 4,
  "rule_id": "S4_R2",
  "rule_name": "Owner 1 country triggers expiration date field",
  "trigger": "#owner_country1",
  "triggerLabel": "Owner 1 Country",
  "target": "#firstExpirationDate",
  "targetLabel": "Owner 1 License Expiration Date",
  "targetWrapper": ".owner1_expiration_date",
  "value": "1",
  "valueName": "United States",
  "effect": "Makes date picker visible and required (US only)",
  "message": "Required because your country is United States."
}
```

#### Rule 4.3: Owner 2 Country → Driver License State
```json
{
  "step": 4,
  "rule_id": "S4_R3",
  "rule_name": "Owner 2 country triggers driver license state field",
  "trigger": "#owner_country2",
  "triggerLabel": "Owner 2 Country",
  "target": "#owner_2_driver_license_state_input",
  "targetLabel": "Owner 2 Driver License State",
  "targetWrapper": ".owner2_license_state",
  "value": "1",
  "valueName": "United States",
  "effect": "Makes license state dropdown visible and required (US only)",
  "message": "Required because your country is United States."
}
```

#### Rule 4.4: Owner 2 Country → License Expiration Date
```json
{
  "step": 4,
  "rule_id": "S4_R4",
  "rule_name": "Owner 2 country triggers expiration date field",
  "trigger": "#owner_country2",
  "triggerLabel": "Owner 2 Country",
  "target": "#secondExpirationDate",
  "targetLabel": "Owner 2 License Expiration Date",
  "targetWrapper": ".owner2_expiration_date",
  "value": "1",
  "valueName": "United States",
  "effect": "Makes date picker visible and required (US only)",
  "message": "Required because your country is United States."
}
```

**Note:** Owner 2 entire section is conditionally shown if Owner 1 ownership < 51% (separate JS logic, not DependentFields.js)

---

### STEP 5: Banking & Payment Information (2 Rules)

#### Rule 5.1: Current Processing → Processor Name
```json
{
  "step": 5,
  "rule_id": "S5_R1",
  "rule_name": "Current processing selection triggers processor name",
  "trigger": "#current_processing",
  "triggerLabel": "Currently Processing Credit Cards",
  "target": "#processor_name",
  "targetLabel": "Processor Name",
  "targetWrapper": ".processor_name_container",
  "value": "1",
  "valueName": "Yes",
  "effect": "Makes text input visible and required",
  "maxLength": 100,
  "message": "Required because you are currently Processing Credit Cards."
}
```

#### Rule 5.2: Country Canada → Customer Pay Currency
```json
{
  "step": 5,
  "rule_id": "S5_R2",
  "rule_name": "Canada country triggers currency selection",
  "trigger": "#country",
  "triggerLabel": "Country",
  "target": "#customer_pay_currency",
  "targetLabel": "Customer Pay Currency",
  "targetWrapper": ".currency_container",
  "value": "2",
  "valueName": "Canada",
  "effect": "Makes currency radio buttons visible and required (CA only)",
  "options": ["USD", "CAD"],
  "message": "Required because your country is Canada."
}
```

---

### STEP 6: Final Details & Preferences (2 Rules)

#### Rule 6.1: Referral Source → Additional Information
```json
{
  "step": 6,
  "rule_id": "S6_R1",
  "rule_name": "Referral source triggers additional info text field",
  "trigger": "#howdidyouhear",
  "triggerLabel": "How Did You Find Us",
  "target": "#hear_about_us_other",
  "targetLabel": "Additional Information",
  "targetWrapper": ".hear_about_us_other_container",
  "value": ["50", "14", "8"],
  "valueNames": ["Other", "Friend", "Live Event / Trade Show"],
  "effect": "Makes text input visible and required",
  "maxLength": 100,
  "messages": [
    "Required because you selected Other for how you found us.",
    "Required because you selected Friend for how you found us.",
    "Required because you selected Live Event / Trade Show for how you found us."
  ]
}
```

#### Rule 6.2: Bad Experience → Provider Name
```json
{
  "step": 6,
  "rule_id": "S6_R2",
  "rule_name": "Bad experience triggers provider name text field",
  "trigger": "#bad_experience",
  "triggerLabel": "Bad Experience with Provider",
  "target": "#bad_experience_happened",
  "targetLabel": "Bad Provider Name",
  "targetWrapper": ".bad_experience_happened_container",
  "value": "1",
  "valueName": "Yes",
  "effect": "Makes text input visible and required",
  "maxLength": 100,
  "message": "Required because you selected Yes for bad experience."
}
```

---

## Cross-Step Dependencies

### Implicit Dependencies (Non-DependentFields)

#### Owner 2 Visibility (Step 4)
```
Ownership Percentage < 51% → Owner 2 section shown + all fields required
Ownership Percentage ≥ 51% → Owner 2 section hidden + all fields cleared + not required
```
**Implementation:** Manual JavaScript in step4/js.blade.php

#### Physical Address Visibility (Step 2)
```
is_physical_address_same_as_legal_address = "1" (Yes) → Physical address fields hidden
is_physical_address_same_as_legal_address = "0" (No) → Physical address fields visible + required
```
**Implementation:** Manual JavaScript with radio button toggle

#### Marketing Model Checkbox Logic (Step 2)
```
marketingModel[] includes "2" → Subscription frequency section visible + required
marketingModel[] does NOT include "2" → Subscription frequency section hidden + cleared
```
**Implementation:** Custom change handler + DependentFields.js

---

## Field Requirement State Changes

### Progressive Requirement
Some fields start as optional but become required based on conditions:

| Field | Initial State | Triggers | Final State |
|-------|---------------|----------|------------|
| business_state | Hidden | country=1 | Visible + Required |
| industry_type_other | Hidden | industry_type=50 | Visible + Required |
| subscription_frequency | Hidden | marketingModel includes 2 | Visible + Required |
| fulfillment_company | Disabled | fulfillment_by in [2,3] | Enabled + Required |
| shopping_cart_other | Disabled | shopping_cart in [8,58] | Enabled + Required |
| processor_name | Disabled | current_processing=1 | Enabled + Required |
| hear_about_us_other | Disabled | howdidyouhear in [50,14,8] | Enabled + Required |
| bad_experience_happened | Disabled | bad_experience=1 | Enabled + Required |

---

## Dependent Field Patterns

### Pattern 1: Single Parent → Single Child (Simple)
- Example: Industry Type (50) → Industry Type Other
- Validation: Parent value = trigger value
- Effect: Show child, make required
- Clear: Remove required, hide, clear value

### Pattern 2: Checkbox Parent → Single Child (Moderate)
- Example: Marketing Model includes "2" → Subscription Frequency
- Validation: Array includes trigger value
- Effect: Show child, make required
- Clear: Not triggered by other checkboxes being unchecked

### Pattern 3: Multi-Value Parent → Single Child (Conditional)
- Example: Referral Source [50, 14, 8] → Additional Info
- Validation: Parent value matches ANY in array
- Effect: Show child, make required, custom message per value
- Messages: Different message for each trigger value

### Pattern 4: Select + Dependent + Further Dependent (Chain)
- Example: Fulfillment (2 or 3) → Vendor Name (required) → Can be submitted
- Level 1: Fulfillment type triggers vendor visibility
- Level 2: If vendor not filled, form validation fails
- No further chaining in current implementation

---

## Validation Integration

### DependentFields.js Validation Flow
1. Field value changes (trigger field)
2. DependentFields.js detects change
3. Checks if value matches rule condition
4. If match: Applies "required" rule to target field via jQuery Validation
5. If no match: Removes "required" rule, clears value, hides field
6. Re-validates form if already validated once

### jQuery Validation Integration
```javascript
$('#target_field').rules('add', { required: true });
// Or
$('#target_field').rules('remove', 'required');
```

### Form Submission Check
- Server receives only visible, enabled fields
- Hidden/disabled fields not submitted
- Validation happens client-side + server-side

---

## Testing Checklist

### Per Dependent Field Rule
- [ ] Trigger field changes to specified value → target appears + becomes required
- [ ] Trigger field changes to different value → target disappears + gets cleared
- [ ] Re-visiting step with saved data → dependent field state matches saved value
- [ ] Form validation fails if dependent required field empty
- [ ] Error message displays correctly
- [ ] Multi-value rules work for ALL specified trigger values
- [ ] Chain dependencies work (if applicable)

### Cross-Field Logic
- [ ] Owner 2 section appears only when Owner 1 < 51%
- [ ] Owner 2 ownership max value adjusts dynamically
- [ ] Physical address toggle shows/hides all 6 fields
- [ ] Subscription frequency disabled when no "Recurring" checked

---

## Common Implementation Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Dependent field doesn't appear | Trigger value string/number mismatch | Ensure trigger value matches exactly (string "1" vs int 1) |
| Validation passes with empty dependent field | Rule not applied | Check DependentFields.js selector matches form field |
| Error message says "required" but field hidden | jQuery validation rule persists | Ensure rules('remove') called when hiding |
| Owner 2 doesn't appear at < 51% | Separate JS logic issue | Check if percentage change handler exists |
| Checkbox dependent field not working | Trigger selector incorrect | Use checkbox selector, not dropdown selector |

---

## Summary by Step

| Step | Rule Count | Complexity | Key Pattern |
|------|-----------|-----------|-----------|
| 1 | 1 | Low | Simple country→state |
| 2 | 5 | High | Multiple patterns, country affects 2 rules |
| 3 | 2 | Moderate | Multi-value triggers, label updates |
| 4 | 4 | Moderate | Paired rules (license + expiration) |
| 5 | 2 | Low | Yes/No + country logic |
| 6 | 2 | Low | Multi-value array + single value |
| **Total** | **16** | **Moderate** | **All major patterns covered** |

---

## Best Practices

1. **Naming:** Use IDs matching field names (e.g., `#country`, not `#select_country`)
2. **Wrapper Classes:** Unique, descriptive class per dependency group
3. **Messages:** Specific to parent value (not generic "required")
4. **Validation:** Always remove rules before clearing fields
5. **Testing:** Test state changes at each step, not just final form
6. **Performance:** DependentFields.js is lightweight; safe for all 6 steps
7. **Accessibility:** Error messages should reference both parent and child fields
