# OpenAI Agents SDK SQL Planning Tools Implementation Prompt

Implement function tools for the existing agent built with the OpenAI Agents SDK.

## Project context

This application is a research management reporting system.

The agent service already exists under:

```text
app/services/agent/
```

The database knowledge base is stored under:

```text
table-wiki-okf/
```

The `table-wiki-okf` folder is an Obsidian vault containing Markdown files that follow, or are intended to follow, the OKF specification.

The vault may contain folders such as:

- `tables/`
- `relationships/`
- `queries/`
- `gateway/`
- `references/`
- `.obsidian/`

The Markdown files may contain:

- datasource definitions
- database names
- schema names
- table names
- column names
- data types
- column descriptions
- primary keys
- foreign keys
- table relationships
- business terminology
- aliases and synonyms
- example queries
- join guidance
- filtering rules
- aggregation rules
- reporting rules
- datasource or gateway metadata
- Obsidian links such as `[[table-name]]`
- standard Markdown links

## Goal

Add function tools that allow the agent to handle user questions that require SQL generation based on the knowledge in `table-wiki-okf`.

For the first version, the agent must:

1. Understand the user’s reporting or data question.
2. Search and inspect the relevant OKF/Obsidian Markdown files.
3. Identify the most appropriate datasource.
4. Identify the required tables, columns and relationships.
5. Produce a safe, read-only SQL statement.
6. Return the datasource, SQL, assumptions and a concise explanation.
7. Not execute the SQL.
8. Not call the actual datasource gateway yet.

The design must make it straightforward to add gateway execution in a later version.

Do not implement actual gateway query execution in this version.

## Required architecture

First inspect the existing codebase, especially:

- `app/services/agent/service.py`
- agent configuration models
- existing OpenAI Agents SDK setup
- current tool registration
- response models
- application configuration
- `table-wiki-okf` structure
- any existing gateway client

Reuse the project’s existing conventions instead of introducing an unrelated architecture.

Create a focused SQL planning/tool layer. Suggested modules may include:

```text
app/services/agent/tools/sql_tools.py
app/services/agent/sql_planner.py
app/services/agent/wiki_repository.py
app/models/sql_plan.py
```

The exact filenames may be adjusted to match the current project structure.

Do not put all new logic into `service.py`.

## OpenAI Agents SDK tools

Implement function tools using the tool mechanism already supported by the installed OpenAI Agents SDK version.

At minimum, provide tools equivalent to the following responsibilities.

### 1. `search_table_wiki`

Purpose:

Search the `table-wiki-okf` vault for files and sections relevant to a user question, business term, table, column, datasource or relationship.

Suggested input:

- `query`: string
- `datasource`: optional string
- `entity_names`: optional list of strings
- `max_results`: optional integer

Suggested output:

- matched files
- datasource candidates
- tables
- columns
- relationships
- relevant excerpts
- confidence or relevance information

The tool must understand:

- Obsidian wikilinks: `[[table-name]]`
- aliased wikilinks: `[[table-name|display name]]`
- relative Markdown links
- frontmatter where present
- headings and sections
- common OKF metadata
- table and column aliases

Ignore `.obsidian` configuration content unless it is required to resolve the vault.

### 2. `get_table_metadata`

Purpose:

Return normalized metadata for one or more tables.

Suggested input:

- `table_names`: list of strings
- `datasource`: optional string

Suggested output for each table:

- datasource
- database
- schema
- table name
- description
- columns
- data types
- nullable information when available
- primary keys
- foreign keys
- aliases
- business definitions
- source file

Do not invent missing metadata.

### 3. `get_relationships`

Purpose:

Find valid join paths among required tables.

Suggested input:

- `table_names`: list of strings
- `datasource`: optional string

Suggested output:

- relationship path
- left table and column
- right table and column
- relationship type
- suggested join type
- evidence/source file
- ambiguity warnings

