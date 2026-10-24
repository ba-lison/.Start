# Estrutura `.memory/` — Memória Persistente de Projeto para Agentes de IA

## Visão Geral

A pasta `.memory/` é uma camada de conhecimento versionada e persistente dentro de um repositório Git, projetada para que agentes de IA possam consultar contexto acumulado antes de iniciar qualquer tarefa e registrar aprendizados após concluí-las. A arquitetura adota o princípio de que **Markdown é a fonte da verdade** e índices (SQLite, vetorial) são derivados e reconstruíveis. O padrão "Memory-as-Governance" — popularizado pelo projeto `projectmem` — adiciona um portão pré-ação que impede o agente de repetir tentativas já fracassadas ou editar arquivos conhecidamente frágeis.[^1][^2][^3]

***

## Árvore de Diretórios Completa

```
.memory/
├── 00_INDEX.md                      # Ponto de entrada: o agente lê ESTE arquivo primeiro
├── META/
│   ├── project_profile.json         # Perfil do projeto (metadados estruturados)
│   ├── metadata.json                # Metadados da própria pasta .memory/
│   └── update_rules.md              # Regras de atualização e governança da memória
├── ARCHITECTURE/
│   ├── module_map.md                # Mapa de módulos e responsabilidades
│   ├── architectural_decisions.md   # Resumo das decisões arquiteturais
│   └── adr/
│       ├── ADR-0001-*.md            # ADRs individuais (append-only, imutáveis após Accepted)
│       ├── ADR-0002-*.md
│       └── _adr_index.json          # Índice JSON de todos os ADRs
├── KNOWLEDGE/
│   ├── glossary.md                  # Glossário do domínio
│   ├── code_conventions.md          # Convenções de código do projeto
│   ├── test_patterns.md             # Padrões e estratégias de teste
│   └── useful_commands.md           # Comandos frequentes e aliases
├── INTEGRATIONS/
│   ├── external_integrations.md     # Integrações externas (APIs, serviços, SDKs)
│   └── important_dependencies.md    # Dependências críticas e versões fixadas
├── HISTORY/
│   ├── migration_history.md         # Histórico de migrações (DB, infraestrutura, stack)
│   ├── resolved_bugs.md             # Bugs resolvidos com causa-raiz e solução
│   └── failed_attempts.md           # Tentativas fracassadas (o que NÃO fazer)
├── RISK/
│   ├── fragile_areas.md             # Áreas frágeis do codebase (gate pré-ação)
│   └── known_risks.md               # Riscos conhecidos e mitigações
├── EVENTS/
│   ├── event_log.jsonl              # Log append-only de eventos (imutável)
│   └── _event_schema.json           # Schema dos eventos
├── SEARCH/
│   ├── semantic_index.json          # Manifesto de indexação semântica
│   └── .gitignore                   # Ignora arquivos binários de embeddings
└── ARCHIVE/
    ├── _archive_index.md            # Índice de memórias arquivadas
    └── obsolete/
        └── YYYY-MM-DD_<slug>.md     # Memórias obsoletas arquivadas com data
```

***

## Explicação de Cada Arquivo

### `00_INDEX.md` — Ponto de Entrada do Agente

O agente **deve ler este arquivo primeiro** em toda sessão. Contém links rápidos para todos os blocos de memória, status geral do projeto e ponteiros para o contexto ativo.[^4]

### `META/project_profile.json`
Perfil estruturado do projeto: nome, stack, objetivo, equipe, ambiente. Permite ao agente contextualizar qualquer tarefa sem ler todos os arquivos.[^5]

### `META/metadata.json`
Metadados da própria pasta `.memory/`: versão do schema, data da última atualização, agentes que contribuíram, estatísticas de saúde da memória.

### `META/update_rules.md`
Define **quando** e **como** o agente deve atualizar a memória, quais arquivos requerem revisão humana e regras de expiração.[^6][^7]

### `ARCHITECTURE/module_map.md`
Diagrama textual e lista de módulos: nome, responsabilidade, arquivo de entrada, dependências internas e status de estabilidade.

### `ARCHITECTURE/architectural_decisions.md`
Resumo consolidado das principais decisões arquiteturais. Funciona como "TL;DR dos ADRs" para leitura rápida pelo agente.[^8]

### `ARCHITECTURE/adr/ADR-XXXX-*.md`
ADRs individuais seguindo o padrão MADR. **Imutáveis após status `Accepted`** — se a decisão mudar, cria-se um novo ADR que supersede o anterior. Numerados sequencialmente.[^9][^10]

### `ARCHITECTURE/adr/_adr_index.json`
Índice JSON de todos os ADRs com status, data e título. Permite ao agente filtrar rapidamente ADRs ativos sem ler cada arquivo.[^11]

### `KNOWLEDGE/glossary.md`
Termos de domínio com definição precisa. Evita que o agente use termos errados ou interprete negócios de forma genérica.[^8]

### `KNOWLEDGE/code_conventions.md`
Convenções do projeto: naming, estrutura de pastas, padrões de imports, formatação, linting. O agente consulta antes de gerar qualquer código.[^5]

### `KNOWLEDGE/test_patterns.md`
Estratégias de teste usadas no projeto: unitários, integração, e2e. Exemplos de mocks, fixtures e asserções idiomáticas.[^4]

### `KNOWLEDGE/useful_commands.md`
Comandos frequentes: build, deploy, seed de banco, geração de chaves, execução de migrations, debugging.[^5]

### `INTEGRATIONS/external_integrations.md`
Cada integração externa documentada: endpoint base, autenticação, rate limits, quirks conhecidos e exemplos de uso.[^8]

### `INTEGRATIONS/important_dependencies.md`
Dependências críticas com versão fixada, motivo da escolha, alternativas consideradas e alertas de breaking changes conhecidos.[^11]

### `HISTORY/migration_history.md`
Histórico cronológico de migrações: banco de dados, infraestrutura, mudanças de stack. Inclui rollback procedure de cada migração.[^4]

### `HISTORY/resolved_bugs.md`
Bugs resolvidos com: ID, sintoma, causa-raiz, solução aplicada e como reproduzir. O agente usa para não reabrir bugs já resolvidos.[^12][^1]

### `HISTORY/failed_attempts.md`
**Registro crítico de tentativas fracassadas.** Cada tentativa inclui o que foi tentado, por que falhou e a solução correta. O portão pré-ação do `projectmem` bloqueia o agente se ele tentar repetir uma abordagem listada aqui.[^1]

### `RISK/fragile_areas.md`
Arquivos e módulos marcados como frágeis. O agente recebe um aviso antes de editar qualquer item listado aqui.[^7][^1]

### `RISK/known_risks.md`
Riscos técnicos e de negócio conhecidos: probabilidade, impacto, status de mitigação e owner.[^6]

### `EVENTS/event_log.jsonl`
**Log append-only de eventos do projeto** em formato JSONL. Cada linha é um evento tipado (fix, decision, attempt, note, migration). **Nunca se edita ou deleta linhas existentes** — apenas se acrescenta. Provê trilha de auditoria imutável.[^13][^1]

### `EVENTS/_event_schema.json`
Schema JSON dos eventos válidos no log. Garante que o agente produza eventos bem-formados.[^14]

### `SEARCH/semantic_index.json`
Manifesto de indexação: lista quais arquivos foram indexados, embeddings provider usado, data do último índice e hash SHA-256 de cada chunk. O índice vetorial em si (binário) é ignorado pelo Git.[^15][^3]

### `ARCHIVE/obsolete/`
Memórias que expiraram, foram supersedidas ou se tornaram irrelevantes. Movidas aqui (não deletadas) para preservar histórico. Cada arquivo tem prefixo de data.[^16][^12]

***

## Templates Markdown

### `00_INDEX.md`

```markdown
# Memória do Projeto: {NOME_DO_PROJETO}

> **Agente:** leia este arquivo PRIMEIRO. Depois consulte os blocos relevantes à sua tarefa.

## Status Atual
- **Fase:** {desenvolvimento / homologação / produção}
- **Foco ativo:** {descrição da sprint ou tarefa corrente}
- **Última atualização:** {YYYY-MM-DD} por {agente/humano}

## Blocos de Memória

| Bloco | Arquivo | Leitura |
|-------|---------|---------|
| Perfil do Projeto | [META/project_profile.json](META/project_profile.json) | Sempre |
| Mapa de Módulos | [ARCHITECTURE/module_map.md](ARCHITECTURE/module_map.md) | Sempre |
| Decisões Arquiteturais | [ARCHITECTURE/architectural_decisions.md](ARCHITECTURE/architectural_decisions.md) | Sempre |
| ADRs | [ARCHITECTURE/adr/](ARCHITECTURE/adr/) | Quando relevante |
| Glossário | [KNOWLEDGE/glossary.md](KNOWLEDGE/glossary.md) | Quando em dúvida |
| Convenções de Código | [KNOWLEDGE/code_conventions.md](KNOWLEDGE/code_conventions.md) | Antes de gerar código |
| Padrões de Teste | [KNOWLEDGE/test_patterns.md](KNOWLEDGE/test_patterns.md) | Antes de escrever testes |
| Comandos Úteis | [KNOWLEDGE/useful_commands.md](KNOWLEDGE/useful_commands.md) | Quando precisar executar |
| Tentativas Fracassadas | [HISTORY/failed_attempts.md](HISTORY/failed_attempts.md) | **ANTES de qualquer solução** |
| Áreas Frágeis | [RISK/fragile_areas.md](RISK/fragile_areas.md) | **ANTES de editar arquivos** |
| Bugs Resolvidos | [HISTORY/resolved_bugs.md](HISTORY/resolved_bugs.md) | Ao diagnosticar bugs |
| Integrações Externas | [INTEGRATIONS/external_integrations.md](INTEGRATIONS/external_integrations.md) | Ao integrar serviços |
| Log de Eventos | [EVENTS/event_log.jsonl](EVENTS/event_log.jsonl) | Para contexto histórico |

## Fluxo de Trabalho do Agente

```
INÍCIO DA TAREFA:
1. Ler 00_INDEX.md (este arquivo)
2. Ler RISK/fragile_areas.md → verificar se a tarefa toca áreas frágeis
3. Ler HISTORY/failed_attempts.md → verificar se a abordagem já foi tentada
4. Ler blocos relevantes conforme tabela acima

