---
name: signup-step-2
description: Step 2 Company Information - Business details, legal address, revenue model, and Google Maps autocomplete
---

# STEP 2: Company Information

Second step of the 7-step signup form. Collects company business details, addresses, and revenue model information.

---

## Fields

**Company Details** (10 fields):

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

**Script**:
```html
<script src="https://maps.google.com/maps/api/js?key={GOOGLE_API_KEY}&libraries=places"></script>
```

**Legal Address Component Mapping**:
```javascript
var componentForm = {
    street_number: 'short_name',
    route: 'long_name',
    locality: 'long_name',
    administrative_area_level_1: 'short_name',
    country: 'long_name',
    postal_code: 'short_name'
};
```

**Physical Address Component Mapping**:
```javascript
var componentMapping = {
    street_number: 'physical_address_street_number',
    route: 'physical_address_street_address',
    locality: 'physical_address_city',
    administrative_area_level_1: 'physical_address_state',
    postal_code: 'physical_address_postal_code',
    country: 'physical_address_country'
};
```

---

## Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

**Level 1 Dependencies:**
- **business_state**: Show if country="US"
- **business_register_number**: Show if country≠"US"
- **Physical address section**: Show if is_physical_address_same_as_legal_address="yes"
- **subscription_frequency**: Show if marketingModel includes "Recurring/Subscription"

**Level 2 Dependencies:**
- **subscription_frequency_other**: Show if subscription_frequency="Other"

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
