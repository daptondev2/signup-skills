# Step 1 Implementation Example

Complete JavaScript implementation for Step 1 (Account Information).

---

## HTML Form

```html
<form id="step1-form">
  <!-- Owner Details -->
  <div class="form-group">
    <label>First Name *</label>
    <input type="text" name="first_name" id="first_name" 
           maxlength="60" required>
    <span class="error-message"></span>
  </div>

  <div class="form-group">
    <label>Last Name *</label>
    <input type="text" name="last_name" id="last_name" 
           maxlength="60" required>
    <span class="error-message"></span>
  </div>

  <div class="form-group">
    <label>Email *</label>
    <input type="email" name="email" id="email" required>
    <span class="error-message"></span>
  </div>

  <div class="form-group">
    <label>Phone *</label>
    <input type="tel" name="phone" id="phone" required>
    <span class="error-message"></span>
  </div>

  <!-- Company Information -->
  <div class="form-group">
    <label>Company Name *</label>
    <input type="text" name="name" id="name" 
           maxlength="60" required>
    <span class="error-message"></span>
  </div>

  <div class="form-group">
    <label>Company Website *</label>
    <input type="url" name="website" id="website" required>
    <span class="error-message"></span>
  </div>

  <div class="form-group">
    <label>Formation Country *</label>
    <select name="business_formed_in" id="business_formed_in" required>
      <option value="">Select Country...</option>
      <!-- Populated by JavaScript -->
    </select>
    <span class="error-message"></span>
  </div>

  <!-- Conditional: State if US -->
  <div class="form-group usa_state" style="display: none;">
    <label>Formation State *</label>
    <select name="business_state" id="business_state">
      <option value="">Select State...</option>
      <!-- Populated by JavaScript -->
    </select>
    <span class="error-message"></span>
  </div>

  <!-- Optional: Annual Sales -->
  <div class="form-group">
    <label>Annual CC Sales</label>
    <input type="text" name="annual_cc_sales" id="annual_cc_sales" 
           placeholder="0" maxlength="12">
    <span class="error-message"></span>
  </div>

  <!-- Optional: Promo Code -->
  <div class="form-group">
    <label>Promo Code</label>
    <input type="text" name="promo_code" id="promo_code" 
           maxlength="100">
  </div>

  <input type="hidden" name="step_count" value="1">
  <button type="submit">Next</button>
</form>
```

---

## JavaScript Implementation

