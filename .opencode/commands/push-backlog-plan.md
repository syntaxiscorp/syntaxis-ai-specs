---
description: Push backlog plan to repository
---

# Role

You are an expert Product Manager with extensive experience in project management tools (Jira, GitHub Projects) and agile workflow automation.

# Questions

You should ask me about which Jira space to integrate the epics/tasks/subtasks

# Target Platform

$ARGUMENTS

`$ARGUMENTS` must be one of: `jira` or `github`. This determines where the backlog will be pushed.

- If `$ARGUMENTS` is `jira`: Use the Jira MCP to create Epics and Stories.
- If `$ARGUMENTS` is `github`: Use the GitHub CLI (`gh`) to create a GitHub Project with issues.

# Goal

Push the product backlog (Epics and enriched User Stories) from `ai-specs/backlog/` to the specified project management platform, creating all Epics and User Stories as trackable work items.

# Prerequisites

Before running this command, ensure:
1. The backlog has been created using `/create-backlog-plan`
2. User Stories have been enriched using `/enrich-us` for each story
3. The backlog folder `ai-specs/backlog/` exists and contains:
   - `backlog.md` — the backlog summary with Epics and prioritized story list
   - `us-XXX.md` — individual User Story files

# Process and rules

1. Read `ai-specs/backlog/backlog.md` to get the full backlog structure (Epics, story order, dependencies, release phases).
2. Read all `ai-specs/backlog/us-*.md` files to get individual User Story details.
3. If the `ai-specs/diagrams/` folder exists, read diagram files to enrich the items pushed to the platform:
   - `use-cases.md` — Attach use case references to each story's description
   - `sequence-diagrams.md` — Include relevant sequence diagram snippets in story descriptions for developer context
   - `entity-relationship.md` — Reference affected entities in story descriptions
   - `c4-architecture.md` — Add architecture context to Epic descriptions
   - `lean-canvas.md` — Include business context in Epic descriptions
4. Validate that all stories referenced in `backlog.md` have corresponding `us-XXX.md` files. Report any missing files and stop if critical stories are missing.
4. Confirm with the user before creating items on the remote platform — show a summary of what will be created (number of Epics, number of Stories, target platform).

## If target is `jira`:

5. Use the Jira MCP to:
   a. Create a new Epic for each Epic defined in `backlog.md`:
      - **Summary**: Epic name
      - **Description**: Epic description + business objective + list of child stories (see ADF format requirement below)
      - **Priority**: Mapped from MoSCoW (Must=Highest, Should=High, Could=Medium, Won't=Low)
   b. Create a Story for each `us-XXX.md` file:
      - **Summary**: Story title (from `# US-XXX: [Title]`)
      - **Description**: Full story content (story statement, acceptance criteria, technical notes, NFRs, dependencies, DoD) — see ADF format requirement below
      - **Story Points**: From estimation section
      - **Priority**: Mapped from MoSCoW
      - **Epic Link**: Link to the parent Epic created in step (a)
      - **Labels**: Add labels for release phase (e.g., `mvp`, `v1.1`)
   c. Create dependency links between stories (e.g., "is blocked by" relationships)
   d. If a Jira board/project does not exist, inform the user and ask for the project key

   ### CRITICAL: Jira description format — always use ADF, never markdown strings

   **Never** pass descriptions as plain markdown strings with `contentFormat: "markdown"`. Multiline strings in JSON use `\n` escape sequences that the Jira MCP double-escapes, rendering as literal `\n` text in Jira (broken formatting).

   **Always** use `contentFormat: "adf"` and pass the description as an Atlassian Document Format (ADF) JSON object:

   ```json
   {
     "version": 1,
     "type": "doc",
     "content": [
       { "type": "heading", "attrs": { "level": 2 }, "content": [{ "type": "text", "text": "Story" }] },
       { "type": "paragraph", "content": [{ "type": "text", "text": "As a ... I want ... so that ..." }] },
       { "type": "heading", "attrs": { "level": 2 }, "content": [{ "type": "text", "text": "Acceptance Criteria" }] },
       { "type": "heading", "attrs": { "level": 3 }, "content": [{ "type": "text", "text": "Scenario 1: ..." }] },
       { "type": "paragraph", "content": [{ "type": "text", "text": "Given ... When ... Then ..." }] },
       { "type": "heading", "attrs": { "level": 2 }, "content": [{ "type": "text", "text": "Technical Notes" }] },
       {
         "type": "bulletList",
         "content": [
           { "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Note 1" }] }] },
           { "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Note 2" }] }] }
         ]
       },
       { "type": "heading", "attrs": { "level": 2 }, "content": [{ "type": "text", "text": "Definition of Done" }] },
       {
         "type": "bulletList",
         "content": [
           { "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Criterion 1" }] }] }
         ]
       }
     ]
   }
   ```

   ADF node types to use:
   - `heading` with `attrs.level` 2 or 3 for section headers
   - `paragraph` for body text (story statement, scenario prose, single notes)
   - `bulletList` → `listItem` → `paragraph` for unordered lists (technical notes, DoD items, dependencies)
   - `orderedList` → `listItem` → `paragraph` for numbered steps if needed

   All text content goes in `{ "type": "text", "text": "..." }` leaf nodes — **never** embed newlines (`\n`) inside text values.

6. After creation, output a summary:
   - Number of Epics created with their Jira keys
   - Number of Stories created with their Jira keys
   - Any stories that failed to create (with error details)
   - Link to the Jira board

## If target is `github`:

5. Use the GitHub CLI (`gh`) to:
   a. Create a GitHub Project (if one doesn't exist for this product):
      - `gh project create --title "[Product Name] Backlog" --owner [org/user]`
   b. Create a GitHub Issue for each Epic:
      - **Title**: `[Epic] [Epic Name]`
      - **Body**: Epic description + business objective + list of child story issue numbers
      - **Labels**: `epic`, priority label (e.g., `priority:must-have`)
   c. Create a GitHub Issue for each `us-XXX.md` file:
      - **Title**: `US-XXX: [Story Title]`
      - **Body**: Full story content (story statement, acceptance criteria, technical notes, NFRs, dependencies, DoD)
      - **Labels**: `user-story`, priority label, release phase label (e.g., `mvp`), story points label (e.g., `points:5`)
      - **Milestone**: Set to release phase if milestones exist
   d. Add all created issues to the GitHub Project
   e. Create cross-references between dependent stories (mention blocked-by issues in the body)
   f. If using GitHub Projects V2, set custom fields for story points and priority

6. After creation, output a summary:
   - Number of Epic issues created with their issue numbers
   - Number of Story issues created with their issue numbers
   - Any issues that failed to create (with error details)
   - Link to the GitHub Project board

# Post-execution

7. Update `ai-specs/backlog/backlog.md` adding a new section at the top:

```markdown
## Sync Status
- **Platform**: [jira|github]
- **Last Synced**: [date]
- **Epics Created**: [count] — [list of IDs/keys]
- **Stories Created**: [count] — [list of IDs/keys]
- **Stories Pending**: [count if any failed]
```

8. Update each `us-XXX.md` file adding the external tracking ID:

```markdown
## Tracking
- **Platform**: [jira|github]
- **External ID**: [JIRA-KEY or #issue-number]
- **URL**: [link to the item]
```

# Error handling

- If the MCP or CLI is not available, inform the user and provide manual instructions.
- If authentication fails, guide the user on how to authenticate (`gh auth login` for GitHub, Jira MCP setup for Jira).
- Never expose credentials or tokens in output.
- If a story fails to create, continue with the remaining stories and report failures at the end.
