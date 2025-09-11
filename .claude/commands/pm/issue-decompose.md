---
allowed-tools: Bash, Read, Write, LS, Glob, Grep, Edit
---

# Issue Decompose

Decompose an issue into interface code, development guide, and testing guide.

## Usage

```
/pm:issue-decompose <issue_number>
```

## Quick Check

1. **Find local task file:**
   - First check if `.claude/epics/*/$ARGUMENTS.md` exists (new naming convention)
   - If not found, search for file containing `github:.*issues/$ARGUMENTS` in frontmatter (old naming)
   - If not found: "❌ No local task for issue #$ARGUMENTS. Run: /pm:import first"

2. **Check for existing decomposition:**
   ```bash
   test -f .claude/epics/*/$ARGUMENTS-interface.md && echo "⚠️ Decomposition already exists. Overwrite? (yes/no)"
   ```

## Instructions

### 1. Read Issue Context

Read the local issue file to understand:

- Technical requirements
- Acceptance criteria
- Dependencies
- Implementation details

### 2. Generate Interface Code

Based on the issue requirements, create appropriate interface code files in the project directory:

**For TypeScript/JavaScript projects:**

- Type definitions in `src/types/`
- Interface contracts in `src/interfaces/`
- API definitions in `src/api/`

**For other languages, follow project conventions:**

- Go: interface files in appropriate packages
- Python: abstract base classes and protocols
- Java: interface definitions
- Rust: trait definitions

**Code Generation Guidelines:**

- Follow existing project patterns and naming conventions
- Include proper imports and dependencies
- Add type annotations where applicable
- Include JSDoc or equivalent documentation comments
- Ensure interfaces are complete and production-ready

### 3. Create Interface Documentation

Create `.claude/epics/{epic_name}/$ARGUMENTS-interface.md`:

````markdown
---
issue: $ARGUMENTS
title: { issue_title }
created: { current_datetime }
interface_files: { array_of_created_files }
---

# Interface Documentation: Issue #$ARGUMENTS

## Overview

{Brief description of the interfaces and their purpose}

## Generated Code Files

### {File Path 1}

**Purpose**: {What this interface/type/contract defines}
**Key Methods/Properties**:

- `{method_name}()`: {description}
- `{property_name}`: {description}

**Usage Example**:

```typescript
// Example of how this interface would be used
```
````

**Dependencies**: {List any dependencies this interface has}

### {File Path 2}

**Purpose**: {What this interface/type/contract defines}
**Key Methods/Properties**:

- `{method_name}()`: {description}
- `{property_name}`: {description}

**Usage Example**:

```typescript
// Example of how this interface would be used
```

**Dependencies**: {List any dependencies this interface has}

## Architecture Notes

{Brief explanation of how these interfaces fit into the overall system architecture}

## Implementation Notes

{Any important notes about the implementation approach, design decisions, or constraints}

````

### 4. Create Development Guide

Create `.claude/epics/{epic_name}/$ARGUMENTS-dev.md`:

