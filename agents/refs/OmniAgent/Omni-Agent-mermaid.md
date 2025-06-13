---
created: 2025-06-12T18:52
updated: 2025-06-12T18:54
---
Here’s a Mermaid flowchart capturing the end-to-end orchestration of the Omni Agent project, with all major components and their interactions:

```mermaid
flowchart TD
  %% UI Layer
  subgraph UI["User Interaction Layer"]
    direction LR
    WebApp[Web App UI]
    DiscordBot[Discord Bot]
  end

  %% Orchestration Layer
  subgraph Orch["Archon/DGM Orchestrator"]
    direction TB
    ArchonDGM[Archon / DGM Orchestrator]
    ToolRegistry[MCP Tool Registry]
    Sandbox[Sandboxed Exec Env]
    Benchmark[Benchmark Harness & Fitness]
    GenomeArchive[Genome & Memory Archive]
  end

  %% Agent Pool
  subgraph Agents["Agent Pool"]
    direction TB
    Agent["Agent Instance<br>with mem01 client"]
  end

  %% Memory & Compute
  subgraph Memory["Memory & Compute"]
    direction TB
    Mem01[mem01/memvid Storage]
    SleepSched[Sleep-Time Compute Scheduler]
    SleepCache[Precompute / Sleep Cache]
  end

  %% Flows
  WebApp -->|user request| ArchonDGM
  DiscordBot -->|user request| ArchonDGM

  ArchonDGM -->|spawn & evolve| Agent
  ArchonDGM --> ToolRegistry
  ArchonDGM --> Sandbox
  ArchonDGM --> Benchmark
  ArchonDGM --> GenomeArchive

  Agent -->|read/write| Mem01
  Agent -->|invoke tools| ToolRegistry
  Agent -->|execute code| Sandbox

  SleepSched -->|dispatch jobs| Agent
  Agent -->|generate offline insights| SleepCache
  ArchonDGM -->|analyze cache| SleepCache
  SleepCache -->|merge into prompt| Agent

  Benchmark -->|score variants| ArchonDGM
  GenomeArchive -->|inherit memory| ArchonDGM
```

**Legend / Notes**

- **Web App UI / Discord Bot**: Front-end entry points for human users.
    
- **Archon / DGM Orchestrator**: Central controller that spawns, mutates, evaluates, and selects agents based on fitness functions and lineage.
    
- **MCP Tool Registry**: Dynamic catalog of available tool‐endpoints agents can discover and call at runtime.
    
- **Sandboxed Exec Env**: Isolated runtime (e.g., Docker/K8s) for agents’ code execution and new tool scaffolding.
    
- **Benchmark Harness & Fitness**: Automated suite for evaluating agent variants on coding, reasoning, or domain-specific tasks.
    
- **Genome & Memory Archive**: Versioned store of agent “genomes” (models, prompts, tool sets) and high-value memory shards for hereditary inheritance.
    
- **Agent Instance**: LLM-driven “subagent” with mem01 client extensions for memory lookup (`search_memory`, `stream_memory`) and MCP tool invocation.
    
- **mem01/memvid Storage**: Video+JSON memory store providing fast retrieval and tombstone editing functions.
    
- **Sleep-Time Compute Scheduler**: Dispatches idle-time jobs to agents for offline chain-of-thought, summarization, and precompute.
    
- **Precompute / Sleep Cache**: Stores embeddings, extracted facts, and summaries to be merged into live prompts for faster, richer responses.