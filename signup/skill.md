---
name: signup-form-creation
description: Create a fully functional 6-step merchant signup form with API integration, validation, and dependent fields
---

# Merchant Signup Form Skill

Create a complete multi-step signup form that integrates with EMAP APIs. All dropdowns, validation, field dependencies, and form submissions are fully documented and ready to implement.

---

## Quick Reference

**Main Endpoints:**
- Signup: `POST https://emap.epd.dev/api/v1/signup` (Step 1)
- Steps: `POST https://emap.epd.dev/api/v1/application/step` (Steps 2-6)
- Ownership: `POST https://emap.epd.dev/api/v1/ownership` (Step 4)

**Dropdown Endpoints:**
- Countries: `GET https://emap.epd.dev/api/partner/countries`
- States: `GET https://emap.epd.dev/api/partner/states`
- Industries: `GET https://emap.epd.dev/api/partner/industry-types`
- Referral: `GET https://emap.epd.dev/api/partner/referral-sources`
- Devices: `GET https://emap.epd.dev/api/partner/shopping-carts`
- Interests: `GET https://emap.epd.dev/api/partner/interest-details`

---

## Authentication

**Critical: Two Different Header Names!**

| Use Case | Header | Value |
|----------|--------|-------|
| Form Submission | `X-API-Key` | user's security_key |
| Dropdown Data | `Authorization` | user's security_key |

Both use the same value (security_key), but with different header names. This is why dropdowns fail without proper headers!

```javascript
// Form submission (Step 1, 2-6, etc.)
headers: { 'X-API-Key': apiKey }

// Dropdown data (countries, states, etc.)
headers: { 'Authorization': apiKey }
```

---

## Step 1: Account Information

**Fields (send ONLY these 9):**
```json
{
  "first_name": "required|string|max:255",
  "last_name": "required|string|max:255",
  "email": "required|email|unique",
  "phone": "required|regex:/^[0-9\-\+\(\) ]+$/",
  "name": "required|string|max:255",
  "website": "required|url",
  "country": "required|exists:country",
  "annual_sales": "required|integer|min:1",
  "business_state": "nullable (show only if country=US)"
}
```

**Submission:**
```javascript
const step1Data = {
  first_name, last_name, email, phone, name, website,
  country, annual_sales, business_state
};

fetch('https://emap.epd.dev/api/v1/signup', {
  method: 'POST',
  headers: {
    'X-API-Key': apiKey,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(step1Data)
})
// Response: { success: true, uuid: "...", step_count: 1 }
```

**Store UUID for next steps:**
```javascript
localStorage.setItem('signup_uuid', response.uuid);
```

---

## Steps 2-6: Progressive Submission

**Step 2 (Company Info):**
```json
{
  "step_count": 2,
  "uuid": "from_step_1",
  "section": "company_info",
  "legal_name", "name", "industry_type", "customer_service_telephone_number",
  "business_location", "business_formed", "business_organized", "federal_tax_id"
}
```

**Step 3 (Products):**
```json
{
  "step_count": 3,
  "uuid": "from_step_1",
  "section": "product_info",
  "card_swiped", "customer_entered", "staff_entered",
  "average_transaction_amount", "highest_transaction_amount",
  "describe_services", "describe_highest_transaction"
}
```

**Step 4 (Ownership - Two Part):**
```javascript
// Part A: Submit owners
fetch('https://emap.epd.dev/api/v1/ownership', {
  method: 'POST',
  headers: { 'X-API-Key': apiKey },
  body: JSON.stringify({
    "uuid": "from_step_1",
    "owner": {
      "1": { first_name, last_name, email, phone, ssn, job_title, ownership_percentage, dob },
      "2": null  // owner 2 data if ownership_percentage < 50
    }
  })
});

// Part B: Mark step 4 complete
fetch('https://emap.epd.dev/api/v1/application/step', {
  method: 'POST',
  headers: { 'X-API-Key': apiKey },
  body: JSON.stringify({
    "step_count": 4,
    "uuid": "from_step_1",
    "section": "ownership_info"
  })
});
```

**Step 5 (Banking):**
```json
{
  "step_count": 5,
  "uuid": "from_step_1",
  "section": "banking_info",
  "institution_number", "customer_pay_currency", "routing_number", "account_number"
}
```

