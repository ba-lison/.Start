# Documentação em Desenvolvimento de Software: Guia Profissional e Acadêmico

***

## 1. Definição Conceitual

**Documentação de software** é o conjunto de artefatos escritos, visuais e estruturados que descrevem, explicam e registram um sistema de software — seu propósito, funcionamento, decisões de projeto, instruções de uso e contexto operacional — de forma que pessoas distintas possam compreender, usar, manter e evoluir esse sistema ao longo do tempo.[^1][^2]

A norma **ISO/IEC/IEEE 26511:2011** define a documentação de usuário de software como parte integrante do ciclo de vida do software e estabelece procedimentos para seu gerenciamento. A norma **IEEE 1063-2001** fornece requisitos mínimos para estrutura, conteúdo e formato de documentação de usuário. Ian Sommerville, em *Software Engineering* (9ª ed.), afirma que o documento de requisitos é "a declaração oficial do que é exigido dos desenvolvedores do sistema".[^3][^4][^5][^6][^7][^8]

### Diferenciação entre tipos

É comum confundir documentação com outros artefatos. A tabela abaixo apresenta as distinções fundamentais:

| Tipo | O que é | Diferença-chave |
|---|---|---|
| **Comentário de código** | Texto embutido no código-fonte para explicar uma linha ou bloco | É interno ao código; explica *como* ou *por que* algo foi feito localmente. Não substitui documentação[^9] |
| **README** | Arquivo de entrada de um repositório ou projeto | Porta de entrada para o projeto; responde "o que é, como instalar, como usar"[^10][^11] |
| **Especificação técnica** | Documento que descreve a solução técnica proposta, antes ou durante a implementação | Orientada a desenvolvedores; detalha design interno, não apenas comportamento externo |
| **Documentação de usuário** | Manuais, tutoriais e guias voltados ao usuário final | Público não-técnico ou semi-técnico; foca em *como usar o produto*[^12][^4] |
| **Documentação de arquitetura** | Descrição das decisões estruturais do sistema | Inclui diagramas, padrões, visões do sistema; usa modelos como C4[^13][^14] |
| **Documentação de API** | Descreve contratos, endpoints, parâmetros, respostas e erros de uma API | Voltada a consumidores e integradores da API; usa padrões como OpenAPI[^15][^16] |
| **ADR (Architecture Decision Record)** | Registro de uma decisão arquitetural significativa | Captura contexto, decisão, alternativas consideradas e consequências[^17][^18] |
| **Documentação de requisitos** | Descreve o que o sistema deve fazer (funcional e não-funcional) | É o "contrato" com stakeholders; não é design[^6][^19] |
| **Documentação operacional / Runbook** | Procedimentos para operar, monitorar, escalar e responder a incidentes | Voltada a equipes de operação e SRE; foco em ações sob pressão[^20][^21] |
| **Documentação de manutenção** | Informações para quem vai modificar o sistema | Inclui estrutura interna, padrões adotados, pontos de extensão |

***

## 2. Por Que Documentar

### Benefícios centrais

**Comunicação entre desenvolvedores.** Fred Brooks, em *The Mythical Man-Month* (1975), demonstrou que o crescimento exponencial de canais de comunicação à medida que a equipe cresce é um dos maiores riscos em projetos de software. A documentação reduz essa carga ao tornar o conhecimento persistente e assíncrono.[^22][^23]

**Onboarding.** Documentação de qualidade é o mecanismo primário pelo qual novos integrantes aprendem a trabalhar em um projeto sem depender exclusivamente de pares disponíveis. A ausência de onboarding documentado é uma das formas mais comuns de dívida técnica relacionada ao conhecimento.[^24][^25]

**Manutenção.** Pesquisa publicada no *Journal of Brazilian Computer Society* (JBCS) aponta que desenvolvedores que realizam manutenção dependem criticamente de documentação de requisitos, arquitetura e código para entender sistemas que não desenvolveram. Estudos da área de engenharia de software estimam que mais de 60% do ciclo de vida de um sistema é gasto em manutenção — e a ausência de documentação multiplica esse custo.[^26]

**Redução de dependência de pessoas.** O chamado "conhecimento tribal" — armazenado na memória de indivíduos — é um risco organizacional sério. Quando uma pessoa sai, o conhecimento vai com ela. Documentação é a estratégia fundamental para mitigar esse risco.[^27][^28]

**Rastreabilidade de decisões.** ADRs e registros de decisões técnicas permitem que equipes entendam *por que* certas escolhas foram feitas, evitando que decisões passadas sejam revertidas por desconhecimento do contexto original.[^18][^17]

**Governança, auditoria e segurança.** Em contextos regulados — sistemas médicos, financeiros, governamentais — documentação é parte da cadeia de evidências para auditorias. A FDA, por exemplo, exige um *Design History File* para dispositivos médicos. Normas como ISO 13485 e FDA 21 CFR Part 820 prescrevem controle formal de documentação.[^29][^30][^31]

**Qualidade técnica.** A documentação força clareza de pensamento. Escrever sobre um sistema frequentemente revela inconsistências e lacunas antes da implementação. Tom Preston-Werner, co-fundador do GitHub, argumenta que "até você ter escrito sobre seu software, você não sabe o que vai codificar".[^10]

**Continuidade e transferência de projeto.** Projetos mudam de mãos — entre times, entre empresas, entre contratantes. Documentação é a única forma confiável de garantir que o projeto continue funcional após essa transição.

**Relação com dívida técnica.** A ausência de documentação é reconhecida como um tipo específico de dívida técnica — "documentation technical debt" — caracterizada por documentação inexistente, inconsistente, desatualizada ou incompleta. Como toda dívida técnica, ela acumula juros: quanto mais tempo passa, mais caro fica corrigi-la.[^32][^27]

### Riscos de não documentar

- Conhecimento concentrado em pessoas-chave cria gargalos e pontos únicos de falha
- Novas integrações ou mudanças quebram funcionalidades porque ninguém entende os efeitos colaterais
- Auditorias falham por ausência de rastreabilidade
- Onboarding lento aumenta o time-to-productivity de novos desenvolvedores
- Decisões ruins são refeitas porque o contexto das boas decisões foi perdido
- Sistemas críticos ficam sem documentação operacional, aumentando o MTTR (Mean Time to Recovery) em incidentes[^20]

***

## 3. Existe uma "Obrigação" de Documentar?

A obrigação de documentar depende fortemente de contexto. Ela pode ser:

### Técnica e profissional
Como boa prática de engenharia de software, documentar é consenso entre as referências mais respeitadas da área. Steve McConnell, em *Code Complete*, trata documentação como parte integrante da construção de software de qualidade. Ian Sommerville inclui documentação em todos os modelos de processo de software que descreve.[^33][^34][^35]

### Contratual e regulatória
Quando há contrato, o escopo e os entregáveis definem o que deve ser documentado. A obrigação pode derivar de:
- Contrato de prestação de serviços com cláusulas de entregáveis
- SLA (Service Level Agreement) que exige documentação operacional
- Edital público ou Termo de Referência (em contratos governamentais)
- Política interna de compliance ou norma de TI corporativa
- Requisito regulatório setorial (ex.: FDA para software médico, Banco Central para fintechs, ANAC para software aeronáutico)

> **Importante:** Aspectos legais e regulatórios variam amplamente por jurisdição, setor e tipo de contrato. Este guia não constitui assessoria jurídica. Consulte o contrato, o edital aplicável e as normas regulatórias do seu setor para determinar obrigações específicas.

### Por contexto de projeto

| Contexto | Nível típico de obrigação |
|---|---|
| Projeto pessoal | Nenhuma — mas documentar ajuda o próprio autor no futuro |
| Startup / equipe pequena | Leve; foque no README e decisões críticas |
| Empresa de médio/grande porte | Alta; inclui onboarding, arquitetura, runbooks, APIs |
| Software crítico (infraestrutura, saúde, finanças) | Muito alta; com rastreabilidade e controle de versão formal |
| Software regulado (ISO 13485, FDA, SOX) | Mandatória por norma; sujeita a auditoria[^29][^31] |
| Open source | Alta; documentação afeta adoção, contribuições e credibilidade[^36] |
| Projeto sob contrato | Definida pelo contrato e pelo escopo acordado |
| Consultoria | Alta; documentação é parte da entrega e do handoff |
| Órgão público | Definida por normas internas, editais e legislação pertinente |

***

## 4. Tipos Principais de Documentação

### README

**Finalidade:** Porta de entrada do projeto. Responde: o que é, para que serve, como instalar, como usar.
**Público:** Qualquer pessoa que chegue ao repositório.
**Quando usar:** Todo projeto deve ter um README; deve ser o primeiro documento criado.[^11][^10]
**Nível ideal:** Suficiente para que alguém rode o projeto localmente em 10 minutos.
**O que incluir:** Nome, descrição em 1-2 frases, badges de status, pré-requisitos, instalação, uso básico, exemplos, licença, como contribuir.
**O que evitar:** Documentação técnica profunda, detalhes de implementação, histórico de decisões.

### Documentação de Requisitos (SRS)

