# Orquestração de Subagents em Coding Agents – Relatório Técnico

## Resumo executivo

Este relatório analisa, com base em documentação oficial recente, como os principais coding agents de mercado implementam **subagents**: Claude Code, OpenAI Codex, VS Code/Copilot, Gemini CLI, Google Antigravity, OpenCode, LangChain Deep Agents e a integração Ollama + Claude Code.[web:1][web:3][web:7][web:8][web:10][web:11][web:12][web:15][web:26] O foco é a orquestração interna de subagents por um agente primário, e não frameworks genéricos de múltiplos agentes externos.

Subagents são agentes especializados, com **contexto separado**, **instruções próprias**, **conjunto de ferramentas/permissões específico** e, muitas vezes, **modelo e memória próprios**, acionados pelo agente principal ou por configuração/programa para executar subtarefas delimitadas e devolver um resultado condensado (texto, JSON tipado ou artefatos).[web:1][web:3][web:7][web:10][web:11] Em todas as plataformas, o objetivo central é mitigar **context pollution/rot**, ganhar paralelismo controlado e tornar tarefas grandes tratáveis via fan-out/fan-in.[web:3][web:7][web:9][web:10]

A análise mostra um conjunto claro de padrões convergentes: (1) subagents definidos via manifesto (YAML/TOML/JSON) com `name`, `description`, `system_prompt`, `tools`, `model` e políticas de permissão; (2) invocação automática por descrição ou explícita via @mention/tool; (3) isolamento de contexto, ferramentas, MCP/servidores, memória e, às vezes, worktrees; (4) paralelismo via múltiplos subagents ou instâncias de um mesmo tipo; (5) retorno consolidado ao agente primário, frequentemente com suporte a resposta estruturada.[web:1][web:3][web:7][web:10][web:11][web:23]

Com base nesses padrões, o relatório propõe um **modelo mental unificado** e uma **arquitetura prática** para um orquestrador próprio de subagents, incluindo formato de manifesto em YAML, componentes essenciais (registry, context packer, sandbox, scheduler, synthesizer, observabilidade) e padrões de execução (fan-out + synthesize, classify + route, adversarial verification, generate + judge/filter, map-reduce por arquivo, batch via CSV, background e nested subagents).[web:3][web:7][web:8][web:9][web:10][web:24]

---

## Definições e taxonomia de conceitos

### Definição curta de subagent

Nas diferentes plataformas:

- **Claude Code**: “Subagents are specialized AI assistants that handle specific types of tasks… Each subagent runs in its own context window with a custom system prompt, specific tool access, and independent permissions.”[web:1]
- **Gemini CLI**: “Subagents are specialized, expert agents that operate alongside your primary Gemini CLI session… Each subagent operates within its own separate context window, custom system instructions, and curated set of tools.”[web:10][web:11]
- **LangChain Deep Agents**: “A deep agent can create subagents to delegate work… Subagents are useful for context quarantine and providing specialized instructions.”[web:7]
- **VS Code/Copilot**: subagent é um agente independente que executa trabalho focado com contexto isolado e retorna um resumo ao agente principal.[web:24]

Em termos práticos, **subagent** é:

> Um agente especializado, invocado por um agente principal, que roda em loop próprio com contexto isolado, ferramentas/permissões específicas e objetivo delimitado, retornando um resultado condensado ou estruturado em vez de seu histórico completo.

### Diferença entre agent orchestration e subagent orchestration

- **Agent orchestration (multi-agent externo)**: múltiplos agentes independentes (por exemplo, serviços separados) coordenados por um orquestrador externo, cada um com sua própria superfície de interface, estado longo e, muitas vezes, identidades organizacionais distintas. É o foco de frameworks amplos tipo “agent teams” generalistas.
- **Subagent orchestration (interna a coding agent)**: o loop principal do coding agent (CLI/IDE) instancia agentes-filho efêmeros ou especializados, sempre subordinados à sessão principal, e gerencia seu ciclo de vida (spawn, controle de ferramentas/sandbox, fan-out/fan-in, coleta de resultados).[web:1][web:3][web:7][web:10]

Subagents aqui são **processos internos** à “mente” do coding agent, não serviços autônomos long-running.

### Por que subagents são melhores que tudo no loop principal

Documentação de Codex, Claude Code, Deep Agents e Gemini CLI convergem em três motivações principais:[web:3][web:7][web:9][web:10][web:11]

- **Evitar context pollution/rot**: logs de testes, grandes trechos de código, resultados de web search, etc., se acumulariam no histórico do agente principal e degradariam a qualidade de raciocínio. Subagents isolam esses passos em janelas próprias e retornam apenas um resumo final.[web:3][web:7][web:18]
- **Paralelismo seguro**: tarefas paralelizáveis (ex.: análise por arquivo, por módulo, por hipótese) podem ser distribuídas entre subagents, reduzindo latência total sem que o agente principal precise gerenciar cada interação de ferramenta.[web:3][web:7][web:9][web:10][web:12]
- **Especialização e governança**: cada subagent pode ter tools, MCP servers, modelos e políticas de permissão ajustados ao seu papel (ex.: read-only explorer vs implementer com write/bash), melhorando segurança, custo e previsibilidade.[web:1][web:7][web:10][web:11][web:15]

Isso é explicitamente citado como solução para context bloat em Deep Agents, que descreve subagents síncronos como efêmeros e totalmente isolados, retornando apenas um resultado final.[web:18]

### Taxonomia de entidades

#### Tabela – Tipos de entidades em orquestração

