# Changelog

All notable changes to the EPD eMerchant Signup API are documented here.

**Format:** [Keep a Changelog](https://keepachangelog.com/)

---

## [1.0.0] - 2026-07-10

### Added
- Complete 6-step merchant onboarding workflow
- 80+ documented fields across all steps
- 16 dependent field rules with JSON specifications
- Full API endpoint documentation with examples
- OpenAPI 3.0 specification
- Comprehensive error handling guide
- Authentication & CSRF token management
- Rate limiting documentation
- Integration examples & code samples
- Validation rules reference
- Dependent fields implementation guide
- Enum values for all dropdowns
- Data types reference
- Glossary of terms

### Features
- Support for single and dual beneficial owners
- Conditional ownership fields based on percentages
- Dependent field visibility & requirement logic
- CSRF token refresh on 419 errors
- Auto-save for Step 1 (lander flows)
- Plaid integration for banking (Step 5)
- Google Places autocomplete for addresses
- Phone number validation via intlTelInput
- Dynamic fields modal (Step 6)
- Multi-owner ownership percentage constraints

### Documentation
- Guides for getting started, authentication, error handling, rate limiting
- Step-by-step API reference for all 6 steps
- Complete dependent fields reference with testing checklist
- Sample API requests and responses
- Error recovery flowcharts
- Field validation patterns

### Removed
- Document extraction (Textract) capabilities
- Plaid/Persona verification details
- Signature step documentation
- Dynamic step rule engine internals

---

## [Unreleased]

### Planned for Q3 2026
- [ ] Webhook notifications for step completion
- [ ] Bulk submission endpoint
- [ ] Server-sent events for real-time updates

### Planned for Q4 2026
- [ ] GraphQL API option
- [ ] Webhook signing & verification
- [ ] Audit log retrieval

### Planned for Q1 2027
- [ ] Mobile SDK (React Native)
- [ ] Form analytics & tracking
- [ ] A/B testing framework

---

## Notes

- Versioning follows [Semantic Versioning](https://semver.org/)
- Breaking changes require major version bump
- 90-day deprecation notice before removing features
- See [VERSIONING.md](./VERSIONING.md) for support timeline
