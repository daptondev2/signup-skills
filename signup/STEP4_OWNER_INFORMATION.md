---
name: step4-owner-information
description: Step 4 - Owner Information form with Owner 1 & conditional Owner 2 details
---

# Step 4: Owner Information

Complete specification for Owner Information form with all fields, validations, and conditional logic.

---

## Overview

Step 4 captures owner details for up to 2 owners. Owner 2 is conditional and appears only when Owner 1 ownership percentage < 51%.

---

## OWNER 1: Basic Information

### Fields

| Field | Type | Required | Name Attribute | Notes |
|-------|------|----------|---|---|
| First Name | text | Yes | owner[1]['first_name'] | Pre-filled from auth, hidden input, max 60 chars |
| Last Name | text | Yes | owner[1]['last_name'] | Pre-filled from auth, hidden input, max 60 chars |
| Email | text | Yes | owner[1]['email'] | Pre-filled from auth, hidden input, valid email |
| Title | select | Yes | owner[1][title] | Static dropdown: 1-14 (see Job Titles section) |
| SSN | text | Yes | owner[1]['ssn'] | Encrypted, masked input, country-specific label |
| Mobile Phone | tel | Yes | owner[1]['phone'] | Use intl-tel-input, 7-15 digits |
| % of Company Owned | number | Yes | owner[1]['ownership_percentage'] | Min: 1, Max: 100, triggers Owner 2 if <51% |
| Date of Birth | date | Yes | owner[1]['dob'] | YYYY-MM-DD format, use date picker |

### Implementation Notes

```html
<!-- Owner 1 is pre-filled from auth user -->
<input type="hidden" name="owner[1]['email']" value="{{Auth::user()->email}}" />
<input type="hidden" name="owner[1]['first_name']" value="{{Auth::user()->first_name}}" />
<input type="hidden" name="owner[1]['last_name']" value="{{Auth::user()->last_name}}" />

<!-- Display name from auth -->
<span class="owner1_name">{{ Auth::user()->full_name }}'s Details</span>

<!-- Ownership percentage triggers conditional display -->
<input type="number" name="owner[1]['ownership_percentage']" min="1" max="100" required>
<small id="additional_owner_hint_text" style="display: block/none;">
    Additional owner information required below
</small>
```

---

## OWNER 1: Financial History

### Fields

| Field | Type | Required | Name Attribute | Notes |
|-------|------|----------|---|---|
| Filed Bankruptcy | radio | Yes | owner[1]['bankruptcy_filed'] | Options: 1 (Yes), 0 (No) |
| Bankruptcy Discharged | radio | Conditional | owner[1]['bankruptcy_discharged'] | Show if filed_bankruptcy=1, Options: 1 (Yes), 0 (No) |
| Discharge Date | date | Conditional | owner[1]['bankruptcy_discharged_date'] | Show if bankruptcy_discharged=1, YYYY-MM-DD format |

### Implementation

```html
<!-- Card Header -->
<div class="card-header bold">
    <span class="owner1_name">{{ Auth::user()->full_name }}'s</span> Financial History
</div>

<!-- Filed Bankruptcy -->
<label>
    <input type="radio" name="owner[1]['bankruptcy_filed']" value="1">
    Yes
</label>
<label>
    <input type="radio" name="owner[1]['bankruptcy_filed']" value="0">
    No
</label>

<!-- Conditional: Bankruptcy Discharged Section -->
<div class="bankruptcy_discharged_section" style="display: none;">
    <label>
        <input type="radio" name="owner[1]['bankruptcy_discharged']" value="1">
        Yes
    </label>
    <label>
        <input type="radio" name="owner[1]['bankruptcy_discharged']" value="0">
        No
    </label>
</div>

<!-- Conditional: Discharge Date -->
<div class="owner_discharged_date_section" style="display: none;">
    <input type="date" name="owner[1]['bankruptcy_discharged_date']" placeholder="YYYY-MM-DD">
</div>
```

