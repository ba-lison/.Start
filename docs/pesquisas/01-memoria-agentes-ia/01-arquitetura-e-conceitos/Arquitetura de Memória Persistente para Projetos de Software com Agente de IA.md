# Arquitetura de Memória Persistente para Projetos de Software com Agente de IA

## Visão Geral

A memória persistente transforma um agente de IA de uma ferramenta sem estado em um colaborador contínuo — capaz de lembrar decisões arquiteturais, bugs resolvidos, padrões de código e o histórico evolutivo do projeto. Esta arquitetura combina arquivos Markdown versionáveis, eventos JSONL auditáveis, recuperação semântica por vetores, metadados relacionais, knowledge graph e um ciclo de curadoria por LLM com validação humana.

***

## 1. Visão Geral dos Componentes

```
.memory/
├── README.md                  # Ponto de entrada e índice da memória
├── project.md                 # Contexto permanente do projeto
├── architecture.md            # Decisões arquiteturais (ADRs)
├── bugs.md                    # Bugs conhecidos e solucionados
├── modules/                   # Memória por módulo
│   ├── auth.md
│   └── billing.md
├── sessions/                  # Notas por sessão de trabalho
│   └── 2026-07-02.md
├── events/                    # Ledger JSONL de eventos
│   ├── memory_events.jsonl
│   └── audit_log.jsonl
├── store/                     # Vector store local
│   ├── chroma/                # ChromaDB ou Qdrant local
│   └── embeddings.db
├── meta/                      # Banco relacional de metadados
│   └── memory.db              # SQLite
├── graph/                     # Knowledge graph opcional
│   └── graph.jsonl
└── .schema/                   # Schemas de validação
    ├── event_schema.json
    └── memory_schema.json
```

A pasta `.memory/` é **versionada no Git** como qualquer outro artefato de código. Os arquivos Markdown são legíveis por humanos e pelas ferramentas de diff do Git; o JSONL persiste o histórico imutável; o SQLite e o vector store são derivados (reconstruíveis) e podem ser ignorados no `.gitignore` padrão ou versionados via LFS.[^1][^2]

***

## 2. Camadas de Memória

A arquitetura implementa quatro camadas com diferentes políticas de retenção e acesso:[^3]

| Camada | Tipo | Armazenamento | Vida útil | Exemplo |
|---|---|---|---|---|
| **Episódica** | Eventos brutos | `events/memory_events.jsonl` | 30–90 dias | "Usuário corrigiu bug #42 em 2026-07-01" |
| **Semântica** | Fatos destilados | Markdown + SQLite + Vector | Longa (curada) | "O módulo auth usa JWT com refresh tokens" |
| **Projeto** | Estado corrente | `project.md`, `modules/*.md` | Ativa enquanto relevante | Goals, blockers, decisões em aberto |
| **Procedural** | Regras e preferências | `architecture.md`, `AGENTS.md` | Quase permanente | "Nunca usar `any` em TypeScript" |

***

## 3. Formatos de Arquivo e Schemas

### 3.1 Markdown (Memória Humana — Camada Longa)

Cada arquivo Markdown segue uma estrutura padronizada com frontmatter YAML para indexação:[^4][^1]

```markdown
---
id: "arch-decision-001"
type: "decision"
module: "auth"
confidence: 0.92
created_at: "2026-07-01T14:22:00Z"
updated_at: "2026-07-02T09:10:00Z"
tags: ["jwt", "auth", "security"]
status: "active"              # active | deprecated | superseded
superseded_by: null
human_validated: true
---

# Decisão: JWT com Refresh Tokens para Autenticação

## Contexto
Precisamos de autenticação stateless para suportar múltiplos serviços.

## Decisão
Adotar JWT (access token 15min) + refresh token (7 dias) armazenado em
HttpOnly cookie. Chave de assinatura rotacionada a cada 30 dias.

## Consequências
- ✅ Escalabilidade horizontal sem sessão centralizada
- ⚠️  Invalidação imediata requer lista de revogação (Redis)

## Arquivos Afetados
- `src/auth/jwt.service.ts`
- `src/auth/refresh.service.ts`
```

### 3.2 JSONL (Ledger de Eventos de Memória)

Cada linha é um objeto JSON autossuficiente e independentemente parseável. O design append-only garante crash-safety: se o processo falha, as linhas já gravadas permanecem intactas.[^5]

```jsonl
{"id":"evt-001","v":2,"surface":"cognitive","trace_id":"abc123","ts":"2026-07-02T09:14:03Z","t":0.0,"event":"memory_write","data":{"memory_id":"arch-decision-001","action":"write","source":"commit:a3f2b1c","content_hash":"sha256:deadbeef","confidence":0.85,"trigger":"post-commit-hook","human_validated":false}}
{"id":"evt-002","v":2,"surface":"operational","trace_id":"abc123","ts":"2026-07-02T09:15:10Z","t":67.0,"event":"memory_validated","data":{"memory_id":"arch-decision-001","validator":"human","confidence_delta":+0.07,"new_confidence":0.92}}
{"id":"evt-003","v":2,"surface":"contextual","trace_id":"xyz456","ts":"2026-07-02T10:00:00Z","t":0.0,"event":"memory_retrieved","data":{"query":"como funciona autenticacao","memory_ids":["arch-decision-001"],"retrieval_method":"hybrid","scores":{"vector":0.91,"bm25":0.78,"final":0.87}}}
{"id":"evt-004","v":2,"surface":"operational","trace_id":"xyz456","ts":"2026-07-02T10:00:01Z","t":1.0,"event":"memory_pruned","data":{"memory_id":"arch-decision-000","reason":"superseded","superseded_by":"arch-decision-001"}}
```

