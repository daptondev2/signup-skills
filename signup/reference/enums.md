# Enums & Reference Data

Complete list of all dropdown values and reference data.

All values obtained via reference endpoints at `https://emap.epd.dev/api/`

---

## Countries

**Endpoint:** `GET https://emap.epd.dev/api/countries`

| ID | Country |
|----|---------|
| 1 | United States |
| 2 | Canada |
| 3 | United Kingdom |
| ... | ... (complete list from endpoint) |

---

## US States

**Endpoint:** `GET https://emap.epd.dev/api/states`

| ID | State Code | State Name |
|----|-----------|-----------|
| 1 | AL | Alabama |
| 2 | AK | Alaska |
| ... | ... | ... (50 states) |

---

## Industry Types

**Endpoint:** `GET https://emap.epd.dev/api/industry-types`

| ID | Industry |
|----|----------|
| 1 | Retail |
| 2 | Services |
| 3 | E-Commerce |
| 4 | Financial Services |
| 5 | Healthcare |
| ... | ... |
| 50 | Other |

**Special:** ID 50 triggers conditional `industry_type_other` field

---

## Business Structure

Common values (not from endpoint, hardcoded):

| ID | Structure |
|----|-----------|
| 1 | Sole Proprietorship |
| 2 | Limited Liability Company (LLC) |
| 3 | C Corporation |
| 4 | Partnership |

---

## Marketing Model

| ID | Model |
|----|-------|
| 1 | One-time Purchase |
| 2 | Recurring/Subscription |

**Multiple selection allowed.** If "2" selected, subscription frequency required.

---

## Subscription Frequency

Used when marketing model includes "2" (Recurring).

| ID | Frequency |
|----|-----------|
| 1 | Monthly |
| 2 | Quarterly |
| 3 | Annual |
| 4 | Other |

**Special:** ID 3 (Other) triggers conditional `subscription_frequency_other` text field

---

## Products/Services Description

**Endpoint:** `GET https://emap.epd.dev/api/shopping-carts`

Used in Step 3 to describe services.

| ID | Service Type |
|----|--------------|
| 1 | Physical Products |
| 2 | Digital Services |
| 3 | SaaS/Subscriptions |
| 4 | Professional Services |
| 5 | Consulting |
| ... | ... |

---

## Customer Service Time

Used in Step 3 to indicate service hours.

| ID | Hours |
|----|-------|
| 1 | 24/7 |
| 2 | Business Hours Only |
| 3 | Limited Hours |
| 4 | Seasonal |

---

## Refund Policy

Used in Step 3.

| ID | Policy |
|----|--------|
| 1 | Full Refund |
| 2 | Partial Refund |
| 3 | No Refund |
| 4 | Store Credit Only |
| 5 | Case-by-Case |

---

## Fulfillment Method

Used in Step 3.

| ID | Method |
|----|--------|
| 1 | In-house |
| 2 | Third-party Vendor |
| 3 | Other |

**Special:** If 2 or 3, `fullfillment_company` field required

---

## Shopping Cart / CRM

**Endpoint:** `GET https://emap.epd.dev/api/shopping-carts`

| ID | Platform |
|----|----------|
| 1 | Shopify |
| 2 | WooCommerce |
| 3 | Magento |
| 4 | BigCommerce |
| 5 | Squarespace |
| 6 | Wix |
| 7 | Custom Platform |
| 8 | Other Sales Platform |
| 58 | API / Custom Integration |

**Special:** 
- If 8: Show "Other Sales Platform" label
- If 58: Show "API / Custom Integration Name" label
- Triggers `shopping_cart_other` text field

---

## Referral Source

**Endpoint:** `GET https://emap.epd.dev/api/referral-sources`

| ID | Source |
|----|--------|
| 1 | Google Search |
| 2 | Direct |
| 3 | Social Media |
| 4 | Email |
| 5 | Partnership |
| 8 | Live Event / Trade Show |
| 14 | Friend |
| 50 | Other |

**Special:** IDs 8, 14, 50 trigger `hear_about_us_other` field

---

## Job Title

Used for beneficial owners in Step 4.

| ID | Title |
|----|-------|
| 1 | Owner |
| 2 | Manager |
| 3 | Director |
| 4 | Officer |
| 5 | Employee |

---

## Business Location Type

Used in Step 2.

| ID | Type |
|----|------|
| 1 | Physical Store |
| 2 | Online Only |
| 3 | Both |
| 4 | Warehouse |
| 5 | Mobile |

---

## Multiple Merchant Accounts

Yes/No dropdown in Step 6.

| Value | Meaning |
|-------|---------|
| 0 | No |
| 1 | Yes |

---

## Transaction Device

Optional dropdown in Step 6.

| ID | Device |
|----|--------|
| 1 | POS Terminal |
| 2 | Mobile Reader |
| 3 | Virtual Terminal |
| 4 | Online Only |
| 5 | Multiple |

---

## Bad Experience

Yes/No dropdown in Step 6.

| Value | Meaning |
|-------|---------|
| 0 | No |
| 1 | Yes |

---

## Interests/Capital

**Step 6** - Optional checkbox array.

| ID | Interest |
|----|----------|
| 1 | Working Capital |
| 2 | Equipment Financing |
| 3 | Credit Line |
| 4 | Business Consulting |
| 5 | Other |

---

## Rates/Pricing

**Endpoint:** `GET https://emap.epd.dev/api/pricing-templates`

Fee structures and pricing tiers.

---

## File Types

**Endpoint:** `GET https://emap.epd.dev/api/files-types`

Document types for extraction/upload.

| ID | Type | Use Case |
|----|------|----------|
| 1 | Driver's License | Owner identification (Step 4) |
| 2 | Void Check | Banking verification (Step 5) |
| 3 | Articles of Incorporation | Company registration (Step 2) |
| 4 | Processing Statement | Transaction history (Step 3) |
| 5 | Bank Statement | Banking verification (Step 5) |

---

## Boolean/Radio Values

Where radio buttons or yes/no used:

| Value | Meaning |
|-------|---------|
| "0" | No |
| "1" | Yes |

---

## Currency

| Code | Currency |
|------|----------|
| USD | US Dollar |
| CAD | Canadian Dollar |

Canada-only option in Step 5.

---

## Tips for Implementation

1. **Cache Reference Data** - Fetch once per session, don't repeat
2. **Use IDs not Names** - Send numeric ID to API, not display name
3. **Handle Missing Values** - If new enum added to reference endpoint, show it
4. **Display Names** - Use returned display names (e.g., "United States" not "1")
5. **Sort Consistently** - List order from endpoint is canonical, maintain it

---

## Example: Fetching and Caching

```javascript
class ReferenceDataCache {
  constructor() {
    this.cache = {};
    this.ttl = 3600000; // 1 hour
    this.loaded = {};
  }

  async fetch(endpoint) {
    const now = Date.now();
    
    // Return cached if fresh
    if (this.cache[endpoint] && 
        (now - this.loaded[endpoint]) < this.ttl) {
      return this.cache[endpoint];
    }

    // Fetch fresh data
    const response = await fetch(`https://emap.epd.dev/api/${endpoint}`);
    const data = await response.json();
    
    this.cache[endpoint] = data;
    this.loaded[endpoint] = now;
    
    return data;
  }
}

// Usage
const cache = new ReferenceDataCache();
const countries = await cache.fetch('countries');
```

---

## See Also

- [Endpoints Reference](./endpoints.md)
- [Validation Rules](./validation-rules.md)
- [Data Types](./data-types.md)
