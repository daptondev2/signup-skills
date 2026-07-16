---
name: signup-form-creation
description: Create signup form UI with all form steps, fields, and dynamic behaviors
---

# Signup Form Creation Skill

## Overview
This skill provides comprehensive guidelines for creating a complete merchant signup form across 6 steps with full dynamic field management and context-aware field rendering.

## Design Principles
- **User-centric flow**: Step-by-step progression reduces cognitive load
- **Progressive disclosure**: Show only relevant fields based on user selections
- **Auto-population support**: Fields can be pre-populated from external sources
- **Context-aware validation**: Validation rules adapt to user selections
- **Accessibility**: All dynamic fields have clear dependency messages
- **API-driven dropdowns**: All select options fetched from backend APIs

## API Integration Overview

### Base URL
**Domain**: `https://emap.epd.dev`

All dropdown options are fetched via REST API from `https://emap.epd.dev/api/partner/` endpoints.

### Response Format
All API endpoints return data in this format:
```json
{
  "data": [
    { "slug": "value_key", "name": "Display Label" },
    { "slug": "value_key2", "name": "Display Label 2" }
  ]
}
```

**Important**: 
- `slug` = form field value (what gets saved to database)
- `name` = dropdown display label (what user sees)

### API Endpoints Mapping

| Endpoint | Used In | Field |
|----------|---------|-------|
| `https://emap.epd.dev/api/partner/countries` | Step 1, 2 | Formation Country, Company Country |
| `https://emap.epd.dev/api/partner/states` | Step 1, 2 | Formation State |
| `https://emap.epd.dev/api/partner/industry-types` | Step 2 | Industry Type |
| `https://emap.epd.dev/api/partner/referral-sources` | Step 6 | How did you find us |
| `https://emap.epd.dev/api/partner/shopping-carts` | Step 6 | Transaction Device |
| `https://emap.epd.dev/api/partner/interest-details` | Step 6 | Help Your Company Grow (Interests) |

### Dropdown Loading Strategy
- **Load on page mount**: Fetch all dropdown data on step load
- **Cache locally**: Store API responses in component/form state
- **Show loading states**: Display spinner while fetching
- **Error handling**: Show fallback message if API fails
- **Dependency-based loading**: Load fields only when parent field shows

---

## Form Submission & Navigation

### Form Submission Endpoints

| Endpoint | Method | Purpose | Payload |
|----------|--------|---------|---------|
| `https://emap.epd.dev/api/v1/signup` | POST | **Initial signup** (Step 1) | Owner details, company info |
| `https://emap.epd.dev/api/v1/application/step` | POST | **Submit each step** (Steps 2-6) | Step data, step number, uuid |
| `https://emap.epd.dev/api/v1/ownership` | POST | **Submit ownership info** (Step 4) | Owner details, percentages |

### Step Submission Flow

```
Step 1 (Account Info)
  └─ POST https://emap.epd.dev/api/v1/signup
     ├─ Payload: first_name, last_name, email, phone, company_name, website, country, business_state, annual_sales
     ├─ Response: { uuid, step_count, redirect_url }
     └─ Navigate to: Step 2 with uuid

Step 2 (Company Information)
  └─ POST https://emap.epd.dev/api/v1/application/step
     ├─ Payload: step_count=2, uuid, legal_name, name, industry_type, phone, location_type, formation_date, etc.
     ├─ Response: { success, step_count, next_step_url }
     └─ Navigate to: Step 3

Step 3 (Product Information)
  └─ POST https://emap.epd.dev/api/v1/application/step
     ├─ Payload: step_count=3, uuid, card_swiped, customer_entered, staff_entered, transaction_amounts, etc.
     └─ Navigate to: Step 4

Step 4 (Owner Information)
  ├─ POST https://emap.epd.dev/api/v1/ownership
  │  ├─ Payload: owner[1], owner[2] (if applicable), ownership percentages
  │  └─ Response: { success }
  └─ POST https://emap.epd.dev/api/v1/application/step
     ├─ Payload: step_count=4, uuid
     └─ Navigate to: Step 5

Step 5 (Banking Information)
  └─ POST https://emap.epd.dev/api/v1/application/step
     ├─ Payload: step_count=5, uuid, institution_number, customer_pay_currency, routing_number, account_number, etc.
     └─ Navigate to: Step 6

Step 6 (Final Details)
  └─ POST https://emap.epd.dev/api/v1/application/step
     ├─ Payload: step_count=6, uuid, howdidyouhear, multiple_merchant_accounts, transaction_device, bad_experience, other_interests_capital
     ├─ Response: { success, redirect_url (dashboard or ccbill) }
     └─ Navigate to: Dashboard or CCBill
```

