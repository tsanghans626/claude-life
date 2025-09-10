---
allowed-tools: Bash, Read, Write, Edit, MultiEdit, Glob, Grep, TodoWrite, Task
---

# Issue Work

Complete implementation of a GitHub issue following interface-first development with parallel dev/test execution.

## Usage

```
/pm:issue-work <issue_number>
```

- `<issue_number>`: GitHub issue number

**Execution Flow:**

1. **Phase 1**: Implement interface contracts (`{issue}-interface.md`)
2. **Phase 2**: Launch parallel dev and test agents in separate worktrees
3. **Phase 3**: Merge worktrees and run comprehensive testing

## Quick Check

1. **Parse arguments:**

   ```bash
   ISSUE_NUMBER="$ARGUMENTS"

   # Validate issue number is provided
   if [ -z "$ISSUE_NUMBER" ]; then
     echo "❌ Issue number is required"
     echo "Usage: /pm:issue-work <issue_number>"
     exit 1
   fi

   # Validate issue number is numeric
   if ! [[ "$ISSUE_NUMBER" =~ ^[0-9]+$ ]]; then
     echo "❌ Issue number must be numeric: $ISSUE_NUMBER"
     exit 1
   fi
   ```

2. **Check required documentation files:**

   ```bash
   # Find interface, dev, and test documentation files
   INTERFACE_FILE=""
   DEV_FILE=""
   TEST_FILE=""

   for epic_dir in .claude/epics/*/; do
     if [ -f "$epic_dir/${ISSUE_NUMBER}-interface.md" ]; then
       INTERFACE_FILE="$epic_dir/${ISSUE_NUMBER}-interface.md"
     fi
     if [ -f "$epic_dir/${ISSUE_NUMBER}-dev.md" ]; then
       DEV_FILE="$epic_dir/${ISSUE_NUMBER}-dev.md"
     fi
     if [ -f "$epic_dir/${ISSUE_NUMBER}-test.md" ]; then
       TEST_FILE="$epic_dir/${ISSUE_NUMBER}-test.md"
     fi
   done

   # Validate all three files exist
   missing_files=()
   [ -z "$INTERFACE_FILE" ] && missing_files+=("${ISSUE_NUMBER}-interface.md")
   [ -z "$DEV_FILE" ] && missing_files+=("${ISSUE_NUMBER}-dev.md")
   [ -z "$TEST_FILE" ] && missing_files+=("${ISSUE_NUMBER}-test.md")

   if [ ${#missing_files[@]} -gt 0 ]; then
     echo "❌ Missing documentation files: ${missing_files[*]}"
     echo "Please run /pm:issue-decompose $ISSUE_NUMBER first"
     exit 1
   fi

   echo "📋 Found documentation files:"
   echo "   Interface: $INTERFACE_FILE"
   echo "   Development: $DEV_FILE"
   echo "   Testing: $TEST_FILE"
   ```

3. **Skip GitHub API calls:**
   - Do not fetch issue details from GitHub API
   - Use only local documentation and files

4. **Ready to proceed:**
   - All validation checks passed
   - Documentation file found and ready to use

## Instructions

### Phase 1: Interface Implementation

#### 1.1 Setup Main Branch

```bash
# Create simple branch name for issue
BRANCH_NAME="issue-$ISSUE_NUMBER"

# Check if branch exists, if not create it
if ! git show-ref --verify --quiet refs/heads/$BRANCH_NAME; then
  git checkout -b "$BRANCH_NAME"
  echo "✅ Created new branch: $BRANCH_NAME"
else
  echo "ℹ️ Branch $BRANCH_NAME already exists"
  git checkout "$BRANCH_NAME"
fi

echo "🎯 Starting Phase 1: Interface Implementation"
echo "📋 Interface Guide: $INTERFACE_FILE"

# Assign issue to self and mark in-progress
gh issue edit $ISSUE_NUMBER --add-assignee @me --add-label "in-progress" || true
```

#### 1.2 Implement Interface Contracts

**CRITICAL: Complete ALL interface definition work in this phase. Do NOT proceed to Phase 2 until interfaces are fully defined.**

Use TodoWrite to create interface tasks:

- Parse the interface documentation file for contract requirements
- Convert to actionable interface definition todos
- Focus on type signatures, behavior contracts, and integration specs

```bash
echo "🔌 Phase 1: Implementing interface contracts"
echo "📋 Reading interface guide: $INTERFACE_FILE"
```

**Implementation Requirements:**

- Create all TypeScript interfaces and type definitions
- Define function signatures with input/output contracts
- Specify error types and handling contracts
- Define data schemas with validation rules
- Document behavior contracts for each interface
- NO IMPLEMENTATION CODE - only signatures and contracts

#### 1.3 Commit Interface Contracts

After completing interface definitions:

```bash
# Commit interface contracts to main branch
git add .
git commit -m "Issue #$ISSUE_NUMBER[interface]: Define contracts and type signatures

🔌 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

echo "✅ Phase 1 Complete: Interface contracts defined and committed"
```

### Phase 2: Parallel Development and Testing

#### 2.1 Setup Parallel Worktrees

