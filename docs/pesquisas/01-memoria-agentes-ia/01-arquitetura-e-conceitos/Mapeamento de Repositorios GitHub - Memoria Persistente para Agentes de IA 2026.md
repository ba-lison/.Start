# Mapeamento de Repositórios GitHub: Memória Persistente para Agentes de IA (2026)

> **Verificação:** 02 de julho de 2026  
> **Critério de ordenação:** relevância para o caso de uso de "memória de projeto para agente de programação que aprende com bugs resolvidos, decisões arquiteturais e implementações anteriores" — combina stars, atividade, adequação ao caso, clareza arquitetural, documentação, licença e facilidade de adoção.

***

## Sumário Executivo

O ecossistema de memória para agentes de IA amadureceu rapidamente em 2025–2026. A memória deixou de ser um problema de pesquisa e tornou-se infraestrutura de produção. Para o caso específico de **memória de projeto de coding agent** — capturar bugs resolvidos, decisões arquiteturais e padrões de implementação entre sessões — existem pelo menos cinco categorias de solução:

1. **Engines de memória focadas em coding agents** (agentmemory, Letta Code, claude-subconscious)
2. **Camadas de memória genéricas + graph** (Mem0, Cognee, Graphiti/Zep)
3. **Integrações nativas com frameworks de agentes** (LangGraph + LangMem)
4. **Plataformas de agentes stateful** (Letta/MemGPT)
5. **Engines de memória rápida** (Supermemory)

Os projetos mais relevantes para coding agents com memória de projeto são **agentmemory**, **Letta Code**, **Cognee** e **Graphiti**, seguidos de **Mem0** (mais genérico, mas amplamente adotado).

***

## Tabela de Comparação Rápida

| Repositório | Stars (jul/26) | Licença | Foco principal | Storage | Melhor para coding agent |
|---|---|---|---|---|---|
| **agentmemory** | ~17–20k | Apache-2.0 | Coding agents | SQLite + vector + graph | ⭐⭐⭐⭐⭐ |
| **Letta Code** (letta-ai/letta-code) | ~2.7k | Apache-2.0 | Coding agent com memória | MemFS git-backed | ⭐⭐⭐⭐⭐ |
| **Cognee** | ~26k | Apache-2.0 | Knowledge graph + ECL | Relacional + vector + graph | ⭐⭐⭐⭐ |
| **Graphiti** | ~28k | Apache-2.0 | Knowledge graph temporal | Neo4j / FalkorDB | ⭐⭐⭐⭐ |
| **Mem0** | ~59k | Apache-2.0 | Camada de memória universal | Vector + graph híbrido | ⭐⭐⭐ |
| **Letta / MemGPT** | ~23k | Apache-2.0 | Agentes stateful | Postgres + vector | ⭐⭐⭐ |
| **OpenMemory MCP** | (em mem0) | Apache-2.0 | MCP local-first | Qdrant local | ⭐⭐⭐ |
| **claude-subconscious** | ~1.6k | MIT | Background para Claude Code | Letta API | ⭐⭐⭐ |
| **Supermemory** | ~27k | MIT | Engine de memória rápida | Híbrido (cloud/local) | ⭐⭐ |
| **LangGraph + LangMem** | 35.9k / 1.1k | MIT | Framework de agentes | Postgres / in-memory | ⭐⭐ |
| **Zep** | ~3.7k (CE) | Apache-2.0 | Plataforma de memória enterprise | KG temporal (Graphiti) | ⭐⭐ |
| **LlamaIndex** | ~48k | MIT | Framework RAG geral | Múltiplos backends | ⭐ |

***

## Repositórios em Detalhe (Ordenados por Relevância ao Caso)

***

### 1. rohitg00/agentmemory

| Campo | Valor |
|---|---|
| **Link** | https://github.com/rohitg00/agentmemory |
| **Organização** | rohitg00 (Rohit Ghumare, independente) |
| **Descrição** | Memória persistente para coding agents baseada em benchmarks reais. Engine iii com 4 camadas de consolidação, busca híbrida BM25+vector+graph, 53 ferramentas MCP[^1] |
| **Stars** | ~17–20k (tendência forte)[^2][^3] |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0[^4] |
| **Linguagem principal** | TypeScript |
| **Último commit** | v0.9.24 — 29 mai 2026[^4] |
| **Releases** | 46 releases[^5] |
| **Maturidade** | Beta avançado — produção para usuários individuais |
| **Documentação** | Extensa: README multilíngue, ROADMAP, DESIGN, benchmarks reproduzíveis[^6] |
| **Tipo** | Serviço + biblioteca + plugins |
| **Storage** | SQLite + embeddings (BM25+vector) + knowledge graph opcional[^1] |
| **Exemplos para código** | ✅ Exemplos específicos de JWT auth, N+1 queries, rate limiting entre sessões[^7] |

**Pontos fortes:** Único projeto focado especificamente em coding agents com memória entre sessões. Instala em um comando (`npm install -g @agentmemory/agentmemory`). Captura automática via hooks (PostToolUse, SessionEnd, etc.) sem esforço manual. Suporta Claude Code, Codex, Cursor, Gemini CLI, Copilot CLI, OpenCode e 30+ outros agentes com o mesmo servidor. 4 camadas de consolidação (Working → Episodic → Semantic → Procedural) inspiradas em memória humana. Token overhead 92% menor que carregar todo contexto (~1.900 tokens/sessão vs 22k+ com CLAUDE.md completo). Busca RRF fusion triplicada (BM25 + vector + graph). Viewer em tempo real (port 3113). Import de históricos JSONL do Claude Code. Benchmarks em sandbox reproduzíveis: 95.2% R@5 no LongMemEval-S, 100% top-5 hit rate no corpus coding-agent-life-v1.[^1][^6][^7]

**Limitações:** Dependência do iii-engine (binário nativo externo, não publicado no crates.io). Setup no Windows mais complexo. Engine iii está em processo de refatoração para nova versão sandbox. Reranker reportado como no-op em versões recentes (issue #725). Embedding multilíngue para PT-BR requer configuração extra (#725 sobre modelo hardcoded inglês). Projeto de um desenvolvedor principal — bus factor relevante.[^8]

**Riscos:** Dependência do iii-engine proprietário cria lock-in. Projeto jovem (~1 ano), sem SLA, sem empresa por trás. Issues abertas com bugs de integração.[^8]

**Quando usar:** Cenário principal do relatório — agente de programação que aprende com bugs, padrões e decisões. Quando você quer zero configuração manual e captura automática via hooks. Quando já usa Claude Code, Cursor ou Codex.

**Quando evitar:** Projetos com requisitos de compliance corporativo. Quando precisa de Python SDK nativo (é TypeScript/Node). Quando o ambiente está air-gapped ou tem restrições de instalação npm global.

***

### 2. letta-ai/letta-code

| Campo | Valor |
|---|---|
| **Link** | https://github.com/letta-ai/letta-code |
| **Organização** | Letta AI (ex-MemGPT, UC Berkeley Sky Computing Lab)[^9] |
| **Descrição** | Coding agent memory-first: MemFS git-backed, sleep-time compute, skill learning, subagents concorrentes[^10][^9] |
| **Stars** | ~2.7k[^11] |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0[^11] |
| **Linguagem principal** | TypeScript (Node) |
| **Último commit** | Jun 2026 (ativo)[^12] |
| **Releases** | Frequentes — múltiplos em 2026[^12] |
| **Maturidade** | Beta/early GA — fundado em empresa real com pesquisa de IA |
| **Documentação** | docs.letta.com — completa com memória, skills, subagents[^13] |
| **Tipo** | Aplicação (coding agent) |
| **Storage** | MemFS git-backed: markdown files versionados por git em `$MEMORY_DIR`[^10][^13] |
| **Exemplos para código** | ✅ `/init` lê codebase e forma memórias iniciais; `/remember` captura explicitamente; skills .md para DB migrations, PostHog, API patterns[^9] |

**Pontos fortes:** Projeto mais academicamente fundamentado — pesquisa de sleep-time compute e continual learning do grupo MemGPT/UC Berkeley. MemFS git-backed é arquitetural: cada mudança de memória tem commit git com mensagem informativa — auditável, rollback possível, compartilhável por PR. Múltiplos subagentes podem escrever memória concorrentemente via git worktrees (resolve problema de single-threaded memory formation). `/init` faz deep research no codebase existente usando subagents paralelos. Skill learning: após resolver uma tarefa complexa, o agente pode aprender uma skill para reutilizar em tarefas similares. Importa históricos Claude Code e Codex. `#1 model-agnostic OSS harness` no TerminalBench. Funciona com qualquer modelo: Claude, GPT, Gemini, GLM, Kimi.[^10][^13][^9][^14]

**Limitações:** Stars ainda baixas (~2.7k) — adoção inicial. Setup mais complexo que agentmemory (`npm install -g @letta-ai/letta-code`). Requer conta Letta para funcionalidades cloud (Constellation). Documentação boa mas ainda crescendo.

**Riscos:** Dependência de plataforma Letta Cloud para multi-session sync em produção. Empresa jovem, embora bem fundada (pesquisa Berkeley + funding).

**Quando usar:** Quando você quer o agente de programação mais sofisticado em termos de memória, com aprendizado real e skills persistentes. Projetos de longo prazo onde o agente precisa aprender padrões do codebase ao longo de semanas/meses. Quando você usa Claude Code, Codex ou outro agente e quer adicionar uma camada de memória profunda.

**Quando evitar:** Protótipos rápidos (curva de adoção maior). Quando prefere Python. Quando não quer dependência de serviço cloud para sync.

***

### 3. topoteretes/cognee

| Campo | Valor |
|---|---|
| **Link** | https://github.com/topoteretes/cognee |
| **Organização** | Topoteretes UG (Berlin) — $7.5M seed fev 2026[^15][^16] |
| **Descrição** | Plataforma de memória open-source com pipeline ECL (Extract, Cognify, Load): ingere dados em qualquer formato e constrói knowledge graph auto-organizado[^15] |
| **Stars** | ~26k (crescimento forte: +140/semana)[^17] |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0[^17] |
| **Linguagem principal** | Python |
| **Último commit** | 29 jun 2026 (muito ativo)[^18] |
| **Releases** | v1.1.2 (mai 2026)[^19] |
| **Maturidade** | Production-ready — 70+ empresas incluindo Bayer[^15] |
| **Documentação** | docs.cognee.ai — completa, com Colab walkthrough[^18] |
| **Tipo** | Biblioteca + serviço |
| **Storage** | Relacional (SQLite/Postgres) + vector (LanceDB/Qdrant/pgvector) + graph (Kuzu/Neo4j) — unificados[^15] |
| **Exemplos para código** | ✅ Plugin oficial para Claude Code; demo com Cognee + LangGraph para coding agent[^20][^18] |

**Pontos fortes:** API elegante de 4 operações: `remember`, `recall`, `forget`, `improve`. Pipeline ECL auto-organizado — não requer estruturação manual do conhecimento. Memória auto-melhora com feedback: rates positivos/negativos pesam arestas do grafo. MCP server oficial com 14 ferramentas (Claude Desktop, Cursor, Cline, Roo Code). Plugin nativo para Claude Code. 5M runs/mês, 500x crescimento em 2025. Plugin disponível para OpenClaw. Integração com LangGraph, OpenAI Agents SDK, Claude Agent SDK, Google ADK, n8n, Neo4j. Demo explícito de usar Cognee + Claude Code hooks para real-time memory de sessões de coding.[^15][^20][^18]

**Limitações:** Python-only SDK — sem suporte TypeScript/Go nativo. Setup requer infraestrutura (Postgres/SQLite + vector DB). Sem LongMemEval benchmark publicado, dificultando comparação objetiva. Async-only API.[^15]

**Riscos:** Empresa jovem ($7.5M seed). Self-hosting adiciona complexidade operacional. Custo cloud não trivial para alta escala.

**Quando usar:** Quando você precisa de memória com semântica relacional (não apenas similaridade vetorial) para um coding agent. Quando quer armazenar não só o quê foi feito mas como entidades se relacionam (bugs → módulos → decisões → padrões). Quando usa Python e LangGraph.

**Quando evitar:** Stacks TypeScript-first. Quando precisa de setup rápido sem infraestrutura. Quando orçamento de infra é limitado para self-hosting.

***

### 4. getzep/graphiti

| Campo | Valor |
|---|---|
| **Link** | https://github.com/getzep/graphiti |
| **Organização** | Zep / getzep[^21] |
| **Descrição** | Framework para construir e consultar context graphs temporais para agentes de IA — rastreia como fatos mudam ao longo do tempo[^22] |
| **Stars** | ~28k[^23] |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0[^23] |
| **Linguagem principal** | Python |
| **Último commit** | Jun 2026 (muito ativo)[^21] |
| **Releases** | 107+ releases[^24] |
| **Maturidade** | Production-ready — usado em produção em startups e enterprises, contribuições AWS, Microsoft, FalkorDB, Neo4j[^21] |
| **Documentação** | docs.getzep.com + paper técnico[^21] |
| **Tipo** | Biblioteca/framework |
| **Storage** | Neo4j (padrão) ou FalkorDB; nós e arestas com propriedades `valid_at`/`invalid_at`[^24] |
| **Exemplos para código** | ✅ Notebook Graphiti + MCP server 1.0 com Claude Desktop, Cursor[^21] |

**Pontos fortes:** Único grafo **temporal**: cada fato tem intervalo de validade — permite raciocinar sobre mudanças históricas ("qual era a arquitetura antes da migração?"). MCP Server 1.0 com suporte multi-provider (OpenAI, Anthropic, Google, Groq, Azure). 9 tipos de entidade pré-configurados: Preference, Requirement, Procedure, Location, Event, Organization, Document, Topic, Object — otimizados para casos reais. Deduplicação determinística com MinHash+LSH reduz chamadas LLM e custo. Suporta FalkorDB (padrão) e Neo4j. OpenTelemetry integrado para debugging em produção. Funciona como engine do Zep Cloud e como standalone.[^24][^21]

**Limitações:** Requer Neo4j ou FalkorDB (infraestrutura mais pesada). API de mais baixo nível que Cognee ou Mem0 — requer mais código de integração. Sem captura automática via hooks para coding agents.

**Riscos:** Dependência de banco de grafos (Neo4j tem custo; FalkorDB é alternativa mais nova). Empresa pequena (getzep) por trás do projeto, embora bem reconhecida.

**Quando usar:** Quando o domínio tem entidades e relacionamentos complexos que mudam (arquitetura de microserviços, decisões de design ao longo do tempo). Quando você precisa responder queries como "por que escolhemos PostgreSQL em vez de MongoDB?". Quando MCP é o protocolo de integração desejado.

**Quando evitar:** Quando a infra de graph DB é proibitiva. Para casos simples de recall de conversas — complexidade desnecessária.

***

### 5. mem0ai/mem0

| Campo | Valor |
|---|---|
| **Link** | https://github.com/mem0ai/mem0 |
| **Organização** | Mem0 AI (empresa, ex-EmbedChain)[^25] |
| **Descrição** | Camada de memória universal para agentes de IA: extrai fatos relevantes, armazena em vector + graph, recupera por relevância+recência+importância[^26] |
| **Stars** | ~59k[^25] |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0[^25] |
| **Linguagem principal** | Python (+ TypeScript SDK) |
| **Último commit** | v0.1.107 — jun 2025 (OSS); cloud continua ativo[^27] |
| **Releases** | 185+ releases[^28] |
| **Maturidade** | Maduro — maior comunidade do espaço[^29] |
| **Documentação** | docs.mem0.ai — completa[^26] |
| **Tipo** | Biblioteca + serviço gerenciado (Mem0 Cloud) |
| **Storage** | Vector (Qdrant, pgvector) + key-value + graph opcional (Neo4j, v1.1+)[^28] |
| **Exemplos para código** | ✅ Exemplos genéricos; menos específico para coding agents[^26] |

**Pontos fortes:** Maior comunidade do espaço (~59k stars). API mais simples: `add()`, `search()`, `get_all()`, `history()`. Suporte Python + TypeScript. Framework-agnostic — funciona com qualquer LLM e qualquer stack. OpenMemory MCP integrado (subpasta no repo): local-first, self-hosted, sem cloud obrigatório. Graph Memory disponível com Neo4j (versão v1.1). Multi-level: User memory, Session memory, Agent memory.[^30][^28][^31]

**Limitações:** Graph memory requer Neo4j e está em versão pro paga ($249/mês) para uso completo — free tier é só vector. Última versão OSS relevante foi jun 2025; desenvolvimento mais ativo na cloud. Captura manual (`add()`) — sem hooks automáticos para coding agents. Menos adequado ao caso de uso de coding agent nativo que agentmemory ou Letta Code.[^15]

**Riscos:** Distância entre OSS e Cloud growing. Sem captura automática de hooks de coding agents.

**Quando usar:** Quando precisa de uma biblioteca de memória genérica, bem testada e com grande comunidade. Quando já tem código LangChain/LlamaIndex. Quando quer managed cloud sem operar infraestrutura.

**Quando evitar:** Quando precisa de captura automática de sessões de coding agent. Quando graph memory é essencial e não quer pagar tier pro.

***

### 6. mem0ai/mem0/openmemory (OpenMemory MCP)

| Campo | Valor |
|---|---|
| **Link** | https://github.com/mem0ai/mem0/tree/main/openmemory |
| **Organização** | Mem0 AI[^32] |
| **Descrição** | Servidor MCP local-first: memória privada e portátil para Claude, Cursor, Windsurf e qualquer cliente MCP[^30] |
| **Stars** | Incluído no repo mem0 (~59k) |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0 |
| **Linguagem principal** | Python (FastAPI) + Next.js (UI) |
| **Último commit** | Mai 2026[^32] |
| **Releases** | Como parte do mem0 |
| **Maturidade** | Beta — funcional, documentação básica[^30] |
| **Documentação** | Dentro do repo mem0 + blog post oficial[^30] |
| **Tipo** | Serviço (MCP server local) |
| **Storage** | Qdrant local (vector) + SQLite[^30] |
| **Exemplos para código** | ✅ Cross-client memory: store em Cursor, recall em Claude[^30] |

**Pontos fortes:** Totalmente local — nada vai para cloud. UI dashboard em localhost:3000 para gerenciar memórias[^30]. Setup em 1 comando: `curl -sL ...run.sh | bash`[^32]. Funciona com Claude Desktop, Cursor, Windsurf, Cline[^30]. Um único servidor compartilhado entre todos os clientes MCP — memória portátil entre ferramentas[^30].

**Limitações:** Requer Docker. Sem captura automática — memórias precisam ser adicionadas manualmente ou via prompt. Menos sofisticado que agentmemory para coding agents.

**Quando usar:** Quando você quer memória simples, local e privada funcionando com múltiplos clientes MCP. Boa opção para começar sem complexidade.

**Quando evitar:** Quando precisa de captura automática ou análise temporal.

***

### 7. letta-ai/letta (framework MemGPT)

| Campo | Valor |
|---|---|
| **Link** | https://github.com/letta-ai/letta |
| **Organização** | Letta AI (ex-MemGPT, UC Berkeley)[^33] |
| **Descrição** | Framework para agentes stateful com memória avançada: core memory (sistema prompt editável), recall memory (histórico searchable), archival memory (vetor ilimitado)[^34] |
| **Stars** | ~23k[^35] |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0[^33] |
| **Linguagem principal** | Python |
| **Último commit** | Jun 2026[^36] |
| **Releases** | Múltiplos em 2026[^36] |
| **Maturidade** | Maduro — plataforma consolidada com empresa por trás |
| **Documentação** | docs.letta.com — completa[^34] |
| **Tipo** | Framework + plataforma (API server, ADE, SDKs) |
| **Storage** | Postgres + vector DB (3-tier: core/recall/archival)[^37] |
| **Exemplos para código** | ✅ via Letta Code e claude-subconscious[^38] |

**Pontos fortes:** Arquitetura 3-tier de memória academicamente fundamentada: core (sempre em contexto), recall (histórico searchable), archival (ilimitado). Suporte completo Python e TypeScript SDKs. ADE (Agent Development Environment) no-code para construir e testar agentes. API REST padronizada. Integração com Vercel AI SDK, n8n, Telegram, Discord. Base do Letta Code e claude-subconscious.[^38][^37][^39]

**Limitações:** Framework mais complexo — overhead maior para casos simples. Alta dependência da plataforma Letta (server obrigatório). Menos plug-and-play para coding agents do que Letta Code especificamente.

**Quando usar:** Quando precisa construir um agente customizado com memória sofisticada. Quando quer controle fino sobre arquitetura de memória. Backend do claude-subconscious e Letta Code.

**Quando evitar:** Para adoção direta em coding workflows — use Letta Code em vez disso.

***

### 8. letta-ai/claude-subconscious

| Campo | Valor |
|---|---|
| **Link** | https://github.com/letta-ai/claude-subconscious |
| **Organização** | Letta AI[^40] |
| **Descrição** | Plugin para Claude Code: agente background que monitora sessões, acumula memórias, injeta contexto antes de cada prompt[^38] |
| **Stars** | ~1.6k (mar 2026)[^41] |
| **Data verificação** | 02/07/2026 |
| **Licença** | MIT[^38] |
| **Linguagem principal** | TypeScript |
| **Último commit** | Mar/abr 2026[^42] |
| **Releases** | Ativo desde jan 2026[^41] |
| **Maturidade** | Alpha/Beta — experimental mas funcional |
| **Documentação** | README + blog Letta[^38] |
| **Tipo** | Plugin (Claude Code) |
| **Storage** | Letta API (memory blocks)[^38] |
| **Exemplos para código** | ✅ Watch sessões Claude Code, injetar memória em próximos prompts[^38] |

**Pontos fortes:** Instalação em 2 comandos via plugin marketplace Claude Code. Background agent — não interfere no fluxo. Compartilhamento de memória entre múltiplas sessões Claude Code paralelas. Detecta quando você está travado no mesmo bug por tempo demais. Computer use support adicionado (pode ler/escrever arquivos).[^40][^43][^44][^45]

**Limitações:** Depende do Letta API (server externo). Stars baixas, projeto experimental. Específico para Claude Code. Atividade de commits desacelerou desde mar 2026.

**Quando usar:** Exclusivamente Claude Code users que querem memória intra-sessão com mínima configuração. Complementa CLAUDE.md.

**Quando evitar:** Usuários de outros coding agents. Quando precisa de privacidade total (usa Letta API externa).

***

### 9. supermemoryai/supermemory

| Campo | Valor |
|---|---|
| **Link** | https://github.com/supermemoryai/supermemory |
| **Organização** | Supermemory AI (Dhravya Shah, seed $2.6-3M, Jeff Dean e Cloudflare CTO como angels)[^46] |
| **Descrição** | Memory API e context engine para agentes de IA — extrai fatos, rastreia mudanças, resolve contradições, forgetting automático[^47] |
| **Stars** | ~27k[^48] |
| **Data verificação** | 02/07/2026 |
| **Licença** | MIT[^49] |
| **Linguagem principal** | TypeScript |
| **Último commit** | Jun 2026[^48] |
| **Releases** | Frequentes[^49] |
| **Maturidade** | Beta/early GA — financiamento e crescimento acelerado |
| **Documentação** | supermemory.ai + GitHub README[^46] |
| **Tipo** | Serviço (API cloud + self-hosted) |
| **Storage** | Híbrido proprietário (Cloudflare infrastructure)[^50] |
| **Exemplos para código** | ⚠️ Exemplos genéricos; MCP server para Claude Code e Cursor[^50] |

**Pontos fortes:** #1 em múltiplos benchmarks: LongMemEval 81.6%, #1 no LoCoMo e ConvoMem. Connectors para Google Drive, Gmail, Notion, OneDrive, GitHub. User profiles em ~50ms. MCP server para Claude Code, Cursor. Free tier disponível.[^47][^50]

**Limitações:** Infraestrutura cloud Cloudflare — self-hosting requer mais configuração. Foco em memória conversacional/pessoal, não específico para coding agents com bugs/arch decisions. TypeScript-first, menos integração Python.

**Quando usar:** Quando precisa de memória de alta performance com benchmarks comprovados para uso conversacional. Como complement de search rápida.

**Quando evitar:** Para o caso de uso de project memory de coding agents — agentmemory é mais especializado.

***

### 10. langchain-ai/langgraph + langchain-ai/langmem

| Campo | Valor |
|---|---|
| **Links** | https://github.com/langchain-ai/langgraph · https://github.com/langchain-ai/langmem |
| **Organização** | LangChain AI[^51] |
| **Descrição** | LangGraph: framework de agentes stateful. LangMem: biblioteca de memória para LangGraph com extração automática, background manager e BaseStore[^52] |
| **Stars** | LangGraph: 35.9k · LangMem: 1.1k[^51] (LangMem de págna GitHub lida diretamente) |
| **Data verificação** | 02/07/2026 |
| **Licença** | MIT[^51] |
| **Linguagem principal** | Python |
| **Último commit** | LangGraph: 28 jun 2026 (muito ativo)[^51]; LangMem: jul 2025[^53] |
| **Releases** | LangGraph: 549 releases[^51]; LangMem: sem releases formais |
| **Maturidade** | LangGraph: production-ready. LangMem: alpha/beta |
| **Documentação** | langchain-ai.github.io/langmem/ — boa[^54] |
| **Tipo** | Framework (LangGraph) + biblioteca (LangMem) |
| **Storage** | PostgreSQL (PostgresStore) + pgvector, Redis, MongoDB, InMemory[^55] |
| **Exemplos para código** | ✅ via LangGraph how-tos; LangMem exemplos funcionais[^52] |

**Pontos fortes:** Se já usa LangGraph, LangMem é a integração natural — zero fricção. PostgresStore com pgvector: semântica de busca + persistência robusta. Background memory manager extrai e consolida conhecimento de conversas automaticamente. Cross-thread memory via namespaces. Usado em produção por Klarna, Replit, Elastic.[^54][^52][^55][^51]

**Limitações:** LangMem muito novo (1.1k stars, sem releases formais, commits pararam em jul 2025 para package). LangGraph requer conhecimento do framework — não é plug-and-play. Sem captura automática de hooks de coding agents sem implementação customizada.

**Quando usar:** Quando já tem um agente LangGraph e quer adicionar memória de forma nativa. Quando prefere stack Python consolidado e governança LangChain.

**Quando evitar:** Fora do ecossistema LangChain. Para coding agents — há opções mais diretas.

***

### 11. getzep/zep (Community Edition)

| Campo | Valor |
|---|---|
| **Link** | https://github.com/getzep/zep |
| **Organização** | Zep / getzep[^56] |
| **Descrição** | Plataforma de memória para agentes com temporal knowledge graph — aprende de interações e dados de negócio[^57] |
| **Stars** | ~3.7k (CE repo)[^58]; Graphiti (engine): ~28k[^23] |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0[^57] |
| **Linguagem principal** | Python |
| **Último commit** | Nov 2024 (CE repo) — Zep é principalmente cloud com Graphiti como engine OSS ativa[^59] |
| **Releases** | CE: última release nov 2024[^59] |
| **Maturidade** | CE: manutenção mínima; Zep Cloud: ativo |
| **Documentação** | help.getzep.com/ce/quickstart[^57] |
| **Tipo** | Plataforma (cloud + CE self-hosted) |
| **Storage** | Graphiti (Neo4j/FalkorDB temporal KG)[^56] |
| **Exemplos para código** | ✅ Exemplos com LangChain, LlamaIndex, CrewAI[^56] |

**Pontos fortes:** O engine Graphiti (28k stars) é o que realmente importa — Zep CE é a plataforma ao redor dele. Zep Cloud tem temporal KG, fact extraction, session history, user-level memory. Integrações Python, TypeScript, Go.[^56][^24]

**Limitações:** CE repo com baixa atividade — desenvolvimento foca no Graphiti OSS e Zep Cloud pago. Para self-hosting de qualidade, use Graphiti diretamente.

**Quando usar:** Use **Graphiti diretamente** se quiser o engine. Zep Cloud para managed service enterprise.

**Quando evitar:** Para coding agents — use Graphiti ou agentmemory.

***

### 12. run-llama/llama_index (Memory Module)

| Campo | Valor |
|---|---|
| **Link** | https://github.com/run-llama/llama_index |
| **Organização** | LlamaIndex / run-llama[^60] |
| **Descrição** | Framework de document agents e RAG com Memory API modular: StaticMemoryBlock, FactExtractionMemoryBlock, VectorMemoryBlock[^61] |
| **Stars** | ~48k[^62] |
| **Data verificação** | 02/07/2026 |
| **Licença** | MIT[^62] |
| **Linguagem principal** | Python |
| **Último commit** | Ativo[^60] |
| **Releases** | Frequentes[^60] |
| **Maturidade** | Maduro — framework bem estabelecido |
| **Documentação** | docs.llamaindex.ai — completa[^61] |
| **Tipo** | Framework |
| **Storage** | Pluggável: vector stores, graph stores, SQL[^61] |
| **Exemplos para código** | ✅ mas genérico — pouco específico para coding agents |

**Pontos fortes:** Memory API nova (mai 2025) com blocos plug-and-play. Maior ecossistema de integrações para document ingestion. Funciona bem se já usa LlamaIndex para RAG.[^61]

**Limitações:** Memory module é complementar ao framework — não standalone. Menos adequado para o caso de coding agent memory do que as ferramentas especializadas. Sem captura automática de sessões.

**Quando usar:** Se você já usa LlamaIndex para RAG e quer adicionar memória sem mudar de framework.

**Quando evitar:** Fora do ecossistema LlamaIndex para o caso específico de coding agent memory.

***

### 13. NirDiamant/Agent_Memory_Techniques (Referência Educacional)

| Campo | Valor |
|---|---|
| **Link** | https://github.com/NirDiamant/Agent_Memory_Techniques |
| **Organização** | Nir Diamant (GenAI Consultant, autor)[^63] |
| **Descrição** | 30 Jupyter notebooks executáveis cobrindo todas as técnicas de memória para agentes LLM: buffers, vetores, grafos, MemGPT, Mem0, Letta, Zep, Graphiti, benchmarks LoCoMo[^64] |
| **Stars** | ~538[^65] |
| **Data verificação** | 02/07/2026 |
| **Licença** | Apache-2.0[^66] |
| **Linguagem principal** | Jupyter Notebook / Python |
| **Último commit** | Jun 2026[^65] |
| **Releases** | N/A — repositório educacional |
| **Maturidade** | Estável como referência |
| **Documentação** | Autocontido — cada técnica tem notebook + writeup[^64] |
| **Tipo** | Template / referência educacional |
| **Storage** | Vários (depende do notebook)[^37] |
| **Exemplos para código** | ✅✅✅ — É inteiramente exemplos com código executável[^64] |

**Pontos fortes:** Recurso de aprendizado mais completo do espaço — comparação de 30 técnicas com código. Benchmarks LoCoMo e LongMemEval implementados. Inclui notebooks específicos sobre memory hierarchy para Claude Code e Anthropic 7 layers of memory. Cross-session memory, multi-agent shared memory, memory as tools.[^67][^37]

**Limitações:** Educacional, não produção. Stars ainda baixas (538). Não é uma biblioteca para instalar.

**Quando usar:** Para **aprender** antes de implementar. Para comparar abordagens antes de escolher uma ferramenta. Para entender os trade-offs de KG vs vector vs episodic.[^37]

**Quando evitar:** Como solução de produção.

***

## Análise para o Caso de Uso Principal

### "Memória de projeto para agente de programação que aprende com bugs resolvidos, decisões arquiteturais e implementações anteriores"

Este caso tem três requisitos específicos que diferenciam de memória conversacional genérica:

**1. Captura automática (hooks):** O agente deve capturar o que aconteceu sem você precisar dizer "lembre isso". Apenas **agentmemory** e **Letta Code** têm isso nativamente via hooks de coding agents.

**2. Memória de projeto (não de usuário):** Bugs resolvidos e decisões arquiteturais são específicos ao _codebase_, não ao usuário. **Graphiti**, **Cognee** e **agentmemory** modelam isso melhor.

**3. Recall contextual ao arquivo/módulo:** Ao abrir `src/auth/jwt.ts`, o agente deve saber automaticamente o histórico de bugs e decisões sobre esse arquivo. Apenas **agentmemory** (via `PreToolUse` + path-scoped memories) e **aide-memory** têm isso explicitamente.

### Recomendação por Perfil

| Perfil | Recomendação principal | Alternativa |
|---|---|---|
| **Claude Code user, quer zero config** | agentmemory | claude-subconscious |
| **Letta Code user** | letta-ai/letta-code | agentmemory como complement |
| **Stack Python + LangGraph** | Cognee + LangGraph | LangMem + PostgresStore |
| **Quer knowledge graph temporal** | Graphiti standalone | Cognee |
| **Múltiplos agentes compartilhando memória** | agentmemory (REST/MCP shared) | Letta (Conversations API) |
| **Aprender antes de implementar** | NirDiamant/Agent_Memory_Techniques | — |

***

## Ecossistema Complementar Relevante

- **claude-code-best-practice (shanraisshan):** Documentação sobre agent memory frontmatter no Claude Code v2.1.33+ — scopes user/project/local para MEMORY.md[^68]
- **aide-memory (aide-memory/aide-memory):** Ferramenta NPM para memória path-scoped em Claude Code e Cursor — captura por hooks com recall localizado por área de código[^69]
- **RAGFlow (infiniflow/ragflow):** 82k stars, Apache-2.0 — RAG engine com agent memory desde dez 2025 (v0.25) — mais voltado a pipelines documentais do que coding agents[^70]
- **Anthropic Managed Agents Memory (beta):** Sistema nativo de memory stores com montagem em `/mnt/memory/`, 8 stores por sessão, permissões read-only/read-write — beta público abr 2026[^71]
- **GitHub Copilot Cross-Agent Memory (preview):** Memory nativa cross-agent entre Coding Agent, CLI e Code Review — public preview jan 2026, desabilitada por padrão[^72]

***

## Tabela de Riscos e Licenças

| Repositório | Licença | Risco OSS | Risco Técnico |
|---|---|---|---|
| agentmemory | Apache-2.0 | Projeto individual (bus factor) | Dependência iii-engine |
| Letta Code | Apache-2.0 | Empresa jovem, bem fundada | Cloud sync requer Letta |
| Cognee | Apache-2.0 | Empresa com funding ($7.5M) | Python-only SDK |
| Graphiti | Apache-2.0 | Empresa pequena (getzep) | Requer graph DB |
| Mem0 | Apache-2.0 | Cloud/OSS split crescente | Graph atrás de paywall |
| Letta/MemGPT | Apache-2.0 | Empresa bem estabelecida | Alta dependência do server |
| OpenMemory MCP | Apache-2.0 | Empresa (Mem0 AI) | Docker obrigatório |
| claude-subconscious | MIT | Projeto experimental Letta | Usa Letta API externa |
| Supermemory | MIT | Startup jovem | Infra Cloudflare |
| LangGraph | MIT | LangChain Inc — consolidada | Vendor lock-in LangChain |
| LangMem | MIT | LangChain Inc | Desenvolvimento pausado |
| Zep CE | Apache-2.0 | Engine ativa (Graphiti) | CE com pouca manutenção |
| LlamaIndex | MIT | Empresa consolidada | Genérico para o caso |

---

## References

1. [rohitg00/agentmemory: #1 Persistent memory for AI coding agents ...](https://github.com/rohitg00/agentmemory) - agentmemory works with any agent that supports hooks, MCP, or REST API. All agents share the same me...

2. [agentmemory · GitHub Topics](https://github.com/topics/agentmemory) - Long-term, cross-project memory for AI coding agents. Your own Obsidian vault as the source of truth...

3. [Agentmemory Awards (2026) - Product Hunt](https://www.producthunt.com/products/agent-memory-dev/awards) - See all awards and badges for Agentmemory on Product Hunt.

4. [package.json - rohitg00/agentmemory - GitHub](https://github.com/rohitg00/agentmemory/blob/main/package.json) - 1 Persistent memory for AI coding agents based on real-world benchmarks - agentmemory/package.json a...

5. [Releases · rohitg00/agentmemory](https://github.com/rohitg00/agentmemory/releases) - #1 Persistent memory for AI coding agents based on real-world benchmarks - rohitg00/agentmemory

6. [2026-05-20-coding-agent-life-v1.md](https://github.com/rohitg00/agentmemory/blob/main/docs/benchmarks/2026-05-20-coding-agent-life-v1.md) - #1 Persistent memory for AI coding agents based on real-world benchmarks - rohitg00/agentmemory

7. [I built agentmemory — your AI coding agent now remembers ...](https://www.reddit.com/r/ChatGPT/comments/1sfr0jy/i_built_agentmemory_your_ai_coding_agent_now/) - agentmemory runs silently in the background and captures everything your agent does tool calls, file...

8. [Issues · rohitg00/agentmemory - GitHub](https://github.com/rohitg00/agentmemory/issues) - 1 Persistent memory for AI coding agents based on real-world benchmarks - Issues · rohitg00/agentmem...

9. [Letta Code: A Memory-First Coding Agent](https://www.letta.com/blog/letta-code/) - Letta Code is a memory-first coding agent, designed for working with agents that learn over time. Wh...

10. [Introducing Context Repositories: Git-based Memory for Coding ...](https://www.letta.com/blog/context-repositories/) - We're introducing Context Repositories, a rebuild of how memory works in Letta Code based on program...

11. [Letta Code v0.25 — Memory-First Coding Agent,… | AI/TLDR](https://ai-tldr.dev/releases/letta-code-app/) - Letta Code stores coding context in persistent memory — switch models mid-session and your agent rem...

12. [Releases · letta-ai/letta-code - GitHub](https://github.com/letta-ai/letta-code/releases) - The memory-first coding agent. Contribute to letta-ai/letta-code development by creating an account ...

13. [Memory | Letta Docs](https://docs.letta.com/letta-code/memory) - Understand Letta Agent's self-improving memory system

14. [Letta Code Tutorial - Memory-First Coding Agent | Claude, GPT, Gemini](https://www.youtube.com/watch?v=ZbrzIkmW0cs) - Learn about Letta Code - the memory-first coding agent that learns from experience.

Unlike stateles...

15. [Cognee Review (2026): AI Agent Memory, Pricing & Verdict](https://theaiagentindex.com/agents/cognee) - Cognee: AI agent memory using knowledge graphs. 17,800+ GitHub stars. Self-hosted free, cloud Develo...

16. [Cognee | Ry Walker Research](https://rywalker.com/research/cognee) - Cognee is an open-source AI memory engine that turns scattered data into a queryable knowledge graph...

17. [topoteretes/cognee - 22k Stars · Global Rank #1875](https://www.star-history.com/topoteretes/cognee/) - Cognee is the open-source AI memory platform for agents. Give your AI agents persistent long-term me...

18. [Cognee — Stars, Velocity & Guide | OSS AI Hub](https://ossaihub.com/tool/cognee/) - Knowledge graph + memory layer for structured, queryable agent understanding.

19. [cognee - vector database (17,717★) | Every Local AI](https://everylocalai.com/tool/cognee) - cognee: Cognee is the open-source AI memory platform for agents. Give your AI agents persistent long...

20. [Real-time memory for Codex + Claude Code: multi-tenant + auth + privacy](https://www.youtube.com/watch?v=uc-cJ2fqknU) - Vasilije Markovic, CEO of Cognee, shows how to give Claude Code and Codex real-time, persistent memo...

21. [Graphiti Hits 20K Stars! + MCP Server 1.0 - Zep](https://blog.getzep.com/graphiti-hits-20k-stars-mcp-server-1-0/) - Graphiti crossed 20,000 GitHub stars today! Thanks for building with us. Graphiti is a temporal know...

22. [getzep/graphiti: Build Real-Time Knowledge Graphs for AI ...](https://github.com/getzep/graphiti) - Graphiti is a framework for building and querying temporal context graphs for AI agents. Unlike stat...

23. [getzep/graphiti - 27.9k Stars · Global Rank #1272](https://www.star-history.com/getzep/graphiti/) - Build Real-Time Knowledge Graphs for AI Agents — 28.2k stars, 44 contributors, This week: +14 stars

24. [Zep](https://llms3.com/node/zep) - An open-source AI memory platform (Apache 2.0) built around the **Graphiti** temporal-knowledge-grap...

25. [mem0ai/mem0 GitHub star history (58.9k stars) - gitstar](https://www.gitstar.co.kr/mem0ai/mem0) - Universal memory layer for AI Agents GitHub star history — rank #339 overall, 59.1k stars, 6.8k fork...

26. [Keep losing your memory during long conversations? 6 design ...](https://ai-chain.tw/en/blog/mem0-long-term-memory-design/) - The biggest problem with long dialogue systems is usually not that they can’t answer, but that they ...

27. [GitHub - mem0ai/mem0: Memory for AI Agents; Announcing OpenMemory MCP - local and secure memory management.](https://github.com/mem0ai/mem0/?tab=readme-ov-file) - Memory for AI Agents; Announcing OpenMemory MCP - local and secure memory management. - mem0ai/mem0

28. [markmbain/mem0ai-mem0: The memory layer for ...](https://github.com/markmbain/mem0ai-mem0) - The memory layer for Personalized AI. Contribute to markmbain/mem0ai-mem0 development by creating an...

29. [Mem0, Open Source Giving Memory to AI Agents [2026] - AI Digester](https://aidigester.org/en/mem0-open-source-giving-memory-to-ai-agents-2026-2/) - Mem0: 3 Key Aspects of the AI Agent Memory Layer GitHub Stars: 46,900+ Languages: Python, TypeScript...

30. [Introducing OpenMemory MCP - Mem0](https://mem0.ai/blog/introducing-openmemory-mcp) - OpenMemory MCP adds persistent, user-owned memory to Claude, Cursor, and any MCP-compatible client. ...

31. [Graph Memory - Mem0](https://mem0.mintlify.app/open-source/features/graph-memory) - Layer relationships onto Mem0 search so agents remember who did what, when, and with whom.

32. [mem0/openmemory at main · mem0ai/mem0](https://github.com/mem0ai/mem0/tree/main/openmemory) - Memory for AI Agents; Announcing OpenMemory MCP - local and secure memory management. - mem0ai/mem0

33. [Letta (formerly MemGPT) - GitHub](https://github.com/letta-ai/letta) - Build AI with advanced memory that can learn and self-improve over time. Letta Code: run agents loca...

34. [GitHub - letta-ai/letta: Letta (formerly MemGPT) is the stateful agents framework with memory, reasoning, and context management.](https://github.com/letta-ai/letta?tab=readme-+%22ov-%22+file) - Letta (formerly MemGPT) is the stateful agents framework with memory, reasoning, and context managem...

35. [letta-ai/letta - 21.8k Stars · Global Rank #1751 - GitHub Star History](https://www.star-history.com/letta-ai/letta/) - Letta is the platform for building stateful agents: AI with advanced memory that can learn and self-...

36. [Releases · letta-ai/letta](https://github.com/letta-ai/letta/releases) - Letta is the platform for building stateful agents: open AI with advanced memory that can learn and ...

37. [Agent_Memory_Techniques/docs/comparison.md at main](https://github.com/NirDiamant/Agent_Memory_Techniques/blob/main/docs/comparison.md) - Agent memory for LLMs: 30 runnable Jupyter notebooks covering conversation buffers, vector stores, k...

38. [Claude Subconscious - Persistent Memory Agent for Claude Code](https://www.youtube.com/watch?v=BX_45Tb5DlA) - Claude Subconscious is a background agent that whispers to Claude Code, providing persistent memory ...

39. [Letta](https://github.com/letta-ai) - The platform for stateful AI agents. Letta has 29 repositories available. Follow their code on GitHu...

40. [letta-ai/claude-subconscious](https://github.com/letta-ai/claude-subconscious) - Using Letta's Conversations feature, a single agent can serve multiple Claude Code sessions in paral...

41. [letta-ai/claude-subconscious - Gource visualisation](https://www.youtube.com/watch?v=wRtc1pWmuvM) - 🚀 Watch the development journey of claude-subconscious by letta-ai!

📝 Give Claude Code a subconscio...

42. [Activity · letta-ai/claude-subconscious](https://github.com/letta-ai/claude-subconscious/activity) - Give Claude Code a subconscious. Contribute to letta-ai/claude-subconscious development by creating ...

43. [Letta Devlog: Adding computer use to Claude Subconscious](https://www.youtube.com/watch?v=3npYYBU9NN4) - Claude Subconscious is a plugin to have a Letta agent manage Claude's memory live. Here's a quick de...

44. [(Podcast) Giving Claude a Brain? Deep Dive into Claude Subconscious!](https://www.youtube.com/watch?v=fxGBDD-zWYw) - 🧠 **Giving Claude a Brain? Deep Dive into Claude Subconscious!** 🚀

Ever feel like your AI assistant...

45. [Claude Subconscious AI Coding Agent Launched - LinkedIn](https://www.linkedin.com/posts/nileshmishra2007_claude-code-just-got-a-subconscious-letta-activity-7455972081571225600--ZoS) - Claude Code just got a subconscious. Letta open-sourced the memory layer AI coding agents have alway...

46. [Supermemory | Ry Walker Research](https://rywalker.com/research/supermemory) - Supermemory is a memory API and context engine for AI agents — store unstructured data, retrieve sub...

47. [Supermemory gives AI a permanent memory — for free | AI for ...](https://aiforautomation.io/news/2026-03-24-supermemory-18k-stars-ai-memory-layer-free) - Supermemory just hit 18K GitHub stars as the #1-ranked memory engine for AI tools. It gives Claude C...

48. [supermemory — Stars, Velocity & Guide | OSS AI Hub](https://ossaihub.com/tool/supermemoryai-supermemory/) - Memory engine and app that is extremely fast, scalable. The Memory API for the AI era.

49. [supermemoryai/supermemory: Memory and context engine ...](https://github.com/supermemoryai/supermemory) - Memory and context engine + app that is extremely fast, scalable, and can be run fully locally. The ...

50. [This AI Memory API Just Hit #1 on Every Benchmark | Supermemory](https://www.youtube.com/watch?v=vRrUTvHW7Ww) - Supermemory is the #1 ranked memory API — 81.6% on LongMemEval, #1 on LoCoMo and ConvoMem. In this d...

51. [langchain-ai/langgraph: Build resilient agents. - GitHub](https://github.com/langchain-ai/langgraph) - LangGraph is a low-level orchestration framework for building, managing, and deploying long-running,...

52. [langchain-ai/langmem](https://github.com/langchain-ai/langmem) - Contribute to langchain-ai/langmem development by creating an account on GitHub.

53. [langmem/pyproject.toml at main · langchain-ai/langmem](https://github.com/langchain-ai/langmem/blob/main/pyproject.toml) - Contribute to langchain-ai/langmem development by creating an account on GitHub.

54. [LangMem](https://langchain-ai.github.io/langmem/) - LangMem helps agents learn and adapt from their interactions over time. It provides tooling to extra...

55. [Memory Store Langchain - LangGraph](https://forum.langchain.com/t/memory-store-langchain/1617) - How to configure external memory Store, for instance, PostgresStore other than InMemoryStore

56. [getzep/zep: Zep | Examples, Integrations, & More](https://github.com/getzep/zep) - It contains example code, framework integrations, and tools for building agent memory with Zep Cloud...

57. [Announcing: Zep Community Edition](https://blog.getzep.com/announcing-zep-community-edition/) - We've just open-sourced Zep Community Edition (CE), a memory layer for AI agents that continuously l...

58. [Package zep-cloud](https://github.com/orgs/getzep/packages/container/package/zep-cloud) - This repository is not Zep's product or service. It contains example code, framework integrations, a...

59. [Releases · getzep/zep - GitHub](https://github.com/getzep/zep/releases) - This release fixes compatibility issues with CrewAI version 0.186.0 and later. Changes. Updated exte...

60. [GitHub - run-llama/llama_index: LlamaIndex is the leading document agent and OCR platform](https://github.com/run-llama/llama_index) - LlamaIndex is the leading document agent and OCR platform - run-llama/llama_index

61. [LlamaIndex (@llama_index) on X](https://x.com/llama_index/status/1922340015499313543)

62. [AWS Bedrock AgentCore vs LlamaIndex - A Tour of Agents](https://tinyagents.dev/vs/aws-agentcore-vs-llamaindex) - Compare AWS Bedrock AgentCore and LlamaIndex side by side. See what each framework abstracts and how...

63. [NirDiamant - Overview](https://github.com/NirDiamant) - GenAI Consultant | Open Source Hub | Best-selling book author - NirDiamant

64. [NirDiamant/Agent_Memory_Techniques: Agent memory for ...](https://github.com/NirDiamant/Agent_Memory_Techniques) - Agent memory for LLMs: 30 runnable Jupyter notebooks covering conversation buffers, vector stores, k...

65. [Agent_Memory_Techniques — Stars, Velocity & Guide | OSS AI Hub](https://ossaihub.com/tool/nirdiamant-agent-memory-techniques/) - Agent memory for LLMs: 30 runnable Jupyter notebooks covering conversation buffers, vector stores, k...

66. [NirDiamant/Agent_Memory_Techniques — GitHub trending stats ...](https://trendshift.io/repositories/33628)

67. [Agent_Memory_Techniques/docs/topics.md at main - GitHub](https://github.com/NirDiamant/Agent_Memory_Techniques/blob/main/docs/topics.md) - Agent memory for LLMs: 30 runnable Jupyter notebooks covering conversation buffers, vector stores, k...

68. [claude-code-best-practice/reports/claude-agent-memory.md at main](https://github.com/shanraisshan/claude-code-best-practice/blob/main/reports/claude-agent-memory.md) - from vibe coding to agentic engineering - practice makes claude perfect - shanraisshan/claude-code-b...

69. [aide-memory: persistent memory for AI coding agents and teams](https://dev.to/ahmedmmeky/aide-memory-persistent-memory-for-ai-coding-agents-and-teams-25je) - What lives in your head doesn't flow across AI sessions, and doesn't always make it to your team....

70. [RAGFlow v0.25.0 — Seven Pipeline Templates,… | AI/TLDR](https://ai-tldr.dev/releases/ragflow-v0-25-0/) - RAGFlow v0.25 ships production-ready agent publishing, persistent user memory, and ingestion pipelin...

71. [Anthropic Bridges the Memory Gap: Claude Managed Agents ...](https://www.ainexusdaily.com/article/anthropic-claude-persistent-memory-managed-agents-beta) - Anthropic launches public beta for persistent memory on Claude Managed Agents, enabling cross-sessio...

72. [GitHub Copilot: Neues Memory-System lernt über Agenten hinweg](https://www.microsoft.com/de-de/techwiese/news/github-copilot-neues-memory-system-lernt-ueber-agenten-hinweg.aspx) - GitHub Copilot erhält ein neues, agentenübergreifendes Memory-System. Copilot kann dadurch projektbe...

