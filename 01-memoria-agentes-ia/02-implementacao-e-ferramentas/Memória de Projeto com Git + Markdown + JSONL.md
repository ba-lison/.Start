# Memória de Projeto com Git + Markdown + JSONL

> Sistema robusto, auditável e adotável em qualquer repositório — sem banco vetorial, sem dependências externas.

***

## Visão Geral

A abordagem file-based de memória de agente tem se consolidado como o padrão pragmático para projetos solo ou de pequenas equipes. O princípio central é simples: arquivos Markdown são legíveis por humanos e por LLMs, diff-friendly no Git, e portáteis entre qualquer agente ou editor. O sistema aqui proposto adiciona JSONL para log auditável append-only e scripts locais para automação — sem frameworks, sem lock-in.[^1][^2][^3][^4]

A estrutura `.memory/` (variante da `.memory-bank/` documentada em projetos como o `agent-markdown-memory-bank-protocol`) funciona como a fonte única da verdade do projeto, consumida pelo agente antes de qualquer tarefa e atualizada após cada sessão.[^3][^4]

***

## Estrutura `.memory/`

```
seu-projeto/
├── .memory/
│   ├── README.md              # Como usar este sistema (para humanos e agentes)
│   ├── project.md             # Contexto permanente do projeto
│   ├── active.md              # Foco atual + próximos passos
│   ├── decisions.md           # ADRs — Architecture Decision Records
│   ├── patterns.md            # Padrões de código, convenções, lições aprendidas
│   ├── progress.jsonl         # Log append-only de sessões (auditável)
│   ├── archive/               # Memórias antigas arquivadas
│   │   └── 2026-Q1.md
│   └── inbox.md               # Propostas do agente aguardando revisão humana
├── AGENTS.md                  # Instrução raiz lida automaticamente por agentes
└── .gitignore
```

**Regras de `.gitignore`:**

```gitignore
# Nunca commitar secrets ou índices vetoriais
.memory/*.key
.memory/*.env
.memory/vectordb/
# Session transcripts grandes ficam locais
.memory/sessions/
```

***

## Arquivos Essenciais — Templates Completos

### `AGENTS.md` (raiz do repositório)

Este arquivo é lido automaticamente por Copilot (`AGENTS.md`), Cursor (`.cursorrules`), Gemini CLI e outros agentes. É a "constituição" do agente.[^4][^1]

```markdown
# Instruções para Agentes de IA

## Protocolo de Memória

Antes de qualquer tarefa, leia obrigatoriamente:
1. `.memory/project.md` — contexto e objetivo do projeto
2. `.memory/active.md` — foco atual e próximos passos
3. `.memory/decisions.md` — decisões arquiteturais já tomadas
4. `.memory/patterns.md` — padrões e convenções do projeto

Após cada tarefa:
1. Appende uma entrada em `.memory/progress.jsonl`
2. Atualiza `.memory/active.md` com os próximos passos
3. Se houver novas decisões ou padrões, proponha adições em `.memory/inbox.md`
4. Confirme ao usuário que a memória foi atualizada

## Regras de Comportamento
- Nunca reescreva o histórico de `progress.jsonl` — apenas appende
- Mantenha cada arquivo de memória abaixo de 200 linhas
- Prefira atualizar entradas existentes a criar duplicatas
- Marque itens desatualizados com `<!-- STALE: YYYY-MM-DD -->` antes de propor remoção
- Não armazene secrets, API keys ou dados pessoais nos arquivos de memória
```

***

### `.memory/project.md`

```markdown
---
updated: 2026-07-02
version: 1
---

# Contexto do Projeto

## Objetivo
[Uma frase descrevendo o que este projeto faz e para quem.]

## Stack Principal
- Linguagens: 
- Frameworks:
- Infraestrutura:

## Restrições Importantes
- [Ex: "Não usar banco vetorial na fase atual"]
- [Ex: "Compatibilidade com Node.js 18+"]

## Escopo Fora dos Limites
- [Ex: "Não há frontend nesta fase"]

## Contexto de Negócio
[Por que este projeto existe? Qual problema resolve?]
```

