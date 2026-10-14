<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Realize uma pesquisa aprofundada sobre estratégias de deployment, release e progressive delivery.

Considere informações e documentação disponíveis até agosto de 2026.
OBJETIVO
Comparar mecanismos usados para implantar e liberar novas versões com diferentes níveis de risco, infraestrutura, controle e capacidade de recuperação.
Investigue:
big bang release;
recreate deployment;
rolling deployment;
blue-green deployment;
canary release;
ring deployment;
phased rollout;
percentage rollout;
dark launch;
shadow traffic;
A/B testing;
feature flags;
kill switches;
release por cliente;
release por região;
release por dispositivo;
deployment desacoplado de release.
Para cada abordagem, explique:
funcionamento;
requisitos;
infraestrutura necessária;
benefícios;
riscos;
custo;
capacidade de rollback;
impacto sobre banco de dados;
impacto sobre estado e sessões;
observabilidade necessária;
adequação a diferentes produtos.
Diferencie claramente:
Estratégia de deployment.
Estratégia de release.
Experimento de produto.
Feature flag.
Controle operacional.
Atualização de software desktop.
Investigue também:
dívida gerada por feature flags;
remoção de flags antigas;
segmentação de usuários;
critérios para interromper rollout;
análise automática de canary;
riscos de diferenças entre grupos;
proteção de dados em shadow traffic.
FORMATO DA RESPOSTA
Entregue:
Resumo executivo.
Matriz comparativa.
Diagramas.
Árvore de decisão.
Exemplos apropriados.
Exemplos inadequados.
Combinação de canary e feature flag.
Estratégia mínima para equipe pequena.
Estratégia para produto crítico.
Estratégia para aplicação desktop.
Antipadrões.
Checklist.
Tabela de fontes e limitações.
Inclua citações junto às afirmações.

Preparei um relatório completo comparando as estratégias de deployment, release e progressive delivery que você pediu — cobrindo desde big bang, recreate, rolling, blue-green e canary até ring deployment, dark launch, shadow traffic, A/B testing, feature flags e kill switches.

Destaques do relatório:

- **Diferenciação central**: deployment (colocar código em produção) vs. release (expor ao usuário) vs. experimento (medir impacto) vs. feature flag (controle runtime) vs. controle operacional (kill switch) vs. atualização desktop (fora do controle direto do vendor).
- **Matriz comparativa** com funcionamento, infraestrutura, rollback, impacto em banco de dados, risco e custo para as 17 abordagens solicitadas.
- **Padrão expand-contract** detalhado para lidar com impacto em banco de dados durante deploys com coexistência de versões — muito relevante dado seu trabalho com Supabase/Firebase.
- **Dívida técnica de feature flags**: causas (nomes ambíguos, ausência de dono, sem data de expiração) e práticas de mitigação (auditorias, PR de remoção criado junto com a flag).
- **Canary analysis automatizada** (Flagger/Kayenta) com fórmulas de tempo de promoção/rollback baseadas em thresholds de métricas.
- **Armadilhas estatísticas de A/B testing** (peeking, sample ratio mismatch, efeitos de novidade).
- Diagramas Mermaid, árvore de decisão, exemplos apropriados/inadequados, estratégias específicas para equipe pequena, produto crítico e aplicação desktop, antipadrões, checklist e tabela de fontes com limitações.
<span style="display:none">[^1_1][^1_10][^1_11][^1_12][^1_13][^1_14][^1_15][^1_16][^1_17][^1_18][^1_19][^1_2][^1_20][^1_21][^1_22][^1_23][^1_24][^1_25][^1_26][^1_27][^1_28][^1_29][^1_3][^1_30][^1_31][^1_32][^1_33][^1_34][^1_35][^1_36][^1_37][^1_38][^1_39][^1_4][^1_40][^1_41][^1_42][^1_43][^1_44][^1_45][^1_46][^1_47][^1_48][^1_49][^1_5][^1_50][^1_51][^1_52][^1_53][^1_54][^1_55][^1_56][^1_57][^1_58][^1_59][^1_6][^1_60][^1_61][^1_62][^1_63][^1_64][^1_65][^1_66][^1_67][^1_68][^1_69][^1_7][^1_70][^1_71][^1_72][^1_73][^1_74][^1_75][^1_8][^1_9]</span>

