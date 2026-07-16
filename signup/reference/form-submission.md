# Form Submission & Step Navigation Guide

Complete guide for implementing end-to-end form submission with API integration.

---

## Overview

The signup form uses **3 API endpoints** to submit data across 6 steps:

1. **POST /v1/signup** - Step 1 initial signup
2. **POST /v1/application/step** - Steps 2-6 progression
3. **POST /v1/ownership** - Step 4 owner details

---

## Step-by-Step Submission Flow

### STEP 1: Account Information

**Endpoint**: `POST /v1/signup`

**Payload**:
```javascript
{
  first_name: "John",
  last_name: "Doe",
  email: "john@example.com",
  phone: "+1-555-1234",
  name: "Company Name",
  website: "https://example.com",
  country: "1",           // slug from /api/partner/countries
  business_state: "CA",   // slug from /api/partner/states (if country = 1)
  annual_sales: "500000"  // if variant requires it
}
```

**Success Response** (201):
```json
{
  "success": true,
  "uuid": "abc123def456",
  "step_count": 1,
  "message": "Account created successfully"
}
```

**Error Response** (400):
```json
{
  "success": false,
  "message": "Validation failed",
  "errors": {
    "email": "Email already exists",
    "phone": "Invalid phone format"
  }
}
```

**Frontend Action**:
```javascript
// On form submit
const formData = new FormData(form);
const data = Object.fromEntries(formData);

try {
  const response = await fetch('/v1/signup', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-TOKEN': csrfToken
    },
    body: JSON.stringify(data)
  });
  
  if (response.status === 201) {
    const result = await response.json();
    // Store uuid for next steps
    localStorage.setItem('signup_uuid', result.uuid);
    // Navigate to Step 2
    window.location.href = `/step/2/${result.uuid}`;
  } else {
    const error = await response.json();
    // Display field errors
    displayErrors(error.errors);
  }
} catch (err) {
  showError('Network error. Please try again.');
}
```

---

### STEP 2: Company Information

**Endpoint**: `POST /v1/application/step`

**Payload**:
```javascript
{
  step_count: 2,
  uuid: "abc123def456",        // from Step 1
  section: "company_info",
  legal_name: "ABC Inc.",
  name: "ABC",
  industry_type: "retail",     // slug from /api/partner/industry-types
  customer_service_telephone_number: "+1-555-5678",
  business_location: "2",      // slug value
  business_formed: "2020-01-15",
  business_organized: "3",     // slug value
  federal_tax_id: "12-3456789",
  business_register_number: "Optional",  // if non-US
  marketingModel: ["1", "2"],  // array of slugs
  subscription_frequency: "1",
  subscription_frequency_other: null,
  company_address_line: "123 Main St",
  // ... all other Step 2 fields
}
```

**Success Response** (200):
```json
{
  "success": true,
  "step_count": 2,
  "next_step_url": "/step/3/abc123def456",
  "message": "Step 2 saved successfully"
}
```

**Frontend Action**:
```javascript
async function submitStep2(formData) {
  const uuid = localStorage.getItem('signup_uuid');
  
  const payload = {
    step_count: 2,
    uuid: uuid,
    section: "company_info",
    ...Object.fromEntries(new FormData(document.getElementById('step2Form')))
  };
  
  const response = await fetch('/v1/application/step', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-TOKEN': csrfToken
    },
    body: JSON.stringify(payload)
  });
  
  if (response.ok) {
    const result = await response.json();
    window.location.href = result.next_step_url;
  } else {
    const error = await response.json();
    displayErrors(error.errors);
  }
}
```

---

### STEP 3: Product Information

**Endpoint**: `POST /v1/application/step`

**Payload**:
```javascript
{
  step_count: 3,
  uuid: "abc123def456",
  section: "product_info",
  card_swiped: "25",        // percentage
  customer_entered: "50",   // percentage
  staff_entered: "25",      // percentage (total must = 100)
  average_transaction_amount: "200",
  highest_transaction_amount: "1000",
  describe_services: "Retail product sales",
  describe_highest_transaction: "Premium package sale"
}
```

**Navigation**: Continue to Step 4

---

### STEP 4: Owner Information (Two-Part Submission)

**Part A - Ownership Details**

**Endpoint**: `POST /v1/ownership`

**Payload**:
```javascript
{
  uuid: "abc123def456",
  owner: {
    1: {
      first_name: "John",
      last_name: "Doe",
      email: "john@example.com",
      phone: "+1-555-1234",
      ssn: "123-45-6789",
      job_title: "1",          // slug value
      ownership_percentage: 100,
      dob: "1980-05-15",
      // ... other owner fields
    },
    2: null  // Only if owner 1 ownership < 50%
  }
}
```

**Success Response**:
```json
{
  "success": true,
  "message": "Ownership information saved"
}
```

**Part B - Step Submission**

After `/v1/ownership` succeeds, submit Step 4 via `/v1/application/step`:

**Endpoint**: `POST /v1/application/step`

**Payload**:
```javascript
{
  step_count: 4,
  uuid: "abc123def456",
  section: "ownership_info"
  // Ownership data was already submitted via /v1/ownership
}
```

**Frontend Action**:
```javascript
async function submitStep4(ownershipData) {
  const uuid = localStorage.getItem('signup_uuid');
  
  // Step 4a: Submit ownership
  const ownershipResponse = await fetch('/v1/ownership', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-TOKEN': csrfToken
    },
    body: JSON.stringify({
      uuid: uuid,
      owner: ownershipData
    })
  });
  
  if (!ownershipResponse.ok) {
    const error = await ownershipResponse.json();
    displayErrors(error.errors);
    return;
  }
  
  // Step 4b: Submit step marker
  const stepResponse = await fetch('/v1/application/step', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-TOKEN': csrfToken
    },
    body: JSON.stringify({
      step_count: 4,
      uuid: uuid,
      section: "ownership_info"
    })
  });
  
  if (stepResponse.ok) {
    const result = await stepResponse.json();
    window.location.href = result.next_step_url;
  }
}
```

---

### STEP 5: Banking Information

**Endpoint**: `POST /v1/application/step`

**Payload**:
```javascript
{
  step_count: 5,
  uuid: "abc123def456",
  section: "banking_info",
  institution_number: "001",         // if Canada
  customer_pay_currency: "USD",      // if Canada
  routing_number: "123456789",       // US bank
  account_number: "9876543210",
  account_holder_name: "John Doe"
}
```

**Navigation**: Continue to Step 6

---

### STEP 6: Final Details

**Endpoint**: `POST /v1/application/step`

**Payload**:
```javascript
{
  step_count: 6,
  uuid: "abc123def456",
  section: "interest_details",
  howdidyouhear: "50",                          // slug from /api/partner/referral-sources
  hear_about_us_other: "Friend recommendation",  // if howdidyouhear = "50"|"14"|"8"
  multiple_merchant_accounts: "1",
  transaction_device: "terminal",               // slug from /api/partner/shopping-carts
  bad_experience: "0",
  bad_experience_happened: null,                // if bad_experience = "1"
  other_interests_capital: ["capital", "resources"]  // slugs from /api/partner/interest-details
}
```

**Success Response**:
```json
{
  "success": true,
  "message": "Application submitted successfully",
  "redirect": "/dashboard/merchant"
}
```

**Alternative Response** (CCBill required):
```json
{
  "success": true,
  "message": "Application submitted",
  "redirect": "/c/ccbill/abc123def456"
}
```

**Frontend Action**:
```javascript
async function submitStep6(formData) {
  const uuid = localStorage.getItem('signup_uuid');
  
  const payload = {
    step_count: 6,
    uuid: uuid,
    section: "interest_details",
    ...formData
  };
  
  const response = await fetch('/v1/application/step', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-TOKEN': csrfToken
    },
    body: JSON.stringify(payload)
  });
  
  if (response.ok) {
    const result = await response.json();
    // Application complete - redirect to dashboard or payment
    window.location.href = result.redirect;
  }
}
```

---

## Error Handling

### Validation Errors

```javascript
function displayErrors(errors) {
  Object.entries(errors).forEach(([field, message]) => {
    const input = document.querySelector(`[name="${field}"]`);
    if (input) {
      input.classList.add('is-invalid');
      const errorDiv = document.createElement('div');
      errorDiv.className = 'invalid-feedback';
      errorDiv.textContent = message;
      input.parentElement.appendChild(errorDiv);
    }
  });
  
  showToast('error', 'Please fix the errors and try again');
}
```

### Network Errors

```javascript
async function submitWithErrorHandling(endpoint, payload) {
  try {
    const response = await fetch(endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-TOKEN': csrfToken
      },
      body: JSON.stringify(payload),
      signal: AbortSignal.timeout(30000) // 30 second timeout
    });
    
    if (response.status === 419) {
      // Session expired
      showToast('error', 'Your session has expired. Please refresh and try again.');
      window.location.reload();
      return;
    }
    
    if (response.status === 422) {
      // Validation error
      const error = await response.json();
      displayErrors(error.errors);
      return;
    }
    
    if (!response.ok) {
      throw new Error(`API error: ${response.status}`);
    }
    
    return await response.json();
    
  } catch (err) {
    if (err.name === 'AbortError') {
      showToast('error', 'Request timeout. Please try again.');
    } else {
      showToast('error', `Error: ${err.message}`);
    }
    return null;
  }
}
```

---

## Form State Management

### Store UUID Across Steps

```javascript
// After Step 1 submission
class SignupForm {
  constructor() {
    this.uuid = localStorage.getItem('signup_uuid');
  }
  
  setUuid(uuid) {
    this.uuid = uuid;
    localStorage.setItem('signup_uuid', uuid);
  }
  
  getUuid() {
    return this.uuid;
  }
  
  clear() {
    localStorage.removeItem('signup_uuid');
  }
}
```

