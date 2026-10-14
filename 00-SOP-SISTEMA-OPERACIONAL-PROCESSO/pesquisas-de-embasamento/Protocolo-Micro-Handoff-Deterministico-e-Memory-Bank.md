# Protocolo de Micro-Handoff Determinístico e Gestão de Memory Bank

## Fundamento do Protocolo

Um Micro-Handoff determinístico resolve o problema central de projetos "1 Task por Chat": cada sessão nasce sem memória, e a qualidade do próximo chat depende inteiramente da integridade do artefato de transferência gerado pelo chat anterior. O padrão consolidado combina três camadas — um Memory Bank (arquivos Markdown persistentes por categoria), Architecture Decision Records (ADRs) para decisões irreversíveis, e um Handoff Artifact compacto em YAML gerado ao final de cada sessão.[^1][^2][^3][^4]

A regra central é separar o Immutable Store (histórico completo, nunca resumido) do Active Context (o que realmente entra no próximo prompt). Isso evita que a compactação apague decisões: comprime-se a apresentação, nunca a fonte.[^5][^6]

## Estrutura de Diretórios

| Camada | Localização | Atualização |
|---|---|---|
| Core (identidade/constraints do projeto) | `00-core/CORE.md` | Raramente |
| Arquitetura ativa | `01-architecture/` | Quando muda design |
| Estado ativo (o que está rolando agora) | `02-active/activeWork.md` | Toda sessão |
| ADRs (uma decisão = um arquivo) | `docs/decisions/ADR-NNNN-titulo.md` + `INDEX.md` | Por decisão relevante |
| Handoff da sessão | `05-sessions/handoff-latest.yaml` | Fim de cada chat |

Esse esquema segue o Memory Bank Pattern popularizado pelo Cline e adaptado por Claude Code, Cursor e Gemini CLI.[^3][^4]

## Formato de Dados: Handoff YAML

O artefato de handoff deve caber em aproximadamente 500 tokens, forçando síntese em vez de despejo de contexto. Campos obrigatórios para integridade de memória:[^1]

```yaml
handoff:
  version: "1.0"
  timestamp: "2026-09-03T16:00:00Z"   # ISO 8601
  from_task_id: "TASK-042"
  to_task_id: ""                       # preenchido na próxima sessão
  project: "nome-do-projeto"
  branch: "feat/nome-da-feature"

  task_context:
    status: "in_progress"              # done | in_progress | blocked
    current_task: "Descrição objetiva da tarefa atual"
    scope_boundary: "O que ESTÁ e o que NÃO ESTÁ incluso nesta task"

  decisions:                            # máx 5, apontar para ADR se existir
    - "Decisão X — ver ADR-0012"
    - "Decisão Y (sem ADR, de baixo impacto)"

  files_modified:                       # máx 10, path + 1 linha de motivo
    - path: "src/api/handler.js"
      reason: "Adicionado retry com backoff exponencial"

  open_questions:
    - "Pergunta não resolvida que bloqueia decisão futura"

  blockers:                             # máx 3
    - "Descrição do bloqueio + dependência externa"

  next_action: "Frase única e imperativa do que fazer a seguir"
  rollback_pointer: "commit_hash ou ADR-ID caso decisão precise ser revertida"
```

Este modelo é inspirado no padrão de handoff de agentes (Swarm/aios-core), com limites explícitos de itens por campo para forçar priorização em vez de despejo bruto.[^1]

## Preservando Decisões Arquiteturais (ADR)

Decisões arquiteturais nunca vão dentro do YAML de handoff — apenas a referência ao arquivo. Cada decisão relevante gera um ADR próprio, seguindo o formato Nygard/MADR consolidado pela comunidade:[^7][^8][^9]

```markdown
# ADR-0012: Usar Redis para cache de sessão de licença
> Quick Reference | Status: Accepted | Date: 2026-09-03
> Decisão: Cache de sessões de licença via Redis com TTL de 15min
> Contexto: Latência de verificação de licença em rede excedia SLA
> Alternativas: Cache em memória local, PostgreSQL com índice
> Impacto: Módulo de licenciamento, infraestrutura de rede

## Contexto
[2-5 frases sobre a situação e forças em jogo]

## Decisão
[1-3 frases, afirmativas: "Vamos usar..."]

## Alternativas Consideradas
- Opção A: prós/contras/por que rejeitada
- Opção B: prós/contras/por que rejeitada

## Consequências
### Positivas
### Negativas
### Critério de Revisão
[Condição que dispara reavaliação, ex: "se latência exceder Xms"]
```

O campo "Alternativas Consideradas" é o de maior valor prático — evita que a próxima sessão reproponha opções já descartadas sem saber o motivo. Um `INDEX.md` central lista todos os ADRs com status e data, permitindo que o próximo chat escaneie decisões sem abrir cada arquivo individualmente.[^10][^11][^2]

## Técnicas de Compactação sem Perda Arquitetural

A pesquisa em context engineering de 2026 define quatro operações complementares — Write, Select, Compress, Isolate:[^12]

- **Write**: mover todo raciocínio intermediário para arquivos externos (Memory Bank/ADR), nunca depender do histórico de chat como fonte de verdade.[^12]
- **Select**: a próxima sessão não lê tudo — lê apenas `INDEX.md` + `handoff-latest.yaml` + ADRs referenciados, por ordem de prioridade (00-core antes de tudo).[^3]
- **Compress**: aplicar compressão hierárquica — contexto de fundo pode ser resumido a 90%, mas decisões e constraints ativas ficam a 0% de compressão (verbatim).[^13]
- **Isolate**: decisões arquiteturais e código nunca são comprimidos juntos no mesmo resumo — misturar as duas camadas é a principal causa de "perda silenciosa" de decisão.[^14][^15]

