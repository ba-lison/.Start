# Manual de Referência: Taxonomia Consolidada de Metodologias, Práticas, Documentos e Standards do Ciclo de Software

## Introdução e Nota Metodológica

Este manual sintetiza criticamente terminologia dispersa em TDD/BDD/DDD, documentação de produto/requisitos, standards ISO/IEC/IEEE, evidência científica e práticas de IA aplicada à engenharia de software. A síntese elimina duplicidades, resolve contradições entre fontes e distingue explicitamente **fato verificável**, **convenção de mercado** e **opinião/marketing**. Onde a literatura apresenta divergência (por exemplo, ordem BRD→MRD→PRD), o manual apresenta as versões concorrentes e assinala qual é mais consolidada.

***

## PARTE 1 — TAXONOMIA REVISADA

A taxonomia inicial proposta (A–P) é majoritariamente válida, mas a pesquisa sugere fundir e reordenar algumas categorias, pois "Requirements Engineering" e "Business Analysis" se sobrepõem fortemente, e "Documentation" não é uma categoria paralela às demais — é uma dimensão transversal (todo documento pertence também a uma disciplina-mãe). Taxonomia final:

1. **Estratégia e Descoberta de Mercado** (Market/Product Strategy, Discovery)
2. **Business Analysis & Requirements Engineering** (fundidas — BA produz artefatos que RE formaliza)
3. **Product Management** 
4. **UX e Product Discovery**
5. **Software Architecture & Systems Engineering** (fundidas em nível macro, com Systems Engineering como superconjunto quando há hardware/domínio regulado)
6. **Software Design** (nível de componente/módulo, abaixo de arquitetura)
7. **Development Practices & Methodologies** (TDD, BDD, XP, práticas de codificação)
8. **Testing & Quality Assurance**
9. **Agile e Processos de Gestão** (Scrum, Kanban — nível de processo/gestão, não de codificação)
10. **DevOps e Delivery**
11. **Operations / SRE**
12. **Security (DevSecOps)**
13. **Standards (ISO/IEC/IEEE, RFC IETF)** — categoria transversal de institucionalização formal
14. **Documentation Artifacts** — categoria transversal (cada documento é etiquetado por disciplina de origem, não isolado)
15. **AI-assisted Software Engineering** (nova disciplina emergente, 2023–2026)

A justificativa: a pesquisa mostra repetidamente que TDD/BDD/DDD não pertencem à mesma categoria conceitual entre si (Parte 4), e que "Documentation" funciona melhor como eixo transversal do que como categoria isolada, pois um PRD é documentação de Produto, um SRS é documentação de Requisitos, e um ADR é documentação de Arquitetura.[^1][^2]

***

## PARTE 2 — TABELA MESTRE

| Sigla/Termo | Nome completo | Categoria | Definição curta | Objetivo | Etapa do ciclo | Quem usa | Status | Evidência |
|---|---|---|---|---|---|---|---|---|
| TDD | Test-Driven Development | Development Practices | Escrever teste antes do código, ciclo red-green-refactor | Design de código orientado por testes | Implementação | Devs | B — prática consolidada | Moderada, conflitante em produtividade[^3][^4] |
| BDD | Behavior-Driven Development | Development Practices/Testing | Especificação executável em linguagem natural (Given/When/Then) | Alinhar comportamento entre negócio e devs | Requisitos→Testes | Devs, QA, PO | B | Limitada (poucos estudos controlados) |
| DDD | Domain-Driven Design | Software Design/Architecture | Modelagem de software alinhada ao domínio de negócio via linguagem ubíqua | Gerenciar complexidade de domínio | Design/Arquitetura | Arquitetos, devs seniores | B (em domínios complexos), C fora deles | Baseada em prática de campo, não experimental[^2][^5] |
| ATDD | Acceptance Test-Driven Development | Development Practices | Definir critérios de aceite antes de codificar, colaborativamente | Alinhar "pronto" antes de codar | Requisitos/Testes | Todo o time | C | Limitada |
| FDD | Feature-Driven Development | Agile/Processos | Entrega orientada a funcionalidades de negócio em blocos modelados | Disciplina de entrega escalável | Planejamento/Entrega | Times grandes | D/E (pouco usado hoje) | Limitada |
| XP | Extreme Programming | Agile/Development Practices | Metodologia ágil com práticas técnicas intensivas (pair programming, TDD, CI) | Qualidade + resposta à mudança | Todo o ciclo dev | Times ágeis | B | Moderada (estudos dos anos 2000) |
| Scrum | Scrum | Agile/Processos | Framework de gestão iterativa com sprints, papéis e cerimônias | Gestão de entrega incremental | Gestão de projeto | PMs, POs, times | A/B — de facto standard industrial (Scrum Guide) | Consenso de adoção, evidência de eficácia mista |
| Kanban | Kanban | Agile/Processos | Sistema visual de fluxo contínuo com limite de WIP | Otimizar fluxo, reduzir gargalos | Gestão de fluxo | Times de operação/dev | B | Moderada (Lean origem) |
| MRD | Market Requirements Document | Documentação/Estratégia | Documento sobre oportunidade de mercado, personas, concorrência | Justificar "por que este mercado" | Estratégia/Discovery | PM sênior, marketing | C — convenção de mercado, não standard formal | Nenhuma (prática de indústria)[^6][^7] |
| BRD | Business Requirements Document | Documentação/Business Analysis | Caso de negócio, ROI, escopo de alto nível | Justificar investimento | Business Case/Estratégia | Analistas de negócio, sponsors | C | Nenhuma |
| PRD | Product Requirements Document | Documentação/Product Mgmt | O que construir, para quem, com que métricas de sucesso | Guiar construção do produto | Requisitos de produto | PM, design, eng | B — convenção quase universal em produto | Nenhuma (prática, não standard) |
| SRS | Software Requirements Specification | Documentação/Requirements Eng. | Especificação formal e verificável de requisitos de software | Base contratual/técnica de requisitos | Requisitos formais | Engenheiros de requisitos | A — normatizado (ISO/IEC/IEEE 29148, sucessor do IEEE 830)[^8] | Formalizado, sem evidência experimental de eficácia |
| SyRS | System Requirements Specification | Documentação/Systems Eng. | Requisitos de sistema (hardware+software+operação) | Base de requisitos de sistema | Requisitos de sistema | Engenheiros de sistemas | A — dentro de ISO/IEC/IEEE 29148[^8] | Formal |
| FRD | Functional Requirements Document | Documentação/Requirements Eng. | Detalha funcionalidades específicas do sistema | Detalhar comportamento funcional | Requisitos detalhados | Analistas, devs | C — convenção, sobreposição com SRS | Nenhuma |
| ADR | Architecture Decision Record | Documentação/Arquitetura | Registro curto e imutável de uma decisão arquitetural já tomada | Preservar racional de decisões | Arquitetura/Design técnico | Arquitetos, devs | B — amplamente adotado, sem norma ISO | Nenhuma formal, prática consolidada[^9][^10] |
| RFC (interno) | Request for Comments (uso de engenharia de time) | Documentação/Arquitetura | Proposta aberta para feedback antes de decisão | Buscar consenso antes de decidir | Design técnico/Arquitetura | Engenheiros, times | C — convenção, herdada da cultura IETF | Nenhuma |
| RFC (IETF) | Request for Comments (IETF) | Standards | Documento padrão publicado da Internet Engineering Task Force | Padronizar protocolos de internet | Standards | Comunidade de internet | A — standard formal | Formal |
| HLD | High-Level Design | Documentação/Design | Visão arquitetural macro de um sistema | Comunicar estrutura geral | Design técnico | Arquitetos | C | Nenhuma |
| LLD | Low-Level Design | Documentação/Design | Detalhamento de módulos, classes, algoritmos | Guiar implementação detalhada | Design técnico | Devs | C | Nenhuma |
| SDD | Software Design Document (ou Spec-Driven Development em contexto de IA) | Documentação/Design — termo ambíguo | Documento de projeto de software OU metodologia de IA orientada por especificação | Depende do contexto | Design técnico OU todo o ciclo com IA | Devs / Arquitetos / Agentes de IA | Ambíguo — ver Parte 9 | Ver Parte 9 |
| SDS | Software Design Specification | Documentação/Design | Sinônimo regional de SDD (design document) | Especificar design | Design técnico | Devs | C | Nenhuma |
| ICD | Interface Control Document | Documentação/Systems Eng. | Especifica interfaces entre sistemas/subsistemas | Garantir interoperabilidade | Design de interfaces | Engenharia de sistemas | B (forte em aeroespacial/defesa) | Nenhuma formal, mas normatizado em setores regulados |
| API Spec | API Specification (OpenAPI, etc.) | Documentação/Interfaces | Especificação formal de contrato de API | Garantir contrato entre serviços | Design/Implementação | Devs backend | A (OpenAPI é standard de fato da indústria) | Nenhuma acadêmica, adoção massiva |
| CI | Continuous Integration | DevOps | Integração frequente de código com build/teste automatizado | Detectar erros rapidamente | Implementação/Entrega | Devs, DevOps | A/B — prática consolidada | Moderada |
| CD (Delivery) | Continuous Delivery | DevOps | Código sempre em estado implantável, deploy manual | Reduzir risco de release | Entrega | DevOps | B | Limitada |
| CD (Deployment) | Continuous Deployment | DevOps | Deploy automático para produção sem intervenção manual | Velocidade máxima de entrega | Deploy | DevOps, SRE | C | Limitada |
| SRE | Site Reliability Engineering | Operations | Disciplina de operação com engenharia de software aplicada a confiabilidade | Garantir SLOs/SLIs | Operação/Observabilidade | SREs | B — consolidado por Google e livros | Estudo de caso, não experimental |
| IaC | Infrastructure as Code | DevOps | Infraestrutura definida e versionada como código | Reprodutibilidade de ambientes | Deploy/Operação | DevOps | B | Nenhuma acadêmica formal |
| Microservices | Microservices Architecture | Software Architecture | Estilo arquitetural com serviços pequenos e independentes | Escalabilidade e desacoplamento | Arquitetura | Arquitetos | B — padrão amplamente usado, não superior universalmente | Conflitante/contextual |
| Modular Monolith | Modular Monolith | Software Architecture | Monolito internamente modularizado por domínio | Reduzir complexidade operacional | Arquitetura | Arquitetos | C — emergente/revalorizado | Limitada |
| Vibe Coding | Vibe Coding | AI-assisted SE | Desenvolvimento via prompts de IA sem revisão detalhada do código gerado | Velocidade máxima, baixa fricção | Implementação (informal) | Devs, hobbyistas | D/E — termo informal, popular desde 2025[^11][^12] | Nenhuma acadêmica robusta, definição em disputa |
| Spec-Driven Development (IA) | Specification-Driven Development (contexto agentic) | AI-assisted SE | Workflow onde specs estruturadas guiam agentes de IA (Specify→Plan→Tasks→Implement) | Reduzir alucinação e drift de agentes | Todo o ciclo, mediado por IA | Devs, PMs, agentes | D — emergente, ferramentas datadas de 2025-2026 (GitHub Spec Kit, Kiro)[^13][^14][^15] | Early-stage, alguns papers arXiv 2026 |
| Context Engineering | Context Engineering | AI-assisted SE | Disciplina de curar informação fornecida a agentes de IA | Manter coerência de agentes em tarefas longas | Todo o ciclo com IA | Devs, arquitetos de IA | D — termo emergente 2025-2026 | Nenhuma formal, popularizado por practitioners[^16][^17] |
| ISO/IEC/IEEE 29148 | Systems and software engineering — Life cycle processes — Requirements engineering | Standards | Norma internacional sobre engenharia de requisitos, sucede IEEE 830 | Padronizar processos de requisitos | Requisitos | Organizações formais | A | Norma vigente[^8] |
| ISO/IEC/IEEE 12207 | Software life cycle processes | Standards | Framework de processos de ciclo de vida de software | Padronizar ciclo de vida | Todo o ciclo | Organizações | A — revisão 2026 publicada[^18] | Norma vigente |
| ISO/IEC/IEEE 15288 | System life cycle processes | Standards | Framework de processos de ciclo de vida de sistemas (inclui hardware) | Padronizar ciclo de vida de sistemas | Todo o ciclo (sistemas) | Engenharia de sistemas | A | Norma vigente[^19][^20] |

