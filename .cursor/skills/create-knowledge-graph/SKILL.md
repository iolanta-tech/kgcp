---
name: create-knowledge-graph
description: Creates Turtle (.ttl) knowledge graph files from free-form text. Accepts user text and a file name, writes a TTL file expressing the meaning using the project ontology, then runs validate-knowledge-graph to validate and fix issues. Use when the user wants to create a knowledge graph from text, convert prose to RDF, or add TTL data from a description.
---

# Create Knowledge Graph from Text

## Arguments

1. **Free-form text** — The user's description of entities, relationships, events, or facts to capture.
2. **File name** — Output Turtle file (e.g. `order-kvm.ttl`). Must end in `.ttl`.

Determine both from the user message. If either is missing, ask.

## Workflow

1. **Ontology** — Use the project ontology if it fits; otherwise define classes and properties inline or use standard vocabularies.
2. **Extract** — Map the text to triples (subject–predicate–object).
3. **Resolve entities** — For each entity (person, place, work, series, organization, concept), use the [find-url-for](../find-url-for/SKILL.md) skill to find an existing linked-data URL (e.g. Wikidata, DBpedia, VIAF). Prefer reusing those URIs instead of minting new ones. Only define local individuals for entities that have no suitable external URI.
4. **Write** — Produce a TTL file. Write to the project directory.
   - **Existing URLs first** — Use the URIs from step 3 where found (with the appropriate prefix, e.g. `wd:Q12345`).
   - **No labels/names for external URIs** — Do not add `rdfs:label` or `schema:name` for entities that use an external linked-data URI (Wikidata, DBpedia, VIAF, etc.); the label/name can be resolved from the source. Add `rdfs:label` (and `schema:name` if used) only for local individuals or for classes/properties from vocabularies that lack labels in the loaded context.
   - **Slash URLs for local only** — For entities without an external URI, define a `@prefix` with a base URI ending in `/` (e.g. `https://www.knowledgegraph.tech/kgcmart/<topic>/`) and use prefixed names (e.g. `topic:IndividualName`). Do not use `#LocalName` for subjects or objects.
5. **Validate** — Apply the [validate-knowledge-graph](../validate-knowledge-graph/SKILL.md) skill on the created file.
