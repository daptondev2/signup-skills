# Step 4: Owner Information - Complete Reference

Complete specification for all Owner Information fields, including Primary Contact, Owner 1, and conditional Owner 2.

---

## Field Count Summary

**Section Breakdown**:
- Primary Contact Details: 2 fields
- Owner 1 Details: 21 fields
- Owner 2 Details: 22 fields (conditional)
- **Total**: 45 fields

**By Category**:
- Basic Info: 8 per owner (name, email, title, ssn, phone, ownership %, DOB)
- Address: 7 per owner (autocomplete + 6 address fields)
- Financial History: 3 per owner (bankruptcy fields)
- License Details: 3 per owner (license, state, expiration)
- Primary Contact: 2 (radio + conditional job title)

---

## Primary Contact Details

**Question**: "Are you the business owner?"

### Fields

| Field | Type | HTML Name | Value | Notes |
|-------|------|-----------|-------|-------|
| primary_contact | radio | primary_contact | 1 = Yes (default), 0 = No | Pre-selected "Yes" |
| primary_contact_job_title | text | primary_contact_job_title | User input | **Conditional**: Show only if primary_contact=0 |

### Conditional Logic

```javascript
IF primary_contact = 0 (No - Not the business owner):
  SHOW primary_contact_job_title text field
  MAKE it optional (not required)
  
  IN OWNER 1 SECTION:
    SHOW first_name, last_name, email as visible TEXT INPUT fields
    Make editable by user
    users_uuid = null

ELSE (primary_contact = 1 - Is the business owner):
  HIDE primary_contact_job_title
  CLEAR the field
  
  IN OWNER 1 SECTION:
    FETCH from API: POST /v1/user/{deal_uuid}
    DISPLAY first_name, last_name, email as HIDDEN inputs (read-only)
    STORE users_uuid from API response
```

### JavaScript Implementation

**Field IDs** (must match HTML):
- `id="primary_contact"` - Radio button for primary contact (values: 1 or 0)
- `id="primary_contact_job_title"` - Text field for job title
- `id="owner_1_first_name"` - Owner 1 first name field
- `id="owner_1_last_name"` - Owner 1 last name field
- `id="owner_1_email"` - Owner 1 email field
- `id="owner_1_users_uuid"` - Hidden field to store user UUID from API

**Container IDs** (for show/hide):
- `id="primary_contact_job_title_container"` - Wrapper for job title field (hidden by default)
- `id="owner_1_name_email_container"` - Wrapper for first/last name & email fields

**Complete Implementation**:
```javascript
$(document).ready(function() {
    const dealUuid = localStorage.getItem('signup_uuid');
    const apiKey = $('#api_key').val(); // or however you store the security key
    
    // Initialize based on current primary_contact value
    handlePrimaryContactChange();
    
    // Listen for radio changes
    $('input[name="primary_contact"]').change(function() {
        handlePrimaryContactChange();
    });
    
    function handlePrimaryContactChange() {
        const primaryContact = $('input[name="primary_contact"]:checked').val();
        const jobTitleContainer = $('#primary_contact_job_title_container');
        const firstNameContainer = $('#owner_1_first_name_container');
        const lastNameContainer = $('#owner_1_last_name_container');
        const emailContainer = $('#owner_1_email_container');
        const firstNameField = $('#owner_1_first_name');
        const lastNameField = $('#owner_1_last_name');
        const emailField = $('#owner_1_email');
        const usersUuidField = $('#owner_1_users_uuid');
        
        if (primaryContact === '1' || primaryContact === 1) {
            // Primary contact = YES (is the business owner)
            // Hide job title field
            jobTitleContainer.hide();
            $('#primary_contact_job_title').val('');
            $('#primary_contact_job_title').removeAttr('required');
            
            // Hide name/email containers (but fields remain type="text")
            firstNameContainer.hide();
            lastNameContainer.hide();
            emailContainer.hide();
            
            // Fetch user data from API
            fetchUserData(dealUuid, firstNameField, lastNameField, emailField, usersUuidField);
            
        } else {
            // Primary contact = NO (not the business owner)
            // Show job title field
            jobTitleContainer.show();
            $('#primary_contact_job_title').attr('required', 'required');
            
            // Show name/email field containers (user can edit them)
            firstNameContainer.show();
            lastNameContainer.show();
            emailContainer.show();
            
            // Remove readonly/disabled from fields
            firstNameField.removeAttr('readonly').removeAttr('disabled');
            lastNameField.removeAttr('readonly').removeAttr('disabled');
            emailField.removeAttr('readonly').removeAttr('disabled');
            
            // Clear API data
            firstNameField.val('');
            lastNameField.val('');
            emailField.val('');
            usersUuidField.val('');
        }
    }
    
    function fetchUserData(uuid, firstNameField, lastNameField, emailField, usersUuidField) {
        if (!uuid) {
            console.error('No signup UUID found');
            return;
        }
        
        $.ajax({
            url: `/v1/user/${uuid}`,
            type: 'POST',
            headers: {
                'X-API-Key': apiKey,
                'Content-Type': 'application/json'
            },
            success: function(response) {
                if (response.status && response.data) {
                    // Populate fields from API response
                    firstNameField.val(response.data.first_name || '');
                    lastNameField.val(response.data.last_name || '');
                    emailField.val(response.data.email || '');
                    usersUuidField.val(response.data.uuid || '');
                    
                    // Make fields readonly and disabled so user can't edit
                    // BUT keep them visible (not hidden) so they're submitted
                    firstNameField.attr('readonly', 'readonly').attr('disabled', 'disabled');
                    lastNameField.attr('readonly', 'readonly').attr('disabled', 'disabled');
                    emailField.attr('readonly', 'readonly').attr('disabled', 'disabled');
                    
                    // Trigger validation
                    firstNameField.trigger('change');
                    lastNameField.trigger('change');
                    emailField.trigger('change');
                    
                    console.log('User data fetched and populated:', {
                        first_name: response.data.first_name,
                        last_name: response.data.last_name,
                        email: response.data.email,
                        users_uuid: response.data.uuid
                    });
                } else {
                    console.error('Invalid API response:', response);
                }
            },
            error: function(xhr) {
                let errorMsg = 'Failed to fetch user information';
                if (xhr.status === 404) {
                    errorMsg = 'User information not found';
                } else if (xhr.status === 500) {
                    errorMsg = 'Server error while fetching user information';
                }
                console.error(errorMsg, xhr);
                
                // On error: show fields as editable text inputs
                $('#owner_1_first_name_container').show();
                $('#owner_1_last_name_container').show();
                $('#owner_1_email_container').show();
                
                // Remove readonly/disabled so user can manually enter
                firstNameField.removeAttr('readonly').removeAttr('disabled').val('');
                lastNameField.removeAttr('readonly').removeAttr('disabled').val('');
                emailField.removeAttr('readonly').removeAttr('disabled').val('');
                
                // Fallback: show primary contact job title
                $('#primary_contact_job_title_container').show();
            }
        });
    }
});
```

