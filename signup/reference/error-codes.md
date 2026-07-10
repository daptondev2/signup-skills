# Error Codes Reference

Complete list of all API error codes, messages, and resolutions.

---

## HTTP Status Codes

| Code | Name | Description | Action |
|------|------|-------------|--------|
| 200 | OK | Request successful | Continue |
| 400 | Bad Request | Validation or logic error | Fix data & retry |
| 401 | Unauthorized | Invalid/expired token | Refresh token |
| 419 | CSRF Token Mismatch | CSRF token expired | Refresh CSRF token |
| 422 | Unprocessable Entity | Business logic error | Review error details |
| 429 | Too Many Requests | Rate limit exceeded | Wait & retry |
| 500 | Internal Server Error | Server error | Retry with backoff |

---

## Validation Errors (400)

### Step 1 Errors

| Field | Message | Cause | Fix |
|-------|---------|-------|-----|
| email | The email has already been taken. | Duplicate email | Use different email |
| email | Invalid email format | Wrong format | Use valid email (RFC 5322) |
| phone | Please enter a valid phone number. | Invalid format | Use valid international number |
| website | Please enter a valid URL. | Missing protocol | Include http:// or https:// |
| business_state | Please select your formation state. | US selected but no state | Select state |
| annual_cc_sales | Please enter a valid amount. | Format/length issue | Max 12 digits, no decimals |

### Step 2 Errors

| Field | Message | Cause | Fix |
|-------|---------|-------|-----|
| industry_type | Please select an industry type. | Not selected | Choose from dropdown |
| industry_type_other | This field is required. | Industry = "Other" | Enter custom industry |
| business_formed | Invalid date. | Wrong format or date range | Use YYYY-MM-DD format |
| federal_tax_id | Invalid tax ID format. | Wrong format for country | Use correct format per country |
| business_registration_number | This field is required. | Non-US country selected | Enter registration number |
| marketingModel | Please select at least one option. | No checkbox selected | Select marketing model |
| subscription_frequency | Please select frequency. | Recurring selected but no freq | Choose frequency |
| street_address | Address validation failed. | Invalid address | Use Google Places autocomplete |

### Step 3 Errors

| Field | Message | Cause | Fix |
|-------|---------|-------|-----|
| card_swiped/customer_entered/staff_entered | Must sum to 100% | Incorrect totals | Adjust slider to 100 |
| describe_highest_transaction | Minimum 15 characters. | Too short | Provide detailed description |
| highest_transaction_amount | Max 16 characters. | Too long | Use amount under 999,999,999,999 |
| fulfillment_by | Please select fulfillment method. | Not selected | Choose fulfillment option |
| fullfillment_company | This field is required. | Vendor/Other selected | Enter vendor name |

### Step 4 Errors

| Field | Message | Cause | Fix |
|-------|---------|-------|-----|
| owner[1][email] | The email has already been taken. | Duplicate email | Use different email |
| owner[1][ssn] | Invalid SSN format. | Wrong format | Use XXX-XX-XXXX format |
| owner[1][ownership_percentage] | Must equal 100% combined. | Wrong total | Adjust percentages to 100 |
| owner[1][dob] | Owner must be 18+ years old. | Too young | Use valid DOB (18+) |
| owner[2] | Owner 2 required. | Owner 1 < 51% | Add Owner 2 details |

### Step 5 Errors

| Field | Message | Cause | Fix |
|-------|---------|-------|-----|
| bank_routing_number | 9 digit routing number required. | Wrong length | Enter 9 digits |
| bank_routing_number | Invalid routing number. | Not a valid US routing | Check routing number |
| bank_account_number | 8-17 characters required. | Wrong length | Use 8-17 character account number |
| current_processing | Please select an option. | Not selected | Choose Yes or No |
| processor_name | This field is required. | Yes selected | Enter processor name |

### Step 6 Errors

