---
created: 2025-06-12T06:25
updated: 2025-06-12T06:29
---
Mem01 will be a **minimal, high-performance fork of Mem0** whose only job is to expose the four standard Model Context Protocol (MCP) tools while storing every memory object—text or PDF page—in ultra-compressed **Memvid MP4 files** instead of a vector database. By limiting scope to text + PDF ingestion (no vision model) we can deliver a production-ready server that runs comfortably on a 2-vCPU / 8 GB VPS in roughly six weeks of work or less.

---

## 1 Executive Summary

- **Mem0 today**: OpenMemory MCP ships with a Qdrant-backed vector store and an abstraction layer (`AbstractStore`) so alternative back-ends can be swapped in. ([github.com](https://github.com/mem0ai/mem0?utm_source=chatgpt.com "mem0ai/mem0: Memory for AI Agents; Announcing OpenMemory MCP"), [mem0.ai](https://mem0.ai/blog/how-to-make-your-clients-more-context-aware-with-openmemory-mcp/?utm_source=chatgpt.com "How to make your clients more context-aware with OpenMemory MCP"))
    
- **Memvid**: stores millions of chunks inside MP4 files and performs semantic k-NN search directly on the video, yielding ~8-10 × smaller footprints and sub-second latency. ([github.com](https://github.com/Olow304/memvid?utm_source=chatgpt.com "Olow304/memvid: Video-based AI memory library. Store ... - GitHub"), [apidog.com](https://apidog.com/blog/memvid/?utm_source=chatgpt.com "MemVid: Replacing Vector Databases with MP4 Files - Apidog"))
    
- **Mem01 goal**: fork Mem0, implement a `MemvidStore` that satisfies the same interface, keep the existing MCP HTTP surface (`add_memories`, `search_memory`, `list_memories`, `delete_all_memories`) and add a lightweight PDF pipeline.
    

---

## 2 Product Scope Changes

|Feature|Original Mem0|**Mem01**|
|---|---|---|
|Vector DB (Qdrant)|✅ default|➡ **Replaced by Memvid**|
|Vision-language ingestion|Planned via LLaVA|**Deferred**|
|PDF support|External example|**Built-in** (PyPDF-2 + PyMuPDF fallback)|
|Observability UI|OpenMemory dashboard|Re-used, add Memvid stats pane|
|Deployment|Docker Compose|**Same**, single container|

---

## 3 Success Metrics

|KPI|Target|
|---|---|
|P95 `search_memory` latency (1 M chunks)|**≤ 300 ms** ([apidog.com](https://apidog.com/blog/memvid/?utm_source=chatgpt.com "MemVid: Replacing Vector Databases with MP4 Files - Apidog"))|
|Storage efficiency vs Qdrant snapshot|**≥ 8 × smaller** ([github.com](https://github.com/Olow304/memvid?utm_source=chatgpt.com "Olow304/memvid: Video-based AI memory library. Store ... - GitHub"))|
|Recall@5 on Mem0 sample set|**≥ 0.85** (≤ 5 % drop)|
|RAM usage on 2 vCPU / 8 GB VPS|**≤ 6 GB peak**|
|Setup time for existing MCP client|**< 30 min** thanks to unchanged API ([mem0.ai](https://mem0.ai/blog/introducing-openmemory-mcp/?utm_source=chatgpt.com "Introducing OpenMemory MCP - Mem0"), [docs.mem0.ai](https://docs.mem0.ai/openmemory/overview?utm_source=chatgpt.com "Overview - Mem0 docs"))|

---

## 4 Functional Requirements

### 4.1 MCP API (unchanged)

- `add_memories`: Accept raw text or extracted PDF paragraphs.
    
- `search_memory`: Hybrid cosine similarity (Memvid) × Mem0 importance score.
    
- `list_memories`, `delete_all_memories`: Administrative. ([mem0.ai](https://mem0.ai/blog/introducing-openmemory-mcp/?utm_source=chatgpt.com "Introducing OpenMemory MCP - Mem0"))
    

### 4.2 Ingestion Pipelines

- **Text** → Mem0 embedder (OpenAI Ada-002 by default) → Memvid encoder. ([docs.mem0.ai](https://docs.mem0.ai/open-source/python-quickstart?utm_source=chatgpt.com "Python SDK - Mem0 docs"))
    
- **PDF**:
    
    1. Try **PyPDF 2** (pure-Py, zero deps). ([reddit.com](https://www.reddit.com/r/learnpython/comments/11ltkqz/which_is_faster_at_extracting_text_from_a_pdf/?utm_source=chatgpt.com "Which is faster at extracting text from a PDF: PyMuPDF or PyPDF2?"))
        
    2. Fallback to **PyMuPDF**, ≈15 × faster on large docs. ([reddit.com](https://www.reddit.com/r/learnpython/comments/11ltkqz/which_is_faster_at_extracting_text_from_a_pdf/?utm_source=chatgpt.com "Which is faster at extracting text from a PDF: PyMuPDF or PyPDF2?"), [kaggle.com](https://www.kaggle.com/code/toddgardiner/nlp-pypdf-vs-pymupdf-speed-test?utm_source=chatgpt.com "NLP-PyPDF vs PyMUPDF Speed Test - Kaggle"), [pymupdf.readthedocs.io](https://pymupdf.readthedocs.io/en/latest/app4.html?utm_source=chatgpt.com "Appendix 4: Performance Comparison Methodology - PyMuPDF"))
        
    3. Chunk by heading or 512-token windows; feed to embedder.
        

### 4.3 Storage Layer – `MemvidStore`

- Implements Mem0’s `AbstractStore` with `add`, `search`, `load_all`, `clear`. ([github.com](https://github.com/mem0ai/mem0?utm_source=chatgpt.com "mem0ai/mem0: Memory for AI Agents; Announcing OpenMemory MCP"))
    
- One MP4 per month; side-car `index.json` for offsets. ([github.com](https://github.com/Olow304/memvid?utm_source=chatgpt.com "Olow304/memvid: Video-based AI memory library. Store ... - GitHub"), [apidog.com](https://apidog.com/blog/memvid/?utm_source=chatgpt.com "MemVid: Replacing Vector Databases with MP4 Files - Apidog"))
    
- AES-256 optional encryption flag passed to Memvid encoder.
    

### 4.4 Admin & Observability

- Re-use OpenMemory React dashboard. Add a “Video Stats” tab (file size, frames, last compaction). ([mem0.ai](https://mem0.ai/blog/how-to-make-your-clients-more-context-aware-with-openmemory-mcp/?utm_source=chatgpt.com "How to make your clients more context-aware with OpenMemory MCP"))
    
- `/metrics` endpoint exporting Prometheus counters.
    

---

## 5 Non-Functional Requirements

|Aspect|Requirement|
|---|---|
|**Performance**|Single-process async FastAPI; Memvid index cached in RAM.|
|**Reliability**|Append-only MP4 writes + journal; crash-safe.|
|**Security**|No outbound network calls; optional token auth.|
|**Portability**|x86-64 & ARM64 (Apple Silicon, Pi 5).|

---

## 6 Technical Architecture

```
┌───MCP Client───┐      HTTP/JSON       ┌──────Mem01──────┐
│ (Cursor, etc.) │ ───────────────────▶ │  FastAPI Router │
└────────────────┘                      │  Mem0 Core      │
                                        │  PDF Worker     │
                                        │  MemvidStore    │
                                        └──────┬──────────┘
                                               ▼
                                   monthly_memory_2025-06.mp4
                                   monthly_memory_2025-06.json
```

---

## 7 Tech Stack

|Layer|Choice|Reason|
|---|---|---|
|API server|**FastAPI**|Async + Pydantic schemas|
|Memory core|**Mem0 0.1.x**|Stable, MIT-licensed ([github.com](https://github.com/mem0ai/mem0?utm_source=chatgpt.com "mem0ai/mem0: Memory for AI Agents; Announcing OpenMemory MCP"))|
|Storage|**Memvid 0.1.x**|MP4 compression, fast search ([github.com](https://github.com/Olow304/memvid?utm_source=chatgpt.com "Olow304/memvid: Video-based AI memory library. Store ... - GitHub"), [apidog.com](https://apidog.com/blog/memvid/?utm_source=chatgpt.com "MemVid: Replacing Vector Databases with MP4 Files - Apidog"))|
|Embeddings|OpenAI Ada-002 (pluggable local BGE)||
|PDF parsing|PyPDF 2 → PyMuPDF fallback ([reddit.com](https://www.reddit.com/r/learnpython/comments/11ltkqz/which_is_faster_at_extracting_text_from_a_pdf/?utm_source=chatgpt.com "Which is faster at extracting text from a PDF: PyMuPDF or PyPDF2?"), [kaggle.com](https://www.kaggle.com/code/toddgardiner/nlp-pypdf-vs-pymupdf-speed-test?utm_source=chatgpt.com "NLP-PyPDF vs PyMUPDF Speed Test - Kaggle"), [pymupdf.readthedocs.io](https://pymupdf.readthedocs.io/en/latest/app4.html?utm_source=chatgpt.com "Appendix 4: Performance Comparison Methodology - PyMuPDF"))||
|Container|Docker Compose; health-check on `/alive`||

---

## 8 Milestones & Timeline

|Week|Deliverable|
|---|---|
|1|Fork Mem0, rename to **mem01**; set up CI/CD & Docker skeleton|
|2|Implement `MemvidStore.add / search`; pass unit tests|
|3|Hook store into Mem0 core; basic text end-to-end demo|
|4|PDF extractor micro-service; ingestion benchmark|
|5|Replace Qdrant in OpenMemory; dashboard stats pane|
|6|Load-test (1 M chunks), docs, and **v0.1.0 beta** release|

---

## 9 Future Considerations

- **Vision support (LLaVA/VILA)** once GPU budget available.
    
- Sharding by user-ID for multi-tenant SaaS.
    
- Optional **Memvid->Parquet** export for analytics.
    

---

## 10 References

1. Mem0 GitHub repo and `AbstractStore` sources. ([github.com](https://github.com/mem0ai/mem0?utm_source=chatgpt.com "mem0ai/mem0: Memory for AI Agents; Announcing OpenMemory MCP"))
    
2. OpenMemory MCP announcement and tool list. ([mem0.ai](https://mem0.ai/blog/introducing-openmemory-mcp/?utm_source=chatgpt.com "Introducing OpenMemory MCP - Mem0"))
    
3. How to integrate clients via MCP. ([docs.mem0.ai](https://docs.mem0.ai/openmemory/overview?utm_source=chatgpt.com "Overview - Mem0 docs"))
    
4. “Make clients context-aware with OpenMemory MCP” blog post. ([mem0.ai](https://mem0.ai/blog/how-to-make-your-clients-more-context-aware-with-openmemory-mcp/?utm_source=chatgpt.com "How to make your clients more context-aware with OpenMemory MCP"))
    
5. Memvid GitHub README (concept & API). ([github.com](https://github.com/Olow304/memvid?utm_source=chatgpt.com "Olow304/memvid: Video-based AI memory library. Store ... - GitHub"))
    
6. Memvid compression benchmark article. ([apidog.com](https://apidog.com/blog/memvid/?utm_source=chatgpt.com "MemVid: Replacing Vector Databases with MP4 Files - Apidog"))
    
7. PyPDF 2 vs PyMuPDF speed discussion. ([reddit.com](https://www.reddit.com/r/learnpython/comments/11ltkqz/which_is_faster_at_extracting_text_from_a_pdf/?utm_source=chatgpt.com "Which is faster at extracting text from a PDF: PyMuPDF or PyPDF2?"))
    
8. Kaggle benchmark showing PyMuPDF speedup. ([kaggle.com](https://www.kaggle.com/code/toddgardiner/nlp-pypdf-vs-pymupdf-speed-test?utm_source=chatgpt.com "NLP-PyPDF vs PyMUPDF Speed Test - Kaggle"))
    
9. PyMuPDF performance appendix. ([pymupdf.readthedocs.io](https://pymupdf.readthedocs.io/en/latest/app4.html?utm_source=chatgpt.com "Appendix 4: Performance Comparison Methodology - PyMuPDF"))
    
10. Mem0 Python quick-start & embedder examples. ([docs.mem0.ai](https://docs.mem0.ai/open-source/python-quickstart?utm_source=chatgpt.com "Python SDK - Mem0 docs"))
    

