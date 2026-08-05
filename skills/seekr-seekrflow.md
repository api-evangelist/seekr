---
name: Seekr
description: Use when building, training, deploying, and managing AI agents and models. Reach for this skill when agents need to create agents with tools, fine-tune models on custom data, build RAG systems with vector databases, run inference, or understand model outputs through explainability features.
metadata:
    mintlify-proj: seekr
    version: "1.0"
---

# SeekrFlow Skill

## Product summary

SeekrFlow is an enterprise AI development platform for building, customizing, and deploying generative AI agents and models with full control over data, training, and inference. Agents are configurable AI systems that reason through problems and execute tasks using models, tools, and instructions. The platform includes a data engine for preparing training data, fine-tuning capabilities for domain-specific models, vector databases for RAG, deployments for hosting models, and explainability tools for understanding model decisions.

**Key entry points:**
- Python SDK: `seekrai` package (Python 3.9+)
- REST API: `https://flow.seekr.com/v1/`
- Authentication: API key in `SEEKR_API_KEY` environment variable or `Authorization` header
- Team scoping: `SEEKR_TEAM_ID` environment variable or `x-team-id` header

**Primary docs:** https://docs.seekr.com/flow

## When to use

Use SeekrFlow when you need to:

- **Create and run agents** – Build AI systems that reason and execute tasks autonomously using models and tools
- **Attach tools to agents** – Enable agents to search files (RAG), search the web, execute Python code, or call other agents
- **Fine-tune models** – Train base models on custom domain data using instruction fine-tuning, context-grounded fine-tuning, or reinforcement learning
- **Build RAG systems** – Upload documents, create vector databases with semantic search, and attach file search tools to agents
- **Deploy models** – Host base or fine-tuned models on dedicated compute for inference
- **Understand model outputs** – Attribute agent responses to their sources (retrieved context, tool calls, system instructions)
- **Prepare training data** – Ingest PDFs, DOCX, Markdown, and PowerPoint files; convert to structured training datasets

## Quick reference

### SDK initialization

```python
from seekrai import SeekrFlow
import os

client = SeekrFlow(api_key=os.environ.get("SEEKR_API_KEY"))
# Or use AsyncSeekrFlow for async/await
```

### Agent workflow

| Task | Code |
|------|------|
| Create agent | `client.agents.create(CreateAgentRequest(name="...", instructions="...", model_id="...", tool_ids=[...]))` |
| List agents | `client.agents.list_agents()` |
| Update agent | `client.agents.update(agent_id, UpdateAgentRequest(...))` |
| Promote (activate) | `client.agents.promote(agent_id)` |
| Demote (deactivate) | `client.agents.demote(agent_id)` |
| Delete agent | `client.agents.delete(agent_id)` |

### Thread and run workflow

| Task | Code |
|------|------|
| Create thread | `client.agents.threads.create()` |
| Add message | `client.agents.threads.create_message(thread_id, role="user", content="...")` |
| Run agent (sync) | `client.agents.runs.run(agent_id, thread_id, stream=False)` |
| Run agent (stream) | `client.agents.runs.run(agent_id, thread_id, stream=True)` |
| List messages | `client.agents.threads.list_messages(thread_id)` |
| Delete thread | `client.agents.threads.delete(thread_id)` |

### Tool types

| Type | Use case | Class |
|------|----------|-------|
| File search | Semantic search across vector databases (RAG) | `CreateFileSearch` |
| Web search | Real-time web information retrieval | `CreateWebSearch` |
| Custom tools | Python code execution with developer functions | `CreateRunPython` |
| Agent as tool | Delegate to sub-agents; multi-agent workflows | `CreateAgentAsTool` |
| MCP connector | Connect to external MCP providers | `CreateMCPConnector` |

### Vector database workflow

| Task | Code |
|------|------|
| Create database | `client.vector_database.create(model="intfloat/e5-mistral-7b-instruct", name="...", description="...")` |
| Upload file | `client.files.upload("path.pdf", purpose="alignment")` |
| Start ingestion | `client.vector_database.create_ingestion_job(database_id, files=[file_id], token_count=512)` |
| Check status | `client.vector_database.retrieve_ingestion_job(database_id, job_id)` |
| List databases | `client.vector_database.list()` |
| Delete database | `client.vector_database.delete(database_id)` |

### Fine-tuning workflow

