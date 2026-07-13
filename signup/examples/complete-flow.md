# Complete 6-Step Flow Example

End-to-end workflow for entire signup process.

---

## Overview

This example shows the complete flow from Step 1 to Step 6, handling:
- Session management (UUID)
- Data persistence
- Dependent fields
- Error recovery
- Completion redirects

---

## Step-by-Step Flow

### Step 1: Account Information

```javascript
// User fills form with:
const step1Data = {
  first_name: 'John',
  last_name: 'Doe',
  email: 'john@example.com',
  phone: '+1-555-123-4567',
  name: 'Acme Corp',
  website: 'https://acme.com',
  business_formed_in: '1',
  business_state: 'CA',
  annual_cc_sales: '500000',
  step_count: 1
};

// POST to /signup/user
const response1 = await fetch('https://emap.epd.dev/signup/user', {
  method: 'POST',
  headers: { ... },
  body: new URLSearchParams(step1Data)
});

// Response
{
  "message": "Success",
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "url": "/step/2/550e8400-e29b-41d4-a716-446655440000",
  "companyExist": false
}

// Store UUID
const uuid = '550e8400-e29b-41d4-a716-446655440000';
localStorage.setItem('signup_uuid', uuid);
```

### Step 2: Company Details

```javascript
// Load pre-filled Step 2 data
const step2PreFilled = await fetch(
  `https://emap.epd.dev/step/2/${uuid}`,
  { headers: { 'Authorization': `Bearer ${token}` } }
).then(r => r.json());

// User fills form with:
const step2Data = {
  name: 'Acme Corp',
  industry_type: '1',
  country: '1',
  business_location_type: '1',
  business_formed: '2020-05-15',
  business_structure: '2',
  federal_tax_id: '12-34-56789',
  customer_service_telephone_number: '+1-555-987-6543',
  marketingModel: ['1'],
  
  // Address
  street_number: '123',
  street_address: 'Business Ave',
  city: 'San Francisco',
  state: 'CA',
  postal_code: '94105',
  address_country: '1',
  is_physical_address_same_as_legal_address: '1',
  
  step_count: 2,
  section: 'company_info',
  uuid: uuid
};

// POST to /signup/companies
const response2 = await fetch('https://emap.epd.dev/signup/companies', {
  method: 'POST',
  headers: { ... },
  body: new URLSearchParams(step2Data)
});

// Response
{
  "message": "Saved successfully",
  "redirect": "/step/3/{uuid}"
}
```

### Step 3: Product Information

```javascript
// User fills slider (3-way distribution)
// Must sum to 100%
const step3Data = {
  card_swiped: '40',
  customer_entered: '35',
  staff_entered: '25',
  
  describe_highest_transaction: 'Premium consulting packages',
  describe_services: '1',
  highest_transaction_amount: '50000',
  average_transaction_amount: '5000',
  
  customer_service_time: '1',
  refund_policy: '1',
  fulfillment_by: '1',  // Triggers vendor name if 2 or 3
  
  shopping_cart: '1',  // Triggers other if 8 or 58
  leave_deposit: '1',
  
  step_count: 3,
  section: 'product_info',
  uuid: uuid
};

const response3 = await fetch('https://emap.epd.dev/signup/companies', {
  method: 'POST',
  headers: { ... },
  body: new URLSearchParams(step3Data)
});

// Response
{
  "message": "Saved successfully",
  "redirect": "/step/4/{uuid}"
}
```

### Step 4: Beneficial Owners

```javascript
// Owner 1 (always required)
const owner1Data = {
  'owner[1][first_name]': 'John',
  'owner[1][last_name]': 'Doe',
  'owner[1][email]': 'john@example.com',
  'owner[1][phone]': '+1-555-123-4567',
  'owner[1][job_title]': '1',
  'owner[1][ssn]': '123-45-6789',
  'owner[1][ownership_percentage]': '100',
  'owner[1][dob]': '1980-05-15',
  'owner[1][bankruptcy_filed]': '0',
  
  // Address
  'owner[1][street_number]': '123',
  'owner[1][street_address]': 'Main St',
  'owner[1][city]': 'San Francisco',
  'owner[1][state]': 'CA',
  'owner[1][postal_code]': '94105',
  'owner[1][country]': '1',
  
  // License (if US)
  'owner[1][driver_license_state_input]': 'CA',
  'owner[1][document_number]': 'D1234567',
  'owner[1][expiration_date]': '2028-05-15'
};

// If ownership < 51%, add Owner 2
// owner[2][first_name], owner[2][last_name], etc.

const step4Data = {
  ...owner1Data,
  step_count: 4,
  uuid: uuid
};

// POST to /signup/handleOwnership
const response4 = await fetch('https://emap.epd.dev/signup/handleOwnership', {
  method: 'POST',
  headers: { ... },
  body: new URLSearchParams(step4Data)
});

