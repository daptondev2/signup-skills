---
name: signup-form-creation
description: Create a complete 7-step merchant signup form with all fields, validations, and integrations
---

# Merchant Signup Form Specification

Production-ready specification for building the complete 7-step signup form.

---

## Form Overview

| Step | Title | Fields | Features |
|------|-------|--------|----------|
| 1 | Account Information | 9 | Phone formatting, country/state selection |
| 2 | Company Information | 28 | Google Maps autocomplete (legal + physical), revenue model with nested conditionals |
| 3 | Product Information | 13 | Fulfillment details, transaction slider (multiples of 5) |
| 4 | Owner Information | 32+ | Owner 1 & conditional Owner 2, financial history, addresses |
| 5 | Banking Information | 10 | Routing validation, country-specific fields |
| 6 | Final Details | 11 | Interest selection, terms acceptance |
| 7 | Signature | - | Display only (no submission) |

---

## STEP 1: Account Information

### Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| first_name | text | Yes | Max 60 chars |
| last_name | text | Yes | Max 60 chars |
| email | email | Yes | Valid email format |
| phone | tel | Yes | Use intl-tel-input library |
| name | text | Yes | Company name, max 60 chars |
| website | url | Yes | Valid URL or social media profile |
| country | select | Yes | API: `/api/partner/countries` (returns code: "US", "CA", etc) |
| business_state | select | Yes | Show only if country="US", API: `/api/partner/states` |
| annual_sales | text | No | Numeric, USD, variant-specific |
| promo_code | text | No | Optional referral code |

### Libraries Required
- `intl-tel-input@22.0.2` - Phone formatting with country selector
- `dependent-fields.js` - Conditional field visibility

### API Integration

**Dropdown Data**:
```
GET /api/partner/countries
GET /api/partner/states
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/signup
Headers: X-API-Key: {user.security_key}
Payload: all Step 1 fields + step_count: 1
Response: { uuid, step_count, message }
Store UUID in localStorage for all subsequent steps
```

### Validation
- First/Last name: required
- Email: valid format
- Phone: valid number (intl-tel-input validates)
- Website: valid URL
- Country: required
- Business State: required if country="1"

---

## STEP 2: Company Information

### Fields

**Company Details**:

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| legal_name | text | Yes | Max 60 chars |
| name | text | Yes | DBA name, max 60 chars |
| industry_type | select | Yes | API: `/api/partner/industry-types` |
| annual_sales | text | No | Numeric, variant-specific |
| customer_service_telephone_number | tel | Yes | Use intl-tel-input |
| business_location | select | Yes | Static (slug): Home-Based, Co-Working, Corporate-Office, Storefront, Others |
| business_formed | date | Yes | YYYY-MM-DD, use date picker |
| business_organized | select | Yes | Static (slug): Corporation, LLC, Partnership, Government, Sole-Proprietorship, Non-Profit, Other |
| federal_tax_id | text | Yes | Encrypted, max 20 chars |
| business_register_number | text | No | Show if country≠"US", max 20 chars |

**Revenue Model** (Checkbox Array with Dependent Hierarchy):

| Field | Type | Required | Name Attribute | Notes |
|-------|------|----------|---|---|
| marketingModel | checkbox array | Yes | marketingModel[] | Options: "One Time Purchase" (1), "Recurring/Subscription" (2), "Trial Offer + Subscription" (3) |
| subscription_frequency | select | Conditional | subscription_frequency | Show if "Recurring/Subscription" (2) checked, Options: "Monthly" (1), "Yearly" (2), "Other" (3) |
| subscription_frequency_other | text | Conditional | subscription_frequency_other | Show if subscription_frequency="Other" (3), min 5 chars |

**Revenue Model Conditional Logic**:
```
IF marketingModel includes "Recurring/Subscription" (value 2):
  - SHOW subscription_frequency dropdown
  - MAKE subscription_frequency REQUIRED
  
  IF subscription_frequency = "Other" (value 3):
    - SHOW subscription_frequency_other text field
    - MAKE subscription_frequency_other REQUIRED
ELSE:
  - HIDE subscription_frequency dropdown
  - HIDE subscription_frequency_other text field
  - CLEAR both fields
```

**Legal Address** (All fields REQUIRED):

