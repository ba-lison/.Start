# Working Context vs Memória de Projeto: Análise Conceitual das Camadas de Memória em Agentes de IA

> **Para:** Tech Lead / Engenharia — Engenharia de Suporte Técnico / Operações, Totalcad
> **Escopo:** Análise técnica precisa de 9 conceitos de memória em sistemas de agentes LLM, com exemplos práticos em repositórios reais de software

***

## 1. Por Que a Distinção Importa

Imagine um programador trabalhando em um codebase complexo. Ele tem na mesa (working memory) os arquivos abertos agora. Num caderno ao lado (session memory), as notas que tomou hoje. Num arquivo de equipe no servidor (project knowledge), as convenções do projeto. Na cabeça (long-term memory), tudo que aprendeu ao longo da carreira. E no DNA (fine-tuning), os reflexos que já são instintivos.

Cada camada tem localização, custo, velocidade e risco distintos. Confundi-las é a raiz da maioria das falhas de produção em sistemas agenticos.[^1][^2]

A distinção fundamental é simples: **working context é o que o LLM tem em frente agora** — é RAM. **Memória é o que o agente pode acessar entre chamadas, sessões e tempo** — é storage. A confusão entre os dois é a causa mais comum de agentes que "esquecem tudo quando a sessão termina".[^3][^1]

***

## 2. Os 9 Conceitos: Definições Precisas

### 2.1 Working Context (Contexto de Trabalho Ativo)

**O que é:** O payload completo enviado ao LLM em uma única chamada de API. Inclui o system prompt, mensagens do histórico atual, resultados de tool calls, contexto recuperado via RAG, e qualquer outra informação injetada naquele momento.[^1]

**Analogia:** A mesa de trabalho do programador — tudo que está visível agora. Se não está na mesa, não existe para o LLM neste momento.

**Onde fica:** Exclusivamente na memória DRAM do servidor durante o processamento do token. Não persiste após a resposta ser gerada.[^4]

**Quem atualiza:** O orchestrator/runtime — automaticamente montado antes de cada chamada de inferência. Em LangGraph, é o resultado de compilar o estado do grafo com o histórico de mensagens.[^5]

**Duração:** Uma única inferência. Zera completamente ao final da API call.[^1]

**Como é recuperado:** Não requer recuperação — já está no contexto. É acesso direto O(1).

**Riscos:**
- **Context overflow:** Janelas de 32K–200K tokens parecem grandes, mas encheram rapidamente com tool outputs longos[^1]
- **Context rot:** Informação relevante "se perde" no meio de contextos muito longos — modelos prestam menos atenção ao "miolo"[^3]
- **Custo por token:** Cada token no contexto é cobrado — encher com histórico irrelevante é desperdício direto[^4]

**Quando usar:** Sempre — é o único modo de o LLM ver qualquer informação. A decisão real é *o que injetar* no working context.

**Quando não usar como "solução de memória":** Não substitui memória persistente. Colar toda a história de conversas em cada request é caro, lento e não escala.[^6][^4]

***

### 2.2 Contexto de Sessão (Session Context)

**O que é:** O registro cronológico persistente de uma conversa específica — a sequência de mensagens de um único thread/conversation. É a "transcrição permanente" de uma interação. Difere do working context: a sessão é o log persistente completo; o working context é a fatia selecionada que vai para o LLM.[^7]

**Analogia:** O histórico de chat salvo — você pode rolar para cima e ver tudo, mas só o que cabe na tela (working context) está realmente em foco.

**Onde fica:** Em um backend de persistência por thread. No OpenAI Agents SDK, na classe `Session` (SQLiteSession para produção). No LangGraph, no Checkpointer (InMemorySaver para dev, PostgresSaver para produção). Na Cloudflare Agents, como estado do Durable Object.[^8][^9][^5]

**Quem atualiza:** O runtime automaticamente a cada troca de mensagens. No LangGraph, via `thread_id` no `RunnableConfig`.[^10]

**Duração:** Enquanto o thread existir — pode ser horas, dias ou indefinido dependendo da política de retenção.[^5]

**Como é recuperado:** Por `thread_id`. O checkpointer carrega o último snapshot do estado para continuar a conversa.[^5]

**Riscos:**
- Checkpoints crescem indefinidamente — precisa de política de poda (`pruning`)[^5]
- `MemorySaver` em LangGraph não persiste entre restarts do processo — use PostgresSaver em produção[^5]
- Não escala para histórico de múltiplas sessões — isso requer Long-Term Memory

**Quando usar:** Para conversas multi-turn, human-in-the-loop workflows, recuperação de falhas (fault tolerance), e "time travel" (voltar a um estado anterior).[^5]

**Quando não usar:** Para compartilhar contexto entre threads diferentes ou entre usuários distintos — use Stores (long-term memory).[^5]

***

### 2.3 Memória de Curta Duração (Short-Term Memory)

**O que é:** O conjunto de informações que o agente mantém disponíveis durante o processamento de uma tarefa multi-step — o que está "in context" em um dado momento dentro de uma sessão ativa. Em LangGraph, é o estado do grafo (`AgentState`) — a estrutura que passa entre nós.[^11][^5]

**Analogia:** A memória de trabalho humana — o que você consegue manter "em mente" enquanto resolve um problema. Limitada e efêmera, mas imediata.

**Onde fica:** No estado do grafo LangGraph (checkpointed), no `AgentState` da sessão, ou na `ChatMemoryBuffer` do LlamaIndex. Em Letta, nos memory blocks em contexto.[^12][^13]