| Conceito                          | Definição prática                                                                                                                                                                                                                              |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Agent                             | Loop de interação LLM + ferramentas + memória, com system prompt e política de ferramentas; pode ser superfície primária (chat principal) ou não.[web:1][web:3][web:7][web:10][web:24]                                                        |
| Main/primary agent                | Agente que representa a sessão principal do usuário (terminal/IDE). Orquestra subagents, toma decisões finais e produz a resposta ao humano.[web:1][web:3][web:10][web:11][web:24]                                                              |
| Subagent                          | Agente especializado, instanciado pelo main agent (ou por código) para subtarefa específica, com contexto, ferramentas e, às vezes, modelo isolados; tipicamente efêmero.[web:1][web:3][web:7][web:10][web:11][web:24]                         |
| Custom agent usado como subagent  | Definição de agent (arquivo .md/.agent.md/.toml etc.) que pode ser usado tanto como agente primário quanto como subagent dependendo da invocação (Claude, Gemini CLI, VS Code, Codex, OpenCode).[web:1][web:3][web:10][web:15][web:24]         |
| Background agent                  | Sessões de agent rodando em segundo plano, tipicamente supervisionadas em um “Agent Manager”/Mission Control, podendo ou não ser subagent direto de uma conversa (Claude background agents, Antigravity background subagents).[web:1][web:26][web:28] |
| Fork                              | Cópia do contexto (mensagens + estado) de um agent principal para uma nova ramificação, que pode seguir em direção diferente. Em alguns sistemas isso é um novo agent thread, não necessariamente tratado como “subagent”.[web:3][web:24]        |
| Agent team                        | Conjunto coordenado de agents, possivelmente cada um inicializado a partir de um subagent manifest, geridos por um “team orchestrator”. Em Claude Code, agent teams podem reutilizar definições de subagents como perfis de teammate.[web:1]         |
| Skill                             | Bloco de conhecimento ou comportamento reutilizável, frequentemente carregado no contexto ou exposto como ferramenta de alto nível; subagents podem ter skills próprios ou pré-carregados.[web:1][web:7]                                            |
| Tool                              | Função chamável pelo modelo (read_file, bash, web_search, mcp__github, etc.), com schema fixo e efeitos visíveis. Subagents recebem subconjunto das tools do agente principal ou ferramentas exclusivas.[web:1][web:7][web:10][web:11][web:24]       |
| MCP Server                        | Backend de ferramentas (Model Context Protocol) que oferece múltiplas tools; pode ser escopado a um subagent (Claude, Gemini CLI).[web:1][web:10][web:11]                                                                                         |
| Workflow/programmatic orchestration | Código (JS/Python) que escreve a orquestração (loops, branches, Promise.all/fan-out, map-reduce), em vez de depender do LLM para orquestrar via tool-calls em sequência (LangChain dynamic subagents, Codex CSV fan-out, Antigravity hooks).[web:3][web:8][web:9][web:28] |

---

## Comparativo por plataforma

### Matriz de plataformas e subagents

#### Claude Code

- **Nome usado**: “Subagents”; built‑in Explore, Plan, general-purpose, outros helpers.[web:1]
- **Definição**: specialized AI assistants que rodam em contexto próprio, com system prompt, tools e permissões independentes, retornando apenas resumo ao main agent.[web:1]
- **Invocação**:
  - Automática, por matching de `description` do subagent com a task do usuário.[web:1]
  - Explícita por linguagem natural (“use the code-improver agent”), ou por configuração/CLI (`--agents`, `/agents`).[web:1]
- **Contexto**:
  - Cada subagent tem contexto isolado (janela de contexto separada).[web:1]
  - Explore/Plan não carregam CLAUDE.md nem git status para baratear pesquisa; outros subagents carregam CLAUDE.md, git status, skills relevantes e opcionalmente memória persistente.[web:1]
  - Não recebe o system prompt completo do Claude Code, apenas o system prompt definido no arquivo + alguns detalhes do ambiente.[web:1]
- **Configuração**:
  - Arquivos Markdown em `.claude/agents/`, `~/.claude/agents/`, diretórios de plugin ou via `--agents` JSON.[web:1]
  - Frontmatter: `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation`, `color`, `initialPrompt`.[web:1]
- **Ferramentas/permissões**:
  - Inherit de ferramentas do main agent, mas podem ser restringidas por allowlist (`tools`) ou denylist (`disallowedTools`), incluindo padrões para MCP (`mcp__github`, `mcp__*`).[web:1]
  - `permissionMode` por subagent: `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`.[web:1]
  - MCP servers adicionais podem ser definidos apenas no subagent, isolando ferramentas de rede do main agent.[web:1]
  - Hooks (`PreToolUse`, etc.) podem aplicar políticas dinâmicas (ex.: bloquear SQL de escrita).[web:1]
- **Modelo por subagent**:
  - `model` por frontmatter, podendo ser alias (`sonnet`, `opus`, `haiku`) ou full ID; precedência: env `CLAUDE_CODE_SUBAGENT_MODEL`, param de invocação, frontmatter, modelo da conversa.[web:1]
- **Paralelismo**:
  - Subagents podem rodar em foreground ou background; o Agent tool permite spawn de múltiplos subagents e nested subagents se listado em `tools`.[web:1]
- **Recursão/nesting**:
  - Subagents podem spawnar subagents se tiverem tool `Agent` em `tools`; CLI `Agent(agent_type)` em agentes main restringe quais tipos podem ser spawnados.[web:1]
- **Retorno ao principal**:
  - Texto livre (resumo), possivelmente estruturado conforme system prompt; quando usado em agent teams, alguns campos do frontmatter são aplicados na criação de teammates.[web:1]
- **Limitações/documentadas**:
  - Alguns tools não estão disponíveis a subagents (`AskUserQuestion`, `ScheduleWakeup` etc.).[web:1]
  - O custo aumenta proporcionalmente ao número de subagents e turnos; `maxTurns` limita loops.[web:1]

#### OpenAI Codex

- **Nome usado**: subagent workflows, subagents, custom agents.[web:3]
- **Definição**: workflows em que Codex “runs parallel agents and combines their results” para lidar com context pollution/rot e paralelizar trabalho; subagent é um agente delegado iniciado para tarefa específica.[web:3]
- **Invocação**:
  - Explicitamente pelo usuário: “spawn two agents”, “delegate this work in parallel”, “use one agent per point”. Codex não cria subagents automaticamente.[web:3][web:5]
  - Para batch em CSV, `spawn_agents_on_csv` cria um subagent worker por linha.[web:5]
- **Contexto**:
  - Cada agent thread tem seu próprio contexto; o CLI permite alternar `/agent` entre threads e inspecionar atividades.[web:3][web:5]
  - Subagents herdam sandbox e overrides de permissões do pai (incluindo `/permissions` e `--yolo`).[web:5]
- **Configuração**:
  - Custom agents via TOML em `~/.codex/agents/` ou `.codex/agents/`, cada um com `name`, `description`, `developer_instructions`, e eventualmente `model`, `model_reasoning_effort`, `sandbox_mode`, `mcp_servers`, `skills.config`, `nickname_candidates`.[web:5]
  - Global `[agents]` no config controla `max_threads`, `max_depth`, `job_max_runtime_seconds`.[web:5]
- **Ferramentas/permissões**:
  - Subagents herdam sandbox policy e approvals do pai; sandbox pode ser override por agent (ex.: read-only).[web:5]