| Field | Type | Name Attribute | Notes |
|-------|------|---|---|
| autocomplete | text | autocomplete | Google Maps Places autocomplete - shown initially if no address data exists |
| street_number | text | street_number | REQUIRED - Auto-populated from autocomplete, max 10 chars |
| street_address | text | route | REQUIRED - Auto-populated from autocomplete, max 60 chars |
| city | text | locality | REQUIRED - Auto-populated from autocomplete, max 60 chars |
| state | text | administrative_area_level_1 | REQUIRED - Auto-populated from autocomplete, max 40 chars |
| postal_code | text | postal_code | REQUIRED - Auto-populated from autocomplete, max 15 chars |
| country | select | country | REQUIRED - Auto-populated from autocomplete |

**Legal Address Autocomplete Behavior**:
- **Initially shown**: If no address data exists in the form
- **On selection**: User searches and selects address from Google autocomplete dropdown
- **Auto-population**: All 6 address fields populate from Google's address_components
- **After selection**: Autocomplete textbox hides, address fields display and become required

**Component Mapping** (Google address_components → Form fields):
- `street_number` (short_name) → field id=`street_number`
- `route` (long_name) → field id=`route`
- `locality` (long_name) → field id=`locality`
- `administrative_area_level_1` (short_name) → field id=`administrative_area_level_1`
- `postal_code` (short_name) → field id=`postal_code`
- `country` (long_name) → field id=`country` (selectpicker with refresh)

**Physical/Mailing Address** (Conditional on radio selection):

| Field | Type | Name Attribute | Notes |
|-------|------|---|---|
| is_physical_address_same_as_legal_address | radio | is_physical_address_same_as_legal_address | Options: yes=different, no=same. Default: no. Controls entire physical address section visibility |
| physical_address_autocomplete | text | physical_address_autocomplete | Google Maps autocomplete - shown only if radio="yes" AND no address data exists |
| physical_address_street_number | text | physical_address_street_number | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_street_address | text | physical_address_street_address | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_city | text | physical_address_city | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_state | text | physical_address_state | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_postal_code | text | physical_address_postal_code | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_country | select | physical_address_country | REQUIRED (when visible) - Auto-populated from autocomplete |

**Physical Address Behavior**:
1. **Radio controls section visibility**:
   - `radio value="no"` (default) → Physical address section hidden
   - `radio value="yes"` → Physical address section shown
2. **Autocomplete shown initially**: If radio="yes" AND no physical address data exists
3. **On selection**: User searches and selects address from Google autocomplete
4. **Auto-population**: All 6 address fields populate from Google's address_components
5. **After selection**: Autocomplete hides, address fields display and become required

**Component Mapping** (Google address_components → Form fields):
- `street_number` (short_name) → field id=`physical_address_street_number`
- `route` (long_name) → field id=`physical_address_street_address`
- `locality` (long_name) → field id=`physical_address_city`
- `administrative_area_level_1` (short_name) → field id=`physical_address_state`
- `postal_code` (short_name) → field id=`physical_address_postal_code`
- `country` (long_name) → field id=`physical_address_country` (selectpicker with refresh)

### Libraries Required
- `Google Maps Places API` - Address autocomplete (legal + physical)
- Date picker library
- `intl-tel-input` - Phone formatting

### Google Maps Implementation Details

**Load Script in Page Header**:
```html
<script src="https://maps.google.com/maps/api/js?key={GOOGLE_API_KEY}&libraries=places"></script>
```

**Legal Address Autocomplete Initialization**:
```javascript
google.maps.event.addDomListener(window, 'load', initialize);

var componentForm = {
    street_number: 'short_name',
    route: 'long_name',
    locality: 'long_name',
    administrative_area_level_1: 'short_name',
    country: 'long_name',
    postal_code: 'short_name'
};

function initialize() {
    var input = document.getElementById('autocomplete');
    var autocomplete = new google.maps.places.Autocomplete(input);
    
    autocomplete.addListener('place_changed', function() {
        var place = autocomplete.getPlace();
        
        // Extract and populate all address components
        for (var i = 0; i < place.address_components.length; i++) {
            var component = place.address_components[i];
            for (var j = 0; j < component.types.length; j++) {
                if (componentForm[component.types[j]]) {
                    var addressType = component.types[j];
                    var val = component[componentForm[addressType]];
                    document.getElementById(addressType).value = val;
                    
                    // Special handling for country selectpicker
                    if (addressType === 'country') {
                        var $countrySelect = $('#country');
                        $countrySelect.selectpicker('val', val);
                        $countrySelect.selectpicker('refresh');
                    } else {
                        $(document.getElementById(addressType)).trigger('change');
                        $(document.getElementById(addressType)).valid();
                    }
                    break;
                }
            }
        }
        
        // Show address fields, hide autocomplete
        $("#street_area").removeClass("d-none");
        $("#address_area").addClass("d-none");
    });
}
```

