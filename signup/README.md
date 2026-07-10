# EPD eMerchant Signup API

**Version:** 1.0.0  
**Base URL:** `https://emap.epd.dev`  
**Authentication:** Bearer Token (OAuth 2.0)  
**Rate Limit:** 100 requests/minute  
**Response Format:** JSON

---

## 📚 Quick Navigation

- **[Getting Started](./guides/getting-started.md)** - Setup & first API call
- **[Authentication](./guides/authentication.md)** - Token management & CSRF handling
- **[Error Handling](./guides/error-handling.md)** - Status codes & error responses
- **[Rate Limiting](./guides/rate-limiting.md)** - Request limits & backoff strategy
- **[API Reference](./reference/endpoints.md)** - Complete endpoint documentation
- **[OpenAPI Spec](./openapi.yaml)** - Machine-readable API specification
- **[Examples](./examples/)** - Code samples & complete workflows

---

## 🚀 What is This?

The EPD eMerchant Signup API is a **6-step merchant onboarding workflow** for integrating signup forms into external websites. 

**Supports:**
- ✅ Account setup & company registration
- ✅ Business structure & financial details
- ✅ Product/transaction information
- ✅ Beneficial owner management (single or dual ownership)
- ✅ Banking & payment processing
- ✅ Final preferences & dynamic fields

---

## 📋 The 6-Step Process

| Step | Purpose | Fields | Time |
|------|---------|--------|------|
| [1](./reference/endpoints.md#step-1-account-information) | Owner & company info | 8 | 2-3 min |
| [2](./reference/endpoints.md#step-2-company-details) | Legal structure & addresses | 15 | 5-7 min |
| [3](./reference/endpoints.md#step-3-product-info) | Products & transaction methods | 13 | 3-4 min |
| [4](./reference/endpoints.md#step-4-beneficial-owners) | Owner personal & financial data | 24+ | 8-10 min |
| [5](./reference/endpoints.md#step-5-banking-info) | Bank account & payment processor | 8 | 2-3 min |
| [6](./reference/endpoints.md#step-6-final-details) | Referral & preferences | 12+ | 2-3 min |

**Total fields documented:** 80+  
**Dependent field rules:** 16  
**Completion time:** ~25-35 minutes

---

## 🔑 Key Concepts

### Dependent Fields
Conditional field visibility based on parent values. [Learn more →](./reference/dependent-fields.md)

**Example:** If country = "US", state dropdown appears and becomes required.

### Session Management
Each signup session has a UUID that persists across all 6 steps.

### Error Handling
Comprehensive error codes with recovery instructions. [See all →](./reference/error-codes.md)

### Auto-Save
Step 1 supports real-time form persistence via `from_lander=1`

---

## 💻 For Developers

### Start Here
1. Read [Getting Started](./guides/getting-started.md)
2. Review [Authentication](./guides/authentication.md)
3. Check [API Reference](./reference/endpoints.md) or [OpenAPI Spec](./openapi.yaml)
4. Follow [Step 1 Example](./examples/step1-example.md)

### Common Tasks
- **Build signup form** → [Step 1-6 Endpoints](./reference/endpoints.md)
- **Handle dependent fields** → [Dependent Fields Guide](./reference/dependent-fields.md)
- **Validate input** → [Validation Rules](./reference/validation-rules.md)
- **Handle errors** → [Error Codes](./reference/error-codes.md)
- **Get dropdown values** → [Enums Reference](./reference/enums.md)

### Integration Patterns
- [Complete workflow example](./examples/complete-flow.md)
- [Sample API responses](./examples/sample-responses.json)
- [Error recovery flow](./guides/error-handling.md)

---

## 📦 Reference Data

All dropdown values available via reference endpoints:

```bash
# Countries & States
https://emap.epd.dev/api/countries
https://emap.epd.dev/api/states

# Business Classification
https://emap.epd.dev/api/industry-types
https://emap.epd.dev/api/shopping-carts

# User Selection
https://emap.epd.dev/api/referral-sources
https://emap.epd.dev/api/pricing-templates
https://emap.epd.dev/api/files-types
```

[Full reference →](./reference/enums.md)

---

## 📝 Documentation

| Document | Purpose |
|----------|---------|
| **Guides** | How-to guides for common tasks |
| **Reference** | Complete API documentation |
| **Examples** | Code samples & workflows |
| **OpenAPI** | Machine-readable specification |

---

## 🔄 Versioning & Support

- **Current Version:** 1.0.0
- **Last Updated:** 2026-07-10
- **Status:** Stable
- **Support:** See [Versioning Policy](./VERSIONING.md)
- **Changes:** See [Changelog](./CHANGELOG.md)

---

## ❓ FAQ

**Q: How long does signup typically take?**  
A: 25-35 minutes for users to complete all 6 steps.

**Q: Can I make any fields optional?**  
A: No. All documented required fields must be provided.

**Q: Do you support Plaid?**  
A: Yes, for Step 5 (banking). [Details](./reference/endpoints.md#plaid-integration)

**Q: What if validation fails?**  
A: Follow error recovery flow. [See error handling →](./guides/error-handling.md)

**Q: How do I handle dependent fields?**  
A: Use the 16 documented rules. [Full guide →](./reference/dependent-fields.md)

---

## 📞 Support

For issues or questions:
1. Check [Error Codes](./reference/error-codes.md)
2. Review relevant [Guide](./guides/)
3. See [Examples](./examples/)

---

**Ready to integrate?** [Get Started →](./guides/getting-started.md)
