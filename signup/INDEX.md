# EPD eMerchant Signup Skills - Complete Index

**Last Updated:** 2026-07-09  
**Total Skills:** 8 (1 Main + 6 Steps + 1 Reference)  
**Total Field Documentation:** 70+ fields  
**Total Dependent Rules:** 16  
**Status:** ✅ Complete & Production Ready

---

## Skill Structure

```
skills/
└── signup/
    ├── MAIN_SIGNUP_SKILL.md              ← Start here
    ├── INDEX.md                           ← You are here
    ├── step1/
    │   └── SKILL_STEP1.md
    ├── step2/
    │   └── SKILL_STEP2.md
    ├── step3/
    │   └── SKILL_STEP3.md
    ├── step4/
    │   └── SKILL_STEP4.md
    ├── step5/
    │   └── SKILL_STEP5.md
    ├── step6/
    │   └── SKILL_STEP6.md
    └── reference/
        └── DEPENDENT_FIELDS_REFERENCE.md
```

---

## Quick Navigation

### Main Overview
📖 **[MAIN_SIGNUP_SKILL.md](./MAIN_SIGNUP_SKILL.md)** - Start here for API overview, key concepts, and integration patterns

### Individual Step Skills

| Step | File | Fields | Dependent | Complexity |
|------|------|--------|-----------|-----------|
| 1 | [step1/SKILL_STEP1.md](./step1/SKILL_STEP1.md) | 8 | 1 | Low |
| 2 | [step2/SKILL_STEP2.md](./step2/SKILL_STEP2.md) | 16 | 5 | High |
| 3 | [step3/SKILL_STEP3.md](./step3/SKILL_STEP3.md) | 12 | 2 | Moderate |
| 4 | [step4/SKILL_STEP4.md](./step4/SKILL_STEP4.md) | 24+ | 4 | High |
| 5 | [step5/SKILL_STEP5.md](./step5/SKILL_STEP5.md) | 8 | 2 | Low |
| 6 | [step6/SKILL_STEP6.md](./step6/SKILL_STEP6.md) | 12 | 2 | Low |

### Reference Documentation
🔍 **[reference/DEPENDENT_FIELDS_REFERENCE.md](./reference/DEPENDENT_FIELDS_REFERENCE.md)** - Complete map of all 16 dependent field rules across all steps

---

## What's Included

### Each Step Skill Contains:
- ✅ Detailed field documentation (name, ID, type, validation, requirements)
- ✅ Field grouping by business section
- ✅ Dependent field rules with JSON structure
- ✅ API endpoint specifications (GET/POST)
- ✅ Request/response examples
- ✅ Validation rules and error scenarios
- ✅ Integration checklist
- ✅ Common issues and solutions

### Reference Skill Contains:
- ✅ Complete dependency map (all 16 rules)
- ✅ Cross-step dependencies
- ✅ Field requirement state changes
- ✅ Dependency patterns and templates
- ✅ Testing checklist
- ✅ Common implementation issues
- ✅ Best practices

### Main Skill Contains:
- ✅ API overview and authentication
- ✅ Core concepts and data structure
- ✅ Error handling standards
- ✅ CSRF token management
- ✅ Data persistence & session management
- ✅ Universal validation rules
- ✅ Integration flow recommendations
- ✅ Best practices by category

---

## Key Features

### 1. **Comprehensive Field Documentation**
Every field includes:
- Machine-readable ID (for API reference)
- Display name (for UI labels)
- Type (text, dropdown, date, currency, etc.)
- Constraints (max length, min/max values, format)
- Validation rules (patterns, custom validators)
- Requirement status (always/conditional/optional)

### 2. **Dependent Field Rules**
All 16 conditional field rules documented with:
- Trigger field identification
- Target field identification
- Trigger value(s) that activate the rule
- Effect (visibility/requirement changes)
- Custom error messages
- Clear-on conditions
- JSON structure for implementation

### 3. **API Integration Examples**
Each step includes:
- Exact GET/POST URLs
- Request body with all fields
- Response structures for success/error
- Status codes and error messages
- Header requirements (CSRF token, auth)

### 4. **Validation Strategy**
Complete validation documentation:
- Field-level rules (length, format, patterns)
- Cross-field validation (percentages, sums)
- Dependent field conditional validation
- Server-side API calls (routing number, email uniqueness)

---

## Usage Scenarios

