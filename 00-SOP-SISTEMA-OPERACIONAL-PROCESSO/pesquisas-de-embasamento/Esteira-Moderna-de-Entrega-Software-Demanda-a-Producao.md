# Esteira Moderna de Entrega de Software: Da Demanda à Produção

## Visão geral do fluxo

Uma esteira moderna de entrega de software conecta a captura de demandas ao valor em produção através de estágios sucessivos: demanda → análise → refinamento → priorização → backlog → Sprint Planning → desenvolvimento → code review → testes → homologação → deploy → produção → conclusão. O Scrum formaliza apenas uma parte central desse fluxo — o ciclo Product Backlog → Sprint Backlog → Incremento — enquanto os estágios de entrada (análise, priorização) e saída (deploy, produção) são normalmente geridos por práticas de engenharia e DevOps que complementam, mas não pertencem, ao framework oficial.[^1][^2]

Representação textual da esteira completa:

```
[Demanda] → [Análise] → [Refinamento] → [Priorização] → [Product Backlog]
     → [Sprint Planning] → [Sprint Backlog] → [Desenvolvimento] 
     → [Code Review] → [Testes/QA] → [Homologação] → [Deploy] 
     → [Produção] → [Conclusão / Done]
```

## Quando uma tarefa entra em uma Sprint

Uma tarefa entra formalmente na Sprint no evento de Sprint Planning, quando os Developers, em discussão com o Product Owner, selecionam itens do Product Backlog considerados "prontos" (Ready) para compor o Sprint Backlog. Segundo o Scrum Guide, "Product Backlog items that can be Done by the Scrum Team within one Sprint are deemed ready for selection in a Sprint Planning event," e essa transparência normalmente só é alcançada após atividades de refinamento prévias. Em ferramentas como Jira, esse momento é operacionalizado ao arrastar o item do Backlog para a Sprint ativa; até então, o item existe apenas como issue não vinculada a nenhum sprint.[^3][^4][^5][^6][^2][^7]

## Backlog geral vs. Sprint Backlog

| Aspecto | Product Backlog (geral) | Sprint Backlog |
|---|---|---|
| Definição | Lista emergente e ordenada de tudo que é necessário no produto[^1] | Sprint Goal + PBIs selecionados + plano de entrega, criado pelos Developers[^8][^9] |
| Responsável | Product Owner (conteúdo, disponibilidade e ordenação)[^1] | Developers (é "um plano por e para os Developers")[^8] |
| Duração | Nunca está completo; evolui continuamente[^1] | Recriado a cada Sprint; válido apenas durante o Sprint[^10] |
| Visualização em ferramentas | Aba "Backlog" no Jira / Azure DevOps[^11] | Board "Active Sprint" / Sprint Board[^12] |
| Objetivo | Repositório e ordenação de tudo que pode ser feito | Produzir um Incremento "Done" que atenda ao Sprint Goal[^12] |

No Jira, tecnicamente, o backlog e a sprint são específicos do board (não do projeto), e um item deixa de ser "backlog item" apenas quando é movido para dentro de uma sprint.[^13][^3]

## Refinamento das tarefas

O refinamento (Product Backlog Refinement) é a atividade contínua de detalhar, estimar e ordenar itens do backlog, conduzida em colaboração entre Product Owner e Developers. É importante destacar: **isso não é um evento oficial do Scrum** — o Scrum Guide não define uma cerimônia específica, cabendo ao time decidir como e quando fazê-lo, consumindo tipicamente não mais que 10% da capacidade dos Developers. Uma boa prática (não uma regra) recomendada por especialistas do Scrum.org é manter de uma a duas Sprints de itens "Ready" no backlog, revisando-os continuamente para que os Developers entendam qualquer item nas próximas duas sprints.[^14][^2][^15][^16][^17]

## Prioridade e capacidade

A priorização dos itens do Product Backlog é responsabilidade do Product Owner, que ordena a lista segundo valor de negócio, risco e dependências. A capacidade é considerada de forma empírica: os Developers usam dados históricos (velocity de sprints anteriores) e a disponibilidade real da equipe (férias, feriados, outras alocações) para decidir quantos itens do topo do backlog cabem no Sprint. É crucial notar que "capacidade" é mencionada apenas duas vezes no Scrum Guide, e conceitos como "focus factor" não aparecem nele — a mecânica exata de cálculo de capacidade é deixada para o time definir.[^5][^18][^1]

