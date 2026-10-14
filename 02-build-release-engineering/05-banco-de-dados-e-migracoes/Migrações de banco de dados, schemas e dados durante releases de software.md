# Migrações de banco de dados, schemas e dados durante releases de software sem downtime

## 1. Resumo executivo

Migrações seguras de schema e dados em produção exigem tratar cada mudança incompatível como uma sequência de passos pequenos, reversíveis e compatíveis com múltiplas versões da aplicação, em vez de um único `ALTER` ou script monolítico. O padrão expand/contract (parallel change) organiza essas mudanças em fases de expandir, migrar (dual read/write, backfill) e contrair, permitindo zero‑downtime ou janelas mínimas mesmo em tabelas grandes.[^1][^2][^3][^4]

Zero‑downtime não depende apenas de ferramentas de DDL online (como `CREATE INDEX CONCURRENTLY`, gh‑ost, pt‑online‑schema‑change, Vitess Online DDL), mas principalmente de compatibilidade entre schema e código, ordem de deploy e disciplina de validação e rollback. A mesma filosofia vale para NoSQL, formatos de arquivos, configurações persistentes, caches, filas e contratos de eventos, onde a evolução é guiada por versionamento e compatibilidade backward/forward.[^5][^6][^7]

Este documento apresenta um fluxo seguro de migração, um exemplo detalhado de expand‑migrate‑contract, um exemplo de backfill com validação, matrizes de risco e critérios de rollback, além de checklists práticos pré/durante/pós‑migração e antipadrões comuns.

## 2. Conceitos básicos

### 2.1 Schema migrations vs data migrations

- **Schema migrations**: alterações estruturais em bancos, como criar/alterar tabelas, colunas, índices, constraints e tipos.[^4][^1]
- **Data migrations**: alterações no conteúdo, por exemplo normalizar um campo, recomputar flags, mover dados entre tabelas ou sistemas.[^2][^8]

Na prática, releases não triviais combinam ambos: primeiro tornam o schema mais capaz (expand), depois migram e validam os dados, e finalmente removem estruturas antigas (contract).[^9][^10]

### 2.2 Compatibilidade backward e forward

- **Backward compatibility**: uma nova versão da aplicação funciona com schemas/dados antigos, sem exigir que tudo seja migrado imediatamente.[^3][^4]
- **Forward compatibility**: versões antigas da aplicação continuam funcionando após alterações de schema, desde que estas sejam apenas aditivas ou tolerem campos extras/nulos.[^11][^12]

O padrão parallel change exige que cada fase seja compatível com pelo menos uma versão em execução (N ou N‑1), permitindo deploys e rollbacks não coordenados entre app e banco.[^3][^4]

## 3. Expand and contract / Parallel change

### 3.1 Definição do padrão

Parallel Change (expand/contract) é um padrão para aplicar mudanças quebradoras em interfaces e schemas sem downtime, dividindo o trabalho em expandir, migrar clientes/dados e contrair. Na fase expand, a interface é ampliada para suportar formas antiga e nova; na fase migrate, os consumidores são movidos gradualmente; na fase contract, o formato antigo é removido quando não há mais dependências.[^12][^11][^3]

Em bancos relacionais, isso significa adicionar colunas/tabelas novas ao lado das antigas, manter versões da aplicação que leem/escrevem ambas (dual write/read) e só dropar o antigo após o horizonte de rollback.[^9][^2]

### 3.2 Fases típicas (5 passos)

Uma decomposição prática de expand/contract para schema é:[^10][^2]

1. **Expand**: adicionar nova estrutura (coluna, tabela, índice) de forma puramente aditiva, tipicamente como nullable ou com default seguro.
2. **Dual‑write**: liberar código que escreve nos formatos antigo e novo, enquanto lê preferencialmente do antigo.
3. **Backfill**: executar um job em background que copia dados antigos para a nova estrutura em batches, com throttling e checkpoints.
4. **Switch reads**: mudar a leitura para a nova estrutura (às vezes com shadow/diff) mantendo dual‑write para rollback rápido.
5. **Contract**: remover escrita e leitura da estrutura antiga no código, e por fim dropar colunas/tabelas antigas.