***

### `.memory/active.md`

```markdown
---
updated: 2026-07-02
---

# Contexto Ativo

## Foco Atual
[O que está sendo trabalhado agora.]

## Decisões Pendentes
- [ ] [Decisão que precisa ser tomada]

## Próximos Passos (não ordenados)
- [ ] [Tarefa A]
- [ ] [Tarefa B]
- [ ] [Tarefa C]

## Bloqueios
- [Qualquer impedimento atual]

## Última Sessão
- Data: YYYY-MM-DD
- O que foi feito: [resumo de 1-2 linhas]
```

***

### `.memory/decisions.md`

```markdown
# Decisões de Arquitetura (ADRs)

## ADR-001 — [Título da Decisão]
- **Data:** YYYY-MM-DD
- **Status:** Aceito | Supersedido por ADR-XXX | Em revisão
- **Contexto:** [Por que esta decisão foi necessária]
- **Decisão:** [O que foi decidido]
- **Consequências:** [Impactos positivos e negativos]
- **Alternativas rejeitadas:** [O que foi considerado e por quê foi descartado]

---

## ADR-002 — [Título]
...
```

***

### `.memory/patterns.md`

```markdown
# Padrões, Convenções e Lições Aprendidas

## Padrões de Código
- [Ex: "Funções assíncronas sempre usam async/await, nunca .then()"]
- [Ex: "Arquivos de configuração ficam em /config, não na raiz"]

## Convenções de Nomenclatura
- [Ex: "Variáveis de ambiente: SCREAMING_SNAKE_CASE"]

## Lições Aprendidas
- [Ex: "A API X retorna 429 após 100 req/min — implementar rate limit"]
- [Ex: "O build falha no CI se package-lock.json não for commitado"]

## Anti-padrões (Evitar)
- [Ex: "Não usar require() dinâmico — quebra o tree-shaking"]
```

***

### `.memory/progress.jsonl`

Arquivo append-only. **Nunca editar linhas existentes** — apenas adicionar novas.[^5][^6]

```jsonl
{"ts":"2026-07-01T14:30:00Z","session":"001","type":"task_done","summary":"Configurado pipeline CI com GitHub Actions","tags":["ci","infra"],"files_changed":["/.github/workflows/ci.yml"],"agent":"claude-4","human_reviewed":false}
{"ts":"2026-07-01T16:00:00Z","session":"001","type":"decision","summary":"Escolhido JSONL em vez de SQLite para logs — auditabilidade e diff limpo","tags":["arquitetura","storage"],"adr_ref":"ADR-003","human_reviewed":true}
{"ts":"2026-07-02T09:15:00Z","session":"002","type":"pattern_learned","summary":"Endpoint /health deve retornar {status,uptime,version} — padrão adotado","tags":["api","padrão"],"human_reviewed":false}
```

**Schema de um registro:**

| Campo | Tipo | Descrição |
|---|---|---|
| `ts` | ISO 8601 | Timestamp UTC |
| `session` | string | ID da sessão de trabalho |
| `type` | enum | `task_done`, `decision`, `pattern_learned`, `blocker`, `archive` |
| `summary` | string | Resumo ≤ 120 caracteres |
| `tags` | array | Categorias para grep/filtragem |
| `files_changed` | array | Arquivos modificados (opcional) |
| `agent` | string | Qual LLM executou (rastreabilidade) |
| `human_reviewed` | bool | Se o humano revisou e aprovou |

***

### `.memory/inbox.md`