| Field | Message | Cause | Fix |
|-------|---------|-------|-----|
| howdidyouhear | Please select how you found us. | Not selected | Choose referral source |
| hear_about_us_other | This field is required. | Referral requires other | Enter additional info |
| bad_experience | Please select an option. | Not selected | Choose Yes or No |
| bad_experience_happened | This field is required. | Yes selected | Enter provider name |
| terms_and_conditions_agreed | You must agree to terms. | Not checked | Check terms agreement |

---

## Authentication Errors

### 401 Unauthorized

```json
{
  "status": 401,
  "message": "Unauthorized",
  "error": "Invalid token"
}
```

**Causes:**
- Token expired
- Token invalid/malformed
- Token revoked
- Wrong token provided

**Fix:**
1. Refresh token from OAuth provider
2. Verify token format (Bearer {token})
3. Check token hasn't been revoked
4. Retry request with new token

### 419 CSRF Token Mismatch

```json
{
  "status": 419,
  "message": "CSRF token mismatch"
}
```

**Causes:**
- CSRF token expired (24 hours)
- Token not matching request
- Token header missing

**Fix:**
1. Fetch fresh CSRF token from meta tag
2. Add X-CSRF-TOKEN header to request
3. Retry with new token

---

## Rate Limiting

### 429 Too Many Requests

```json
{
  "status": 429,
  "message": "Too Many Requests",
  "retry_after": 60
}
```

**Causes:**
- Exceeded 100 requests/minute
- Exceeded 5,000 requests/hour
- Exceeded 100,000 requests/day

**Fix:**
1. Wait `retry_after` seconds
2. Implement exponential backoff
3. Cache reference data to reduce requests
4. Contact support for quota increase

---

## Server Errors

### 500 Internal Server Error

```json
{
  "status": 500,
  "message": "Internal Server Error",
  "request_id": "req_abc123xyz"
}
```

**Causes:**
- Server-side processing error
- Database error
- Unexpected condition

**Fix:**
1. Log error with request_id
2. Retry after 1-2 seconds
3. Implement exponential backoff
4. After 3 failures, contact support with request_id

### 502 Bad Gateway

```json
{
  "status": 502,
  "message": "Bad Gateway"
}
```

**Causes:**
- Server maintenance
- Network error
- Load balancer issue

**Fix:**
1. Retry after 2-5 seconds
2. Implement exponential backoff
3. If persists, contact support

### 503 Service Unavailable

```json
{
  "status": 503,
  "message": "Service Unavailable"
}
```

**Causes:**
- Server maintenance
- Temporary outage
- High traffic

**Fix:**
1. Retry after 5-10 seconds
2. Check status page for announcements
3. Implement exponential backoff

---

## Business Logic Errors

### 422 Unprocessable Entity

```json
{
  "status": 422,
  "message": "Unprocessable Entity",
  "error": "Cannot process request at this time"
}
```

**Possible causes:**
- Duplicate signup application
- Invalid state transition
- Missing prerequisite
- Conflicting data

**Fix:**
1. Review error message
2. Check application state
3. Verify data consistency
4. Contact support if unclear

---

## Error Recovery Flowchart

```
User submits form
      ↓
Server returns response
      ↓
   Status = ?
   ├─ 2xx → Success (advance to next step)
   ├─ 400 → Validation error
   │   └─ Display field errors
   │   └─ Allow user to correct
   │   └─ Resubmit
   ├─ 401 → Auth error
   │   └─ Refresh token
   │   └─ Retry request
   ├─ 419 → CSRF error
   │   └─ Refresh CSRF token
   │   └─ Retry request
   ├─ 429 → Rate limited
   │   └─ Wait retry_after seconds
   │   └─ Retry request
   └─ 5xx → Server error
       └─ Wait 1-2 seconds
       └─ Retry with backoff
       └─ After 3 failures, show error message
```

---

## Tips

- **Always log request_id** for server errors
- **Implement retry logic** with exponential backoff
- **Cache responses** to reduce requests
- **Show user-friendly messages** (don't expose technical errors)
- **Handle 419 automatically** with CSRF refresh
- **Monitor rate limits** using response headers

---

## See Also

- [Error Handling Guide](../guides/error-handling.md)
- [Validation Rules](./validation-rules.md)
- [Rate Limiting](../guides/rate-limiting.md)
