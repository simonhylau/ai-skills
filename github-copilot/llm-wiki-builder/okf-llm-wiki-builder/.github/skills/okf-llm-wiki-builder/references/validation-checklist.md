# Validation Checklist

## Files and metadata

- [ ] Every concept has one primary purpose.
- [ ] YAML frontmatter parses correctly.
- [ ] `type` is present and non-empty.
- [ ] Path is lowercase kebab-case.
- [ ] Canonical ID is unique.
- [ ] Technical names match source casing exactly.

## Links and graph

- [ ] Internal links resolve.
- [ ] Reverse relationships exist where practical.
- [ ] Relationship types use the controlled vocabulary.
- [ ] Relationship status and evidence are present.
- [ ] No confirmed relationship relies only on similar naming.
- [ ] No orphan concept is omitted from indexes without reason.

## Database semantics

- [ ] Table grain is documented.
- [ ] Keys and constraints are documented or marked unknown.
- [ ] Join cardinality is documented.
- [ ] Fan-out and many-to-many risks are documented.
- [ ] Timezone, units, currency, and status semantics are documented when relevant.
- [ ] Soft-delete and active-record rules are documented.
- [ ] Tenant and security boundaries are documented.

## Metrics

- [ ] Formula is explicit.
- [ ] Grain is explicit.
- [ ] Source tables and columns are linked.
- [ ] Required joins are confirmed.
- [ ] Filters and exclusions are explicit.
- [ ] Default date field and timezone are explicit.
- [ ] Null, duplicate, and distinct-count behaviour is explicit.
- [ ] SQL expression is read-only and dialect-correct.

## Safety

- [ ] Secrets are redacted.
- [ ] Sensitive samples are omitted or anonymized.
- [ ] SQL uses explicit columns and qualified objects.
- [ ] User values are parameterized.
- [ ] Exploratory detail SQL has a row limit where supported.
- [ ] No write or DDL statement is presented as normal user-query SQL.

## Completeness

- [ ] Root and category indexes are updated.
- [ ] Business wording links to semantic and technical concepts.
- [ ] SQL-capable paths are complete from metric to columns and joins.
- [ ] Conflicts and unknowns are visible.
- [ ] Provenance is retained.
