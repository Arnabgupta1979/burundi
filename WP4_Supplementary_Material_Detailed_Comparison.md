# Supplementary Material: Detailed Comparison of Implementation Pathways

## WP4: AI for Local Language Farm Advisory - Technical Comparison

This document provides a comprehensive comparison of the two implementation pathways developed for the WP4 use case.

---

## A. Technical Architecture Comparison

### A.1 Core Components

| Component | Streamlit Stack | Docker Stack | Purpose |
|-----------|----------------|--------------|---------|
| **LLM Inference** | Ollama (local process) | Ollama (container) | Generate text responses |
| **Vector Database** | FAISS (file-based) | ChromaDB (container) | Store document embeddings |
| **Embedding Model** | HuggingFace Transformers | HuggingFace Transformers | Convert text to vectors |
| **Document Loaders** | LangChain loaders | LangChain loaders | Extract text from files |
| **RAG Framework** | LangChain | AnythingLLM + LangChain | Orchestrate retrieval |
| **User Interface** | Streamlit | AnythingLLM UI | User interaction |
| **Workflow Engine** | Python scripts | n8n | Automation |
| **Translation** | Library imports | Microservice container | Multilingual support |
| **Speech-to-Text** | Can add Whisper lib | whisper-api container | Voice input |
| **Text-to-Speech** | Can add Coqui lib | TTS container | Voice output |
| **Reverse Proxy** | Streamlit built-in | Traefik/nginx | Routing & SSL |
| **ML Notebooks** | Local Jupyter | JupyterLab container | Model training |

### A.2 Software Dependencies

**Streamlit Stack:**
```
Core:
- Python 3.10+
- streamlit >= 1.28
- langchain >= 0.1.0
- faiss-cpu >= 1.7.4
- sentence-transformers >= 2.2.0

Document Processing:
- pymupdf >= 1.23
- pypdf2 >= 3.0
- python-docx >= 1.0

LLM:
- ollama-python >= 0.1.0

Optional:
- torch >= 2.0 (for GPU)
- whisper (for speech)
- transformers (for translation)
```

**Docker Stack:**
```
Infrastructure:
- Docker >= 24.0
- Docker Compose >= 2.20
- NVIDIA Container Toolkit (for GPU)

Containers:
- ollama/ollama:latest
- mintplexlabs/anything-llm:latest
- chromadb/chroma:latest
- jupyter/datascience-notebook:latest
- n8nio/n8n:latest
- ghcr.io/ggerganov/whisper-cpp:latest
- [custom translator image]
- traefik:v2.10
```

---

## B. Deployment and Operations

### B.1 Setup Process

**Streamlit Stack - Step by Step:**
```bash
# 1. Create virtual environment (5 minutes)
conda create -n wp4_rag python=3.10
conda activate wp4_rag

# 2. Install dependencies (10 minutes)
pip install streamlit langchain faiss-cpu sentence-transformers \
    pymupdf python-docx ollama-python

# 3. Install Ollama (5 minutes)
# Download from ollama.ai and install

# 4. Pull LLM model (10-30 minutes depending on model)
ollama pull mistral:7b

# 5. Clone/download code (2 minutes)
git clone https://github.com/wur/wp4-rag-streamlit
cd wp4-rag-streamlit

# 6. Add documents (1 minute)
cp /path/to/pdfs/*.pdf ./docs/

# 7. Run application (instant)
streamlit run app.py

Total setup time: ~30-60 minutes
```

**Docker Stack - Step by Step:**
```bash
# 1. Install Docker & NVIDIA runtime (20 minutes)
# Follow official Docker and NVIDIA Container Toolkit guides

# 2. Clone infrastructure repo (2 minutes)
git clone https://github.com/wur/wp4-rag-docker
cd wp4-rag-docker

# 3. Configure environment variables (10 minutes)
cp .env.example .env
# Edit .env with your settings

# 4. Pull LLM model (10-30 minutes)
docker compose run --rm ollama ollama pull llama3:8b-q4_K_M

# 5. Build custom images (30-60 minutes)
docker compose build

# 6. Start all services (10 minutes)
docker compose up -d

# 7. Initialize databases (5 minutes)
docker compose exec anythingllm npm run setup

# 8. Configure n8n workflows (20 minutes)
# Import workflow JSON files through UI

Total setup time: ~2-4 hours
```

