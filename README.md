# RAG-Based Job Listing Matcher

A Retrieval-Augmented Generation (RAG) application that matches a candidate's resume against job listings and returns the **top 10 most relevant jobs**, each with a **relevance score** and a short **explanation of why it matches**.

> **Status: in progress.** The data pipeline is done and the project is at the embedding stage. See the [Roadmap](#roadmap) for what is finished and what is next.

---

## Problem

Job boards return thousands of results, and keyword search misses roles that are a good fit but use different wording. This project uses semantic search to find relevant jobs from the *meaning* of a resume, then uses an LLM to explain each match.

## How it works (planned architecture)

```
Resume ──► Embed ──► Vector search over job postings ──► Top-K candidates
                                                            │
                                                            ▼
                                       LLM re-ranks + writes per-match rationale
                                                            │
                                                            ▼
                                     Top 10 jobs with score + explanation
```

1. **Ingest:** clean and deduplicate raw job postings, then save them as Parquet.
2. **Chunk and embed:** split job descriptions into chunks and create embeddings.
3. **Store:** keep embeddings in a vector database for similarity search.
4. **Retrieve:** embed the candidate's resume and fetch the most similar job chunks.
5. **Generate:** an LLM ranks the results and explains each match in plain language.

## Dataset

- Source file: `postings.csv` (123,849 job postings)
- Fields used include `job_id`, `title`, `company_name`, `location`, `description`, `listed_time`, `original_listed_time`, `expiry` and `normalized_salary`.
- The raw data file is **not included** in this repo because of its size. Add your copy as `postings.csv` in the project folder.

## Data cleaning

Cleaning is in the first stage of the pipeline.

- **Deduplication:** postings with the same description, title, company and location are treated as duplicates. The newest `listed_time` is kept.
- **Result:** 123,849 rows reduced to **119,176 unique postings**.
- **`first_seen` column:** keeps the earliest `original_listed_time` for each deduplicated posting.
- **Output:** `cleaned_postings.parquet`, which loads much faster than CSV.

### Working within limited memory

The project runs in WSL with about 3.8 GB of RAM. A first attempt to clean the whole file in memory was killed for running out of memory. The fix was a **chunked, two-pass approach** that processes the file in pieces instead of loading it all at once.

## Tech stack

| Area | Tools |
|---|---|
| Language | Python |
| Data | Pandas, Parquet |
| Embeddings and search | Embedding model + vector database (Qdrant or Chroma) |
| LLM | LLM API for re-ranking and explanations |
| Serving (planned) | FastAPI, Docker |
| Environment | Linux (WSL) |

## Roadmap

- [x] Load and inspect the job postings dataset
- [x] Deduplicate postings (123,849 to 119,176 rows)
- [x] Memory-efficient chunked cleaning
- [x] Save cleaned data to Parquet
- [ ] Chunk job descriptions
- [ ] Generate embeddings *(in progress)*
- [ ] Load embeddings into a vector database
- [ ] Retrieve top candidates for a given resume
- [ ] LLM re-ranking and per-match rationale
- [ ] Evaluate against manually labeled test resumes (target: at least 8 of 10 relevant matches)
- [ ] Wrap in a FastAPI endpoint and containerize with Docker
- [ ] Deploy and add a demo

## Evaluation plan

I will build a small set of test resumes and label which jobs are relevant for each one by hand. For every resume I'll check how many of the top 10 results are relevant. The goal is **8 or more relevant matches out of 10**.

## Getting started

```bash
# 1. Clone the repo
git clone https://github.com/YashviRexwal/<Job-Listing-matcher>.git
cd <repo-name>

# 2. Create an environment and install dependencies
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 3. Add the dataset
# Place postings.csv in the project folder
```

> Run instructions will be updated as each stage is finished.

## Author

**Yashvi Rexwal** — [GitHub](https://github.com/YashviRexwal) | [LinkedIn](https://www.linkedin.com/in/yashvi-rexwal-7823b4405/)
