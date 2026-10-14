# MODO CONTINUAR — trabalho normal

> propósito: executar a próxima unidade de trabalho a partir de um estado verificado
> ler quando: `.start/INICIO.md` indicou CONTINUAR
> unidade: a indicada em `.memory/00_INDEX.md`

## Passo 1 — Ler, nesta ordem

1. `.memory/00_INDEX.md`
2. `.memory/PROJECT_GUARDRAILS.md`
3. `.memory/PROJECT_CONTEXT.md`
4. `.memory/DECISIONS.md` — entradas `ACTIVE` e `INFERRED`
5. o arquivo da próxima unidade, e a Build e a Sprint dela
6. o último handoff (indicado em `00_INDEX.md`)
7. a FASE da unidade (`.start/PROTOCOLO_MESTRE_EXECUCAO.md` §5)
8. só as referências que a unidade cita

SE a próxima unidade não está definida ("a definir pelo usuário"): anotar e seguir. A pergunta é feita no Passo 3.

## Passo 2 — Conferir a realidade (sem alterar)

- O `commit_principal` do último handoff existe e é ancestral da `branch:` citada no handoff (`git merge-base --is-ancestor [hash] [branch]`)? Com worktree por Task, conferir no worktree dessa branch.
- Os commits depois dele só alteram `.memory/` (registros de fechamento)? SE há commit do usuário fora do processo: relatar e perguntar.
- Há mudanças não commitadas? SE sim: relatar e perguntar.
- Sobrou `.memory/_ACTIVE_TASK.lock` ou `.memory/_rascunho/`?
- A unidade anterior está no estado que o `TASKS/INDEX.md` diz?
- Os caminhos citados existem?
- SE a próxima unidade está `PAUSED`: ler a seção "Se PAUSED: onde parei" do último handoff `_P[n]`, inclusive o contador de rodadas de cada objetivo.

SE houver divergência: relatar e perguntar antes de editar (GF-14).

## Passo 3 — Anunciar e confirmar

SE a próxima unidade não está definida: antes de anunciar, perguntar qual é (D-01), com opções tiradas das pendências e do último handoff. Só depois da resposta: registrar DEC, acrescentar a linha em `.memory/TASKS/INDEX.md` e atualizar `.memory/00_INDEX.md`.

```text
UNIDADE: [ID] — [objetivo]
ESCOPO: [lista curta]
FORA DE ESCOPO: [lista curta]
VERIFICAÇÃO: [comandos]
DECISÕES PENDENTES PARA COMEÇAR: [nenhuma | lista]
AUTORIZAÇÕES NECESSÁRIAS: [nenhuma | lista]
Confirma?
```

Depois do "sim": marcar a unidade como `RUNNING` em `.memory/TASKS/INDEX.md`.

## Passo 4 — Executar

Seguir a FASE da unidade, com GUARDRAILS (decisões §3; rodadas §5) e `.start/CONTRATO-SUBAGENTE.md`.

## Passo 5 — Fechar

Seguir `.start/PROTOCOLO_MESTRE_EXECUCAO.md` §8. PARE.

## Pedido do usuário fora da próxima unidade

SE o usuário pedir outra coisa: dizer qual é a unidade prevista e perguntar (D-01; DEC):

```text
A) Fazer o seu pedido como a unidade deste chat (a prevista fica para o próximo)
B) Registrar o pedido como pendência e seguir a unidade prevista
```
