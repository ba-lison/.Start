# Rollback, Roll-forward e Resposta a Incidentes em Releases

## 1. Visão geral e objetivos

Este relatório sintetiza práticas modernas até agosto de 2026 sobre como detectar problemas após uma release, decidir entre rollback e roll-forward (fix-forward), estruturar observabilidade e organizar a resposta operacional a incidentes de deployment/release. O foco é fornecer um material prático: árvore de decisão, checklists, runbook de falha de release, modelos de incidente/post‑mortem e métricas de sucesso, cobrindo também quando o rollback pode ser perigoso devido a migrações, dados, integrações e efeitos colaterais irreversíveis.[^1][^2][^3][^4][^5]


## 2. Conceitos fundamentais

### 2.1 Rollback, roll-forward, redeploy e hotfix

- **Rollback**: reverter para uma versão estável anterior do sistema, normalmente redeployando o último artefato conhecido como bom (container, binário, release anterior). É a opção padrão em muitos times porque é mais rápida e exige menos diagnóstico durante o incidente.[^6][^7][^1]
- **Roll-forward / fix-forward**: manter a versão atual com problema e implantar rapidamente uma correção (novo build/hotfix) por cima, preservando a linha de deployment e o estado de dados.[^7][^3][^1]
- **Redeploy**: redeployar a mesma versão (por exemplo, para mitigar corrupção de pods ou nós) sem mudar código; é útil quando o problema é infra/ambiente e não necessariamente da versão.[^2]
- **Hotfix**: mudança pequena, altamente priorizada, aplicada fora do ciclo normal de release para corrigir um defeito crítico em produção; em muitos fluxos, é um roll-forward com pipeline mais curto e aprovação acelerada.[^8][^7]

### 2.2 Automatic rollback vs manual rollback

- **Automatic rollback**: mecanismo em que o sistema de deployment/progressive delivery reverte sozinho para a versão anterior quando SLIs/SLOs ou métricas de canary são violados, tipicamente durante canary ou rollout progressivo.[^5][^9][^10]
- **Manual rollback**: decisão humana (IC/on-call) acionando o rollback via pipeline ou orquestrador (botão de rollback, `kubectl rollout undo`, ArgoCD sync para revisão anterior, etc.), baseada em análise de impacto e risco.[^3][^11][^1]

### 2.3 Health checks, readiness e liveness

- **Health checks/probes**: diagnósticos automatizados de saúde das aplicações, usados para decidir se instâncias devem receber tráfego ou ser reiniciadas.[^12][^13]
- **Readiness probes**: respondem “este pod está pronto para servir tráfego?”; quando falham, o pod é removido dos endpoints de serviços, sem reinício, permitindo que ele recupere dependências temporariamente indisponíveis.[^14][^13][^12]
- **Liveness probes**: respondem “este pod ainda está vivo?”; quando falham repetidamente, o container é reiniciado para sair de deadlocks, loops infinitos ou estados irrecuperáveis.[^13][^14][^12]


## 3. Observabilidade focada em releases

### 3.1 Logs, métricas, traces e crash reporting

- **Logs estruturados** com campo de versão/release, ambiente, commit SHA e estado de feature flags permitem filtrar rapidamente erros surgidos após um deployment específico e correlá-los com o change set.[^15][^16]
- **Métricas** (golden signals: latência, taxa de erro, tráfego, saturação) e KPIs de negócio (pagamentos, conversões, licenças ativadas) são a base para SLIs/SLOs e canary analysis.[^17][^10][^5]
- **Traces distribuídos** com atributos de versão e deployment markers permitem seguir a jornada de requisições através de serviços, identificando exatamente onde a nova versão introduziu regressão.[^18][^19]
- **Crash/exception reporting** (Sentry, Bugsnag, Rollbar etc.) com tagging de release mostra quais versões introduziram novas exceções ou aumentaram a taxa de crashes, acelerando decisões de rollback.[^16][^20][^21]

### 3.2 Deployment markers e correlação versão‑erro

