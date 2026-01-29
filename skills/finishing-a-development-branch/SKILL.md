---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for integration, PR, or cleanup
---

# Finishing a Development Branch

## Overview

Guide completion of development work by presenting clear options and handling chosen workflow.

**Core principle:** Verify tests → Present options → Execute choice → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / uv run pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with integration/PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Determine Base Revision

```bash
# Check where your work diverged from trunk
jj log

# Identify the base - typically trunk() or the 'main' bookmark
jj log -r "trunk()"
jj log -r "bookmarks(exact:main)"
```

Or ask: "This work is based on main - is that correct?"

### Step 3: Present Options

Present exactly 4 options:

```
Implementation complete. What would you like to do?

1. Integrate into main locally (rebase + move bookmark)
2. Push and create a Pull Request
3. Keep the work as-is (I'll handle it later)
4. Discard this work

Which option?
```

**Don't add explanation** - keep options concise.

### Step 4: Execute Choice

#### Option 1: Integrate Locally

```bash
# Rebase your changes onto trunk if needed
jj rebase -r <your-changes> -d 'trunk()'

# Move the main bookmark forward
jj bookmark set main -r <your-final-change>
# Or if the user has the 'tug' alias:
jj tug

# Verify tests on integrated result
<test command>

# Verify
jj log -r 'trunk()..@'

# Start fresh
jj new
```

Then: Cleanup workspace (Step 5), then Post-Integration Cleanup (Step 6)

#### Option 2: Push and Create PR

```bash
# Ensure change has good description
jj desc -r <your-change> -m "<meaningful description>"

# Set a bookmark on your work if not already set
jj bookmark set <feature-name> -r <your-change>

# Push the bookmark
jj git push --remote origin -b <feature-name> --allow-new

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

**Keep the workspace** until PR is merged. Don't cleanup.

#### Option 3: Keep As-Is

```bash
# Ensure it has a good description so you remember what it is
jj desc -r <your-change> -m "WIP: <what this is and what's left to do>"

# Optionally bookmark it for easy reference
jj bookmark set wip-<feature-name> -r <your-change>
```

Report: "Keeping work at revision <change-id>. Workspace preserved at <path>."

**Don't cleanup workspace.**

#### Option 4: Discard

**Confirm first:**
```
This will permanently discard:
- Changes: <change-list>
- Workspace at <path> (if applicable)

Type 'discard' to confirm.
```

Wait for exact confirmation.

If confirmed:
```bash
jj abandon <revisions-to-discard>
```

Then: Cleanup workspace (Step 5)

### Step 5: Cleanup Workspace

**For Options 1, 2, 4:**

Check if working in a jj workspace:
```bash
jj workspace list
```

If in a workspace other than the default:
```bash
# Return to main workspace
cd <repo-root>

# Forget the workspace record
jj workspace forget <workspace-name>

# Remove the directory (verify path first!)
rm -rf <workspace-path>
```

**For Option 3:** Keep workspace.

### Step 6: Post-Integration Cleanup

After integration, especially if there were conflicts:

```bash
# Clean any stale jj artifacts from git's index
# (jj uses git for storage, conflict markers can get stuck)
git status --porcelain | grep -E "\.jj-" && git rm --cached .jj-* 2>/dev/null

# Verify environment still works
direnv allow 2>/dev/null

# Run the actual app, not just tests
# (tests use temp dirs, won't catch missing production paths)
make serve  # or equivalent
```

## Quick Reference

| Option | Integrate | Push | Keep Workspace | Cleanup | Post-Integration |
|--------|-----------|------|----------------|---------|------------------|
| 1. Integrate locally | ✓ | - | - | ✓ | ✓ |
| 2. Create PR | - | ✓ | ✓ | - | - |
| 3. Keep as-is | - | - | ✓ | - | - |
| 4. Discard | - | - | - | ✓ | - |

## Common Mistakes

**Skipping test verification**
- **Problem:** Integrate broken code, create failing PR
- **Fix:** Always verify tests before offering options

**Open-ended questions**
- **Problem:** "What should I do next?" → ambiguous
- **Fix:** Present exactly 4 structured options

**Automatic workspace cleanup**
- **Problem:** Remove workspace when might need it (Option 2, 3)
- **Fix:** Only cleanup for Options 1 and 4

**No confirmation for discard**
- **Problem:** Accidentally delete work
- **Fix:** Require typed "discard" confirmation

## Red Flags

**Never:**
- Proceed with failing tests
- Integrate without verifying tests on result
- Delete work without confirmation
- Force-push without explicit request
- Use `git merge`, `git checkout`, `git branch` - use jj equivalents
- Delete workspace directory before `jj workspace forget`
- Cleanup workspace when PR is pending review (Option 2)

**Always:**
- Verify tests before offering options
- Present exactly 4 options
- Get typed confirmation for Option 4
- Clean up workspace for Options 1 & 4 only
- Use meaningful descriptions on changes
- Confirm workspace path before `rm -rf`
- Run actual application after integration, not just tests

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - After all tasks complete
- **executing-plans** (Step 5) - After all batches complete

**Pairs with:**
- **using-jj-workspaces** - Cleans up workspace created by that skill
