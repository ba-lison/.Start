# GUARDRAILS DO PROJETO — [PROJETO]

> propósito: o que o agente pode e não pode fazer neste projeto, aprovado pelo usuário
> ler quando: início de todo chat, antes de qualquer ação
> atualizar quando: durante o questionário (uma gravação por resposta); depois, só por pedido explícito do usuário (GUARDRAILS §10)
> status: DRAFT | APPROVED
> versao: [n]
> aprovado_em: [YYYY-MM-DD | PENDING]
> decisao_aprovacao: [DEC-xxx | PENDING]

## Progresso do questionário

- bloco_0_fixos: PENDING | ANSWERED
- bloco_1_chat: PENDING | ANSWERED
- bloco_2_subagentes: PENDING | ANSWERED
- bloco_3_rodadas: PENDING | ANSWERED
- bloco_4_decisoes: PENDING | ANSWERED
- bloco_5_git: PENDING | ANSWERED
- bloco_6_extras: PENDING | ANSWERED
- aprovacao: PENDING | ANSWERED

## Guardrails fixos

Valem integralmente: GF-01 a GF-14 de `.start/GUARDRAILS-E-CIRCUIT-BREAKERS.md` §2.

### Endurecimentos de GF

- [PENDING | nenhum | GF-xx: regra mais rígida — motivo]

## Parâmetros

Cada linha nasce `PENDING (padrão: ...)` e recebe o valor escolhido quando o bloco é respondido.

- GP-01 unidade_por_chat: PENDING (padrão: Task)
- GP-02 max_subagentes_por_chat: PENDING (padrão: 10)
- GP-03 max_caracteres_relatorio: PENDING (padrão: 500)
- GP-04 max_rodadas_por_objetivo: PENDING (padrão: 3)
- GP-05 forma_de_perguntar: PENDING (padrão: uma por vez)
- GP-06 commit_local: PENDING (padrão: automatico_ao_concluir)
- GP-07 acoes_remotas: PENDING (padrão: pedir_sempre)
- GP-08 branch_de_trabalho: PENDING (padrão: principal)
- GP-09 formato_commit: PENDING (padrão: <tipo>(<escopo>): <descrição> [UNIDADE])
- GP-10 areas_protegidas: PENDING (padrão: nenhuma além de segredos)
- GP-11 ferramentas: PENDING (padrão: ferramenta atual)
- GP-12 idioma_registros: PENDING (padrão: pt-BR)
- GP-13 gatilho_troca_de_chat: PENDING (padrão: compactacao_ou_40_mensagens)

## Itens adicionais de decisão

Além de D-01 a D-12 (`.start/GUARDRAILS-E-CIRCUIT-BREAKERS.md` §3.1):

- [PENDING | nenhum | D-P01: item]

## Regras extras do usuário

- [PENDING | nenhuma | GX-01: regra — motivo — alcance — origem: usuário | importada de `[arquivo]`]

## Histórico deste arquivo

- v[n] — YYYY-MM-DD — [aprovado no onboarding | aprovado no ajuste | mudança: item] — DEC-xxx
