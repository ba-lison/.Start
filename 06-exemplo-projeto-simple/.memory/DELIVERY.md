# DELIVERY — CLI de saudação

> propósito: como este projeto é verificado, versionado e entregue (comandos reais)
> ler quando: ao verificar, buildar ou entregar
> atualizar quando: comando, pipeline, ambiente ou forma de entrega mudou (sobrescrever)
> perfil: SIMPLE
> atualizado_em: 2026-09-12

## Produto e destino

- tipo: CLI
- destinatário: uso local
- forma de entrega: pacote zip local

## Comandos canônicos

| Ação | Comando |
| :--- | :--- |
| Setup | `npm ci` |
| Lint/tipos | N/A — exemplo mínimo sem linter (DEC-006) |
| Testes | `npm test` |
| Build limpa | `npm run build` |
| Pacote | compactar `dist/` em `cli-saudacao-<versão>.zip` |
| Smoke | `node dist/cli.js Ana` → `Olá, Ana!` |

## Git e CI

- commit local: GP-06 = automatico_ao_concluir
- push, PR, merge, tag remota e deploy: GP-07 = pedir_sempre
- branch ou worktree por Task: GP-08 = principal
- formato de commit: GP-09 = `<tipo>(<escopo>): <descrição> [UNIDADE]`
- workflow de CI: N/A — projeto local (DEC-006)
- lock `.memory/_ACTIVE_TASK.lock`: não usado

## Versão e artefato

- política de versão: SemVer; bump no fechamento da Build
- artefato: `dist/` compactado, ligado ao commit de origem e ao SHA-256
- retenção, assinatura e SBOM: N/A — exemplo local (DEC-006)

## Ambientes e configuração

- ambientes: local
- segredos e configuração: nenhum

## Entrega ou deploy

- procedimento: gerar o zip, testar o artefato e registrar o SHA-256 no Release
- deploy: N/A — sem serviço (DEC-006)

## Recuperação e incidentes

- recuperação pelo Git; sem cópias `.backup/`

## Segurança e métricas

- itens N/A e justificativas: CI, deploy, banco, feature flags, DORA e SBOM — exemplo local sem serviço (DEC-006)