FIM DA TAREFA:
1. Registrar evento em EVENTS/event_log.jsonl
2. Atualizar arquivo(s) relevante(s) se houver novo conhecimento
3. Se decisão arquitetural: criar ADR e atualizar architectural_decisions.md
4. Se bug resolvido: registrar em resolved_bugs.md
5. Se tentativa fracassada: registrar em failed_attempts.md
```
```

***

### Template ADR: `ARCHITECTURE/adr/ADR-XXXX-<slug>.md`

```markdown
# ADR-{NÚMERO}: {Título da Decisão}

## Status
<!-- Proposed | Accepted | Deprecated | Superseded by ADR-XXXX -->
Accepted

## Data
{YYYY-MM-DD}

## Contexto
<!-- Qual é o problema ou força que motivou esta decisão? -->

## Decisão
<!-- O que foi decidido? Seja explícito e direto. -->

## Opções Consideradas
| Opção | Prós | Contras |
|-------|------|---------|
| **{Escolhida}** | | |
| {Alternativa 1} | | |
| {Alternativa 2} | | |

## Consequências

### Positivas
-

### Negativas
-

### Neutras / Follow-ups
-

## Critérios de Revisão
<!-- Quando esta decisão deve ser reavaliada? Qual evento tornaria esta decisão obsoleta? -->

## Referências
-

<!-- Se supersede outro ADR: -->
<!-- **Supersede:** ADR-XXXX -->
```

***

### Template de Bug Resolvido: `HISTORY/resolved_bugs.md`

```markdown
# Bugs Resolvidos

<!-- Adicionar sempre ao TOPO. Nunca remover entradas. -->

---

## BUG-{ID} — {Título Curto}

- **Data de resolução:** {YYYY-MM-DD}
- **Severidade:** {critical / high / medium / low}
- **Componente afetado:** {módulo/arquivo}

### Sintoma
<!-- O que o usuário/sistema observou? -->

### Causa-Raiz
<!-- Por que aconteceu? Seja específico — evite "bug no código". -->

### Solução Aplicada
<!-- O que foi feito para corrigir? Incluir referência ao commit se possível. -->
<!-- Commit: abc1234 -->

### Como Reproduzir
```
{passos para reproduzir, se relevante}
```

### Como Verificar a Correção
```
{comando ou teste que confirma a correção}
```

---
```

***

### Template de Tentativas Fracassadas: `HISTORY/failed_attempts.md`

```markdown
# Tentativas Fracassadas

> ⛔ **Agente:** consulte este arquivo ANTES de propor qualquer solução.
> Se sua abordagem está listada aqui, ela JÁ FOI TENTADA e FALHOU.

---

## FALHA-{ID} — {Título Curto}

- **Data:** {YYYY-MM-DD}
- **Tarefa relacionada:** {descrição breve da tarefa original}
- **Status:** `failed` | `abandoned` | `superseded`

### O que foi tentado
<!-- Descrever a abordagem com detalhes suficientes para que um agente reconheça se está prestes a repetir. -->

### Por que falhou
<!-- Causa técnica ou de negócio. -->

### Evidência
<!-- Erro, log, output, link para issue, commit revertido. -->

### Solução Correta (se encontrada)
<!-- Se existe uma solução melhor, documentar aqui ou referenciar resolved_bugs.md -->

---
```

***

### Template de Áreas Frágeis: `RISK/fragile_areas.md`

