# Memória de Projeto com Embeddings e Vector Store — Implementação Intermediária para Agente de Programação

## Visão Geral

Esta arquitetura propõe um sistema de **memória persistente e semântica** para agentes de programação, combinando uma pasta `.memory/` versionada em Git com um vector store local ou gerenciado. O objetivo é permitir recuperação semântica de decisões de projeto, contexto de módulos, erros conhecidos e padrões de código, com filtros por metadados, atualização incremental e proteção contra segredos.

***

## 1. Arquitetura Geral

```
project-root/
├── .memory/
│   ├── raw/                    # Arquivos fonte (markdown, YAML)
│   │   ├── decisions/
│   │   ├── modules/
│   │   ├── errors/
│   │   └── patterns/
│   ├── index/                  # Vector store persistido (ex: LanceDB ou FAISS)
│   │   └── memories.lance      # (ou faiss.index + docstore.pkl)
│   ├── manifest.json           # Inventário com hash SHA256 de cada arquivo
│   ├── config.yaml             # Modelo de embedding, dimensão, thresholds
│   └── .gitignore              # Exclui secrets e arquivos temporários
├── CLAUDE.md / AGENTS.md       # Contexto carregado automaticamente pelo agente
└── ...
```

A pasta `.memory/raw/` contém os arquivos Markdown com YAML frontmatter como fonte de verdade, enquanto `.memory/index/` armazena os vetores derivados. Esse padrão é análogo ao que projetos como `agent-memories` e `Agents Remember` utilizam — memórias como arquivos versionados no Git com metadados estruturados.[^1][^2]

A vantagem de versionar `.memory/` inteira no Git é que cada commit representa um snapshot da memória do agente, permitindo `git log`, `git diff` e rollback completo do estado cognitivo do projeto.[^3][^4]

***

## 2. Schema de Memória (YAML Frontmatter)

Cada arquivo em `.memory/raw/` segue um schema estruturado:

```yaml
---
id: "mem_2026_0702_001"
title: "Decisão: usar LanceDB como vector store embarcado"
type: decision           # decision | error | pattern | context | dependency
module: "memory-subsystem"
tags: ["vector-store", "lancedb", "embedding"]
confidence: 0.92         # 0.0 a 1.0 — degradada pelo sistema ao longo do tempo
created_at: "2026-07-02T08:00:00-03:00"
updated_at: "2026-07-02T08:00:00-03:00"
expires_at: null         # null = permanente; ISO-date = obsolescência programada
related_files:
  - "src/memory/store.py"
  - ".memory/config.yaml"
related_memories:
  - "mem_2026_0701_003"
author: "agent:claude-4"
status: active           # active | obsolete | superseded | draft
superseded_by: null
checksum: "sha256:abc123..."  # hash do conteúdo — detecta modificação externa
secret_scan_passed: true     # flag de auditoria pré-ingestão
---

## Conteúdo

LanceDB foi escolhido por rodar embutido (sem servidor), suportar versionamento nativo
via formato Lance, e ter integração direta com PyArrow e embeddings do SentenceTransformers.
Custo: $0 para uso local. Troca futura para Qdrant Cloud requer apenas swap de endpoint.
```

### Campos de Metadados — Propósito

| Campo | Tipo | Finalidade |
|-------|------|-----------|
| `type` | enum | Filtro por categoria de memória |
| `module` | string | Escopo de recuperação por módulo do projeto |
| `confidence` | float | Threshold de relevância; degradado por obsolescência |
| `status` | enum | Filtro ativo/obsoleto durante recuperação |
| `related_files` | list | Recuperação contextual ao abrir um arquivo específico |
| `expires_at` | ISO date | TTL — declara obsolescência programada |
| `checksum` | sha256 | Detecta modificações externas, dispara reindexação |
| `secret_scan_passed` | bool | Auditoria obrigatória antes da ingestão no vector store |

***

## 3. Pipeline de Ingestão e Reindexação

### 3.1 Fluxo de Ingestão (pseudo-código Python)