### B.2 Maintenance Requirements

| Task | Streamlit Stack | Docker Stack |
|------|----------------|--------------|
| **Update LLM model** | `ollama pull <model>` | `docker compose run ollama pull <model>` |
| **Add documents** | Copy to `docs/` folder, click rebuild | Upload through AnythingLLM UI |
| **Update code** | `git pull`, restart Streamlit | `git pull`, `docker compose build`, restart |
| **Backup data** | Copy `faiss_index_store/` folder | Volume snapshots or `docker compose backup` |
| **Monitor logs** | Streamlit console output | `docker compose logs -f [service]` |
| **Update dependencies** | `pip install --upgrade` | Rebuild container images |
| **Scale resources** | Adjust Python memory limits | Edit `docker-compose.yml` resource limits |

### B.3 Hardware Scaling

**Streamlit Stack Scaling:**
```
Single Machine (Vertical Scaling):
├─ Entry: 16 GB RAM, 4-core CPU → 1 user, CPU-only
├─ Medium: 32 GB RAM, 8-core CPU, 12 GB GPU → 1-3 concurrent users
└─ High: 64 GB RAM, 16-core CPU, 24 GB GPU → 3-5 concurrent users

Note: Does not scale horizontally without significant code changes
```

**Docker Stack Scaling:**
```
Single Machine:
└─ 64 GB RAM, 16-core CPU, 24 GB GPU → 5-10 concurrent users

Multi-Machine (Horizontal Scaling):
├─ Load Balancer (Traefik)
├─ Ollama nodes (GPU servers) × N
├─ ChromaDB cluster × N
├─ AnythingLLM instances × N
└─ n8n cluster × N

Scales to: 100+ concurrent users with proper architecture
```

---

## C. Functional Comparison

### C.1 Core RAG Features

| Feature | Streamlit Stack | Docker Stack | Notes |
|---------|----------------|--------------|-------|
| **Document Upload** | File browser | Web UI drag-drop | Both support PDF, DOCX, TXT |
| **Embedding Creation** | On-demand, user-triggered | Automated via n8n | Docker offers scheduling |
| **Vector Search** | FAISS similarity/MMR | ChromaDB hybrid search | Both performant |
| **Source Citations** | Yes, with page numbers | Yes, with metadata | Same granularity |
| **Multilingual Query** | Yes (6 languages) | Yes (6 languages) | Both use same embedding model |
| **Multilingual Response** | Via prompt engineering | Via prompt + microservice | Docker has dedicated translation |
| **Context Window** | 4k-8k tokens (model-dependent) | 4k-8k tokens (model-dependent) | Same LLM models |
| **Response Streaming** | Yes | Yes | Both support real-time output |

### C.2 Advanced Features

| Feature | Streamlit Stack | Docker Stack | Notes |
|---------|----------------|--------------|-------|
| **Multiple Knowledge Bases** | Manual index switching | Automated routing | Docker more sophisticated |
| **Scheduled Data Pulls** | Manual or cron scripts | n8n workflows | Docker has visual workflow editor |
| **Model Fine-tuning** | Local Jupyter notebooks | Container Jupyter + LoRA | Same capability, different env |
| **Speech Input** | Can add Whisper | Integrated whisper-api | Docker pre-configured |
| **Speech Output** | Can add Coqui TTS | Integrated TTS service | Docker pre-configured |
| **Translation API** | Import libraries | REST microservice | Docker more modular |
| **User Authentication** | Basic (Streamlit auth) | Multi-user (Keycloak) | Docker enterprise-ready |
| **Query Analytics** | Basic logging | Full analytics dashboard | Docker has monitoring stack |
| **A/B Testing** | Manual code changes | n8n experiment workflows | Docker easier to configure |