```markdown
---
issue: $ARGUMENTS
title: { issue_title }
role: developer
status: pending
created: { current_datetime }
interface_ref: $ARGUMENTS-interface.md
---

# Development Guide: Issue #$ARGUMENTS

## Task Overview

{Clear description of what needs to be implemented}

## Interface Contracts

Refer to `$ARGUMENTS-interface.md` for:
- Type definitions and interfaces
- Method signatures
- Data structures
- API contracts

## Implementation Tasks

### 1. Core Implementation

**Files to Create/Modify**:
- `{file_path}`: {description of changes needed}
- `{file_path}`: {description of changes needed}

**Key Functions to Implement**:
- `{function_name}()`: {description and requirements}
- `{function_name}()`: {description and requirements}

### 2. Integration Points

**Database Layer**:
- {Specific database changes needed}
- {Migration requirements}

**API Layer**:
- {Endpoint implementations needed}
- {Middleware or validation requirements}

**UI Layer** (if applicable):
- {Component implementations}
- {State management changes}

### 3. Configuration and Setup

{Any configuration changes, environment variables, or setup requirements}

## Implementation Guidelines

### Code Standards
- Follow existing project conventions
- Use interfaces defined in the interface documentation
- Ensure proper error handling
- Add logging where appropriate

### Performance Considerations
{Any performance requirements or considerations}

### Security Considerations
{Any security requirements or considerations}

## Dependencies

### Internal Dependencies
{List any internal modules or components this depends on}

### External Dependencies
{List any new packages or external services needed}

## Acceptance Criteria

{Copy from original issue and make more specific for implementation}

## Definition of Done

- [ ] All interface contracts implemented
- [ ] Code follows project standards
- [ ] Error handling implemented
- [ ] Logging added where appropriate
- [ ] Integration tests pass
- [ ] Performance requirements met
- [ ] Security requirements met
- [ ] Code reviewed and approved
````

### 5. Create Testing Guide

Create `.claude/epics/{epic_name}/$ARGUMENTS-test.md`:

````markdown
---
issue: $ARGUMENTS
title: { issue_title }
role: tester
status: pending
created: { current_datetime }
interface_ref: $ARGUMENTS-interface.md
dev_ref: $ARGUMENTS-dev.md
---

# Testing Guide: Issue #$ARGUMENTS

## Testing Overview

{Description of what needs to be tested}

## Interface Contracts

Refer to `$ARGUMENTS-interface.md` for:

- Expected method signatures
- Input/output specifications
- Data structure requirements
- Error handling contracts

## Test Strategy

### 1. Unit Tests

**Test Files to Create**:

- `{test_file_path}`: {description of what to test}
- `{test_file_path}`: {description of what to test}

**Key Test Cases**:

#### {Function/Component Name}

- **Happy Path**: {description}
- **Edge Cases**: {list specific edge cases}
- **Error Cases**: {list error conditions to test}
- **Input Validation**: {validation rules to test}

#### {Function/Component Name}

- **Happy Path**: {description}
- **Edge Cases**: {list specific edge cases}
- **Error Cases**: {list error conditions to test}
- **Input Validation**: {validation rules to test}

### 2. Integration Tests

**Integration Points to Test**:

- {Description of integration with other components}
- {Database interaction tests}
- {API endpoint tests}
- {Third-party service integration}

**Test Scenarios**:

- {End-to-end workflow tests}
- {Cross-component interaction tests}
- {Data flow validation tests}

### 3. Performance Tests

**Performance Requirements** (from dev guide):

- {Performance benchmarks to validate}
- {Load testing requirements}
- {Response time requirements}

### 4. Security Tests

**Security Requirements** (from dev guide):

- {Authentication/authorization tests}
- {Input sanitization tests}
- {Data privacy tests}

## Test Data Requirements

### Mock Data

{Describe what mock data is needed}

### Test Fixtures

{Describe any test fixtures or setup data needed}

### External Service Mocking

{Describe what external services need to be mocked and how}

## Test Environment Setup

### Prerequisites

{List any setup requirements for testing}

### Test Configuration

{Any special configuration needed for tests}

### Database Setup

{Any database setup or migration requirements for tests}

## Test Execution

### Running Tests

```bash
# Commands to run different types of tests
npm test                    # All tests
npm run test:unit          # Unit tests only
npm run test:integration   # Integration tests only
npm run test:e2e          # End-to-end tests
```
````

### Test Coverage Requirements

- Minimum coverage: 80%
- Critical paths: 100% coverage
- Error handling: 100% coverage

## Acceptance Testing

### Manual Testing Checklist

- [ ] {User workflow 1}
- [ ] {User workflow 2}
- [ ] {Error scenario 1}
- [ ] {Error scenario 2}
- [ ] {Performance requirement validation}
- [ ] {Security requirement validation}

### Automated Testing Checklist

- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Performance tests pass
- [ ] Security tests pass
- [ ] Code coverage meets requirements

## Test Reporting

### Test Results Documentation

{How to document and report test results}

### Bug Reporting

{Process for reporting bugs found during testing}

### Performance Metrics

{How to capture and report performance metrics}

## Definition of Done (Testing)

- [ ] All test cases implemented and passing
- [ ] Test coverage requirements met
- [ ] Performance requirements validated
- [ ] Security requirements validated
- [ ] Integration tests confirm interface contracts
- [ ] Manual acceptance testing completed
- [ ] Test documentation updated
- [ ] Test results documented and approved

````

### 6. Commit Changes

After creating all files, commit them:

```bash
git add .claude/epics/*/$ARGUMENTS-interface.md .claude/epics/*/$ARGUMENTS-dev.md .claude/epics/*/$ARGUMENTS-test.md
# Add any generated code files
git add src/
git commit -m "Issue #$ARGUMENTS[decompose]: Add interface code, dev and test guides

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"
````

### 7. Output

```
✅ Issue #$ARGUMENTS decomposed successfully

Created files:
  📋 $ARGUMENTS-interface.md  (Interface documentation and code index)
  🛠️  $ARGUMENTS-dev.md       (Development implementation guide)
  🧪 $ARGUMENTS-test.md      (Testing strategy and guide)

Generated interface code files:
  {list of actual code files created}

Next steps:
  - Review interface contracts in $ARGUMENTS-interface.md
  - Start development with $ARGUMENTS-dev.md
  - Plan testing approach with $ARGUMENTS-test.md
  - Use /pm:issue-start $ARGUMENTS to begin implementation
```

## Important Notes

- Interface files contain only code file indexes and brief descriptions
- No code content is stored in interface documentation
- Generated code files follow existing project patterns
- Dev and test guides reference interface documentation
- All files are committed together for traceability
- Focus on creating complete, production-ready interfaces