```python
import hashlib, yaml, re
from pathlib import Path
from sentence_transformers import SentenceTransformer
import lancedb
import pyarrow as pa

EMBED_MODEL = SentenceTransformer("all-MiniLM-L6-v2")  # 384 dims, local, rápido
SECRET_PATTERNS = [
    r"(?i)(api[_\s]?key|token|secret|password)\s*[:=]\s*['\"]?\w{16,}",
    r"AKIA[0-9A-Z]{16}",    # AWS Access Key
    r"ghp_[A-Za-z0-9]{36}", # GitHub token
    r"sk-[A-Za-z0-9]{48}",  # OpenAI key
]

def scan_secrets(text: str) -> list[str]:
    """Retorna lista de padrões detectados — bloqueia ingestão se não-vazio."""
    found = []
    for pat in SECRET_PATTERNS:
        if re.search(pat, text):
            found.append(pat)
    return found

def parse_memory_file(path: Path) -> dict | None:
    raw = path.read_text(encoding="utf-8")
    
    # 1. Proteção contra segredos — ANTES de qualquer processamento
    hits = scan_secrets(raw)
    if hits:
        print(f"[BLOCKED] {path}: secret patterns detected: {hits}")
        return None
    
    # 2. Parse frontmatter + corpo
    parts = raw.split("---", 2)
    if len(parts) < 3:
        return None
    meta = yaml.safe_load(parts[^1])
    body = parts[^2].strip()
    
    # 3. Verificar checksum (detectar modificação externa)
    current_hash = "sha256:" + hashlib.sha256(raw.encode()).hexdigest()
    if meta.get("checksum") and meta["checksum"] != current_hash:
        print(f"[WARN] Checksum mismatch: {path} — reindexação necessária")
    
    return {**meta, "body": body, "path": str(path), "checksum": current_hash}

def embed_and_upsert(db_path: str, memory: dict):
    db = lancedb.connect(db_path)
    table_name = "memories"
    
    vector = EMBED_MODEL.encode(memory["body"]).tolist()
    
    record = {
        "id": memory["id"],
        "vector": vector,
        "title": memory["title"],
        "body": memory["body"],
        "type": memory["type"],
        "module": memory.get("module", ""),
        "confidence": memory.get("confidence", 1.0),
        "status": memory.get("status", "active"),
        "created_at": memory.get("created_at", ""),
        "updated_at": memory.get("updated_at", ""),
        "expires_at": memory.get("expires_at", ""),
        "related_files": str(memory.get("related_files", [])),
        "tags": str(memory.get("tags", [])),
        "path": memory["path"],
    }
    
    schema = pa.schema([
        pa.field("id", pa.string()),
        pa.field("vector", pa.list_(pa.float32(), 384)),
        pa.field("title", pa.string()),
        pa.field("body", pa.string()),
        pa.field("type", pa.string()),
        pa.field("module", pa.string()),
        pa.field("confidence", pa.float32()),
        pa.field("status", pa.string()),
        pa.field("created_at", pa.string()),
        pa.field("updated_at", pa.string()),
        pa.field("expires_at", pa.string()),
        pa.field("related_files", pa.string()),
        pa.field("tags", pa.string()),
        pa.field("path", pa.string()),
    ])
    
    if table_name not in db.table_names():
        tbl = db.create_table(table_name, schema=schema)
    else:
        tbl = db.open_table(table_name)
    
    # Upsert: delete + insert (LanceDB suporta update nativo na v0.33+)
    try:
        tbl.delete(f"id = '{memory['id']}'")
    except Exception:
        pass
    tbl.add([record])
```

### 3.2 Reindexação Incremental com `manifest.json`

