# BUILD[xxx] — [OBJETIVO]

> propósito: marco completo de entrega: escopo, Sprints, gates e resultado
> ler quando: unidades desta Build
> atualizar quando: plano aprovado; fechamento de Sprint; fechamento da Build
> estado: PLANNED | BUILDING | VERIFIED | READY_FOR_RELEASE | READY_FOR_DEPLOY | RELEASED | REJECTED | ROLLED_BACK | HALT_BLOCKED
> versao_alvo: [versão]
> perfil: SIMPLE | STANDARD | CRITICAL

## Valor e requisitos

## Escopo e fora de escopo

## Definition of Ready

- [ ] PRD (ou escopo aprovado em DEC), SDD e DELIVERY suficientes
- [ ] dependências conhecidas
- [ ] Tasks verificáveis

## Sprints e unidades

| Ordem | Sprint | Unidade | Dependências | Estado |
| :--- | :--- | :--- | :--- | :--- |
| 1 | S01 | BUILDxxx_S01_T01 | nenhuma | READY |
| 2 | S01 | BUILDxxx_S01_CLOSE | S01_T01 | READY |
| final | — | BUILDxxx_CLOSE | todas as Sprints | READY |

Toda Build tem pelo menos uma Sprint. Toda Sprint termina com fechamento. A Build termina com `BUILDxxx_CLOSE`.

## Gates da Build

- [ ] verificações completas
- [ ] build limpa
- [ ] artefato e smoke
- [ ] histórico consolidado
- [ ] entrega ou deploy conforme `.memory/DELIVERY.md`

## Artefato e recuperação

## Resultado final
