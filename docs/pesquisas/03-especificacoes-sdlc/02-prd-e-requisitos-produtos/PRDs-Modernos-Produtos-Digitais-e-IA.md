# PRDs modernos para produtos digitais e IA: síntese acadêmica, industrial e orientada a LLMs

## Resumo executivo

Este relatório sintetiza literatura acadêmica em engenharia de requisitos, práticas recomendadas da indústria de produto e documentação oficial de provedores de IA (OpenAI, Anthropic, Google) para responder: **quais são as melhores práticas atuais para criar, escrever, validar, manter e usar PRDs (Product Requirements Documents) em produtos digitais, SaaS e plataformas de IA, em um contexto onde humanos e LLMs consomem esses documentos?**

Em engenharia de requisitos, normas e guias como IEEE/ISO sobre Software Requirements Specifications enfatizam que bons requisitos devem ser claros, completos, consistentes, rastreáveis, verificáveis e relevantes, e que requisitos mal escritos estão fortemente correlacionados com falhas de projeto, retrabalho e defeitos em produção. Estudos empíricos e revisões sistemáticas mostram que erros de requisitos (não apenas falhas de implementação) são uma fonte dominante de problemas em projetos de software, reforçando a importância de documentos de requisitos bem estruturados, ainda que “just enough” em contextos ágeis.[^1][^2][^3][^4][^5]

Na prática de produto, PRDs modernos convergem para um papel de **ferramenta de alinhamento e tomada de decisão**, não de contrato estático. Templates institucionais como os da Atlassian enfatizam contexto, objetivos, hipóteses, métricas de sucesso, escopo e fora de escopo, bem como perguntas em aberto e riscos. Conteúdos profissionais como Reforge e Lenny Rachitsky reforçam que PRDs devem ser vivos, proporcionais ao risco e integrados ao fluxo discovery–delivery, com forte foco em problema, evidências e métricas, não apenas lista de features.[^6][^7][^8][^9]

Documentação oficial de OpenAI e Anthropic sobre prompt engineering, long context e outputs estruturados oferece um conjunto de princípios diretamente aplicáveis à escrita de PRDs “IA-legíveis”: ser explícito sobre objetivos, contexto, formato de saída, critérios de sucesso, uso de exemplos e de estruturas (listas, seções, tags/XML) para separar instruções, contexto e dados. Esses materiais recomendam usar seções bem marcadas, numerar requisitos, fornecer exemplos positivos/negativos e tornar critérios de aceitação testáveis, o que coincide com as boas práticas clássicas de requisitos.[^10][^11][^12]

A partir da convergência entre academia, indústria e orientação de provedores de IA, este relatório propõe: (1) uma estrutura detalhada de PRD com seções específicas para produtos digitais e IA; (2) uma tabela de “faça/não faça”; (3) uma lista de antipadrões; (4) templates completo e enxuto, pensados tanto para humanos quanto para LLMs; (5) um checklist de qualidade graduado (0–4) e um framework passo a passo de criação; (6) um conjunto de prompts prontos para usar IA de forma segura na redação e revisão de PRDs.

## Metodologia da pesquisa

- **Fontes acadêmicas principais**: normas e guias de requisitos (IEEE/Computer Society sobre SRS, alinhado a ISO/IEC/IEEE 29148), revisões sistemáticas sobre erros de requisitos e sua relação com falhas de software, e pesquisas sobre documentação “just enough” em contextos ágeis.[^2][^3][^4][^5][^1]
- **Fontes industriais**: templates oficiais de PRD da Atlassian/Confluence (incluindo a variante traduzida e a blueprint de product requirements), materiais públicos de Reforge sobre PRDs, template de Lenny Rachitsky disponibilizado via Atlassian, e templates estruturados recentes de PRD para SaaS.[^7][^8][^9][^13][^6]
- **Fontes de IA**: documentação oficial de OpenAI sobre prompt engineering, structured outputs, function/tool calling e evals; documentação de Anthropic sobre prompt engineering, long context e uso de XML tags; guias de prompting (incluindo o GPT‑4.1 Prompting Guide).[^11][^12][^14][^10]
- **Seleção e checagem**: foram priorizadas fontes institucionais (normas, documentação oficial, portais de produto de empresas e artigos científicos revisados por pares) e, em segundo nível, fontes profissionais bem estabelecidas (Reforge, Lenny, artigos técnicos de empresas). Blogs pessoais e posts em redes sociais só são usados como contexto quando corroborados por fontes mais fortes.
- **Limitações**: o termo “PRD” varia entre empresas; muitas organizações usam design docs, product specs ou RFCs em vez de “PRD”. Onde não há fontes oficiais sobre “PRD”, o relatório infere boas práticas a partir de documentação oficial sobre requisitos, design docs e processos adjacentes, deixando essas inferências explicitamente marcadas.

## Definição e escopo de PRD

### O que é um PRD e para que serve

Na prática moderna, um **Product Requirements Document (PRD)** é um artefato estruturado que captura problema, contexto, objetivos, escopo, requisitos funcionais e não funcionais, métricas de sucesso, riscos e critérios de aceitação de uma iniciativa de produto, em um nível de detalhe suficiente para alinhar produto, design, engenharia, dados e demais stakeholders, sem substituir design docs técnicos detalhados.[^8][^6]

Normas de SRS definem especificação de requisitos como documento que descreve o que o software deve fazer e as restrições sob as quais deve operar, servindo como base de contrato entre stakeholders de negócio e equipe técnica; o PRD é uma instância mais orientada a produto desse conceito, geralmente com mais foco em problema e métricas de negócio do que uma SRS clássica.[^1]

### Quem usa um PRD

Organizações e literatura convergem em ver o PRD como um objeto compartilhado, consumido por:

- **Product managers e founders** – responsáveis por problema, trade‑offs, priorização e narrativa.[^8]
- **Engenharia (backend, frontend, plataforma, ML/IA)** – derivam arquitetura, design docs, tasks técnicas e estimativas.
- **Design (UX/UI, research)** – derivam fluxos, wireframes, protótipos e testes de usabilidade.[^6]
- **QA/Testing** – derivam suites de teste, planos de regressão e critérios de aceite.
- **Dados/Analytics** – definem tracking, eventos, métricas e dashboards alinhados ao PRD.[^6]
- **Segurança, privacidade, compliance e jurídico** – avaliam riscos, requisitos regulatórios e impactos de dados.
- **Stakeholders executivos e de negócio (vendas, CS, marketing)** – alinham expectativas de impacto, go‑to‑market e messaging.
- **Sistemas de IA (LLMs, agentes, copilotos)** – consomem o PRD como contexto para decomposição em backlog, geração de testes, análise de gaps etc., quando bem estruturado.

### Diferenças entre PRD e outros documentos

A tabela abaixo resume diferenças conceituais, combinando normas de requisitos, práticas de produto e guias de design docs.[^15][^2][^1]

| Artefato | Foco principal | Nível | Dono típico | Relação com PRD |
| --- | --- | --- | --- | --- |
| **PRD** | Problema, objetivos, requisitos de produto, métricas | Produto/feature | PM / trio produto‑design‑engenharia | Documento central de produto; referencia outros |
| **MRD (Market Requirements Doc)** | Mercado, segmentos, necessidades e oportunidades | Mercado/linha de produto | PMM / PM | Alimenta o PRD com contexto de mercado |
| **BRD (Business Requirements Doc)** | Objetivos de negócio, regras de negócio, benefícios | Negócio/processo | Business owner / BA | PRD herda e refina requisitos de negócio |
| **Especificação funcional / SRS** | Comportamento do sistema, entradas/saídas, regras | Sistema/software | BA / Eng. requisitos | PRD muitas vezes é versão “produto‑friendly” dessa SRS |
| **User stories** | Trabalho granular do usuário | Backlog/sprint | PM / time | Derivadas do PRD, com INVEST e critérios de aceite |
| **RFC** | Proposta de mudança técnica ou de protocolo | Técnica | Eng | Pode referenciar PRD ou ser referenciado por ele |
| **Design doc técnico** | Arquitetura, trade‑offs, APIs, storage | Sistema/serviço | Eng | Complemento técnico profundo do PRD |
| **Technical spec (detailed)** | Interfaces detalhadas, schemas, contratos | Componente | Eng | Detalha “como” do que o PRD descreve como “o quê” |
| **Product brief / one‑pager** | Visão condensada para alinhamento rápido | Iniciativa | PM | Versão condensada do PRD para executivos |
| **Roadmap** | Sequência de iniciativas ao longo do tempo | Portfolio | PM | PRD detalha itens do roadmap |