```python
import json
from datetime import datetime

def build_manifest(memory_dir: Path) -> dict:
    manifest = {}
    for f in memory_dir.glob("raw/**/*.md"):
        content = f.read_bytes()
        manifest[str(f)] = {
            "sha256": hashlib.sha256(content).hexdigest(),
            "indexed_at": datetime.utcnow().isoformat()
        }
    return manifest

def incremental_reindex(memory_dir: Path, db_path: str):
    manifest_path = memory_dir / "manifest.json"
    old_manifest = json.loads(manifest_path.read_text()) if manifest_path.exists() else {}
    new_manifest = {}
    
    for f in memory_dir.glob("raw/**/*.md"):
        key = str(f)
        sha = hashlib.sha256(f.read_bytes()).hexdigest()
        new_manifest[key] = {"sha256": sha, "indexed_at": datetime.utcnow().isoformat()}
        
        # Indexar apenas se novo ou modificado
        if old_manifest.get(key, {}).get("sha256") != sha:
            memory = parse_memory_file(f)
            if memory and memory.get("secret_scan_passed", False) is not False:
                embed_and_upsert(db_path, memory)
                print(f"[INDEXED] {f.name}")
    
    manifest_path.write_text(json.dumps(new_manifest, indent=2))
```

***

## 4. Recuperação Semântica com Filtros

```python
from datetime import datetime, timezone

def retrieve_memories(
    query: str,
    db_path: str,
    module: str | None = None,
    memory_type: str | None = None,
    min_confidence: float = 0.5,
    related_file: str | None = None,
    top_k: int = 5,
) -> list[dict]:
    db = lancedb.connect(db_path)
    tbl = db.open_table("memories")
    
    query_vec = EMBED_MODEL.encode(query).tolist()
    
    # Filtro SQL-style (LanceDB suporta WHERE nativo)
    filters = [
        "status = 'active'",
        f"confidence >= {min_confidence}",
    ]
    if module:
        filters.append(f"module = '{module}'")
    if memory_type:
        filters.append(f"type = '{memory_type}'")
    if related_file:
        # Busca substring (LanceDB WHERE com LIKE)
        filters.append(f"related_files LIKE '%{related_file}%'")
    
    where_clause = " AND ".join(filters)
    
    results = (
        tbl.search(query_vec)
           .where(where_clause)
           .limit(top_k)
           .to_list()
    )
    
    # Pós-processamento: filtrar obsoletas por data
    now = datetime.now(timezone.utc).isoformat()
    filtered = [
        r for r in results
        if not r["expires_at"] or r["expires_at"] > now
    ]
    
    return filtered
```

### 4.1 Exemplo de Consulta do Agente

```python
# Agente abrindo src/memory/store.py — busca contexto relevante
memories = retrieve_memories(
    query="como o vector store está sendo inicializado e conectado",
    db_path=".memory/index",
    module="memory-subsystem",
    related_file="store.py",
    min_confidence=0.7,
    top_k=3
)

for m in memories:
    print(f"[{m['type'].upper()}] {m['title']}")
    print(f"  Confidence: {m['confidence']:.2f} | Score: {m['_distance']:.4f}")
    print(f"  {m['body'][:200]}...")
    print()
```

***

## 5. Detecção de Obsolescência

A obsolescência pode ser passiva (por TTL) ou ativa (por detecção de divergência):

```python
def detect_obsolete_memories(db_path: str, project_files: list[Path]) -> list[str]:
    """
    Marca como 'obsolete' memórias cujos related_files não existem mais
    ou cujo expires_at já passou.
    """
    db = lancedb.connect(db_path)
    tbl = db.open_table("memories")
    all_memories = tbl.to_pandas()
    
    existing_files = {str(f) for f in project_files}
    now = datetime.now(timezone.utc).isoformat()
    obsolete_ids = []
    
    for _, row in all_memories.iterrows():
        # Obsolescência por TTL
        if row["expires_at"] and row["expires_at"] < now:
            obsolete_ids.append(row["id"])
            continue
        
        # Obsolescência por arquivo removido
        related = eval(row["related_files"])  # lista serializada como string
        if related and not any(f in existing_files for f in related):
            obsolete_ids.append(row["id"])
    
    # Atualizar status no vector store
    for mem_id in obsolete_ids:
        tbl.update(
            where=f"id = '{mem_id}'",
            values={"status": "obsolete", "confidence": 0.0}
        )
    
    return obsolete_ids

# Degradação temporal de confiança (decaimento exponencial)
def decay_confidence(db_path: str, half_life_days: int = 90):
    """Reduz confidence de memórias antigas gradualmente."""
    db = lancedb.connect(db_path)
    tbl = db.open_table("memories")
    df = tbl.to_pandas()
    now = datetime.now(timezone.utc)
    
    for _, row in df.iterrows():
        if row["status"] != "active":
            continue
        created = datetime.fromisoformat(row["created_at"])
        age_days = (now - created).days
        decay = 0.5 ** (age_days / half_life_days)
        new_confidence = round(float(row["confidence"]) * decay, 4)
        if new_confidence < 0.1:
            tbl.update(where=f"id = '{row['id']}'", values={"status": "obsolete"})
        else:
            tbl.update(where=f"id = '{row['id']}'", values={"confidence": new_confidence})
```

