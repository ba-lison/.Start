# Especificação Completa: Comando `memory-curator`

> Especificação técnica para um agente de curadoria de memória em projetos de software.  
> Versão: 1.0 | Data: julho/2026

***

## 1. Visão Geral

O comando `memory-curator` é um subcomando executado por um agente de IA ao final (ou durante checkpoints) de uma sessão de trabalho em projeto de software. Seu objetivo é transformar informação efêmera — conversas, diffs, commits, resultados de testes e arquivos alterados — em **memória durável, curada e rastreável**, armazenada no diretório `.memory/` do repositório.

A distinção entre **working context** (contexto de trabalho imediato, descartável) e **memória durável** é a decisão arquitetural central do sistema. Working memory é o que o agente tem ativo no contexto atual; long-term memory é o subconjunto curado que persiste entre sessões. O `memory-curator` opera nessa fronteira.[^1][^2][^3][^4]

***

## 2. Arquitetura do Fluxo

```
Input Sources
┌─────────────────────────────────────────────┐
│  conversation_transcript  │  git diff        │
│  commit messages          │  test results    │
│  changed files            │  task state      │
└─────────────────────────────────────────────┘
                    │
                    ▼
         ┌──────────────────┐
         │  CURATOR AGENT   │
         │  (análise + LLM) │
         └────────┬─────────┘
                  │
         ┌────────▼─────────┐
         │  Extração de     │  ← identifica fatos, separa working context
         │  Candidatos      │    de memória durável, classifica por tipo
         └────────┬─────────┘
                  │
         ┌────────▼─────────┐
         │  Validação &     │  ← remove duplicatas, detecta segredos,
         │  Filtragem       │    detecta suposições, verifica evidências
         └────────┬─────────┘
                  │
         ┌────────▼─────────┐
         │  Proposta de     │  ← JSON com memórias a criar/atualizar/revogar
         │  Atualização     │
         └────────┬─────────┘
                  │
         ┌────────▼─────────┐
         │  HUMAN APPROVAL  │  ← diff legível, aprovação explícita necessária
         └────────┬─────────┘
                  │
         ┌────────▼─────────┐
         │  Escrita em      │  ← .memory/ com frontmatter YAML
         │  .memory/        │
         └──────────────────┘
```

***

## 3. Categorias de Memória

O sistema utiliza 15 categorias mutuamente exclusivas, baseadas em padrões observados em sistemas de memória para coding agents e frameworks de externalização de conhecimento implícito:[^5][^6]

| Categoria | Descrição | Exemplos de Gatilho |
|-----------|-----------|---------------------|
| `bug_fix` | Correção de bug com causa-raiz e solução | Commit com `fix:`, teste antes falhava, agora passa |
| `architecture_decision` | Decisão estrutural com justificativa e alternativas rejeitadas | "Escolhemos PostgreSQL por X; Redis foi descartado por Y" |
| `implementation_pattern` | Padrão de código reutilizável comprovado | Função usada 3+ vezes, helper extraído, snippet aprovado |
| `failed_attempt` | Tentativa que não funcionou, com motivo | Rollback, revert, comentário "tentei X mas causou Y" |
| `fragile_area` | Código ou integração com histórico de problemas | 3+ edições no mesmo arquivo, teste flaky recorrente |
| `command` | Comando de terminal/CLI não-óbvio para a base de código | Comandos de build, deploy, debug específicos do projeto |
| `dependency_note` | Comportamento inesperado ou restrição de dependência | Breaking change, versão específica obrigatória |
| `test_pattern` | Estratégia ou estrutura de teste confirmada | Setup de mock, fixture reutilizável, padrão de assertion |
| `security_warning` | Risco de segurança identificado | Endpoint sem autenticação, dado sensível logado |
| `performance_note` | Observação sobre gargalos ou otimizações | Query N+1, cache necessário, limite de tamanho de payload |
| `migration_note` | Passo de migração crítico ou não-óbvio | Schema change, seed de dados, ordem de execução |
| `domain_rule` | Regra de negócio que impacta o código | "Desconto nunca pode ser 100%", "status só avança, nunca retrocede" |
| `integration_detail` | Detalhe de integração com serviço externo | Formato de webhook, header obrigatório, limite de rate |
| `deprecation` | API, função ou padrão que não deve mais ser usado | `@deprecated`, substituição identificada |
| `open_question` | Questão não resolvida que precisa de decisão futura | Dúvida de design, comportamento indefinido, tech debt |

***

## 4. Estrutura de Arquivos `.memory/`

Inspirada no Agent Memory Protocol (AMP) e no formato Open Knowledge Format do Google, a estrutura é markdown-first, git-friendly e legível sem ferramentas especiais:[^7][^8]

