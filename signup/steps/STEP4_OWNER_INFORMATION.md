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
| driver_license_state | select | owner[1]['driver_license_state'] | Required | US States dropdown (2-letter codes) |
| driver_license_expiration_date | date | owner[1]['driver_license_expiration_date'] | Required | Format: YYYY-MM-DD |

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
| driver_license_state | select | owner[2]['driver_license_state'] | Required | US States dropdown |
| driver_license_expiration_date | date | owner[2]['driver_license_expiration_date'] | Required | Format: YYYY-MM-DD |

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

**Library**: Google Maps Places API v3

**Both Owner 1 & 2 Use Same Pattern**:

```javascript
// Owner 1
const owner1Autocomplete = new google.maps.places.Autocomplete(
    document.getElementById('autocomplete2'),
    { types: ['geocode'] }
);

// Owner 2
const owner2Autocomplete = new google.maps.places.Autocomplete(
    document.getElementById('autocomplete3'),
    { types: ['geocode'] }
);

// Component mapping (both owners)
var componentForm = {
    street_number: 'short_name',      // e.g., "123"
    route: 'long_name',                // e.g., "Main Street"
    locality: 'long_name',             // e.g., "San Francisco"
    administrative_area_level_1: 'short_name',  // e.g., "CA"
    country: 'long_name',              // e.g., "United States"
    postal_code: 'short_name'          // e.g., "94102"
};
```

**Field Mapping**:

Owner 1:
- `street_number` → `owner_1_street_number`
- `route` → `owner_1_street_address`
- `locality` → `owner_1_city`
- `administrative_area_level_1` → `owner_1_state`
- `postal_code` → `owner_1_postal_code`
- `country` → `owner[1][country]` (selectpicker)

Owner 2:
- `street_number` → `owner_2_street_number`
- `route` → `owner_2_street_address`
- `locality` → `owner_2_city`
- `administrative_area_level_1` → `owner_2_state`
- `postal_code` → `owner_2_postal_code`
- `country` → `owner[2][country]` (selectpicker)

**Implementation Notes**:
- Extract `short_name` or `long_name` from each address_components item
- Trigger 'change' event on populated fields for validation
- Use selectpicker refresh for country dropdowns
- Hide autocomplete container, show address fields after selection

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

**JavaScript Implementation**:
```javascript
// Build owner data object
const ownerData = {
  owner: {
    1: {
      first_name: 'John',
      last_name: 'Doe',
      email: 'john@example.com',
      users_uuid: 'uuid-from-api', // if from primary_contact API
      phone: '+1-555-0123',
      title: '1',
      ssn: '123-45-6789',
      dob: '1985-03-15',
      ownership_percentage: '75',
      street_number: '123',
      street_address: 'Main Street',
      city: 'San Francisco',
      state: 'CA',
      postal_code: '94102',
      country: 'United States',
      license: 'DL123456789',
      driver_license_state: 'CA',
      driver_license_expiration_date: '2028-05-10',
      bankruptcy_filed: '0'
    },
    2: null // or owner 2 data if ownership_percentage < 51
  }
};

// Convert to URL-encoded string
const params = new URLSearchParams();
Object.entries(ownerData.owner).forEach(([ownerIndex, ownerObj]) => {
  if (ownerObj) {
    Object.entries(ownerObj).forEach(([key, value]) => {
      params.append(`owner[${ownerIndex}][${key}]`, value);
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