***

## 6. Proteção Contra Segredos

A proteção opera em três camadas complementares:[^5][^6]

### Camada 1 — Pre-commit hook (`.memory/hooks/pre-commit`)
```bash
#!/bin/bash
# Instalar: cp .memory/hooks/pre-commit .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit
echo "🔍 Scanning .memory/ for secrets..."
trufflehog git file://. --since-commit HEAD --branch HEAD --fail --only-verified
if [ $? -ne 0 ]; then
    echo "❌ Secrets detected — commit abortado. Remova as credenciais antes de commitar."
    exit 1
fi
echo "✅ No secrets found."
```

### Camada 2 — `.memory/.gitignore`
```gitignore
# Nunca versionar esses arquivos
*.env
*.key
*.pem
secrets.*
*_token.*
.env.*
# Pastas temporárias
tmp/
cache/
__pycache__/
```

### Camada 3 — Sanitização inline antes da ingestão
```python
REDACT_PATTERNS = [
    (r"(api[_\s]?key\s*[:=]\s*)['\"]?[\w\-]{16,}['\"]?", r"\1[REDACTED]"),
    (r"(AKIA[0-9A-Z]{16})", "[AWS_KEY_REDACTED]"),
    (r"(sk-[A-Za-z0-9]{48})", "[OPENAI_KEY_REDACTED]"),
    (r"(ghp_[A-Za-z0-9]{36})", "[GITHUB_TOKEN_REDACTED]"),
]

def sanitize_content(text: str) -> str:
    for pattern, replacement in REDACT_PATTERNS:
        text = re.sub(pattern, replacement, text, flags=re.IGNORECASE)
    return text
```

***

## 7. Exemplo de Fluxo Completo

```
1. Agente conclui uma decisão arquitetural
   → Cria `.memory/raw/decisions/2026-07-02-lancedb-choice.md` com frontmatter completo

2. Pre-commit hook executa TruffleHog
   → Nenhum segredo detectado → commit realizado
   → Git versiona o estado da memória

3. incremental_reindex() detecta novo arquivo via manifest.json
   → parse_memory_file() valida frontmatter e roda scan_secrets()
   → embed_and_upsert() gera vetor e persiste no LanceDB

4. Próxima sessão do agente ao abrir store.py:
   → retrieve_memories(query="inicialização do vector store", related_file="store.py")
   → Retorna as 3 memórias mais relevantes com scores e metadados

5. Sistema de manutenção semanal:
   → detect_obsolete_memories() verifica related_files vs. arquivos existentes
   → decay_confidence() degrada memórias com >90 dias
   → Memórias expiradas recebem status="obsolete" e saem das buscas
```

***

## 8. Comparativo de Vector Stores

### 8.1 Tabela de Decisão

