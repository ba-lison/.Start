# Documentação Oficial OpenAI: Memória, Contexto e Recuperação de Informação

> **Escopo:** Apenas fontes oficiais da OpenAI — platform.openai.com, help.openai.com, developers.openai.com, openai.github.io. Acesso: julho de 2025–2026.

***

## 1. O que a OpenAI oferece oficialmente hoje

A OpenAI disponibiliza **múltiplas camadas** de memória e contexto, segmentadas entre produto (ChatGPT) e plataforma (API). É fundamental não confundir os dois mundos: recursos do produto ChatGPT geralmente **não são acessíveis via API**, e os recursos de API exigem implementação pelo desenvolvedor.

### Visão Geral dos Recursos

| Recurso | Camada | Memória/Persistência | Controle |
|---|---|---|---|
| ChatGPT Memory (Saved Memories) | Produto ChatGPT | Persistente (enquanto ativo) | Usuário final |
| ChatGPT Reference Chat History | Produto ChatGPT | Transitória (chats existentes) | Usuário final |
| ChatGPT Projects | Produto ChatGPT | Persistente por projeto | Usuário / Admin |
| Custom GPTs (Knowledge) | Produto ChatGPT | Persistente (arquivos estáticos) | Builder do GPT |
| Responses API (`previous_response_id`) | API | 30 dias (store=true) | Desenvolvedor |
| Conversations API (`conversation_id`) | API | Sem TTL (persistente) | Desenvolvedor |
| Vector Stores + File Search | API | Persistente até expirar | Desenvolvedor |
| Agents SDK Sessions | API / SDK | Controlado pelo dev (banco escolhido) | Desenvolvedor |
| Agents SDK Sandbox Memory | API / SDK (Beta) | Por sessão de sandbox | Desenvolvedor |
| Assistants API (Threads) | API — **DEPRECATED** | Até sunset: 26 ago 2026 | Desenvolvedor |

***

## 2. Diferença entre os principais recursos

### ChatGPT Memory (Produto)

A memória do ChatGPT é um recurso **exclusivo do produto ChatGPT** (não exposto na API). Opera em duas modalidades:[^1][^2]

- **Saved Memories (novo sistema):** síntese automática e contínua de contexto de chats, arquivos e apps conectados. Exposta ao usuário via "memory summary" com edição manual. Disponível em Plus, Pro e planos selecionados.[^1]
- **Reference Chat History:** o ChatGPT acessa chats anteriores para personalização. Sem limite de armazenamento; itens mais antigos podem ser despriorizados automaticamente.[^1]

> **⚠️ Importante para developers:** Memória do ChatGPT **não está disponível em Custom GPTs** criados com GPT Builder. Cada conversa com um GPT começa sem estado, independentemente de as configurações de memória do usuário estarem ativas.[^2][^3]

### ChatGPT Projects (Produto)

Projects são workspaces do ChatGPT que agrupam chats, arquivos e instruções por tema. Recursos chave:[^4]

- **Arquivos:** upload de PDFs, planilhas, imagens, links Google Drive e Slack. Limites por plano: Free 5 arquivos, Plus/Go 25, Pro/Enterprise/Edu 40.[^4]
- **Project Memory:** por padrão, referencia os chats dentro do projeto. Com "project-only memory", isola completamente o contexto da memória global do usuário.[^4]
- **Colaboração:** disponível para todos os planos desde out/2025. Até 100 colaboradores para Pro, 10 para Plus/Go, 5 para Free.[^4]
- **Natureza:** produto ChatGPT, **sem acesso via API de desenvolvimento**.[^5]

### Custom GPTs (Produto)

GPTs são versões configuradas do ChatGPT com instruções, knowledge files e capabilities. Características de memória:[^3]

- Suportam até **20 arquivos de knowledge**, cada arquivo até 512 MB e 2.000.000 tokens.[^6]
- Usam busca semântica ou revisão de documento completo para responder.[^6]
- **Não possuem memória persistente entre sessões.** Cada conversa começa do zero.[^2][^3]
- Gerenciamento de arquivos **somente via GPT Builder UI**, não há API para atualização programática.[^6]

### Responses API (API)

A Responses API é a **principal API de geração de texto** recomendada para novos projetos. Substituiu diretamente o Chat Completions como padrão para aplicações agentic.[^7][^8]

