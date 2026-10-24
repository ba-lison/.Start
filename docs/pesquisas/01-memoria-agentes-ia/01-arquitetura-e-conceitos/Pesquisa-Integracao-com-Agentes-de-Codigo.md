<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Integração com agentes de código

Pesquise como integrar memória persistente de projeto com agentes de código e IDEs/ferramentas de desenvolvimento.
Considere:
Claude Code;
Cursor;
Continue;
Cline;
Aider;
OpenHands;
Devin-like agents, se houver fontes confiáveis;
GitHub Copilot;
LangGraph agents;
LlamaIndex agents;
OpenAI Agents SDK;
Anthropic tool use/MCP;
Google ADK/Vertex AI agents;
outras ferramentas relevantes.
Responda:
quais suportam memória nativa;
quais suportam arquivos de projeto como contexto;
quais suportam MCP;
quais suportam tools customizadas;
quais podem ler .memory/;
quais podem atualizar arquivos;
quais têm riscos de segurança;
como integrar um comando memory-curator;
como criar um fluxo antes/depois da tarefa.
Inclua tabela comparativa com fontes oficiais, datas de verificação e limitações.

Hoje praticamente todos esses agentes ainda são “stateless por padrão”, mas já existem caminhos bem claros para ter memória persistente de projeto: alguns têm camadas nativas (Claude Code, LangGraph, Vertex AI Agents, OpenAI Agents SDK), e quase todos os outros podem ser estendidos via MCP + uma pasta de memória tipo `.memory/` ou um servidor de memória externo (Hindsight, VEKTOR, etc.).[^1][^2]

Abaixo foco em: memória nativa, uso de arquivos de projeto como contexto, suporte a MCP/tools, leitura de pastas de memória, capacidade de atualizar arquivos, riscos de segurança e como encaixar um comando `memory-curator` com fluxo antes/depois da tarefa.

***

## Conceitos de memória em agentes de código

Existem três padrões principais que aparecem repetidamente na documentação e nos exemplos:

- **Memória interna do agente / sessão**
    - Exemplos: Claude Code com `MEMORY.md` por projeto, LangGraph com *checkpointers* e *stores*, Vertex AI Memory Bank, sessões do OpenAI Agents SDK.[^3][^4][^2][^5]
    - Normalmente armazenam histórico de conversa, decisões e preferências, com algum limite de tamanho por contexto.[^2][^3]
- **Memória via MCP (Model Context Protocol)**
    - MCP é o padrão da Anthropic para expor ferramentas/recursos a LLMs; hoje é suportado em Claude Code/Desktop, Cursor, Continue, Cline, OpenHands, GitHub Copilot, entre outros.[^6][^7][^8][^9]
    - Vários projetos implementam “servidores de memória” MCP (Hindsight, Pippa, Memory Keeper, SuperMemory, Stompy, Cursor Brain, Cline Memory Bank etc.).[^10][^11][^12][^8]
- **Pasta de memória dentro do repositório**
    - Alguns padrões criam uma pasta dedicada (`memory/`, `memory-bank/`, `memory-bank/*.md`, `~/.claude-memory`, etc.) com arquivos markdown + regras de atualização.[^13][^14][^15][^16][^10]
    - Isso é exatamente o que você imagina para um `.memory/` versionado ou semi-versionado.

***

## Tabela – IDE / agentes de código

**Verificado em: 2026-07-02.**

### Agents em IDEs

