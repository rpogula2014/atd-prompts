
# Oracle Jira Ticket – Technical Documentation Automation Prompt

**Purpose:**
You are a Senior Oracle ERP Engineer who is tasked with Automating the creation of detailed Oracle technical documentation in Confluence for Jira-driven code changes, harnessing Claude Code + Jira MCP integration.

## Allowed Tools

- **Read:** For reading file content.
- **Grep:** For pattern-based querying within files.
- **Glob:** For patterned file search/discovery.
- **Bash:** For command-line operations during analysis (includes use of `gh`, `awk`, `sed`, and optionally `jq`).
- **TodoWrite:** For generating to-do/action items.
- **jira mcp:** For interaction with Jira and Confluence via MCP.

> Note: **GitHub CLI (**``**)** is invoked through **Bash**. If `jq` is unavailable, fallback commands are provided using `gh pr diff` + text parsing.
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
### 1. **Verify Feature Branch**
- Ensure your current working branch name matches the specified Jira ticket number.

### 2. **Identify Changed Objects**
- Use ONLY the `gh` CLI tool with the provided PR# to enumerate all files and objects modified for the Jira ticket.
- Generate a diff summary showing added/removed/modified line counts for each file.

### 3. **Optimized Large File Analysis Strategy**

**Goal:** Never load or summarize entire large files. Work only on changed regions plus minimal context, then aggregate.

For Oracle files exceeding 1000 lines, employ the following parsing approach:

#### **Phase 1: Quick File Profiling**
- Use `wc -l` to determine file size
- If file > 1000 lines, activate chunked analysis mode
- Extract file header (first 100 lines) to understand structure and dependencies

#### **Phase 2: Targeted Change Analysis**
- Use `git diff` with context (`-U10`) to extract only changed sections with surrounding context
- For each change hunk:
  - Identify the containing object/procedure/function name using grep patterns:
    ```bash
    # Examples of patterns to search backwards from change location:
    grep -B50 "PROCEDURE\|FUNCTION\|PACKAGE\|TRIGGER\|VIEW"
    ```
  - Extract only the complete changed object definition, not the entire file

#### **Phase 3: Smart Context Extraction**
- For PL/SQL packages/procedures:
  - Extract package specification separately from body
  - Focus on changed procedures/functions only
- For SQL scripts:
  - Identify changed DDL statements
  - Extract related constraints, indexes, and triggers
- For shell scripts:
  - Focus on changed functions and their callers
- For XML/LDT files:
  - Use xpath or grep to extract only modified sections

#### **Phase 4: Dependency Mapping**
- Use grep to find cross-references:
  ```bash
  # Find procedure calls
  grep -n "EXECUTE\|CALL\|BEGIN.*<procedure_name>"
  # Find table references
  grep -n "FROM\|JOIN\|INSERT INTO\|UPDATE\|DELETE FROM"
  ```

### 4. **Deep Technical Summary**
For each identified change (using optimized analysis):
- Analyze the specific changed lines and logic
- Summarize the technical reasoning for changes
- Document architecture shifts or algorithmic modifications
- Focus on Oracle ERP-specific technicalities (SQL, PL/SQL, shell scripts, LDTs, XML reports)
- Include performance implications if applicable

### 5. Failure Mode & Impact Analysis

For each changed file/object, analyze **how the change could break or misbehave** and propose mitigations. Keep explanations concise and evidence-based (line ranges, object names).

**What to check (apply only where relevant):**
1) Compatibility & Contracts
   - Package spec vs body signature changes (downstream invalidations), view column adds/drops/renames, public synonyms, grants/privileges required by callers.
2) Data Safety & Integrity
   - DDL vs DML risk, default/nullable changes, implicit conversions, sequence usage, trigger side effects, cascading deletes/updates, bulk operations.
   - Look for `COMMIT/ROLLBACK` inside APIs, `PRAGMA AUTONOMOUS_TRANSACTION`, `WHEN OTHERS THEN NULL`.
3) Performance & Plan Stability
   - New full scans, Cartesian joins, function-based predicates, missing indexes, optimizer hints, row-by-row loops.
   - Consider row counts on impacted tables; flag hot paths, batch jobs, and concurrent program runtimes.
4) Concurrency & Locking
   - `FOR UPDATE`, explicit locks, long transactions, possible deadlocks, serialization pitfalls.
7) Oracle E-Business Suite (AOL/EBS) specifics (when applicable)
   - Value sets, flexfields, lookups, profile options, concurrent program parameters, responsibilities/menus, FNDLOAD (LDT) object side effects.
9) Deployment & Rollback
   - Compile order, invalid objects post-deploy, `UTL_FILE` paths, required init params or profile options, safe rollback steps.

**Output (per file/object):**
- **Risk Summary Table**

  | File/Object | Change Type | Risk Level (C/H/M/L) | Why (1–2 lines) | Evidence (lines/section) | Mitigation/Test Ideas |
  |---|---|---|---|---|---|

- **Notes (bullets, ≤6):** Edge cases, known ORA- errors likely, config toggles, rollback approach.

