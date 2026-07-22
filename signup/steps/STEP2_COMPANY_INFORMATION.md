---
name: signup-step-2
description: Step 2 Company Information - Business details, legal address, revenue model, and Google Maps autocomplete
---

# STEP 2: Company Information

Second step of the 7-step signup form. Collects company business details, addresses, and revenue model information.

---

## Fields

**Company Details** (11 fields):

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| legal_name | text | Yes | Max 60 chars |
| name | text | Yes | DBA name, max 60 chars |
| **country** | **select** | **Yes** | **API: `/api/partner/countries`** - Controls business_state & business_register_number visibility |
| business_state | select | Conditional | Show ONLY if country="1" (United States), API: `/api/partner/states` |
| industry_type | select | Yes | API: `/api/partner/industry-types` |
| annual_sales | text | No | Numeric, variant-specific |
| customer_service_telephone_number | tel | Yes | Use intl-tel-input |
| business_location | select | Yes | Static (slug): Home-Based, Co-Working, Corporate-Office, Storefront, Others |
| business_formed | date | Yes | YYYY-MM-DD, use date picker |
| business_organized | select | Yes | Static (slug): Corporation, LLC, Partnership, Government, Sole-Proprietorship, Non-Profit, Other |
| federal_tax_id | text | Yes | Encrypted, max 20 chars |
| business_register_number | text | Conditional | Show ONLY if country≠"1" (not US), max 20 chars |

**Revenue Model** (3 fields - Checkbox Array with Dependent Hierarchy):

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

**Legal Address** (7 fields - All REQUIRED):

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
- **Initially shown**: If no address data exists
- **On selection**: User searches and selects address from Google autocomplete
- **Auto-population**: All 6 address fields populate from Google's address_components
- **After selection**: Autocomplete hides, address fields display and become required

**Component Mapping** (Google address_components → Form fields):
- `street_number` (short_name) → field id=`street_number`
- `route` (long_name) → field id=`route`
- `locality` (long_name) → field id=`locality`
- `administrative_area_level_1` (short_name) → field id=`administrative_area_level_1`
- `postal_code` (short_name) → field id=`postal_code`
- `country` (long_name) → field id=`country` (selectpicker with refresh)

**Physical/Mailing Address** (8 fields - Conditional on radio selection):

| Field | Type | Name Attribute | Notes |
|-------|------|---|---|
| is_physical_address_same_as_legal_address | radio | is_physical_address_same_as_legal_address | Options: yes=different, no=same. Default: no |
| physical_address_autocomplete | text | physical_address_autocomplete | Google Maps autocomplete - shown only if radio="yes" AND no address data exists |
| physical_address_street_number | text | physical_address_street_number | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_street_address | text | physical_address_street_address | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_city | text | physical_address_city | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_state | text | physical_address_state | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_postal_code | text | physical_address_postal_code | REQUIRED (when visible) - Auto-populated from autocomplete |
| physical_address_country | select | physical_address_country | REQUIRED (when visible) - Auto-populated from autocomplete |

---

## Libraries Required

- `Google Maps Places API` - Address autocomplete (legal + physical)
- Date picker library
- `intl-tel-input` - Phone formatting

---

## Google Maps Implementation

**Script Include**:
```html
<script src="https://maps.google.com/maps/api/js?key={GOOGLE_API_KEY}&libraries=places"></script>
```

