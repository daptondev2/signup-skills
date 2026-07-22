---
name: signup-form-creation
description: Create a complete 7-step merchant signup form with all fields, validations, and integrations
---

# Merchant Signup Form Specification

Production-ready specification for building the complete 7-step signup form.

---

## Form Overview

| Step | Title | File | Fields | Features |
|------|-------|------|--------|----------|
| 1 | Account Information | [STEP1_ACCOUNT_INFORMATION.md](steps/STEP1_ACCOUNT_INFORMATION.md) | 9 | Phone formatting, country/state selection |
| 2 | Company Information | [STEP2_COMPANY_INFORMATION.md](steps/STEP2_COMPANY_INFORMATION.md) | 28 | Google Maps autocomplete (legal + physical), revenue model with nested conditionals |
| 3 | Product Information | [STEP3_PRODUCT_INFORMATION.md](steps/STEP3_PRODUCT_INFORMATION.md) | 13 | Fulfillment details, transaction slider (multiples of 5) |
| 4 | Owner Information | [STEP4_OWNER_INFORMATION.md](steps/STEP4_OWNER_INFORMATION.md) | 45+ | Primary contact, Owner 1 & conditional Owner 2, Google Maps autocomplete (2), driver's license, financial history |
| 5 | Banking Information | [STEP5_BANKING_INFORMATION.md](steps/STEP5_BANKING_INFORMATION.md) | 10 | Routing validation, country-specific fields |
| 6 | Final Details | [STEP6_FINAL_DETAILS.md](steps/STEP6_FINAL_DETAILS.md) | 11 | Interest selection, terms acceptance |
| 7 | Signature | - | - | Display only (no submission) |

---

## Quick Reference Files

**Supporting Documentation**:
- [reference/DEPENDENT_FIELDS.md](reference/DEPENDENT_FIELDS.md) - Complete conditional field logic
- [reference/DROPDOWNS_REFERENCE.md](reference/DROPDOWNS_REFERENCE.md) - All dropdown values (API + static)
- [reference/api-examples.md](reference/api-examples.md) - Copy-paste JavaScript & cURL code
- [GETTING_STARTED.md](GETTING_STARTED.md) - Quick start guide for developers
- [INDEX.md](INDEX.md) - File index and navigation

---

## Global Implementation

### Authentication Headers (CRITICAL)

**Form Submission (POST)**:
```javascript
headers: {
    'Content-Type': 'application/json',
    'X-API-Key': user.security_key,
    'Accept': 'application/json'
}
```

**Dropdown Data (GET)**:
```javascript
headers: {
    'Authorization': user.security_key,
    'Content-Type': 'application/json'
}
```

---

### State Management

```javascript
// After Step 1, store UUID
localStorage.setItem('signup_uuid', response.uuid);

// Retrieve for subsequent steps
const uuid = localStorage.getItem('signup_uuid');

// Clear on completion
localStorage.removeItem('signup_uuid');
```

---

### Form Submission Pattern

1. Collect form data from all visible fields
2. Validate client-side
3. Filter: Remove hidden/disabled fields
4. Add step_count, section, uuid
5. POST to API
6. Navigate to next step or dashboard

---

### Required Libraries

```html
<!-- All Steps -->
<script src="/assets/js/dependent-fields.js"></script>

<!-- Step 1 -->
<script src="https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/js/intl-tel-input.min.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/intl-tel-input@22.0.2/build/css/intlTelInput.css">

<!-- Step 2, 4 -->
<script src="https://maps.google.com/maps/api/js?key={GOOGLE_API_KEY}&libraries=places"></script>

<!-- Step 3 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/ion-rangeslider/2.3.1/ion.rangeSlider.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/ion-rangeslider/2.3.1/css/ion.rangeSlider.min.css">

<!-- Date Picker (Steps 2, 4, 5) -->
<!-- Include your date picker library -->

<!-- Form Validation -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.5/jquery.validate.min.js"></script>
```

---

### Error Handling

```javascript
// API Response Status Codes
201/200: Success - proceed to next step
400: Bad request - check payload format
401/403: Auth error - check headers
422: Validation error - display field errors
500: Server error - retry or report
```

---

## Field Summary

**Total Fields**: 116+ (9+28+13+45+10+11 across all 6 steps)  
**Required Fields**: 70+  
**Conditional Fields**: 40+ (multi-level dependencies)  
**Google Maps Autocomplete Instances**: 4  
  - Step 2: Legal address + Physical address
  - Step 4: Owner 1 home address + Owner 2 home address  
**API Dropdowns**: 8  
**Static Dropdowns**: 12  
**Phone Fields**: 4 (intl-tel-input)  
**Date Fields**: 8  
**Radio/Checkbox Fields**: 12+  
**Masked/Encrypted Fields**: SSN (Owner 1 & 2), Federal Tax ID  
**Validations**: 40+ business logic rules  
**Multi-Level Conditionals**: Revenue Model (Step 2), Bankruptcy (Steps 4)

---

## Developer Quick Start

1. **Begin with overview**: This file (skill.md) for form structure
2. **Check step details**: Go to the specific step file in `steps/` folder
3. **Reference conditionals**: See [DEPENDENT_FIELDS.md](reference/DEPENDENT_FIELDS.md)
4. **Copy code examples**: Use [reference/api-examples.md](reference/api-examples.md)
5. **Check dropdown values**: Reference [reference/DROPDOWNS_REFERENCE.md](reference/DROPDOWNS_REFERENCE.md)

---

**Production Ready** ✅  
**Last Updated**: 2026-07-22  
**Documentation Structure**: Main index + 6 step-specific files + 3 reference files  
**Lines per file**: All under 500 lines (follows skill best practices)