**Finalidade:** Descrever o que o sistema deve fazer — funcional e não-funcionalmente.
**Público:** Stakeholders, clientes, desenvolvedores, QA.
**Quando usar:** Antes do desenvolvimento; evolui durante discovery e refinamento.[^6][^19]
**Nível ideal:** O suficiente para implementar sem ambiguidade; evitar especificar como implementar.
**O que incluir:** Requisitos funcionais, não-funcionais, restrições, glossário, referências.
**O que evitar:** Detalhes de design de código; soluções técnicas no lugar de comportamentos esperados.

O padrão **IEEE 830** (Recommended Practice for Software Requirements Specifications) fornece uma estrutura amplamente adotada para SRS.[^37][^38]

### Especificação Técnica

**Finalidade:** Descrever como uma funcionalidade ou sistema será implementado tecnicamente.
**Público:** Desenvolvedores, arquitetos, revisores técnicos.
**Quando usar:** Para features complexas antes da implementação; evita retrabalho.
**O que incluir:** Problema sendo resolvido, solução proposta, alternativas consideradas, diagrama se necessário, pontos de risco, critérios de aceitação técnica.
**O que evitar:** Implementação linha-a-linha; não é código.

### Documentação de Arquitetura

**Finalidade:** Descrever a estrutura geral do sistema, componentes, conexões e decisões estruturais.
**Público:** Desenvolvedores, arquitetos, novos membros.
**Quando usar:** Ao definir a arquitetura inicial e ao introduzir mudanças significativas.
**O que incluir:** Visão geral, diagramas (C4 recomendado), tecnologias principais, padrões adotados, fronteiras do sistema.[^13][^14]
**O que evitar:** Detalhes de implementação de cada classe; diagramas que ninguém mantém.

**C4 Model (Simon Brown):** Um dos modelos mais adotados para documentar arquitetura. Organiza diagramas em quatro níveis — Context, Container, Component e Code — cada um voltado a um público distinto. É notation-independent e tooling-independent.[^39][^14][^13]

### Architecture Decision Records (ADRs)

**Finalidade:** Registrar decisões arquiteturais significativas com contexto, alternativas e consequências.
**Público:** Equipe atual e futura de desenvolvimento.
**Quando usar:** Sempre que uma decisão técnica importante é tomada e sua reversão seria custosa.[^17][^18]
**Formato mínimo (Nygard):**
```
# ADR-NNN: Título
## Status: Proposed | Accepted | Deprecated | Superseded
## Contexto
## Decisão
## Consequências
```
**Onde armazenar:** No próprio repositório, em `docs/adr/`, com numeração sequencial.[^18][^17]
**O que evitar:** ADRs genéricos; registrar apenas decisões que realmente precisam de rastreabilidade.

Martin Fowler descreve ADRs como "um documento curto que captura e explica uma única decisão relevante para um produto ou ecossistema".[^17]

### Diagramas

**Finalidade:** Comunicar estruturas, fluxos e relacionamentos que seriam difíceis de descrever em texto.
**Tipos comuns:** Diagrama de contexto (C4 L1), diagrama de containers (C4 L2), diagrama de sequência, ER (banco de dados), fluxo de autenticação, pipeline de CI/CD.
**Boas práticas:** Incluir legenda; manter diagramas versionados junto ao código; preferir ferramentas que geram diagramas a partir de código (PlantUML, Mermaid, Structurizr).[^14]

### Documentação de Banco de Dados

**Finalidade:** Descrever o modelo de dados, tabelas, campos, relacionamentos, índices e políticas de retenção.
**O que incluir:** Diagrama ER, descrição de cada tabela e campo, índices importantes, convenções de nomenclatura, dados sensíveis e políticas de LGPD/GDPR.

### Documentação de API

**Finalidade:** Descrever os contratos da API para consumidores e integradores.
**Padrão recomendado:** OpenAPI (anteriormente Swagger) para REST APIs.[^15][^16]
**O que incluir:** Endpoints, métodos HTTP, parâmetros (path, query, header, body), exemplos de request/response, códigos de erro, autenticação, rate limits.[^40]
**Boas práticas da OpenAPI:** Design-first; manter spec em controle de versão; integrar ao CI/CD.[^15]
**O que evitar:** Documentação gerada automaticamente sem revisão; exemplos inválidos; endpoints sem descrição.

### Documentação de Instalação e Configuração

**Finalidade:** Permitir que alguém configure e rode o sistema do zero.
**O que incluir:** Pré-requisitos, passos de instalação, variáveis de ambiente (nunca com valores reais de produção), comandos de verificação, troubleshooting de instalação.

### Documentação de Deploy

**Finalidade:** Descrever como o sistema vai de código para produção.
**O que incluir:** Pipeline de CI/CD, ambientes existentes, processo de release, rollback, feature flags, dependências externas, smoke tests pós-deploy.

### Documentação Operacional / Runbook

**Finalidade:** Guiar operadores em tarefas rotineiras e em incidentes.[^21][^20]
**Público:** Time de operações, SRE, plantão.
**O que incluir:** Visão geral do serviço, dependências, procedimentos de start/stop/restart, alertas e seus significados, passos de diagnóstico, escalação, contatos.
**Formato recomendado (Tom Limoncelli):** Service Overview, Build Information, Deploy Instructions, Common Tasks, Pager Playbook, Disaster Recovery Plans, SLA.[^20]

### Documentação de Troubleshooting

**Finalidade:** Registro de problemas conhecidos, sintomas e soluções.
**O que incluir:** Sintoma, causa raiz, solução, como verificar se resolveu, links para postmortems se aplicável.

### Documentação de Segurança

**Finalidade:** Descrever controles de segurança, autenticação, autorização, dados sensíveis, vetores de ataque conhecidos.
**Público:** Equipe de segurança, desenvolvedores, auditores.
**Aviso:** Documentação de segurança deve ser armazenada com acesso controlado. Nunca expor detalhes que facilitem ataques.

### Documentação de Testes

**Finalidade:** Descrever estratégia de testes, cobertura, como rodar os testes, como interpretar resultados.
**O que incluir:** Tipos de testes adotados (unit, integration, e2e), ferramentas, comandos, critérios de aceitação, mocks e fixtures necessários.
**Padrão de referência:** IEEE 829 — Standard for Software and System Test Documentation.[^38]

### Documentação de Código (Code-level)

**Finalidade:** Explicar código que não se explica sozinho — *por que*, não *o quê*.[^9][^41]
**Boas práticas:** Comentar o "porquê" de decisões não óbvias; evitar comentários que apenas repetem o código; usar docstrings/JSDoc para APIs públicas.
**Robert C. Martin (Clean Code, 2008):** Prefira código autoexplicativo; use comentários apenas quando o código realmente não consegue expressar a intenção.[^41][^42]

### Documentação para Usuário Final

**Finalidade:** Guiar usuários não-técnicos no uso do software.
**Tipos (Diátaxis):** Tutoriais (orientados a aprendizado), How-to guides (orientados a tarefas), Reference (referência técnica), Explanation (orientados a compreensão).[^43][^44][^45]
**Padrão de referência:** IEEE/ISO/IEC 26514-2021.[^12]

### Changelog

**Finalidade:** Registro cronológico de mudanças entre versões.
**Formato recomendado:** "Keep a Changelog" (keepachangelog.com); seções: Added, Changed, Deprecated, Removed, Fixed, Security.
**O que evitar:** Mensagens de commit brutas como changelog; entradas sem data ou versão.

### ADR / Documentação de Decisões de Produto

**Finalidade:** Registrar decisões de produto e negócio, não apenas técnicas.
**Público:** Product managers, stakeholders, equipe de desenvolvimento.

### Documentação de Onboarding

**Finalidade:** Guiar novos membros da equipe nos primeiros dias.
**O que incluir:** Acesso a ferramentas e sistemas, setup local, arquitetura do produto, fluxos principais, glossário, pessoas-chave, rituais da equipe.

***

## 5. Quando a Documentação Começa

A documentação deve ser um processo **contínuo e incremental**, não um evento final.[^46][^47]

### Linha do tempo recomendada

| Fase | Documentação prioritária |
|---|---|
| Discovery / Ideação | Requisitos, glossário, personas, escopo e não-escopo |
| Arquitetura | Diagramas C4 nível 1 e 2, ADRs iniciais, decisões tecnológicas |
| Implementação | README, documentação de API (design-first), ADRs de decisões técnicas, comentários críticos |
| Testes | Plano de testes, setup de testes, critérios de aceitação |
| Antes do deploy | Runbook, documentação de deploy, configuração, variáveis de ambiente |
| Após deploy | Documentação operacional, monitoramento, alertas |
| Após incidentes | Postmortem, atualização de troubleshooting e runbook |
| Manutenção | Atualização contínua conforme mudanças |

### Documentação upfront vs. incremental

**Documentação upfront excessiva** (modelo waterfall clássico) exige especificação completa antes do desenvolvimento. Pode ser necessária em contextos regulados ou contratos governamentais, mas tende a criar documentação desatualizada rapidamente, pois o sistema evolui durante o desenvolvimento.[^6]