### Error Handling

**On API Error**:
```
{
  "success": false,
  "message": "Error message",
  "errors": {
    "field_name": "Validation error message"
  }
}
```

**Handling**:
- Display validation error messages below relevant fields
- Show toast notification for critical errors
- Disable submit button while processing
- Allow user to retry

### Client-Side Validation Before Submission

Before sending to API, validate:
1. All required fields have values
2. Email format is valid
3. Phone number is valid
4. URLs are valid
5. Dependent fields are visible and validated
6. Form hasn't already been submitted (prevent double submit)

---

## Form Structure

### STEP 1: Account Information
**Purpose**: Collect initial account and company setup details
**Key Validation**: All fields required except Referral Code

#### Fields:
1. **Owner Details** (from partial)
   - First Name (required)
   - Last Name (required)
   - Email Address (required, email format)
   - Phone Number (required, validates international format)

2. **Company Name** (required)
   - Type: Text input
   - Max length: 60 characters
   - Placeholder: "Company Name"
   - Auto-save enabled for lander prefill

3. **Company Website** (required)
   - Type: URL input
   - Validation: Must be valid URL format
   - Placeholder: "Company Website"
   - Hint: "If unavailable, please link to your social media profile (Facebook, Instagram, etc)"
   - Tooltip: "Add a website url or social media profile url if you currently have no active website"

4. **Formation Country** (required, dynamic trigger)
   - Type: Searchable dropdown select
   - Options: Full countries list with live search enabled
   - **API Endpoint**: `GET /api/partner/countries`
   - **Response Format**: `{ slug: "country_code", name: "Country Name" }`
   - Field Value: Uses `slug` (country code like "1" for US, "2" for Canada)
   - Field Label: Uses `name` (full country name)
   - Searchable: Yes (enable live search)
   - Trigger: Controls visibility of Formation State field
   - **Dynamic Rule**: 
     - Value = "1" (United States) → Show "Formation State" field
     - Any other value → Hide "Formation State" field
   - Hint: "Select the country where your company is legally registered"

5. **Formation State** (dynamic, required when country = US)
   - Type: Dropdown select
   - Visibility: Conditional (hidden by default)
   - Shows when: Formation Country = "1" (United States)
   - **API Endpoint**: `GET /api/partner/states`
   - **Response Format**: `{ slug: "state_code", name: "State Name" }`
   - Field Value: Uses `slug` (state code)
   - Field Label: Uses `name` (full state name)
   - Required message: "Required because your Formation Country is the United States"
   - Placeholder: "Please Select"

6. **Estimated Annual Sales** (conditional variant, required for no_apollo & textract variants)
   - Type: Currency input with formatting
   - Max length: 16 characters
   - Input mode: Numeric
   - Format: Cleave.js with thousand separator (commas)
   - Placeholder: "1,000,000"
   - Unit: USD
   - Validation: Max 12 digits before formatting
   - Error message: "Annual sales must not exceed $999,999,999,999"
   - Tooltip: "Please only include the revenue that will be processed via Credit Card Processing"

7. **Referral Code** (optional)
   - Type: Text input
   - Placeholder: "Referral Code"
   - Optional note displayed

#### Dynamic Behaviors:
- **Auto-save**: When all required fields are filled and user is from lander
- **State field dependency**: Hidden by default, shows only when country is US
- **Phone validation**: Uses intl-tel-input library with US/CA country order
- **Annual sales masking**: Comma formatting on blur

---

### STEP 2: Company Information
**Purpose**: Detailed company profile and operational details
**Key Validation**: All marked fields required

#### Card 1: Company Details

1. **Legal Business Name** (required)
   - Type: Text input
   - Placeholder: "Legal Company Name"
   - Auto-populate support (with source indicator)
   - Hint: Used for compliance/legal documents

2. **Company Name (DBA)** (required)
   - Type: Text input
   - Placeholder: "Company Name (DBA)"
   - Auto-populate support

