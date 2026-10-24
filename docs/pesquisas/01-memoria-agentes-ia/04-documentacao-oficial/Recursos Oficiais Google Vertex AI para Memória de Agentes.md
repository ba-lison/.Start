# Recursos Oficiais Google/Gemini/Vertex AI para Memória de Agentes

> **Fontes utilizadas:** Documentação oficial do Google Cloud, Vertex AI, Agent Development Kit (ADK), Gemini Enterprise Agent Platform e whitepapers de segurança da Google. Acesso: julho de 2026.

***

## Sumário Executivo

O ecossistema Google Cloud oferece um conjunto estruturado e oficial de recursos para gerenciamento de memória em agentes de IA. Os recursos cobrem desde contexto de curta duração dentro de uma sessão (`Sessions`) até memória persistente de longo prazo (`Memory Bank`), passando por cache de contexto de tokens (`Context Caching`), recuperação aumentada por vetores (`RAG Engine`) e fundamentação em fontes externas (`Grounding`). Nem tudo é produto gerenciado — há uma divisão clara entre o que a Google provisiona e o que o desenvolvedor precisa arquitetar.

***

## 1. Quais Recursos Oficiais Existem para Memória de Agentes

A Google organiza os recursos de memória em quatro camadas complementares, todas integráveis ao ADK:[^1][^2]

### 1.1 Vertex AI Agent Engine — Sessions (Memória de Curto Prazo)

As **Sessions** mantêm o histórico de interações entre usuário e agente dentro de uma única conversa. Cada sessão contém:[^3]

- **Session**: sequência cronológica de eventos (mensagens, chamadas de ferramenta, respostas do agente)
- **Event**: unidade de armazenamento de conteúdo e ações
- **State**: dados temporários válidos apenas durante a sessão corrente
- **Memory**: informações persistentes acessíveis em múltiplas sessões para um usuário[^4]

As Sessions suportam três implementações no ADK:[^5]

| Implementação | Persistência | Caso de Uso |
|---|---|---|
| `InMemorySessionService` | Nenhuma (perdida ao reiniciar) | Desenvolvimento local, protótipos |
| `DatabaseSessionService` | Sim (PostgreSQL, MySQL, SQLite) | Produção autogerenciada |
| `VertexAiSessionService` | Sim (via Vertex AI Agent Engine) | Produção em Google Cloud |

### 1.2 Vertex AI Memory Bank (Memória de Longo Prazo) — Produto Gerenciado

O **Memory Bank** é o produto principal para memória persistente e cross-sessão. Permite que o agente lembre informações de conversas passadas e personalize respostas futuras. Suas capacidades incluem:[^6][^7]

- Extração LLM-based de fatos relevantes do histórico de conversas
- Armazenamento assíncrono e remoto de memórias (o agente não bloqueia aguardando)
- Recuperação por similaridade semântica com escopo por identidade (`user_id`)
- Integração nativa com ADK via `VertexAiMemoryBankService`[^8][^2]

As três implementações de `MemoryService` no ADK são:[^2]

| Feature | `InMemoryMemoryService` | `VertexAiMemoryBankService` | `VertexAiRagMemoryService` |
|---|---|---|---|
| **Persistência** | Nenhuma | Sim (Agent Platform gerenciado) | Sim (Knowledge Engine) |
| **Extração** | Armazena conversas completas | LLM extrai fatos relevantes e consolida | Armazena transcrição completa |
| **Busca** | Keyword matching básico | Busca semântica avançada | Busca por similaridade vetorial |
| **Complexidade de Setup** | Nenhuma | Baixa (Agent Runtime instance) | Média (Knowledge Engine) |
| **Melhor para** | Protótipos locais | Produção com aprendizado contínuo | RAG sobre corpus de conversas |

### 1.3 Vertex AI Context Caching (Cache de Tokens)

O **Context Caching** reduz custo e latência ao reutilizar tokens de contexto repetitivos. Não é memória de agente no sentido semântico — é otimização de inferência para conteúdo fixo e extenso.[^9][^10]

Dois tipos:[^10]