```
.memory/
├── index.md                        # Índice gerado, não editado manualmente
├── bug_fix/
│   └── 2026-07-02_null-ptr-login.md
├── architecture_decision/
│   └── 2026-06-15_auth-strategy.md
├── implementation_pattern/
│   └── 2026-05-10_retry-with-backoff.md
├── failed_attempt/
│   └── 2026-07-01_graphql-migration.md
├── fragile_area/
│   └── file_upload_handler.md
├── security_warning/
│   └── 2026-07-02_unvalidated-redirect.md
└── [demais categorias]/
    └── *.md
```

Cada arquivo de memória usa **YAML frontmatter** obrigatório:[^9]

```yaml
---
id: bug_fix_null-ptr-login_20260702
type: bug_fix
title: "NullPointerException no login quando email tem letra maiúscula"
confidence: 0.90           # 0.0-1.0
created_at: "2026-07-02T10:34:00-03:00"
last_updated: "2026-07-02T10:34:00-03:00"
source_session: "session_abc123"
source_files:
  - "src/auth/login_handler.js"
  - "tests/auth/login.test.js"
source_commits:
  - "a1b2c3d"
source_type: "diff+test_result"  # conversation|diff|commit|test_result|combined
evidence_level: high            # low|medium|high
verified_by_human: true
tags: ["auth", "email", "null-check"]
superseded_by: null             # id da memória que substitui esta
revoked: false
revoked_reason: null
---

## Contexto
O handler de login falhava com NPE quando o email enviado continha letras maiúsculas...

## Causa-Raiz
`user.email` não era normalizado antes da busca no banco.

## Solução Aplicada
```js
const normalizedEmail = email.toLowerCase().trim();
```

## Evidência
- Teste `login.test.js:45` passava com email minúsculo, falhava com maiúsculo
- Commit `a1b2c3d` corrigiu o problema
- 3 reportes de usuário no Sentry (#1234, #1235, #1240)

## Não Fazer
Não usar `.toLowerCase()` apenas na comparação — normalizar na entrada.
```

***

## 5. System Prompt do Curador

```
Você é o memory-curator, um agente especializado em curadoria de memória para projetos de software.

## Sua Missão
Transformar informações efêmeras de sessões de trabalho em memória durável, curada, rastreável e útil para futuras sessões. Você opera segundo o princípio de que memória de agente deve ser um sistema de controle de qualidade, não apenas acumulação de dados.

## Princípios Fundamentais

1. **Epistemologia rigorosa**: Só salve o que tem evidência suficiente. Suposições não verificadas NÃO são memória.
2. **Proveniência obrigatória**: Todo fato deve ter origem rastreável (arquivo, commit, teste, conversa).
3. **Confiança calibrada**: Atribua confidence scores (0.0–1.0) com base na qualidade da evidência.
4. **Segurança primeiro**: Detecte e bloqueie segredos, credenciais e PII antes de qualquer outra análise.
5. **Minimalismo**: É melhor não salvar do que salvar lixo. Contexto de trabalho efêmero não é memória.
6. **Human-in-the-loop**: Nunca escreva memória sem aprovação humana explícita.
7. **Não duplicar**: Verifique memórias existentes antes de propor novas. Prefira atualizar a criar.
8. **Contradições são dados**: Quando detectar contradição com memória existente, marque para resolução, não ignore.

## O Que NÃO É Memória
- Transcrição bruta de conversa
- Resultado de debug temporário
- Código que foi imediatamente revertido (a menos que o fracasso seja o aprendizado)
- Informação que já está no README, docstring ou código comentado
- Segredos, credenciais, tokens, chaves de API
- Dados pessoais identificáveis (PII)
- Suposições sem confirmação ("acho que...", "talvez...", "não tenho certeza")
- Contexto específico de apenas uma instância que não generaliza

## Processo de Análise
1. Leia e compreenda o estado atual da tarefa
2. Analise todas as fontes: conversa, diff, commits, testes, arquivos
3. Extraia candidatos a memória (fatos observados com evidência)
4. Separe working context (descarte) de memória durável (candidatos)
5. Classifique cada candidato em uma das 15 categorias
6. Para cada candidato:
   a. Verifique se já existe memória similar (busca semântica em .memory/)
   b. Se existir: decida entre atualizar, confirmar ou marcar conflito
   c. Se não existir: avalie se merece ser criada
7. Execute verificação de segurança em todo o conteúdo
8. Atribua confidence score e evidence_level
9. Gere a proposta de atualização em JSON
10. Apresente ao humano para aprovação

## Output Format
Sempre responda com o JSON schema definido na especificação.
Nunca escreva arquivos sem aprovação humana.
```

***

## 6. User Prompt (Template)

