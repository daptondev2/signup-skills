# Data Types Reference

Comprehensive guide to all field data types, formats, and constraints.

---

## Text

**Description:** Plain text string

**Examples:**
- First name, last name
- Company name
- Address fields

**Constraints:**
- Max length specified per field
- No special characters required
- Case-insensitive

**Example:**
```
"John"
"Acme Corporation"
"123 Main Street"
```

---

## Email

**Description:** Email address

**Format:** RFC 5322 compatible

**Validation:**
```regex
^([a-zA-Z0-9_\.\-])+\@(([a-zA-Z0-9\-])+\.)+([a-zA-Z0-9]{2,4})+$
```

**Examples:**
```
john@example.com
user.name@company.co.uk
test+tag@domain.org
```

**Constraints:**
- Max 255 characters
- Must be unique (no duplicates in database)
- Different from other owner email if both present

---

## Phone

**Description:** International phone number

**Format:** E.164 (with country code)

**Validation:**
- Must be valid number for country
- Returns E.164 format from API

**Examples:**
```
+1-555-123-4567      (US)
+1-416-555-0123      (Canada)
+44-20-7946-0958     (UK)
```

**Constraints:**
- Max 20 characters
- Must include country code
- Library: intlTelInput.js
- Returns via `getNumber()`

---

## URL

**Description:** Web address

**Format:** Must include protocol (http:// or https://)

**Validation:**
```regex
^(http:\/\/www\.|https:\/\/www\.|http:\/\/|https:\/\/)?.+\..{1,}(:[0-9]{1,5})?(\/.*)?$
```

**Examples:**
```
https://example.com
http://www.company.com
https://subdomain.example.co.uk:8080/path
```

**Constraints:**
- Max 255 characters
- Must include protocol
- Valid domain required

---

## Date

**Description:** Calendar date

**Format:** YYYY-MM-DD (ISO 8601)

**Examples:**
```
2024-07-09
1980-05-15
2000-01-01
```

**Constraints:**
- No other formats accepted
- Must be valid date (no 2024-02-30)
- Range varies by field:
  - DOB: 18+ to 100 years old
  - Business formed: 100 years ago to today
  - Expiration: Past dates usually disallowed

**Library:** daterangepicker.js for UI

---

## Number

**Description:** Integer or decimal number

**Variants:**

### Integer
- Ownership percentage: 1-100
- Slider values: 0-100

**Examples:**
```
50
75
100
```

### Decimal (Currency)
- Amount fields: Usually integers (no decimals in this API)
- Format: 12-digit maximum

---

## Currency

**Description:** Monetary amount

**Format:** Numeric, no symbols or commas sent to API

**Display Format:** Comma-separated thousands

**Examples:**

Input (to API):
```
500000
1234567
```

Display (to user):
```
500,000
1,234,567
```

**Constraints:**
- Max 12 digits (before comma)
- No decimal places in this API
- No currency symbol in data
- Library: Cleave.js for formatting
- Stored as integer without commas

---

## Dropdown / Select

**Description:** Single value from predefined list

**Format:** Numeric ID from reference endpoint

**Examples:**
```json
{
  "id": 1,
  "name": "United States"
}
```

**Send to API:** ID only ("1")

**Display to User:** Name only ("United States")

**Constraints:**
- Must match value from reference data
- Single selection only
- Required fields must have selection

---

## Checkbox Array

**Description:** Multiple selections from list

**Format:** Array of numeric IDs

**Examples:**
```json
[1, 3, 5]
[2]
["1", "2", "3"]
```

**Send to API:**
```
other_interests_capital[]=1&other_interests_capital[]=2
```

**Constraints:**
- Multiple selections allowed
- IDs must be from reference data
- May have minimum selections (e.g., marketing model: at least 1)

---

## Radio Button

**Description:** Single yes/no or binary choice

**Format:** "0" or "1" (string)

**Examples:**
```
"0"  (No)
"1"  (Yes)
```

**Send to API:** String "0" or "1"

**Constraints:**
- Exactly two options
- String format, not boolean

---

## SSN/SIN

**Description:** Social Security Number / Social Insurance Number

**Format:** XXX-XX-XXXX (with hyphens in display, stored without)

**Examples:**
```
Display: 123-45-6789
API: 123456789
```

**Constraints:**
- Exactly 9 digits
- Must pass Luhn algorithm (US only)
- Masked input for security
- Country-specific formats:
  - US/Canada/PR: 3-2-4 format (XXX-XX-XXXX)
  - Other countries: Not typically required

**Library:** Cleave.js for formatting

---

## State Code

**Description:** US state abbreviation

**Format:** Two-letter code

**Examples:**
```
CA
TX
NY
FL
```

**Constraints:**
- 2 uppercase letters
- Must be valid US state
- Only shown if country = "1" (US)
- Source: GET /api/states endpoint

---

## Country Code

**Description:** Numeric country identifier

**Format:** Numeric ID from reference

**Examples:**
```
1  (United States)
2  (Canada)
3  (United Kingdom)
```

**Constraints:**
- Must match country list
- Controls visibility of country-specific fields
- Used to determine tax ID format, state visibility, etc.

---

## Address Components

**Structure:**
```
street_number (e.g., "123")
street_address (e.g., "Main Street")
city (e.g., "San Francisco")
state (e.g., "CA")
postal_code (e.g., "94105")
country (e.g., "1")
```

**Source:** Google Places API autocomplete

**Constraints:**
- All 6 fields required if any field filled
- Google Places validation required
- State conditional on country = US

---

## Textarea

**Description:** Multi-line text

**Format:** Plain text, line breaks allowed

**Examples:**
```
Premium consulting packages covering:
- Strategic planning
- Implementation support
- Ongoing advisory services
```

**Constraints:**
- Max 1000 characters
- Min 15 characters (Step 3 description)
- Line breaks preserved

---

## Percentage

**Description:** Numeric percentage value

**Format:** 1-100

**Examples:**
```
25
50
75
100
```

**Constraints:**
- Min 1, Max 100
- If multiple owners, must sum to exactly 100
- If Owner 1 < 51%, Owner 2 required
- Owner 2 max = 100 - Owner 1

---

## Boolean (Hidden)

**Description:** True/false value (not user-visible)

**Format:** "0" or "1"

**Examples:**
```
is_plaid: "0" or "1"
from_lander: "0" or "1"
```

---

## Textarea Array

**Description:** Multiple text entries

**Format:** Array of strings

**Send to API:**
```
textarea[field_id]=value1
textarea[field_id]=value2
```

---

## JSON Date Range

**Description:** Date range with start and end

**Format:** Object with `from` and `to` properties

**Examples:**
```json
{
  "from": "2020-01-01",
  "to": "2020-12-31"
}
```

---

## Tips

- **Always send as string to API** - Not native types
- **Formatting is display-only** - Strip before sending
- **Dates always YYYY-MM-DD** - No other formats
- **Currency without symbols** - Just digits
- **Phone includes country code** - E.164 format
- **Dropdowns send ID not name** - Use numeric ID
- **Validate both client & server** - Never trust client only

---

## See Also

- [Validation Rules](./validation-rules.md)
- [Error Codes](./error-codes.md)
- [Enums Reference](./enums.md)