```markdown
# Inbox — Propostas do Agente Aguardando Revisão

> Agente: adicione suas propostas aqui. Humano: revise, aprove (→ mova para o arquivo correto) ou rejeite (→ delete com comentário).

---

## [PROPOSTA] YYYY-MM-DD — Título da Proposta

**Tipo:** Novo padrão | Nova decisão | Atualização de contexto  
**Proposta pelo agente:** claude-4 às 2026-07-02T10:00Z  
**Conteúdo:**
> [Texto exato a ser adicionado no arquivo de memória]

**Arquivo de destino:** `.memory/patterns.md`  
**Justificativa:** [Por que isso deve ser memorizado]  
**Status:** 🟡 Pendente | ✅ Aprovado | ❌ Rejeitado

---
```

***

## Scripts Locais

### `memory-update` — Comando manual para atualizar memória

Salve como `.memory/scripts/memory-update.sh` e adicione ao `PATH` ou use via `npm scripts`.

```bash
#!/usr/bin/env bash
# Uso: ./memory-update.sh "Resumo do que foi feito" [tag1,tag2]
# Ex:  ./memory-update.sh "Implementado endpoint /health" "api,backend"

set -euo pipefail

MEMORY_DIR="$(git rev-parse --show-toplevel)/.memory"
JSONL="$MEMORY_DIR/progress.jsonl"
SUMMARY="${1:?'Informe um resumo: ./memory-update.sh \"Resumo\" [tags]'}"
TAGS="${2:-geral}"
SESSION=$(date +%Y%m%d-%H%M)
TS=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# Converte tags separadas por vírgula em array JSON
TAGS_JSON=$(echo "$TAGS" | python3 -c "import sys,json; print(json.dumps(sys.stdin.read().strip().split(',')))")

ENTRY=$(python3 -c "
import json, sys
entry = {
    'ts': '$TS',
    'session': '$SESSION',
    'type': 'task_done',
    'summary': sys.argv[^1],
    'tags': $TAGS_JSON,
    'agent': 'human',
    'human_reviewed': True
}
print(json.dumps(entry, ensure_ascii=False))
" "$SUMMARY")

echo "$ENTRY" >> "$JSONL"

# Atualiza campo "Última Sessão" em active.md
python3 - "$SUMMARY" "$SESSION" "$TS" <<'PYEOF'
import sys, re
summary, session, ts = sys.argv[^1], sys.argv[^2], sys.argv[^3]
path = sys.argv.replace('progress.jsonl', 'active.md') if False else \
       __import__('os').path.join(__import__('os').environ.get('MEMORY_DIR',''), 'active.md')

import os
path = os.path.join(os.environ.get('MEMORY_DIR', '.memory'), 'active.md')
with open(path, 'r') as f:
    content = f.read()

new_section = f"- Data: {ts[:10]}\n- O que foi feito: {summary}"
content = re.sub(
    r'## Última Sessão\n.*?(?=\n##|\Z)',
    f'## Última Sessão\n{new_section}\n',
    content, flags=re.DOTALL
)
with open(path, 'w') as f:
    f.write(content)
PYEOF

echo "✅ Memória atualizada: $SUMMARY"
echo "   Entrada: $ENTRY"
```

***

### `memory-check` — Verificações de saúde da memória

```bash
#!/usr/bin/env bash
# Verifica tamanho dos arquivos e alertas de manutenção

MEMORY_DIR="$(git rev-parse --show-toplevel)/.memory"

echo "=== 📋 Status da Memória ==="

for file in project.md active.md decisions.md patterns.md inbox.md; do
    lines=$(wc -l < "$MEMORY_DIR/$file" 2>/dev/null || echo 0)
    status="✅"
    [ "$lines" -gt 180 ] && status="⚠️  GRANDE ($lines linhas)"
    [ "$lines" -gt 250 ] && status="🔴 ARQUIVAR ($lines linhas)"
    printf "  %-20s %s\n" "$file" "$status"
done

echo ""
echo "=== 📊 Últimas 5 entradas do log ==="
tail -5 "$MEMORY_DIR/progress.jsonl" | python3 -c "
import sys, json
for line in sys.stdin:
    e = json.loads(line)
    reviewed = '✅' if e.get('human_reviewed') else '🟡'
    print(f'  {reviewed} [{e[\"ts\"][:10]}] {e[\"summary\"][:70]}')
"

echo ""
pending=$(grep -c "🟡 Pendente" "$MEMORY_DIR/inbox.md" 2>/dev/null || echo 0)
[ "$pending" -gt 0 ] && echo "⚠️  $pending proposta(s) pendente(s) em inbox.md"
```