| Ferramenta | Memória nativa | Arquivos de projeto como contexto | MCP nativo | Tools customizadas | Semântica especial tipo `.memory/` | Atualiza arquivos? | Principais riscos |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| **Claude Code** | Sim: conversas + diretório de memória por projeto (`~/.claude/projects/<project>/memory/` com `MEMORY.md` auto-carregado).[^3][^17][^18] Plugins como `claude-mem` e `claude-mem`/MCP dão memória persistente em SQLite.[^19][^20] | Sim; lê o repositório e usa arquivos relevantes como contexto, incluindo `CLAUDE.md`, `.claude/rules/*.md`.[^21][^18] | Sim (cliente MCP em Claude Desktop/Code, com suporte a servidores como `@modelcontextprotocol/server-memory`, Memory Keeper etc.).[^13][^12][^20] | Sim, via MCP (servers próprios) e via plugins que expõem tools.[^19][^10] | Sim, mas com nome próprio: diretório `memory/` com `MEMORY.md` é carregado automaticamente; limite de ~200 linhas em `MEMORY.md`, demais detalhes ficam em arquivos auxiliares nesse diretório.[^3] Você pode apontar esse diretório para algo equivalente a `.memory/` (ex.: symlink). | Sim, edita múltiplos arquivos, roda comandos, gerencia alterações.[^19][^3] | MCP com tools de escrita e servidores de terceiros podem expor dados sensíveis; memória local (`~/.claude-memory`, SQLite) precisa ser protegida (FS perms, backup, criptografia).[^10][^13] |
| **Cursor** | Memória nativa de sessão + índice de codebase (mapa do repo); memória persistente é obtida via MCP (Cursor Brain, Cursor10x, Recallium etc.).[^22][^23][^24][^25][^26] | Sim; indexa o repositório e oferece comandos `@codebase`, `@folder`, `@file` para recuperar partes relevantes.[^22][^27] | Sim, suporte MCP nativo com config em UI e arquivos `~/.cursor/config/mcp.json` ou `.cursor/mcp.json` por projeto.[^9][^28] | Sim, via MCP servers (node/http/stdio). Você pode expor qualquer API/DB/serviço como tool.[^9][^29] | Não há diretório `.memory/` padrão, mas MCPs como Cursor Memory Bank/Recallium criam pastas como `memory-bank/` para armazenar contexto (decisions, progress etc.).[^14][^16][^25][^24] Você pode padronizar `.memory/` e ensinar o MCP a usá-lo. | Sim, edita arquivos, cria, deleta, roda comandos e faz refactors automáticos.[^9][^22] | Tools MCP com acesso de escrita podem alterar arquivos, chamar APIs ou até executar comandos; servidores externos (cloud memory, search) podem vazar código se mal configurados.[^9][^28][^25] |
| **Continue** (VS Code / JetBrains) | Não tem uma “memória longa” própria; usa histórico da sessão + integrações de memória via MCP (ex.: Stompy).[^30][^11] | Sim; usa o workspace e contextos configuráveis como fonte (similar a “chat with codebase”).[^31] | Sim, integra MCP servers via blocos `mcpServers` no config (YAML/JSON).[^30][^6][^32] | Sim, qualquer MCP server que você adicionar vira um conjunto de tools; docs mostram vários cookbooks (GitHub, Atlassian, etc.).[^6][^31] | Não há pasta dedicada nativa, mas Stompy implementa memória persistente “open memory” via MCP; você pode convencionar `.memory/` e deixá-lo como source de um MCP.[^11][^30] | Não diretamente (quem edita é o LLM via o plugin no editor), mas na prática sim: Continue sugere/aplica edits no código do workspace.[^31] | Mesmos riscos de MCP: exposição de código/dados para servidores externos e tools mal projetadas; cuidado com tokens nos configs.[^6][^30] |
| **Cline** (VS Code) | Memória própria de sessão, e memória persistente via `cline-mcp-memory-bank` que mantém contexto/progresso entre conversas.[^16][^33] | Sim; lê arquivos da workspace e usa como contexto, e o próprio Memory Bank serializa contexto do projeto em markdown.[^16] | Sim (é uma extensão construída em cima de MCP, inclusive o Memory Bank é um MCP server com tools e resources).![^16][^34] | Sim, via MCP servers adicionais, como qualquer cliente MCP.[^16] | Sim, via **`memory-bank/`**: o Memory Bank cria e mantém markdowns nesse diretório com decisões, contexto e progresso de projeto, acessíveis via resource `memory://project/context`.[^16][^14] Você pode adaptar conceito para `.memory/`. | Sim, agente edita arquivos, trackeia progresso, etc.[^16] | MCP tools com escrita e memória persistente podem registrar segredos ou decisões sensíveis; é preciso decidir o que realmente entra em memória do projeto.[^16] |
| **Aider** (CLI) | Sem memória persistente nativa entre runs; usa histórico da sessão, o repo map e o próprio git como “memória”.[^35] | Sim; lê e mapeia o repositório (repo map) e você adiciona arquivos com `/add` ou via config `read:` para contexto permanente.[^35] | Não usa MCP hoje nas docs oficiais; integra diretamente com APIs de LLM (Claude, GPT-4, etc.).[^35] | Não no padrão MCP; você pode criar wrappers de CLI ou scripts externos, mas não há mecanismo de tool-plug-in padrão documentado.[^35] | Sem semântica especial; você pode criar `.memory/` e incluí-la em `read:` ou `/add` para que Aider sempre leia antes de trabalhar.[^35] | Sim; é justamente o foco: editar arquivos, rodar lint/test, criar commits git.[^35] | Por rodar comandos, editar código e com acesso a toda a árvore do projeto, principais riscos são mudança indesejada de código, comandos perigosos e vazar código via o provedor de LLM.[^35] |
| **OpenHands** (ex-OpenDevin) | Por padrão, cada tarefa é stateless; não lembra decisões entre tasks.[^8][^36] Hindsight adiciona memória persistente por projeto via MCP (banco por repo).![^8] | Sim; lê o repo, executa comandos, edita arquivos e usa isso como contexto por tarefa.[^37][^8] | Sim, suporte MCP “streamable HTTP”; Hindsight conecta via `[mcp]` em `config.toml`.[^8][^37] | Sim, via MCP servers (Hindsight, outros) ou tools definidas no Agent SDK da própria OpenHands.[^37][^8] | Integração com Hindsight cria um “memory bank” por projeto (identificado por `bank-id`), mas não força um diretório específico; a regra é escrita em `AGENTS.md`, carregado sempre na tarefa.[^8] Você pode adicionar `.memory/` como parte desse fluxo. | Sim; faz file edits, roda comandos shell, etc. É um agente “full stack”.[^37][^8] | Altíssimo risco se rodando em produção: executa comandos arbitrários, pode instalar pacotes, alterar infra e, via MCP, gravar/levar dados sensíveis para serviços externos.[^8][^37] |
| **GitHub Copilot** (Chat \& Coding Agent) | Memória básica de chat; nenhuma “memória de longo prazo” de projeto nativa, mas pode ser estendida via MCP servers de memória ou prompts tipo “memory bank”.[^7][^15] | Sim; Copilot coding agent usa repo atual como contexto, com acesso ao código e metadados do repositório.[^38] | Sim, MCP é suportado na maioria dos editores (VS Code, JetBrains, Eclipse, Xcode) com GA em VS Code desde 2025-07.[^39][^38][^40][^41] | Sim, via MCP; você pode plugar servidores próprios e escolher quais tools ficam disponíveis.[^38][^42] | Não há pasta oficial `.memory/`, mas há um padrão de “Memory Bank” orientando Copilot a usar `memory-bank/*.md` e comandos “UMB” para atualizar os arquivos de memória.[^15] Você pode adaptar para `.memory/`. | Sim, o coding agent pode criar/editar arquivos no repo conforme as tasks atribuídas.[^38] | GitHub enfatiza que MCP de terceiros podem afetar segurança e qualidade; recomenda limitar tools de escrita e revisar cuidadosamente configs, pois o agente usa tools autonomamente e pode ter escrita dependendo da configuração.[^38][^7][^39] |