**HTML Structure Required**:

⚠️ **CRITICAL**: These fields MUST use `type="text"` or `type="email"` (NOT `type="hidden"`) so they're included in form submission.

```html
<!-- Primary Contact Section -->
<div>
    <label>Are you the business owner?</label>
    <input type="radio" name="primary_contact" id="primary_contact" value="1" checked>
    <label>Yes (Primary Contact)</label>
    <input type="radio" name="primary_contact" id="primary_contact_no" value="0">
    <label>No</label>
</div>

<!-- Job Title (hidden by default, shown only if primary_contact=0) -->
<div id="primary_contact_job_title_container" style="display: none;">
    <label>Job Title</label>
    <input type="text" id="primary_contact_job_title" name="primary_contact_job_title" maxlength="255">
</div>

<!-- Owner 1 Name & Email Fields 
     - MUST be type="text" or type="email" (NOT type="hidden")
     - Hidden VISUALLY (display: none) when primary_contact=1
     - Made READONLY + DISABLED when filled from API
     - This ensures they're submitted in form data
-->
<div id="owner_1_name_email_container">
    <div id="owner_1_first_name_container" style="display: none;">
        <label>First Name *</label>
        <input type="text" id="owner_1_first_name" name="owner[1][first_name]" required>
    </div>
    <div id="owner_1_last_name_container" style="display: none;">
        <label>Last Name *</label>
        <input type="text" id="owner_1_last_name" name="owner[1][last_name]" required>
    </div>
    <div id="owner_1_email_container" style="display: none;">
        <label>Email *</label>
        <input type="email" id="owner_1_email" name="owner[1][email]" required>
    </div>
    <!-- Hidden field to store user UUID from API -->
    <input type="hidden" id="owner_1_users_uuid" name="owner[1][users_uuid]" value="">
</div>
```

**API Response Mapping**:
- `response.data.uuid` → `owner[1][users_uuid]` (stored in hidden field)
- `response.data.first_name` → `owner[1][first_name]` (hidden text input if primary_contact=1)
- `response.data.last_name` → `owner[1][last_name]` (hidden text input if primary_contact=1)
- `response.data.email` → `owner[1][email]` (hidden text input if primary_contact=1)

---

## Owner 1 Details

**Conditional Visibility Based on primary_contact**

### Name & Email Fields (Conditional on primary_contact)

**IF primary_contact = 1 (Yes - Business Owner)**:
- Fetch from API: `POST /v1/user/{deal_uuid}`
- Response contains: uuid (users_uuid), first_name, last_name, email, phone
- Display first_name, last_name, email as HIDDEN inputs (read-only, pre-filled)
- Auto-populate from API response.data
- Store users_uuid for later submission

