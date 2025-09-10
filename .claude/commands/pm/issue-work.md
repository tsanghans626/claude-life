---
allowed-tools: Bash, Read, Write, Edit, MultiEdit, Glob, Grep, TodoWrite, Task
---

# Issue Work

Start working on a GitHub issue using traditional development workflow (no parallel agents or worktrees).

## Usage

```
/pm:issue-work <issue_number>
```

## Quick Check

1. **Get issue details:**

   ```bash
   gh issue view $ARGUMENTS --json state,title,labels,body
   ```

   If it fails: "❌ Cannot access issue #$ARGUMENTS. Check number or run: gh auth login"

2. **Check if already assigned:**
   - If issue is assigned to someone else, ask user if they want to continue
   - If issue has "in-progress" label, warn about potential conflicts

3. **Check and update local task status:**

   ```bash
   # Check GitHub issue status first
   issue_state=$(gh issue view $ARGUMENTS --json state --jq '.state')
   if [ "$issue_state" = "closed" ]; then
     echo "⚠️ GitHub issue #$ARGUMENTS is closed. Updating local status..."

     # Find the local task file
     task_file=""
     for epic_dir in .claude/epics/*/; do
       if [ -f "$epic_dir/$ARGUMENTS.md" ]; then
         task_file="$epic_dir/$ARGUMENTS.md"
         break
       fi
     done

     if [ -n "$task_file" ]; then
       # Update task status to closed
       current_datetime=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

       # Update frontmatter status and updated timestamp
       sed -i '' "s/^status: .*/status: closed/" "$task_file"
       sed -i '' "s/^updated: .*/updated: $current_datetime/" "$task_file"

       echo "✅ Updated local task status to closed: $task_file"

       # Update epic progress
       epic_dir=$(dirname "$task_file")
       bash .claude/scripts/pm/update-epic-progress.sh "$(basename "$epic_dir")"

       echo "✅ Issue #$ARGUMENTS is already completed. Local files updated."
       exit 0
     else
       echo "⚠️ GitHub issue is closed but no local task file found"
     fi
   fi

   # If issue is open, ensure local task file status is also open
   task_file=""
   for epic_dir in .claude/epics/*/; do
     if [ -f "$epic_dir/$ARGUMENTS.md" ]; then
       task_file="$epic_dir/$ARGUMENTS.md"
       break
     fi
   done

   if [ -n "$task_file" ]; then
     current_status=$(grep "^status:" "$task_file" | head -1 | sed 's/^status: *//')
     if [ "$current_status" = "closed" ]; then
       # Reopen local task if GitHub issue is open
       current_datetime=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
       sed -i '' "s/^status: .*/status: open/" "$task_file"
       sed -i '' "s/^updated: .*/updated: $current_datetime/" "$task_file"
       echo "🔄 Reopened local task status to match GitHub issue"
     fi
   fi
   ```

## Instructions

### 1. Setup Branch

```bash
# Create and switch to feature branch
git checkout -b "issue-$ARGUMENTS-$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-\|-$//g')"

# Assign issue to self and mark in-progress
gh issue edit $ARGUMENTS --add-assignee @me --add-label "in-progress"
```

### 2. Create Todo List

Use TodoWrite to create task breakdown from issue acceptance criteria:

- Parse issue body for acceptance criteria (- [ ] items)
- Convert to actionable todos
- Add testing and validation tasks

### 3. Work Implementation

For each todo:

1. Mark as in_progress
2. Implement the feature/fix
3. Test the changes
4. Mark as completed
5. Commit with format: "Issue #$ARGUMENTS: {specific change}"

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

# Create pull request
gh pr create --title "Issue #$ARGUMENTS: $TITLE" --body "$(cat <<'EOF'
## Summary

Resolves #$ARGUMENTS

{Brief description of changes}

## Changes Made

{List key changes from todo completion}

## Testing

{Testing approach and results}

Closes #$ARGUMENTS
EOF
)"

# Remove in-progress label, keep assignee
gh issue edit $ARGUMENTS --remove-label "in-progress"
```

### 6. Post-Completion Status Update

After creating the PR, check if work is fully complete and update local status:

```bash
# Check if all todos are completed
if [ "all todos completed" ]; then
  # Find the local task file
  task_file=""
  for epic_dir in .claude/epics/*/; do
    if [ -f "$epic_dir/$ARGUMENTS.md" ]; then
      task_file="$epic_dir/$ARGUMENTS.md"
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
✅ Started work on issue #$ARGUMENTS: $TITLE

Branch: issue-$ARGUMENTS-{sanitized-title}
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
