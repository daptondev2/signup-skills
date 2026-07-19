# Signup Skill - File Structure

## Main Skill File

**`skill.md`** - Start here! Complete implementation guide (325 lines)
- Covers all 6 steps
- All field definitions
- API endpoints with correct headers
- Dropdown integration with correct Authorization header
- Form submission examples
- Error handling
- Complete checklist

---

## Reference Files (in `reference/` folder)

Use these for detailed examples and patterns:

- **`ENDPOINTS.md`** - All 9 endpoints documented:
  - 3 form submission endpoints (signup, application/step, ownership)
  - 6 dropdown data endpoints
  - Full request/response examples
  - Headers and authentication

- **`form-submission.md`** - Step-by-step submission guide:
  - Detailed examples for each step
  - JavaScript implementation code
  - Error handling patterns
  - Complete flow walkthrough

- **`api-integration.md`** - Dropdown integration patterns:
  - How to fetch dropdown data
  - Caching strategies
  - Dependent field logic
  - Real-world implementation examples

---

## Configuration Files

- **`specification.json`** - Machine-readable field definitions:
  - All 59 fields with validation rules
  - API endpoint mappings
  - Field dependencies
  - Response formats

---

## Critical Implementation Details

### Authentication Headers (Most Common Error)

**Two Different Headers - Don't Mix Them!**

| Use Case | Header | Value |
|----------|--------|-------|
| Form submissions (signup, step, ownership) | `X-API-Key` | user.security_key |
| Dropdown data (countries, states, etc.) | `Authorization` | user.security_key |

### Why Dropdowns Fail

If dropdowns show empty:
1. Check if using `X-API-Key` instead of `Authorization` ❌
2. Fix: Use `Authorization` header ✅
3. Response will be 403 with message "The API key provided is not valid."

### Why Form Submission Fails

If getting "API key is required in X-API-Key header":
1. Check if Authorization header instead of `X-API-Key` ❌
2. Fix: Use `X-API-Key` header ✅

---

## How to Use This Skill

1. Read `skill.md` - 5 minutes to understand the full structure
2. Copy code examples from reference files
3. Test dropdowns first (most common issue)
4. Then test form submissions
5. Finally test full end-to-end flow

---

## Common Mistakes (Checklist)

- [ ] ❌ Sending ALL form fields at Step 1 (send only 9)
- [ ] ❌ Using X-API-Key for dropdowns (use Authorization)
- [ ] ❌ Using Authorization for form submission (use X-API-Key)
- [ ] ❌ Not showing business_state only for country=US
- [ ] ❌ Not showing Owner 2 only when Owner 1 < 50%
- [ ] ❌ Not showing bad_experience_happened only when bad_experience=1
- [ ] ❌ Forgetting to store UUID between steps
- [ ] ❌ Not validating before submission

---

## End-to-End Validation

Before declaring the form complete:

- [ ] User can log in (gets security_key)
- [ ] Dropdowns load on Step 1 (countries, states)
- [ ] Conditional fields show/hide correctly
- [ ] Form validates all required fields
- [ ] Step 1 submits successfully (gets UUID)
- [ ] UUID persists to Step 2
- [ ] Each subsequent step submits only step data
- [ ] All error messages display correctly
- [ ] Final step redirects to success page

---

**Last Updated**: 2026-07-19  
**Status**: Production Ready ✅