- **Modelo por subagent**:
  - Cada custom agent pode especificar `model` (ex.: `gpt-5.4-mini`), `model_reasoning_effort` (`low`/`medium`/`high`).[web:3][web:5]
- **Paralelismo**:
  - Codex lida com spawn de múltiplos agents em paralelo, espera por todos e consolida resultado.[web:3][web:5]
  - `spawn_agents_on_csv` cria um subagent por linha com `max_concurrency` configurável.[web:5]
- **Recursão/nesting**:
  - `agents.max_depth` controla profundidade de nesting; default 1 permite child direto mas evita delegação recursiva profunda.[web:5]
- **Retorno ao principal**:
  - Codex retorna resposta consolidada com resultados de todos os agents; no caso CSV, gera arquivo com colunas `job_id`, `item_id`, `status`, `result_json` etc.[web:3][web:5]
- **Limitações**:
  - Custo de tokens aumenta com número de subagents; docs enfatizam cuidado especial com workflows de escrita paralela.[web:3]

#### VS Code/Copilot

- **Nome usado**: subagents, runSubagent tool, custom agents as subagents.[web:24]
- **Definição**: subagent é um agente independente que executa trabalho focado (pesquisa, análise de código, review) e reporta de volta ao main agent; foco em context isolation e paralelismo.[web:24]
- **Invocação**:
  - Agent-initiated via tool `agent/runSubagent`; o usuário normalmente não digita “run subagent”.[web:24]
  - Hints via prompt (“perform isolated research…”, “run these reviews in parallel”) ajudam o main agent a decidir usar subagents.[web:24]
  - Em prompt files, incluir `'agent'` ou `runSubagent` em `tools` do frontmatter para permitir delegação.[web:24]
- **Contexto**:
  - Subagent roda em contexto isolado; principal recebe apenas resumo.[web:24]
- **Configuração**:
  - Custom agents (.agent.md) podem ser usados como subagents; frontmatter permite `user-invocable`, `disable-model-invocation`, `agents` (allowlist de subagents) etc.[web:24]
- **Ferramentas/permissões**:
  - Por agente, via frontmatter `tools`; `agents` controla quais outros agents podem ser usados como subagents.[web:24]
- **Modelo por subagent**:
  - Prioridade: model explícito no `runSubagent`; depois `model` do custom agent; depois modelo do main agent.[web:24]
- **Paralelismo**:
  - O main agent pode rodar múltiplos subagents em paralelo (exemplos de “Perform these tasks in parallel”).[web:24]
- **Recursão/nesting**:
  - Por padrão subagents não podem spawnar subagents; é possível habilitar nested subagents via `chat.subagents.allowInvocationsFromSubagents` com depth até 5.[web:24]
- **Retorno ao principal**:
  - Resumo consolidado; UI mostra subagent como seção colapsável com detalhes de tool calls, custo etc.[web:24]

#### Gemini CLI

- **Nome usado**: subagents.[web:10][web:11]
- **Definição**: specialists que o main agent “contrata” para tarefas específicas; cada subagent tem system prompt próprio, ferramentas especializadas e contexto independente.[web:10][web:11]
- **Invocação**:
  - Automática: o main agent decide usar subagent quando task bate com `description`.[web:10][web:11]
  - Explícita com `@agent` syntax (ex.: `@frontend-specialist`): força delegação a um subagent específico.[web:10][web:11]
  - Listagem e gestão via comando `/agents`.[web:10][web:11]
- **Contexto**:
  - Contexto separado (loop independente), com system prompt e ferramentas definidas; execução inteira consolidada numa única resposta ao main agent.[web:10][web:11]
- **Configuração**:
  - Arquivos .md com YAML frontmatter em `.gemini/agents` (projeto) ou `~/.gemini/agents` (user); extensões podem empacotar agents em `agents/`.[web:10][web:11]
  - Campos: `name`, `description`, `tools`, `mcpServers`, `model`, `temperature`, `max_turns`, `timeout_mins`, `kind` (local/remote).[web:11]
- **Ferramentas/permissões**:
  - `tools` com wildcards (`*`, `mcp_*`, `mcp_server_*`) define tool surface; `mcpServers` frontmatter adiciona MCP servers apenas ao subagent; Policy Engine pode aplicar regras por `subagent` name.[web:11]
- **Modelo por subagent**:
  - `model` frontmatter, default `inherit`; `modelConfigs.overrides` permite model configs específicos por subagent.[web:11]
- **Paralelismo**:
  - Suporte a “parallel subagents”: múltiplos subagents ou instâncias do mesmo tipo em paralelo; docs alertam para conflitos de edição e limite de uso.[web:10]
- **Recursão/nesting**:
  - Docs enfatizam “recursion protection”: subagents não podem chamar outros subagents, mesmo com wildcard de tools.[web:11]
- **Retorno ao principal**:
  - Resultado condensado (texto ou resultado conforme system prompt); main agent mantém contexto enxuto.[web:10][web:11]

#### Google Antigravity

- **Nome usado**: dynamic subagents, background subagents, Agent Manager/Mission Control; conceito de multi-agent orchestration no nível de projeto.[web:26][web:28]
- **Definição**:
  - Antigravity 2.0 é “agent-first workspace” com dynamic subagents: main agent pode dinamicamente definir e invocar subagents para subtarefas focadas, preservando contexto e permitindo paralelismo.[web:28]
  - Agent Manager (Mission Control) orquestra múltiplos agentes locais em paralelo.[web:26][web:28]
- **Invocação**:
  - Não há ainda docs públicas detalhando definição de subagents customizáveis no mesmo nível de Claude/Gemini; posts destacam dynamic subagents e scheduled tasks, e fórum indica que custom subagents estilo Claude Code ainda são feature pedida, não entregue.[web:13][web:26][web:28]
- **Contexto / ferramentas / modelo / recursão**:
  - Sites oficiais falam em dynamic subagents e múltiplos agentes por projeto, mas não expõem o manifesto público de subagents ou os limites de nesting/custo.[web:26][web:28]
  - Fórum deixa claro que hoje Antigravity tem Agent Manager que delega a sub-agents “internos” (Browser subagent, Terminal subagent), mas falta suporte a custom subagents configuráveis com modelos diferentes.[web:13]
- **Conclusão factual**:
  - Não há evidência oficial suficiente, nas fontes públicas atuais, para descrever schema exato de subagent manifest, regras de sandbox e limites de profundidade ou tokens em Antigravity.

#### OpenCode