Durante todos os passos 1–4 há um caminho de rollback relativamente simples; após o contract, rollback costuma exigir migrar dados de volta ou restaurar de backup, deixando de ser trivial.[^2][^10]

## 4. Zero‑downtime migrations

### 4.1 Migrações online vs offline

- **Migrações offline**: exigem janela de manutenção, normalmente com bloqueio de escrita ou parada da aplicação; aceitáveis para sistemas internos de baixo SLA ou pequenas bases.[^13][^4]
- **Migrações online**: mantêm o sistema atendendo requisições, com eventuais degradações controladas; envolvem DDL online, expand/contract, backfills batelados e controles de feature flag e tráfego.[^14][^1]

Ferramentas como Vitess Online DDL, gh‑ost e pt‑online‑schema‑change clonam a tabela para um formato novo e aplicam mudanças de forma incremental, minimizando locks no MySQL.[^7][^15][^5]

### 4.2 Transações, locks e índices

Alterações de schema podem adquirir locks que bloqueiam leitura e/ou escrita, causando downtime mesmo sem parar a aplicação.[^6][^4]

- No PostgreSQL, `CREATE INDEX` normal bloqueia escrituras enquanto o índice é construído, enquanto `CREATE INDEX CONCURRENTLY` permite writes à custa de mais tempo e I/O.[^6]
- Em bancos que não suportam DDL online avançado, ferramentas externas (gh‑ost, pt‑osc) ou estratégias por réplicas/blue‑green são necessárias.[^5][^7]

Transações longas de backfill podem aumentar locks, gerar bloat e pressionar replicação; a prática recomendada é usar batches pequenos, commits frequentes, lock_timeout curtos e evitar `ALTER TABLE` que reescrevem tabelas inteiras em uma tacada.[^10][^2]

### 4.3 Backups, restore e ponto de recuperação

Antes de migrações destrutivas ou cutovers de grande impacto, é essencial ter backup recente ou snapshot com ponto de recuperação (PITR).[^16][^14]

- Bancos como PostgreSQL, MySQL e SQL Server suportam backups lógicos e físicos com recovery point objective configurável.[^16][^5]
- Em MySQL gerenciado via Vitess com VReplication, o estado da migração é rastreado em tabelas internas (_vt.vreplication, _vt.copy_state) possibilitando reversão para um GTID salvo.[^17][^7]

A estratégia deve considerar quanto dado é aceitável perder (RPO) e quanto tempo é aceitável de indisponibilidade em um restore (RTO).

## 5. Fluxo seguro de migração (alto nível)

Um fluxo seguro de mudanças incompatíveis, combinando schema e dados, pode ser descrito assim:[^18][^8][^4]

1. **Planejamento e modelagem**
   - Mapear mudanças, dependências entre serviços, queries, jobs e integrações.
   - Decidir se é necessária migração online, blue‑green, canary e/ou expand/contract.
2. **Projeto da compatibilidade**
   - Garantir que o novo schema seja compatível com app N e, tanto quanto possível, com app N‑1 (regra N/N‑1).[^19][^4]
   - Definir estratégia de dual‑write/dual‑read ou triggers em banco.
3. **Preparação de ambiente e testes**
   - Reproduzir volume de produção em staging ou ambiente sintético representativo.[^8][^18]
   - Validar migrações de schema e backfills em carga realista, medindo tempo, locks e impacto.
4. **Expand (schema aditivo)**
   - Executar migrações apenas aditivas (colunas/tabelas novas, índices concurrent/online, constraints NOT VALID etc.).[^18][^9]
5. **Deploy de código compatível**
   - Liberar versão da aplicação que entende o novo schema, faz dual‑write quando necessário e tolera dados parcialmente migrados.[^4][^2]
