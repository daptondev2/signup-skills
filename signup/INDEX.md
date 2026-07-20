# Signup Form Skill - Quick Reference

Production-ready skill for building the complete 7-step merchant signup form.

---

## 📚 Documentation Files

### Main Specification
- **skill.md** - Complete form specification (all 73+ fields, validations, APIs)
- **STEP4_OWNER_INFORMATION.md** - Detailed Step 4 specification (43 owner fields, conditionals)

### Implementation Guides
- **reference/DEPENDENT_FIELDS.md** - All conditional/dependent field logic with code examples
- **reference/DROPDOWNS_REFERENCE.md** - All dropdown values (API + static) 
- **reference/api-examples.md** - Copy-paste JavaScript & cURL code
- **reference/ENDPOINTS.md** - API endpoints, headers, payloads, error codes

---

## 🚀 Quick Start

### For Developers
1. Read `skill.md` - Understand all fields and requirements
2. Read `reference/DEPENDENT_FIELDS.md` - Implement conditional logic
3. Copy code from `reference/api-examples.md` - Use working implementations
4. Reference `reference/ENDPOINTS.md` - Verify API calls

### For QA/Testing
1. Read `skill.md` → Testing Checklist section
2. Reference `reference/DEPENDENT_FIELDS.md` - Test all conditionals
3. Use `reference/ENDPOINTS.md` - Verify error handling

---

## 📋 What's Included

✅ **All 7 Steps** - Account → Company → Product → Owners → Banking → Final → Signature

✅ **73+ Fields** - Complete field specifications with validation rules

✅ **Dependent Fields** - 15+ conditional show/hide rules with implementation code

✅ **API Integration** - 9 endpoints (3 form submission + 6 dropdown fetch)

✅ **Dropdown Values** - 70+ static options mapped

✅ **Code Examples** - JavaScript & cURL ready to use

✅ **Error Handling** - All HTTP status codes documented

---

## 📊 Form Structure

| Step | Fields | Key Features |
|------|--------|--------------|
| 1 | 9 | Phone formatting, country/state selection |
| 2 | 23 | Address autocomplete, conditional physical address |
| 3 | 13 | Fulfillment, transaction slider (step: 5) |
| 4 | 32+ | Owners (1 required, 2 conditional), addresses |
| 5 | 10 | Routing validation, country-specific fields |
| 6 | 11 | Interest selection, terms acceptance |
| 7 | - | Display only (no submission) |

---

## 🔑 Critical Information

**Two Different Authentication Headers** (Don't mix them up!):
```
Form Submission (POST):  X-API-Key: {user.security_key}
Dropdown Data (GET):     Authorization: {user.security_key}
```

**Country ID for Conditionals** (Not country code!):
```
USA = "1"
Canada = "2"
Mexico = "3"
```

**Required Libraries**:
- intl-tel-input (phones)
- Google Maps Places (addresses)
- ion-rangeslider (slider with step: 5)
- dependent-fields.js (conditionals)
- Date picker library

---

**Status**: Production Ready ✅  
**Last Updated**: 2026-07-19  
**Total Documentation**: 5 files, ~4000 lines