- **Deployment markers** ou spans de deployment são eventos registrados nos sistemas de observabilidade (APM, tracing, dashboards) contendo versão, horário, ambiente e, idealmente, quem disparou o deploy.[^19][^18][^16]
- Esses marcadores aparecem como linhas verticais em gráficos de erros/latência, permitindo ver de imediato se um spike de erro ou queda de conversão ocorreu logo após uma release ou mudança de feature flag.[^22][^15]
- Ferramentas de correlação de deploys e exceções usam esses markers e tags de versão para apontar qual deployment provavelmente causou um incidente, reduzindo o tempo gasto garimpando logs e diffs.[^23][^20][^21]

### 3.3 SLIs, SLOs e error budget

- **Service Level Indicator (SLI)**: métrica que representa o nível de serviço percebido (por exemplo, proporção de requisições bem-sucedidas, latência abaixo de um limiar).[^24][^17]
- **Service Level Objective (SLO)**: alvo numérico para um SLI, como “97% das requisições da API sem 5xx em 4 semanas” ou “p95 < 200 ms”.[^24][^17]
- **Error budget**: 1 – SLO; exemplo: SLO de 97% dá budget de 3% de requisições com erro em uma janela de tempo, usado para decidir se é aceitável continuar fazendo releases.[^25][^17][^24]
- Políticas de error budget definem quando congelar deploys (por exemplo, ao estourar o budget na janela de 4 semanas, suspender novas releases exceto correções P0 e segurança).[^25][^24]

### 3.4 Synthetic monitoring e Real User Monitoring

- **Synthetic monitoring**: scripts automatizados que simulam jornadas críticas (login, checkout, ativação de licença, upload de desenho CAD) em produção em intervalos regulares, detectando regressões antes de afetar muitos usuários reais.[^9][^26][^10]
- **Real User Monitoring (RUM)**: coleta passiva de dados de desempenho e erros do tráfego real (por exemplo, Web Vitals, taxas de JS error, performance por país/ISP), excelente para dimensionar impacto real e priorizar melhorias.[^26][^10]
- A combinação de sintético (proativo) com RUM (verdade de campo) é considerada prática recomendada para garantir saúde de releases e visualizar o impacto em negócio.[^10][^26]


## 4. Critérios para interromper rollout, rollback e roll-forward

### 4.1 Interromper rollout (pausar ou cancelar)

Em estratégias progressivas (canary, rolling, traffic shifting), pausar/cancelar o rollout é frequentemente o primeiro passo antes de qualquer rollback.[^4][^9][^10]

Critérios típicos para **interromper o rollout**:

- Violação de SLIs chave no canary (taxa de erro, latência p95/p99, saturação) além de thresholds pré‑definidos, mesmo que ainda dentro do error budget global.[^5][^9][^10]
- Degradação relevante de métricas de negócio (queda de conversão, aumento de falhas em pagamento, aumento de cancelamentos) na coorte canary.[^10][^5]
- Falhas em smoke tests pós‑deploy ou monitores sintéticos críticos imediatamente após início do rollout.[^2][^9]
- Observação de crashes, memory leaks ou loops perigosos em crash reporting/tracing no subset canary.[^20][^21]

A pausa possibilita investigar, escalar a zero os pods canary ou ajustar feature flags, sem ainda reverter a versão já estável no restante da frota.[^3][^10]

### 4.2 Critérios para rollback (reverter)

Fontes modernas convergem em que rollback deve ser o **default** quando o incidente é claramente induzido por mudança recente e o rollback é seguro.[^1][^6][^3]

Critérios frequentes para **optar por rollback**:

- **Falha change‑induced**: o início da falha acompanha claramente o horário do deploy/config push/flag flip; gráficos mostram spike de erros ou queda de sucesso logo após o marker de deploy.[^19][^1][^3]
- **Impacto alto**: indisponibilidade total ou quase total, falhas em pagamentos, perda de dados iminente, incidentes de segurança ou interrupção severa de fluxo de trabalho crítico (por exemplo, abertura de desenhos CAD).[^7][^4][^1]
- **Rollback reversível e rápido**: versão anterior ainda deployável, sem migrações irreversíveis, e rollback é uma operação única e confiável (botão ou comando) que leva poucos minutos.[^1][^3]
- **Fix-forward não está pronto**: correção ainda não escrita, não testada ou baseada em hipótese fraca; diagnosticar sob pressão é mais lento e arriscado do que voltar a conhecido bom.[^27][^6][^1]
- **Regressão em caminho crítico**: autenticação, checkout, gravações de dados, licenciamento ou API central; meia-correção via hotfix parcial pode introduzir mais risco que voltar a versão anterior inteira.[^3][^1]

Diretrizes adicionais comuns:

- Se há dúvidas fortes sobre o diagnóstico e o rollback é limpo, priorizar rollback em vez de apostar em fix-forward.[^6][^4]
- Escolher o caminho mais reversível: se rollback é claro e fix-forward envolve múltiplas mudanças, rollback ganha.[^27][^6]

### 4.3 Quando rollback é perigoso

Rollback deixa de ser o default quando reverter a versão pode agravar o incidente, especialmente por causa de **estado e efeitos colaterais irreversíveis**.[^8][^6][^3]

Cenários típicos:

- **Migrações de banco irreversíveis ou forward‑only**: migrations que dropam colunas, mudam tipos de dados, adicionam `NOT NULL` com valores default, ou executam transformações destrutivas; nesse caso, código antigo pode não entender o novo schema ou corromper dados.[^4][^8][^3]
- **Dados em formato novo**: a nova versão grava registros, arquivos, payloads de fila, caches ou blobs em formato que o código antigo não consegue ler (por exemplo, JSON com campos obrigatórios não conhecidos pela versão anterior).[^8][^3]
- **Eventos publicados para terceiros**: a release acabou de publicar eventos ou webhooks que já foram consumidos por sistemas externos; rollback não desfaz eventos entregues, e reprocessar pode criar duplicidade ou inconsistência.[^3][^8]
- **Integrações externas com efeitos colaterais**: chamadas a gateways de pagamento, CRMs, sistemas fiscais ou de assinatura que já provocaram ações irreversíveis em sistemas externos (criação de faturas, cobranças, renovações).[^8][^3]
- **E-mails enviados e notificações**: comunicação já disparada (e‑mails de reset de senha, avisos de cobrança, convites de compartilhamento) não será “desenviada” por rollback, podendo produzir divergência entre estado do sistema e expectativas do usuário.[^3][^8]
- **Pagamentos processados**: transações já autorizadas/capturadas pelo processador; voltar código pode levar a reprocessamentos ou falha em conciliações.[^10][^3]
- **Atualizações de clientes desktop ou plugins**: versões clientes instaladas em massa (por exemplo, plugins CAD, extensões SketchUp) podem ter migrado dados locais (configurações, caches, arquivos) para formato novo; voltar servidor sem considerar esses clientes pode quebrar compatibilidade.[^4][^8]
- **Dados produzidos pela nova versão com semântica diferente**: registros já acumulados com regras de negócio novas (licenças, planos, tipos de documento) podem não ser tratados corretamente pela versão antiga.[^8][^3]

Nesses casos, recomenda‑se extreme caution: evitar rollback “cego” de aplicação sem plano de dados (down migrations, restauração controlada, migração compensatória) e privilegiar roll-forward com mitigação de impacto (desativar feature via flag, canary inverso, etc.).[^4][^3]

### 4.4 Critérios para roll-forward (fix-forward)

Roll-forward é indicado quando rollback é inviável ou mais perigoso, e quando a correção é trivial, segura e bem compreendida.[^6][^1][^3]

Critérios frequentes:

- **Rollback inseguro**: deploy continha migração irreversível, dados já migrados, ou rollback traria de volta vulnerabilidade crítica recém corrigida.[^28][^6][^3]
- **Nada a reverter**: incidente não está ligado a deploy recente (por exemplo, dados ruins, incidente de capacidade, falha em provider externo), então rollback não ajudaria.[^6][^3]
- **Fix-forward trivial e certo**: bug claramente localizado com correção óbvia (por exemplo, variável de ambiente errada, typo em URL, feature flag invertida), já codificado, testado em staging e pronto para deploy.[^1][^6]
- **Impacto baixo e contido**: problema afeta pequena fração de usuários em fluxo não crítico, tornando seguro esperar alguns minutos para fix-forward em vez de reverter toda a release.[^7][^1]
- **Rollback reintroduz problema pior**: versão anterior tem bug grave conhecido; voltar a ela trocaria um problema por outro mais sério ou repetiria incidente anterior.[^28][^1]

Boas práticas incluem definir **critérios de abort** para fix-forward: se o hotfix não resolver em X minutos ou piorar métricas, abandonar a estratégia e retornar para rollback ou outra mitigação.[^27]


## 5. Como medir impacto e identificar a versão causadora

### 5.1 Medindo impacto

Medir impacto é essencial para priorizar respostas e decidir severidade do incidente.[^29][^2]

Eixos comuns:

- **SLIs técnicos**: variação de taxa de erro, latência, throughput, disponibilidade (porcentagem de requisições bem‑sucedidas) em relação ao baseline pré‑deploy.[^30][^17]
- **Métricas de negócio**: quedas em conversão, aumento em falhas de pagamento, redução de sessões ativas, queda em uploads concluídos ou documentos salvos.[^5][^10]
- **Error budget consumido**: percentual do orçamento de erros queimado pelo incidente (por exemplo, uma só falha consumindo 50% do budget de quatro semanas).[^24][^25]
- **Escopo de impacto**: número de usuários afetados, regiões/tenants afetados, serviços dependentes impactados.[^29][^2]

Estas medidas são usadas para classificar severidade (por exemplo, Sev0/Sev1) e para avaliar se a estratégia escolhida (rollback ou roll-forward) está efetivamente recuperando o serviço (queda de erro, retorno de conversão a patamares normais).[^31][^29]

### 5.2 Identificando a versão causadora

Para identificar qual release causou o incidente, as práticas modernas enfatizam **fazer da versão um atributo de primeira classe na telemetria**.[^18][^16]

Mecanismos comuns:

- **Tag de release em erros e traces**: configurando o crash reporting/APM para enviar o identificador de release em cada evento; dashboards podem então agrupar erros por versão e destacar “new issue in release X.Y.Z”.[^21][^23][^20]
- **Deployment markers** em gráficos de métricas, emitidos pela pipeline via API do APM/observabilidade; isso permite correlacionar spikes de erro com o exato momento do deploy.[^15][^16][^19]
- **Logs com campo de versão**: estruturando logs com `app_version`, `commit_sha`, `release_channel`; consultas simples (por exemplo, "erros 5xx onde app_version = nova") revelam regressões pós‑release.[^16][^15]
- **Linha do tempo commit→build→deploy→incidente**: ferramentas de AIOps e SRE combinam diffs de VCS, builds CI, syncs GitOps e métricas de runtime em uma única timeline, priorizando mudanças mais correlacionadas com o sintoma.[^18][^19]


## 6. Automatizar rollback sem criar instabilidade

### 6.1 Conceito de automatic rollback em canary/progressive delivery

Automated rollback é típico em sistemas de progressive delivery com **Automated Canary Analysis (ACA)**.[^9][^5][^10]

- O sistema compara continuamente métricas do canary com o baseline e emite um veredito de deployment; se SLIs canary degradam além de thresholds ou consomem error budget de forma insustentável, a ferramenta dispara rollback automático.[^9][^5]
- Condições típicas incluem limites absolutos (error rate > 1%), degradação relativa (latência +50% vs baseline) e violações de KPIs de negócio.[^5][^10]

### 6.2 Como evitar instabilidade com rollback automático