**Quem atualiza:** O próprio agente via tool calls e outputs dos nós do grafo. Em LlamaIndex, automaticamente quando o histórico de chat excede o `chat_history_token_ratio`.[^12]

**Duração:** Dentro da sessão/thread ativo. Quando o contexto atinge o limite de tokens, as mensagens mais antigas são "flushed" para long-term memory (se configurado).[^12]

**Como é recuperado:** Diretamente do estado atual — sem necessidade de busca semântica.

**Riscos:**
- Limite de tokens — em LlamaIndex, o padrão é 30K tokens de token_limit total[^12]
- Perda de contexto se não houver long-term memory configurada — mensagens flushed são simplesmente descartadas

**Quando usar:** Para o rastreamento do estado atual de uma tarefa em andamento (subtasks, resultados parciais de tools, raciocínio intermediário).[^1]

***

### 2.4 Memória Longa Duração (Long-Term Memory)

**O que é:** Informações que persistem além de uma única sessão, acessíveis em conversas futuras com o mesmo usuário ou agente. É a solução para o problema fundamental: *agentes que esquecem tudo quando a sessão fecha*.[^14][^6]

**Analogia:** O arquivo pessoal do profissional — documentos, anotações de projetos passados, que ele pode consultar mesmo meses depois.

**Onde fica:**
- **LangGraph:** Store persistente (InMemoryStore para dev, Redis/PostgreSQL para produção) — escopo cross-thread[^5]
- **LlamaIndex:** VectorMemoryBlock (vector DB), FactExtractionMemoryBlock (SQLite por padrão, configurável)[^12]
- **Mem0:** Arquitetura dual-store — vector database (Pinecone, Qdrant) + knowledge graph (opcional, tier Pro)[^15]
- **Letta:** Archival Memory — append-only store com busca semântica on-demand[^13]
- **Zep/Graphiti:** Knowledge graph temporal — cada fato tem timestamps de validade[^16][^17]

**Quem atualiza:**
- LangGraph: código do agente via `store.put()`[^10]
- LlamaIndex: automaticamente quando mensagens são flushed da short-term memory[^12]
- Mem0: chamada explícita `memory.add()` ou hooks automáticos[^18][^19]
- Letta: o próprio agente via memory tools (autonomamente)[^13]

**Duração:** Indefinida — até que uma política de expiração ou deleção explícita remova.[^6]

**Como é recuperado:** Busca semântica (similarity search em embeddings), busca por metadados/filtros, ou graph traversal (Zep).[^16][^15]

**Riscos:**
- **Memória desatualizada (stale memory):** Fatos que eram verdadeiros não são mais — Zep resolve com timestamps de validade[^16]
- **Ruído acumulado:** Armazenar mensagens irrelevantes degrada a qualidade de retrieval[^6]
- **Privacidade:** Dados persistentes criam obrigações de conformidade (GDPR, HIPAA, SOC 2)[^6]
- **Retrieval hallucination:** Busca semântica pode trazer contextos superficialmente similares mas semanticamente errados

**Quando usar:** Para personalização cross-session, preferências do usuário, fatos sobre projetos que o agente precisa lembrar entre conversas.[^19][^6]

***

### 2.5 Memória de Projeto (Project Memory / Project Knowledge)

**O que é:** Conhecimento estruturado específico de um projeto, compartilhado por todos os membros/agentes que trabalham nele. Não é memória do usuário individual — é o contexto da *base de código ou domínio específico*. Pode ser tanto gerenciada por humanos (CLAUDE.md, AGENTS.md) quanto auto-gerada pelo agente (MEMORY.md no Claude Code).[^20]

**Analogia:** A wiki interna da equipe — convenções de código, arquitetura do sistema, runbooks. Todo dev novo lê antes de começar.

**Onde fica:**
- **Claude Code:** `CLAUDE.md` (instruções escritas pelo dev) e `~/.claude/projects/<repo>/memory/MEMORY.md` (notas geradas pelo próprio Claude)[^20]
- **OpenAI Sandbox Agents:** `memories/MEMORY.md` no workspace do sandbox[^21]
- **LangGraph:** Store com namespace de projeto[^5]
- **Anthropic Claude Projects:** Arquivos de conhecimento na interface web do Claude

**Quem atualiza:**
- CLAUDE.md: o desenvolvedor humano (ou o comando `/init` do Claude Code)[^20]
- MEMORY.md: o próprio agente, baseado em correções e preferências observadas[^21][^20]
- Claude Projects: o usuário manualmente

**Duração:** Permanente enquanto o arquivo existir no repositório — tipicamente versionado em Git com o projeto.[^20]

**Como é recuperado:** Carregado integralmente no início de cada sessão (working context injection). CLAUDE.md é injetado como user message após o system prompt. Apenas os primeiros 200 linhas ou 25KB do MEMORY.md são carregados automaticamente — o resto sob demanda.[^20]

**Riscos:**
- Arquivos muito grandes (>200 linhas) consomem tokens e reduzem aderência do modelo[^20]
- Instruções vagas ou contraditórias entre diferentes CLAUDE.md num monorepo[^20]
- Não é enforcement — Claude pode ignorar instruções se vagas; use hooks para regras críticas[^20]

**Quando usar:** Para convenções de código, arquitetura do sistema, comandos de build/test, fluxos de trabalho que qualquer agente (ou dev novo) precisaria saber.[^20]