```
## Sessão de Curadoria de Memória

### Estado Atual da Tarefa
{{task_state}}

### Transcrição da Conversa (últimos N turnos relevantes)
{{conversation_excerpt}}

### Git Diff (staged + unstaged)
```diff
{{git_diff}}
```

### Commits da Sessão
{{commit_messages_and_hashes}}

### Resultados de Testes
{{test_results_summary}}

### Arquivos Alterados
{{changed_files_list}}

### Memórias Existentes Relevantes (pré-recuperadas por similaridade)
{{existing_memories_context}}

---

Por favor, execute o fluxo memory-curator completo:
1. Analise todas as fontes acima
2. Identifique candidatos a memória durável
3. Verifique segurança (segredos, PII)
4. Detecte suposições não verificadas
5. Compare com memórias existentes
6. Gere a proposta de atualização no JSON schema definido
7. Apresente o diff legível para aprovação humana
```

***

## 7. JSON Schema de Saída

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MemoryCuratorOutput",
  "type": "object",
  "required": [
    "session_id",
    "curator_version",
    "analysis_timestamp",
    "security_scan",
    "working_context_discarded",
    "memory_proposals",
    "contradictions_detected",
    "open_questions_flagged",
    "human_approval_required"
  ],
  "properties": {
    "session_id": {
      "type": "string",
      "description": "Identificador único da sessão de curadoria"
    },
    "curator_version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$"
    },
    "analysis_timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "security_scan": {
      "type": "object",
      "required": ["passed", "issues_found", "items_blocked"],
      "properties": {
        "passed": { "type": "boolean" },
        "issues_found": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "type": {
                "type": "string",
                "enum": ["api_key", "password", "token", "pii_name", "pii_email",
                         "pii_cpf", "connection_string", "private_key", "other"]
              },
              "location": { "type": "string" },
              "pattern_matched": { "type": "string" },
              "severity": { "type": "string", "enum": ["critical", "high", "medium"] },
              "redacted_preview": { "type": "string" }
            }
          }
        },
        "items_blocked": {
          "type": "integer",
          "description": "Número de candidatos bloqueados por conterem segredos"
        }
      }
    },
    "working_context_discarded": {
      "type": "array",
      "description": "Lista de informações descartadas por serem apenas working context",
      "items": {
        "type": "object",
        "properties": {
          "description": { "type": "string" },
          "reason_discarded": { "type": "string" }
        }
      }
    },
    "memory_proposals": {
      "type": "array",
      "items": {
        "type": "object",
        "required": [
          "proposal_id", "action", "memory_type", "title",
          "content", "confidence", "evidence_level",
          "evidence_sources", "source_session",
          "source_files", "source_commits"
        ],
        "properties": {
          "proposal_id": { "type": "string" },
          "action": {
            "type": "string",
            "enum": ["create", "update", "revoke", "confirm_existing"]
          },
          "target_memory_id": {
            "type": "string",
            "description": "Obrigatório para action=update|revoke|confirm_existing"
          },
          "memory_type": {
            "type": "string",
            "enum": [
              "bug_fix", "architecture_decision", "implementation_pattern",
              "failed_attempt", "fragile_area", "command",
              "dependency_note", "test_pattern", "security_warning",
              "performance_note", "migration_note", "domain_rule",
              "integration_detail", "deprecation", "open_question"
            ]
          },
          "title": {
            "type": "string",
            "maxLength": 120
          },
          "content": {
            "type": "string",
            "description": "Conteúdo markdown da memória, sem segredos"
          },
          "confidence": {
            "type": "number",
            "minimum": 0.0,
            "maximum": 1.0,
            "description": "Score de confiança na memória (0=especulação, 1=fato comprovado)"
          },
          "evidence_level": {
            "type": "string",
            "enum": ["low", "medium", "high"],
            "description": "low=observação; medium=padrão recorrente; high=comprovado por teste/prod"
          },
          "evidence_sources": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "type": {
                  "type": "string",
                  "enum": ["conversation", "diff", "commit", "test_result",
                           "error_log", "code_review", "user_report"]
                },
                "reference": { "type": "string" },
                "excerpt": { "type": "string", "maxLength": 500 }
              }
            },
            "minItems": 1
          },
          "source_session": { "type": "string" },
          "source_files": {
            "type": "array",
            "items": { "type": "string" }
          },
          "source_commits": {
            "type": "array",
            "items": { "type": "string" }
          },
          "tags": {
            "type": "array",
            "items": { "type": "string" }
          },
          "unverified_assumption_flags": {
            "type": "array",
            "description": "Partes do conteúdo que são suposições ainda não verificadas",
            "items": {
              "type": "object",
              "properties": {
                "assumption": { "type": "string" },
                "why_unverified": { "type": "string" },
                "suggested_verification": { "type": "string" }
              }
            }
          },
          "revoke_reason": {
            "type": "string",
            "description": "Obrigatório quando action=revoke"
          },
          "supersedes": {
            "type": "string",
            "description": "ID da memória que esta proposta substitui (para updates)"
          }
        }
      }
    },
    "contradictions_detected": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "existing_memory_id": { "type": "string" },
          "existing_claim": { "type": "string" },
          "new_claim": { "type": "string" },
          "resolution_strategy": {
            "type": "string",
            "enum": ["revoke_old", "keep_both_with_context", "escalate_to_human", "new_supersedes"]
          },
          "resolution_rationale": { "type": "string" }
        }
      }
    },
    "open_questions_flagged": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "question": { "type": "string" },
          "context": { "type": "string" },
          "urgency": { "type": "string", "enum": ["low", "medium", "high"] }
        }
      }
    },
    "human_approval_required": {
      "type": "object",
      "required": ["summary_for_human", "diff_preview", "requires_explicit_approval"],
      "properties": {
        "summary_for_human": {
          "type": "string",
          "description": "Resumo legível das mudanças propostas"
        },
        "diff_preview": {
          "type": "string",
          "description": "Diff das mudanças em .memory/ para revisão humana"
        },
        "requires_explicit_approval": {
          "type": "boolean",
          "default": true
        },
        "auto_approvable": {
          "type": "boolean",
          "description": "true apenas para revisões menores de baixo risco"
        },
        "high_risk_items": {
          "type": "array",
          "description": "Itens que precisam de atenção especial na revisão",
          "items": { "type": "string" }
        }
      }
    }
  }
}
```

***

## 8. Regras de Validação

### 8.1 Regras de Conteúdo

| Regra | Descrição | Ação |
|-------|-----------|------|
| `SEC-001` | Nenhum conteúdo pode ter padrões de segredo (regex: API keys, tokens, senhas)[^10][^11] | Bloquear e reportar |
| `SEC-002` | PII identificável (CPF, email pessoal, nome completo) deve ser removida | Redact antes de salvar |
| `CONF-001` | `confidence < 0.4` → evidence_level deve ser "low" | Rejeitar se incompatível |
| `CONF-002` | `evidence_level: high` exige ao menos 1 evidência de tipo `test_result`, `error_log` ou `commit` | Rejeitar se não atende |
| `EVID-001` | Todo `memory_proposal` precisa de pelo menos 1 `evidence_source` | Rejeitar candidato |
| `EVID-002` | Suposições explícitas ("acho que", "provavelmente", "talvez") devem ser sinalizadas em `unverified_assumption_flags` | Sinalizar, não bloquear |
| `DUP-001` | Similaridade semântica > 0.85 com memória existente = duplicata | Propor `update` em vez de `create` |
| `LEN-001` | `title` máx 120 chars, `content` máx 5000 chars | Truncar com alerta |
| `TYPE-001` | `memory_type` deve ser uma das 15 categorias definidas | Rejeitar se inválido |

### 8.2 Regras de Processo

- **Aprovação humana é OBRIGATÓRIA** para qualquer `action: create` ou `action: revoke`[^12][^13]
- `action: update` com `confidence` não reduzido e sem alteração de tipo pode ser auto-aprovável se marcado como `auto_approvable: true`
- `action: confirm_existing` não requer aprovação (apenas aumenta confidence de memória existente)
- Memórias com `security_warning` como tipo requerem revisão manual adicional antes de commit

### 8.3 Regras de Coerência

- Uma memória não pode `supersedes` a si mesma
- Uma memória revogada (`revoked: true`) não pode ser `superseded_by` uma memória também revogada
- `open_question` não deve ter `evidence_level: high` (se há alta evidência, deve ser outra categoria)

***

## 9. Heurísticas de Decisão

### 9.1 O Que Salvar

Salve quando **dois ou mais** dos seguintes critérios forem atendidos:[^4][^5]

1. **Recorrência**: O padrão apareceu em 2+ sessões ou arquivos diferentes
2. **Dor evitada**: O fato evitou ou teria evitado um erro real
3. **Não-óbvio**: Não está em README, docstring ou documentação padrão
4. **Verificado**: Há evidência concreta (teste passou, bug resolvido em prod, commit funcionou)
5. **Generalizável**: Aplica-se além da instância específica da sessão atual
6. **Decisão irreversível**: Especialmente para architecture_decision — o custo de refazer é alto
7. **Falha instrutiva**: Para failed_attempt — o custo de tentar novamente pode ser alto

**Regra de ouro**: *"Um desenvolvedor novo no projeto se beneficiaria dessa informação em 3 meses?"* Se sim, salve.

### 9.2 O Que Descartar

Descarte quando qualquer um dos seguintes critérios for atendido:

1. **É working context puro**: Raciocínio step-by-step, plano de debugging que não generalizou
2. **Já está no código**: Comentário, docstring, type hint ou README já documenta isso
3. **Muito específico**: Aplica-se apenas a um commit ou sprint específico sem valor futuro
4. **Baixa confiança sem evidência**: `confidence < 0.3` e nenhuma evidência objetiva
5. **É um segredo**: Qualquer valor sensível, mesmo que "anonimizado" parcialmente
6. **É PII**: Qualquer informação que identifique uma pessoa real
7. **Temporário por natureza**: "Por agora estamos usando X" sem perspectiva de duração
8. **Redundante**: Cobre exatamente o mesmo que uma memória existente com confiança igual ou maior
9. **Não verificado e especulativo**: Baseado apenas em impressão do agente, sem dado externo

***

## 10. Estratégia de Atualização Incremental

Baseada no padrão de supersession links e no modelo de confiança progressiva:[^14][^15][^5]

### 10.1 Pipeline de Atualização

```
Nova evidência detectada
        │
        ▼
