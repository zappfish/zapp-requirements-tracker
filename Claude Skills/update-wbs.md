---
name: update-wbs
description: Update WBS diagram colors in the zapp-requirements-tracker README to match current project board statuses
disable-model-invocation: true
allowed-tools: Bash, Read, Edit, Write, Grep
---

# Update WBS Diagram Colors

Sync Mermaid WBS diagram node colors in the zapp-requirements-tracker README with current statuses from the GitHub Projects V2 board.

## Arguments

- `$ARGUMENTS` (optional): Path to the README file. Defaults to `~/zapp-requirements-tracker/README.md`.

## Instructions

### Step 1: Fetch project board data

Use `gh api graphql` to fetch all items and their Status field values from the ZAPP project board. The query must paginate to get all items.

```bash
gh api graphql --paginate -f query='
query($endCursor: String) {
  organization(login: "zappfish") {
    projectV2(number: 2) {
      items(first: 100, after: $endCursor) {
        pageInfo { hasNextPage endCursor }
        nodes {
          content {
            ... on Issue {
              number
            }
          }
          fieldValues(first: 20) {
            nodes {
              ... on ProjectV2ItemFieldSingleSelectValue {
                name
                field {
                  ... on ProjectV2SingleSelectField {
                    name
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}'
```

### Step 2: Build issue → status class lookup

From the query results, extract each item's issue number and its Status field value. Map status names to Mermaid class names using these mappings:

| Status Name   | Class Name     |
|---------------|----------------|
| Backlog       | `backlog`      |
| Up Next       | `upnext`       |
| In Progress   | `inprogress`   |
| In Review     | `inreview`     |
| Stable        | `stable`       |
| Done          | `done`         |

If a status name doesn't match any of the above (or is missing), default to `backlog`.

Build a lookup object/map: `{ issue_number: "classname", ... }` (e.g., `{ 23: "inprogress", 16: "backlog" }`).

### Step 3: Read the README

Read the file at the path from `$ARGUMENTS`, or default to `~/zapp-requirements-tracker/README.md`.

### Step 4: Update Mermaid node classes

For each Mermaid node in the README that matches the pattern `["#N"]:::classname`:

1. Extract the issue number `N`
2. Look up the current status class from the lookup built in Step 2
3. If the class differs from what's in the file, update it

Use the Edit tool to make replacements. For each issue whose class changed, replace the old pattern with the new one. Since issue numbers appear in multiple Mermaid diagrams (overview + detail), use `replace_all: true` for each replacement to update all occurrences.

Example: if issue #23 changed from `backlog` to `inprogress`, replace all occurrences of `["#23"]:::backlog` with `["#23"]:::inprogress`.

**Important**: Only update nodes where the class actually changed. Skip issues that are already correct.

### Step 5: Report summary

Print a summary table showing which issues changed status. Format:

```
## WBS Color Update Summary

| Issue | Old Status | New Status |
|-------|-----------|------------|
| #23   | backlog   | inprogress |
| #24   | backlog   | inreview   |

X issues updated, Y issues unchanged.
```

If no issues changed, report "All WBS colors are already up to date."

### Step 6: Create a PR

If any changes were made:

1. Navigate to the repo directory (`~/zapp-requirements-tracker`)
2. Create a new branch: `update-wbs-YYYY-MM-DD` (using today's date)
3. Stage and commit the README changes with message: `Update WBS diagram colors to match project board statuses`
4. Push the branch and create a PR:

```bash
gh pr create --title "Update WBS diagram colors" --body "$(cat <<'EOF'
## Summary
- Synced Mermaid WBS diagram node colors with current GitHub project board statuses

## Changes
[Include the summary table from Step 5 here]

Generated with `/update-wbs`
EOF
)"
```

5. Report the PR URL to the user.

If no changes were made, skip this step entirely.

## Error Handling

- If `gh api graphql` fails, check authentication with `gh auth status` and report the error
- If the README file doesn't exist at the specified path, report the error and suggest the correct path
- If an issue number in the README isn't found on the project board, leave its class unchanged and note it in the summary