## Divisão de tarefas grandes

Itens grandes do backlog (épicos) devem ser decompostos em itens menores durante o refinamento, até que caibam dentro de um Sprint segundo a Definition of Done. Durante a própria Sprint Planning, os Developers decompõem cada PBI selecionado em tarefas de execução de um dia ou menos, processo cuja forma exata "está a exclusivo critério dos Developers" — nenhuma outra pessoa dita como fazer essa quebra. Em ferramentas como Jira e Azure DevOps, essa hierarquia é representada como Epic > Story/PBI > Task/Subtask, uma convenção de mercado e não uma nomenclatura do Scrum Guide.[^2][^17][^5]

## Dependências entre tarefas

O Scrum Guide não prescreve um mecanismo formal para dependências; na prática de mercado, elas são tratadas via: (1) sequenciamento no refinamento, priorizando itens bloqueadores antes dos dependentes; (2) uso de campos de "linked issues" ou "blocks/is blocked by" em Jira e Azure DevOps; e (3) coordenação entre times via eventos como Scrum of Scrums em contextos multiequipe (Nexus). Dependências não resolvidas antes do Planning são um sinal de que o item ainda não estava "Ready" para entrar na Sprint.

## Bugs descobertos durante a Sprint

Bugs identificados durante a Sprint tornam-se trabalho adicional que compete com o Sprint Goal. Conforme o Scrum Guide, durante a Sprint "nenhuma mudança é feita que colocaria em risco o Sprint Goal" e "o escopo pode ser esclarecido e renegociado com o Product Owner conforme mais se aprende". Na prática de mercado, bugs críticos que impedem a entrega do Incremento planejado são geralmente tratados dentro da própria Sprint (o time decide trocar um item de menor prioridade), enquanto bugs não bloqueantes são registrados e priorizados no Product Backlog para sprints futuras — essa é uma prática de gestão de fluxo, não uma regra do Scrum.[^2]

## Demandas urgentes e incidentes de produção

Incidentes de produção urgentes (ex.: outages) representam uma tensão direta com o princípio de que "nenhuma mudança é feita que colocaria em risco o Sprint Goal". Como boa prática amplamente adotada (não oficial), muitas equipes reservam uma capacidade de buffer (ex.: 10-20%) para atendimento a incidentes, ou tratam a interrupção como uma renegociação de escopo com o Product Owner. Em casos extremos, se as informações tornam o Sprint Goal obsoleto, o próprio Product Owner pode cancelar a Sprint — esta é uma cláusula oficial do Scrum Guide, embora rara.[^9][^2]

## Tarefas bloqueadas

O Scrum não define formalmente um estado "bloqueado", mas ele é universalmente usado em ferramentas Kanban/Jira/Azure DevOps para sinalizar impedimentos (falta de acesso, dependência externa, decisão pendente). Como boa prática, impedimentos devem ser levantados no Daily Scrum — evento oficial cujo propósito é inspecionar o progresso rumo ao Sprint Goal e adaptar o Sprint Backlog conforme necessário. A remoção de impedimentos organizacionais é uma das responsabilidades atribuídas ao Scrum Master pelo Scrum Guide.[^2]

## Desenvolvimento concluído, mas testes/homologação pendentes

Um item cujo código está pronto mas não passou por testes/homologação não atende à Definition of Done e, portanto, **não pode ser contado como "Done"** nem integrar o Incremento. Isso reforça por que muitos times de mercado adotam quadros com colunas intermediárias (Em Code Review, Em Teste, Em Homologação) — uma prática operacional de ferramentas como Jira/Azure DevOps, não uma exigência do Scrum Guide, que só reconhece os estados binários "parte do Sprint Backlog" e "Done".[^19][^20]

## Quando uma tarefa é realmente concluída

Uma tarefa é considerada verdadeiramente concluída apenas quando atende integralmente à Definition of Done acordada pelo time — "uma descrição formal do estado do Incremento quando atende às medidas de qualidade exigidas para o produto". Esse é um conceito e artefato oficial do Scrum; sem ele, não há garantia de transparência sobre o real estado de "pronto" de cada item.[^20][^7][^19]