Busca semântica em .memory/ (threshold: 0.75)
        │
   ┌────┴────────┐
   │             │
   Existe?       Não existe
   │             │
   ▼             ▼
Comparação       Nova memória candidata
de confiança     → Regras de Salvamento (Seção 9.1)
   │
   ├─ Nova > Antiga + 0.1: propose UPDATE
   ├─ Nova ≈ Antiga (±0.05): propose CONFIRM_EXISTING (reforço)
   ├─ Nova < Antiga - 0.2: descartar novo (antiga é mais confiável)
   └─ Contradição detectada: → Seção 11
```

### 10.2 Decay de Confiança

Implementar decay temporal conforme práticas de sistemas de memória com provenance:[^15][^16]

- Memórias não acessadas por **90 dias**: confidence × 0.9
- Memórias não acessadas por **180 dias**: confidence × 0.75
- Memórias não acessadas por **365 dias**: confidence × 0.5 + flag `needs_review: true`
- Threshold de revogação automática (proposta): confidence < 0.15

Memórias dos tipos `architecture_decision`, `domain_rule` e `security_warning` são **protegidas do decay** — requerem revogação explícita humana.

### 10.3 Reforço por Acesso

Quando uma memória é recuperada e **aplicada com sucesso** na sessão atual:
- confidence += min(0.05, 1.0 - current_confidence)
- `last_reinforced_at` atualizado
- Adicionar evidence_source do tipo "session_application"

***

## 11. Estratégia para Memórias Contraditórias

Baseada em AGM Belief Revision Theory e no modelo de reconciliação causal distribuída:[^16][^17][^18]

### 11.1 Tipos de Contradição

| Tipo | Exemplo | Estratégia |
|------|---------|------------|
| **Substituição direta** | "Use Vue" vs anterior "Use React" | `new_supersedes` — antiga recebe `superseded_by` |
| **Exceção parcial** | "Sempre use cache" vs "Cache não funciona para endpoint X" | `keep_both_with_context` — escopo delimitado |
| **Evolução temporal** | Comportamento da lib mudou na versão 3.x | `revoke_old` + `create` nova com versão explícita |
| **Ambiguidade genuína** | Dois arquitetos decidiram coisas opostas | `escalate_to_human` — marcar como conflito |
| **Especificidade de contexto** | Padrão A funciona em dev, padrão B em prod | `keep_both_with_context` — adicionar `scope` field |

### 11.2 Algoritmo de Resolução

```
score_conflict(m_old, m_new) =
  w1 * source_trust(m_new) +
  w2 * corroboration_count(m_new) +
  w3 * recency_decay(m_new.created_at) +
  w4 * evidence_level_score(m_new) -
  w5 * evidence_level_score(m_old)