***

## PARTE 3 — FICHAS INDIVIDUAIS (conceitos principais)

### TDD — Test-Driven Development
- **Definição**: Prática de escrever um teste automatizado que falha antes de escrever o código de produção correspondente, seguida de implementação mínima e refatoração.[^21][^22]
- **Tradução**: Desenvolvimento Orientado a Testes.
- **Origem/Autores**: Popularizado por Kent Beck no contexto de Extreme Programming (fim dos anos 1990/2003, livro "Test-Driven Development: By Example").[^5][^23]
- **Finalidade**: Melhorar design de código e cobertura de testes através de ciclos curtos.
- **Funcionamento**: Ciclo Red → Green → Refactor, repetido para cada unidade de comportamento.[^2][^5]
- **Etapa do ciclo**: Implementação/codificação.
- **Participantes**: Desenvolvedores.
- **Relacionados**: BDD (estende TDD), XP (TDD é uma das práticas centrais), Refactoring.
- **Diferenças**: TDD opera no nível de código/unidade; BDD opera no nível de comportamento/negócio; DDD opera no nível de modelagem de domínio.[^1][^2]
- **Status/Formalização**: Prática amplamente consolidada (nível B), sem normatização ISO.
- **Evidência**: Meta-análise de Rafique & Mišić (2013, IEEE TSE) encontrou melhoria de qualidade externa com resultados inconclusivos sobre produtividade. Revisões sistemáticas posteriores (2024) confirmam ganhos de cobertura interna (~76% dos estudos) e qualidade externa (~88%), mas 44% reportam queda de produtividade.[^3][^4]
- **Fontes**: Rafique & Mišić, IEEE TSE 39(6), 2013, DOI 10.1109/TSE.2012.28; MDPI Information 13(3), 2024.[^4][^3]

### BDD — Behavior-Driven Development
- **Definição**: Abordagem que estende TDD escrevendo especificações executáveis em linguagem natural estruturada (Given/When/Then), promovendo colaboração entre negócio e engenharia.[^24][^2]
- **Origem/Autores**: Dan North, por volta de 2003, como resposta e complemento ao TDD e ao DDD de Eric Evans.[^23]
- **Finalidade**: Garantir que "a coisa certa" seja construída, alinhando requisitos e testes.
- **Funcionamento**: Cenários em Gherkin (Given/When/Then), ferramentas como Cucumber, SpecFlow, JBehave.[^24]
- **Etapa do ciclo**: Interseção Requisitos↔Testes.
- **Participantes**: Devs, QA, stakeholders de negócio.
- **Relacionados**: ATDD (frequentemente tratado como sinônimo ou subconjunto), Specification by Example, Acceptance Criteria.
- **Status**: Prática consolidada em times ágeis (nível B), sem evidência experimental robusta e independente — a maior parte da literatura é descritiva/prática, não experimental controlada.