***

## Tabela – Frameworks / SDKs de agentes

### LangGraph agents

- **Memória nativa:**
    - LangGraph tem uma camada de *persistence* com **checkpointers** (curto prazo, por thread) e **stores** (cross-thread, longo prazo) para agentes.[^2]
    - Você compila o grafo com `checkpointer` (ex.: `SqliteSaver`, `PostgresSaver`) e/ou com um *store*; isso dá memória de conversa e também memória “semântica” por usuário, via consultas ao store.[^43][^44][^2]
- **Arquivos de projeto como contexto:**
    - O framework em si é agnóstico de IDE, mas nada impede de plugar tools que leem a árvore de arquivos, watchers ou integrações com VS Code/CLI para prover essas infos ao grafo.[^2]
- **MCP / tools customizadas:**
    - LangGraph não depende especificamente de MCP; você define nodes/tools como funções Python, HTTP calls, DB, etc., e pode integrar com MCP servers externos se quiser, mas isso é decisão sua na aplicação.[^2]
- **Pasta `.memory/`:**
    - Excelente encaixe: você pode usar um *store* ou um tool personalizado que leia e escreva em `.memory/` (por exemplo, gravando decisões em markdown e usando *semantic search* via store ou um serviço externo).[^44][^2]
- **Segurança:**
    - Como você controla as tools (incluindo file IO, shell, bancos), os riscos são equivalentes a qualquer orquestrador de agentes com execução de código; é crítico isolar credenciais, FS e sistemas alvo.[^43][^2]


