# ÍNDICE DA MEMÓRIA — [PROJETO]

> propósito: ponto de entrada da memória; diz o estado e a próxima unidade
> ler quando: primeiro arquivo de todo chat, depois de `AGENTS.md`
> atualizar quando: início do onboarding ou do ajuste; mudança de parte; fechamento de toda unidade (`DONE`, `BLOCKED` ou `PAUSED`)
> atualizado_em: YYYY-MM-DD
> versao_start: [versão do kit | 2.x durante a migração da versão 2]

## Estado

- modo_atual: NOVO | ADOTAR | AJUSTAR | CONTINUAR
- parte_atual: [n | —]
- perfil: SIMPLE | STANDARD | CRITICAL | a definir
- build_ativa: [BUILDxxx | nenhuma]
- sprint_ativa: [BUILDxxx_Syy | nenhuma]

## Última unidade

- unidade: [ID | nenhuma]
- estado: [DONE | BLOCKED | PAUSED | —]
- handoff: [`.memory/TASKS/HANDOFF_[ID].md` | `.memory/TASKS/HANDOFF_[ID]_P[n].md` | —]
- commit_principal: [`hash` | —]

## Próxima unidade

- unidade: [ID | a definir pelo usuário]
- arquivo: [`caminho` | a criar: `caminho`]
- pré-condições: [lista | nenhuma]
- prompt: seção "Prompt para o próximo chat" do último handoff

## Mapa da memória

- `.memory/PROJECT_GUARDRAILS.md` — regras aprovadas — ler: sempre
- `.memory/PROJECT_CONTEXT.md` — fotografia atual — ler: sempre
- `.memory/DECISIONS.md` — decisões — ler: sempre (`ACTIVE` e `INFERRED`)
- `.memory/TASKS/INDEX.md` — unidades e estados — ler: sempre
- `.memory/DELIVERY.md` — comandos reais e entrega — ler: ao verificar, buildar ou entregar
- `.memory/CHANGELOG.md` — mudanças entregues — ler: no fechamento
- `.memory/DIAGNOSTICO.md` — diagnóstico inicial — ler: se a unidade depender dele
- `.memory/prd/`, `.memory/sdd/`, `.memory/adrs/` — especificação e arquitetura — ler: quando a unidade citar
- `.memory/BUILDS/`, `.memory/SPRINTS/` — planos — ler: a Build e a Sprint da unidade
- `.memory/RELEASES/` — entregas — ler: no fechamento de Build
- `.memory/TASKS/PENDENCIAS_FUTURAS.md` — pendências — ler: no planejamento