### Conditional Logic

```javascript
// Show bankruptcy discharged section if filed_bankruptcy=1
$('input[name="owner[1][bankruptcy_filed]"]').on('change', function() {
    if (this.value === '1') {
        $('.bankruptcy_discharged_section').show();
    } else {
        $('.bankruptcy_discharged_section').hide();
    }
});

// Show discharge date section if bankruptcy_discharged=1
$('input[name="owner[1][bankruptcy_discharged]"]').on('change', function() {
    if (this.value === '1') {
        $('.owner_discharged_date_section').show();
    } else {
        $('.owner_discharged_date_section').hide();
    }
});
```

---

## OWNER 1: Home Address

### Fields

| Field | Type | Required | Name Attribute | Notes |
|-------|------|----------|---|---|
| Address Autocomplete | text | No | autocomplete2 | Google Maps Places autocomplete |
| Street Number | text | Yes | owner_1_street_number | Max 10 chars, auto-populated from autocomplete |
| Street Address | text | Yes | owner_1_street_address | Max 60 chars, auto-populated from autocomplete |
| City | text | Yes | owner_1_city | Max 60 chars, auto-populated from autocomplete |
| State/Province | text | Yes | owner_1_state | Max 40 chars, label varies by country |
| Postal Code | text | Yes | owner_1_postal_code | Max 15 chars, auto-populated from autocomplete |
| Country | select | Yes | owner[1][country] | API: /api/partner/countries |

### Implementation

```html
<!-- Autocomplete field (hidden if address already filled) -->
<input type="text" name="autocomplete2" id="autocomplete2" 
       class="form-control" placeholder="Enter in Your Residence Location">

<!-- Address fields (initially hidden) -->
<div id="owner_1_street_area" style="display: none;">
    <input type="text" name="owner_1_street_number" placeholder="Street Number">
    <input type="text" name="owner_1_street_address" placeholder="Street Address">
    <input type="text" name="owner_1_city" placeholder="City">
    <input type="text" name="owner_1_state" placeholder="State/Province">
    <input type="text" name="owner_1_postal_code" placeholder="Postal Code">
    <select name="owner[1][country]"><!-- Countries from API --></select>
</div>
```

### Google Maps Initialization

```javascript
// Initialize Google Maps autocomplete for owner 1
const autocomplete = new google.maps.places.Autocomplete(
    document.getElementById('autocomplete2')
);

autocomplete.addListener('place_changed', function() {
    const place = autocomplete.getPlace();
    // Auto-populate fields
    document.querySelector('[name="owner_1_street_number"]').value = place.street_number || '';
    document.querySelector('[name="owner_1_street_address"]').value = place.route || '';
    document.querySelector('[name="owner_1_city"]').value = place.locality || '';
    document.querySelector('[name="owner_1_state"]').value = place.administrative_area_level_1 || '';
    document.querySelector('[name="owner_1_postal_code"]').value = place.postal_code || '';
});
```

---

## OWNER 1: Driver's License Details

### Fields

| Field | Type | Required | Name Attribute | Notes |
|-------|------|----------|---|---|
| License Number | text | Yes | owner[1]['license'] | Min 5, Max 25 chars, Driver's License or Passport |
| License State | select | Conditional | owner[1]['driver_license_state'] | Show if country="US", US states dropdown |
| License Expiration | date | Conditional | owner[1]['driver_license_expiration_date'] | Show if country="US", YYYY-MM-DD format |

### Implementation

```html
<!-- License Number (always visible) -->
<input type="text" name="owner[1]['license']" placeholder="Driver's License / Passport Number"
       minlength="5" maxlength="25" required>

<!-- License State (USA only) -->
<div class="driver_license_state1" style="display: none;">
    <select name="owner[1]['driver_license_state']">
        <!-- US States from API -->
    </select>
</div>

<!-- License Expiration (USA only) -->
<div class="driver_license_state1" style="display: none;">
    <input type="date" name="owner[1]['driver_license_expiration_date']" 
           placeholder="YYYY-MM-DD">
</div>
```