```markdown
# Áreas Frágeis

> ⚠️ **Agente:** verifique este arquivo ANTES de editar qualquer arquivo do projeto.
> Arquivos listados aqui exigem atenção redobrada e, idealmente, revisão humana após qualquer mudança.

---

## {Módulo / Arquivo}

- **Caminho:** `{src/caminho/para/arquivo.ext}`
- **Nível de fragilidade:** {crítico / alto / médio}
- **Motivo:** {Por que é frágil? Ex: sem testes, lógica de negócio implícita, dependências ocultas}
- **Efeitos colaterais conhecidos:** {O que pode quebrar se este arquivo for alterado?}
- **Precauções:** {O que fazer antes de editar? Ex: rodar suite de testes X, avisar equipe Y}
- **Última revisão:** {YYYY-MM-DD}

---
```

***

### Template de Glossário: `KNOWLEDGE/glossary.md`

```markdown
# Glossário do Domínio

<!-- Termos ordenados alfabeticamente. Atualizar sempre que surgir um novo conceito. -->

---

## {Termo}

- **Definição:** {Explicação precisa no contexto deste projeto}
- **Contexto de uso:** {Onde e quando este termo aparece}
- **Não confundir com:** {Termos similares mas diferentes}
- **Referência:** {ADR, documento externo ou código onde o termo é central}

---
```

***

### Template de Integrações Externas: `INTEGRATIONS/external_integrations.md`

```markdown
# Integrações Externas

---

## {Nome do Serviço / API}

- **Tipo:** {REST API / Webhook / SDK / OAuth / gRPC}
- **Endpoint base:** `{https://api.exemplo.com/v2}`
- **Autenticação:** {API Key via header `X-API-Key` / OAuth2 / JWT}
- **Rate limits:** {100 req/min por token}
- **Ambiente de sandbox:** {URL ou instrução}
- **Variável de ambiente:** `{NOME_DA_ENV_VAR}`

### Quirks e Comportamentos Inesperados
- {Descrever comportamentos não óbvios da API}

### Exemplo Mínimo
```{linguagem}
{código de exemplo}
```

### Problemas Conhecidos
- {Listar problemas conhecidos, workarounds}

---
```

***

## Schemas JSON

### `META/project_profile.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["name", "version", "stack", "objective", "status"],
  "properties": {
    "name": { "type": "string", "description": "Nome do projeto" },
    "version": { "type": "string", "description": "Versão semântica atual (ex: 1.4.2)" },
    "objective": { "type": "string", "description": "Uma frase descrevendo o objetivo do projeto" },
    "status": {
      "type": "string",
      "enum": ["ideation", "development", "staging", "production", "maintenance", "archived"]
    },
    "stack": {
      "type": "object",
      "properties": {
        "language": { "type": "array", "items": { "type": "string" } },
        "framework": { "type": "array", "items": { "type": "string" } },
        "database": { "type": "array", "items": { "type": "string" } },
        "infra": { "type": "array", "items": { "type": "string" } },
        "ci_cd": { "type": "string" }
      }
    },
    "team": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "role": { "type": "string" },
          "contact": { "type": "string" }
        }
      }
    },
    "repository": { "type": "string", "format": "uri" },
    "environments": {
      "type": "object",
      "properties": {
        "development": { "type": "string" },
        "staging": { "type": "string" },
        "production": { "type": "string" }
      }
    },
    "memory_last_updated": { "type": "string", "format": "date-time" },
    "memory_updated_by": { "type": "string" }
  }
}
```

***

### `META/metadata.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "memory_version": { "type": "string", "description": "Versão do schema da .memory/ (ex: 2.1.0)" },
    "created_at": { "type": "string", "format": "date-time" },
    "last_updated_at": { "type": "string", "format": "date-time" },
    "last_updated_by": { "type": "string", "description": "Nome do agente ou humano que atualizou" },
    "total_events": { "type": "integer" },
    "total_adrs": { "type": "integer" },
    "total_resolved_bugs": { "type": "integer" },
    "total_failed_attempts": { "type": "integer" },
    "health_score": {
      "type": "number",
      "minimum": 0,
      "maximum": 100,
      "description": "Score de saúde da memória (0-100). Penaliza memórias sem validação, conflitos e entradas obsoletas não arquivadas."
    },
    "stale_threshold_days": {
      "type": "integer",
      "default": 90,
      "description": "Número de dias sem atualização após o qual uma memória é marcada como stale"
    }
  }
}
```

***

### `EVENTS/_event_schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "Schema para cada linha do event_log.jsonl",
  "type": "object",
  "required": ["id", "type", "timestamp", "summary", "author"],
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^EVT-[0-9]{6}$",
      "description": "ID sequencial único (ex: EVT-000042)"
    },
    "type": {
      "type": "string",
      "enum": ["fix", "decision", "attempt_failed", "migration", "integration", "refactor", "note", "risk_identified", "risk_mitigated", "dependency_update", "breaking_change"]
    },
    "timestamp": { "type": "string", "format": "date-time" },
    "author": { "type": "string", "description": "Nome do agente ou humano" },
    "summary": { "type": "string", "maxLength": 200 },
    "detail": { "type": "string", "description": "Detalhes adicionais em Markdown" },
    "related_files": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Caminhos de arquivos relacionados ao evento"
    },
    "related_adr": { "type": "string", "description": "Ex: ADR-0005" },
    "related_bug": { "type": "string", "description": "Ex: BUG-012" },
    "status": {
      "type": "string",
      "enum": ["hypothesis", "active", "validated", "deprecated", "superseded"],
      "default": "active"
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" }
    }
  }
}
```

***

### `ARCHITECTURE/adr/_adr_index.json`

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "required": ["id", "slug", "title", "status", "date"],
    "properties": {
      "id": { "type": "string", "pattern": "^ADR-[0-9]{4}$" },
      "slug": { "type": "string" },
      "title": { "type": "string" },
      "status": {
        "type": "string",
        "enum": ["Proposed", "Accepted", "Deprecated", "Superseded"]
      },
      "date": { "type": "string", "format": "date" },
      "superseded_by": { "type": "string", "nullable": true },
      "tags": { "type": "array", "items": { "type": "string" } }
    }
  }
}
```