**Physical Address Autocomplete Initialization**:
```javascript
function initializePhysicalAddressAutocomplete() {
    var physicalInput = document.getElementById('physical_address_autocomplete');
    if (physicalInput) {
        var physicalAutocomplete = new google.maps.places.Autocomplete(physicalInput);
        
        physicalAutocomplete.addListener('place_changed', function() {
            var place = physicalAutocomplete.getPlace();
            
            // Map Google components to physical address field IDs
            var componentMapping = {
                street_number: 'physical_address_street_number',
                route: 'physical_address_street_address',
                locality: 'physical_address_city',
                administrative_area_level_1: 'physical_address_state',
                postal_code: 'physical_address_postal_code',
                country: 'physical_address_country'
            };
            
            // Extract and populate all address components
            for (var i = 0; i < place.address_components.length; i++) {
                var component = place.address_components[i];
                var addressType = null;
                
                for (var j = 0; j < component.types.length; j++) {
                    if (componentMapping[component.types[j]]) {
                        addressType = component.types[j];
                        break;
                    }
                }
                
                if (addressType) {
                    var val = component[componentForm[addressType]];
                    var fieldId = componentMapping[addressType];
                    var fieldElement = document.getElementById(fieldId);
                    
                    if (fieldElement) {
                        if (addressType === 'country') {
                            // Special handling for country selectpicker
                            var $countrySelect = $('#' + fieldId);
                            $countrySelect.selectpicker('val', val);
                            $countrySelect.selectpicker('refresh');
                            $countrySelect.trigger('change');
                        } else {
                            fieldElement.value = val;
                            $(fieldElement).trigger('change');
                            $(fieldElement).valid();
                        }
                    }
                }
            }
            
            // Show address fields, hide autocomplete
            $("#physical_address_street_area").removeClass("d-none");
            $("#physical_address_area").addClass("d-none");
        });
    }
}
```

**Key Points**:
- Both autocompletes use `types: ['geocode']` to restrict to address results
- Extract address components from Google's place object
- Map component types to form field IDs using componentForm mapping
- Trigger 'change' and 'valid' events on populated fields
- Update selectpicker with refresh for country fields
- Hide autocomplete container and show address fields after selection

### Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`). They only appear when trigger value is selected. When visible, dependent fields become REQUIRED.

**Level 1 Dependencies:**
- **business_state**: Show if country="US" (hidden by default)
- **business_register_number**: Show if country≠"US" (hidden by default)
- **Physical address section**: Show if is_physical_address_same_as_legal_address="yes" (hidden by default)
- **subscription_frequency**: Show if marketingModel includes "Recurring/Subscription" (2) (hidden by default)

**Level 2 Dependencies (Nested):**
- **subscription_frequency_other**: Show if subscription_frequency="Other" (3) (hidden by default)
  - Only visible when subscription_frequency is visible AND set to "Other"

### API Integration

**Dropdown Data**:
```
GET /api/partner/countries
GET /api/partner/industry-types
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 2
  section: company_info
  all Step 2 fields
Filter: Remove hidden/disabled fields before submission
Response: { next_step_url, message }
```

---

## STEP 3: Product Information

### Fields

**Fulfillment Details**:

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| customer_service_time | select | Yes | Static: slug values "0-7-days", "7-30-days", "31+days" |
| refund_policy | select | Yes | Static (slug): Full-Refund, No-Refund, Exchange-Only, Partial-Refund |
| fulfillment_by | select | Yes | Static (slug): Direct-By-You, Service-Only, Vendor, Others |
| fullfillment_company | text | Conditional | Show if fulfillment_by="Vendor" or "Others" |
| shopping_cart | select | Yes | API: `/api/partner/shopping-carts` (slug: "Other", "API / Custom Integration", etc) |
| shopping_cart_other | text | Conditional | Show if shopping_cart="Other" or "API / Custom Integration" |
| leave_deposit | select | Yes | Options: Yes(1), No(0) |

**Transaction Entry Methods**:

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| card_swiped | number | Yes | Slider: multiples of 5 (0-100%) |
| customer_entered | number | Yes | Slider: multiples of 5 (0-100%) |
| staff_entered | number | Yes | Slider: multiples of 5 (0-100%) |
| Validation | | | Sum must equal 100% |

**Product Details**:

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| average_transaction_amount | text | Yes | Numeric, USD |
| highest_transaction_amount | text | Yes | Numeric, USD |
| describe_services | textarea | Yes | Min 15 chars, max 1500 |
| describe_highest_transaction | textarea | Yes | Min 15 chars, max 1500 |