***

## Prompts para o Agente

### Prompt de Consulta (antes de trabalhar)

Inclua este bloco no início do prompt do sistema ou como primeira mensagem da sessão:[^3][^4]

```
## Protocolo de Memória — LEITURA OBRIGATÓRIA

Antes de começar qualquer trabalho neste repositório, execute os seguintes passos:

1. Leia `.memory/project.md` para entender o objetivo, stack e restrições do projeto.
2. Leia `.memory/active.md` para saber o foco atual e próximos passos.
3. Leia `.memory/decisions.md` para conhecer decisões arquiteturais já tomadas.
4. Leia `.memory/patterns.md` para aplicar convenções e evitar anti-padrões conhecidos.
5. Verifique se há propostas pendentes em `.memory/inbox.md`.

Após a leitura, confirme com uma linha:
> "Memória carregada: [resumo em 1 linha do contexto atual]"

Só então inicie a tarefa solicitada.
```

***

### Prompt de Atualização (após a tarefa)

```
## Protocolo de Memória — ATUALIZAÇÃO OBRIGATÓRIA

Ao concluir esta tarefa, execute os passos abaixo ANTES de encerrar a sessão:

1. **Appende** em `.memory/progress.jsonl` uma nova linha JSON com:
   - `ts`: timestamp UTC atual (use o comando `date -u +"%Y-%m-%dT%H:%M:%SZ"`)
   - `type`: "task_done" | "decision" | "pattern_learned" | "blocker"
   - `summary`: ≤ 120 caracteres descrevendo o que foi feito
   - `tags`: array de categorias relevantes
   - `agent`: seu identificador (ex: "claude-4", "gpt-4o")
   - `human_reviewed`: false (o humano revisará depois)

2. **Atualize** `.memory/active.md`:
   - Marque como concluídos os itens da seção "Próximos Passos" que foram resolvidos
   - Adicione novos próximos passos se identificados
   - Atualize "Última Sessão"

3. Se você identificou novos padrões, decisões ou aprendizados relevantes:
   - **NÃO edite diretamente** `decisions.md` ou `patterns.md`
   - **PROPONHA** a adição em `.memory/inbox.md` usando o template padrão
   - O humano revisará e aprovará (ou rejeitará) as propostas

4. Confirme ao usuário: "Memória atualizada ✅ — [resumo em 1 linha]"
```

***

## Estratégias de Gestão

### Revisão Humana

A revisão humana é o ponto de controle mais importante do sistema. O fluxo recomendado é:[^7]

1. **Inbox como filtro**: o agente só propõe, nunca escreve diretamente em `decisions.md` ou `patterns.md`
2. **Revisão semanal rápida**: 5-10 minutos verificando `inbox.md` e as últimas entradas de `progress.jsonl`
3. **`human_reviewed: true`** sinaliza no JSONL que a entrada foi validada por um humano
4. **Git diff como ferramenta de revisão**: `git diff HEAD~1 .memory/` mostra exatamente o que o agente mudou

```bash
# Alias útil para revisão rápida
alias memory-review="git diff HEAD~5 .memory/ | less"
alias memory-inbox="cat .memory/inbox.md"
```

***

### Manter Memórias Curtas

O problema central de sistemas file-based é o crescimento ilimitado dos arquivos — um `MEMORY.md` com 10.000 linhas anula os benefícios ao lotar o context window. A estratégia:[^8][^1]

