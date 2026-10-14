# Memória Persistente para Agentes de IA em Projetos de Software
## Pesquisa Mestre: Visão Completa, Verificável e Atualizada (Jul 2026)

***

## Resumo Executivo

Implementar memória persistente para agentes/modelos de IA em projetos de software é **tecnicamente possível hoje**, em múltiplas camadas e com ferramentas maduras disponíveis — open source e gerenciadas. O problema não é mais se pode ser feito, mas *como* fazê-lo corretamente.

A distinção central é esta: **o modelo não "aprende" entre sessões** (sem fine-tuning ou re-treinamento); em vez disso, um **sistema externo mantém e recupera memória** que é injetada no contexto em cada nova sessão. Essa memória pode ser de baixa granularidade (arquivo Markdown com instruções) até alta complexidade (grafo de conhecimento temporal com embeddings semânticos e relações tipadas).

Para projetos de código, a arquitetura recomendada combina: (1) arquivos de projeto versionados como `CLAUDE.md` / `AGENTS.md` / `.memory/`, (2) um banco de dados vetorial para recuperação semântica, (3) ADRs (Architecture Decision Records) como memória de decisões e (4) um processo de curadoria que extrai e valida memórias automaticamente a cada evento relevante — bug resolvido, decisão arquitetural tomada, padrão identificado.

Os riscos mais críticos são **memory poisoning** (memórias falsas ou maliciosas que persistem entre sessões), **context drift** (acúmulo de memórias obsoletas) e **vazamento de segredos** armazenados em texto claro. Todas essas ameaças têm mitigações conhecidas, mas exigem governança ativa.

***

## Glossário dos Conceitos

| Termo | Definição Precisa |
|---|---|
| **Working context** | O conjunto de tokens *ativo no momento* dentro da janela de contexto do LLM em uma inferência. É o que o modelo "vê" naquele instante. Tudo fora dele não existe para o modelo[^1]. |
| **Contexto de sessão** | Tudo que foi trocado entre usuário e agente dentro de uma sessão única (thread/conversation). Inclui histórico de mensagens, resultados de ferramentas, etc. Reseta ao fechar a sessão. |
| **Memória curta (short-term)** | Memória *intra-sessão*. Em frameworks como LangGraph, é mantida por checkpointers vinculados a um `thread_id`[^2]. Equivale ao buffer de conversa ativo. |
| **Memória longa (long-term)** | Memória *cross-session*. Persiste além de uma sessão através de armazenamento externo (banco, arquivo, grafo). É recuperada e injetada no contexto na próxima sessão[^3]. |
| **Memória de projeto** | Subconjunto de memória longa específico a um repositório/projeto: arquitetura, convenções, bugs conhecidos, comandos, ADRs. Representada por arquivos como `CLAUDE.md`[^4] ou `.memory/`. |
| **Memória durável** | Sinônimo de memória persistente com garantias de integridade: versionada, auditável, com proveniência rastreável e resistente a corrupção. |
| **RAG (Retrieval-Augmented Generation)** | Padrão arquitetural onde documentos/memórias são indexados em um store externo; no momento da inferência, os fragmentos mais relevantes para a query atual são recuperados e injetados no contexto do LLM[^5]. O modelo não muda — apenas o contexto muda. |
| **Vector store** | Banco de dados especializado que armazena embeddings (representações vetoriais de alta dimensão de texto/código) e suporta busca por similaridade semântica (nearest-neighbor search). Ex.: Chroma, Qdrant, pgvector, Pinecone[^5]. |
| **Knowledge graph** | Estrutura de grafo onde nós representam entidades e arestas representam relações tipadas entre elas. Permite consultas relacionais multi-hop ("qual bug foi resolvido que afetou o módulo X?")[^6]. |
| **Memória episódica** | Memória de *eventos específicos*: "Na sessão de 15 jun, resolvi o bug de race condition no módulo de cache usando um lock reentrant." Análogo cognitivo: lembranças autobiográficas[^3][^7]. |
| **Memória semântica** | Memória de *fatos e conhecimento geral*: "O módulo de autenticação usa JWT com expiração de 24h." Análogo cognitivo: conhecimento factual. Tipicamente implementada via RAG[^3]. |
| **Memória procedural** | Memória de *como fazer coisas*: "Para rodar os testes de integração, execute `npm run test:e2e --env=staging`." Inclui padrões de código, workflows, comandos recorrentes[^3][^8]. |
| **Fine-tuning** | Atualização dos *pesos* do modelo em um dataset proprietário. O modelo aprende padrões de estilo, formato ou vocabulário específico, mas **não aprende fatos dinâmicos** nem os atualiza entre inferências. Custoso, pontual, não é mecanismo de memória[^9]. |
| **Continuação de treinamento (continual learning)** | Re-treinamento incremental do modelo com novos dados ao longo do tempo, sem esquecer o conhecimento anterior (problema do catastrophic forgetting). Área de pesquisa, raramente usado em produção. |
| **Agent state** | Estado interno do agente durante uma execução: variáveis, progresso de tarefas, resultados intermediários de ferramentas. Em LangGraph, é o `StateDict` persistido por checkpointers[^2]. |
| **Project knowledge** | Termo usado pela OpenAI e Anthropic para o conjunto de arquivos/documentos que definem o contexto de um projeto específico, carregados automaticamente nas sessões. Ex.: files attached to an Assistant, CLAUDE.md files[^10][^4]. |
| **Scratchpad** | Espaço de rascunho temporário usado pelo agente durante uma sessão para notas, cálculos, planos. Não persiste. Equivale a `chain-of-thought` visível ou arquivos `.tmp`. |
| **ADRs (Architecture Decision Records)** | Documentos curtos e versionados que registram uma decisão arquitetural significativa: contexto, decisão tomada, consequências. Formato proposto por Michael Nygard (Cognitect, 2011)[^11]. São *memória durável de decisões* — persistem no repositório como `doc/arch/adr-NNN.md`. |
| **Changelogs** | Registros cronológicos de mudanças no software. São memória episódica versionada do código. |
| **Documentação viva (living docs)** | Documentação que é gerada ou atualizada automaticamente a partir do código ou de eventos do sistema, permanecendo sincronizada com o estado real do projeto. |

***

## Mapa Mental Textual do Ecossistema