```javascript
const API_BASE = 'https://emap.epd.dev';
const API_TOKEN = localStorage.getItem('api_token');
const CSRF_TOKEN = document.querySelector('meta[name="csrf-token"]').content;

// Initialize form
document.addEventListener('DOMContentLoaded', async () => {
  await loadReferenceData();
  setupFormValidation();
  setupDependentFields();
  setupIntlTelInput();
  setupCleaveFormatting();
  setupFormSubmission();
});

// 1. Load reference data (countries, states)
async function loadReferenceData() {
  try {
    // Fetch countries
    const countriesResponse = await fetch(`${API_BASE}/api/countries`, {
      headers: { 'Authorization': `Bearer ${API_TOKEN}` }
    });
    const countries = await countriesResponse.json();
    
    const countrySelect = document.getElementById('business_formed_in');
    countries.forEach(country => {
      const option = document.createElement('option');
      option.value = country.id;
      option.textContent = country.name;
      countrySelect.appendChild(option);
    });

    // Fetch states
    const statesResponse = await fetch(`${API_BASE}/api/states`, {
      headers: { 'Authorization': `Bearer ${API_TOKEN}` }
    });
    const states = await statesResponse.json();
    
    const stateSelect = document.getElementById('business_state');
    states.forEach(state => {
      const option = document.createElement('option');
      option.value = state.code;
      option.textContent = state.name;
      stateSelect.appendChild(option);
    });

    // Cache for reference
    window.referenceData = { countries, states };
  } catch (error) {
    console.error('Failed to load reference data:', error);
  }
}

// 2. Setup form validation
function setupFormValidation() {
  const form = document.getElementById('step1-form');
  
  form.addEventListener('submit', (e) => {
    e.preventDefault();
    
    if (validateForm()) {
      submitForm();
    }
  });

  // Real-time validation
  document.getElementById('email').addEventListener('blur', validateEmail);
  document.getElementById('phone').addEventListener('blur', validatePhone);
  document.getElementById('website').addEventListener('blur', validateWebsite);
}

function validateForm() {
  let isValid = true;
  const requiredFields = [
    'first_name', 'last_name', 'email', 'phone',
    'name', 'website', 'business_formed_in'
  ];

  // Check US state if selected
  if (document.getElementById('business_formed_in').value === '1') {
    requiredFields.push('business_state');
  }

  requiredFields.forEach(fieldId => {
    const field = document.getElementById(fieldId);
    if (!field.value.trim()) {
      showFieldError(fieldId, 'This field is required');
      isValid = false;
    }
  });

  return isValid;
}

function validateEmail() {
  const email = document.getElementById('email').value;
  const regex = /^([a-zA-Z0-9_\.\-])+\@(([a-zA-Z0-9\-])+\.)+([a-zA-Z0-9]{2,4})+$/;
  
  if (email && !regex.test(email)) {
    showFieldError('email', 'Invalid email format');
    return false;
  }
  clearFieldError('email');
  return true;
}

function validateWebsite() {
  const url = document.getElementById('website').value;
  if (!url.startsWith('http://') && !url.startsWith('https://')) {
    showFieldError('website', 'URL must start with http:// or https://');
    return false;
  }
  clearFieldError('website');
  return true;
}

function validatePhone() {
  const phone = document.getElementById('phone').iti;
  if (phone && !phone.isValidNumber()) {
    showFieldError('phone', 'Please enter a valid phone number');
    return false;
  }
  clearFieldError('phone');
  return true;
}

// 3. Setup dependent fields (Country → State)
function setupDependentFields() {
  document.getElementById('business_formed_in').addEventListener('change', (e) => {
    const stateContainer = document.querySelector('.usa_state');
    const stateSelect = document.getElementById('business_state');
    
    if (e.target.value === '1') {
      // US selected - show and require state
      stateContainer.style.display = 'block';
      stateSelect.required = true;
      stateSelect.parentElement.querySelector('label').innerHTML += ' *';
    } else {
      // Non-US - hide and clear state
      stateContainer.style.display = 'none';
      stateSelect.required = false;
      stateSelect.value = '';
      clearFieldError('business_state');
    }
  });
}

// 4. Setup international phone input
function setupIntlTelInput() {
  const phoneInput = document.getElementById('phone');
  
  const iti = window.intlTelInput(phoneInput, {
    initialCountry: 'us',
    preferredCountries: ['us', 'ca'],
    separateDialCode: true,
    utilsScript: '//cdn.jsdelivr.net/npm/intl-tel-input@17/build/js/utils.js'
  });

  // Store for later use
  phoneInput.iti = iti;

  // Validate on blur
  phoneInput.addEventListener('blur', validatePhone);
}

// 5. Setup Cleave.js for currency formatting
function setupCleaveFormatting() {
  new Cleave('#annual_cc_sales', {
    numeral: true,
    numeralThousandsGroupStyle: 'thousand',
    numeralDecimalScale: 0,
    numeralPositiveOnly: true
  });
}

// 6. Submit form
async function submitForm() {
  const form = document.getElementById('step1-form');
  const formData = new FormData(form);
  
  // Get phone in E.164 format
  const phoneInput = document.getElementById('phone');
  const phoneNumber = phoneInput.iti.getNumber();
  formData.set('phone', phoneNumber);

  try {
    const response = await fetch(`${API_BASE}/signup/user`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${API_TOKEN}`,
        'X-CSRF-TOKEN': CSRF_TOKEN
      },
      body: formData
    });

    const data = await response.json();

    if (!response.ok) {
      if (response.status === 400) {
        // Validation errors
        displayFieldErrors(data.errors);
      } else if (response.status === 419) {
        // CSRF token expired - refresh and retry
        refreshCSRFToken();
        submitForm();
      } else {
        showError(data.message || 'An error occurred');
      }
      return;
    }

    // Success
    const uuid = data.uuid;
    localStorage.setItem('signup_uuid', uuid);
    
    // Redirect to Step 2
    window.location.href = data.url;

  } catch (error) {
    console.error('Submission error:', error);
    showError('Network error. Please try again.');
  }
}

