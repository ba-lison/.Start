# RELEASE — [BUILD_ID / VERSÃO]

> propósito: registro da entrega de uma Build: o que foi entregue, com qual evidência, onde e em que estado
> destino: `.memory/RELEASES/RELEASE_BUILDxxx.md`
> atualizar quando: nunca depois de fechado; incidente, recuperação ou erro factual só por seção anexada (`.start/FORMATO-LLM.md` §5)
> estado: VERIFIED | READY_FOR_RELEASE | READY_FOR_DEPLOY | RELEASED | REJECTED | ROLLED_BACK | HALT_BLOCKED
> data: YYYY-MM-DD

## Resumo e requisitos entregues

## Unidades

| Unidade | Estado | Handoff | Verificação |
| :--- | :--- | :--- | :--- |

## Gates

| Gate | Resultado | Evidência |
| :--- | :--- | :--- |
| Suíte aplicável | PASS/FAIL/N/A | [link ou resumo] |
| Build limpa | PASS/FAIL/N/A | |
| Smoke do artefato | PASS/FAIL/N/A | |
| Segurança/migração | PASS/FAIL/N/A | |

## Artefatos

| Nome | Versão/plataforma | SHA-256 | Origem | Destino |
| :--- | :--- | :--- | :--- | :--- |

## Entrega ou deploy

- ambiente ou canal:
- aprovação: [DEC-xxx]
- resultado:
- verificação pós-entrega:
- observação:

## Recuperação e incidentes

## Histórico consolidado

- changelog:
- decisões e ADRs:
- pendências:

## Próxima ação

## Git

Preenchido no commit dos registros de fechamento. O `HEAD` final aparece só no chat.

- commit de origem da Release (commit principal): `[hash]`
- tag local, quando aplicável: `[tag]`