### Prevent Double Submission

```javascript
let isSubmitting = false;

function submitForm(form) {
  // Prevent double submit
  if (isSubmitting) return;
  
  isSubmitting = true;
  document.querySelector('[type="submit"]').disabled = true;
  
  // Show loader
  document.getElementById('loader').style.display = 'block';
  
  submitWithErrorHandling(/* ... */)
    .then(() => {
      // Success - navigation happens in then()
    })
    .catch(err => {
      showToast('error', 'Submission failed');
    })
    .finally(() => {
      isSubmitting = false;
      document.querySelector('[type="submit"]').disabled = false;
      document.getElementById('loader').style.display = 'none';
    });
}
```

---

## Complete Form Submission Example

```javascript
class SignupFormController {
  constructor(csrfToken) {
    this.csrfToken = csrfToken;
    this.uuid = localStorage.getItem('signup_uuid');
    this.currentStep = this.getCurrentStep();
    this.initEventListeners();
  }
  
  getCurrentStep() {
    // Parse URL to get current step
    const match = window.location.pathname.match(/\/step\/(\d+)/);
    return match ? parseInt(match[1]) : 1;
  }
  
  initEventListeners() {
    const form = document.querySelector('form');
    form?.addEventListener('submit', (e) => this.handleSubmit(e));
  }
  
  async handleSubmit(event) {
    event.preventDefault();
    
    if (!this.validateForm()) {
      showToast('error', 'Please fix validation errors');
      return;
    }
    
    const formData = this.getFormData();
    
    try {
      if (this.currentStep === 1) {
        await this.submitStep1(formData);
      } else if (this.currentStep === 4) {
        await this.submitStep4(formData);
      } else if (this.currentStep === 6) {
        await this.submitStep6(formData);
      } else {
        await this.submitStep(formData);
      }
    } catch (error) {
      showToast('error', error.message);
    }
  }
  
  async submitStep1(data) {
    const response = await fetch('/v1/signup', {
      method: 'POST',
      headers: this.getHeaders(),
      body: JSON.stringify(data)
    });
    
    if (response.status === 201) {
      const result = await response.json();
      this.setUuid(result.uuid);
      window.location.href = `/step/2/${result.uuid}`;
    } else {
      throw new Error('Step 1 submission failed');
    }
  }
  
  async submitStep4(data) {
    // First submit ownership
    await fetch('/v1/ownership', {
      method: 'POST',
      headers: this.getHeaders(),
      body: JSON.stringify({
        uuid: this.uuid,
        owner: data.owner
      })
    });
    
    // Then submit step
    await this.submitStep(data);
  }
  
  async submitStep6(data) {
    const response = await fetch('/v1/application/step', {
      method: 'POST',
      headers: this.getHeaders(),
      body: JSON.stringify({
        step_count: 6,
        uuid: this.uuid,
        section: 'interest_details',
        ...data
      })
    });
    
    if (response.ok) {
      const result = await response.json();
      window.location.href = result.redirect;
    }
  }
  
  async submitStep(data) {
    const response = await fetch('/v1/application/step', {
      method: 'POST',
      headers: this.getHeaders(),
      body: JSON.stringify({
        step_count: this.currentStep,
        uuid: this.uuid,
        ...data
      })
    });
    
    if (response.ok) {
      const result = await response.json();
      window.location.href = result.next_step_url;
    } else {
      const error = await response.json();
      throw new Error(error.message);
    }
  }
  
  validateForm() {
    // Use jQuery validation or HTML5 validation
    return document.querySelector('form').checkValidity();
  }
  
  getFormData() {
    const formData = new FormData(document.querySelector('form'));
    return Object.fromEntries(formData);
  }
  
  setUuid(uuid) {
    this.uuid = uuid;
    localStorage.setItem('signup_uuid', uuid);
  }
  
  getHeaders() {
    return {
      'Content-Type': 'application/json',
      'X-CSRF-TOKEN': this.csrfToken
    };
  }
}

// Initialize on page load
document.addEventListener('DOMContentLoaded', () => {
  const csrfToken = document.querySelector('meta[name="csrf-token"]')?.content;
  new SignupFormController(csrfToken);
});
```

---

## Checklist for End-to-End Implementation

- [ ] All form fields map to API payload fields
- [ ] Client-side validation before submission
- [ ] CSRF token included in all requests
- [ ] UUID stored and passed through all steps
- [ ] Error responses handled and displayed
- [ ] Loading states show during submission
- [ ] Double-submit prevention implemented
- [ ] Session timeout handling (419 responses)
- [ ] Network error handling and retry logic
- [ ] Successful responses redirect correctly
- [ ] All dependent fields validate before submit
- [ ] Step-specific endpoints called correctly

---

**Ready for End-to-End Form Generation** ✅