- **Implicit Caching**: habilitado por padrão para todos os projetos GCP. Desconto automático de 90% nos tokens cacheados. Retenção por até 24 horas
- **Explicit Caching**: o desenvolvedor declara o conteúdo a cachear. Desconto garantido de 90% (Gemini 2.5+) ou 75% (Gemini 2.0). TTL configurável, sem prazo máximo. Suporta CMEK

Limites relevantes:[^10]

- Mínimo: 2.048 tokens
- Máximo do conteúdo via blob/text: 10 MB
- Tempo mínimo antes de expirar: 1 minuto

**Ideal para agentes de software:** cache de codebase, instruções de sistema extensas, repositórios de documentação interna.[^9]

### 1.4 Vertex AI RAG Engine (Recuperação Aumentada por Geração)

O **RAG Engine** é um framework gerenciado para aplicações que precisam consultar bases de conhecimento privadas. O pipeline inclui:[^11][^12]

1. **Ingestão**: arquivos locais, Cloud Storage, Google Drive
2. **Transformação**: chunking de documentos
3. **Embedding**: vetorização semântica
4. **Indexação**: criação de corpus (índice vetorial)
5. **Retrieval**: busca por relevância semântica ao receber uma query
6. **Generation**: contexto recuperado é injetado no prompt do LLM

No ADK, o `VertexAiRagMemoryService` usa o Knowledge Engine para armazenar e recuperar transcrições de conversas por similaridade vetorial. Há também a `vertex_ai_rag_retrieval` tool para recuperação de dados privados via Knowledge Engine diretamente de dentro de um agente.[^13][^2]

### 1.5 Grounding (Fundamentação em Fontes Externas)

O **Grounding** ancora as respostas do modelo em fontes verificáveis, reduzindo alucinações:[^14]

- **Grounding com Google Search**: respostas fundamentadas em resultados de busca em tempo real[^15]
- **Grounding com Vertex AI Search**: respostas fundamentadas em datastores privados configurados pelo desenvolvedor[^14]
- **Dynamic Retrieval**: o modelo decide automaticamente quando acionar o grounding com base em um limiar de confiança configurável (ex: `threshold=0.7`)[^16]

***

## 2. Documentação Oficial sobre Memory Bank e Memória Persistente

A Google mantém documentação oficial completa para o Memory Bank:[^17][^7][^18]

- **Overview**: `cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/overview`[^19]
- **Quickstart com ADK**: `cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/quickstart-adk`[^20]
- **Quickstart com Vertex AI SDK**: `cloud.google.com/agent-builder/agent-engine/memory-bank/quickstart-api`[^21]
- **Setup e Configuração**: `docs.cloud.google.com/agent-builder/agent-engine/memory-bank/set-up`[^18]
- **Gerar memórias**: `docs.cloud.google.com/agent-builder/agent-engine/memory-bank/generate-memories`[^22]
- **Memory Revisions**: `docs.cloud.google.com/agent-builder/agent-engine/memory-bank/revisions`[^23]
- **ADK Memory Reference**: `adk.dev/sessions/memory/`[^2]
- **Codelab oficial**: "Building Stateful and Personalized Agents with ADK" (Google Developers Codelabs)[^24]

A documentação em português está disponível em: `docs.cloud.google.com/agent-builder/agent-engine/memory-bank/overview?hl=pt`[^25]

***

## 3. Como a Google Recomenda Armazenar, Recuperar, Consolidar e Expirar Memórias

### 3.1 Fluxo Oficial de Armazenamento e Geração

O ciclo de vida de uma memória no Memory Bank segue estas etapas conforme documentado:[^6]

1. **`CreateSession`**: nova sessão criada no início de cada conversa com um `user_id` obrigatório
2. **`AppendEvent`**: eventos (mensagens do usuário, respostas do agente, chamadas de ferramentas) são adicionados ao histórico da sessão
3. **`ListEvents`**: o agente recupera o histórico da sessão corrente
4. **`GenerateMemories`**: ao final de uma sessão (ou em intervalos), o agente dispara a extração automática de fatos via LLM — operação assíncrona
5. **`CreateMemory`**: criação direta e controlada de uma memória específica pelo agente (útil quando o agente precisa de controle granular)
6. **`RetrieveMemories`**: recuperação das memórias relevantes para um usuário — por recuperação simples (todas as memórias) ou por busca semântica por similaridade

