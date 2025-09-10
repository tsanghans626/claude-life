---
allowed-tools: Bash, Read, Write, LS
---

# Issue Decompose

Decompose an issue into separate interface, development and testing documents that can be executed independently and simultaneously.

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
   test -f .claude/epics/*/$ARGUMENTS-interface.md && echo "⚠️ Interface guide already exists. Overwrite? (yes/no)"
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

### 3. Analyze Interface Requirements

Define the public API and contracts that will enable parallel development and testing:

**Interface Considerations:**

- What functions/methods need to be exposed?
- What are the input/output types and schemas?
- What are the expected behaviors and contracts?
- What error conditions and handling are needed?
- What data structures and models are involved?
- What external dependencies and services are required?

### 4. Analyze Testing Requirements

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

### 5. Create Interface Guide

Get current datetime: `date -u +"%Y-%m-%dT%H:%M:%SZ"`

Create `.claude/epics/{epic_name}/$ARGUMENTS-interface.md`:

````markdown
---
issue: $ARGUMENTS
title: { issue_title }
type: interface
decomposed: { current_datetime }
estimated_hours: { interface_design_hours }
---

# Interface Guide: Issue #$ARGUMENTS

## Overview

{Brief description of the interfaces and contracts that need to be defined}

## Prerequisites

- [ ] Local task file reviewed: `.claude/epics/{epic_name}/$ARGUMENTS.md`
- [ ] GitHub issue understood: `gh issue view $ARGUMENTS`
- [ ] Existing codebase patterns analyzed
- [ ] Dependencies and integration points identified

## Interface Design Tasks

### Core Interface Definitions

#### Interface 1: {Service/Component Name}

**Description**: {What this interface does and why it's needed}
**Files to create**:

- `{file_path}` - {Interface definition file}
- `{file_path}` - {Type definitions}

**Interface Specification**:

```typescript
// Function signatures
export interface {InterfaceName} {
  {method1}({params}): {ReturnType}
  {method2}({params}): Promise<{ReturnType}>
}

// Data types
export interface {DataType} {
  {field1}: {type}
  {field2}: {type}
}

// Error types
export interface {ErrorType} {
  code: string
  message: string
  details?: {DetailType}
}
```
````

**Behavior Contract**:

- **Input validation**: {validation rules}
- **Output guarantees**: {what the function promises to return}
- **Error conditions**: {when and how errors are thrown}
- **Side effects**: {any state changes or external calls}

**Dependencies**: {External interfaces or services required}
**Estimated Hours**: {hours}
**Priority**: High

#### Interface 2: {Next Interface Name}

**Description**: {Interface description}
**Files to create**:

- `{file_path}` - {description}

**Interface Specification**:

```typescript
// Interface definitions
```

**Behavior Contract**:

- {contract details}

**Dependencies**: Interface 1
**Estimated Hours**: {hours}
**Priority**: High/Medium/Low

### Data Schema Definitions

#### Schema 1: {Database/API Schema Name}

**Description**: {Data structure definition}
**Files to create**:

- `{schema_file}` - {Zod/Prisma/etc schema definition}

**Schema Definition**:

```typescript
// Data validation schema
export const {SchemaName} = z.object({
  {field1}: z.{type}(),
  {field2}: z.{type}(),
})

export type {TypeName} = z.infer<typeof {SchemaName}>
```

**Validation Rules**:

- {field validation rules}
- {business logic constraints}
- {format requirements}

**Usage Context**: {Where this schema is used}
**Estimated Hours**: {hours}
**Priority**: High

## Integration Contracts

### External Service Interfaces

#### External Service 1: {Service Name}

**Description**: {Integration requirements}
**Interface Requirements**:

- **Endpoints**: {API endpoints to integrate with}
- **Authentication**: {auth requirements}
- **Data Format**: {expected input/output format}
- **Error Handling**: {how to handle service errors}

**Mock Requirements**: None (real service integration only)
**Estimated Hours**: {hours}
**Priority**: Medium

## Interface Dependencies

### Interface Creation Flow

1. **Core Data Types**: Define fundamental data structures first
2. **Service Interfaces**: Define method signatures and contracts
3. **Integration Contracts**: Define external service interfaces
4. **Error Handling**: Define comprehensive error types and handling

### Parallel Development Enablement

Once interfaces are defined, teams work simultaneously:

- **Development Team**: Implements functions against defined interfaces (Red-Green-Refactor cycle)
- **Testing Team**: Writes tests using interface contracts (tests initially fail until implementation)
- **Integration Team**: Sets up service connections using interface contracts
- **All Teams**: Coordinate through shared interface documentation, not through waiting

## Definition of Done

### Interface Design Complete When:

- [ ] All function signatures defined with TypeScript types
- [ ] Input/output contracts specified with validation rules
- [ ] Error conditions and types clearly defined
- [ ] Data schemas created with validation
- [ ] Integration contracts specified for external services
- [ ] Interface documentation is comprehensive
- [ ] No mock interfaces - all definitions are for real implementations

## Development Handoff

**Development Guide**: `.claude/epics/{epic_name}/$ARGUMENTS-dev.md`
**Testing Guide**: `.claude/epics/{epic_name}/$ARGUMENTS-test.md`

### Handoff Requirements:

- Interface files created but functions only contain type signatures
- All types exported and available for import
- Documentation includes usage examples
- Contract specifications are detailed and unambiguous

## Interface Notes

{Special considerations for interface design, breaking changes, backwards compatibility, etc.}

````

### 6. Create Development Guide

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
- [ ] Interface specifications defined: `.claude/epics/{epic_name}/$ARGUMENTS-interface.md`
- [ ] GitHub issue understood: `gh issue view $ARGUMENTS`
- [ ] Development environment set up
- [ ] Interface contracts available for implementation

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

## Testing Coordination

- **Testing Guide**: `.claude/epics/{epic_name}/$ARGUMENTS-test.md`
- **Test-Driven Development**: Tests can be written based on specifications, not implementation
- **Parallel Work**: Testing and development can proceed simultaneously once API interfaces are defined
- **Continuous Integration**: Tests provide immediate feedback during development

## Development Notes

{Special considerations, warnings, or recommendations for development implementation}

## Handoff to Testing

When development tasks are complete:

1. Notify testing team that implementation is ready
2. Provide build/deployment instructions if needed
3. Share any implementation details that affect testing
4. Review testing guide together if needed
````

### 7. Create Testing Guide

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

- [ ] Interface contracts defined: `.claude/epics/{epic_name}/$ARGUMENTS-interface.md` (function signatures & behavior)
- [ ] Development specifications understood: `.claude/epics/{epic_name}/$ARGUMENTS-dev.md` (implementation approach)
- [ ] Testing strategy aligned with interface contracts
- [ ] Test environment set up (independent of development completion)
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
**Dependencies**: Function signatures and expected behavior defined (tests will initially fail until implementation)

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
**Dependencies**: Expected interface behavior defined (tests validate implementation against specifications)

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
**Dependencies**: Component implementations available for real integration testing

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
**Dependencies**: Full feature implementation completed and deployed to test environment

## Testing Dependencies

### Testing Task Flow

1. **Test-First Approach**: Unit tests written from specifications (will fail until implementation exists)
2. **Red-Green-Refactor Cycle**: Tests fail (red) → implementation makes them pass (green) → refactor code
3. **Integration Testing**: Requires actual component implementations to test real interactions
4. **E2E Validation**: Requires fully implemented features for complete workflow testing
5. **Performance/Security**: Requires stable implementations for meaningful validation

### Parallel Testing Opportunities

- **Interface-Based Testing**: Unit tests written from specifications, not implementations
- **Test Case Design**: Write test logic based on specifications before implementation exists
- **Test Infrastructure**: Setup testing frameworks and CI/CD pipelines immediately
- **Test Data Preparation**: Create real test databases and fixtures independently
- **Continuous Feedback**: Tests provide real-time validation during development

## Test Data Requirements

### Test Fixtures

- {List of test data needed}
- {Sample data files}
- {Database setup requirements}

### Real Service Dependencies

- {External services required for testing}
- {API endpoints that need to be functional}
- {Service setup requirements for test environment}

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

### 8. Validate Decomposition

Ensure:

- **Interface contracts** are complete with clear function signatures and behavior specifications
- **Development work** is broken into concrete tasks that implement against interfaces
- **Testing coverage** is comprehensive and tests interface contracts, not implementations
- **File paths** are specific and accurate across all three guides
- **Context requirements** are detailed for independent execution
- **Time estimates** are reasonable for each phase (interface → parallel dev+test)
- **Dependencies** are clear with proper interface-first sequencing

### 9. Output

```
✅ Decomposition complete for issue #$ARGUMENTS

🔌 Interface Guide: .claude/epics/{epic_name}/$ARGUMENTS-interface.md
   Interface Tasks: {count}
   Estimated hours: {interface_hours}h

📋 Development Guide: .claude/epics/{epic_name}/$ARGUMENTS-dev.md
   Development Tasks: {count}
   Estimated hours: {dev_hours}h

🧪 Testing Guide: .claude/epics/{epic_name}/$ARGUMENTS-test.md
   Testing Tasks: {count}
   Estimated hours: {test_hours}h

Total estimated effort: {total_hours}h

Key interfaces to be defined:
  {list main interfaces}

Key files to be modified:
  {list main files}

Execution Order:
1. **Phase 1 - Interface Definition**: Define contracts and type signatures first
2. **Phase 2 - Parallel Development**: Once interfaces are defined:
   - Development team: Implement against defined interfaces
   - Testing team: Write tests using interface contracts (simultaneously)
3. **Coordination**: All teams sync through shared interface documentation
```

## Important Notes

### TDD and Testing Philosophy

- **No Mock Services**: All tests use real services and real data - no mocking allowed
- **Test-First Development**: Unit tests can be written before implementation based on specifications
- **Red-Green-Refactor**: Tests initially fail (red), implementation makes them pass (green), then refactor
- **Real Dependencies**: Integration and E2E tests require actual implementations and services

### Agent Independence

- **Separate Agent Execution**: Use different Claude agents for dev and test work to avoid cross-contamination
- **Interface-Only Coordination**: Agents communicate only through shared interface contracts, not implementation details
- **No Context Sharing**: Dev agents don't see test code, test agents don't see implementation code
- **Prevent Overfitting**: Independent execution ensures tests truly validate behavior, not implementation specifics

### Document Coordination

- **Three separate documents** enable parallel interface, development and testing work
- **Interface guide** defines contracts, types, and behavior specifications
- **Development guide** focuses on implementation tasks against defined interfaces
- **Testing guide** focuses on verification and validation using interface contracts and real systems
- All documents are local only - not synced to GitHub
- Documents cross-reference each other for coordination
- Teams can work independently while maintaining alignment through shared interface contracts

### Task Requirements

- Focus on concrete, actionable tasks with clear deliverables
- Include sufficient context for each task to be executed independently
- Consider both happy path and error scenarios in testing
- Specify real service dependencies and setup requirements

### Execution Recommendations

- **Use `/pm:issue-work` command**: Execute each guide with separate Claude instances:
  - `/pm:issue-work {issue_number} interface` - Define contracts first
  - `/pm:issue-work {issue_number} dev` - Implement against interfaces (parallel)
  - `/pm:issue-work {issue_number} test` - Test using interfaces (parallel)
- **Interface First**: Always complete interface definition before starting parallel dev/test work
- **Agent Isolation**: Use different agents for dev and test to ensure independence
- **Regular Interface Updates**: If interfaces change, update all three documents and notify teams