Para que automações não causem “flapping” (repetição de deploy/rollback), recomenda‑se:

- **Definir janelas de observação adequadas**: aguardar tempo suficiente no canary antes da decisão para evitar agir sobre ruído estatístico ou spikes transitórios.[^10][^5]
- **Usar múltiplas métricas**: combinar taxa de erro, latência e métricas de negócio; evitar rollback por flutuação em sinal único marginal.[^9][^5]
- **Codificar thresholds no contexto do error budget**: disparar rollback quando a taxa de consumo de budget for insustentável, não por qualquer pequena violação momentânea.[^25][^5]
- **Histerese e limites de frequência**: impedir que o mesmo deployment seja promovido/rollbackado várias vezes em sequência, exigindo intervenção humana após certo número de ciclos automáticos.[^5]
- **Fallbacks mais leves**: antes de reverter toda a release, considerar reduzir tráfego do canary a zero ou desativar a feature via flag.[^4][^3]


## 7. Organização da resposta a incidentes de release

### 7.1 Papéis e responsabilidades (Incident Command)

Literatura de SRE e incident management recomenda um modelo de **Incident Command System (ICS)** adaptado para tecnologia.[^31][^29]

Papéis típicos:

- **Incident Commander (IC)**: responsável por decisões táticas, prioridades, escopo, e por manter todos alinhados; é a autoridade clara durante o incidente.

---

## References

