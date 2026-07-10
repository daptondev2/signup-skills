# EPD eMerchant Signup API Skills

**Version:** 1.0.0  
**Last Updated:** 2026-07-09  
**API Base URL:** `/api/v1`  
**Authentication:** Bearer Token Required  
**Rate Limit:** 100 requests/minute  
**Response Format:** JSON

---

## Overview

The EPD eMerchant Signup API provides a comprehensive multi-step merchant onboarding workflow. This skill set enables external systems to programmatically manage the complete signup process with built-in validation, dependent field management, and real-time data population.


---

## Core Concepts

### Step Structure
Each signup step follows a consistent pattern:
1. **GET** - Retrieve step data with pre-filled values and dropdown options
2. **POST** - Submit step data with validation
3. **Auto-save** - Real-time field persistence (Step 1 only)
4. **Dependent Fields** - Conditional field visibility based on parent field values

### Key Endpoints

| Step | GET Endpoint | POST Endpoint | Fields Count |
|------|--------------|---------------|--------------|
| 1 | `/signup/step/1/{uuid}` | `/signup/user` | 8 |
| 2 | `/signup/step/2/{uuid}` | `/signup/companies` | 16 |
| 3 | `/signup/step/3/{uuid}` | `/signup/companies` | 12 |
| 4 | `/signup/step/4/{uuid}` | `/signup/handleOwnership` | 24+ |
| 5 | `/signup/step/5/{uuid}` | `/signup/companies` | 8 |
| 6 | `/signup/step/6/{uuid}` | `/signup/companies` | 12 |

---

## Reference Data Routes

All dropdown values can be fetched from the Partner API Integration endpoints:

```
GET https://emap.epd.dev/api/countries
GET https://emap.epd.dev/api/states
GET https://emap.epd.dev/api/industry-types
GET https://emap.epd.dev/api/shopping-carts
GET https://emap.epd.dev/api/referral-sources
GET https://emap.epd.dev/api/files-types
GET https://emap.epd.dev/api/pricing-templates
```

**Base URL:** `https://emap.epd.dev`

---

## Error Handling

### Standard Error Response
```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "field_name": ["Error message"]
  }
}
```

### HTTP Status Codes
- `200` - Success
- `400` - Validation error
- `401` - Unauthorized
- `419` - CSRF token expired
- `422` - Unprocessable entity
- `500` - Server error

---

## CSRF Token Management

- All POST requests require `X-CSRF-TOKEN` header
- Token available in meta tag: `<meta name="csrf-token" content="...">`
- Token expiration: 24 hours
- Expired token returns `419` status

---

## Sub-Skills Reference

| Skill | Purpose | Fields | Dependent Fields |
|-------|---------|--------|------------------|
| [Step 1 - Account Info](./step1/SKILL_STEP1.md) | Owner & company basic info | 8 | 1 |
| [Step 2 - Company Info](./step2/SKILL_STEP2.md) | Detailed company details | 16 | 5 |
| [Step 3 - Product Info](./step3/SKILL_STEP3.md) | Product & transaction details | 12 | 2 |
| [Step 4 - Owner Info](./step4/SKILL_STEP4.md) | Beneficial owner personal data | 24+ | 4 |
| [Step 5 - Banking Info](./step5/SKILL_STEP5.md) | Bank account & processing | 8 | 2 |
| [Step 6 - Final Details](./step6/SKILL_STEP6.md) | Referral & preferences | 12 | 2 |
| [Dependent Fields Reference](./reference/DEPENDENT_FIELDS_REFERENCE.md) | All conditional field rules | N/A | 16 Total |

---

## Data Persistence

### Session Management
- Session timeout: 30 minutes of inactivity
- Auto-save available on Step 1 from lander flows
- Local storage caching for form recovery
- Data persisted per application UUID

### UUID Tracking
Each application gets a unique identifier:
```
applicationInfo->uuid (format: 8-4-4-4-12)
```
Use UUID for all step navigation and data retrieval.

---

## Validation Rules

### Universal Rules
- All dates: `YYYY-MM-DD` format
- Phone numbers: International format (intlTelInput validation)
- Email: RFC 5322 compliant regex
- URLs: Must include protocol (http:// or https://)
- Percentages: 1-100, integers only
- Currency: Integers, thousands grouping supported

### Step-Specific Validation
See individual step skills for field-level rules.

---

## Integration Patterns

### Recommended Flow
```
1. Initialize application (GET /signup/step/1/{uuid})
2. Iterate through steps 1-6
3. For each step:
   - Fetch reference data (countries, industries, etc.)
   - Apply dependent field rules
   - Validate before submission
   - Handle errors and retry logic
4. On completion, redirect to dashboard
```

### Handling Dependent Fields
1. Fetch dependent field mapping (see reference skill)
2. Track parent field changes in real-time
3. Update child field visibility/requirement dynamically
4. Re-validate on parent field change
5. Clear conditional fields when parent value changes

---

## Best Practices

1. **Validation Strategy**
   - Validate on blur for better UX
   - Re-validate on dependent field changes
   - Show field-level error messages
   - Group related errors by section

2. **Error Recovery**
   - Auto-save progress frequently
   - Provide clear error messages
   - Offer retry mechanism
   - Don't lose user input on validation failure

3. **Security**
   - Always include CSRF token
   - Validate all inputs server-side
   - Sanitize user input for XSS prevention
   - Use HTTPS for all requests

4. **Performance**
   - Cache reference data (24-hour TTL)
   - Batch dependent field updates
   - Implement request debouncing
   - Use CDN for static assets

5. **Accessibility**
   - Label all form fields
   - Provide error feedback via aria-live
   - Ensure keyboard navigation
   - Support screen readers

---

## Support & Documentation

- **API Documentation:** See individual step skills
- **Error Codes:** Included in each response
- **Rate Limiting:** X-RateLimit-* headers in responses
- **Support Email:** api-support@epd-epos.com

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-07-09 | Initial release with Steps 1-6 |

---

**Note:** Plaid integration and Persona verification steps are handled separately and not included in this skill set.
