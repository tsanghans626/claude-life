---
allowed-tools: Bash, Read, Write, LS
---

# Issue Decompose

Decompose an issue into separate development and testing documents that can be executed independently and simultaneously.

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
   test -f .claude/epics/*/$ARGUMENTS-dev.md && echo "⚠️ Development guide already exists. Overwrite? (yes/no)"
   test -f .claude/epics/*/$ARGUMENTS-test.md && echo "⚠️ Testing guide already exists. Overwrite? (yes/no)"
   ```

## Instructions

### 1. Read Issue Context

Get issue details from GitHub:

```bash
gh issue view $ARGUMENTS --json title,body,labels
```

Read local task file to understand:

- Technical requirements
- Acceptance criteria
- Dependencies
- Effort estimate

### 2. Analyze Development Requirements

Break down the issue into concrete development tasks:

**Development Considerations:**

- What files need to be created/modified?
- What components/functions need implementation?
- What data models or schemas are needed?
- What APIs or endpoints need to be built?
- What UI components are required?
- What configuration changes are needed?

### 3. Analyze Testing Requirements

Identify comprehensive testing needs:

**Testing Considerations:**

- Unit tests for new functions/methods
- Integration tests for API endpoints
- Component tests for UI elements
- End-to-end tests for user workflows
- Performance tests if applicable
- Security tests if applicable
- Data validation tests
- Error handling tests

### 4. Create Development Guide

Get current datetime: `date -u +"%Y-%m-%dT%H:%M:%SZ"`

Create `.claude/epics/{epic_name}/$ARGUMENTS-dev.md`:

```markdown
---
issue: $ARGUMENTS
title: { issue_title }
type: development
decomposed: { current_datetime }
estimated_hours: { development_hours }
---

# Development Guide: Issue #$ARGUMENTS

## Overview

{Brief description of what needs to be developed}

## Prerequisites

- [ ] Local task file reviewed: `.claude/epics/{epic_name}/$ARGUMENTS.md`
- [ ] GitHub issue understood: `gh issue view $ARGUMENTS`
- [ ] Development environment set up
- [ ] Dependencies identified

## Development Task List

### Core Implementation Tasks

#### Task 1: {Task Name}

**Description**: {What needs to be implemented}
**Files to modify/create**:

- `{file_path}` - {description of changes}
- `{file_path}` - {description of changes}

**Context needed**:

- {Understanding of existing patterns}
- {Dependencies on other components}
- {Configuration requirements}

**Estimated Hours**: {hours}
**Priority**: High/Medium/Low
**Dependencies**: {none or list dependencies}

#### Task 2: {Task Name}

**Description**: {What needs to be implemented}
**Files to modify/create**:

- `{file_path}` - {description of changes}

**Context needed**:

- {Required knowledge}
- {Integration points}

**Estimated Hours**: {hours}
**Priority**: High/Medium/Low
**Dependencies**: Task 1

### Configuration & Setup Tasks

#### Task 3: {Configuration Task}

**Description**: {Configuration changes needed}
**Files to modify/create**:

- `{config_file}` - {changes needed}

**Context needed**:

- {Current configuration structure}
- {Environment considerations}

**Estimated Hours**: {hours}
**Priority**: Medium
**Dependencies**: {none or list}

## Task Dependencies

### Development Task Flow

1. {Task order and dependencies}
2. {Sequential requirements}
3. {Parallel opportunities}

## Definition of Done

### Development Tasks Complete When:

- [ ] Code implemented and follows existing patterns
- [ ] Code reviewed and meets quality standards
- [ ] Documentation updated if needed
- [ ] No lint/type errors
- [ ] Basic functionality verified
- [ ] Ready for testing (see companion testing guide)

## Testing Readiness

- **Testing Guide**: `.claude/epics/{epic_name}/$ARGUMENTS-test.md`
- **Test Dependencies**: Development tasks must be completed before testing begins
- **Parallel Work**: Testing team can prepare test cases while development is in progress

## Development Notes

{Special considerations, warnings, or recommendations for development implementation}

## Handoff to Testing

When development tasks are complete:

1. Notify testing team that implementation is ready
2. Provide build/deployment instructions if needed
3. Share any implementation details that affect testing
4. Review testing guide together if needed
```

### 5. Create Testing Guide

Create `.claude/epics/{epic_name}/$ARGUMENTS-test.md`:

```markdown
---
issue: $ARGUMENTS
title: { issue_title }
type: testing
decomposed: { current_datetime }
estimated_hours: { testing_hours }
---

# Testing Guide: Issue #$ARGUMENTS

## Overview

{Brief description of what needs to be tested}

## Prerequisites

- [ ] Development guide reviewed: `.claude/epics/{epic_name}/$ARGUMENTS-dev.md`
- [ ] Understanding of implementation approach
- [ ] Test environment set up
- [ ] Testing frameworks/tools available

