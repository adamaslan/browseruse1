

# Custom Agent Graph for Optimized arXiv Research

## 1) Why a Graph-Based Agent?

* **Linear pipeline drawback**
  Current design is linear: fetch papers → send to agent → save outputs. Every step runs fully even if some conditions don’t require it.

* **Graph advantage**
  A graph allows **parallelization**, **conditional branching**, and **early exits**, which reduce latency.

  * Only trigger deep analysis when a paper meets criteria.
  * Skip browsing or external lookups when abstracts are already rich.
  * Parallelize summary generation and structured JSON extraction.

---

## 2) Node Types in the Graph

* **Fetch Node**

  * Uses `feedparser` to query arXiv.
  * Outputs raw list of paper entries.
* **Filter Node**

  * Removes visited or irrelevant papers.
  * Adds metadata: query term, abstract length, year relevance.
* **Relevance Scorer Node**

  * Uses `llm` (via LangGraph) to assign a score (0–10).
  * Condition: only papers above threshold (e.g., 7) go to **Deep Analysis**.
* **Summarizer Node**

  * Creates concise markdown for all new papers.
  * Runs in parallel with Relevance Scorer to save time.
* **Deep Analysis Node**

  * Activated conditionally (if relevance > 7 or abstract too short).
  * Uses `Agent` with browsing enabled.
  * Fetches full paper content or references.
* **Expansion Node**

  * Optional: generates related queries (e.g., “momentum + reinforcement learning”).
  * Sends new queries back into the **Fetch Node**.
* **Persistence Nodes**

  * Markdown Writer: append summaries.
  * JSON Writer: append structured metadata.
  * Visited Tracker: update visited IDs.

---

## 3) Conditional Edges

* **Example 1: Abstract check**

  ```text
  Fetch → Filter → (if abstract < 300 words → Deep Analysis, else → Summarizer)
  ```

* **Example 2: Relevance threshold**

  ```text
  Relevance Scorer → (score ≥ 7 → Deep Analysis, score < 7 → Summarizer only)
  ```

* **Example 3: Discovery loop**

  ```text
  Deep Analysis → Expansion → Fetch (new related queries)
  ```

These **conditional edges** are key: they prevent expensive browsing for irrelevant or already-sufficient papers, optimizing **latency and cost**.

---

## 4) LangGraph Integration

LangGraph lets you declaratively build this graph with nodes and edges:

* **Nodes as functions**
  Each node is just a function: `fetch_papers`, `filter_papers`, `score_relevance`, `summarize`, `deep_analyze`, `expand_queries`.

* **Edges as conditions**
  You define conditional transitions with simple Python logic:

  ```python
  graph.add_edge("RelevanceScorer", "DeepAnalysis", condition=lambda x: x["score"] >= 7)
  graph.add_edge("RelevanceScorer", "Summarizer", condition=lambda x: x["score"] < 7)
  ```

* **Parallelization**
  LangGraph can run nodes like `Summarizer` and `Relevance Scorer` in parallel since they don’t depend on each other.

* **Looping edges**
  For discovery:

  ```python
  graph.add_edge("DeepAnalysis", "Expansion")
  graph.add_edge("Expansion", "Fetch")
  ```

  This enables iterative “digging deeper” into the literature.

---

## 5) Momentum Investing Research Enhancement

For the specific use case of momentum investing:

* **Deeper article analysis**

  * Abstracts often omit quantitative details (datasets, backtest methods).
  * Deep Analysis Node browses full PDFs and extracts numerical findings.
* **Article expansion**

  * Expansion Node can suggest queries like:

    * “cross-sectional momentum ETF performance”
    * “intraday reversal and momentum interaction”
  * These queries yield fresh arXiv results beyond the static list.
* **Best research output**

  * Markdown file now contains:

    * High-level summaries for all papers.
    * Deep-dive sections for relevant ones.
    * Iterative expansions with related work.
  * JSON file includes relevance scores, structured insights, and new related tags.

---

## 6) Example Graph Flow

```mermaid
graph TD
  A[Fetch from arXiv] --> B[Filter Papers]
  B --> C[Relevance Scorer]
  C -->|score >= 7| D[Deep Analysis (Browsing)]
  C -->|score < 7| E[Summarizer]
  D --> F[Expansion Node]
  F --> A
  E --> G[Markdown Writer]
  D --> G
  E --> H[JSON Writer]
  D --> H
  G --> I[Visited Tracker]
  H --> I
```

---

✅ With this design:

* Low-latency for low-value papers (quick summary only).
* High-value papers get full browsing + deep summarization.
* System self-expands into new areas automatically.