3. **Industry Type** (required, dynamic trigger)
   - Type: Searchable dropdown
   - **API Endpoint**: `GET /api/partner/industry-types`
   - **Response Format**: `{ slug: "industry_code", name: "Industry Name" }`
   - Field Value: Uses `slug` (industry code)
   - Field Label: Uses `name` (full industry name)
   - Live search enabled
   - Trigger: Controls visibility of "Industry Type Other" field
   - **Dynamic Rule**:
     - When specific "Other" option selected → Show "Industry Type Other" field
   - Auto-populate support
   - Searchable with 'Please Select' placeholder

4. **Industry Type Other** (dynamic, required when industry = "Other")
   - Type: Text input
   - Visibility: Conditional (hidden by default)
   - Shows when: Industry Type = "Other" option
   - Placeholder: "Industry Type Other"
   - Auto-populate support

5. **Estimated Annual Sales** (conditional - variant A only)
   - Type: Currency input
   - Max length: 16 characters
   - Format: Cleave.js thousand separator
   - Placeholder: "1,000,000"
   - Unit: USD
   - Auto-populate support

6. **Customer Service Phone Number** (required)
   - Type: Tel input
   - Uses intl-tel-input library
   - Auto-populate support
   - No placeholder (standard phone format)

7. **Company Country** (dynamic, visible only if from_lander)
   - Type: Dropdown select
   - Visibility: Conditional (hidden unless from_lander = true)
   - Controls business registration number visibility

8. **Type of Company Location** (required)
   - Type: Dropdown select
   - Options: business_location values
   - Placeholder: "Please Select"
   - Auto-populate support

9. **Company Formation Date** (required)
   - Type: Date input
   - Format: YYYY-MM-DD
   - Hint: "YYYY-MM-DD"
   - Auto-populate support

10. **Company Legal Structure** (required, dynamic trigger)
    - Type: Dropdown select
    - Options: business_organized values
    - Trigger: May affect tax ID label/requirements
    - Placeholder: "Please Select"
    - Auto-populate support

11. **Federal Tax ID** (required)
    - Type: Text input with numeric mode
    - Max length: 20 characters
    - Placeholder: "Federal Tax ID"
    - Input mode: Numeric
    - Label: Dynamic based on country
      - US: "Federal Tax ID"
      - Non-US: "Federal Tax ID (or Corporation Tax Number equivalent)"
    - Tooltip: "Federal Tax ID is Federally required. All information is encrypted and stored in 256-bit encryption"
    - Locked icon displayed
    - Auto-populate support
    - Hint text: "Sole proprietors: You will be asked for your SSN when setting up your account" (shown conditionally)

12. **Business Register Number** (dynamic, required when country ≠ US)
    - Type: Text input
    - Visibility: Conditional (hidden by default)
    - Shows when: Country ≠ "1" (United States)
    - Max length: Dynamic based on country (Canada = 11, others = 20)
    - Placeholder: "Business Registration Number"
    - Required message: "Required because your Formation Country is not the United States"
    - Auto-populate support

#### Card 2: Revenue Model & Subscription

13. **Revenue Model** (required, dynamic trigger)
    - Type: Checkbox group
    - Options: Multiple checkboxes for marketing models
    - Trigger: Controls visibility of "Subscription Frequency" field
    - **Dynamic Rule**: 
      - When checkbox "Subscription" (id=2) is checked → Show "Subscription Frequency" field
    - Auto-populate support
    - Display: Vertical checkbox list

14. **Subscription Frequency** (dynamic, required when revenue = subscription)
    - Type: Dropdown select
    - Visibility: Conditional (hidden by default)
    - Shows when: Revenue Model includes "Subscription" (id=2)
    - Options: Monthly (1), Yearly (2), Other (3)
    - Trigger: Controls visibility of "Subscription Frequency Other" field
    - **Dynamic Rule**:
      - When value = "3" (Other) → Show "Subscription Frequency Other" field
    - Placeholder: "Please Select"
    - Auto-populate support

15. **Subscription Frequency Other** (dynamic, required when subscription = "Other")
    - Type: Text input
    - Visibility: Conditional (hidden by default)
    - Shows when: Subscription Frequency = "3" (Other)
    - Placeholder: "Other"
    - Min length: 5 characters
    - Auto-populate support