O campo `"v":2` permite evolução de schema sem quebrar leitores antigos. A superfície (`surface`) categoriza eventos em `cognitive` (interações LLM), `operational` (chamadas de ferramenta) e `contextual` (I/O externo), seguindo o modelo AgentTrace.[^5]

### 3.3 Schema SQLite (Metadados Relacionais)

O SQLite é escolhido por não exigir infraestrutura — um único arquivo `.db` viaja junto ao repositório ou é reconstruído a partir do JSONL.[^3]

```sql
-- Tabela principal de memórias
CREATE TABLE memories (
    id          TEXT PRIMARY KEY,              -- UUID ou slug legível
    tier        TEXT NOT NULL CHECK(tier IN ('episodic','semantic','project','procedural')),
    type        TEXT NOT NULL,                 -- 'decision','bug','module','preference','adr'
    source      TEXT NOT NULL,                 -- 'commit:abc','pr:42','session:2026-07-02'
    content     TEXT NOT NULL,
    file_path   TEXT,                          -- caminho no .memory/
    module      TEXT,                          -- módulo do projeto relacionado
    confidence  REAL DEFAULT 0.6,              -- [0.0, 1.0]
    distilled   BOOLEAN DEFAULT 0,             -- passou pela curadoria LLM?
    human_validated BOOLEAN DEFAULT 0,
    access_count INTEGER DEFAULT 0,
    expires_at  DATETIME,
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Embeddings vetoriais (ou usar store externo)
CREATE TABLE embeddings (
    memory_id   TEXT REFERENCES memories(id) ON DELETE CASCADE,
    model       TEXT NOT NULL,                 -- 'text-embedding-3-small'
    vector      BLOB NOT NULL,                 -- float32[] serializado
    dim         INTEGER NOT NULL,
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Entidades do knowledge graph
CREATE TABLE entities (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    name        TEXT NOT NULL UNIQUE,
    type        TEXT NOT NULL,                 -- 'module','decision','bug','file','person'
    properties  JSON
);

-- Relações do knowledge graph
CREATE TABLE relationships (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    source_entity   INTEGER REFERENCES entities(id),
    target_entity   INTEGER REFERENCES entities(id),
    relation_type   TEXT NOT NULL,             -- 'depends_on','fixes','decided_by','affects'
    properties      JSON,
    confidence      REAL DEFAULT 1.0
);

-- Logs de auditoria (imutável)
CREATE TABLE audit_log (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    event_id    TEXT NOT NULL,
    action      TEXT NOT NULL,                 -- 'write','read','validate','prune','reject'
    actor       TEXT NOT NULL,                 -- 'llm:gpt-4o','human:developer','hook:post-commit'
    memory_id   TEXT,
    details     JSON,
    ts          DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Índices
CREATE INDEX idx_memories_tier       ON memories(tier);
CREATE INDEX idx_memories_module     ON memories(module);
CREATE INDEX idx_memories_confidence ON memories(confidence);
CREATE INDEX idx_memories_distilled  ON memories(distilled, human_validated);
CREATE INDEX idx_rel_source          ON relationships(source_entity);
CREATE INDEX idx_rel_target          ON relationships(target_entity);
```

### 3.4 Knowledge Graph (JSONL Opcional)

Para projetos que precisam rastrear relações entre módulos, bugs, decisões e arquivos, o graph pode ser mantido como JSONL de entidades e relações:[^6]

```jsonl
{"op":"entity","id":"mod-auth","type":"module","name":"auth","properties":{"path":"src/auth/","status":"active"}}
{"op":"entity","id":"dec-001","type":"decision","name":"JWT com refresh tokens","properties":{"confidence":0.92}}
{"op":"entity","id":"bug-042","type":"bug","name":"Token expirado não gera 401","properties":{"status":"fixed","commit":"a3f2b1c"}}
{"op":"relation","source":"bug-042","target":"mod-auth","relation":"affects","confidence":1.0}
{"op":"relation","source":"bug-042","target":"dec-001","relation":"motivated_by","confidence":0.8}
{"op":"relation","source":"mod-auth","target":"dec-001","relation":"implements","confidence":0.95}
```

Tipos de relação mais úteis em projetos de software: `depends_on`, `fixes`, `decided_by`, `affects`, `supersedes`, `implements`, `motivated_by`.[^3]

***

