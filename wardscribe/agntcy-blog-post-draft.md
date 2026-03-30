# Building WardScribe: A Universal Registry for the Agent Economy

*The agent ecosystem is fragmenting. Here's how I'm building the infrastructure layer that outlasts it.*

---

## The Problem Nobody Wants to Talk About

AI agents are proliferating fast. Every major lab, every SaaS vendor, every enterprise engineering team is building them. But the agent ecosystem looks a lot like the container world in 2012, right before Docker. Except we're all shipping virtual machines in custom formats with no shared registry, no interoperability story, and no standard way to say "this agent does X."

OpenAI's agent format doesn't speak to Anthropic's. LangChain Hub is essentially a prompt library. There's no equivalent of `npm install` for agents. No versioning guarantees. No dependency resolution. No cryptographic signing. No quality scoring. You either build inside a single provider's ecosystem and accept the lock-in, or you rebuild the same infrastructure from scratch every time you switch.

I built WardScribe to solve that. It's a universal agent registry: cross-provider, format-agnostic, with a native packaging spec (WardPack), ML-driven quality scoring, and compatibility with the open standards emerging in the space.

One of those standards is **OASF**, from the AGNTCY project, and joining that ecosystem is part of the interoperability story I want to tell here.

---

## Interoperability: WardPack, OASF, A2A, and Everything Else

WardScribe's native format is **WardPack**, a packaging spec I designed specifically for production agents. It covers multi-dimensional confidence scoring, provider-specific LLM profiles, rich persona configuration, full SemVer dependency resolution, and Ed25519 signing. WardPack is what you use when you publish to WardScribe natively.

But WardPack alone doesn't solve the ecosystem problem. Agents exist in many formats, published by many teams using many different tools. WardScribe needs to speak all of them: import from anywhere, export to anywhere. That's the multi-format layer.

The format I'm most invested in standardizing around, after WardPack, is **OASF** from the AGNTCY project.

### What AGNTCY and OASF Are

