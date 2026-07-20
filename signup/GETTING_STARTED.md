# Getting Started - Signup Form Skill

**Quick start guide for using this skill to generate complete signup forms.**

---

## 📋 For Developers: Where to Start

### Day 1: Understanding the Requirements

1. **Read** `AUDIT_SUMMARY.md` (10 min)
   - Get overview of what's documented and what's broken

2. **Read** `skill.md` Quick Overview section (5 min)
   - Understand form structure: 7 steps, 73+ fields

3. **Reference** `ENDPOINTS.md` (15 min)
   - Understand API endpoints, headers, payloads
   - Note: Two different authentication headers!

### Day 2: Understanding Implementation Details

1. **Read** `skill.md` Step 1-3 sections (30 min)
   - Understand each field, validation, conditional logic
   - Note the critical issues marked with ⚠️

2. **Read** `DROPDOWNS_REFERENCE.md` (15 min)
   - See all dropdown values you need
   - Understand static vs API-based dropdowns

3. **Review** `reference/BUGS_FOUND.md` (20 min)
   - Understand 5 critical issues blocking completion
   - Learn root causes

### Day 3: Building Forms

1. **Copy** code templates from `reference/api-examples.md`
   - JavaScript fetch examples
   - cURL examples for testing
   - Error handling patterns

2. **Reference** `skill.md` for each step
   - Exact field names, validation rules
   - Conditional display logic
   - Submission endpoints

3. **Follow** testing checklist in `skill.md`
   - Verify each step works
   - Check conditionals trigger correctly
   - Validate API payloads

### Day 4+: Fixing Known Issues

1. **Priority 1 Issues** (MUST FIX TODAY)
   - Step 1: State dropdown conditional
   - Step 2: Google Maps autocomplete
   - Step 3: Slider step validation
   - Step 4: View data passing

2. **Priority 2 Issues** (FIX THIS WEEK)
   - Value conversions
   - Conditional verification
   - Field filtering

3. **Priority 3 Issues** (NICE TO HAVE)
   - Enhanced UX features
   - Auto-save
   - Progress indicators

---

## 📱 For Product Managers: What You Need to Know

### Features
- **7-step progressive disclosure form** - User fills one step at a time
- **Multi-library integration** - Google Maps, phone formatting, sliders, signatures
- **Conditional fields** - Show/hide based on selections
- **Auto-population** - Pre-fill from external sources
- **Two-phase API** - Different headers for different calls
- **Data persistence** - localStorage between page refreshes

### Current Status
- ✅ 73+ fields fully documented
- ✅ 9 integrations documented
- ⚠️ 5 critical bugs preventing full functionality
- ✅ All API endpoints documented
- ✅ Complete testing checklist ready

### Estimated Fix Time
- **Critical Issues**: 2-3 days
- **Medium Issues**: 1 week
- **Full Testing**: 1-2 weeks
- **Production Ready**: 3 weeks

### Key Metrics
- **Form Completion Rate**: Currently 40% (because of bugs)
- **Target**: 85%+ (after fixes)
- **Field Count**: 73+ across 7 steps
- **Validation Rules**: 30+ business logic rules
- **API Calls**: 3 form submission + 6 dropdown fetch = 9 total

---

## 🔧 For QA: Testing Framework

### Test Order

**Phase 1: Individual Step Testing** (Use testing checklist)
```
✓ Step 1: 15 tests - Account info
✓ Step 2: 25 tests - Company info  
✓ Step 3: 18 tests - Product info
✓ Step 4: 18 tests - Owner info
✓ Step 5: 12 tests - Banking info
✓ Step 6: 12 tests - Final details
✓ Step 7: 4 tests - Signature
```

**Phase 2: Conditional Field Testing** (Use BUGS_FOUND.md)
```
✓ State dropdown (Step 1)
✓ Physical address (Step 2)
✓ Fulfillment vendor (Step 3)
✓ Shopping cart other (Step 3)
✓ Owner 2 section (Step 4)
✓ Country-specific fields (Step 5)
✓ Conditional text fields (Step 6)
```

