# Métricas de Engenharia: DORA, CI/CD, Recuperação e Maturidade sem Gaming

## 1. Resumo executivo

Este relatório apresenta um catálogo estruturado de métricas para avaliar velocidade, qualidade, estabilidade, segurança operacional e experiência do desenvolvedor, com foco em times de engenharia de software que utilizam práticas modernas de CI/CD. As métricas se baseiam em pesquisas DORA originais, literatura de SRE/DevOps, estudos de produtividade e experiência de desenvolvedor, sempre ressaltando limitações, riscos de Goodhart’s Law e gaming de indicadores.[^1][^2][^3][^4][^5]

A pesquisa mostra que os cinco indicadores DORA (frequência de deploy, lead time para mudanças, taxa de falha de mudanças, tempo de recuperação de deploys com falha e taxa de retrabalho) são bons preditores de desempenho organizacional, mas foram obtidos por surveys observacionais não experimentais, com amostras enviesadas por auto‑seleção e uso de cluster analysis — logo, as correlações não são garantias causais. Métricas de CI (build duration, queue time, pipeline success rate, flaky test rate) e saúde de release (rollback, hotfix, escaped defects, crash‑free sessions, release adoption) complementam o quadro ao conectar fluxo de entrega com confiabilidade percebida pelos usuários.[^6][^7][^2][^8][^4][^9][^1]

O relatório propõe um conjunto mínimo para começar (Deployment Frequency, Lead Time for Changes, Failed Deployment Recovery Time, Change Failure Rate, Flaky Test Rate, Build Duration, Toil e Satisfação do Desenvolvedor) e um modelo de maturidade em cinco níveis, indo de “Ad hoc orientado a heróis” até “Otimizado por outcomes e experiência do desenvolvedor”. São discutidos riscos de métricas de vaidade, comparações indevidas entre equipes e burocracia (por exemplo, coleta manual de dados de pipeline sem uso efetivo), além de estratégias para mitigar Goodhart’s Law, como uso de pares de métricas em tensão, foco em times (não indivíduos) e combinação de dados quantitativos com feedback qualitativo.[^7][^3][^10][^11][^12][^13][^1]


## 2. Fundamentos: DORA, metodologia e limitações

### 2.1 O que é o programa DORA

DORA (DevOps Research and Assessment) é um programa de pesquisa iniciado em 2014 para entender capacidades técnicas e culturais que impulsionam desempenho em entrega de software e operação de serviços. Os relatórios Accelerate / State of DevOps consolidaram quatro métricas originais (Deployment Frequency, Lead Time for Changes, Mean Time to Recovery, Change Failure Rate), expandidas em 2024/2025 para incluir Failed Deployment Recovery Time e Deployment Rework Rate como quinto indicador.[^14][^15][^2][^1][^6]

Essas métricas são agrupadas em dois fatores: throughput (Deployment Frequency, Lead Time for Changes, Failed Deployment Recovery Time) e estabilidade/instabilidade (Change Failure Rate, Deployment Rework Rate). A literatura subsequente posiciona as métricas DORA como leading indicators de performance organizacional (lucro, crescimento, satisfação do cliente), mas enfatiza que são proxies e não substitutos de outcomes de negócio.[^2][^6][^14]

### 2.2 Metodologia dos relatórios DORA

Os relatórios DORA utilizam surveys anuais com milhares de profissionais de tecnologia, combinando perguntas sobre práticas (CI/CD, automação de testes, cultura) com auto‑avaliações de desempenho tecnológico e organizacional. O desenho é transversal (cross‑sectional) inferencial‑preditivo, não experimental: mede‑se correlações entre práticas/ capacitações e perfis de desempenho, sem manipulação de variáveis em experimentos controlados.[^4][^16]

Amostras são obtidas via canais orgânicos (listas de email, redes sociais, blog posts) e painéis pagos, com snowball sampling, o que significa que os respondentes não constituem uma amostra aleatória da população global de equipes de software. O relatório de 2021, por exemplo, cita mais de 32 mil respostas acumuladas ao longo de sete anos, com cerca de 1.200 respondentes na edição de 2021, cobrindo múltiplos setores e tamanhos de organização. Modelos estatísticos como análise fatorial e cluster analysis são usados para agrupar organizações em perfis de desempenho (elite, alto, médio, baixo) com base nas quatro métricas principais.[^16][^2][^4]

### 2.3 Limitações e cuidados de interpretação