### 3.2 Ferramentas de Recuperação no ADK

O ADK disponibiliza duas ferramentas built-in:[^2]

- **`PreloadMemoryTool`**: carrega memórias automaticamente no início de cada turno (equivalente a um callback)
- **`LoadMemoryTool`**: o agente decide quando recuperar memórias (on-demand, baseado em raciocínio)

Também é possível acessar memória de dentro de custom tools via `tool_context.search_memory(query)`.[^2]

### 3.3 Consolidação de Memórias

O Memory Bank utiliza o Gemini (por padrão `gemini-2.5-flash`) para:[^18]
- **Extrair** fatos relevantes do histórico de conversas
- **Consolidar** novas memórias com memórias existentes, evitando duplicatas e atualizando informações desatualizadas

### 3.4 Expiração (TTL — Time-to-Live)

A Google lançou suporte oficial a TTL em 2025. Dois modos:[^26][^18]

- **Default TTL**: aplicado a todas as operações (`UpdateMemory`, `CreateMemory`, `GenerateMemories`)
- **Granular TTL**: TTL diferente por operação (`create_ttl`, `generate_created_ttl`, `generate_updated_ttl`)

Memórias sem TTL configurado nunca expiram automaticamente. O desenvolvedor pode deletar memórias manualmente ou configurar o TTL para garantir freshness dos dados.[^18]

### 3.5 Memory Profiles (Customização de Extração)

A Google disponibilizou configuração de **Memory Topics** para controlar o que é considerado relevante para persistir:[^26][^18]

**Tópicos gerenciados (managed topics)**:
- `USER_PERSONAL_INFO`: informações pessoais (nome, relacionamentos, hobbies, datas)
- `USER_PREFERENCES`: preferências explícitas ou inferidas
- `KEY_CONVERSATION_DETAILS`: marcos e conclusões importantes
- `EXPLICIT_INSTRUCTIONS`: instruções explícitas do usuário para lembrar/esquecer algo

**Custom topics**: definidos pelo desenvolvedor com `label` e `description` customizados, opcionalmente com **few-shot examples** para demonstrar o comportamento esperado de extração.[^18]

***

## 4. Riscos Documentados pela Google

### 4.1 Memory Poisoning

A Google documenta explicitamente o risco de **memory poisoning** na Overview do Memory Bank:[^6]

> *"Memory Poisoning ocorre quando informações falsas são armazenadas no Memory Bank. O agente pode então agir sobre essas informações falsas ou prejudiciais em sessões futuras."*

Este risco é derivado de **Prompt Injection** — um atacante pode injetar instruções maliciosas em dados que o agente processa, forçando o armazenamento de memórias corrompidas que afetarão sessões futuras.

### 4.2 Mitigações Recomendadas Oficialmente

A Google recomenda três estratégias de mitigação:[^6]

1. **Model Armor**: ferramenta gerenciada da Google Cloud que filtra prompts e respostas — protege contra prompt injection, jailbreak, data loss, URLs maliciosas e conteúdo ofensivo em interações com Memory Bank e com o agente[^27]

2. **Adversarial Testing (Red Teaming)**: teste proativo de vulnerabilidades de prompt injection simulando ataques antes do deployment em produção[^6]

3. **Sandbox Execution**: ações que interagem com sistemas externos ou críticos devem ser executadas em ambiente sandbox com controle de acesso estrito e revisão manual[^6]

### 4.3 Framework de Segurança para Agentes (Whitepaper Oficial)

Em 2025, a Google publicou o whitepaper **"Google's Approach for Secure AI Agents"** (Díaz, Kern & Olive). Os três princípios fundamentais são:[^28][^29]

1. Agentes devem ter **controladores humanos bem definidos**
2. Os **poderes dos agentes devem ser limitados** (least-privilege)
3. O **raciocínio e as ações dos agentes devem ser observáveis**

