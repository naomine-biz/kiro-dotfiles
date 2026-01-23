---
inclusion: always
---

# Task Management Rules

## tasks.md Rules

`tasks.md` should contain **only plans and instructions**. Do not write execution results.

### What to Include

- Task objectives and goals
- Steps to execute (high-level overview)
- Referenced Requirements

### What NOT to Include

- Detailed commands that were actually executed
- Specific values like ARNs, endpoints, IDs
- Troubleshooting records
- Details of additional work performed

### Structure Constraints (Important)

**Do not use `###` headings within the `## Tasks` section.**

Kiro's task parser expects `- [ ]` checkboxes directly under `## Tasks`. Adding `###` headings in between will prevent the play button from appearing.

❌ Bad:
```markdown
## Tasks

### 1. Setup

- [ ] 1.1 Create project
```

✅ Good:
```markdown
## Tasks

- [ ] 1. Setup
  - [ ] 1.1 Create project
```

### Example

```markdown
- [x] 14.1 Login to ECR
  - `aws ecr get-login-password | docker login ...`
  - _Requirements: EFM-2.1_
```
