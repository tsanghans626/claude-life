---
allowed-tools: Bash, Read, Write, Edit, MultiEdit, Glob, Grep, TodoWrite, Task
---

# Issue Work

Execute specific phases of GitHub issue implementation to optimize context usage and prevent OOM issues.

## Usage

```
/pm:issue-work <issue_number> <type>
```

- `<issue_number>`: GitHub issue number
- `<type>`: Work phase type
  - `interface`: Implement interface contracts and type signatures
  - `dev`: Execute functional implementation coding
  - `write-test`: Execute test case coding
  - `exec-test`: Run test cases and validate functionality

**Phase-Based Execution:**

Each type executes a specific phase independently to manage context efficiently:

- `interface`: Phase 1 - Define contracts and type signatures
- `dev`: Phase 2A - Implement functionality against contracts
- `write-test`: Phase 2B - Write test cases using contracts
- `exec-test`: Phase 3 - Execute tests and validate implementation

## Quick Check

1. **Parse arguments:**

   ```bash
   # Parse space-separated arguments
   read -r ISSUE_NUMBER TYPE <<< "$ARGUMENTS"

   # Validate issue number is provided
   if [ -z "$ISSUE_NUMBER" ]; then
     echo "❌ Issue number is required"
     echo "Usage: /pm:issue-work <issue_number> <type>"
     exit 1
   fi

   # Validate issue number is numeric
   if ! [[ "$ISSUE_NUMBER" =~ ^[0-9]+$ ]]; then
     echo "❌ Issue number must be numeric: $ISSUE_NUMBER"
     exit 1
   fi

   # Validate type is provided and valid
   if [ -z "$TYPE" ]; then
     echo "❌ Type is required"
     echo "Usage: /pm:issue-work <issue_number> <type>"
     echo "Types: interface, dev, write-test, exec-test"
     exit 1
   fi

   case "$TYPE" in
     interface|dev|write-test|exec-test)
       echo "✅ Valid type: $TYPE"
       ;;
     *)
       echo "❌ Invalid type: $TYPE"
       echo "Valid types: interface, dev, write-test, exec-test"
       exit 1
       ;;
   esac
   ```

2. **Check required documentation files:**

   ```bash
   # Find required documentation file based on type
   REQUIRED_FILE=""

   # Determine which file is needed for this type
   case "$TYPE" in
     interface)
       REQUIRED_SUFFIX="-interface.md"
       ;;
     dev)
       REQUIRED_SUFFIX="-dev.md"
       ;;
     write-test)
       REQUIRED_SUFFIX="-test.md"
       ;;
     exec-test)
       # For exec-test, we need both dev and test files to exist
       REQUIRED_SUFFIX="-test.md"
       ;;
   esac

   # Find the required documentation file
   for epic_dir in .claude/epics/*/; do
     candidate_file="$epic_dir/${ISSUE_NUMBER}${REQUIRED_SUFFIX}"
     if [ -f "$candidate_file" ]; then
       REQUIRED_FILE="$candidate_file"
       break
     fi
   done

   # Validate required file exists
   if [ -z "$REQUIRED_FILE" ]; then
     echo "❌ Missing documentation file: ${ISSUE_NUMBER}${REQUIRED_SUFFIX}"
     echo "Please run /pm:issue-decompose $ISSUE_NUMBER first"
     exit 1
   fi

   # For exec-test, also check that dev implementation exists
   if [ "$TYPE" = "exec-test" ]; then
     DEV_FILE=""
     for epic_dir in .claude/epics/*/; do
       candidate_file="$epic_dir/${ISSUE_NUMBER}-dev.md"
       if [ -f "$candidate_file" ]; then
         DEV_FILE="$candidate_file"
         break
       fi
     done

     if [ -z "$DEV_FILE" ]; then
       echo "❌ Missing development documentation: ${ISSUE_NUMBER}-dev.md"
       echo "Please run /pm:issue-work $ISSUE_NUMBER dev first"
       exit 1
     fi
   fi

   echo "📋 Found required documentation file:"
   echo "   File: $REQUIRED_FILE"
   echo "   Type: $TYPE"
   ```

3. **Skip GitHub API calls:**
   - Do not fetch issue details from GitHub API
   - Use only local documentation and files

4. **Ready to proceed:**
   - All validation checks passed
   - Required documentation file found and ready to use

## Instructions

**CRITICAL:** Execute only the phase specified by the type parameter. This prevents context overflow and enables focused execution.

```bash
# Execute type-specific logic
case "$TYPE" in
  interface)
    echo "🔌 Executing Phase 1: Interface Implementation"
    ;;
  dev)
    echo "⚡ Executing Phase 2A: Development Implementation"
    ;;
  write-test)
    echo "🧪 Executing Phase 2B: Test Case Implementation"
    ;;
  exec-test)
    echo "🚀 Executing Phase 3: Test Execution and Validation"
    ;;
esac
```

### Type: interface

**Purpose:** Define TypeScript interfaces, contracts, and type signatures without implementation.

#### Setup Branch

```bash
BRANCH_NAME="issue-$ISSUE_NUMBER"

if ! git show-ref --verify --quiet refs/heads/$BRANCH_NAME; then
  git checkout -b "$BRANCH_NAME"
  echo "✅ Created new branch: $BRANCH_NAME"
else
  git checkout "$BRANCH_NAME"
  echo "ℹ️ Using existing branch: $BRANCH_NAME"
fi

echo "🔌 Reading interface guide: $REQUIRED_FILE"
gh issue edit $ISSUE_NUMBER --add-assignee @me --add-label "in-progress" || true
```

#### Implementation Requirements

- Create TypeScript interfaces and type definitions
- Define function signatures with input/output contracts
- Specify error types and handling contracts
- Define data schemas with validation rules
- Document behavior contracts
- **NO IMPLEMENTATION CODE** - only signatures and contracts

#### Commit

```bash
git add .
git commit -m "Issue #$ISSUE_NUMBER[interface]: Define contracts and type signatures

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

echo "✅ Interface contracts completed and committed"
```

### Type: dev

**Purpose:** Implement functionality against existing interface contracts.

#### Prerequisites Check

```bash
# Ensure interface contracts exist
if ! git log --oneline | grep -q "\[interface\]"; then
  echo "❌ Interface contracts not found. Run: /pm:issue-work $ISSUE_NUMBER interface"
  exit 1
fi

BRANCH_NAME="issue-$ISSUE_NUMBER"
git checkout "$BRANCH_NAME"

echo "⚡ Reading development guide: $REQUIRED_FILE"
echo "📋 Interface contracts available from previous commit"
```

#### Implementation Requirements

- Implement functions against interface contracts
- Follow existing interface signatures exactly
- Handle all specified error cases
- Validate against data schemas
- **NO TEST CODE** - implementation only

#### Commit

```bash
git add .
git commit -m "Issue #$ISSUE_NUMBER[dev]: Implementation complete

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

echo "✅ Development implementation completed"
```

### Type: write-test

**Purpose:** Write comprehensive test cases using interface contracts.

#### Prerequisites Check

```bash
# Ensure interface contracts exist
if ! git log --oneline | grep -q "\[interface\]"; then
  echo "❌ Interface contracts not found. Run: /pm:issue-work $ISSUE_NUMBER interface"
  exit 1
fi

BRANCH_NAME="issue-$ISSUE_NUMBER"
git checkout "$BRANCH_NAME"

echo "🧪 Reading test guide: $REQUIRED_FILE"
echo "📋 Interface contracts available for test design"
```

#### Implementation Requirements

- Write tests using interface contracts
- Test all interface methods and behaviors
- Include error handling test cases
- Test edge cases and boundary conditions
- **NO IMPLEMENTATION CODE** - tests only
- Use real services (no mocking)

#### Commit

```bash
git add .
git commit -m "Issue #$ISSUE_NUMBER[test]: Test suite complete

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

echo "✅ Test cases completed"
```

### Type: exec-test

**Purpose:** Execute tests and validate complete functionality.

#### Prerequisites Check

```bash
# Ensure development and test code exist
if ! git log --oneline | grep -q "\[dev\]"; then
  echo "❌ Development implementation not found. Run: /pm:issue-work $ISSUE_NUMBER dev"
  exit 1
fi

if ! git log --oneline | grep -q "\[test\]"; then
  echo "❌ Test cases not found. Run: /pm:issue-work $ISSUE_NUMBER write-test"
  exit 1
fi

BRANCH_NAME="issue-$ISSUE_NUMBER"
git checkout "$BRANCH_NAME"

echo "🚀 Running comprehensive test validation"
echo "📋 Test guide: $REQUIRED_FILE"
```

#### Validation Steps

```bash
# Run all tests
echo "🧪 Running test suite..."
cd ai-info && pnpm test

# Run linting
echo "🔍 Running lint checks..."
cd ai-info && pnpm lint

# Run type checking
echo "📝 Running type checks..."
cd ai-info && pnpm typecheck || pnpm tsc --noEmit

echo "✅ All validation passed"
```

#### Final Integration

```bash
# Final commit and PR creation
git add .
git commit -m "Issue #$ISSUE_NUMBER: Complete implementation with testing validation

✅ Interface contracts defined
✅ Implementation completed
✅ Test suite implemented
✅ All validation passed

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

# Push and create PR
git push -u origin HEAD

PR_TITLE="Issue #$ISSUE_NUMBER: Complete implementation"
gh pr create --title "$PR_TITLE" --body "$(cat <<'EOF'
## Summary
Resolves #$ISSUE_NUMBER

Complete implementation with interface-first development:
- ✅ Interface contracts defined
- ✅ Implementation completed
- ✅ Test suite implemented
- ✅ All validation passed

## Testing
- Real service integration (no mocking)
- Comprehensive test coverage
- All lint and type checks passed

Closes #$ISSUE_NUMBER
EOF
)"

gh issue edit $ISSUE_NUMBER --remove-label "in-progress"
echo "✅ Pull request created and ready for review"
```

## Error Handling

If any step fails, provide clear guidance:

- Git conflicts: "❌ Branch conflicts exist. Resolve manually before continuing"
- Missing tools: "❌ gh CLI not found. Install: brew install gh"
- Permission issues: "❌ Cannot assign issue. Check repository permissions"
- Missing prerequisites: "❌ Previous phase not completed. Run prerequisite command first"

## Important Notes

**Phase-Based Execution:**

- Each type executes one specific phase independently
- Prevents context overflow and OOM issues
- Enables focused implementation without distraction
- Clear separation of concerns between phases

**Execution Order:**

1. `interface` - Define contracts and types
2. `dev` - Implement functionality
3. `write-test` - Create test cases
4. `exec-test` - Run tests and create PR

**Context Management:**

- Each phase has minimal context requirements
- Uses only required documentation file for that phase
- No unnecessary parallel processing or worktree complexity
- Focused execution prevents memory issues

**Key Principles:**

- **CRITICAL: Do not call `gh issue view` or any GitHub API commands**
- Use only local documentation files ({issue}-{type}.md) for instructions
- Branch names use issue number only, not GitHub title
- TodoWrite tracks progress through acceptance criteria
- Auto-links PR to issue for closure
