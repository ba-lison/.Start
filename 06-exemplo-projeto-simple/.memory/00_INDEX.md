# ÍNDICE DA MEMÓRIA — CLI de saudação

> propósito: ponto de entrada da memória; diz o estado e a próxima unidade
> ler quando: primeiro arquivo de todo chat, depois de `AGENTS.md`
> atualizar quando: início do onboarding; mudança de parte; fechamento de toda unidade (`DONE`, `BLOCKED` ou `PAUSED`)
> atualizado_em: 2026-09-14
> versao_start: 3.0.0

## Estado

- modo_atual: CONTINUAR
- parte_atual: —
- perfil: SIMPLE
- build_ativa: nenhuma (BUILD001 entregue como v0.1.0)
- sprint_ativa: nenhuma

## Última unidade

- unidade: BUILD001_CLOSE
- estado: DONE
- handoff: `.memory/TASKS/HANDOFF_BUILD001_CLOSE.md`
- commit_principal: `f607182`

## Próxima unidade

- unidade: a definir pelo usuário (DEC-007)
- arquivo: a criar quando o usuário pedir a próxima entrega
- pré-condições: pedido do usuário para uma nova entrega (PRD-002)
- prompt: seção "Prompt para o próximo chat" do último handoff

## Mapa da memória

- `.memory/PROJECT_GUARDRAILS.md` — regras aprovadas — ler: sempre
- `.memory/PROJECT_CONTEXT.md` — fotografia atual — ler: sempre
- `.memory/DECISIONS.md` — decisões — ler: sempre (`ACTIVE` e `INFERRED`)
- `.memory/TASKS/INDEX.md` — unidades e estados — ler: sempre
- `.memory/DELIVERY.md` — comandos reais e entrega — ler: ao verificar, buildar ou entregar
- `.memory/CHANGELOG.md` — mudanças entregues — ler: no fechamento
- `.memory/prd/`, `.memory/sdd/` — especificação e arquitetura — ler: quando a unidade citar
- `.memory/BUILDS/`, `.memory/SPRINTS/` — planos — ler: a Build e a Sprint da unidade
- `.memory/RELEASES/` — entregas — ler: no fechamento de Build
- `.memory/TASKS/PENDENCIAS_FUTURAS.md` — pendências — ler: no planejamento
