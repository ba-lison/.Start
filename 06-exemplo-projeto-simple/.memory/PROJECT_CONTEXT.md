# CLI de saudação — CONTEXTO ATUAL

> propósito: fotografia atual do projeto: o que é, como está e como verificar
> ler quando: início de todo chat
> atualizar quando: a fotografia mudou (sobrescrever; o Git guarda o histórico)
> atualizado_em: 2026-09-14
> perfil: SIMPLE

## Problema e usuários

Demonstrar o processo .start com o menor produto possível: uma CLI que recebe um nome e imprime `Olá, <nome>!`. Usuário: quem estuda o processo.

## Resultado esperado

Uma Release local verificável (v0.1.0), com histórico completo da ideia à entrega.

## Estado atual

v0.1.0 entregue como pacote local (BUILD001 `RELEASED`). Nenhuma Build em andamento.

## Arquitetura resumida

Um módulo puro valida e formata o nome; uma camada CLI traduz argumentos e códigos de saída (`.memory/sdd/SDD-001.md`).

## Restrições

- sem rede, persistência ou interface gráfica;
- Node.js com saída textual (DEC-005).

## Riscos e dúvidas abertas

- nenhum

## Como executar e verificar

- setup: `npm ci`
- teste: `npm test`
- build: `npm run build`
- smoke: `node dist/cli.js Ana`

## Entrega atual

- build_versao: `BUILD001` / `v0.1.0`
- estado: `RELEASED` (entrega local)

## Ponteiros

- guardrails: `.memory/PROJECT_GUARDRAILS.md`
- decisões: `.memory/DECISIONS.md`
- PRD: `.memory/prd/PRD-001.md`
- SDD: `.memory/sdd/SDD-001.md`
- delivery: `.memory/DELIVERY.md`
- build ativa: nenhuma — última: `.memory/BUILDS/BUILD001.md`
- documentação própria do projeto: nenhuma
