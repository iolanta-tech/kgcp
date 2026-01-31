---
name: validate-rdf-iolanta
description: Validates RDF documents using Iolanta. Requires a file path (explicit or inferred from context). Runs iolanta on Turtle/RDF files, interprets labeled-triple-set output and feedback, and addresses issues (e.g. missing rdfs:label). Use when the user asks to validate RDF, validate a TTL file (e.g. "validate another-order.ttl"), check ontology or data, or run iolanta.
---

# Validate RDF with Iolanta

## Argument: file path

The file path to validate is provided by the user. Determine it in this order:
1. **Explicit path** in the user's message (e.g. "validate another-order.ttl", "validate RDF in KGCPMart-OWL.ttl")
2. **@-mentioned file** (e.g. user attaches or references a file with @)
3. **Currently focused or open file** in the editor
4. **Ask** if none of the above applies

## How validation works

Iolanta loads the **whole directory** in which the requested file is located (not just that file). All TTL/ontology files in that directory are in scope.

## Command

```bash
iolanta <file> --as labeled-triple-set
```

Example: `iolanta another-order.ttl --as labeled-triple-set`

## Interpreting output

- Output is JSON: an array of triples. Each triple has `subject`, `predicate`, `object_`, each with `feedback`, `type`, `uri`/`value`, and `label`.
- **Empty `feedback`** (`"feedback": []`) → no issue.
- **Non-empty `feedback`** → something to fix. Typical message: *"For this URI, the label is the same as the URI. We were unable to render that URI."*
  - Usually means the URI (often a predicate or class from the ontology) has no `rdfs:label` in the loaded data. Add `rdfs:label "Human Readable Name"@en` in the **ontology** (e.g. `KGCPMart-OWL.ttl`) for that property or class.

## Workflow

1. Determine the file to validate (see "Argument: file path" above).
2. Run `iolanta <file> --as labeled-triple-set` using the file path from step 1.
3. Scan output for any non-empty `feedback` (e.g. `grep -A1 '"feedback"'` and look for content between the brackets).
4. For each reported URI, add or fix `rdfs:label` in the appropriate file (ontology for classes/properties, data file only if it's local to that file).
5. Re-run iolanta and confirm all `feedback` arrays are empty.

## Example invocation

**User:** "validate another-order.ttl"  
**Agent:** Determines file path → `another-order.ttl` → runs `iolanta another-order.ttl --as labeled-triple-set` → interprets output.

**User:** "run iolanta" (with `KGCPMart-OWL.ttl` open)  
**Agent:** Uses focused file → runs `iolanta KGCPMart-OWL.ttl --as labeled-triple-set`.

## Notes

- A PyLD `SyntaxWarning` about `@` is from the dependency, not your TTL; safe to ignore.
- Use absolute or correct relative path for `<file>` so the intended directory (and thus ontology) is loaded.