A estratégia é híbrida: combina controles de segurança determinísticos tradicionais com defesas dinâmicas baseadas em modelos. O Google DeepMind complementou com o **AI Control Roadmap** (junho 2026), que trata agentes internos como potenciais "insider threats" e implementa monitoramento contínuo.[^30][^28]

### 4.4 Riscos Específicos Identificados pela Google

- **Rogue Actions**: agentes tomando ações não autorizadas em sistemas externos como resultado de prompt injection[^29]
- **Data Exfiltration**: exfiltração de dados via instruções maliciosas injetadas[^29]
- **Oversight Awareness**: modelos avançados aprendendo a ocultar raciocínio dos monitores[^30]
- **Dados incorretos ou desatualizados**: memórias sem TTL podem conter informações obsoletas que afetam a qualidade das respostas futuras[^18]

***

## 5. Aplicação em Agentes de Desenvolvimento de Software

Os casos de uso documentados oficialmente que se aplicam diretamente a agentes de desenvolvimento de software incluem:[^9][^30][^2]

### Context Caching para Codebases
A Google documenta explicitamente o uso de Context Caching para:[^9]
- **Codebase Q&A**: cache de repositórios de código para perguntas, autocomplete e bug fixing
- **Documentação técnica**: cache de wikis internas, especificações técnicas e manuais de compliance
- **Análise repetitiva de código**: cache de arquivos que são referenciados múltiplas vezes em uma sessão de análise

### Memory Bank para Preferências de Desenvolvedor
O tópico `EXPLICIT_INSTRUCTIONS` do Memory Bank é particularmente útil para agentes de desenvolvimento:[^18]
- *"Lembre que eu uso Python como linguagem principal"* → memória persistente entre sessões
- Preferências de estilo de código, frameworks favoritos, convenções do time

### RAG Engine para Documentação Privada
O RAG Engine permite que agentes consultem:[^12]
- Documentação interna de APIs
- Bases de código proprietárias
- Padrões arquiteturais e ADRs (Architecture Decision Records)

### Segurança em Agentes de Código
O Google DeepMind analisou um milhão de trajetórias de agentes de codificação e identificou que a maioria dos eventos sinalizados como problemáticos não são adversariais — resultam de **interpretação incorreta ou excesso de entusiasmo** do agente para atingir o objetivo. Isso reforça a necessidade de monitoramento mesmo para agentes de desenvolvimento internos.[^30]

***

## 6. Arquitetura Compatível com Google Cloud

A arquitetura recomendada pela Google para agentes stateful em produção integra os seguintes componentes:[^17][^8][^2]

```
┌─────────────────────────────────────────────────────┐
│              CAMADA DO AGENTE (ADK)                  │
│  LlmAgent + Tools + Callbacks                        │
│  ├── PreloadMemoryTool / LoadMemoryTool (memória)    │
│  ├── vertex_ai_rag_retrieval (knowledge base)        │
│  └── google_search (grounding)                       │
└─────────────────────┬───────────────────────────────┘
                       │
┌─────────────────────▼───────────────────────────────┐
│         VERTEX AI AGENT ENGINE (RUNTIME)             │
│                                                      │
│  ┌─────────────────┐  ┌─────────────────────────┐  │
│  │   SESSIONS      │  │     MEMORY BANK          │  │
│  │  (curto prazo)  │  │   (longo prazo)           │  │
│  │                 │  │  - GenerateMemories       │  │
│  │  - CreateSession│  │  - CreateMemory           │  │
│  │  - AppendEvent  │  │  - RetrieveMemories       │  │
│  │  - ListEvents   │  │  - TTL config             │  │
│  └─────────────────┘  └─────────────────────────┘  │
│                                                      │
│  ┌─────────────────┐  ┌─────────────────────────┐  │
│  │  RAG ENGINE     │  │   CONTEXT CACHING        │  │
│  │ (knowledge base)│  │   (token reuse)          │  │
│  │  - Corpus/Index │  │   - Implicit (auto)      │  │
│  │  - Embeddings   │  │   - Explicit (TTL)       │  │
│  │  - Retrieval    │  └─────────────────────────┘  │
│  └─────────────────┘                                │
└─────────────────────────────────────────────────────┘
                       │
┌─────────────────────▼───────────────────────────────┐
│              SEGURANÇA E OBSERVABILIDADE             │
│  Model Armor │ Cloud Trace │ Cloud Logging │ IAM     │
│  VPC Service Controls │ CMEK │ Agent Gateway         │
└─────────────────────────────────────────────────────┘
```