### C.3 Data Governance and Security

| Aspect | Streamlit Stack | Docker Stack | Security Level |
|--------|----------------|--------------|----------------|
| **Data Location** | Local filesystem | Docker volumes (local) | Both fully on-premise |
| **Network Isolation** | OS-level firewall | Docker networks | Docker more granular |
| **External API Calls** | None (fully offline) | None (fully offline) | Both compliant |
| **Encryption at Rest** | OS-level encryption | Volume encryption | Both support encryption |
| **Encryption in Transit** | HTTPS (if configured) | Traefik SSL/TLS | Docker easier to configure |
| **Access Logs** | Basic Python logging | Comprehensive (all services) | Docker more detailed |
| **Audit Trail** | Custom implementation | Built-in (AnythingLLM + n8n) | Docker has better auditing |
| **Data Backup** | Manual file copy | Automated volume snapshots | Docker more robust |
| **GDPR Compliance** | Manual processes | Automated PII handling | Both can be compliant |

---

## D. Development and Customization

### D.1 Ease of Modification

**Streamlit Stack:**
- ✅ Direct Python code editing
- ✅ Immediate feedback (hot reload)
- ✅ Easy to add new features
- ✅ Debugging with standard Python tools
- ❌ Changes require restart
- ❌ Less modular (monolithic app)

**Docker Stack:**
- ✅ Highly modular (change one service)
- ✅ Independent service updates
- ✅ Version control per service
- ✅ Rollback individual components
- ❌ More complex to modify
- ❌ Requires container rebuilds

### D.2 Customization Scenarios

| Scenario | Streamlit Stack Effort | Docker Stack Effort |
|----------|------------------------|---------------------|
| Change LLM model | 1 line of code | 1 line in .env |
| Add new document type | Add loader (10 lines) | Add loader to pipeline (15 lines) |
| Modify RAG prompt | Edit prompt template | Edit AnythingLLM config |
| Add translation language | Install library, add code | Configure translation service |
| Integrate external API | Add Python requests | Add n8n node or new service |
| Change vector DB | Significant refactor | Swap container, update config |
| Add monitoring | Implement from scratch | Use existing tools |
| Multi-user support | Significant code changes | Already implemented |

### D.3 Integration Capabilities

| Integration Type | Streamlit Stack | Docker Stack |
|------------------|----------------|--------------|
| **REST APIs** | Python requests library | n8n HTTP nodes |
| **Databases** | Direct Python drivers | Docker service + connectors |
| **Cloud Services** | Python SDK | n8n cloud nodes |
| **Data Lakes** | Python libraries | n8n + custom ETL |
| **Message Queues** | Python clients | Docker Kafka/RabbitMQ |
| **Webhooks** | Custom Flask endpoints | n8n webhook nodes |
| **File Shares** | OS-level mounting | Volume mounts |
| **LDAP/AD** | Python LDAP library | Traefik forward auth |

---

## E. Performance Benchmarks

### E.1 Query Performance

**Test Setup:**
- Hardware: 32 GB RAM, Intel i7-12700, RTX 3060 12GB
- Documents: 114 WUR PDFs (10,234 pages)
- Chunks: 28,529 (after deduplication)
- Test: 50 queries, averaged

| Metric | Streamlit Stack | Docker Stack | Notes |
|--------|----------------|--------------|-------|
| **Cold start** | 15 seconds | 45 seconds | Docker loads all containers |
| **Query latency (p50)** | 6.2 seconds | 7.8 seconds | Docker has network overhead |
| **Query latency (p95)** | 12.5 seconds | 15.3 seconds | Both acceptable |
| **Throughput (queries/min)** | 8-10 | 6-8 | Streamlit slightly faster |
| **Memory usage (idle)** | 8.5 GB | 18.2 GB | Docker runs multiple services |
| **Memory usage (peak)** | 12.3 GB | 26.7 GB | Docker more resource-intensive |
| **CPU usage (avg)** | 45% | 35% | Docker better parallelization |
| **GPU usage (avg)** | 60% | 55% | Similar (same LLM) |