Mecanismos de contexto:

- **`previous_response_id`:** encadeia respostas. O servidor recarrega o contexto da resposta anterior, eliminando o reenvio do histórico completo. Responses ficam armazenadas por **30 dias** por padrão com `store=true`.
- **`store: false`:** desabilita persistência. Obrigatório para organizações com Zero Data Retention (ZDR).[^9]
- **Stateful conversations:** custo de tokens — mesmo usando `previous_response_id`, todos os tokens de input das respostas encadeadas são cobrados como input tokens.

### Conversations API (API)

Introduzida como substituta das Threads do Assistants API. Características:[^10]

- Cria objetos `conversation` com ID durável persistindo entre sessões, dispositivos e workers.
- Armazena `items` (mensagens, tool calls, outputs, etc.), não apenas mensagens simples.[^10]
- **Sem TTL de 30 dias** — diferente de responses isoladas, itens em uma conversation persistem indefinidamente.
- Compatível com Responses API: `openai.responses.create(..., conversation: "conv_xxx")`.

### Vector Stores + File Search (API)

A **Retrieval API** é uma busca semântica sobre dados do desenvolvedor, alimentada por **vector stores**.

**Vector Stores:**
- Containers para arquivos indexados via chunking + embeddings.
- Ao adicionar arquivo: chunking automático (800 tokens, overlap 400), embedding com `text-embedding-3-large` @ 256 dimensões.[^1]
- **Pricing:** até 1 GB gratuito; acima, $0.10/GB/dia.
- Limites: 512 MB por arquivo, 5M tokens por arquivo, 10.000 arquivos por vector store.[^1]
- **Expiration policy** configurável via `expires_after` (por dias de inatividade).
- Suporte a **attribute filtering** (ex.: filtrar por região, data, categoria) com até 16 atributos por arquivo.

**File Search (ferramenta da Responses API):**
- Ferramenta hosted — o desenvolvedor não executa código; o modelo decide quando acioná-la.
- Disponível na Responses API, Chat Completions e Assistants API.
- `max_num_results` configurável (padrão 10, máximo 50).
- Busca retorna `file_search_call` + `message` com `file_citation` annotations.
- Sem suporte a imagens dentro de documentos; sem suporte a formatos estruturados como CSV/JSONL.[^1]

### Agents SDK (API/SDK)

SDK Python/TypeScript para orquestração de agentes multi-turn e multi-step. Primitivos principais:

- **Agents** com instructions e tools.
- **Handoffs** para delegação entre agentes.
- **Guardrails** para validação de inputs/outputs.
- **Tracing** embutido.

#### Sessions (memória conversacional, SDK)

O Agents SDK provê **session memory** para manter histórico de conversação sem gerenciamento manual.

Implementações disponíveis (oficialmente documentadas):

| Implementação | Ideal para | Storage |
|---|---|---|
| `SQLiteSession` | Dev local e apps simples | Arquivo SQLite |
| `AsyncSQLiteSession` | SQLite assíncrono | aiosqlite |
| `RedisSession` | Multi-worker distribuído | Redis |
| `SQLAlchemySession` | Apps produção com DB existente | PostgreSQL, MySQL, etc. |
| `MongoDBSession` | Apps MongoDB | MongoDB Atlas/local |
| `DaprSession` | Cloud-native com Dapr | 30+ backends |
| `OpenAIConversationsSession` | Storage gerenciado pela OpenAI | Conversations API |
| `EncryptedSession` | Criptografia + TTL sobre qualquer backend | Wrapper |

Sessions **não podem ser combinadas** com `conversation_id`, `previous_response_id` ou `auto_previous_response_id` no mesmo run.

#### Sandbox Memory (Beta)

Memória persistente entre runs de Sandbox Agents, separada de Sessions. Armazena lessons aprendidas em `memories/MEMORY.md` e `memory_summary.md`. Dois phases: extração de conversação e consolidação de layout. Requer `Memory()` capability com `Shell()` e `Filesystem()`.

### Assistants API (DEPRECATED)

A Assistants API foi **oficialmente depreciada em 26 de agosto de 2025**, com sunset em **26 de agosto de 2026**. Todos os novos projetos devem usar a Responses API.

Tabela de equivalência para migração:

| Antes (Assistants) | Agora (Responses) |
|---|---|
| `Assistants` | `Prompts` (configurados no dashboard) |
| `Threads` | `Conversations` |
| `Runs` | `Responses` |
| `Run steps` | `Items` |

***

## 3. A OpenAI oferece memória persistente automática via API?

**Parcialmente, dependendo do mecanismo.**

| Mecanismo | Persistência automática? | Quem implementa? |
|---|---|---|
| `previous_response_id` (Responses API) | Parcial — server guarda por 30 dias | Desenvolvedor passa o ID |
| Conversations API | Sim — sem TTL | Desenvolvedor cria a conversation |
| Vector Stores | Sim — até policy de expiração definida | Desenvolvedor faz upload e configura |
| Agents SDK Sessions | Depende do backend escolhido | Desenvolvedor escolhe e provisiona |
| Sandbox Memory (Beta) | Por sessão de sandbox | Desenvolvedor habilita capability |
| ChatGPT Memory | Sim — automático | Produto ChatGPT (não disponível via API) |

A OpenAI **não oferece memória episódica automática via API** (como o ChatGPT faz para usuários finais). O desenvolvedor deve escolher e implementar a estratégia de persistência adequada. A Conversations API é a opção mais próxima de "persistência automática gerenciada", pois o servidor OpenAI mantém o estado indefinidamente.

***

## 4. Limites, Riscos e Boas Práticas Documentados Oficialmente

### Limites técnicos

- **Context window:** cada modelo tem limite de tokens para input + output + reasoning. Exceder resulta em truncamento.
- **Response storage:** 30 dias com `store=true`; objetos em Conversations não têm TTL.
- **File Search (Assistants):** `max_prompt_tokens` recomendado de no mínimo 20.000; idealmente sem limite para melhor qualidade.[^1]
- **Vector store:** 512 MB/arquivo, 5M tokens/arquivo, 10k arquivos/store, 100 GB por organização.[^1]
- **Rate limits Assistants API:** GET 1000 RPM, POST 300 RPM, POST /runs 200 RPM.[^1]
- **Custom GPT knowledge:** máximo 20 arquivos, 512 MB/arquivo, 2M tokens/arquivo.[^6]
- **ChatGPT Projects:** 5 (Free) a 40 (Enterprise) arquivos por projeto.[^4]

### Riscos documentados

- **Tokens duplicados em chaining:** ao usar `previous_response_id`, todos os tokens anteriores são cobrados novamente como input tokens.
- **Stale memory:** Sandbox Memory pode ficar desatualizada; agentes são instruídos a tratar memórias como orientação, não verdade absoluta.
- **Context contamination:** sem `project-only memory` no ChatGPT, contexto de projetos diferentes pode vazar.
- **ZDR incompatibility:** organizações com Zero Data Retention não podem usar Responses API de forma stateful; usar `store=false` com encrypted reasoning items.[^9]
- **Dados de API não são usados para treino** sem consentimento explícito. Para usuários Free/Plus do ChatGPT, dados de projetos podem ser usados se "Improve the model for everyone" estiver ativo.[^4][^1]
- **File Search não suporta CSV/JSONL** nem imagens dentro de documentos.[^1]
- **Sessions + server-managed state:** não misturar Sessions do SDK com `conversation_id`/`previous_response_id` — duplica contexto.

### Boas práticas (documentação oficial)

- Usar Responses API para novos projetos; migrar de Assistants API antes de ago/2026.
- Usar `previous_response_id` para conversas simples; usar Conversations API quando múltiplos workers precisam compartilhar estado.
- Para File Search, usar `max_num_results` configurável para balancear qualidade e custo de tokens.
- Configurar `expires_after` em vector stores para controlar custos de storage.
- Usar atributos (`attributes`) em vector store files para filtros precisos.
- Para agents com múltiplos especialistas, usar layouts separados de Sandbox Memory para isolar contexto.
- Reescrever queries com `rewrite_query=true` para melhor performance de busca semântica.
- Ao usar `session_input_callback` no Agents SDK, implementar pruning de histórico para controlar tamanho de contexto.

***

## 5. Como Montar Memória de Projeto via API

A documentação oficial não define um padrão único chamado "memória de projeto", mas os blocos de construção estão disponíveis:

### Arquitetura recomendada (baseada na documentação)

```
┌─────────────────────────────────────────────────────┐
│  Memória Semântica (conhecimento do projeto)        │
│  ─ Vector Store com arquivos do projeto             │
│  ─ File Search tool na Responses API                │
│  ─ Attribute filtering por tipo/fase/autor          │
├─────────────────────────────────────────────────────┤
│  Memória Conversacional (histórico de interações)   │
│  ─ Conversations API (server-managed, sem TTL)      │
│    OU Agents SDK Session (SQLiteSession, Redis etc) │
├─────────────────────────────────────────────────────┤
│  Memória Episódica (lições aprendidas)              │
│  ─ Agents SDK Sandbox Memory (Beta)                 │
│    OU implementação custom (ex.: banco vetorial ext)|
└─────────────────────────────────────────────────────┘
```

### Fluxo prático (Node.js / Python)

**1. Criar o knowledge base do projeto:**
```python
vector_store = client.vector_stores.create(name="projeto-X")
client.vector_stores.files.upload_and_poll(
    vector_store_id=vector_store.id,
    file=open("briefing_projeto.pdf", "rb")
)
```

**2. Criar conversation persistente para o usuário:**
```python
conversation = openai.conversations.create()
# Salvar conversation.id no banco associado ao usuário
```

**3. Executar respostas com contexto combinado:**
```python
response = openai.responses.create(
    model="gpt-4.1",
    input=[{"role": "user", "content": user_message}],
    conversation="conv_xxx",  # histórico persistente
    tools=[{
        "type": "file_search",
        "vector_store_ids": ["vs_xxx"]  # knowledge semântico
    }]
)
```

**4. Para agentes multi-step (Agents SDK):**
```python
from agents import Agent, Runner
from agents.extensions.memory import SQLAlchemySession

session = SQLAlchemySession("user_projeto_x", engine=engine, create_tables=True)
result = await Runner.run(agent, user_input, session=session)
```

***

## 6. Tabela Consolidada: Recurso × Finalidade × Persistência