### LlamaIndex agents

- **Memória nativa:**
    - Core LlamaIndex tem `ChatMemoryBuffer`, que guarda alguns turns da conversa, mas não lembra cross-sessão por padrão.[^45]
    - A memória persistente é tipicamente implementada via integrações como Hindsight (`HindsightMemory` / `HindsightToolSpec`) ou BotWire Memory, que adicionam recall/retain automáticos.[^46][^47]
- **Contexto de projeto:**
    - Ele é centrado em índices e RAG, então ler arquivos de projeto é natural (Document loaders, etc.); você mesmo decide como mapear a pasta de código para índices.[^45]
- **Tools / MCP:**
    - LlamaIndex já tem abstração de tools; você pode expor operações customizadas, inclusive para leitura/gravação em `.memory/` ou integração com MCP/Hindsight.[^47]
- **Pasta `.memory/`:**
    - Similar ao LangGraph: crie um índice específico alimentado por `.memory/` (arquivos markdown) e use isso como fonte de contextos longos.[^47]
- **Segurança:**
    - Riscos vêm de como você configura backends de vetores (cloud vs local), LLMs externos e tools de escrita; frameworks como Hindsight ajudam mas também podem armazenar dados sensíveis em cloud.[^45][^47]


### OpenAI Agents SDK

- **Memória nativa:**
    - O SDK traz o conceito de **sessions**; você pode usar `SQLiteSession`, `OpenAIConversationsSession` etc. para persistir histórico de mensagens entre execuções, substituindo lógica manual de DB.[^4]
    - Há artigos mostrando que os agentes são basicamente stateless se você não configurar sessão; memória mais “semântica” de longo prazo é frequentemente adicionada com VEKTOR ou Hindsight.[^48][^49][^50]
- **Contexto de projeto:**
    - Você mesmo define tools para ler arquivos, git, etc.; o SDK não é acoplado a nenhuma IDE, então o contexto de projeto vem de tools/integrações que você escreve.[^4]
- **Tools customizadas:**
    - Nativo: todo agente se baseia em tools; você define tools como funções (ex.: `remember`, `recall`) e o executor chama essas tools conforme a política.[^50][^48][^4]
- **Pasta `.memory/`:**
    - Bem direto: crie tools `read_memory`, `write_memory`, `summarize_diff_to_memory` que operam em `.memory/` e/ou em um DB local (ex.: VEKTOR usa SQLite + embeddings locais).[^48]
- **Segurança:**
    - Como o agente pode chamar tools arbitrárias, os riscos dependem de quão permissivas são as tools (file IO, shell, redes) e de onde a memória é persistida (SQLite local vs serviços cloud).[^50][^4][^48]


### Anthropic tool use / MCP (lado modelo)

- **Memória nativa:**
    - Modelos da Anthropic em si só têm memória de janela de contexto; persistência requer camadas externas (Claude Code, MCP servers de memória, etc.).[^10][^13]
- **MCP / tools:**
    - MCP é o protocolo proposto pela Anthropic; servidores como `@modelcontextprotocol/server-memory` e inúmeros projetos open source (Pippa, Memory Keeper, Claude Memory MCP, SuperMemory, etc.) expõem memória persistente via tools `memory_read/append` etc.[^12][^13][^10]
- **.memory:**
    - Vários desses MCPs usam pasta local como backend (`~/.claude-memory`, markdowns, SQLite), então é trivial adaptá-los para `.memory/` por projeto.[^12][^13][^10]


### Google ADK / Vertex AI Agents (Agent Engine)

- **Memória nativa:**
    - ADK e Vertex AI Agent Engine têm **sessions** e **Memory Bank**, com APIs dedicadas para memorizar preferências, fatos e recuperar memórias longas, inclusive com TTL.[^5][^51][^52][^53]
