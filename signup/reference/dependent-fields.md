# Dependent Fields Reference

Complete guide to all 16 conditional field dependencies across the 6-step signup flow.

---

## Overview

Dependent fields create dynamic form experience where field visibility and requirements change based on parent field values.

**Total Rules:** 16  
**Implementation:** DependentFields.js library  
**Activation:** On DOM ready

---

## Step 1: Account Information (1 Rule)

### Rule 1.1: Country → State

**Trigger:** Formation Country field changes  
**Target:** Formation State field  
**Condition:** Country = "1" (United States)

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
- When country = "1" → State becomes visible and required
- When country ≠ "1" → State hidden and cleared
- State field disabled for non-US countries

**Implementation:**
```javascript
if (businessCountry === "1") {
  showStateField();
  makeStateRequired();
} else {
  hideStateField();
  clearStateValue();
}
```

---

## Step 2: Company Details (5 Rules)

### Rule 2.1: Industry → Other

**Trigger:** Industry Type field changes  
**Target:** Industry Type (Other) text field  
**Condition:** Industry = "50" (Other)

```json
{
  "trigger": "#industry_type",
  "target": "#industry_type_other",
  "targetWrapper": ".industry_type_other_container",
  "value": "50",
  "message": "Required because you selected Other for industry type."
}
```

**Behavior:**
- When industry = "50" → Other field visible and required
- When industry ≠ "50" → Other field hidden and cleared

### Rule 2.2: Marketing Model → Subscription Frequency

**Trigger:** Marketing Model checkboxes  
**Target:** Subscription Frequency dropdown  
**Condition:** Marketing model includes "2" (Recurring)

```json
{
  "trigger": "input[name='marketingModel[]']",
  "target": "#subscription_frequency",
  "targetWrapper": ".subscription_frequency_container",
  "value": "2",
  "message": "Required because you selected Recurring/Subscription for marketing model."
}
```

**Behavior:**
- When marketing model includes value "2" → Frequency visible and required
- When "2" unchecked → Frequency hidden and cleared
- Only one frequency can be selected

### Rule 2.3: Subscription Frequency → Frequency Other

**Trigger:** Subscription Frequency dropdown changes  
**Target:** Subscription Frequency (Other) text field  
**Condition:** Frequency = "4" (Other)

```json
{
  "trigger": "#subscription_frequency",
  "target": "#subscription_frequency_other",
  "targetWrapper": ".subscription_frequency_other_container",
  "value": "4",
  "message": "Required because you selected Other for subscription frequency."
}
```

**Behavior:**
- When frequency = "4" → Other text visible and required
- When frequency ≠ "4" → Other text hidden and cleared

### Rule 2.4: Country → State

**Trigger:** Company Country field changes  
**Target:** Legal Address State field  
**Condition:** Country = "1" (United States)

```json
{
  "trigger": "#country",
  "target": "#state",
  "targetWrapper": ".state_container",
  "value": "1",
  "message": "Required because your country is United States."
}
```

**Behavior:**
- When country = "1" → State visible and required
- When country ≠ "1" → State hidden and cleared
- State field only accepts valid US state codes

### Rule 2.5: Country → Institution Number (Canada)

**Trigger:** Company Country field changes  
**Target:** Institution Number field  
**Condition:** Country = "2" (Canada)

```json
{
  "trigger": "#country",
  "target": "#institution_number",
  "targetWrapper": ".institution_number_container",
  "value": "2",
  "message": "Required because your country is Canada."
}
```

**Behavior:**
- When country = "2" → Institution number visible and required
- When country ≠ "2" → Institution number hidden and cleared
- Institution number: 3 digits (Canadian transit number)

---

## Step 3: Product Information (2 Rules)

### Rule 3.1: Fulfillment Method → Vendor Name

**Trigger:** Fulfillment Method dropdown changes  
**Target:** Fulfillment Vendor Name text field  
**Condition:** Fulfillment = "2" (Vendor) or "3" (Other)