***

## Exemplo Preenchido (Projeto: API de Licenças CAD)

### `META/project_profile.json` — Exemplo

```json
{
  "name": "cad-license-api",
  "version": "2.3.1",
  "objective": "API REST para gerenciamento de licenças de software CAD (ZWCAD, SketchUp, Archline) com ativação online e offline.",
  "status": "production",
  "stack": {
    "language": ["Node.js 20", "TypeScript 5"],
    "framework": ["Express 4", "Firebase Admin SDK"],
    "database": ["Firestore", "Cloud SQL (PostgreSQL 15)"],
    "infra": ["Google Cloud Run", "Cloud Pub/Sub", "Secret Manager"],
    "ci_cd": "GitHub Actions → Cloud Build"
  },
  "environments": {
    "development": "http://localhost:3000",
    "staging": "https://api-staging.totalcad.com.br",
    "production": "https://api.totalcad.com.br"
  },
  "memory_last_updated": "2026-07-02T08:00:00-03:00",
  "memory_updated_by": "agent/claude-code"
}
```

***

### `HISTORY/failed_attempts.md` — Exemplo Preenchido

```markdown
# Tentativas Fracassadas

> ⛔ Agente: consulte ANTES de propor soluções.

---

## FALHA-003 — Usar Firestore como fila de processamento de ativações

- **Data:** 2026-03-14
- **Tarefa:** Processar ativações de licença de forma assíncrona
- **Status:** `superseded`

### O que foi tentado
Usar listeners `onSnapshot` do Firestore para reagir a novos documentos de ativação e processar na função Cloud Run.

### Por que falhou
O Firestore não garante entrega única (at-most-once delivery) para listeners. Em condições de rede instável, o mesmo documento disparava 2-3 processamentos simultâneos, gerando duplicação de licenças ativas.

### Evidência
Incident #INC-2026-03-15 — 47 licenças duplicadas detectadas em produção. Rollback em 4h.

### Solução Correta
Migrado para Cloud Pub/Sub com dead-letter queue. Veja ADR-0008 e EVT-000031.

---

## FALHA-002 — Validação de licença via client-side JWT

- **Data:** 2026-01-22
- **Status:** `abandoned`

### O que foi tentado
Assinar o JWT de licença com a chave pública embarcada no binário do ZWCAD para validação offline.

### Por que falhou
Chave pública extraída por engenharia reversa em 48h após release. Relatado por pesquisador de segurança via responsible disclosure.

### Solução Correta
Validação offline usa HMAC com chave derivada do hardware ID (fingerprint da máquina). Veja ADR-0004.

---
```

***

### `EVENTS/event_log.jsonl` — Exemplo de Linhas

```jsonl
{"id":"EVT-000031","type":"migration","timestamp":"2026-03-16T10:22:00-03:00","author":"agent/claude-code","summary":"Migração de Firestore listener para Cloud Pub/Sub para processamento de ativações","related_adr":"ADR-0008","related_files":["src/activation/queue.ts","infra/pubsub.tf"],"status":"validated","tags":["pubsub","activation","infra"]}
{"id":"EVT-000032","type":"fix","timestamp":"2026-03-16T14:05:00-03:00","author":"lead.engineer","summary":"BUG-012 resolvido: licenças duplicadas por processamento paralelo em Firestore","related_bug":"BUG-012","related_files":["src/activation/processor.ts"],"status":"validated","tags":["bug","firestore","duplication"]}
{"id":"EVT-000033","type":"decision","timestamp":"2026-04-02T09:30:00-03:00","author":"agent/gemini-pro","summary":"Adotado padrão de idempotency key em todas as rotas de ativação","related_adr":"ADR-0009","status":"active","tags":["idempotency","api-design"]}
```

