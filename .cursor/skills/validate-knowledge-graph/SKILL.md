---
name: validate-rdf-iolanta
description: Validates RDF documents using Iolanta kglint. Requires a file path (explicit or inferred from context). Runs iolanta on Turtle/RDF files with --as kglint/json, interprets assertions and labels, and addresses issues (e.g. missing rdfs:label). Use when the user asks to validate RDF, validate a TTL file (e.g. "validate another-order.ttl"), check ontology or data, or run iolanta.
---

# Validate RDF with Iolanta (kglint)

## Argument: file path

The file path to validate is provided by the user. Determine it in this order:
1. **Explicit path** in the user's message (e.g. "validate another-order.ttl", "validate RDF in KGCPMart-OWL.ttl")
2. **@-mentioned file** (e.g. user attaches or references a file with @)
3. **Currently focused or open file** in the editor
4. **Ask** if none of the above applies

## How validation works

Iolanta loads the **whole directory** in which the requested file is located (not just that file). All TTL/ontology files in that directory are in scope. Validation uses the **kglint** facet, which outputs a report with **assertions** (lint findings) and **labels** (nodes and their triples).

## Command

```bash
iolanta <file> --as kglint/json
```

Example: `iolanta another-order.ttl --as kglint/json`

**Timeout:** Iolanta loads the directory and may pull remote content (e.g. Wikidata, schema.org). Use the **maximum timeout** the tooling supports when running this command (e.g. 600000 ms / 10 minutes) so validation can complete. Do not conclude the TTL is valid without successfully running Iolanta and checking the report.

## Kglint output schema

The output is a JSON object with two top-level keys:

- **`assertions`** — Array of lint findings. **Empty `[]` = validation pass.** Each assertion has:
  - `severity`: `"error"` or `"warning"`
  - `code`: assertion code (see below)
  - `target`: the node or triple the finding refers to (URI/blank node object, or full triple)
  - `message`: human-readable explanation

- **`labels`** — Array of label entries (one per URI/blank node): `node` (type, value, label) and `triples` (triples that node participates in). Used for context; not used to decide pass/fail.

**Assertion codes and how to fix:**

| Code | Meaning | Fix |
|------|---------|-----|
| `uri-label-identical` | URI has no usable label; rendered label equals the URI. | Add `rdfs:label "Human Readable Name"@en` in the **ontology** (e.g. `KGCPMart-OWL.ttl`) for that property or class, or ensure the URI is dereferenceable so Iolanta can load a label. |
| `blank-label-identical` | Blank node has no usable label. | Add or fix label for that blank node in the data/ontology. |
| `literal-looks-like-uri` | Literal value looks like a URL. | Consider using a URI reference instead of a literal. |
| `literal-looks-like-qname` | Literal value looks like a QName (e.g. `ex:foo`). | Consider using a URI reference instead of a literal. |

## Workflow

1. Determine the file to validate (see "Argument: file path" above).
2. Run `iolanta <file> --as kglint/json` using the file path from step 1, with **maximum timeout** (e.g. 600000 ms) so remote loading can complete.
3. Parse the JSON and check **`assertions`**. If `assertions` is not empty, there are issues to fix.
4. For each assertion, use `code`, `target`, and `message` to fix (e.g. add `rdfs:label` in the ontology for `uri-label-identical`; fix literals for literal codes).
5. Re-run `iolanta <file> --as kglint/json` and repeat until **`assertions`** is empty.

## Example invocation

**User:** "validate another-order.ttl"  
**Agent:** Determines file path → `another-order.ttl` → runs `iolanta another-order.ttl --as kglint/json` → checks `assertions`; if empty, validation passed; otherwise fixes and re-runs.

**User:** "run iolanta" (with `KGCPMart-OWL.ttl` open)  
**Agent:** Uses focused file → runs `iolanta KGCPMart-OWL.ttl --as kglint/json`.

## Notes

- A PyLD `SyntaxWarning` about `@` is from the dependency, not your TTL; safe to ignore.
- Use absolute or correct relative path for `<file>` so the intended directory (and thus ontology) is loaded.
