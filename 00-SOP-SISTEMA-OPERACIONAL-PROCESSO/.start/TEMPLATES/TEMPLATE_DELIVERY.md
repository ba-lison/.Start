# DELIVERY — [PROJETO]

> propósito: como este projeto é verificado, versionado e entregue (comandos reais)
> ler quando: ao verificar, buildar ou entregar
> atualizar quando: comando, pipeline, ambiente ou forma de entrega mudou (sobrescrever)
> perfil: SIMPLE | STANDARD | CRITICAL
> atualizado_em: YYYY-MM-DD

## Produto e destino

- tipo: biblioteca | CLI | desktop | plugin | web | API | mobile
- destinatário:
- forma de entrega:

## Comandos canônicos

| Ação | Comando |
| :--- | :--- |
| Setup | `[comando]` |
| Lint/tipos | `[comando | N/A + motivo]` |
| Testes | `[comando]` |
| Build limpa | `[comando]` |
| Pacote | `[comando | N/A]` |
| Smoke | `[comando ou procedimento]` |

Comando que ainda não foi conferido: marcar `hipótese:`.

## Git e CI

Valores de `.memory/PROJECT_GUARDRAILS.md`:

- commit local: GP-06
- push, PR, merge, tag remota e deploy: GP-07
- branch ou worktree por Task: GP-08
- formato de commit: GP-09
- branch principal e proteção:
- workflow de CI e eventos:
- gates:
- lock `.memory/_ACTIVE_TASK.lock`: usado | não usado

## Versão e artefato

- política de versão:
- momento do bump e da tag:
- nome e local do artefato:
- commit e SHA-256: obrigatórios
- retenção, assinatura e SBOM:

## Ambientes e configuração

- ambientes:
- segredos e configuração (só nomes de variáveis, nunca valores):
- dados de teste:
- autoridade de promoção:

## Entrega ou deploy

- procedimento:
- homologação:
- verificação pós-entrega:
- janela de observação:
- critério de sucesso:

## Recuperação e incidentes

- rollback ou roll-forward:
- dados e migrações:
- critério de acionamento:
- responsável:

## Segurança e métricas

- scans e controles:
- métricas aplicáveis:
- itens N/A e justificativas:
