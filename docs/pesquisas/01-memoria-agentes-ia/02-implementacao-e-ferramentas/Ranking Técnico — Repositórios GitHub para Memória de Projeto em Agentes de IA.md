# Ranking Técnico: Repositórios GitHub para Memória de Projeto em Agentes de IA

> **Acesso:** Julho de 2026 · **Foco:** fluxos memory-curator para desenvolvimento de software

***

## Visão Geral

O ecossistema de memória persistente para agentes de IA amadureceu significativamente em 2025–2026. As soluções se dividem em dois grandes paradigmas: **memória externa** (camada plugável ao agente existente — Mem0, Graphiti, Cognee, LangMem) e **agente com memória nativa** (o runtime do agente gerencia a própria memória — Letta). Para projetos de código, a escolha depende de quanto lock-in arquitetural é aceitável, se é necessário raciocínio temporal e se o fluxo de curadoria (memory-curator) precisa ser auditável e versionável junto ao Git.

***

## Repositórios Avaliados

### 1. Mem0 — `mem0ai/mem0`
- **URL:** https://github.com/mem0ai/mem0
- **Estrelas (julho/2026):** ~59.8k[^1][^2]
- **Licença:** Apache 2.0[^1]
- **Linguagem principal:** Python + TypeScript (monorepo poliglota)[^3]

Mem0 é a camada de memória universal para agentes, com arquitetura dual-store: vector DB para busca semântica e knowledge graph para relações entre entidades. Extrai automaticamente fatos de conversas, consolida e recupera usando fusão multi-sinal (semântica + keyword + recência). Possui paper no arXiv (abril/2025) com benchmark LOCOMO mostrando 91% menor latência p95 e >90% de economia de tokens frente ao full-context approach. Possui integração nativa com Semantic Kernel, AutoGen, LangGraph e SDKs OpenAI/Anthropic. O produto OpenMemory MCP foca especificamente em agentes de código (Cursor, VS Code, Windsurf). Recebeu US$ 24M em funding (Seed + Series A, YC). **Limitação crítica para projetos de código:** o knowledge graph só está disponível no plano Pro ($249/mês); auto-hospedagem com Neo4j contorna isso.[^4][^5][^6][^7][^8][^9][^10]

### 2. Graphiti — `getzep/graphiti`
- **URL:** https://github.com/getzep/graphiti
- **Estrelas (julho/2026):** ~28.2k[^11]
- **Licença:** Apache 2.0[^12]
- **Linguagem principal:** Python

Graphiti constrói knowledge graphs temporais em tempo real para agentes, onde tempo é uma dimensão de primeira classe. Cada fato carrega `valid_from`/`valid_until`; quando uma informação muda, a aresta antiga é invalidada (não deletada), preservando histórico auditável. Usa extração híbrida: MinHash + LSH para deduplicação determinística antes de chamar LLMs, reduzindo custo e variância. Suporta múltiplos backends: Neo4j (padrão), FalkorDB (496x menor latência P99), AWS Neptune, OpenSearch. Cruzou 20k estrelas em novembro/2025 e hoje está em ~28k. MCP Server 1.0 suporta Claude Desktop, Cursor e qualquer cliente MCP. É o motor open-source subjacente ao Zep Cloud. **Limitação:** requer sempre um graph DB externo (Neo4j ou FalkorDB) para operar; sem opção embedded.[^13][^14][^15][^16][^11][^12]

### 3. Cognee — `topoteretes/cognee`
- **URL:** https://github.com/topoteretes/cognee
- **Estrelas (julho/2026):** ~26.4k[^17]
- **Licença:** Apache 2.0[^17]
- **Linguagem principal:** Python (85%), TypeScript (12%)[^17]