## 4. Fluxo de Dados

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FONTES DE EVENTOS                                   │
│  git commit │ git PR/issue │ tarefa do agente │ sessão de trabalho          │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │    GIT HOOKS (pre/post)      │  ←
                    │  • secret scanning           │
                    │  • extração de contexto      │
                    │  • append → events.jsonl     │
                    └──────────────┬───────────────┘
                                   │
              ┌────────────────────▼───────────────────────┐
              │          PIPELINE DE INGESTÃO               │
              │                                             │
              │  1. Validação de segredos (Gitleaks)        │  ←
              │  2. Extração de fatos via LLM               │
              │  3. Score de confiança inicial (0.5–0.7)    │
              │  4. Fila de validação humana                 │
              └────────────────────┬───────────────────────┘
                                   │
              ┌────────────────────▼───────────────────────┐
              │      VALIDAÇÃO HUMANA (checkpoint)          │
              │                                             │
              │  • Interface CLI ou webhook                 │
              │  • Aprovar / Rejeitar / Editar              │
              │  • confidence += 0.2 se aprovado            │
              │  • Registra em audit_log                    │
              └────────────────────┬───────────────────────┘
                                   │
         ┌─────────────────────────▼──────────────────────────────┐
         │                  ARMAZENAMENTO DUAL                     │
         │                                                         │
         │  ┌────────────────────┐   ┌──────────────────────────┐ │
         │  │   .memory/*.md     │   │   SQLite memory.db        │ │
         │  │   (Markdown)       │   │   + embeddings            │ │
         │  │   Human-readable   │   │   + entities              │ │
         │  └────────────────────┘   │   + audit_log             │ │
         │                           └──────────────────────────┘ │
         │  ┌────────────────────┐   ┌──────────────────────────┐ │
         │  │   Vector Store     │   │   Knowledge Graph         │ │
         │  │   (ChromaDB/Qdrant)│   │   (graph.jsonl)           │ │
         │  │   embeddings dense │   │   entidades + relações    │ │
         │  └────────────────────┘   └──────────────────────────┘ │
         └─────────────────────────┬──────────────────────────────┘
                                   │
         ┌─────────────────────────▼──────────────────────────────┐
         │               CICLO DE RECUPERAÇÃO                      │
         │                                                         │
         │   Antes de cada tarefa do agente:                       │
         │   1. Búsca híbrida (Vector + BM25) → top-K             │  ←
         │   2. Filtro por confidence ≥ threshold                  │
         │   3. Re-ranking por recência + relevância               │
         │   4. Injeção no contexto (≤ 4000 tokens)                │
         └─────────────────────────┬──────────────────────────────┘
                                   │
         ┌─────────────────────────▼──────────────────────────────┐
         │         PIPELINE DE CONSOLIDAÇÃO (periódico)           │
         │                                                         │
         │   1. Deduplicação trigrama (threshold 0.85)             │
         │   2. Compressão episódica → semântica (LLM)             │
         │   3. Verificação TrustMem (coverage, preservation,      │  ←
         │      faithfulness)                                      │
         │   4. Remoção de memórias obsoletas                      │
         │   5. Atualização do knowledge graph                     │
         └────────────────────────────────────────────────────────┘
```

***

## 5. Integração com Git Hooks

Os hooks são mantidos em `.githooks/` (versionado) com `git config core.hooksPath .githooks`:[^7]

### `post-commit` — Captura de Contexto

```bash
#!/bin/bash
# .githooks/post-commit

set -e

MEMORY_DIR=".memory"
EVENTS_FILE="$MEMORY_DIR/events/memory_events.jsonl"
COMMIT_MSG=$(git log -1 --pretty=%B)
COMMIT_HASH=$(git rev-parse HEAD)
CHANGED_FILES=$(git diff-tree --no-commit-id -r --name-only HEAD)
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
TRACE_ID=$(uuidgen | tr -d '\n')

# 1. Escanear segredos nos arquivos staged
if command -v gitleaks &>/dev/null; then
    gitleaks detect --source . --no-git 2>/dev/null || {
        echo "⚠️  Segredos detectados. Abortando captura de memória."
        exit 0  # não bloquear o commit, mas não capturar
    }
fi

# 2. Construir evento de memória
EVENT=$(cat <<JSON
{"id":"evt-$(uuidgen)","v":2,"surface":"operational","trace_id":"$TRACE_ID","ts":"$TIMESTAMP","t":0.0,"event":"memory_capture_trigger","data":{"trigger":"post-commit","commit":"$COMMIT_HASH","message":$(echo "$COMMIT_MSG" | python3 -c 'import json,sys; print(json.dumps(sys.stdin.read().strip()))'),"changed_files":$(echo "$CHANGED_FILES" | python3 -c 'import json,sys; print(json.dumps(sys.stdin.read().strip().split("\n")))'),"status":"pending_llm_curation"}}
JSON
)

echo "$EVENT" >> "$EVENTS_FILE"

# 3. Disparar curadoria LLM assíncrona
python3 .memory/scripts/curator.py --trigger commit --commit "$COMMIT_HASH" &
disown
```

### `pre-commit` — Proteção contra Segredos

```bash
#!/bin/bash
# .githooks/pre-commit

# Bloquear segredos usando detect-secrets ou gitleaks
if command -v detect-secrets &>/dev/null; then
    detect-secrets scan --baseline .secrets.baseline
    detect-secrets audit .secrets.baseline || exit 1
fi

# Escanear arquivos .memory/ por possíveis segredos antes de commitar
if git diff --cached --name-only | grep -q "^\.memory/"; then
    git diff --cached -- ".memory/**" | grep -E \
        "(password|api_key|secret|token|private_key)\s*[:=]\s*['\"]?[A-Za-z0-9+/]{20,}" \
        && { echo "❌ Possível segredo em .memory/. Revise antes de commitar."; exit 1; }
fi

echo "✅ Pre-commit checks passed"
```

### `commit-msg` — Conventional Commits para Rastreabilidade

```bash
#!/bin/bash
# .githooks/commit-msg
MSG=$(cat "$1")
if ! echo "$MSG" | grep -qE '^(feat|fix|chore|docs|refactor|test|memory):'; then
    echo "❌ Commit deve começar com: feat|fix|chore|docs|refactor|test|memory"
    exit 1
fi
```

### Integração com PRs e Issues (GitHub Actions)

```yaml
# .github/workflows/memory-sync.yml
name: Memory Sync

on:
  pull_request:
    types: [opened, merged]
  issues:
    types: [closed]

jobs:
  sync-memory:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Extract PR/Issue context
        env:
          PR_BODY: ${{ github.event.pull_request.body }}
          PR_TITLE: ${{ github.event.pull_request.title }}
        run: |
          python3 .memory/scripts/curator.py \
            --trigger pr \
            --pr-number ${{ github.event.number }} \
            --pr-title "$PR_TITLE"
```

***

## 6. Algoritmo de Recuperação

A recuperação híbrida combina busca semântica (vetorial) e léxica (BM25) usando Reciprocal Rank Fusion (RRF). Isso supera a busca vetorial pura em ~12% de recall em casos reais.[^8][^9]

### Pseudo-código do Motor de Recuperação

```python
def retrieve_relevant_memories(query: str, task_context: dict, k: int = 10) -> list[Memory]:
    """
    Recuperação híbrida com scoring final composto.
    FinalScore = (VectorScore × 0.5) + (KeywordScore × 0.3) + (RecencyScore × 0.2)
    """

    # 1. Busca vetorial densa
    query_embedding = embed(query)  # ex: text-embedding-3-small (1536d)
    vector_results = vector_store.similarity_search(
        embedding=query_embedding,
        k=50,
        filter={"confidence": {"$gte": 0.5}, "human_validated": True}
    )

    # 2. Busca léxica BM25
    bm25_results = bm25_index.search(query, k=50)

    # 3. Reciprocal Rank Fusion (RRF, k=60)
    def rrf_score(rank: int, k: int = 60) -> float:
        return 1.0 / (k + rank)

    scores = {}
    for rank, mem in enumerate(vector_results):
        scores[mem.id] = scores.get(mem.id, 0) + rrf_score(rank) * 0.5
    for rank, mem in enumerate(bm25_results):
        scores[mem.id] = scores.get(mem.id, 0) + rrf_score(rank) * 0.3

    # 4. Fator de recência com decaimento exponencial (half-life = 30 dias)
    import math
    now = datetime.utcnow()
    for mem_id in scores:
        mem = get_memory(mem_id)
        age_days = (now - mem.updated_at).days
        half_life = 30.0
        recency_score = math.exp(-0.693 * age_days / half_life)
        scores[mem_id] += recency_score * 0.2

    # 5. Boost por confidence
    for mem_id in scores:
        scores[mem_id] *= get_memory(mem_id).confidence

    # 6. Retornar top-k ordenados
    sorted_ids = sorted(scores, key=lambda x: scores[x], reverse=True)[:k]
    memories = [get_memory(mid) for mid in sorted_ids]

    # 7. Registrar recuperação no audit log
    log_retrieval(query=query, retrieved_ids=sorted_ids, scores=scores)

    return memories
```

### Filtros de Pré-Busca por Relevância de Contexto

```python
# Filtros aplicados antes da busca para reduzir espaço de busca
filters = {
    "human_validated": True,
    "confidence": {"$gte": 0.6},
    "tier": ["semantic", "project", "procedural"],  # excluir episódico bruto
}
if task_context.get("module"):
    filters["module"] = task_context["module"]
```

***

## 7. Algoritmo de Consolidação

A consolidação (ou destilação) converte memórias episódicas brutas em memórias semânticas curadas. O framework TrustMem demonstrou que a verificação por transição reduz omissões em 40,1%, corrupções em 79,1% e alucinações em 50,0% em relação às melhores baselines.[^10][^3]

```python
def consolidate_memories(db_path: str, batch_size: int = 50):
    """
    Pipeline de consolidação em 3 estágios:
    1. Deduplicação  →  2. Compressão episódica→semântica  →  3. Verificação
    """

    conn = sqlite3.connect(db_path)

    # ── Estágio 1: Deduplicação por trigrama ──────────────────────────────
    # Threshold 0.85 captura duplicatas sem precisar de embeddings
    dupes = conn.execute("""
        SELECT m1.id as keep_id, m2.id as drop_id
        FROM memories m1
        JOIN memories m2 ON m1.module = m2.module
            AND m1.id < m2.id
            AND m1.tier = m2.tier
        WHERE trigram_similarity(m1.content, m2.content) > 0.85
    """).fetchall()

    for keep_id, drop_id in dupes:
        conn.execute("UPDATE memories SET distilled=1 WHERE id=?", (drop_id,))
        log_audit(action="prune", reason="duplicate", memory_id=drop_id)

    # ── Estágio 2: Compressão episódica → semântica ───────────────────────
    # Memórias episódicas com > 7 dias são candidatas
    raw_episodes = conn.execute("""
        SELECT id, content, source, module, created_at
        FROM memories
        WHERE tier = 'episodic'
          AND distilled = 0
          AND created_at < datetime('now', '-7 days')
        ORDER BY created_at DESC
        LIMIT ?
    """, (batch_size,)).fetchall()

    if raw_episodes:
        # Prompt de curadoria para LLM barato (ex: Claude Haiku, GPT-4o-mini)
        prompt = build_curation_prompt(raw_episodes)
        extracted_facts = llm_extract_facts(prompt)

        for fact in extracted_facts:
            mem = Memory(
                id=generate_id(),
                tier="semantic",
                type=fact["type"],
                content=fact["content"],
                source="distillation",
                module=fact.get("module"),
                confidence=0.65,          # confiança inicial pós-distilação
                distilled=True,
                human_validated=False      # → fila de validação humana
            )
            write_memory(mem)
            queue_for_human_validation(mem)

        # Marcar episódicos como distilados
        for ep in raw_episodes:
            conn.execute("UPDATE memories SET distilled=1 WHERE id=?", (ep["id"],))

    # ── Estágio 3: Verificação TrustMem ──────────────────────────────────
    # Verificar cobertura, preservação e fidelidade de cada transição
    pending_verification = conn.execute("""
        SELECT id, content, source
        FROM memories
        WHERE confidence < 0.8 AND distilled = 1 AND human_validated = 0
        LIMIT 20
    """).fetchall()

    for mem in pending_verification:
        # Cross-reference: git log, arquivos atuais, configs
        evidence = gather_evidence(mem["source"])
        trust_score = verify_memory_trustworthiness(
            memory_content=mem["content"],
            evidence=evidence,
            dimensions=["coverage", "preservation", "faithfulness"]  # TrustMem
        )
        new_confidence = min(1.0, mem["confidence"] + trust_score * 0.2)
        conn.execute(
            "UPDATE memories SET confidence=? WHERE id=?",
            (new_confidence, mem["id"])
        )
```

### Política de Limpeza de Memórias Obsoletas

```python
def prune_stale_memories(conn):
    """Remove ou arquiva memórias obsoletas."""

    # Expiradas por TTL
    conn.execute("""
        UPDATE memories SET distilled=1
        WHERE expires_at IS NOT NULL
          AND expires_at < datetime('now')
    """)

    # Não acessadas em 60 dias (episódicas) ou 180 dias (semânticas)
    conn.execute("""
        UPDATE memories SET distilled=1
        WHERE tier = 'episodic'
          AND access_count = 0
          AND created_at < datetime('now', '-60 days')
    """)

    # Supersedidas por versão mais nova
    conn.execute("""
        UPDATE memories SET status='deprecated'
        WHERE id IN (
            SELECT DISTINCT m.id FROM memories m
            JOIN memories newer ON newer.source = 'supersedes:' || m.id
            WHERE newer.confidence > m.confidence
        )
    """)
```

***

## 8. Curadoria por LLM e Validação Humana

O LLM curador analisa o diff de commits ou eventos para extrair memórias candidatas. **Memórias duráveis só são persistidas após aprovação humana** — isso previne alucinações de se tornarem erros persistentes.[^10]

```python
CURATION_PROMPT = """
Você é um curador de memória para um projeto de software.
Analise o contexto abaixo e extraia memórias estruturadas.

COMMIT: {commit_hash}
MENSAGEM: {commit_message}
DIFF:
{diff}

Para cada memória extraída, responda no formato JSON com os campos:
- type: "decision" | "bug_fix" | "refactoring" | "architecture" | "preference"
- content: descrição concisa (máx. 200 caracteres)
- module: módulo afetado (se aplicável)
- confidence: 0.5–0.8 (nunca > 0.8 sem validação humana)
- tags: lista de tags relevantes
- should_save: true | false

REGRAS:
1. Nunca inclua senhas, chaves de API, tokens ou segredos
2. Nunca afirme algo como fact sem evidência no diff
3. Se incerto, baixe o confidence e marque para validação humana
4. Prefira memórias curtas e acionáveis a longas e vagas
"""

def curate_commit(commit_hash: str) -> list[MemoryCandidate]:
    diff = get_commit_diff(commit_hash)
    msg = get_commit_message(commit_hash)
    response = llm.complete(CURATION_PROMPT.format(
        commit_hash=commit_hash,
        commit_message=msg,
        diff=diff[:8000]  # limitar tokens
    ))
    candidates = parse_json_response(response)

    # Filtrar e sanitizar
    sanitized = []
    for c in candidates:
        if detect_secrets(c["content"]):
            log_security_event(c)
            continue
        if c["confidence"] > 0.8:
            c["confidence"] = 0.8   # cap para forçar validação humana
        sanitized.append(c)

    return sanitized
```

### Interface de Validação Humana (CLI)

```bash
# Exemplo: listar memórias pendentes de validação
$ memory validate list
┌─────────────────────────────────────────────────────────────────────┐
│  ID          │ Type       │ Confidence │ Content                    │
├─────────────────────────────────────────────────────────────────────┤
│  mem-abc123  │ decision   │ 0.72       │ JWT com refresh tokens...  │
│  mem-def456  │ bug_fix    │ 0.65       │ Token expirado retorna 500 │
└─────────────────────────────────────────────────────────────────────┘

# Aprovar memória
$ memory validate approve mem-abc123
✅ Memória aprovada. Confidence: 0.72 → 0.92

# Rejeitar com motivo
$ memory validate reject mem-def456 --reason "impreciso, o status code é 401"
❌ Memória rejeitada. Registrada no audit log.
```

***

## 9. Estratégia de Versionamento

### 9.1 O que Versionar no Git

```gitignore
# .gitignore
.memory/store/        # vector store local (reconstruível)
.memory/meta/*.db-shm
.memory/meta/*.db-wal
.memory/events/*.jsonl.gz   # compactados → usar Git LFS

# Versionar sempre:
# .memory/**/*.md
# .memory/events/*.jsonl   (não compactados)
# .memory/graph/*.jsonl
# .memory/meta/memory.db   (opcional: depende do tamanho)
```

### 9.2 Git LFS para Arquivos Grandes

```bash
# Configurar LFS para eventos históricos e banco SQLite
git lfs track ".memory/events/*.jsonl"
git lfs track ".memory/meta/*.db"
git add .gitattributes
```

### 9.3 Estratégia de Branching para Memória

```
main
 ├── feature/auth-refactor
 │    └── .memory/ ← memórias do branch coexistem com main
 └── memory/consolidation-2026-07  ← branch dedicado para grandes consolidações
