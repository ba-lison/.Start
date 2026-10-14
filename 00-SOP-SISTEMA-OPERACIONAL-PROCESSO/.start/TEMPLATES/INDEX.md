# ÍNDICE DOS TEMPLATES

> propósito: qual template usar, onde o arquivo final fica e quando ele é criado
> ler quando: antes de criar qualquer arquivo do processo

| Template | Destino no projeto | Criado em |
| :--- | :--- | :--- |
| `TEMPLATE_AGENTS.md` | `AGENTS.md` | Parte 0 (NOVO) ou Preparação (ADOTAR, AJUSTAR) |
| `TEMPLATE_PONTEIROS.md` | `CLAUDE.md`, `.agents/rules/start.md` | Parte 0 ou Preparação (ferramenta atual); aprovação dos guardrails (demais ferramentas de GP-11) |
| `TEMPLATE_00_INDEX.md` | `.memory/00_INDEX.md` | Parte 0 (NOVO) ou Preparação (ADOTAR, AJUSTAR); todo fechamento |
| `TEMPLATE_PROJECT_GUARDRAILS.md` | `.memory/PROJECT_GUARDRAILS.md` | questionário |
| `TEMPLATE_DIAGNOSTICO.md` | `.memory/DIAGNOSTICO.md` ou `.memory/DIAGNOSTICO-AJUSTE-YYYY-MM-DD.md` | ADOTAR e AJUSTAR |
| `TEMPLATE_PROJECT_CONTEXT.md` | `.memory/PROJECT_CONTEXT.md` | Fase 01 |
| `TEMPLATE_DECISIONS.md` | `.memory/DECISIONS.md` | Parte 0 |
| `TEMPLATE_ADR.md` | `.memory/adrs/ADR-NNNN.md` | decisão duradoura |
| `TEMPLATE_CHANGELOG.md` | `.memory/CHANGELOG.md` | Parte 0 |
| `TEMPLATE_PENDENCIAS_FUTURAS.md` | `.memory/TASKS/PENDENCIAS_FUTURAS.md` | Parte 0 |
| `TEMPLATE_TASK_INDEX.md` | `.memory/TASKS/INDEX.md` | Parte 0 |
| `TEMPLATE_PRD.md` | `.memory/prd/PRD-xxx.md` | Fase 02 |
| `TEMPLATE_SDD.md` | `.memory/sdd/SDD-xxx.md` | Fase 03 |
| `TEMPLATE_DELIVERY.md` | `.memory/DELIVERY.md` | Fase 03 (ADOTAR: parcial na Parte 4) |
| `TEMPLATE_BUILD.md` | `.memory/BUILDS/BUILDxxx.md` | Fase 03 |
| `TEMPLATE_SPRINT.md` | `.memory/SPRINTS/BUILDxxx_Syy.md` | Fase 03 |
| `TEMPLATE_TASK.md` | `.memory/TASKS/BUILDxxx_Syy_Tzz.md` | Fase 03 |
| `TEMPLATE_HANDOFF.md` | `.memory/TASKS/HANDOFF_[UNIDADE].md` | fechamento de toda unidade |
| `TEMPLATE_PROMPT_CONTINUIDADE_CHAT.md` | fim do handoff e última mensagem do chat | fechamento de toda unidade |
| `TEMPLATE_BLOQUEIO.md` | `.memory/TASKS/BLOQUEIO_[UNIDADE].md` | parada (GUARDRAILS §8) |
| `TEMPLATE_RELEASE.md` | `.memory/RELEASES/RELEASE_BUILDxxx.md` | Fase 05 |
| `TEMPLATE_ACTIVE_TASK_LOCK.json` | `.memory/_ACTIVE_TASK.lock` | opcional, se `.memory/DELIVERY.md` adotar |