## Como a Definition of Done afeta a esteira

A Definition of Done (DoD) é um dos elementos formais do Scrum e influencia toda a esteira: ela orienta a decomposição de PBIs em tarefas durante o Planning, define os critérios de aceite usados no code review, testes e homologação, e determina se o Incremento gerado no fim da Sprint é utilizável e potencialmente entregável. Sem uma DoD compartilhada entre Desenvolvimento, QA e Produto, a esteira perde consistência entre sprints, pois cada squad pode interpretar "pronto" de forma diferente.[^19][^20]

## Tarefas que não terminam dentro da Sprint

O Scrum Guide não define uma regra fixa sobre o destino de itens incompletos; a decisão pertence ao Product Owner, que pode decidir devolvê-los ao Product Backlog para reordenação ou negociar sua permanência no próximo Sprint Backlog. É prática comum de mercado — não regra oficial — simplesmente "rolar" (carry over) o item automaticamente para a próxima Sprint em ferramentas como Jira.[^13]

## Riscos de transferir tarefas automaticamente entre Sprints

Essa prática, embora conveniente, pode mascarar problemas estruturais: itens mal dimensionados ou insuficientemente refinados durante o Planning; capacidade real da equipe subestimada de forma recorrente; dependências externas não identificadas a tempo; e falta de decomposição adequada em tarefas pequenas o bastante para caber em um Sprint. O carry-over recorrente do mesmo item indica falha na etapa de refinamento (ver seção acima) e deveria disparar uma inspeção na Retrospectiva, e não ser tratado como normal.

## Organização do fluxo com Dev, QA, UX, DevOps e Produto

Squads multidisciplinares tipicamente organizam o fluxo com UX e Produto atuando na fase de análise/refinamento (antes do item entrar no Sprint), Desenvolvimento e QA colaborando dentro da Sprint (idealmente em paralelo, não sequencialmente, para evitar "mini-cascatas" dentro do Sprint) e DevOps provendo pipelines de CI/CD que automatizam build, testes e deploy contínuos. O Scrum Guide de 2020 unificou os papéis dentro de um único "Scrum Team" sem subtimes formais, deixando essa divisão de responsabilidades como decisão organizacional, não como regra do framework.[^7]

## CI/CD e deploy contínuo em relação às Sprints

CI/CD é uma prática de engenharia de software complementar ao Scrum, não parte dele. A integração contínua permite que código passe por build e testes automatizados a cada commit, e a entrega/deploy contínuo desacopla o momento de deploy do calendário da Sprint. Isso significa que **o deploy não precisa ocorrer apenas ao final da Sprint** — em times maduros com pipelines de CI/CD, itens concluídos e aprovados podem ir para produção a qualquer momento durante a Sprint, aumentando a frequência de entrega de valor e reduzindo risco de lotes grandes de deploy no fim do ciclo.

## Sprint e Kanban coexistindo (Scrumban)

É prática comum de mercado combinar o timebox e os eventos do Scrum (Planning, Daily, Review, Retrospective) com um quadro de fluxo estilo Kanban, usando limites de Work in Progress (WIP) dentro da Sprint para controlar quantos itens estão simultaneamente "Em Desenvolvimento", "Em Review" ou "Em Teste". Essa combinação, às vezes chamada de "Scrumban", não é definida no Scrum Guide, mas é amplamente documentada como abordagem híbrida por consultorias e pela própria comunidade ágil.

## Métricas: finalidade e origem oficial