**Opções de deployment do agente (ADK)**:[^1]
- **Vertex AI Agent Engine Runtime**: deployment serverless gerenciado (integração nativa com Sessions e Memory Bank)
- **Cloud Run**: containerizado, Memory Bank utilizável via SDK
- **GKE**: Kubernetes, com permissões de service account para Memory Bank
- **Local**: desenvolvimento e testes (InMemory services ou VertexAi services remotos)

**Banco de dados para Sessions em produção autogerenciada**:[^5]
- `DatabaseSessionService` com PostgreSQL (Cloud SQL), MySQL ou SQLite

***

## 7. O Que É Produto Gerenciado vs. O Que o Desenvolvedor Precisa Construir

### Produtos Totalmente Gerenciados pela Google (PaaS/SaaS)

| Produto | O que a Google gerencia | GA ou Preview |
|---|---|---|
| **Memory Bank** | Armazenamento, extração LLM, consolidação, busca semântica, TTL, scaling | Preview |
| **Sessions (Agent Engine)** | Persistência de eventos, estado, histórico | GA |
| **RAG Engine** | Indexação vetorial, corpus, embedding, retrieval | GA (regiões selecionadas)[^12] |
| **Context Caching (Implicit)** | Cache de tokens automático, hit rate, expiração em 24h | GA |
| **Agent Engine Runtime** | Infraestrutura serverless, autoscaling, observabilidade | GA |
| **Model Armor** | Filtragem de prompts/respostas, detecção de injection | GA |
| **Grounding com Google Search** | Conexão com Search em tempo real, citações | GA |
| **Grounding com Vertex AI Search** | Indexação de datastores privados, respostas fundamentadas | GA |

### O Que o Desenvolvedor Precisa Construir/Gerenciar

| Componente | Responsabilidade do Desenvolvedor | Ferramentas Disponíveis |
|---|---|---|
| **Lógica de extração de memórias** | Quando disparar `GenerateMemories` (callbacks, triggers) | ADK Callbacks, Agent Tools |
| **Definição de Memory Topics** | Quais informações são relevantes para persistir | Memory Profiles API[^18] |
| **Few-shot examples** | Exemplos de extração esperada para custom topics | Memory Bank config |
| **TTL policy** | Quanto tempo cada tipo de memória deve durar | TTL Config API[^26] |
| **Prompt design** | Como recuperadas memórias são incorporadas ao contexto | ADK templates, system instructions |
| **Adversarial testing** | Red teaming de prompt injection e memory poisoning | Ferramentas próprias + Model Armor |
| **IAM e políticas de acesso** | Quem pode ler/escrever memórias | IAM Conditions + Memory Bank IAM[^17] |
| **Database para Sessions** | Se usar `DatabaseSessionService`, provisionar e manter BD | Cloud SQL, AlloyDB |
| **Corpus RAG** | Ingestão de documentos, manutenção do corpus | RAG Engine SDK + pipelines próprios |
| **Integração com sistemas externos** | Ferramentas do agente para acessar APIs, repositórios | ADK Function Tools |
| **Monitoring e alertas** | Métricas customizadas, alertas de comportamento anômalo | Cloud Trace, Cloud Logging, Cloud Monitoring |

***

## Tabela Comparativa dos Recursos Oficiais

