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
3. **Write** — Produce a TTL file. Write to the project directory.
4. **Validate** — Apply the [validate-knowledge-graph](../validate-knowledge-graph/SKILL.md) skill on the created file.
