# Signup Form - API Integration Guide

## Overview

All dropdown and checkbox values in the signup form are fetched from REST APIs. This guide explains how to integrate these APIs into your form.

---

## Base Configuration

**Base URL**: `/api/partner`

**Response Format (All Endpoints)**:
```json
{
  "data": [
    { "slug": "value_code", "name": "Display Name" },
    { "slug": "value_code2", "name": "Display Name 2" }
  ]
}
```

**Field Mapping**:
- `slug` → Form field value (what gets saved to database)
- `name` → Dropdown label (what user sees)

---

## API Endpoints & Usage

### 1. Countries List
**Endpoint**: `GET /api/partner/countries`

**Used In**:
- Step 1: Formation Country field
- Step 2: Company Country field (optional, from_lander only)

**Field Name**: `country`, `business_formed_in`

**Example Response**:
```json
{
  "data": [
    { "slug": "1", "name": "United States" },
    { "slug": "2", "name": "Canada" },
    { "slug": "3", "name": "Mexico" }
  ]
}
```

**Frontend Implementation**:
```javascript
// On Step 1 mount
fetch('/api/partner/countries')
  .then(res => res.json())
  .then(data => {
    const countrySelect = document.getElementById('country');
    data.data.forEach(country => {
      const option = document.createElement('option');
      option.value = country.slug;        // Form value
      option.text = country.name;         // Display label
      countrySelect.appendChild(option);
    });
  });
```

**Caching**: YES - Cache this locally, it doesn't change often

---

### 2. US States List
**Endpoint**: `GET /api/partner/states`

**Used In**:
- Step 1: Formation State field (conditional - visible only when country = "1")

**Field Name**: `business_state`

**Example Response**:
```json
{
  "data": [
    { "slug": "AL", "name": "Alabama" },
    { "slug": "AK", "name": "Alaska" },
    { "slug": "AZ", "name": "Arizona" }
  ]
}
```

**Conditional Loading**:
- Only load this when Formation Country is selected as "1" (United States)
- Hide state field for all other countries

**Caching**: YES

---

### 3. Industry Types
**Endpoint**: `GET /api/partner/industry-types`

**Used In**:
- Step 2: Industry Type field

**Field Name**: `industry_type`

**Example Response**:
```json
{
  "data": [
    { "slug": "retail", "name": "Retail" },
    { "slug": "ecommerce", "name": "E-commerce" },
    { "slug": "saas", "name": "SaaS" },
    { "slug": "other", "name": "Other" }
  ]
}
```

**Features**:
- Searchable/live search enabled
- When "Other" is selected, show "Industry Type Other" text field

**Caching**: YES

---

### 4. Referral Sources
**Endpoint**: `GET /api/partner/referral-sources`

**Used In**:
- Step 6: "How did you find us?" field

**Field Name**: `howdidyouhear`

**Example Response**:
```json
{
  "data": [
    { "slug": "1", "name": "Google Search" },
    { "slug": "2", "name": "Social Media" },
    { "slug": "50", "name": "Other" },
    { "slug": "14", "name": "Friend" },
    { "slug": "8", "name": "Live Event / Trade Show" }
  ]
}
```

**Dynamic Trigger**:
- When value is "50" (Other), "14" (Friend), or "8" (Live Event) → Show "Additional Information" text field
- Other values → Hide the field

**Caching**: YES

---

### 5. Transaction Devices / Shopping Carts
**Endpoint**: `GET /api/partner/shopping-carts`

**Used In**:
- Step 6: "What device will you use to process transactions?" field

**Field Name**: `transaction_device`

**Example Response**:
```json
{
  "data": [
    { "slug": "terminal", "name": "Card Terminal" },
    { "slug": "software", "name": "Software / Web" },
    { "slug": "virtual", "name": "Virtual Terminal" },
    { "slug": "mobile", "name": "Mobile App" }
  ]
}
```

**Features**:
- Optional field (not required)
- User can select any or none

**Caching**: YES

---

### 6. Interest Details
**Endpoint**: `GET /api/partner/interest-details`

**Used In**:
- Step 6: "Help Your Company Grow" section (checkboxes)

**Field Name**: `other_interests_capital[]` (array)

**Example Response**:
```json
{
  "data": [
    { "slug": "capital", "name": "Capital" },
    { "slug": "resources", "name": "Resources" },
    { "slug": "marketing", "name": "Marketing" },
    { "slug": "coaching", "name": "Business Coaching" },
    { "slug": "networking", "name": "Networking" }
  ]
}
```

**Features**:
- Checkbox group (multiple selection allowed)
- User can select multiple interests
- Values stored as array: `["capital", "resources"]`
- Optional field

**Caching**: YES

---

## Implementation Patterns

### Pattern 1: Basic Dropdown (Single Select)

