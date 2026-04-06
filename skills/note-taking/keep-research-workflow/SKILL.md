---
name: keep-research-workflow
description: "Use Keep as a research operating system: collect, promote, connect, and distill a small corpus of high-value sources without getting lost in the weeds."
version: 1.0.2
author: Hermes Agent
license: MIT
---

# Keep research workflow

Keep is a research tool, not a bookmark dump.

Quickstart:
0. put the source in Keep first, so it exists as a real note before you start organizing it
1. collect the smallest corpus that plausibly answers the question
2. promote the sources worth revisiting with titles and a small tag set
3. connect them with the right edge type (`references`, `cites`, `informs`)
4. distill the answer into a short synthesis note

Minimal example:
```bash
keep put /path/to/source.pdf -t type=paper -t topic=your-topic
keep tag https://example.com/source --tag title='Readable Title'
```

What `put` does, briefly:
- it creates the source note right away
- analysis, OCR, embeddings, and part creation happen asynchronously in the background
- the source may look sparse at first and then fill in after the queue runs
- for URLs, `put` may create or refresh a stub that later becomes a richer note after processing

If you do those well, Keep becomes a small, navigable research graph instead of a citation swamp.

## The mechanics in one sentence

Keep stores documents, tags, and graph edges in the same note system: a source note is the primary object, tags shape search and analysis, and edge tags create navigable relationships to other notes.

## When to use this

Use this workflow when you are:
- researching a topic from papers, blog posts, docs, or references
- building a source graph around a question
- deciding which sources matter enough to keep revisiting
- creating a concise research artifact others can follow

## 1) Collect

Collect the sources that plausibly matter.

Good collection targets:
- the anchor paper or survey
- a few high-value references
- key author pages or venue pages
- stubs created by reference edges that look important

Rules:
- do not collect everything
- start with the smallest corpus that can answer the question
- if the source is a PDF, tag it as `type=paper`

### Why `type=paper` matters

`type=paper` is not just a label. It changes how Keep analyzes the document.

In the updated workflow, a PDF tagged `type=paper` uses a paper-structured analysis prompt instead of the generic default analysis path. That means:
- better section-aware decomposition
- more paper-shaped parts
- cleaner retrieval anchors
- less generic blob-summary behavior

This is the right default for papers and surveys.

## 2) Promote

Promote only the sources that are worth revisiting.

Promotion means:
- add a readable `title`
- add a topic tag
- add a small set of relationship tags
- optionally add an author note if the author identity matters

Use this when the source is likely to be reused in the final answer or future follow-up.

Suggested tags:
- `type=paper`
- `topic=...`
- `title=...`
- `references=[...]`
- `cites=[...]`
- `informs=[...]`

Important:
- use wikilink-style edge values like `https://url[[Readable Title]]`
- Keep can also parse markdown-style links into a bare URL plus title now
- do not create numbered pseudo-keys like `informs2` or `referenced_by3`

## 3) Connect

Keep the graph useful by preserving provenance and follow-up paths.

### The data model for references and citations

References are not special text blobs. They are edges.

An edge tag is an ordinary tag key whose tagdoc declares an inverse relationship via `.tag/KEY`.

For example:
- `references` is an edge tag for any parseable link or extracted outbound link
- its inverse is `referenced_by`
- `cites` is a formal bibliographic citation edge
- its inverse is `cited_by`
- `informs` is an edge tag for provenance / research influence
- its inverse is `informed_by`
- `author` can also be an edge tag when the tagdoc defines it that way

That means a tag like:

```bash
keep flow tag -p id='https://example.com/paper' -p 'tags={"references":["https://example.com/related[[Related Paper]]"],"cites":["https://doi.org/10.1145/3460231.3474243[[TKG Reasoning]]"]}'
```

creates graph relationships, not just metadata.

Why this matters:
- the note stays searchable by its own content and tags
- the target note gets the inverse listing automatically
- the graph becomes navigable in both directions
- `cites` captures deliberate bibliographic structure, while `references` can remain mechanical / extracted

### Edge-tag rules

Use edge tags to capture relationships:
- `references`: what this note cites or links to mechanically
- `cites`: what this note formally cites in bibliographic structure
- `referenced_by`: what points at this note
- `cited_by`: what formally cites this note
- `informs`: what this note meaningfully helps explain or unlock

Best practice:
- keep the number of promoted nodes small
- preserve links to important adjacent sources
- prefer stable identities when you have them
- for authors, use a stable author ID when available; otherwise use the name as a readable label
- use `cites` when a structured enricher or citation parser knows the bibliography; use `references` for plain links discovered in text