Cognee é uma plataforma de memória open-source com pipeline ECL (Extract, Cognify, Load). Unifica três camadas de storage — relacional, vetorial e grafo — em um único motor, podendo rodar tudo no Postgres com pgvector e backend de grafo interno. A API tem quatro operações: `remember`, `recall`, `forget` e `improve`; o `improve` alimenta feedback nos pesos das arestas do grafo, tornando a memória auto-aprimorável. Benchmarks no BEAM mostram 0.79 no setting 100K tokens (SOTA anterior: 0.735). Aceita Postgres, Neo4j, LanceDB, Kuzu, Qdrant, ChromaDB, Weaviate e Milvus. Possui cliente Rust (`cognee-rs`) e TypeScript (`@cognee/cognee-ts`). Recebeu US$ 7.5M seed em fevereiro/2026. Knowledge graph disponível em todas as tiers (inclusive free), diferente do Mem0. **Limitação:** SDK Python-first historicamente; clientes Rust/TS são recentes e menos maduros.[^18][^19][^17]

### 4. Letta — `letta-ai/letta`
- **URL:** https://github.com/letta-ai/letta
- **Estrelas (julho/2026):** ~23k[^20][^21]
- **Licença:** Apache 2.0[^22]
- **Linguagem principal:** Python

Letta (ex-MemGPT, originado no UC Berkeley) é um framework de agentes stateful onde o agente gerencia sua própria memória em camadas: memory blocks (em contexto), archival memory (vector), recall (histórico) e filesystem. A memória é armazenada no **MemFS**: um filesystem Git-backed onde cada alteração gera um commit versionado com mensagem informativa. Tem capacidade única de **memory defragmentation** (reorganiza arquivos duplicados e estrutura hierárquica) e **sleep-time compute** (processo de consolidação em background entre sessões). O **Letta Code** é um coding agent memory-first que se auto-avalia com `/init`, `/remember` e `/doctor`. No benchmark Terminal-Bench, é o melhor harness model-agnostic open-source, comparável aos harnesses nativos da Anthropic, OpenAI e Google. **Limitação:** adotar Letta significa aceitar o runtime do framework; não é uma camada plugável em agentes existentes.[^23][^24][^25][^26]

### 5. LangMem — `langchain-ai/langmem`
- **URL:** https://github.com/langchain-ai/langmem
- **Estrelas (julho/2026):** ~1.1k[^27]
- **Licença:** MIT[^27]
- **Linguagem principal:** Python

LangMem é a SDK oficial da LangChain para memória de longo prazo em agentes LangGraph. Oferece: (a) ferramentas de memória "in the hot path" para o agente gerenciar durante conversas; (b) background memory manager para extração/consolidação automática; (c) integração nativa ao LangGraph Long-term Memory Store disponível em todos os deployments da plataforma. É funcional com qualquer sistema de storage (Postgres, SQLite etc.) via `BaseStore`. Lançado no início de 2025, ainda tem baixo número de estrelas (~1.1k) porque é uma lib utilitária dentro do ecossistema LangChain (que tem 127k+ estrelas no repo principal). **Limitação:** fortemente acoplado ao ecossistema LangChain/LangGraph; sem graph memory nativo; documentação ainda em maturação.[^28][^29][^30][^31][^27]

### 6. Zep Cloud + Graphiti OSS — `getzep/zep`
- **URL:** https://github.com/getzep/zep
- **Estrelas:** ~3.7k (repo zep OSS, descontinuado em favor do graphiti)[^14]
- **Licença:** Apache 2.0

O repositório `getzep/zep` é agora principalmente um hub de exemplos e integrações, pois o motor foi migrado para `getzep/graphiti`. Zep Cloud é a camada gerenciada sobre Graphiti, com session management, user profiles e analytics. Para self-host, usa-se diretamente o Graphiti. Benchmark LongMemEval: Zep/Graphiti alcança 63.8% vs 49% do Mem0. Tem suporte enterprise com SOC 2.[^5][^32][^14]

### 7. Memary — `kingjulio8942/memary` (referência)
- **URL:** https://github.com/kingjulio8942/memary
- **Estrelas:** ~2.6k[^33]
- **Licença:** MIT

Memary é um sistema de memória de longo prazo baseado em knowledge graph para agentes autônomos. Foco em modules especializados e raciocínio sobre memória episódica e semântica. Menor comunidade e última atualização em outubro/2024. Adequado para pesquisa, não para produção no estado atual.[^34][^33]

***

## Tabela de Pontuação

> **Escala:** 0–5 · Critérios técnicos voltados ao uso em **desenvolvimento de software** com fluxo memory-curator