### Quando o PRD é útil e quando é excessivo

Evidências acadêmicas e práticas ágeis apontam para um equilíbrio entre documentação suficiente e excesso burocrático.[^3][^5][^16][^2]

**Mais útil quando**:

- Iniciativas com **alto risco** (técnico, regulatório, de UX ou de negócio) e várias equipes envolvidas.
- Produtos **SaaS B2B complexos**, com muitas integrações, SLAs e requisitos não funcionais fortes.
- Funcionalidades com impacto em **dados sensíveis, privacidade, segurança ou compliance** (ex.: LGPD, GDPR).[^1]
- Projetos com dependências cruzadas (mobile/web/backend/ML, parceiros externos, squads diferentes).
- Times distribuídos, alta rotatividade ou forte dependência de terceiros.

**Tende a ser excessivo quando**:

- Experimentos de **baixa aposta** (small bets) e testes rápidos em contexto de discovery.
- Pequenas melhorias de UI/fluxo já bem compreendidos e localizados.
- Ambientes com **Shape Up/Basecamp** onde o “pitch” ou “shaping doc” cumpre o papel de PRD enxuto.
- Times colocalizados e pequenos, trabalhando em produto pouco complexo, com ciclo rápido e forte comunicação síncrona.[^16]

### PRD em agile, lean, discovery contínuo e dual‑track

Pesquisas sobre agile requirements, documentação “just enough” e estudos em equipes Scrum/Kanban mostram que requisitos podem (e devem) ser evolutivos, com detalhamento progressivo.[^4][^3][^16]

- **Agile/Scrum/Kanban**: recomenda‑se capturar no PRD o problema, objetivos, métricas, escopo e principais requisitos/fluxos, mantendo detalhes de implementação e tarefas no backlog de user stories e tarefas técnicas. Documentação deve ser atualizada “just in time, just enough”.[^16]
- **Lean e hipothesis‑driven development**: o PRD incorpora hipóteses, experimentos, métricas e critérios de sucesso do tipo “se… então… medido por…”, conectando Discovery e Delivery.
- **Discovery contínuo / Teresa Torres**: recomenda‑se usar artefatos como Opportunity Solution Tree para mapear oportunidades, hipóteses e soluções; o PRD consolida a versão escolhida com evidências de discovery e decisões tomadas.[^9]
- **Dual‑track agile**: o PRD é frequentemente o “contrato leve” que encerra discovery para uma oportunidade e entra no track de delivery como insumo para design doc técnico e backlog.

## Visão acadêmica sobre requisitos e PRDs

### Papel dos requisitos no sucesso ou fracasso de software

Revisões sistemáticas, estudos empíricos e taxonomias de erros apontam que problemas de requisitos são uma das principais causas de falhas de software, replanejamento e estouro de custos.[^5][^3][^4]

- Walia e Carver (2009) realizaram uma **revisão sistemática** de 149 estudos e construíram uma taxonomia de erros de requisitos, conectando erros cognitivos (interpretação incorreta, omissão, simplificação excessiva, suposição não verificada) a falhas em software; concluem que focar apenas em falhas/faults é insuficiente e que é necessário atacar as **fontes de erro na fase de requisitos**.[^3]
- Estudos de impacto de engenharia de requisitos em sucesso de projeto mostram correlação positiva entre maturidade em RE (elicitação, análise, validação e gestão de mudanças) e métricas de sucesso (cumprimento de prazo, orçamento, qualidade percebida).[^4]
- Pesquisas de prática corrente mostram que em muitos contextos ágeis, backlog items têm documentação insuficiente ou inconsistente, levando a retrabalho e re‑estimativas frequentes.[^5]

**Tipos de estudo e nível de evidência**: incluem revisões sistemáticas (alto nível de evidência agregada), surveys internacionais com centenas de participantes (evidência observacional robusta), estudos de caso em organizações e experimentos controlados em menor escala.[^3][^4][^5]

### Qualidade de requisitos: clareza, completude, consistência, testabilidade

Guias da IEEE Computer Society e SWEBOK listam propriedades desejáveis de requisitos: **não ambíguos, completos, consistentes, rastreáveis, verificáveis, relevantes, atuais, factíveis e priorizados**.[^1]

- **Não ambíguos**: cada requisito deve ter uma única interpretação; evitar termos vagos como “rápido”, “fácil”, “intuitivo”.[^1]
- **Completos**: incluir todos os comportamentos pertinentes dentro do escopo definido; lacunas são fonte comum de bugs.[^3][^1]
- **Consistentes**: requisitos não devem se contradizer; inconsistências são frequentes em docs longos.[^2][^3]
- **Verificáveis/testáveis**: requisitos devem permitir teste objetivo (passa/falha), não apenas juízo subjetivo.[^1]
- **Rastreáveis**: cada requisito deve se ligar a origem (stakeholder, caso de uso, objetivo de negócio) e a artefatos derivados (design, código, testes).[^1]

Estudos sobre “requirements quality metrics” propõem métricas para avaliar corretude, completude, consistência e testabilidade, e conectam baixos escores de qualidade a maiores taxas de defeitos e esforço de retrabalho em fases posteriores.[^2]

### Limitações de documentos extensos e tensão com métodos ágeis

A literatura reconhece limitações de documentos de requisitos extensos:

- Alto custo de manutenção, tornando‑os rapidamente desatualizados.[^2]
- Baixa probabilidade de leitura completa por times ocupados; partes críticas podem ser ignoradas.[^2]
- Tendência a tratar requisitos como contrato rígido, dificultando mudança adaptativa.[^3]

Por outro lado, estudos sobre documentação em Agile mostram que equipes que abraçam o lema “working software over comprehensive documentation” de forma literal demais acabam com **backlogs pouco descritos, critérios vagos** e impactos negativos em estimativas e qualidade.[^4][^5][^16]

Trabalhos recentes sobre “just‑enough documentation for agile effort estimation” mostram, com survey de mais de 100 praticantes, que **98%** consideram documentação importante para estimar esforço e que tipos de informação mais úteis incluem: requisitos funcionais, user stories, definição de pronto, wireframes de UI, critérios de aceitação e dependências – exatamente os elementos que um bom PRD deve consolidar.[^5]

### Recomendações acadêmicas para capturar, validar e evoluir requisitos

A partir das normas e estudos revisados, emergem recomendações que se traduzem bem em práticas de PRD:

- **Iteratividade e validação contínua**: elicitar, documentar, revisar com stakeholders, prototipar e validar requisitos de forma incremental.[^3][^1]
- **Uso de cenários, casos de uso e user stories** como técnicas de captura de requisitos de interação humano‑computador.[^1]
- **Integração com testes**: uso de acceptance criteria, ATDD e BDD (Given/When/Then) para tornar requisitos testáveis e evitar ambiguidades.[^1]
- **Gestão de mudanças e rastreabilidade**: manter histórico de versões, links entre requisitos e artefatos derivados, e mecanismos de impacto de mudança.[^2][^1]
- **Foco em erros de requisitos (não só falhas)**: estudar fontes humanas de erro (memória limitada, heurísticas, vieses) e projetar processos e artefatos para mitigá‑las (checklists, templates, revisões cruzadas).[^3]

## Visão da indústria sobre criação de PRDs

### Recomendações de fontes profissionais e institucionais

#### Atlassian (Confluence Product Requirements)

A Atlassian fornece um **template oficial de requisitos de produto** em Confluence, cujo fluxo recomendado é: definir contexto e dados básicos, objetivos e métricas, hipóteses, opções/requisitos, documentação de apoio, perguntas abertas e fora de escopo.[^7][^6]

- **Contexto e meta**: tabelas iniciais para data de lançamento, status, time, links de Jira e etc.[^6]
- **Objetivo e métricas de sucesso**: seção para explicar como a iniciativa se conecta às metas da empresa e para listar métricas específicas (ex.: aumento de NPS em 15%).[^6]
- **Hipóteses e opções**: lista de suposições sobre usuários, restrições técnicas e metas de negócio, e tabela de opções de requisitos (com user story, importância, link de Jira).[^6]
- **Documentação de apoio**: anexar protótipos, diagramas, designs visuais e outros artefatos.
- **Perguntas abertas e fora de escopo**: registrar dúvidas e o que explicitamente não será feito, como forma de evitar scope creep.[^7][^6]

#### Lenny’s PRD Template (via Atlassian)

A Atlassian também distribui um template baseado na abordagem de Lenny Rachitsky, focado em “cristalizar o problema” antes da solução.[^7]

- Perguntas centrais: o que é, que problema resolve, por que isso é um problema real e vale resolver, como saberemos se resolvemos (sucesso), para quem estamos construindo, como isso aparece no produto.[^7]
- Ênfase em alinhar toda a equipe ao problema e objetivos, e retornar ao problema durante design e execução.

#### Reforge

Conteúdos da Reforge sobre PRDs enfatizam que um PRD deve ser um **documento dinâmico que evolui** com a fase do projeto, e não um pacote fechado.[^15][^8]

- Reforçam a ideia de ajustar profundidade do PRD ao estágio (exploração, validação, build, rollout) e ao risco.
- Sugerem que PRDs devem conter **decisões tomadas, trade‑offs, riscos e experimentos planejados**, além de requisitos.
- Destacam a diferença entre **PRD “de roteiro”** (mais high‑level, para planejamento) e **PRD “de entrega”** (mais detalhado, para execução).

#### Outros templates SaaS

Templates modernos de PRD para SaaS enfatizam overview, objetivos, personas, features, métricas, constraints, riscos, milestones e stakeholders, em um documento de 4–6 páginas “conciso mas completo”.[^13]

### Boas práticas recorrentes na indústria

A partir dessas fontes, algumas práticas se repetem:

- Começar pelo **problema e contexto**, não pela solução.
- Amarrar tudo a **métricas de sucesso claras e mensuráveis**.
- Ser explícito sobre **fora de escopo** para evitar escopo implícito.
- Manter **hipóteses e evidências** visíveis, com data e fonte (pesquisas, entrevistas, dados quantitativos).[^8][^6]
- Garantir que o PRD seja **linkável** a issues/tickets, design docs, dashboards e documentos de suporte.
- Tratar o PRD como **vivo**, com histórico de alterações e status, e revisá‑lo após aprendizado de rollout.[^8]

### Contrapontos: Shape Up / Basecamp

O método Shape Up da Basecamp propõe “pitches” de 6 semanas com foco em **appetite** (quanto tempo vale investir) e solução em alto nível, sem PRDs extensos.[^16]

- O pitch cumpre papel semelhante a um PRD enxuto, com problema, appetite, solução em alto nível, riscos e out of scope.
- É um exemplo de que PRD não precisa ser um “livro” – o importante é o conteúdo crítico para alinhamento e decisão.

## Recomendações do Google (implícitas) para PRDs

### Documentação pública do Google relevante

Não há um guia oficial público do Google chamado “PRD” para produto, mas há forte documentação sobre **design docs de engenharia** e boas práticas de documentação em geral.[^10][^15]

Documentos públicos relevantes incluem:

- **Design Docs at Google** (artigo de Malte Ubl) – descreve o papel central de design docs no ciclo de engenharia; estrutura típica: contexto e escopo, goals e non‑goals, design, alternativas consideradas, preocupações transversais (segurança, privacidade, observabilidade).[^15]
- **Google Developer Documentation Style Guide** – recomenda headings claros, hierarquia consistente, linguagem simples, uso de listas e tabelas quando necessário.[^10]
- Práticas de **OKRs, launch criteria, UX research e Material Design** documentadas em sites Google Developers e Google Design, que influenciam como objetivos, métricas, UX e critérios de lançamento são articulados.[^10]

### Como isso se relaciona com PRDs

A partir dessas fontes (que falam de design docs e docs técnicos, não de “PRDs” de produto), é possível derivar boas práticas aplicáveis a PRDs modernos:

- **Separar goals e non‑goals** explicitamente – isso é idêntico a objetivos e fora de escopo no PRD.[^15]
- **Focar em trade‑offs e alternativas consideradas** – registrar no PRD as opções de solução, por que foram descartadas e quais trade‑offs foram aceitos.
- **Cross‑cutting concerns** – incluir seções específicas para segurança, privacidade, observabilidade, acessibilidade e compliance.[^15]
- **Tamanho ideal** – docs suficientemente detalhados, mas lidos (10–20 páginas para grandes projetos); se o documento ficar grande demais, dividir em subproblemas e docs específicos.[^15]

É importante notar que **não há recomendação oficial pública do Google sobre PRD de produto**; as recomendações acima são **inferências** a partir da cultura de design docs de engenharia e de guias de documentação.

## Recomendações oficiais da OpenAI aplicáveis a PRDs

### Especificação clara de tarefas, prompting e outputs estruturados

A OpenAI publica guias detalhados de **prompt engineering**, outputs estruturados e function calling, que, embora voltados a prompts, carregam princípios diretamente aplicáveis à redação de PRDs legíveis por LLMs.[^12][^14][^11]

Pontos relevantes:

- **Escreva instruções claras, específicas e detalhadas** sobre contexto, objetivo, formato de saída, estilo e restrições.[^14]
- Use **delimitadores** (""", ###, tags) para separar instruções, contexto e dados de entrada.[^14]
- Forneça **exemplos** (few‑shot) para ensinar o modelo sobre formato de output, tom e edge cases.[^14]
- Especifique **comprimento e estrutura** da saída (número de itens, seções, campos obrigatórios).[^14]
- Use **structured outputs / JSON schema** para forçar formato e validade quando o output será consumido por código (analogia direta a requisitos bem estruturados).[^12]
- Em function/tool calling, recomenda‑se **nomes e descrições claras de funções, parâmetros com tipos, enums e proibição de estados inválidos** – o equivalente a requisitos precisos e tipados em um PRD.[^12]

### Evals e critérios de sucesso

Guia de evals da OpenAI recomenda descrever tarefas, definir critérios de aceitação claros e automatizáveis, e avaliar outputs contra rótulos de referência.[^10]

Isso ecoa a necessidade, em PRDs, de:

- Definir critérios de sucesso mensuráveis.
- Codificar acceptance criteria de forma verificável (por testes ou métricas). 
- Tratar o PRD como base para **E2E evals**: testes automáticos, checagens de consistência e monitoração pós‑deploy.

### Tradução para PRDs IA‑legíveis

Aplicando essas recomendações a PRDs:

- Estruturar o documento com headings e subseções estáveis, cada uma atacando um aspecto específico (contexto, requisitos, critérios, métricas).
- Ser explícito sobre formato esperado em seções críticas (por ex., requisitos numerados, critérios de aceite em Gherkin, listas numeradas).[^14]
- Adicionar exemplos de casos típicos e edge cases (“exemplos positivos/negativos”) em seções de requisitos e critérios de aceitação.
- Quando o PRD for usado pela IA para gerar backlog ou testes, fornecer **DSLs leves** (JSON, tabelas, pseudo‑Gherkin) para reduzir ambiguidade.

## Recomendações oficiais da Anthropic aplicáveis a PRDs

### Prompt engineering, XML tags e contexto longo

A documentação da Anthropic sobre prompt engineering e long context enfatiza a importância de **clareza, contexto, exemplos e estrutura com XML tags**.[^11][^12]

Pontos centrais:

- **Ser claro e direto**: instruções específicas sobre o que o modelo deve fazer e o formato do output.[^12]
- **Adicionar contexto** sobre objetivos e por que o comportamento é importante, para ajudar o modelo a generalizar corretamente.[^12]
- **Usar exemplos relevantes, diversos e estruturados**, encapsulados em `<example>` e `<examples>`.[^12]
- **Usar XML tags para separar partes**: `<instructions>`, `text>`, `<input>`, `<documents>`, `<document index="n">`, etc., especialmente em tarefas complexas com múltiplos documentos.[^12]
- Para long context, colocar documentos longos no topo e a query no fim, pedir que o modelo **cite trechos relevantes** antes de responder e usar tags para documentos e metadados.[^11]

### Tradução para PRDs IA‑legíveis

Em PRDs, isso se traduz em:

- Estruturar o documento de forma que uma IA possa mapear facilmente seções a tipos de informação (por exemplo, prefixar seções com tags ou headings consistentes: `Objetivos`, `Escopo`, `Requisitos`, `Critérios de Aceitação`).[^11]
- Em workflows com agentes, representar partes críticas em blocos estruturados (por exemplo, um bloco JSON com requisitos numerados, ou uma tabela de eventos e dados).
- Se o PRD for passado como texto bruto, considerar uma versão “machine friendly” com tags tipo `<requisitos>`, `texto_negocio>`, `riterios_aceitacao>`, para facilitar parsing por LLMs, seguindo a ideia de XML para evitar ambiguidade.[^12]

Anthropic não publica guias específicos de “PRD”, então essas são **inferências práticas** a partir de documentação oficial de prompting e long‑context.

## Como sistemas de IA leem, criam e interpretam PRDs

### Processamento de documentos longos por LLMs

Guias de OpenAI e Anthropic para long context e prompting deixam claro que LLMs:

- Processam documentos longos melhor quando estes são **estruturados em headings, listas e tabelas**, com contexto relevante no topo e instruções no fim.[^11][^10]
- Se beneficiam de **seções explicitas** indicando objetivos, restrições, exemplos e formato esperado.[^11][^14]
- Podem perder detalhes em contextos muito grandes se a informação não estiver salientada (por headings, listas numeradas, repetição sucinta de itens chave).[^11]

### Riscos na interpretação de PRDs por IA

Principais riscos:

- **Ambiguidade**: termos vagos levam o modelo a “inventar” interpretações plausíveis mas incorretas.
- **Lacunas de contexto**: se o PRD assume conhecimento não declarado (por ex., regras de negócio internas, limites técnicos), a IA preenche com suposições.
- **Contradições internas**: LLMs podem seguir qualquer parte que esteja mais próxima da consulta, ignorando seções conflitantes distantes.[^11]
- **Desalinhamento com métricas**: se objetivos e métricas não estão claros, a IA pode otimizar para proxies errados (como “minimizar escrita de código” vs. “maximizar testabilidade”).

### Como IA pode transformar PRDs em artefatos derivados

Com PRDs bem estruturados, é possível:

- Gerar **user stories** e epics a partir de requisitos funcionais numerados.
- Criar **tarefas técnicas** a partir de requisitos não funcionais, restrições técnicas e seções de dados e eventos.
- Derivar **critérios de aceitação e casos de teste** a partir das seções de critérios de aceitação, casos de borda e regras de negócio.
- Elaborar **roadmaps e planos de rollout** a partir de seções de plano de rollout, riscos e dependências.
- Gerar **prompts especializados e specs de ferramentas** usando seções de objetivo, escopo, dados e APIs.

### Como escrever PRDs para reduzir alucinações e erros da IA

- **Ser explícito e numerar requisitos**: `R1, R2…` para permitir referência estável.
- Fornecer **exemplos positivos/negativos** para requisitos complexos (por ex., como comportar‑se em determinados casos de UX ou de lógica).[^11]
- Tornar **critérios de aceitação testáveis** com Given/When/Then ou formatos tabelares claros.
- Evitar linguagem vaga; preferir quantificadores (ex.: “responder em até 2 s para 95% das requisições”) e condições específicas.
- Ter uma seção de **glossário**, reduzindo polissemia e termos internos.
- Manter uma seção de **decisões registradas** para clarificar escolhas e evitar que a IA “reabra” discussões de design.

### PRDs em workflows com agentes, RAG e coding assistants

- Em setups de **RAG**, o PRD deve ser **facilmente chunkable** – seções bem segmentadas, headings descritivos e poucos cross‑refs implícitos.
- Em agentes de coding (Cursor, Copilot, Claude Code), é útil ter uma versão reduzida do PRD com as seções mais relevantes para código (requisitos funcionais, não funcionais, APIs, dados/eventos, critérios de aceitação) em formato mais “estrutural”.
- Para **dividir PRDs grandes** sem perder rastreabilidade, manter IDs de requisito globais (`R-123`), uma matriz de rastreabilidade e links cruzados entre subdocs.

## Estrutura recomendada de PRD

A seguir, propõe‑se uma estrutura detalhada, com objetivo, conteúdo esperado, exemplo bom/ruim, como IA leria e como tornar testável/menos ambíguo. Pelas restrições de espaço, exemplos serão curtos, mas o template completo na seção de modelo consolida tudo.

### 1. Título

- **Objetivo**: identificar claramente produto/feature e contexto.
- **Escrever**: algo como “PRD – [Produto] – [Feature] – [Plataforma] – [Data]”.
- **Evitar**: títulos vagos (“Nova tela”, “Versão 2”).
- **Exemplo bom**: “PRD – Portal Parceiros – Onboarding Automatizado – Web – 2026Q3”.
- **Exemplo ruim**: “Onboarding”.
- **IA**: usa o título para classificação e busca; um bom título melhora RAG.
- **Testabilidade/clareza**: incluir escopo no título ajuda a evitar confusão com iniciativas semelhantes.

### 2. Status do documento

- **Objetivo**: informar maturidade (Draft, Em revisão, Aprovado, Obsoleto) e se pode guiar implementação.
- **Escrever**: status + data + responsáveis pela aprovação.
- **Evitar**: PRDs sem status explícito ou com status nunca atualizado.
- **IA**: pode usar status para decidir se deve propor mudanças radicais ou apenas incrementais.
- **Testabilidade**: check simples no checklist de qualidade (ver seção específica).

### 3. Owner

- **Objetivo**: apontar responsável final pela integridade do PRD.
- **Escrever**: nome, função, contato.
- **Evitar**: múltiplos owners pouco claros.

### 4. Data e versão

- **Objetivo**: versionamento mínimo.
- **Escrever**: data de criação, última atualização, versão semantic (ex.: 0.3, 1.0).
- **IA**: pode comparar versões, gerar diff e resumo de alterações.

### 5. Resumo executivo

- **Objetivo**: dar visão rápida a executivos/eng/dados.
- **Conteúdo**: problema, solução em alto nível, impacto esperado, principais riscos.
- **Exemplo bom**: 3–6 frases objetivas, com métricas alvo.
- **Exemplo ruim**: narrativa longa sem números.
- **IA**: usa essa seção para contexto rápido; boa para few‑shot em prompts.

### 6. Problema

- **Objetivo**: explicitar dor ou oportunidade.
- **Conteúdo**: situação atual, impacto negativo, tamanho do problema.
- **Evitar**: confundir problema com solução (“precisamos de um chatbot”).
- **Testabilidade**: vincular a métricas atuais (por ex., tempo médio de onboarding, churn na etapa X).

### 7. Contexto

- **Objetivo**: colocar problema em contexto de negócio, técnico e organizacional.
- **Conteúdo**: produto, segmento, histórico de tentativas, constraints macro.
- **IA**: ajuda a evitar soluções incoerentes com o restante do sistema.

### 8. Objetivos

- **Objetivo**: definir o que queremos alcançar; idealmente em formato SMART.
- **Conteúdo**: lista numerada de objetivos de negócio e de usuário.
- **Testabilidade**: objetivos devem mapear em métricas (seção seguinte).

### 9. Não objetivos (fora de escopo de objetivo)

- **Objetivo**: esclarecer o que poderia ser objetivo, mas não será agora.
- **IA**: evita “expandir” o escopo nos outputs (user stories, código) para áreas não desejadas.

### 10. Usuários/personas

- **Objetivo**: clarificar para quem é o produto/feature.
- **Conteúdo**: 2–4 personas chave com trabalho, contexto e necessidades relevantes.
- **Testabilidade**: usar personas em critérios de aceitação (por ex., “para persona X deverá ocorrer Y”).

### 11. Jobs to be done

- **Objetivo**: ligar personas a jobs específicos.
- **Conteúdo**: frases no formato “Quando [situação], quero [motivação], para [resultado esperado]”.
- **IA**: transforma facilmente JTBD em user stories.

### 12. Evidências de discovery

- **Objetivo**: documentar dados que sustentam o problema/solução.
- **Conteúdo**: pesquisas, entrevistas, experimentos anteriores, análises quantitativas.
- **Testabilidade**: cada evidência com fonte, data, N, link.

### 13. Hipóteses

- **Objetivo**: explicitar suposições testáveis.
- **Conteúdo**: formato “Acreditamos que [mudança] levará a [impacto] para [segmento], medido por [métrica]”.
- **IA**: pode derivar experiências A/B e planos de medição.

### 14. Métricas de sucesso

- **Objetivo**: tornar sucesso verificável.
- **Conteúdo**: lista de métricas (leading e lagging), baseline e alvo.
- **Testabilidade**: liga diretamente a dashboards e tracking.

### 15. Escopo

- **Objetivo**: dizer o que será incluído.
- **Conteúdo**: lista de capacidades e domínios incluídos; relacionar a requisitos.

### 16. Fora de escopo

- **Objetivo**: prevenir escopo implícito/creep.
- **Conteúdo**: funcionalidades e domínios explicitamente excluídos.

### 17. Requisitos funcionais

- **Objetivo**: listar comportamentos do sistema.
- **Conteúdo**: requisitos numerados (RF‑1, RF‑2…), com descrição clara, trigger, comportamento esperado e resultado.
- **Exemplo bom**: “RF‑3 – Ao aprovar um parceiro, o sistema deve enviar e‑mail de boas‑vindas com link único de ativação, válido por 7 dias, apenas para usuários com status ‘aprovado’.”
- **Exemplo ruim**: “Enviar e‑mail para parceiros”.
- **IA**: facilmente transformável em user stories, tasks, testes.

### 18. Requisitos não funcionais

- **Objetivo**: capturar qualidades e constraints (performance, disponibilidade, segurança, usabilidade).
- **Conteúdo**: RNF‑1…n com valores mensuráveis.

### 19. UX / fluxo do usuário

- **Objetivo**: descrever journeys chave.
- **Conteúdo**: fluxos principais, wireframes, estados vazios, mensagens de erro; links para Figma etc.

### 20. Regras de negócio

- **Objetivo**: explicitar regras determinísticas e elegibilidade.
- **Conteúdo**: RB‑1…n, preferencialmente em forma tabular.

### 21. Dados e eventos

- **Objetivo**: descrever entidades, campos críticos, eventos de domínio e tracking.
- **Conteúdo**: tabelas de entidades, atributos e eventos (ex.: `partner_created`, `partner_activated`).

### 22. Analytics

- **Objetivo**: definir instrumentação.
- **Conteúdo**: eventos, propriedades, funis e dashboards desejados.

### 23. Segurança, privacidade e compliance

- **Objetivo**: capturar requisitos legais e de risco.
- **Conteúdo**: tipos de dados pessoais tratados, base legal, retenção, controles de acesso, logs, cifragem.

### 24. Acessibilidade

- **Objetivo**: garantir inclusão (WCAG, etc.).
- **Conteúdo**: requisitos como foco visível, navegação via teclado, contrastes, labels ARIA.

### 25. Internacionalização/localização

- **Objetivo**: definir responsabilidades de idioma, formatos e fuso horário.

### 26. Dependências

- **Objetivo**: listar dependências internas e externas (sistemas, squads, vendors).

### 27. Restrições técnicas

- **Objetivo**: explicitar decisões já tomadas (stack, padrões obrigatórios, limites de infra).

### 28. Critérios de aceitação

- **Objetivo**: tornar verificação objetiva.
- **Conteúdo**: Given/When/Then por requisito ou por user story, ou tabela de critérios.

### 29. Casos de borda

- **Objetivo**: evitar “happy path only”.
- **Conteúdo**: lista de edge cases e comportamento esperado.

### 30. Riscos

- **Objetivo**: explicitar incertezas e risco residual.
- **Conteúdo**: tabela com probabilidade, impacto, mitigação.

### 31. Trade‑offs

- **Objetivo**: documentar decisões de trade‑off (por ex., time‑to‑market vs. escalabilidade).

### 32. Plano de rollout

- **Objetivo**: planejar lançamento seguro.
- **Conteúdo**: fases, feature flags, cohorts, rollback plan.

### 33. Experimentos ou testes A/B

- **Objetivo**: desenhar experimentos necessários para validar hipóteses.

### 34. Plano de QA

- **Objetivo**: definir escopo e foco do QA (tipos de teste, ambientes, responsabilidades).

### 35. Comunicação e lançamento

- **Objetivo**: alinhar GTM, comunicação interna e externa.

### 36. Perguntas abertas

- **Objetivo**: visibilizar dúvidas ainda não resolvidas.

### 37. Decisões registradas

- **Objetivo**: manter um log de decisões chave (tipo ADR de produto).

### 38. Histórico de alterações

- **Objetivo**: rastreabilidade de evolução do PRD.

### 39. Links e referências

- **Objetivo**: centralizar links para docs, boards, protótipos, specs técnicas.

Por questão de espaço, os exemplos bom/ruim e notas de IA para cada seção são consolidados nos templates práticos (seções seguintes).

## O que fazer e o que não fazer em PRDs

Tabela de alto nível.

| Faça | Não faça |
| --- | --- |
| Descreva problema, contexto e objetivos antes da solução | Começar pela tela ou API que quer construir |
| Use linguagem clara, termos definidos e requisitos numerados | Usar termos vagos (“rápido”, “intuitivo”) sem definição |
| Amarre cada objetivo a métricas de sucesso mensuráveis | Definir sucesso como “melhor experiência” sem indicador |
| Mantenha escopo e fora de escopo explícitos | Deixar escopo implícito, confiando em “bom senso” |
| Priorize requisitos (must/should/could) | Tratar tudo como prioridade máxima |
| Escreva critérios de aceitação testáveis (Given/When/Then) | Deixar apenas descrições gerais de comportamento esperado |
| Documente restrições técnicas e dependências reais | Especificar implementação detalhada de engenharia no PRD de produto |
| Inclua seções de UX, fluxos, estados vazios e erros | Ignorar UX e estados de erro, focando só no happy path |
| Envolva stakeholders chaves na revisão (eng, design, dados, segurança) | Escrever o PRD isoladamente e “jogar por cima do muro” |
| Versione o documento, registre decisões, mantenha histórico de alterações | “Atualizar em silêncio”, mudando requisitos sem registro |
| Use IA como coautor para rascunho, revisão e checagem de consistência | Deixar IA gerar PRD final sem revisão humana |
| Alimente IA com PRD bem estruturado para derivar backlog, testes e docs | Passar um PRD caótico e esperar backlog perfeito |
| Vincule requisitos a evidências e hipóteses | Declarar requisitos sem base empírica clara |
| Mantenha PRD vivo, atualizado conforme decisões e descobertas | Tratar PRD como arquivo morto, sem manutenção |
| Documente riscos, trade‑offs e planos de mitigação | Ignorar riscos e pressupor que tudo “vai dar certo” |

## Antipadrões comuns em PRDs

Para cada antipadrão: problema, como identificar, como corrigir, consequência se ignorado.

1. **PRD longo demais e pouco usado**
   - Problema: documento de 30+ páginas, raramente lido; contém detalhes irrelevantes e duplicações.
   - Identificar: time não sabe resumir o PRD, ninguém consegue apontar seção de critérios de aceitação ou métricas.
   - Corrigir: extrair um resumo executivo forte, mover detalhes técnicos para design docs, criar versões enxutas por público.
   - Consequência: desalinhamento prático, decisões sendo tomadas fora do doc, alto custo de manutenção.

2. **PRD curto demais e ambíguo**
   - Problema: “mini PRD” com uma página dizendo “criar tela de X”, sem objetivos, critérios, fluxos ou edge cases.
   - Identificar: perguntas frequentes de engenharia/dados/QA para “descobrir” o que fazer.
   - Corrigir: adicionar seções mínimas (problema, objetivos, escopo, requisitos numerados, critérios de aceite, métricas, riscos).
   - Consequência: divergência de implementação, retrabalho, bugs e frustração.

3. **PRD que descreve solução sem explicar problema**
   - Problema: doc centrado em wireframes, rotas e APIs, sem chamar claramente o problema a ser resolvido.
   - Identificar: se você puder trocar o problema por outro sem mudar o PRD, algo está errado.
   - Corrigir: adicionar seção de problema, evidências e JTBD; revisar solução à luz disso.
   - Consequência: priorização ruim, risco de product/market misfit incremental.

4. **PRD sem métrica de sucesso**
   - Problema: sucesso definido apenas como “lançar feature”.
   - Identificar: ausência de métricas na seção de objetivos.
   - Corrigir: definir 2–3 métricas com baseline e target, ligadas a objetivos de negócio.
   - Consequência: impossibilidade de avaliar impacto, decisões políticas sobre continuar/encerrar.

5. **PRD sem critério de aceite**
   - Problema: difícil saber quando a feature está “pronta”; discussões subjetivas.
   - Corrigir: escrever critérios Given/When/Then por fluxo principal.
   - Consequência: testes incompletos, disputas entre produto e eng na hora da entrega.

6. **PRD sem não escopo**
   - Problema: cada stakeholder assume que o seu desejo está incluso implicitamente.
   - Corrigir: adicionar seção de fora de escopo, mantê‑la atualizada.
   - Consequência: scope creep, atrasos e desgaste.

7. **PRD sem evidência de usuário**
   - Problema: requisitos nascem de opiniões internas, não de dados ou pesquisa.
   - Corrigir: documentar discovery, mesmo que pequeno (análises, entrevistas, feedback de suporte).
   - Consequência: features pouco usadas, desperdício de esforço.

8. **PRD que mistura opinião, decisão e requisito**
   - Problema: frases como “devemos usar micro‑frontend porque é moderno” misturadas com “o sistema deve…”
   - Corrigir: separar requisitos (o que) de decisões e racional (por que), usando seção própria de decisões.
   - Consequência: confusão sobre o que é negociável e o que é constraint real.

9. **PRD que ignora engenharia**
   - Problema: decisões tomadas sem input técnico; restrições irreais.
   - Corrigir: incluir design/eng na revisar inicial, registrar constraints técnicos reais.
   - Consequência: soluções inviáveis, retrabalho em massa.

10. **PRD que ignora UX**
   - Problema: foco em APIs e backoffice, sem fluxos de usuário ou estados de erro.
   - Corrigir: incluir seção de UX, fluxos, wireframes ou pelo menos descrições textuais.
   - Consequência: interface confusa, alta carga de suporte.

11. **PRD que ignora dados e analytics**
   - Problema: feature sem eventos instrumentados; difícil medir sucesso.
   - Corrigir: seção de analytics com eventos, propriedades e dashboards.
   - Consequência: decisões cegas, incapacidade de rodar experimentos.

12. **PRD que ignora privacidade, segurança e compliance**
   - Problema: dados sensíveis tratados sem análise de risco (LGPD, GDPR, PCI etc.).
   - Corrigir: incluir seção específica, envolver time de segurança/jurídico quando necessário.
   - Consequência: risco legal e reputacional significativo.

13. **PRD criado só para “alimentar IA” sem validação humana**
   - Problema: PRD gerado automaticamente, sem revisão de stakeholders.
   - Corrigir: tratar IA como assistente, não dono; revisar criticamente com produto, eng, design.
   - Consequência: requisitos incoerentes, gaps importantes não capturados.

14. **PRD gerado por IA sem revisão crítica**
   - Similar ao anterior; risco de “falsa completude” e requisitos inventados.

15. **PRD com requisitos contraditórios**
   - Problema: seções diferentes exigem comportamentos incompatíveis.
   - Corrigir: revisão manual + uso de IA para detectar inconsistências.
   - Consequência: decisões ad hoc em implementação, bugs sutis.

16. **PRD sem rastreabilidade**
   - Problema: não se sabe qual requisito gerou qual user story/teste; difícil avaliar impacto de mudanças.
   - Corrigir: numerar requisitos, manter matriz de rastreabilidade leve.

17. **PRD sem plano de atualização**
   - Problema: doc fica obsoleto; decisões recentes não refletidas.
   - Corrigir: combinar com time cadência de review (por release ou grande decisão).

## Modelo prático de PRD (completo)

Abaixo, um template em português brasileiro, adaptável para SaaS, produtos digitais e features, pensado para uso humano e por IA. Colchetes indicam campos preenchíveis.

```markdown
# PRD – [Produto] – [Feature] – [Plataforma] – [Data]

## 1. Metadados
- Status: [Draft | Em revisão | Aprovado | Obsoleto]
- Versão: [v0.3]
- Owner: [Nome, função, contato]
- Co‑owners: [Design, Eng, Data]
- Data de criação: [YYYY‑MM‑DD]
- Última atualização: [YYYY‑MM‑DD]

## 2. Resumo executivo
[2–5 frases sobre problema, solução em alto nível, impacto esperado, principais riscos.]

## 3. Problema
- Situação atual: [descrever]
- Impacto: [métricas atuais, dores de usuários, custos]
- Quem é afetado: [personas/segmentos]

## 4. Contexto
- Produto/área: [ex.: Portal Parceiros]
- Histórico: [tentativas anteriores, decisões relevantes]
- Restrições macro: [regulatórias, técnicas, organizacionais]

## 5. Objetivos
- OBJ‑1: [objetivo de negócio mensurável]
- OBJ‑2: [...]

## 6. Não objetivos
- NOBJ‑1: [ex.: não vamos suportar [canal/plataforma] neste release]

## 7. Usuários e personas
- Persona 1: [nome, descrição curta, necessidades relevantes]
- Persona 2: [...]

## 8. Jobs to be done
- JTBD‑1: Quando [situação], quero [motivação], para [resultado]
- JTBD‑2: ...

## 9. Evidências de discovery
- [Tipo] – [Resumo] – [Data] – [Fonte/link]

## 10. Hipóteses
- H‑1: Acreditamos que [mudança] levará a [impacto] para [segmento], medido por [métrica].

## 11. Métricas de sucesso
- Métrica 1: [nome] – baseline: [X] – alvo: [Y] – janela: [período]
- Métrica 2: ...

## 12. Escopo
- Inclusões:
  - [Capacidade 1]
  - [Capacidade 2]

## 13. Fora de escopo
- Exclusões:
  - [Item 1]
  - [Item 2]

## 14. Requisitos funcionais
- RF‑1: [Descrição clara, com trigger e resultado]
- RF‑2: ...

## 15. Requisitos não funcionais
- RNF‑1 (Performance): [ex.: resposta < 2 s para 95% das requisições]
- RNF‑2 (Disponibilidade): [...]
- RNF‑3 (Segurança): [...]

## 16. UX e fluxos
- Fluxo principal: [descrição + link para protótipo]
- Fluxos alternativos: [...]
- Estados de erro / vazios: [...]

## 17. Regras de negócio
- RB‑1: [regra]
- RB‑2: ...

## 18. Dados e eventos
- Entidades principais: [lista + campos críticos]
- Eventos de domínio: [nome, quando dispara, payload]
- Eventos de tracking: [nome, propriedades, onde dispara]

## 19. Analytics
- Funis: [descrição]
- Dashboards desejados: [...]

## 20. Segurança, privacidade e compliance
- Dados pessoais tratados: [quais]
- Base legal / políticas: [...]
- Controles: [acesso, logs, cifra]

## 21. Acessibilidade
- Requisitos: [WCAG, navegação por teclado, etc.]

## 22. Internacionalização/localização
- Idiomas suportados: [...]
- Formatos (data, moeda, etc.): [...]

## 23. Dependências
- Dependências internas: [squad/sistema]
- Dependências externas: [fornecedores, APIs de terceiros]

## 24. Restrições técnicas
- Stack obrigatória: [...]
- Limitações de infra: [...]

## 25. Critérios de aceitação
Para cada RF:
- RF‑1:
  - CA‑1.1: Given [contexto], When [ação], Then [resultado esperado].

## 26. Casos de borda
- CE‑1: [descrição + comportamento esperado]

## 27. Riscos e mitigação
- R‑1: [risco] – Prob: [Baixa/Média/Alta] – Impacto: [B/M/A] – Mitigação: [...]

## 28. Trade‑offs
- T‑1: [decisão de trade‑off + rationale]

## 29. Plano de rollout
- Estratégia: [flag, beta, big bang]
- Fases: [lista]
- Rollback plan: [...]

## 30. Experimentos / testes A/B
- Experimento 1: [hipótese, métrica, segmento, desenho]

## 31. Plano de QA
- Tipos de teste: [unitário, integração, E2E, carga]
- Escopo específico desta iniciativa: [...]

## 32. Comunicação e lançamento
- Interno: [treinamentos, anúncios]
- Externo: [release notes, campanha]

## 33. Perguntas abertas
- P‑1: [pergunta]

## 34. Decisões registradas
- D‑1: [decisão, data, participantes]

## 35. Histórico de alterações
- [Data] – [Versão] – [Resumo da mudança] – [Autor]

## 36. Links e referências
- Jira: [...]
- Figma: [...]
- Design doc técnico: [...]
- Outros: [...]
```

## Template enxuto

Para startups ou features pequenas, um PRD enxuto pode ter 2–4 páginas, mantendo apenas as seções essenciais:

```markdown
# PRD Enxuto – [Produto] – [Feature]

## 1. Metadados
Status, versão, owner, datas.

## 2. Problema
[O que dói, para quem, por quê agora.]

## 3. Objetivos e métricas
- OBJ‑1: [...]
- Métricas: [...]

## 4. Usuários / personas
[1–2 personas]

## 5. Escopo / fora de escopo
[Listas curtas]

## 6. Solução em alto nível
[Resumo da abordagem, fluxos principais]

## 7. Requisitos chave
- RF‑1…RF‑n
- RNF mais críticos

## 8. Critérios de aceitação
[Given/When/Then principais]

## 9. Riscos principais
[Listar 3–5]

## 10. Links
[Jira, design, etc.]
```

Para ambientes regulados ou alto risco, usar o template completo e, se necessário, anexar SRS detalhada e documentos de compliance.

## Checklist de avaliação de PRD

Proposta de checklist com escala 0–4 para cada critério (0 = ausente, 1 = fraco, 2 = aceitável, 3 = bom, 4 = excelente).

Critérios (exemplos):

1. **Clareza** – linguagem objetiva, sem ambiguidades.
2. **Completude mínima** – seções essenciais presentes (problema, objetivos, escopo, RFs, critérios de aceitação, métricas, riscos).
3. **Testabilidade** – critérios de aceitação e métricas permitem verificar sucesso.
4. **Priorização** – requisitos e escopo com noção de prioridade.
5. **Rastreabilidade** – requisitos numerados, links a tickets/tests.
6. **Alinhamento ao problema** – requisitos claramente conectados ao problema e evidências.
7. **Alinhamento a métricas** – objetivos e métricas bem ligados.
8. **Compreensibilidade para engenharia** – requisitos e constraints técnicos suficientes.
9. **Compreensibilidade para design** – contexto, personas, fluxos de usuário.
10. **Compreensibilidade para QA** – critérios de aceitação e casos de borda.
11. **Legibilidade por IA** – estrutura, headings, numeração, exemplos.
12. **Ausência de contradições** – sem conflitos aparentes.
13. **Ausência de linguagem vaga** – termos vagos identificados e tratados.
14. **Atualização** – datas, versão e histórico de alterações.
15. **Privacidade/compliance** – riscos e requisitos de dados endereçados.

Interpretação da pontuação final (somar, por exemplo, 15 critérios x 0–4 = 0–60):

- 0–20: **PRD insuficiente** – precisa de reescrita substancial.
- 21–35: **PRD fraco** – usar com cuidado, alto risco de desalinhamento.
- 36–48: **PRD bom** – adequado para uso, ainda há pontos de melhoria.
- 49–60: **PRD excelente** – bem estruturado, deve servir como referência.

Times podem adaptar pesos por critério conforme contexto (por ex., dar mais peso a privacidade em fintech/health). 

## Framework passo a passo para criar PRDs

1. **Definir o problema** – consolidar dores de usuários e gaps de negócio; usar dados de suporte.
2. **Definir objetivos de negócio** – conectar problema a resultados desejados (receita, churn, eficiência etc.).
3. **Identificar usuários** – personas e segmentos afetados.
4. **Coletar evidências** – discovery qualitativo e quantitativo; registrar na seção específica.
5. **Mapear jobs, dores e cenários** – JTBD, fluxos principais, contextos de uso.
6. **Definir hipóteses** – o que se acredita que a solução fará.
7. **Definir solução em nível adequado** – alto nível no PRD; detalhamento técnico em design docs.
8. **Definir escopo e não escopo** – delimitar o que entra e o que não entra.
9. **Definir requisitos (funcionais e não funcionais)** – numerados, claros, ligados aos objetivos.
10. **Definir critérios de aceitação** – por RF/fluxo principal.
11. **Definir métricas de sucesso** – baseline, targets, janelas de medição.
12. **Revisar com design** – garantir coerência UX, fluxos e estados.
13. **Revisar com engenharia** – validar viabilidade técnica e restrições.
14. **Revisar com dados** – garantir tracking e capacidade de medir impacto.
15. **Revisar com segurança/compliance (quando aplicável)** – avaliar riscos regulatórios.
16. **Validar com stakeholders** – review com lideranças de produto, negócio e times afetados.
17. **Gerar backlog** – decompor RFs em epics/stories, manter links para PRD.
18. **Criar plano de rollout** – fases, flags, rollback.
19. **Atualizar o PRD com decisões** – registrar trade‑offs, mudanças de escopo.
20. **Usar IA para revisão, decomposição e checagem, não como autoridade final** – rodar check de inconsistência, gerar listas derivadas, mas manter revisão humana.

## Como usar IA para criar e revisar PRDs

### Usos produtivos e seguros

- **Gerar primeiro rascunho**: fornecer contexto (problema, objetivos, dados brutos) e pedir estrutura inicial de PRD.
- **Transformar notas de discovery em PRD**: colar notas de entrevistas, análises e pedir síntese estruturada.
- **Identificar lacunas**: pedir à IA que liste perguntas em aberto, requisitos ausentes, edge cases não cobertos.
- **Encontrar ambiguidade**: pedir para localizar termos vagos e sugerir reformulações.
- **Criar critérios de aceitação**: derivar Given/When/Then a partir de RFs.
- **Criar user stories e casos de teste**: gerar backlog inicial e testes baseados no PRD.
- **Criar perguntas para engenharia**: a IA pode listar pontos a confirmar com times técnicos.
- **Criar plano de tracking**: derivar eventos, propriedades e funis a partir de seções de dados e métricas.
- **Gerar documentação auxiliar**: FAQs, docs de suporte, treinamento interno.
- **Comparar versões de PRD**: gerar diff textual e resumo de alterações.
- **Verificar consistência interna**: checar conflitos entre requisitos, objetivos, escopo e critérios.

### Riscos

- **Alucinação**: invenção de requisitos, constraints ou dados não presentes.
- **Falsa completude**: doc “polido” mas com buracos significativos.
- **Requisitos inventados**: IA preenchendo lacunas sem marcar que são suposições.
- **Omissão de restrições**: ignorar limitações legais ou técnicas.
- **Vazamento de dados sensíveis**: colar dados de produção ou informações confidenciais em prompts sem ambiente seguro.
- **Confusão entre requisito e sugestão**: IA propor design como se fosse requisito obrigatório.
- **Excesso de confiança**: aceitar PRD gerado sem revisão.

### Prompts prontos (ajustar à sua realidade)

1. **Prompt para gerar PRD a partir de briefing**

```text
Você é um(a) PM experiente em produtos SaaS B2B.

Tenho o seguinte briefing de produto (entre aspas):
"""
[cole aqui notas de discovery, objetivos de negócio, contexto]
"""

Quero que você produza um PRD em português brasileiro, estruturado nas seções:
- Metadados (pode sugerir placeholders)
- Resumo executivo
- Problema
- Contexto
- Objetivos
- Não objetivos
- Usuários/personas
- Jobs to be done
- Evidências de discovery
- Hipóteses
- Métricas de sucesso
- Escopo / Fora de escopo
- Requisitos funcionais (RF‑1, RF‑2...)
- Requisitos não funcionais (RNF‑1, RNF‑2...)
- UX / fluxos (em alto nível)
- Regras de negócio
- Dados e eventos
- Analytics
- Segurança, privacidade e compliance (só se aplicável)
- Dependências e restrições técnicas
- Critérios de aceitação (Given/When/Then principais)
- Casos de borda
- Riscos e trade‑offs
- Plano de rollout
- Perguntas abertas
- Decisões sugeridas (marque claramente como sugestões)

Não invente números nem fatos; se algo não estiver claro, inclua na seção "Perguntas abertas".
Use linguagem objetiva e numerar requisitos.
```

2. **Prompt para revisar PRD**

```text
Você é um(a) especialista em engenharia de requisitos.

Segue um PRD entre aspas:
"""
[cole o PRD]
"""

Tarefas:
1) Liste ambiguidades, termos vagos ou subjetivos e sugira reformulações mais testáveis.
2) Aponte requisitos contraditórios ou potencialmente conflitantes.
3) Indique lacunas óbvias nas seções (problema, objetivos, métricas, critérios de aceitação, riscos, segurança).
4) Gere uma lista de perguntas que o time deveria responder antes de aprovar este PRD.

Responda em tópicos claros.
```

3. **Prompt para encontrar lacunas**

```text
Considere o PRD abaixo:
"""
[PRD]
"""

Liste de forma estruturada:
- Requisitos que não têm critérios de aceitação claros.
- Objetivos que não têm métricas associadas.
- Áreas típicas de risco (segurança, privacidade, dados, performance, acessibilidade) que não são mencionadas.
- Fluxos de usuário que parecem incompletos ou sem casos de erro.

Não crie requisitos novos; apenas destaque lacunas.
```

4. **Prompt para transformar PRD em backlog**

```text
Você é um(a) product owner.

A partir do PRD abaixo, gere um backlog inicial em formato de user stories no padrão:
"Como [tipo de usuário], quero [objetivo] para [benefício]".

Inclua para cada story:
- Título
- Descrição (user story)
- Requisitos relacionados (RF‑x)
- Critérios de aceitação (Given/When/Then)
- Tipo (feature, spike, tarefa técnica)
- Prioridade sugerida (Alta/Média/Baixa)

PRD:
"""
[PRD]
"""

Não invente funcionalidades além das presentes no PRD.
```

5. **Prompt para criar critérios de aceite**

```text
A partir dos requisitos funcionais abaixo, crie critérios de aceitação no formato Given/When/Then:

[lista RF‑1...n]

Para cada RF, forneça 2–5 cenários cobrindo happy path e pelo menos um caso de erro.
```

6. **Prompt para criar plano de QA**

```text
Você é um(a) QA lead.

Dado o PRD abaixo, produza um plano de QA focado nesta feature:
"""
[PRD]
"""

Inclua:
- Escopo de testes (o que entra / sai)
- Tipos de teste necessários (unitário, integração, E2E, carga, segurança)
- Principais casos de teste E2E (em alto nível)
- Riscos de qualidade e estratégias de mitigação

Não replique o PRD; foque em como testar que os requisitos foram atendidos.
```

7. **Prompt para avaliar se o PRD está pronto para engenharia**

```text
Considere o PRD abaixo:
"""
[PRD]
"""

Avalie, em uma escala de 0 a 4, o quão pronto ele está para ser implementado por engenharia, considerando:
- Clareza de problema e objetivos
- Definição de escopo / não escopo
- Detalhamento de requisitos funcionais
- Presença de requisitos não funcionais críticos
- Critérios de aceitação
- Riscos e dependências

Justifique a nota e liste o que falta para chegar a 4.
```

8. **Prompt para checar legibilidade por IA**

```text
Você é um(a) engenheiro(a) de IA avaliando se um PRD é adequado para ser usado como contexto por LLMs.

Analise o PRD abaixo:
"""
[PRD]
"""

Responda:
1) A estrutura (headings, listas, numeração) é suficiente para que um LLM extraia requisitos, objetivos e critérios de aceitação com baixa ambiguidade?
2) Quais seções deveriam ser reestruturadas (por exemplo, separar melhor contexto de instruções, numerar requisitos, agrupar critérios de aceitação)?
3) Sugira uma versão "machine‑friendly" do sumário de requisitos (ex.: tabela ou JSON) que poderia ser anexada ao PRD.
```

## Limitações da pesquisa

- **Ausência de guias oficiais de PRD** de Google, OpenAI e Anthropic – recomendações para PRDs foram inferidas a partir de documentos de design, prompt engineering e requirements/SRS, não de materiais específicos de PRD.[^10][^15][^11]
- **Variabilidade entre empresas** – a sigla PRD é usada de formas diferentes; algumas organizações preferem specs técnicas e RFCs, outras product briefs, outras Shape Up. Este relatório foca no conceito funcional de “documento de requisitos de produto”, não em terminologia estrita.
- **Escopo de literatura acadêmica** – priorizou‑se normas, revisões sistemáticas e estudos empíricos amplos; pode haver trabalhos relevantes mais recentes ou específicos de domínios que não foram capturados.
- **Evolução rápida de práticas de IA** – recomendações de OpenAI e Anthropic sobre prompting e long context estão em rápida evolução; este documento reflete o estado público disponível até meados de 2026.[^14][^10][^11]

## Fontes essenciais

- IEEE Computer Society – guia sobre Software Requirements Specifications e propriedades de requisitos de qualidade.[^1]
- Walia & Carver (2009) – systematic literature review de erros de requisitos e taxonomia de erros.[^3]
- Estudos sobre “just‑enough documentation for agile effort estimation”, destacando importância de requisitos, critérios de aceite, user stories e wireframes mesmo em Agile.[^5]
- Atlassian – template oficial de Product Requirements e Lenny’s Product Requirements Template.[^7][^6]
- Reforge – materiais sobre PRDs como documentos vivos, ajustados à fase e risco.[^8][^15]
- “Design Docs at Google” – visão da cultura de design docs e sua anatomia (contexto, goals, non‑goals, alternativas, cross‑cutting concerns).[^15]
- OpenAI – guias de prompt engineering, structured outputs, function calling e evals.[^10][^14][^12]
- Anthropic – guias de prompt engineering, best practices (clareza, contexto, exemplos, XML tags) e long context prompting.[^12][^11]

---

## References

1. [Free Product Requirements Document (PRD) Template | Confluence](https://www.atlassian.com/software/confluence/templates/product-requirements) - Streamline your build with this product requirements template. Create clear PRDs, track Jira issues,...

2. [Prompt engineering for Claude's long context window - Anthropic](https://www.anthropic.com/news/prompting-long-context) - Anthropic is an AI safety and research company that's working to build reliable, interpretable, and ...

3. [Free Product management templates collection | Confluence](https://www.atlassian.com/software/confluence/templates/collections/product-managers) - Confluence templates help teams create, organize, and discuss work. Get started with our template li...

4. [Free Software development templates collection | Confluence](https://www.atlassian.com/software/confluence/templates/collections/software-development) - Confluence templates help teams create, organize, and discuss work. Get started with our template li...

5. [제품 요구 사항 문서(PRD) 템플릿 템플릿 | Confluence](https://www.atlassian.com/ko/software/confluence/templates/product-requirements) - 이 제품 요구 사항 템플릿으로 빌드를 간소화하세요. 명확한 제품 요구 사항 문서를 만들고 Jira 이슈를 추적하며 하나의 정보 출처에 팀을 정렬하세요.

6. [Anthropic Context Engineering Guide 2026: A Field Manual - HiveTrail](https://hivetrail.com/blog/anthropic-context-engineering-guide) - HiveTrail develops solutions for Model Context Protocol (MCP) security, identity and access control.

7. [Free Product Management templates | Confluence - Atlassian](https://www.atlassian.com/software/confluence/templates/categories/product-management) - Confluence templates help teams create, organize, and discuss work. Get started with our template li...

8. [Using Anthropic: Best Practices, Parameters, and Large Context ...](https://www.prompthub.us/blog/using-anthropic-best-practices-parameters-and-large-context-windows) - Best practices and tips when prompting with Claude 2. Learn how to best leverage large context windo...

9. [Prompt-engineering-OpenAI-API.pdf - KI-Insights](https://www.ki-insights.com/wp-content/uploads/2024/02/Prompt-engineering-OpenAI-API.pdf)

10. [What is a Product Requirements Document (PRD)? - Agile - Atlassian](https://www.atlassian.com/agile/product-management/requirements) - A product requirements document (PRD) defines the purpose, features, and behavior of a product, alig...

11. [prompt-blueprint/guides/openai-best-practices__chatgpt-4_5.md at main · thibaultyou/prompt-blueprint](https://github.com/thibaultyou/prompt-blueprint/blob/main/guides/openai-best-practices__chatgpt-4_5.md) - Enterprise-grade prompt engineering toolkit: Distilled best practices, production-ready meta-prompts...

12. [Long context prompting tips - Simon Willison's Weblog](https://simonwillison.net/2024/Aug/26/long-context-prompting-tips/) - Interesting tips here from Anthropic's documentation about how to best prompt Claude to work with lo...

13. [Optimize Prompts - OpenAI Developers](https://developers.openai.com/cookbook/examples/optimize_prompts) - Crafting effective prompts is a critical skill when working with AI models. Even experienced users c...

14. [Long Context Prompting Tips — Anthropic - aiwithgrant](https://www.aiwithgrant.com/guides/anthropic-long-context-tips) - Techniques for leveraging Claude's 200K token context window with large documents and data.

15. [Anthropic Prompt Engineering Guide: Techniques For Claude](https://aiflowchat.com/blog/articles/anthropic-prompt-engineering-guide) - Anthropic's Claude models are some of the most capable AI systems available right now. But capabilit...

16. [[PDF] A Closer Look on the Difficulties to Determine the Quality of Software ...](https://ceur-ws.org/Vol-2308/ase2019paper04.pdf) - We present an overview of existing research approaches based on the standard ISO/IEC/IEEE 29148:2011...