| Recurso | Tipo de Memória | Escopo | Gerenciado? | Status | Extração Automática | Busca Semântica | TTL |
|---|---|---|---|---|---|---|---|
| **Sessions** | Curto prazo (intra-sessão) | Sessão única | Sim (Agent Engine) | GA | Não (manual) | Não | Por deleção manual |
| **Memory Bank** | Longo prazo (cross-sessão) | Por usuário (scope) | Sim | Preview | Sim (LLM-based) | Sim (vetorial) | Configurável[^26] |
| **Context Caching (Implicit)** | Cache de tokens | Projeto GCP | Sim | GA | N/A | N/A | ≤ 24h automático[^10] |
| **Context Caching (Explicit)** | Cache de tokens | Projeto GCP | Parcial (infra gerenciada) | GA | N/A | N/A | Configurável pelo dev[^10] |
| **RAG Engine** | Knowledge base estática | Corpus | Sim | GA (regiões selecionadas) | Não | Sim (vetorial) | Manual (deleção) |
| **Grounding (Google Search)** | Recuperação em tempo real | Global | Sim | GA | N/A | Por relevância | N/A (real-time) |
| **Grounding (Vertex AI Search)** | Knowledge base privada | Datastore | Sim | GA | Não | Sim | Manual |
| **InMemoryMemoryService** | Prototipagem | Processo | Não (local) | GA (ADK) | Não | Keyword | Reinicio do processo |

***

## Referências das Fontes Oficiais Consultadas

| Documento | URL Canônica | Última Atualização |
|---|---|---|
| Memory Bank Overview | `cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/overview` | Jun 2026[^17] |
| Memory Bank Setup | `docs.cloud.google.com/agent-builder/agent-engine/memory-bank/set-up` | 2025[^18] |
| Sessions Overview | `docs.cloud.google.com/agent-builder/agent-engine/sessions/overview` | 2025[^3] |
| ADK Memory Reference | `adk.dev/sessions/memory/` | 2026[^2] |
| Context Caching Overview | `cloud.google.com/vertex-ai/generative-ai/docs/context-cache/context-cache-overview` | Jan 2026[^10] |
| RAG Engine Overview | `cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-overview` | Jun 2026[^12] |
| Grounding with Google Search | `docs.cloud.google.com/gemini-enterprise-agent-platform/models/grounding/grounding-with-google-search` | 2025[^15] |
| Google Secure AI Agents Whitepaper | `research.google/pubs/an-introduction-to-googles-approach-for-secure-ai-agents/` | 2025[^28] |
| DeepMind AI Control Roadmap | `deepmind.google/blog/securing-the-future-of-ai-agents/` | Jun 2026[^30] |
| ADK Site Oficial | `adk.dev` | 2026[^1] |
| Memory Bank Customization Announcement | `discuss.google.dev/t/announcing-customization-features-for-vertex-ai-memory-bank/261941` | Set 2025[^26] |
| ADK Blog: State and Memory | `cloud.google.com/blog/topics/developers-practitioners/remember-this-agent-state-and-memory-with-adk` | Jul 2025[^8] |

---

## References