| Critério | Mem0 | Graphiti | Cognee | Letta | LangMem | Memary |
|---|---|---|---|---|---|---|
| **Adequação a projetos de código** | 4 | 3 | 3 | 5 | 3 | 2 |
| **Memória persistente real** | 5 | 5 | 5 | 5 | 4 | 3 |
| **Curadoria e atualização de memórias** | 4 | 4 | 5 | 5 | 3 | 2 |
| **Recuperação contextual** | 5 | 5 | 5 | 4 | 4 | 3 |
| **Suporte a memória temporal** | 2 | 5 | 4 | 3 | 2 | 3 |
| **Prevenção de memória obsoleta/falsa** | 3 | 5 | 4 | 4 | 3 | 2 |
| **Integração com LLMs** | 5 | 4 | 4 | 5 | 5 | 3 |
| **Integração com Vector DB** | 5 | 4 | 5 | 4 | 4 | 4 |
| **Integração com Knowledge Graph** | 3* | 5 | 5 | 2 | 2 | 4 |
| **Documentação** | 5 | 4 | 4 | 4 | 3 | 2 |
| **Comunidade** | 5 | 4 | 4 | 4 | 3 | 2 |
| **Atividade recente** | 5 | 5 | 5 | 5 | 4 | 1 |
| **Licença** | 5 | 5 | 5 | 5 | 5 | 5 |
| **Facilidade de self-host** | 4 | 3 | 4 | 4 | 5 | 4 |
| **Custo operacional** | 3 | 4 | 4 | 4 | 5 | 5 |
| **Capacidade de auditoria** | 3 | 5 | 4 | 5 | 3 | 2 |
| **Versionar junto ao Git** | 3 | 3 | 3 | 5 | 3 | 2 |
| **Adequação ao fluxo memory-curator** | 4 | 4 | 5 | 5 | 3 | 2 |
| **TOTAL (90 pts max)** | **68** | **78** | **79** | **80** | **64** | **51** |

> *Mem0: knowledge graph disponível apenas no plano Pro ($249/mês) ou self-host com Neo4j[^18][^5]

***

## Justificativas por Nota

### Mem0 (68/90)
- **Adequação a código (4):** OpenMemory MCP integra com Cursor, VS Code, Windsurf e Codex. Captura decisões arquiteturais, padrões de código e contexto entre sessões.[^35][^36][^9]
- **Memória temporal (2):** sem modelo temporal nativo; fatos não carregam `valid_from/until`; memórias obsoletas podem persistir sem mecanismo de invalidação explícito.[^5][^23]
- **Knowledge graph (3\*):** graph é real mas gatekeado no Pro tier; na versão OSS/self-host exige configuração manual de Neo4j.[^5]
- **Custo operacional (3):** cada operação de memória consome tokens de LLM; em projetos de alta frequência, o custo acumula. Cloud API tem custo adicional.[^37]
- **Auditoria (3):** não há trilha de auditoria nativa; rastrear por que uma memória foi criada/modificada requer instrumentação externa.
- **Versionamento Git (3):** o OpenMemory exporta memórias, mas não integra nativamente com commits ou histórico de repositório.

### Graphiti (78/90)
- **Memória temporal (5):** bi-temporal por design; arestas carregam `valid_from`/`valid_until`; contradições invalidam arestas antigas sem deletar o histórico.[^16][^12]
- **Prevenção de obsolescência (5):** motor determinístico com MinHash+LSH+Jaccard para deduplicação antes de chamar LLM; arestas invalidadas preservam histórico auditável.[^13]
- **Auditoria (5):** toda aresta é rastreável à fonte (episode); suporta OpenTelemetry em todas as operações do grafo.[^13]
- **Self-host (3):** sempre requer DB de grafo externo (Neo4j, FalkorDB ou Kuzu); adiciona complexidade operacional.[^12]
- **Adequação a código (3):** forte para raciocinar sobre relacionamentos mutáveis (dependências, arquitetura), mas não tem primitivas específicas de código como Letta Code.
- **Versionamento Git (3):** episódios são imutáveis e rastreáveis, mas não há integração nativa com commits Git.