**Quando não usar:** Para dados sensíveis (credentials, secrets), informações que mudam por usuário (preferências pessoais), ou regras que devem ser *enforçadas* — use settings/hooks para isso.[^20]

***

### 2.6 Memória Durável (Durable Memory / Agent State Persistence)

**O que é:** O estado completo do agente persistido de forma que sobrevive a crashes, restarts de processo e deploys — não é apenas o histórico de chat, mas todo o estado interno do agente (grafo de execução, variáveis de estado, posição no workflow). Em Letta, é a garantia de que *todo* estado (mensagens, tool calls, memory blocks) é persistido em banco de dados e nunca é perdido — mesmo após eviction do contexto.[^10][^13]

**Analogia:** O save game de um videogame — você pode fechar o console, e quando volta, retoma exatamente de onde parou. Diferente do session context, que seria apenas o log do que aconteceu.

**Onde fica:**
- **LangGraph:** PostgresSaver ou SqliteSaver como checkpointer — snapshots do estado do grafo[^5]
- **Letta:** Database server-side (PostgreSQL + pgvector recomendado para produção) — todos os states, mensagens e memory blocks[^22][^13]
- **AWS Bedrock AgentCore:** AgentCoreMemorySaver — persiste checkpoint objects[^10]
- **Cloudflare Agents:** Durable Objects — state do agente no edge[^8]

**Quem atualiza:** O runtime automaticamente após cada step do agente. Em Letta, *cada* operação é persistida — não há risco de perda por eviction.[^13]

**Duração:** Indefinida. Controlada por políticas de retenção e pruning manual.[^5]

**Como é recuperado:** Por `thread_id` + `actor_id` — carregamento direto do último checkpoint.[^10]

**Riscos:**
- `InMemorySaver` (LangGraph) perde tudo no restart — crítico em produção[^5]
- Checkpoints crescem indefinidamente — precisa de política de poda[^5]
- Letta: monitor token spend — cada turn pode triggerar self-directed memory edits[^22]

**Quando usar:** Workflows long-running, agentes com human-in-the-loop (podem ser interrompidos e retomados), sistemas multi-step com recuperação de falhas, e quando o agente precisa ser stateful *garantido*.[^13][^5]

***

### 2.7 RAG (Retrieval-Augmented Generation)

**O que é:** Arquitetura onde o LLM tem acesso a um corpus de documentos externos — em tempo de query, os documentos mais relevantes são recuperados e injetados no working context. O modelo não é alterado; os documentos são o knowledge layer. A distinção com memória de agente: RAG é para corpora estáticos (documentação, políticas, bases de conhecimento); memória de agente é para fatos específicos do usuário/sessão que *evoluem* com as interações.[^23][^24][^6]

**Analogia:** O exame a livro aberto — o estudante não precisa memorizar tudo, pode consultar os materiais durante o exame. Os materiais não mudam (são estáticos no corpus).

**Onde fica:** Vector database (Pinecone, Qdrant, pgvector, Chroma), search index (Elasticsearch), ou hybrid store. O corpus pode ter milhões de documentos — isso não vai para o working context, apenas os top-K chunks recuperados.[^24][^25]

**Quem atualiza:** O time de engenharia, quando documentos são atualizados ou adicionados. A atualização é rápida (minutos) comparada ao fine-tuning.[^23]

**Duração:** Permanente enquanto o documento existir no store. Documentos podem ser atualizados em minutos sem retraining.[^24][^23]

**Como é recuperado:** Pipeline de embedding + similarity search (ANN) → reranking → injection no working context.[^25][^24]

**Riscos:**
- **Retrieval quality cap:** "Retrieval ruim = resposta ruim, independente da qualidade do modelo"[^25]
- **Chunking strategy:** Estratégia incorreta de divisão de documentos é causa frequente de falhas[^25]
- **Latência adicional:** Embedding + vector search + reranking adicionam 200–500ms por query[^26]
- **Contexto fragmentado:** Respostas que requerem síntese de 15+ documentos diferentes são problemáticas[^26]
- RAG não muda o comportamento do modelo — apenas os inputs[^25]

**Quando usar:** Base de conhecimento que muda frequentemente (docs, políticas, catálogos), quando precisar de citações/rastreabilidade da fonte, corpus maior que ~100K tokens, multi-tenant (cada cliente tem dados diferentes).[^27][^25]

**Quando não usar:** Raciocínio que requer síntese de muitos documentos simultâneos, quando latência é crítica (sub-100ms), quando o corpus cabe inteiramente no context window para prototyping.[^25]

***

### 2.8 Project Knowledge (Knowledge Base Estruturada)

**O que é:** Corpus de documentos, especificações, e conhecimento de domínio que define o contexto técnico completo de um projeto — distinto da project memory (que é específica de uma instância do agente) e do RAG genérico (que é para qualquer documento). Project knowledge é o conjunto *curado e autorizado* de artefatos que um agente precisa para trabalhar efetivamente num projeto: ADRs, specs de API, convenções de código, diagramas de arquitetura.[^28][^20]

**Analogia:** O onboarding package de um novo engenheiro — o pacote de documentos que ele *deve* ler antes de fazer seu primeiro commit.

**Onde fica:**
- Em repositórios Git como arquivos Markdown (AGENTS.md, CLAUDE.md, ADRs)[^28][^20]
- Em Claude Projects como "Knowledge files"[^29]
- Em vector stores do RAG pipeline do projeto
- Em wikis internas (Confluence, Notion) indexadas pelo pipeline RAG[^28]

