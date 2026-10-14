# PROMPT DE CONTINUIDADE

> propósito: texto que o usuário cola para abrir o próximo chat
> usar quando: fechamento de toda unidade (vai no fim do handoff e na última mensagem do chat)

## Versão curta — a ferramenta carrega o `AGENTS.md` sozinha

```text
Continue o projeto pelo processo .start.
Unidade: [ID]. Handoff anterior: .memory/TASKS/HANDOFF_[ID_ANTERIOR].md
```

SE a próxima unidade está "a definir pelo usuário":

```text
Continue o projeto pelo processo .start.
Quero: [descrever a próxima entrega]. Handoff anterior: .memory/TASKS/HANDOFF_[ID_ANTERIOR].md
```

## Versão completa — a ferramenta não carrega o `AGENTS.md`, ou há dúvida

```text
Atue somente na unidade [ID] deste repositório, pelo processo .start.

Antes de qualquer ação:
1. leia AGENTS.md e .start/INICIO.md;
2. leia .memory/00_INDEX.md, .memory/PROJECT_GUARDRAILS.md, .memory/PROJECT_CONTEXT.md e .memory/DECISIONS.md;
3. leia [CAMINHO_DA_UNIDADE] e .memory/TASKS/HANDOFF_[ID_ANTERIOR].md;
4. confira Git e workspace contra o handoff e me mostre as divergências;
5. me mostre unidade, escopo, fora de escopo, verificação e autorizações, e peça confirmação.

Regras: não decida nada da lista de decisões sem me perguntar; registre cada resposta em .memory/DECISIONS.md na hora; respeite os limites de subagentes e de rodadas de .memory/PROJECT_GUARDRAILS.md; ao terminar, faça o handoff e os commits locais, atualize .memory/00_INDEX.md, me dê o prompt do próximo chat e pare.
```
