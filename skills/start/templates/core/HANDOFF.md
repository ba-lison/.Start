# HANDOFF — [UNIDADE]

> propósito: passagem de bastão verificável da unidade no trilho FULL (imutável após fechado)
> ler quando: abertura do próximo chat (junto com `git status` e `git log -n 3`)
> atualizar quando: fechamento da unidade (`DONE`, `PAUSED` ou `BLOCKED`)
> data: YYYY-MM-DD
> estado: [DONE | PAUSED | BLOCKED]
> branch: [branch]

## Resultado e Arquivos Tocados
- **Objetivo da unidade:** [1 linha]
- **O que foi entregue:** [resumo direto]
- **Arquivos criados/editados:** [lista de caminhos]

## Verificação Comprovada
- `[comando]` → `[resultado real]`

## Decisões e Pendências do Chat
- **Decisões registradas:** `[nenhuma | DEC-xxx]`
- **Pendências fora de escopo anotadas:** `[nenhuma | PEND-xxx]`

## Se `PAUSED` ou `BLOCKED` (Onde Parei & Contador de Rodadas)
- `[— | passo exato + Rodada n/3 por objetivo + motivo]`

## Próxima Unidade e Prompt de Retomada
- **Próxima unidade:** `[ID — arquivo]`

```text
Continue o projeto em modo CONTINUAR (skill `start`): leia `.memory/00_INDEX.md` e `.memory/TASKS/HANDOFF_[UNIDADE].md`, confira `git status`/`HEAD` e confirme o escopo da próxima unidade (`[PRÓXIMA_UNIDADE]`).
```