```html
<!-- HTML -->
<select id="industry_type" name="industry_type" required>
  <option value="">Please Select</option>
  <!-- Options loaded dynamically -->
</select>

<!-- JavaScript -->
<script>
  document.addEventListener('DOMContentLoaded', async () => {
    const response = await fetch('/api/partner/industry-types');
    const { data } = await response.json();
    
    const select = document.getElementById('industry_type');
    data.forEach(industry => {
      const option = document.createElement('option');
      option.value = industry.slug;
      option.textContent = industry.name;
      select.appendChild(option);
    });
  });
</script>
```

### Pattern 2: Conditional Dropdown (Depends on Another Field)

```javascript
// Formation State depends on Formation Country
document.getElementById('country').addEventListener('change', async (e) => {
  const stateDiv = document.querySelector('.usa_state');
  
  if (e.target.value === '1') { // United States
    stateDiv.style.display = 'block';
    
    // Load states only when needed
    const response = await fetch('/api/partner/states');
    const { data } = await response.json();
    
    const select = document.getElementById('business_state');
    select.innerHTML = '<option value="">Please Select</option>';
    data.forEach(state => {
      const option = document.createElement('option');
      option.value = state.slug;
      option.textContent = state.name;
      select.appendChild(option);
    });
  } else {
    stateDiv.style.display = 'none';
    document.getElementById('business_state').value = '';
  }
});
```

### Pattern 3: Checkbox Group (Multiple Select)

```html
<!-- HTML -->
<div id="interest_details">
  <!-- Checkboxes loaded dynamically -->
</div>

<!-- JavaScript -->
<script>
  document.addEventListener('DOMContentLoaded', async () => {
    const response = await fetch('/api/partner/interest-details');
    const { data } = await response.json();
    
    const container = document.getElementById('interest_details');
    data.forEach(interest => {
      const label = document.createElement('label');
      const checkbox = document.createElement('input');
      checkbox.type = 'checkbox';
      checkbox.name = 'other_interests_capital[]';
      checkbox.value = interest.slug;
      label.appendChild(checkbox);
      label.appendChild(document.createTextNode(' ' + interest.name));
      container.appendChild(label);
    });
  });
</script>
```

### Pattern 4: Conditional Field Visibility

```javascript
// Additional Information field depends on referral source
document.getElementById('howdidyouhear').addEventListener('change', (e) => {
  const additionalInfoDiv = document.querySelector('.hear_about_us_other_container');
  const additionalInfoInput = document.getElementById('hear_about_us_other');
  
  const triggerValues = ['50', '14', '8']; // Other, Friend, Live Event
  
  if (triggerValues.includes(e.target.value)) {
    additionalInfoDiv.style.display = 'block';
    additionalInfoInput.required = true;
  } else {
    additionalInfoDiv.style.display = 'none';
    additionalInfoInput.required = false;
    additionalInfoInput.value = '';
  }
});
```

---

## Caching Strategy

### Local Storage / Session State

Since these values don't change frequently, implement caching:

```javascript
const apiCache = {};

async function fetchDropdownData(endpoint) {
  // Check cache first
  if (apiCache[endpoint]) {
    return apiCache[endpoint];
  }
  
  // Fetch from API
  const response = await fetch(`/api/partner/${endpoint}`);
  const data = await response.json();
  
  // Cache for this session
  apiCache[endpoint] = data.data;
  
  return data.data;
}
```

### Pre-load on Page Load

Load all possible dropdown data when the form page loads:

```javascript
async function preloadAllDropdowns() {
  const endpoints = [
    'countries',
    'states',
    'industry-types',
    'referral-sources',
    'shopping-carts',
    'interest-details'
  ];
  
  for (const endpoint of endpoints) {
    await fetchDropdownData(endpoint);
  }
}

// Call on page load
document.addEventListener('DOMContentLoaded', preloadAllDropdowns);
```

---

## Error Handling

```javascript
async function fetchDropdownWithErrorHandling(endpoint) {
  try {
    const response = await fetch(`/api/partner/${endpoint}`);
    
    if (!response.ok) {
      throw new Error(`API returned status ${response.status}`);
    }
    
    const { data } = await response.json();
    return data;
    
  } catch (error) {
    console.error(`Failed to load ${endpoint}:`, error);
    
    // Show error message to user
    const select = document.querySelector(`[name="${endpoint}"]`);
    if (select) {
      select.disabled = true;
      select.innerHTML = '<option>Failed to load options</option>';
    }
    
    return [];
  }
}
```

---

## Loading States

Show loading feedback while fetching data:

```javascript
async function loadDropdownWithSpinner(elementId, endpoint) {
  const select = document.getElementById(elementId);
  
  // Show loading state
  select.innerHTML = '<option>Loading...</option>';
  select.disabled = true;
  
  try {
    const data = await fetchDropdownData(endpoint);
    
    select.innerHTML = '<option value="">Please Select</option>';
    data.forEach(item => {
      const option = document.createElement('option');
      option.value = item.slug;
      option.textContent = item.name;
      select.appendChild(option);
    });
    
  } finally {
    select.disabled = false;
  }
}
```