**Step 6 (Final):**
```json
{
  "step_count": 6,
  "uuid": "from_step_1",
  "section": "interest_details",
  "howdidyouhear", "hear_about_us_other", "multiple_merchant_accounts",
  "transaction_device", "bad_experience", "other_interests_capital"
}
```

---

## Dropdown APIs (Critical Implementation)

### Load Dropdown Data

```javascript
async function loadDropdownData(endpoint, apiKey) {
  try {
    const response = await fetch(`https://emap.epd.dev/api/partner/${endpoint}`, {
      method: 'GET',
      headers: {
        'Authorization': apiKey,  // ← MUST use Authorization header!
        'Content-Type': 'application/json'
      }
    });

    if (response.status === 403) {
      console.error('Invalid API key for dropdowns');
      return [];
    }

    if (!response.ok) throw new Error(`Failed to fetch ${endpoint}`);
    
    const result = await response.json();
    return result.data;  // Array of {name, slug/code}
  } catch (error) {
    console.error(`Dropdown error (${endpoint}):`, error);
    return [];
  }
}

// Usage
const countries = await loadDropdownData('countries', apiKey);
const industries = await loadDropdownData('industry-types', apiKey);
```

### Display & Populate

```javascript
// Populate select element
const countries = await loadDropdownData('countries', apiKey);
const countrySelect = document.querySelector('select[name="country"]');

countries.forEach(country => {
  const option = document.createElement('option');
  option.value = country.code;
  option.textContent = country.name;
  countrySelect.appendChild(option);
});

// Add change listener for dependent fields
countrySelect.addEventListener('change', (e) => {
  if (e.target.value === 'US') {
    document.querySelector('[name="business_state"]').style.display = 'block';
  } else {
    document.querySelector('[name="business_state"]').style.display = 'none';
  }
});
```

---

## Field Dependencies

### business_state (Step 1)
Show only when: `country === 'US'`

### owner_2 section (Step 4)
Show only when: `owner[1].ownership_percentage < 50`

### bad_experience_happened (Step 6)
Show only when: `bad_experience === '1'`

---

## Validation & Error Handling

```javascript
async function submitStep(stepNumber, data) {
  const apiKey = sessionStorage.getItem('api_key');
  
  if (!apiKey) {
    showError('Please log in first');
    return false;
  }

  try {
    const endpoint = stepNumber === 1 ? '/v1/signup' : '/v1/application/step';
    const response = await fetch(`https://emap.epd.dev/api${endpoint}`, {
      method: 'POST',
      headers: {
        'X-API-Key': apiKey,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      const error = await response.json();
      if (response.status === 401 || response.status === 403) {
        showError('API key invalid. Please log in again.');
      } else if (response.status === 422) {
        displayFieldErrors(error.errors);  // Show validation errors
      } else {
        showError(error.message || 'Error submitting form');
      }
      return false;
    }

    const result = await response.json();
    if (stepNumber === 1) {
      localStorage.setItem('signup_uuid', result.uuid);
    }
    return true;
  } catch (error) {
    showError('Network error. Please try again.');
    return false;
  }
}

function displayFieldErrors(errors) {
  Object.entries(errors).forEach(([field, messages]) => {
    const input = document.querySelector(`[name="${field}"]`);
    if (input) {
      input.classList.add('is-invalid');
      const errorEl = input.parentElement.querySelector('.error-text');
      if (errorEl) errorEl.textContent = messages[0];
    }
  });
}
```

---

## Complete Checklist

Before releasing the form:

- [ ] User authentication sets `sessionStorage.setItem('api_key', user.security_key)`
- [ ] Step 1 sends ONLY 9 fields, not all form fields
- [ ] All dropdown endpoints use `Authorization` header
- [ ] All form submissions use `X-API-Key` header
- [ ] UUID persists across all steps
- [ ] business_state shows only when country=US
- [ ] Owner 2 shows only when owner1 < 50%
- [ ] bad_experience_happened shows only when bad_experience=1
- [ ] All error messages display correctly
- [ ] Validation errors show per-field
- [ ] Form prevents submission if validation fails
- [ ] Success redirects to next step
- [ ] No extra fields sent in any payload

---

## Reference Files

- `specification.json` - Complete field definitions & validation rules
- `reference/ENDPOINTS.md` - All 9 endpoints with examples
- `reference/form-submission.md` - Step-by-step implementation guide
- `reference/api-integration.md` - Dropdown integration patterns