**Quem atualiza:** O time de engenharia — é conhecimento humano-curado. Versionado com o código.[^28][^20]

**Duração:** Permanente — vive no repositório enquanto o projeto existir.

**Como é recuperado:** Injeção direta no working context (se pequeno o suficiente) ou via RAG (se o corpus for grande). Claude Code carrega CLAUDE.md integralmente no início de cada sessão.[^20]

**Riscos:**
- Documentação desatualizada é pior que nenhuma documentação — o agente age com convicção baseado em informação errada
- Necessita manutenção ativa — se o projeto evolui mas o knowledge base não, há drift

**Quando usar:** Para todo projeto não-trivial. Um `CLAUDE.md` bem escrito é a diferença entre um agente que entende a arquitetura e um que faz suposições incorretas consistentemente.[^20]

***

### 2.9 Agent State

**O que é:** A estrutura de dados que encapsula toda a informação necessária para o agente continuar seu trabalho — não apenas o histórico de mensagens, mas variáveis de decisão, subtasks em andamento, resultados parciais de tools, e metadados de execução. Em LangGraph, é tipicamente uma `TypedDict` ou classe Pydantic que flui entre os nós do grafo.[^30][^31]

**Analogia:** O estado de um processo de sistema operacional — inclui não apenas o código sendo executado, mas registradores, stack, heap, file descriptors abertos. É a "foto" completa do processo num momento.

```python
# LangGraph: Agent State típico para um agente de repositório
class RepoAgentState(TypedDict):
    messages: list[BaseMessage]       # histórico de chat (short-term)
    current_task: str                 # tarefa em execução
    files_read: list[str]             # arquivos já lidos nesta sessão
    test_results: dict                # resultados parciais de testes
    pr_draft: Optional[str]           # rascunho de PR em andamento
    user_id: str                      # para recuperar long-term memory
```

**Onde fica:** Na memória do processo (RAM) durante execução. Serializado pelo checkpointer (PostgreSQL, SQLite) para persistência entre steps.[^5]

**Quem atualiza:** Cada nó do grafo retorna um dicionário com as atualizações de estado — LangGraph faz merge automaticamente.[^30][^5]

**Duração:** Dentro da execução do grafo. Persiste entre steps se checkpointer estiver configurado. Sobrevive a restarts se PostgresSaver for usado.[^5]

**Como é recuperado:** Por `thread_id` — LangGraph carrega o último checkpoint e reconstrói o estado.[^5]

**Riscos:**
- Estados muito grandes aumentam custo de serialização e latência de checkpoint
- Subgrafos têm namespaces próprios — dados de subgraph não são visíveis no parent automaticamente[^5]

**Quando usar:** Sempre que construir agentes com LangGraph, Letta, ou frameworks orientados a grafo. É o mecanismo fundamental de passagem de estado entre steps.[^30][^5]

***

### 2.10 Fine-Tuning

**O que é:** O processo de treinar adicionalmente um modelo pré-treinado em um dataset específico, modificando permanentemente os pesos do modelo. O modelo *aprende* novos padrões comportamentais, estilos de output ou raciocínio de domínio — não apenas acessa novos dados.[^32][^24]

**Distinção crítica:** Fine-tuning é para *comportamento*, não para *conhecimento factual*. "Fine-tune para estilo, RAG para fatos". Fine-tuning não é um mecanismo de atualização de conhecimento — um modelo fine-tuned não aprende sobre eventos após o seu training cutoff.[^33][^27]

**Analogia:** Reflexo condicionado vs. consultar um livro. Depois de treinar karatê por anos, os movimentos defensivos são automáticos (fine-tuning). Mas se precisar de uma lei específica, você consulta o código jurídico (RAG).

**Onde fica:** Nos pesos do modelo (parâmetros) — distribuído nos servidores de inferência. Uma vez treinado, o comportamento é intrínseco ao modelo.[^24]

**Quem atualiza:** Time de ML/Data Science. Requer dataset curado (centenas a milhares de exemplos de input/output), pipeline de treinamento, e avaliação. Ciclos trimestais para a maioria dos sistemas de produção.[^27]

**Duração:** Permanente nos pesos — até o próximo ciclo de fine-tuning ou retraining.[^27]

**Como é recuperado:** Não requer recuperação — o comportamento está baked into the model.

**Riscos:**
- **Catastrophic forgetting:** Treinar muito numa tarefa específica degrada capacidades gerais[^26]
- **Conhecimento estático:** Não atualiza com novos eventos — dados de fine-tuning são snapshot no tempo[^27]
- **Custo:** $2K–10K por training run; ciclos de retraining para dados que mudam[^26]
- **Sem rastreabilidade:** Modelo fine-tuned não cita fontes — maior risco de alucinação confiante[^23]
- **Disponibilidade limitada:** Não disponível para todos os modelos[^23]

**Quando usar:**
- Output em formato ou estilo consistente e específico (JSON schemas, language register, tom de marca)[^27]
- Tarefas narrow e estruturadas em alto volume (classificação, NER, extração)[^27]
- Redução de custo em escala extrema — fine-tune de modelo menor (Llama 8B) para task específica pode reduzir custo em 60–80%[^27]

**Quando não usar:**
- Conhecimento que muda frequentemente — use RAG[^27]
- Quando o dataset de treinamento não está disponível (precisa de centenas de exemplos curados)[^27]
- Como primeiro passo — comece com RAG, fine-tune apenas quando necessário[^27]

***

