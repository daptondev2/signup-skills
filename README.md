# Claude Code Skills

Professional, reusable skills and specifications for AI code generation.

---

## 📋 Signup Form Skill

**Location**: `./signup/`

A comprehensive skill for creating a merchant signup form with 6 steps, 59 fields, dynamic behaviors, and complete API integration.

### ⭐ Main Skill Files

These are the **PRIMARY FILES** to use:

1. **`skill.md`** (26KB) - Complete form specification
   - All 6 steps with detailed field documentation
   - 10+ dynamic field behaviors
   - Validation rules and auto-save strategy
   - Best practices and design principles

2. **`specification.json`** (32KB) - Machine-readable specification
   - Exact field definitions and types
   - API endpoint mappings
   - Validation rules with regex patterns
   - Dependency graph for dynamic fields

### 📚 Reference Documentation

Additional resources in `reference/` folder:

- `quick-start.md` - Quick reference guide for all fields
- `api-integration.md` - Implementation patterns and examples
- `implementation-summary.md` - API changes and checklist
- `creation-notes.md` - Historical creation summary

---

## 🚀 How to Use

### For AI Code Generation (Recommended)

Share the **main skill files** with AI:

```
"Generate the signup form using these specifications:
1. .claude/skills/signup/skill.md
2. .claude/skills/signup/specification.json

Also reference:
3. .claude/skills/signup/reference/api-integration.md
   (for implementation patterns)"
```

### For Developer Reference

1. Read `skill.md` for complete understanding
2. Use `specification.json` for exact field definitions
3. Review `reference/api-integration.md` for implementation patterns
4. Use `reference/quick-start.md` for quick lookup

---

## 📁 Folder Structure

```
skills/
├── README.md ................................. This file (index)
│
└── signup/ ................................... Signup form skill
    ├── skill.md ............................. ⭐ MAIN: Complete specification
    ├── specification.json .................. ⭐ MAIN: Machine-readable spec
    │
    └── reference/ .......................... Supporting documentation
        ├── quick-start.md
        ├── api-integration.md
        ├── implementation-summary.md
        └── creation-notes.md
```

**Key**: Files at skill root are MAIN. Files in reference/ are supporting.

---

## ✨ Signup Skill Features

✅ **Complete Coverage**: 6 steps, 59 fields  
✅ **Dynamic Fields**: 10+ conditional behaviors  
✅ **API Integration**: 6 endpoints documented  
✅ **Validation**: All rules specified  
✅ **Best Practices**: Industry-standard patterns  
✅ **Two Formats**: Markdown + JSON

**API Endpoints**:
- `GET /api/partner/countries`
- `GET /api/partner/states`
- `GET /api/partner/industry-types`
- `GET /api/partner/referral-sources`
- `GET /api/partner/shopping-carts`
- `GET /api/partner/interest-details`

---

## 🎯 Creating New Skills

To add a new skill, follow this structure:

```
skills/
└── {skill-name}/
    ├── skill.md ............................ Main specification
    ├── specification.json ................. Machine-readable spec
    └── reference/
        ├── quick-start.md
        ├── {feature}-guide.md
        └── notes.md
```

Then update this README with the new skill.

---

## 📊 Current Skills

| Skill | Status | Files |
|-------|--------|-------|
| Signup Form | ✅ Ready | skill.md, specification.json |

---

## 📞 Support

**Main specification**: `./signup/skill.md`  
**Field definitions**: `./signup/specification.json`  
**API docs**: `./signup/reference/api-integration.md`  
**Quick lookup**: `./signup/reference/quick-start.md`

---

**Status**: ✅ Production Ready  
**Total Size**: ~106KB  
**Last Updated**: 2025-07-15
