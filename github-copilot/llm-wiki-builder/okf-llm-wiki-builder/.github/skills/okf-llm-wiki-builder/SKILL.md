---
name: okf-llm-wiki-builder
description: Convert documents, database schemas, SQL DDL, CSV, JSON, XML, APIs, business rules, metrics, and existing Markdown into a connected OKF-style LLM Wiki. Use this skill when creating or updating knowledge files, modelling relationships between concepts, documenting database semantics, building a semantic layer for question answering, or preparing trustworthy context for agents that generate read-only SQL.
---

# OKF LLM Wiki Builder

Use this skill to build and maintain a complete, connected, machine-readable Markdown knowledge base that other agents can use to answer business questions and generate safe SQL.

## Primary outcomes

1. Convert supplied content into stable Markdown concept files.
2. Separate independent concepts without over-fragmenting the wiki.
3. Create explicit, typed, bidirectional relationships between concepts.
4. Preserve technical identifiers and business meaning.
5. Build navigation and indexes for human and agent discovery.
6. Document enough database semantics for another agent to map a user question to valid read-only SQL.
7. Never invent facts, joins, constraints, formulas, ownership, or data meanings.

## Inputs

Handle:

- Plain text, documents, notes, policies, procedures, decisions, and business rules
- Database catalog exports, schemas, tables, views, columns, indexes, and SQL DDL
- CSV, JSON, XML, YAML, API specifications, API responses, and configuration files
- Existing Markdown or an existing OKF wiki
- Source-code models, DTOs, ORM entities, repository code, and query examples

## Working modes

Choose the smallest mode that satisfies the task:

- `create`: create a new wiki or concept files
- `update`: update existing concepts without unnecessary path changes
- `link`: add or repair relationships and indexes
- `audit`: find contradictions, missing metadata, broken links, orphan concepts, and unsafe SQL assumptions
- `semantic-model`: create the business-to-database mapping needed for question answering and SQL generation

## Repository discovery

Before writing:

1. Inspect the repository for an existing wiki root, OKF specification, templates, naming rules, and indexes.
2. Prefer existing conventions over this skill's defaults.
3. Search for existing concepts before creating new files.
4. Reuse stable paths and identifiers whenever possible.
5. Read relevant DDL, queries, models, documentation, and tests before inferring structure.
6. Treat sample data as observations, not schema declarations.

If the repository has no established location, use:

```text
wiki/
├── index.md
├── systems/
├── databases/
├── schemas/
├── tables/
├── views/
├── columns/
├── datasets/
├── apis/
├── business-concepts/
├── metrics/
├── dimensions/
├── relationships/
├── processes/
├── policies/
├── glossary/
├── queries/
├── sources/
└── validation/
```

Do not create empty directories.

## Concept-file rules

Each primary concept normally has one Markdown file.

Every concept file must:

- Use UTF-8 Markdown.
- Begin with valid YAML frontmatter.
- Include a non-empty `type`.
- Use a stable lowercase kebab-case path.
- Represent one primary concept.
- Be understandable without the original conversation.
- Preserve exact identifiers using backticks.
- Link to related concepts.
- Separate confirmed facts from observations and inferences.

Default frontmatter:

```yaml
---
type: Database Table
title: Customer Orders
description: One row per customer order.
canonical_id: table:sales.customer_orders
status: active
tags:
  - sales
  - orders
aliases:
  - customer order table
source_refs:
  - source:ddl/sales-schema.sql
---
```

Only `type` is universally required. Add other fields only when supported. Never fabricate values.

Recommended stable `canonical_id` forms:

- `system:<name>`
- `database:<name>`
- `schema:<database>.<schema>`
- `table:<schema>.<table>`
- `view:<schema>.<view>`
- `column:<schema>.<table>.<column>`
- `metric:<name>`
- `dimension:<name>`
- `business-concept:<name>`
- `process:<name>`
- `api:<name>`
- `endpoint:<method>:<path>`

Do not change a canonical ID merely because a title changes.

## Type system

Prefer descriptive types such as:

- System, Application, Service
- Database System, Database Schema, Database Table, Database View, Database Column
- Dataset, Data Dictionary, CSV Dataset, JSON Document, XML Document
- API, API Endpoint
- Business Concept, Business Rule, Metric, KPI, Dimension
- Process, Procedure, Runbook, Policy, Decision
- Reference, Source, Query Pattern, Glossary Term