**Complete JavaScript Implementation**:
```javascript
var componentForm = {
    street_number: 'short_name',
    route: 'long_name',
    locality: 'long_name',
    administrative_area_level_1: 'short_name',
    country: 'long_name',
    postal_code: 'short_name'
};

google.maps.event.addDomListener(window, 'load', initialize);

function initialize() {
    // Legal Address Autocomplete (id="autocomplete")
    var input = document.getElementById('autocomplete');
    var autocomplete = new google.maps.places.Autocomplete(input);
    autocomplete.addListener('place_changed', function () {
        var place = autocomplete.getPlace();

        // Clear all fields first
        for (var component in componentForm) {
            document.getElementById(component).value = '';
            $(document.getElementById(component)).trigger('change');
            document.getElementById(component).disabled = false;
        }

        // Fill in address components
        for (var i = 0; i < place.address_components.length; i++) {
            var component = place.address_components[i];
            var addressType = null;
            
            for (var j = 0; j < component.types.length; j++) {
                if (componentForm[component.types[j]]) {
                    addressType = component.types[j];
                    break;
                }
            }
            
            if (addressType) {
                var val = component[componentForm[addressType]];
                if (addressType === 'country') {
                    // Special handling for selectpicker
                    $('#country').selectpicker('val', val);
                    $('#country').selectpicker('refresh');
                    $(document.getElementById('country')).trigger('change');
                    if ($('#country').valid()) {
                        $('.company-address-country .custom-select').removeClass('is-invalid');
                    }
                } else {
                    document.getElementById(addressType).value = val;
                    $(document.getElementById(addressType)).trigger('change');
                    $(document.getElementById(addressType)).valid();
                }
            }
        }
        
        // Show individual address fields, hide autocomplete
        $("#street_area").removeClass("d-none");
        $("#city_area").removeClass("d-none");
        $("#state_area").removeClass("d-none");
        $("#country_area").removeClass("d-none");
        $("#postal_area").removeClass("d-none");
        $("#address_area").addClass("d-none");
        $("#street_area input").attr("required", "required");
        $("#city_area input").attr("required", "required");
        $("#state_area input").attr("required", "required");
        $("#postal_area input").attr("required", "required");
        $("#address_area input").removeAttr("required");
    });
}

// Call this function when physical address radio is changed to "yes" (different address)
function initializePhysicalAddressAutocomplete() {
    var physicalInput = document.getElementById('physical_address_autocomplete');
    if (!physicalInput) return;
    
    var physicalAutocomplete = new google.maps.places.Autocomplete(physicalInput);
    var physicalComponentForm = {
        street_number: 'short_name',
        route: 'long_name',
        locality: 'long_name',
        administrative_area_level_1: 'short_name',
        country: 'long_name',
        postal_code: 'short_name'
    };

    physicalAutocomplete.addListener('place_changed', function () {
        var place = physicalAutocomplete.getPlace();
        
        // Clear all physical address fields
        var physicalFields = ['physical_address_street_number', 'physical_address_street_address', 'physical_address_city', 'physical_address_state', 'physical_address_postal_code'];
        physicalFields.forEach(function(fieldId) {
            var field = document.getElementById(fieldId);
            if (field) {
                field.value = '';
                $(field).trigger('change');
            }
        });

        // Fill in physical address components
        for (var i = 0; i < place.address_components.length; i++) {
            var component = place.address_components[i];
            var addressType = null;
            
            for (var j = 0; j < component.types.length; j++) {
                if (physicalComponentForm[component.types[j]]) {
                    addressType = component.types[j];
                    break;
                }
            }
            
            if (addressType) {
                var val = component[physicalComponentForm[addressType]];
                var fieldId = '';
                
                switch(addressType) {
                    case 'street_number':
                        fieldId = 'physical_address_street_number';
                        break;
                    case 'route':
                        fieldId = 'physical_address_street_address';
                        break;
                    case 'locality':
                        fieldId = 'physical_address_city';
                        break;
                    case 'administrative_area_level_1':
                        fieldId = 'physical_address_state';
                        break;
                    case 'postal_code':
                        fieldId = 'physical_address_postal_code';
                        break;
                    case 'country':
                        fieldId = 'physical_address_country';
                        break;
                }
                
                if (fieldId) {
                    var fieldElement = document.getElementById(fieldId);
                    if (fieldElement) {
                        if (addressType === 'country') {
                            var $countrySelect = $('#' + fieldId);
                            $countrySelect.selectpicker('val', val);
                            $countrySelect.selectpicker('refresh');
                            $countrySelect.removeClass('is-invalid error');
                            $('.physical-address-country .custom-select').removeClass('is-invalid error');
                            $countrySelect.trigger('change');
                        } else {
                            fieldElement.value = val;
                            $(fieldElement).trigger('change');
                            $(fieldElement).valid();
                        }
                    }
                }
            }
        }

        // Show individual fields, hide autocomplete
        $("#physical_address_street_area").removeClass("d-none");
        $("#physical_address_street_area input").attr("required", "required");
        $("#physical_address_city_area input").attr("required", "required");
        $("#physical_address_state_area input").attr("required", "required");
        $("#physical_address_postal_code_area input").attr("required", "required");
        $("#physical_address_area").addClass("d-none");
        $("#physical_address_area input").removeAttr("required");
    });
}

// Initialize physical address autocomplete when radio changes to "yes"
$(document).ready(function() {
    $('#is_physical_address_same_as_legal_address').change(function() {
        if ($(this).val() === 'yes') {
            initializePhysicalAddressAutocomplete();
        }
    });
});
```