6. **Backfill e sincronização**
   - Executar backfill por batches com checkpoints, validando contagens, checksums e invariantes de negócio.[^20][^2]
7. **Switch de leitura e features**
   - Ativar feature flags gradualmente para ler da nova estrutura ou do novo sistema, com shadow reads/dual reads para comparação.[^21][^8]
8. **Contract (limpeza)**
   - Depois de um horizonte de rollback (dias/semanas) sem erros, remover código e schema antigos.[^22][^2]
9. **Monitoramento e documentação**
   - Manter métricas, alarmes e documentação de versão de schema e dependências release ↔ migração.

## 6. Exemplo expand‑migrate‑contract (relacional)

### 6.1 Cenário: renomear coluna `email` → `email_address`

Suponha uma tabela `users(id, email)` usada intensamente por múltiplos serviços e jobs. A meta é renomear o campo para `email_address` sem downtime, permitindo rollback até certo ponto.[^23][^2]

#### Fase 1 – Expand (schema aditivo)

- Adicionar nova coluna, inicialmente nullable, e manter a antiga:
  - `ALTER TABLE users ADD COLUMN email_address VARCHAR(320) NULL;`
- Criar índices ou constraints necessárias de forma online/concurrent se o banco suportar.
- Nenhuma mudança de código ainda; versão atual continua funcionando.[^9][^18][^6]

#### Fase 2 – Dual‑write + backfill

- Deploy da versão N da aplicação com lógica:
  - On write: escrever em `email` e `email_address` (dual write).
  - On read: continuar lendo de `email` (nova coluna é apenas sombra).[^21][^2]
- Iniciar job de backfill que preenche `email_address` a partir de `email` para linhas antigas:
  - Iterar por `id` em batches (por exemplo 1k–10k), com pausas e métricas para não saturar I/O ou causar replication lag.[^8][^10]
- O job deve ser idempotente (pode reexecutar o mesmo batch) e registrar high‑water marks e contagens.[^20][^2]

#### Fase 3 – Switch de leitura

- Deploy da versão N+1 que lê preferencialmente `email_address`, mas ainda escreve ambos os campos para permitir rollback de código.[^23][^2]
- Usar feature flag para mudar gradualmente rotas da API ou serviços que leem esse campo, com shadow reads comparando `email` e `email_address` para detectar divergências.[^21][^8]

#### Fase 4 – Contract (limpeza)

- Após:
  - backfill 100% concluído,
  - métricas de divergência zeradas,
  - período de cooling/rollback (por exemplo 1–2 sprints) sem regressões,

  então:

  - Deploy da versão N+2 que só escreve/usa `email_address`.
  - Rodar migração destrutiva em janela controlada:
    - `ALTER TABLE users DROP COLUMN email;` (online ou com estratégia por réplicas).

- A partir desse ponto, rollback de código para versões que esperam `email` já não é seguro sem migração reversa ou restore.[^2][^10]

## 7. Exemplo de backfill com validação

### 7.1 Estratégia de backfill por snapshot + CDC

Para migrações entre bancos ou grandes transformações, o padrão é combinar snapshot inicial, backfill incremental e CDC (Change Data Capture).[^20][^8]

1. Capturar posição de log (LSN/GTID) da origem.
2. Executar backfill até esse ponto, em batches, para copiar histórico.[^20]
3. Usar CDC para aplicar mudanças em tempo real pós‑snapshot.
4. Validar origem/destino continuamente até a divergência aceitável ser zero ou dentro de thresholds definidos.[^8][^20]

### 7.2 Técnicas de validação

- **Contagem de linhas** por tabela/partição.
- **Checksums por faixa de chave** (por exemplo, hash de linhas por intervalo de `id`).[^8][^20]
- **Invariantes de negócio** (tudo que deve ser impossível na modelagem) como:
  - faturas pagas sempre com lançamentos no razão;
  - nenhuma ordem em estado inválido.
- **Dual reads** na aplicação para pequena percentagem de requests, comparando respostas de origem e destino (shadow reads).[^21][^8]