## 3. Tabela Comparativa

| Conceito | Localização | Quem Atualiza | Duração | Como Recupera | Riscos Principais | Usar Quando | Não Usar Quando |
|---|---|---|---|---|---|---|---|
| **Working Context** | DRAM do servidor (inference) | Orchestrator/runtime | Uma inferência | Acesso direto (já está lá) | Context overflow, context rot, custo por token | Sempre — é o único canal de percepção do LLM | Como substituto de memória persistente |
| **Session Context** | Checkpointer (SQLite/Postgres) | Runtime (auto, por thread_id) | Duração do thread | Carregar checkpoint por thread_id | Crescimento ilimitado, perde-se sem backend persistente | Multi-turn, human-in-loop, recuperação de falhas | Compartilhar contexto cross-thread |
| **Short-Term Memory** | AgentState in-graph / ChatBuffer | Agente (tool outputs, nodes) | Dentro da sessão ativa | Via estado do grafo — sem busca | Limite de tokens, perda sem long-term config | Estado da tarefa em andamento, subtasks | Contexto que deve sobreviver além da sessão |
| **Long-Term Memory** | Vector DB + Graph (Mem0, Zep) | Agente ou hooks de post-session | Indefinida (expiração por policy) | Busca semântica / graph traversal | Stale memory, noise acumulado, GDPR | Preferências cross-session, fatos de usuário | Informações que não precisam sobreviver entre sessões |
| **Project Memory** | CLAUDE.md / MEMORY.md (Git/filesystem) | Dev (CLAUDE.md) / Agente (MEMORY.md) | Permanente (enquanto arquivo existir) | Carregamento completo no início de sessão | Instrução vaga ignorada, overflow de tokens | Convenções, arquitetura, comandos de build | Segredos, dados de usuário individual |
| **Durable Memory** | PostgreSQL/pgvector (server-side) | Runtime (cada step auto-persistido) | Indefinida | Por actor_id + thread_id | InMemorySaver perde em restart, custo de serialização | Workflows long-running, fault tolerance | Prototyping / dev local simples |
| **RAG** | Vector store (Pinecone, pgvector) | Equipe de engenharia (corpus update) | Permanente, atualização rápida | Embedding + ANN search + injection | Retrieval quality cap, latência, chunking ruim | Conhecimento que muda, corpora grandes, citabilidade | Síntese de muitos docs simultâneos, latência crítica |
| **Project Knowledge** | Git repo (Markdown), Claude Projects | Time de engenharia (curado) | Permanente no repo | Injeção direta ou RAG | Drift se não mantido, falsa confiança com dados velhos | Todo projeto não-trivial com agente | Conhecimento efêmero ou específico de usuário |
| **Agent State** | RAM → Checkpointer (Postgres/SQLite) | Nós do grafo (auto-merge) | Entre steps; persiste se checkpointed | Por thread_id via checkpointer | Subgraph namespace isolation, tamanho do checkpoint | Toda arquitetura LangGraph/Letta | Necessidades simples de stateless chat |
| **Fine-Tuning** | Pesos do modelo (parâmetros) | Time de ML (ciclos de treinamento) | Permanente nos pesos | Não requer recuperação (intrínseco) | Catastrophic forgetting, dados estáticos, custo alto | Estilo/formato consistente, tarefas narrow em volume | Conhecimento dinâmico, primeiro passo de um projeto |

***

## 4. Hierarquia de Tempo de Vida

```
Fine-tuning (pesos)          ← Permanente, muda raramente
Project Knowledge (Git)      ← Permanente, versionado
Long-Term Memory (DB)        ← Persistente, expiração por policy
Durable Memory / Agent State ← Persistente, por thread
Session Context              ← Duração do thread
Short-Term Memory            ← Dentro da sessão ativa
Working Context              ← Uma única inferência
```

***

## 5. Exemplo Prático: Agente Trabalhando num Repositório Real

Imagine um agente de IA (construído com LangGraph + Claude + Mem0) trabalhando no repositório `totalcad/zwcad-plugin-api`. Veja como cada camada entra em cena:

### Cenário: "Agente recebe issue #42: 'Falha ao ativar licença offline no ZWCAD 2025'"

**1. Project Knowledge (antes de iniciar)**
```
CLAUDE.md do repositório contém:
- Arquitetura da API de licenciamento
- Endpoints do servidor de licença
- Convenções de código (TypeScript strict, Jest para testes)
- Fluxo de ativação offline (passos 1-7)
```
O agente lê isso ao iniciar a sessão — é injetado no working context como contexto de projeto.[^20]

**2. Working Context (durante a sessão)**
```
System prompt + CLAUDE.md + mensagens recentes + resultados de tool calls
= ~15K tokens total no momento da inferência
```
O agente vê: o issue, os arquivos abertos, os últimos 5 resultados de tool calls.[^1]

**3. Agent State (tracking do progresso)**
```python
state = {
    "current_issue": "#42",
    "files_read": ["src/license/offline.ts", "tests/license.test.ts"],
    "hypothesis": "Token validation failing on air-gapped networks",
    "tests_run": [],
    "pr_draft": None
}
```
Cada step do grafo atualiza e persiste esse estado via checkpointer.[^5]

**4. Session Context (histórico da conversa)**
```
Thread: issue-42-investigation
Messages: [
  user: "Investigate issue #42",
  assistant: "I'll start by reading the offline activation module...",
  tool_call: read_file("src/license/offline.ts"),
  tool_result: "teúdo do arquivo>",
  ...
]
```
Persiste via PostgresSaver — se o agente for interrompido, retoma exatamente aqui.[^5]