**Documentação incremental / contínua** é o modelo recomendado na maioria dos contextos modernos. A documentação evolui junto com o código — cada PR pode incluir atualizações de docs, cada sprint pode incluir revisão da documentação existente.[^36][^48]

### Comparação de abordagens

| Abordagem | Filosofia de documentação | Pontos fortes | Limitações |
|---|---|---|---|
| **Waterfall** | Upfront, formal, completa antes do desenvolvimento | Rastreabilidade; necessário em contextos regulados | Tende a ficar desatualizada; caro de manter |
| **Ágil** | Just-in-time, mínima suficiente, iterativa | Alinhada com o código; menos burocracia | Pode resultar em sub-documentação se não houver disciplina |
| **DevOps** | Docs como parte do pipeline; runbooks; postmortems | Integrada ao ciclo de vida operacional | Foca mais em ops do que em produto |
| **Docs-as-Code** | Docs versionadas junto ao código; revisadas em PR | Consistência; rastreabilidade; automação possível[^49][^50] | Requer ferramentas e cultura adequadas |
| **Living Documentation** | Documentação gerada ou extraída do próprio sistema[^51][^46][^47] | Sempre atualizada; baixo overhead manual | Requer arquitetura e disciplina para funcionar |

O **Manifesto Ágil** (2001) declara "working software over comprehensive documentation" — o que significa que documentação importa, mas não deve ser produzida a custo do software funcionando. Não é uma permissão para não documentar.[^52][^53]

***

## 6. Boas Práticas Modernas

### Documentar para o público certo
Antes de escrever, pergunte: *quem vai ler isso?* Um runbook é para o engenheiro de plantão sob pressão. Um tutorial é para o usuário iniciante. Uma especificação técnica é para o desenvolvedor que vai implementar.[^36][^43]

### Docs-as-Code
Tratar documentação como código: escrever em Markdown ou reStructuredText, armazenar em Git, revisar em pull requests, publicar via CI/CD. Isso garante que a documentação passe pelo mesmo rigor de revisão que o código.[^54][^50][^49]

### Manter documentação próxima do código
Quando possível, docs devem viver no mesmo repositório que o código — especialmente ADRs, documentação de API e README. Isso garante que mudanças no código disparem revisões na documentação.[^55][^54]

### Incluir documentação no Definition of Done (DoD)
A documentação deve ser critério de conclusão de uma user story ou feature. Exemplos de critérios em um DoD:[^56][^48][^57]
- README atualizado se comportamento externo mudou
- ADR criado se decisão arquitetural foi tomada
- Docs de API atualizadas se endpoint foi adicionado/modificado
- Runbook atualizado se novo alerta foi criado

### Revisar documentação em pull requests
Assim como o código, documentação deve ser revisada por pares. O Google Engineering Practices recomenda verificar se o desenvolvedor atualizou documentação relevante como parte do code review.[^58][^9]

### Registrar decisões, não apenas estado final
Documentação que só descreve *o que está* não explica *por que está assim*. ADRs e comentários contextuais respondem ao "por quê" — o que é essencial para manutenção e evolução do sistema.[^18][^17]

### Framework Diátaxis (Daniele Procida)
Organiza documentação técnica em quatro quadrantes com propósitos distintos:[^44][^45][^43]
- **Tutorials:** Orientados a aprendizado; conduzem o leitor por uma experiência
- **How-to guides:** Orientados a tarefas; resolvem um problema específico
- **Reference:** Orientados a informação; descrevem de forma precisa e completa
- **Explanation:** Orientados a compreensão; aprofundam o entendimento conceitual

Adotado pelo Canonical (Ubuntu), Python, Django e outros grandes projetos open source.[^44]

### C4 Model para arquitetura
Usar o modelo C4 de Simon Brown para diagramas de arquitetura: Context → Container → Component → Code. Níveis superiores são mais estáveis; nível de código raramente precisa ser documentado explicitamente.[^13][^39][^14]

### Living Documentation (Cyrille Martraire)
Filosofia de extrair e gerar documentação a partir de anotações no código, testes automatizados e artefatos do próprio sistema. Reduz o esforço manual de manutenção de docs ao tornar o código a fonte da verdade.[^51][^47][^46]

### README-Driven Development (Tom Preston-Werner)
Escrever o README *antes* do código força clareza sobre o que está sendo construído. É especialmente útil para bibliotecas e ferramentas.[^59][^10]

### Automatizar geração quando adequado
- OpenAPI specs geradas a partir de anotações no código
- Changelogs gerados a partir de commits convencionais (Conventional Commits)
- Diagramas gerados a partir de código com Mermaid, PlantUML ou Structurizr
- Documentação de código gerada com JSDoc, Sphinx, Javadoc

### Evitar duplicação (DRY aplicado a docs)
Documentação duplicada tende a divergir. Prefira uma única fonte da verdade e links para ela.[^15]

### Preferir exemplos práticos
Exemplos concretos comunicam mais rapidamente que descrições abstratas. Toda documentação de API deve ter exemplos de request/response. Todo tutorial deve produzir algo tangível.[^36]

***

## 7. O Que Deve Entrar na Documentação

Uma documentação completa de projeto/software tipicamente inclui:

**Sobre o produto:**
- Objetivo do sistema e problema que resolve
- Visão geral em uma frase (elevator pitch)
- Público-alvo do sistema
- Escopo e explicitamente o que está fora do escopo (non-goals)
- Premissas e restrições conhecidas
- Roadmap (se aplicável e aprovado)
- Limitações conhecidas

**Sobre os requisitos:**
- Requisitos funcionais principais
- Requisitos não-funcionais (performance, segurança, escalabilidade)
- Regras de negócio validadas com stakeholders

**Sobre a arquitetura:**
- Diagrama de visão geral (C4 nível 1 ou 2)
- Tecnologias utilizadas e motivo da escolha
- Decisões técnicas importantes (ADRs)
- Padrões e convenções adotados
- Integrações com sistemas externos
- Estratégia de autenticação/autorização

**Para desenvolvedores:**
- Como rodar localmente (passo a passo)
- Como configurar o ambiente de desenvolvimento
- Variáveis de ambiente necessárias (sem valores sensíveis)
- Dependências e como instalá-las
- Comandos principais (build, test, lint, run)
- Estrutura de pastas e responsabilidade de cada módulo
- Fluxo principal do sistema
- Modelo de dados / documentação de banco de dados
- Documentação de APIs (internas e externas)
- Como rodar os testes

**Para operação:**
- Processo de deploy
- Ambientes (dev, staging, prod)
- Monitoramento e alertas
- Logs: onde encontrar, como interpretar
- Runbook para operações comuns
- Troubleshooting de problemas conhecidos
- Plano de contingência / rollback

**Sobre segurança:**
- Dados sensíveis manipulados pelo sistema
- Controles de segurança implementados
- Política de acesso

**Sobre gestão:**
- Changelog versionado
- Responsáveis ou owners de cada componente
- Links úteis (repositório, staging, monitoramento, ticketing)
- Documentação de onboarding para novos membros

***

## 8. O Que Não Deve Entrar

### Informação óbvia
Comentar `i = i + 1; // incrementa i em 1` é ruído, não documentação. Documente o que não é óbvio pelo contexto.[^42][^41]

### Duplicação desnecessária do código
Documentação que apenas repete o que o código já expressa claramente não agrega valor e se tornará inconsistente rapidamente.[^60]

### Segredos, senhas, tokens e chaves
Jamais incluir credentials, tokens de API, chaves criptográficas ou senhas em documentação, mesmo em repositórios privados. Use referências a variáveis de ambiente ou gestores de segredos (ex.: `.env.example` com valores de placeholder).[^54]

### Dados pessoais sensíveis
Documentação não é o lugar para dados de usuários, logs com PII, ou informações protegidas por LGPD/GDPR.

### Informação desatualizada
Documentação errada é pior que nenhuma documentação — ela induz ao erro. Prefira remover documentação obsoleta a mantê-la incorreta.[^36]

### Opiniões sem decisão registrada
"Talvez devêssemos usar PostgreSQL em vez de MySQL" não pertence à documentação sem uma decisão tomada. Use ADRs para registrar decisões; não deixe incertezas sem resolução na documentação.

### Excesso de burocracia
Documentação deve agregar valor, não demonstrar esforço. Se um documento não é lido por ninguém, é overhead. A Atlassian recomenda identificar quem vai usar cada documento antes de escrevê-lo.[^36]

### Conteúdo sem dono
Documentação sem responsável designado tende a ficar desatualizada. Todo documento precisa de um owner que o mantenha.[^36]

### Diagramas impossíveis de manter
Diagramas extremamente detalhados de código ou de sequências complexas ficam obsoletos na primeira mudança. Prefira diagramas de alto nível que mudam menos frequentemente.[^14]

### Regras de negócio inventadas ou não validadas
Documentar regras de negócio que não foram validadas com stakeholders introduz comportamento incorreto no sistema.

***

## 9. Como Fazer na Prática: Guia Passo a Passo

