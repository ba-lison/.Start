# HANDOFF — [UNIDADE]

> propósito: o que este chat fez, com evidência, e como o próximo chat continua
> destino: `.memory/TASKS/HANDOFF_[UNIDADE].md` no fechamento final; `.memory/TASKS/HANDOFF_[UNIDADE]_P[n].md` em cada pausa (n = 1, 2, …)
> atualizar quando: nunca depois de fechado; erro factual só por seção anexada (`.start/FORMATO-LLM.md` §5)
> estado: DONE | BLOCKED | PAUSED
> data: YYYY-MM-DD
> branch: [nome]
> commit_principal: `[hash]` (o commit de fechamento aparece só no chat, como `HEAD` final)

## Resultado

[1 a 3 frases: o que existe agora e não existia antes]

## Alterações

- `[arquivo]` — [o que mudou e por quê]

## Verificação

- `[comando]` → [resultado resumido] — evidência: [onde está]

## Decisões deste chat

- [DEC-xxx — título | nenhuma]
- pendentes de registro: [decisão ainda não gravada em `.memory/DECISIONS.md` e por quê | nenhuma]

## Rodadas e subagentes

- objetivo [O1]: rodadas [n]/[GP-04] — [ok | em andamento | esgotou → `.memory/TASKS/BLOQUEIO_[UNIDADE].md`]
- subagentes usados: [n]/[GP-02]

## Pendências e riscos

- pendências: [PEND-xxx | nenhuma]
- riscos restantes: [lista | nenhum]

## Estado verificável

- workspace: [limpo | alterações restantes e por quê]
- lock: [removido | não usado]
- divergências conhecidas: [nenhuma | lista]

## Se PAUSED: onde parei

- feito:
- falta:
- próximo passo exato:
- contador de rodadas por objetivo: [O1: n/GP-04; …]

## Próxima unidade

- unidade: [ID]
- arquivo: `[caminho]`
- pré-condições: [lista | nenhuma]

## Prompt para o próximo chat

SE todas as ferramentas de GP-11 carregam `AGENTS.md` sozinhas, usar a versão curta. SE não, usar a versão completa de `.start/TEMPLATES/TEMPLATE_PROMPT_CONTINUIDADE_CHAT.md`.

```text
Continue o projeto pelo processo .start.
Unidade: [ID]. Handoff anterior: [caminho deste handoff]
```

SE a próxima unidade está "a definir pelo usuário":

```text
Continue o projeto pelo processo .start.
Quero: [descrever a próxima entrega]. Handoff anterior: [caminho deste handoff]
```