A própria equipe DORA enfatiza que se trata de pesquisa observacional com auto‑relato: não é possível inferir causalidade estrita entre “melhorar uma métrica DORA” e “aumentar receita”, apenas há associação consistente entre boas práticas, bons resultados em DORA e melhores outcomes organizacionais. O uso de auto‑seleção e painéis implica vieses (por exemplo, times mais engajados em DevOps respondem mais ao survey), o que torna os percentuais e benchmarks úteis como referências, não como normas universais.[^2][^4][^16]

Além disso, a definição operacional das métricas evoluiu: MTTR foi rebatizada como Failed Deployment Recovery Time, explicitando que o foco é recuperação após deploy com falha, não qualquer incidente; Deployment Rework Rate foi adicionada para captar retrabalho pós‑incidente. Em 2026, críticos apontam que automação com IA e geração de código em larga escala tornam Deployment Frequency e Lead Time mais fáceis de “inflar” sem necessariamente agregar valor, reforçando a necessidade de combinar essas métricas com indicadores de qualidade e impacto.[^17][^18][^19][^6][^2]


## 3. Goodhart’s Law, gaming e efeitos colaterais de métricas

### 3.1 Definição de Goodhart’s Law

Goodhart’s Law afirma que “quando uma medida se torna um alvo, ela deixa de ser uma boa medida”, originalmente formulada em um contexto de política monetária, mas hoje aplicada amplamente em gestão e engenharia. A ideia central é que métricas são proxies para o que se deseja medir; ao transformá‑las em objetivos rígidos (por exemplo, bônus atrelados a um número), agentes racionais passam a otimizar o número, mesmo que isso deturpe a realidade que o indicador pretendia representar.[^3][^11][^12][^20]

Exemplos clássicos em engenharia incluem: inflar story points para aumentar “velocity”, escrever testes vazios para alcançar 90% de code coverage, dividir bugs em múltiplos tickets para aumentar “bugs resolvidos”, ou fazer deploys triviais para inflar Deployment Frequency. A literatura recomenda tratar métricas como sinais para conversas e aprendizado, não como scorecards rígidos, e sempre em conjunto com outras métricas em tensão e julgamento humano.[^10][^5][^11][^21][^20][^22]

### 3.2 Métricas de vaidade e comparações inadequadas

Métricas de vaidade são aquelas que produzem números atraentes, fáceis de coletar, mas com pouca relação com valor ou capacidade real, como linhas de código escritas, commits por desenvolvedor, horas logadas e contagem bruta de tickets fechados sem considerar severidade ou impacto. Quando usadas para avaliar indivíduos ou comparar equipes sem contexto, essas métricas levam a comportamentos disfuncionais (por exemplo, evitar refatorações que removem código, priorizar bugs triviais sobre problemas estruturais, fatiar trabalho artificialmente para aumentar contadores).[^5][^20][^23][^10]

Comparar diretamente equipes com contextos distintos (domínio de negócio, stack, nível de automação, maturidade de testes, volume de legado) apenas pelo número de deploys ou lead time é considerado um anti‑pattern, pois ignora diferenças estruturais e incentiva gaming para “não ficar mal no ranking”. Fontes especializadas recomendam focar em tendências dentro da mesma equipe ao longo do tempo e em relação a metas de outcomes (satisfação do cliente, estabilidade do serviço), em vez de rankings entre times.[^12][^22][^5]

### 3.3 Quantitativo vs. qualitativo e métricas em pares

Diversos autores sugerem combinar métricas quantitativas (tempo, taxa, contagem) com feedback qualitativo (surveys, entrevistas, comentários em retros) para captar dimensões difíceis de medir diretamente, como segurança psicológica, qualidade de colaboração e frustração com ferramentas. Outra defesa contra Goodhart’s Law é parear métricas em tensão: para cada indicador de velocidade (deploy frequency, lead time, throughput de tickets), acompanhar um de qualidade/estabilidade (change failure rate, escaped defects, crash‑free sessions) e um de satisfação (NPS ou NSAT de desenvolvedores e usuários).[^24][^25][^26][^7][^3][^5]

Recomenda‑se também rotacionar métricas de foco (por trimestre, por exemplo) mantendo um núcleo estável apenas para análise de tendência, reduzindo o incentivo a gaming prolongado em um número específico. Boas práticas incluem manter métricas em dashboards de uso interno, sem ligação direta com avaliação individual ou bônus, e usar os dados como gatilho para perguntas (“por que nosso lead time aumentou?”), não como veredicto (“seu lead time acima de X dias é inaceitável”).[^11][^20][^22][^3][^12]