**Input Field IDs** (must match HTML):
- Legal Address: `id="autocomplete"`
- Physical Address: `id="physical_address_autocomplete"`

**Output Field IDs** (where data gets populated):

Legal Address:
- `id="street_number"` ← street_number component
- `id="route"` ← route component  
- `id="locality"` ← locality component
- `id="administrative_area_level_1"` ← state component
- `id="postal_code"` ← postal code component
- `id="country"` ← country selectpicker

Physical Address:
- `id="physical_address_street_number"` ← street_number component
- `id="physical_address_street_address"` ← route component
- `id="physical_address_city"` ← locality component
- `id="physical_address_state"` ← administrative_area_level_1 component
- `id="physical_address_postal_code"` ← postal code component
- `id="physical_address_country"` ← country selectpicker

**Container IDs** (for show/hide logic):
- Legal address container: `id="address_area"` (hidden after autocomplete)
- Legal fields container: `id="street_area"`, `id="city_area"`, `id="state_area"`, `id="country_area"`, `id="postal_area"` (shown after autocomplete)
- Physical address container: `id="physical_address_area"` (hidden after autocomplete)
- Physical fields container: `id="physical_address_street_area"` (shown after autocomplete)

---

## Dependent Fields & Conditionals

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

### Country-Based Conditionals (Level 1)

**country Field** (Dropdown with Selectpicker):
- **ID**: `id="business_formed_in"`
- **Name**: `name="country"`
- **API**: `GET /api/partner/countries` with header `Authorization: {security_key}`
- **Required**: Yes
- **Searchable**: Yes (selectpicker with live-search)
- **Values**: { "name": "United States", "id": "1" }, { "name": "Canada", "id": "2" }, etc.
- **CRITICAL**: Use `id` value for conditionals, not `name`

**business_state Field** (Show ONLY if country="1"):
- **API**: `GET /api/partner/states` with header `Authorization: {security_key}`
- **Conditional Logic**:
  ```javascript
  IF country = "1" (United States):
    SHOW business_state dropdown
    MAKE business_state REQUIRED
  ELSE:
    HIDE business_state dropdown
    CLEAR value
  ```

**business_register_number Field** (Show ONLY if country≠"1"):
- **Conditional Logic**:
  ```javascript
  IF country ≠ "1" (not United States):
    SHOW business_register_number field
    MAKE business_register_number REQUIRED
    maxlength = 20 (or 11 if Canada)
  ELSE:
    HIDE business_register_number field
    CLEAR value
  ```

**federal_tax_id Label** (Changes based on country):
- **If country="1" (US)**: Label = "Federal Tax ID"
- **If country≠"1"**: Label = "Federal Tax ID (or Corporation Tax Number equivalent)"

### Address Conditionals (Level 1)

- **Physical address section**: Show if is_physical_address_same_as_legal_address="yes"

### Revenue Model Conditionals (Level 2)

- **subscription_frequency**: Show if marketingModel includes "Recurring/Subscription" (value 2)
- **subscription_frequency_other**: Show if subscription_frequency="Other" (value 3)

---

## Form Submission & Redirect

**On Success (HTTP 200/201)**:
```javascript
// Response example:
{
  "status": true,
  "message": "Company information saved successfully",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "step_count": 2
}

// UUID already stored from Step 1, retrieve and use
const uuid = localStorage.getItem('signup_uuid');

// Redirect to Step 3
window.location.href = `/signup/step/3/${uuid}`;
```

**On Validation Error (HTTP 422)**:
```javascript
// Response example:
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "company_name": ["Company name is required"],
    "legal_address_street_number": ["Street number is required"]
  }
}

// Display field errors
// DO NOT change URL - user stays on Step 2
for (const [field, messages] of Object.entries(response.errors)) {
  displayErrorForField(field, messages[0]);
}
```

---

## API Integration

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

## Field Summary

**Total Fields**: 28  
**Required Fields**: 22  
**Conditional Fields**: 6  
**Google Maps Autocomplete**: 2 (legal + physical)

---

**Production Ready** ✅