| Vector Store | Modo Local | Servidor? | Escalabilidade | Filtros Metadata | Quantização | Custo Cloud | Maturidade | Ideal para agente |
|---|---|---|---|---|---|---|---|---|
| **Chroma** | ✅ embarcado | Opcional | ≤500k vetores[^7] | Básico (Python-side)[^7] | ❌ | Free tier[^8] | Alta (DX) | Protótipos |
| **FAISS** | ✅ embarcado | ❌ | 100M+ (RAM) | ❌ nativo[^9] | ✅ PQ, SQ[^9] | ❌ (lib) | Muito Alta | RAM ilimitada, sem filtros |
| **LanceDB** | ✅ embarcado | ❌ | 10M+ (disco)[^7] | ✅ SQL-style[^10] | Parcial | Free OSS[^11] | Média-Alta | **Agente local ideal** |
| **Qdrant** | ✅ in-memory/disk | Docker/nativo | 100M+[^7] | ✅ payload index (pré-filtro)[^12] | ✅ INT8 (~4x)[^7] | Free 1GB[^13] | Alta | Produção OSS |
| **Weaviate** | ✅ self-host | Docker | 1B+ cloud | ✅ GraphQL + BM25[^14] | ✅ | Flex $45/mês[^15] | Alta | Hybrid search |
| **pgvector** | ✅ (Postgres) | Postgres | ≤50M vetores[^7] | ✅ SQL completo[^16] | ❌ (pgvectorscale) | Managed Postgres | Muito Alta | Stack Postgres existente |
| **Pinecone** | ❌ managed only | N/A | Ilimitado | ✅ | ✅ | $70+/mês | Alta | SaaS managed, sem ops |
| **Vertex AI Vector** | ❌ GCP only | N/A | Ilimitado | ✅ | ✅ | Pay-per-query[^17] | Alta | Projetos GCP/BigQuery |

### 8.2 Análise Detalhada por Opção

#### Chroma
Chroma é a opção de menor fricção: três linhas de Python e um `pip install chromadb` criam um vector store funcional com persistência em disco via SQLite + HNSW. A interface é limpa e a integração com LangChain e LlamaIndex é nativa. O problema central é a **arquitetura single-process**: sem sharding, sem quantização nativa, sem suporte a multi-tenant. Uma coleção de 10M vetores de 1536 dimensões ocupa ~57GB de RAM sem caminho de otimização. Para uma pasta `.memory/` de projeto pequeno (< 100k memórias), Chroma é legítimo. Para qualquer escala maior ou carga concorrente, é contraproducente.[^18][^19][^7]

#### FAISS
FAISS (Facebook AI Similarity Search) é uma **biblioteca**, não um banco de dados. Ela não persiste metadados, não faz filtragem nativa por payload, e todos os índices precisam caber em RAM. Oferece os tipos de índice mais variados do mercado: `IndexFlatL2` (exato, brute-force), `IndexIVFFlat` (aproximado por clusters), `IndexHNSWFlat` (grafo navegável) e `IndexIVFPQ` (product quantization para máxima compressão). Para memória de agente, exige um `docstore` externo (pickle, SQLite) para mapear IDs de vetores aos metadados — acoplamento frágil e sem transações. O ponto forte é o **suporte a GPU** e a capacidade de operar com bilhões de vetores em RAM. A ausência de filtros nativos e a impossibilidade de deletar vetores em índices HNSW são desvantagens críticas para atualização incremental.[^20][^9][^21][^22]

#### LanceDB
LanceDB é a **recomendação primária** para este caso de uso. Roda embutido sem servidor, persiste dados no formato columnar Lance/Arrow em disco (não em RAM), suporta versionamento automático de tabelas a cada mutação, e oferece filtros SQL-style nativos com suporte a `WHERE`. O formato Lance V2 permite evolução de schema sem reescrita completa da tabela. A ausência de autenticação nativa e a menor maturidade dos filtros complexos em relação ao Qdrant são limitações reais, mas irrelevantes para uso local em um único projeto. O upgrade para Qdrant Cloud ou LanceDB Cloud requer apenas swap de endpoint.[^10][^11][^7][^23][^24]