## 4. Catálogo de métricas: definição, fórmula, fonte, frequência, interpretação, riscos

A seguir, um catálogo das principais métricas solicitadas, organizado por domínio. Os exemplos de fórmulas têm caráter operacional; a implementação exata dependerá de ferramentas de CI/CD, issue trackers e observabilidade.

### 4.1 Métricas DORA e afins

| Métrica | Definição / Fórmula | Fonte de dados | Frequência recomendada | Interpretação | Riscos / gaming |
|--------|---------------------|----------------|------------------------|--------------|-----------------|
| Deployment Frequency (DF) | Número de deploys bem‑sucedidos em produção por período; medir por serviço/equipe. | Logs de deploy (CI/CD, orquestradores), Git tags, change management. | Diário / semanal. | DF alta com lotes pequenos tende a correlacionar com ciclos rápidos de feedback e menor risco por mudança.[^1][^18][^2] | Forçar deploys pequenos sem valor, fazer deploy de no‑ops ou re‑deploys do mesmo artefato apenas para “subir o número”.[^5][^20] |
| Lead Time for Changes (LT) | Tempo entre o primeiro commit associado a uma mudança e seu deploy em produção; tipicamente medido da merge no main até produção (change lead time).[^1][^18][^2] | Git (timestamps de commit/merge), logs de deploy. | Semanal / quinzenal, monitorando distribuição (p50, p90). | LT baixo indica fluxo eficiente entre desenvolvimento, revisão, CI e deploy; aumenta previsibilidade.[^1][^2] | Otimizar LT sacrificando revisão de código ou testes, pular etapas de qualidade, agrupar múltiplas mudanças em um único PR para “diluir” atrasos. |
| Change Failure Rate (CFR) | Percentual de deploys que causam falha em produção exigindo rollback, hotfix ou mitigação urgente. \(CFR = \frac{deploys\_com\_falha}{deploys\_totais}\).[^1][^18][^2] | Ferramentas de incident response, change management, registros de rollback/hotfix. | Mensal, com acompanhamento pós‑release. | CFR baixo indica boa qualidade de mudanças e processos de teste / revisão eficazes.[^1][^2] | Sub‑notificar incidentes, reclassificar incidentes como “não relacionados ao deploy”, evitar deploys arriscados mesmo quando gerariam alto valor. |
| Failed Deployment Recovery Time (FDRT / MTTR) | Tempo para restaurar o serviço após um deploy que causou falha; da detecção à restauração do serviço.[^6][^18][^2] | Ferramentas de incident management (PagerDuty, Opsgenie), logs de alerta, sistemas de status. | Mensal, por incidente e como mediana/p90. | FDRT baixo indica boa observabilidade, runbooks eficazes e capacidade de rollback rápida; correlaciona com resiliência operacional.[^6][^2] | Resolver sintomas sem tratar causa raiz, encerrar incidentes prematuramente para “parar o relógio”. |
| Deployment Rework Rate | Percentual de deploys não planejados feitos para corrigir incidentes em produção (retrabalho).[^6][^2] | Change management, registros de hotfix/patches. | Mensal. | Alta taxa indica grande volume de retrabalho e possivelmente baixa qualidade de releases; queda sustentada sugere maturidade em qualidade e testes.[^6][^2] | Reclassificar hotfix como “feature”, adiar correções para evitar contar retrabalho, mascarar defeitos latentes. |

### 4.2 Métricas de CI / build / pipeline