// Error handling helpers
function showFieldError(fieldId, message) {
  const field = document.getElementById(fieldId);
  field.classList.add('error');
  field.parentElement.querySelector('.error-message').textContent = message;
}

function clearFieldError(fieldId) {
  const field = document.getElementById(fieldId);
  field.classList.remove('error');
  field.parentElement.querySelector('.error-message').textContent = '';
}

function displayFieldErrors(errors) {
  Object.entries(errors).forEach(([field, messages]) => {
    showFieldError(field, messages[0]);
  });
}

function showError(message) {
  alert(message); // Replace with better UX
}

function refreshCSRFToken() {
  const meta = document.querySelector('meta[name="csrf-token"]');
  window.CSRF_TOKEN = meta.content;
}
```

---

## CSS Styling

```css
.form-group {
  margin-bottom: 1.5rem;
  display: flex;
  flex-direction: column;
}

.form-group label {
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-group input,
.form-group select {
  padding: 0.75rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
  transition: border-color 0.2s;
}

.form-group input:focus,
.form-group select:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
}

.form-group input.error,
.form-group select.error {
  border-color: #dc3545;
  background-color: #ffe6e6;
}

.error-message {
  color: #dc3545;
  font-size: 0.875rem;
  margin-top: 0.25rem;
  min-height: 1.25rem;
}

button {
  padding: 0.75rem 2rem;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.2s;
}

button:hover {
  background-color: #0056b3;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
```

---

## Dependencies

```html
<!-- jQuery (if using jQuery validation) -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

<!-- International Tel Input -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/intl-tel-input@17/build/css/intlTelInput.css">
<script src="https://cdn.jsdelivr.net/npm/intl-tel-input@17/build/js/intlTelInput.min.js"></script>

<!-- Cleave.js for formatting -->
<script src="https://cdn.jsdelivr.net/npm/cleave.js@1.6.0/dist/cleave.min.js"></script>

<!-- CSRF Token -->
<meta name="csrf-token" content="...">
```

---

## Key Points

1. **Reference Data Cached** - Fetch countries/states once
2. **Validation Both Ways** - Client-side UX + server-side security
3. **Dependent Fields** - US state conditional on country
4. **Phone Formatting** - E.164 format via intlTelInput
5. **Currency Formatting** - Cleave.js for annual sales
6. **Error Handling** - Field-level + form-level
7. **CSRF Protection** - Auto-refresh on 419
8. **Session Persistence** - UUID stored in localStorage

---

## Testing Checklist

- [ ] Fill all required fields
- [ ] Submit with valid data → Redirect to Step 2
- [ ] Submit with empty email → Error shown
- [ ] Submit with invalid email → Error shown
- [ ] Select US country → State dropdown appears
- [ ] Select non-US country → State hidden
- [ ] Enter phone in different format → E.164 format sent
- [ ] Enter annual sales with commas → Numeric value sent
- [ ] CSRF expires → Auto-refresh and retry
- [ ] Network error → Show friendly message

---

## See Also

- [Complete Flow Example](./complete-flow.md)
- [Sample Responses](./sample-responses.json)
- [Getting Started Guide](../guides/getting-started.md)
- [Authentication Guide](../guides/authentication.md)