```

Cada branch pode ter suas próprias memórias episódicas. Ao fazer merge, usa-se a estratégia `ours` para conflitos em `.memory/`, priorizando a versão do branch principal, e posteriormente rodando a consolidação.[^7]

***

## 10. Tratamento de Conflitos e Refatorações

### Conflitos em Memórias ao Fazer Merge

```python
def resolve_memory_conflict(local: Memory, remote: Memory) -> Memory:
    """
    Estratégia: maior confidence vence; empate → mais recente.
    Gerar evento de conflito para revisão humana se confidence similar.
    """
    if abs(local.confidence - remote.confidence) < 0.1:
        # Conflito ambíguo → fila de validação humana
        queue_conflict_for_human(local, remote)
        return local  # mantém local até resolução

    winner = local if local.confidence >= remote.confidence else remote
    loser = remote if winner == local else local

    log_conflict_resolution(
        winner_id=winner.id,
        loser_id=loser.id,
        reason="confidence"
    )
    return winner
```

### Refatorações de Código

Quando um arquivo é renomeado ou um módulo é reestruturado, o hook `post-rewrite` atualiza as referências na memória:

```bash
#!/bin/bash
# .githooks/post-rewrite

# Detectar renames/moves
git diff --diff-filter=R --name-status HEAD~1 HEAD | while read STATUS OLD NEW; do
    if [ "$STATUS" = "R100" ]; then
        python3 .memory/scripts/update_file_refs.py \
            --old "$OLD" --new "$NEW"
    fi