```json
{
  "trigger": "#fulfillment_by",
  "target": "#fullfillment_company",
  "targetWrapper": ".fulfillment_vendor_container",
  "value": ["2", "3"],
  "message": [
    "Required because you selected Vendor for fulfillment method.",
    "Required because you selected Other for fulfillment method."
  ]
}
```

**Behavior:**
- When fulfillment = "1" (In-house) → Vendor field hidden and cleared
- When fulfillment = "2" or "3" → Vendor field visible and required
- Error message varies by selected value
- Max 100 characters for vendor name

### Rule 3.2: Shopping Cart → Other

**Trigger:** Shopping Cart dropdown changes  
**Target:** Shopping Cart Other text field  
**Condition:** Shopping Cart = "8" (Other) or "58" (API/Custom)

```json
{
  "trigger": "#shopping_cart",
  "target": "#shopping_cart_other",
  "targetWrapper": ".shopping_cart_other_container",
  "value": ["8", "58"],
  "message": [
    "Required because you selected Other Sales Platform.",
    "Required because you selected API / Custom Integration."
  ]
}
```

**Behavior:**
- When cart = "8" → Show "Other Sales Platform" label + placeholder "Instagram DMs, Custom CRM, etc."
- When cart = "58" → Show "API / Custom Integration Name" label + placeholder "API / Custom Integration Name"
- When cart ≠ "8" and ≠ "58" → Field hidden and cleared
- Error message varies by selected value

---

## Step 4: Beneficial Owners (4 Rules)

### Rule 4.1: Owner 1 Country → Driver License State

**Trigger:** Owner 1 Country dropdown changes  
**Target:** Owner 1 Driver License State field  
**Condition:** Country = "1" (United States)

```json
{
  "trigger": "#owner_country1",
  "target": "#owner_1_driver_license_state_input",
  "targetWrapper": ".owner1_license_state",
  "value": "1",
  "message": "Required because your country is United States."
}
```

**Behavior:**
- When country = "1" → License state visible and required
- When country ≠ "1" → License state hidden and cleared
- Disabled for non-US countries

### Rule 4.2: Owner 1 Country → Expiration Date

**Trigger:** Owner 1 Country dropdown changes  
**Target:** Owner 1 License Expiration Date field  
**Condition:** Country = "1" (United States)

```json
{
  "trigger": "#owner_country1",
  "target": "#firstExpirationDate",
  "targetWrapper": ".owner1_expiration_date",
  "value": "1",
  "message": "Required because your country is United States."
}
```

**Behavior:**
- When country = "1" → Expiration date visible and required
- When country ≠ "1" → Expiration date hidden and cleared
- Only shown for US merchants

### Rule 4.3: Owner 2 Country → Driver License State

**Trigger:** Owner 2 Country dropdown changes  
**Target:** Owner 2 Driver License State field  
**Condition:** Country = "1" (United States) AND Owner 2 section visible

```json
{
  "trigger": "#owner_country2",
  "target": "#owner_2_driver_license_state_input",
  "targetWrapper": ".owner2_license_state",
  "value": "1",
  "message": "Required because your country is United States."
}
```

**Behavior:** Same as Rule 4.1, but for Owner 2

### Rule 4.4: Owner 2 Country → Expiration Date

**Trigger:** Owner 2 Country dropdown changes  
**Target:** Owner 2 License Expiration Date field  
**Condition:** Country = "1" (United States) AND Owner 2 section visible

```json
{
  "trigger": "#owner_country2",
  "target": "#secondExpirationDate",
  "targetWrapper": ".owner2_expiration_date",
  "value": "1",
  "message": "Required because your country is United States."
}
```

**Behavior:** Same as Rule 4.2, but for Owner 2

### Additional Step 4 Logic (Not DependentFields.js Rules)

**Owner 2 Section Visibility:**
- Triggered by: Owner 1 Ownership Percentage
- Condition: Ownership < 51%
- Action: Show/hide entire Owner 2 section + make fields required/optional
- Handler: JavaScript change listener on ownership field

**Email Uniqueness:**
- Field: Owner email (both owners)
- Validation: AJAX POST to /signup/checkEmail
- Condition: Email must be unique and differ from other owner
- Triggered: On blur of email field

---