Use repository-defined types when available.

## Relationship model

Relationships are first-class knowledge. Use the controlled vocabulary in `references/relationship-model.md`.

For each relationship:

1. Identify source concept, target concept, and relationship type.
2. State whether it is `confirmed`, `inferred`, `possible`, `deprecated`, or `conflicting`.
3. Preserve evidence or provenance.
4. Add the relationship to both related concept files when practical.
5. Add join details when the relationship supports SQL.
6. Never infer a foreign key solely from similar names.

Use this section in concept files:

```markdown
## Relationships

| Relationship | Target | Status | Evidence | Notes |
|---|---|---|---|---|
| belongs-to | [Sales schema](/schemas/sales.md) | confirmed | DDL | Physical ownership |
| references | [Customers](/tables/customers.md) | confirmed | FK `fk_orders_customer` | `customer_orders.customer_id = customers.customer_id` |
| provides-data-for | [Net Revenue](/metrics/net-revenue.md) | confirmed | Metric definition | Supplies order amounts |
```

For SQL-capable joins, include:

```markdown
### Join contract: Customers to Orders

- Cardinality: one-to-many
- Join type default: `INNER JOIN`
- Left key: `customers.customer_id`
- Right key: `customer_orders.customer_id`
- Null behaviour: orders without a valid customer are excluded by an inner join
- Status: confirmed
- Evidence: foreign-key constraint `fk_orders_customer`
- SQL:

```sql
JOIN sales.customer_orders o
  ON o.customer_id = c.customer_id
```
```

If the join is not confirmed, mark it unsafe for automatic SQL generation.

## Database-table template

Use these sections when relevant:

```markdown
# Overview

# Identity

- Database:
- Schema:
- Object:
- Object type:
- Row grain:
- Update pattern:
- Data latency:

# Business Meaning

# Schema

| Column | Data type | Nullable | Key | Default | Business meaning | Sensitivity |
|---|---|---:|---|---|---|---|

# Keys and Constraints

# Relationships

# Join Contracts

# Business Rules

# Data Quality

# Usage

# Safe Query Examples

# Security and Access Notes

# Source and Evidence

# Open Questions
```

For every table, determine or explicitly mark unknown:

- Row grain
- Primary key
- Time columns and timezone
- Soft-delete or active-record rules
- Tenant/security boundaries
- Duplicate behaviour
- Slowly changing dimension behaviour
- Valid status values
- Units and currencies
- Data freshness
- Personally identifiable or sensitive fields

## Metric template

Every metric intended for SQL generation should contain:

```markdown
# Definition

# Business Interpretation

# Formula

# Grain

# Source Tables

# Required Joins

# Filters and Exclusions

# Time Semantics

# Dimensions

# Null and Duplicate Handling

# SQL Expression

# Validation Examples

# Open Questions
```

Record:

- Additive, semi-additive, or non-additive behaviour
- Numerator and denominator
- Default date field
- Default timezone
- Currency and unit
- Distinct-count keys
- Required exclusions
- Aggregation rules
- Known reconciliation source

Do not create executable SQL for a metric whose grain or join path is unresolved.

## Dimension template

Document:

- Business meaning
- Source table and column
- Key and display label
- Hierarchies
- Allowed values
- Default grouping
- Null or unknown-member handling
- Validity dates
- Relationship to metrics and facts

## Structured-data conversion

### CSV

Identify dataset purpose, row grain, columns, observed types, missing-value patterns, representative values, and row count when known. Do not reproduce all rows unless it is a short reference list. Distinguish declared, observed, and inferred types.

### JSON

Preserve nested objects and arrays. Use:

| JSON path | Type | Required | Description | Example |
|---|---|---:|---|---|

Do not mark a field required from one sample.

### XML

Preserve namespaces, elements, attributes, hierarchy, and repeating structures. Use:

| XPath | Kind | Type | Cardinality | Description |
|---|---|---|---|---|

Only claim cardinality when supported by XSD, DTD, specification, or explicit rules.

## Semantic path for question answering and SQL

The wiki must enable this reasoning path:

```text
User wording
  -> glossary term or business concept
  -> metric and dimensions
  -> business rules
  -> source tables/views
  -> confirmed join contracts
  -> columns and filters
  -> database dialect
  -> read-only SQL
  -> result interpretation
```