Backfills sem validação podem “passar” mesmo com dados de negócio incorretos; a validação precisa ir além de contagens brutas.[^20][^8]

## 8. Vários tipos de armazenamento

### 8.1 Bancos relacionais

Em bancos SQL (PostgreSQL, MySQL, SQL Server, Oracle), os principais desafios são lock de DDL, integridade referencial e migrações destrutivas em tabelas grandes.[^5][^4][^6]

Boas práticas incluem:
- usar DDL online quando possível (`CREATE INDEX CONCURRENTLY`, `ONLINE=ON`, `INSTANT` DDL);[^7][^16][^6]
- evitar renomear/dropar colunas diretamente, preferindo expand/contract;
- usar constraints em modo não validado seguidas de validação assíncrona (PostgreSQL `NOT VALID`);[^18][^9]
- executar backfills via jobs externos ou workers, não acoplar à pipeline de deploy.[^2][^5]

### 8.2 Bancos NoSQL e documentos

No NoSQL, esquemas são mais flexíveis, mas o custo de versões convivendo por longos períodos aumenta complexidade de código.[^24]

Padrões comuns:
- **Lazy migration**: documentos são migrados na leitura se `schemaVersion` < atual, com gravação da nova versão.[^24]
- **Bulk backfill** para campos críticos, combinado com validações por versão e invariantes de domínio.[^24][^8]

Versionar documentos e eventos explicitamente facilita a convivência de múltiplas versões de app e consumidores.[^13][^24]

### 8.3 Arquivos locais e formatos de projeto

Para formatos de arquivo (por exemplo projetos CAD, BIM, configs em JSON/YAML), o padrão paralelo também se aplica:[^11][^12]

- associar versão de schema/format no próprio arquivo;
- leitores precisam ser tolerantes a campos desconhecidos (forward compatibilidade) e fornecer migrações em memória para versões antigas (backward compatibilidade);
- migrações massivas podem ser feitas offline (batch de conversão) ou lazy na abertura do arquivo, salvando já no novo formato.

### 8.4 Configurações persistentes e feature flags

Config persistente e feature flags são parte essencial de migrações online:[^13][^8]

- mudanças incompatíveis devem ser guardadas atrás de flags, permitindo habilitar gradualmente por coortes de usuários;
- a configuração deve ser versionada, com possibilidade de rollback rápido (por exemplo, reverter um toggle em segundos);
- mudanças no schema de config (p. ex. adicionar novos campos obrigatórios) seguem expand/contract, mantendo defaults e tolerância a ausências.

### 8.5 Caches, filas e eventos

- **Caches**: geralmente são derivados e podem ser invalidados ou reconstruídos após migrações; porém, chaves e formatos de valor devem ser versionados para evitar colisões entre versões de app.[^13][^8]
- **Filas e eventos**: event schemas evoluem com compatibilidade, usando versionamento de payload e consumidores tolerantes a campos extras/faltantes.[^13]
  - mudanças irreversíveis em eventos exigem producers capazes de emitir múltiplas versões de payload durante a janela de transição.[^25][^12]

## 9. Várias versões da aplicação e ordem de atualização

### 9.1 Regra N/N‑1 e clientes legados

A regra N/N‑1 recomenda que o schema seja suportado por pelo menos duas versões consecutivas do serviço, permitindo rollbacks e deploys progressivos sem coordenação rígida.
Isso é crítico em ambientes com mobile apps que atualizam lentamente.[^19][^13]

A sequência típica segura é:
1. expand schema;
2. deploy app compatível (aceitando old/new formas, dual‑write se necessário);
3. backfill;
4. migrar consumidores gradualmente;
5. contract schema.[^4][^13]

### 9.2 Blue‑green e canary deployments com banco

- **Blue‑green**: dois ambientes completos (blue e green); o banco pode ser compartilhado ou duplicado com replicação; a troca de tráfego é um switch.[^13]
- **Canary**: uma pequena fração de tráfego roda em nova versão primeiro, aumentando gradualmente.[^13]