## Testing Strategy

### Testing Levels

- **Unit Tests**: Individual functions and methods
- **Integration Tests**: Component interactions and API endpoints
- **End-to-End Tests**: Complete user workflows
- **Performance Tests**: (if applicable)
- **Security Tests**: (if applicable)

## Testing Task List

### Unit Testing Tasks

#### Test 1: {Component/Function Name} Unit Tests

**Description**: Test individual functions and methods
**Test files to create/modify**:

- `{test_file_path}` - {description of test coverage}

**Context needed**:

- {Understanding of function behavior}
- {Edge cases to test}
- {Mock requirements}

**Test scenarios**:

- Happy path scenarios
- Edge cases
- Error conditions
- Boundary conditions

**Estimated Hours**: {hours}
**Priority**: High
**Dependencies**: Corresponding development task completed

#### Test 2: {Next Component} Unit Tests

**Description**: {Test description}
**Test files to create/modify**:

- `{test_file_path}` - {coverage description}

**Context needed**:

- {Required knowledge}
- {Dependencies}

**Test scenarios**:

- {scenario list}

**Estimated Hours**: {hours}
**Priority**: High/Medium/Low
**Dependencies**: {corresponding dev task}

### Integration Testing Tasks

#### Test 3: {API/Component} Integration Tests

**Description**: Test component interactions
**Test files to create/modify**:

- `{test_file_path}` - {integration scenarios}

**Context needed**:

- {API contracts}
- {Data flow understanding}
- {Service dependencies}

**Test scenarios**:

- {scenario 1}
- {scenario 2}
- {error scenarios}

**Estimated Hours**: {hours}
**Priority**: High
**Dependencies**: Multiple development tasks completed

### End-to-End Testing Tasks

#### Test 4: {User Workflow} E2E Tests

**Description**: Test complete user workflows
**Test files to create/modify**:

- `{e2e_test_path}` - {user journey tests}

**Context needed**:

- {User workflow understanding}
- {UI element selectors}
- {Test environment setup}

**Test scenarios**:

- {complete user journey}
- {alternative paths}
- {error recovery}

**Estimated Hours**: {hours}
**Priority**: Medium
**Dependencies**: All development tasks completed

## Testing Dependencies

### Testing Task Flow

1. Unit tests can start as soon as corresponding dev tasks complete
2. Integration tests require multiple dev tasks complete
3. E2E tests require full feature implementation
4. Performance/Security tests run after core functionality verified

### Parallel Testing Opportunities

- Unit tests for different components can run in parallel
- Test case preparation can happen during development phase
- Test environment setup can happen independently

## Test Data Requirements

### Test Fixtures

- {List of test data needed}
- {Sample data files}
- {Database setup requirements}

### Mock Services

- {External services to mock}
- {API endpoints to simulate}
- {Error conditions to simulate}

## Definition of Done

### Testing Tasks Complete When:

- [ ] All test scenarios implemented
- [ ] Tests pass consistently
- [ ] Code coverage meets requirements
- [ ] Edge cases covered
- [ ] Error scenarios tested
- [ ] Performance requirements verified (if applicable)
- [ ] Security requirements verified (if applicable)
- [ ] Test documentation updated

## Testing Notes

{Special considerations, warnings, or recommendations for testing implementation}

## Development Coordination

- **Development Guide**: `.claude/epics/{epic_name}/$ARGUMENTS-dev.md`
- **Communication**: Regular sync with development team on progress
- **Blocking Issues**: Process for handling test-blocking development issues
```

### 6. Validate Decomposition

Ensure:

- All major development work is broken into concrete tasks
- Testing coverage is comprehensive
- File paths are specific and accurate
- Context requirements are detailed
- Time estimates are reasonable
- Dependencies are clear

### 7. Output

```
✅ Decomposition complete for issue #$ARGUMENTS

📋 Development Guide: .claude/epics/{epic_name}/$ARGUMENTS-dev.md
   Development Tasks: {count}
   Estimated hours: {dev_hours}h

🧪 Testing Guide: .claude/epics/{epic_name}/$ARGUMENTS-test.md
   Testing Tasks: {count}
   Estimated hours: {test_hours}h

Total estimated effort: {total_hours}h

Key files to be modified:
  {list main files}

Next Steps:
- Development team: Start with development guide
- Testing team: Review testing guide and prepare test cases
- Both teams: Coordinate on dependencies and handoff points
```

## Important Notes

- **Two separate documents** enable parallel development and testing work
- **Development guide** focuses on implementation tasks and technical requirements
- **Testing guide** focuses on verification, validation, and quality assurance
- Both documents are local only - not synced to GitHub
- Documents cross-reference each other for coordination
- Teams can work independently while maintaining alignment
- Focus on concrete, actionable tasks with clear deliverables
- Include sufficient context for each task to be executed independently
- Consider both happy path and error scenarios in testing