- **Nome usado**: Primary Agents e Subagents; agentes em geral.[web:15]
- **Definição**:
  - Primary Agents: agentes principais com os quais o usuário interage (Build, Plan, etc.).[web:15]
  - Subagents: assistentes auxiliares que podem ser invocados pelos Primary Agents ou via `@mention` para tarefas específicas.[web:15]
- **Invocação**:
  - Automática por agentes principais com base em `description` dos subagents.[web:15]
  - Manual via `@subagent` em mensagens.[web:15]
- **Contexto**:
  - O texto descreve que subagents podem abrir “sesje podrzędne” (sessões filhas) e se navega parent↔child com atalhos, o que implica contexto separado por sessão.[web:15]
- **Configuração**:
  - JSON em `opencode.json` ou Markdown em `.opencode/agents/` e `~/.config/opencode/agents/`.[web:15]
  - Campos: `description` (obrigatório), `prompt` (system prompt), `model`, `tools`, `permissions`, `mode` (`primary`, `subagent` ou `all`), `hidden`, `permission.task` (quais subagents podem ser invocados), `temperature`, `steps`, `disabled`, `color`, `top_p` e passthrough para parâmetros de provedor.[web:15]
- **Ferramentas/permissões**:
  - `tools` permite habilitar/desabilitar tools, com wildcards para MCP; `permissions` controla `edit`, `bash`, `webfetch` com `ask/allow/deny`; `permission.task` controla quais subagents um agente pode chamar.[web:15]
- **Modelo por subagent**:
  - `model` por agente, com possibilidade de usar modelos diferentes para Plan vs Build etc.[web:15]
- **Paralelismo**:
  - Multi-sessão com parent/child, mas docs não detalham fan-out massivo; o modelo mental é similar ao Claude/Gemini, porém menos explícito em termos de concorrência e limites.[web:15]
- **Recursão/nesting**:
  - Não há seção explícita sobre limites de profundidade; `permission.task` com globs e `mode` permitem restringir quais subagents podem chamar outros.

#### LangChain Deep Agents

- **Nome usado**: subagents, dynamic subagents.[web:7][web:8][web:9]
- **Definição**:
  - Deep agent pode criar subagents para delegar trabalho; subagents são usados para **context quarantine** e instruções/skills especializados.[web:7]
  - Dynamic subagents: subagents despachados via código JS em interpreter (QuickJS), usando global `task()` dentro de middleware de code interpreter.[web:8][web:9]
- **Invocação**:
  - Síncronos: via `task()` tool exposto pelo `SubAgentMiddleware` quando há subagents configurados; main agent decide chamar `task(subagentType=...)`.[web:7][web:18]
  - Dinâmicos: com interpreter, o modelo escreve JS que chama `task({ description, subagentType, responseSchema })` em loops/Promise.all.[web:8][web:9]
  - Palavra “workflow” no prompt é usada como trigger para optar por orquestração dinâmica.[web:8][web:9]
- **Contexto**:
  - Subagents são efêmeros, com memória isolada; o supervisor “blocks until subagent finishes” e obtém apenas o resultado final.[web:7][web:18]
  - Skills podem ser isoladas por subagent; `skills` em config define paths de skills separados para cada subagent.[web:7]
- **Configuração**:
  - `subagents` como lista de dicts `SubAgent` com campos: `name`, `description`, `system_prompt`, `tools`, `model`, `middleware`, `interrupt_on`, `skills`, `response_format`, `permissions`.[web:7]
  - `CompiledSubAgent` aceita `runnable` (LangGraph compilado), `name`, `description`.[web:7]
- **Ferramentas/permissões**:
  - `tools` por subagent; `permissions` substituem permissões do pai, permitindo FS sandbox específico.[web:7]
- **Modelo por subagent**:
  - `model` pode ser string (`openai:gpt-5.5`, `google_genai:gemini-3.5-flash`) ou chat model instanciado; se omitido, herda modelo do main agent.[web:7]
- **Paralelismo**:
  - O main agent pode fazer várias chamadas `task()` em uma única rodada para rodar subagents em paralelo; dynamic subagents com `Promise.all` são o padrão fan-out + synthesize.[web:7][web:8][web:9]
- **Recursão/nesting**:
  - Dynamic subagents permitem workflows recursivos (Recursive Language Models), incluindo padrões como Loop Until Done; limite é controlado pelo código + limites de execução do interpreter.[web:8][web:9]
- **Retorno ao principal**:
  - Sem `response_format`, o supervisor recebe texto final do subagent; com `response_format`, recebe JSON estruturado conforme schema (Pydantic ou schema raw).[web:7]

#### Ollama + Claude Code

- **Nome usado**: Subagents (no contexto da integração), web search.[web:12]
- **Definição**: subagents podem executar tarefas em paralelo (file search, code exploration, research), cada um com seu próprio contexto, usando modelos rodando em Ollama cloud; integração expõe subagents de Claude Code sem MCP nem API keys.[web:12]
- **Invocação**:
  - Alguns modelos naturalmente usam subagents quando necessário (minimax-m2.5, glm-5, kimi-k2.5); o usuário pode forçar com prompts como “spawn subagents…”, “create subagents to…” etc.[web:12]
- **Contexto/ferramentas/modelos**:
  - Cada subagent tem contexto próprio; podem usar web search embutido na camada de compatibilidade Anthropic da Ollama; não há manifesto próprio de subagents em Ollama — ela apenas hospeda o endpoint que suporta subagents do lado Claude Code.[web:12]
- **Limitações**:
  - Documentação da Ollama não detalha tools/permissões por subagent; assume-se que isso é herdado da configuração do Claude Code. Não há evidência oficial suficiente para descrever isolamento de ferramentas além do texto genérico.

---

## Modelo mental unificado de subagent orchestration

Com base nas plataformas analisadas, é possível extrair um modelo abstrato com componentes recorrentes.

### Componentes principais

1. **Subagent registry**
   - Tabela de subagents registrados, indexados por `name` e `description`, com metadados como `model`, `tools`, `permissions`, `mcpServers`, `memory`, `maxTurns`, `timeout`, etc.[web:1][web:3][web:7][web:10][web:11][web:15]
   - Em Claude/Gemini/OpenCode, esse registry é montado a partir de arquivos de manifesto (Markdown/TOML/JSON) e/ou configuração CLI.[web:1][web:10][web:11][web:15]