| Métrica | Definição | Finalidade | Origem oficial |
|---|---|---|---|
| Velocity | Quantidade de trabalho (normalmente story points) que o time entrega "Done" por Sprint[^21][^22] | Ajudar Developers a prever quanto trabalho cabe na próxima Sprint | **Não é do Scrum Guide** — termo ausente do documento oficial; é prática complementar[^23][^21][^24] |
| Capacidade | Disponibilidade real de horas/pessoas para a Sprint | Base para o forecast de quantos itens selecionar no Planning | Mencionada apenas 2x no Scrum Guide, sem método prescrito[^18] |
| Burndown | Gráfico de trabalho remanescente ao longo do tempo | Visualizar se o time está no ritmo para cumprir o Sprint Goal | **Não é mais parte do Scrum Guide** — burndown de release foi removido em versões após 2011[^25][^26] |
| Burnup | Gráfico de trabalho concluído acumulado vs. escopo total | Mostrar progresso e mudanças de escopo ao longo do tempo | Prática complementar, não citada no Scrum Guide[^27] |
| Lead time | Tempo entre a criação/solicitação de um item e sua entrega em produção[^28][^29] | Medir a experiência do cliente/solicitante do início ao fim | Termo de origem Lean/Kanban, não do Scrum[^30][^28] |
| Cycle time | Tempo entre o início do trabalho ativo em um item e sua conclusão[^29][^31] | Medir a eficiência interna do fluxo de trabalho da equipe | Termo de origem Lean/Kanban, não do Scrum[^30][^32] |
| Throughput | Número de itens concluídos por unidade de tempo | Medir a taxa de entrega do sistema, base para forecasts probabilísticos | Prática de Kanban/métricas de fluxo, não do Scrum[^30] |
| Work in Progress (WIP) | Quantidade de itens simultaneamente em andamento | Controlar sobrecarga e reduzir cycle time via limites de WIP | Conceito central do Kanban, não do Scrum |

Cross-checagem entre fontes independentes (Scrum.org, Agile Alliance, Scrum Alliance) confirma consistentemente que **velocity, burndown, burnup, lead time, cycle time, throughput e WIP não constam do Scrum Guide oficial**. Apenas a **Definition of Done** e o conceito qualitativo de **capacidade** (sem metodologia prescrita) têm base textual direta no documento oficial. O Scrum Guide 2020 explicitamente evita prescrever métricas, mantendo-se como framework minimalista baseado em empirismo (transparência, inspeção, adaptação).[^23][^24][^18][^33][^27][^34][^7][^19]

## Exemplo concreto: Sprint de duas semanas

Considere uma equipe de oito pessoas (4 devs, 1 QA, 1 UX, 1 DevOps, 1 PO) trabalhando em um produto SaaS, com Sprint de 03/09 a 16/09/2026 (data de referência atual). O Sprint Backlog contém 10 itens selecionados no Planning a partir do Product Backlog priorizado pelo PO.

| Item | Tipo | Estado ao longo da Sprint | Observação |
|---|---|---|---|
| PBI-101 | Story | Concluída normalmente (Done) | Passou por dev, review e QA dentro do prazo |
| PBI-102 | Story | Bloqueada | Aguardando credencial de API de terceiros |
| PBI-103 | Task técnica | Aguardando code review | Dev finalizou, PR aberto há 2 dias |
| PBI-104 | Bug | Descoberto durante a Sprint | Reportado por QA ao testar PBI-101; priorizado no lugar de item de menor valor |
| PBI-105 | Incidente | Demanda urgente | Outage em produção; time pausou item planejado para atender |
| BUG-106 | Bug | Aguardando testes | Correção implementada, à espera de ciclo de regressão |
| PBI-107 | Story | Não terminou na Sprint | Subestimada; devolvida ao Product Backlog para nova decomposição |
| PBI-108 | Story | Concluída normalmente (Done) | Deploy contínuo realizado no 6º dia da Sprint via CI/CD |
| PBI-109 | Spike técnico | Concluído normalmente | Investigação de viabilidade, sem código em produção |
| PBI-110 | Story | Em homologação | Aguardando validação funcional do PO antes do deploy final |

## Estados possíveis das tarefas

| Estado | Significado |
|---|---|
| Backlog | Item aguardando refinamento e priorização, fora da Sprint |
| Pronto (Ready) | Refinado, estimado, elegível para seleção em Planning |
| Sprint Backlog / A Fazer | Selecionado para a Sprint, ainda não iniciado |
| Em Desenvolvimento | Código sendo escrito |
| Aguardando Code Review | Pull request aberto, à espera de aprovação de pares |
| Em Teste/QA | Validação funcional e de regressão em curso |
| Bloqueado | Impedido por dependência externa ou interna |
| Em Homologação | Validação de negócio/PO antes do deploy |
| Em Deploy | Publicação em produção via pipeline de CI/CD |
| Done | Atende à Definition of Done; incremento entregue |

## Responsabilidades por papel