onde w1=0.3, w2=0.25, w3=0.2, w4=0.15, w5=0.1
```

Este é análogo ao modelo de reconciliação do sistema DCM (Distributed Causal Memory).[^16]

Se `score_conflict > 0.6`: propor `new_supersedes`  
Se `0.2 < score < 0.6`: propor `escalate_to_human`  
Se `score < 0.2`: manter antiga, descartar nova

### 11.3 Representação de Conflito no Arquivo

```yaml
---
...
conflicting_with: "architecture_decision_auth-strategy_20260115"
conflict_type: "partial_exception"
conflict_scope: "módulo de autenticação de serviços externos"
conflict_resolution: "pending_human"
---
```

***

## 12. Estratégia de Revogação de Memórias Antigas

Inspirada no lifecycle management de memória de agentes de produção e no conceito de "esquecer com propósito":[^19][^4][^15]

### 12.1 Gatilhos de Revogação

| Gatilho | Ação |
|---------|------|
| Dependência removida do projeto | Revogar todas memórias do tipo `dependency_note` relacionadas |
| Breaking change de API externa | Revogar `integration_detail` da versão anterior |
| Refatoração maior de módulo | Revogar `fragile_area` e `implementation_pattern` do módulo |
| Decisão arquitetural revertida | Revogar `architecture_decision` e suas memórias derivadas |
| Bug marcado como "não vai corrigir" | Converter `bug_fix` em `domain_rule` explicando decisão |
| Confiança abaixo de threshold | Propor revogação automática (aprovação humana ainda necessária) |

### 12.2 Processo de Revogação

Memórias **nunca são deletadas** do histórico — são marcadas como revogadas. Isso preserva a auditabilidade e permite análise histórica de decisões:[^20]

```yaml
---
revoked: true
revoked_at: "2026-07-02T15:00:00-03:00"
revoked_reason: "Biblioteca axios foi substituída por fetch nativo no projeto"
superseded_by: "dependency_note_fetch-api_20260702"
revoked_by_session: "session_xyz789"
---
```

Arquivo movido para `.memory/_revoked/` após revogação.

### 12.3 Garbage Collection Periódico (propostas)

A cada 30 dias, o curador pode propor um relatório de "memórias candidatas à revogação", incluindo:
- Memórias com `confidence < 0.2`
- Memórias não acessadas por > 365 dias
- Memórias cujos `source_files` foram deletados do repositório
- Memórias de `open_question` com > 90 dias sem atualização

***

## 13. Detecção de Segredos

### 13.1 Padrões de Detecção

Baseado no OWASP Secrets Management e nas práticas do secrets-patterns-db:[^10][^11]

```json
{
  "secret_patterns": [
    {
      "name": "generic_api_key",
      "pattern": "(?i)(api[_-]?key|apikey|api_secret)\\s*[:=]\\s*['\"]?[A-Za-z0-9+/]{20,}['\"]?",
      "severity": "critical"
    },
    {
      "name": "aws_access_key",
      "pattern": "AKIA[0-9A-Z]{16}",
      "severity": "critical"
    },
    {
      "name": "generic_password",
      "pattern": "(?i)(password|passwd|pwd)\\s*[:=]\\s*['\"]?(?!\\$\\{)[A-Za-z0-9!@#$%^&*]{6,}['\"]?",
      "severity": "high"
    },
    {
      "name": "jwt_token",
      "pattern": "eyJ[A-Za-z0-9_-]{10,}\\.[A-Za-z0-9_-]{10,}\\.[A-Za-z0-9_-]{10,}",
      "severity": "critical"
    },
    {
      "name": "private_key_header",
      "pattern": "-----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----",
      "severity": "critical"
    },
    {
      "name": "connection_string",
      "pattern": "(?i)(mongodb|postgres|mysql|redis)://[^\\s]{10,}",
      "severity": "high"
    },
    {
      "name": "google_service_account",
      "pattern": "\"type\":\\s*\"service_account\"",
      "severity": "critical"
    },
    {
      "name": "pii_cpf_br",
      "pattern": "\\b\\d{3}\\.\\d{3}\\.\\d{3}-\\d{2}\\b",
      "severity": "high"
    }
  ]
}
```

### 13.2 Análise Semântica de Variáveis

Além de regex, aplicar análise semântica de nomes de variáveis suspeitos:[^21]

Palavras-chave que disparam inspeção adicional do valor:
`secret`, `token`, `password`, `credential`, `auth`, `key`, `cert`, `private`, `sensitive`, `confidential`

Se o valor associado tiver entropia de Shannon > 3.5 e comprimento > 15 chars, classificar como potencial segredo.

***

## 14. Detecção de Suposições Não Verificadas

### 14.1 Marcadores Linguísticos

O curador analisa o conteúdo buscando:

- **Suposições explícitas**: "acho que", "acredito que", "provavelmente", "talvez", "deve ser", "parece que", "imagino que", "não tenho certeza mas"
- **Correlações sem causalidade**: "depois que fiz X, Y parou de funcionar" (sem confirmação)
- **Generalizações prematuras**: "sempre funciona assim", "nunca vai acontecer"
- **Cargo cult**: "fizemos assim porque o projeto anterior fazia"

### 14.2 Output de Suposição Detectada

```json
{
  "unverified_assumption_flags": [
    {
      "assumption": "O cache Redis sempre invalida em 5 minutos",
      "why_unverified": "Afirmação baseada em conversa, sem código ou teste confirmando o TTL",
      "suggested_verification": "Verificar configuração em config/redis.js ou variável REDIS_TTL"
    }
  ]
}
```

***

## 15. Exemplos de Entrada e Saída

### 15.1 Exemplo: Bug Fix com Alta Evidência

**Entrada (fragmento)**:
```
Conversa: "Descobri que o upload de avatar falha quando o nome do arquivo tem espaços"
Diff: - filename = req.file.originalname
      + filename = req.file.originalname.replace(/\s+/g, '_')