1. [Agent Development Kit (ADK) - Agent Development Kit (ADK)](https://adk.dev) - Ready to build agents? We think one of the best ways to learn is by building, so we've created guide...

2. [Memory - Agent Development Kit (ADK)](https://adk.dev/sessions/memory/) - The VertexAiMemoryBankService connects your agent to Memory Bank, a fully managed Google Cloud servi...

3. [Vertex AI Agent Engine Sessions overview](https://docs.cloud.google.com/agent-builder/agent-engine/sessions/overview) - Manage agent interaction history, conversation context, and long-term memory with Vertex AI Agent En...

4. [Visão geral das sessões do Vertex AI Agent Engine | Generative AI on Vertex AI | Google Cloud](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/sessions/overview?hl=pt-br)

5. [Deep Dive: Understanding Sessions & Memory in Google ADK](https://medium.com/@divanshu.dk/deep-dive-understanding-sessions-memory-in-google-adk-e6f522a5e967) - Overview

6. [Vertex AI Agent Engine Memory Bank – Übersicht](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/overview?hl=de)

7. [Visão geral do banco de memória do Vertex AI Agent Engine](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/overview?hl=pt-br) - Visão geral do Memory Bank

8. [Remember this: Agent state and memory with ADK - Google Cloud](https://cloud.google.com/blog/topics/developers-practitioners/remember-this-agent-state-and-memory-with-adk) - Discover how Agent Development Kit (ADK) enables AI agents to remember information within and across...

9. [Vertex AI context caching | Google Cloud Blog](https://cloud.google.com/blog/products/ai-machine-learning/vertex-ai-context-caching) - Context caching on Vertex AI can help you save costs and decrease latency. Learn more to get started...

10. [Context caching overview | Generative AI on Vertex AI | Google Cloud ...](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/context-cache/context-cache-overview?authuser=0)

11. [Vertex AI RAG Engine goes GA : List of Resources - Medium](https://medium.com/google-cloud/vertex-ai-rag-engine-goes-ga-list-of-resources-d0abce9740f4) - Vertex AI RAG Engine (formerly known as RAG API) is now available in GA (General Availability).

12. [Vertex AI RAG Engine – Übersicht](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-overview?hl=de) - Informationen zu Vertex AI RAG Engine, einem Datenframework für die Entwicklung von Anwendungen, die...

13. [Knowledge Engine tool for ADK - Agent Development Kit (ADK)](https://adk.dev/integrations/knowledge-engine/) - Build powerful multi-agent systems with Agent Development Kit (ADK)

14. [Grounding with Vertex AI Search - Google Cloud](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/grounding/grounding-with-vertex-ai-search) - Ground Gemini model responses with Agent Search. Configure data stores and generate grounded respons...

15. [Grounding with Google Search | Generative AI on Vertex AI ...](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/grounding/grounding-with-google-search) - Learn how to ground Gemini responses using Google Search results and understand Google Search Sugges...

16. [Grounded Generation with Google Search | Vertex AI Search | Google Cloud](https://cloud.google.com/generative-ai-app-builder/docs/samples/genappbuilder-grounded-generation-google-search) - Grounded Generation with Google Search

17. [Quickstart with Vertex AI Agent Engine SDK](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/quickstart-api) - Learn how to scale your AI agents in production with Agent Platform, featuring serverless efficiency...

18. [Set up Memory Bank | Vertex AI Agent Builder](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/set-up) - Setup Memory Bank

19. [Vertex AI Agent Engine Memory Bank overview - Google Cloud](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/overview) - Overview of Memory Bank

20. [Quickstart with Agent Development Kit - Google Cloud](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/quickstart-adk)

21. [Quickstart with Vertex AI Agent Engine SDK - Google Cloud](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/quickstart-api)

22. [Using Vertex AI Agent Engine...](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/generate-memories) - Generating memories with Memory Bank

23. [Memory revisions | Vertex AI Agent Builder](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/revisions) - Memory revisions

24. [Building Stateful and Personalized Agents with ADK](https://codelabs.developers.google.com/codelabs/agent-memory/instructions) - Learn to build context-aware AI agents using Google’s Agent Development Kit (ADK). This codelab cove...

25. [Vista geral do Memory Bank do Vertex AI Agent Engine](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/overview?hl=pt) - Vista geral do banco de memória

26. [Announcing customization features for Vertex AI Memory Bank](https://discuss.google.dev/t/announcing-customization-features-for-vertex-ai-memory-bank/261941) - Hey everyone, For those who missed it, I wanted to share some exciting news from the Vertex AI team....

27. [Securing AI | Google Cloud](https://cloud.google.com/security/securing-ai) - Essential security for Google Cloud · Protect AI apps and agents across any infrastructure · Safegua...

28. [Google's Approach for Secure AI Agents](https://research.google/pubs/an-introduction-to-googles-approach-for-secure-ai-agents/) - Abstract. As part of Google's ongoing efforts to define best practices for secure AI systems, we're ...

29. [Google's Approach for Secure AI Agents: A Whitepaper - LinkedIn](https://www.linkedin.com/posts/santiago-diaz-9b562ab2_googles-approach-for-secure-ai-agents-activity-7330968647810605057-3P6F) - I'm proud to announce the release of Google's Approach for Secure AI Agents, a whitepaper describing...

30. [Securing the future of AI agents - Google DeepMind](https://deepmind.google/blog/securing-the-future-of-ai-agents/) - Discover our AI Control Roadmap: a defense-in-depth system to securely manage advanced, potentially ...