### Passo 1: Comece pelo README
Antes de qualquer código ou imediatamente quando iniciar o projeto, crie um `README.md` com: nome, descrição de uma frase, status (em desenvolvimento), como rodar localmente (mesmo que ainda não funcione). Isso força clareza sobre o que está sendo construído.[^10]

### Passo 2: Documente requisitos e decisões iniciais
Antes de desenvolver funcionalidades complexas, escreva uma especificação técnica simples ou pelo menos bullets de decisões que precisam ser tomadas. Se escolher uma tecnologia, banco de dados ou padrão de arquitetura, registre em um ADR.[^17]

### Passo 3: Escolha o nível de detalhe adequado
Uma boa heurística: documente o suficiente para que um desenvolvedor de nível júnior-médio consiga entender o propósito e contribuir sem pedir ajuda a você. Se seu projeto tem zero contribuidores externos, um README robusto e ADRs podem ser suficientes. Se é um produto de equipe, adicione arquitetura, APIs e runbooks.

### Passo 4: Organize a documentação
Estrutura recomendada para repositório:
```
/docs
  /adr          # Architecture Decision Records
  /api          # Documentação de API (ou link para spec OpenAPI)
  /architecture # Diagramas e decisões de arquitetura
  /runbooks     # Procedimentos operacionais
  /guides       # Como fazer coisas específicas
README.md
CHANGELOG.md
CONTRIBUTING.md
```

### Passo 5: Defina responsáveis
Cada documento crítico deve ter um owner. Em times ágeis, o squad dono do componente é responsável pela documentação do mesmo. Crie um `CODEOWNERS` no repositório para mapear responsabilidades.[^36]

### Passo 6: Integre ao fluxo de desenvolvimento
- Inclua "documentação atualizada" no Definition of Done do time[^48][^56]
- Adicione um checklist de PR que inclua verificação de docs
- Revise documentação em code review[^58]
- Reserve tempo em sprints para atualizar documentação existente

### Passo 7: Revise periodicamente
Faça uma revisão trimestral de documentação crítica (README, arquitetura, runbooks). Identifique o que ficou desatualizado, o que pode ser removido e o que está faltando.[^36]

### Passo 8: Evite que vire burocracia
- Não exija templates para tudo; priorize documentação que alguém vai ler
- Comece pequeno e itere
- Prefira documentação boa e curta a documentação longa e ignorada
- Trate documentação como produto: ela tem usuários e deve agregar valor a eles

***

## 10. Checklist Prático

### A. Antes de Desenvolver
- [ ] README criado com descrição do projeto e objetivo
- [ ] Requisitos principais documentados (mesmo que em bullets)
- [ ] Decisões arquiteturais relevantes registradas em ADR
- [ ] Variáveis de ambiente documentadas (`.env.example`)
- [ ] Glossário de termos de domínio iniciado

### B. Durante o Desenvolvimento
- [ ] Comentários de código explicam o "porquê" de decisões não óbvias
- [ ] ADRs criados para cada decisão técnica significativa
- [ ] Documentação de API atualizada conforme endpoints são criados/modificados
- [ ] Estrutura de pastas documentada se não for autoexplicativa
- [ ] Testes têm nomes descritivos que funcionam como documentação viva

### C. Antes de Abrir Pull Request
- [ ] README atualizado se comportamento externo mudou
- [ ] Documentação de API atualizada (OpenAPI spec, se aplicável)
- [ ] ADR criado se decisão arquitetural foi tomada
- [ ] Comentários de código desnecessários removidos
- [ ] Changelog atualizado se mudança afeta usuários finais

### D. Antes do Deploy
- [ ] Guia de deploy atualizado
- [ ] Runbook de operações atualizado com novos cenários
- [ ] Alertas de monitoramento documentados
- [ ] Variáveis de ambiente de produção documentadas (sem valores)
- [ ] Rollback testado e documentado

### E. Após Deploy
- [ ] Postmortem de qualquer incidente ocorrido durante deploy
- [ ] Changelog publicado (se release pública)
- [ ] Documentação de usuário atualizada (se funcionalidade visível ao usuário)
- [ ] Runbook atualizado com comportamentos observados em produção

### F. Após Incidente ou Bug Crítico
- [ ] Postmortem escrito com: timeline, causa raiz, impacto, ações corretivas
- [ ] Runbook atualizado com procedimento para esse tipo de incidente
- [ ] Troubleshooting guide atualizado com sintoma e solução
- [ ] ADR criado se mudança arquitetural for decidida como resultado

### G. Durante Manutenção
- [ ] Documentação revisada quando código é modificado
- [ ] ADRs criados para mudanças de direção técnica
- [ ] Documentação obsoleta removida ou marcada como deprecated
- [ ] Dívida técnica de documentação registrada no backlog

### H. Ao Entregar o Projeto para Outra Pessoa / Equipe / Cliente
- [ ] README completo e atualizado
- [ ] Guia de setup local testado por alguém que não conhece o projeto
- [ ] Arquitetura documentada com diagramas
- [ ] Todos os ADRs atualizados e revisados
- [ ] Runbooks completos para todas as operações rotineiras
- [ ] Documentação de API completa e testada
- [ ] Credenciais transferidas via canal seguro (nunca em docs)
- [ ] Guia de onboarding para o time receptor
- [ ] Lista de dívida técnica conhecida entregue formalmente
- [ ] Contatos e escalação documentados

***

## 11. Templates

### Template: README.md

```markdown
# Nome do Projeto

> Uma frase descrevendo o que o projeto faz e para quem.

[![Status](badge-url)](link)  [![License](badge-url)](link)

## O que é

Descrição em 2-3 parágrafos: qual problema resolve, quem usa, contexto de uso.

## Pré-requisitos

- Node.js >= 18
- Docker >= 24
- Conta no Serviço X

## Instalação

```bash
git clone https://github.com/org/projeto
cd projeto
cp .env.example .env
npm install
```

## Como rodar localmente

```bash
npm run dev
# Acesse: http://localhost:3000
```

## Como rodar os testes

```bash
npm test
npm run test:e2e
```

## Estrutura do projeto

```
src/
  api/        # Endpoints e controllers
  services/   # Lógica de negócio
  models/     # Modelos de dados
docs/
  adr/        # Architecture Decision Records
```

## Variáveis de ambiente

Veja `.env.example` para lista completa. Variáveis obrigatórias:

| Variável | Descrição |
|---|---|
| DATABASE_URL | String de conexão com o banco |
| API_KEY | Chave da API do Serviço X |

## Links úteis

- [Documentação de API](docs/api.md)
- [Arquitetura](docs/architecture.md)
- [Board de tarefas](link)
- [Ambiente de staging](link)

## Contribuindo

Veja [CONTRIBUTING.md](CONTRIBUTING.md).

## Licença

[MIT](LICENSE)
```

***

### Template: ADR

```markdown
# ADR-NNN: Título descritivo da decisão

**Data:** YYYY-MM-DD
**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXX
**Decisores:** Nome1, Nome2

## Contexto

Descreva o problema ou a força que levou a esta decisão. Use linguagem neutra e factual. Mencione restrições técnicas, de negócio ou organizacionais relevantes.

## Decisão

"Nós decidimos [ação concreta]."

Explique a decisão em voz ativa. Por que esta opção foi escolhida em relação às alternativas?

## Alternativas consideradas

- **Opção A:** Prós: [...] Contras: [...]
- **Opção B:** Prós: [...] Contras: [...]

## Consequências

**Positivas:**
- [Benefício 1]

**Negativas / Trade-offs:**
- [Custo ou limitação 1]

**Riscos:**
- [Risco que deve ser monitorado]
```

***

### Template: Documentação de API (OpenAPI excerpt)

```yaml
openapi: 3.1.0
info:
  title: Nome da API
  description: Descrição do propósito e contexto da API.
  version: 1.0.0
  contact:
    name: Time de Desenvolvimento
    email: dev@exemplo.com

servers:
  - url: https://api.exemplo.com/v1
    description: Produção
  - url: https://api-staging.exemplo.com/v1
    description: Staging

paths:
  /users/{id}:
    get:
      summary: Busca usuário por ID
      description: Retorna os dados públicos de um usuário específico.
      operationId: getUserById
      tags: [Users]
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
          description: UUID do usuário
      responses:
        '200':
          description: Usuário encontrado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
              example:
                id: "123e4567-e89b-12d3-a456-426614174000"
                name: "Tech Lead"
                email: "techlead@exemplo.com"
        '404':
          description: Usuário não encontrado
        '401':
          description: Não autenticado
      security:
        - bearerAuth: []
```

***

### Template: Documentação de Requisito