#### Card 3: Company Address
- Includes full address fields (see partial)
- Auto-complete with Google Maps API
- Required fields: Street, City, State/Province, Postal Code, Country

#### Card 4: Physical Address
- Separate from company address
- May differ from company address
- Full address validation

---

### STEP 3: Product Information
**Purpose**: Transaction methods and product/service details

#### Card 1: Transaction Entry Methods

1. **Transaction Entry Methods Slider** (required)
   - Type: Range slider with 3-way distribution
   - Visual: Dual/triple range slider with percentages
   - Fields:
     - **Card Swiped %** (is-first)
       - Label: "Customer presents card at point of sale"
       - Hidden input: card_swiped
       - Required, max=100
     - **Customer Entered %** (is-second)
       - Label: "Customer enters card details online"
       - Hidden input: customer_entered
       - Required, max=100
       - Default value: 100
     - **Staff Entered %** (is-third)
       - Label: "Card details are manually entered"
       - Hidden input: staff_entered
       - Required, max=100
   - Validation: Percentages must sum to 100%
   - Display: Real-time percentage labels update as slider moves
   - Hint: "Approximate percentage breakdown of how payment card information is collected from customers"

#### Card 2: Fulfillment Methods
- Partial include (see fulfillment.blade.php)
- Checkboxes for delivery/fulfillment types

#### Card 3: Product Details

2. **Average Transaction Amount** (required)
   - Type: Currency input
   - Max length: 16 characters
   - Input mode: Numeric
   - Placeholder: "200"
   - Hint: "The average transaction amount (USD) you expect to process"
   - Format: Cleave.js thousand separator
   - Auto-populate support

3. **Highest Transaction Amount** (required, trigger)
   - Type: Currency input
   - Max length: 16 characters
   - Input mode: Numeric
   - Placeholder: "1,000"
   - Hint: "The highest single transaction amount (USD) you expect to process"
   - Format: Cleave.js thousand separator
   - Validation: Must be ≥ Average Transaction Amount
   - Error display: Warning span shows validation error
   - Auto-populate support

4. **Products/Services Description** (required)
   - Type: Textarea
   - Placeholder: "Briefly describe your primary products or services"
   - Min length: Reasonable text (enforced via form submit)
   - Auto-populate support (describe_services)

5. **Highest Transaction Description** (required)
   - Type: Textarea
   - Placeholder: "Briefly describe your highest-priced product or service"
   - Min length: 15 characters
   - Auto-populate support (describe_highest_transaction)

#### Dynamic Behaviors:
- **Highest transaction warning**: Real-time validation warning if > average
- **Slider validation**: Ensures percentages sum to 100%

---

### STEP 4: Owner Information
**Purpose**: Collect details of company owners (up to 2 owners minimum)

#### Section: Persona
- **EXCLUDED PER REQUIREMENTS** (Plaid Persona Connection)

#### Section: Primary Contact Details
- Partial include (see primary_contact_details.blade.php)
- Contact information for business

#### Section: Owner #1 (Required)
**Card Header**: "[User Full Name]'s Details"

1. **First Name** (required, hidden, pre-filled from auth)
   - Type: Text input
   - Value: Pre-filled from Auth::user()->first_name
   - Visibility: Hidden input (display: none)

2. **Last Name** (required, hidden, pre-filled from auth)
   - Type: Text input
   - Value: Pre-filled from Auth::user()->last_name
   - Visibility: Hidden input

3. **Email Address** (required, hidden, pre-filled from auth)
   - Type: Email input
   - Value: Pre-filled from Auth::user()->email
   - Visibility: Hidden input

4. **Title** (required)
   - Type: Dropdown select
   - Options: ownerJobTitle values (ID => Job Title)
   - Required attribute
   - Auto-populate support (job_title)
   - Placeholder: "Select Job Title"

5. **SSN/SIN** (required, masked)
   - Type: Text input with numeric mode
   - Placeholder mask: Dynamic based on country
     - US/Canada/Mexico: "SSN/SIN"
     - Others: "SSN (or personal Tax ID equivalent)"
   - Label: Dynamic
     - US/Canada/Mexico: "SSN/SIN"
     - Others: "SSN (or personal Tax ID equivalent)"
   - Tooltip: "SSN is Federally required. All information is encrypted and stored in 256-bit encryption"
   - Locked icon displayed
   - Post-Hog mask: true (sanitizes in analytics)
   - Required