---

## Form Submission

When submitting the form, send the `slug` values (not the display names):

```javascript
// Form data to submit
{
  country: "1",                                    // slug value
  business_state: "CA",                           // slug value
  industry_type: "retail",                        // slug value
  howdidyouhear: "50",                            // slug value
  transaction_device: "terminal",                 // slug value
  other_interests_capital: ["capital", "resources"] // array of slugs
}
```

---

## API Response Format Specification

All dropdown endpoints follow this exact format:

### Success Response (200)
```json
{
  "data": [
    {
      "slug": "unique_code",
      "name": "Human Readable Name"
    }
  ]
}
```

### Properties:
- **slug** (string, required)
  - Unique identifier for this option
  - Used as form field value
  - Lowercase, alphanumeric, underscore/hyphen allowed
  - Examples: "us", "ca", "retail", "other", "1", "50"

- **name** (string, required)
  - Display label shown to user
  - May contain spaces, special characters
  - Examples: "United States", "E-commerce", "Other"

---

## Loading Order (By Step)

### Step 1 Load
1. Countries (preload)
2. States (load conditionally when country changes)

### Step 2 Load
1. Industry Types (preload)
2. Company Country (optional, if from_lander)

### Step 3 Load
- No dropdowns

### Step 4 Load
- No dropdowns from APIs

### Step 5 Load
- No dropdowns from APIs

### Step 6 Load
1. Referral Sources (preload)
2. Transaction Devices (preload)
3. Interest Details (preload)

---

## Performance Optimization

### Request Batching
Consider fetching multiple endpoints in parallel:

```javascript
async function loadMultipleDropdowns() {
  const endpoints = ['countries', 'industry-types', 'referral-sources'];
  
  const promises = endpoints.map(ep => 
    fetch(`/api/partner/${ep}`).then(r => r.json())
  );
  
  const results = await Promise.all(promises);
  
  // Process all results
  results.forEach((response, index) => {
    processDropdown(endpoints[index], response.data);
  });
}
```

### Lazy Loading
Load non-critical dropdowns only when their fields become visible:

```javascript
// Only load interest details when user scrolls to step 6
const step6Observer = new IntersectionObserver((entries) => {
  if (entries[0].isIntersecting) {
    loadDropdownWithSpinner('other_interests_capital', 'interest-details');
    step6Observer.disconnect();
  }
});

step6Observer.observe(document.getElementById('step_6'));
```

---

## Testing the APIs

### Test Individual Endpoints

```bash
# Countries
curl http://localhost/api/partner/countries

# States
curl http://localhost/api/partner/states

# Industry Types
curl http://localhost/api/partner/industry-types

# Referral Sources
curl http://localhost/api/partner/referral-sources

# Transaction Devices
curl http://localhost/api/partner/shopping-carts

# Interests
curl http://localhost/api/partner/interest-details
```

---

## Troubleshooting

### No data returned
- Check API endpoint URL is correct
- Verify `/api/partner/` prefix is used
- Check browser console for CORS errors
- Test endpoint directly in browser/Postman

### Wrong values in form
- Verify you're using `slug` for form value, not `name`
- Confirm form field name matches documentation
- Check form submission captures all values

### Slow loading
- Implement caching as shown above
- Use `Promise.all()` for parallel requests
- Consider pre-loading data before step loads
- Add loading states so users know what's happening

### Conditional fields not showing
- Verify trigger condition matches API response values
- Use exact slug values in conditions (case-sensitive)
- Test trigger with browser console
- Check CSS selector is correct

---

## Summary Table

| Endpoint | Used In | Field | Type | Cacheable | Required |
|----------|---------|-------|------|-----------|----------|
| `/countries` | Step 1,2 | country | select | Yes | Yes |
| `/states` | Step 1,2 | business_state | select | Yes | Conditional |
| `/industry-types` | Step 2 | industry_type | select | Yes | Yes |
| `/referral-sources` | Step 6 | howdidyouhear | select | Yes | Yes |
| `/shopping-carts` | Step 6 | transaction_device | select | Yes | No |
| `/interest-details` | Step 6 | other_interests_capital | checkbox | Yes | No |

---

## Integration Checklist

- [ ] All endpoints are reachable from frontend
- [ ] CORS is properly configured
- [ ] Responses follow slug/name format
- [ ] Form uses `slug` as value, not `name`
- [ ] Conditional fields trigger correctly
- [ ] Caching is implemented
- [ ] Error handling is in place
- [ ] Loading states are displayed
- [ ] All dropdowns pre-populate with saved data
- [ ] Form submission sends correct values
- [ ] API endpoints are tested with real data