<div align="center">⁂</div>

[^1_1]: https://launchdarkly.com/docs/guides/infrastructure/deployment-strategies

[^1_2]: https://circleci.com/blog/deployment-strategies-types-trade-offs-and-how-to-choose/

[^1_3]: https://www.linkedin.com/pulse/two-pillars-software-delivery-release-deployment-satya--7zmhc

[^1_4]: https://dev.to/beefedai/safe-deployment-strategies-blue-green-canary-and-rolling-1okk

[^1_5]: https://launchdarkly.com/blog/operational-flags-best-practices/

[^1_6]: https://nemorize.com/roadmaps/debugging-under-pressure/lessons/feature-flags-and-kill-switches

[^1_7]: https://www.statsig.com/perspectives/feature-flag-debt-management

[^1_8]: https://flagshark.com/blog/feature-flag-technical-debt-guide/

[^1_9]: https://launchdarkly.com/blog/3-ways-to-avoid-technical-debt-when-feature-flagging/

[^1_10]: https://www.growthbook.io/blog/engineering-guide-feature-flag-technical-debt

[^1_11]: https://docs.flagger.app/usage/deployment-strategies

[^1_12]: https://woobox.com/articles/ab-testing-sample-size-and-significance

[^1_13]: https://oneuptime.com/blog/post/2026-02-09-canary-analysis-rollback-flagger/view

[^1_14]: https://www.evanmiller.org/how-not-to-run-an-ab-test.html

[^1_15]: https://www.awa-digital.com/blog/ab-testing-mistakes/

[^1_16]: https://docs.getunleash.io/get-started/what-is-a-feature-flag

[^1_17]: https://itsyconnect.com/guides/app-store-phased-release

[^1_18]: https://learn.microsoft.com/en-us/training/modules/implement-test-progressive-exposure-deployment/3-explore-ci-cd-deployment-rings

[^1_19]: https://octopus.com/devops/kubernetes-deployments/kubernetes-deployment-strategies/

[^1_20]: https://www.qovery.com/blog/kubernetes-deployment-strategies

[^1_21]: https://pandastack.io/blog/deployment-strategies-compared

[^1_22]: https://oracles.cloud/blue-green-vs-canary-vs-rolling-deployments-which-release-strategy-fits-your-risk-profile

[^1_23]: https://www.getunleash.io/blog/canary-release-vs-rolling-deployment

[^1_24]: https://testrigor.com/blog/what-is-dark-launch-and-how-to-test-it/

[^1_25]: https://www.growthbook.io/insights/dark-launching

[^1_26]: https://alivedise.github.io/backend-engineering-essentials/cicd-devops/grey-and-dark-releases-and-a-b-testing

[^1_27]: https://medium.com/@nikoo.asadnejad.work/the-ultimate-guide-to-deployment-strategies-blue-green-canary-rolling-and-beyond-a5b916aed693

[^1_28]: https://www.cloudzero.com/blog/deployment-strategies/

[^1_29]: https://thenewstack.io/5-deployment-strategies-the-pros-and-cons/

[^1_30]: https://www.youtube.com/watch?v=Rq46UPLJRUo

[^1_31]: https://www.baeldung.com/ops/deployment-strategies

[^1_32]: https://devtron.ai/blog/top-6-kubernetes-deployment-strategies/

[^1_33]: https://www.groundcover.com/blog/kubernetes-deployment-strategies

[^1_34]: https://blog.theappcode.net/zero-downtime-database-migrations

[^1_35]: http://essay.utwente.nl/92098/1/vanKampen_MA_EEMCS.pdf

[^1_36]: https://www.techinterview.org/post/3233468857/lld-dark-launch/

[^1_37]: https://scalablesystem.dev/system-design/build-a-shadow-traffic--dark-launch-system

[^1_38]: https://mdsanwarhossain.me/blog-traffic-mirroring-dark-launch.html