## Step 5: Banking Information (2 Rules)

### Rule 5.1: Current Processing → Processor Name

**Trigger:** Currently Processing dropdown changes  
**Target:** Processor Name text field  
**Condition:** Processing = "1" (Yes)

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
- When processing = "1" → Processor name visible and required
- When processing = "0" → Processor name hidden and cleared
- Max 100 characters for processor name

### Rule 5.2: Country → Currency (Canada Only)

**Trigger:** Company Country dropdown changes  
**Target:** Customer Pay Currency radio buttons  
**Condition:** Country = "2" (Canada)

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
- When country = "2" → Currency radio buttons visible and required (USD or CAD)
- When country ≠ "2" → Currency hidden and defaults to USD
- Options: USD or CAD

---

## Step 6: Final Details (2 Rules)

### Rule 6.1: Referral Source → Additional Info

**Trigger:** How Did You Find Us dropdown changes  
**Target:** Additional Information text field  
**Condition:** Referral = "50" (Other) OR "14" (Friend) OR "8" (Live Event)

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

**Behavior:**
- When referral = "50", "14", or "8" → Additional info visible and required
- When referral = other → Additional info hidden and cleared
- Error message varies by selected referral source
- Max 100 characters
- Dynamic placeholder based on referral type

### Rule 6.2: Bad Experience → Provider Name

**Trigger:** Bad Experience dropdown changes  
**Target:** Bad Provider Name text field  
**Condition:** Bad Experience = "1" (Yes)

```json
{
  "trigger": "#bad_experience",
  "target": "#bad_experience_happened",
  "targetWrapper": ".bad_experience_happened_container",
  "value": "1",
  "message": "Required because you selected Yes for bad experience."
}
```

**Behavior:**
- When bad experience = "1" → Provider name visible and required
- When bad experience = "0" → Provider name hidden and cleared
- Max 100 characters for provider name
- Examples: "Square", "Stripe", "PayPal"

---

## Implementation Checklist

- [ ] Review all 16 rules
- [ ] Understand trigger → target mapping
- [ ] Map to HTML element IDs in form
- [ ] Set up DependentFields.js configuration
- [ ] Test each rule independently
- [ ] Test rule combinations (multiple dependent fields)
- [ ] Test clearing fields when condition no longer met
- [ ] Test error messages display correctly
- [ ] Test form submission with/without dependent fields
- [ ] Verify validation works for conditional fields
- [ ] Test across browsers and mobile

---

## Testing Checklist

### Step 1
- [ ] Select US → State appears
- [ ] Select non-US → State hidden

### Step 2
- [ ] Select "Other" industry → Industry other appears
- [ ] Select "Recurring" marketing → Frequency appears
- [ ] Select "Other" frequency → Frequency other appears
- [ ] Select US country → State appears
- [ ] Select Canada → Institution number appears

### Step 3
- [ ] Select "Vendor" fulfillment → Vendor name appears
- [ ] Select "Other" fulfillment → Vendor name appears
- [ ] Select "In-house" → Vendor name hidden
- [ ] Select "Other" shopping cart → Other name appears
- [ ] Select "API/Custom" → Other name appears

### Step 4
- [ ] Owner 1 < 51% → Owner 2 section appears
- [ ] Owner 1 ≥ 51% → Owner 2 section hidden
- [ ] Owner 1 country = US → License fields appear
- [ ] Owner 1 country ≠ US → License fields hidden
- [ ] Same for Owner 2

### Step 5
- [ ] Select "Yes" processing → Processor name appears
- [ ] Select "No" processing → Processor name hidden
- [ ] Select Canada → Currency radio buttons appear
- [ ] Select non-Canada → Currency hidden

### Step 6
- [ ] Select "Other", "Friend", or "Live Event" → Additional info appears
- [ ] Select other referral → Additional info hidden
- [ ] Select "Yes" bad experience → Provider name appears
- [ ] Select "No" bad experience → Provider name hidden

---

## See Also

- [Validation Rules](./validation-rules.md)
- [Error Codes](./error-codes.md)
- [Enums Reference](./enums.md)
- [Guides](../guides/)