| Task | Code |
|------|------|
| Create project | `client.projects.create(name="...", description="...")` |
| Upload training file | `client.files.upload("training.jsonl", purpose="fine-tune")` |
| Create fine-tune job | `client.fine_tuning.create(training_config=..., infrastructure_config=..., project_id=...)` |
| Check status | `client.fine_tuning.retrieve(fine_tune_id)` |
| List fine-tunes | `client.fine_tuning.list()` |
| Cancel job | `client.fine_tuning.cancel(fine_tune_id)` |

### Deployment workflow

| Task | Code |
|------|------|
| Deploy base model | `client.deployments.create(name="...", model_type=DeploymentType.BASE_MODEL, model_id="...", n_instances=1)` |
| Deploy fine-tuned model | `client.deployments.create(name="...", model_type=DeploymentType.FINE_TUNED_RUN, model_id="ft-id", n_instances=1)` |
| Promote deployment | `client.deployments.promote(deployment_id)` |
| List deployments | `client.deployments.list()` |

### Explainability

| Task | Code |
|------|------|
| Attribute from run | `client.explainability.get_context_attribution_from_run(thread_id, granularity="sentence", top_k=5)` |
| Attribute raw context | `client.explainability.get_context_attribution(context="...", query="...", response="...")` |

## Decision guidance

### When to use reasoning effort levels

| Level | Use case |
|-------|----------|
| `LOW` | Simple, latency-sensitive tasks with few tools; prioritize speed |
| `MEDIUM` | General-purpose agents; balances speed and accuracy (default) |
| `HIGH` | Complex workflows with many tools; accuracy matters more than latency |

### When to use ingestion methods for vector databases

| Method | Use case |
|--------|----------|
| `accuracy-optimized` | PDFs with complex layouts, tables, or OCR needs; up to 30 min for 100+ pages |
| `speed-optimized` | Large documents where speed matters; ~3 min regardless of size |

### When to use chunking methods

| Method | Use case |
|--------|----------|
| `markdown` | Documents with clear heading hierarchy; default choice |
| `semantic` | Preserve semantic boundaries; slower but more coherent chunks |
| `sliding` | Fixed-size overlapping windows; good for dense technical content |

### When to use in-context learning vs. RAG vs. fine-tuning

| Approach | When to use | Constraints |
|----------|------------|-------------|
| **In-context learning** | Single document fits in context window (4k–16k tokens); no training needed | Limited to one document; impractical for large document sets |
| **RAG (vector database)** | Multiple documents; semantic search needed; no model retraining | Requires vector database setup; retrieval quality depends on chunking |
| **Fine-tuning** | Domain-specific knowledge; consistent precision needed; scale beyond single document | Requires training data preparation; longer setup time |

## Workflow

### Build a simple RAG agent

1. **Upload a document** – Use `client.files.upload(path, purpose="alignment")` to upload a PDF, DOCX, or Markdown file
2. **Create a vector database** – Call `client.vector_database.create(model="intfloat/e5-mistral-7b-instruct", name="...", description="...")`
3. **Ingest the document** – Start an ingestion job with `client.vector_database.create_ingestion_job(database_id, files=[file_id], token_count=512, overlap_tokens=50)` and poll until `status == "completed"`
4. **Create a FileSearch tool** – Wrap the database with `client.tools.create(CreateFileSearch(name="...", description="...", config=FileSearchConfig(file_search_index=database_id)))`
5. **Create an agent** – Call `client.agents.create(CreateAgentRequest(name="...", instructions="...", model_id="...", tool_ids=[tool_id]))`
6. **Promote the agent** – Call `client.agents.promote(agent_id)` and wait for `status == "Active"`
7. **Run the agent** – Create a thread, add a message, run with `client.agents.runs.run(agent_id, thread_id, stream=False)`, and retrieve the response

### Fine-tune a model on custom data

1. **Prepare training data** – Upload raw documents (PDFs, DOCX, Markdown) or prepare a JSONL/Parquet file with training examples
2. **Create a project** – Call `client.projects.create(name="...", description="...")`
3. **Upload training file** – Use `client.files.upload(path, purpose="fine-tune")` to upload JSONL or Parquet
4. **Configure training** – Create `TrainingConfig` with model, epochs, batch size, learning rate; create `InfrastructureConfig` with compute specs
5. **Start fine-tuning** – Call `client.fine_tuning.create(training_config=..., infrastructure_config=..., project_id=...)`
6. **Monitor progress** – Poll `client.fine_tuning.retrieve(fine_tune_id)` to check status and retrieve events for loss visualization
7. **Deploy the model** – Create a deployment with `client.deployments.create(model_type=DeploymentType.FINE_TUNED_RUN, model_id=fine_tune_id, ...)`