**Phase 3: API Integration Testing** (Use ENDPOINTS.md)
```
✓ Dropdown data loads (6 GET calls)
✓ Form submission succeeds (3 POST calls)
✓ UUID persists correctly
✓ Error handling works (422, 401, 403, 500)
✓ Validation messages display
```

**Phase 4: End-to-End Testing**
```
✓ Complete form with all fields
✓ Submit all 7 steps
✓ Verify database records created
✓ Check email sent
✓ Confirm redirect to dashboard
```

### Critical Tests

**MUST PASS**:
1. [ ] State appears when US selected (Step 1)
2. [ ] Google Maps autocomplete works (Step 2)
3. [ ] Slider snaps to 5% increments (Step 3)
4. [ ] Owner 2 appears when < 51% (Step 4)
5. [ ] Form completes all 7 steps

---

## 🚀 Quick Reference: Most Important Things

### The Two Headers (DON'T GET WRONG!)

```javascript
// FOR FORM SUBMISSION (POST)
headers: {
    'X-API-Key': user.security_key  // ← X-API-Key
}

// FOR DROPDOWN DATA (GET)
headers: {
    'Authorization': user.security_key  // ← Authorization (different!)
}
```

### The Country/State Logic

```javascript
// Country IDs:
1 = United States (show state dropdown)
2 = Canada (hide state dropdown)
3+ = Other countries (hide state dropdown)

// When to show state:
if (country_id === "1") {
    show(".usa_state");  // Show state selector
}
```

### The Physical Address Logic

```
Radio value "no" = Physical address is SAME as legal
  → Do NOT show physical address fields
  → Do NOT submit physical address data
  → Submit: is_physical_address_same_as_legal_address: 1

Radio value "yes" = Physical address is DIFFERENT from legal
  → DO show physical address fields
  → DO submit physical address data
  → Submit: is_physical_address_same_as_legal_address: 0
```

### The Slider Rules

```javascript
// Slider MUST:
- Only allow values: 0, 5, 10, 15... 100
- Sum all three to exactly 100%
- Have step: 5 configured
- Real-time percentage display

// Three fields:
card_swiped (point of sale)
customer_entered (online)
staff_entered (manually)
```

### The Owner 2 Conditional

```javascript
// Show Owner 2 if:
ownership_percentage_of_owner_1 < 51

// Hide Owner 2 if:
ownership_percentage_of_owner_1 >= 51

// When visible:
All fields required
Must fill name, email, phone, etc.
```

---

## 📚 File Guide: Where to Find What

| What I Need | Where to Look |
|---|---|
| **Full specification** | `skill.md` (1700 lines, everything) |
| **Known bugs** | `BUGS_FOUND.md` (5 critical issues) |
| **API endpoints** | `ENDPOINTS.md` (3 POST + 6 GET) |
| **Code examples** | `api-examples.md` (JavaScript + cURL) |
| **Dropdown values** | `DROPDOWNS_REFERENCE.md` (all 70+ options) |
| **Testing checklist** | `skill.md` → Testing Checklist section |
| **Common problems** | `skill.md` → Common Issues & Solutions |
| **How to get started** | This file (GETTING_STARTED.md) |
| **Overall status** | `AUDIT_SUMMARY.md` (what's done, what's broken) |

---

## 🎯 Success Criteria

### For Developers (Before Deployment)
- [ ] All Priority 1 bugs fixed
- [ ] All Priority 2 bugs fixed
- [ ] Full testing checklist passes
- [ ] All 6 dropdown APIs returning data
- [ ] All 3 form submission endpoints working
- [ ] Error handling for all status codes
- [ ] Mobile responsive design verified