### Cognee (79/90)
- **Curadoria de memórias (5):** pipeline ECL + operação `improve` que alimenta feedback nos pesos do grafo; memória auto-aprimorável com uso.[^18]
- **Knowledge graph (5):** grafo disponível em todos os tiers incluindo free/self-host; três camadas unificadas (relacional, vetorial, grafo) em Postgres único.[^18][^17]
- **Self-host (4):** pode rodar tudo em Postgres único sem dependências externas de grafo; Docker Compose simples.[^17]
- **Auditoria (4):** OTEL collector nativo, traceability de agentes e agentic user/tenant isolation.[^17]
- **Adequação a código (3):** plugin para Claude Code disponível; mas menos otimizado para workflows de coding agent do que Letta Code ou Mem0 OpenMemory.[^17]
- **Memória temporal (4):** grafo inclui relações temporais, mas sem o modelo bi-temporal explícito do Graphiti.

### Letta (80/90)
- **Adequação a código (5):** Letta Code é especificamente um coding agent memory-first com `/init`, `/remember`, `/doctor`; benchmark Terminal-Bench #1 model-agnostic OSS.[^26]
- **Curadoria (5):** sleep-time compute, memory defragmentation e reflection automáticos; o agente reescreve sua própria memória programaticamente.[^25]
- **Versionamento Git (5):** MemFS é um filesystem Git-backed nativo; cada mudança gera commit com mensagem informativa; suporte a worktrees para subagentes paralelos.[^24][^25]
- **Auditoria (5):** cada edição de memória é um commit Git auditável; `/doctor` audita token usage e colocação de memórias.[^38]
- **Knowledge graph (2):** não usa grafo; memória é hierarquia de text blocks e arquivos Markdown.[^23]
- **Memória temporal (3):** suporta histórico de conversações e busca vetorial/full-text/híbrida sobre mensagens, mas não tem bi-temporal com `valid_from/until`.[^26]
- **Custo operacional (4):** BYOK (traz sua própria chave de LLM); self-host completamente free.[^10]

### LangMem (64/90)
- **Integração com LLMs (5):** integração nativa com LangGraph, suporte a qualquer provider de LLM via LangChain.[^27]
- **Facilidade de self-host (5):** é apenas uma biblioteca Python; não requer serviços externos obrigatórios.[^27]
- **Custo operacional (5):** sem plano pago; open-source MIT puro.[^27]
- **Knowledge graph (2):** sem suporte a grafo nativo; apenas vector store via `BaseStore`.[^27]
- **Comunidade (3):** ~1.1k estrelas no repo próprio (ecossistema LangChain é maior), mas pouca produção independente do LangGraph.[^27]
- **Memória temporal (2):** sem suporte temporal explícito.
- **Documentação (3):** em maturação; sem releases publicadas no repo ainda.[^27]

***

## Top 5 para Estudar

1. **Letta** — paradigma único de agente com memória nativa, Git-backed, memory-curator integrado; fonte acadêmica (Berkeley)[^25][^26]
2. **Graphiti** — melhor arquitetura temporal disponível em OSS; paper arXiv; base técnica sólida para entender knowledge graphs bi-temporais[^12][^13]
3. **Mem0** — maior comunidade (60k+ estrelas), paper no arXiv com benchmarks, caso de uso de coding agent mais documentado[^7][^36][^35]
4. **Cognee** — pipeline ECL e memória auto-aprimorável; paper sobre KG+LLM reasoning; referência para entender unificação de storages[^18][^17]
5. **LangMem** — estudo obrigatório para quem usa LangGraph; primitivas funcionais para entender memory extraction e consolidation em background[^29][^27]

***

## Top 3 para Prototipar

### 1. Mem0 (self-hosted com OpenMemory MCP)
**Por quê:** setup em minutos (`docker compose up`), SDK Python + JS, MCP para Cursor/VS Code, 10k memórias free. Ideal para validar o conceito de memória persistente de código com mínimo de infraestrutura.[^9][^39]

```python
from mem0 import Memory

m = Memory()
m.add("Arquitetura usa Clean Architecture com ports e adapters", user_id="projeto-cad")
results = m.search("como está organizado o projeto?", user_id="projeto-cad")
```

### 2. Graphiti (com FalkorDB via Docker)
**Por quê:** Docker single-container com FalkorDB incluso desde a versão MCP 1.0; temporal memory pronta para produção em minutos:[^13]