```
MEMÓRIA PARA AGENTES DE IA EM PROJETOS DE CÓDIGO
│
├── CAMADA 0 — Contexto de Sessão (efêmero)
│   ├── Working context (tokens ativos)
│   ├── Histórico de mensagens da sessão
│   └── Resultados de ferramentas (scratchpad)
│
├── CAMADA 1 — Memória de Projeto (persistência estruturada)
│   ├── CLAUDE.md / AGENTS.md / .cursorrules (instruções humanas)
│   ├── Auto-memory / MEMORY.md (notas do agente)
│   ├── ADRs: doc/arch/adr-NNN.md
│   ├── Changelogs: CHANGELOG.md
│   └── .memory/ (diretório de memória estruturada)
│
├── CAMADA 2 — Memória Semântica (RAG / Vector Store)
│   ├── Embeddings de código, docs, bugs
│   ├── Vector DBs: Chroma, Qdrant, pgvector, Pinecone
│   └── Retrieval: top-k semântico + BM25 híbrido
│
├── CAMADA 3 — Memória Relacional (Knowledge Graph)
│   ├── Entidades: módulos, bugs, decisões, pessoas
│   ├── Relações tipadas com validade temporal
│   └── Ferramentas: Graphiti/Zep, Neo4j, FalkorDB
│
├── CAMADA 4 — Memória Gerenciada (Plataformas)
│   ├── Google: Vertex AI Memory Bank (preview, jul 2026)
│   ├── Anthropic: Claude Code CLAUDE.md + Auto Memory
│   ├── OpenAI: Responses API + File Search + Vector Stores
│   └── AWS: Bedrock AgentCore Memory
│
├── FRAMEWORKS OPEN SOURCE
│   ├── Mem0 (~52k ⭐, Apache-2.0, jun 2026)
│   ├── Letta / MemGPT (Apache-2.0, UC Berkeley)
│   ├── Graphiti / Zep (Apache-2.0)
│   ├── LangGraph Memory Store (MIT)
│   └── LlamaIndex Memory Blocks
│
└── RISCOS E GOVERNANÇA
    ├── Memory poisoning (InjecMEM, MINJA)
    ├── Context drift (memórias obsoletas)
    ├── Secret leakage
    ├── Benchmark inflation
    └── Memory curator (processo de curadoria)
```

***

## É Tecnicamente Possível Hoje?

**Sim, completamente.** Em julho de 2026, múltiplas soluções maduras estão disponíveis:

- **Google** lançou o Vertex AI Memory Bank em public preview (jul 2025), integrado ao Agent Development Kit (ADK), com geração assíncrona de memórias, busca por similaridade e TTL configurável.[^12][^13]
- **Anthropic** disponibiliza oficialmente o Claude Code com `CLAUDE.md` (instruções humanas persistentes) e Auto Memory (notas escritas pelo próprio Claude), carregados automaticamente em cada sessão. Também lançou a Memory Tool em beta público na plataforma Claude.[^4][^1]
- **OpenAI** disponibiliza vector stores via Responses API com file search, suporte a atributos para filtragem e busca semântica persistente. O Assistants API está sendo descontinuado em favor da Responses API.[^14][^15][^16]
- **LangGraph** (MIT) tem suporte nativo a short-term memory via checkpointers e long-term memory via cross-thread store com JSON documents e TTL.[^2]
- **Mem0** (~52k estrelas no GitHub, verificado jun 2026, Apache-2.0) demonstrou em benchmark superar o sistema de memória da OpenAI em 26% e reduzir latência p95 em 91%.[^17]

***

## O Que Deve Ser Salvo Como Memória — e o Que Não Deve

### ✅ O que salvar

| Categoria | Exemplos | Tipo de Memória |
|---|---|---|
| Decisões arquiteturais | "Usamos JWT sem refresh token por causa do X" | Semântica + ADR |
| Bugs resolvidos com causa raiz | "Race condition no cache de sessão — corrigido com lock reentrant" | Episódica |
| Comandos recorrentes | `npm run test:e2e`, `docker compose up --build` | Procedural |
| Padrões do codebase | "Todos os handlers seguem o padrão Repository + Service" | Semântica |
| Áreas frágeis | "O módulo de billing tem cobertura de testes insuficiente" | Semântica |
| Dependências críticas | "pgvector v0.7+ requerido pelo módulo de busca" | Semântica |
| Histórico de implementação | "Feature X foi implementada na sprint 12 com abordagem Y" | Episódica |
| Preferências do projeto | "Sempre usar pnpm, nunca npm" | Procedural |

### ❌ O que NÃO salvar

| Categoria | Razão |
|---|---|
| Secrets, tokens, senhas | Risco de vazamento em logs e retrieval[^10][^18] |
| Hipóteses não validadas | Podem virar memórias falsas que guiam decisões erradas |
| Conteúdo de usuário não-autorizado | Risco de violação de privacidade e LGPD/GDPR[^19] |
| Logs de debug verbosos e não processados | Aumentam ruído sem signal; degradam retrieval |
| Resultados de ferramentas brutos após processados | Ocupam espaço sem valor residual[^1] |
| Informações desatualizadas sem invalidação | Causam context drift e decisões erradas |
| Todo o histórico de conversa sem curadoria | "Context rot": qualidade do LLM degrada com contexto inchado[^1] |

***

## A Diferença Entre "o Modelo Aprender" e "Sistema Externo Manter Memória"

Esta distinção é fundamental e frequentemente confundida:

**O modelo aprender** (fine-tuning/continual learning) significa **alterar os pesos do modelo neural**. Esse processo:
- É custoso e lento (horas a dias de treinamento)
- Requer dataset curado e labeling
- Ensina *padrões e estilo*, não fatos dinâmicos
- Tem "cutoff": o modelo não sabe de eventos após o treinamento
- Não é um mecanismo prático de memória por projeto[^9]

**Sistema externo manter memória** significa **injetar informação no contexto antes da inferência**. Esse processo:
- É instantâneo (recuperação em milissegundos)
- Atualiza fatos dinamicamente sem re-treinar o modelo
- Permite rollback, versionamento e auditoria
- É a abordagem usada por todos os frameworks de memória em 2026[^20][^12][^2]
- É análogo ao que humanos fazem com notas, documentação e buscas

A analogia precisa: fine-tuning é como **aprender a nadar** (muda capacidade interna); RAG/memória externa é como **ter uma lista de passos em papel** (contexto externo acessado na hora). Para projetos de código, o segundo é quase sempre o correto.

***

## Arquitetura Recomendada para Projetos de Código

Esta arquitetura segue o princípio de **menor conjunto de tokens de alto sinal** descrito pela equipe da Anthropic:[^1]

### Visão Geral em Camadas

```
┌─────────────────────────────────────────────────────────────┐
│                    NOVA SESSÃO DO AGENTE                    │
├─────────────────────────────────────────────────────────────┤
│  CAMADA 1 — CONTEXTO FIXO (carregado automaticamente)       │
│  • CLAUDE.md / AGENTS.md (instruções do projeto)            │
│  • .memory/MEMORY.md (índice de memórias do agente)         │
│  • ADRs relevantes (top-3 mais recentes ou por tarefa)      │
├─────────────────────────────────────────────────────────────┤
│  CAMADA 2 — RECUPERAÇÃO JUST-IN-TIME (via ferramentas)      │
│  • Vector store: busca semântica por tarefa atual           │
│  • Knowledge graph: relações entre módulos/bugs/decisões    │
│  • Arquivos do projeto: glob/grep sob demanda               │
├─────────────────────────────────────────────────────────────┤
│  CAMADA 3 — ESTADO DA SESSÃO (checkpointer)                 │
│  • Histórico da conversa atual (thread_id)                  │
│  • Resultados de ferramentas desta sessão                   │
│  • Scratchpad / notas temporárias                           │
├─────────────────────────────────────────────────────────────┤
│  CAMADA 4 — SAÍDA: MEMORY CURATOR (ao final da sessão)      │
│  • Extrai memórias novas da sessão                          │
│  • Valida, deduplica, classifica                            │
│  • Persiste no store correto (file / vector / graph)        │
└─────────────────────────────────────────────────────────────┘
```