- **Limite de 200 linhas por arquivo** de memória (exceto `progress.jsonl`)
- Ao atingir 180 linhas, o script `memory-check` alerta para arquivar[^9]
- Prefira **substituir** entradas desatualizadas a acumular: marque com `<!-- STALE: YYYY-MM-DD -->` e remova na próxima revisão
- `progress.jsonl` pode crescer indefinidamente — ele é para auditoria, não para contexto do LLM[^6][^5]
- O agente **nunca lê o JSONL inteiro** — apenas os arquivos `.md` compactos são injetados no contexto

***

### Evitar Duplicidade

Antes de propor uma nova entrada, o agente deve:[^7][^8]

1. Buscar em `decisions.md` e `patterns.md` por entradas com keywords semelhantes
2. Verificar `inbox.md` por propostas idênticas ainda pendentes
3. Se encontrar conteúdo similar: propor **atualização da entrada existente**, não criação de nova
4. Incluir na proposta do `inbox.md` o campo `"conflita_com"` referenciando a entrada existente

```bash
# Script de busca por duplicatas
memory-search() {
    grep -rn "$1" "$(git rev-parse --show-toplevel)/.memory/" \
        --include="*.md" \
        --include="*.jsonl" \
        | grep -v "archive/"
}
```

***

### Arquivar Memórias Antigas

Execute trimestralmente ou quando um arquivo ultrapassar 200 linhas:[^1][^9]

```bash
#!/usr/bin/env bash
# memory-archive.sh — Arquiva entradas antigas de decisions.md ou patterns.md
# Uso: ./memory-archive.sh decisions 2026-Q1

MEMORY_DIR="$(git rev-parse --show-toplevel)/.memory"
FILE="$MEMORY_DIR/${1}.md"
ARCHIVE_NAME="${2:-$(date +%Y-Q%q)}"
ARCHIVE_FILE="$MEMORY_DIR/archive/$ARCHIVE_NAME-${1}.md"

mkdir -p "$MEMORY_DIR/archive"

# Move seções com status "Supersedido" ou "STALE" para o arquivo
python3 - "$FILE" "$ARCHIVE_FILE" <<'PYEOF'
import sys, re
src, dst = sys.argv[^1], sys.argv[^2]

with open(src) as f:
    content = f.read()

# Separa seções por ## ADR- ou ## [bloco]
sections = re.split(r'(?=\n## )', content)
active, archived = [], []

for s in sections:
    if 'Supersedido' in s or 'STALE' in s or 'Arquivado' in s:
        archived.append(s)
    else:
        active.append(s)

with open(src, 'w') as f:
    f.write(''.join(active))

# Appende ao arquivo de archive
with open(dst, 'a') as f:
    f.write(f'\n\n<!-- Arquivado em {__import__("datetime").date.today()} -->\n')
    f.write(''.join(archived))

print(f"✅ {len(archived)} seção(ões) arquivada(s) em {dst}")
PYEOF
```

Para o `progress.jsonl`, rotacione arquivos antigos por ano:

```bash
# Arquivar logs do ano anterior
memory-rotate-log() {
    YEAR="${1:-$(date +%Y -d 'last year')}"
    MEMORY_DIR="$(git rev-parse --show-toplevel)/.memory"
    grep "\"ts\":\"$YEAR" "$MEMORY_DIR/progress.jsonl" \
        > "$MEMORY_DIR/archive/$YEAR-progress.jsonl"
    # Remove linhas do ano anterior do arquivo principal
    sed -i "/\"ts\":\"$YEAR/d" "$MEMORY_DIR/progress.jsonl"
    echo "✅ Log de $YEAR arquivado"
}
```

***

## Integração com Git

### Integração com Commits (opcional)

Adicione um `commit-msg` hook que referencia o estado da memória automaticamente:[^2][^10]