Em ambos os casos, é necessário que o schema seja compatível com old e new app; se a mudança é quebradora (drop/rename direto), nem blue‑green nem canary evitam indisponibilidade ou rollback complexo.[^22][^13]

## 10. Quando rollback deixa de ser seguro

Rollback deixa de ser trivial quando:

- o schema foi **contraído** (colunas/tabelas antigas droppadas) e versões anteriores do app ainda esperam esses elementos;[^10][^2]
- houve **mudanças irreversíveis** de dados (por exemplo, truncar históricos, agrupar registros sem manter rastreabilidade, criptografia sem chave antiga disponível);[^8][^20]
- novas escritas foram feitas apenas no destino (novo banco) sem estratégia de retro‑replicação para a origem.[^20]

Nessas situações, o caminho seguro costuma ser **roll‑forward**: corrigir o problema em uma nova versão ou migração, possivelmente com scripts de reparo de dados, em vez de tentar voltar o mundo ao estado anterior.[^8][^20]

## 11. Como recuperar migração parcialmente executada

### 11.1 Migrações de schema interrompidas

Ferramentas de online DDL como Vitess, gh‑ost e pt‑osc rastreiam estado da migração e permitem cancelamento, retry ou revert.[^17][^7][^5]

- Em caso de falha, a recomendação usual é dropar objetos intermediários (índices inválidos, tabelas ghost) e reexecutar a migração, ou usar comandos de retry/revert quando suportados.[^6][^7]

### 11.2 Backfills parciais

Para backfills e data migrations:

- jobs devem ser idempotentes e registrar checkpoints (por exemplo, maior `id` migrado, contagens, checksums);[^2][^20]
- na retomada, reler metadados de progresso e continuar de onde parou;
- em caso de dúvida, reprocessar intervalos pequenos em modo re‑write (com upsert) para garantir consistência.[^20][^8]

Tratar migrações como máquinas de estado com critérios de entrada, saída e rollback para cada fase reduz o risco de “meio caminho” incoerente.

---

## References