**IF primary_contact = 0 (No - Not Business Owner)**:
- Display first_name, last_name, email as VISIBLE text input fields
- User enters manually
- No API call made
- users_uuid = empty/null

| Field | HTML Name | Type | When Shown | Source |
|-------|-----------|------|-----------|--------|
| first_name | owner[1]['first_name'] | text/hidden | primary_contact=1: hidden, primary_contact=0: visible input | API response.data.first_name if yes, user input if no |
| last_name | owner[1]['last_name'] | text/hidden | primary_contact=1: hidden, primary_contact=0: visible input | API response.data.last_name if yes, user input if no |
| email | owner[1]['email'] | text/hidden | primary_contact=1: hidden, primary_contact=0: visible input | API response.data.email if yes, user input if no |
| users_uuid | owner[1]['users_uuid'] | hidden | Always hidden | API response.data.uuid if primary_contact=yes, null if no |
| phone | owner[1]['phone'] | tel/hidden | Can pre-fill from API if yes | API response.data.phone if yes, user input if no |

### Basic Information

| Field | Type | HTML Name | Validation | Notes |
|-------|------|-----------|------------|-------|
| title | select | owner[1]['title'] | Required | Options: 1-14 (CEO, CFO, Chairman, Co-owner, Controller, Director, General-Manager, Office-Manager, Owner, Partner, President, Treasurer, Vice-President, Other) |
| ssn | text | owner[1]['ssn'] | Required | Masked input (ph-mask-input class), uses inputmode="numeric" |
| phone | tel | owner[1]['phone'] | Required | intl-tel-input library for formatting |
| ownership_percentage | number | owner[1]['ownership_percentage'] | Required | Range: 1-100, **triggers Owner 2 visibility if < 51%** |
| dob | date | owner[1]['dob'] | Required | Format: YYYY-MM-DD, date picker |

### Home Address (with Google Maps Autocomplete)

**Autocomplete Field**:
| Field | Type | HTML ID | Visibility | Notes |
|-------|------|---------|------------|-------|
| address_search | text | autocomplete2 | Shown initially if no address data | Google Maps Places autocomplete, placeholder: "Enter in Your Residence Location" |

**After Autocomplete Selection** - These 6 fields auto-populate and become required:

| Field | Type | HTML Name | Component | Notes |
|-------|------|-----------|-----------|-------|
| street_number | text | owner_1_street_number | street_number (short_name) | Required when visible |
| street_address | text | owner_1_street_address | route (long_name) | Required when visible |
| city | text | owner_1_city | locality (long_name) | Required when visible |
| state | text | owner_1_state | administrative_area_level_1 (short_name) | Required when visible |
| postal_code | text | owner_1_postal_code | postal_code (short_name) | Required when visible |
| country | select | owner[1]['country'] | country (long_name) | Required when visible, uses selectpicker |

