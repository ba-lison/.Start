# GUARDRAILS DO PROJETO — CLI de saudação

> propósito: o que o agente pode e não pode fazer neste projeto, aprovado pelo usuário
> ler quando: início de todo chat, antes de qualquer ação
> atualizar quando: durante o questionário (uma gravação por resposta); depois, só por pedido explícito do usuário (GUARDRAILS §10)
> status: APPROVED
> versao: 1
> aprovado_em: 2026-09-12
> decisao_aprovacao: DEC-001

## Progresso do questionário

- bloco_0_fixos: ANSWERED
- bloco_1_chat: ANSWERED
- bloco_2_subagentes: ANSWERED
- bloco_3_rodadas: ANSWERED
- bloco_4_decisoes: ANSWERED
- bloco_5_git: ANSWERED
- bloco_6_extras: ANSWERED
- aprovacao: ANSWERED

## Guardrails fixos

Valem integralmente: GF-01 a GF-14 de `.start/GUARDRAILS-E-CIRCUIT-BREAKERS.md` §2.

### Endurecimentos de GF

- nenhum

## Parâmetros

- GP-01 unidade_por_chat: Task
- GP-02 max_subagentes_por_chat: 10
- GP-03 max_caracteres_relatorio: 500
- GP-04 max_rodadas_por_objetivo: 3
- GP-05 forma_de_perguntar: uma por vez
- GP-06 commit_local: automatico_ao_concluir
- GP-07 acoes_remotas: pedir_sempre
- GP-08 branch_de_trabalho: principal
- GP-09 formato_commit: <tipo>(<escopo>): <descrição> [UNIDADE]
- GP-10 areas_protegidas: nenhuma além de segredos
- GP-11 ferramentas: Codex, Cursor
- GP-12 idioma_registros: pt-BR
- GP-13 gatilho_troca_de_chat: compactacao_ou_40_mensagens

## Itens adicionais de decisão

- nenhum

## Regras extras do usuário

- nenhuma

## Histórico deste arquivo

- v1 — 2026-09-12 — aprovado no onboarding — DEC-001