1. [Zero Downtime Database Migrations: Safe Schema Changes](https://www.harness.io/blog/zero-downtime-database-migrations-safe-schema-changes) - Learn zero downtime database migration strategies using backward-compatible schema changes, dual wri...

2. [Zero-Downtime Database Migrations: The Expand/Contract ...](https://palakorn.com/blog/zero-downtime-database-migrations/) - How to change a production schema without taking traffic down — expand/contract, backfills, dual-wri...

3. [Parallel Change](https://martinfowler.com/bliki/ParallelChange.html) - Parallel change, also known as expand and contract, is a pattern to ... changes can be introduced in...

4. [[BEE-19039] Zero-Downtime Schema Migrations](https://alivedise.github.io/backend-engineering-essentials/distributed-systems/zero-downtime-schema-migrations) - Backend Engineering Essentials documentation

5. [Vitess | Making Schema Changes](https://vitess.io/docs/archive/18.0/user-guides/schema-changes/)

6. [Documentation: 18: CREATE INDEX](https://www.postgresql.org/docs/current/sql-createindex.html) - PostgreSQL supports building indexes without locking out writes. This method is invoked by specifyin...

7. [Online DDL strategies - 《Vitess v17.0 Documentation》](https://www.bookstack.cn/read/vitess-17.0-en/923381f9e0f7866a.md?wd=Percona) - Specifying a DDL strategyVia vtctlclientVia VTGateChoosing a DDL strategyvitessgh-ostUsing pt-online...

8. [How do you plan zero‑downtime data migrations and backfills?](https://www.designgurus.io/answers/detail/how-do-you-plan-zerodowntime-data-migrations-and-backfills) - Learn a complete playbook for zero downtime data migrations and backfills using snapshot plus CDC, d...

9. [Zero-downtime database migrations: the expand-contract ...](https://matthewpalma.dev/blog/zero-downtime-database-migrations-expand-contract-pattern) - Ship relational schema changes without maintenance windows using expand-contract phases, backfills, ...

10. [Zero-Downtime Schema Migrations: The Expand-Contract Pattern](https://woyable.com/en/posts/zero-downtime-schema-migrations) - One ALTER TABLE locked a production table for minutes. Here's the expand-contract pattern, batched b...

11. [Expand and Contract - A Pattern to Apply Breaking ...](https://www.tim-wellhausen.de/papers/ExpandAndContract/ExpandAndContract.html) - The Expand and Contract pattern provides a way to implement breaking changes to a system in a safe m...

12. [Parallel Change](https://aipatternbook.com/parallel-change) - Contract – a parallel change is a disciplined way to evolve a contract without breaking it. · Interf...

13. [Blue-green vs canary deployments: safer API and DB changes](https://appmaster.io/blog/blue-green-canary-api-db-migrations) - Blue-green vs canary deployments explained for API and database changes, with practical steps to red...

14. [Zero-Downtime Database Migrations: The Complete Playbook](https://fordelstudios.com/research/database-migration-zero-downtime) - Every schema migration is a risk. The expand-contract pattern eliminates the risk by making the chan...

15. [Experimental: automated, scheduled, dependency free online DDL via gh-ost/pt-online-schema-change by shlomi-noach · Pull Request #6547 · vitessio/vitess](https://github.com/vitessio/vitess/pull/6547) - This PR (work in progress) introduces zero dependency online schema changes with gh-ost/pt-online-sc...

16. [Zero-Downtime Database Migration: Expand-Contract, EF Core ...](https://anhtu.dev/zero-downtime-database-migration-expand-contract-ef-core-batch-backfill-production-1151) - Zero-downtime database migration strategies for .NET 10 and SQL Server: Expand-Contract pattern, EF ...

17. [Revertible, recoverable schema migrations in Vitess](https://www.percona.com/sites/default/files/presentations/Revertible,%20recoverable%20schema%20migrations%20in%20Vitess.pdf)

18. [Zero-Downtime Database Migrations: Expand-Contract Pattern](https://scopeforged.com/blog/database-migration-zero-downtime) - Learn the expand-contract pattern for safe database schema changes. Rename columns, add NOT NULL con...

19. [Database Migrations. The Expand-Contract Pattern](https://www.enolcasielles.com/en/blog/database-migrations-strategy) - In this article I share my experience on how I manage database migrations in high-availability envir...

20. [Database Migration Cutover Workflow: Dual Writes, CDC ...](https://rajivonai.com/blog/2024-07-29-database-migration-cutover-workflow-dual-writes-cdc-backfill-freeze-and-rollback/) - Database migration cutover using dual writes, CDC, backfill, and freeze phases — with rollback bound...

21. [Zero-Downtime Database Migrations: Expand/Contract, Triggers, and ...](https://thebackenddevelopers.substack.com/p/zero-downtime-database-migrations) - The Day I Learned That “ALTER TABLE” Is a Four-Letter Word

22. [How to Modernize Legacy Systems Without Downtime](https://snowmanlabs.com/insights/modernize-legacy-systems-without-downtime) - Modernize legacy systems without downtime using expand-contract migrations, CDC replication, shadow ...

23. [Zero-Downtime Schema Migrations: The Expand-Contract Pattern ...](https://www.abstractalgorithms.dev/zero-downtime-schema-migrations-expand-contract) - TLDR: Altering database schemas in a high-traffic production system without downtime requires the Ex...

24. [Lazy Migration](https://softwarepatternslexicon.com/102/3/23/) - Updating documents in a NoSQL database to a new schema version lazily, i.e., as they are accessed.

25. [Ep #116: Zero-Downtime Data Architecture (Part 2)](https://thearchitectsnotebook.substack.com/p/ep-116-zero-downtime-data-architecture) - To achieve this, we use the most critical pattern in Zero-Downtime Data Architecture: The Expand and...