done
```

```python
def update_file_refs(old_path: str, new_path: str):
    """Atualiza referências ao arquivo nos arquivos Markdown de memória."""
    memory_files = glob.glob(".memory/**/*.md", recursive=True)
    for mf in memory_files:
        content = open(mf).read()
        if old_path in content:
            updated = content.replace(old_path, new_path)
            open(mf, 'w').write(updated)
            # Registrar evento de refatoração
            log_event("memory_ref_updated", {
                "file": mf, "old": old_path, "new": new_path
            })
```

***

## 11. Proteção contra Segredos

A proteção opera em múltiplas camadas:[^11][^12]

1. **Pre-commit hook**: Gitleaks ou detect-secrets escaneia arquivos staged antes de qualquer commit[^13]
2. **Curador LLM**: Prompt explicitamente instrui a não capturar segredos
3. **Sanitização no pipeline**: Regex para padrões comuns (AWS keys, JWTs, tokens)
4. **GitHub Secret Scanning**: Varredura automática em PRs e histórico[^12]

```python
SECRET_PATTERNS = [
    r"(password|passwd|pwd)\s*[:=]\s*['\"]?[^\s'\"]{8,}",
    r"(api_key|apikey|api-key)\s*[:=]\s*['\"]?[A-Za-z0-9_\-]{16,}",
    r"(secret|token)\s*[:=]\s*['\"]?[A-Za-z0-9+/=]{20,}",
    r"sk-[A-Za-z0-9]{48}",          # OpenAI
    r"AKIA[0-9A-Z]{16}",            # AWS Access Key
    r"ghp_[A-Za-z0-9]{36}",         # GitHub PAT
]