## 4) Distill

Turn the corpus into a compact research answer.

The final output should usually be one of:
- a short synthesis note
- a curated bibliography
- a research memo with a clear conclusion
- a small graph of the sources that actually mattered

The distillation step should answer:
- what did we learn?
- what sources matter most?
- what should be followed up later?
- what should be ignored for now?

## Working with papers

For PDFs, the preferred path is:
1. put / ingest the source first
2. tag as `type=paper`
3. fetch / rehydrate the source if needed
4. process the background queue
5. inspect the parent note first
6. treat parts as search anchors, not as the primary synthesis surface

If the source is scanned or image-based, OCR happens during ingestion before the paper analysis step, so the note you promote is already the text-backed result.

### What parts are

Parts are the note fragments Keep creates when it analyzes a document.
Think of them as lightweight sub-notes:
- each part captures a slice of the parent document
- parts are individually searchable
- parts help you recover a useful section or snippet later
- parts are not the main place to do synthesis

### Why parts are useful

Parts are useful because they give you retrieval hooks without forcing the whole paper into one blob.
With the updated paper flow:
- parts stay simpler
- they search together with the parent note’s context
- they do not inherit the full parent tag pile directly
- the parent note remains the main synthesis surface

That makes parts good for:
- finding a specific definition again
- recovering a table explanation or a key paragraph
- anchoring follow-up searches

### How to use parts

Default behavior:
- leave parts alone
- use the parent note for synthesis
- promote a part only if it is genuinely reusable

Promote a part only when it contains a reusable nugget:
- a definition
- a key claim
- a table explanation
- a future-directions paragraph
- a quote worth revisiting

## Rehydrating auto-vivified URL notes

When a URL note exists only as a stub created from a reference edge, rehydrate it by re-putting the same URL as a URI-backed note.

### Flow tool example

```bash
keep flow get -p item_id='https://example.com/page'
keep flow tag -p id='https://example.com/page' -p 'tags={"title":"Readable Title"}'
keep flow put -p id='https://example.com/page' -p uri='https://example.com/page'
keep pending
keep flow get -p item_id='https://example.com/page'
```

### CLI example

```bash
keep get https://example.com/page
keep tag https://example.com/page --tag title='Readable Title'
keep put https://example.com/page
keep pending
keep get https://example.com/page
```

That is the standard rehydration sequence.

## Research assignment pattern

A good Keep workflow for a question like “Are there applications of compressive-sensing techniques to graph/embedding search and re-rank tasks?” looks like this:

1. collect an anchor survey or baseline paper
2. promote the sources that are truly relevant
3. connect them with clean provenance edges
4. distill the actual answer into a short synthesis note

The answer should come from the curated corpus, not from a single search result.

## A concrete example flow

Suppose you start from a survey PDF and want to work outward.

1. Collect the survey

```bash
keep put /path/to/survey.pdf -t type=paper -t topic=temporal-knowledge-graph
```

2. Promote it

```bash
keep tag https://example.com/survey --tag title='Useful Survey Title'
```

3. Connect to an important cited paper

```bash
keep tag https://example.com/survey --tag 'cites=https://doi.org/10.1145/3460231.3474243[[TKG Reasoning]]'
```

4. Rehydrate any stub reference note you actually care about

```bash
keep flow put -p id='https://example.com/cited' -p uri='https://example.com/cited'
keep pending
```

5. Distill the result into a short note

```bash
keep put "Summary: this line of work uses X, not Y; the important papers are A, B, and C." -t type=learning -t topic=your-topic
```

## Good defaults

- prefer a small corpus over exhaustive collection
- prefer readable titles over raw URLs for important notes
- prefer stable identities for people and papers when possible
- prefer parent-note synthesis over part-note synthesis
- prefer a few strong edges over a large noisy graph

## Pitfalls

- do not treat every citation as equally important
- do not over-tag parts or create lots of one-off keys
- do not keep drilling down once the research question is answered
- do not rely on parts as if structural chunking were perfect
- do not forget that `type=paper` changes analysis behavior

## Verification checklist

- the corpus is small and intentional
- the important notes have titles
- papers are tagged `type=paper`
- edge values use `https://url[[Title]]` or parsed equivalent
- the graph preserves provenance without noise
- the parent note carries synthesis; parts stay lightweight
- the final output is a compact synthesis, not a citation dump