Commit: a4f89bc "fix: sanitize filename spaces in avatar upload"
Teste: PASS upload.test.js > "should handle filename with spaces"
```

**Saída (fragmento)**:
```json
{
  "memory_proposals": js\nfilename = req.file.originalname.replace(/\\s+/g, '_')\n```\n\n## Não Fazer\nNão usar encode URL como solução — o S3 storage rejeita URLs com %20.",
      "confidence": 0.95,
      "evidence_level": "high",
      "evidence_sources": [
        { "type": "commit", "reference": "a4f89bc", "excerpt": "fix: sanitize filename spaces" },
        { "type": "test_result", "reference": "upload.test.js:87", "excerpt": "PASS should handle filename with spaces" }
      ],
      "source_files": ["src/upload/avatar_handler.js", "tests/upload.test.js"],
      "source_commits": ["a4f89bc"],
      "tags": ["upload", "avatar", "filename", "s3"]
    }
  ]
}
```

### 15.2 Exemplo: Bloqueio por Segredo Detectado

**Entrada**:
```
Conversa: "Para conectar ao Supabase de staging usa postgresql://admin:S3cr3t@db.staging.internal/app"
```

**Saída (fragmento)**:
```json
{
  "security_scan": {
    "passed": false,
    "issues_found": [
      {
        "type": "connection_string",
        "location": "conversation_transcript:line_34",
        "pattern_matched": "connection_string",
        "severity": "critical",
        "redacted_preview": "postgresql://admin:****@db.staging.internal/app"
      }
    ],
    "items_blocked": 1
  },
  "memory_proposals": []
}
```

### 15.3 Exemplo: Contradição Detectada

**Entrada**: Nova sessão afirma "Use Axios para todas as chamadas HTTP"  
**Memória existente**: `integration_detail_20260601` afirma "Axios foi removido; usar fetch nativo (PR #234)"

**Saída (fragmento)**:
```json
{
  "contradictions_detected": [
    {
      "existing_memory_id": "integration_detail_20260601",
      "existing_claim": "Axios foi removido; usar fetch nativo",
      "new_claim": "Use Axios para todas as chamadas HTTP",
      "resolution_strategy": "escalate_to_human",
      "resolution_rationale": "Conflito direto de stack decision. Memória existente tem evidence_level=high (PR confirmado). Nova afirmação vem de conversa sem evidência de código. Requer confirmação humana antes de qualquer ação."
    }
  ]
}
```

***

## 16. Referências Técnicas

As práticas desta especificação são fundamentadas em:

- **Taxonomia de memória de agentes** (working, episodic, semantic, procedural)[^4][^22] — base para a separação working context/memória durável
- **AGM Belief Revision Theory** aplicada a grafos de memória[^17] — fundamento para resolução de contradições
- **Distributed Causal Memory (DCM)** com provenance tracking e reconciliação por confiança ponderada[^16] — algoritmo de score de conflito
- **Knowledge Objects (KOs)** — framework de externalização de conhecimento implícito com validação humana[^6]
- **PROV-AGENT / W3C PROV** — modelo de proveniência para workflows agênticos[^20]
- **Agent Memory Protocol (AMP)** — estrutura de diretório markdown-first, git-friendly[^7]
- **Open Knowledge Format (Google Cloud)** — formato de memória como arquivos markdown com YAML frontmatter[^8]
- **SmartMemory Decision Confidence Scoring** — modelo de decay e reforço de confiança[^14]
- **Human-in-the-Loop Approval Patterns** — gates de aprovação para ações de agente com consequências duráveis[^13][^23]
- **OWASP Secrets Management Cheat Sheet** — padrões de detecção de credenciais[^10]
- **secrets-patterns-db** — banco de 1600+ regex para detecção de segredos[^11]
- **MemPrivacy / PII Taxonomy** — classificação hierárquica PL1-PL4 para privacidade em memória de agentes[^24]
- **Subtask-Level Memory for Software Engineering Agents** (arXiv:2602.21611)[^25] — granularidade de memória por fase de raciocínio
- **Conare Coding Agent Memory Architecture** — lifecycle de memória com supersession, decay e reranking[^5]

---

## References

1. [Building Agent Memory Systems: Short-Term Context vs Long-Term ...](https://www.mikul.me/blog/agent-memory-systems-short-term-long-term) - How to design memory architecture for AI agents that need to maintain context across tool calls, ses...

2. [Agent Memory Types (What Breaks in Production + Code) | Agent ...](https://www.agentpatterns.tech/en/foundations/agent-memory) - Without this, every new action would be like a first attempt. Sometimes endlessly.

3. [Agent Memory Types Explained - 601MEDIA](https://www.601media.com/agent-memory-types-explained/) - Agent Memory Types Explained: Short-Term, Long-Term, Shared, and “Do-Not-Store” AI agents don’t “rem...

4. [Agent memory systems: context, retrieval, and persistence](https://www.kunwar.page/chapter/132-agent-memory-systems-context-retrieval-and-persistence) - Every conversation with an LLM begins with amnesia. The model receives a prompt, generates a respons...

5. [AI Memory for Coding Agents: Architecture and Design](https://conare.ai/blog/ai-memory-for-coding-agents) - Coding agents without persistent memory can restart cold between sessions, losing context and forcin...

6. [Position: Reliable AI Needs to Externalize Implicit Knowledge](https://arxiv.org/html/2605.02010v2) - We propose Knowledge Objects (KOs)—structured artifacts that externalize implicit knowledge into for...

7. [Agent Memory Protocol (AMP) — An open standard for portable ...](https://github.com/agentmemoryprotocol/agentmemoryprotocol) - Agent Memory Protocol (AMP) — An open standard for portable, structured AI agent memory - agentmemor...

8. [Google publishes Open Knowledge Format for portable AI agent memory](https://vibin.live/google-publishes-open-knowledge-format-for-portable-ai-agent-memory)

9. [agent-memories - Lib.rs](https://lib.rs/crates/agent-memories) - Convention and validation for committed AI agent memories

10. [Secrets Management - OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) - Website with the collection of all the cheat sheets of the project.

11. [mazen160/secrets-patterns-db - GitHub](https://github.com/mazen160/secrets-patterns-db) - Secrets Patterns DB: The largest open-source Database for detecting secrets, API keys, passwords, to...

12. [Building a 'Human-in-the-Loop' Approval Gate for Autonomous Agents](https://machinelearningmastery.com/building-a-human-in-the-loop-approval-gate-for-autonomous-agents/) - In this article, you will learn how to implement state-managed interruptions in LangGraph so an agen...

13. [Human-in-the-Loop Approval Framework - Agentic Patterns](https://www.agentic-patterns.com/patterns/human-in-loop-approval-framework/) - Systematically insert human approval gates for designated high-risk functions while maintaining agen...

14. [Decision Memory](https://docs.smartmemory.ai/smartmemory/concepts/decision-memory/)

15. [Agent Memory MCP Server | Conare](https://conare.ai/marketplace/mcp/agent-memory) - An MCP server for managing agent memory using provenance tracking, decay-weighted retrieval, and fee...

16. [Distributed Causal Memory for Multi-Agent AI Systems: A Hybrid Architecture Combining Semantic Retrieval, Causal Graph Storage, Provenance Tracking, and Confidence-Weighted Reconciliation](https://zenodo.org/records/19141645) - We present the Distributed Causal Memory (DCM) system, a hybrid memory architecture for multi-agent ...

17. [Graph-Native Cognitive Memory for AI Agents: Formal Belief Revision Semantics for Versioned Memory Architectures](https://arxiv.org/pdf/2603.17244v1.pdf)

18. [AGM-Like Paraconsistent Belief Change](https://www.cle.unicamp.br/prof/coniglio/AGMparacons.pdf)

19. [AI Agent Memory Systems: Production Architecture Deep Dive](https://www.youtube.com/watch?v=pwxNsk5yQ7Y) - AI Agent Memory Systems: Production Architecture Deep Dive

A comprehensive technical deep dive into...

20. [PROV-AGENT: Unified Provenance for Tracking AI Agent ... - arXiv](https://arxiv.org/html/2508.02866v1)

21. [Building Rulesets](https://owasp.org/www-project-deepsecrets/) - Secrets scanner that understands code

22. [Memory Architecture Patterns | The AI Agent Factory - Panaversity](https://agentfactory.panaversity.org/docs/Building-Agent-Factories/augmented-memory/memory-architecture-patterns) - Learn the five types of agent memory—conversation, working, long-term, episodic, and semantic—and ho...

23. [Human-in-the-loop patterns · Cloudflare Agents docs](https://developers.cloudflare.com/agents/concepts/agentic-patterns/human-in-the-loop/) - Human-in-the-loop (HITL) patterns allow agents to pause execution and wait for human approval, confi...

24. [openai/privacy-filter · Great initiative! Exploring fine ... - Hugging Face](https://huggingface.co/openai/privacy-filter/discussions/22) - Hi,

25. [Subtask-Level Memory for Software Engineering Agents — AgentPatterns.ai](https://agentpatterns.ai/agent-design/subtask-level-memory/) - Store and retrieve memory at individual reasoning stages, not whole sessions, to prevent misguided r...