#### Qdrant
Qdrant é o **melhor vetor store open-source para produção**. Escrito em Rust, apresenta latência p99 de ~12ms em 10M vetores e suporte a filtros de payload com pré-indexação (ao contrário do pgvector que faz pós-filtro). A quantização INT8 reduz uso de RAM em ~4x com menos de 1% de perda de recall. Desde julho 2025, o **Qdrant Edge** oferece execução in-process (sem Docker) para ambientes embarcados, tornando-o competitivo com LanceDB para uso local. O free tier gerenciado inclui 1GB e ~1M vetores. A principal desvantagem para um agente de programação local é a dependência histórica de Docker para o modo server — mitigada pelo Qdrant Edge (ainda em beta privado).[^25][^26][^27][^7][^17][^13]

#### Weaviate
Weaviate é líder em **hybrid search** (vetor + BM25 + filtros), com uma API GraphQL que diferencia do restante. É a escolha quando a recuperação precisa combinar similaridade semântica com correspondência exata de palavras-chave (nomes de funções, versões, IDs). O plano gerenciado Flex custa $45/mês. Para uso local de agente, o overhead operacional (Docker + configuração de schema) é excessivo, a menos que hybrid search seja requisito. O novo Weaviate Embeddings como serviço gerenciado e o Query Agent pré-construído tornam-no atraente para stacks de IA mais complexas.[^14][^15][^17]

#### pgvector
pgvector é a **opção zero-overhead para quem já roda PostgreSQL**. Uma linha SQL (`CREATE EXTENSION vector`) adiciona suporte a vetores e índices HNSW ou IVFFlat. A filtragem SQL completa é a mais expressiva do mercado. Os limites são claros: acima de 50M vetores a performance degrada, e filtros seletivos em HNSW fazem pós-filtro (reduzindo recall). Para a pasta `.memory/` de um projeto, raramente se excede 100k vetores, tornando pgvector uma escolha legítima se o projeto já usa Supabase, Neon ou PostgreSQL local. O `pgvectorscale` (extensão da Timescale) adiciona quantização e melhora o throughput para 50M vetores, mas é uma dependência adicional.[^16][^7][^17]

#### Opções Gerenciadas (Pinecone, Vertex AI Vector Search)
Pinecone é o default gerenciado multi-cloud, sem necessidade de operações de infraestrutura, com hybrid search nativo e escala ilimitada. O custo começa em ~$70/mês e cresce com volume. Vertex AI Vector Search é a escolha natural para projetos 100% GCP, com integração nativa ao BigQuery e ao ecossistema Vertex AI — relevante para o contexto da Totalcad, dado o uso de Google Cloud. Ambas são **inadequadas para o cenário de agente local de programação** onde confidencialidade do código e custo zero são requisitos.[^17]

***

## 9. Recomendação de Escolha

```
Contexto: agente de programação local, projeto único, equipe pequena
        ↓
[ Já usa PostgreSQL? ]
    → SIM → pgvector (zero infra, SQL, < 50M vetores)
    → NÃO ↓

[ Precisa de servidor zero / embarcado? ]
    → SIM → LanceDB (recomendado: disco, versioning, SQL-filter)
    → NÃO ↓

[ Precisa de filtros complexos em produção? ]
    → SIM → Qdrant (Docker ou Qdrant Edge quando sair do beta)
    → NÃO → Chroma (protótipo rápido)

[ Hybrid search é requisito (semântico + keyword)? ]
    → SIM → Weaviate (OSS self-hosted) ou Qdrant
    → NÃO → LanceDB / pgvector
```

**Para este sistema de memória de projeto: LanceDB como default local, com migração opcional para Qdrant Cloud (free tier) em caso de uso multi-agente ou necessidade de API REST.**

***

## 10. Configuração de Exemplo — `.memory/config.yaml`

```yaml
# .memory/config.yaml
embedding:
  model: "sentence-transformers/all-MiniLM-L6-v2"  # 384 dims, local, MIT license
  dimension: 384
  batch_size: 32

vector_store:
  backend: "lancedb"          # chroma | faiss | lancedb | qdrant
  path: ".memory/index"
  table_name: "memories"

retrieval:
  default_top_k: 5
  min_confidence: 0.5
  decay_half_life_days: 90    # confiança reduzida à metade a cada 90 dias
  max_results_per_module: 10

security:
  scan_secrets_on_ingest: true
  block_on_secret_detected: true
  redact_before_embed: true
  tools:
    - "detect-secrets"        # pip install detect-secrets
    - "trufflehog"            # ou via pre-commit hook

indexing:
  auto_reindex_on_change: true
  manifest_path: ".memory/manifest.json"
  watch_glob: "raw/**/*.md"

obsolescence:
  check_on_startup: true
  check_related_files: true
  ttl_field: "expires_at"
```