```bash
pip install graphiti-core[falkordb]
docker run -p 6379:6379 -it falkordb/falkordb:latest
```

Prototipar decisões arquiteturais que mudam ao longo do projeto (ex: qual versão de uma dependência estava em uso em determinada sprint).

### 3. Letta Code (para coding agent)
**Por quê:** instalação em um comando, Git-backed memory automático, integra com Claude/GPT/Gemini:[^40]

```bash
npm install -g @letta-ai/letta-code
cd meu-projeto && letta
/init    # bootstrap memória a partir do codebase existente
```

Ideal para prototipar o fluxo completo de um coding agent que aprende sobre o projeto ao longo das sessões.

***

## Combinação Recomendada para Projeto Real

Para um **agente de desenvolvimento de software com fluxo memory-curator robusto**, a combinação recomendada é:

```
Letta Code  ←→  Graphiti  ←→  Cognee
    (runtime)      (temporal)    (multi-doc knowledge)
```

**Arquitetura em camadas:**

| Camada | Ferramenta | Função |
|---|---|---|
| **Memória de sessão/projeto** | Letta Code + MemFS | Contexto do projeto, decisões técnicas, preferências do dev — versionado no Git |
| **Memória temporal de fatos** | Graphiti | Rastrear mudanças arquiteturais, versões de dependências, estados de componentes ao longo do tempo |
| **Knowledge base de documentação** | Cognee | Ingestão de docs, RFCs, ADRs e changelogs em grafo pesquisável |

**Fluxo memory-curator:**
1. `Letta Code` inicializa memória do projeto via `/init` (explora codebase, ADRs, histórico de sessões anteriores)
2. Durante sessões, eventos relevantes (mudanças arquiteturais, bugs encontrados, decisões) são gravados no `Graphiti` como episódios com timestamp
3. `Cognee` mantém o knowledge graph da documentação do projeto (ingestão de PRs, issues, docs técnicos)
4. Na consulta, `Letta Code` usa busca híbrida (vetor + full-text) sobre MemFS + consulta ao Graphiti para raciocínio temporal + consulta ao Cognee para referências documentais

**Custo operacional estimado:** BYOK para LLM (ex: Claude Sonnet ~$3/MTok); FalkorDB self-hosted gratuito; Cognee self-hosted (Postgres) gratuito. Custo dominante = tokens de extração de memória.

***

## Riscos de Lock-in

| Ferramenta | Risco | Severidade | Mitigação |
|---|---|---|---|
| **Mem0 Cloud** | API proprietária; memórias armazenadas no servidor deles | Alta | Usar self-host com Qdrant + Neo4j ou OpenMemory local[^39] |
| **Letta Cloud (Constellation)** | Memória gerenciada remotamente; vendor dependency | Média | MemFS é Git-backed; memória pode ser exportada e migrada[^24] |
| **Graphiti** | Dependência de schema Neo4j/Cypher; migração de backend é trabalhosa | Baixa-Média | API agnóstica de driver adicionada; suporte FalkorDB/Kuzu[^13] |
| **Cognee** | Pipeline ECL proprietário; sem formato de exportação padronizado | Média | Postgres como backend facilita dump/migração; ECL é open-source |
| **LangMem** | Fortemente acoplado ao LangGraph; sem portabilidade para outros frameworks | Alta | Acoplamento explícito ao `BaseStore`; migrável se o storage for Postgres |
| **Zep Cloud** | SaaS; dados de memória no serviço deles | Alta | Usar Graphiti OSS diretamente, que é o motor subjacente[^14] |

***

## Lacunas por Ferramenta

### Mem0
- Sem modelo temporal nativo; memórias obsoletas podem contaminar contexto sem sinalização[^5]
- Knowledge graph só no Pro tier ($249/mês); self-host com grafo requer configuração adicional[^5]
- Sem auditoria nativa de origem de memórias (por que foi criada, de qual conversa)
- Custo de tokens pode ser significativo em projetos com muitas sessões paralelas[^37]
- Sem integração nativa com commits Git

