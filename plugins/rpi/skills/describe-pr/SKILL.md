---
name: describe-pr
description: Generate PR Description in CI (no user input)
---

# Generate PR Description

You are tasked with generating a comprehensive pull request description and creating a PR using the repository's standard template with rich linking and deviation analysis.

## Steps to follow:

1. **Read the PR description template:**

   `Read({SKILLBASE}/references/pr_description_template.md)`

2. **Identify or create the PR to describe:**
   - Check if the current branch has an associated PR: `gh pr view --json url,number,title,state,headRefName 2>/dev/null`
   - If a PR exists for the current branch, use that PR and continue to step 3.
   - If no PR exists for the current branch, inspect the current branch/worktree before asking the user:
     - Check current branch and worktree state: `git status --short --branch`
     - If the current branch contains the task implementation, create the PR for the current branch. Do **not** stop after listing open PRs and do **not** ask which PR to use.
     - If there are uncommitted implementation changes that belong to this task, commit them first using the repository's normal git safety protocol, then push the branch and create the PR.
     - If the branch has commits but no upstream/PR, push the branch with `git push -u origin <current-branch>` and create the PR.
   - Only list open PRs with `gh pr list --limit 10 --json number,title,headRefName,author` and ask the user to select one when the current branch does not contain relevant work and there is no safe current-branch PR to create.

3. **Gather PR metadata:**
   - Get PR info: `gh pr view {number} --json url,number,title,state,baseRefName,headRefName,commits,files`
   - Get repo info: `gh repo view --json owner,name`
   - Store the PR URL for diff link generation

4. **Discover task directory and ticket:**
   - Get branch name from PR: extract `headRefName` from step 2
   - Extract task slug (strip prefix before `/`, e.g., `dexter/eng-2612-feature` -> `eng-2612-feature`)
   - Extract ticket ID (e.g., `ENG-2612` or `LL-25` from the slug)
   - Check for task directory: `ls -La .humanlayer/tasks/ | grep -i "{ticket-id}"`
   - Get ticket URL: check `ticket.md` for ticket URL
       - If no URL found, skip — this is fine
   - If task directory exists, set `TASK_DIR`
   - Note: Task-level cloud URLs (artifacts page + deep link) are automatically provided when this skill is invoked via the Skill tool hook. Use these for the header links in the template.

5. **Gather comprehensive PR information:**
   - Get full PR diff: `gh pr diff {number}`
   - Read through the entire diff carefully
   - For context, read any files referenced but not shown in the diff
   - Understand the purpose and impact of each change
   - Identify user-facing changes vs internal implementation details

7. **Analyze for plan deviations (if plan file exists):**
   - Check if task directory has a plan file: `ls -La .humanlayer/tasks/{task-dir}/*plan*.md 2>/dev/null`
   - If plan file exists, use the Task tool with `subagent_type=implementation-reviewer`:
     ```
     Analyze deviations between the plan at .humanlayer/tasks/{task-dir}/{plan-file}
     and the current implementation. Compare against the base branch.
     ```
   - Include the agent's output in the "Deviations from the plan" section

8. **Determine output path:**
   - If task directory exists: `.humanlayer/tasks/{task-slug}/pr-description.md`
   - If no task directory: `.humanlayer/tasks/pr-{number}/description.md`

9. **Generate the description:**
   Fill out each section from the template:
   - **Header links**: Include ticket link, artifacts cloud URL, and task deep link (provided automatically via hook when this skill was invoked)
   - **What problems**: Based on ticket/plan context and code changes
   - **What user-facing changes**: Bulleted list with diff permalinks from step 5
   - **How I implemented it**: Journey through the PR with file/line permalinks
   - **Deviations from plan**: Include agent output from step 7 (or "No plan file found")
   - **How to verify it**: Include worktree setup commands with actual branch name
   - **Changelog entry**: Concise one-line summary

10. **Save the description:**
    - Write the completed description to the path from step 8
    - A cloud permalink will be automatically provided in the hook response
    - Show the generated description

11. **Create the PR if it does not already exist**
    - Always read the template from `{SKILLBASE}/references/pr_description_template.md`
    - If step 2 already found or created a PR, skip creation here and use that PR.
    - If no PR exists yet, create it with the `gh` CLI: `gh pr create --base <default branch> --head <current branch> --title <concise title>`
    - A missing PR is not, by itself, a reason to stop. Create the PR for the current task branch whenever the branch/worktree clearly contains the work being described.

12. **Update the PR:**
    - Update PR: `gh pr edit {number} --body-file {output-path}`
    - Confirm the update was successful

13. **Update the user:**
    - Read key artifacts (ticket file, research, design discussion, structured outline, plan) using the `Read` tool but only read a few lines NOT the entire artifact - this will give you the cloud URLs for each of them.
    - Read the final output template:
      `Read({SKILLBASE}/references/describe_pr_final_answer.md)`
    - Respond with a summary following the template, including the PR URL and key details including `RPI_TASK_ARTIFACTS_URLS` as the URLS you obtained by reading the artifacts

## Important notes:
- Always read the template from `{SKILLBASE}/references/pr_description_template.md`
- Use the `implementation-reviewer` agent for deviation analysis when a plan exists
- Focus on the "why" as much as the "what"
- Include breaking changes or migration notes prominently

Remember, you must respond to the user according to the output template at `{SKILLBASE}/references/describe_pr_final_answer.md`