| Métrica | Definição / Fórmula | Fonte de dados | Frequência | Interpretação | Riscos / gaming |
|--------|---------------------|----------------|-----------|--------------|-----------------|
| Build Duration (local e CI) | Tempo de execução de builds (local e pipeline) da mudança até feedback de sucesso/falha; acompanhar p50 e p90.[^24][^7][^8] | Logs de ferramenta de build (Gradle, npm, msbuild), CI (Jenkins, GitLab, GitHub Actions). | Semanal, por pipeline/tipo de build. | Builds curtos (por exemplo, <10 min em CI para PR típico) reduzem tempo ocioso de dev e incentivam commits frequentes.[^7][^8] | Otimizar apenas a “métrica” removendo testes importantes, fragmentar pipelines excessivamente, evitar builds completos localmente. |
| Queue Time | Intervalo entre submissão do job de CI e início efetivo da execução.[^8] | Orquestrador de CI/CD. | Semanal. | Queue time alto indica falta de capacidade de CI ou configuração de filas/prioridades inadequadas; reduz produtividade e aumenta frustração.[^8][^7] | Forçar priorização manual de jobs “importantes” sem resolver causa raiz, limitar acesso ao CI para poucos times. |
| Pipeline Success Rate | Percentual de execuções de pipeline que terminam em sucesso (build + testes + steps de análise).[^24][^8] | Logs de CI. | Semanal / mensal. | Taxa saudável indica pipelines estáveis; falhas frequentes por infraestrutura apontam problemas de plataforma, falhas por testes indicam problemas de qualidade.[^8] | Reclassificar falhas como “skipped”, desativar conjuntos de testes problemáticos, ignorar falhas intermitentes em vez de corrigi‑las. |
| Flaky Test Rate | Percentual de execuções de testes que alternam entre passar/falhar sem mudança de código; ou % de testes marcados como flaky.[^8][^27] | Ferramentas de análise de testes, extensões de CI, relatórios de flakiness. | Mensal. | Flakiness baixa (<5%) aumenta confiança na CI e reduz reruns; alta flakiness gera desperdício e leva devs a ignorar falhas.[^27][^8] | Rotular testes como “flaky” para não investigar causas reais, excluir testes instáveis sem substituição, mascarar problemas estruturais (race conditions, dependências externas). |
| Custo de infraestrutura de CI | Custo por build ou por minuto de execução de pipeline, agregando uso de máquinas, contêineres e storage de artefatos.[^8][^7] | Faturas de cloud, métricas de uso (por job), billing API. | Mensal. | Permite otimizar paralelismo, caching e dimensionamento, equilibrando velocidade de feedback e custo.[^8] | Otimizar custo à custa de feedback lento (desligar paralelismo em excesso), subdimensionar ambientes levando a builds extremamente lentos ou instáveis. |

### 4.3 Métricas de fluxo de trabalho (PRs, promoção, hotfix, rollback)

| Métrica | Definição / Fórmula | Fonte | Frequência | Interpretação | Riscos |
|--------|---------------------|-------|-----------|--------------|--------|
| Tempo de feedback de Pull Request | Tempo entre abertura do PR e primeiro feedback significativo (comentário, review, aprovação). | Git hosting (GitHub/GitLab/Bitbucket). | Semanal. | Feedback rápido melhora fluxo e experiência de dev; tempos longos indicam gargalos de revisão ou sobrecarga de revisores.[^24][^7] | Aprovações superficiais apenas para “bater meta”, dividir PRs artificialmente para reduzir números sem reduzir batch size real. |
| Tempo de promoção | Tempo entre aprovação/merge e deploy em produção (às vezes parte do Lead Time, mas útil isoladamente). | CI/CD, change management. | Semanal. | Gap grande entre merge e deploy indica gargalos pós‑merge (janelas de release, processos manuais, gates de aprovação).[^18][^2] | Automação de deploy sem controles adequados, pular steps de validação para reduzir tempo. |
| Frequência de rollback | Número e taxa de rollbacks por período; \(rollback\_rate = \frac{rollbacks}{deploys}\). | Ferramentas de release, change management. | Mensal. | Alguns rollbacks indicam coragem em experimentar

---

## References

