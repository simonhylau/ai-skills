# Relationship Model

Use lowercase kebab-case relationship names.

## Structural

| Relationship | Reverse | Meaning |
|---|---|---|
| contains | belongs-to | Physical or logical containment |
| has-schema | schema-of | Database-to-schema relationship |
| has-table | table-of | Schema-to-table relationship |
| has-column | column-of | Table/view-to-column relationship |
| exposes | exposed-by | System/API exposes an object or capability |

## Data and SQL

| Relationship | Reverse | Meaning |
|---|---|---|
| references | referenced-by | Foreign-key or semantic reference |
| joins-to | joins-to | Valid query join path |
| derived-from | source-for | Data or expression derives from another concept |
| reads-from | read-by | Consumer reads data from source |
| writes-to | written-by | Producer writes data to target |
| materializes | materialized-as | View/table materializes another concept |
| aggregates | aggregated-by | Summary relationship |
| filters-by | filter-for | Required filtering dimension or rule |

## Semantic

| Relationship | Reverse | Meaning |
|---|---|---|
| represents | represented-by | Technical object represents a business concept |
| measures | measured-by | Metric measures a concept |
| dimensioned-by | dimension-of | Metric can be grouped by dimension |
| calculated-from | calculation-source-for | Metric formula dependency |
| synonym-of | synonym-of | Equivalent terminology |
| broader-than | narrower-than | Taxonomy relationship |
| governed-by | governs | Policy or rule governance |

## Operational

| Relationship | Reverse | Meaning |
|---|---|---|
| depends-on | dependency-of | Runtime or process dependency |
| produced-by | produces | Output provenance |
| consumed-by | consumes | Consumer relationship |
| owned-by | owns | Accountable owner when explicitly known |
| maintained-by | maintains | Operational maintenance responsibility |
| supersedes | superseded-by | Replacement relationship |
| triggers | triggered-by | Event or process trigger |

## Required relationship attributes

- `type`
- `target`
- `status`: confirmed, observed, inferred, possible, conflicting, deprecated
- `evidence`
- `notes`, when useful

For SQL joins also capture:

- left object and key
- right object and key
- cardinality
- default join type
- null behaviour
- fan-out risk
- tenant/security predicates
- validity-date predicates
- SQL dialect, when relevant