| Recurso | Finalidade | Persistência | Quem controla | Limitações principais | Link oficial | Obs. para código |
|---|---|---|---|---|---|---|
| **ChatGPT Memory** (Saved Memories) | Personalização automática do ChatGPT entre sessões | Persistente (até deletar) | Usuário final | Não disponível via API; não funciona em Custom GPTs | [help.openai.com/en/articles/8590148](https://help.openai.com/en/articles/8590148-memory-faq) | Produto ChatGPT exclusivamente |
| **ChatGPT Reference Chat History** | Personalização via chats anteriores | Vinculada a chats existentes | Usuário final | Não disponível via API; deletar chat remove referência | [help.openai.com/en/articles/8590148](https://help.openai.com/en/articles/8590148-memory-faq) | Idem |
| **ChatGPT Projects** | Workspace com arquivos, instruções e chats agrupados | Persistente por projeto | Usuário / Admin | Produto ChatGPT; sem API; 5-40 arquivos por plano | [help.openai.com/en/articles/10169521](https://help.openai.com/en/articles/10169521-projects-in-chatgpt) | Não acessível programaticamente |
| **Custom GPTs (Knowledge)** | RAG estático em GPTs configurados | Persistente (arquivos fixos) | Builder do GPT | Sem memória entre sessões; max 20 arquivos; sem API de update | [help.openai.com/en/articles/8843948](https://help.openai.com/en/articles/8843948-knowledge-in-gpts) | Somente via GPT Builder UI |
| **Responses API (`previous_response_id`)** | Encadeamento de contexto entre chamadas | 30 dias (com store=true) | Desenvolvedor | Tokens duplicados a cada chamada; não funciona com ZDR | [platform.openai.com/docs/guides/conversation-state](https://platform.openai.com/docs/guides/conversation-state) | Passar ID da resposta anterior no próximo request |
| **Conversations API** | Histórico multi-turn server-managed | Sem TTL (indefinido) | Desenvolvedor | Novo (2025); substitui Threads; requer migration de Assistants | [platform.openai.com/docs/guides/conversation-state](https://platform.openai.com/docs/guides/conversation-state) | `openai.conversations.create()` + passar `conversation` em responses |
| **Vector Stores** | Índice semântico para arquivos do desenvolvedor | Configurável (ou permanente) | Desenvolvedor | $0.10/GB/dia após 1 GB; 512 MB/arquivo; sem CSV/JSONL | [platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval) | `client.vector_stores.create()` + upload de arquivos |
| **File Search (Responses API)** | RAG automático via vector stores em cada response | Depende do vector store | Desenvolvedor | Sem imagens em docs; sem CSV/JSONL; sem chunking customizado via Assistants | [platform.openai.com/docs/guides/tools-file-search](https://platform.openai.com/docs/guides/tools-file-search) | Tool hosted; declarar `type: "file_search"` no array de tools |
| **Retrieval API (busca direta)** | Busca semântica manual sem modelo | Depende do vector store | Desenvolvedor | Resultado bruto; dev sintetiza resposta com modelo separado | [platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval) | `client.vector_stores.search(query=...)` |
| **Agents SDK Sessions** | Histórico conversacional persistente para agentes | Controlado pelo dev (backend escolhido) | Desenvolvedor | Não combinar com `previous_response_id` / `conversation_id` | [openai.github.io/openai-agents-python/sessions/](https://openai.github.io/openai-agents-python/sessions/) | Múltiplos backends: SQLite, Redis, PostgreSQL, MongoDB |
| **Agents SDK Sandbox Memory** | Lições aprendidas entre runs de sandbox agents | Por workspace de sandbox (persistível via snapshot) | Desenvolvedor | **Beta** — API pode mudar; requer Shell + Filesystem capabilities | [openai.github.io/openai-agents-python/sandbox/memory/](https://openai.github.io/openai-agents-python/sandbox/memory/) | `Memory()` capability em `SandboxAgent` |
| **Assistants API (Threads)** | ~~Histórico persistente server-managed~~ | Até 26/08/2026 (sunset) | ~~Desenvolvedor~~ | **DEPRECATED — sunset 26/08/2026** | [platform.openai.com/docs/assistants/migration](https://platform.openai.com/docs/assistants/migration) | Migrar para Conversations API + Responses API |

***

## 7. Produto ChatGPT vs. API: Fronteira clara

### Exclusivo produto ChatGPT (sem acesso via API)

- ChatGPT Memory (Saved Memories e Reference Chat History)[^1]
- ChatGPT Projects e project-only memory[^5][^4]
- Colaboração em projetos compartilhados[^4]
- Custom GPTs como experiência final do usuário[^11]

### Exclusivo API (não disponível no produto ChatGPT)

- Responses API com `previous_response_id`
- Conversations API programática
- Vector Stores com controle de chunking, embeddings e attribute filters
- Agents SDK com sessions, guardrails, handoffs e sandbox execution
- Retrieval API para busca semântica headless

### Compartilhado (funciona em ambos os contextos)

- File Search como ferramenta (disponível em Responses API, Chat Completions e Assistants API)
- Modelos gpt-4.1, gpt-4o, o1, o3 etc.

***

## 8. O que é documentado vs. o que seria inferência

### Documentado oficialmente

- Parâmetros e comportamento exato de `previous_response_id`, `store`, Conversations API
- Estrutura de vector stores, chunking, embeddings e attribute filters
- Sessions do Agents SDK com todos os backends listados
- Sandbox Memory com phases 1 e 2 de consolidação
- Deprecação do Assistants API com data exata (26/08/2026)
- Limites precisos de arquivos, tokens e custos de storage[^1]
- ChatGPT Memory não funciona em Custom GPTs[^3]

### Inferência arquitetural (não documentado explicitamente)

- Como exatamente a síntese de memória do ChatGPT é feita internamente (o "novo sistema" é descrito funcionalmente, não tecnicamente)[^1]
- Qual modelo específico é usado para geração/consolidação de Sandbox Memory (mencionado como "memory-generating model" sem especificar)
- A estrutura de embedding interna das vector stores além de `text-embedding-3-large @ 256 dims`[^1]
- Algoritmo exato de rankeamento quando `ranker: auto` é usado
- Como o ChatGPT Projects decide quais chats usar para personalização em cada query

***

## Referências Oficiais

- [platform.openai.com/docs/guides/conversation-state](https://platform.openai.com/docs/guides/conversation-state) — Conversation State Guide (acessado jul/2026)
- [platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval) — Retrieval & Vector Stores Guide (acessado jul/2026)
- [platform.openai.com/docs/guides/tools-file-search](https://platform.openai.com/docs/guides/tools-file-search) — File Search Tool Guide (acessado jul/2026)
- [platform.openai.com/docs/guides/responses-vs-chat-completions](https://platform.openai.com/docs/guides/responses-vs-chat-completions) — Migrating to Responses API (acessado jul/2026)
- [platform.openai.com/docs/assistants/overview](https://platform.openai.com/docs/assistants/overview) — Assistants Migration Guide (acessado jul/2026)
- [openai.github.io/openai-agents-python/sessions/](https://openai.github.io/openai-agents-python/sessions/) — Agents SDK Sessions (acessado jul/2026)
- [openai.github.io/openai-agents-python/sandbox/memory/](https://openai.github.io/openai-agents-python/sandbox/memory/) — Agents SDK Sandbox Memory (acessado jul/2026)
- [help.openai.com/en/articles/8590148-memory-faq](https://help.openai.com/en/articles/8590148-memory-faq) — ChatGPT Memory FAQ (acessado jul/2026)
- [help.openai.com/en/articles/10169521-projects-in-chatgpt](https://help.openai.com/en/articles/10169521-projects-in-chatgpt) — Projects in ChatGPT (acessado jul/2026)
- [help.openai.com/en/articles/8843948-knowledge-in-gpts](https://help.openai.com/en/articles/8843948-knowledge-in-gpts) — Knowledge in GPTs (acessado jul/2026)
- [help.openai.com/en/articles/8983148-does-memory-function-with-gpts](https://help.openai.com/en/articles/8983148-does-memory-function-with-gpts) — Memory and GPTs FAQ (acessado jul/2026)
- [help.openai.com/en/articles/8554407-gpts-in-chatgpt](https://help.openai.com/en/articles/8554407-gpts-in-chatgpt) — GPTs in ChatGPT (acessado jul/2026)
- [help.openai.com/en/articles/8550641-assistants-api-v2-faq](https://help.openai.com/en/articles/8550641-assistants-api-v2-faq) — Assistants API v2 FAQ (acessado jul/2026)

---

## References

1. [Assistants API (v2) FAQ](https://help.openai.com/de-de/articles/8550641-assistants-api-v2-faq)

2. [Does memory function with GPTs? - OpenAI Help Center](https://help.openai.com/en/articles/8983148-does-memory-function-with-gpts) - Understand how memory works in ChatGPT and why it’s not yet available for custom GPTs.

3. [GPTs in ChatGPT - OpenAI Help Center](https://help.openai.com/en/articles/8554407-gpts-in-chatgpt) - Learn what GPTs are in ChatGPT, what they can include, who can use and build them, and how they inte...

4. [Projects in ChatGPT | OpenAI Help Center](https://help.openai.com/en/articles/10169521-projects-in-chatgpt) - Organize, revisit, and continue your work—all in one place

5. [Projects - Resource | OpenAI Academy](https://academy.openai.com/public/clubs/work-users-ynjqu/resources/projects) - Projects can help you keep your chats, knowledge, and instructions organized in one place, and allow...

6. [Knowledge in GPTs | OpenAI Help Centerhelp.openai.com › articles › 8843948-knowledge-in-gpts](https://help.openai.com/en/articles/8843948-knowledge-in-gpts) - Here is what you need to know about the GPT knowledge feature

7. [API](https://developers.openai.com/api/docs) - Explore guides, API docs, and examples for the OpenAI API.

8. [Agents SDK | OpenAI API](https://developers.openai.com/api/docs/guides/agents) - Use the Responses API when one model call plus tools and application-owned logic is enough. Use the ...

9. [Vector Stores](https://developers.openai.com/api/docs/guides/retrieval) - Learn how to search your data using semantic similarity with the OpenAI API.

10. [Assistants API - OpenAI Platform](https://platform.openai.com/docs/assistants) - Explore resources, tutorials, API docs, and dynamic examples to get the most out of OpenAI's develop...

11. [Enterprise Customers Can...](https://openai.com/index/introducing-gpts/) - You can now create custom versions of ChatGPT that combine instructions, extra knowledge, and any co...

