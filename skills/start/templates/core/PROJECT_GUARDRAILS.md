# GUARDRAILS DO PROJETO — [NOME DO PROJETO]

> propósito: parâmetros operacionais e regras extras aprovadas pelo usuário neste projeto
> ler quando: início de todo chat no trilho FULL
> atualizar quando: aprovação no onboarding ou alteração explícita autorizada pelo usuário (`DEC-xxx`)
> status: [DRAFT | APPROVED]
> aprovado_em: YYYY-MM-DD
> decisao_aprovacao: DEC-001

## Parâmetros Ativos (`GP`)
- `GP-01` **unidade_por_chat:** `Task`
- `GP-02` **max_subagentes_por_chat:** `10`
- `GP-03` **max_caracteres_relatorio:** `500`
- `GP-04` **max_rodadas_por_objetivo:** `3`
- `GP-05` **commit_local:** `automatico_ao_concluir`
- `GP-06` **acoes_remotas:** `pedir_sempre`
- `GP-07` **branch_de_trabalho:** `principal`
- `GP-08` **areas_protegidas:** `nenhuma além de segredos (.env*, *.pem, *.key, credentials*)`
- `GP-09` **idioma_registros:** `pt-BR`
- `GP-10` **gatilho_pausa_chat:** `compactacao_ou_40_msgs`

## Endurecimentos de Guardrails Fixos (`GF`) e Regras Extras (`GX`)
- `[nenhuma regra extra | GX-01: descrição e origem]`

## Itens Adicionais de Decisão Obrigatória (`D-Pxx`)
- `[nenhum além de D-01..D-12 | D-P01: descrição]`