Prefer explicitly documented relationships.

Do not infer a join only because two columns have similar names unless the output clearly labels it as an unverified assumption.

### 4. `build_sql_plan`

Purpose:

Create a structured SQL plan from the user’s question and the wiki metadata.

Suggested input:

- `user_question`: string
- `datasource`: optional string
- `preferred_sql_dialect`: optional string
- `row_limit`: optional integer

Suggested structured output:

- status
- interpreted request
- datasource
- SQL dialect
- tables
- selected columns
- joins
- filters
- grouping
- aggregations
- sorting
- row limit
- SQL
- assumptions
- warnings
- clarification questions
- supporting wiki files

This tool must generate SQL but must not execute it.

### 5. `validate_generated_sql`

Purpose:

Validate the generated SQL against the available wiki metadata and safety rules.

Suggested input:

- `sql`: string
- `datasource`: string
- `supporting_tables`: optional list of strings

Suggested output:

- `valid`: boolean
- validation errors
- unknown tables
- unknown columns
- unverified joins
- safety violations
- warnings

The validation does not need to connect to a database.

It should validate using the `table-wiki-okf` metadata.

## Structured models

Use Pydantic models for tool inputs and outputs where appropriate.

Create a result model similar to:

### `SqlPlanResult`

Fields:

- `status`
  - `ready`
  - `clarification_required`
  - `insufficient_metadata`
  - `unsupported`
  - `error`
- `interpreted_request`: string
- `datasource`: optional string
- `sql_dialect`: optional string
- `sql`: optional string
- `tables`: list
- `columns`: list
- `joins`: list
- `filters`: list
- `assumptions`: list of strings
- `warnings`: list of strings
- `clarification_questions`: list of strings
- `supporting_wiki_files`: list of strings

Return JSON-serializable data from tools.

## User request scenarios

The implementation must account for different user request patterns.

### Scenario 1: Direct table request

Example:

> Show the latest 100 research reports.

Expected behaviour:

- identify the research report table
- identify the relevant date column
- select useful display columns
- add descending date sorting
- limit the result
- return datasource and SQL

### Scenario 2: Business terminology instead of physical names

Example:

> Show reports waiting for sign-off.

Expected behaviour:

- map “waiting for sign-off” to documented statuses, tables and columns
- use wiki aliases, descriptions and business rules
- do not require the user to know table or column names

### Scenario 3: Aggregation

Example:

> How many reports did each analyst publish last month?

Expected behaviour:

- identify report and analyst information
- join only through documented relationships
- add date filtering
- group by analyst
- count reports

### Scenario 4: Multiple-table query

Example:

> Show report title, analyst, role and sign-off status.

Expected behaviour:

- identify all required tables
- find a valid relationship path
- clearly report any ambiguous join

### Scenario 5: Summary request

Example:

> Summarize report production by department this year.

Expected behaviour:

- build an aggregate SQL query
- explain which returned fields could be used for a summary
- do not fabricate a natural-language data summary because the SQL has not been executed
- clearly state that actual figures are not yet available

### Scenario 6: User provides a table name

Example:

> Query `pipelinerplus.workitems`.

Expected behaviour:

- verify that the table exists in the wiki
- qualify the table correctly
- select documented columns only

### Scenario 7: User provides partial SQL

Example:

> Add analyst name to this SQL: `SELECT report_id, title FROM ...`

Expected behaviour:

- validate the supplied tables and columns
- identify the required join
- return revised SQL
- preserve the user’s intent where safe

### Scenario 8: Ambiguous request

Example:

> Show active users.

Possible ambiguities:

- application users
- workflow users
- analysts
- employees
- active accounts
- users from different datasources

Expected behaviour:

- do not guess silently
- return `clarification_required`
- ask one or more focused clarification questions
- provide likely interpretations when useful

### Scenario 9: Missing metadata

Expected behaviour:

- return `insufficient_metadata`
- identify exactly which table, column, relationship or datasource definition is missing
- do not invent schema information

### Scenario 10: Multiple possible datasources

Expected behaviour:

- rank datasource candidates using wiki evidence
- automatically select one only when confidence is high
- otherwise request clarification

### Scenario 11: Unsupported non-data request

Example:

> Reset a user password.

Expected behaviour:

- do not generate SQL for an operational write action
- explain that the SQL reporting tool supports read-only analysis
- classify the request as unsupported or route it back to the general agent

### Scenario 12: Follow-up request

Example conversation:

User:

> Show reports published last month.

Then:

> Only include Hong Kong analysts.

Expected behaviour:

- preserve the previous query intent
- update the plan with the additional filter
- avoid rebuilding an unrelated query

## SQL generation rules

Generated SQL must be read-only.

Allow:

- `SELECT`
- `WITH` / CTE
- `JOIN`
- `WHERE`
- `GROUP BY`
- `HAVING`
- `ORDER BY`
- safe aggregate functions
- safe window functions where appropriate

Reject or avoid:

- `INSERT`
- `UPDATE`
- `DELETE`
- `MERGE`
- `DROP`
- `ALTER`
- `TRUNCATE`
- `CREATE`
- `GRANT`
- `REVOKE`
- `EXEC` / `EXECUTE`
- stored procedure execution
- multiple SQL statements
- comments intended to hide additional statements
- unrestricted `SELECT *` unless clearly justified

Additional rules:

- Use fully qualified table names when the wiki provides database or schema information.
- Use explicit columns instead of `SELECT *`.
- Use aliases that make joins readable.
- Add a sensible row limit for detail queries when the user does not provide one.
- Do not add a row limit to aggregate queries unless useful.
- Respect the datasource SQL dialect.
- Do not assume PostgreSQL, Oracle, SQL Server or MySQL syntax without evidence.
- If the dialect cannot be determined, use conservative ANSI SQL and include a warning.
- Escape identifiers according to the selected dialect only when necessary.
- Do not place user values directly into executable SQL when parameterization is possible.
- Represent dynamic values as named parameters, for example:

```sql
:start_date
:end_date
:status
```

- Return a parameters section when parameters are used.
- Never claim that SQL is valid against the live database because no live validation has occurred.

## Date handling

Interpret common relative date phrases, including:

- today
- yesterday
- this week
- last week
- this month
- last month
- this quarter
- last quarter
- this year
- last year
- year to date

Prefer explicit date-range parameters rather than database-specific date functions when this improves portability.

For example:

```sql
WHERE published_date >= :start_date
  AND published_date < :end_date
```

Include the interpreted date range in the plan.

## Wiki repository behaviour

Build a reusable repository/index abstraction for `table-wiki-okf`.

Requirements:

- vault root path must be configurable
- default to the current project’s `table-wiki-okf` folder
- recursively read Markdown files
- support UTF-8
- ignore irrelevant binary files
- parse YAML frontmatter when available
- parse Obsidian wikilinks
- parse Markdown links
- normalize paths safely
- prevent path traversal
- cache parsed metadata to avoid rereading the complete vault for every tool call
- invalidate or refresh the cache when source files change, if practical
- log parsing problems without crashing the agent
- preserve source file references for traceability

The first version may use an in-memory index.

Do not add a vector database unless the project already uses one.

A lightweight search approach is acceptable, using:

- normalized exact matches
- aliases
- token matching
- heading matching
- descriptions
- table and column names
- relationship references

## Agent orchestration

Update the existing agent instructions and tool registration so that the agent follows this process:

1. Determine whether the user’s question requires data retrieval or SQL.
2. Search the table wiki before generating SQL.
3. Resolve datasource, tables, columns and relationships.
4. Ask for clarification when essential information is ambiguous.
5. Build the SQL plan.
6. Validate the SQL.
7. Return the result to the user.
8. Do not execute the SQL.

