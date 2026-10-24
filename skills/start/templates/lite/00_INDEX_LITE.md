# ÍNDICE E ESCOPO DO PROJETO — [NOME DO PROJETO]

> propósito: ponto único de entrada do trilho LITE (estado, escopo ativo, guardrails e próximo passo)
> ler quando: início de todo chat neste projeto
> atualizar quando: início e fechamento de cada entrega (`DONE`, `PAUSED` ou `BLOCKED`)
> versao_start: 4.0.0
> trilho: LITE
> status: APPROVED

## 1. Objetivo e Stack
- **O que é:** [1–2 linhas diretas]
- **Stack & Comandos Reais:**
  - dev/run: `[comando]`
  - test/check: `[comando]`
  - build: `[comando | N/A]`

## 2. Escopo Ativo
- **Dentro do escopo:**
  - [ ] [entrega / critério verificável 1]
  - [ ] [entrega / critério verificável 2]
- **Fora de escopo (não fazer agora):**
  - [item explícito fora de escopo]

## 3. Guardrails Ativos
- **Circuit Breaker:** máx. `3` rodadas por objetivo (repetiu erro sem hipótese nova → para e pergunta).
- **Subagentes:** profundidade `1`, máx. `10` por chat, relatório `≤ 500` caracteres.
- **Git:** 1 commit local limpo por entrega concluída; `push`/PR/deploy somente com autorização explícita.
- **Áreas protegidas / Regras extras:** [nenhuma além de segredos `.env*` | listar]

## 4. Estado Atual e Ponteiros
- **Estado:** `[READY | RUNNING | PAUSED | BLOCKED | DONE]`
- **Último handoff:** `.memory/HANDOFF.md`
- **Decisões ativas:** `.memory/DECISIONS.md`
- **Próximo passo:** [descrição curta da próxima ação]
