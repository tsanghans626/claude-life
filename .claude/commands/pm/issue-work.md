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
  - `dev`: Execute functional implementation coding
  - `test`: Execute test case coding
  - `review`: Run test cases and validate functionality

**Phase-Based Execution:**

Each type executes a specific phase independently to manage context efficiently:

- `dev`: Phase 1 - Implement functionality against interface contracts
- `test`: Phase 2 - Write test cases using interface contracts
- `review`: Phase 3 - Execute tests and validate implementation

**Prerequisites:**

- Interface contracts must be implemented via `/pm:issue-decompose` before dev/test phases
- Both dev and test phases must be completed before review phase

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
     echo "Types: dev, test, review"
     exit 1
   fi

   case "$TYPE" in
     dev|test|review)
       echo "✅ Valid type: $TYPE"
       ;;
     *)
       echo "❌ Invalid type: $TYPE"
       echo "Valid types: dev, test, review"
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
     dev)
       REQUIRED_SUFFIX="-dev.md"
       ;;
     test)
       REQUIRED_SUFFIX="-test.md"
       ;;
     review)
       # For review, we need both dev and test files to exist
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

   # Check for interface file (required for dev and test)
   if [ "$TYPE" = "dev" ] || [ "$TYPE" = "test" ]; then
     INTERFACE_FILE=""
     for epic_dir in .claude/epics/*/; do
       candidate_file="$epic_dir/${ISSUE_NUMBER}-interface.md"
       if [ -f "$candidate_file" ]; then
         INTERFACE_FILE="$candidate_file"
         break
       fi
     done

     if [ -z "$INTERFACE_FILE" ]; then
       echo "❌ Missing interface documentation: ${ISSUE_NUMBER}-interface.md"
       echo "Please run /pm:issue-decompose $ISSUE_NUMBER first"
       exit 1
     fi
   fi

   # For review, check that both dev and test are completed
   if [ "$TYPE" = "review" ]; then
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

     # Check if dev phase is completed
     if ! grep -q "^status: completed" "$DEV_FILE" 2>/dev/null; then
       echo "❌ Development phase not completed"
       echo "Please run /pm:issue-work $ISSUE_NUMBER dev first"
       exit 1
     fi

     # Check if test phase is completed
     if ! grep -q "^status: completed" "$REQUIRED_FILE" 2>/dev/null; then
       echo "❌ Test phase not completed"
       echo "Please run /pm:issue-work $ISSUE_NUMBER test first"
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
  dev)
    echo "⚡ Executing Phase 1: Development Implementation"
    ;;
  test)
    echo "🧪 Executing Phase 2: Test Case Implementation"
    ;;
  review)
    echo "🚀 Executing Phase 3: Test Execution and Validation"
    ;;
esac
```

### Type: dev

**Purpose:** Implement functionality against existing interface contracts.

#### Prerequisites Check

```bash
# Interface contracts already verified in Quick Check
BRANCH_NAME="issue-$ISSUE_NUMBER"

if ! git show-ref --verify --quiet refs/heads/$BRANCH_NAME; then
  git checkout -b "$BRANCH_NAME"
  echo "✅ Created new branch: $BRANCH_NAME"
else
  git checkout "$BRANCH_NAME"
  echo "ℹ️ Using existing branch: $BRANCH_NAME"
fi

echo "⚡ Reading development guide: $REQUIRED_FILE"
echo "📋 Interface contracts available from decomposition phase"
gh issue edit $ISSUE_NUMBER --add-assignee @me --add-label "in-progress" || true
```

#### Implementation Requirements

**CRITICAL: Use Documentation Only - No Code Analysis**

- **Read the development guide file directly**: Use the Read tool on `$REQUIRED_FILE` to get implementation requirements
- **Read interface contracts**: Use the Read tool on `{issue}-interface.md` to understand interfaces
- **DO NOT analyze existing codebase** - use only the documentation files
- **DO NOT use Task tool to analyze code** - documentation contains all needed information

**Development Implementation Rules:**

- Implement functions against interface contracts from documentation
- Follow existing interface signatures exactly as specified in interface docs
- Handle all specified error cases from contract specifications
- Validate against data schemas from interface documentation
- **NO TEST CODE** - implementation only

#### Commit and Status Update

```bash
git add .
git commit -m "Issue #$ISSUE_NUMBER[dev]: Implementation complete

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

# Update status in dev.md file
sed -i '' 's/^status: .*/status: completed/' "$REQUIRED_FILE"

echo "✅ Development implementation completed and status updated"
```

### Type: test

**Purpose:** Write comprehensive test cases using interface contracts.

#### Prerequisites Check

```bash
# Interface contracts already verified in Quick Check
BRANCH_NAME="issue-$ISSUE_NUMBER"
git checkout "$BRANCH_NAME"

echo "🧪 Reading test guide: $REQUIRED_FILE"
echo "📋 Interface contracts available for test design"
gh issue edit $ISSUE_NUMBER --add-assignee @me --add-label "in-progress" || true
```

#### Implementation Requirements

**CRITICAL: Use Documentation Only - No Code Analysis**

- **Read the test guide file directly**: Use the Read tool on `$REQUIRED_FILE` to get test requirements
- **Read interface contracts**: Use the Read tool on `{issue}-interface.md` to understand interfaces
- **DO NOT analyze existing codebase** - use only the documentation files
- **DO NOT use Task tool to analyze code** - documentation contains all needed information

**Test Implementation Rules:**

- Write tests using interface contracts from documentation
- Test all interface methods and behaviors specified in docs
- Include error handling test cases from contract specifications
- Test edge cases and boundary conditions from requirements
- **NO IMPLEMENTATION CODE** - tests only
- Use real services (no mocking)

#### Commit and Status Update

```bash
git add .
git commit -m "Issue #$ISSUE_NUMBER[test]: Test suite complete

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

# Update status in test.md file
sed -i '' 's/^status: .*/status: completed/' "$REQUIRED_FILE"

echo "✅ Test cases completed and status updated"
```

### Type: review

**Purpose:** Execute tests and validate complete functionality.

#### Prerequisites Check

```bash
# Development and test completion already verified in Quick Check
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

✅ Interface contracts defined (via decomposition)
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
- ✅ Interface contracts defined (via decomposition)
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

1. `dev` - Implement functionality
2. `test` - Create test cases
3. `review` - Run tests and create PR

**Status Management:**

- Each phase updates its documentation file status to "completed" upon successful execution
- Review phase checks for completion status before proceeding
- Status ensures proper execution order and prevents incomplete phases

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