**CRITICAL: Create separate worktrees for development and testing to enable parallel work.**

**IMPORTANT: All file operations must be performed with absolute paths relative to the WORKTREE_DIR since the shell resets working directory after each command.**

```bash
# Create separate worktrees for dev and test work
DEV_WORKTREE="../${PWD##*/}-dev"
TEST_WORKTREE="../${PWD##*/}-test"

# Clean up any existing worktrees
git worktree remove "$DEV_WORKTREE" --force 2>/dev/null || true
git worktree remove "$TEST_WORKTREE" --force 2>/dev/null || true

# Create fresh worktrees from current branch (with interface contracts)
git worktree add "$DEV_WORKTREE" "$BRANCH_NAME"
git worktree add "$TEST_WORKTREE" "$BRANCH_NAME"

# Get absolute paths
DEV_PATH="$(realpath "$DEV_WORKTREE")"
TEST_PATH="$(realpath "$TEST_WORKTREE")"

echo "🎯 Phase 2: Parallel development setup complete"
echo "   Development worktree: $DEV_PATH"
echo "   Testing worktree: $TEST_PATH"
```

#### 2.2 Launch Parallel Development Agents

**CRITICAL: Use separate agents to avoid context contamination between development and testing.**

Launch development agent:

```bash
echo "🚀 Launching development agent..."
# Use Task tool to launch development agent with DEV_FILE
```

Launch testing agent (simultaneously):

```bash
echo "🧪 Launching testing agent..."
# Use Task tool to launch testing agent with TEST_FILE
```

**Agent Requirements:**

- **Development Agent**: Implement functions against interface contracts, no access to test code
- **Testing Agent**: Write tests using interface contracts, no access to implementation
- **Independent Work**: Each agent works in its own worktree with no shared context
- **Interface Coordination**: Both agents use the same interface contracts for consistency

**Implementation Requirements:**

- ALL file operations (Read, Write, Edit, MultiEdit) must use absolute paths: `$WORKTREE_PATH/relative/path`
  - Example: `Read file_path="$WORKTREE_PATH/ai-info/src/server/trpc.ts"`
  - Example: `Write file_path="$WORKTREE_PATH/ai-info/src/services/rss/index.ts"`
- ALL bash commands must start with `cd "$WORKTREE_PATH" && ...`
  - Example: `cd "$WORKTREE_PATH" && pnpm install fast-xml-parser`
  - Example: `cd "$WORKTREE_PATH/ai-info" && pnpm run lint`

### Phase 3: Merge and Integration Testing

#### 3.1 Wait for Parallel Completion

**CRITICAL: Do not proceed until both development and testing agents have completed their work.**

```bash
echo "⏳ Waiting for parallel development and testing to complete..."
echo "   Development agent should implement all interface contracts"
echo "   Testing agent should create comprehensive test suites"
echo "   Both agents should commit their changes to their respective worktrees"
```

#### 3.2 Merge Worktrees

**CRITICAL: Combine development and test work into the main branch.**

```bash
echo "🔄 Phase 3: Merging development and test work"

# Switch to main branch
git checkout "$BRANCH_NAME"

# Merge development work
echo "📥 Merging development implementation..."
cd "$DEV_PATH"
git add . && git commit -m "Issue #$ISSUE_NUMBER[dev]: Implementation complete" || true
cd -
git merge --no-ff -m "Merge development work from worktree" || {
  echo "❌ Merge conflict in development work - manual resolution required"
  exit 1
}

# Merge testing work
echo "🧪 Merging test implementation..."
cd "$TEST_PATH"
git add . && git commit -m "Issue #$ISSUE_NUMBER[test]: Test suite complete" || true
cd -
git merge --no-ff -m "Merge testing work from worktree" || {
  echo "❌ Merge conflict in testing work - manual resolution required"
  exit 1
}

echo "✅ Worktrees merged successfully"
```

#### 3.3 Run Integration Tests

**CRITICAL: Validate that implementation and tests work together.**

```bash
echo "🧪 Running comprehensive test suite..."

# Run all tests with real data
cd ai-info && pnpm test

# Run linting and type checking
cd ai-info && pnpm lint
cd ai-info && pnpm typecheck || pnpm tsc --noEmit

echo "✅ All validation passed"
```

#### 4.1 Clean Up Worktrees

```bash
echo "🧹 Cleaning up worktrees..."

# Remove development and testing worktrees
git worktree remove "$DEV_WORKTREE" --force || true
git worktree remove "$TEST_WORKTREE" --force || true
git worktree prune

echo "✅ Worktrees cleaned up"
```

#### 4.2 Final Commit and PR Creation