- **Contexto de projeto e tools:**
    - Os exemplos focam em agentes conversacionais/web, mas o mesmo modelo se aplica a agentes de código: você define serviços de memória, faz *callbacks* para geração/uso de memória e usa o Agent Engine SDK para chamar Memory Bank.[^51][^5]
- **Pasta `.memory/`:**
    - Nada nativo, mas você pode ter um worker que sincroniza `.memory/` com o Memory Bank, gerando memórias através do ADK ou da Agent Engine (por exemplo consolidando arquivos markdown em entradas de memória).[^5][^51]
- **Segurança:**
    - Memória fica em infra do Google Cloud; você ganha controles empresariais (IAM, criptografia, TTL) ao custo de depender da nuvem para esses dados.[^52][^5]

***

## Quem lê arquivos de projeto e pode ler `.memory/`

Praticamente todos os agentes que trabalham “em cima de um repo” conseguem ler `.memory/` se você a tratar como parte do projeto:

- **Claude Code / Cursor / Continue / Cline / Aider / OpenHands / Copilot**:
    - Todos leem o workspace; se `.memory/` estiver no repo, pode ser incluído em contexto manualmente (prompt) ou via instruções/regras (ex.: `.cursorrules`, `CLAUDE.md`, `AGENTS.md`, configs de MCP).[^54][^15][^21][^8][^16][^3]
    - Cline Memory Bank já usa um diretório dedicado (`memory-bank/`) como “cérebro” do projeto, então adaptar o nome para `.memory/` é só questão de configurar o MCP.[^14][^16]
- **Frameworks (LangGraph, LlamaIndex, Agents SDK, Vertex)**:
    - Basta definir tools que leem/gravam nessa pasta e, se quiser, indexá-la com embeddings para busca semântica; isso é exatamente o que guias de Hindsight/VEKTOR mostram para memória de agentes.[^44][^48][^47][^2]

***

## Quem suporta MCP e tools customizadas

- **Suporte MCP nativo (cliente):**
    - Claude Code/Desktop, Cursor, Continue, Cline, OpenHands, GitHub Copilot (em VS Code/JetBrains/Eclipse/Xcode/CLI) têm suporte explícito a MCP servers.[^7][^39][^8][^9][^29][^28][^6]
    - Isso significa que qualquer servidor MCP de memória ou ferramenta (Hindsight, Stompy, SuperMemory, VEKTOR via ponte MCP, etc.) pode ser plugado como tool.
- **Tools customizadas sem MCP:**
    - Aider, LangGraph, LlamaIndex, OpenAI Agents SDK, ADK/Vertex usam o próprio mecanismo de tools (funções, HTTP, DB) em vez de MCP, mas podem interoperar com MCP via bridges se necessário.[^35][^51][^4][^47][^2]

***

## Riscos de segurança típicos

Alguns padrões de risco se repetem nas docs:

- **Tools MCP com escrita**
    - Copilot deixa claro que MCP de terceiros podem ter tools de escrita e recomenda revisar quais tools estão habilitadas e se realmente precisam de write access.[^38][^7]
    - O mesmo vale para Cursor, Continue, Cline e OpenHands: tools que escrevem em arquivos, bancos ou chamadas HTTP podem vazar código ou credenciais se mal configuradas.[^8][^9][^28]
- **Execução de comandos / shell**
    - Agents como OpenHands, Claude Code, Cursor e Aider podem rodar comandos; isso é poderoso, mas abre superfícies para destruição de dados, instalação de malware, exfiltração etc. se houver bug no fluxo ou prompt injection.[^9][^3][^8][^35]
- **Memória externa em nuvem**
    - Soluções como Hindsight Cloud, VEKTOR + n1n.ai, Memory Bank em cloud, etc., armazenam memórias fora do seu ambiente; isso exige avaliação de compliance, LGPD, escopo de dados, separação por projeto/usuário.[^8][^48][^47][^5]
- **Injeção de instruções via arquivos de memória**
    - Pastas como `memory-bank/`, `.memory/`, `MEMORY.md` e arquivos de regras (`.cursorrules`, `AGENTS.md`) entram direto no contexto do modelo; se alguém editar isso maliciosamente, consegue reprogramar o agente.[^16][^54][^3][^8]

***

## Padrão `memory-curator`: como integrar