### Scenario 1: Building a Form Integration
1. Read [MAIN_SIGNUP_SKILL.md](./MAIN_SIGNUP_SKILL.md) for overview
2. Jump to specific step skill (e.g., [step2/SKILL_STEP2.md](./step2/SKILL_STEP2.md))
3. Reference [Dependent Fields](./reference/DEPENDENT_FIELDS_REFERENCE.md) for conditional logic
4. Use field IDs and API endpoints for integration
5. Follow validation rules and error handling

### Scenario 2: Understanding Data Flow
1. Start with [MAIN_SIGNUP_SKILL.md](./MAIN_SIGNUP_SKILL.md) - "Integration Patterns" section
2. Review Step 1 GET/POST examples in [step1/SKILL_STEP1.md](./step1/SKILL_STEP1.md)
3. Follow flow through all 6 steps
4. See completion scenarios in [step6/SKILL_STEP6.md](./step6/SKILL_STEP6.md)

### Scenario 3: Implementing Dependent Fields
1. Open [Dependent Fields Reference](./reference/DEPENDENT_FIELDS_REFERENCE.md)
2. Find relevant rule by step number
3. Reference implementation pattern
4. Check testing checklist for validation

### Scenario 4: Adding to External System
1. Extract field IDs from each step skill
2. Map to your system's field names
3. Implement dependent field rules from reference
4. Use API endpoints for data submission
5. Handle responses per specification

### Scenario 5: Error Handling & Troubleshooting
1. See "Error Scenarios" table in each step
2. Check "Common Issues" section per step
3. Reference "Validation Rules" for root cause
4. Review [Dependent Fields](./reference/DEPENDENT_FIELDS_REFERENCE.md) for conditional issues

---

## Field Summary by Step

### Step 1: Account Information
- **Fields:** 8
- **Required:** 5 + 1 conditional (state if US) + 1 variant conditional (annual sales)
- **Dependent:** 1 rule
- **Section Focus:** Owner personal info + basic company data
- **Key Field:** First/Last Name, Email, Phone, Company Name, Website, Country, [State if US], [Annual Sales if variant]

### Step 2: Company Information
- **Fields:** 16
- **Required:** 12 + 4 conditional/variant
- **Dependent:** 5 rules
- **Section Focus:** Legal structure, registration, revenue model, addresses
- **Key Fields:** Legal Name, Industry, Formation Date, Tax ID, Marketing Model, Addresses

### Step 3: Product & Transaction
- **Fields:** 12
- **Required:** 9
- **Dependent:** 2 rules
- **Section Focus:** Transaction entry methods, product details, fulfillment
- **Key Fields:** 3-way Slider (100% requirement), Product Description, Amounts, Fulfillment

### Step 4: Beneficial Owner
- **Fields:** 24+ (Owner 1 always, Owner 2 if < 51%)
- **Required:** 16 (Owner 1) + 16 (Owner 2 if shown)
- **Dependent:** 4 rules
- **Section Focus:** Personal info, financial history, addresses, driver's license
- **Key Fields:** SSN, DOB, Bankruptcy, Address, License (US), Ownership %

### Step 5: Banking & Payment
- **Fields:** 8
- **Required:** 4 (routing + account + current processing + currency if CA)
- **Dependent:** 2 rules
- **Section Focus:** Bank account, payment processing, currency
- **Key Fields:** Routing (US), Account, Processor, Currency (CA)

### Step 6: Final Details
- **Fields:** 12 (+ 0-N dynamic)
- **Required:** 4
- **Dependent:** 2 rules
- **Section Focus:** Referral, preferences, interests, terms
- **Key Fields:** How Found Us, Multiple Accounts, Bad Experience, Terms Agreement

---

## API Reference URLs

### Data Retrieval
```
GET https://emap.epd.dev/api/countries                  → Country list (Step 1, 2, 4, 5)
GET https://emap.epd.dev/api/states                     → US States list (Step 1, 2, 4)
GET https://emap.epd.dev/api/industry-types             → Industry dropdown options (Step 2)
GET https://emap.epd.dev/api/shopping-carts             → Shopping cart/CRM options (Step 3)
GET https://emap.epd.dev/api/referral-sources           → Referral source list (Step 6)
GET https://emap.epd.dev/api/pricing-templates          → Pricing templates (referenced in main skill)
GET https://emap.epd.dev/api/files-types                → File type options (referenced in main skill)
```

