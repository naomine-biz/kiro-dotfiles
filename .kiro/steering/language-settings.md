---
inclusion: always
---

# Language Settings

- All responses should be in Japanese
- Code comments should also be in Japanese (when appropriate)
- Technical terms can remain in English (e.g., API, TypeScript, PostgreSQL)
- Error message explanations should be in Japanese

## Spec Document Language

**Important**: All documents under `.kiro/specs/` (requirements.md, design.md, tasks.md) must be written in Japanese.

- Section headings: Japanese (e.g., "要件", "受け入れ基準", "設計", "タスク")
- User stories: Japanese
- Acceptance criteria: Japanese (EARS pattern keywords WHEN/THEN/SHALL can remain in English)
- Design explanations: Japanese
- Task descriptions: Japanese

### Example

```markdown
# Requirements Document

## Introduction
[Explain in Japanese]

## Glossary
- **Term**: [Definition in Japanese]

## Requirements

### Requirement XXX-1

**User Story:** As a user, I want to... so that...

#### Acceptance Criteria

- XXX-1.1: WHEN ... THEN THE System SHALL ...
```

## ASCII Diagram Rules

**Important**: Text inside ASCII art diagrams must be written in **English**.

Japanese characters are full-width, causing misalignment in ASCII diagrams even with monospace fonts.

### Bad Example

```
+------------------+
| 設定マネージャー  |  ← Japanese causes misalignment
+------------------+
```

### Good Example

```
+------------------+
| ConfigManager    |  ← English aligns properly
+------------------+
```

### Where This Applies

- System architecture diagrams in design.md
- Data flow diagrams in design.md
- Any other ASCII art illustrations