### DDD — Domain-Driven Design
- **Definição**: Abordagem de design de software que modela o sistema espelhando o domínio de negócio, usando linguagem ubíqua compartilhada entre especialistas de domínio e desenvolvedores.[^25][^21]
- **Origem/Autores**: Eric Evans, livro "Domain-Driven Design: Tackling Complexity in the Heart of Software" (2003).[^5][^23]
- **Finalidade**: Gerenciar complexidade em domínios de negócio ricos.
- **Funcionamento**: Bounded Contexts, Entities, Value Objects, Aggregates, Domain Events, Ubiquitous Language.[^25]
- **Etapa do ciclo**: Design/Arquitetura (não é prática de teste nem de gestão de projeto).
- **Participantes**: Arquitetos, desenvolvedores seniores, especialistas de domínio.
- **Diferenças cruciais**: DDD **não é uma metodologia de desenvolvimento** — é uma abordagem/filosofia de design de arquitetura; pode coexistir com Scrum, Kanban, XP, TDD e BDD simultaneamente.[^2][^1]
- **Status**: Consolidado (B) em domínios de negócio complexos; menos aplicável (C) em sistemas simples ou CRUD.
- **Evidência**: Baseada em relatos de prática industrial e estudos de caso, não em estudos experimentais controlados comparáveis aos de TDD.

### ADR — Architecture Decision Record
- **Definição**: Documento curto e imutável que registra uma decisão arquitetural, seu contexto, alternativas consideradas e consequências.[^9][^10]
- **Origem**: Formato popularizado por Michael Nygard (2011); variante MADR também usada.
- **Finalidade**: Preservar racional de decisões para engenheiros futuros.
- **Funcionamento**: Um documento por decisão; status: Proposed/Accepted/Superseded; nunca editado após aceito — decisões revistas geram novo ADR.[^26][^10]
- **Etapa do ciclo**: Arquitetura/Design técnico, geralmente pós-decisão.
- **Relacionados**: Frequentemente precedido por um RFC interno.
- **Diferença de RFC**: RFC é pré-decisão, aberto, busca consenso; ADR é pós-decisão, fechado, registra o que foi decidido.[^27][^28][^26]
- **Status**: Prática amplamente consolidada na indústria (B), sem normatização formal ISO/IEEE.

### RFC (uso interno de engenharia, distinto do RFC do IETF)
- **Definição**: Documento de proposta técnica aberto para comentários antes de uma decisão ser tomada.[^29][^30]
- **Origem do termo**: Emprestado da cultura IETF (onde RFC significa documento de padrão já publicado, não literalmente "pedido de comentários" nesse estágio).[^26]
- **Nota crítica**: Há ambiguidade real e reconhecida entre o uso do termo "RFC" na cultura de engenharia interna (proposta pré-decisão) e o uso original do IETF (padrão publicado). Ver Parte 9.
- **Status**: Convenção de engenharia (C), não normatizada fora do contexto IETF.

### ISO/IEC/IEEE 29148 (SRS/SyRS)
- **Definição**: Norma internacional que padroniza processos e produtos de engenharia de requisitos ao longo do ciclo de vida, sucedendo IEEE 830-1998 (SRS), IEEE 1233-1998 e IEEE 1362-1998.[^8]
- **Status**: Norma internacional ativa (A), organização ISO/IEC JTC 1/SC 7 e IEEE.
- **Uso**: Define atributos de "boa" especificação de requisitos; aplicável tanto a SRS (software) quanto SyRS (sistema).

### Vibe Coding
- **Definição**: Prática de desenvolvimento assistida por IA em que o desenvolvedor descreve intenção em linguagem natural e aceita o código gerado sem revisão detalhada.[^11][^12][^31]
- **Origem/Autor**: Termo cunhado por Andrej Karpathy em post no X, 2 de fevereiro de 2025.[^12][^32][^11]
- **Finalidade**: Maximizar velocidade de prototipagem, minimizando fricção de leitura de código.
- **Status**: Termo informal/emergente (nível D/E), rapidamente incorporado a dicionários informais (Merriam-Webster listou como gíria em março de 2025). Definição ainda em disputa entre autores (ex.: Simon Willison exige "sem revisar o código" como critério definidor).[^31][^33]
- **Evidência**: Nenhuma pesquisa acadêmica робusta sobre eficácia; existe um paper arXiv (2025) tentando formalizar o conceito academicamente, mas isso é reformulação pós-hoc de um termo popular.[^33]

***

## PARTE 4 — NÍVEIS CONCEITUAIS

A pesquisa confirma que misturar esses níveis é um erro comum. Hierarquia de abstração, do mais abstrato ao mais concreto:

| Nível | Definição | Exemplos |
|---|---|---|
| **Princípio** | Ideia geral orientadora, não prescritiva | SOLID, "Ubiquitous Language" |
| **Metodologia/Framework de processo** | Conjunto estruturado de papéis, cerimônias, regras de gestão | Scrum, XP, Kanban |
| **Abordagem/Filosofia de design** | Forma de pensar estrutura do sistema, sem prescrever processo de gestão | DDD |
| **Prática** | Atividade técnica específica e repetível | TDD, Pair Programming, Refactoring, Code Review |
| **Técnica** | Método específico dentro de uma prática | Red-Green-Refactor, Gherkin/Given-When-Then |
| **Arquitetura** | Estrutura macro de um sistema | Microservices, Layered Architecture, Event-Driven Architecture |
| **Padrão arquitetural** | Solução recorrente e nomeada para um problema arquitetural | CQRS, Saga, API Gateway |
| **Documento** | Artefato escrito que registra decisão/requisito/design | PRD, ADR, SRS |
| **Especificação** | Documento formal e verificável, frequentemente normativo | SRS conforme ISO/IEC/IEEE 29148, OpenAPI Spec |
| **Requisito** | Afirmação atômica sobre o que o sistema deve fazer | "O sistema deve autenticar via OAuth2" |
| **Processo** | Sequência lógica de atividades organizacionais | Processo de release, processo de requisitos |
| **Standard** | Norma formalmente publicada por organismo reconhecido | ISO/IEC/IEEE 29148, ISO/IEC/IEEE 12207, RFC do IETF |
| **Ferramenta** | Software que operacionaliza uma prática | Cucumber (BDD), JUnit (TDD), GitHub Spec Kit |
| **Workflow** | Sequência operacional específica, muitas vezes ferramental | Specify→Plan→Tasks→Implement (Spec Kit) |
| **Termo informal** | Expressão de uso corrente sem definição formal estável | Vibe coding, "10x engineer" |

Exemplos citados no pedido, verificados:
- **TDD não está no mesmo nível conceitual de Scrum**: verdadeiro — TDD é prática técnica de codificação; Scrum é framework de gestão de processo. São ortogonais e combináveis.[^5]
- **DDD não está no mesmo nível de PRD**: verdadeiro — DDD é abordagem de design; PRD é documento de produto. Categorias completamente diferentes.[^1][^25]
- **PRD não está no mesmo nível de SRS**: parcialmente verdadeiro — ambos são documentos, mas PRD é convenção de mercado (nível C) e SRS é objeto normatizado formalmente (nível A, ISO/IEC/IEEE 29148).[^34][^8]
- **Microservices não está no mesmo nível de DevOps**: verdadeiro — Microservices é estilo arquitetural; DevOps é cultura/conjunto de práticas organizacionais que abrange CI/CD, automação e observabilidade, independentemente da arquitetura escolhida.
- **ADR não está no mesmo nível de Clean Architecture**: verdadeiro — ADR é um documento (formato de registro); Clean Architecture é um padrão arquitetural (proposta estrutural concreta de Robert C. Martin).

***

## PARTE 5 — RELAÇÕES (árvores)