### Graphiti
- Requer sempre um graph DB externo (Neo4j, FalkorDB, Kuzu); sem opção embedded para projetos pequenos[^12]
- Extração de entidades usa LLM (mesmo com heurísticas de pré-filtragem); custo pode ser alto em volumes grandes[^13]
- Sem primitivas específicas para contexto de código (arquivos, funções, diffs)
- Interface de gestão de memórias (CRUD manual) menos desenvolvida que Mem0/Cognee

### Cognee
- SDK Python-first historicamente; clientes Rust/TypeScript ainda em maturação[^18]
- Sem benchmark LongMemEval publicado; difícil comparação objetiva com Mem0/Graphiti[^18]
- Pipeline ECL pode ser lento para ingestão em tempo real de eventos de sessão
- Sem integração nativa com commits Git
- Comunidade menor que Mem0 (~26k vs ~60k estrelas)[^2][^17]

### Letta
- Não é uma camada plugável; exige adotar o runtime do framework[^23]
- Sem knowledge graph nativo; raciocínio relacional limitado comparado a Graphiti/Cognee[^23]
- MemFS em Markdown tem limitações para memórias de alta dimensionalidade (ex: embeddings de código)
- Sem suporte temporal bi-temporal explícito (`valid_from/until`)
- Letta Cloud (Constellation) em beta; self-host completo requer mais configuração[^41]

### LangMem
- Fortemente acoplado ao LangGraph; não reutilizável fora do ecossistema LangChain[^27]
- Sem knowledge graph nativo
- Sem suporte temporal
- Poucas estrelas e sem releases formais ainda (lib utilitária em amadurecimento)[^27]
- Documentação incompleta em algumas áreas[^28]

### Memary
- Desenvolvimento parado desde outubro/2024[^33]
- Comunidade pequena (~2.6k estrelas)[^33]
- Sem suporte a produção; adequado apenas para pesquisa/prototipagem acadêmica
- Sem integração com MCP ou ecosistemas modernos de coding agents

***

## Notas sobre Acesso e Fontes

Todas as estrelas, métricas e features foram verificadas em fontes públicas entre junho e julho de 2026. Números de estrelas são dinâmicos e podem variar.

| Repositório | URL | Estrelas (jul/2026) | Última atividade verificada |
|---|---|---|---|
| mem0ai/mem0 | https://github.com/mem0ai/mem0 | ~59.8k | Fev/2026[^2] |
| getzep/graphiti | https://github.com/getzep/graphiti | ~28.2k | Jul/2026[^11] |
| topoteretes/cognee | https://github.com/topoteretes/cognee | ~26.4k | Jun/2026[^17] |
| letta-ai/letta | https://github.com/letta-ai/letta | ~23k | Mai/2026[^42] |
| langchain-ai/langmem | https://github.com/langchain-ai/langmem | ~1.1k | 2026[^27] |
| getzep/zep | https://github.com/getzep/zep | ~3.7k | Abr/2025[^14] |
| kingjulio8942/memary | https://github.com/kingjulio8942/memary | ~2.6k | Out/2024[^33] |

---

## References