***

## Regras de Atualização

### Quando atualizar (gatilhos)

| Evento | Arquivo(s) a atualizar |
|--------|----------------------|
| Bug corrigido | `HISTORY/resolved_bugs.md` + `EVENTS/event_log.jsonl` |
| Abordagem falhou | `HISTORY/failed_attempts.md` + `EVENTS/event_log.jsonl` |
| Decisão arquitetural tomada | Novo `ARCHITECTURE/adr/ADR-XXXX.md` + `architectural_decisions.md` + `_adr_index.json` + evento |
| Nova integração adicionada | `INTEGRATIONS/external_integrations.md` + `important_dependencies.md` |
| Migration executada | `HISTORY/migration_history.md` + evento |
| Novo risco identificado | `RISK/known_risks.md` + evento |
| Arquivo torna-se frágil | `RISK/fragile_areas.md` |
| Novo termo de domínio | `KNOWLEDGE/glossary.md` |
| Sprint/fase concluída | `00_INDEX.md` (atualizar foco ativo e status) |

### Regras de imutabilidade

1. **`EVENTS/event_log.jsonl`** — **nunca editar ou deletar linhas existentes.** Apenas `append`. Este é o registro imutável.[^13][^1]
2. **ADRs com status `Accepted`** — **nunca editar.** Se a decisão mudar, criar novo ADR com status `Accepted` que contenha `Supersede: ADR-XXXX`.[^10]
3. **`HISTORY/failed_attempts.md` e `HISTORY/resolved_bugs.md`** — **nunca deletar entradas.** Apenas marcar como `superseded` se substituída.[^12]

### Regras de atualização pelo agente

```
ANTES de qualquer tarefa:
  1. Ler 00_INDEX.md
  2. Verificar RISK/fragile_areas.md
  3. Verificar HISTORY/failed_attempts.md
  4. Se a abordagem listada em failed_attempts → PARAR e consultar humano

APÓS tarefa concluída:
  1. Registrar evento em event_log.jsonl (obrigatório)
  2. Atualizar no máximo 3 arquivos por sessão (evitar escrita em excesso)
  3. Adicionar ao TOPO dos arquivos de histórico (não ao fim)
  4. Marcar claramente novos registros com data e autor

O agente NÃO deve:
  - Deletar entradas existentes
  - Editar ADRs com status Accepted
  - Editar linhas do event_log.jsonl
  - Atualizar metadata.json diretamente (apenas scripts automatizados)
```

***

## Regras para Evitar Lixo na Memória

Baseado em pesquisas sobre degradação de memória em agentes de IA, as seguintes regras minimizam ruído:[^17][^16][^7]

### 1. Ciclo de Vida de Status

Toda entrada de memória segue o ciclo:[^12]

```
hypothesis → active → validated
                   ↓
              deprecated (disproven)
              superseded (replaced by newer)
```

Apenas entradas com status `active` ou `validated` aparecem em buscas normais. O agente **só vê verdade atual**.

### 2. Regra do Threshold de Staleness

- Qualquer entrada não atualizada por **90 dias** recebe flag `stale: true` no metadata.[^7]
- Entradas `stale` requerem revisão humana antes de serem usadas como base para decisões.
- O valor padrão é configurável em `META/metadata.json` → `stale_threshold_days`.

### 3. Regras Anti-Lixo para o Agente

| Proibido | Motivo |
|----------|--------|
| Duplicar informação já existente | Cria conflito de verdade |
| Adicionar entrada sem causa-raiz | Memória não acionável |
| Registrar observações triviais | Polui o log de eventos |
| Copiar stack traces sem diagnóstico | Sem valor semântico |
| Atualizar memória com informação não verificada | Propaga hipóteses como fatos |
| Deixar ADRs em status `Proposed` por mais de 30 dias | ADRs fantasma |

### 4. Deduplicação Automática

O `SEARCH/semantic_index.json` usa hashes SHA-256 por chunk. Se um novo conteúdo tem hash idêntico a um existente, o sistema descarta silenciosamente o duplicado sem re-embedar.[^18][^3]

### 5. Compactação de Histórico