| Papel | Responsabilidades principais |
|---|---|
| Product Owner | Ordena e prioriza o Product Backlog; define e comunica o Sprint Goal; aceita ou rejeita itens em homologação[^1][^2] |
| Developers | Estimam, decompõem tarefas, criam o Sprint Backlog e o plano de entrega, e produzem o Incremento[^8][^5] |
| Scrum Master | Remove impedimentos, facilita eventos, protege o time de interrupções que ameacem o Sprint Goal[^2] |
| QA | Executa testes funcionais/regressão, valida critérios da Definition of Done |
| UX | Participa do refinamento definindo fluxos e protótipos antes da entrada no Sprint Backlog |
| DevOps | Mantém pipelines de CI/CD, automatiza build/teste/deploy, monitora produção |

## Regras recomendadas para movimentação de tarefas

- Um item só avança de coluna quando os critérios daquela etapa (ex.: aprovação de PR, testes passando) estão satisfeitos.
- Nenhum item deve pular etapas de qualidade (ex.: ir direto de "Em Desenvolvimento" para "Done") sem passar por review e teste.
- Impedimentos devem ser sinalizados no Daily Scrum e escalados ao Scrum Master imediatamente.[^2]
- Mudanças de escopo dentro da Sprint devem ser negociadas com o Product Owner, preservando o Sprint Goal sempre que possível.[^2]

## Boas práticas

- Manter de uma a duas Sprints de itens "Ready" no backlog para evitar Plannings improvisados.[^17]
- Limitar WIP para reduzir cycle time e evitar multitarefa excessiva (prática de Kanban aplicável dentro da Sprint).
- Automatizar pipelines de CI/CD para permitir deploys contínuos e reduzir lotes de risco no fim da Sprint.
- Tratar velocity apenas como ferramenta interna de forecast do time, nunca como KPI comparativo entre equipes.[^21][^22]

## Anti-patterns comuns

- Usar velocity como métrica de desempenho individual ou comparação entre times, distorcendo estimativas.[^22][^35]
- Aceitar carry-over sistemático de itens sem investigar causas raiz no refinamento ou na estimativa.
- Tratar Product Backlog refinement como reunião formal obrigatória em vez de atividade contínua e flexível.[^16]
- Permitir que demandas urgentes interrompam a Sprint sem negociação explícita com o Product Owner, corroendo o Sprint Goal.[^2]

## Scrum oficial vs. adaptações de mercado

| Elemento | Status no Scrum Guide oficial | Realidade de mercado |
|---|---|---|
| Sprint, Sprint Backlog, Product Backlog, DoD | Artefatos/eventos oficiais[^2][^7] | Usados como definidos |
| Refinamento | Mencionado, mas não é evento formal[^14] | Frequentemente tratado como reunião fixa semanal |
| Velocity, burndown, burnup | Ausentes do texto oficial[^23][^27] | Amplamente usados em Jira/Azure DevOps como padrão |
| Lead time, cycle time, throughput, WIP | Conceitos de Lean/Kanban, não do Scrum[^30][^28] | Comumente combinados com Scrum em dashboards híbridos |
| Colunas de board (Code Review, QA, Homologação) | Não especificadas pelo Scrum Guide | Convenção de ferramentas como Jira e Azure DevOps[^11][^36] |
| Papéis de Dev/QA/UX/DevOps separados | Scrum Guide 2020 unifica em "Scrum Team" sem subpapéis[^7] | Squads reais mantêm especializações distintas |

---

## References