```markdown
# REQ-NNN: Título do Requisito

**Tipo:** Funcional | Não-Funcional
**Prioridade:** Must Have | Should Have | Could Have
**Status:** Draft | Approved | Implemented | Deprecated
**Autor:** Nome | **Data:** YYYY-MM-DD

## Descrição

Como [persona/papel], eu quero [ação/funcionalidade], para que [benefício/valor].

## Critérios de Aceitação

- [ ] Dado [contexto], quando [ação], então [resultado esperado]
- [ ] Dado [contexto], quando [ação inválida], então [mensagem de erro X]

## Premissas

- [O usuário está autenticado]
- [O sistema X está disponível]

## Restrições

- [Tempo de resposta < 200ms]
- [Deve funcionar em mobile]

## Fora do Escopo

- [Funcionalidade Y não está incluída neste requisito]

## Referências

- [Link para wireframe]
- [Link para spec técnica]
```

***

### Template: Documentação de Arquitetura

```markdown
# Arquitetura do Sistema: [Nome do Sistema]

**Versão:** 1.0 | **Data:** YYYY-MM-DD | **Autores:** [Nome(s)]

## Visão Geral

Uma frase: o que o sistema faz e seu papel no ecossistema maior.

## Diagrama de Contexto (C4 - Nível 1)

[Diagrama ou link para diagrama]

Atores principais: [Usuário Final, Sistema Externo X]
O sistema recebe/envia: [descrição de fluxos]

## Diagrama de Containers (C4 - Nível 2)

[Diagrama ou link para diagrama]

| Container | Tecnologia | Responsabilidade |
|---|---|---|
| API Backend | Node.js + Express | Lógica de negócio e persistência |
| Frontend | React | Interface do usuário |
| Banco de Dados | PostgreSQL 15 | Persistência principal |

## Decisões Arquiteturais

Ver `docs/adr/` para registro completo de ADRs.

Resumo das principais:
- [ADR-001: Uso de PostgreSQL como banco principal](docs/adr/001-postgresql.md)

## Tecnologias Principais

| Tecnologia | Versão | Motivo da escolha |
|---|---|---|
| Node.js | 20 LTS | Ecossistema, familiaridade do time |
| PostgreSQL | 15 | Consistência, suporte a JSON, maturidade |

## Padrões e Convenções

- REST com versionamento de URL (/v1/)
- Conventional Commits para mensagens de commit
- OpenAPI para documentação de API

## Integrações

| Sistema | Tipo | Protocolo | Finalidade |
|---|---|---|---|
| SendGrid | Externo | REST | Envio de e-mails |
| S3 | Externo | SDK AWS | Armazenamento de arquivos |

## Segurança

- Autenticação: JWT com RS256
- Autorização: RBAC (Role-Based Access Control)
- Dados em trânsito: TLS 1.3

## Pontos de Melhoria Conhecidos

- [Item 1: descrição e motivo de não ter sido resolvido ainda]
```

***

### Template: Runbook Operacional

```markdown
# Runbook: [Nome do Serviço / Procedimento]

**Serviço:** [Nome] | **Owner:** [Time/Pessoa] | **Versão:** 1.0
**Contato de plantão:** [e-mail/Slack]

## Visão Geral do Serviço

O que o serviço faz, seu impacto para o negócio e suas dependências críticas.

## Arquitetura resumida

[Diagrama ou descrição em texto]

**Dependências:**
- PostgreSQL (crítico)
- Redis (cache, degradável)
- API de pagamento externa

## Procedimentos Comuns

### Reiniciar o serviço

```bash
kubectl rollout restart deployment/nome-do-servico
kubectl rollout status deployment/nome-do-servico
```

**Verificação:** `curl https://api.exemplo.com/health` deve retornar `{"status":"ok"}`

### Escalar horizontalmente

```bash
kubectl scale deployment/nome-do-servico --replicas=5
```

## Alertas e Respostas

| Alerta | Possível Causa | Ação |
|---|---|---|
| High CPU > 80% por 5min | Spike de tráfego ou query lenta | Verificar APM; escalar se necessário |
| Erros 5xx > 1% | Bug em deploy recente | Verificar logs; considerar rollback |
| Latência P99 > 2s | Query lenta ou dependência degradada | Checar banco e integrações |

## Rollback

```bash
kubectl rollout undo deployment/nome-do-servico
# Verificar versão anterior:
kubectl rollout history deployment/nome-do-servico
```

## Disaster Recovery

[Procedimento detalhado]

## SLA

- Disponibilidade alvo: 99.9%
- RTO (Recovery Time Objective): 30 minutos
- RPO (Recovery Point Objective): 1 hora
```

***

### Template: Guia de Setup Local

```markdown
# Setup Local: [Nome do Projeto]

**Tempo estimado:** ~15 minutos
**Testado em:** macOS 14, Ubuntu 22.04

## Pré-requisitos

- [ ] Git
- [ ] Docker 24+ e Docker Compose
- [ ] Node.js 20 LTS
- [ ] Conta no [Serviço X] com permissões de dev

## Passo a Passo

### 1. Clone o repositório

```bash
git clone https://github.com/org/projeto.git
cd projeto
```

### 2. Configure variáveis de ambiente

```bash
cp .env.example .env
# Edite .env com suas credenciais de desenvolvimento
# Obtenha as credenciais em: [link interno]
```

### 3. Inicie os serviços de infraestrutura

```bash
docker-compose up -d
# Aguarde os serviços iniciarem (~30s)
docker-compose ps  # Todos devem estar "Up"
```

### 4. Instale dependências e configure o banco

```bash
npm install
npm run db:migrate
npm run db:seed  # Dados de exemplo
```

### 5. Inicie a aplicação

```bash
npm run dev
```

Acesse: http://localhost:3000

Credenciais de teste: `dev@exemplo.com` / `senha123`

## Verificação

- [ ] http://localhost:3000 carrega sem erros
- [ ] http://localhost:3000/api/health retorna `{"status":"ok"}`
- [ ] `npm test` passa sem erros

## Problemas Comuns

**Erro: "Port 5432 already in use"**
Outro PostgreSQL está rodando. Pare-o: `brew services stop postgresql`

**Erro: "Cannot connect to Redis"**
Docker não iniciou. Verifique: `docker-compose logs redis`
```

***

### Template: Changelog

```markdown
# Changelog