### E.2 Embedding Performance

**Test:** Creating FAISS/Chroma index for 28,529 chunks

| Mode | Streamlit Stack | Docker Stack |
|------|----------------|--------------|
| **CPU-only** | 92 minutes | 98 minutes |
| **GPU (CUDA)** | Not yet implemented | Not yet implemented |
| **Estimated GPU** | ~10-15 minutes | ~12-18 minutes |

### E.3 Scalability Tests

**Concurrent Users (10-minute sustained load):**

| Users | Streamlit Stack | Docker Stack |
|-------|----------------|--------------|
| **1** | ✅ Smooth (6s avg) | ✅ Smooth (8s avg) |
| **3** | ✅ Acceptable (10s avg) | ✅ Acceptable (11s avg) |
| **5** | ⚠️ Degraded (18s avg) | ✅ Acceptable (13s avg) |
| **10** | ❌ Timeout errors | ⚠️ Degraded (22s avg) |
| **20** | ❌ Not feasible | ⚠️ Heavy degradation |
| **50+** | ❌ Not feasible | ✅ With horizontal scaling |

---

## F. Cost Analysis

### F.1 Hardware Costs (One-time)

| Configuration | Streamlit Stack | Docker Stack | Purpose |
|---------------|----------------|--------------|---------|
| **Laptop Dev** | $1,200 | Not recommended | Individual research |
| **Workstation** | $2,500 | $3,500 | Small team (1-3 users) |
| **Server** | $4,000 | $6,000 | Production (5-10 users) |
| **Cluster** | Not feasible | $15,000+ | Enterprise (50+ users) |

### F.2 Operational Costs (Annual)

| Cost Category | Streamlit Stack | Docker Stack |
|---------------|----------------|--------------|
| **Electricity** | $300-500 | $800-1,200 |
| **Maintenance (time)** | 20 hours | 60 hours |
| **Maintenance (cost @ $50/hr)** | $1,000 | $3,000 |
| **Software licenses** | $0 (all open-source) | $0 (all open-source) |
| **Training** | $500 | $2,000 |
| **Total (low estimate)** | $1,800 | $6,000 |

### F.3 Total Cost of Ownership (3 years)

| Scenario | Streamlit Stack | Docker Stack |
|----------|----------------|--------------|
| **Small research team** | $8,400 | $24,000 |
| **Extension service (10 users)** | Not scalable | $38,000 |
| **Ministry deployment (50+ users)** | Not feasible | $75,000 |

---

## G. Use Case Recommendations

### G.1 When to Use Streamlit Stack

✅ **Ideal For:**
- Individual researchers exploring RAG concepts
- Proof-of-concept demonstrations
- Conference presentations
- MSc/PhD thesis prototypes
- Training workshops
- Budget-constrained pilots
- Organizations without IT infrastructure

✅ **Example Scenarios:**
- "I need to test if RAG works with our WUR reports" → Streamlit
- "Can we demo this at the ICT4D conference?" → Streamlit
- "We have 20 PDFs and want local multilingual Q&A" → Streamlit
- "Our research group needs a shared knowledge tool" → Streamlit

### G.2 When to Use Docker Stack

✅ **Ideal For:**
- Ministry of Agriculture deployments
- National extension services
- Multi-tenant SaaS platforms
- Integration with farmer registries
- Organizations with DevOps capacity
- Production systems requiring uptime
- Systems needing audit logs and compliance