```bash
# Final commit with all integrated work
git add .
git commit -m "Issue #$ISSUE_NUMBER: Complete implementation with interface-first development

✅ Interface contracts defined
✅ Implementation completed
✅ Test suite implemented
✅ Integration testing passed

🔌 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

# Push to origin
git push -u origin HEAD

# Create pull request
PR_TITLE="Issue #$ISSUE_NUMBER: Interface-first development complete"

gh pr create --title "$PR_TITLE" --body "$(cat <<'EOF'
## Summary

Resolves #$ISSUE_NUMBER

Complete implementation using interface-first development methodology with parallel development and testing.

## Implementation Approach

**Phase 1 - Interface Definition:**
- Defined TypeScript interfaces and contracts
- Established behavior specifications
- Created data schemas and validation rules

**Phase 2 - Parallel Development:**
- Development agent implemented functions against interface contracts
- Testing agent created comprehensive test suites using interface contracts
- Independent execution in separate worktrees to prevent cross-contamination

**Phase 3 - Integration:**
- Merged development and testing work
- Ran comprehensive integration tests with real data
- Validated all functionality works together

## Key Features

- Interface-first development ensures consistency
- Real service integration (no mocking)
- Comprehensive test coverage
- Type-safe implementation
- Independent development and testing

## Testing Strategy

- Unit tests for all interface implementations
- Integration tests with real external services
- End-to-end workflow validation
- Error handling and edge case coverage
- Performance validation

Closes #$ISSUE_NUMBER
EOF
)"

# Remove in-progress label, keep assignee
gh issue edit $ISSUE_NUMBER --remove-label "in-progress"
```

### 5. Final Output

```
✅ Completed interface-first development for issue #$ISSUE_NUMBER

Implementation Summary:
═══════════════════════

Phase 1 - Interface Definition:
✅ TypeScript interfaces and contracts defined
✅ Behavior specifications documented
✅ Data schemas with validation created

Phase 2 - Parallel Development:
✅ Development agent implemented functionality
✅ Testing agent created comprehensive tests
✅ Independent execution prevented over-fitting

Phase 3 - Integration:
✅ Development and test work merged successfully
✅ Integration tests passed with real data
✅ All validation checks completed

Final Status:
- Branch: issue-{issue_number}
- Pull Request: Created and ready for review
- Tests: All passing with real data
- Implementation: Complete and validated
- Documentation: Interface contracts available

Ready for code review and deployment!
```

### 6. Post-Completion Status Update

After creating the PR, check if work is fully complete and update local status:

```bash
# Check if all todos are completed
if [ "all todos completed" ]; then
  # Find the local task file
  task_file=""
  for epic_dir in .claude/epics/*/; do
    if [ -f "$epic_dir/$ISSUE_NUMBER.md" ]; then
      task_file="$epic_dir/$ISSUE_NUMBER.md"
      break
    fi
  done

  if [ -n "$task_file" ]; then
    echo "🎯 All work completed. Updating local task status..."
    current_datetime=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

    # Update task status to closed (work is done, waiting for PR merge)
    sed -i '' "s/^status: .*/status: closed/" "$task_file"
    sed -i '' "s/^updated: .*/updated: $current_datetime/" "$task_file"

    echo "✅ Updated local task status to closed: $task_file"

    # Update epic progress
    epic_dir=$(dirname "$task_file")
    bash .claude/scripts/pm/update-epic-progress.sh "$(basename "$epic_dir")"

    echo "📊 Epic progress updated"
  fi
fi
```

### 7. Output

```
✅ Completed interface-first development on issue #$ISSUE_NUMBER

Methodology: Interface-First Development with Parallel Execution
Branch: issue-{number}
Implementation: COMPLETED with full integration
Documentation: Interface + Development + Testing guides
Status: Ready for code review

Completed phases:
- Phase 1: Interface contracts defined and committed
- Phase 2: Parallel development and testing with independent agents
- Phase 3: Integration testing with real services
- All validation passed (lint/typecheck/tests)
- Pull request created with comprehensive documentation

Final status:
- Branch pushed to origin with integrated work
- Pull request created with detailed implementation summary
- Local task status updated to closed
- Epic progress recalculated
- Ready for code review and deployment
```

## Error Handling

If any step fails, provide clear guidance:

- Git conflicts: "❌ Branch exists. Run: git checkout existing-branch"
- Missing tools: "❌ gh CLI not found. Install: brew install gh"
- Permission issues: "❌ Cannot assign issue. Check repository permissions"

## Worktree Management

### List Worktrees

```bash
git worktree list
```

### Remove Worktree (when done)

```bash
# Clean up dev worktree
git worktree remove ../${PWD##*/}-dev

# Clean up test worktree
git worktree remove ../${PWD##*/}-test

# Prune deleted worktrees
git worktree prune
```

### Switch Between Worktrees

```bash
# Go to dev environment
cd ../${PWD##*/}-dev

# Go to test environment
cd ../${PWD##*/}-test

# Back to main
cd ../${PWD##*/}
```

## Important Notes

- Uses worktrees for parallel dev/test environments
- Single branch without -dev/-test suffix
- Multiple Claude Code instances can work simultaneously
- Shared .git directory, separate working directories
- TodoWrite helps track progress through acceptance criteria
- Auto-links PR to issue for closure
- Follows conventional commit message format
- Validates changes before completion
- **CRITICAL: Do not call `gh issue view` or any other GitHub API commands to fetch issue details**
- Use only local documentation files ({issue}-{work_type}.md) for workflow instructions
- Branch names and PR titles should use issue number only, not title from GitHub