### Libraries Required
- `ion-rangeslider` - Three-way transaction slider with step: 5
- `dependent-fields.js` - Conditional field visibility

### Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`). They only appear when trigger value is selected.

- **fullfillment_company**: Show if fulfillment_by = "Vendor" or "Others" (slug values, hidden by default)
- **shopping_cart_other**: Show if shopping_cart = "Other" or "API / Custom Integration" (slug values, hidden by default)
- **Slider validation**: Sum of all three must equal 100%

### API Integration

**Dropdown Data**:
```
GET /api/partner/shopping-carts
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 3
  section: product_info
  all Step 3 fields
Filter: Remove hidden/disabled fields
Response: { next_step_url, message }
```

---

## STEP 4: Owner Information

### Fields

**Owner 1** (Required):

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| first_name | text | Yes | Pre-filled from auth, hidden |
| last_name | text | Yes | Pre-filled from auth, hidden |
| email | text | Yes | Pre-filled from auth, hidden |
| title | select | Yes | Static: CEO, CFO, Chairman, Co-owner, Controller, Director, General-Manager, Office-Manager, Owner, Partner, President, Treasurer, Vice-President, Other |
| ssn | text | Yes | Encrypted, masked input |
| phone | tel | Yes | Use intl-tel-input |
| ownership_percentage | number | Yes | 1-100, trigger Owner 2 visibility if <51% |
| dob | date | Yes | YYYY-MM-DD, date picker |
| street_number | text | No | Max 10 chars, Google Maps address |
| street_address | text | No | Max 50 chars, autocomplete |
| city | text | No | Max 60 chars |
| state | text | No | Max 40 chars |
| postal_code | text | No | Max 15 chars |
| country | select | No | API: `/api/partner/countries` |
| filed_bankruptcy | select | No | Options: Yes(1), No(0) |
| bankruptcy_discharged | select | Conditional | Show if filed_bankruptcy=1 |
| bankruptcy_discharged_date | date | Conditional | Show if bankruptcy_discharged=1, past dates only |

**Owner 2** (Conditional - Show if Owner 1 ownership_percentage < 51%):

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| first_name | text | Yes (when shown) | Max 60 chars |
| last_name | text | Yes (when shown) | Max 60 chars |
| email | text | Yes (when shown) | Valid email |
| title | select | Yes (when shown) | Same static list as Owner 1 |
| ssn | text | Yes (when shown) | Encrypted, masked |
| phone | tel | Yes (when shown) | intl-tel-input |
| ownership_percentage | number | Yes (when shown) | 1-100 |
| dob | date | Yes (when shown) | YYYY-MM-DD |
| street_number | text | No | Google Maps address |
| street_address | text | No | Autocomplete |
| city | text | No | |
| state | text | No | |
| postal_code | text | No | |
| country | select | No | API: `/api/partner/countries` |
| filed_bankruptcy | select | No | Options: Yes(1), No(0) |
| bankruptcy_discharged | select | Conditional | Show if filed_bankruptcy=1 |
| bankruptcy_discharged_date | date | Conditional | Show if bankruptcy_discharged=1 |

### Libraries Required
- `intl-tel-input` - Phone formatting
- Date picker library
- `Google Maps Places API` - Home address autocomplete
- `dependent-fields.js` - Owner 2 visibility

### Dependent Fields
- **Owner 2 section**: Show if owner[1][ownership_percentage] < 51
- **bankruptcy_discharged**: Show if filed_bankruptcy=1
- **bankruptcy_discharged_date**: Show if bankruptcy_discharged=1
- **Home address autocomplete**: Google Maps for both owners

### API Integration

**Dropdown Data**:
```
GET /api/partner/countries
```

Header: `Authorization: {user.security_key}`

**Form Submission** (2 calls):

1. Submit ownership data:
```
POST /api/v1/ownership
Headers: X-API-Key: {user.security_key}
Payload:
  uuid: {uuid}
  owner:
    1: { owner 1 fields }
    2: { owner 2 fields or null }
Response: { success: true }
```

2. Submit step marker:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 4
  section: ownership_info
  uuid: {uuid}
Response: { next_step_url, message }
```

---

## STEP 5: Banking Information

### Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| institution_number | text | Conditional | Show if country="CA" (Canada), 3 digits |
| customer_pay_currency | radio | Conditional | Show if country="CA", Options: USD, CAD |
| routing_number | text | Yes | Label varies by country |
| account_number | text | Yes | Alphanumeric |
| account_type | select | No | Static: Checking, Savings |
| currently_processing | select | No | Options: Yes(1), No(0) |
| processor_name | text | Conditional | Show if currently_processing=1, max 300 chars |

### Field Labels by Country

```
Routing Number:
  US: "Routing Number" (9 digits)
  Canada: "Transit Number" (5 digits)
  UK: "Sort Code" (6 digits)
  Australia: "BSB" (6 digits)

