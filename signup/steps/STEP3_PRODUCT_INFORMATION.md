---
name: signup-step-3
description: Step 3 Product Information - Fulfillment details, transaction methods, and product descriptions
---

# STEP 3: Product Information

Third step of the 7-step signup form. Collects product fulfillment details, transaction entry methods, and service descriptions.

---

## Fields

**Fulfillment Details** (7 fields):

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| customer_service_time | select | Yes | Static: slug values "0-7-days", "7-30-days", "31+days" |
| refund_policy | select | Yes | Static (slug): Full-Refund, No-Refund, Exchange-Only, Partial-Refund |
| fulfillment_by | select | Yes | Static (slug): Direct-By-You, Service-Only, Vendor, Others |
| fullfillment_company | text | Conditional | Show if fulfillment_by="Vendor" or "Others" |
| shopping_cart | select | Yes | API: `/api/partner/shopping-carts` (slug: "Other", "API / Custom Integration", etc) |
| shopping_cart_other | text | Conditional | Show if shopping_cart="Other" or "API / Custom Integration" |
| leave_deposit | select | Yes | Options: Yes(1), No(0) |

**Transaction Entry Methods** (3 fields - Slider):

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| card_swiped | number | Yes | Slider: multiples of 5 (0-100%) |
| customer_entered | number | Yes | Slider: multiples of 5 (0-100%) |
| staff_entered | number | Yes | Slider: multiples of 5 (0-100%) |

**Validation**: Sum of all three must equal 100%

**Product Details** (4 fields):

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| average_transaction_amount | text | Yes | Numeric, USD |
| highest_transaction_amount | text | Yes | Numeric, USD |
| describe_services | textarea | Yes | Min 15 chars, max 1500 |
| describe_highest_transaction | textarea | Yes | Min 15 chars, max 1500 |

---

## Libraries Required

- `ion-rangeslider` - Three-way transaction slider with step: 5
- `dependent-fields.js` - Conditional field visibility

---

## Dependent Fields

⚠️ **All conditional fields MUST be hidden on page load** (`style="display: none;"`).

- **fullfillment_company**: Show if fulfillment_by = "Vendor" or "Others"
- **shopping_cart_other**: Show if shopping_cart = "Other" or "API / Custom Integration"
- **Slider validation**: Sum of all three must equal 100%

---

## API Integration

**Dropdown Data**:
```
GET /api/partner/shopping-carts
```

Header: `Authorization: {user.security_key}`

**Form Submission**:
```
POST /api/v1/application/step
Headers: X-API-Key: {user.security_key}
Payload:
  step_count: 3
  section: product_info
  all Step 3 fields
Filter: Remove hidden/disabled fields
Response: { next_step_url, message }
```

---

## Field Summary

**Total Fields**: 13  
**Required Fields**: 10  
**Conditional Fields**: 3  
**Slider Fields**: 3 (must sum to 100%)

---

**Production Ready** ✅