1. [DORA's software delivery metrics: the four keysdora.dev › guides › dora-metrics-four-keys](https://dora.dev/guides/dora-metrics/) - DORA is a long running research program that seeks to understand the capabilities that drive softwar...

2. [What Are DORA Metrics? | IBM](https://www.ibm.com/think/topics/dora-metrics) - DORA metrics are a set of standardized software delivery performance indicators that describe how ef...

3. [Goodhart's Law: Why Metrics Get Gamed and How to Prevent It](https://kpitree.co/guides/frameworks/goodharts-law) - Goodhart's Law warns that when a measure becomes a target, it ceases to be a good measure. Learn how...

4. [Accelerate State Of DevOps 2021](https://cloud.google.com/resources/state-of-devops) - Explore the practices that drive successful software delivery and operational performance so you can...

5. [Software Engineering Metrics: The Complete Guide for 2026](https://oobeya.io/blog/software-engineering-metrics-complete-guide-2026) - Goodhart's Law says that when a measure becomes a target, it stops being a good measure. In engineer...

6. [A history of DORA's software delivery metrics](https://dora.dev/insights/dora-metrics-history/) - DORA is a long running research program that seeks to understand the capabilities that drive softwar...

7. [9 Developer Experience Metrics for Engineering Leaders in 2026](https://pensero.ai/blog/developer-experience-metrics) - Learn the top 9 developer experience metrics engineering leaders should track in 2026 to improve pro...

8. [Top CI Metrics Platform Engineering Leaders Should Track](https://www.harness.io/blog/top-continuous-integration-metrics-every-platform-engineering-leader-should-track) - Track essential Continuous Integration metrics to boost developer productivity, reduce costs, and op...

9. [Release health: deploys, sessions, and crash-free users](https://urgentry.com/guides/observability/release-health/) - What release health measures, how crash-free sessions and users are calculated, and how urgentry's s...

10. [Goodhart's Law and Engineering Metrics](https://www.ruzora.com/blog/goodharts-law-and-engineering-metrics) - Goodhart's Law and engineering metrics: when a measure becomes a target, teams game it. Why velocity...

11. [Goodhart's Law — Yagnipedia](https://yagnipedia.com/wiki/goodharts-law) - Goodhart's Law — When a Measure Becomes a Target, It Ceases to Be a Good Measure

12. [Goodhart's Law](https://lawsofsoftwareengineering.com/laws/goodharts-law/) - Metrics are proxies for what you value (productivity, quality, etc.) ... velocity, code coverage, an...

13. [What is Toil in SRE: Understanding Its Impact](https://sre.google/sre-book/eliminating-toil/) - Toil is the kind of work tied to running a production service that tends to be manual, repetitive, a...

14. [DORA metrics | Technology Radar](https://www.thoughtworks.com/en-br/radar/techniques/dora-metrics) - The metrics defined by the DORA research program have been widely adopted and have proven to be stro...

15. [DORA e-book 2024 - new branding - EN](https://axify.io/hubfs/Axify%20-%20DORA%20e-book/DORA%20E-book%202024%20-%20EN.pdf)

16. [Accelerate](https://services.google.com/fh/files/misc/2023_final_report_sodr.pdf)

17. [DORA Metrics Explained: The Complete Guide (2026)](https://larridin.com/developer-productivity-hub/dora-metrics-explained-complete-guide-2026) - What DORA metrics are, how they work, their history, and where they stand in 2026. Comprehensive ref...

18. [DORA metrics: the five software-delivery indicators](https://cicd.watch/learn/dora-metrics) - DORA metrics measure software delivery performance across five dimensions: how fast you ship and how...

19. [Deployment Frequency](https://dev.to/jreock/dora-metrics-what-are-they-and-whats-new-in-2023-4l50) - Despite some recent criticism, DORA metrics remain the most asked about framework for measuring...

20. [Goodhart's Law in Software: Why Your Metrics Get Gamed](https://codepulsehq.com/guides/goodharts-law-engineering-metrics) - Every metric you track will eventually be gamed if you're not careful. This guide explains why, show...

21. [Goodhart's Law | Learning Lab | oswald.build](https://www.oswald.build/learning-lab/goodharts-law) - When a measure becomes a target, it ceases to be a good measure.

22. [Goodhart's Law Is Quietly Breaking Your Engineering Metrics](https://ctoconnect.io/articles/goodharts-law-is-breaking-your-metrics/) - When a measure becomes a target it stops being a good measure. This explains most of your broken das...

23. [Goodhart's Law - Agile Laws](https://agilelaws.com/goodharts-law.html) - Understand Goodhart's Law and how misused metrics can distort behavior. Learn how Agile teams apply ...

24. [Inside Look: Measuring Developer Productivity and Happiness at LinkedIn](https://www.linkedin.com/blog/engineering/developer-experience-productivity/inside-look-measuring-developer-productivity-and-happiness-at-l)

25. [Developer Sentiment &...](https://getstream.io/blog/developer-experience-metrics/) - Learn which developer experience metrics matter most. This guide breaks down 12 DevEx metrics with e...

26. [I wish more software engineering leaders remembered ...](https://www.linkedin.com/posts/robertroskam_i-wish-more-software-engineering-leaders-activity-7340498621840216064-BHwx) - Goodhart's Law is always in my mind, especially in QA. Code coverage, bug counts, velocity… these me...

27. [Flaky Test Benchmark Report 2026: Rates, Causes, Cost](https://testdino.com/blog/flaky-test-benchmark) - Benchmark flaky test rates, understand why they happen, and estimate CI impact. Learn how to measure...