Account Number:
  US: "Account Number" (max 17 chars)
  Canada: "Account Number" (max 12 chars)
  Others: "Account Number" (max 34 chars)
```

### Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`). They only appear when trigger value is selected.

- **institution_number**: Show if country="CA" (hidden by default)
- **customer_pay_currency**: Show if country="CA" (hidden by default)
- **processor_name**: Show if currently_processing=1 (hidden by default)

### API Integration

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 5
  section: banking_info
  all Step 5 fields
  uuid: {uuid}
Response: { next_step_url, message }
```

---

## STEP 6: Final Details

### Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| howdidyouhear | select | No | API: `/api/partner/referral-sources` |
| howdidyouhear_other | text | Conditional | Show if howdidyouhear="Other" or "Friend", max 300 chars |
| multiple_merchant_accounts | select | No | Options: Yes(1), No(0) |
| transaction_device | select | No | Static options or from API |
| bad_experience | select | No | Options: Yes(1), No(0) |
| bad_experience_provider | text | Conditional | Show if bad_experience=1, max 300 chars |
| other_interests_capital | checkbox array | No | API: `/api/partner/interest-details`, multiple selection |
| accept_terms | checkbox | Yes | Required to submit, value must be 1 |

### Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`). They only appear when trigger value is selected.

- **howdidyouhear_other**: Show if howdidyouhear="Other" or "Friend" (hidden by default)
- **bad_experience_provider**: Show if bad_experience=1 (hidden by default)

### API Integration

**Dropdown Data**:
```
GET /api/partner/referral-sources
GET /api/partner/interest-details
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 6
  section: interest_details
  all Step 6 fields
  uuid: {uuid}
Response: { redirect: "/dashboard/merchant" }
```

---

## STEP 7: Signature

Display only step - no form submission.

```
- Show success message
- Display completion status
- No next button or form submission
```

---

## Global Implementation

### Authentication Headers (CRITICAL)

**Form Submission (POST)**:
```javascript
headers: {
    'Content-Type': 'application/json',
    'X-API-Key': user.security_key,
    'Accept': 'application/json'
}
```

**Dropdown Data (GET)**:
```javascript
headers: {
    'Authorization': user.security_key,
    'Content-Type': 'application/json'
}
```

### State Management

```javascript
// After Step 1, store UUID
localStorage.setItem('signup_uuid', response.uuid);

// Retrieve for subsequent steps
const uuid = localStorage.getItem('signup_uuid');

// Clear on completion
localStorage.removeItem('signup_uuid');
```

### Form Submission Pattern

1. Collect form data from all visible fields
2. Validate client-side
3. Filter: Remove hidden/disabled fields
4. Add step_count, section, uuid
5. POST to API
6. Navigate to next step or dashboard

### Required Libraries

```html
<!-- All Steps -->
<script src="/assets/js/dependent-fields.js"></script>

<!-- Step 1 -->
<script src="https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/js/intl-tel-input.min.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/css/intlTelInput.css">

<!-- Step 2, 4 -->
<script src="https://maps.google.com/maps/api/js?key={GOOGLE_API_KEY}&libraries=places"></script>

<!-- Step 3 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/ion-rangeslider/2.3.1/ion.rangeSlider.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/ion-rangeslider/2.3.1/css/ion.rangeSlider.min.css">

<!-- Date Picker (Steps 2, 4, 5) -->
<!-- Include your date picker library -->

<!-- Form Validation -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.5/jquery.validate.min.js"></script>
```

### Error Handling

```javascript
// API Response Status Codes
201/200: Success - proceed to next step
400: Bad request - check payload format
401/403: Auth error - check headers
422: Validation error - display field errors
500: Server error - retry or report
```

---

## Field Summary

**Total Fields**: 78+ (added Revenue Model + Google Maps autocomplete for both addresses)
**Required Fields**: 48+  
**Conditional Fields**: 25+ (including 2-level dependent hierarchy)
**Google Maps Autocomplete Fields**: 2 (legal address + physical address)  
**API Dropdowns**: 6  
**Static Dropdowns**: 8  
**Address Autocompletes**: 3 (legal, physical, home addresses)  
**Phone Fields**: 4 (intl-tel-input)  
**Date Fields**: 4 (date pickers)  
**Validations**: 30+ rules  

---

**Production Ready** ✅  
Last Updated: 2026-07-19