***

## 11. Integração com Git

```bash
# Inicialização do sistema de memória
git init  # se ainda não inicializado
mkdir -p .memory/{raw/{decisions,modules,errors,patterns},index}

# Configurar pre-commit hook para scan de segredos
pip install pre-commit detect-secrets
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
        files: '^\.memory/'
EOF
pre-commit install

# Inicializar baseline de segredos (arquivos legítimos)
detect-secrets scan .memory/ > .secrets.baseline

# Commitar estrutura inicial
git add .memory/ .pre-commit-config.yaml .secrets.baseline
git commit -m "chore: initialize .memory/ project memory system"
```

***

## 12. Resumo Executivo

A implementação proposta combina a simplicidade do Git para versionamento com a potência semântica de um vector store local. O fluxo é: **arquivo Markdown com YAML frontmatter → scan de segredos → embedding local → upsert no vector store → recuperação filtrada por módulo/tipo/confiança**.

LanceDB é a escolha primária por rodar embutido, ter versionamento automático por design e suporte a filtros SQL-style sem servidor. Qdrant é a escolha para quando o projeto crescer ou precisar de API REST. pgvector é o caminho de menor resistência para stacks já PostgreSQL.[^7][^23][^10]

A proteção contra segredos opera em três camadas (pre-commit, `.gitignore`, sanitização inline), seguindo as recomendações de segurança para vector stores que tratam dados sensíveis como ativos de produção. A detecção de obsolescência combina TTL explícito (campo `expires_at`), verificação de `related_files` existentes e decaimento exponencial de confiança temporal — garantindo que a memória do agente reflita o estado atual do projeto, não seu histórico.[^5]

---

## References

