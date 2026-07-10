# Rate Limiting

How rate limiting works and how to handle it.

---

## Rate Limit Quota

| Metric | Limit |
|--------|-------|
| **Requests per minute** | 100 |
| **Requests per hour** | 5,000 |
| **Requests per day** | 100,000 |
| **Concurrent connections** | 10 |

Limits are **per API token**.

---

## Rate Limit Headers

Every response includes rate limit information:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1720000000
```

**Headers mean:**
- `Limit`: Maximum requests per minute (100)
- `Remaining`: Requests left before limit hit (87)
- `Reset`: Unix timestamp when limit resets

---

## When Rate Limited

If you exceed limits, you get a 429 response:

```json
{
  "status": 429,
  "message": "Too Many Requests",
  "retry_after": 60
}
```

**Response headers:**

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1720000060
Retry-After: 60
```

---

## Best Practices

### 1. Monitor Remaining Quota

Before making request, check `X-RateLimit-Remaining`:

```javascript
const remaining = response.headers.get('X-RateLimit-Remaining');

if (remaining < 10) {
  console.warn('Approaching rate limit');
  // Consider slowing down requests
}
```

### 2. Implement Backoff Strategy

```javascript
async function requestWithBackoff(url, options = {}, retries = 3) {
  for (let i = 0; i < retries; i++) {
    const response = await fetch(url, options);

    if (response.status !== 429) {
      return response;
    }

    // Rate limited - wait and retry
    const retryAfter = response.headers.get('Retry-After') || (Math.pow(2, i) * 1000);
    console.log(`Rate limited. Waiting ${retryAfter}ms before retry.`);
    
    await new Promise(resolve => setTimeout(resolve, retryAfter));
  }

  throw new Error('Max retries exceeded');
}
```

### 3. Cache Responses

Reference data changes infrequently. Cache it:

```javascript
const cache = {
  countries: null,
  cachedAt: 0,
  TTL: 3600000 // 1 hour
};

async function getCountries() {
  const now = Date.now();
  
  if (cache.countries && (now - cache.cachedAt) < cache.TTL) {
    return cache.countries; // Use cached
  }

  // Fetch fresh data
  const response = await fetch('https://emap.epd.dev/api/countries');
  cache.countries = await response.json();
  cache.cachedAt = now;
  
  return cache.countries;
}
```

### 4. Batch Operations When Possible

Instead of:
```javascript
// ❌ 10 separate requests
for (let i = 0; i < 10; i++) {
  await submitStep(stepData, i);
}
```

Do:
```javascript
// ✅ Still sequential but optimized
for (let i = 0; i < 10; i++) {
  await submitStep(stepData, i);
  // Check remaining quota
  // Implement exponential backoff if needed
}
```

### 5. Optimize Request Frequency

- **Don't:** Poll endpoints rapidly for updates
- **Do:** Use webhooks or implement reasonable poll intervals (30+ seconds)

### 6. Handle 429 Gracefully

```javascript
async function submitWithRateLimitHandling(data) {
  try {
    const response = await fetch('https://emap.epd.dev/signup/companies', {
      method: 'POST',
      body: new URLSearchParams(data)
    });

    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After');
      
      // Show user-friendly message
      showMessage(`Too many requests. Please wait ${retryAfter} seconds.`);
      
      // Disable submit button
      submitButton.disabled = true;
      
      // Wait then re-enable
      setTimeout(() => {
        submitButton.disabled = false;
        submitButton.textContent = 'Try Again';
      }, retryAfter * 1000);
      
      return;
    }

    return response.json();
  } catch (error) {
    console.error('Error:', error);
  }
}
```

---

## Rate Limit Scenarios

### Scenario 1: Single User Signup

**Typical requests:**
- GET /step/1 → 1 request
- POST /signup/user → 1 request
- GET /step/2 → 1 request
- POST /signup/companies → 1 request
- ... repeat for steps 3-6
- Total: ~20 requests

**With reference data caching:**
- Total: ~20 requests

✅ **Well within limits.** No issues.

### Scenario 2: Batch Signups (10 concurrent users)

**Scenario:**
- 10 users simultaneously filling forms
- Each user: ~20 API calls
- Total: ~200 requests

**Per minute breakdown:**
- Users complete steps at different rates
- Spread across 20-30 minutes
- ~7-10 requests/minute average

✅ **Well within limits.** No issues.

### Scenario 3: High-Volume Processing (1000 signups/day)

**Scenario:**
- Processing 1000 signups daily
- Average completion time: 25 minutes
- Peak: 50 concurrent users

**Per minute breakdown:**
- 50 users × ~1 request/minute average = 50 requests
- Throughout business hours (10 hours): 500-600 requests/hour
- Total daily: ~6,000 requests

⚠️ **Approaching limit.** Consider:
- Spreading load across multiple API tokens
- Optimizing reference data caching
- Implementing queue system for submissions

### Scenario 4: Reference Data Polling

**❌ Wrong (Rate Limited):**
```javascript
// Polls every 1 second
setInterval(() => {
  fetch('https://emap.epd.dev/api/countries')
}, 1000);
```

**✅ Right (Cached):**
```javascript
// Fetch once per hour
const countries = await getCountries(); // Cached for 1 hour
```

---

## Quota Increases

If you need higher limits:

1. Contact EPD Support
2. Provide use case details
3. Explain expected volume
4. Request quota increase

Typical increase: 10x for qualified partners.

---

## Monitoring & Alerts

### Set Up Alerts

```javascript
const ALERT_THRESHOLD = 20; // Alert at 20 requests remaining

response.headers.get('X-RateLimit-Remaining') < ALERT_THRESHOLD
  ? console.warn('Approaching rate limit')
  : null;
```

### Track Metrics

Log rate limit data for analysis:

```javascript
const rateLimitMetrics = {
  timestamp: new Date(),
  limit: response.headers.get('X-RateLimit-Limit'),
  remaining: response.headers.get('X-RateLimit-Remaining'),
  reset: response.headers.get('X-RateLimit-Reset')
};

// Save to analytics
logMetric(rateLimitMetrics);
```

---

## Need Help?

- [Getting Started](./getting-started.md)
- [Error Handling](./error-handling.md)
- [API Reference](../reference/endpoints.md)
