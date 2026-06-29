# LLM Zoomcamp 2026 — Homework 4: Evaluation

My solution for Module 4. I reused the search pipeline from homework 2 (same
chunks, same `text_search` / `vector_search` / `hybrid_search`) and added the
vector and hybrid evaluation, so I could compare keyword, vector and hybrid
search on Hit Rate and MRR instead of guessing. Everything pins commit `8c1834d`
(72 lesson pages, 295 chunks, 360 ground-truth questions).

The full pipeline is in `homework4_evaluation.ipynb`.

## Setup

- `uv add openai pydantic python-dotenv pandas` plus `scikit-learn`, `minsearch`,
  `gitsource`, `sentence-transformers`, `tiktoken`.
- OpenAI key in a `.env` file, model `gpt-5.4-mini` for the question generation.
- Embeddings: `SentenceTransformer("all-MiniLM-L6-v2")`, same as the module.
- Text index: `minsearch.Index`; vector index: `minsearch.VectorSearch`; both
  keyed on `filename`.

## Answers

**Q1 — Average input tokens over the first 3 pages**
> 1400

I generated 5 questions for `01-intro.md`, `02-environment.md` and `03-rag.md`,
and averaged `response.usage.input_tokens` across the 3 calls. Each call sends a
full lesson page plus the instructions, so the input lands around 1k-1.7k tokens
per call and the average sits near ~1300-1400.

**Q2 — First result with text search**
> 01-agentic-rag/lessons/03-rag.md

For the first ground-truth question, keyword search puts `03-rag.md` on top.

**Q3 — First result with vector search**
> 01-agentic-rag/lessons/01-intro.md

The question was written from `01-intro.md`. Vector search returns it at the top
while text search doesn't - the exact point of the exercise: the two methods
disagree on the same query, which is why we measure across the whole set.

**Q4 — Text search Hit Rate**
> 0.76

`evaluate(text_search, ground_truth)` gives a Hit Rate of 0.7583.

**Q5 — Vector search MRR**
> 0.55

`evaluate(vector_search, ground_truth)["mrr"]`. The ground-truth questions are
deliberately reworded (not copied from the pages), which suits semantic search,
so vector MRR comes out in the same ballpark as text search.

**Q6 — Best k for hybrid search**
> 1

I evaluated `hybrid_search` for `k` in 1, 50, 100, 200. A smaller `k` sharpens
the weight of the top ranks, so `k = 1` gives the best MRR (smallest `k` on a
tie).

## Metrics, briefly

- `compute_relevance` runs a search and marks each result 1/0 by `filename`.
- `hit_rate` = share of questions where the right page is anywhere in the top results.
- `mrr` = reciprocal rank of the first correct page, averaged over all questions.
- `evaluate` runs any search function over the ground truth and returns both.

## What I took away

The reworded questions are what make this interesting: keyword search wins on
exact terms, vector search wins on meaning, and on the same query they often
disagree. Once there's a fixed ground truth, picking between methods (or tuning
`k` in RRF) stops being a matter of taste and becomes a number you can move and
re-measure.