### Step Endpoints
```
GET  /step/{1-6}/{uuid}             → Retrieve step data with pre-filled values
POST /signup/user                   → Submit Step 1
POST /signup/companies              → Submit Steps 2, 3, 5, 6
POST /signup/handleOwnership        → Submit Step 4
```

### Validation Endpoints
```
POST /signup/validateRoutingNumber  → Validate US routing number (Step 5)
POST /signup/checkEmail             → Check email uniqueness (Step 4)
POST /signup/auto-save              → Auto-save from lander (Step 1)
```

---

## Implementation Checklist

### Before Building
- [ ] Review MAIN_SIGNUP_SKILL.md for overview
- [ ] Understand Step 1-6 flow from individual skills
- [ ] Study Dependent Fields Reference for conditional logic
- [ ] Extract all field IDs for mapping
- [ ] Plan API integration approach

### During Development
- [ ] Fetch reference data (countries, states, industries)
- [ ] Implement validation rules per step
- [ ] Set up dependent field rules (16 total)
- [ ] Configure CSRF token handling
- [ ] Implement error display/handling
- [ ] Test each step independently
- [ ] Test all field validations
- [ ] Verify all dependent fields (16 rules)

### Quality Assurance
- [ ] Test error scenarios from each step
- [ ] Verify API response handling
- [ ] Test cross-step data flow
- [ ] Validate CSRF token refresh on 419
- [ ] Verify redirect flows

### Post-Deployment
- [ ] Monitor error logs for validation failures
- [ ] Track user drop-off by step
- [ ] Verify dependent field behavior in production
- [ ] Monitor API response times
- [ ] Collect feedback on field clarity

---

## Version Control & Updates

**Current Version:** 1.0.0  
**Last Updated:** 2026-07-09

### What's Documented:
- ✅ Steps 1-6 (complete)
- ✅ Dependent fields (all 16 rules)
- ✅ API endpoints & examples
- ✅ Validation rules
- ✅ Error handling

### What's Excluded (by design):
- ❌ Plaid/Persona verification
- ❌ Signature step (Step 7)
- ❌ Dynamic step logic (rule engine internals)
- ❌ Dashboard/merchant portal
- ❌ Document extraction/Textract

---

## Support & Questions

For technical questions or clarifications:
- Step-specific issues → Check relevant step skill's "Common Issues" section
- Dependent field questions → Check [Dependent Fields Reference](./reference/DEPENDENT_FIELDS_REFERENCE.md)
- API integration questions → Check MAIN_SIGNUP_SKILL.md or individual step API sections
- Validation questions → Check "Validation Rules" section in each step skill

---

## Quick Reference: Common Dependent Fields

| Trigger | Target | Condition |
|---------|--------|-----------|
| Country = 1 (US) | State | Make visible + required |
| Industry = 50 (Other) | Industry Other | Make visible + required |
| Marketing includes 2 (Recurring) | Subscription Frequency | Make visible + required |
| Subscription Freq = 3 (Other) | Subscription Other | Make visible + required |
| Fulfillment in [2,3] | Vendor Name | Make visible + required |
| Shopping Cart in [8,58] | Shopping Cart Other | Make visible + required |
| Ownership % < 51 | Owner 2 Section | Show entire section + require fields |
| Current Processing = 1 | Processor Name | Make visible + required |
| Bad Experience = 1 | Bad Provider Name | Make visible + required |
| Referral in [50,14,8] | Additional Info | Make visible + required |
| Owner Country = 1 | License State + Expiration | Make visible + required (both) |
| Canada (country=2) | Institution # + Currency | Make visible + required |

---

## Next Steps for Users

1. **First Time?** → Read [MAIN_SIGNUP_SKILL.md](./MAIN_SIGNUP_SKILL.md)
2. **Building Step 1?** → Go to [step1/SKILL_STEP1.md](./step1/SKILL_STEP1.md)
3. **Understanding Dependencies?** → Check [Dependent Fields Reference](./reference/DEPENDENT_FIELDS_REFERENCE.md)
4. **Need Field IDs?** → Search step skills for "Field ID" column
5. **Troubleshooting?** → Find your step skill and check "Common Issues" section

---

## Skill Maintenance

These skills are maintained as the source of truth for signup integration. When implementing in external systems:
- Always reference the current version
- Cross-check field IDs and names
- Verify API endpoints match production
- Test all dependent field rules
- Monitor for any changes to reference data structure

**Tip:** Bookmark this INDEX.md for quick access to all skills.