Do not force SQL tools for ordinary conversational questions.

The agent should not generate schema details from its general model knowledge when the project wiki is expected to contain the answer.

## Response format

For a successful SQL request, return a user-facing response similar to:

```text
Datasource:
<datasource name>

SQL dialect:
<dialect>

SQL:
<generated SQL>

Parameters:
- start_date: ...
- end_date: ...

Tables used:
- schema.table_a
- schema.table_b

Assumptions:
- ...

Warnings:
- SQL was generated from the local table wiki.
- It has not been executed against the datasource.

Explanation:
<brief explanation of what the SQL returns>
```

For clarification:

```text
Status:
Clarification required

Questions:
1. Which datasource do you mean: ...?
2. Does “active user” mean ...?
```

For insufficient metadata:

```text
Status:
Insufficient metadata

Missing information:
- Relationship between table A and table B is not documented.

Relevant wiki files:
- ...
```

## Gateway preparation

Although gateway execution is out of scope, design the result so that a future execution tool can consume it.

The future execution input will likely require:

- datasource
- SQL
- parameters
- timeout
- maximum rows

Do not implement or invoke that execution in this task.

Consider defining an interface or protocol such as:

```python
SqlQueryExecutor
```

Do not create fake execution results.

## Logging and errors

Use the project’s existing logging conventions.

Log:

- tool invocation
- wiki files selected
- selected datasource
- selected tables
- validation failures
- parsing failures

Do not log:

- secrets
- credentials
- complete sensitive user prompts where avoidable
- datasource passwords
- access tokens

Convert internal exceptions into controlled tool results.

Do not expose stack traces to the end user.

## Testing

Add unit tests for at least:

1. Obsidian wikilink parsing.
2. Aliased wikilink parsing.
3. Frontmatter parsing.
4. Table metadata extraction.
5. Relationship extraction.
6. Exact table lookup.
7. Business-term lookup.
8. Multiple datasource ambiguity.
9. Missing relationship handling.
10. Read-only SQL validation.
11. Rejection of `UPDATE`, `DELETE`, `DROP` and multi-statement SQL.
12. Detail query with a default row limit.
13. Aggregate query generation.
14. Relative date interpretation.
15. Unknown column validation.
16. Follow-up filter application, if conversation context is handled in the current architecture.

Use temporary test vault files rather than depending only on the real `table-wiki-okf` content.

## Acceptance criteria

The work is complete when:

- the existing agent can call the new SQL-related function tools
- the tools read from `table-wiki-okf`
- Obsidian links are resolved
- datasource candidates can be identified
- tables and columns can be selected from documented metadata
- relationships can be resolved from the wiki
- a structured SQL plan is produced
- generated SQL is validated as read-only
- ambiguous requests produce clarification questions
- missing metadata is reported instead of invented
- the final response includes datasource and SQL
- no actual gateway or datasource query is executed
- the backend compiles
- existing tests continue to pass
- new tests pass
- no unnecessary changes are made to unrelated modules

## Implementation process

Proceed in this order:

1. Inspect the existing agent architecture and `table-wiki-okf` structure.
2. Summarize the proposed implementation plan.
3. Implement the wiki repository and normalized metadata models.
4. Implement the OpenAI Agents SDK function tools.
5. Implement SQL planning and validation.
6. Register the tools with the existing agent.
7. Update the agent instructions.
8. Add tests.
9. Run the relevant test and compile commands.
10. Report the changed files, design decisions, limitations and validation results.

Do not stop after only describing the solution. Implement it in the repository.

## Important constraints

- Do not execute SQL.
- Do not call the gateway.
- Do not fabricate query results.
- Do not fabricate tables, columns or relationships.
- Do not silently select an ambiguous datasource.
- Do not introduce a large framework for this first version.
- Keep the implementation modular so gateway execution can be added later.
- Preserve existing application behaviour outside the agent SQL workflow.