def sanitize_memory_content(content: str) -> tuple[str, bool]:
    """
    Returns: (sanitized_content, found_secret)
    """
    for pattern in SECRET_PATTERNS:
        if re.search(pattern, content, re.IGNORECASE):
            return "[REDACTED]", True
    return content, False
```

***

## 12. Logs de Auditoria

Todo acesso, escrita, validação e remoção de memória é registrado de forma imutável:[^5]

```python
def log_audit(
    action: str,
    actor: str,
    memory_id: str = None,
    details: dict = None
):
    """Registro imutável no audit_log (SQLite) e events/audit_log.jsonl."""

    record = {
        "id": f"audit-{uuid4()}",
        "ts": datetime.utcnow().isoformat() + "Z",
        "action": action,        # write|read|validate|reject|prune|conflict
        "actor": actor,          # "llm:gpt-4o"|"human:developer"|"hook:post-commit"
        "memory_id": memory_id,
        "details": details or {}
    }

    # Append ao JSONL (crash-safe: flush imediato)
    with open(".memory/events/audit_log.jsonl", "a") as f:
        f.write(json.dumps(record) + "\n")
        f.flush()

    # Inserir no SQLite
    db.execute(
        "INSERT INTO audit_log (event_id, action, actor, memory_id, details, ts) VALUES (?,?,?,?,?,?)",
        (record["id"], action, actor, memory_id, json.dumps(details), record["ts"])
    )
```

***

## 13. Avaliação da Qualidade da Memória

### Métricas de Saúde do Sistema

```sql
-- 1. Razão de memórias validadas (meta: > 70%)
SELECT
    ROUND(100.0 * SUM(human_validated) / COUNT(*), 1) as validated_pct
FROM memories WHERE distilled = 1;

