---
name: code-refine
description: Code review and cleanup skill that reviews all changed files for reuse, quality, and efficiency, then fixes any issues found.
disable-model-invocation: false
---

# Code-refine: Code Review and Cleanup

Review all changed files for reuse, quality, and efficiency. Fix any issues found.

## Phase 1: Identify Changes

Run `git diff` (or `git diff HEAD` if there are staged/unstaged changes) to see what changed. If there are no git changes, review the most recently modified files that the user mentioned or that you edited earlier in this conversation.

### GIT DIFF LOGIC PRIORITIZATION:
1. **EXPLICIT_ID**: If `commit_id` argument(s) are provided, determine the scope:
   - **Default (1 ID)**: Execute `git diff <id>^ <id>` (Review this specific commit only)
   - **Inclusive to HEAD** (1 ID + intent "from/including"): Execute `git diff <id>^ HEAD`
   - **Exclusive to HEAD** (1 ID + intent "after/since"): Execute `git diff <id> HEAD`
   - **Specific Range** (2 IDs): Execute `git diff <id1> <id2>`
2. **BRANCH_CHECK**: If both `target_branch` and `source_branch` are provided:
   - Execute `git diff <target_branch>...<source_branch>` (Standard PR review)
3. **UNCOMMITTED_CHECK**: If `git status --porcelain` is NOT empty:
   - **Step 1**: Execute `git add -N .` (Ensure untracked files are captured)
   - **Step 2**: Execute `git diff HEAD` (Review all staged and unstaged changes)
4. **FALLBACK**: If all above conditions fail (no inputs provided AND workspace is clean):
   - Execute `git diff HEAD~1 HEAD` (Review the most recent local commit)

**Save the diff to a temporary file with project name** (e.g., using `git diff > /tmp/git_diff_<project_name>.txt`) 

## Phase 2: Launch Three Review Agents in Parallel
Use the Agent tool to launch all three agents concurrently in a single message.

**For each agent, pass the diff file path from Phase 1** so it can read the changes. 

Example prompt format (replace with actual path)

```
The git diff is saved at: /tmp/git_diff_<project_name>.txt

Please read this file and review the changes for [reuse/quality/efficiency] issues...
```

The review agents:

1. **Code Reuse Reviewer** - Focus on duplications, missed opportunities to use existing utilities/functions
2. **Code Quality Reviewer** - Focus on clean code, maintainability, patterns
3. **Efficiency Reviewer** - Focus on performance, resource usage, concurrency

## Phase 3: Prioritize Issues

Wait for all the agents to complete. Aggregate their findings, assign a unique **Issue ID** (e.g., 1, 2) to each identified problem, and rank them by impact and effort:
1. **Critical**: Security vulnerabilities, bugs, performance-critical issues
2. **High**: Significant duplication, major quality issues, important optimizations
3. **Medium**: Moderate improvements, style violations, minor optimizations
4. **Low**: Cosmetic changes, trivial improvements

Present the prioritized list of issues grouped by severity, ensuring each issue clearly displays its ID, finding, location and a brief suggestion.

## Phase 4: Fix Issues

After presenting the prioritized list, **PAUSE and await user instructions**. Offer these 4 options:
* **Recommended Fix**: Address only Critical and High severity issues (or those most impactful).
* **Fix All**: Resolve all identified issues.
* **Selective Fix**: Address specific IDs (e.g., "fix 1, 3") or severity levels (e.g., "High+").
* **Skip**: Apply no fixes and proceed.

Wait for the user's input, then apply the chosen fixes. If a finding within the selected scope is a false positive or not worth addressing, note it and move on — do not argue with the finding, just skip it.

When done, briefly summarize what was fixed and reference the Issue IDs that were resolved (or confirm the code was already clean).

### Implementation Guidelines

**Preserve functionality:** All fixes must maintain existing behavior unless explicitly requested otherwise.
**Incremental changes:** Prefer small, focused improvements over massive rewrites.
**Test preservation:** Ensure all existing tests continue to pass after changes.
**Consistency:** Follow existing codebase patterns and conventions.
**Documentation:** Add or update comments to explain non-obvious logic.
**Performance focus:** Only optimize code that is proven to be performance-critical.