# RELEASE — BUILD001 / v0.1.0

> propósito: registro da entrega de uma Build: o que foi entregue, com qual evidência, onde e em que estado
> destino: `.memory/RELEASES/RELEASE_BUILD001.md`
> atualizar quando: nunca depois de fechado
> estado: RELEASED
> data: 2026-09-14

## Resumo e requisitos entregues

RF-001 e RF-002 entregues depois do fechamento da BUILD001_S01.

## Unidades

| Unidade | Estado | Handoff | Verificação |
| :--- | :--- | :--- | :--- |
| BUILD001_S01_T01 | DONE | `.memory/TASKS/HANDOFF_BUILD001_S01_T01.md` | testes, build, smoke |
| BUILD001_S01_CLOSE | DONE | `.memory/TASKS/HANDOFF_BUILD001_S01_CLOSE.md` | verificação integrada |

## Gates

| Gate | Resultado | Evidência |
| :--- | :--- | :--- |
| Suíte aplicável | PASS | `npm test` |
| Build limpa | PASS | `npm run build` |
| Smoke do artefato | PASS | `node dist/cli.js Ana` a partir do zip |
| Segurança/migração | N/A | exemplo local, sem dados (DEC-006) |

## Artefatos

| Nome | Versão/plataforma | SHA-256 | Origem | Destino |
| :--- | :--- | :--- | :--- | :--- |
| `cli-saudacao-0.1.0.zip` | 0.1.0 / Node.js | `[exemplo]` | `f607182` | pasta local |

## Entrega ou deploy

- ambiente ou canal: pacote local
- aprovação: DEC-006 (entrega local prevista no plano)
- resultado: pacote gerado e testado
- verificação pós-entrega: smoke do artefato extraído
- observação: deploy N/A

## Histórico consolidado

- changelog: `.memory/CHANGELOG.md`, versão 0.1.0
- decisões: DEC-001 a DEC-007
- pendências: nenhuma

## Próxima ação

A definir pelo usuário (DEC-007).

## Git

- commit de origem da Release (commit principal): `f607182` (ilustrativo)
- tag local: `v0.1.0`