1. [agent-memories - Lib.rs](https://lib.rs/crates/agent-memories) - Convention and validation for committed AI agent memories

2. [Agents Remember: Git-Verified Project Memory for Coding Agents](https://mcpmarket.com/server/agents-remember-1) - Provides durable, git-verified repository memory for coding agents to ensure project-specific rules ...

3. [2 years building agent memory systems, ended up just using Git](https://www.reddit.com/r/AI_Agents/comments/1mw4jvp/2_years_building_agent_memory_systems_ended_up/) - I've seen many teams overcomplicate persistent memory for agents. Using Git for versioning conversat...

4. [Introducing Memoria: The World's First Git for AI Agent Memory](https://dev.to/origin_matrix_b790e656217/introducing-memoria-the-worlds-first-git-for-ai-agent-memory-4108) - The openclaw-memoria plugin replaces OpenClaw's default file-based memory with the full Memoria back...

5. [How should security teams protect vector databases that contain sensitive AI data?](https://nhimg.org/faq/how-should-security-teams-protect-vector-databases-that-contain-sensitive-ai-dat/) - Security teams should treat vector databases like production data stores, not lightweight AI infrast...

6. [General usage](https://trufflesecurity.com/docs/scanning-in-ci) - TruffleHog Docs

7. [pgvector vs Chroma vs Qdrant for Local RAG 2026 - AIFoss](https://aifoss.dev/blog/pgvector-vs-chroma-vs-qdrant-local-rag-2026/) - Should your local RAG app use pgvector, Chroma, or Qdrant? 2026 comparison: setup cost, benchmarks, ...

8. [Reference: Chroma vector store | Vectors | Mastra Docs](https://mastra.ai/reference/vectors/chroma) - Documentation for the ChromaVector class in Mastra, which provides vector search using ChromaDB.

9. [Faiss indexes · facebookresearch/faiss Wiki](https://github.com/facebookresearch/faiss/wiki/Faiss-indexes) - In addition to the restrictions of the Flat index HNSW uses, HNSW does not support removing vectors ...

10. [LanceDB Embedded Vector Store (No Server) | OSS AI Hub Code Starters](https://ossaihub.com/code/lancedb-embedded-vector-store/) - LanceDB stores vectors in a columnar Parquet/Lance format on disk — no server, no Docker. Great for ...

11. [Embedded OSS - LanceDB](https://www.lancedb.com/lp/embedded-oss)

12. [Qdrant---Vector-Database-/README.md at master · Decentralised-AI/Qdrant---Vector-Database-](https://github.com/Decentralised-AI/Qdrant---Vector-Database-/blob/master/README.md) - Qdrant - Vector Database for the next generation of AI applications. Also available in the cloud htt...

13. [Qdrant Cloud pricing (2026): plans, free tier & compliance · The Front Desk Review](https://frontdeskreview.com/software/vector-databases/qdrant-cloud/) - Qdrant Cloud vector databases pricing: from $0/mo, 4 plans, HIPAA Yes, free tier Yes. Every figure l...

14. [Weaviate Database | Weaviate Documentation](https://docs.weaviate.io/weaviate) - Complete documentation for Weaviate, the open-source vector database for AI applications.

15. [Weaviate Pricing (2026): Cloud and Self-Hosted Tiers](https://www.modern-datatools.com/tools/weaviate/pricing) - Weaviate pricing breakdown for 2026: cloud-managed and self-hosted vector database plans, hidden cos...

16. [Exploring vector-based search in Databases for PostgreSQL using ...](https://cloud.ibm.com/docs/databases-for-postgresql?topic=databases-for-postgresql-pgvector) - Find documentation, API & SDK references, tutorials, FAQs, and more resources for IBM Cloud products...

17. [Vector Databases for AI Agents: 8 DBs Compared. - Digital Applied](https://www.digitalapplied.com/blog/vector-databases-for-ai-agents-pinecone-qdrant-2026) - Pinecone, Qdrant, Weaviate, Milvus, Chroma, pgvector, Vertex Vector, Vespa compared on latency, cost...

18. [Chroma Docs: Introduction](https://docs.trychroma.com/docs/overview/introduction) - Chroma gives you everything you need for retrieval: store embeddings with metadata, search with dens...

19. [Chroma — LangChain documentation](https://reference.langchain.com/v0.3/python/community/vectorstores/langchain_community.vectorstores.chroma.Chroma.html)

20. [Working with FAISS Vector DB - Medium](https://medium.com/@nchshanmukha999/working-with-faiss-vector-db-86f7154bb937) - Exploring vector storage is pivotal in RAG frameworks, with FAISS emerging as a beginner-friendly so...

21. [Guidelines to choose an index · facebookresearch/faiss Wiki - GitHub](https://github.com/facebookresearch/faiss/wiki/Guidelines-to-choose-an-index) - A library for efficient similarity search and clustering of dense vectors. - facebookresearch/faiss

22. [The Faiss Library](https://arxiv.org/html/2401.08281v4) - The basic structure of Faiss is an index that can have multiple implementations described in this pa...

23. [Versioning & Reproducibility in LanceDB](https://lancedb.com/docs/concepts/tables/versioning/) - Learn how to implement versioning and ensure reproducibility in LanceDB. Includes version control, d...

24. [Lance format - LanceDB](https://docs.lancedb.com/lance)

25. [Qdrant Edge | Embedded and Edge AI Systems](https://qdrant.tech/edge/) - Qdrant Edge is a lightweight, in-process vector search engine designed for embedded devices, autonom...

26. [Qdrant Edge: Vector Search for Embedded AI](https://qdrant.tech/blog/qdrant-edge/) - Private Beta launching for Qdrant Edge - lightweight, embedded vector search engine designed to run ...

27. [Qdrant Edge](https://qdrant.tech/documentation/edge/) - Qdrant Edge is a lightweight, embedded vector search engine for in-process retrieval with a minimal ...

