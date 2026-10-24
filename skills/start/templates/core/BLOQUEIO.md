# RELATÓRIO DE BLOQUEIO — [UNIDADE]

> propósito: registrar parada por Circuit Breaker (rodadas esgotadas), risco de segurança/destrutivo ou falta de permissão
> ler quando: retomada de unidade em estado `BLOCKED`
> atualizar quando: abertura do bloqueio + seção `## Resposta do Usuário` após decisão
> data: YYYY-MM-DD
> motivo_parada: [RODADAS_ESGOTADAS | ERRO_REPETIDO_SEM_HIPOTESE | RISCO_SEGURANCA | FALTA_PERMISSAO]

## Contexto do Bloqueio
- **Objetivo travado:** [descrição]
- **Rodadas gastas:** `[n/3]`

## Tentativas e Evidências Reais
1. **Rodada 1:** [hipótese testada] → `[comando]` → `[erro resumido]`
2. **Rodada 2:** [hipótese testada] → `[comando]` → `[erro resumido]`
3. **Rodada 3:** [hipótese testada] → `[comando]` → `[erro resumido]`

## Estado Preservado no Repositório
- **Arquivos alterados sem commit (ou wip):** [lista]
- **Nada foi revertido à força (`git reset` não executado).**

## Opções para Destravar (Decisão do Usuário)
- **A)** [opção recomendada + impacto]
- **B)** [alternativa + impacto]