Formato baseado em [Keep a Changelog](https://keepachangelog.com/).
Versionamento segue [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [2.1.0] - 2026-06-15

### Added
- Suporte a autenticação via SSO (Google Workspace)
- Endpoint `GET /v1/reports/summary` para relatórios consolidados

### Changed
- Timeout de sessão aumentado de 30 para 60 minutos
- Mensagens de erro da API padronizadas segundo RFC 7807

### Fixed
- Correção de race condition no processamento de uploads simultâneos (#342)
- Validação incorreta de CEPs com dígito verificador 0 (#367)

### Security
- Atualização de dependência `jsonwebtoken` para corrigir CVE-2022-XXXXX

## [2.0.0] - 2026-03-01

### Breaking Changes
- Endpoint `/v1/users` agora requer header `X-API-Version: 2`
- Campo `user.name` dividido em `user.firstName` e `user.lastName`

[Unreleased]: https://github.com/org/projeto/compare/v2.1.0...HEAD
[2.1.0]: https://github.com/org/projeto/compare/v2.0.0...v2.1.0
```

***

### Template: Documentação de Troubleshooting

```markdown
# Troubleshooting: [Nome do Serviço]

## Como usar este guia

1. Identifique o sintoma observado
2. Siga os passos de diagnóstico
3. Aplique a solução indicada
4. Se não resolver, escale conforme indicado

---

## Problema: [Descrição do sintoma]

**Severidade:** Alta | Média | Baixa
**Frequência:** Comum | Esporádico | Raro

### Sintomas

- O usuário vê [mensagem de erro específica]
- Os logs mostram `ERROR: [trecho de log]`
- Alerta `[Nome do alerta]` foi disparado

### Diagnóstico

```bash
# Verificar logs do serviço
kubectl logs -l app=nome-servico --tail=100

# Verificar status do banco
psql $DATABASE_URL -c "SELECT pg_is_in_recovery();"
```

### Causa mais comum

[Descrição da causa raiz típica]

### Solução

```bash
# Passo 1
comando-1

# Passo 2
comando-2
```

### Verificação

`curl https://api.exemplo.com/health` deve retornar `{"status":"ok"}`

### Se não resolver

Escale para: [Nome] via Slack `#canal-de-escalacao`

### Postmortem relacionado

[Link para postmortem se houver]
```

***

### Template: Checklist de Entrega Técnica

```markdown
# Checklist de Entrega Técnica: [Nome do Projeto / Feature]

**Data de entrega:** YYYY-MM-DD
**Responsável:** [Nome]
**Destinatário:** [Time / Pessoa / Cliente]

## Documentação

- [ ] README atualizado e testado por terceiro
- [ ] Documentação de arquitetura com diagramas atuais
- [ ] ADRs de todas as decisões técnicas relevantes
- [ ] Documentação de API (OpenAPI spec ou equivalente)
- [ ] Guia de setup local testado em máquina limpa
- [ ] Runbooks para operações comuns
- [ ] Guia de troubleshooting para problemas conhecidos
- [ ] Changelog atualizado
- [ ] Lista de dívida técnica conhecida

## Código

- [ ] Testes passando (unit, integration, e2e)
- [ ] Cobertura de testes em nível aceitável
- [ ] Sem credenciais hardcoded
- [ ] Variáveis de ambiente documentadas em `.env.example`
- [ ] Linting e formatação passando

## Infraestrutura

- [ ] Deploy testado em ambiente de staging
- [ ] Rollback testado e documentado
- [ ] Monitoramento configurado e alertas definidos
- [ ] Backups configurados (se aplicável)
- [ ] Certificados e segredos transferidos de forma segura

## Acesso e Permissões

- [ ] Destinatário tem acesso ao repositório
- [ ] Destinatário tem acesso aos ambientes (staging, prod)
- [ ] Acessos a serviços externos transferidos
- [ ] Documentação de acesso entregue via canal seguro

## Sessão de Handoff

- [ ] Walkthrough técnico realizado com o time receptor
- [ ] Perguntas do time receptor respondidas
- [ ] Contato de suporte pós-entrega definido e comunicado
- [ ] Período de suporte pós-entrega acordado
```

***

## 12. Exemplos Concretos

### README ruim vs. README bom

**README ruim:**
```markdown
# Projeto

Este é o projeto.

Para rodar: npm start
```
*Problemas: sem contexto, sem pré-requisitos, sem exemplos, sem instruções de configuração.*

**README bom:**
```markdown
# NotifyHub — Sistema de Notificações Multi-canal

> Serviço de disparo de notificações por e-mail, SMS e push para aplicações internas.

## O que faz
Centraliza o envio de notificações de múltiplas aplicações via uma única API REST.
Suporta templates, agendamento e rastreamento de entrega.

## Pré-requisitos
- Node.js 20 LTS
- Docker e Docker Compose
- Conta SendGrid (para e-mail)

## Setup em 5 minutos
[Passos claros...]
```

***

### Comentário de código ruim vs. útil

**Ruim:**
```python
# Incrementa o contador
count = count + 1
```

**Útil:**
```python
# O índice começa em -1 porque a API retorna o cursor da última posição
# processada, não da próxima. Ver ADR-007 para contexto completo.
cursor = last_processed_id - 1
```

***

### ADR simples

```markdown
# ADR-004: Uso de UUIDs como identificadores de recursos

**Status:** Accepted — 2026-01-15

## Contexto
O sistema precisa gerar IDs para recursos criados por múltiplas instâncias sem coordenação central. IDs sequenciais exigiriam um gerador centralizado (ponto único de falha) ou coordenação entre nós.

## Decisão
Usaremos UUIDs v4 (aleatórios) como identificadores primários para todos os recursos da API.

## Alternativas consideradas
- **Auto-increment do banco:** Mais simples, mas cria dependência do banco como gerador e expõe volume de recursos via enumeração.
- **Snowflake IDs:** Garante ordenação temporal, mas adiciona complexidade de implementação sem benefício claro para nosso caso.

## Consequências
- Positivo: IDs podem ser gerados no cliente sem coordenação
- Positivo: Não expõe volume de dados via enumeração sequencial
- Negativo: UUIDs são menos legíveis por humanos em logs
- Negativo: Performance ligeiramente pior em índices B-tree (vs. sequential IDs)
```

***

### Checklist de entrega de feature (simplificado)

```markdown
# Feature: Autenticação SSO com Google Workspace

- [x] Código implementado e revisado
- [x] Testes unitários escritos (cobertura: 87%)
- [x] Testes de integração passando
- [x] Documentação de API atualizada (endpoint `/auth/sso`)
- [x] README atualizado com instruções de configuração do Google OAuth
- [x] ADR-012 criado (Decisão: usar OAuth 2.0 com PKCE)
- [x] Variáveis de ambiente documentadas no `.env.example`
- [x] Feature flag configurada para rollout gradual
- [ ] Runbook de troubleshooting para erros de SSO (em progresso)
```

***

## 13. Critérios de Qualidade

Uma boa documentação pode ser avaliada pelos seguintes critérios:

| Critério | O que verificar |
|---|---|
| **Clareza** | Um leitor do público-alvo entende sem perguntar[^36] |
| **Completude suficiente** | Contém o necessário para o propósito; não mais, não menos |
| **Atualidade** | Reflete o estado atual do sistema[^36] |
| **Rastreabilidade** | Decisões têm contexto e referências; mudanças têm histórico |
| **Utilidade** | Alguém realmente usa este documento? |
| **Facilidade de manutenção** | É possível atualizar sem grande esforço[^49] |
| **Precisão técnica** | Não contém erros factuais ou comandos quebrados |
| **Acessibilidade** | Encontrável, navegável, pesquisável |
| **Ausência de ambiguidade** | Não admite múltiplas interpretações em pontos críticos |
| **Custo-benefício** | O valor gerado justifica o esforço de criar e manter |

**Teste prático de qualidade:** Peça a alguém que não conhece o projeto para executar a tarefa mais crítica (ex.: rodar localmente) usando apenas a documentação. Os pontos de dúvida são os gaps a corrigir.

***

## 14. Erros Comuns

**1. Documentar tarde demais**
"Vou documentar quando terminar." O contexto se perde; a documentação fica incompleta ou incorreta. Documente durante, não depois.[^46]

**2. Documentar demais (sobre-documentação)**
Documentação que ninguém lê é pior que nenhuma — ela gera ruído e custo de manutenção. Priorize pela utilidade.[^32][^36]

**3. Documentar de menos (sub-documentação)**
A lacuna mais comum em times ágeis. A pressão por velocidade sacrifica documentação que será cara de recuperar.[^32]

**4. Deixar documentação separada do fluxo real**
Documentação que não é parte do processo de desenvolvimento (PR review, Definition of Done) tende a ficar obsoleta.[^49][^48]

**5. Não atualizar**
Código muda, documentação fica estática. Documentação errada é perigosa — induz desenvolvedores a erros.[^36]

**6. Escrever para si mesmo**
O autor já sabe tudo — e isso cria viés cego sobre o que é óbvio. Escreva para o leitor que não tem o seu contexto.[^43]

**7. Confundir documentação com comentário**
Comentário explica uma linha; documentação explica um sistema. São complementares, não substitutos.[^9][^60]

**8. Esconder decisões importantes em conversas de Slack/Teams**
Decisões tomadas em mensagens privadas ou canais de chat são perdidas. ADRs existem precisamente para preservar esse contexto.[^17]

**9. Depender só de conhecimento oral**
"Pergunta pro João, ele sabe" é um risco organizacional. João sai de férias, muda de empresa, ou simplesmente esquece.[^27]

**10. Usar diagramas sem manutenção**
Um diagrama desatualizado comunica incorretamente. Se não há processo para mantê-lo, talvez seja melhor não criá-lo — ou usar ferramentas que geram diagramas do código.[^14]

**11. Não definir dono**
Documentação sem responsável não tem quem a atualize. Defina owners para documentação crítica.[^36]

***

## 15. Visão Equilibrada

### "Código autoexplicativo" vs. documentação
Martin Fowler argumenta que "código é a principal fonte de documentação de um sistema de software — a única suficientemente detalhada e precisa". Isso não significa que só código é suficiente: significa que código claro *reduz a necessidade* de documentação redundante. O "porquê" das decisões, o contexto do negócio, o comportamento esperado em produção — nenhum desses cabe no código.[^60]

### Agilidade vs. documentação
O Manifesto Ágil valoriza "software funcionando sobre documentação abrangente" — mas isso é uma questão de *prioridade em conflito*, não uma licença para eliminar documentação. Times ágeis maduros documentam continuamente, de forma incremental e integrada ao fluxo de desenvolvimento.[^53][^52]

### Documentação mínima vs. suficiente
"Documentação mínima" é o que um cargo ou norma exige. "Documentação suficiente" é o que o contexto do projeto realmente precisa. O objetivo é a segunda — que pode ser mais ou menos que a primeira, dependendo do projeto.[^46]

### Documentação técnica vs. de negócio
Ambas são necessárias. Documentação técnica sem contexto de negócio gera sistemas que resolvem o problema errado. Documentação de negócio sem especificação técnica gera implementações inconsistentes.

### Documentação manual vs. automatizada
Automatização (OpenAPI gerado de anotações, changelogs de commits convencionais, diagramas de código) reduz overhead e melhora consistência — mas não elimina a necessidade de documentação manual para decisões, contexto e narrativa.[^51][^15]

**A documentação ideal é aquela que agrega valor real, é mantida com custo razoável e é de fato utilizada por seu público-alvo.** Nem burocracia, nem negligência.

***

## 16. Referências Comentadas

### Livros

**Brooks, F. P. (1975). *The Mythical Man-Month: Essays on Software Engineering*. Addison-Wesley.**
Clássico absoluto da engenharia de software. Aborda comunicação, documentação e gestão de projetos complexos. Confiável por décadas de validação empírica e acadêmica. Limitação: escrito antes da era ágil; alguns aspectos processuais estão datados.[^61][^22]

**Sommerville, I. (2011). *Software Engineering*, 10ª ed. Pearson.**
Principal livro-texto universitário de engenharia de software. Cobertura sistemática de requisitos, documentação e ciclo de vida. Usado em centenas de universidades globalmente. Limitação: abordagem mais formal/tradicional; cobertura de práticas ágeis menos profunda.[^62][^63]

**McConnell, S. (2004). *Code Complete*, 2ª ed. Microsoft Press.**
Guia prático definitivo de construção de software, incluindo documentação de código, comentários e boas práticas. Amplamente adotado na indústria. Limitação: foca em práticas de código; menos cobertura de documentação de produto.[^34][^35]

**Martin, R. C. (2008). *Clean Code: A Handbook of Agile Software Craftsmanship*. Prentice Hall.**
Define princípios de código legível e autoexplicativo. Relevante para entender quando e como comentar código. Amplamente respeitado na comunidade. Limitação: opiniões fortes do autor sobre comentários podem ser interpretadas de forma excessivamente absolutista.[^41][^42]

**Martraire, C. (2019). *Living Documentation: Continuous Knowledge Sharing by Design*. Addison-Wesley.**
Apresenta padrões e práticas para documentação que evolui com o sistema, extraída de anotações, testes e artefatos. Inovador e prático. Limitação: requer maturidade técnica e arquitetural para implementação plena.[^51][^46]

### Artigos e Fontes Acadêmicas

**Scielo JBCS: "Which documentation for software maintenance?" (iSys 2024)**
Pesquisa com desenvolvedores de manutenção sobre quais artefatos de documentação são mais relevantes em contexto real. Metodologia empírica. Relevante para priorizar documentação prática.[^26]

**Journals-sol.sbc.org.br: "A Method to Support Documentation Technical Debt" (2024)**
Apresenta o método DOTED para gerenciar dívida técnica de documentação, seguindo Design Science Research. Publicado em revista revisada por pares.[^32]

**Parnas, D. L. (1972). "On the Criteria To Be Used in Decomposing Systems into Modules". CACM.**
Fundamento da modularização e ocultamento de informação — base para entender por que documentação de interfaces é mais importante que documentação de implementações.[^64][^65]

**IEEE 829 — Standard for Software and System Test Documentation.**
Padrão para documentação de testes. Amplamente referenciado em contextos corporativos e regulados.[^38]

**ISO/IEC/IEEE 26511:2011 — Systems and software engineering: Requirements for managers of user documentation.**
Norma internacional para gestão de documentação de usuário ao longo do ciclo de vida.[^7][^8]

### Fontes Institucionais e Técnicas

**IEEE Standards (ieee.org): IEEE 1063-2001, IEEE 830, IEEE 829.**
Normas técnicas com requisitos mínimos para diferentes tipos de documentação de software. Autoritativas, revisadas por pares técnicos.[^3][^38]

**ISO/IEC 26514-2021 — Systems and software engineering: Design and development of information for users.**
Norma internacional para design e desenvolvimento de documentação de usuário de software. Publicada pela ISO.[^12]

**Google Engineering Practices (google.github.io/eng-practices).**
Guias técnicos públicos do Google sobre code review, incluindo verificação de documentação. Baseados em práticas de uma das maiores organizações de software do mundo.[^9][^58]

**OpenAPI Initiative (learn.openapis.org).**
Especificação e melhores práticas para documentação de APIs REST. Padrão de facto da indústria.[^15]

**Write the Docs (writethedocs.org): Docs as Code.**
Comunidade global de technical writers; guia sobre a filosofia Docs-as-Code.[^49]

### Frameworks e Práticas Modernas

**Diátaxis Framework (diataxis.fr) — Daniele Procida.**
Framework sistemático para organizar documentação técnica em quatro tipos (tutorials, how-to, reference, explanation). Leve, pragmático, tooling-independent. Adotado pelo Canonical, Python, Django.[^66][^43][^44]

**C4 Model (c4model.com) — Simon Brown.**
Modelo hierárquico de 4 níveis para diagramas de arquitetura de software. Notation-independent, developer-friendly. Amplamente adotado globalmente.[^39][^13][^14]

**Architecture Decision Records (ADRs) — Michael Nygard / Martin Fowler.**
Padrão para registro de decisões arquiteturais. Simples, versionável, integrado ao código. Documentado e popularizado por Martin Fowler.[^18][^17]

---

## References

1. [Software Documentation - Literate Programming](http://www.literateprogramming.com/documentation.pdf) - System documentation includes all of the documents describing the system itself from the requirement...

2. [Software Documentation Ian Sommerville](https://www.eecs.ucf.edu/~turgut/COURSES/EEL6883_SEII_Spr07/PaperPresentations/Sommerville-p143.ppt) - Requirements Spec; Architectural Design; Detailed Design; Commented Source Code. Including output su...

3. [1063-2001 - IEEE Standard for Software User Documentation](https://ieeexplore.ieee.org/document/974401) - This standard provides minimum requirements for the structure, information content, and format of us...

4. [IEEE 1063-2001](https://standards.ieee.org/standard/1063-2001.html) - 1063-1987. IEEE Standard for Software User Documentation. Minimum requirements for the structure and...

5. [Software Requirements](ftp://ftp.gmu.edu/drine/cs_se2/CourseMaterials/PDFLectureNotes_InClassMaterials/Chapter6_SoftwareRequirements_Handout-1.pdf) - The requirements document is the official statement of what is required of the system developers. ○....

6. [Requirements (selected from Ian Sommerville slides for “ ...](https://www.slideserve.com/federico/requirements-selected-from-ian-sommerville-slides-for-software-engineering) - Requirements (selected from Ian Sommervilleslides for “Software Engineering”). Topics covered • The ...

7. [ISO/IEC/IEEE 26512:2011 - Systems and software ...](https://www.iso.org/standard/43071.html) - ISO/IEC/IEEE 26512:2011 was developed to assist users of ISO/IEC 15288:2008 or ISO/IEC 12207:2008 to...

8. [26511-2012 - ISO/IEC/IEEE International Standard](https://ieeexplore.ieee.org/document/6170926) - ISO/IEC/IEEE 26511:2012 specifies procedures for managing user documentation throughout the software...

9. [Google's Engineering Practices – What to Look for in a ...](https://www.codingblocks.net/podcast/googles-engineering-practices-what-to-look-for-in-a-code-review/) - We learn what to look for in a code review while reviewing Google's engineering practices documentat...

10. [Readme Driven Development - Tom Preston-Werner](https://tom.preston-werner.com/2010/08/23/readme-driven-development) - Writing a Readme is absolutely essential to writing good software. Until you've written about your s...

11. [READMEs](https://lab.abilian.com/Tech/Documentation/READMEs/) - https://tom.preston-werner.com/2010/08/23/readme-driven-development.html. README-Driven Development:...

12. [IEEE/ISO/IEC 26514-2021](https://standards.ieee.org/ieee/26514/7467/) - This document covers the development process for designers and developers of information for users o...

13. [C4 model: Home](https://c4model.com) - This is the official website for the “C4 model for visualising software architecture”, written by it...

14. [The C4 Model for Software Architecture](https://www.infoq.com/articles/C4-architecture-model/) - The C4 model consists of a hierarchical set of software architecture diagrams. These diagrams are bo...

15. [Best Practices - OpenAPI Documentation](https://learn.openapis.org/best-practices.html) - Use a Design-First Approach · Keep a Single Source of Truth · Add OpenAPI Descriptions to Source Con...

16. [OpenAPI Specification Guide: Structure Implementation & ...](https://www.gravitee.io/blog/openapi-specification-structure-best-practices) - In-Depth Guide to Open API specification: Structure, Implementation, and Best Practices · 1. Paths a...

17. [Architecture Decision Record](https://martinfowler.com/bliki/ArchitectureDecisionRecord.html) - An Architecture Decision Record (ADR) is a short document that captures and explains a single decisi...

18. [Preserving critical software knowledge using Architectural ...](https://codesoapbox.dev/preserving-critical-software-knowledge-using-architectural-decision-records/) - Writing an ADR. An Architecture Decision Record is a short text file, using a lightweight text forma...

19. [Software Requirements: Definition & Specifications](https://studylib.net/doc/5543781/requirements-engineering-processes) - Learn about software requirements, functional/non-functional types, user & system needs. University-...

20. [What is a Runbook?](https://www.pagerduty.com/resources/automation/learn/what-is-a-runbook/) - A runbook is a detailed “how-to” guide for completing a commonly repeated task or procedure within a...

21. [How to Create a DevOps Runbook Template](https://firehydrant.com/blog/runbook-template-devops/) - Use this DevOps runbook template to optimize your development, operations workflows, and incident re...

22. [The Mythical Man-Month](https://en.wikipedia.org/wiki/The_Mythical_Man-Month) - The Mythical Man-Month: Essays on Software Engineering is a book on software engineering and project...

23. [Mythical Man Month](https://martinfowler.com/bliki/MythicalManMonth.html) - The book contains Brooks's law: “Adding manpower to a late software project makes it later.” The iss...

24. [Being the "documentation" person or the "tech debt ...](https://www.reddit.com/r/ExperiencedDevs/comments/1gjov1l/being_the_documentation_person_or_the_tech_debt/) - Make it everyone's problem e.g. last step in the onboarding process is for the new hire to update th...

25. [Knowledge Management in Technical Documentation](https://www.linkedin.com/pulse/knowledge-management-technical-documentation-bkmi-du9cf) - Explore how tech companies turned docs into strategic assets, using tools like Confluence, Notion, a...

26. [Which documentation for software maintenance?](https://www.scielo.br/j/jbcos/a/zsF5pwLk9twFvTBnvQ4DxLB/?lang=en) - In this article, we present the results of a survey of software maintainers to try to establish what...

27. [Managing Technical Debt: Approaches and Perspectives](https://www.scalablepath.com/project-management/technical-debt) - Technical debt has considerable impact on four main areas: software architecture, knowledge manageme...

28. [Technical Debt Management Checklist](https://www.manifest.ly/use-cases/software-development/technical-debt-management-checklist) - Documentation and Knowledge Sharing. Maintaining comprehensive documentation is critical for enablin...

29. [Medical Device Document Control](https://qt9software.com/blog/medical-device-document-control) - The primary medical device document control standards are outlined in FDA 21 CFR parts 820 and 11, a...

30. [Streamlining Medical Device Design Controls - FDA QMSR ...](https://www.cognidox.com/blog/what-is-design-control-iso-13485-fda-qmsr) - Learn how to meet FDA's new QMSR and ISO 13485 design control requirements with practical steps, che...

31. [Quality and Compliance (Medical Devices)](https://www.fda.gov/medical-devices/device-advice-comprehensive-regulatory-assistance/quality-and-compliance-medical-devices) - The FDA works with manufacturers to help them achieve regulatory compliance, and takes enforcement a...

32. [A Method to Support Documentation Technical Debt ...](https://journals-sol.sbc.org.br/index.php/isys/article/view/2904) - We followed Design Science Research principles to create DOTED, a method for managing Documentation ...

33. [Engenharia de Software - Uma Abordagem Profissional](https://www.kufunda.net/publicdocs/Engenharia%20de%20Software%20-%207.ed.%20(Roger%20S.%20Pressman).pdf) - P934e Pressman, Roger S. Engenharia de software [recurso eletrônico] : uma abordagem profissional / ...

34. [Code Complete](https://en.wikipedia.org/wiki/Code_Complete) - Code Complete is a software development book, written by Steve McConnell and published in 1993 by Mi...

35. [Code Complete - Steve McConnell](https://books.google.com.br/books?id=pDsFCAAAQBAJ) - Widely considered one of the best practical guides to programming, Steve McConnell's original CODE C...

36. [9 Software Documentation Best Practices + Real Examples](https://www.atlassian.com/blog/loom/software-documentation-best-practices) - This guide explores nine best practices for transforming software documentation into an invaluable r...

37. [Software Engineering [R18A0511] LECTURE NOTES](https://mrcet.com/downloads/digital_notes/CSE/III%20Year/Software%20Engineering.pdf) - The students will be able: 1. To comprehend the various software process models. 2. To understand th...

38. [IEEE Software Standards | SE Goldmine](https://segoldmine.ppi-int.com/node/44387) - IEEE Std 1002 - IEEE Standard Taxonomy for Software Engineering Standards ... IEEE Std 1063 - IEEE S...

39. [C4 model | Lightweight standard for visualizing software ...](https://c4model.info) - The C4 model is a simple way to bring consistency to how you and your team describe and diagram soft...

40. [14 Best Practices to Write OpenAPI for Better API ...](https://www.apimatic.io/blog/2022/11/14-best-practices-to-write-openapi-for-better-api-consumption) - 1. No Empty Servers List · 2. Titles, Names, And Summaries Should Not Exceed 50 Characters · 3. No I...

41. [What Is Clean Code? A Guide to Principles and Best ...](https://blog.codacy.com/what-is-clean-code) - Clean code is a term used to refer to code that is easy to read, understand, and maintain. It was ma...

42. [Clean Code & SOLID: Uncle Bob's Practical Guide](https://cleancodeguy.com/blog/robert-martin-uncle-bob) - The goal is practical: write clearer code, reduce technical debt, and keep teams productive over tim...

43. [Diátaxis](https://diataxis.fr) - Diátaxis is a way of thinking about and doing documentation. It prescribes approaches to content, ar...

44. [Diátaxis, a new foundation for Canonical documentation](https://ubuntu.com/blog/diataxis-a-new-foundation-for-canonical-documentation) - The Diátaxis framework will be our map and compass, our standards and our guide, towards quality in ...

45. [Diataxis Framework for Technical Documentation with ...](https://www.katara.ai/diataxis-framework-for-technical-documentation) - Developed by Daniele Procida, the Diataxis framework organizes technical documentation into four typ...

46. [Living Documentation: Continuous Knowledge Sharing by ...](https://www.oreilly.com/library/view/living-documentation-continuous/9780134689418/) - Living Documentation shows how to create documentation that evolves throughout your entire design an...

47. [Living Documentation · Domain-Driven Design Courses](https://ddd.academy/living-documentation-cyrille-martraire/) - A Living Documentation changes at the same pace as software design and development, from establishme...

48. [What is the Definition of Done (DoD) in Agile?](https://www.atlassian.com/agile/project-management/definition-of-done) - The definition of done (DoD) is a shared set of criteria that determines when a product increment is...

49. [Docs as Code](https://www.writethedocs.org/guide/docs-as-code/) - Docs as Code¶ · Issue Trackers · Version Control (Git) · Plain Text Markup (Markdown, reStructuredTe...

50. [How to use Docs as Code without Git commands](https://www.linkedin.com/posts/manuelrbsilva_docsascode-activity-7384396628113309696-B5gU) - The docs as code approach treats documentation like source code. Writers use plain text files (often...

51. [Q&A with Cyrille Martraire on the Book Living Documentation](https://www.infoq.com/articles/book-review-living-documentation/) - They share lessons on controlling tombstone accumulation, building continuous audit loops, and gaini...

52. [Working software over comprehensive documentation](https://www.reddit.com/r/agile/comments/r771u9/working_software_over_comprehensive_documentation/) - The Agile Manifesto states for working software over comprehensive documentation; that is, while it ...

53. [Agile Manifesto and Priorities in Software Development - Visie](https://visieus.com/agile-manifesto/) - Individuals and interactions over processes and tools; Working software over comprehensive documenta...

54. [Documentation Version Control: Best Practices 2024](https://daily.dev/blog/documentation-version-control-best-practices-2024/) - Use version control, Manage code and docs together (e.g., Git). Set up CI process, Auto-update docs ...

55. [Documentation and version control](https://stackoverflow.com/questions/1488441/documentation-and-version-control) - Given a project I'm about to start there will be documentation produced.  What is the best practice ...

56. [Definition of done (DoD) in Agile: Meaning and examples](https://www.wrike.com/project-management-guide/faq/what-is-definition-of-done-agile/) - The Agile definition of done is a collection of criteria that must be completed for a project to be ...

57. [Agile Definition of Done](https://www.productboard.com/glossary/agile-definition-of-done/) - The definition of done is an agreement between a product team on the set of conditions that must be ...

58. [Introduction {#intro} | eng-practices](https://google.github.io/eng-practices/review/) - This page is an overview of our code review process. There are two other large documents that are a ...

59. [README driven development](https://dev.to/gregmartinez44/readme-driven-development-373k) - I recently read an article by Tom Preston-Werner about how to solve this problem. Preston-Werner's s...

60. [Code As Documentation](https://martinfowler.com/bliki/CodeAsDocumentation.html) - The rationale for the code being the primary source of documentation is that it is the only one that...

61. [The mythical man-month (anniversary ed.) | Guide books](https://dl.acm.org/doi/abs/10.5555/207583) - With a blend of software engineering facts and thought-provoking opinions, Fred Brooks offers insigh...

62. [Software Engineering 9th Edition by Ian Sommerville](https://engineering.futureuniversity.com/BOOKS%20FOR%20IT/Software-Engineering-9th-Edition-by-Ian-Sommerville.pdf) - I strongly believe that agile methods have their place but so too does 'tradi- tional' plan-driven s...

63. [SOFTWARE ENGINEERING - Ninth Edition - Ian Sommerville](https://www.facom.ufu.br/~william/Disciplinas%202018-2/BSI-GSI030-EngenhariaSoftware/Livro/softwareEngineeringSomerville.pdf) - We must continue to educate software engineers and develop the disci- pline so that we can create mo...

64. [[PDF] The Mythical Good Software - arXiv](https://arxiv.org/pdf/2507.09596.pdf) - In 1971, David L. Parnas authored an influential essay called ”On the Criteria To Be Used in Decompo...

65. [On the criteria to be used in decomposing systems into modules](https://blog.acolyer.org/2016/09/05/on-the-criteria-to-be-used-in-decomposing-systems-into-modules/) - On the criteria to be used in decomposing systems into modules David L Parnas, 1971. Welcome back to...

66. [Colophon](https://diataxis.fr/colophon/) - Diátaxis is the work of Daniele Procida. It has been developed over a number of years, and continues...

