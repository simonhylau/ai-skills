# OKF LLM Wiki Builder — GitHub Copilot Skill

Install this project skill by copying the folder:

```text
.github/skills/okf-llm-wiki-builder/
```

to the root of your repository.

The skill helps Copilot create and maintain a connected Markdown LLM Wiki, including database relationships, semantic metrics, join contracts, and safety rules for agents that generate read-only SQL.

Example prompts:

- Convert `schema.sql` into a connected OKF wiki and create confirmed join contracts.
- Audit the wiki for broken links, orphan concepts, and unsafe inferred joins.
- Build the semantic path for “monthly active customers” from business definition to SQL columns.
- Update the wiki from these CSV data dictionaries without duplicating existing tables.
- Create `query-generation-contract.md` for PostgreSQL and mark all prohibited schemas.