A cada **6 meses** ou quando `EVENTS/event_log.jsonl` ultrapassar **500 linhas**, um humano deve:
1. Gerar um sumário consolidado com LLM
2. Salvar como `ARCHIVE/YYYY-MM_event_summary.md`
3. Mover eventos antigos para `ARCHIVE/obsolete/event_log_YYYY-MM.jsonl`
4. Manter os últimos 90 dias no log ativo[^16][^17]

***

## Checklist de Revisão Humana

A revisão humana deve ocorrer **mensalmente** ou **a cada marco de sprint**.

### ✅ Checklist de Saúde da Memória

**Perfil e Metadados**
- [ ] `project_profile.json` reflete a versão e status atuais do projeto
- [ ] `metadata.json` tem `last_updated_at` com menos de 30 dias
- [ ] `health_score` em `metadata.json` está acima de 70

**Arquitetura**
- [ ] Todos os ADRs em status `Proposed` têm menos de 30 dias (senão: aceitar, rejeitar ou deletar)
- [ ] `architectural_decisions.md` está sincronizado com os ADRs ativos
- [ ] `module_map.md` reflete a estrutura atual de módulos do código

**Conhecimento**
- [ ] `glossary.md` cobre todos os termos usados em issues e ADRs recentes
- [ ] `code_conventions.md` reflete as convenções efetivamente praticadas
- [ ] `useful_commands.md` tem todos os comandos funcionando (testar ao menos um por revisão)

**Histórico**
- [ ] Bugs dos últimos 30 dias foram documentados em `resolved_bugs.md`
- [ ] Não há tentativas fracassadas não documentadas na memória da equipe
- [ ] `migration_history.md` inclui todas as migrations executadas no período

**Risco**
- [ ] `fragile_areas.md` está atualizado (remover itens que foram estabilizados)
- [ ] `known_risks.md` tem owner e data de última revisão para cada risco

**Eventos e Indexação**
- [ ] `event_log.jsonl` não tem linhas editadas (verificar com `git log -- EVENTS/event_log.jsonl`)
- [ ] Entradas com status `stale` foram revisadas e resolvidas (validar ou arquivar)
- [ ] `semantic_index.json` tem data de indexação com menos de 7 dias

**Arquivo**
- [ ] Memórias com mais de 90 dias sem atualização e sem uso foram movidas para `ARCHIVE/obsolete/`
- [ ] `_archive_index.md` está atualizado com os arquivos arquivados

**Limpeza**
- [ ] Não há duplicatas óbvias entre arquivos de memória
- [ ] Entradas `deprecated` e `superseded` não estão sendo referenciadas por entradas `active`
- [ ] `event_log.jsonl` não ultrapassou 500 linhas (se sim, fazer compactação)

***

## Indexação para Busca Semântica

O arquivo `SEARCH/semantic_index.json` funciona como manifesto do índice. O índice vetorial binário (`.db`, `.index`) fica no `.gitignore`.[^3][^18]

```json
{
  "indexer_version": "1.0",
  "embedding_provider": "google/text-embedding-004",
  "last_indexed_at": "2026-07-01T18:00:00Z",
  "chunk_strategy": "by_heading",
  "files": [
    {
      "path": ".memory/ARCHITECTURE/module_map.md",
      "sha256": "a3f9c1...",
      "chunks": 12,
      "indexed_at": "2026-07-01T18:00:00Z"
    },
    {
      "path": ".memory/HISTORY/failed_attempts.md",
      "sha256": "b82d44...",
      "chunks": 8,
      "indexed_at": "2026-07-01T18:00:00Z"
    }
  ],
  "search_strategy": "hybrid_bm25_vector",
  "reranking": "rrf"
}
```

O índice é **sempre reconstruível** a partir dos arquivos Markdown. Se o provider de embeddings mudar, basta re-indexar sem perda de dados.[^3]

```
# Reconstruir índice completo
memsearch index .memory/

# Busca semântica
memsearch search "tentativas com Firestore que falharam" --threshold 0.65
```

### `.memory/SEARCH/.gitignore`

```gitignore
# Índices binários (reconstruíveis)
*.db
*.index
*.bin
chroma/
qdrant_storage/
.cache/
```

***

## Notas de Implementação

- **Git como ground truth:** toda a memória é versionada. `git blame` e `git log` revelam a autoria e histórico de cada mudança.[^19][^8]
- **Arquivos Markdown como fonte da verdade:** qualquer índice vetorial é derivado e descartável. Mudou o provider de embeddings? Reconstrua.[^2][^18][^3]
- **Sem dependência de servidor:** a pasta funciona sem MCP, sem banco externo, sem API — qualquer agente que leia arquivos pode consumir.[^20][^8]
- **MCP opcional:** se o projeto usar MCP, ferramentas como `projectmem` ou `memsearch` podem servir a pasta como contexto estruturado via Model Context Protocol.[^19][^1]