### Componentes Técnicos

```
projeto/
├── .memory/                    ← Camada 1: memória de projeto
│   ├── MEMORY.md              ← Índice das memórias (max 200 linhas)
│   ├── architecture.md        ← Decisões arquiteturais
│   ├── bugs.md                ← Bugs resolvidos com causa raiz
│   ├── commands.md            ← Comandos e workflows
│   ├── patterns.md            ← Padrões do codebase
│   └── fragile.md             ← Áreas frágeis e débito técnico
├── CLAUDE.md                  ← Instruções para o agente (versionadas no git)
├── doc/arch/                  ← ADRs
│   ├── adr-001.md
│   └── adr-002.md
├── CHANGELOG.md               ← Histórico de mudanças
│
└── memory-store/              ← Camada 2: vector/graph store
    ├── vector_db/             ← Chroma/Qdrant/pgvector (local)
    └── graph_db/              ← Opcional: Neo4j/FalkorDB
```

***

## Fluxo "Memory-Curator" Passo a Passo

O memory-curator é o processo que garante que a memória permaneça atualizada, limpa e útil. Deve ser executado ao final de cada sessão significativa ou após eventos-gatilho.

### Gatilhos de Curadoria

- Bug resolvido com causa raiz identificada
- Decisão arquitetural tomada
- Novo padrão identificado no código
- Erro corrigido que o agente cometeu antes
- Dependência importante adicionada/removida
- Área frágil descoberta

### Passo a Passo

**Passo 1 — Coleta de eventos da sessão**
O agente ou um processo externo coleta o histórico comprimido da sessão. Em Claude Code, isso acontece via compaction automática. Em LangGraph, via checkpointer.[^1]

**Passo 2 — Extração de candidatos a memória**
Um LLM analisa o histórico e extrai candidatos estruturados. O prompt de curadoria (ver seção dedicada abaixo) guia a extração por tipo: episódica, semântica, procedural.

**Passo 3 — Validação e deduplicação**
Cada candidato é comparado com memórias existentes:
- Se duplicata exata → descartar
- Se atualiza uma memória existente → invalidar a anterior e inserir a nova com timestamp
- Se contradiz uma memória existente → escalar para revisão humana

**Passo 4 — Classificação de confiança**
Cada memória recebe um score de confiança baseado em: fonte (output de ferramenta > afirmação do usuário), validação (testado > hipotético), recência. Memórias de baixa confiança têm TTL menor.[^18]

**Passo 5 — Persistência no store correto**
- Memória semântica/factual → vector store (com embedding)
- Memória relacional → knowledge graph (se disponível)
- Memória procedimental → arquivo `.memory/commands.md` ou `patterns.md`
- Decisão → novo ADR em `doc/arch/`

**Passo 6 — Atualização do índice MEMORY.md**
O índice é atualizado com uma linha por nova memória: data, tipo, resumo de 1 linha, referência ao arquivo de detalhe.

**Passo 7 — Invalidação de memórias obsoletas**
Memórias com TTL expirado, marcadas como supersedidas ou que contradizem o estado atual verificável (ex.: versão de dependência que mudou) são marcadas como obsoletas e movidas para quarentena antes de exclusão definitiva.[^18]

***

## Como Recuperar Apenas as Memórias Relevantes

A recuperação de memória relevante é um problema de recuperação de informação, não de storage. As estratégias principais são:

### Busca Semântica (Vector)
Embeddins da tarefa atual são comparados com embeddings das memórias armazenadas. Retorna top-k mais similares semanticamente. Funciona bem para "o que sei sobre autenticação?". Limitação: não captura relações estruturais.[^5]

### Busca Híbrida (BM25 + Vector)
Combina similaridade semântica com match léxico (palavras-chave exatas). É o padrão recomendado para produção em 2026. Resolve casos onde termos técnicos específicos são mais relevantes que semântica geral.[^21]

### Busca por Grafo (Graph Traversal)
Para perguntas relacionais: "quais módulos foram afetados pelo bug X?" ou "qual foi a decisão que levou à escolha de Y?". Graphiti/Zep implementa isso com validade temporal.[^22]

### Busca Temporal
Filtragem por recência (últimas 2 semanas, sprint atual). Útil para contexto de tarefa recente. Implementado via `valid_from`/`valid_until` no grafo ou metadados no vector store.

### Expansão por Tarefa
O agente descreve a tarefa atual em linguagem natural; o curator expande a query com termos relacionados antes de buscar. LongMemEval demonstrou que time-aware query expansion melhora significativamente a recall.[^23]

***

## Como Avaliar se a Memória Está Ajudando ou Atrapalhando

### Métricas Quantitativas

| Métrica | Como Medir | Limiar de Alerta |
|---|---|---|
| Retrieval Recall@K | % de memórias relevantes recuperadas nas top-K | < 0.80 para K=10[^9] |
| Precision@K | % das recuperadas que são realmente relevantes | < 0.70 indica ruído |
| Answer Faithfulness | Respostas ancoradas nas memórias recuperadas vs. hallucination | < 0.85 |
| Session Token Savings | Tokens economizados vs. ler tudo do zero | Benchmark baseline |
| Memory Freshness | % de memórias com menos de 30 dias | < 0.50 indica drift |

### Sinais Qualitativos

- O agente para de repetir perguntas que já foram respondidas
- O agente referencia decisões passadas corretamente
- Redução de bugs recorrentes que já foram resolvidos
- Aumento na velocidade de onboarding de novas tasks

### Benchmarks Externos

O LongMemEval (ICLR 2025, 500 questões) avalia 5 habilidades: extração de informação, raciocínio multi-sessão, raciocínio temporal, atualização de conhecimento e abstenção correta. LLMs sem memória estruturada mostram queda de 30% vs. sistemas com memória.[^23]

**Atenção**: benchmarks publicados pelos próprios desenvolvedores de ferramentas devem ser tratados com ceticismo. Um artigo de maio 2026 documenta que a maioria dos scores publicados (incluindo Mem0, Letta, Zep) tem problemas estruturais: avaliadores LLM lenientes, benchmarks curtos o suficiente para caber em um único context window, e falta de avaliação independente.[^24]

***

## Riscos e Mitigações

### 1. Memory Poisoning