```
XP (Extreme Programming)
├── TDD
├── Pair Programming
├── Continuous Integration
├── Refactoring
└── Small/Frequent Releases

BDD
├── TDD (estende)
├── Specification by Example
├── Acceptance Criteria
└── Gherkin (Given/When/Then)

DDD
├── Ubiquitous Language
├── Bounded Context
├── Aggregates
├── Entities
├── Value Objects
└── Domain Events

DevOps
├── CI/CD
├── Automation
├── Observability
├── Infrastructure as Code
└── (relaciona-se com, mas não contém) SRE

Documentação de Produto (fluxo típico, não universal)
├── MRD (mercado)
├── BRD (negócio)
├── Product Vision/Strategy
├── PRD (produto)
│   ├── User Stories
│   ├── Use Cases
│   └── Acceptance Criteria
└── Roadmap

Documentação de Arquitetura/Design
├── Architecture Document (HLD)
├── ADR (decisões pontuais)
├── RFC interno (propostas pré-decisão)
├── SDD/SDS/Technical Design Document (LLD)
└── ICD / API Specification (interfaces)

AI-assisted Software Engineering
├── Coding Assistants (autocomplete, ex: Copilot)
├── Coding Agents (autônomos, ex: Devin, Claude Code)
├── Spec-Driven Development (workflows estruturados)
├── Context Engineering (curadoria de contexto)
└── Vibe Coding (uso informal, sem revisão)
```

***

## PARTE 6 — FLUXO DO CICLO COMPLETO (não linear)

O fluxo abaixo é uma aproximação pedagógica; na prática moderna (ágil, iterativa, orientada por IA) as etapas se sobrepõem e retroalimentam constantemente, e a pesquisa desqualifica qualquer leitura estritamente sequencial/waterfall como representação fiel da prática atual.[^14]

```
Mercado          → MRD, análise competitiva
   ↓
Estratégia       → Business Case, BRD, Product Vision/Strategy
   ↓
Discovery        → UX Research, Product Discovery, Prototipagem
   ↓
Requisitos       → PRD, SRS, SyRS, FRD, User Stories, Use Cases, Acceptance Criteria
   ↓
UX               → Wireframes, Fluxos, Design System
   ↓
Arquitetura      → Architecture Document, ADR, RFC, Microservices/Monolith, DDD
   ↓
Design técnico   → SDD/SDS/Technical Design Document, HLD, LLD, ICD, API Spec
   ↓
Implementação    → TDD, Pair Programming, coding agents, vibe coding
   ↓
Testes           → BDD, ATDD, unit/integration/E2E tests
   ↓
Entrega          → CI, Continuous Delivery
   ↓
Deploy           → Continuous Deployment, IaC
   ↓
Operação         → SRE, runbooks
   ↓
Observabilidade  → Logs, métricas, traces, SLOs
   ↓
Feedback         → Analytics, suporte, pesquisa com usuário
   ↓
Evolução         → Retroalimenta Discovery/Requisitos (loop, não linha reta)
```

Note-se que TDD/BDD atuam continuamente em Implementação e Testes; DDD atua continuamente em Arquitetura e Design técnico; ADRs são produzidos em qualquer momento em que uma decisão arquitetural relevante ocorra, não apenas em uma fase fixa.

***

## PARTE 7 — DOCUMENTOS (estrutura corrigida)

A estrutura proposta no pedido está majoritariamente correta, mas a pesquisa revela que a ordem BRD→PRD não é universal: algumas fontes colocam BRD antes do MRD (decisão de investir vem antes de entender o mercado específico), enquanto outras colocam MRD antes do BRD (entender o mercado justifica o caso de negócio). Ambas as ordens são atestadas na prática; não há consenso único.[^6][^7][^35][^34]

**Mercado**: MRD — documento estratégico sobre oportunidade e demanda de mercado; não especifica funcionalidades.[^7][^34]

**Negócio**: BRD (caso de negócio formalizado, ROI, riscos, escopo) e Business Case (versão mais executiva/resumida, às vezes usada como sinônimo informal de BRD).[^35][^6]

**Produto**: Product Vision (aspiração de longo prazo), Product Strategy (como alcançar a visão), PRD (o que construir agora), Product Brief (versão condensada do PRD, comum para features pequenas), Roadmap (sequenciamento temporal).

**Requisitos**: SRS e SyRS são objetos normatizados (ISO/IEC/IEEE 29148); FRD é convenção de mercado com forte sobreposição funcional com o SRS, sem norma própria; User Stories, Use Cases e Acceptance Criteria são artefatos de granularidade mais fina, tipicamente vinculados a metodologias ágeis.

**Arquitetura**: Architecture Document (HLD, visão macro), ADR (decisões pontuais imutáveis), RFC (propostas abertas pré-decisão — nota: aqui usado no sentido de engenharia interna, distinto do RFC IETF).

**Design**: SDS/SDD/Technical Design Document funcionam como sinônimos regionais/organizacionais de "documento de design detalhado" (LLD); não há consenso terminológico único — algumas organizações usam SDD, outras SDS, outras apenas "Technical Design Doc". **Atenção à ambiguidade**: em 2026, "SDD" também significa "Spec-Driven Development" no contexto de IA — ver Parte 9.[^15][^17][^14]

**Interfaces**: ICD (forte em setores regulados como aeroespacial/defesa) e API Specification (OpenAPI é o padrão de fato da indústria de software para APIs REST).

***

## PARTE 8 — COMPARAÇÕES OBRIGATÓRIAS

### TDD × BDD × ATDD
| Dimensão | TDD | BDD | ATDD |
|---|---|---|---|
| Nível | Código/unidade | Comportamento/sistema | Critérios de aceite |
| Linguagem | Linguagem de programação | Natural (Given/When/Then) | Natural, colaborativa |
| Participantes | Devs | Devs+QA+negócio | Todo o time |
| Origem | Kent Beck, XP | Dan North, 2003[^23] | Evolução de práticas de aceite ágil |
| Relação | Base técnica | Estende TDD | Frequentemente sobreposto/sinônimo parcial de BDD[^5] |

### DDD × TDD × BDD
Ver tabela detalhada em  — DDD atua no nível de design/arquitetura (modelo de domínio); TDD no nível de implementação (correção de código); BDD no nível de especificação de comportamento. São complementares, não excludentes ou hierárquicos entre si.[^2]

### Scrum × XP × Kanban
| Dimensão | Scrum | XP | Kanban |
|---|---|---|---|
| Tipo | Framework de gestão | Metodologia técnica+gestão | Sistema de fluxo visual |
| Cadência | Sprints fixos | Iterações curtas | Fluxo contínuo |
| Foco | Papéis e cerimônias | Práticas técnicas (TDD, pairing) | Limitar WIP, visualizar fluxo |
| Prescreve práticas de código | Não | Sim | Não |

### PRD × BRD × MRD
Ver tabela completa na Parte 2 e Parte 7. Resumo: MRD = por que o mercado importa; BRD = por que o negócio deve investir; PRD = o que efetivamente construir.[^6][^7][^34]

### PRD × SRS
PRD é convenção de produto (nível C, orientado a stakeholders de negócio/design/eng, formato livre); SRS é especificação normatizada (nível A, ISO/IEC/IEEE 29148, formato formalmente definido, orientado a verificabilidade e rastreabilidade). PRD tende a ser "vivo" e atualizado semanalmente; SRS tende a ser mais estático e contratual.[^8]

### FRD × SRS
Forte sobreposição prática — muitas organizações usam os termos de forma intercambiável. Diferença nominal: FRD foca estritamente em requisitos funcionais; SRS (segundo 29148) inclui também requisitos não-funcionais, restrições e atributos de qualidade. Não há norma formal específica para FRD.

### SRS × SyRS
Ambos normatizados por ISO/IEC/IEEE 29148. SRS especifica requisitos apenas de software; SyRS especifica requisitos do sistema como um todo (incluindo hardware, operação humana, integração), sendo hierarquicamente superior quando o software é parte de um sistema maior.[^8]

### SDD × SDS × Technical Design Document
Termos amplamente sinônimos na prática de mercado para "documento de design de software", sem padronização terminológica única entre organizações. **Alerta de ambiguidade**: SDD também é usado para "Spec-Driven Development" em contextos de IA — ver Parte 9.