1. [[PDF] 2017 Scrum Guides](https://scrumguides.org/docs/scrumguide/v2017/2017-Scrum-Guide-US.pdf)

2. [The 2020 Scrum Guide TM](https://scrumguides.org/scrum-guide.html) - Product Backlog refinement is the act of breaking down and further defining Product Backlog items in...

3. [Sprint is part of project or board ?](https://community.atlassian.com/forums/Jira-questions/Sprint-is-part-of-project-or-board/qaq-p/2837937) - I'm trying to understand how jira works with respect to backlogs & sprints. Observation 1: My genera...

4. [Product backlog and sprint backlog - Jira](https://community.atlassian.com/forums/Jira-questions/Product-backlog-and-sprint-backlog/qaq-p/715171) - Basically the Backlog which you will see in your scrum board right now will serve as a Product Backl...

5. [What is Sprint Planning? - Scrum.org](https://www.scrum.org/resources/what-is-sprint-planning) - Sprint Planning initiates the Sprint by laying out the work to be performed for the Sprint. This res...

6. [Introduction to Sprint Planning](https://www.scrum.org/resources/introduction-sprint-planning) - Every Sprint starts with Sprint Planning where the Scrum Team determines what they plan to accomplis...

7. [Ken Schwaber and Jeff Sutherland](https://scrumguides.org/docs/scrumguide/v2020/2020-Scrum-Guide-US.pdf)

8. [What is a Sprint Backlog? - Scrum.org](https://www.scrum.org/resources/what-is-a-sprint-backlog) - The Sprint Backlog is a plan by and for the Developers. It is a highly visible, real-time picture of...

9. [Introduction to the Sprint Backlog](https://www.scrum.org/resources/introduction-sprint-backlog) - The Sprint Backlog represents what the Developers plan to accomplish during the Sprint. The Sprint B...

10. [What Is a Sprint Backlog? | Definition and Complete Guide](https://resources.scrumalliance.org/Article/sprint-backlog) - Enable transparency, inspection, and adaptation with the sprint backlog. It includes the sprint goal...

11. [Creating your backlog | Jira Software Data Center 9.12confluence.atlassian.com › jirasoftwareserver0912 › creating-your-backlog...](https://confluence.atlassian.com/jirasoftwareserver0912/creating-your-backlog-1346048645.html)

12. [Backlog and Sprint Planning in Jira Software - What's the Difference? Jira Guru](https://community.atlassian.com/forums/App-Central-discussions/Backlog-and-Sprint-Planning-in-Jira-Software-What-s-the/td-p/2482320) - Hi Community, Welcome back! This is Wednesday again, and Weekly knowledge growth with Jira Guru! Tod...

13. [what is the difference between the type "Backlog item" and "User story" in active sprint](https://community.atlassian.com/forums/Jira-questions/what-is-the-difference-between-the-type-quot-Backlog-item-quot/qaq-p/2429230) - Hello, currently, we create backlogs in the form of epics, user stories or backlog (as type) in our ...

14. [Why is Backlog Refinement not an official scrum event yet?](https://www.scrum.org/forum/scrum-forum/8000/why-backlog-refinement-not-official-scrum-event-yet) - While backlog refinement is mentioned in the scrum guide in the product backlog section, it is still...

15. [Product Backlog Refinement](https://www.scrum.org/resources/product-backlog-refinement) - The activity of continually improving PBIs until they are ready to be worked on is called “refinemen...

16. [If your Backlog is not Refined then you are Doing it Wrong - Scrum.org](https://www.scrum.org/resources/blog/if-your-backlog-not-refined-then-you-are-doing-it-wrong) - Most Scrum Teams that I encounter don’t do refinement of their Product Backlog and try to work on th...

17. [Optimizing Product Backlog Refinement - Scrum.org](https://www.scrum.org/resources/blog/optimizing-product-backlog-refinement) - I've recently been involved with several Scrum Teams that are struggling to have the right level of ...

18. [Capacity vs Velocity - Scrum.org](https://www.scrum.org/forum/scrum-forum/7293/capacity-vs-velocity) - I'm currently the Scrum Master of a 10 developer strong team. We've adopted Agile/Scrum methodologie...

19. [Definition of 'Done': What It Is and How It Supports Scrum Events](https://www.scrum.org/resources/blog/definition-done-what-it-and-how-it-supports-scrum-events) - The Definition of 'Done' is probably the most misunderstood concept of the Scrum framework. In this ...

20. [What's the big deal about the Definition of Done in Scrum?](https://www.scrum.org/resources/blog/whats-big-deal-about-definition-done-scrum) - There are numerous benefits to a Definition of Done, but consider this: The entire purpose of Sprint...

21. [Velocity](https://www.scrum.org/forum/scrum-forum/66555/velocity) - Hi, I couldn't find "Velocity" in the Scrum guid anymore. Is "Velocity" still a part of the scrum fr...

22. [Velocity - Definition | Scrumling Glossary](https://scrumling.com/glossary/velocity) - A measure of the amount of work a team completes in a Sprint, usually expressed in story points, use...

23. [and what it doesn't ? | Suhel Mulla - LinkedIn](https://www.linkedin.com/posts/suhel-mulla-3005_scrum-agile-scrumguide2020-activity-7332979811755229185-vYoP) - 𝗪𝗵𝗮𝘁 𝘁𝗵𝗲 𝗦𝗰𝗿𝘂𝗺 𝗚𝘂𝗶𝗱𝗲 2020 𝘀𝗮𝘆𝘀 𝗮𝗯𝗼𝘂𝘁 𝗩𝗲𝗹𝗼𝗰𝗶𝘁𝘆 — 𝗮𝗻𝗱 𝘄𝗵𝗮𝘁 𝗶𝘁 𝗱𝗼𝗲𝘀𝗻’𝘁 ? One of the most common terms i...

24. [Moving Beyond Velocity | Scrum.org](https://www.scrum.org/resources/blog/moving-beyond-velocity) - In my role as an Agile coach and Scrum trainer, I get the chance to talk to a lot of Scrum Masters a...

25. [Gone Are Release Planning and the Release Turndown - Scrum.org](https://www.scrum.org/resources/gone-are-release-planning-and-release-turndown) - Any Product Manager that has successfully delivered a product to a customer knows how incredibly imp...

26. [Release burndown chart](https://www.scrum.org/forum/scrum-forum/5616/release-burndown-chart) - Hi, I am working as a Product Owner and I would like to introduce Release burndown chart to my produ...

27. [Should management review the scrum metrics of scrum teams?](https://pm.stackexchange.com/questions/23220/should-management-review-the-scrum-metrics-of-scrum-teams) - I read somewhere that team's velocity, burn-up chart, and other scrum indicators are only for team's...

28. [What is Lead Time in Software Development?](https://agilealliance.org/glossary/lead-time/) - Lead Time is the time between a customer order and delivery. In software development, it can also be...

29. [What is lead time - Scrum Alliance agile glossary](https://www.scrumalliance.org/glossary/lead-time) - Lead time measures how long it takes for a request to be fulfilled, starting from the moment of requ...

30. [Metrics for Understanding Flow](https://agilealliance.org/resources/experience-reports/metrics-understanding-flow/) - Cycle Time analysis shows how long it takes for work items to pass through one or a subset of workfl...

31. [7.2.1: Delivery Metrics- Velocity, Throughput, Lead Time, ...](https://eng.libretexts.org/Workbench/De_Anza_-_95G/07:_Agile_Metrics_Risk_and_Delivery_Discipline/7.02:_Agile_Delivery_Discipline_Guide/7.2.01:_Delivery_Metrics-_Velocity_Throughput_Lead_Time_and_Cycle_Time) - This page explains Agile delivery metrics—velocity, lead time, cycle time, throughput, and work in p...

32. [Cycle Time vs. Lead Time: Why the Difference Matters - Agile Analytics](https://agileanalytics.cloud/blog/cycle-time-vs-lead-time-why-the-difference-matters) - Cycle time measures the work. Lead time measures the waiting too. See both on one timeline, and how ...

33. [Velocity. The Revolutionary Way to Measure in Scrum](https://www.scrum.org/resources/blog/velocity-revolutionary-way-measure-scrum) - The Scrum Guide does not mention Velocity. Instead, it refers to the need for forecasting, planning ...

34. [Published Patterns - Notes on Velocity](https://sites.google.com/a/scrumplop.org/published-patterns/value-stream/notes-on-velocity) - Velocity is a much-misunderstood concept, and the purpose of this note is to help clarify what it is...

35. [Velocity; Bad or Good?! - Scrum.orgwww.scrum.org › resources › blog › velocity-bad-or-good](https://www.scrum.org/resources/blog/velocity-bad-or-good) - Velocity is a widely used practice in Agile environments. But it has a big potential to be used in t...

36. [azure-devops-docs/docs/boards/sprints/define-sprints.md at main · MicrosoftDocs/azure-devops-docs](https://github.com/MicrosoftDocs/azure-devops-docs/blob/main/docs/boards/sprints/define-sprints.md) - This repo is the home of the official Azure DevOps documentation for Microsoft. GitHub Issues filed ...