**Risco**: Um atacante injeta conteúdo malicioso em uma interação normal que é armazenada como memória. Em sessões futuras, a memória envenenada é recuperada e guia decisões erradas ou prejudiciais.

O ataque InjecMEM (ICLR 2026, em revisão) demonstrou taxa de sucesso de 76.6% em sistemas de memória em produção usando apenas uma interação. O ataque MINJA (NeurIPS 2025) atingiu 95%+ de sucesso.[^25][^18]

**Mitigações**:
- Validação de conteúdo na ingestão (detecção de padrões de instrução)[^18]
- Score de confiança por fonte (output de ferramenta verificada > input de usuário não autenticado)
- Particionamento de memória por trust level (system read-only / user-scoped / agent-scoped)[^18]
- Monitoramento de drift comportamental: se o agente começa a consultar fontes de baixa confiança com frequência crescente, investigar[^18]
- Quarentena antes de exclusão definitiva para forense

### 2. Context Drift (Memórias Obsoletas)

**Risco**: Memórias que eram verdadeiras no passado mas se tornaram falsas (ex.: versão de API atualizada, padrão de código substituído) continuam sendo recuperadas e usadas.

**Mitigações**:
- TTL (Time-to-Live) em memórias com expiração automática — Vertex AI Memory Bank implementa isso nativamente[^13]
- Invalidação explícita quando uma atualização supersede uma memória anterior
- Graphiti/Zep usa `valid_from`/`valid_until` em cada aresta do grafo[^22]
- Revisão periódica humana do MEMORY.md

### 3. Vazamento de Segredos

**Risco**: Tokens de API, senhas, chaves privadas são inadvertidamente salvos como memória e depois expostos via retrieval.

**Mitigações**:
- Nunca salvar secrets em arquivos de memória — usar variáveis de ambiente[^10]
- Redaction automático antes de salvar (regex ou classifier para padrões de token)
- Memórias de projeto versionadas no git devem ser auditadas antes de commit
- Usar read-only memory stores para conteúdo sensível[^10]

### 4. Duplicidade

**Risco**: A mesma informação é salva múltiplas vezes em formatos ligeiramente diferentes, inflando o store e degradando precision.

**Mitigações**:
- Deduplicação semântica no momento da ingestão (embedding similarity > threshold → merge ou discard)
- Vertex AI Memory Bank implementa memory consolidation nativa via Gemini[^13]
- Mem0 consolida memórias conflitantes automaticamente[^20]

### 5. Contexto Irrelevante ("Lost in the Middle")

**Risco**: Recuperar memórias demais enche o contexto com informação irrelevante para a tarefa atual, degradando a qualidade de raciocínio.

**Mitigações**:
- Busca por similaridade com threshold mínimo, não top-k irrestrito
- Anthropic documenta explicitamente que "context rot" ocorre quando contexto irrelevante aumenta[^1]
- Limitar MEMORY.md a 200 linhas / 25KB (Claude Code impõe isso automaticamente)[^4]
- Reranking cross-encoder sobre top-k inicial para filtrar irrelevantes

***

## Ferramentas Open Source — Tabela Comparativa

*Dados de GitHub verificados em jul 2026. Estrelas mudam diariamente.*

| Ferramenta | Stars (jul 2026) | Licença | Tipo de Memória | Backend Storage | Destaques | Limitações |
|---|---|---|---|---|---|---|
| **Mem0** | ~52.000[^26] | Apache-2.0 | Híbrida (vector + grafo opcional) | Qdrant, Chroma, pgvector, Neo4j | API simples; Mem0g (versão com grafo); supera OpenAI memory em 26% em benchmark[^17] | Benchmarks próprios; cloud pago para features avançadas |
| **Letta / MemGPT** | ~30.000+ | Apache-2.0 | Três camadas OS-inspired: core (RAM), archival (disk), recall (index) | Qualquer SQLite/Postgres + vector | Auto-edição de memória pelo agente; #1 em Terminal-Bench (agentes de código)[^27] | Mais complexo de configurar; overhead de latência |
| **Graphiti (Zep)** | ~5.000+ | Apache-2.0 | Knowledge graph temporal | Neo4j, FalkorDB, Kuzu | Temporal validity per edge; 107 releases (alta atividade)[^28]; extração via LLM | Requer grafo DB; latência de escrita (extração LLM) |
| **LangGraph Memory Store** | (parte do LangGraph ~50.000) | MIT | Cross-thread document store + checkpointers | InMemory, SQLite, Postgres, MongoDB[^3] | Nativo ao LangGraph; TTL; namespacing flexível | Depende do ecossistema LangChain |
| **LlamaIndex Memory** | (parte do LlamaIndex ~38.000) | MIT | Short-term buffer + long-term vector/fact blocks | Qualquer vector store | Composable Memory (múltiplos backends); Vector Memory nativo[^29][^30] | Documentação fragmentada para feature de memória |
| **LangMem** | (parte do LangGraph) | MIT | Episódica, semântica, procedural | Via LangGraph store | Ferramentas pré-construídas para extração por tipo[^3] | Relativamente novo; evolução rápida |

***

## Abordagens de Armazenamento — Tabela Comparativa

| Abordagem | Melhor Para | Vantagens | Desvantagens | Custo de Implementação |
|---|---|---|---|---|
| **Markdown Versionado** (.memory/, CLAUDE.md) | Memória procedural e de projeto; projetos pequenos | Zero infraestrutura; versionado no git; auditável; legível por humanos | Sem busca semântica; escala linear | Baixo |
| **Vector DB** (Chroma, Qdrant, pgvector) | Memória semântica; grandes bases de conhecimento | Busca por significado; suporte a alta volumetria; híbrido BM25+vector | Custo de embedding; precisão limitada em multi-hop; requer re-indexação | Médio |
| **Banco Relacional + pgvector** | Projetos que já usam PostgreSQL; memória estruturada + semântica | Transações ACID; filtros SQL; extensão vetorial (pgvector)[^5] | Setup mais complexo; não é grafo | Médio |
| **Grafo de Conhecimento** (Neo4j, FalkorDB) | Memória relacional; raciocínio multi-hop; dependências entre entidades | Multi-hop nativo; temporal validity; relações tipadas | Setup complexo; custo de manutenção; extração LLM-dependente | Alto |
| **Memória Nativa de Plataforma** (Vertex AI Memory Bank, Claude Auto Memory, OpenAI Vector Stores) | Integração rápida; projetos cloud-first | Zero overhead de infra; managed; consolidação automática[^12][^13] | Lock-in de fornecedor; custo de API; menor controle | Baixo (uso) / Médio (custo) |
| **RAG Híbrido** (Vector + BM25 + Rerank + Graph) | Produção; alta precisão; queries complexas | Melhor recall+precision; robusto a variações de query; padrão 2026[^21] | Latência maior; infraestrutura complexa | Alto |

***

## Documentações Oficiais — Tabela por Provedor