### HLD × LLD
HLD (High-Level Design) descreve arquitetura macro (módulos, integrações, fluxo de dados); LLD (Low-Level Design) detalha implementação (classes, algoritmos, estruturas de dados). HLD precede LLD na maior parte dos fluxos documentais tradicionais.

### ADR × RFC
Já detalhado na Parte 3: RFC é pré-decisão/aberto; ADR é pós-decisão/imutável.[^10][^9][^27][^26]

### User Story × Use Case × Requirement
| Aspecto | User Story | Use Case | Requirement (formal) |
|---|---|---|---|
| Origem | Ágil (XP/Scrum) | Engenharia de software orientada a objetos (UML) | Engenharia de requisitos clássica |
| Formato | "Como [persona], quero [ação], para [benefício]" | Fluxo de interação ator-sistema, passos alternativos | Afirmação atômica e verificável |
| Granularidade | Pequena, iterativa | Média, cobre um cenário completo | Variável, deve ser testável |
| Formalização | Baixa | Média | Alta (conforme 29148) |

### Functional × Non-Functional Requirements
Requisitos funcionais especificam o que o sistema faz (comportamento); requisitos não-funcionais especificam qualidades (performance, segurança, usabilidade, escalabilidade). Ambos são categorias formalmente reconhecidas em engenharia de requisitos e ISO/IEC/IEEE 29148.[^8]

### Microservices × Modular Monolith
Nenhuma arquitetura é objetivamente superior — a afirmação "microservices são superiores" é uma simplificação enganosa (ver Parte 11). Modular monolith reduz complexidade operacional (deploy único, sem latência de rede entre módulos); microservices favorecem escalabilidade independente e times autônomos, ao custo de complexidade operacional e de rede.

### CI × Continuous Delivery × Continuous Deployment
CI = integração/build/teste automatizado frequente; Continuous Delivery = código sempre pronto para deploy, mas deploy é decisão manual; Continuous Deployment = deploy automático para produção sem intervenção humana. São etapas progressivas de automação, não sinônimos.

### DevOps × SRE
DevOps é cultura/movimento amplo que busca integrar desenvolvimento e operações; SRE é uma implementação específica e mais prescritiva de DevOps, originada no Google, com práticas concretas (SLOs, error budgets, postmortems blameless). SRE pode ser descrito como "uma implementação de DevOps com engenharia de software aplicada a problemas de operação".

### API-First × Contract-First
Ambos os termos frequentemente usados como sinônimos na prática, mas API-First é uma filosofia organizacional (a API é o produto primário, projetada antes da implementação); Contract-First é uma prática técnica específica (o contrato/especificação — ex. OpenAPI — é escrito antes do código, servindo de fonte de verdade para geração de stubs/testes).

### Specification-Driven Development clássico × uso moderno com IA
O termo "Specification-Driven Development" clássico (pré-2023) refere-se a práticas de engenharia formal onde especificações completas precedem a implementação (próximo ao Waterfall). O uso moderno (2025-2026), popularizado por ferramentas como GitHub Spec Kit, Amazon Kiro e Augment Cosmos, refere-se a workflows onde specs estruturadas e versionadas (Specify→Plan→Tasks→Implement) guiam agentes de IA, com ciclos de feedback em minutos, não meses — uma diferença central em relação ao Waterfall clássico. É importante não confundir os dois usos.[^17][^14][^15]

***

## PARTE 9 — SIGLAS AMBÍGUAS

**RFC**
- Significado 1: Request for Comments (IETF) — documento de padrão de internet já aprovado e publicado.
- Significado 2: Request for Comments (uso interno de engenharia) — proposta técnica aberta para feedback antes da decisão.
- Contexto: o significado 1 é o original e formal; o significado 2 é uma adaptação cultural de times de engenharia de produto/software, e é ironicamente o inverso do significado 1 em termos de "abertura" do processo.[^26]
- Interpretação dominante em contexto de engenharia de produto/software (o contexto deste manual): significado 2.

**SDD**
- Significado 1: Software Design Document — artefato de design detalhado, pré-IA.
- Significado 2: Spec-Driven Development — metodologia/workflow para orquestrar agentes de IA via especificações estruturadas, termo em ascensão desde 2025-2026.[^13][^14][^15][^17]
- Interpretação dominante: depende inteiramente do contexto textual (documentação clássica vs. AI-assisted engineering); em 2026, o significado 2 está ganhando tração acelerada na literatura de IA.

**BDD**
- Significado 1 (este manual): Behavior-Driven Development.
- Significado 2 (raro, fora do escopo de software): Big Design Data / outras siglas específicas de domínio — não encontrado uso relevante conflitante na pesquisa; ambiguidade baixa neste caso.

**SRD**
- Aparece ocasionalmente como sinônimo informal de SRS (Software/System Requirements Document) em algumas fontes, mas não é um termo normatizado — deve ser tratado como variante não-oficial de SRS.[^35]

***

## PARTE 10 — ESCALA DE CONSOLIDAÇÃO

| Nível | Termos classificados |
|---|---|
| **A — padrão/norma formal** | ISO/IEC/IEEE 29148, ISO/IEC/IEEE 12207, ISO/IEC/IEEE 15288, RFC (IETF), OpenAPI Specification |
| **B — prática amplamente consolidada** | TDD, XP, Scrum, Kanban, DDD (em domínios complexos), ADR, CI, PRD (como convenção universal de produto), Microservices (como estilo, não como "melhor prática" universal) |
| **C — prática reconhecida em domínio específico** | BDD, ATDD, MRD, BRD, FRD, ICD (forte em aeroespacial/defesa), RFC interno de engenharia, API-First |
| **D — termo emergente** | Spec-Driven Development (uso IA), Context Engineering, Modular Monolith (revalorização recente), Agentic Software Engineering |
| **E — uso informal** | Vibe Coding, "10x engineer", jargões de comunidade de agentes de IA |
| **F — termo raro/pouco documentado** | FDD (Feature-Driven Development, hoje quase obsoleto), SDS como termo isolado (sempre aparece como sinônimo, raramente definido de forma independente) |

***

## PARTE 11 — MITOS E SIMPLIFICAÇÕES VERIFICADOS

- **"BDD é simplesmente TDD usando Given/When/Then"** — Simplificação parcial. BDD de fato usa essa sintaxe, mas sua diferença central é o propósito colaborativo (alinhar negócio e engenharia), não apenas a sintaxe de teste.[^24][^2]
- **"DDD é uma metodologia de desenvolvimento"** — Falso. DDD é uma abordagem de design/modelagem de domínio, não prescreve processo de gestão de projeto nem cerimônias — pode ser usado com Scrum, Kanban ou nenhum framework formal.[^25][^2]
- **"PRD sempre vem depois do BRD"** — Simplificação não universal. A pesquisa mostra ordens divergentes entre fontes (MRD→BRD→PRD é a sequência mais citada, mas não há consenso rígido, e muitas organizações pulam BRD/MRD inteiramente).[^7][^34][^6]
- **"FRD sempre vem depois do PRD"** — Convenção comum, não regra universal; muitas organizações fundem FRD e PRD em um único documento.
- **"SDD sempre significa Software Design Document"** — Falso a partir de 2025-2026; o termo é hoje genuinamente ambíguo (ver Parte 9).[^14][^17]
- **"TDD comprovadamente reduz bugs"** — Simplificação. A evidência mostra melhoria de qualidade externa em boa parte dos estudos, mas resultados sobre produtividade são inconclusivos ou até negativos em ~44% dos estudos revisados; não há consenso de causalidade forte e universal.[^3][^4]
- **"Microservices são arquitetura superior a monólitos"** — Falso como afirmação geral; é uma escolha de trade-off contextual (escalabilidade organizacional vs. simplicidade operacional), não uma hierarquia de qualidade objetiva.
- **"Agile significa não escrever documentação"** — Falso; o Manifesto Ágil valoriza "software funcionando" sobre "documentação extensa", mas não elimina documentação — apenas prioriza documentação just-enough e viva sobre documentação burocrática estática.

