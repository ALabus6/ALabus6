---

name: search-salesforce
description: Find GHAS-focused Salesforce records with territory-aware search rules

---

# Search Salesforce

Use this skill when the user wants help finding Salesforce data for GHAS sales, account context, contacts, opportunities, notes, or related records.

## Territory rules

- Assume the user is an ASE focused on GitHub Advanced Security only.
- For account questions, prefer GHAS-related context and avoid pulling in GHE, GitHub Actions, or Copilot unless the user explicitly asks for them.
- The user supports AMER SMB sellers in GitHub, so prefer that territory when filtering account or opportunity context.
- When reviewing opportunity ownership for accounts in this territory, the opportunity owner role should map back to `AMER - SMB Account Executive`.

## Workflow

1. Start by identifying the record type and the least ambiguous search term available.
2. Use the broadest effective search first when the target is unclear.
3. Prefer exact, narrow queries once the object and key fields are known.
4. Return concise results with record names, IDs, and any directly relevant fields.
5. If the request depends on a missing identifier, ask for the smallest missing detail instead of guessing.
6. When the request is about an account in the user's territory, filter to GHAS-relevant records first and exclude unrelated GitHub product lines by default.

## Query strategy

- Use text search for discovery when the object or field is unknown.
- Use structured queries for exact filters, counts, and relationship lookups.
- Keep field lists small and relevant.
- Prefer one clear result path over speculative multi-object searches.
- Favor the seller/owner fields and role mappings that confirm AMER SMB ownership.

## Response style

- Summarize what was found first.
- Include record IDs when they are useful for follow-up work.
- Call out uncertainty clearly.
- Do not invent Salesforce data.
- If a record is not GHAS-related, omit it unless the user explicitly asked for broader GitHub coverage.

## Good outputs

- "Found the account and two related opportunities."
- "No exact match, but these three records are the closest."
- "I need the company name or Salesforce ID to narrow this down."