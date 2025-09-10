---
allowed-tools: Bash, Read, Write, Edit, MultiEdit, Glob, Grep, TodoWrite, Task
---

# Issue Work

Start working on a GitHub issue using worktree-based workflow for parallel dev/test environments.

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

3. **Skip GitHub API calls:**
   - Do not fetch issue details from GitHub API
   - Use only local documentation and files

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
# Create simple branch name without work type suffix
BRANCH_NAME="issue-$ISSUE_NUMBER"

# Check if branch exists, if not create it
if ! git show-ref --verify --quiet refs/heads/$BRANCH_NAME; then
  git checkout -b "$BRANCH_NAME"
  echo "✅ Created new branch: $BRANCH_NAME"
else
  echo "ℹ️ Branch $BRANCH_NAME already exists"
fi

# Create or switch to worktree based on work type
WORKTREE_DIR="../${PWD##*/}-$WORK_TYPE"

if [ ! -d "$WORKTREE_DIR" ]; then
  git worktree add "$WORKTREE_DIR" "$BRANCH_NAME"
  echo "✅ Created worktree: $WORKTREE_DIR"
else
  echo "ℹ️ Worktree already exists: $WORKTREE_DIR"
  # Ensure it's on the correct branch
  cd "$WORKTREE_DIR" && git checkout "$BRANCH_NAME"
  cd - > /dev/null
fi

echo "🔄 Switch to worktree directory: cd $WORKTREE_DIR"
echo "📝 This worktree is dedicated to $WORK_TYPE work on issue #$ISSUE_NUMBER"

# Assign issue to self and mark in-progress (only once, not per worktree)
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
# Ensure we're in the correct worktree
cd "$WORKTREE_DIR"

# Push branch
git push -u origin HEAD

# Create pull request title with work type (use issue number only)
PR_TITLE="Issue #$ISSUE_NUMBER [$WORK_TYPE]"

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
✅ Started work on issue #$ISSUE_NUMBER

Work Type: {dev|test}
Branch: issue-{number} (no suffix)
Worktree: ../{project}-{dev|test}
Documentation: {epic_dir}/{issue}-{work_type}.md
Status: in-progress

Next steps:
1. cd ../{project}-{work_type}  # Switch to dedicated worktree
2. Start Claude Code instance in that directory
3. Todo list will be created with {count} tasks
4. Begin implementation

Parallel work:
- Dev instance: ../{project}-dev
- Test instance: ../{project}-test
- Both work on same branch: issue-{number}

When complete:
- Push branch and create PR
- Local task status will be updated automatically
- Epic progress will be recalculated
- Clean up worktrees when no longer needed
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