### Conditional Display Logic

```javascript
// Show license state/expiration only for USA
$('#owner_country1').on('change', function() {
    if (this.value === 'US') {
        $('.driver_license_state1').show();
    } else {
        $('.driver_license_state1').hide();
    }
});
```

---

## OWNER 2: Conditional Section

### ⚠️ Important: Initial State

**Owner 2 section MUST be hidden on page load** — it should not appear until ownership percentage < 51 is selected.

### Trigger Condition

**Show when**: `owner[1]['ownership_percentage'] < 51`

**Hide when**: `owner[1]['ownership_percentage'] >= 51`

```html
<!-- Owner 2 section - MUST be hidden by default (display: none;) -->
<!-- Do NOT remove this style attribute or Owner 2 will show immediately -->
<div id="owner-section-section" style="display: none;">
    <!-- All Owner 2 fields stay hidden until ownership_percentage < 51 -->
    <!-- JavaScript only toggles this visibility - doesn't create initial state -->
</div>

<script>
$('#owner\\[1\\]\\[ownership_percentage\\]').on('change', function() {
    const pct = parseInt($(this).val());
    if (pct < 51) {
        $('#owner-section-section').show();
        // Make fields required
    } else {
        $('#owner-section-section').hide();
        // Clear and make optional
    }
});
</script>
```

---

## OWNER 2: Basic Information

### Fields

| Field | Type | Required | Name Attribute | Notes |
|-------|------|----------|---|---|
| First Name | text | No | owner[2]['first_name'] | Optional, max 60 chars, required when Owner 2 visible |
| Last Name | text | No | owner[2]['last_name'] | Optional, max 60 chars, required when Owner 2 visible |
| Email | text | No | owner[2]['email'] | Optional, valid email format, required when shown |
| User ID | hidden | - | owner[2]['user_id'] | Hidden field for existing owner |
| Title | select | No | owner[2]['title'] | Optional, same job titles as Owner 1 |
| SSN | text | No | owner[2]['ssn'] | Optional, encrypted, masked, required when shown |
| Mobile Phone | tel | No | owner[2]['phone'] | Optional, intl-tel-input, required when shown |
| % of Company Owned | number | No | owner[2]['ownership_percentage'] | Optional, Min 1-100, required when shown |
| Date of Birth | date | No | owner[2]['dob'] | Optional, YYYY-MM-DD, required when shown |

### Implementation

```html
<div class="card-header bold">
    <span class="owner2_title_text">{{ $owner2Title }}'s</span> Details
</div>

<input type="hidden" name="owner[2]['user_id']" value="...">

<!-- All fields same as Owner 1 but with owner[2] name attribute -->
<input type="text" name="owner[2]['first_name']" placeholder="First Name">
<input type="email" name="owner[2]['email']" placeholder="Email Address">
<select name="owner[2]['title']"><!-- Job titles --></select>
<input type="text" name="owner[2]['ssn']" placeholder="SSN/SIN">
<input type="tel" name="owner[2]['phone']" placeholder="Mobile Phone">
<input type="number" name="owner[2]['ownership_percentage']" min="1" max="100">
<input type="date" name="owner[2]['dob']" placeholder="Date of Birth">
```

---

## OWNER 2: Financial History

Same as Owner 1, but with:
- Name attribute: `owner[2]['bankruptcy_filed']`, `owner[2]['bankruptcy_discharged']`, `owner[2]['bankruptcy_discharged_date']`
- Same conditional logic (discharged → discharge date)

---

## OWNER 2: Home Address

Same as Owner 1, but with:
- Autocomplete field: `autocomplete3`
- Name attributes: `owner_2_street_number`, `owner_2_street_address`, `owner_2_city`, `owner_2_state`, `owner_2_postal_code`
- Select field: `owner[2][country]`

---