// Response
{
  "message": "Saved successfully",
  "redirect": "/step/5/{uuid}"
}
```

### Step 5: Banking Information

```javascript
const step5Data = {
  bank_routing_number: '123456789',  // US only
  bank_account_number: '98765432',
  bank_name: 'Chase Bank',
  
  current_processing: '1',  // Triggers processor name if 1
  processor_name: 'Square',
  
  // Canada only
  institution_number: '',  // If Canada
  customer_pay_currency: 'USD',  // If Canada
  
  step_count: 5,
  section: 'product_info',
  is_plaid: '0',
  uuid: uuid
};

const response5 = await fetch('https://emap.epd.dev/signup/companies', {
  method: 'POST',
  headers: { ... },
  body: new URLSearchParams(step5Data)
});

// Response
{
  "message": "Saved successfully",
  "redirect": "/step/6/{uuid}"
}
```

### Step 6: Final Details & Preferences

```javascript
const step6Data = {
  howdidyouhear: '1',  // Triggers other if 50, 14, or 8
  hear_about_us_other: '',
  
  multiple_merchant_accounts: '0',
  transaction_device: '1',
  bad_experience: '0',  // Triggers provider name if 1
  bad_experience_happened: '',
  
  other_interests_capital: ['1', '2'],  // Optional checkbox array
  terms_and_conditions_agreed: 'on',  // Required
  
  step_count: 6,
  section: 'interest_details',
  uuid: uuid
};

const response6 = await fetch('https://emap.epd.dev/signup/companies', {
  method: 'POST',
  headers: { ... },
  body: new URLSearchParams(step6Data)
});

// Response: Option 1 - No Dynamic Fields
{
  "message": "Application submitted successfully",
  "redirect": "/dashboard/merchant?landing=1"
}

```

### Step 6b: Final Redirect

```javascript
// After Step 6 submission, redirect to dashboard
window.location.href = '/dashboard/merchant?landing=1';
```

---

## Error Handling During Flow

```javascript
async function submitStep(stepNumber, data) {
  const endpoints = {
    1: '/signup/user',
    2: '/signup/companies',
    3: '/signup/companies',
    4: '/signup/handleOwnership',
    5: '/signup/companies',
    6: '/signup/companies'
  };

  const endpoint = endpoints[stepNumber];
  const maxRetries = 3;
  let retryCount = 0;

  while (retryCount < maxRetries) {
    try {
      const response = await fetch(
        `https://emap.epd.dev${endpoint}`,
        {
          method: 'POST',
          headers: {
            'Authorization': `Bearer ${token}`,
            'X-CSRF-TOKEN': csrfToken
          },
          body: new URLSearchParams(data)
        }
      );

      const result = await response.json();

      if (response.ok) {
        return { success: true, data: result };
      }

      // Handle errors
      if (response.status === 400) {
        // Validation error - show to user
        return { success: false, errors: result.errors };
      }

      if (response.status === 419) {
        // CSRF expired - refresh and retry
        csrfToken = document.querySelector('meta[name="csrf-token"]').content;
        retryCount++;
        continue;
      }

      if (response.status === 429) {
        // Rate limited - wait and retry
        const waitSeconds = result.retry_after || Math.pow(2, retryCount);
        await new Promise(r => setTimeout(r, waitSeconds * 1000));
        retryCount++;
        continue;
      }

      if (response.status >= 500) {
        // Server error - retry with backoff
        await new Promise(r => setTimeout(r, Math.pow(2, retryCount) * 1000));
        retryCount++;
        continue;
      }

      return { success: false, error: result.message };

    } catch (error) {
      console.error(`Step ${stepNumber} error:`, error);
      retryCount++;
    }
  }

  return { success: false, error: 'Max retries exceeded' };
}
```

---

## Data Persistence

```javascript
// Save to localStorage between page refreshes
function saveStepData(stepNumber, data) {
  const allData = JSON.parse(localStorage.getItem('signup_data') || '{}');
  allData[`step${stepNumber}`] = data;
  localStorage.setItem('signup_data', JSON.stringify(allData));
}

// Retrieve pre-filled data
function loadStepData(stepNumber) {
  const allData = JSON.parse(localStorage.getItem('signup_data') || '{}');
  return allData[`step${stepNumber}`] || {};
}

// Clear on completion
function clearSignupData() {
  localStorage.removeItem('signup_data');
  localStorage.removeItem('signup_uuid');
}
```

---

## Complete Workflow Summary

```
Step 1: Account Info
  ↓ (uuid generated)
Step 2: Company Details
  ↓
Step 3: Product Info
  ↓
Step 4: Beneficial Owners
  ↓
Step 5: Banking Info
  ↓
Step 6: Final Details
  ├─ No Dynamic Fields → Dashboard
  └─ Has Dynamic Fields → Dynamic Modal → Dashboard
  
Total Time: ~25-35 minutes
```

---

## See Also

- [Step 1 Example](./step1-example.md)
- [Sample Responses](./sample-responses.json)
- [Error Handling Guide](../guides/error-handling.md)
- [Dependent Fields Reference](../reference/dependent-fields.md)