**5. Long-Term Memory via Mem0 (conhecimento cross-sessão)**
```python
# Agente busca memórias relevantes antes de iniciar
memories = mem0.search(
    query="ZWCAD offline license activation",
    user_id="agent-zwcad-support"
)
# Resultado: "Em março/2025, issue similar #31 foi resolvido 
#  desabilitando certificate pinning na rede corporativa"
```
Esse insight de sessão anterior é injetado no working context.[^6]

**6. RAG (documentação técnica)**
```python
# Agente consulta documentação técnica durante a investigação
docs = rag_retriever.search(
    "ZWCAD 2025 offline activation certificate validation"
)
# Retorna chunks relevantes da documentação oficial ZWCAD
# + notas de release do ZWCAD 2025
```
A documentação (corpus) tem milhares de páginas — apenas os chunks relevantes entram no working context.[^24]

**7. Project Memory (aprendizado persistido)**
```
MEMORY.md (auto-gerado pelo agente em sessões anteriores):
## License Issues
- Offline activation requires valid NTP sync even on air-gapped networks
- ZWCAD 2025 introduced new certificate pinning - see issue #31
- Test with: zwcad-license-test --offline --verbose
```
Este arquivo foi escrito pelo agente em sessões anteriores e é carregado no início de toda nova sessão neste repositório.[^20]

**8. Fine-Tuning (não aplicável aqui)**
O modelo base (Claude Sonnet) não foi fine-tuned para este caso. Se o volume de issues fosse muito alto e houvesse padrões muito específicos de resposta (ex: formato exato de PR description), seria candidato a fine-tuning. Mas RAG + project knowledge é suficiente aqui.[^25][^27]

***

## 6. Frameworks e Implementações de Referência

### LangGraph (LangChain)
- **Short-term:** AgentState + InMemorySaver[^5]
- **Long-term:** Stores (InMemoryStore → Redis/PostgreSQL em produção)[^5]
- **Documentação:** `checkpointers` para thread-scoped, `stores` para cross-thread[^5]

### Letta (MemGPT)
Três camadas de memória no agente:
- **Core Memory:** Blocos sempre em contexto (persona, human) — editáveis pelo próprio agente[^13]
- **Archival Memory:** Store append-only com busca semântica — ilimitado[^13]
- **Recall Memory:** Histórico de mensagens — pesquisável[^34]
Todo estado persiste server-side — agentes têm `agent_id` estável.[^22]

### Mem0
Arquitetura dual-store: vector DB (semantic search) + knowledge graph (entity relationships, tier Pro). ECAI 2025: primeiro benchmark head-to-head de 10 abordagens de memória.[^35][^15]