Dá para padronizar um comando único (`memory-curator`) que todos esses agentes usam, inspirado nos padrões já existentes (Memory Bank do Copilot, Cline Memory Bank, Hindsight para OpenHands).

### 1. Papel do `memory-curator`

- **Pré-tarefa (“recall”)**
    - Localizar memórias relevantes (em `.memory/` + servidor de memória) com base no task atual, e gerar um *bundle* compacto para injetar no agente (ex.: `activeContext.md`, ou retorno via MCP `resource` ou tool).[^15][^16][^8]
- **Pós-tarefa (“retain/reflect”)**
    - Ler diffs de código, commits, logs e mensagens, extrair decisões / padrões estáveis e consolidar em arquivos de memória (ex.: `decisionLog.md`, `systemPatterns.md`, `progress.md`) e/ou em um banco de memória remoto via tools `retain`/`remember`.[^15][^8][^48][^47]

Esse padrão é exatamente o que:

- O **Memory Bank para Copilot** faz com a pasta `memory-bank/` e o comando “UMB/Update Memory Bank”.[^15]
- O **Hindsight para OpenHands** faz com tools `recall`/`retain` e uma regra em `AGENTS.md` dizendo ao agente para sempre chamar `recall` no começo e `retain` no fim.[^8]
- Integrações Hindsight/VEKTOR para LlamaIndex e OpenAI Agents fazem com tools `remember` / `recall` ligadas a um banco de memória.[^48][^50][^47]


### 2. Implementação genérica do comando

Um `memory-curator` poderia ser:

- Um binário/CLI (Python/Node) chamado pelo agente ou por scripts de automação;
- Um MCP server com tools `memory_curate_before_task` e `memory_curate_after_task`;
- Um serviço HTTP interno (para LangGraph/LlamaIndex/Agents SDK chamarem como tool.

Exemplo de comportamento:

1. **`memory-curator before <task-id> <query>`**
    - Lê `.memory/` (ou `memory-bank/`), roda busca semântica, filtra as entradas relevantes e produz um resumo em markdown (ex.: `activeContext.md`).[^16][^47][^15]
    - Opcional: chama um servidor de memória (Hindsight/VEKTOR) para fazer `recall(query)` e mescla o resultado nesse contexto.[^8][^48]
2. **`memory-curator after <task-id>`**
    - Descobre o diff (via `git diff`, log do agente, etc.), extrai decisões importantes, atualiza `decisionLog.md`, `systemPatterns.md`, `progress.md` em `.memory/`, sempre com append + timestamps.[^16][^15]
    - Chama tools `retain`/`remember` na camada de memória remota, para que essas decisões fiquem disponíveis cross-projeto ou cross-agente se fizer sentido.[^47][^48][^8]

***

## Fluxos “antes/depois da tarefa” por ferramenta

### Claude Code / Cursor / Cline / Continue

- **Antes da tarefa**
    - Use um hook de agente (ex.: regras em `.cursorrules`, `CLAUDE.md`, `AGENTS.md` de Cline/Claude) instruindo: “Antes de resolver tasks complexas, chame o MCP tool `memory_search`/`recall` com a descrição da task.”[^54][^3][^16][^8]
    - O MCP retorna memórias relevantes (ou um arquivo `memory://project/context` no caso do Cline Memory Bank) que entram como contexto base.[^10][^16]
- **Depois da tarefa**
    - Regras adicionais mandam o agente resumir decisões e chamar `memory_add`/`update_context`/`retain` em um MCP de memória, além de atualizar arquivos do tipo `decisionLog.md` em `.memory/`.[^23][^15][^16][^8]


### OpenHands

- **Antes**: Hindsight já insere regra em `AGENTS.md` dizendo “No início de cada task, chame `recall` com o pedido do usuário.”[^8]
- **Depois**: mesma regra manda chamar `retain` para armazenar fatos duráveis (decisões, convenções, preferências).[^8]
- Você pode estender isso para também chamar seu `memory-curator` local para manter `.memory/` sincronizado com o banco Hindsight (por exemplo, armazenando apenas resumos versionados no repo).[^8]


### GitHub Copilot

- **Antes**: Copilot agent pode ler `memory-bank/*.md` e tratar como parte do contexto (via prompt/config), começando cada sessão com `productContext.md`/`systemPatterns.md`/`activeContext.md`.[^38][^15]
- **Depois**: você define o comando “UMB” como gatilho: Copilot revisa o histórico da sessão e atualiza os arquivos de memória com as decisões e progresso.[^15]
- Esse mesmo padrão pode ser portado 1:1 para `.memory/` em vez de `memory-bank/`.[^15]


### LangGraph / LlamaIndex / OpenAI Agents / Vertex

- **Antes**
    - No grafo (LangGraph) ou no agente (LlamaIndex/Agents SDK/ADK), faça o primeiro passo do fluxo chamar um *tool* de memória (`recall`, `memory_curator.before`, Memory Bank API) com o input do usuário; o retorno alimenta o estado inicial do grafo ou mensagens do agente.[^51][^4][^44][^2]
- **Depois**
    - O último nó/step chama uma tool de escrita (`retain`, `remember`, Memory Bank API, `memory_curator.after`) que persiste as novas memórias no store / DB / Memory Bank / `.memory/`.[^5][^48][^47][^2]

***

## Resumindo para o seu contexto

Para um ambiente multi-agente com CAD, plugins e repos diversos, uma abordagem bem pragmática seria:

1. **Escolher um formato de pasta de memória padrão** (por exemplo `.memory/` ou `memory-bank/` com `productContext.md`, `decisionLog.md`, `systemPatterns.md`, `activeContext.md`, `progress.md`).[^16][^15]
2. **Criar o comando/agente `memory-curator`** que todos os agentes chamam (direto ou via MCP) antes/depois das tarefas, sempre trabalhando nessa pasta e em um backend de memória (Hindsight/VEKTOR/SQLite próprio).[^48][^47][^8]
3. **Nos agentes que já suportam MCP** (Claude Code, Cursor, Continue, Cline, OpenHands, Copilot), expor o `memory-curator` como MCP server, e escrever regras simples para sempre chamar `recall` antes e `retain` depois.[^6][^9][^38][^16][^8]
4. **Nos frameworks** (LangGraph, LlamaIndex, OpenAI Agents, ADK/Vertex), modelar `memory-curator` como tools e integrar com os mecanismos de persistence nativos (checkpointers/stores, sessions, Memory Bank).[^4][^47][^5][^2]

Se você quiser, posso ajudar a desenhar uma especificação concreta de `.memory/` e do protocolo do `memory-curator` (schema dos arquivos, comandos MCP, e exemplos de integração para Claude Code + Cursor + OpenHands).
<span style="display:none">[^55][^56][^57][^58][^59][^60][^61][^62][^63][^64][^65][^66][^67][^68][^69][^70][^71][^72][^73]</span>

<div align="center">⁂</div>

[^1]: https://code.claude.com/docs/en/memory

[^2]: https://docs.langchain.com/oss/python/langgraph/persistence

[^3]: https://michaellivs.com/blog/claude-code-memory-directory/

[^4]: https://www.youtube.com/watch?v=01c6iGdwKPw

[^5]: https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/quickstart-api

[^6]: https://www.docs.continue.dev/customize/deep-dives/mcp

[^7]: https://docs.github.com/de/copilot/concepts/agents/coding-agent/mcp-and-coding-agent?wt.mc_id=3reg_webpage_reactor

[^8]: https://hindsight.vectorize.io/blog/2026/06/19/openhands-persistent-memory

[^9]: https://mcpcursor.com/docs

[^10]: https://lobehub.com/pl/mcp/nocluetoday-claude-memory-mcp

[^11]: https://www.stompy.ai/use-cases/continue-dev

[^12]: https://github.com/mkreyman/mcp-memory-keeper

[^13]: https://github.com/randall-gross/claude-memory-mcp

[^14]: https://github.com/shadowofdoom/cursor-memory-bank

[^15]: https://gist.github.com/JonyBepary/c3756c9f9447bf0176e417d507b62ee2

[^16]: https://mcp.aibase.com/server/1916356097731436545

[^17]: https://code.claude.com/docs/pt/memory

[^18]: https://milvus.io/pt/blog/claude-code-memory-memsearch.md

[^19]: https://agentfactory.panaversity.org/docs/Building-Agent-Factories/augmented-memory/memory-for-claude-code

[^20]: https://lobehub.com/it/mcp/mkreyman-mcp-memory-keeper

[^21]: https://todatabeyond.substack.com/p/claude-code-memorymd-everything-you

[^22]: https://milvus.io/ai-quick-reference/can-cursor-read-my-entire-codebase

[^23]: https://forum.cursor.com/t/feedback-on-cursor-brain/152709

[^24]: https://github.com/aurda012/cursor10x-mcp

[^25]: https://forum.cursor.com/t/persistent-ai-memory-for-cursor/145660

[^26]: https://www.varunpratap.com/blog/how-to-give-cursor-persistent-memory

[^27]: https://www.youtube.com/watch?v=eeKwGTL46ys

[^28]: https://docs.aigateway.cequence.ai/docs/client-config/cursor/

[^29]: https://github.com/mapbox/mcp-server/blob/HEAD/docs/cursor-setup.md

[^30]: https://docs.continue.dev/customize/mcp-tools

[^31]: https://docs.continue.dev/guides/overview

[^32]: https://docs.continue.dev/guides/continue-docs-mcp-cookbook

[^33]: https://github.com/dazeb/cline-mcp-memory-bank

[^34]: https://mcp.aibase.com/tw/server/1916354677028397057

[^35]: https://terminalskills.io/skills/aider

[^36]: https://github.com/All-Hands-AI/OpenHands/issues/1748

[^37]: https://arxiv.org/html/2511.03690v1

[^38]: https://docs.github.com/en/copilot/concepts/agents/coding-agent/mcp-and-coding-agent

[^39]: https://github.blog/changelog/2025-07-14-model-context-protocol-mcp-support-in-vs-code-is-generally-available/

[^40]: https://github.com/orgs/community/discussions/181639

[^41]: https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp

[^42]: https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp

[^43]: https://www.youtube.com/watch?v=YE6A5d8kNp4

[^44]: https://www.youtube.com/watch?v=dS554YJRFIM

[^45]: https://ricord.ai/compare/best-for-llamaindex

[^46]: https://botwire.dev/articles/llamaindex-agent-memory

[^47]: https://hindsight.vectorize.io/guides/2026/05/04/guide-llamaindex-memory-with-hindsight

[^48]: https://explore.n1n.ai/blog/persistent-memory-openai-agents-sdk-vektor-2026-04-05

[^49]: https://hindsight.vectorize.io/blog/2026/04/17/openai-agents-persistent-memory

[^50]: https://hindsight.vectorize.io/sdks/integrations/openai-agents

[^51]: https://www.youtube.com/watch?v=sMtrelDNxIc

[^52]: https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/overview

[^53]: https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/quickstart-adk?hl=es-419

[^54]: https://news.ycombinator.com/item?id=43506068

[^55]: https://www.youtube.com/watch?v=mKXzPVIMFYE

[^56]: https://github.com/neobundy/cwkCursorPippaMCP/blob/main/README.md

[^57]: https://code.claude.com/docs/de/memory

[^58]: https://zenn.dev/junko_ai/articles/311b69300fde6d?locale=en

[^59]: https://www.youngleaders.tech/p/how-i-finally-sorted-my-claude-code-memory

[^60]: https://www.youtube.com/watch?v=VktAdYrK6io

[^61]: https://code.claude.com/docs/ko/memory

[^62]: https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp/extend-copilot-chat-with-mcp?tool=xcode

[^63]: https://www.youtube.com/watch?v=O73egpvWcpY

[^64]: https://hindsight.vectorize.io/blog/2026/03/30/llamaindex-agent-memory

[^65]: https://cursor.com/docs/mcp.md

[^66]: https://github.com/cursor/mcp-servers

[^67]: https://docs.maximem.ai/integrations/openai-agents

[^68]: https://aihola.com/article/openai-agents-sdk-memory-personalization

[^69]: https://www.youtube.com/watch?v=khjxYTWpOYE

[^70]: https://github.com/JeredBlu/guides/blob/main/cursor-mcp-setup.md

[^71]: https://www.youtube.com/watch?v=2Cf-HYyMDDE

[^72]: https://www.youtube.com/watch?v=83_s2lfZm-E

[^73]: https://www.youtube.com/watch?v=Ct4mSNy3UyM