6. **Mobile Phone** (required)
   - Type: Tel input
   - Input mode: Numeric
   - Uses intl-tel-input library
   - International format support
   - Required

7. **% of Company Owned** (required, dynamic trigger)
   - Type: Number input
   - Min: 1, Max: 100
   - Input mode: Numeric
   - Placeholder: "% of Company Owned"
   - Trigger: Shows hint if value < 51%
   - **Dynamic Behavior**:
     - When value < 51% → Show additional owner hint: "Additional owner information required below"
     - When value ≥ 51% → Hide hint
   - Required

8. **Date of Birth** (required)
   - Type: Date input
   - Format: YYYY-MM-DD
   - Placeholder: "Date of Birth"
   - Auto-populate support (dob)

9. **Financial History** (partial include)
   - Includes bankruptcy history, judgments, liens questions

10. **Home Address** (partial include)
    - Full address fields with auto-complete

11. **License Details** (partial include)
    - Driver's license or ID document details

#### Section: Owner #2 (Conditional, appears when first owner ownership < 50%)

**Card Header**: "[Owner 2 Name] Details" (dynamic based on first_name + last_name if available, else "Owner #2")

1-8. **Same fields as Owner #1** (same structure and validation)
   - Not pre-filled (optional collection)
   - First Name, Last Name, Email, Title, SSN, Phone, Ownership %, DOB
   - Same placeholders and validation rules

9. **Financial History** (partial)
10. **Home Address** (partial)
11. **License Details** (partial)

#### Dynamic Behaviors:
- **Second owner section visibility**: Only required if Owner 1 ownership < 50%
- **Ownership percentage hint**: "Additional owner information required below" shows when first owner < 51%
- **SSN label localization**: Changes based on company country

---

### STEP 5: Banking Information
**Purpose**: Bank account and transaction processing currency setup

#### Section: Plaid Integration
- **EXCLUDED PER REQUIREMENTS** (Plaid Persona Connection)

#### Card: Bank Account Information

1. **Institution Number** (dynamic, required for Canada)
   - Type: Number input
   - Visibility: Conditional (hidden by default)
   - Shows when: Country = "2" (Canada)
   - Max length: 3 characters
   - Min: 0
   - Placeholder: "Institution Number"
   - Auto-populate support
   - Required message: Displayed only for Canada

2. **Currency Customer Will Pay In** (dynamic, required for Canada)
   - Type: Radio button group (2 options)
   - Visibility: Conditional (hidden by default)
   - Shows when: Country = "2" (Canada)
   - Options:
     - USD (value="USD")
     - CAD (value="CAD")
   - Required
   - Auto-populate support (customer_pay_currency)
   - Label: "Currency your customers will pay in"

#### Card: Transaction Information
- Partial include (see transaction_information.blade.php)
- Expected monthly volume, transaction frequency fields

---

### STEP 6: Final Details & Preferences
**Purpose**: Collection preferences, support, and terms agreement

#### Card 1: Referral Source

1. **How did you find us?** (required, dynamic trigger)
   - Type: Dropdown select
   - **API Endpoint**: `GET /api/partner/referral-sources`
   - **Response Format**: `{ slug: "source_code", name: "Source Name" }`
   - Field Value: Uses `slug` (e.g., "50", "14", "8")
   - Field Label: Uses `name` (e.g., "Other", "Friend", "Live Event / Trade Show")
   - Trigger: Controls visibility of "Additional Information" field
   - **Dynamic Rule**:
     - When value IN ["50", "14", "8"] (Other, Friend, Live Event) → Show "Additional Information" field
     - Other values → Hide field
   - Placeholder: "Please Select"
   - Required validation message: "Please Select"

2. **Additional Information** (dynamic, required when applicable)
   - Type: Text input
   - Visibility: Conditional (hidden by default)
   - Shows when: "How did you find us" = "50" (Other) OR "14" (Friend) OR "8" (Live Event)
   - Placeholder: "Other / Friend"
   - Required message: Varies by trigger value
     - Other: "Required because you selected Other for how you found us"
     - Friend: "Required because you selected Friend for how you found us"
     - Live Event: "Required because you selected Live Event / Trade Show for how you found us"
   - Auto-populate support (hear_about_us_other)

