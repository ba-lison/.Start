<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Realize uma pesquisa aprofundada sobre migrações de banco de dados, schemas e dados durante releases de software.

Considere fontes e documentação disponíveis até agosto de 2026.
OBJETIVO
Explicar como modificar estruturas e dados sem interromper o sistema, perder informações ou impedir rollback.
Investigue:
schema migrations;
data migrations;
expand and contract;
backward compatibility;
forward compatibility;
zero-downtime migrations;
dual read;
dual write;
backfill;
feature flags;
migrações online;
migrações offline;
transações;
locks;
índices;
backups;
restore;
validação;
versionamento de schema;
ordem de atualização;
múltiplas versões da aplicação;
rollback;
roll-forward;
migrações destrutivas;
retenção temporária de colunas antigas;
testes de migração.
Inclua também cenários com:
banco relacional;
banco NoSQL;
arquivos locais;
formatos de projeto;
configurações persistentes;
caches;
filas e eventos.
Investigue:
Como lançar uma alteração incompatível em etapas.
Como atualizar aplicação e banco sem downtime.
Como lidar com versões antigas da aplicação.
Quando rollback deixa de ser seguro.
Como validar um backfill.
Como recuperar uma migração parcialmente executada.
Como testar com volume realista.
Como tratar mudanças irreversíveis.
Como registrar a versão do schema.
Como documentar dependências entre release e migração.
CRITÉRIOS DE FONTES
Priorize documentação oficial de bancos, literatura de sistemas distribuídos e publicações técnicas reconhecidas.
Para alegações de desempenho ou confiabilidade, apresente contexto e limitações.
FORMATO DA RESPOSTA
Entregue:
Resumo executivo.
Fluxo seguro.
Exemplo expand-migrate-contract.
Exemplo de backfill.
Matriz de riscos.
Critérios de rollback.
Checklist antes da migração.
Checklist durante a migração.
Checklist depois da migração.
Procedimento de recuperação.
Antipadrões.
Tabela de fontes e limitações.
Inclua citações junto às afirmações.

Segue o relatório em markdown com a pesquisa aprofundada sobre migrações de schema e dados sem downtime, cobrindo até fontes de 2026.[^1_1][^1_2]

Ele inclui, na ordem:

- Resumo executivo enfatizando expand/contract (parallel change), zero‑downtime e compatibilidade backward/forward.[^1_3][^1_4][^1_1]
- Um fluxo seguro de migração passo a passo (planejamento, expand, dual‑write, backfill, switch, contract, monitoramento).[^1_2][^1_5][^1_6]
- Exemplo detalhado de expand‑migrate‑contract para renomear coluna em banco relacional, com dual read/write, backfill batelado e horizonte de rollback.[^1_7][^1_8][^1_2]
- Exemplo de backfill com snapshot + CDC, validação com contagens, checksums e invariantes de negócio (e como definir critérios de aprovação/falha).[^1_9][^1_6]
- Matrizes de risco, critérios de rollback vs roll‑forward, e quando rollback deixa de ser seguro (p. ex. após contract ou mudanças irreversíveis).[^1_5][^1_10][^1_9]
- Checklists antes/durante/depois da migração, procedimento de recuperação de migração parcial e padrões para múltiplas versões da aplicação (N/N‑1, blue‑green, canary).[^1_11][^1_10][^1_4]
- Seções específicas para relacional, NoSQL, arquivos/formatos de projeto, configurações persistentes, caches, filas e eventos, todas usando variações de expand/contract, versionamento de schema e compatibilidade.[^1_12][^1_13][^1_10]
- Uma tabela de fontes e limitações, destacando onde há dependência de engine (PostgreSQL `CREATE INDEX CONCURRENTLY`, Vitess/gh‑ost/pt‑osc, etc.).[^1_14][^1_15][^1_16]

Se quiser, posso agora extrair desse relatório um “playbook enxuto” em formato mais operacional (por exemplo, só o fluxo seguro + checklists) adaptado para o teu contexto de CAD/BIM, licenciamento e formatos de projeto.
<span style="display:none">[^1_17][^1_18][^1_19][^1_20][^1_21][^1_22][^1_23][^1_24][^1_25][^1_26][^1_27][^1_28][^1_29][^1_30]</span>

<div align="center">⁂</div>

[^1_1]: https://www.harness.io/blog/zero-downtime-database-migrations-safe-schema-changes

[^1_2]: https://palakorn.com/blog/zero-downtime-database-migrations/

[^1_3]: https://martinfowler.com/bliki/ParallelChange.html

[^1_4]: https://alivedise.github.io/backend-engineering-essentials/distributed-systems/zero-downtime-schema-migrations

[^1_5]: https://vitess.io/docs/archive/18.0/user-guides/schema-changes/

[^1_6]: https://www.postgresql.org/docs/current/sql-createindex.html

[^1_7]: https://www.bookstack.cn/read/vitess-17.0-en/923381f9e0f7866a.md?wd=Percona

[^1_8]: https://www.designgurus.io/answers/detail/how-do-you-plan-zerodowntime-data-migrations-and-backfills

[^1_9]: https://matthewpalma.dev/blog/zero-downtime-database-migrations-expand-contract-pattern

[^1_10]: https://woyable.com/en/posts/zero-downtime-schema-migrations

[^1_11]: https://www.tim-wellhausen.de/papers/ExpandAndContract/ExpandAndContract.html

[^1_12]: https://aipatternbook.com/parallel-change

[^1_13]: https://appmaster.io/blog/blue-green-canary-api-db-migrations

[^1_14]: https://fordelstudios.com/research/database-migration-zero-downtime

[^1_15]: https://github.com/vitessio/vitess/pull/6547

[^1_16]: https://anhtu.dev/zero-downtime-database-migration-expand-contract-ef-core-batch-backfill-production-1151

[^1_17]: https://www.percona.com/sites/default/files/presentations/Revertible, recoverable schema migrations in Vitess.pdf

[^1_18]: https://scopeforged.com/blog/database-migration-zero-downtime

[^1_19]: https://www.enolcasielles.com/en/blog/database-migrations-strategy

[^1_20]: https://rajivonai.com/blog/2024-07-29-database-migration-cutover-workflow-dual-writes-cdc-backfill-freeze-and-rollback/

[^1_21]: https://thebackenddevelopers.substack.com/p/zero-downtime-database-migrations

[^1_22]: https://snowmanlabs.com/insights/modernize-legacy-systems-without-downtime

[^1_23]: https://www.abstractalgorithms.dev/zero-downtime-schema-migrations-expand-contract

[^1_24]: https://softwarepatternslexicon.com/102/3/23/

[^1_25]: https://thearchitectsnotebook.substack.com/p/ep-116-zero-downtime-data-architecture

[^1_26]: https://www.zero-downtime-schema.com/zero-downtime-schema-evolution-patterns/expand-and-contract-methodology/

[^1_27]: https://medium.com/@k2lu/expand-and-contract-key-to-zero-downtime-database-migration-13aa9ab49e87

[^1_28]: https://www.youtube.com/watch?v=ONSCQWLD9d0

[^1_29]: https://www.diva-portal.org/smash/get/diva2:1931289/FULLTEXT01.pdf

[^1_30]: https://www.linkedin.com/posts/danish-mehmud_database-migrations-zerodowntime-activity-7431145182319943680-lHyU