2. **Subagent manifest**
   - Definição declarativa de um subagent: `name`, `description`, `system_prompt`, `model`, `tools`, `disallowed_tools`, `mcp_servers`, `permissions`, `memory`, `max_turns`, `timeout`, `isolation_mode`, etc.[web:1][web:7][web:10][web:11][web:15]
   - Base para carregar agentes tanto como subagents quanto como agentes primários (VS Code, Gemini CLI, Codex, Claude).[web:1][web:3][web:10][web:24]

3. **Description-based routing**
   - Main agent compara a task (ou o histórico) com `description` dos subagents para decidir se delega e para qual subagent (Claude Code, Gemini CLI, Deep Agents).[web:1][web:7][web:10][web:11]
   - Descriptions precisam conter domínios, gatilhos (“Use PROACTIVELY after doc changes”, “Git expert agent…”) para boa roteamento automático.[web:1][web:11]

4. **Explicit invocation**
   - Usuário ou orquestrador programático pode forçar um subagent específico via @mention (Gemini CLI, OpenCode), via tool (`task`, `Agent`, `runSubagent`), ou via prompt explícito (“use the security-reviewer agent”).[web:1][web:3][web:7][web:10][web:11][web:15][web:24]

5. **Context packer**
   - Responsável por montar o contexto que será entregue ao subagent: system prompt do subagent, task prompt, alguns detalhes do ambiente (ex.: diretório de trabalho, config do projeto, memory, skills, snapshot de git), mas não o histórico completo do main agent.[web:1][web:7][web:18]
   - Em Deep Agents, o subagent recebe state próprio (`messages`), separado do supervisor; Claude/Gemini/OpenCode fazem algo análogo na camada de runtime.[web:1][web:7][web:11][web:15]

6. **Tool/permission sandbox**
   - Compõe lista efetiva de tools e permissões para o subagent a partir de manifest + políticas globais (permissions, Policy Engine, enterprise policies).[web:1][web:5][web:7][web:10][web:11][web:15]
   - Inclui scoping de MCP servers, filesystem permissions, bloqueio de writes, approval modes, etc.[web:1][web:5][web:7][web:10][web:11]

7. **Execution runtime**
   - Loop de execução do subagent: LLM + tool calling + políticas; pode ser síncrono (supervisor bloqueia) ou assíncrono/background.[web:1][web:7][web:18]
   - Em dynamic subagents (Deep Agents, Antigravity, workflows de Claude/Codex), este runtime é acionado via interpreter (JS) que chama `task()` ou equivalente.[web:8][web:9][web:28]

8. **Scheduler/concurrency controller**
   - Controla quantos subagents podem rodar em paralelo (`agents.max_threads` em Codex, limites internos em outros), e possivelmente define grupos de concorrência por tarefa (não documentado de forma uniforme).[web:5]

9. **Result collector**
   - Fase fan-in: espera todos subagents escolhidos, coleta seus outputs (texto ou JSON) e os normaliza para consumo do supervisor.[web:3][web:7][web:9][web:10][web:24]

10. **Synthesizer**
    - Supervisor (main agent) sintetiza resultados dos subagents em uma resposta final ou em um plano consolidado, possivelmente rodando mais um passo de raciocínio.[web:3][web:7][web:9][web:24]

11. **Memory/transcript manager**
    - Controle de logs e memória: subagents têm transcripts próprios (e, se habilitado, pastas de memory dedicadas) que podem ser persistidos em disco ou observabilidade (LangSmith, Antigravity artifacts).[web:1][web:7][web:18][web:26]

12. **Approval/guardrail layer**
    - Aplica políticas de segurança, approvals de file write/bash, domain allowlists, restrictions de MCP por subagent e por tarefa; herdadas do main agent + overrides do manifest.[web:1][web:5][web:7][web:10][web:11][web:15]

13. **Observability UI/logs**
    - UI/CLI para visualizar subagents vivos, histórico, custo, logs de tool calls e resultados (Claude `/agents` Running tab, Gemini `/agents`, VS Code subagent sections, Deep Agents + LangSmith, Antigravity Mission Control).[web:1][web:7][web:10][web:24][web:26]

---

## Padrões de execução

As plataformas estudadas convergem em um conjunto de padrões de orquestração, alguns explicitamente nomeados em Deep Agents (dynamic subagents) e Codex.

### Fan-out + synthesize

- **Descrição**: main agent distribui tarefa homogênea em muitos itens (arquivos, módulos, linhas de CSV), cria um subagent por item (ou por partition) e depois sintetiza os resultados.[web:3][web:7][web:8][web:9]
- **Exemplos oficiais**:
  - Codex: spawn one agent per review point (security, code quality, bugs etc.), summarizar por categoria.[web:3]
  - Deep Agents: “fanout and synthesize” para revisar cada arquivo em `src/` com um subagent reviewer.[web:9]
  - Gemini CLI: rodar frontend-specialist em cada package em paralelo.[web:10]

### Classify + route

- **Descrição**: primeiro passo classifica itens (bug/feature/question, componente A/B/C), depois roteia cada item a subagent especializado.[web:9]
- **Exemplos**: LangChain dynamic subagents “Classify and act”: triagem de tickets, logs, feedbacks.[web:9]

### Adversarial verification

- **Descrição**: subagent A gera hipóteses/achados; subagents B/C verificam de forma independente, mantendo apenas o que passa pelo consensus ou majority vote.[web:9]
- **Exemplos**: Deep Agents “Adversarial verification” para auditoria de segurança com verifiers independentes.[web:9]

### Generate + judge/filter

- **Descrição**: múltiplos subagents geram alternativas; outro subagent (judge) ou o main agent avalia e seleciona melhor opção, às vezes via torneio.
- **Exemplos**: Deep Agents “Generate and filter” e “Tournament” para propostas de arquitetura, refactors, reescrita de funções.[web:9]

### Pipeline sequencial

- **Descrição**: cadeia de subagents, onde o output de um alimenta o próximo (ex.: Planner → Architect → Implementer → Reviewer).[web:24]
- **Exemplos**: VS Code “Coordinator and worker pattern” (Feature Builder + Planner + Plan Architect + Implementer + Reviewer).[web:24]

### Map-reduce por arquivo/módulo

- **Descrição**: map stage com subagent por arquivo/módulo, reduce stage com synthesizer; equivalente a fan-out + synthesize, mas enfatizando agregação com estrutura (ex.: relatórios por módulo, por serviço).[web:7][web:9][web:18]

### Batch via CSV/lista

- **Descrição**: spawn de workers por item de CSV, com output consolidado em outro CSV com result_json e metadados (Codex `spawn_agents_on_csv`).[web:5]

### Background subagents

- **Descrição**: subagents rodando assíncronos, não bloqueando main agent; podem reportar artefatos e resultados depois (Claude background agents, Antigravity scheduled tasks).[web:1][web:26][web:28]

### Nested subagents

- **Descrição**: subagents que podem spawnar outros subagents (VS Code nested, Claude nested via Agent tool, Deep Agents dynamic subagents com loops recursivos).[web:1][web:7][web:8][web:24]
- **Limites**: VS Code impõe depth máximo 5; Codex controla via `agents.max_depth`; outras plataformas não documentam explicitamente limites de profundidade, apenas alertam para custo/recursão.[web:5][web:24]

### Forked context

- **Descrição**: fork do contexto da sessão principal em threads/agents separados (Codex agent threads, OpenCode parent/child sessions, Antigravity Projects + Agents, Claude Agent tool lançando subagents com isolation worktree).[web:1][web:3][web:15][web:26]

### Dynamic subagents via código/interpreter

- **Descrição**: orquestração escrita em código (JS via QuickJS, Python etc.) que invoca `task()`/equivalente para despachar subagents com loops, branches, Promise.all; pioneiro em Deep Agents, semelhante à ideia de Recursive Language Models.[web:8][web:9]

---

## Diferença crítica: subagents vs agents externos

Plataformas como Claude Code, Codex, Gemini CLI, OpenCode e Deep Agents chamam suas definições de “agents” nos manifests, mas **funcionalmente** eles se comportam como subagents quando:

- são instanciados e controlados pelo main agent ou por um harness/CLI, não pelo usuário diretamente;[web:1][web:3][web:7][web:10][web:15]
- operam dentro do mesmo processo/ambiente de execução, compartilhando políticas e infraestrutura (sandbox, Policy Engine, MCP config, fs permissions);[web:1][web:5][web:7][web:10][web:11]
- têm ciclo de vida efêmero, ligado a uma tarefa (Deep Agents synchronous subagents, Codex subagent workflows, Gemini subagents);[web:3][web:7][web:10][web:11][web:18]
- retornam resumo ou JSON para o main agent em vez de se tornarem superfícies independentes para o usuário.[web:3][web:7][web:10][web:24]

“Custom agent” em Codex/Gemini/VS Code/OpenCode é, portanto, um **profile** que pode ser usado como agent principal, mas, na maior parte das vezes, atua como subagent quando é invocado via tool ou @mention a partir de outra sessão.[web:3][web:10][web:15][web:24]

### Ambiguidade terminológica

- **Claude Code** usa “subagents” para esses perfis especializados, e “agent teams” para múltiplas sessões que reutilizam subagent definitions como teammates.[web:1]
- **Codex** fala em “custom agents” e “subagent workflows”: o manifesto define um agent, que se torna subagent quando usado num workflow spawnado por outro agent.[web:3][web:5]
- **VS Code** fala em “agents” (custom agents) e “subagents” quando esses agentes são invocados via `runSubagent`/tool.[web:24]
- **Gemini CLI** chama todos de “agents”, mas especificamente “subagents” para os especialistas delegados pelo main session; mesmo arquivo .md pode atuar como subagent ou como agent primário, dependendo da invocação.[web:10][web:11]
- **OpenCode** diferencia explicitamente “Primary Agents” e “Subagents” com campo `mode`.[web:15]
- **LangChain Deep Agents** usa “subagents” para workers do supervisor e “dynamic subagents” para aqueles invocados via interpreter.[web:7][web:8][web:9]

Para um orquestrador próprio, a recomendação é manter a distinção conceitual clara:

- **Agent profile** (manifesto) = definição reutilizável.
- **Main agent instance** = uso desse profile como sessão principal do usuário.
- **Subagent instance** = uso desse profile como worker delegado pelo main agent.

---

## Design de um orquestrador próprio de subagents

### Arquitetura proposta

Inspirada pelos padrões observados, uma arquitetura prática para seu orquestrador poderia ter os seguintes componentes concretos:

1. **Manifest loader & registry**
   - Carrega manifests (YAML/TOML) de diretórios como `~/.myagent/agents/` (user), `.myagent/agents/` (projeto) e diretórios de extensões/plug-ins.
   - Resolve prioridades de override (org > CLI > project > user > plugin), similar a Claude.[web:1]

2. **Agent profile model**
   - Estrutura de dados que representa um manifesto: campos mínimos e avançados (ver manifest exemplo abaixo).
   - Permite uso tanto como main agent quanto como subagent, com flags `mode` e `can_spawn_subagents`.

3. **Router**
   - Implementa description-based routing (matching simples ou embedding) + explicit invocation (@mention, tool) + regras programáticas (workflow code).
   - Expõe API `route(task)` que retorna lista de subagents candidatos com scores.

4. **Context packer**
   - Dado o main context, manifesto e task, monta input para o subagent: system prompt, task prompt, environment snapshot (ex.: `CLAUDE.md`-equivalente), memória escopada, skills, git status (opcional).[web:1][web:7][web:18]

5. **Tool/permission sandbox**
   - Constrói allowlist/denylist de tools por subagent, aplicando políticas globais (Policy Engine) e regras específicas de manifesto (tools, disallowed_tools, mcp_servers, permissions).[web:1][web:5][web:7][web:10][web:11][web:15]
   - Suporta scoping de servidores externos (MCP-equivalentes) a subagents.

6. **Execution engine**
   - Responsável por rodar o loop do subagent (chamadas LLM + ferramentas) até `max_turns`, `timeout` ou conclusão.
   - Suporta modos síncrono (blocking) e assíncrono (background) com cancelamento e reattach.

7. **Scheduler/concurrency controller**
   - Gestão de fan-out: fila de tasks, pools por `concurrency_group`, limites `max_parallel_subagents` globais e por grupo, timeouts por job (similar a Codex `job_max_runtime_seconds`).[web:5]

8. **Result collector & synthesizer**
   - Coleção de outputs dos subagents e entrega ao main agent em formato estruturado; synthesizer pode ser outro LLM pass ou lógica estática (map-reduce, merges, deduplicação, ordens de severidade etc.).[web:3][web:7][web:9][web:24]

9. **Error handling + guardrails**
   - Tratamento de timeouts, falhas de tool, violações de política, loops infindáveis; fallback para main agent ou para outra estratégia (ex.: repeat with fewer subagents).[web:3][web:5][web:7]

10. **Observability/Logs UI**
    - UI mínima com lista de subagents ativos, estado (Running/Completed/Failed), custo estimado, transcripts por subagent e árvore parent/child.[web:1][web:7][web:10][web:26]

