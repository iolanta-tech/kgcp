---
name: find-url-for
description: Finds a semantic web / linked data URL for a notion the user asks about (person, place, work, concept). Use when the user wants a canonical URI for an entity in RDF/Turtle, when creating knowledge graphs and needing to link to external data instead of defining local individuals, or when they ask for a Wikidata/DBpedia/VIAF URI for someone or something.
---

# Find URL For

## Argument

The **notion** to resolve: person, place, work, organization, or concept. Determine from the user message (e.g. "Isaac Asimov", "Foundation series", "Paris"). If unclear, ask.

## Workflow

1. **Search** — Use web search to find the entity and common linked-data sources. Query for the name plus "Wikidata", "DBpedia", or "linked data URI".
2. **Choose source** — Prefer these in order when a stable, dereferenceable URI is needed:
   - **Wikidata** — `https://www.wikidata.org/entity/Q<id>` for people, places, works, concepts. Widely used, has labels in many languages. Prefix: `wd: <https://www.wikidata.org/entity/>`.
   - **DBpedia** — `http://dbpedia.org/resource/<Name>` (spaces as underscores). Good for Wikipedia-derived entities.
   - **VIAF** — For authors: `https://viaf.org/viaf/<id>`.
   - **id.loc.gov** — For authority control (authors, subjects): `https://id.loc.gov/authorities/names/<id>`.
   - **schema.org** — Only for types (e.g. Book, Person); not for individual entities.
3. **Verify** — Confirm the URI matches the intended entity (e.g. fetch the Wikidata entity page or DBpedia resource and check the label/description).
4. **Return** — Give the user the canonical URI and, if relevant, the Turtle prefix and example usage (e.g. `wd:Q34981` for Isaac Asimov).

## Output

Provide:

- **URI** — The chosen linked data URL.
- **Source** — e.g. Wikidata, DBpedia, VIAF.
- **Usage in TTL** — Prefix declaration and example triple, e.g.:
  ```turtle
  @prefix wd: <https://www.wikidata.org/entity/> .
  ...
  schema:author wd:Q34981 .
  ```

If no suitable URI is found, say so and suggest defining the entity locally (e.g. in the project’s ontology or data file) or trying a different spelling/variant name.

## Examples

**User:** "Find a linked data URI for Isaac Asimov"  
**Agent:** Search → find Wikidata Q34981 (Isaac Asimov) → verify label "Isaac Asimov" → return `https://www.wikidata.org/entity/Q34981`, prefix `wd:`, example `schema:author wd:Q34981`.

**User:** "We need a URI for the Foundation series"  
**Agent:** Search for "Foundation series Asimov Wikidata" or DBpedia → if found, return that URI; otherwise suggest a local URI under the project’s base (e.g. `foundation:FoundationSeries`).

**User:** "Can we use an existing URI for the author instead of defining them in our ontology?"  
**Agent:** Treat as find-url-for for the author mentioned in context (e.g. from the current TTL file) → find Wikidata/VIAF URI → return it and show how to replace the local individual.