***

## PARTE 12 — EVIDÊNCIA CIENTÍFICA

| Prática | Nível de evidência | Estudos principais | Resultado | Limitações/Controvérsias |
|---|---|---|---|---|
| TDD | Evidência moderada, conflitante em produtividade | Rafique & Mišić (2013, IEEE TSE, meta-análise de 27 estudos)[^3][^4]; MDPI SLR (2024)[^4] | ~76-88% dos estudos reportam ganhos de qualidade interna/externa; 44% reportam queda de produtividade | Falta de rigor experimental em muitos estudos primários; efeito de "quem já testa bem" é confundido com efeito do TDD em si |
| BDD | Evidência limitada | Poucos estudos experimentais controlados isolando BDD | Benefícios relatados são majoritariamente qualitativos/de prática industrial | Ausência de meta-análises robustas comparáveis às de TDD |
| DDD | Ausência de evidência experimental adequada | Estudos de caso, relatos de prática | Consenso qualitativo entre praticantes em domínios complexos | Não há estudo controlado comparando DDD vs. não-DDD |
| Scrum/Agile | Evidência moderada, mista | Diversos estudos de adoção corporativa | Melhoria percebida em satisfação de time e velocidade de entrega em muitos relatos | Efeito de seleção (empresas que adotam Agile podem já ser mais maduras) |
| Vibe Coding/IA agentic | Ausência de evidência científica madura | Apenas papers iniciais (2025-2026, arXiv)[^13][^33] | Resultados de produtividade divulgados majoritariamente por vendors, não por pesquisa independente | Alto risco de viés de marketing; poucas revisões sistemáticas até 2026 |

***

## PARTE 13 — LINHA DO TEMPO (datas verificáveis)

- **1995**: Primeira versão de ISO/IEC 12207 (ciclo de vida de software).[^18]
- **1998**: IEEE 830-1998 (SRS), posteriormente superseded.[^8]
- **1999**: Extreme Programming formalizado por Kent Beck (livro "Extreme Programming Explained").
- **2003**: Eric Evans publica "Domain-Driven Design"; Dan North introduz conceitos de BDD como resposta/complemento.[^23][^5]
- **2008**: ISO/IEC/IEEE 12207:2008 (harmonização com 15288).[^20][^36]
- **2011**: ISO/IEC/IEEE 29148:2011 substitui IEEE 830, 1233 e 1362.[^8]
- **2013**: Rafique & Mišić publicam meta-análise sobre TDD no IEEE Transactions on Software Engineering.[^4][^3]
- **2015**: ISO/IEC/IEEE 15288:2015.[^20]
- **2017**: ISO/IEC/IEEE 12207:2017.[^37]
- **2023**: ISO/IEC/IEEE 15288:2023.[^19]
- **2 de fevereiro de 2025**: Andrej Karpathy cunha o termo "vibe coding" em post no X.[^32][^11][^12][^31]
- **Março de 2025**: Merriam-Webster lista "vibe coding" como termo de gíria.[^33]
- **2026**: Publicação de ISO/IEC/IEEE 12207:2026 (nova edição); proliferação de ferramentas de Spec-Driven Development para IA (GitHub Spec Kit, Amazon Kiro, Augment Cosmos).[^38][^18][^15][^17]

***

## PARTE 14 — DESENVOLVIMENTO DE SOFTWARE NA ERA DE IA

| Termo | Classificação | Detalhe |
|---|---|---|
| AI-assisted development | Tendência industrial consolidada | Uso de IA (autocomplete, chat, agentes) em qualquer etapa do desenvolvimento |
| Coding Assistants | Categoria de ferramenta | Ex.: GitHub Copilot — sugestão de código em tempo real, não autônomo |
| Coding Agents | Categoria de ferramenta emergente | Ex.: Devin, Claude Code — executam tarefas multi-passo com autonomia parcial |
| Spec-Driven Workflows | Metodologia emergente (D) | Specify→Plan→Tasks→Implement, formalizada por ferramentas como GitHub Spec Kit[^13][^14] |
| Context Engineering | Disciplina emergente (D) | Curadoria do que é fornecido ao contexto de um agente de IA — termo popularizado por Karpathy e outros em 2025-2026[^16][^17] |
| Vibe Coding | Termo informal/marketing-adjacente (E) | Popular, mas contestado academicamente; definição ainda instável[^11][^31][^33] |
| Agentic Software Engineering | Tendência industrial emergente | Uso de múltiplos agentes coordenados (multi-agent) para tarefas de engenharia completas[^13] |
| Software Engineering Benchmarks (ex. SWE-bench) | Conceito acadêmico | Benchmarks usados para medir capacidade de agentes de IA em tarefas reais de engenharia |
| Human-in-the-loop | Conceito acadêmico/prático | Supervisão humana obrigatória em pontos críticos do workflow de IA |

Distinção importante: **conceitos acadêmicos** (benchmarks, formalizações de vibe coding em papers) têm processo de revisão por pares; **tendências industriais** (Spec-Driven Development, Context Engineering) são impulsionadas por empresas de ferramentas e ainda carecem de validação acadêmica robusta; **marketing** inclui afirmações de produtividade de vendors sem estudos independentes (ex.: "IA responsável por 41% do código global", número não verificado por fonte acadêmica primária); **termos informais** (vibe coding) surgem de comunidades de prática antes de qualquer formalização.[^32][^33]

***

## PARTE 15 — DIFERENÇAS POR CONTEXTO

| Contexto | Como as práticas variam |
|---|---|
| Startup | Prioriza velocidade; documentação mínima; PRD leve, ADRs esparsos; vibe coding e coding agents amplamente adotados |
| SaaS | Forte adoção de CI/CD, DevOps, API-first; PRD e roadmap centrais; testing automatizado moderado |
| Enterprise | BRD/MRD formais mais comuns; governança de arquitetura via ADR/RFC estruturados; adoção mais lenta de práticas de IA agentic |
| Fintech | Requisitos formais mais rígidos (compliance), forte ênfase em testes e auditoria; SRS mais provável |
| Saúde | Regulação forte (ex. FDA, ANVISA); documentação regulada mais próxima de standards formais (ISO 13485, IEC 62304) |
| Automotivo | Uso extensivo de ISO 26262 (safety), ICD para integração de subsistemas, ênfase em Systems Engineering |
| Aviação | DO-178C (não coberto em profundidade nesta pesquisa) para software crítico; SyRS e ICD centrais |
| Defesa | Uso intensivo de MIL-STD e ICD; RFC/ADR menos comuns, documentação formal dominante |
| Governo | Processos formais de aquisição, forte uso de standards ISO/IEC/IEEE, menor adoção de práticas ágeis puras |
| Sistemas safety-critical | TDD/BDD usados, mas como complemento a verificação formal; SyRS e SRS extremamente rigorosos, rastreabilidade obrigatória |

***

## PARTE 16 — NÍVEIS DE CONHECIMENTO

**Nível Essencial** (qualquer desenvolvedor): TDD, Git/versionamento, User Stories, CI, Requisitos funcionais vs. não-funcionais, conceito básico de Scrum/Kanban — porque são a base operacional do trabalho diário de qualquer desenvolvedor, independentemente de senioridade.

**Nível Intermediário** (Tech Leads, Product Engineers): BDD, DDD, ADR, RFC, HLD/LLD, PRD vs. SRS, Microservices vs. Monolith, CI/CD completo, Context Engineering básico — porque envolvem decisões de design e comunicação entre times, exigindo maturidade além da codificação isolada.