11. **Nesting support**
    - Estrutura para depth limitado (configurável); recusar spawn quando depth > N, similar a `agents.max_depth` (Codex) ou `allowInvocationsFromSubagents` (VS Code).[web:5][web:24]

### Manifesto genérico em YAML

Exemplo inspirado em Claude Code, Gemini CLI, Deep Agents e OpenCode (campos agregados; ajuste conforme sua stack):

```yaml
---
name: security-reviewer
description: >-
  Security-focused code reviewer. Use for analyzing authentication, authorization,
  and data exposure issues in backend services.

system_prompt: |-
  You are a senior security engineer. When invoked, review the provided code
  for security issues. Focus on authentication, authorization, input validation,
  injection, sensitive data exposure, and logging.

model: claude-sonnet-5

# Tools explicitly permitidos
tools:
  - read_file
  - grep_search
  - glob
  - list_directory
  - web_fetch

# Tools explicitamente negados (override de herança)
disallowed_tools:
  - write_file
  - edit_file
  - bash

# Configuração de MCP/servidores externos apenas para este subagent
mcp_servers:
  github:
    type: http
    url: https://my-mcp-github-proxy

permissions:
  fs:
    # Exemplo de DSL de permissão; detalhe a seu critério
    - path: ./src
      actions: [read]
    - path: ./infra
      actions: [read]
      mode: ask

max_turns: 12

timeout: 600   # segundos

concurrency_group: security

output_schema:
  type: object
  properties:
    findings:
      type: array
      items:
        type: object
        properties:
          file:
            type: string
          line:
            type: integer
          severity:
            type: string
          description:
            type: string

memory_scope: project   # user | project | local | none

can_spawn_subagents: false

isolation_mode: worktree  # none | worktree | container
```

Esse schema é coerente com os campos das principais plataformas, ainda que com nomenclatura consolidada.[web:1][web:7][web:10][web:11][web:15]

### Decisões de design a evitar (à luz das plataformas)

- **Subagents com acesso irrestrito a todas as tools + bypassPermissions**: Claude alerta que `bypassPermissions` permite operações perigosas sem prompt; deve ser exceção, não regra.[web:1]
- **Nesting sem limites explícitos**: Codex e VS Code documentam riscos de `max_depth` alto; sem limites, fan-out recursivo pode explodir custo e latência.[web:5][web:24]
- **Paralelismo write-heavy sem coordenação**: Codex e Gemini CLI enfatizam cuidado com múltiplos agents editando código simultaneamente; sem mecanismos de lock/diff/merge, o risco de conflitos é alto.[web:3][web:10]
- **Subagents genéricos demais**: docs de Codex sugerem “narrow opinionated roles”; subagents muito genéricos viram mais um generalist agent, perdendo vantagem de especialização e governança.[web:5]

---

## Riscos e trade-offs

Principais riscos levantados nas docs e inferências técnicas baseadas nelas:

- **Aumento de custo/tokens**: cada subagent possui seu próprio histórico, tool calls e possivelmente modelo mais caro; Codex e Deep Agents enfatizam que workflows com muitos subagents consomem bem mais tokens.[web:3][web:7][web:9]
- **Perda de previsibilidade**: description-based routing pode escolher subagent “errado” se descrições forem ambíguas; VS Code e Gemini CLI sugerem escrever descrições e exemplos claros.[web:11][web:24]
- **Loops recursivos**: agentes que podem spawnar outros sem `max_depth` ou guardrails podem entrar em loop (RecursiveProcessor em VS Code só é seguro porque depth máximo é 5 e prompt impõe stop conditions).[web:24]
- **Excesso de paralelismo**: sem scheduler, fan-out massivo pode saturar limites de API, IO, e aumentar latência por competição de recursos; Codex introduz `agents.max_threads` e `job_max_runtime_seconds` para mitigar.[web:5]
- **Conflitos de edição**: múltiplos subagents write-heavy atuando no mesmo workspace sem isolamento (worktree/branch/tmp-dir) aumentam risco de conflitos; Claude oferece `isolation: worktree` para minimizar isso.[web:1][web:3]
- **Permissões perigosas**: modos como `bypassPermissions` (Claude) ou sandbox laxo (Codex) exigem uso criterioso; Policy Engines por subagent (Gemini CLI) ajudam a mitigar.[web:1][web:5][web:11]
- **Prompt injection & context leakage**: subagents que acessam web/docs devem ter instruções robustas para não aceitar instruções maliciosas de conteúdo externo; tool isolation e domain allowlists (browser agent em Gemini CLI) são contramedidas.[web:11]
- **Resultados inconsistentes**: diferentes subagents podem chegar a conclusões conflitantes; padrões como adversarial verification e judge/filter ajudam, mas aumentam custo.[web:9]
- **Dificuldade de debugging**: muitos subagents paralelos tornam rastreio complexo; LangSmith (Deep Agents), Agent Manager (Antigravity), `/agents` (Claude/Gemini) e UI de VS Code são respostas a isso.[web:1][web:7][web:10][web:24][web:26]
- **Quando não usar subagents**:
  - Tasks simples, one-shot.[web:7]
  - Quando o valor de manter todo o contexto em um fluxo único é maior do que o custo de bloat (ex.: debugging interativo curto).[web:18]
  - Quando overhead de spawn de subagents é comparável ao próprio trabalho.

---

## Recomendações práticas (MVP, intermediário, avançado)

### MVP

Baseado no que as plataformas consideram “default subagent support”:[web:1][web:3][web:7][web:10][web:11]

- **Subagent registry**: carregar manifests de 1–2 diretórios (`user`, `project`).
- **Prompt isolado**: cada subagent com `system_prompt` próprio e contexto minimal (task + env snapshot curto).
- **Tool allowlist**: `tools` obrigatório; sem allowlist implícito, para evitar que subagents herdarem tudo automaticamente.
- **Task prompt enxuto**: main agent envia a menor quantidade de contexto possível (ex.: apenas caminhos relevantes, trechos-chave, instruções e expected output format).
- **Execução paralela limitada**: limite global pequeno (ex.: 4 subagents simultâneos); sem nested subagents inicialmente.
- **Retorno em JSON/schema**: exigir `output_schema` por subagent que produza dados, para facilitar fan-in determinístico.[web:7][web:9]
- **Sintetizador central**: uma etapa de síntese no main agent, possivelmente com LLM ou lógica estática.
- **Logs por subagent**: transcript separado por subagent, com ID, nome, ferramentas usadas, custo estimado, status.

### Versão intermediária

- **Nesting limitado**: permitir `can_spawn_subagents` com depth máximo (ex.: 2–3), inspirado em Codex/VS Code.[web:5][web:24]
- **Background subagents**: tasks long-running em background com API para consultar progresso e outputs (similar a background agents de Claude e scheduled tasks de Antigravity).[web:1][web:26][web:28]
- **Política de permissões avançada**: Policy Engine com regras `subagent=name` (Gemini CLI style) para permitir/negas ferramentas e domínios específicos.[web:11]
- **Worktree/container isolation**: modo de isolamento do workspace para subagents write-heavy.
- **Dynamic subagents simples**: camada de interpreter leve que permite scripts de fan-out + synthesize em JS/Python chamando a API de spawn do orquestrador (inspirado em Deep Agents dynamic subagents).[web:8][web:9]

### Versão avançada

- **Full dynamic subagents/Recursive LM**: permitir que o modelo escreva código de orquestração (JS) executado em interpreter, com global `task()` para spawn de subagents, replicando o modelo Deep Agents/Antigravity workflows.[web:8][web:9][web:28]
- **Observabilidade rica**: integrações com painel tipo LangSmith, com filtros por `subagent_name`, árvore de runs e métricas agregadas.[web:7]
- **Policy Engine multi-camada**: policies por projeto, por subagent, por tool, por domínio externo; approvals dinâmicos estilo Claude permissions & Gemini Policy Engine.[web:1][web:11]
- **Agent teams internos**: permitir composições mais complexas, com orquestrador de times (Planner, Architect, Implementer, Reviewer) definido via manifests, semelhante a VS Code coordinator pattern e Claude agent teams.[web:1][web:24]

---

## Conclusão – convergência de mercado e arquitetura mínima defensável

### O que os grandes players estão convergindo

Das fontes oficiais, há convergência em:

- **Subagents como mecanismo primário de context isolation** para tarefas ruidosas/longas.[web:3][web:7][web:9][web:10][web:11][web:18]
- **Manifests declarativos** com name/description/system_prompt/model/tools/permissions.[web:1][web:5][web:7][web:10][web:11][web:15][web:24]
- **Delegação mista**: description-based routing + explicit invocation (@mention/tool).[web:1][web:7][web:10][web:11][web:24]
- **Paralelismo controlado** (fan-out/fan-in), mas com forte recomendação de cuidado em writes em paralelo.[web:3][web:7][web:9][web:10]
- **Structured output opcional** para subagents, facilitando workflows programáticos.[web:7][web:5]
- **Nesting limitado** para evitar loops e explosão de custos.[web:5][web:24]

### Features essenciais para um orquestrador de subagents

- Registry de subagents baseado em manifestos.
- Contexto isolado por subagent (janela de contexto separada, sem herdar histórico completo do main agent).
- Tool/permission sandbox com allowlist/denylist por subagent.
- Suporte a modelo por subagent (com fallback para o modelo principal).
- Fan-out/fan-in básico, com scheduler simples e limites de concorrência.
- Retorno estruturado (JSON/schema) e sintetizador central.
- Logs/transcripts por subagent com identificação clara e status.

### Nice-to-have

- Dynamic subagents via interpreter (JS/Python) com `task()` global.[web:8][web:9]
- Agent teams internos pré-configurados (Planner/Architect/Implementer/Reviewer).[web:1][web:24]
- Policy Engine refinado com regras por subagent, tool, domínio externo.[web:11]
- UI rica de observabilidade (timeline, fase por fase, custos por subagent).[web:7][web:26]
- Background/scheduled subagents (cron-like workflows).[web:26][web:28]

### Arquitetura mínima defensável para começar

Com base na literatura técnica de produto, uma arquitetura mínima defensável para seu orquestrador de subagents deveria oferecer, no MVP:

1. **Manifestos em YAML/TOML** com campos: `name`, `description`, `system_prompt`, `model`, `tools`, `disallowed_tools`, `permissions`, `max_turns`, `timeout`, `output_schema`, `memory_scope`, `can_spawn_subagents`, `isolation_mode`.
2. **Registry + loader** para buscar esses manifests por diretório de user/projeto.
3. **Router** com description-based routing simples + explicit invocation.
4. **Context packer** que envia apenas task + env snapshot ao subagent.
5. **Tool/permission sandbox** por subagent.
6. **Execução paralela limitada**, com limite global de subagents simultâneos e timeouts.
7. **Retorno em JSON** conforme `output_schema` + synthesizer central.
8. **Logs/transcrições** por subagent, com árvore parent/child e estados.

Essa base já te coloca, conceitualmente, no mesmo espaço de Claude Code, Gemini CLI, Deep Agents e OpenCode em termos de orquestração interna de subagents, mantendo espaço para evoluir depois para dynamic subagents e agent teams.[web:1][web:7][web:10][web:11][web:15]

---

## Bibliografia

### Fontes oficiais

- Anthropic – Claude Code Docs – Create custom subagents.[web:1]
- OpenAI – Codex Subagents – Concepts e Approvals and sandbox controls.[web:3][web:5]
- Visual Studio Code Docs – Subagents in Visual Studio Code.[web:24]
- LangChain Docs – Subagents e Dynamic subagents (Deep Agents).[web:7][web:8]
- LangChain Blog – Introducing Dynamic Subagents in Deep Agents.[web:9]
- Google Developers Blog – Subagents have arrived in Gemini CLI.[web:10]
- Gemini CLI Docs – Subagents.[web:11]
- Google Antigravity – Site oficial e blog “Introducing Google Antigravity 2.0”.[web:26][web:28]
- OpenCode Docs – Agenci.[web:15]
- Ollama Blog – Subagents and web search in Claude Code.[web:12]

### Fontes secundárias/contextuais

- Google AI Developers Forum – Discussão “Antigravity sub agents”.[web:13]
- YouTube/DevStories EU – Deep Agents · Context Isolation with Synchronous Subagents (transcrição textual).[web:18]

### Fontes acadêmicas/conceituais

- LangChain Docs/Blog – Referências a Recursive Language Models e workflows de orquestração por código (interpreters, dynamic subagents).[web:8][web:9]

> Nos pontos em que este relatório extrapola para recomendações de design (por exemplo, campos de manifesto agregados, arquitetura mínima), trata-se de **inferência técnica** baseada em padrões observados nas documentações citadas, não de afirmações documentadas literalmente nas fontes oficiais.