Regra prática de proteção: qualquer texto que comece com "Decidimos", "Vamos usar" ou "Rejeitamos" deve ser extraído para ADR antes de qualquer resumo automático rodar sobre a conversa. Times de produção também disparam compactação proativa a 70-80% da capacidade de contexto, nunca esperando o limite físico, pois resumos gerados sob pressão perdem qualidade.[^15][^14]

## Modelo de Prompt de Continuidade

Para abrir o próximo chat de forma determinística, use um prompt fixo que injeta o handoff estruturado:

```
Você está retomando TASK-043 do projeto [nome].
Leia o handoff abaixo antes de qualquer ação:

--- HANDOFF ---
[colar handoff-latest.yaml]
--- FIM HANDOFF ---

Decisões arquiteturais vigentes: ADR-0010, ADR-0012 (ver INDEX.md se precisar do racional completo).

Regras:
1. Não reabra decisões já registradas em ADR sem justificativa nova.
2. Antes de codificar, confirme entendimento do "next_action" em 1 frase.
3. Ao finalizar, gere um novo bloco YAML de handoff no mesmo formato para a próxima sessão.

Tarefa desta sessão: [preencher com escopo específico se diferente do next_action]
```

Uma variante "mini" para tasks pequenas e rápidas, quando o handoff completo é excesso:

```
Continuação rápida — TASK-043.
Última decisão: [1 frase]. Arquivos tocados: [lista]. 
Próximo passo: [1 frase imperativa]. Bloqueio: [se houver].
Confirme e proceda.
```

Esse padrão de duas variantes (completa/mini) segue a lógica de "Mini-ADR" usada quando velocidade importa mais que completude, aplicada aqui ao prompt de continuidade em vez do ADR.[^16]

---

## References

1. [aios-core - UNPKG](https://app.unpkg.com/aios-core@4.3.0/files/.aios-core/development/templates/agent-handoff-tmpl.yaml) - The CDN for everything on npm

2. [everything-claude-code/skills/architecture-decision-records ... - GitHub](https://github.com/affaan-m/ECC/blob/main/skills/architecture-decision-records/SKILL.md) - The agent harness performance optimization system. Skills, instincts, memory, security, and research...

3. [Memory Bank Pattern](https://docs.tinyfat.com/guides/memory-bank/) - Organize project knowledge for AI agent collaboration.

4. [The AI Memory Bank pattern: what it is, why it breaks, and the upgrade](https://kitforai.com/blog/ai-memory-bank) - An AI Memory Bank is the folder of Markdown files an agent reads each session to remember your proje...

5. [LCM: Lossless Context Management](https://arxiv.org/html/2605.04050v1) - We introduce Lossless Context Management (LCM), a deterministic architecture for LLM memory that out...

6. [Lossless Context Management: LCM](https://www.losslesscontext.ai/) - Traditional agents use compaction systems that replace your conversation with lossy summaries when c...

7. [Active ADRs - Architecture Decision Records - llm-council](https://llm-council.dev/architecture/adrs/) - A multi-LLM deliberation system for collaborative AI reasoning

8. [ADR Templates | Architectural Decision Records](https://adr.github.io/adr-templates/) - The following UML class diagram shows that many templates for ADR capturing exist, including (but no...

9. [Architecture Decision Records: Templates & Review Workflow](https://creately.com/creately-system-design/architecture-decision-records-adrs-templates-review/) - Use practical architecture decision record templates and a clear ADR review workflow to document con...

10. [https://raw.githubusercontent.com/NeverSight/skill...](https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/mhylle/claude-skills-collection/adr/SKILL.md)

11. [AI Architecture Decision Record (ADR) Template for ...](https://www.institutepm.com/knowledge-hub/ai-architecture-decision-record-template) - Build real AI products, master GenAI & ML, and launch your AI PM career. 25+ hands-on modules, exper...

12. [Context Engineering in 2026: Developer Guide for LLM ...](https://devtoollab.com/blog/context-engineering-guide) - Context engineering replaced prompt engineering in 2026. Learn RAG strategies, token budget manageme...

13. [Context Management: Compression, Memory Systems, and Token ...](https://deepwiki.com/jasontang-ai/Context-Engineering/5.3-context-management:-compression-memory-systems-and-token-budgeting) - This page details the mechanisms for managing information within the finite constraints of a Large L...

14. [Context Window Economics — Managing Token Budgets in ...](https://zylos.ai/en/research/2026-05-27-context-window-economics-persistent-agents/) - Engineering strategies for token budget management in long-running AI agents: compaction, tiered mem...

15. [What Fills the Context Window: A Guide to Context Engineering](https://www.henryvu.blog/series/ai-engineering/part1.html) - The seven components of a context window, four strategies for managing them, failure modes, token bu...

16. [ADR (Architecture Decision Record) — Documentation Pattern.md](https://gist.github.com/thedavidyoungblood/bccce859af7a476e44a290a2230e0913) - ADR (Architecture Decision Record) — Documentation Pattern.md - ADR (Architecture Decision Record) —...