---

## References

1. [PROJECTMEM: A Local-First, Event-Sourced Memory and Judgment Layer for AI Coding Agents](https://arxiv.org/abs/2606.12329v1) - AI coding assistants now support a growing share of software work, from quick scripts to production ...

2. [memweave: Zero-Infra AI Agent Memory with Markdown and SQLite](https://aissential.tech/articles/54aff3e6-f0f4-41d9-b895-291f822b1ff6) - LLM agent memory can be managed effectively using version-controlled Markdown files and a local SQLi...

3. [Using Milvus as a "rebuildable index" for AI agent memory](https://www.reddit.com/r/vectordatabase/comments/1r3kz3r/using_milvus_as_a_rebuildable_index_for_ai_agent/) - The project is memsearch, a semantic memory library for AI agents. The workflow is: Agent writes obs...

4. [Memory Bank](https://docs.cline.bot/best-practices/memory-bank) - A structured documentation system that helps Cline maintain context across sessions. Memory Bank is ...

5. [Memory Bank System | Agentic Coding Handbook](https://tweag.github.io/agentic-coding-handbook/WORKFLOW_MEMORY_BANK/)

6. [Memory Management](https://www.agent-zero.ai/p/docs/memory/) - When to clean memory: Agent performance degrades due to too many irrelevant memories; Project comple...

7. [Stop AI Agents Acting on Stale Memory | Armalo](https://www.armalo.ai/blog/stop-ai-agent-acting-on-stale-memory)

8. [The Git-Backed Brain: Using a Repo as Shared Agent Memory](https://kidakaka.com/2026/05/31/the-git-backed-brain/)

9. [About MADR - Architectural Decision Records](https://adr.github.io/madr/)

10. [ADR (Architecture Decision Record) — Documentation Pattern.md](https://gist.github.com/thedavidyoungblood/bccce859af7a476e44a290a2230e0913) - ADR (Architecture Decision Record) — Documentation Pattern.md - ADR (Architecture Decision Record) —...

11. [GitHub - kopach/adr-templates: ADR (Architectural Decision Record) templates](https://github.com/kopach/adr-templates) - ADR (Architectural Decision Record) templates. Contribute to kopach/adr-templates development by cre...

12. [My AI remembered the wrong thing and broke my build. So ...](https://dev.to/michael_onyekwere/my-ai-remembered-the-wrong-thing-and-broke-my-build-so-i-built-memory-governance-50b2) - Storage isn't the hard part. Knowing what's still true is. I added lifecycle states, conflict detect...

13. [ESAA-Conversational: Event-Sourced Memory for Seamless LLM...](https://theneuralfeed.com/article/esaa-conversational-an-event-sourced-memory-layer-for-continuity-handoff-and-cur/S35TjBUe) - Turns disjointed AI coding sessions into a unified, persistent conversation store.

14. [AgentLog is a lightweight event bus for AI agents built on ... - GitHub](https://github.com/sumant1122/agentlog) - AgentLog is a lightweight event bus for AI agents built on append-only JSONL logs. Agents publish an...

15. [Using Open-Source memsearch, Giving Your AI Agents Persistent, Human-Readable Memory](https://www.youtube.com/watch?v=bf9eiFEs1JA) - 🔗 GitHub: https://github.com/zilliztech/memsearch

Memsearch is a lightweight library that gives AI ...

16. [Why Your AI Agent Needs Memory Lifecycle Management](https://www.varunpratap.com/blog/ai-memory-lifecycle-management) - AI agents generate thousands of memories but never clean up. Learn how automatic lifecycle managemen...

17. [Memory eviction and forgetting in AI agents - Mem0](https://mem0.ai/blog/memory-eviction-and-forgetting-in-ai-agents) - Whar is memory eviction, why an agent that remembers everything recalls badly and how to design forg...

18. [memsearch by zilliztech - SourcePulse](https://www.sourcepulse.org/projects/24544655) - Markdown-first memory system for AI agents

19. [Give Your AI Coding Agents Shared, Persistent Project Memory](https://dev.to/_2340687267e5cacfe32da1/memorix-give-your-ai-coding-agents-shared-persistent-project-memory-1pk2) - TL;DR: Every coding agent forgets between sessions. Memorix is an open-source MCP memory layer that....

20. [AGENTS.md Is Not Enough: Building Project Memory for AI Coding ...](https://dev.to/reapollo/agentsmd-is-not-enough-building-project-memory-for-ai-coding-agents-1o01) - Most AI coding workflows still treat project context as chat state. That works until you switch...

