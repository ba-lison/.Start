# HANDOFF — BUILD001_S01_T01

> propósito: o que este chat fez, com evidência, e como o próximo chat continua
> destino: `.memory/TASKS/HANDOFF_BUILD001_S01_T01.md`
> atualizar quando: nunca depois de fechado
> estado: DONE
> data: 2026-09-13
> branch: main
> commit_principal: `d4e5f60` (ilustrativo)

## Resultado

CLI implementada: nome válido imprime a saudação; nome vazio falha com código diferente de 0.

## Alterações

- `src/saudacao.js` — validação e formatação do nome
- `src/cli.js` — leitura do argumento e código de saída
- `tests/saudacao.test.js` — casos válido e vazio

## Verificação

- `npm test` → 4/4 passaram
- `npm run build` → `dist/cli.js` gerado
- `node dist/cli.js Ana` → `Olá, Ana!`

## Decisões deste chat

- nenhuma

## Rodadas e subagentes

- objetivo O1 (testes): rodadas 2/3 — ok (a 1ª falhou por espaço extra na saudação)
- objetivo O2 (build): rodadas 1/3 — ok
- objetivo O3 (smoke): rodadas 1/3 — ok
- subagentes usados: 0/10

## Pendências e riscos

- pendências: nenhuma
- riscos restantes: nenhum

## Estado verificável

- workspace: limpo
- lock: não usado
- divergências conhecidas: nenhuma

## Próxima unidade

- unidade: BUILD001_S01_CLOSE
- arquivo: `.memory/TASKS/BUILD001_S01_CLOSE.md`
- pré-condições: nenhuma

## Prompt para o próximo chat

```text
Continue o projeto pelo processo .start.
Unidade: BUILD001_S01_CLOSE. Handoff anterior: .memory/TASKS/HANDOFF_BUILD001_S01_T01.md
```
