# BUILD001_S01 — CLI funcional

> propósito: incremento funcional verificável dentro da Build
> ler quando: Tasks e fechamento desta Sprint
> atualizar quando: cada Task fechada; fechamento da Sprint
> estado: DONE
> estados possíveis: PLANNED | RUNNING | PAUSED | BLOCKED | DONE
> build: BUILD001

## Objetivo funcional

Saudação com validação de entrada.

## Incremento demonstrável

`node dist/cli.js Ana` imprime `Olá, Ana!`; nome vazio falha com código diferente de 0.

## Tasks

| Ordem | Unidade | Dependências | Estado | Handoff |
| :--- | :--- | :--- | :--- | :--- |
| 1 | BUILD001_S01_T01 | nenhuma | DONE | `.memory/TASKS/HANDOFF_BUILD001_S01_T01.md` |
| final | BUILD001_S01_CLOSE | S01_T01 | DONE | `.memory/TASKS/HANDOFF_BUILD001_S01_CLOSE.md` |

## Critérios de conclusão integrados

- [x] todas as Tasks previstas foram concluídas
- [x] a verificação integrada da Sprint passou
- [x] handoffs e histórico estão consistentes
- [x] pendências registradas (nenhuma)
- [x] próxima etapa identificada: BUILD001_CLOSE

## Resultado do fechamento

- commit principal do fechamento da Sprint: `e5f6071`
