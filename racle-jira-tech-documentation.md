
# Oracle Jira Ticket – Technical Documentation Automation Prompt

**Purpose:**
Automate the creation of detailed Oracle technical documentation in Confluence for Jira-driven code changes, harnessing Claude Code + Jira MCP integration.

## Allowed Tools

- **Read:** For reading file content.
- **Grep:** For pattern-based querying within files.
- **Glob:** For patterned file search/discovery.
- **Bash:** For command-line operations during analysis.
- **TodoWrite:** For generating to-do/action items.
- **jira mcp:** For interaction with Jira and Confluence via MCP.

## Prompt Name

```
/rahul:oracle-jira-tech-documentation - Comprehensive documentation of Oracle code in confluence
```

## Usage

```bash
/rahul:oracle-jira-tech-documentation [jiraticket] [--pr#] [--path]
```

**Arguments:**

- `jiraticket` – The Jira ticket number associated with the change.
- `--pr#` – (Optional) Github pull request number; used to extract a list of modified objects.
- `--path` – (Optional) Destination folder/path for the new Confluence page.

## Execution
1. **Verify Feature Branch**
    - Ensure your current working branch name matches the specified Jira ticket number.

2. **Identify Changed Objects**
    - Use the `gh` CLI tool with the provided PR# to enumerate all files and objects modified for the Jira ticket.

3. **Deep Technical Summary**
    - For each listed file/object (ignore binary files):
        - Analyze lines and changes in detail.
        - Summarize the technical logic and reasoning for changes, architecture shifts, or algorithmic modifications.
        - Focus on Oracle erp-related technicalities (e.g., SQL, PL/SQL, shell scripts, ldts, xml reports and so on).

4. **Create Confluence Documentation**
    - Generate a new Confluence page by copying the existing page id `1332346900` _under a parent page ID or folder provided above_
    - **Base the page on the existing template** (do not change the overall structure).
    - Update only the following fields in the template:
        - **Title:** Set as `<Jira Ticket #> : <Jira Ticket Description>`
          Example: `EBS-12345 : Testing creating new template`
        - **Jira Ticket Section:** Add a hyperlink to the Jira ticket.
        - **Design Section:**
                Update the content within the
            ```html
            <h3>Design</h3><p></p>
            ```
            paragraph to reflect code change design decisions.
        - **Technical Details Section:**
                Update the content within the
            ```html
            <h3>Technical Details</h3><p></p>
            ```
            Provide detailed technical analysis of the code changes.
        - **Object List Section:** List all created, changed, or modified objects.
    - **DO NOT:**
        - Override or restructure the template
        - Include test cases

5. **Comprehensive Analysis Report**
    - Present a full report covering all reviewed technical aspects and objects.
    - Ensure clarity, depth, and conformance to documentation standards.

## Prohibited Actions (“DO NOT Execute List”)

- **Never delete** any Confluence page or Jira ticket.
- **Never change** the Git repository (no commits, merges, deletions, etc.).
- **Only document**, do not perform any destructive or modifying operations.

## Integration Notes

- The prompt leverages:
    - **Glob:** For recursive file discovery.
    - **Grep:** To search for key patterns (e.g., `CREATE TABLE`, `ALTER PROCEDURE`).
    - **Read:** For extracting substantive code blocks and comments for documentation.
    - **jira mcp:** To automate page creation and link Jira/Confluence assets.
- **All output** is structured for clarity and direct transfer to Confluence with correct formatting.

## Review Checklist

- [ ] Feature branch matches Jira ticket number
- [ ] All changed objects extracted using `gh` tool
- [ ] Technical summaries for each relevant source file/object
- [ ] Confluence page created as child under parent (ID: 1332346900)
- [ ] Template only updated in:**Title**, **Jira Ticket (hyperlink)**, **Design** section, **Technical Details**, **Object List**
- [ ] No test cases included
- [ ] No structural or destructive operations performed
- [ ] Report is comprehensive, clear, and ready for review

**Reminder:**
This prompt is strictly for Oracle code technical documentation related to feature branches connected to Jira tickets, ensuring seamless, auditable, and standards-compliant records in Confluence.