## OWNER 2: Driver's License Details

Same as Owner 1, but with:
- Name attribute: `owner[2]['license']`, `owner[2]['driver_license_state']`, `owner[2]['driver_license_expiration_date']`
- Same USA-only conditional logic

---

## Static Dropdowns

### Job Titles (Both Owner 1 and Owner 2)

```
1: CEO
2: CFO
3: Chairman
4: Co-owner
5: Controller
6: Director
7: General-Manager
8: Office-Manager
9: Owner
10: Partner
11: President
12: Treasurer
13: Vice-President
14: Other
```

### Note on Value Formats

**Numeric ID values** (1-14 for job_title):
- Submit as: "1", "2", "3", etc.

**Slug values** (for other dropdowns):
- Submit as: "0-7-days", "7-30-days", etc.
- Always use slug format from API responses

---

## API Integration

### Dropdown Data

```
GET /api/partner/countries
Header: Authorization: {user.security_key}
```

Returns country list for home address selection.

### Form Submission

```
POST /api/v1/ownership
Headers: X-API-Key: {user.security_key}

Payload:
{
    "uuid": "{signup_uuid}",
    "owner": {
        "1": {
            "first_name": "from auth",
            "last_name": "from auth",
            "email": "from auth",
            "title": "1-14",
            "ssn": "encrypted",
            "phone": "formatted",
            "ownership_percentage": "1-100",
            "dob": "YYYY-MM-DD",
            "bankruptcy_filed": "0|1",
            "bankruptcy_discharged": "0|1|null",
            "bankruptcy_discharged_date": "YYYY-MM-DD|null",
            "street_number": "...",
            "street_address": "...",
            "city": "...",
            "state": "...",
            "postal_code": "...",
            "country": "US|CA|etc",
            "license": "...",
            "driver_license_state": "CA|NY|etc (USA only)",
            "driver_license_expiration_date": "YYYY-MM-DD (USA only)"
        },
        "2": null  // or complete owner 2 object if ownership_percentage[1] < 51
    }
}

Response: { success: true, message: "..." }
```

### Then Submit Step Marker

```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}

Payload:
{
    "step_count": 4,
    "section": "ownership_info",
    "uuid": "{signup_uuid}"
}

Response: { next_step_url: "/step/5/{uuid}" }
```

---

## Key Implementation Details

### SSN Label (Country-Specific)

```javascript
// For USA, Canada, Mexico
if (country === "US" || country === "CA" || country === "MX") {
    label = "SSN/SIN";
} else {
    label = "SSN (or personal Tax ID equivalent)";
}
```

### Driver's License (USA Only)

Show `driver_license_state` and `driver_license_expiration_date` only when:
- Country = "US"
- Both fields required when visible

### Owner 2 Visibility

```javascript
if (ownership_percentage[1] < 51) {
    show("#owner-section-section");
    make_fields_required();
} else {
    hide("#owner-section-section");
    clear_owner_2_fields();
    make_fields_optional();
}
```

---

## Field Count Summary

**Owner 1**: 
- Basic Info: 8 fields
- Financial History: 3 fields (1 required, 2 conditional)
- Home Address: 7 fields
- Driver's License: 3 fields (1 required, 2 conditional USA only)
- **Total**: 21 fields

**Owner 2** (Conditional if Owner 1 < 51%):
- Basic Info: 9 fields (including hidden user_id)
- Financial History: 3 fields
- Home Address: 7 fields
- Driver's License: 3 fields
- **Total**: 22 fields

**Grand Total**: 43 fields (21 Owner 1 + 22 Owner 2 conditional)

---

## Libraries Required

- `intl-tel-input@22.0.2` - Phone formatting
- `Google Maps Places API` - Address autocomplete
- `dependent-fields.js` - Conditional visibility
- Date picker library - Date of birth, expiration, discharge date
- jQuery Validate - Client-side validation

---

**Production Ready** ✅  
Last Updated: 2026-07-19