```bash
# .git/hooks/post-commit
#!/usr/bin/env bash

MEMORY_DIR="$(git rev-parse --show-toplevel)/.memory"
JSONL="$MEMORY_DIR/progress.jsonl"
COMMIT_MSG=$(git log -1 --pretty=%B)
COMMIT_HASH=$(git log -1 --pretty=%H | head -c 8)
TS=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
SESSION=$(date +%Y%m%d-%H%M)

# Registra o commit no log de memória
ENTRY=$(python3 -c "
import json
entry = {
    'ts': '$TS',
    'session': '$SESSION',
    'type': 'git_commit',
    'summary': 'commit $COMMIT_HASH: ${COMMIT_MSG:0:80}',
    'tags': ['git'],
    'commit': '$COMMIT_HASH',
    'agent': 'git-hook',
    'human_reviewed': True
}
print(json.dumps(entry, ensure_ascii=False))
")

echo "$ENTRY" >> "$JSONL"
```

Para ativar: `chmod +x .git/hooks/post-commit`

Alternativamente, use `prepare-commit-msg` para **injetar contexto de memória na mensagem do commit**:

```bash
# .git/hooks/prepare-commit-msg
#!/usr/bin/env bash
COMMIT_MSG_FILE=$1
MEMORY_DIR="$(git rev-parse --show-toplevel)/.memory"

# Adiciona contexto da memória como comentário
ACTIVE_FOCUS=$(head -20 "$MEMORY_DIR/active.md" | grep -A2 "## Foco Atual" | tail -1)
echo "" >> "$COMMIT_MSG_FILE"
echo "# Contexto de memória: $ACTIVE_FOCUS" >> "$COMMIT_MSG_FILE"
```

***

### Integração com Pull Requests (opcional)

Para projetos com PRs, adicione um arquivo `.github/pull_request_template.md` que pede a atualização de memória como checklist:[^11]

```markdown
## Descrição
[Descreva o que foi feito]

## Checklist de Memória
- [ ] `progress.jsonl` atualizado com entradas desta PR
- [ ] `active.md` atualizado com próximos passos
- [ ] Novas decisões propostas em `inbox.md` (se aplicável)
- [ ] Novas convenções propostas em `inbox.md` (se aplicável)
- [ ] Arquivos de memória ainda dentro do limite de 200 linhas

## Entradas adicionadas ao JSONL
```jsonl
[cole aqui as novas linhas adicionadas ao progress.jsonl]
```
```

Para automação via GitHub Actions, valide o tamanho dos arquivos de memória a cada PR:

```yaml
# .github/workflows/memory-check.yml
name: Memory Health Check
on: [pull_request]

jobs:
  check-memory:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check memory file sizes
        run: |
          for file in .memory/project.md .memory/active.md .memory/decisions.md .memory/patterns.md; do
            lines=$(wc -l < "$file")
            if [ "$lines" -gt 200 ]; then
              echo "::warning file=$file::Arquivo de memória com $lines linhas (limite: 200). Considere arquivar."
            fi
          done
      - name: Validate JSONL
        run: |
          python3 -c "
          import json, sys
          errors = []
          with open('.memory/progress.jsonl') as f:
              for i, line in enumerate(f, 1):
                  try:
                      e = json.loads(line)
                      required = ['ts','session','type','summary','agent']
                      missing = [k for k in required if k not in e]
                      if missing:
                          errors.append(f'Linha {i}: campos ausentes {missing}')
                  except json.JSONDecodeError as ex:
                      errors.append(f'Linha {i}: JSON inválido — {ex}')
          if errors:
              for e in errors: print(e)
              sys.exit(1)
          print(f'✅ JSONL válido')
          "
```

***

## Exemplos Práticos

### Exemplo 1 — Início de projeto (preenchimento inicial)