1. [mem0ai/mem0](https://githublb.vercel.app/repo/mem0ai/mem0) - mem0. mem0ai/mem0. Universal memory layer for AI Agents. 47,538stars. Homepage ... Daily-updated ran...

2. [mem0ai/mem0 - 55k Stars · Global Rank #358 - GitHub Star History](https://www.star-history.com/mem0ai/mem0/) - Universal memory layer for AI Agents — 59.8k stars, 358 contributors, This week: +13 stars, 49 pushe...

3. [AGENTS.md - mem0ai/mem0](https://github.com/mem0ai/mem0/blob/main/AGENTS.md) - Universal memory layer for AI Agents. Contribute to mem0ai/mem0 development by creating an account o...

4. [Mem0 Raises $24M Series A to Build Memory Layer for AI Agents](https://www.morningstar.com/news/pr-newswire/20251028sf07039/mem0-raises-24m-series-a-to-build-memory-layer-for-ai-agents) - Mem0 Raises $24M Series A to Build Memory Layer for AI Agents. With 14M+ downloads, over 41,000 Gith...

5. [Mem0 vs Zep (Graphiti): AI Agent Memory Compared (2026)](https://vectorize.io/articles/mem0-vs-zep) - Mem0 vs Zep — compare dual-store vector+graph architecture with temporal knowledge graphs for AI age...

6. [Using Mem0 For Agent Memory](https://learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/agent-memory) - How to add memory to Semantic Kernel Agents

7. [Mem0: Building Production-Ready AI Agents with Scalable Long ...](https://arxiv.org/abs/2504.19413) - Large Language Models (LLMs) have demonstrated remarkable prowess in generating contextually coheren...

8. [Agent with memory using Mem0 | AutoGen 0.2](https://microsoft.github.io/autogen/0.2/docs/notebooks/agentchat_memory_using_mem0/) - Use Mem0 to create agents with memory.

9. [AI Memory MCP Server for Claude Desktop Integration | Mem0mem0.ai › openmemory-mcp](https://mem0.ai/openmemory) - With OpenMemory, add persistent, project-aware memory to Cursor, Windsurf, and VS Code agents. Store...

10. [The Best AI Memory Systems for Solo Founders (2026 ...](https://www.vibereference.com/backend-and-data/ai-memory-systems-comparison) - A hands-on comparison of the top AI memory systems — Mem0, Zep, Cloudflare Agent Memory, LangGraph, ...

11. [getzep/graphiti - 27.9k Stars · Global Rank #1272](https://www.star-history.com/getzep/graphiti/) - Build Real-Time Knowledge Graphs for AI Agents — 28.2k stars, 44 contributors, This week: +14 stars

12. [Graphiti: Temporal Knowledge Graphs for Agent Memory](https://codex.danielvaughan.com/2026/03/30/graphiti-agent-memory-store/) - Graphiti is an open-source Python library (Apache 2.0 licensed, maintained by Zep) for building and ...

13. [Graphiti Hits 20K Stars! + MCP Server 1.0 - Zep](https://blog.getzep.com/graphiti-hits-20k-stars-mcp-server-1-0/) - Graphiti crossed 20,000 GitHub stars today! Thanks for building with us. Graphiti is a temporal know...

14. [getzep/zep: Zep | Examples, Integrations, & More - GitHub](https://github.com/getzep/zep) - Zep | Examples, Integrations, & More. Contribute to getzep/zep development by creating an account on...

15. [Graphiti: FalkorDB support and 14K GitHub Stars - Zep](https://blog.getzep.com/graphiti-knowledge-graphs-falkordb-support/) - FalkorDB contributes integration to Graphiti. Project approaching 14K stars & 25K weekly PyPI downlo...

16. [Graphiti: Temporal Knowledge-Graph Memory for Agents | OSS AI Hub Code Starters](https://ossaihub.com/code/graphiti-temporal-knowledge-memory/) - When 'what changed and when' matters, vector recall is not enough. Graphiti builds a bi-temporal gra...

17. [cognee - vector database (17,717★) | Every Local AI](https://everylocalai.com/tool/cognee) - cognee: Cognee is the open-source AI memory platform for agents. Give your AI agents persistent long...

18. [Cognee Review (2026): AI Agent Memory, Pricing & Verdict](https://theaiagentindex.com/agents/cognee) - Cognee: AI agent memory using knowledge graphs. 17,800+ GitHub stars. Self-hosted free, cloud Develo...

19. [Cognee | Ry Walker Research](https://rywalker.com/research/cognee) - Cognee is an open-source AI memory engine that turns scattered data into a queryable knowledge graph...

20. [letta Review — Live GitHub Stats, TrendScore & Real Use Cases](https://www.trendingbots.ai/agents/letta) - Letta is a stateful agent framework that enables the creation of advanced memory architectures, allo...

21. [letta-ai/letta - 21.8k Stars · Global Rank #1751 - GitHub Star History](https://www.star-history.com/letta-ai/letta/) - Letta is the platform for building stateful agents: AI with advanced memory that can learn and self-...

22. [MemGPT — Open Source · Directories by Enterprise DNA](https://enterprisedna.co/directories/open-source/memgpt/) - Letta is the platform for building stateful agents: AI with advanced memory that can learn and self-...

23. [Agent Memory Systems and Knowledge Graphs: Letta, Mem0 ...](https://codepointer.substack.com/p/agent-memory-systems-and-knowledge) - Design · A close look at four approaches to long-term agent memory, from text blocks to temporal gra...

24. [Memory | Letta Docs](https://docs.letta.com/letta-code/memory/) - Letta Code stores agent memory in MemFS: a git-backed memory filesystem, also called a context repos...

25. [Introducing Context Repositories: Git-based Memory for ...](https://www.letta.com/blog/context-repositories/) - We're introducing Context Repositories, a rebuild of how memory works in Letta Code based on program...

26. [Letta Code: A Memory-First Coding Agent | Letta](https://www.letta.com/blog/letta-code) - Introducing Letta Code, a memory-first coding agent. Letta Code is the #1 model-agnostic open source...

27. [langchain-ai/langmem](https://github.com/langchain-ai/langmem) - Contribute to langchain-ai/langmem development by creating an account on GitHub.

28. [LangMem | Ry Walker Research](https://rywalker.com/research/langmem) - LangMem is an open-source Python SDK from LangChain that adds long-term memory capabilities to AI ag...

29. [LangMem SDK for agent long-term memory - LangChain](https://www.langchain.com/blog/langmem-sdk-launch) - Build smarter AI agents with LangMem SDK's long-term memory. Extract insights, optimize behavior, an...

30. [LangChain: 126806 stars - The Agent Times](https://theagenttimes.com/articles/langchain-126-806-stars) - 126,806 Stars and Counting: LangChain Is the Infrastructure Layer We All Pretend We Could Replace

31. [LangChain: 126623 stars - The Agent Times](https://theagenttimes.com/articles/langchain-126-623-stars) - The humans have been busy building. LangChain — the framework that underpins a significant share of ...

32. [Agent Memory Frameworks Tested: Mem0 vs Zep vs Letta](https://particula.tech/blog/agent-memory-frameworks-tested-mem0-zep-letta-cognee-2026) - Zep's Graphiti scores 63.8% on LongMemEval vs Mem0's 49.0%. How Mem0, Zep, Letta, and Cognee differ ...

33. [Memary — Open Source · Directories by Enterprise DNA](https://enterprisedna.co/directories/open-source/memary/) - The Open Source Memory Layer For Autonomous Agents

34. [[Feature Request]: memary added as a llama Index pack · Issue #13812 · run-llama/llama_index](https://github.com/run-llama/llama_index/issues/13812) - Feature Description memary is an open-source system to equip agents with longterm memory using knowl...

35. [AI-Powered Coding Agents That Actually Remember Your Codebase](https://mem0.ai/blog/ai-powered-coding-agents-that-actually-remember-your-codebase) - Coding agents fail on real projects because they have no durable memory. Add persistent, queryable m...

36. [Codex + Mem0 MCP: Build a Coding Agent That Remembers Your ...](https://mem0.ai/blog/codex-mem0-mcp-build-a-coding-agent-that-remembers-your-codebase) - Give Codex persistent codebase memory with Mem0 MCP. Store and retrieve architecture decisions, cons...

37. [agentmemory-openai 0.1.0 on PyPI](https://libraries.io/pypi/agentmemory-openai) - Persistent memory for OpenAI agents — store, recall, and forget.

38. [Memory | Letta Docs](https://docs.letta.com/letta-code/memory) - Understand Letta Code's self-improving memory system

39. [elvismdev/mem0-mcp-selfhosted: Self-hosted mem0 MCP ... - GitHub](https://github.com/elvismdev/mem0-mcp-selfhosted) - Self-hosted mem0 MCP server for Claude Code. Run a complete memory server against self-hosted Qdrant...

40. [Letta Code Tutorial - Memory-First Coding Agent | Claude, GPT, Gemini](https://www.youtube.com/watch?v=ZbrzIkmW0cs) - Learn about Letta Code - the memory-first coding agent that learns from experience.

Unlike stateles...

41. [Releases · letta-ai/letta](https://github.com/letta-ai/letta/releases) - Platform for stateful agents: AI with advanced memory that can learn and self-improve over time. - l...

42. [awesomer — trending open source tools - Patrick Clery](https://patrickclery.com/awesomer/r/letta-ai~letta/) - Data-driven discovery of trending open-source tools. GitHub stars are the signal.