#### Card 2: Account Setup Preferences

3. **Do you need multiple merchant accounts?** (required)
   - Type: Dropdown select
   - Options: Yes (1), No (0)
   - Placeholder: "Please Select"
   - Boolean storage (0 or 1)
   - Required

4. **What device will you use to process transactions?** (optional)
   - Type: Dropdown select
   - **API Endpoint**: `GET /api/partner/shopping-carts`
   - **Response Format**: `{ slug: "device_code", name: "Device Name" }`
   - Field Value: Uses `slug` (device code)
   - Field Label: Uses `name` (full device name)
   - Placeholder: "Please Select"
   - Optional (no required attribute)
   - Auto-populate support (transaction_device)

5. **Have you had a bad experience with a provider?** (required, dynamic trigger)
   - Type: Dropdown select
   - Options: Yes (1), No (0)
   - Placeholder: "Please Select"
   - Trigger: Controls visibility of "What Provider?" field
   - **Dynamic Rule**:
     - When value = "1" (Yes) → Show "What Provider?" field
     - When value = "0" (No) → Hide field, clear & disable input
   - Required

6. **What Provider did you have a bad experience with?** (dynamic, required when applicable)
   - Type: Text input
   - Visibility: Conditional (hidden by default)
   - Shows when: "Bad experience" = "1" (Yes)
   - Placeholder: "What Provider did you have a bad experience with?"
   - Required message: "Required because you selected Yes for bad experience"
   - Auto-populate support (bad_experience_happened)
   - On disable: Clears value, removes validation classes

#### Card 3: Additional Interests & Terms

7. **Help Your Company Grow** (dynamic checkboxes from API)
   - Type: Checkbox group
   - **API Endpoint**: `GET /api/partner/interest-details`
   - **Response Format**: `{ slug: "interest_code", name: "Interest Name" }`
   - Field Value: Uses `slug` (interest code) - stored in array
   - Field Label: Uses `name` (full interest name)
   - Display: Vertical checkbox list with all interest options
   - Optional: User can select multiple interests
   - Examples from API: Capital, Resources, Marketing, Business Coaching, etc.
   - Note: Options are completely dynamic based on API response

8. **Terms and Conditions** (partial include)
   - Checkbox: "I agree to terms and conditions"
   - Required validation: Must be checked

#### Dynamic Modal: Dynamic Fields (Conditional)
- **Purpose**: Processor-specific dynamic fields
- **Trigger**: Shown if rule engine determines additional fields needed
- **Excluded**: Dynamic processor field (per requirements)
- Modal structure: Auto-generated form fields (text, textarea, checkbox, radio, divider)
- Field types supported: text, number, textarea, checkbox, radio, divider
- Validation: Rule-based (required, email, min, max)
- Examples: B2B/B2C percentage split, etc.

#### Dynamic Behaviors:
- **Referral source dependency**: Shows "Additional Information" for specific options
- **Bad experience dependency**: Shows/hides provider field
- **Modal auto-show**: Displays if dynamic fields needed on page load
- **Disabled state management**: Clears and disables fields when dependency not met
- **Validation toggling**: Re-validates dependent fields when trigger changes

---

## Context Window Optimization

### Smart Field Documentation
- Only relevant field details included per step
- Duplicate field patterns referenced (not repeated)
- Partials documented once with reference

### Data Model Structure
```json
{
  "step": 1,
  "fields": [
    {
      "name": "field_id",
      "type": "text|email|tel|number|date|select|checkbox|radio|textarea",
      "required": true,
      "validation": "rule1|rule2",
      "dynamic": false,
      "triggers_dependency": null,
      "depends_on": null,
      "dependency_condition": null
    }
  ]
}
```

### Validation Rules Reference
- **required**: Field must have value
- **email**: Valid email format
- **validUrl**: Valid URL format
- **validatePhone**: Valid phone number (intl-tel-input)
- **maxlength**: Maximum character length
- **minlength**: Minimum character length
- **min/max**: Numeric bounds
- **numeric**: Numbers only
- **validPercentage**: Whole number percentage
- **custom**: Step-specific rules (e.g., sum to 100%, >= previous field)