```bash
# 1. Cria a estrutura
mkdir -p .memory/archive .memory/scripts

# 2. Inicializa os arquivos a partir dos templates
# (copie os templates das seções acima)

# 3. Primeira entrada no log
echo '{"ts":"2026-07-02T12:00:00Z","session":"bootstrap","type":"task_done","summary":"Estrutura .memory/ inicializada no projeto","tags":["setup","memory"],"agent":"human","human_reviewed":true}' >> .memory/progress.jsonl

# 4. Commita a estrutura de memória
git add .memory/ AGENTS.md
git commit -m "feat: adiciona sistema de memória .memory/ ao projeto"
```

***

### Exemplo 2 — Sessão típica com agente

**Prompt enviado ao agente:**

```
Leia os arquivos em .memory/ conforme o protocolo definido em AGENTS.md.
Depois, implemente a autenticação JWT no endpoint /api/auth.
```

**Agente responde após leitura:**

```
Memória carregada: API REST Node.js, decisão ADR-002 usa JWT RS256, 
padrão de async/await obrigatório, próximos passos incluem autenticação.

[...implementação...]

Memória atualizada ✅ — Implementado middleware JWT RS256 em /api/auth/middleware.js
```

**Entrada gerada no `progress.jsonl`:**

```jsonl
{"ts":"2026-07-02T15:30:00Z","session":"20260702-1530","type":"task_done","summary":"Implementado middleware JWT RS256 em /api/auth/middleware.js","tags":["auth","api","security"],"files_changed":["/api/auth/middleware.js","/api/auth/routes.js"],"agent":"claude-4","human_reviewed":false}
```

**Proposta gerada no `inbox.md`:**

```markdown
## [PROPOSTA] 2026-07-02 — Padrão de validação JWT

**Tipo:** Novo padrão  
**Proposta pelo agente:** claude-4 às 2026-07-02T15:30Z  
**Conteúdo:**
> Sempre validar JWT com biblioteca `jose` (não `jsonwebtoken`) — compatível com Edge Runtime e suporte nativo a RS256/ES256.

**Arquivo de destino:** `.memory/patterns.md`  
**Justificativa:** Padrão estabelecido durante implementação do endpoint /api/auth  
**Status:** 🟡 Pendente
```

***

### Exemplo 3 — Consulta de histórico via linha de comando

```bash
# Ver todas as decisões dos últimos 30 dias
python3 -c "
import json
from datetime import datetime, timedelta
cutoff = (datetime.utcnow() - timedelta(days=30)).isoformat()
with open('.memory/progress.jsonl') as f:
    for line in f:
        e = json.loads(line)
        if e['ts'] > cutoff and e['type'] == 'decision':
            print(f\"[{e['ts'][:10]}] {e['summary']}\")
"

# Listar entradas não revisadas por humano
grep '"human_reviewed": false' .memory/progress.jsonl | \
    python3 -c "import sys,json; [print(f\"🟡 {json.loads(l)['summary']}\") for l in sys.stdin]"

# Buscar por tag específica
grep '"auth"' .memory/progress.jsonl | python3 -m json.tool
```

***

## Comparativo de Abordagens

| Critério | `.memory/` (este sistema) | Vector DB | Framework (LangChain) |
|---|---|---|---|
| Dependências | Nenhuma | Alto (embeddings, DB) | Alto |
| Auditabilidade | ✅ Git diff nativo | ❌ Black box | ❌ Parcial |
| Portabilidade | ✅ Qualquer agente | ❌ Vendor lock-in | ❌ Lock-in |
| Escala | ✅ Projetos solo/small team | ✅ 10k+ memórias | ✅ Produção |
| Legibilidade humana | ✅ Total | ❌ Nenhuma | ❌ Parcial |
| Setup inicial | < 10 min | Dias | Horas-dias |
| Revisão humana | ✅ Git review | ❌ Impossível | ❌ Difícil |
| Busca semântica | ❌ Apenas `grep` | ✅ Nativa | ✅ Nativa |

A principal limitação desta abordagem é a ausência de busca semântica — resolvida, quando necessário, adicionando uma camada vetorial opcional acima dos mesmos arquivos `.md`.[^8][^4]

***

## Resumo do Fluxo Completo

```
┌─────────────────────────────────────────────────────────┐
│                    FLUXO DE SESSÃO                      │
│                                                         │
│  1. Agente lê .memory/*.md ──→ Contexto carregado      │
│  2. Agente executa tarefa                               │
│  3. Agente appende em progress.jsonl                   │
│  4. Agente atualiza active.md                          │
│  5. Agente propõe em inbox.md (se houver aprendizado)  │
│                                                         │
│              ↓ REVISÃO HUMANA (semanal)                 │
│                                                         │
│  6. Humano revisa inbox.md                             │
│  7. Aprova → move para decisions.md / patterns.md      │
│  8. Rejeita → deleta com comentário                    │
│  9. git commit .memory/ (rastreabilidade total)        │
│ 10. Trimestral: memory-archive.sh + memory-rotate-log  │
└─────────────────────────────────────────────────────────┘
```

---

## References

1. [3 Markdown Files Fix AI Agent Memory Forever](https://www.youtube.com/watch?v=1PRgb-0s9V4) - A dev fixed AI agent amnesia with 3 plain text files. No database, no framework — just Markdown. Her...

2. [2 years building agent memory systems, ended up just ...](https://www.reddit.com/r/AI_Agents/comments/1mw4jvp/2_years_building_agent_memory_systems_ended_up/) - Using Git for versioning conversational memories is elegant it gives you transparency, audit trails,...

3. [rtoma/agent-markdown-memory-bank-protocol](https://github.com/rtoma/agent-markdown-memory-bank-protocol) - Lightweight protocol for AI coding agents to maintain persistent project context across sessions, us...

4. [Agent Memory with Markdown: A Simple, Reliable Solution - LinkedIn](https://www.linkedin.com/posts/joseph-pearson-296bb5122_ai-agenticai-productivity-activity-7434251558105661440-QNBU) - Your AI agent forgets everything when you close the chat. Every session starts from zero. Like worki...

5. [JSONL as the Native Observability Format for AI Agent Runtimes | Zylos Research](https://zylos.ai/research/2026-06-06-jsonl-agent-observability-data-format/) - Why JSONL's append-only, line-delimited structure maps naturally onto agent execution, and how to de...

6. [Replacing SQLite with a JSONL Journal for AI Agent Observability](https://eka.weiyen.net/posts/unified-jsonl-journal-for-ai-agents/) - Why I replaced a SQLite database with an append-only JSONL file for logging all agent activity — mes...

7. [Building a Local Markdown Memory Layer for AI Agents](https://dev.to/gowtham09/building-a-local-markdown-memory-layer-for-ai-agents-544) - Is this memory reviewed? · Is it project-specific? · Does it conflict with another memory? · Should ...

8. [Markdown is not agent memory](https://blog.getzep.com/markdown-is-not-agent-memory/) - Some of the most capable agents in production keep their memory in plain markdown files, and for a s...

9. [How to Build an Observation Log That Makes AI Agents Self-Improving](https://agent.ceo/blog/observation-log-self-improving-ai-agents) - A practical tutorial on designing a structured observation log that records significant agent action...

10. [Shared Memory for AI Coding Agents: A Git-Backed Approach](https://www.linkedin.com/pulse/shared-memory-ai-coding-agents-git-backed-approach-georgios-dxi9e) - Your AI agents forget everything. Everyone's trying to fix that.

11. [Alfrada Manages GitHub Repos with Autonomy | Strategize Labs posted on the topic | LinkedIn](https://www.linkedin.com/posts/strategize-inc_alfrada-ai-devops-activity-7445436783489638400-6pTU) - #Alfrada Can Autonomously Manage Your Repos. No kidding. Everyone is hyped about AI writing code. Bu...