-- 2. Memórias não verificadas (meta: < 1000)
SELECT COUNT(*) as unverified FROM memories WHERE distilled = 0;

-- 3. Memórias obsoletas (não acessadas em 30+ dias)
SELECT COUNT(*) as stale FROM memories
WHERE access_count = 0 AND created_at < datetime('now', '-30 days');

-- 4. Distribuição de confidence
SELECT
    CASE
        WHEN confidence >= 0.9 THEN 'alta (≥0.9)'
        WHEN confidence >= 0.7 THEN 'média (0.7–0.9)'
        ELSE 'baixa (<0.7)'
    END as faixa,
    COUNT(*) as total
FROM memories GROUP BY faixa;

-- 5. Taxa de uso (memórias úteis vs. nunca acessadas)
SELECT
    ROUND(100.0 * SUM(CASE WHEN access_count > 0 THEN 1 ELSE 0 END) / COUNT(*), 1) as usage_rate
FROM memories WHERE distilled = 1;
```

### Métricas de Qualidade de Recuperação

```python
def evaluate_retrieval_quality(
    test_queries: list[dict],   # [{"query": "...", "expected_memory_ids": [...]}]
) -> dict:
    """
    Métricas RAG padrão: Precision@K, Recall@K, MRR, NDCG.
    """
    precisions, recalls, mrr_scores = [], [], []

    for test in test_queries:
        retrieved = retrieve_relevant_memories(test["query"], k=5)
        retrieved_ids = [m.id for m in retrieved]
        expected = set(test["expected_memory_ids"])
        retrieved_set = set(retrieved_ids)

        precision = len(retrieved_set & expected) / len(retrieved_set)
        recall = len(retrieved_set & expected) / len(expected)

        # Mean Reciprocal Rank
        mrr = 0.0
        for rank, mid in enumerate(retrieved_ids, start=1):
            if mid in expected:
                mrr = 1.0 / rank
                break

        precisions.append(precision)
        recalls.append(recall)
        mrr_scores.append(mrr)

    return {
        "precision_at_5": round(sum(precisions) / len(precisions), 3),
        "recall_at_5":    round(sum(recalls) / len(recalls), 3),
        "mrr":            round(sum(mrr_scores) / len(mrr_scores), 3),
    }
```

### Dashboard de Saúde (Output semanal)

```
╔══════════════════════════════════════════════════════════╗
║         RELATÓRIO DE SAÚDE DA MEMÓRIA  –  2026-07-02     ║
╠══════════════════════════════════════════════════════════╣
║  Total de memórias:          1,247                       ║
║  Distiladas e validadas:       894  (71.7%) ✅           ║
║  Pendentes de validação:       203  (16.3%) ⚠️           ║
║  Não acessadas (30d):          150  (12.0%) ⚠️           ║
║  Confiança média:              0.81                      ║
║  Precision@5 (última avaliação): 0.847                   ║
║  Recall@5:                     0.792                     ║
║  MRR:                          0.731                     ║
╠══════════════════════════════════════════════════════════╣
║  🔴 AÇÃO NECESSÁRIA: 203 memórias aguardam validação     ║
║  🟡 SUGESTÃO: Rodar consolidação (estágio 2)             ║
╚══════════════════════════════════════════════════════════╝
```

***

## 14. Escolha do Vector Store

| Critério | ChromaDB | Qdrant | pgvector |
|---|---|---|---|
| **Ambiente** | Desenvolvimento local | Produção / escala | Se já usa PostgreSQL |
| **Setup** | Zero (Python puro) | Docker / binário Rust | Extensão PostgreSQL |
| **Hybrid search** | Limitado | ✅ Nativo (dense + sparse) | ✅ Com tsvector |
| **Performance (50M vetores)** | Não recomendado | ~1ms p99 | ~2–5ms p99 |
| **Git-friendly** | ✅ (persiste em disco) | ❌ (servidor separado) | ❌ (servidor separado) |
| **Custo infra** | $0 | $0 (self-hosted) | $0 (self-hosted) |

Para projetos de software individual ou equipes pequenas, **ChromaDB local** é suficiente e git-friendly. Para produção com múltiplos agentes, **Qdrant** oferece busca híbrida nativa e performance superior.[^14][^15][^16][^17]

***

## 15. Exemplo Completo de Sessão de Trabalho

```
[09:00] Agente inicia nova tarefa: "Refatorar módulo de autenticação"

→ retrieve_relevant_memories("refatorar autenticação JWT")
   Carrega:
   - arch-decision-001: "JWT com refresh tokens" (confidence: 0.92, score: 0.91)
   - bug-042: "Token expirado deve retornar 401" (confidence: 0.88, score: 0.87)
   - module-auth: "src/auth/ – 3 services" (confidence: 0.95, score: 0.84)
   Total: 3.200 tokens de contexto injetado

→ Agente executa refatoração
→ git commit -m "refactor(auth): extrair TokenService"

[09:15] post-commit hook dispara
   → Diff enviado ao curador LLM
   → LLM extrai 2 memórias candidatas (confidence: 0.70)
   → Sanitização: nenhum segredo encontrado
   → Fila de validação humana

[09:16] Notificação para desenvolvedor:
   "2 memórias candidatas aguardam validação em .memory/"

[09:20] Desenvolvedor aprova ambas via CLI
   → confidence: 0.70 → 0.90
   → Escrita em .memory/modules/auth.md
   → git add .memory/ && git commit -m "memory: atualizar auth module"
   → Embeddings atualizados no vector store
   → Knowledge graph: novo nó "TokenService" + relação "part_of:auth"
