---
name: search-salesforce
description: Find and summarize Salesforce records with the right search/query strategy
---

# Search Salesforce

Use this skill when the user wants help finding Salesforce data, account context, contacts, opportunities, notes, or related records.

## Workflow

1. Start by identifying the record type and the least ambiguous search term available.
2. Use the broadest effective search first when the target is unclear.
3. Prefer exact, narrow queries once the object and key fields are known.
4. Return concise results with record names, IDs, and any directly relevant fields.
5. If the request depends on a missing identifier, ask for the smallest missing detail instead of guessing.

## Query strategy

- Use text search for discovery when the object or field is unknown.
- Use structured queries for exact filters, counts, and relationship lookups.
- Keep field lists small and relevant.
- Prefer one clear result path over speculative multi-object searches.

## Response style

- Summarize what was found first.
- Include record IDs when they are useful for follow-up work.
- Call out uncertainty clearly.
- Do not invent Salesforce data.

## Good outputs

- "Found the account and two related opportunities."
- "No exact match, but these three records are the closest."
- "I need the company name or Salesforce ID to narrow this down."
