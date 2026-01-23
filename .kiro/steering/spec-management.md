---
inclusion: always
---

# Spec Management Rules

## ⚠️ Date Prefix (Required)

When creating a Spec in `.kiro/specs/`, you **must add a `YYYYMMDD-` prefix**.

**Never create specs in `.kiro/specs/` without a date prefix.**

## Folder Structure

| Folder | Purpose | Date Prefix |
|--------|---------|-------------|
| `.kiro/specs/` | In progress / Completed | **Required** (`YYYYMMDD-`) |
| `docs/specs-draft/` | Ideas / Drafts | **Not required** |

## Workflow

1. **Idea stage**: Create in `docs/specs-draft/{feature-name}/` (no date)
2. **When starting**: Move to `.kiro/specs/YYYYMMDD-{feature-name}/` (add commit date)
3. **After completion**: Keep in `.kiro/specs/` (retain as history)

## Date Prefix Rules

- **Format**: `YYYYMMDD-NNN-` (e.g., `20241223-001-`)
- **NNN**: 3-digit sequence number (order within the same day)
- **Reference date**: Git commit date
- **Purpose**: Chronological organization, tracking start order

### Sequence Number Rules

- When creating multiple Specs on the same day, use sequence numbers to manage order
- Start from 001 and assign sequential numbers in creation order
- Example: `20260117-001-api-rate-limiting`, `20260117-002-data-export-feature`

## Requirement Numbering Rules

All Spec requirement documents (requirements.md) **must include a Spec prefix** in requirement numbers.

### Prefix Format

- **Format**: `{SPEC_PREFIX}-{requirement_number}.{acceptance_criteria_number}`
- **Examples**:
  - User Authentication: `UAT-1.1`, `UAT-1.2`, `UAT-2.1`
  - Payment Processing: `PPR-1.1`, `PPR-2.1`

### How to Determine Prefix

1. Convert Spec name to kebab-case
2. Take the first letter of each word in uppercase
3. Create a 3-4 character abbreviation

Examples:
- `user-authentication` → `UAT`
- `payment-processing` → `PPR`
- `notification-service` → `NTS`

### ⚠️ Prefix Duplication Check (Required)

When creating a new Spec, **verify that the prefix doesn't conflict with existing ones**.

Duplicates cause confusion in requirement tracking and test mapping.

**Check command:**
```bash
grep -h "^### Requirement" .kiro/specs/*/requirements.md | sed 's/### Requirement \([A-Z]*\).*/\1/' | sort -u
```

If there's a conflict, choose a more specific abbreviation or add a number (e.g., `VPI2`).

### Where to Apply

**Requirements section:**
```markdown
### Requirement {SPEC_PREFIX}-1
#### Acceptance Criteria
- {SPEC_PREFIX}-1.1: WHEN ... THEN ...
```

**Design document:**
```markdown
**Validates: Requirements {SPEC_PREFIX}-1.1**
```

**Tasks document:**
```markdown
- _Requirements: {SPEC_PREFIX}-1.1, {SPEC_PREFIX}-2.3_
```

## Folder Structure Example

```
.kiro/
└── specs/                          # In progress (loaded by Kiro)
    ├── 20241216-001-user-authentication/
    ├── 20241217-001-payment-processing/
    ├── 20241218-001-notification-service/
    ├── 20260117-001-api-rate-limiting/
    ├── 20260117-002-data-export-feature/
    └── ...

docs/
└── specs-draft/                    # Drafts (no date, not loaded)
    ├── dashboard-redesign/
    ├── mobile-app-support/
    └── ...
```

## Notes

- Only place items that Kiro should auto-load under `.kiro/`
- Place draft Specs in `docs/specs-draft/` (reduces noise)
- Always verify Git commit date when starting work and add the prefix
- When starting multiple Specs on the same day, use sequence numbers (001, 002, ...)
