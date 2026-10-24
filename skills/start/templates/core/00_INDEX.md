# ÍNDICE DA MEMÓRIA — [NOME DO PROJETO]

> propósito: ponto único de entrada da memória no trilho FULL; informa estado e próxima unidade
> ler quando: primeiro arquivo de todo chat, logo após `AGENTS.md`
> atualizar quando: fechamento de toda unidade (`DONE`, `PAUSED` ou `BLOCKED`)
> atualizado_em: YYYY-MM-DD
> versao_start: 4.0.0
> trilho: FULL

## Estado Atual
- **modo_atual:** `[ONBOARDING | CONTINUAR | MIGRAR]`
- **perfil:** `[SIMPLE | STANDARD | CRITICAL]`
- **build_ativa:** `[nenhuma | BUILD001]`
- **sprint_ativa:** `[nenhuma | BUILD001_S01]`

## Última Unidade
- **unidade:** `[ONBOARDING | BUILD001_S01_T01]`
- **estado:** `[DONE | PAUSED | BLOCKED]`
- **handoff:** `.memory/TASKS/HANDOFF_[UNIDADE].md`

## Próxima Unidade
- **unidade:** `[PRD-001 | PLANO-BUILD001 | BUILD001_S01_T01 | a definir pelo usuário]`
- **arquivo:** `[caminho exato em .memory/]`
- **objetivo curto:** `[1 linha]`

## Mapa da `.memory/` (Carregar Apenas o Necessário)
- `.memory/PROJECT_GUARDRAILS.md` — parâmetros e regras aprovadas — **ler: sempre**
- `.memory/PROJECT_CONTEXT.md` — fotografia atual do produto e stack — **ler: sempre**
- `.memory/DECISIONS.md` — decisões `ACTIVE` e `INFERRED` — **ler: sempre**
- `.memory/TASKS/INDEX.md` — fila de unidades e estados — **ler: sempre**
- `.memory/DELIVERY.md` — comandos reais, CI e release — **ler: ao testar, buildar ou entregar**
- `.memory/prd/` e `.memory/sdd/` — requisitos e arquitetura — **ler: quando a unidade citar**
- `.memory/BUILDS/` e `.memory/SPRINTS/` — escopo da Build/Sprint ativa — **ler: na execução**
- `.memory/CHANGELOG.md` e `.memory/TASKS/PENDENCIAS_FUTURAS.md` — **ler: no planejamento/fechamento**