### For QA (Before Launch)
- [ ] All 114 test cases pass
- [ ] Form completes all 7 steps without errors
- [ ] All conditional fields work correctly
- [ ] API payload validation passes
- [ ] Database records created correctly
- [ ] Cross-browser testing complete
- [ ] Performance testing (< 2s form load, < 1s step submit)

### For Product (Before Live)
- [ ] Form completion rate > 80%
- [ ] Error rate < 2%
- [ ] Average time to complete < 10 minutes
- [ ] User feedback positive
- [ ] No critical bugs reported

---

## 💡 Pro Tips

1. **Use cURL for quick API testing**
   ```bash
   curl -X GET "https://emap.epd.dev/api/partner/countries" \
     -H "Authorization: YOUR_API_KEY"
   ```

2. **Check browser console for errors**
   - Look for dependent-fields.js errors
   - Check Google Maps API errors
   - Monitor network tab for failed requests

3. **Test on real data**
   - Don't just hardcode test values
   - Use actual API responses
   - Test with edge cases (long names, special characters)

4. **Verify localStorage**
   - UUID should persist between page refreshes
   - Check DevTools → Application → localStorage
   - Clear cache if form state broken

5. **Use the testing checklist**
   - Mark off tests as you go
   - Don't skip "optional" features
   - Test both happy path and error cases

---

## 🆘 Troubleshooting Quick Links

| Problem | Where to Look |
|---|---|
| State dropdown not appearing | BUGS_FOUND.md → Issue #1 |
| Addresses not auto-populating | BUGS_FOUND.md → Issue #2 |
| Slider values wrong | BUGS_FOUND.md → Issue #3 |
| Step 4 fields missing | BUGS_FOUND.md → Issue #5 |
| API returns 403 | ENDPOINTS.md → "Error Response Format" |
| API returns 401 | ENDPOINTS.md → Check headers section |
| Form data lost on refresh | skill.md → "localStorage State Management" |
| Conditional fields not showing | DROPDOWNS_REFERENCE.md → "Conditional Logic" |

---

## 📞 Support Resources

**For exact field specifications**: Read `skill.md` Step 1-7 sections

**For API details**: See `ENDPOINTS.md` or `api-examples.md`

**For dropdown values**: Check `DROPDOWNS_REFERENCE.md`

**For issues**: Review `BUGS_FOUND.md` and `AUDIT_SUMMARY.md`

**For testing**: Follow checklist in `skill.md`

---

## 🎓 Learning Path (Recommended Reading Order)

### Beginner (Non-technical)
1. This file (GETTING_STARTED.md) - 10 min
2. AUDIT_SUMMARY.md - 15 min
3. skill.md → Quick Overview - 5 min
4. ENDPOINTS.md → Summary tables only - 5 min

### Developer
1. This file (GETTING_STARTED.md) - 10 min
2. AUDIT_SUMMARY.md - 20 min
3. skill.md → Complete read - 60 min
4. ENDPOINTS.md → Complete - 30 min
5. api-examples.md → Code examples - 20 min
6. DROPDOWNS_REFERENCE.md → Implementation notes - 20 min
7. BUGS_FOUND.md → Deep dive - 30 min

### QA/Tester
1. AUDIT_SUMMARY.md - 20 min
2. skill.md → Testing Checklist - 30 min
3. ENDPOINTS.md → Error formats - 10 min
4. BUGS_FOUND.md → Testing priorities - 20 min
5. DROPDOWNS_REFERENCE.md → Test data - 15 min

### Project Manager
1. AUDIT_SUMMARY.md - 15 min
2. skill.md → Quick Overview + Feature list - 10 min
3. BUGS_FOUND.md → Priority sections only - 10 min

---

**Total Documentation**: ~8000 lines  
**Total Fields**: 73+  
**Total Endpoints**: 9  
**Total Integrations**: 9  
**Estimated Fix Time**: 1-2 weeks  
**Status**: ✅ Documentation Complete, ⚠️ Implementation In Progress

---

**Start here, then pick the path that matches your role above.**

Good luck! 🚀