**Nível Avançado** (Arquitetos, Staff/Principal Engineers, especialistas de qualidade e Systems Engineering): ISO/IEC/IEEE 29148/12207/15288, SyRS, ICD, DDD em escala organizacional, SRE avançado, Spec-Driven Development agentic em produção, avaliação crítica de evidência científica de práticas — porque exigem visão sistêmica, responsabilidade por normas formais e capacidade de julgar trade-offs arquiteturais de longo prazo.

***

## PARTE 17 — GLOSSÁRIO ALFABÉTICO

**ADR** — Architecture Decision Record. Registro imutável de uma decisão arquitetural, com contexto, decisão e consequências.[^9][^10]

**Agentic Software Engineering** — Uso de agentes de IA autônomos ou semi-autônomos coordenando múltiplas etapas do desenvolvimento de software.

**ATDD** — Acceptance Test-Driven Development. Prática de definir critérios de aceite colaborativamente antes de codificar.[^5]

**BDD** — Behavior-Driven Development. Especificação executável de comportamento em linguagem natural, geralmente Given/When/Then.[^24][^2]

**BRD** — Business Requirements Document. Documento que justifica o investimento de negócio em um projeto.[^6][^35]

**CI** — Continuous Integration. Integração e teste automatizado frequente de código.

**Context Engineering** — Disciplina de curar as informações fornecidas a um agente de IA para maximizar coerência e desempenho.[^16][^17]

**DDD** — Domain-Driven Design. Abordagem de design que modela software conforme o domínio de negócio.[^25][^5]

**DevOps** — Cultura/conjunto de práticas que integra desenvolvimento e operações, incluindo CI/CD, automação e observabilidade.

**FRD** — Functional Requirements Document. Documento com detalhamento de requisitos funcionais.

**HLD** — High-Level Design. Documento de design arquitetural macro.

**ICD** — Interface Control Document. Especificação de interfaces entre sistemas/subsistemas, comum em setores regulados.

**LLD** — Low-Level Design. Documento de design detalhado no nível de implementação.

**Microservices** — Estilo arquitetural baseado em serviços pequenos e independentes.

**MRD** — Market Requirements Document. Documento sobre oportunidade e necessidade de mercado.[^34][^7]

**PRD** — Product Requirements Document. Documento que define o que construir, para quem e com que métricas de sucesso.[^7][^6]

**RFC** — Request for Comments. Ambíguo: no IETF, padrão publicado; na engenharia interna, proposta pré-decisão aberta para feedback.[^26]

**Scrum** — Framework ágil de gestão iterativa baseado em sprints, papéis e cerimônias.

**SDD** — Ambíguo: Software Design Document (documentação de design) OU Spec-Driven Development (metodologia de IA orientada por especificação).[^17][^14]

**SDS** — Software Design Specification, sinônimo regional de SDD/documento de design.

**Specification by Example** — Técnica associada a BDD onde requisitos são ilustrados por exemplos concretos.

**Spec-Driven Development (contexto IA)** — Workflow onde agentes de IA implementam a partir de especificações estruturadas e versionadas.[^13][^15][^14]

**SRE** — Site Reliability Engineering. Disciplina de operação que aplica engenharia de software a problemas de confiabilidade, originada no Google.

**SRS** — Software Requirements Specification. Especificação formal de requisitos de software, normatizada por ISO/IEC/IEEE 29148.[^8]

**SyRS** — System Requirements Specification. Especificação formal de requisitos de sistema (hardware+software), também sob ISO/IEC/IEEE 29148.[^8]

**TDD** — Test-Driven Development. Prática de escrever testes antes do código, em ciclo red-green-refactor.[^22][^2]

**Vibe Coding** — Desenvolvimento assistido por IA em que o código gerado não é revisado detalhadamente pelo desenvolvedor; termo cunhado por Andrej Karpathy em fevereiro de 2025.[^11][^12][^31]

**XP** — Extreme Programming. Metodologia ágil com forte ênfase em práticas técnicas (TDD, pair programming, CI, refatoração, releases pequenos).

***

## PARTE 18 — FONTES E RASTREABILIDADE

**Standards:**
- ISO/IEC/IEEE 29148:2011 — Systems and software engineering — Life cycle processes — Requirements engineering. Status: ativo, sucede IEEE 830-1998, IEEE 1233-1998, IEEE 1362-1998.[^8]
- ISO/IEC/IEEE 12207:2017 e edição 2026 — Software life cycle processes. Status: ativo (2026 publicado em abril de 2026).[^37][^38][^18]
- ISO/IEC/IEEE 15288:2023 — System life cycle processes. Status: ativo.[^19]

**Papers/Evidência:**
- Rafique, Y.; Mišić, V. B. "The Effects of Test-Driven Development on External Quality and Productivity: A Meta-Analysis". IEEE Transactions on Software Engineering, 39(6), 2013. DOI: 10.1109/TSE.2012.28.[^3][^4]
- Systematic Literature Review sobre TDD e Test Smells. MDPI Information, 13(3):79, 2024.[^4]
- "Vibe Coding: Toward an AI-Native Paradigm for Semantic and Intent-Driven Programming". arXiv 2510.17842, 2025.[^33]
- "Spec Kit Agents: Context-Grounded Agentic Workflows". arXiv 2604.05278, 2026.[^13]

**Fontes institucionais/técnicas de referência:**
- Wikipedia, "ISO/IEC 12207" — histórico e evolução da norma.[^18]
- Wikipedia, "Vibe coding" — origem e definição do termo.[^11]
- IEEE Standards Association, páginas oficiais de 830, 12207, 15288.[^38][^37][^19][^20][^8]

Fontes de prática de mercado (não normativas, usadas para convenções de documentação de produto): makemyprd.com, vantageos.tech, clickhelp.com, blog.brucira.com — usadas exclusivamente para descrever convenções de indústria, nunca tratadas como evidência científica ou normativa.[^35][^34][^6][^7]

***

## PARTE 19 — CONTROLE DE QUALIDADE APLICADO

Verificações realizadas nesta síntese: (1) termos duplicados como "SDD" foram marcados explicitamente como ambíguos em vez de resolvidos arbitrariamente; (2) convenções de mercado (PRD, BRD, MRD, FRD) foram mantidas no nível C/B e não elevadas a "standard" (nível A), reservado apenas a normas ISO/IEC/IEEE e RFC do IETF; (3) termos emergentes de IA (vibe coding, context engineering, spec-driven development moderno) foram mantidos nos níveis D/E, não promovidos a "metodologia consolidada"; (4) afirmações de marketing sobre produtividade de IA (ex. "41% do código global") foram citadas como alegação de fonte secundária, não como fato científico estabelecido; (5) a relação entre TDD e redução de bugs foi apresentada como correlação observada em estudos, não como causalidade comprovada, dada a inconsistência entre estudos primários.[^32][^3][^4]

---

## References

1. [Test Driven Development (TDD) vs Behavior ...](https://dev.to/me_janki/test-driven-development-tdd-vs-behavior-driven-development-bdd-vs-domain-driven-design-ddd-26b7) - Software development methodologies are constantly evolving to help developers write better code...

2. [DDD vs BDD vs TDD: How They Differ and Work Together ...](https://www.linkedin.com/pulse/ddd-vs-bdd-tdd-how-differ-work-together-modern-software-queiroz-hc1ef) - In software engineering, we often encounter three popular acronyms that guide how teams design, buil...

3. [[PDF] uma revisão sistemática Test-Driven Development - UPF](https://ojs.upf.br/index.php/rbca/article/download/11154/114115861/15305354)

4. [Exploring the Connection between the TDD Practice and Test Smells—A Systematic Literature Review](https://www.mdpi.com/2073-431X/13/3/79) - Test-driven development (TDD) is an agile practice of writing test code before production code, foll...

5. [TDD, BDD, ATDD, DDD, FDD Explained - by Slawa Warda - Medium](https://medium.com/@slawa.warda/tdd-bdd-atdd-ddd-fdd-explained-principles-workflows-and-trade-offs-254d9d9033b0) - Acronyms like TDD and BDD get thrown around in dev conversations, sometimes as badges of craftsmansh...

6. [PRD vs MRD vs BRD: which document to write and when](https://www.makemyprd.com/guides/prd-vs-mrd-vs-brd) - PRD vs MRD vs BRD compared with definitions, audiences, and real examples. When to write each, what ...

7. [PRD vs BRD vs MRD: What's the Difference? - Vantage](https://www.vantageos.tech/blog/prd-vs-brd-vs-mrd) - PRD vs BRD vs MRD explained. When to use each document type, who writes them, and how they connect a...

8. [IEEE 830-1998 - IEEE SA](https://standards.ieee.org/ieee/830/1222/) - Superseded by ISO/IEC/IEEE 29148:2011. The content and qualities of a good software requirements spe...

9. [ADRs and RFCs: Their Differences and Templates](https://candost.blog/adrs-rfcs-differences-when-which/) - Differences between ADRs and RFCs in software development. When and how to use each with practical g...

10. [PRD vs ADR vs RFC: The Documents Every Engineer Should ...](https://aridanemartin.dev/blog/prd-adr-rfc-decision-documents/) - PRDs, ADRs, and RFCs each answer a different question about what you're building. Learn what each do...

11. [Vibe coding - Wikipedia](https://en.wikipedia.org/wiki/Vibe_coding)

12. [Vibe Coding: definition, Karpathy's 2025 term for AI- ...](https://www.startups.com/lexicon/vibe-coding) - Vibe coding is the practice of building software by describing intent in natural language and lettin...

13. [Spec Kit Agents: Context-Grounded Agentic Workflows](https://arxiv.org/html/2604.05278v1)

14. [Spec-Driven Development 2026: Future of AI Coding or Waterfall?](https://app.daily.dev/posts/spec-driven-development-2026-future-of-ai-coding-or-waterfall--oks3psfwf) - Spec-driven development (SDD) is a workflow where detailed specifications are written before any cod...

15. [6 Best Spec-Driven Development Tools for AI Coding in 2026](https://www.augmentcode.com/tools/best-spec-driven-development-tools) - Tested 6 spec-driven development tools for AI coding: Augment Cosmos, Kiro, GitHub Spec Kit, OpenSpe...

16. [Aligning Spec-Driven Development and Context ...](https://www.webuild-ai.com/insights/aligning-spec-driven-development-and-context-engineering-for-2026) - Are spec-driven development (SDD) and context engineering competing, or complementary - and how do w...

17. [AI SDLC 2026: How AI Agents Rewrite Development](https://www.ideaplan.io/guides/ai-sdlc-product-management-2026) - AI SDLC explained for 2026: how AI agents reshape the software development lifecycle, the new PM rol...

18. [ISO/IEC 12207 - Wikipedia](https://en.wikipedia.org/wiki/ISO/IEC_12207)

19. [IEEE/ISO/IEC 15288-2023](https://standards.ieee.org/ieee/15288/10424/) - This document establishes a common framework of process descriptions for describing the life cycle o...

20. [IEEE/ISO/IEC 15288-2015](https://standards.ieee.org/ieee/15288/5673/) - This International Standard establishes a common process framework for describing the life cycle of ...

21. [Test-Driven Development (TDD), Behavior-Driven Development (BDD), and Domain-Driven Design (DDD)…](https://medium.com/@ogounaya/test-driven-development-tdd-behavior-driven-development-bdd-and-domain-driven-design-ddd-8611cf760ad2) - Test-Driven Development (TDD)

22. [TDD, BDD, DDD: What Every Software Engineer Should Really Know](https://medium.com/@trabelsisofiane/tdd-bdd-ddd-what-every-software-engineer-should-really-know-1e6845738d58) - In the world of software development, acronyms are everywhere — and three of the most powerful (and ...

23. [TDD, BDD e DDD: entenda o que são essas siglas! - Gaea Consulting](https://gaea.com.br/tdd-bdd-ddd/) - Você conhece TDD, BDD e DDD? Se ainda não, pode estar perdendo ótimas oportunidades de crescimento p...

24. [Difference between BDD Vs TDD in Software Engineering](https://www.geeksforgeeks.org/software-engineering/difference-between-bdd-vs-tdd-in-software-engineering/) - Your All-in-One Learning Portal: GeeksforGeeks is a comprehensive educational platform that empowers...

25. [Understand the Fundamentals of TDD, DDD, and BDD: A Comprehensive Guide to these Key Software Development Concepts](https://dev.to/junedang/understand-the-fundamentals-of-tdd-ddd-and-bdd-a-comprehensive-guide-to-these-key-software-development-concepts-5aoh) - With the demand for quick response to business change in the software development world today. Many....

26. [ADR vs RFC: What's the Difference and When Should You Use ...](https://whychose.com/seo/adr-vs-rfc) - ADR vs RFC explained: RFC proposes decisions before they're made (open, seeking input); ADR records ...

27. [ADR vs RFC: When to Use Each (and How to Find Them Later) - Moai](https://askmoai.com/blogs/adr-vs-rfc) - When to Use Each (and How to Find Them Later)

28. [RFCs vs ADRs: When to Use Which - DesignDoc](https://designdoc.tech/blog/rfcs-vs-adrs) - RFCs and ADRs serve different purposes — here's how to decide which one fits your team's needs and w...

29. [RFC and ADR Culture: How Engineering Orgs Make Decisions in Writing — and Why It Scales](https://pawanjoshi.com.np/playbook/rfc-adr-culture-written-decision-making) - RFCs (Request for Comments) and ADRs (Architecture Decision Records) are the load-bearing rituals of...

30. [How to Make Architecture Decisions: RFCs, ADRs, and Getting ...](https://www.lukasniessen.com/blog/149-architecture-decision-process/) - A practical guide to making software architecture decisions - from writing RFCs to running effective...

31. [What Is Vibe Coding? A 2026 Guide for Developers](https://scrimba.com/articles/what-is-vibe-coding/) - Vibe coding means building software by describing it to an AI without reading the code. Learn where ...

32. [What Is Vibe Coding? A 2026 Field Guide for Developers](https://daily.dev/blog/vibe-coding-field-guide-2026-developers/) - Vibe coding turns English prompts into production-ready code fast, but it demands rigorous review to...

33. [Vibe Coding: Toward an AI‑Native Paradigm for Semantic and Intent ...](https://arxiv.org/html/2510.17842v1)

34. [PRD vs MRD, BRD, Tech Spec, and User Stories](https://clickhelp.com/clickhelp-technical-writing-blog/prd-vs-mrd-brd-tech-spec-and-user-stories-whats-the-difference/) - Confused about PRD, MRD, BRD, Tech Spec, and User Stories? This guide clarifies what each document i...

35. [Breaking Down BRD, PRD, SRD, and MRD - Blog | Brucira](https://blog.brucira.com/breaking-down-brd-prd-srd-and-mrd/) - Brucira explains the differences between BRD and PRD and how integral they are to product developmen...

36. [IEEE/ISO/IEC 12207-2008](https://standards.ieee.org/ieee/12207/4260/) - Supersedes Std. 12207.0 and also makes 12207.2 obsolete. This International Standard was jointly dev...

37. [ISO/IEC/IEEE 12207:2017 - Software life cycle processes](https://www.iso.org/standard/63712.html) - ISO/IEC/IEEE 12207:2017 also provides processes that can be employed for defining, controlling, and ...

38. [IEEE/ISO/IEC 12207-2026](https://standards.ieee.org/ieee/12207/11416/) - This document establishes a common process framework for describing the full life cycle of software ...