### Understand agent responses with explainability

1. **Run an agent** – Complete a thread run with an agent
2. **Get attribution** – Call `client.explainability.get_context_attribution_from_run(thread_id, granularity="sentence", top_k=5)`
3. **Interpret results** – Review `segments` (one per sentence) and their `sources` (retrieved context, tool calls, instructions) with `attribution` scores (positive = supported, near 0 = little influence)

## Common gotchas

- **Agent status after creation** – Agents are automatically promoted on creation. If you update an agent in `Inactive` or `Failed` state, you must manually promote it again.
- **Tool updates propagate immediately** – Updating a tool affects all agents using it right away. Demote agents if you need to test changes first.
- **Cannot delete active tools** – Demote or delete the agent using the tool before deleting the tool itself.
- **Ingestion method differences** – The UI always uses `speed-optimized` ingestion; the SDK defaults to `accuracy-optimized`. Choose explicitly based on your document size and accuracy needs.
- **Markdown files skip ingestion** – Markdown files do not need the ingestion step (Step 3 in file-ingestion docs). Upload them directly for use in fine-tuning or RAG.
- **Vector database ingestion is async** – Always poll the ingestion job status; do not assume it completes immediately.
- **Custom function docstrings are required** – Custom tools must have Google-style docstrings with Args, Returns, and descriptions. The agent relies on these to decide when to invoke the function.
- **Custom functions run in Pyodide sandbox** – Only pure Python packages work; C extensions and system dependencies are not supported. For complex logic, call an external service.
- **Fine-tuning hyperparameters interact** – Learning rate and batch size are coupled; increasing batch size often requires increasing learning rate proportionally. Tune learning rate first.
- **Deployment status is async** – Deployments move through `Pending` → `Active` states. Poll `client.deployments.retrieve()` to wait for readiness.
- **Team scoping is required for multi-tenant setups** – Set `SEEKR_TEAM_ID` or pass `x-team-id` header if working with multiple teams; otherwise requests use your personal workspace.

## Verification checklist

Before submitting agent or model work:

- [ ] Agent has a clear, specific `instructions` field that guides reasoning and tool use
- [ ] All tool IDs referenced in `tool_ids` exist and are not linked to other active agents (if deleting)
- [ ] Agent is promoted and status is `Active` before running inference
- [ ] Thread and message IDs are valid and belong to the correct agent
- [ ] Vector database ingestion job shows `status == "completed"` before attaching FileSearch tool
- [ ] Fine-tuning training file is in correct format (JSONL or Parquet) with proper schema
- [ ] Fine-tuning job status is checked; loss curve shows downward trend (not flat or rising)
- [ ] Deployment is promoted and status is `Active` before routing inference traffic
- [ ] Custom function has valid Google-style docstring with Args and Returns
- [ ] Custom function tested locally before uploading
- [ ] Explainability attribution scores are interpreted correctly (positive = supported, near 0 = little influence)
- [ ] API key is set as environment variable or passed securely; never hardcoded in version control

## Resources

**Comprehensive page listing:** https://docs.seekr.com/llms.txt

**Critical documentation:**
- [SeekrFlow SDK Getting Started](https://docs.seekr.com/flow/sdk/getting-started) – Install SDK, authenticate, make first API call
- [Create and manage agents](https://docs.seekr.com/flow/sdk/agents/create-agents) – Agent configuration, reasoning effort, output settings
- [Run an agent](https://docs.seekr.com/flow/sdk/agents/run-an-agent) – Threads, messages, runs, streaming
- [Tools overview](https://docs.seekr.com/flow/sdk/agents/tools) – Tool types, creation, linking to agents
- [Create and populate a vector database](https://docs.seekr.com/flow/sdk/data-engine/create-and-populate-a-vector-database) – Vector DB setup, ingestion, chunking
- [Create a fine-tuning job](https://docs.seekr.com/flow/sdk/fine-tuning/create-fine-tuning-job) – Training config, hyperparameters, monitoring
- [Deployments](https://docs.seekr.com/flow/sdk/deployments) – Deploy base and fine-tuned models
- [Attribute a response to its sources](https://docs.seekr.com/flow/sdk/explainability/context-attribution) – Understand model decisions
- [Simple RAG agent recipe](https://docs.seekr.com/flow/recipes/simple-rag-agent) – End-to-end RAG example

---

> For additional documentation and navigation, see: https://docs.seekr.com/llms.txt