```

***

## Referências de Frameworks e Ferramentas

- **Mem0** / **MemoryOS** — frameworks de memória para agentes LLM em produção[^10]
- **TrustMem** (Samsung / Notre Dame, 2026) — verificação trustworthy de transições de memória[^10]
- **ChromaDB / Qdrant** — vector stores para busca semântica local e em produção[^16][^14]
- **Gitleaks / detect-secrets / GitHub Secret Scanning** — proteção contra vazamento de segredos[^11][^12][^13]
- **Reciprocal Rank Fusion (RRF)** — fusão de rankings para busca híbrida BM25 + vector[^9][^8]
- **Neo4j GraphRAG** — construção de knowledge graphs a partir de texto não estruturado[^18]
- **AGENTS.md / CLAUDE.md** — padrão de memory files para agentes de código[^19][^4]
- **AgentTrace / JSONL** — formato nativo para observabilidade de agentes[^5]

---

## References

1. [Building Persistent Memory for AI Agents: A 4-Layer File- ...](https://dev.to/oblivionlabz/building-persistent-memory-for-ai-agents-a-4-layer-file-based-architecture-4gc4) - Building Persistent Memory for AI Agents: A 4-Layer File-Based Architecture. As AI agents become mor...

2. [AI Agent Memory Systems: How to Give Your AI Persistent ...](https://www.openclawplaybook.ai/blog/ai-agent-memory-systems/) - The three-layer memory architecture that makes AI agents actually useful across sessions — short-ter...

3. [I Gave My AI Agent Memory Across Sessions. Here's the Schema.](https://dev.to/thestack_ai/i-gave-my-ai-agent-memory-across-sessions-heres-the-schema-57ng) - It tracks 187 entities, 128 relationships, and distills thousands of raw memories into actionable co...

4. [Project Memory & Rules | Luca Nerlich - Tech Documentation](https://lucanerlich.com/ai/project-memory-and-rules/) - Always-on context for coding agents - AGENTS.md, CLAUDE.md, Cursor rules, and how to split conventio...

5. [JSONL as the Native Observability Format for AI Agent Runtimes | Zylos Research](https://zylos.ai/research/2026-06-06-jsonl-agent-observability-data-format/) - Why JSONL's append-only, line-delimited structure maps naturally onto agent execution, and how to de...

6. [Building Agent Memory with the Ontology Skill: A Knowledge Graph for Clawdbot](https://clawdev.to/posts/building-agent-memory-with-the-ontology-skill-a-knowledge-graph-for-clawdbot-mlr6k7x8) - Ever wanted your AI agent to actually *remember* things in a structured way? Not just dumping text i...

7. [Git hooks are your best defense against AI-generated mess](https://jonesrussell.github.io/blog/git-hooks-ai-agents/) - A pre-commit hook catches it before the commit exists. The agent gets the failure, adjusts, and trie...

8. [Why we stopped using vector-only retrieval for agent memory (and what we use instead)](https://www.reddit.com/r/LangChain/comments/1sc786c/why_we_stopped_using_vectoronly_retrieval_for/) - Why we stopped using vector-only retrieval for agent memory (and what we use instead)

9. [Hybrid Search (BM25 + Vector)](https://aiundecided.com/rag-strategies/hybrid-search/) - Combine keyword search (BM25) with semantic vector search

10. [TrustMem: Learning Trustworthy Memory Consolidation for LLM ...](https://arxiv.org/html/2606.25161v1)

11. [Secret Scanning Explained: Stop Credential Leaks - GraphNode](https://www.graphnodesoftware.com/guides/secret-scanning) - A vendor-neutral guide to secret scanning: how scanners detect hardcoded credentials, where to run t...

12. [Customizing Secret Scanning](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning) - GitHub scans repositories for known types of secrets, to prevent fraudulent use of secrets that were...

13. [Pre-Commit Secret Scanning - Checkmarx Documentation](https://docs.checkmarx.com/en/34965-364702-pre-commit-secret-scanning.html) - The Pre-Commit Secret Scanning feature helps prevent accidental exposure of sensitive information su...

14. [LangChain - Qdrant](https://qdrant.tech/documentation/frameworks/langchain/) - Use the LangChain Qdrant integration to power semantic search and RAG with dense, sparse, and hybrid...

15. [Choosing the Best Vector Database For Your RAG Pipeline (Chroma vs Pinecone vs pgvector vs Qdrant)](https://www.youtube.com/watch?v=JTAI-7pUWE8) - Most teams default to ChromaDB in development and then scramble when they need to go to production. ...

16. [Qdrant vs ChromaDB: Choosing the Right Vector Database for Your ...](https://facsiaginsa.com/ai/qdrant-vs-chromadb-vector-database-comparison) - A hands-on comparison of Qdrant and ChromaDB across architecture, API design, embedding support, per...

17. [Chroma vs Qdrant: Best Vector Database for Local Development](https://zenvanriel.com/ai-engineer-blog/chroma-vs-qdrant-local-development/) - For large vector stores, this competes with your application's memory. Qdrant runs in a separate pro...

18. [User Guide: Knowledge Graph Builder - Neo4j](https://neo4j.com/docs/neo4j-graphrag-python/current/user_guide_kg_builder.html)

19. [AGENTS.md - Wener Live & Life](https://wener.me/notes/ai/dev/memory) - - openai/agents.md