1. [Roll Back or Roll Forward? Making the Call During a Deploy Incident](https://irm.nearlunar.com/blog/deploy-incident-rollback-or-forward) - A practical framework for deciding whether to revert a bad deploy or push a hotfix during an active ...

2. [Recommendations for designing a deployment failure ...](https://learn.microsoft.com/en-us/power-platform/well-architected/operational-excellence/mitigation-strategy) - This guide describes the recommendations for designing a standardized strategy to effectively handle...

3. [Rollback or fix forward? An on-call decision playbook](https://intellira.ai/sre-playbooks/rollback-decision-playbook)

4. [Deployment Strategies - Learning Module | OneNoughtOne](https://www.onenoughtone.com/learning-path/system-design-hld/learning/deployment-strategies/5) - Learning Module from System Design (HLD) learning path. Master this topic with interactive learning.

5. [Automated Rollback: Definition & MLOps Guide | Inference Systems](https://inferensys.com/glossary/evaluation-driven-development/production-canary-analysis/automated-rollback) - Automated rollback is a deployment safety mechanism that automatically reverts a software release wh...

6. [Rollback vs Roll-Forward: Deciding Under Pressure](https://devopsaitoolkit.com/blog/rollback-vs-roll-forward-deciding-under-pressure/) - In the middle of an outage, do you revert or fix forward? A senior SRE's decision framework, the que...

7. [Rollback vs Rollforward: Key Differences](https://hokstadconsulting.com/blog/rollback-vs-rollforward-key-differences) - Rollforward: Deploys a fix; slower (30 mins–1 hour) but safer for data integrity. Rollback is better...

8. [The Sultry Art of Rollback vs. Rollforward: A Deployment ...](https://www.sultry.pro/posts/the-sultry-art-of-rollback-vs-rollforward-a-deployment-statecraft-workflow-analysis) - This comprehensive guide delves into the strategic decision-making between rollback and rollforward ...

9. [[BEE-15006] Testing in Production](https://alivedise.github.io/backend-engineering-essentials/testing/testing-in-production) - Backend Engineering Essentials documentation

10. [Testing in Production -- Canary Deployments, Feature Flags, and ...](https://qaskills.sh/blog/testing-in-production-strategies) - Complete guide to testing in production safely. Covers canary deployments, feature flags, synthetic ...

11. [Deployment Rollback Strategies](https://www.linkedin.com/top-content/technology/devops-integration-strategies/deployment-rollback-strategies/) - Deployment rollback strategies are plans and procedures used to quickly undo software updates and br...

12. [Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/concepts/workloads/pods/probes/) - The liveness probe passes when the app itself is healthy, but the readiness probe additionally check...

13. [Kubernetes Health Checks: Types, Configuration & ...](https://spacelift.io/blog/kubernetes-health-check) - Liveness probes allow Kubernetes to detect and restart failed Pods, while readiness probes prevent t...

14. [How to Implement Health Checks That Distinguish ...](https://oneuptime.com/blog/post/2026-02-09-health-checks-liveness-vs-readiness/view) - Liveness probes should have higher failure thresholds to avoid restart loops. Readiness probes can b...

15. [Correlate incidents with releases and feature flags | Aporeon](https://aporeon.com/drills/release-correlation-deploy-markers-feature-flags)

16. [Deployment Monitoring: What to Track, Why It Matters, and ...](https://www.deployhq.com/blog/monitoring-your-deployments-with-deployhq-ensuring-smooth-releases) - Deployment monitoring catches failures before users notice them. Learn what metrics to track, how to...

17. [Chapter 2 - Implementing SLOs](https://sre.google/workbook/implementing-slos/) - Learn to use Service Level Objectives (SLOs) for continuous improvement in reliability and customer ...

18. [The causality chain: from commit to incident](https://intellira.ai/blog/the-causality-chain)

19. [How to Use OpenTelemetry to Correlate Deployments with ...](https://oneuptime.com/blog/post/2026-02-06-correlate-deployments-production-incidents-opentelemetry/view) - Learn how to use OpenTelemetry to correlate deployment events with production incidents for faster r...

20. [Best Exception Software (2026)](https://wifitalents.com/best/exception-software/) - Compare the top 10 Exception Software tools, including Sentry, Rollbar, and Bugsnag, with ranking in...

21. [Release Health Monitoring & Deploy Tracking - LightTrace](https://light-trace.robomiri.com/blog/release-health-monitoring/) - Tag every event with a release to see which deploy introduced a regression. Learn how release health...

22. [Correlating Exceptions with Deployments](https://logz.io/blog/correlating-exceptions-with-deployments/) - Finding the Bug in the Haystack: You can now correlate Exceptions with your deployments on the Logz....

23. [Deployment Tracking - Airbrake](https://www.airbrake.io/deployment-tracking)

24. [Appendix A - Example SLO Document](https://sre.google/workbook/slo-document/) - Example of SLO document detailing SLO for API, HTTP server, and score pipeline with metrics on avail...

25. [Error Budget Policy for Service Reliability](https://sre.google/workbook/error-budget-policy/) - Learn how error budget policy manages SLO misses, balances reliability with features, and addresses ...

26. [Synthetic Monitoring vs Real User Monitoring (RUM)](https://ip-label.com/synthetic-monitoring-vs-real-user-monitoring/) - Understand the differences between synthetic monitoring and real user monitoring (RUM), when to use ...

27. [Incident Response Failure Mode: Fixing Forward With No ...](https://devopsaitoolkit.com/blog/incident-response-fix-forward-without-rollback-plan/) - Shipping a hot fix into a live incident with no way back can turn one outage into two. Spot the no-r...

28. [Deployment Risk Management & Rollback Strategy](https://interviewstack.io/cloud_architect/categories/question-bank/safe-deployment-and-rollback-strategies) - Master your next interview with AI mock interviews and personalized guides. Showcase your skills in ...

29. [Incident Response: Runbooks & Blameless Postmortems](https://beefed.ai/en/incident-response-postmortem-process) - Operational playbooks for incident response, real-time coordination, and blameless postmortems to re...

30. [Prometheus Alerting: Turn SLOs into Alerts](https://sre.google/workbook/alerting-on-slos/) - Turn SLOs into actionable alerts on significant events using Prometheus alerting. Improve precision,...

31. [Runbooks & Operational Docs — Middle Level - Senior Stack](https://senior-stack.uz/Roadmap/Programming/code-craft/documentation/07-runbooks-and-operational-docs/middle/) - At the junior level, a runbook was a recipe for one alert. At the middle level you zoom out to the i...