### Zep / Graphiti
Knowledge graph temporal — cada fato tem timestamps de validade (T e T'). Supera MemGPT no DMR benchmark (94.8% vs 93.4%) e melhora 18.5% no LongMemEval. Ideal para dados que evoluem: "o usuário mudou de emprego", "a API foi descontinuada".[^17][^16]

### LlamaIndex
`Memory` class com `StaticMemoryBlock` (sempre em contexto), `FactExtractionMemoryBlock` (extrai fatos de mensagens flushed), e `VectorMemoryBlock` (embeddings de histórico). Default: SQLite para persistência.[^12]

### OpenAI Agents SDK
- **Session:** `SQLiteSession` para conversational history[^9]
- **Sandbox Memory:** `MEMORY.md` no workspace — gerado em 2 fases (conversation extraction + layout consolidation)[^21]
- **Multi-agent isolation:** Layouts separados por agente[^21]

***

## 7. Guia de Decisão: Qual Usar Quando?

### Problema → Solução

| Sintoma / Problema | Solução |
|---|---|
| "O agente esquece o que discutimos na semana passada" | Long-Term Memory (Mem0, Zep, LangGraph Store) |
| "O agente não conhece as convenções do projeto" | Project Knowledge (CLAUDE.md, AGENTS.md) |
| "O agente perde o raciocínio a cada step" | Agent State + Checkpointer |
| "O agente precisa consultar 10.000 páginas de documentação" | RAG |
| "O agente responde no formato errado consistentemente" | Fine-Tuning (último recurso) |
| "O agente esquece o início da conversa longa" | Short-term flush → Long-term memory blocks |
| "O agente não recupera bem de crashes" | Durable Memory (PostgresSaver) |
| "O agente não sabe que fizemos um hotfix ontem" | RAG com corpus atualizado (minutos de delay) |
| "Quero que o agente aprenda minhas preferências ao longo do tempo" | Long-Term Memory + Episodic/Semantic memory types |
| "O agente em produção perde estado entre deploys" | InMemorySaver → PostgresSaver imediatamente |

***

## 8. Riscos Sistêmicos e Anti-Padrões

### Anti-padrão #1: "Context window como memória"
Colocar toda a história de conversas em cada request. Funciona para prototyping, mas não escala: custo quadrático de tokens, latência crescente, e o modelo presta menos atenção no "miolo" de contextos longos.[^4][^3]

### Anti-padrão #2: "Fine-tuning como knowledge update"
Fazer fine-tuning para "ensinar o modelo sobre nossa empresa" e depois perguntar coisas que mudam frequentemente (estoque, tickets recentes). Fine-tuning é snapshot no tempo — use RAG para isso.[^26][^27]

### Anti-padrão #3: "MemorySaver em produção"
LangGraph `InMemorySaver` e `MemorySaver` são para desenvolvimento. Em produção, qualquer restart ou deploy destrói todo o estado dos agentes. Use `PostgresSaver` ou `SqliteSaver`.[^5]

### Anti-padrão #4: "RAG como solução de memória de usuário"
RAG é para corpora estáticos. Preferências do usuário, histórico de sessões, e contexto pessoal evoluem a cada interação — precisam de Long-Term Memory com update dinâmico (Mem0, Zep).[^6]

### Anti-padrão #5: "CLAUDE.md como enforcer"
Instruções no CLAUDE.md são contexto, não regras enforçadas. Para comportamentos críticos (nunca commitar em main, sempre rodar testes), use hooks (PreToolUse) — não CLAUDE.md.[^20]

***

## 9. Custo e Complexidade de Implementação (2026)

| Abordagem | Custo de Build | Custo Operacional/mês | Complexidade | Time-to-Ship |
|---|---|---|---|---|
| RAG básico | $15–40K engenharia | $500–1.500 (infra) | Média | Semanas[^27] |
| Long-term memory (Mem0 cloud) | Baixo (SDK) | $19–249/mês + API | Baixa | Dias |
| Long-term memory (self-hosted) | Média | Custo de infra próprio | Alta | Semanas |
| LangGraph + PostgreSQL | Média | PostgreSQL + compute | Média | Dias–semanas |
| Letta server | Baixa (Docker) | PostgreSQL + compute | Baixa | Dias |
| Fine-tuning (run único) | $20–60K data prep | $5–15K/ciclo retraining | Alta | Meses[^27] |

***

## 10. Conclusão Conceitual

A hierarquia de memória em agentes de IA é fundamentalmente análoga à hierarquia de memória em sistemas de computação clássicos: há trade-offs invariáveis entre velocidade de acesso, capacidade, custo e persistência.

**Working context é RAM** — imediato, caro por byte, efêmero. **Session context é cache L1/L2** — rápido, thread-local. **Long-term memory é disco** — lento para busca, mas quase ilimitado e persistente. **Fine-tuning é o próprio processador** — o comportamento está gravado no hardware, não precisa ser recuperado, mas é difícil e caro de atualizar.

A armadilha mais comum é tratar o contexto como memória e a memória como contexto. Sistemas de produção robustos usam as duas camadas em conjunto: **working context para raciocínio imediato**, **session context para continuidade da conversa**, **RAG para conhecimento de domínio**, **long-term memory para personalização**, e **fine-tuning apenas quando o comportamento precisa ser baked in** — não os fatos.[^26][^1][^25]

Para um agente atuando em repositório real de software (como o cenário Totalcad acima), a arquitetura mais pragmática em 2026 é: LangGraph com PostgresSaver (durable state) + Mem0 ou Zep (long-term memory cross-session) + RAG sobre documentação técnica + CLAUDE.md bem mantido (project knowledge). Fine-tuning só se o volume justificar e o comportamento for estável.[^27]

---

## References

1. [Building Agent Memory Systems: Short-Term Context vs Long-Term ...](https://www.mikul.me/blog/agent-memory-systems-short-term-long-term) - How to design memory architecture for AI agents that need to maintain context across tool calls, ses...

2. [The New Reality of Agent Memory: The Complete Guide (2026)](https://www.sitepoint.com/ai-agent-memory-guide/) - Learn how AI agent memory works, the 5 failure modes that break production agents, and practical fix...

3. [Context vs Memory Explained: The Architecture Every AI Agent Developer Must Know](https://www.youtube.com/watch?v=gf4DmGTgaSk) - Most AI agent developers are treating context and memory as the same thing. They're not — and this c...

4. [Memory vs Context Window for LLM and AI Agents](https://mem0.ai/blog/context-window-is-ram-not-storage-why-most-agent-failures-happen-how-to-fix-them-in-2026) - Working Memory: RAM Layer. Working memory holds everything the agent needs to complete the current t...

5. [Persistence - Docs by LangChain](https://docs.langchain.com/oss/python/langgraph/persistence)

6. [The AI Memory Layer: What It Is, How It Works and Why Agents ...](https://mem0.ai/blog/ai-memory-layer-guide) - The AI memory layer gives LLMs and autonomous agents persistent context across sessions. Learn how i...

7. [Basics of Context Engineering: Session and Memory - Momo](https://www.usemomo.com/insights/context-engineering-basics) - Context is the carefully selected information payload sent to the LLM. Managing session history for ...

8. [Loadable Context (skills)](https://developers.cloudflare.com/agents/concepts/conversation-state-and-memory/) - How agents store and recall information, including read-only context, writable short-form memory, se...

9. [OpenAI Agents SDK その1 Memory #OpenAIAgentsSDK - Qiita](https://qiita.com/TsuyoshiUshio@github/items/a54b8bef8c6d0f950862) - 自分の学びのために、小さなブログを書くことにしました。Open AI Agents SDK を理解してみようと思います。 初回は、Memory の機能に関して、書いてみます。今回はデフォルトの短期記憶...

10. [Integrate AgentCore Memory with LangChain or LangGraph](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-integrate-lang.html) - LangChain and LangGraph are powerful open-source frameworks for developing agents through a graph-ba...

11. [Add and manage memory - Docs by LangChain](https://docs.langchain.com/oss/python/add-memory)

12. [Memory | Developer Documentation - LlamaParse - LlamaIndex](https://developers.llamaindex.ai/python/framework/module_guides/deploying/agents/memory/)

13. [Introduction to Stateful Agents | Letta Docs](https://docs.letta.com/guides/core-concepts/stateful-agents/) - Important “core” memories are injected into the context window of the LLM, and the agent can modify ...

14. [Agent Memory System Design: From Session to Long-Term Memory ...](https://eastondev.com/blog/en/posts/ai/agent-memory-system/) - Building an Agent memory system from scratch: Four memory types selection, five-stage pipeline imple...

15. [Mem0 vs Zep (Graphiti): AI Agent Memory Compared (2026)](https://vectorize.io/articles/mem0-vs-zep) - Zep takes a graph-first approach. Its engine, Graphiti, builds a temporal knowledge graph where ever...

16. [Zep: A Temporal Knowledge Graph Architecture for Agent Memory](https://arxiv.org/abs/2501.13956) - Abstract:We introduce Zep, a novel memory layer service for AI agents that outperforms the current s...

17. [getzep/graphiti: Build Real-Time Knowledge Graphs for AI Agents](https://github.com/getzep/graphiti) - Graphiti is a framework for building and querying temporal context graphs for AI agents. Unlike stat...

18. [GitHub - mem0ai/mem0: Universal memory layer for AI Agents](https://github.com/mem0ai/mem0) - Mem0 ("mem-zero") enhances AI assistants and agents with an intelligent memory layer, enabling perso...

19. [Mem0: Long-Term Memory and Personalization for Agents](https://microsoft.github.io/autogen/0.2/docs/ecosystem/mem0/) - Mem0 Platform provides a smart, self-improving memory layer for Large Language Models (LLMs), enabli...

20. [Manage Claude's memorydocs.claude.com › docs › claude-code › memory](https://code.claude.com/docs/en/memory)

21. [Agent memory - OpenAI Agents SDK](https://openai.github.io/openai-agents-python/sandbox/memory/)

22. [Letta (MemGPT) Stateful Agent With Persistent Memory | OSS AI Hub Code Starters](https://ossaihub.com/code/letta-stateful-agent-server/) - Run agents server-side with Letta so their memory survives process restarts: editable persona/human ...

23. [Comparing Retrieval Augmented Generation and fine-tuning](https://docs.aws.amazon.com/prescriptive-guidance/latest/retrieval-augmented-generation-options/rag-vs-fine-tuning.html) - Learn about the advantages and disadvantages of Retrieval Augmented Generation and fine-tuning and l...

24. [Retrieval-Augmented Generation vs Fine-Tuning: What's Right for ...](https://www.k2view.com/blog/retrieval-augmented-generation-vs-fine-tuning/) - When an LLM doesn't meet your expectations, you can optimize it using retrieval-augmented generation...

25. [RAG vs Fine-Tuning vs Agents | When to Use Each (2026)](https://aisoftwaredev.io/learn/rag-vs-fine-tuning-vs-agents) - Compare RAG, fine-tuning, and agentic AI. When each works, costs, tradeoffs, and how to combine them...

26. [RAG vs Fine-Tuning — What Actually Works in Production ( ...](https://dev.to/tyson_cung/rag-vs-fine-tuning-what-actually-works-in-production-2026-20jg) - I've spent the last year building AI systems that actually serve real users — not demos, not proofs....

27. [RAG vs Fine-Tuning vs Agents: Choosing the Right LLM ...](https://www.infinititechpartners.com/insights/rag-vs-fine-tuning-vs-agents-2026) - How to choose between RAG, fine-tuning, and agents for an enterprise AI system — with a decision tre...

28. [Set up a Git knowledge repository that every AI agent reads](https://www.productmap.io/blog/git-product-knowledge-repository-ai-agents) - How a plain-text GitHub repository becomes the shared context layer for every AI agent on your team,...

29. [Why Doesn't Claude Remember You? (And How to Fix It in ...](https://fostera.ai/blog/why-claude-forgets-you) - Claude's 200K context window is huge — and still session-scoped. The architectural reason Claude for...

30. [Building Context-Aware Agents with LangGraph](https://dev.to/richard_abishai/building-context-aware-agents-with-langgraph-c2a) - How to add memory, state, and long-term reasoning to LangGraph agents. Most AI agents behave like...

31. [LangGraph Tutorial: Understanding Short-Term Memory with Thread ID | React Agent Example](https://www.youtube.com/watch?v=CvuHHrgOr9Y&vl=es) - In this video, we dive into how short-term memory works in LangGraph using the thread_id concept. We...

32. [RAG vs. Fine-tuning - IBM](https://www.ibm.com/think/topics/rag-vs-fine-tuning) - RAG uses an organization's internal data to augment prompt engineering, while fine-tuning retrains a...

33. [RAG vs Fine-Tuning vs Long Context — Where Should Your AI's Knowledge Live?](https://www.youtube.com/watch?v=rDfWRh-EmYY) - "Should we use RAG, or fine-tune the model?" It's the question every AI team fights about — and in 2...

34. [From Letta (MemGPT)](https://sonz.ai/docs/en/guides/migrating/letta) - Migrate core memory blocks and archival memory from Letta (formerly MemGPT) agents into Sonzai. Huma...

35. [AI Agent Memory 2026: Progress Benchmark Report Evaluations](https://mem0.ai/blog/state-of-ai-agent-memory-2026) - The Mem0 research paper published at ECAI 2025 (arXiv:2504.19413) established the first broad head-t...

