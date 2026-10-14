# ÍNDICE DE UNIDADES

> propósito: todas as unidades de trabalho do projeto (uma por chat) e seus estados
> ler quando: início de todo chat; planejamento
> atualizar quando: unidade criada, iniciada (`RUNNING`) ou fechada (sobrescrever a linha)
> estados: BACKLOG | READY | RUNNING | PAUSED | BLOCKED | DONE | CANCELLED
> tipos: onboarding | prd | plano | task | fechamento-sprint | sprint | fechamento-build | ajuste

| Ordem | Unidade | Tipo | Objetivo | Estado | Commit principal | Handoff |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | ONBOARDING | onboarding | modo, guardrails e contexto | DONE | `a1b2c3d` | `HANDOFF_ONBOARDING.md` |
| 2 | PRD-001 | prd | requisitos da CLI | DONE | `b2c3d4e` | `HANDOFF_PRD-001.md` |
| 3 | PLANO-BUILD001 | plano | SDD, DELIVERY, Build, Sprint e Task | DONE | `c3d4e5f` | `HANDOFF_PLANO-BUILD001.md` |
| 4 | BUILD001_S01_T01 | task | implementar a CLI | DONE | `d4e5f60` | `HANDOFF_BUILD001_S01_T01.md` |
| 5 | BUILD001_S01_CLOSE | fechamento-sprint | validar e fechar a Sprint 01 | DONE | `e5f6071` | `HANDOFF_BUILD001_S01_CLOSE.md` |
| 6 | BUILD001_CLOSE | fechamento-build | fechar a Build e entregar v0.1.0 | DONE | `f607182` | `HANDOFF_BUILD001_CLOSE.md` |

## Próxima unidade

- unidade: a definir pelo usuário
- motivo: DEC-007 — PRD-002 só quando o usuário pedir
- bloqueios: nenhum
