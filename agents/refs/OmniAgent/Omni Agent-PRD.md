---
created: 2025-06-12T17:20
updated: 2025-06-12T17:47
---
# OmniAgent – Product Requirements Document (MVP & Roadmap)

---

## 1. Project Overview

**OmniAgent** is a user-centric, multi-platform AI companion that adapts and evolves over time. Building on a core multi-agent, evolutionary architecture inspired by [Agent Foundry](Agent Foundry.md) and self-optimizing pipelines in [CogniFlow](https://chatgpt.com/g/g-p-684b4b2eaeb081918e1a40caf8cf4086-omniagent-mem01/c/CogniFlow.md), OmniAgent orchestrates best-of-breed open-source tools and FOSS repositories—such as mem01, Archon, and magejosh/nekro-agent—into a unified system. The core LLM itself is not bundled; instead, users configure API endpoints for any LLM provider (OpenAI, Gemini, Claude, OpenRouter, Ollama, or others), balancing state-of-the-art performance with economic viability and scalability. The MVP demonstrates this vision with a virtual companion agent performing tasks as a digital human companion across web, Discord, and native clients.

- **Web Portal:** A React/Next.js web app with secure login, a chat UI, and support for file uploads (text, images, audio, video).
    
- **Discord Integration:** A bot that joins servers and interacts as a user—reading channel messages & attachments, reacting with emoji/GIFs, and conversing naturally in threads.
    
- **Browser Extension:** A sidebar “companion” (à la Clippy) that observes page context and offers overlay chat on any website.
    
- **Native Companion Apps:** Lightweight avatar applications for Windows, macOS, Linux (Electron) and iOS/Android (React Native), providing system-wide accessible AI assistance.
    

Users can plug in their own API endpoints and model IDs, control compute priorities, and define evolutionary goals. Under the hood, the architecture is **modular and extensible**—OmniAgent’s multi-agent system can discover or even generate new tools and skills over time, continuously improving its capabilities.

---

## 2. Goals & Objectives

- **Seamless UX:** Meet users where they are—in the browser, on Discord, and on desktop or mobile—with a consistent and synchronized experience.
    
- **Adaptive Evolution:** Continuously optimize prompts, memory configurations, tool usage, and model/provider choices via an Archon-driven Darwin–Gödel Machine (DGM) controller that orchestrates self-improvement and evolutionary cycles.
    
- **Shard-Based Memory:** Manage both user and agent memory as isolated shards via a **mem01 MCP Server**, allowing per-tenant, per-agent MP4+JSON stores that can be inherited by new agent generations upon task success.
    
- **User-Controlled LLM Providers:** Let users configure any API-based LLM endpoints and model IDs—OpenAI, Gemini, Claude, OpenRouter, Ollama, or others—and set cost vs. latency priorities.
    
- **Tool Ecosystem:** Dynamically discover, bind, and evolve external tools via the Model Context Protocol (MCP)—spawn coding sub-agents that can scaffold new MCP servers or integrations on demand.
    
- **Open & Modular Foundation:** Leverage well-supported open-source projects and design loosely coupled modules to ensure iterative extensibility by both developers and the AI’s own evolutionary loops.
    

---

## 3. Feature Requirements

|#|Feature|Priority|
|--:|---|:-:|
|1|**Sleep-Time Compute** – Scheduled offline “thinking” jobs that anticipate user needs (pre-compute embeddings or reasoning chains) and integrate results into memory to reduce live query costs.|High|
|2|**Mem01 MCP Server & Hereditary Memory Shards** – A zero-DB memory service using [Memvid Memory MCP Server](memvid Memory MCP Server.md) + mem01 MCP server to manage agent- and user-specific memory shards. Successful task memories are archived with agent metadata so they can be inherited by cloned or evolved agents.|High|
|3|**Web Portal & Discord Bot (MVP)** – Secure web chat portal and Discord bot integration for multi-channel conversations (file uploads, channel monitoring, reactions, etc.). Demonstrates the virtual companion agent as a digital human companion.|High|
|4|**Config Screen** – An interface (web UI and Discord slash commands) to whitelist AI endpoints/models and manage API keys with a secure secrets backend (e.g., Vault or Kubernetes Secrets).|High|
|5|**Nekro-Agent Scaffold** – Integration of a sandboxed RPC framework (magejosh/nekro-agent) for safe code execution and multi-modal plugin extensions.|Medium|
|6|**Provider & Model Manager** – UI to manage endpoints, allowed model IDs, and define “priority profiles” (cost vs. speed preferences), with evolutionary A/B testing harness to optimize provider–model pairings.|High|
|7|**Archon-Orchestrated Darwin–Gödel Evolution Engine** – An Archon-based meta-controller that runs continuous self-improvement cycles, mutating agent “genomes” (prompts, tool configs, memory parameters) and validating new agent versions via benchmarks.|High|
|8|**Browser Extension** – In-page sidebar for any website, with content-script context awareness and quick-invoke buttons to trigger OmniAgent (e.g. summarize page, answer questions using page content).|Medium|
|9|**Native Companion Apps** – Installable desktop (Electron) and mobile (React Native) apps providing a persistent on-screen avatar and hotkey-triggered chat, syncing with web/Discord data.|Medium|
|10|**Observability & Security** – Comprehensive monitoring (analytics and logs) plus security features like prompt/embedding injection detection, JSON schema validators for tool outputs, and sandboxing/circuit-breakers for untrusted code.|High|

_(MVP will prioritize items 1–4, 6–7; items 5, 8–9 are slated for later phases.)_

---

## 4. Interaction Interfaces

1. **Web Portal**
    
    - OAuth2 / SSO options, role-based access control.
        
    - Rich chat UI with Markdown support, file uploads (images/audio/video), emoji picker, and GIF reactions.
        
    - Conversation history with memory-search backed by mem01 shards.
        
2. **Discord Bot**
    
    - Install via OAuth2; respects server permissions and configured channels.
        
    - Listens to messages & attachments, reacts with emoji/GIFs, replies in threads.
        
    - Supports slash commands for config and direct prompts; leverages Discord’s User-Installable App feature for personal bot instances.
        
3. **Browser Extension**
    
    - Sidebar popup on any webpage; reads page content with user permission for context-aware responses.
        
    - Optional screen capture/DOM snippet sharing for multi-modal memory ingestion.
        
    - Quick-action buttons (e.g., “Summarize this page”) invoking MCP tools in context.
        
4. **Native Companion Apps**
    
    - Electron desktop and React Native mobile builds.
        
    - Persistent avatar/hotkey to summon chat overlay on any app.
        
    - Clipboard and notification integration for seamless content sharing.
        
    - Syncs context and configuration via mem01 MCP server.
        

---

## 5. Provider & Model Management

- **Endpoint Registry:** Users can add custom REST or WebSocket AI service endpoints, each with authentication details.
    
- **Model Whitelist:** Specify allowed model IDs per endpoint to enforce compliance and preference.
    
- **Priority Profiles:** Define optimization goals (_minimize cost_, _optimize speed_, _maximize accuracy_) via sliders or presets; integrated into the Archon-orchestrated evolution decisions.
    
- **Evolutionary A/B Testing:** The DGM controller conducts periodic benchmarks swapping provider–model pairs, analyzing success rate, latency, and cost to refine priority profiles automatically.
    

---

## 6. Architecture Components

1. **Core Agent Framework**
    
    - A modular `Agent` class exposing `run(input)` that stages input through intent interpretation, shard-based memory retrieval, tool selection via MCP, prompt assembly, and LLM invocation.
        
2. **Search & Evolution**
    
    - **Block-Level Optimization:** Optuna for hyperparameter tuning; PEFT for prompt-prefix adaptation.
        
    - **Workflow Evolution:** DEAP + NetworkX to explore agent-topology graphs.
        
    - **Archon-Orchestrated DGM:** Uses Archon as the meta-controller to mutate agent genomes and archive performance+memory shards for successive generations.
        
3. **Memory Service (mem01 MCP Server)**
    
    - Exposes MCP-compliant endpoints (`search_memory`, `add_memory`, `stream_memory`) over FastAPI or Node.js.
        
    - Stores each tenant’s memory as an MP4+JSON index ([Memvid Memory MCP Server](memvid Memory MCP Server.md)) shard, isolating user and agent memories.
        
    - Enforces multi-tenancy, per-shard encryption, and role-based scopes.
        
    - Supports SSE for real-time memory streams.
        
4. **MCP Tool Platform**
    
    - A microservice registry (FastAPI) for JSON-RPC 2.0 tool invocation.
        
    - Dynamic discovery of tools (e.g., web search, calculator, custom agent-generated MCP servers).
        
    - Schema-validated input/output per MCP spec.
        
5. **Integration Layers**
    
    - Platform adapters: Discord bot (discord.py), browser extension content scripts, Electron/React Native wrappers.
        
    - Standardized communication via WebSockets or REST to the core agent.
        
6. **Provider Manager**
    
    - Stores endpoint & model profiles, user preferences, and priority profiles.
        
    - Monitors endpoint health (heartbeat checks, error rates) and enables failover logic.
        
    - Exposes APIs for the agent to fetch the optimal endpoint/model per query.
        
7. **Infrastructure & Orchestration**
    
    - Monorepo with Docker Compose for local MVP and Kubernetes + Helm for production scaling.
        
    - CI/CD pipelines via GitHub Actions.
        
    - Sandboxed Docker environments for coding agents generating new MCP tools, maintaining system safety.
        
8. **Observability & Security**
    
    - Experiment tracking (MLflow, W&B) and infra metrics (Prometheus/Grafana).
        
    - Error tracking with Sentry.
        
    - Input/output sanitization: `jsonschema`, regex filters, circuit-breakers for untrusted tool calls.
        
    - Regular security audits and formal safe-mode toggles for production deployments.
        

---

## 7. Technology Stack

|Layer / Component|Technologies & Libraries|
|---|---|
|**Core LLM Integration**|Configurable API endpoints for any LLM provider (OpenAI, Gemini, Claude, OpenRouter, Ollama), with optional local FOSS runtimes|
|**Evolution Engine**|Optuna, DEAP, NetworkX, Archon-based DGM algorithms ([2505.22954](https://arxiv.org/abs/2505.22954))|
|**Memory & MCP Server**|mem01 MCP Server (FastAPI or Node.js), memvid (MP4+JSON), FAISS, SSE streams|
|**Sleep-Time Compute**|Celery or APScheduler + Redis for background precompute jobs|
|**MCP Tools & Sandbox**|FastAPI (Python) or Express/Koa (Node.js) for JSON-RPC; magejosh/nekro-agent for sandbox RPC|
|**Web App UI**|React + Next.js, Tailwind CSS, Socket.io|
|**Discord Bot**|discord.py|
|**Browser Extension**|WebExtension API (JS/TS)|
|**Native Apps**|Electron (desktop), React Native (mobile)|
|**Config & Secrets**|Vault, AWS Secrets Manager, or Kubernetes Secrets|
|**CI/CD & Deployment**|Docker, Docker Compose, GitHub Actions, Kubernetes, Helm|
|**Monitoring & Logging**|Sentry, Prometheus, Grafana, MLflow, Weights & Biases|

_(All major components use well-supported open-source libraries. Archon and mem01 MCP server are included to enable orchestrated evolution and shard-based memory inheritance.)_

---

## 8. Phases & Milestones

1. **Phase I – Agent Foundation & Virtual Companion MVP**
    
    - Stand up **mem01 MCP Server** with basic `add_memory` and `search_memory` endpoints.
        
    - Integrate **Archon-Orchestrated Darwin–Gödel Evolution Engine**, enabling meta-controller loops to mutate agent genomes and archive successful memory shards for inheritance.
        
    - Build **Web Portal** and **Discord Bot** with config UI tied to secure secrets storage, demonstrating the virtual companion agent as a digital human companion.
        
2. **Phase II – Sleep-Time Compute & Optimization**
    
    - Implement **Sleep-Time Compute** pipeline (Celery or APScheduler + Redis) for scheduled offline “thinking” jobs, precomputing embeddings and reasoning chains to enrich memory before live interactions.
        
    - Validate integration of precomputed insights into live chat scenarios, reducing LLM invocation costs and latency.
        
3. **Phase III – Coding Agents & Tool Scaffold**
    
    - Prototype **Coding Agents** using magejosh/nekro-agent to scaffold MCP tool servers.
        
    - Expose an admin UI for human review and approval of AI-generated tools and workflows.
        
4. **Phase IV – Extended Interfaces**
    
    - Launch **Browser Extension** and test context-script accuracy.
        
    - Release **Electron** desktop app and basic **React Native** mobile app.
        
    - Ensure seamless memory sync across all clients via mem01 MCP server.
        
5. **Phase V – Hardening & Scale-Up**
    
    - Conduct security audits (prompt-injection, shard isolation).
        
    - Set up full observability stack (Prometheus/Grafana, Sentry) and run load tests.
        
    - Beta release with community feedback and iterate.
        
6. **Phase VI – Public Launch & Continuous Improvement**
    
    - Promote OmniAgent to wider audiences.
        
    - Refine Archon/DGM loops based on real-world performance and shard-heritage analytics.
        
    - Expand MCP tool ecosystem through community contributions.
        

---

## References

- [Agent Foundry](Agent Foundry.md)
    
- [CogniFlow](https://chatgpt.com/g/g-p-684b4b2eaeb081918e1a40caf8cf4086-omniagent-mem01/c/CogniFlow.md)
    
- [OmniAgent Overview](https://chatgpt.com/g/g-p-684b4b2eaeb081918e1a40caf8cf4086-omniagent-mem01/c/OmniAgent.md)
    
- [Original OmniAgent PRD](https://chatgpt.com/g/g-p-684b4b2eaeb081918e1a40caf8cf4086-omniagent-mem01/c/OmniAgent-PRD.md)
    
- [Memvid Memory MCP Server](memvid Memory MCP Server.md)
    
- [Mem01 MCP Server](https://chatgpt.com/g/g-p-684b4b2eaeb081918e1a40caf8cf4086-omniagent-mem01/c/mem01.md)