[^1_39]: https://alivedise.github.io/backend-engineering-essentials/testing/testing-in-production

[^1_40]: https://softwaremodernizationservices.com/blog/dark-launching-new-architecture/

[^1_41]: https://cxl.com/blog/12-ab-split-testing-mistakes-i-see-businesses-make-all-the-time/

[^1_42]: https://www.easy.bi/blog/ab-testing-best-practices/

[^1_43]: https://www.kameleoon.com/blog/data-accuracy-pitfalls-ab-testing

[^1_44]: https://towardsdatascience.com/online-controlled-experiment-8-common-pitfalls-and-solutions-ea4488e5a82e/

[^1_45]: https://devops.com/technical-debt-management-with-feature-flags-strategy-best-practices/

[^1_46]: https://codewisdom.io/blog/programming-basic-programming-thoughts-api-database-synchronization/

[^1_47]: https://www.deployhq.com/blog/database-migration-strategies-for-zero-downtime-deployments-a-step-by-step-guide

[^1_48]: https://matthewpalma.dev/blog/zero-downtime-database-migrations-expand-contract-pattern

[^1_49]: https://dev.to/whoffagents/prisma-migrations-in-production-zero-downtime-deployments-with-expand-contract-2l1p

[^1_50]: https://anhtu.dev/zero-downtime-database-migration-expand-contract-ef-core-batch-backfill-production-1151

[^1_51]: https://www.linkedin.com/posts/danish-mehmud_database-migrations-zerodowntime-activity-7431145182319943680-lHyU

[^1_52]: https://scopeforged.com/blog/database-migration-zero-downtime

[^1_53]: https://www.enolcasielles.com/en/blog/database-migrations-strategy

[^1_54]: https://apxml.com/courses/monitoring-managing-ml-models-production/chapter-4-automated-retraining-updates/advanced-deployment-patterns

[^1_55]: https://spot.rackspace.com/blog/deployment-strategies

[^1_56]: https://developer.harness.io/docs/feature-management-experimentation/admin-best-practices/managing-technical-debt

[^1_57]: https://opensenselabs.com/blog/deployment-release-strategies

[^1_58]: https://dev.to/alaikrm/feature-flags-how-they-prevent-technical-debt-and-how-they-quietly-create-it-3p8f

[^1_59]: https://microsoft.github.io/code-with-engineering-playbook/automated-testing/shadow-testing/

[^1_60]: https://automatic.co/blog/shadow-deployment-guide

[^1_61]: https://www.linkedin.com/posts/harrisonwhitaker_shadowfunnels-growthengineering-plg-activity-7344522337112702977-bfkY

[^1_62]: https://www.sigmoda.com/blog/shadow-evals-for-llm-changes

[^1_63]: https://thebackenddevelopers.substack.com/p/zero-downtime-database-migrations

[^1_64]: https://rafftechnologies.com/learn/guides/zero-downtime-database-migrations-small-teams

[^1_65]: https://www.statsig.com/perspectives/bigger-sample-sizes-reliable-abtests

[^1_66]: https://signalvnoise.com/posts/3004-ab-testing-tech-note-determining-sample-size

[^1_67]: https://blog.analytics-toolkit.com/2017/statistical-significance-ab-testing-complete-guide/

[^1_68]: https://marketingexperiments.com/a-b-testing/why-different-estimates-of-statistical-significance

[^1_69]: https://mcpanalytics.ai/articles/ab-testing-statistical-significance

[^1_70]: https://instapage.com/blog/validating-ab-tests

[^1_71]: https://medium.com/@abhishekpandey229/deployment-strategies-choosing-the-right-approach-for-seamless-releases-78b74d146b00

[^1_72]: https://akvanewsletter.substack.com/p/the-4-kubernetes-deployment-strategies

[^1_73]: https://thenewstack.io/deployment-strategies/

[^1_74]: https://www.youtube.com/watch?v=-0dasPTbdFg

[^1_75]: https://www.linkedin.com/posts/eniolaamiola_infra-opentowork-kubernetes-activity-7376163245142757377-eOsI