---

## Implementation Notes

### Auto-Save Strategy
- **Step 1**: Saves when all required fields filled (lander flow only)
- **Trigger**: On blur from last required field
- **Debounce**: Prevents rapid successive saves

### Dependent Fields Library
- **File**: dependent-fields.js
- **Usage**: DependentFields class with rules configuration
- **Pattern**:
  ```javascript
  new DependentFields({
    rules: [
      {
        trigger: '#selector',        // Field that triggers change
        target: '#target-selector',   // Field to show/hide
        targetWrapper: '.wrapper',    // Container for display control
        value: 'value',              // Value that triggers (string or array)
        message: 'msg'               // Validation message
      }
    ]
  });
  ```

### Pre-population Support
- All fields marked with auto-populate support have `$autoPopulatedFields` check
- Source indicator shown next to auto-populated fields
- Users can override auto-populated values

### Accessibility Considerations
- Labels associated with all inputs
- Hints and validation messages clear and contextual
- Tooltips for complex fields
- Keyboard navigation support
- Focus management for dependent fields

---

## Excluded Features (Per Requirements)

1. **Plaid Persona Connection** - Step 4 persona section excluded
2. **Dynamic Processor Fields** - Conditional dynamic modal form fields excluded
3. **Signature Step** - Step 7 (signature upload) not included

---

## Form Submission Flow

### Step Navigation
1. Step 1 → /step/2/{uuid}
2. Step 2 → /step/3/{uuid}
3. Step 3 → /step/4/{uuid}
4. Step 4 → /step/5/{uuid}
5. Step 5 → /step/6/{uuid}
6. Step 6 → /dashboard/merchant or /c/ccbill/{uuid}

### Endpoint: POST /signup/user (Step 1)
### Endpoint: POST /signup/companies (Steps 2-6)
### Endpoint: POST /signup/dynamic_fields (Dynamic fields)

---

## Reference Materials

### Auto-populate Helper Functions
- `isAuto(field, array)`: Check if field is auto-populated
- `autoSourceText(field, array)`: Display source indicator

### JavaScript Libraries
- jQuery Validation: Form validation
- Cleave.js: Number/currency formatting
- intl-tel-input: International phone support
- Bootstrap Selectpicker: Enhanced dropdowns
- Toastr: Toast notifications
- Google Maps Autocomplete: Address suggestions

### CSS Classes
- `.form-control`: Base input styling
- `.custom-select`: Dropdown styling
- `.is-invalid`: Error state
- `.ph-mask-input`: Post-hog masked field
- `.ph-unmask-input`: Post-hog unmasked field

---

## Testing Checklist

### Dynamic Field Scenarios
- [ ] US selected → State field appears
- [ ] Non-US country → State field hidden, business register number visible
- [ ] Industry = Other → Industry other field appears
- [ ] Subscription revenue → Subscription frequency field appears
- [ ] Subscription frequency = Other → Other field appears
- [ ] Owner 1 ownership < 50% → Owner 2 section appears
- [ ] Bad experience = Yes → Provider field appears
- [ ] Referral source = Other/Friend/Event → Additional info field appears
- [ ] Canada selected → Institution number and currency fields appear

### Form Behaviors
- [ ] Required field validation on submit
- [ ] Auto-save triggers on Step 1 (lander)
- [ ] Dependent fields show correct messages
- [ ] Slider percentages sum to 100%
- [ ] Currency formatting with thousand separators
- [ ] Phone validation with country support
- [ ] Disabled states clear values
- [ ] Pre-populated fields show source

---

## Field Reference By Step

| Step | Field Count | Dynamic Fields | Complex Fields |
|------|------------|---|---|
| 1 | 7 | 1 (State) | Slider (implicit), Phone |
| 2 | 15 | 3 (Industry other, Reg number, Subscription freq) | Address x2, Checkboxes |
| 3 | 5 | 0 | Slider (3-way), Textareas |
| 4 | 22 | 1 (Owner 2 visibility) | Owner x2, Address x2, License x2 |
| 5 | 2 | 2 (Institution, Currency) | Partial: Transaction info |
| 6 | 8 | 3 (Info detail, Provider detail, Dynamic modal) | Checkboxes, Modal form |
| **Total** | **59** | **10+** | **Multiple complex patterns** |