[AGNTCY](https://agntcy.org) (pronounced "agency") is a Linux Foundation project backed by Cisco, Google, Dell, Oracle, Red Hat, and over 20 other companies. Their mission is to create the open infrastructure layer for agent interoperability: the standards and protocols that let agents work together regardless of who built them or what LLM they run on.

The core of AGNTCY is the **Open Agent Schema Framework (OASF)**, a structured specification for describing what an agent is and what it can do. OASF defines:

- **Agent records**: structured metadata with versioning, skills, domains, and locators
- **Skill taxonomy**: standardized capability classifications (50+ skills across 15 categories)
- **Domain ontologies**: industry-specific context (24 domains from healthcare to devops)
- **Protocol modules**: declarative support for MCP, SLIM, A2A, and observability
- **Evaluation data**: standardized quality metrics and performance benchmarks

Think of OASF as the schema behind an agent's `package.json`. The machine-readable contract that answers: what does this agent do, how do you run it, and how good is it?

It's also an IETF draft (draft-mp-agntcy-ads-01), which matters for enterprise procurement. "IETF-standard format" ends a lot of vendor evaluation conversations quickly.

I recently joined AGNTCY as an **Adopter**. I haven't contributed code yet, but I'm actively participating in the community and working toward Contributor status. My immediate intention is to formalize WardPack as an OASF Profile and contribute the extended skill taxonomies I've developed from real usage.

---

## OASF Compatibility: How It Works

The key decision on OASF: don't replace WardPack with it. Treat WardPack as a superset.

WardPack covers things OASF doesn't: multi-dimensional confidence scoring, provider-specific LLM profiles, rich persona configuration, full SemVer dependency resolution. Ditching WardPack would mean giving up the features that make WardScribe useful beyond basic discoverability.

Instead, I built **bidirectional OASF conversion**. Every agent in WardScribe can be exported as a valid OASF record and imported back with zero data loss. WardPack-specific data rides in the OASF `extensions` field for round-trip fidelity.

### Bidirectional Conversion in Practice

```bash
# Export any agent as OASF, compatible with any AGNTCY tooling
ward export @acme/research-agent --format oasf > agent.oasf.json

# Import an OASF record from AGNTCY Agent Directory
ward import-oasf ./agent.oasf.json
```

A WardPack manifest:

```yaml
name: Research Assistant
version: 2.4.1
description: Multi-step research with citations
skills:
  - web-search
  - summarization
  - citation-formatting
llm_profiles:
  claude:
    provider: anthropic
    model_family: claude-sonnet
confidence:
  global: 0.94
  dimensions:
    factual_accuracy: 0.96
    reasoning: 0.93
```

Becomes a valid OASF record:

```json
{
  "content_id": "sha256:abc123...",
  "record": {
    "name": "Research Assistant",
    "version": "2.4.1",
    "schema_version": "1.0.0",
    "skills": [
      "natural_language_processing/information_retrieval_synthesis",
      "natural_language_processing/summarization"
    ],
    "domains": ["knowledge_management"],
    "modules": ["mcp", "observability"]
  },
  "evaluation_data": {
    "overall_rating": 4.7,
    "overall_scores": {
      "factual_accuracy": 0.96,
      "reasoning": 0.93
    }
  },
  "extensions": {
    "wardpack:confidence_tensor": { "global": 0.94, "dimensions": { ... } },
    "wardpack:llm_profiles": { "claude": { "provider": "anthropic", ... } }
  }
}
```

The OASF record is fully consumable by any AGNTCY-compatible tool. The `extensions` field carries WardPack-specific data, so importing it back into WardScribe restores every field exactly.

---

## Architecture

WardScribe is a Go 1.23 backend on Fly.io, a Next.js frontend on Cloudflare Pages, a `ward` CLI distributed via Homebrew and goreleaser, and WardMind, a suite of fine-tuned models for agent quality evaluation.

### Backend: The Registry Core

```
┌─────────────────────────────────────────────────────────────┐
│                     WardScribe Registry                      │
├─────────────────────────────────────────────────────────────┤
│  OASF Converter   │  WardPack Parser  │  Confidence Engine  │
│  Skill Graph      │  Dependency Res.  │  Signature Verifier │
│  Security Scanner │  A2A Adapter      │  Export Formatters  │
└─────────────────────────────────────────────────────────────┘
         │                   │                   │
    PostgreSQL            Redis              MeiliSearch
  (versioned store)    (ML cache)          (semantic search)
```

OASF endpoints are first-class in the API:

```
GET  /api/v1/objects/:type/:slug/export?format=oasf
POST /api/v1/objects/import-oasf
GET  /api/v1/skills/taxonomy
GET  /api/v1/domains/taxonomy
```

The `backend/pkg/oasf/` package covers the full OASF type system, taxonomy mappings, and bidirectional conversion.

### Memory Efficiency: sync.Pool Throughout WardMind

WardMind's hot paths are built around `sync.Pool` to keep GC pressure low at inference time.

The **batch router** pools `[]batchItem` slices (cap 32) for each flush cycle. Rather than allocating a new slice every tick, the batcher grabs from the pool on lock, processes the batch, zeros the items to release references, and returns the slice for reuse:

```go
var batchItemPool = sync.Pool{
    New: func() interface{} {
        return make([]batchItem, 0, 32)
    },
}

func (b *RouterBatcher) flush() {
    b.mu.Lock()
    batch := b.pending
    b.pending = batchItemPool.Get().([]batchItem) // swap in pooled slice
    b.mu.Unlock()

    defer func() {
        for i := range batch { batch[i] = batchItem{} } // zero for GC
        batchItemPool.Put(batch[:0])
    }()
    // ... process batch
}
```

There's also a **buffer pool** for `bytes.Buffer` on every LLM API call, and a **builder pool** that pre-allocates 2KB `strings.Builder` instances for prompt construction, cutting out the grow-copy-discard cycle on every inference request.

### The Skill Graph

The Skill Graph is a knowledge graph built on top of OASF's taxonomy. It's the part of WardScribe I find most interesting to work on, because it turns "find agents" from keyword matching into something closer to actual reasoning about capability.

Each agent and skill node carries four distinct vector embeddings: a capability embedding (what the agent does), a semantic embedding (how it describes itself), a usage embedding (what problems it gets used for), and a composite that blends the three. That multi-embedding structure handles the gap between how publishers describe agents and how users search for them.

Nodes connect through eight relationship types:

```
requires        skill A needs skill B to function
enhances        skill A makes skill B more effective
conflicts       skill A is incompatible with skill B
replaces        skill A deprecates skill B
similar_to      skill A and B are semantically equivalent
composes        skill A works well alongside skill B
depends_on      runtime dependency
compatible_with interface-compatible (bidirectional)
```

Each edge carries a weight (0.0 to 1.0) and a source field. Edges can come from manual curation, vector similarity, usage co-occurrence patterns, LLM inference, or schema matching. The source matters: an edge built from 10,000 co-usage observations carries more weight than one inferred by a model once.

Search runs through a **GraphRAG** layer that combines three signals:

```
1. Vector search      cosine similarity on composite embeddings (weight 0.6)
2. Capability search  exact schema hash matching for interface compatibility
3. Graph expansion    traverse enhances/composes/similar_to edges from candidates
```

Final score:

```go
score = vectorScore*0.6 + graphScore*0.2 + capabilityScore*0.2
```

A query like "I need something that does summarization and citation tracking" surfaces agents tagged with those exact skills, but also agents that co-occur with them in real projects, and agents whose capability schemas are compatible even if they're described completely differently.

The graph also powers composition recommendations. Given a set of skills already in use, `RecommendComposition` pulls from co-usage patterns across all projects to surface what else teams tend to reach for. Collaborative filtering applied to agent composition.

### WardMind: Four Models, One Job

WardMind is four fine-tuned models, each with a specific job:

| Model | Size | Job |
|-------|------|-----|
| **Router** | ~0.6B | Route workflow steps to the right agent |
| **Classifier** | ~1.5B | Classify task domain, complexity, cost cap |
| **Architect** | ~8B | Generate WardPack YAML from natural language |
| **Evaluator** | ~8B | Score agent output quality (0.0 to 1.0 per dimension) |

The Router is small on purpose. It fires on every inference step and needs to return fast. The Architect and Evaluator are full 8B models because they're doing actual reasoning.

All four run on vLLM with SSE streaming, served from VastAI GPU instances.

Inference results write asynchronously to a partitioned `inference_logs` table (by month). Examples where confidence drops below 0.70 get flagged as `is_hard_example = true` and become the priority retraining set. A weekly GitHub Actions workflow checks the count, pulls new examples, and triggers a retrain if the threshold is exceeded. The pipeline is already running autonomously.

---

## The Training Pipeline

### Data

The training corpus is built from real-world data gathered from a wide range of sources. The pipeline ingests from multiple streams and processes them through a teacher-student distillation stage, where Claude Sonnet 4.6, Claude Haiku, Claude Opus 4.6, and Kimi-K2 are used as teacher models to label, structure, and quality-score examples before they enter fine-tuning.

The pipeline runs in three layers.

First, a Firecrawl-powered scraper ingests public agent repositories, documentation sites, and engineering content across the web. GitHub READMEs, official platform docs, and production engineering references are normalized and converted into structured training examples, teaching the models what real-world agent patterns look like across domains, languages, and use cases.

Second, I'm actively crawling and indexing publicly available agents from GitHub: repos that expose agent configurations, system prompts, tool definitions, and WardPack-compatible manifests. These get processed through the OASF converter, enriched with WardMind confidence scores, and made available in the registry. The goal is straightforward: if you've published an AI agent on GitHub with a manifest, it should show up in WardScribe without you doing anything.

Third, the registry itself is a data source. Inference logs from live usage, flagged hard examples, and co-usage patterns from real projects feed back into the training pipeline on a weekly cadence. The models improve as the registry grows.

Everything goes through a quality filter that deduplicates, validates format integrity, and removes low-entropy examples before anything enters the training corpus.

### Training Stack

Training runs on VastAI GPU instances using Unsloth with 4-bit quantization and LoRA adapters. The setup:

- **Base model:** Qwen2.5 series (0.6B, 1.5B, 8B)
- **Fine-tuning:** Unsloth `FastLanguageModel` with LoRA (r=16, alpha=32, dropout=0.05)
- **Quantization:** 4-bit NF4 (`load_in_4bit=True`)
- **Trainer:** HuggingFace TRL `SFTTrainer` with early stopping
- **Experiment tracking:** Weights & Biases

After training, weights are merged, quantized to GGUF, and pushed to a Cloudflare R2 model registry with semantic version tags.

### Evaluation: Automated Quality Gates

Before any WardMind model ships, it passes an automated evaluation harness benchmarking all four models against held-out validation sets. The harness measures:

- **Accuracy**: does the prediction match the expected output?
- **JSON parse rate**: is the structured output always valid JSON?
- **Latency**: p50, p95, p99 in milliseconds
- **Throughput**: requests/second at batch size 8

`thresholds.py` enforces minimum scores. If a Router model drops below 92% accuracy or p99 latency exceeds 200ms, the deployment is blocked. Every WardMind version that reaches production has been numerically validated.

### A2A Protocol Support

I implemented Google's Agent-to-Agent (A2A) protocol with server-side endpoints:

```
GET /api/v1/agents/:type/:slug/a2a   A2A Agent Card JSON
GET /.well-known/agent.json          A2A discovery endpoint
```

A2A cards are generated from WardPack manifests and expose the agent's capabilities in Google's format, another interoperability bridge alongside OASF.

### LLM Routing via Cloudflare Tunnel

The four WardMind models run on GPU instances. Connecting the Go backend to them securely, without opening firewall ports or managing VPN configs, needed a cleaner solution.

I use **Cloudflare Tunnel** (`cloudflared`) to create an encrypted, outbound-only connection from each GPU instance to Cloudflare's edge. Each of the four models gets its own subdomain route through the tunnel. The backend dispatches each inference call to the right model based on task type, and all traffic flows through Cloudflare's network, picking up DDoS protection, TLS termination, and request logging without additional infrastructure.

The Router model is called on every workflow step and needs to return in under 200ms, so endpoint selection has to be deterministic and the dispatch path has to be tight. The architecture means I can swap GPU instances or scale individual models without touching the backend. Only the tunnel configuration changes.

The frontend is on Cloudflare Pages. Model artifacts live in Cloudflare R2. From edge to storage to inference routing, Cloudflare is the connective tissue across WardScribe's infrastructure. That was an intentional choice for cost, latency, and operational simplicity.

### Security

A registry is only as good as the trust you can place in what it serves.

Every published WardPack is cryptographically signed by the publisher (Ed25519). Every artifact gets a SHA-256 content ID, matching OASF's `content_id` field. On publish, a security scanner runs pattern-based rules: shell injection, secret detection, encoded payload analysis, prototype pollution, XSS. Every agent composition generates an AI BOM tracking every model, tool, and dependency in the chain.

I'm also watching [KYA](https://kyapay.org), which came up in a recent AGNTCY Identity working group meeting, as a potential future layer. The idea of verifiable agent identity at the network level, something analogous to how TLS works for servers but applied to agents acting on behalf of users, is exactly the kind of trust primitive a registry needs over time. Nothing is integrated yet, but it's on the radar as an additional layer on top of the signing and scanning that's already in place.

---

## Immediate Plans

WardScribe is in **private beta** with early users already onboarded. Public launch is the immediate next milestone.

Three priorities for Q2:

**AGNTCY Agent Directory Integration.** Direct federation with the AGNTCY Agent Directory Service: one-click import into WardScribe, automatic sync of updates, and cross-registry search. The OASF bidirectional converter is already the foundation for this.

**SLIM (Secure Low-Latency Interactive Messaging) Support.** AGNTCY's messaging protocol provides the secure, scalable transport layer for agent-to-agent communication. Built on gRPC with end-to-end MLS encryption, SLIM enables real-time, multi-modal state exchange between agents running anywhere—data centers, browsers, mobile devices, or across organizational boundaries. Adding SLIM support means WardScribe-registered agents can communicate securely with any SLIM-compatible runtime, with microsecond-level latencies and quantum-safe cryptography. This complements A2A (Agent-to-Agent) for capability discovery and MCP for tool access, completing the interoperability stack.

**WardPack as an OASF Profile.** I've already proposed this to the AGNTCY working group. The idea is to formalize WardPack's enterprise extensions (confidence tensor, LLM profiles, persona configuration) as an OASF Profile, similar to how OCI artifacts extend the container image spec. The proposed new skill categories for the OASF taxonomy:

```yaml
agent_quality_assurance:
  - confidence_scoring
  - behavioral_evaluation
  - drift_detection

cross_provider:
  - model_portability
  - provider_abstraction
  - unified_tooling
```

---

## Future Vision

The long-term goal: make WardScribe the infrastructure layer for the open agent economy, where agents are as composable and portable as npm packages but with the trust and governance requirements that enterprise AI demands.

**Phase 1: Foundation (Now)**
The registry is in private beta with early users. OASF export/import is shipped. WardMind is running. The `ward` CLI is available.

**Phase 2: Federation (2026)**
Multi-registry federation. Agents published on WardScribe are discoverable from AGNTCY Directory, and vice versa. Decentralized discovery backed by OASF's content-addressed record format. Cross-registry reputation that can't be gamed by gaming a single platform.

**Phase 3: Autonomous Composition (2027)**
Agent-to-agent negotiation via SLIM. The Skill Graph composing multi-agent workflows from natural language requirements. Automated compliance verification for EU AI Act and SOC 2. Agents that improve their own WardPack metadata based on observed performance.

### Nemotron 3 Super and Agent Swarms

One model I'm watching closely for the next generation of WardMind is NVIDIA's **Nemotron 3 Super 120B-A12B** (released March 2026). It's a hybrid architecture: interleaved Mamba-2 and Transformer attention layers, a LatentMoE routing mechanism that activates only 12B of the 120B parameters per forward pass, and native Multi-Token Prediction heads that enable speculative decoding out of the box. The headline number is a 1M token context window with strong retention out to 512k (RULER: 95.67%).

What makes it relevant for WardScribe is that it was RL-trained across 21 agentic environments, including multi-turn tool use, coding agents, and complex task completion benchmarks (TauBench, SWE-Bench, Terminal Bench). NVIDIA's own recommended deployment pattern is a "Super + Nano" split: a smaller, fast model handles high-volume routing and classification while Super handles deep multi-step reasoning. That maps almost exactly onto how WardMind is already structured.

The plan is to explore Nemotron 3 Super as the base model for the next iteration of the WardMind Architect, the 8B model that currently generates WardPack definitions from natural language. Moving to a Nemotron-based Architect brings a dramatically larger reasoning budget and longer context for complex multi-agent workflow planning.

More interesting is what this opens up for **agent swarms**: instead of a single Architect generating a monolithic workflow, a swarm of specialized WardMind models could collaborate, each proposing, critiquing, and refining different parts of a multi-agent composition before a final plan is committed. Nemotron's programmable reasoning effort (you can budget exactly how many tokens the model spends thinking before it responds) makes it feasible to orchestrate this kind of model-to-model deliberation at reasonable latency and cost. This is still a forward-looking idea, not a shipped feature. But Nemotron 3 Super is the first model that makes it feel tractable.

### Agent Identity

One thread I keep pulling on is agent identity. AGNTCY has an Identity working group focused on publisher verification and trust models, and that intersection of registry and identity is where I think some genuinely hard problems live.

The question isn't just "who published this agent?" I already have Ed25519 publisher signing for that. The deeper question is: what is the identity of the agent itself? When an agent acts on behalf of a user, calls external APIs, or communicates with another agent via SLIM, how does the receiving system know what it's talking to, what it's authorized to do, and whether it's been tampered with since it was signed?

In a recent AGNTCY Identity working group meeting, someone shared [kyapay.org](https://kyapay.org), and it sparked some thinking about decentralized identity models applied to agents. Something analogous to DIDs but designed for the agentic context: portable across providers, revocable, and verifiable without a central authority. The constraint is that it has to work across the LLM and runtime diversity that WardScribe already handles. An agent identity system that only works with OpenAI-hosted agents is useless here.

Nothing is decided. It might be a contribution to the AGNTCY Identity working group. It might be a WardPack extension for identity metadata. But a registry without a coherent agent identity model is just a file store with a search box, and that's not what I'm building.

---

## Why This Matters

Container infrastructure fragmented before Docker. Package ecosystems fragmented before npm and pip. Every time, the industry converged on open standards, and the platforms that embraced those standards early ended up with network effects that proprietary alternatives couldn't match.

AI agents are going through the same thing. OASF is the most credible open standard in the space right now: IETF-draft status, Linux Foundation hosting, Cisco and Google as backers. I'm not betting on a standard winning. I'm building on the one that's already winning.

I built WardScribe end-to-end: the Go backend, the CLI, the ML training pipeline, the frontend, the OASF implementation. It's a solo project so far, which is both a constraint and a clarity. Every architectural decision reflects a real opinion about what agent infrastructure should look like, not a committee compromise.

I'm looking for the right partners: infrastructure companies, enterprise AI teams, and platform builders who see the same fragmentation problem and want to solve it at the standards layer rather than inside a single provider. If you're building something that needs a registry, or if you want to see WardPack proposals land in the AGNTCY specification, I'd like to talk. I'm also raising a pre-seed round to accelerate the public launch, the federation work, and the WardMind training pipeline.

The agent economy is coming. The question isn't whether there will be a registry. There will be several. The question is whether those registries are built on open standards that let agents work anywhere, or proprietary silos that recreate the same fragmentation we're trying to solve.

I'm building for the open version.

---

**Try it:**
```bash
# Install the CLI
brew install wardscribe/tap/ward

# Initialize an agent
ward init my-agent && cd my-agent
ward add skill web-search
ward publish

# Export to OASF
ward export --format oasf
```

**Get in touch:**
- Registry: [wardscribe.io](https://wardscribe.io)
- AGNTCY: [agntcy.org](https://agntcy.org)
- CLI source: Apache 2.0

---

*WardScribe is an AGNTCY Adopter, a member of the Linux Foundation project for agent interoperability. March 2026.*