| Provedor | Recurso | URL / Referência | Data de Verificação |
|---|---|---|---|
| **OpenAI** | Responses API (substitui Assistants API) | platform.openai.com/docs | Mar 2025 (lançamento)[^14][^15] |
| **OpenAI** | File Search + Vector Stores | platform.openai.com/docs/guides/retrieval | Mai 2025[^14][^16] |
| **OpenAI** | Agents SDK — Memory module | openai.github.io/openai-agents-python/ref/memory/ | 2025[^31] |
| **OpenAI** | Context Engineering for Personalization (Cookbook) | developers.openai.com/cookbook | 2025[^32] |
| **Anthropic** | Claude Code Memory (CLAUDE.md + Auto Memory) | code.claude.com/docs/en/memory | Jun 2026[^4] |
| **Anthropic** | Memory Tool (beta) | platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool | 2025[^33] |
| **Anthropic** | Context Editing (beta) | platform.claude.com/docs/en/build-with-claude/context-editing | Jun 2026[^34] |
| **Anthropic** | Effective Context Engineering (artigo técnico oficial) | anthropic.com/engineering/effective-context-engineering | Set 2025[^1] |
| **Anthropic** | Context Windows | platform.claude.com/docs/en/build-with-claude/context-windows | 2025[^35] |
| **Google / Vertex AI** | Memory Bank (public preview) | cloud.google.com + docs.cloud.google.com/gemini-enterprise-agent-platform | Jul 2025[^12][^13] |
| **Google / Vertex AI** | Memory Bank Quickstart API | cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank | Jun 2025[^36] |
| **Google / Vertex AI** | Context Engineering | cloud.google.com/discover/ai-context-engineering | Abr 2026[^37] |
| **LangChain / LangGraph** | Long-Term Memory Support | langchain.com/blog/launching-long-term-memory | Abr 2026[^2] |
| **LangChain / LangGraph** | Memory Concepts | docs.langchain.com/oss/python/concepts/memory | 2025[^38] |
| **LlamaIndex** | Agent Memory Guide | developers.llamaindex.ai/python/framework/module_guides/deploying/agents/memory/ | 2025[^39] |
| **Zep / Graphiti** | Documentação oficial | help.getzep.com/graphiti | Jun 2026[^40] |

***

## Padrões de Arquitetura Mais Usados por Agentes de Código

### 1. CLAUDE.md + Auto Memory (Minimalista)
Adotado pelo Claude Code. Sem infraestrutura adicional. Arquivos Markdown no repositório carregados em cada sessão. Ideal para times pequenos e projetos individuais.[^4][^1]

### 2. RAG sobre Codebase + ADRs (Intermediate)
Vector store indexado com código, documentação, ADRs e changelogs. O agente recupera contexto relevante via busca semântica antes de cada tarefa. Frameworks: LlamaIndex, LangGraph + Chroma/Qdrant.[^39][^5]

### 3. Letta Three-Tier Memory (OS-Inspired)
Core memory (sempre no context), archival memory (vector store), recall memory (histórico indexado). O agente *decide* quando mover informação entre camadas via chamadas de ferramentas. Melhor para agentes autônomos de longa duração.[^27]

### 4. Graphiti Temporal Knowledge Graph
Entidades e relações com validade temporal. Ideal quando o projeto tem muitas interdependências (módulos, bugs, equipes, decisões) e o agente precisa raciocinar sobre relações, não apenas recuperar fatos isolados.[^28][^22]

### 5. RAG Híbrido + Agentic Search (Production)
Arquitetura multi-agente onde um agente especializado em memória mantém o store, outro realiza a busca híbrida (BM25 + vector + rerank + graph), e o agente principal recebe contexto pré-filtrado. Padrão de produção em 2026.[^41][^21]

### 6. Plataforma Gerenciada (Cloud-First)
Vertex AI Memory Bank, AWS Bedrock AgentCore Memory, ou OpenAI Vector Stores. Zero infraestrutura, consolidação automática, mas lock-in de fornecedor.[^12][^13]

***

## Exemplo de Estrutura de Pasta `.memory/`

```
projeto/
├── .memory/
│   ├── MEMORY.md               ← Índice (≤200 linhas, carregado em toda sessão)
│   ├── architecture.md         ← Padrões e decisões de arquitetura
│   ├── bugs.md                 ← Bugs resolvidos com causa raiz e solução
│   ├── commands.md             ← Comandos de build, test, deploy, debug
│   ├── patterns.md             ← Padrões de código identificados
│   ├── fragile.md              ← Módulos frágeis, débito técnico, TODOs críticos
│   ├── dependencies.md         ← Dependências importantes e versões críticas
│   └── history.md              ← Histórico resumido de implementações relevantes
├── CLAUDE.md                   ← Instruções do projeto (versionado no git)
├── doc/arch/
│   ├── adr-001-autenticacao.md
│   └── adr-002-cache-estrategia.md
└── CHANGELOG.md
```

**MEMORY.md (exemplo de índice):**
```markdown
# Memória do Projeto — Índice
Última atualização: 2026-07-01

## Bugs Resolvidos
- [2026-06-15] Race condition no cache de sessão → `bugs.md#race-condition-cache`
- [2026-05-28] Timeout no módulo de billing → `bugs.md#billing-timeout`

## Decisões Arquiteturais
- [ADR-001] JWT sem refresh token (decisão por simplicidade) → `doc/arch/adr-001.md`
- [ADR-002] Redis para cache de sessão (descartado Memcached) → `doc/arch/adr-002.md`

## Áreas Frágeis
- Módulo `src/billing/` — cobertura de testes < 40% → `fragile.md#billing`
- Integração com API externa X — sem retry logic implementado

## Comandos Importantes
- Build: `npm run build:prod`
- Testes: `npm run test:e2e --env=staging`
- Deploy: `./scripts/deploy.sh --env production`
```

***

## Exemplo de Schema JSON para Eventos de Memória

```json
{
  "memory_event": {
    "id": "mem_2026-07-01_001",
    "created_at": "2026-07-01T14:32:00Z",
    "type": "episodic",
    "subtype": "bug_resolved",
    "confidence": 0.92,
    "ttl_days": 365,
    "valid_from": "2026-07-01",
    "valid_until": null,
    "supersedes": null,
    "source": {
      "session_id": "session_abc123",
      "agent_id": "claude-code",
      "derived_from": "tool_output",
      "trust_level": "high"
    },
    "content": {
      "title": "Race condition resolvida no módulo de cache de sessão",
      "summary": "Bug causava leitura e escrita simultâneas no Redis sem lock, levando a dados corrompidos sob alta concorrência.",
      "cause": "Ausência de lock reentrant na classe SessionCacheManager",
      "solution": "Adicionado threading.RLock() em src/cache/session_cache.py linhas 45-67",
      "files_affected": ["src/cache/session_cache.py"],
      "test_added": "tests/integration/test_session_cache_concurrency.py",
      "related_modules": ["auth", "billing"],
      "tags": ["concurrency", "redis", "session", "bug-fix"]
    },
    "embedding_stored": true,
    "graph_stored": false
  }
}
```

***

## Prompt de Curadoria de Memória

```markdown
Você é um curador de memória de projeto de software. Analise o histórico da sessão abaixo e extraia memórias candidatas.