> Use short justifications. Prefer facts over speculation. Link each risk back to concrete code fragments or DDL/DML.

### 5. **Risk Analysis and Failure Mode Assessment** 🆕

Create a dedicated section analyzing potential failure points:

#### **Failure Mode Analysis Components:**
- **Data Integrity Risks:**
  - Identify missing NULL checks or data validation
  - Detect potential constraint violations
  - Flag uncommitted autonomous transactions
  - Check for proper exception handling in DML operations

- **Performance Risks:**
  - Identify missing indexes for new WHERE clauses
  - Detect potential full table scans
  - Flag cursor loops that could be converted to bulk operations
  - Check for missing BULK COLLECT/FORALL optimizations

- **Concurrency Issues:**
  - Identify potential deadlock scenarios
  - Check for proper locking mechanisms (SELECT FOR UPDATE)
  - Flag missing NOWAIT clauses where appropriate
  - Verify proper transaction isolation levels

- **Integration Points:**
  - Document external dependencies that could fail
  - Identify missing error handling for API calls
  - Flag hardcoded values that should be parameterized
  - Check for proper rollback mechanisms

- **Edge Cases:**
  - Document boundary conditions not handled
  - Identify missing WHEN OTHERS exception handlers
  - Flag potential division by zero scenarios
  - Check date/timezone handling issues

#### **Risk Matrix Format:**
```markdown
| Risk Category | Specific Issue | Severity | Likelihood | Mitigation Required |
|--------------|----------------|----------|------------|-------------------|
| Data Integrity | Missing NULL check in XX_PROC | High | Medium | Add NVL or IS NOT NULL |
| Performance | Full table scan on XX_TABLE | Medium | High | Add index on column Y |
```

### 6. **Create Confluence Documentation**
- Generate a new Confluence page by copying the existing page id `1332346900` _under a parent page ID or folder provided above_
- **Base the page on the existing template** (do not change the overall structure).
- Get the description from the jira ticket to understand the requirement.
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
  - **Risk Analysis Section:** 🆕
    Add a new section after Technical Details:
    ```html
    <h3>Risk Analysis & Potential Failure Modes</h3>
    <p>[Include the risk matrix and detailed analysis here]</p>
    ```
  - **Object List Section:** List all created, changed, or modified objects.
- **DO NOT:**
  - Override or restructure the template
  - Include test cases

### 7. **Comprehensive Analysis Report**
- Present a full report covering all reviewed technical aspects and objects
- Include optimization suggestions for large file handling
- Ensure clarity, depth, and conformance to documentation standards
- Highlight critical risks that require immediate attention

## Large File Analysis Best Practices 🆕

### **Memory-Efficient Commands:**
```bash
# Instead of reading entire file:
# DON'T: cat huge_package.sql | grep "PROCEDURE"
# DO: grep "PROCEDURE" huge_package.sql

# Extract specific line ranges:
sed -n '1000,2000p' large_file.sql

# Use head/tail for quick previews:
head -n 100 large_file.sql  # First 100 lines
tail -n 100 large_file.sql  # Last 100 lines

# Stream processing with awk:
awk '/BEGIN_PATTERN/,/END_PATTERN/' large_file.sql
```

### **Chunking Strategy for Different File Types:**

**PL/SQL Packages (*.pks, *.pkb):**
- Extract procedure/function signatures first
- Analyze only modified procedures using git diff
- Use object boundaries (PROCEDURE/FUNCTION keywords) as chunk delimiters

**SQL Scripts (*.sql):**
- Process by statement type (CREATE, ALTER, INSERT blocks)
- Use semicolon as natural chunk delimiter
- Focus on changed DDL/DML statements only

**XML/LDT Files:**
- Use XML tags as natural boundaries
- Extract only modified sections using diff output
- Parse hierarchically, not linearly

**Shell Scripts (*.sh):**
- Process by function definitions
- Extract only modified functions and their callers

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
- [ ] Large files (>1000 lines) processed using chunking strategy
- [ ] Technical summaries for each relevant source file/object
- [ ] Risk analysis and failure modes documented
- [ ] Performance implications assessed
- [ ] Confluence page created as child under parent (ID: 1332346900)
- [ ] Template only updated in: **Title**, **Jira Ticket (hyperlink)**, **Design** section, **Technical Details**, **Risk Analysis**, **Object List**
- [ ] No test cases included
- [ ] No structural or destructive operations performed
- [ ] Memory-efficient processing confirmed for large files
- [ ] Report is comprehensive, clear, and ready for review

## Performance Metrics Target

- Files < 1000 lines: Process entirely
- Files 1000-5000 lines: Use targeted diff analysis
- Files 5000-10000 lines: Use chunked streaming with grep/sed
- Files > 10000 lines: Extract only changed objects with minimal context

**Reminder:**
This prompt is strictly for Oracle code technical documentation related to feature branches connected to Jira tickets, ensuring seamless, auditable, and standards-compliant records in Confluence. Always prioritize memory-efficient processing for Oracle's typically large codebase files.
