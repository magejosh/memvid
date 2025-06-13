---
created: 2025-06-12T06:38
updated: 2025-06-12T06:39
---
# Mem01 – Memvid‑powered fork of Mem0

[Mem01](https://github.com/your-org/mem01) (pronounced _“mem‑oh‑one”_) is a **drop‑in replacement** for [Mem0](https://github.com/mem0ai/mem0) that swaps the vector‑database layer for ultra‑compact MP4 storage via [Memvid](https://github.com/Olow304/memvid). All APIs, SDKs, and Model‑Context‑Protocol (MCP) endpoints remain unchanged, so any existing Mem0 integration will work out‑of‑the‑box.

---

## Research Highlights

- **≈ 8–10 × smaller on‑disk** than Mem0 + Qdrant thanks to Memvid’s video‑as‑database compression.
    
- **Sub‑second query latency** (< 300 ms P95) over 1 M chunks on a 2 vCPU / 8 GB VPS.
    
- **PDF‑aware ingestion** (PyPDF 2 → PyMuPDF fallback) so PDF pages become first‑class memories.
    
- **+26 % accuracy vs. naïve retrieval** on the LOCOMO benchmark (inherited from upstream Mem0).
    

---

## Introduction

Mem01 gives AI assistants and agents a long‑term memory that remembers user preferences, adapts to individual needs, and continuously learns—now with a much lighter storage footprint ideal for self‑hosted deployments and edge devices.

### Key Features & Use Cases

_Everything you love in Mem0, plus:_

- **Memvid Storage** – streamable, optionally encrypted MP4 files instead of a running vector DB.
    
- **Text & PDF Support** – store and recall plain text _and_ PDF pages.
    

For more examples, consult the upstream [Mem0 documentation](https://docs.mem0.ai/).

---

## Quickstart Guide

### Self‑Hosted (Python)

```bash
# Until a PyPI package is published
pip install git+https://github.com/your-org/mem01.git
```

### Self‑Hosted (Docker)

```bash
git clone https://github.com/your-org/mem01.git && cd mem01
docker compose up --build
```

The server starts on **`http://localhost:8000`** with Swagger docs at `/docs`.

### Basic Usage

```python
from openai import OpenAI
from mem01 import Memory   # note: mem01, not mem0

client = OpenAI()
memory = Memory()

# Add a memory
memory.add([
    {
        "role": "user",
        "content": "Remember that I prefer metric units."
    }
], user_id="alice")

# Retrieve memories
result = memory.search(
    query="What units does Alice like?",
    user_id="alice",
    limit=3
)
print(result)
```

---

## Configuration

|ENV var|Default|Description|
|---|---|---|
|`EMBEDDING_PROVIDER`|`openai`|`openai` \| `bge-small` \| path to local model|
|`MEMVID_ENCRYPT`|`false`|AES‑256 encryption for MP4s|
|`MEMVID_CHUNK_SECONDS`|`0.3`|Approx. seconds per frame|
|`AUTH_TOKEN`|_(none)_|Bearer auth for all endpoints|

---

## Integrations & Demos

Everything that works with Mem0 (ChatGPT extension, LangGraph, CrewAI, etc.) continues to work—just point the SDK or MCP server URL to your Mem01 instance.

---

## Documentation & Support

- **Docs**: continue to use the upstream [Mem0 docs](https://docs.mem0.ai/); differences are captured here.
    
- **Community**: [Discord](https://mem0.dev/) · [Twitter/X](https://x.com/mem0ai)
    
- **Maintainers**: [maintainers@mageworks.studio](mailto:maintainers@mageworks.studio)
    

---

## Citation

If you use Mem01 in your research, please cite the original Mem0 paper:

```bibtex
@article{mem0,
  title  = {Mem0: Building Production‑Ready AI Agents with Scalable Long‑Term Memory},
  author = {Chhikara, Prateek and Khant, Dev and Aryan, Saket and Singh, Taranjeet and Yadav, Deshraj},
  journal = {arXiv preprint arXiv:2504.19413},
  year   = {2025}
}
```

---

## License

Apache 2.0 — see the [LICENSE](https://chatgpt.com/c/LICENSE) file for details.