**Autocomplete Behavior**:
1. Shown initially IF no address data exists
2. User searches and selects from dropdown
3. place_changed event fires → auto-populates all 6 fields
4. Autocomplete container hides (#owner_1_address_area gets d-none class)
5. Address fields container shows (#owner_1_street_area)
6. All fields become required

### Financial History

| Field | Type | HTML Name | Value | Notes |
|-------|------|-----------|-------|-------|
| bankruptcy_filed | radio | owner[1]['bankruptcy_filed'] | 1 = Yes, 0 = No | Required, shows conditionals if Yes |
| bankruptcy_discharged | radio | owner[1]['bankruptcy_discharged'] | 1 = Yes, 0 = No | **Conditional**: Show if bankruptcy_filed=1 |
| bankruptcy_discharged_date | date | owner[1]['bankruptcy_discharged_date'] | YYYY-MM-DD | **Conditional**: Show if bankruptcy_discharged=1 |

**Conditional Logic**:
```javascript
IF bankruptcy_filed = 1:
  SHOW bankruptcy_discharged radio
  MAKE it required
  
  IF bankruptcy_discharged = 1:
    SHOW bankruptcy_discharged_date
    MAKE it required
ELSE:
  HIDE bankruptcy_discharged
  HIDE bankruptcy_discharged_date
  CLEAR both fields
```

### Driver's License Details

| Field | Type | HTML Name | Validation | Notes |
|-------|------|-----------|------------|-------|
| license | text | owner[1]['license'] | Required | Driver's License / Passport Number, 5-25 chars, min/max validation |
| driver_license_state | select | owner[1]['driver_license_state'] | Required (when country=US) | **CONDITIONAL**: Show/require ONLY when country="United States". US States dropdown (2-letter codes). Wrapper: `.driver_license_state1` |
| driver_license_expiration_date | date | owner[1]['driver_license_expiration_date'] | Required (when country=US) | **CONDITIONAL**: Show/require ONLY when country="United States". Format: YYYY-MM-DD. Wrapper: `.driver_license_state1` |

---

## Owner 2 Details (Conditional)

**Visibility**: Show ONLY if owner[1][ownership_percentage] < 51%

**Label**: "Owner #2's Details" (or dynamically shows "FirstName LastName's Details" if data exists)

### Basic Information

| Field | Type | HTML Name | Validation | Notes |
|-------|------|-----------|------------|-------|
| first_name | text | owner[2]['first_name'] | Required (when shown) | Max 60 chars |
| last_name | text | owner[2]['last_name'] | Required (when shown) | Max 60 chars |
| email | text | owner[2]['email'] | Required (when shown) | Valid email format |
| user_id | hidden | owner[2]['user_id'] | - | Hidden field for internal reference |
| title | select | owner[2]['title'] | Required (when shown) | Same 14-option list as Owner 1 |
| ssn | text | owner[2]['ssn'] | Required (when shown) | Masked input (ph-mask-input class) |
| phone | tel | owner[2]['phone'] | Required (when shown) | intl-tel-input library |
| ownership_percentage | number | owner[2]['ownership_percentage'] | Required (when shown) | Range: 1-100 |
| dob | date | owner[2]['dob'] | Required (when shown) | Format: YYYY-MM-DD |

### Home Address (with Google Maps Autocomplete)

**Autocomplete Field**:
| Field | Type | HTML ID | Visibility | Notes |
|-------|------|---------|------------|-------|
| address_search | text | autocomplete3 | Shown initially if Owner 2 visible AND no address data | Google Maps Places autocomplete, placeholder: "Enter Residence Location" |

**After Autocomplete Selection** - These 6 fields auto-populate and become required:

| Field | Type | HTML Name | Component | Notes |
|-------|------|-----------|-----------|-------|
| street_number | text | owner_2_street_number | street_number (short_name) | Required when visible |
| street_address | text | owner_2_street_address | route (long_name) | Required when visible |
| city | text | owner_2_city | locality (long_name) | Required when visible |
| state | text | owner_2_state | administrative_area_level_1 (short_name) | Required when visible |
| postal_code | text | owner_2_postal_code | postal_code (short_name) | Required when visible |
| country | select | owner[2]['country'] | country (long_name) | Required when visible, uses selectpicker |

**Autocomplete Behavior**: Same as Owner 1 (shown initially, hides after selection, auto-populates fields)

### Financial History

| Field | Type | HTML Name | Value | Notes |
|-------|------|-----------|-------|-------|
| bankruptcy_filed | radio | owner[2]['bankruptcy_filed'] | 1 = Yes, 0 = No | Optional, shows conditionals if Yes |
| bankruptcy_discharged | radio | owner[2]['bankruptcy_discharged'] | 1 = Yes, 0 = No | **Conditional**: Show if bankruptcy_filed=1 |
| bankruptcy_discharged_date | date | owner[2]['bankruptcy_discharged_date'] | YYYY-MM-DD | **Conditional**: Show if bankruptcy_discharged=1 |

**Conditional Logic**: Same as Owner 1

### Driver's License Details

| Field | Type | HTML Name | Validation | Notes |
|-------|------|-----------|------------|-------|
| license | text | owner[2]['license'] | Required | Driver's License / Passport Number, 5-25 chars |
| driver_license_state | select | owner[2]['driver_license_state'] | Required (when country=US) | **CONDITIONAL**: Show/require ONLY when country="United States". US States dropdown (2-letter codes). Wrapper: `.driver_license_state2` |
| driver_license_expiration_date | date | owner[2]['driver_license_expiration_date'] | Required (when country=US) | **CONDITIONAL**: Show/require ONLY when country="United States". Format: YYYY-MM-DD. Wrapper: `.driver_license_state2` |

---

## Multi-Level Conditional Logic

### Owner 2 Visibility

```
IF owner[1][ownership_percentage] < 51:
  SHOW entire Owner 2 section (#owner-section-section)
  SET all Owner 2 fields to visible
  MAKE required fields required
ELSE:
  HIDE entire Owner 2 section
  CLEAR all Owner 2 field values
  SKIP Owner 2 in form submission
```

### Bankruptcy Cascade (Both Owners)

```
IF bankruptcy_filed = 1:
  SHOW bankruptcy_discharged radio (make required)
  
  IF bankruptcy_discharged = 1:
    SHOW bankruptcy_discharged_date (make required)
  ELSE:
    HIDE bankruptcy_discharged_date
    CLEAR value
ELSE (bankruptcy_filed = 0):
  HIDE bankruptcy_discharged
  HIDE bankruptcy_discharged_date
  CLEAR both values
```

### Driver's License State & Expiration Visibility (Both Owners)

**Trigger**: Country field selection

```
IF country = "United States" (country option value = 1):
  SHOW driver_license_state field (make required)
  SHOW driver_license_expiration_date field (make required)
  DISPLAY error message: "Required because your Country is the United States."
  
ELSE (country ≠ "United States"):
  HIDE driver_license_state field
  HIDE driver_license_expiration_date field
  CLEAR both field values
  REMOVE required validation
```

**Implementation Requirements**:

Owner 1:
- Trigger field ID: `#owner_country1` (country dropdown)
- Target fields IDs: `#owner_1_driver_license_state_input`, `#firstExpirationDate`
- Container wrapper class: `.driver_license_state1` (for hide/show)
- Trigger value: `1` (the option value for United States in the countries dropdown)

Owner 2:
- Trigger field ID: `#owner_country2` (country dropdown)
- Target field IDs: `#driver_license_state`, `#secondExpirationDate`
- Container wrapper class: `.driver_license_state2` (for hide/show)
- Trigger value: `1` (same as Owner 1)

**JavaScript Implementation** (using DependentFields library):

```javascript
window.dependentFieldsInstance = new DependentFields({
    rules: [
        // Owner 1 - Driver License State
        {
            trigger: '#owner_country1',
            target: '#owner_1_driver_license_state_input',
            targetWrapper: '.driver_license_state1',
            value: '1',  // United States
            message: 'Required because your Country is the United States.'
        },
        // Owner 1 - Driver License Expiration
        {
            trigger: '#owner_country1',
            target: '#firstExpirationDate',
            targetWrapper: '.driver_license_state1',
            value: '1',  // United States
            message: 'Required because your Country is the United States.'
        },
        // Owner 2 - Driver License State
        {
            trigger: '#owner_country2',
            target: '#driver_license_state',
            targetWrapper: '.driver_license_state2',
            value: '1',  // United States
            message: 'Required because your Country is the United States.'
        },
        // Owner 2 - Driver License Expiration
        {
            trigger: '#owner_country2',
            target: '#secondExpirationDate',
            targetWrapper: '.driver_license_state2',
            value: '1',  // United States
            message: 'Required because your Country is the United States.'
        }
    ]
});
```

**HTML Structure Requirements**:

Owner 1 fields must be wrapped in a container with class `.driver_license_state1`:
```html
<div class="driver_license_state1" style="display: none;">
    <div class="form-group">
        <label>Driver's License State</label>
        <select id="owner_1_driver_license_state_input" 
                name="owner[1]['driver_license_state']"></select>
    </div>
</div>

<div class="driver_license_state1" style="display: none;">
    <div class="form-group">
        <label>Driver's License Expiration</label>
        <input type="text" id="firstExpirationDate" 
               name="owner[1]['driver_license_expiration_date']">
    </div>
</div>
```

Owner 2 fields must be wrapped in a container with class `.driver_license_state2`:
```html
<div class="driver_license_state2" style="display: none;">
    <div class="form-group">
        <label>Driver's License State</label>
        <select id="driver_license_state" 
                name="owner[2]['driver_license_state']"></select>
    </div>
</div>

<div class="driver_license_state2" style="display: none;">
    <div class="form-group">
        <label>Driver's License Expiration</label>
        <input type="text" id="secondExpirationDate" 
               name="owner[2]['driver_license_expiration_date']">
    </div>
</div>
```

**Trigger Handler** (when country changes via autocomplete or manual selection):
```javascript
// After setting country value (either from autocomplete or user selection)
if (window.dependentFieldsInstance) {
    window.dependentFieldsInstance.triggerFieldsForTrigger('#owner_country1');
    // Or for Owner 2:
    window.dependentFieldsInstance.triggerFieldsForTrigger('#owner_country2');
}
```

---

## API Endpoints for Step 4

### User Information Fetch (when primary_contact="yes")

**Endpoint**: `POST /v1/user/{deal_uuid}`

**Headers**: 
```
X-API-Key: {user.security_key}
Content-Type: application/json
```

**Path Parameter**:
- `deal_uuid` - The signup UUID tracking this merchant application

**Response**:
```json
{
  "status": true,
  "message": "User fetched successfully",
  "data": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone": "+1-555-1234"
  }
}
```

**Field Mapping**:
- `data.uuid` → `owner[1]['users_uuid']` (User's UUID)
- `data.first_name` → `owner[1]['first_name']`
- `data.last_name` → `owner[1]['last_name']`
- `data.email` → `owner[1]['email']`
- `data.phone` → `owner[1]['phone']` (optional, can pre-fill)

**Usage**:
1. When page loads, check if primary_contact="yes"
2. If yes, make this API call with deal_uuid
3. Extract fields from response.data and map to owner[1]
4. Store users_uuid (user's UUID from API)
5. Display first_name, last_name, email as hidden, read-only inputs
6. Users_uuid stored for later use in form submission

**Error Handling**:
```
404: Application not found (invalid UUID)
404: User not found (application exists but no user)
500: Server error
```

---

## Google Maps Autocomplete Integration

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
    // Owner 1 Autocomplete (id="autocomplete2")
    var input2 = document.getElementById('autocomplete2');
    var autocomplete2 = new google.maps.places.Autocomplete(input2);
    autocomplete2.addListener('place_changed', function () {
        var place2 = autocomplete2.getPlace();

        // Clear all fields first
        for (var component in componentForm) {
            if (component === "country") {
                $('#owner_country1').selectpicker('val', 'Please Select');
                $('#owner_country1').selectpicker('refresh');
                $('#owner_country1').trigger('change');
            } else {
                document.getElementById('owner_1_' + component).value = '';
                $(document.getElementById('owner_1_' + component)).trigger('change');
                document.getElementById('owner_1_' + component).disabled = false;
            }
        }

        // Fill in address components
        for (var i = 0; i < place2.address_components.length; i++) {
            var component = place2.address_components[i];
            var addressType2 = null;
            
            for (var j = 0; j < component.types.length; j++) {
                if (componentForm[component.types[j]]) {
                    addressType2 = component.types[j];
                    break;
                }
            }
            
            if (addressType2) {
                var val2 = component[componentForm[addressType2]];
                if (addressType2 === 'country') {
                    // Match country name to option value and set via selectpicker
                    if ($('#owner_country1').length) {
                        $('#owner_country1 option').each(function () {
                            if ($(this).text() === val2) {
                                $('#owner_country1').selectpicker('val', $(this).val());
                                $('#owner_country1').selectpicker('refresh');
                                $('#owner_country1').trigger('change');
                                if ($('#owner_country1').valid()) {
                                    $('#owner_country1').parent('.custom-select').removeClass('is-invalid');
                                }
                                if (window.dependentFieldsInstance) {
                                    window.dependentFieldsInstance.triggerFieldsForTrigger('#owner_country1');
                                }
                                return false;
                            }
                        });
                        $('#owner_country1').valid();
                    }
                } else {
                    document.getElementById('owner_1_' + addressType2).value = val2;
                    $(document.getElementById('owner_1_' + addressType2)).trigger('change');
                    $(document.getElementById('owner_1_' + addressType2)).valid();
                }
            }
        }
        
        // Show individual address fields, hide autocomplete
        $("#owner_1_street_area").removeClass("d-none");
        $("#owner_1_city_area").removeClass("d-none");
        $("#owner_1_state_area").removeClass("d-none");
        $("#owner_1_country_area").removeClass("d-none");
        $("#owner_1_postal_area").removeClass("d-none");
        $("#owner_1_address_area").addClass("d-none");
        $("#owner_1_street_area input").attr("required", "required");
        $("#owner_1_city_area input").attr("required", "required");
        $("#owner_1_state_area input").attr("required", "required");
        $("#owner_1_postal_area input").attr("required", "required");
        $("#owner_1_address_area input").removeAttr("required");
    });

    // Owner 2 Autocomplete (id="autocomplete3") - only if ownership < 51%
    var input3 = document.getElementById('autocomplete3');
    if (input3) {
        var autocomplete3 = new google.maps.places.Autocomplete(input3);
        autocomplete3.addListener('place_changed', function () {
            var place3 = autocomplete3.getPlace();

            // Clear all fields first
            for (var component in componentForm) {
                if (component === "country") {
                    $('#owner_country2').selectpicker('val', 'Please Select');
                    $('#owner_country2').selectpicker('refresh');
                    $('#owner_country2').trigger('change');
                } else {
                    document.getElementById('owner_2_' + component).value = '';
                    $(document.getElementById('owner_2_' + component)).trigger('change');
                    document.getElementById('owner_2_' + component).disabled = false;
                }
            }

            // Fill in address components
            for (var i = 0; i < place3.address_components.length; i++) {
                var component = place3.address_components[i];
                var addressType3 = null;
                
                for (var j = 0; j < component.types.length; j++) {
                    if (componentForm[component.types[j]]) {
                        addressType3 = component.types[j];
                        break;
                    }
                }
                
                if (addressType3) {
                    var val3 = component[componentForm[addressType3]];
                    if (addressType3 === 'country') {
                        // Match country name to option value and set via selectpicker
                        if ($('#owner_country2').length) {
                            $('#owner_country2 option').each(function () {
                                if ($(this).text() === val3) {
                                    var $countrySelect2 = $('#owner_country2');
                                    $countrySelect2.selectpicker('val', $(this).val());
                                    $countrySelect2.selectpicker('refresh');
                                    $countrySelect2.trigger('change');
                                    if ($countrySelect2.valid()) {
                                        $('#owner_country2').parent('.custom-select').removeClass('is-invalid');
                                    }
                                    if (window.dependentFieldsInstance) {
                                        window.dependentFieldsInstance.triggerFieldsForTrigger('#owner_country2');
                                    }
                                    return false;
                                }
                            });
                            $('#owner_country2').valid();
                        }
                    } else {
                        document.getElementById('owner_2_' + addressType3).value = val3;
                        $(document.getElementById('owner_2_' + addressType3)).trigger('change');
                        $(document.getElementById('owner_2_' + addressType3)).valid();
                    }
                }
            }
            
            // Show individual address fields, hide autocomplete
            $("#owner_2_street_area").removeClass("d-none");
            $("#owner_2_city_area").removeClass("d-none");
            $("#owner_2_state_area").removeClass("d-none");
            $("#owner_2_country_area").removeClass("d-none");
            $("#owner_2_postal_area").removeClass("d-none");
            $("#owner_2_address_area").addClass("d-none");
            $("#owner_2_street_area input").attr("required", "required");
            $("#owner_2_city_area input").attr("required", "required");
            $("#owner_2_state_area input").attr("required", "required");
            $("#owner_2_postal_area input").attr("required", "required");
            $("#owner_2_address_area input").removeAttr("required");
            $('#owner_2_street_area').find(':input').each(function() { $(this).valid(); });
        });
    }
}
```

**Input Field IDs** (must match HTML):
- Owner 1 Autocomplete: `id="autocomplete2"`
- Owner 2 Autocomplete: `id="autocomplete3"` (shown only if ownership_percentage < 51%)

**Output Field IDs** (where data gets populated):

Owner 1:
- `id="owner_1_street_number"` ← street_number component
- `id="owner_1_street_address"` ← route component
- `id="owner_1_city"` ← locality component
- `id="owner_1_state"` ← administrative_area_level_1 component
- `id="owner_1_postal_code"` ← postal code component
- `id="owner_country1"` ← country selectpicker (matches option text)

Owner 2:
- `id="owner_2_street_number"` ← street_number component
- `id="owner_2_street_address"` ← route component
- `id="owner_2_city"` ← locality component
- `id="owner_2_state"` ← administrative_area_level_1 component
- `id="owner_2_postal_code"` ← postal code component
- `id="owner_country2"` ← country selectpicker (matches option text)

**Container IDs** (for show/hide logic):

Owner 1:
- Autocomplete container: `id="owner_1_address_area"` (hidden after autocomplete)
- Individual fields containers: `id="owner_1_street_area"`, `id="owner_1_city_area"`, `id="owner_1_state_area"`, `id="owner_1_country_area"`, `id="owner_1_postal_area"` (shown after autocomplete)

Owner 2:
- Autocomplete container: `id="owner_2_address_area"` (hidden after autocomplete)
- Individual fields containers: `id="owner_2_street_area"`, `id="owner_2_city_area"`, `id="owner_2_state_area"`, `id="owner_2_country_area"`, `id="owner_2_postal_area"` (shown after autocomplete)

---

## Form Submission

**Endpoint**: `POST /v1/ownership`

**Headers**:
```
X-API-Key: {user.security_key}
Content-Type: application/x-www-form-urlencoded
```

**Payload**:
```
uuid={uuid}
primary_contact=1
primary_contact_job_title={optional}
owner_object=owner%5B1%5D%5Bfirst_name%5D=John&owner%5B1%5D%5Bemail%5D=john%40example.com&...
step_count=4
```

**CRITICAL**: The `owner_object` parameter MUST be a URL-encoded string (like from `http_build_query`), NOT JSON.

**JavaScript Implementation** (Proper Form Data Collection):

⚠️ **CRITICAL**: Ensure first_name, last_name, and email are ALWAYS included:

```javascript
// Collect all form data - including readonly fields from API
const form = document.getElementById('step4Form');
const formData = new FormData(form);

// Extract owner data from form fields
const ownerData = {
  owner: {
    1: {
      // REQUIRED fields - these MUST be present for API
      first_name: formData.get('owner[1][first_name]') || '',
      last_name: formData.get('owner[1][last_name]') || '',
      email: formData.get('owner[1][email]') || '',
      
      // Optional/Conditional fields
      users_uuid: formData.get('owner[1][users_uuid]') || '',
      phone: formData.get('owner[1][phone]') || '',
      title: formData.get('owner[1][title]') || '',
      ssn: formData.get('owner[1][ssn]') || '',
      dob: formData.get('owner[1][dob]') || '',
      ownership_percentage: formData.get('owner[1][ownership_percentage]') || '',
      street_number: formData.get('owner[1][street_number]') || '',
      street_address: formData.get('owner[1][street_address]') || '',
      city: formData.get('owner[1][city]') || '',
      state: formData.get('owner[1][state]') || '',
      postal_code: formData.get('owner[1][postal_code]') || '',
      country: formData.get('owner[1][country]') || '',
      license: formData.get('owner[1][license]') || '',
      driver_license_state: formData.get('owner[1][driver_license_state]') || '',
      driver_license_expiration_date: formData.get('owner[1][driver_license_expiration_date]') || '',
      bankruptcy_filed: formData.get('owner[1][bankruptcy_filed]') || '0'
    },
    2: null // or owner 2 data if ownership_percentage < 51
  }
};

// Validate required fields
if (!ownerData.owner[1].first_name) {
  console.error('Validation Error: first_name is required');
  alert('First name is required');
  return false;
}
if (!ownerData.owner[1].last_name) {
  console.error('Validation Error: last_name is required');
  alert('Last name is required');
  return false;
}
if (!ownerData.owner[1].email) {
  console.error('Validation Error: email is required');
  alert('Email is required');
  return false;
}

// Convert to URL-encoded string
const params = new URLSearchParams();
Object.entries(ownerData.owner).forEach(([ownerIndex, ownerObj]) => {
  if (ownerObj) {
    Object.entries(ownerObj).forEach(([key, value]) => {
      params.append(`owner[${ownerIndex}][${key}]`, value || '');
    });
  }
});

// Submit form
const formData = new FormData();
formData.append('uuid', dealUuid);
formData.append('primary_contact', primaryContactValue); // 1 or 0
formData.append('primary_contact_job_title', jobTitleValue || '');
formData.append('owner_object', params.toString()); // URL-encoded string
formData.append('step_count', 4);

const response = await fetch('/v1/ownership', {
  method: 'POST',
  headers: { 'X-API-Key': apiKey },
  body: formData
});
```

**Success Response**:
```json
{
  "status": true,
  "message": "Owners created successfully",
  "uuid": "{uuid}",
  "ruleEngineResponse": {...}
}
```

**On Success - Redirect to Step 5**:
```javascript
if (response.status) {
  const uuid = localStorage.getItem('signup_uuid');
  
  // Redirect to Step 5
  window.location.href = `/signup/step/5/${uuid}`;
}
```

**On Validation Error (HTTP 422)**:
```javascript
// Response example:
{
  "status": false,
  "message": "Validation failed",
  "errors": {
    "owner_object": ["Owner object data is required"],
    "owner.1.first_name": ["First name is required"]
  }
}

// Display field errors
// DO NOT change URL - user stays on Step 4
for (const [field, messages] of Object.entries(response.errors)) {
  displayErrorForField(field, messages[0]);
}
```

**When Owner 2 is Shown** (ownership_percentage < 51):

```json
{
  "owner": {
    "1": { ... },
    "2": {
      "first_name": "Jane",
      "last_name": "Smith",
      "email": "jane@example.com",
      "user_id": "",
      "title": "2",
      "ssn": "987-65-4321",
      "phone": "+1-555-0456",
      "ownership_percentage": "25",
      "dob": "1988-07-22",
      "owner_2_street_number": "456",
      "owner_2_street_address": "Oak Avenue",
      "owner_2_city": "Los Angeles",
      "owner_2_state": "CA",
      "owner_2_postal_code": "90210",
      "country": "United States",
      "bankruptcy_filed": "0",
      "license": "DL987654321",
      "driver_license_state": "CA",
      "driver_license_expiration_date": "2027-11-15"
    }
  }
}
```

---

## Validation Rules

**Owner 1** (All Required):
- ✅ First/Last name: 1-60 chars (pre-filled)
- ✅ Email: Valid format (pre-filled)
- ✅ Title: Must select from dropdown
- ✅ SSN: Valid format, encrypted in DB
- ✅ Phone: Valid international format (intl-tel-input validates)
- ✅ Ownership %: 1-100, integer
- ✅ DOB: Valid date, YYYY-MM-DD
- ✅ Address: Auto-populated from Google, all 6 required when visible
- ✅ Bankruptcy_filed: Must select
- ✅ License: 5-25 chars
- ✅ Driver License State: Required (US only)
- ✅ Driver License Expiration: Future date

**Owner 2** (When Visible, if < 51%):
- ✅ Same as Owner 1
- ✅ Bankruptcy fields optional (not required)

---

## Testing Checklist

**Primary Contact**:
- [ ] Default to "Yes" (business owner)
- [ ] Show job_title field when "No" selected
- [ ] Hide job_title when "Yes" selected

**Owner 1**:
- [ ] Pre-filled fields from auth (name, email)
- [ ] All required fields show and are required
- [ ] Phone formatting works (intl-tel-input)
- [ ] DOB date picker works
- [ ] Google autocomplete shows initially
- [ ] Address fields hidden initially
- [ ] Selecting address populates all 6 fields
- [ ] Autocomplete hides after selection
- [ ] Address fields become visible + required
- [ ] Bankruptcy cascade works correctly
- [ ] License validation (5-25 chars)

**Owner 2 Visibility**:
- [ ] Hidden when ownership >= 51%
- [ ] Shown when ownership < 51%
- [ ] Section clears on hiding

**Owner 2 Fields** (when shown):
- [ ] All required fields are required
- [ ] Google autocomplete for address
- [ ] Same behavior as Owner 1

**Form Submission**:
- [ ] Correct payload structure
- [ ] Owner 2 null when not shown
- [ ] All required fields included
- [ ] Addresses from autocomplete submitted

---

**Last Updated**: 2026-07-22