Para cada memória candidata, produza um objeto JSON seguindo o schema de memory_event.

**REGRAS DE EXTRAÇÃO:**
1. Extraia SOMENTE informações que seriam úteis em sessões futuras
2. NUNCA extraia: segredos, tokens, senhas, dados de usuário, hipóteses não validadas
3. Classifique por tipo: episodic (evento específico), semantic (fato geral), procedural (como fazer algo)
4. Atribua confiança: 0.9+ (confirmado por teste/ferramenta), 0.7-0.9 (raciocínio sólido), <0.7 (hipótese)
5. Se a informação contradiz uma memória existente conhecida, marque como conflicting e não salve automaticamente

**HISTÓRICO DA SESSÃO:**
{session_history}

**MEMÓRIAS EXISTENTES RELEVANTES (para deduplicação):**
{existing_memories}

**SAÍDA ESPERADA:**
Lista de objetos memory_event em JSON, com campo "action": "add" | "update" | "skip" | "conflict"
Para action="update", inclua o ID da memória que está sendo supersedida.
Para action="skip", inclua "reason" explicando por que não salvar.
Para action="conflict", inclua "conflicting_memory_id" e "explanation".
```

***

## Recomendações Práticas por Nível

### Nível Simples (sem infraestrutura adicional)

1. Crie um arquivo `CLAUDE.md` na raiz do projeto com:
   - Comandos de build, test e deploy
   - Arquitetura geral (2-3 parágrafos)
   - Convenções de código que o agente deve seguir
   - Áreas frágeis conhecidas
2. Crie uma pasta `.memory/` com os arquivos de índice conforme o exemplo acima
3. Ao final de cada sessão produtiva, peça ao agente: *"Atualize o MEMORY.md com o que foi descoberto hoje"*
4. Versione `CLAUDE.md` e `.memory/` no git — exceto `CLAUDE.local.md` (pessoal)
5. Adote ADRs para decisões importantes: crie `doc/arch/adr-001.md` com o template de Michael Nygard[^11]

### Nível Intermediário (com vector store local)

1. Implemente os itens do nível simples
2. Configure um vector store local (Chroma é o mais simples para desenvolvimento):
   ```bash
   pip install chromadb langchain-community
   ```
3. Indexe os arquivos `.memory/`, ADRs e código-fonte relevante
4. Implemente um script `memory_curator.py` que:
   - Lê o histórico da sessão comprimido
   - Envia ao LLM com o prompt de curadoria
   - Salva novos eventos como JSON em `.memory/events/`
   - Upserta embeddings no Chroma
5. Configure recuperação: antes de cada nova tarefa, busque top-5 memórias mais relevantes e injete no contexto do sistema
6. Avalie retrieval com LangSmith ou logs simples

### Nível Avançado (produção, multi-sessão, multi-agente)

1. Implemente todos os níveis anteriores
2. Adote Graphiti/Zep para memória relacional com validade temporal[^22]
3. Implemente busca híbrida: BM25 + embeddings + cross-encoder reranker
4. Adicione provenance tracking em cada memory event (schema JSON acima)
5. Implemente trust-scored memory com partições: system (read-only), project, agent
6. Configure TTL e decay automático para memórias de baixa confiança[^18]
7. Monitore behavioral drift: distribição de fontes recuperadas por semana
8. Implemente o memory-curator como serviço async (não bloqueante)
9. Avalie com LongMemEval ou RAGAS[^23]
10. Considere Vertex AI Memory Bank se já usa Google Cloud[^12]

***

## Checklist de Implementação

### Fundação
- [ ] `CLAUDE.md` criado com instruções do projeto
- [ ] `.memory/MEMORY.md` criado como índice de memórias
- [ ] `doc/arch/` criado para ADRs
- [ ] `.gitignore` inclui `CLAUDE.local.md` e qualquer arquivo com secrets

### Curadoria
- [ ] Processo definido: quando atualizar memória (gatilhos)
- [ ] Prompt de curadoria escrito e testado
- [ ] Schema JSON de memory event definido
- [ ] Deduplicação básica implementada
- [ ] Validação anti-secret implementada (regex ou classifier)

### Recuperação
- [ ] Estratégia de retrieval definida (arquivo estático / vector / híbrido)
- [ ] Limite de contexto injetado por sessão definido (token budget)
- [ ] Fallback quando retrieval não encontra nada relevante

### Governança
- [ ] TTL definido por tipo de memória
- [ ] Processo de invalidação de memórias obsoletas
- [ ] Revisão periódica do MEMORY.md agendada
- [ ] Secrets nunca salvos em memória (validado)
- [ ] Provenance de cada memória registrado

### Avaliação
- [ ] Baseline de qualidade estabelecido antes de ativar memória
- [ ] Métricas de retrieval sendo coletadas (precision, recall)
- [ ] Comparação periódica com/sem memória em tarefas representativas

***

## Lacunas, Incertezas e Pontos que Exigem Validação Adicional

1. **Benchmarks independentes são escassos**: A maioria dos números publicados (Mem0 vs. Zep vs. Letta) são auto-reportados pelos próprios desenvolvedores das ferramentas. Um artigo de mai 2026 documenta que muitos benchmarks têm problemas estruturais sérios — benchmarks curtos o suficiente para caber em um context window, avaliadores LLM lenientes, e ausência de avaliação por terceiros. **Recomendação**: avaliar com dados próprios do projeto antes de adotar qualquer ferramenta baseada em marketing de benchmark.[^24]

2. **Consolidação e resolução de conflitos**: Como resolver conflitos entre memórias contraditórias de forma confiável ainda é um problema em aberto. Vertex AI Memory Bank usa Gemini para consolidação, Mem0 usa um LLM interno, mas a qualidade é variável e dependente do modelo.[^13][^20]

3. **Privacidade e compliance**: A maioria das ferramentas de memória não oferece controles granulares de LGPD/GDPR out-of-the-box. Para projetos com dados de usuário, é necessário implementar redaction e políticas de retenção adicionais.[^19]

4. **Limites de fine-tuning combinado com RAG**: A combinação de fine-tuning (para estilo/formato) + RAG (para conhecimento dinâmico) ainda carece de guidelines sólidos sobre como distribuir responsabilidades entre as duas abordagens.[^9]

5. **Segurança de memória em multi-agente**: Em sistemas onde múltiplos agentes compartilham uma memory store, os vetores de ataque são exponencialmente maiores. A pesquisa sobre defesas para cross-agent memory contamination ainda é incipiente.[^18]

6. **Avaliação de memory poisoning em produção**: Ferramentas práticas para detectar memory poisoning em tempo real em sistemas de produção ainda não são mainstream. Os mecanismos de defesa descritos na literatura (perplexity detectors, behavioral drift monitoring) têm falsos positivos significativos.[^25]

7. **Context rot quantificado por modelo**: Embora seja documentado que "context rot" ocorre em todos os modelos com janelas grandes, os dados quantitativos de quão rapidamente ocorre para cada modelo são escassos e evoluem rapidamente com cada versão de modelo.[^1]

***

## Referências Citadas

-  Google Cloud Blog — "Vertex AI Agent Engine Memory Bank in public preview", Jul 2025. cloud.google.com/blog/products/ai-machine-learning/vertex-ai-memory-bank-in-public-preview[^12]
-  mer.vin — "Claude Memory Architecture Explained", Abr 2026[^10]
-  docs.cloud.google.com — "Agent Platform Memory Bank", Google Cloud official docs[^13]
-  cloud.google.com/vertex-ai — "Quickstart with Vertex AI Agent Engine SDK", Jun 2025[^36]
-  platform.claude.com — "Memory tool", Anthropic official docs[^33]
-  openai.github.io — "Memory - OpenAI Agents SDK", OpenAI[^31]
-  cloud.google.com — "What Is AI context engineering?", Abr 2026[^37]
-  langchain.com/blog — "Launching Long-Term Memory Support in LangGraph", Abr 2026[^2]
-  code.claude.com/docs — "How Claude remembers your project", Anthropic Claude Code official docs, Jun 2026[^4]
-  mongodb.com — "Powering Long-Term Memory for Agents With LangGraph", Ago 2025[^3]
-  docs.langchain.com — "Memory overview", LangChain official docs[^38]
-  codex.danielvaughan.com — "Graphiti: Temporal Knowledge Graphs for Agent Memory", Mar 2026[^22]
-  llms3.com — "Zep", Mai 2026[^28]
-  aidigester.org — "Mem0, Open Source Giving Memory to AI Agents ", Fev 2026[^20]
-  youtube.com — "Mem0: The Memory Layer Every AI Agent Needs in 2026", referência ao paper arXiv:2504.19413, Mai 2026[^17]
-  vantaige.io — "Letta: Stateful Agent Memory Framework", Abr 2026[^27]
-  aikickstart.com.au — "Mem0 Review: Agent Memory That Persists (52k Stars)", Jun 2026[^26]
-  help.getzep.com — "Zep Documentation — agent memory at enterprise scale", Jun 2026[^40]
-  dev.to — "RAG with Vector Databases", Jul 2025[^5]
-  cognitect.com — "Documenting Architecture Decisions" (Michael Nygard), 2011[^11]
-  airbyte.com — "RAG vs Fine-Tuning: A Practical Guide for AI Agents", Jun 2026[^42]
-  infinititechpartners.com — "RAG vs Fine-Tuning vs Agents: Choosing the Right LLM Strategy", Jun 2026[^9]
-  arxiv.org/abs/2309.02427 — "Cognitive Architectures for Language Agents (CoALA)", 2023 (ICLR 2024)[^7]
- [^39] developers.llamaindex.ai — "Memory | Developer Documentation - LlamaIndex"
-  llamaindex.ai/blog — "Improved Long & Short-Term Memory for LlamaIndex Agents", Mai 2025[^29]
-  x.com/llama_index — "Memory is a key component of truly agentic RAG", Jun 2024[^30]
-  aclanthology.org/2025.acl-long.1227.pdf — "Unveiling Privacy Risks in LLM Agent Memory", ACL 2025[^19]
-  agentsled.co — "Knowledge Graph vs Vector Database for Agent Memory", Abr 2026[^6]
-  openreview.net — "InjecMEM: Memory Injection Attack on LLM Agent Memory Systems" (em revisão ICLR 2026)[^25]
-  workos.com — "Memory and context poisoning: Don't let attackers rewrite your AI agent's memory", Dez 2025[^18]
-  openai.com — "New tools and features in the Responses API", Mai 2025[^14]
-  platform.claude.com — "Context editing", Anthropic official docs, Jun 2026[^34]
-  anthropic.com/engineering — "Effective context engineering for AI agents", Set 2025[^1]
-  platform.openai.com — "What's new in Assistants API", OpenAI official docs, Mar 2025[^15]
-  platform.claude.com — "Context windows", Anthropic official docs[^35]
-  essays.bloo-mind.ai — "The Benchmark Theatre", Mai 2026[^24]
-  developers.openai.com — "File search", OpenAI official docs[^16]
-  arxiv.org/abs/2410.10813 — "LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory", ICLR 2025[^23]
-  jbinternational.co.uk — "Agentic RAG in 2026", Mai 2026[^21]
-  developers.openai.com/cookbook — "Context Engineering for Personalization", OpenAI Cookbook[^32]
-  tresslersgroup.com — "How AI Agents Remember, Learn, and Stay Coherent", Mai 2026[^41]

---

## References

1. [Effective context engineering for AI agents - Anthropic](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Claude playing Pokémon demonstrates how memory transforms agent capabilities in non-coding domains. ...

2. [Launching Long-Term Memory Support in LangGraph - LangChain](https://www.langchain.com/blog/launching-long-term-memory-support-in-langgraph) - Build AI agents that remember. LangGraph's long-term memory stores and recalls information across co...

3. [Powering Long-Term Memory For Agents With LangGraph And ...](https://www.mongodb.com/company/blog/product-release-announcements/powering-long-term-memory-for-agents-langgraph) - Enhance your AI agents with long-term memory using the new MongoDB Store for LangGraph. Build intell...

4. [Manage Claude's memorydocs.claude.com › docs › claude-code › memory](https://code.claude.com/docs/en/memory)

5. [Retrieval-Augmented Generation (RAG) with Vector ...](https://dev.to/nikhilwagh/retrieval-augmented-generation-rag-with-vector-databases-powering-context-aware-ai-in-2025-4930) - Retrieval-Augmented Generation (RAG) with Vector Databases: Powering Context-Aware AI in 2025 · Conv...

6. [Knowledge Graph vs Vector Database for Agent Memory](https://www.agentsled.co/en/blog/post/knowledge-graph-vs-vector-database) - Vector databases retrieve similar content. Knowledge graphs store structured relationships that pers...

7. [Cognitive Architectures for Language Agents](https://arxiv.org/html/2309.02427v3) - Semantic memory stores facts about the world (Lindes and Laird, 2016) , while episodic memory stores...

8. [Episodic vs Semantic Memory for AI Agents (2026) | SurePrompts](https://sureprompts.com/blog/episodic-vs-semantic-memory-for-agents) - Episodic memory is memory of specific events; semantic memory is memory of general facts; procedural...

9. [RAG vs Fine-Tuning vs Agents: Choosing the Right LLM ...](https://www.infinititechpartners.com/insights/rag-vs-fine-tuning-vs-agents-2026) - How to choose between RAG, fine-tuning, and agents for an enterprise AI system — with a decision tre...

10. [Claude Memory Architecture Explained: How Persistent ...](https://mer.vin/2026/05/claude-memory-architecture-explained-how-persistent-context-makes-coding-agents-reliable/) - Most agent failures in long-running workflows come from context drift, not model weakness. The pract...

11. [Documenting Architecture Decisions - Cognitect.com](https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions) - Context

12. [Vertex AI Memory Bank in public preview](https://cloud.google.com/blog/products/ai-machine-learning/vertex-ai-memory-bank-in-public-preview) - It understands and extracts memories from interactions: Using Gemini models, Memory Bank can analyze...

13. [Agent Platform Memory Bank | Gemini Enterprise ...](https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank) - Persistent and accessible storage: Store memories that can be accessed from multiple environments, i...

14. [New tools and features in the Responses API](https://openai.com/index/new-tools-and-features-in-the-responses-api/) - New features in the Responses API: Remote MCP, image gen, Code Interpreter, and more. Powering faste...

15. [What's new in Assistants API - OpenAI API](https://platform.openai.com/docs/assistants/whats-new/agents) - Discover new features and improvements in the Assistants API, including enhanced tools and model con...

16. [File search | OpenAI API](https://developers.openai.com/api/docs/guides/tools-file-search) - Allow models to search your files for relevant information before generating a response.

17. [Mem0: The Memory Layer Every AI Agent Needs in 2026](https://www.youtube.com/watch?v=6plmP6eH-LY) - Mem0 is the memory layer every AI agent will need in 2026. This paper from Chhikara et al. (arXiv:25...

18. [Memory and context poisoning: Don't let attackers rewrite your AI ...](https://workos.com/blog/ai-agent-memory-poisoning) - Prompt injection ends when the session closes. Memory poisoning persists across sessions, activates ...

19. [[PDF] Unveiling Privacy Risks in LLM Agent Memory - ACL Anthology](https://aclanthology.org/2025.acl-long.1227.pdf)

20. [Mem0, Open Source Giving Memory to AI Agents [2026] - AI Digester](https://aidigester.org/en/mem0-open-source-giving-memory-to-ai-agents-2026-2/) - Mem0: 3 Key Aspects of the AI Agent Memory Layer GitHub Stars: 46,900+ Languages: Python, TypeScript...

21. [Agentic RAG in 2026: How Retrieval-Augmented Generation Has ...](https://www.jbinternational.co.uk/article/view/4681) - Agentic RAG in 2026: How Retrieval-Augmented Generation Has Moved Far Beyond Simple Pipelines

22. [Graphiti: Temporal Knowledge Graphs for Agent Memory](https://codex.danielvaughan.com/2026/03/30/graphiti-agent-memory-store/) - Graphiti is an open-source Python library (Apache 2.0 licensed, maintained by Zep) for building and ...

23. [LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory](https://arxiv.org/abs/2410.10813) - Recent large language model (LLM)-driven chat assistant systems have integrated memory components to...

24. [The Benchmark Theatre: Why Almost Nothing You've Read About ...](https://essays.bloo-mind.ai/posts/2026-05-20-mem-eval/) - The agent long-term memory leaderboard is theatre. I work through the MemPalace controversy, the Mem...

25. [[PDF] InjecMEM: MEMORY INJECTION ATTACK](https://openreview.net/pdf/5f0aedb32147ba6ec36d85f54a53ad381dc22324.pdf)

26. [Mem0 Review: Agent Memory That Persists (52k Stars)](https://aikickstart.com.au/news/mem0-review-2026) - Mem0 Review: Agent Memory That Persists (52k Stars): Mem0 gives AI agents long-term memory.

27. [Letta: Stateful Agent Memory Framework | Vantaige](https://vantaige.io/ai-tool/letta) - Letta (formerly MemGPT) is an open-source platform for building AI agents with persistent memory. Ap...

28. [Zep](https://llms3.com/node/zep) - An open-source AI memory platform (Apache 2.0) built around the **Graphiti** temporal-knowledge-grap...

29. [Improved Long & Short-Term Memory for LlamaIndex Agents](https://www.llamaindex.ai/blog/improved-long-and-short-term-memory-for-llamaindex-agents) - In this article, we will walk through some of the core features of the new LlamaIndex memory compone...

30. [LlamaIndex 🦙 on X: "Memory is a key component of truly agentic RAG. New in LlamaIndex: two memory modules for agents! ⭐️ Vector Memory stores every user message in a vector store of your choice and uses vector search to retrieve relevant messages at every turn. Demo notebook: https://t.co/3E1O0lfKk1" / X](https://x.com/llama_index/status/1799114410985988399)

31. [Memory - OpenAI Agents SDK](https://openai.github.io/openai-agents-python/ref/memory/)

32. [Context Engineering for Personalization - State Management with ...](https://developers.openai.com/cookbook/examples/agents_sdk/context_personalization) - It allows developers to define structured state objects that persist across runs, enabling memory, n...

33. [Memory tool - Claude Docsdocs.claude.com › docs › agents-and-tools › tool-use › memory-tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) - Let Claude store and retrieve information across conversations by implementing the memory tool's fil...

34. [Context editing - Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/context-editing) - Context editing is in beta with support for tool result clearing and thinking block clearing. To ena...

35. [Context windows - Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/context-windows) - Understand how the context window works, how extended thinking and tool use count toward it, and how...

36. [Quickstart with Vertex AI Agent Engine SDK](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/quickstart-api) - Learn how to scale your AI agents in production with Agent Platform, featuring serverless efficiency...

37. [What Is AI context engineering? | Google Cloud](https://cloud.google.com/discover/ai-context-engineering) - Learn about AI context engineering to build effective models. Optimize the AI context window and man...

38. [Memory overview - Docs by LangChain](https://docs.langchain.com/oss/python/concepts/memory) - LangGraph manages short-term memory as a part of your agent's state. State ... For more information ...

39. [Memory | Developer Documentation - LlamaParse - LlamaIndex](https://developers.llamaindex.ai/python/framework/module_guides/deploying/agents/memory/) - Using the Memory class, you can create a memory that has both short-term memory (i.e. a FIFO queue o...

40. [Zep Documentation — agent memory at enterprise scale](https://help.getzep.com/graphiti/getting-started/welcome) - Graphiti is Zep's open-source framework for temporal knowledge graphs — Context Graphs — for AI agen...

41. [How AI Agents Remember, Learn, and Stay Coherent Across ...](https://tresslersgroup.com/insights/agent-memory-systems-rag-knowledge-graphs-2026) - Context windows have expanded from 4K to 1M+ tokens in three years. But context windows alone do not...

42. [RAG vs Fine-Tuning: A Practical Guide for AI Agents - Airbyte](https://airbyte.com/agentic-data/rag-vs-fine-tuning) - Decide between RAG vs fine-tuning based on knowledge gaps, behavior gaps, and production tradeoffs. ...

