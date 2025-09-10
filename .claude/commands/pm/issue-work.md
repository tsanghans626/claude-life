---
allowed-tools: Bash, Read, Write, Edit, MultiEdit, Glob, Grep, TodoWrite, Task
---

# Issue Work

Start working on a GitHub issue using traditional development workflow (no parallel agents or worktrees).

## Usage

```
/pm:issue-work <issue_number> <dev|test>
```

- `<issue_number>`: GitHub issue number
- `<dev|test>` (required): Work type specification
  - `dev`: Follow development workflow using `{issue}-dev.md` documentation
  - `test`: Follow testing workflow using `{issue}-test.md` documentation

## Quick Check

1. **Parse arguments:**

   ```bash
   ISSUE_NUMBER=$(echo "$ARGUMENTS" | cut -d' ' -f1)
   WORK_TYPE=$(echo "$ARGUMENTS" | cut -d' ' -f2)

   # Validate that both parameters are provided
   if [ -z "$ISSUE_NUMBER" ] || [ -z "$WORK_TYPE" ]; then
     echo "❌ Both issue number and work type are required"
     echo "Usage: /pm:issue-work <issue_number> <dev|test>"
     exit 1
   fi

   # Validate work type
   if [ "$WORK_TYPE" != "dev" ] && [ "$WORK_TYPE" != "test" ]; then
     echo "❌ Invalid work type: $WORK_TYPE. Must be 'dev' or 'test'"
     exit 1
   fi
   ```

2. **Check work type documentation:**

   ```bash
   # Find the documentation file in epics directory
   DOC_FILE=""
   for epic_dir in .claude/epics/*/; do
     if [ -f "$epic_dir/${ISSUE_NUMBER}-${WORK_TYPE}.md" ]; then
       DOC_FILE="$epic_dir/${ISSUE_NUMBER}-${WORK_TYPE}.md"
       break
     fi
   done

   if [ -z "$DOC_FILE" ]; then
     echo "❌ Documentation file not found: ${ISSUE_NUMBER}-${WORK_TYPE}.md"
     echo "Please create the documentation file first in the appropriate epic directory."
     exit 1
   fi
   echo "📋 Using workflow documentation: $DOC_FILE"
   ```

3. **Check if already assigned:**
   - If issue is assigned to someone else, ask user if they want to continue
   - If issue has "in-progress" label, warn about potential conflicts

4. **Check local task status:**

   ```bash
   # Find the local task file
   task_file=""
   for epic_dir in .claude/epics/*/; do
     if [ -f "$epic_dir/$ISSUE_NUMBER.md" ]; then
       task_file="$epic_dir/$ISSUE_NUMBER.md"
       break
     fi
   done

   if [ -n "$task_file" ]; then
     current_status=$(grep "^status:" "$task_file" | head -1 | sed 's/^status: *//')
     if [ "$current_status" = "closed" ]; then
       echo "⚠️ Local task #$ISSUE_NUMBER is marked as closed."
       echo "Continue anyway to reopen and work on it? (y/n)"
     fi
   fi
   ```

## Instructions

### 1. Setup Branch

```bash
# Create branch name with work type suffix
BRANCH_NAME="issue-$ISSUE_NUMBER-$WORK_TYPE-$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-\|-$//g')"

# Create and switch to feature branch
git checkout -b "$BRANCH_NAME"

# Assign issue to self and mark in-progress
gh issue edit $ISSUE_NUMBER --add-assignee @me --add-label "in-progress"
```

### 2. Create Todo List

Use TodoWrite to create task breakdown:

- Parse the `{issue}-{work_type}.md` file for specific workflow instructions
- Convert to actionable todos
- Add testing and validation tasks

```bash
echo "📋 Creating todos based on $WORK_TYPE workflow documentation"
# Read and parse the specific documentation file from epics directory
# Found in $DOC_FILE variable from previous step
```

### 3. Work Implementation

For each todo:

1. Mark as in_progress
2. Implement the feature/fix
3. Test the changes
4. Mark as completed
5. Commit with format: "Issue #$ISSUE_NUMBER[$WORK_TYPE]: {specific change}"

### 4. Validation

Before finishing:

- Run linting: `npm run lint` (if exists)
- Run tests: `npm test` (if exists)
- Verify all acceptance criteria are met
- Check that no breaking changes introduced

### 5. Completion

```bash
# Push branch
git push -u origin HEAD

# Create pull request title with work type
PR_TITLE="Issue #$ISSUE_NUMBER [$WORK_TYPE]: $TITLE"

gh pr create --title "$PR_TITLE" --body "$(cat <<'EOF'
## Summary

Resolves #$ISSUE_NUMBER

{Brief description of changes}

## Work Type

{dev|test workflow}

## Changes Made

{List key changes from todo completion}

## Testing

{Testing approach and results}

Closes #$ISSUE_NUMBER
EOF
)"

# Remove in-progress label, keep assignee
gh issue edit $ISSUE_NUMBER --remove-label "in-progress"
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
✅ Started work on issue #$ISSUE_NUMBER: $TITLE

Work Type: {dev|test}
Branch: {branch-name}
Documentation: {epic_dir}/{issue}-{work_type}.md
Assigned: @me
Status: in-progress

Todo list created with {count} tasks.
Begin implementation with first todo.

When complete:
- Push branch and create PR
- Local task status will be updated automatically
- Epic progress will be recalculated
```

## Error Handling

If any step fails, provide clear guidance:

- Git conflicts: "❌ Branch exists. Run: git checkout existing-branch"
- Missing tools: "❌ gh CLI not found. Install: brew install gh"
- Permission issues: "❌ Cannot assign issue. Check repository permissions"

## Important Notes

- No worktrees required - uses traditional git branching
- TodoWrite helps track progress through acceptance criteria
- Auto-links PR to issue for closure
- Follows conventional commit message format
- Validates changes before completion