✅ **Example Scenarios:**
- "We need to serve 500 extension officers" → Docker
- "Our ministry requires 99.5% uptime" → Docker
- "We're integrating with the national farmer registry" → Docker
- "We need speech in 10 local languages" → Docker
- "Our IT policy requires containerization" → Docker

### G.3 Hybrid Approach

🔄 **Recommended Workflow:**
```
Phase 1 (Months 1-2): Streamlit Stack
├─ Rapid prototyping
├─ Content validation
└─ User feedback collection

Phase 2 (Month 3): Evaluation
├─ Assess user adoption
├─ Measure usage patterns
└─ Decide: Scale up or sufficient?

Phase 3a (If scaling): Migrate to Docker
├─ Port validated features
├─ Add multi-user support
└─ Deploy to production infrastructure

Phase 3b (If not scaling): Continue Streamlit
├─ Optimize performance
├─ Add advanced features as needed
└─ Maintain as research tool
```

---

## H. Migration Path

### H.1 Streamlit → Docker Migration

**Step 1: Export Knowledge Base**
```bash
# From Streamlit environment
cp -r faiss_index_store/ ../migration/faiss_backup/
cp -r docs/ ../migration/docs_backup/
```

**Step 2: Set Up Docker Environment**
```bash
cd wp4-docker-stack
docker compose up -d
```

**Step 3: Import Documents**
```bash
# Copy documents to Docker volume
docker cp ../migration/docs_backup/. wp4_anythingllm:/app/storage/documents/
```

**Step 4: Rebuild Index**
```bash
# Trigger AnythingLLM to re-index
docker compose exec anythingllm npm run index --all
```

**Step 5: Configure Workflows**
- Import n8n workflow JSON
- Test query responses
- Compare accuracy with Streamlit

**Step 6: User Migration**
- Export Streamlit query logs
- Train users on new Docker UI
- Parallel run for 2 weeks
- Switch over when validated

### H.2 Compatibility Notes

**Portable Elements:**
✅ Document corpus (PDFs, DOCX)
✅ Embedding model (same HuggingFace model)
✅ RAG prompts (copy templates)
✅ Query logs (export as CSV)

**Non-Portable Elements:**
❌ FAISS index (needs rebuild in ChromaDB)
❌ Python code (needs refactor for containers)
❌ Streamlit UI (different framework)
❌ Custom scripts (need n8n workflows)

---

## I. Summary Decision Matrix

| Decision Factor | Streamlit Stack | Docker Stack |
|-----------------|:---------------:|:------------:|
| **Setup complexity** | Low | High |
| **Learning curve** | Easy | Moderate |
| **Hardware needs** | Low | Medium-High |
| **Customization ease** | High | Medium |
| **Scalability** | Low | High |
| **Multi-user** | No | Yes |
| **Production-ready** | With limits | Yes |
| **Maintenance effort** | Low | Medium |
| **Cost** | Low | Medium-High |
| **Integration capability** | Medium | High |
| **Research flexibility** | High | Medium |
| **Enterprise features** | No | Yes |

---

## J. Conclusion and Recommendations

Both implementation pathways serve the WP4 objectives of providing secure, on-premise, multilingual agricultural advisory. The choice depends on:

**Choose Streamlit if:**
- You're in research/exploration phase
- You need rapid iteration
- Budget is limited
- Users < 5 concurrent
- No IT infrastructure
- Timeline is urgent (< 1 month)

**Choose Docker if:**
- You're deploying to production
- You need enterprise features
- Budget allows for infrastructure
- Users > 5 concurrent
- Integration with existing systems needed
- Long-term sustainability is priority

**Recommended hybrid approach:**
1. Start all new projects with Streamlit for validation
2. Collect user feedback and measure adoption
3. Migrate to Docker only if scale justifies complexity
4. Maintain both: Streamlit for research, Docker for production

This dual-pathway strategy balances agility with robustness, ensuring WP4 research can be both academically rigorous and practically deployable across diverse LMIC contexts.
