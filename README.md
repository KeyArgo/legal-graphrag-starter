# Colorado Legal GraphRAG — contributor starter

Welcome! This is a **self-contained** starting point for adding a **knowledge-graph /
GraphRAG** retrieval layer to a Colorado legal research system. Everything you need to
prototype is in this folder — a real citation graph + runnable starter code. You do **not**
need any server, database, credentials, or network access to get going.

---

## The big picture (30 seconds)

The existing system is a legal RAG search engine over ~1.24M passages of Colorado + federal
statutes, regulations, case law, and court rules. Today retrieval is **hybrid**: dense vector
search (bge-m3) + keyword/BM25 (SQLite FTS5), fused and then **reranked** by a cross-encoder.
That's strong, but it treats every document as an island.

**Case law is a graph.** Opinions cite statutes and prior cases; statutes have structure; some
authorities are far more central than others. A GraphRAG layer lets retrieval *traverse* those
relationships — find the on-point case, then pull the authorities it relies on and the later
cases that cite it — which flat vector search can't do. **That layer doesn't exist yet. It's
yours to build.**

---

## What's in this folder

```
data/
  nodes.jsonl   119,770 legal authorities  {id, type, court, date, title, url}
  edges.jsonl   160,793 relationships       {src, dst, kind}
  stats.json    summary + the 25 most-cited authorities
starter.py      loads the graph (NetworkX) and demos the core traversals
requirements.txt
```

### Graph model
- **Nodes** = legal authorities. `type` ∈ {caselaw, statute, co_statute, us_statute,
  regulation, cfr, rule, form, guide, unknown}.
- **Edges** (directed):
  - `kind="cites"` — `src` opinion/section **cites** `dst` authority → **the citation graph** (110k edges)
  - `kind="part_of"` — `src` section is **part of** `dst` parent statute → structure (50k edges)

The data is derived from the corpus's citation metadata (`cross_references` / `parent_citation`).
It is **public legal material** — safe to work with freely.

## Quickstart
```bash
pip install -r requirements.txt   # networkx, numpy, scipy
python starter.py
```
You'll see the most-cited authorities (42 U.S.C. § 1983 tops it, as it should), a real traversal
("1,598 documents cite § 1983 → …"), a PageRank authority ranking, and a 1-hop GraphRAG expansion
demo. `starter.py` has the building blocks: `cites()`, `cited_by()`, and a
`graphrag_expand(seeds, hops)` stub to grow into a retriever.

---

## Your mission (prototype → wire → measure)

A three-step arc that matches "build a bit, then prove it":

1. **Prototype (offline, this folder).** Turn `graphrag_expand()` into a real **GraphRAG
   retriever**: given a few seed authorities (imagine they're the top hits from the existing
   hybrid search), traverse the citation graph to assemble a *better* candidate set — the relied-on
   authorities + the notable later cases — and rank them (in-degree, PageRank, or personalized
   PageRank seeded on the query hits). Load into NetworkX now; graduate to Neo4j if you like.

2. **Wire.** Expose it as a function with a clean signature —
   `graphrag_retrieve(seed_citations: list[str], k: int) -> list[Authority]` — so it can slot in
   behind the existing search as a "GraphRAG mode." (We'll hand you the thin integration point when
   you're ready; you don't need the full system to design the interface.)

3. **Measure.** The whole point is *does it help?* There's an existing eval methodology (retrieval
   recall@k, NDCG, and a citation-grounding check). Show GraphRAG vs. plain hybrid on a set of
   realistic questions: does traversal surface the *controlling* authority that flat search misses?
   A clear before/after number is the deliverable that makes this real.

### Good first experiments
- **Authority-aware reranking:** boost candidates by citation PageRank — does the controlling case
  rise? - **Missing-link recall:** for a question whose answer hinges on a case that *cites* the
  obvious statute, does 1-hop expansion find it when vector search doesn't? - **Neighborhoods:**
  render the k-hop subgraph around a query's hits (great for a demo + a paper figure). - **Temporal:**
  use `date` to weight *later* citing cases (is the doctrine still good law?).

---

## Scope & boundaries (please read)
- This is a **data-only** starter: work entirely against the files here. No production access,
  hostnames, or credentials are needed — or provided — to do this work.
- The graph is public legal data; share/republish the *derived graph* freely, but coordinate before
  publishing anything framed as the full system.
- Before writing code we should agree on **contribution terms** (how your work is credited/licensed,
  and whether this heads toward a write-up you'd co-author). Ask your point of contact — it keeps
  things clean and is genuinely worth sorting first.

## Questions worth asking your contact
- Where should PRs land (which repo)? - What does the existing hybrid-search hit list look like as
  input (so your interface matches)? - Access to the eval question set for step 3?

Have fun — this is the part of the system with the most upside and the least prior art.