Create links for every available step. A metric should never point only to a table name; it should identify columns, grain, filters, joins, and time semantics.

Create `wiki/queries/query-generation-contract.md` when building a SQL-capable wiki. Include:

- Supported SQL dialects
- Read-only policy
- Approved schemas/views
- Prohibited objects and columns
- Default row limit
- Timeout or cost expectations
- Tenant and authorization rules
- Date/time conventions
- Parameterization rules
- Ambiguity handling
- Validation requirements

## SQL safety requirements

When documenting or generating SQL:

1. Generate read-only SQL by default.
2. Never generate `INSERT`, `UPDATE`, `DELETE`, `MERGE`, `DROP`, `ALTER`, `TRUNCATE`, grants, or procedural execution unless explicitly authorized for a development task.
3. Prefer approved views or semantic-layer objects over raw tables.
4. Use only confirmed relationships for automatic joins.
5. Qualify tables with schema names.
6. Use explicit columns; avoid `SELECT *`.
7. Parameterize user-provided values.
8. Apply a row limit for exploratory detail queries when supported.
9. Respect tenant, entitlement, row-level security, and sensitive-column rules.
10. Detect fan-out and many-to-many joins before aggregation.
11. Aggregate at the metric's declared grain.
12. Do not guess a date column, currency conversion, status filter, or distinct-count key.
13. Return clarification requirements when ambiguity could materially change the result.
14. Explain assumptions separately from SQL.

## Indexes and discovery

When multiple files exist, maintain `wiki/index.md` and relevant category indexes.

Indexes must:

- Group concepts by type or business domain.
- Link to each concept using relative Markdown links.
- Provide a one-line factual description.
- Avoid duplicate links.
- Mark deprecated, planned, or unresolved concepts.

Also maintain `wiki/relationships/index.md` when the relationship graph is substantial. Summarize high-value paths such as metric-to-table, table-to-table, API-to-dataset, and process-to-system.

## Provenance and confidence

Use evidence levels consistently:

- `confirmed`: directly supported by schema, constraint, specification, code, or authoritative documentation
- `observed`: seen in supplied sample data or examples
- `inferred`: logical interpretation with supporting evidence but not formally declared
- `possible`: weak hypothesis requiring confirmation
- `conflicting`: sources disagree
- `unknown`: not available

Never upgrade observed or inferred information to confirmed without evidence.

## Security and privacy

Redact passwords, API keys, tokens, cookies, private keys, connection credentials, and unnecessary personal information. Preserve structural meaning with placeholders such as `<REDACTED_TOKEN>`.

Classify sensitive columns when evidence exists. Do not expose sample values from sensitive fields merely to improve documentation.

## Updating an existing wiki

1. Search by canonical ID, aliases, technical name, and title.
2. Update existing concepts rather than duplicating them.
3. Preserve stable paths and unknown frontmatter fields.
4. Do not silently overwrite conflicting facts.
5. Add a `Review Required` section for unresolved conflicts.
6. Repair links only when the intended target is clear.
7. Update affected indexes and reverse relationships.
8. Record meaningful changes in `wiki/log.md` when present or requested.
9. Report broken links, orphan concepts, duplicate concepts, and unresolved references.

## Validation

Before completing a task, check:

- Every concept file has valid frontmatter and a non-empty `type`.
- Paths use lowercase kebab-case.
- Canonical IDs are unique and stable.
- Links resolve to existing or clearly planned files.
- Relationships use the controlled vocabulary.
- Bidirectional relationships are consistent.
- Join keys, cardinality, and status are documented.
- Metrics include grain, source, formula, filters, and time semantics.
- Technical identifiers match the source exactly.
- No unsupported facts were invented.
- Secrets and sensitive samples are redacted.
- SQL examples are read-only and dialect-correct.
- Many-to-many and fan-out risks are identified.
- The wiki supports discovery from business wording to SQL-capable database concepts.

Use `references/validation-checklist.md` for a full audit.

## Completion report

After making changes, report:

- Files created
- Files updated
- Concepts and canonical IDs
- Relationships created or changed
- Confirmed versus inferred relationships
- SQL-capable paths added
- Redactions
- Validation performed
- Broken links or unresolved questions
- Content not converted due to missing evidence

Do not claim a file was created or validated unless the repository was actually changed or checked.
