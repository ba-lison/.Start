# Spec-Driven Development (SDD) na engenharia de software contemporânea e no desenvolvimento com IA

## 1. Definição e escopo conceitual

Spec-Driven Development (SDD) é uma abordagem em que especificações estruturadas e versionadas se tornam o artefato central do desenvolvimento, e o código passa a ser um derivado gerado ou verificado a partir dessas especificações. Em vez de partir diretamente para código ou para prompts ad‑hoc para IA, a equipe escreve um “spec” suficientemente detalhado e verificável (contratos, critérios de aceitação, schemas, exemplos), que serve como fonte de verdade para implementação, testes e evolução do sistema.[^1][^2][^3][^4]

Na literatura recente, SDD é descrito como uma inversão de fluxo: primeiro se consolida a intenção e o contrato em uma especificação, depois agentes humanos e/ou de IA implementam, validam e mantêm o código contra esse contrato. Essa especificação pode ser natural language estruturada (Markdown com seções rígidas), artefatos formais (modelos, contratos, schemas) ou uma combinação, desde que seja suficientemente precisa para orientar geração e verificação de código.[^4][^5]

### Diferenciação em relação a abordagens relacionadas

| Abordagem | Foco principal | Papel da especificação | Relação com SDD |
| --- | --- | --- | --- |
| Requirements Engineering / SRS | Elicitar, documentar e gerir requisitos ao longo do ciclo de vida, tipicamente em um Software Requirements Specification (SRS). | Documento de requisitos, muitas vezes em linguagem natural estruturada (ex.: IEEE 830, ISO/IEC/IEEE 29148).[^6][^7] | SDD assume requisitos claros, mas enfatiza specs mais próximas do design e do código, com estrutura pensada para consumo por humanos e IA. Pode reutilizar normas como 29148 para garantir qualidade de requisitos. |
| TDD (Test‑Driven Development) | Escrever testes antes do código, em ciclos curtos (test–code–refactor). | Especificação é implícita nos testes; foco em unidade e design incremental.[^8][^9][^10] | SDD tende a anteceder TDD: usa specs mais amplas (contratos, critérios de aceitação, schemas). Testes podem ser derivados automaticamente destas specs. Muitas propostas combinam “spec + TDD”.[^11] |
| BDD (Behavior‑Driven Development) | Descrever comportamentos em cenários Given–When–Then executáveis.[^12][^13] | Especificações executáveis em linguagem ubíqua, frequentemente usadas como documentação viva.[^14] | SDD pode englobar BDD como uma forma específica de spec executável. Vários trabalhos tratam BDD como uma instância de specification‑by‑example, alinhada com SDD, mas SDD foca também em arquitetura, contratos e segurança. |
| DDD (Domain‑Driven Design) | Modelar domínio, linguagem ubíqua, bounded contexts, agregados etc.[^15][^16] | Modelo de domínio e linguagem ubíqua são formas de especificação conceitual e de design. | SDD pode consumir modelos DDD como insumo para specs (por exemplo, explicitar invariantes de agregados, regras de negócio e context maps). Porém DDD não dita necessariamente um fluxo “spec‑first para IA”; é mais conceitual. |
| MDD/MDE (Model‑Driven Development/Engineering) | Usar modelos formais (UML, DSLs) como artefatos primários e gerar código a partir dos modelos.[^17] | Modelos são specs formais ou semi‑formais que podem ser transformadas em código. | SDD é conceitualmente próximo, mas usualmente usa specs em texto estruturado + schemas em vez de modelos gráficos. Em alguns domínios, SDD pode ser visto como uma forma pragmática de MDD focada em IA. |
| Design by Contract (DbC) | Contratos (pré/pós‑condições, invariantes) como parte do código para garantir correção.[^18][^19] | Contratos são especificações executáveis e verificáveis, fortemente acopladas ao código. | SDD herda princípios de DbC (contratos claros, verificáveis), mas desloca o foco para specs em nível de feature, API e arquitetura, muitas vezes fora do código e consumíveis por IA. Trabalhos como “Constitutional SDD” explicitamente codificam contratos de segurança em documentos de constituição.[^20][^21] |
| Documentação viva | Documentação que evolui junto com o sistema, frequentemente gerada de testes BDD, código e artefatos de build.[^15][^22][^14][^23] | Especificações e documentação são continuamente atualizadas e, muitas vezes, executáveis. | SDD normalmente exige que o spec seja um artefato vivo e versionado; combina bem com práticas de living documentation e docs‑as‑code para evitar drift entre spec e código. |
| ADRs (Architecture Decision Records) | Registros curtos de decisões arquiteturais e seu contexto e consequências.[^24][^25][^26] | Documentam decisões, trade‑offs e alternativas; não são specs completas. | SDD complementa ADRs: ADRs explicam o porquê das decisões; o spec descreve o quê e o como. Muitos workflows SDD recomendam registrar decisões arquiteturais críticas em ADRs referenciadas pela especificação. |
| Desenvolvimento assistido por IA (code assistants, agents) | Usar LLMs para gerar, modificar e revisar código a partir de prompts. | “Especificação” muitas vezes é um prompt curto, issue de GitHub ou comentário informal. | SDD é uma resposta aos limites do “vibe coding”: formaliza o uso de IA a partir de specs mais ricas, previsíveis e verificáveis, tratando IA como executor de intent, não como adivinho.[^27][^28][^5] |

### Nomenclatura, variações e grau de consolidação

A literatura recente usa explicitamente o termo “Spec‑Driven Development (SDD)” para descrever práticas em que especificações textuais estruturadas, frequentemente em Markdown, orientam agentes de IA na geração de código. O termo “Specification‑Driven” também aparece em estudos sobre geração de código com LLMs, com foco em workflows spec–tests–code.[^27][^3][^29][^5][^30]

Historicamente, “specification‑driven” já foi usado em contextos de Design by Contract e ensino com Eiffel, mas com foco em especificação formal de componentes, não em IA. A nova onda de SDD é, portanto, mais uma recombinação de ideias clássicas (SRS, DbC, MDD, BDD, docs‑as‑code) sob o contexto de IA generativa do que uma metodologia totalmente inédita.[^31][^32]

Quanto ao grau de formalização:

- A academia começa a tratar SDD como tema emergente de pesquisa, com artigos em arXiv, SANER e estudos de projeto, mas ainda não é um padrão normatizado como IEEE 29148.[^20][^5][^30][^4]
- A indústria (IBM, Microsoft, Thoughtworks, DeepLearning.AI e Google developers) apresenta SDD como uma prática de engenharia recomendada para uso eficaz de agentes de IA, muitas vezes vinculada a ferramentas específicas (Spec Kit, KIRO, Antigravity etc.).[^33][^2][^34][^28][^35][^36][^37][^38]

Em resumo, SDD hoje é melhor entendido como: prática emergente, com forte resgate de conceitos clássicos de especificação e uma camada nova de operacionalização para agentes de IA.


## 2. Como a academia vê e usa SDD e especificações

### Presença explícita do termo “Spec‑Driven Development”

Há pelo menos três trabalhos recentes que utilizam explicitamente o termo “Spec‑Driven Development” no contexto de IA:

- Piskala (2026), “Spec‑Driven Development: From Code to Contract in the Age of AI Coding Assistants”, propõe princípios, níveis de rigor (spec‑first, spec‑anchored, spec‑as‑source) e um framework de decisão quando aplicar SDD, com estudos de caso em APIs, sistemas corporativos e embarcados.[^4]
- Marri (2026), “Constitutional Spec‑Driven Development”, foca em segurança: define uma “Constitution” de segurança, versionada e machine‑readable, que codifica princípios derivados de CWE/MITRE e regulamentações, e mostra empiricamente redução de 73% em defeitos de segurança em comparação a geração irrestrita.[^20]
- Taghavi e Bhavani (2026), “Spec Kit Agents: Context‑Grounded Agentic Workflows”, apresentam uma arquitetura multi‑agente baseada em SDD com fases Specify–Plan–Tasks–Implement e hooks de grounding para repositório, obtendo ganhos em qualidade julgada e compatibilidade com testes.[^5]

Além disso, Rosa et al. (2026) apresentam um estudo empírico planejado sobre “Specification‑Driven Code Generation with LLMs”, usando a ferramenta CURRANTE: workflow em três estágios (Specification, Tests, Function), com logging detalhado de interações para entender como qualidade da especificação e dos testes afeta o código gerado.[^30]

Esses trabalhos ainda são predominantemente conceituais ou de design de estudo, com alguns estudos de caso e experimentos controlados, mas mostram um movimento claro da academia em analisar SDD como padrão de interação com LLMs.

### Evidências sobre benefícios da especificação prévia

Embora o rótulo “SDD” seja recente, há vasta literatura sobre requisitos, especificações e traceability que fundamenta os benefícios de especificação prévia:

- Normas como ISO/IEC/IEEE 29148 definem processos e critérios de qualidade para requisitos, reforçando clareza, não ambiguidade, verificabilidade e rastreabilidade como atributos desejáveis.[^6][^39][^7]
- Estudos clássicos de traceabilidade (Gotel & Finkelstein, 1994; Ramesh & Jarke, diversos trabalhos) mostram que requisitos bem estruturados e rastreáveis facilitam manutenção, impacto de mudanças e conformidade regulatória.[^40][^41][^42]
- Revisões sistemáticas recentes indicam que requisitos de boa qualidade (não ambíguos, completos, consistentes) melhoram a capacidade de recuperar links de traceabilidade e reduzir defeitos, ao passo que requisitos vagos dificultam automação e análise.[^43][^44][^45]
- Trabalhos em BDD e specification‑by‑example mostram que cenários executáveis reduzem ambiguidade, promovem alinhamento entre stakeholders e servem como documentação viva, especialmente quando integrados a pipelines automatizados.[^13][^15][^22][^14]

No contexto específico de LLMs e SDD:

- Taghavi e Bhavani demonstram empiricamente que hooks de grounding em um workflow SDD melhoram a qualidade e reduzem violações de contexto e APIs, medidos por avaliações com LLM‑as‑judge e testes automáticos.[^5]
- Rosa et al. desenham um estudo para quantificar como qualidade do spec e do test suite influencia pass rate, tempo até solução e padrões de iteração, reforçando a hipótese de que specs estruturadas são determinantes para eficácia de LLM‑based code generation.[^30]
- Trabalhos em guidelines de prompting para code generation identificam que especificação clara de I/O, pré‑condições, exemplos e restrições melhora significativamente taxas de sucesso, servindo como evidência indireta a favor de SDD.[^46]

Em termos de nível de evidência, a situação atual pode ser resumida assim:

- Evidência empírica moderada: casos de estudo em contextos específicos (segurança, pipelines multi‑agente, ferramentas IDE como CURRANTE) e resultados preliminares em benchmarks.[^20][^5][^30]
- Estudos de caso e experiências industriais relatadas: principalmente em whitepapers e blogs institucionais (IBM, DeepLearning.AI, empresas de tooling), frequentemente sem avaliação rigorosa, mas coerentes com literatura de requisitos e BDD.[^28][^47][^29]
- Propostas conceituais: frameworks e taxonomias de SDD, níveis de rigor e integrações com ferramentas de agentes.[^48][^4]
- Documentação técnica sem validação formal: material de cursos, codelabs e documentação de ferramentas de IA que promovem specs estruturadas como boas práticas.[^3][^37][^38]

### Críticas e limitações relacionadas à especificação pesada

Críticas clássicas à documentação e especificações extensas continuam válidas no contexto de SDD:

- Custo inicial e overhead: normas como 29148 e livros de requirements engineering enfatizam o custo de produzir e manter documentação detalhada, especialmente quando a organização não possui disciplina de gestão de requisitos.[^49][^6]
- Risco de desatualização: literatura sobre living documentation e docs‑as‑code destaca o problema de documentação que não acompanha o código, reduzindo sua confiabilidade.[^15][^50][^14]
- Falsa sensação de precisão: especificações em linguagem natural podem parecer precisas, mas ainda conter ambiguidades, o que é agravado quando LLMs interpretam essas ambiguidades de forma imprevisível.[^44][^51]
- Rigidez em contextos ágeis: estudos de adoção de processos pesados de requisitos em ambientes agile mostram tensões entre planejamento extensivo upfront e descoberta contínua de requisitos emergentes.[^52][^53]
- Dificuldade de capturar requisitos emergentes e não funcionais: revisões de traceability indicam baixa cobertura de requisitos não funcionais e de qualidades como segurança, privacidade e observabilidade.[^45][^43]

Trabalhos recentes em SDD procuram mitigar essas limitações com:

- Níveis graduais de rigor (spec‑first vs spec‑anchored vs spec‑as‑source), escolhidos conforme risco e domínio.[^4]
- Integração com documentação viva, BDD, docs‑as‑code e ADRs para reduzir custo de manutenção e manter rastreabilidade.[^14][^24][^15]
- Uso de IA para auxiliar na criação e atualização de specs, desde que com ciclos de revisão humana e testes automáticos.[^21][^5][^30]


## 3. SDD no contexto de IA generativa

### Como modelos de IA leem e usam especificações estruturadas

Modelos baseados em LLM consomem specs como texto, muitas vezes enriquecido com estruturas como Markdown, YAML, TOML, JSON, XML ou schemas formais. O fluxo típico é:[^54][^55][^56][^46]

1. O desenvolvedor fornece um spec (por exemplo, `mission.md`, `tech-stack.md`, `feature-spec.md`) como parte do contexto do modelo.[^29][^54]
2. O prompt indica explicitamente como o modelo deve usar o spec (por exemplo, “implemente apenas o que está definido na seção X”, “não invente endpoints não listados no schema”).[^57][^54]
3. O modelo gera código, testes ou documentação complementar com base nesse contexto; em workflows mais avançados, agentes consultam o repositório, rodando hooks de grounding para alinhar spec, código existente e testes.[^5]
4. Ferramentas de validação (tests, linters, verificadores estáticos, structured outputs/JSON Schema) verificam se o resultado está consistente com a especificação.[^56][^20][^5]

Plataformas de IA modernas (Google Gemini, Anthropic Claude, OpenAI) recomendam explicitamente prompts estruturados, uso de tags ou delimitadores para separar instruções, dados e exemplos, schemas para outputs e uso de poucos‑shots demonstrando o formato esperado.[^58][^59][^60][^55][^61][^62][^56]

### Impacto da qualidade da especificação

Estudos e orientações técnicas convergem em alguns efeitos da qualidade do spec sobre o código gerado:

- Qualidade do código gerado: especificações detalhadas de I/O, tipos, restrições e edge cases aumentam taxa de sucesso em testes e reduzem iterações de prompt‑refinement; trabalhos em guidelines de prompting e CURRANTE mostram forte correlação entre clareza do spec/test suite e pass rate.[^63][^46][^30]
- Aderência a requisitos: specs que incluem critérios de aceitação e cenários de uso explícitos ajudam modelos BDD‑like a alinhar comportamentos, reduzindo discrepâncias entre “o que eu queria” e “o que o código faz”.[^13][^14][^5]
- Testabilidade: quando specs incluem condições observáveis e mensuráveis, derivar testes (manualmente ou via IA) torna‑se mais direto; CURRANTE e pesquisas de prompt‑guidelines reforçam que especificar pré/pós‑condições e casos extremos melhora geração de testes e oráculos.[^46][^30]
- Segurança: Constitutional SDD mostra que codificar requisitos de segurança em uma “Constitution” machine‑readable, usada em todas as chamadas de geração, reduz significativamente defeitos de segurança, deslocando a preocupação de verificação reativa para prevenção por construção.[^21][^20]
- Consistência arquitetural: frameworks como Spec Kit Agents e ferramentas de SDD para infraestrutura mostram como specs de arquitetura (camadas, dependências permitidas, contratos de API) podem ser usadas para evitar violações estruturais, com hooks de validação automatizados.[^38][^48][^5]
- Redução de alucinações: orientações de Anthropic e OpenAI recomendam separar dados de instruções, usar tags e schemas, e pedir ao modelo que cite fontes ou se limite a informações fornecidas; isso reduz a necessidade do modelo “inventar” detalhes, mitigando hallucinations.[^55][^61][^62][^64][^57]
- Capacidade de revisão humana: specs bem estruturadas tornam revisões de código gerado mais eficientes, pois críticos podem verificar diretamente se o código implementa cada requisito e critério de aceitação, e ferramentas podem mapear trechos de código a elementos do spec.[^65][^24][^4]

### Práticas usuais em SDD com IA

A prática emergente de SDD com IA integra várias técnicas:

- Prompts estruturados com seções claras (contexto, instruções, dados, exemplos, formato de saída), usando delimitadores ou tags para cada parte.[^59][^62][^58][^55]
- Arquivos de especificação versionados (ex.: `mission.md`, `tech-stack.md`, `roadmap.md`, specs por feature) que servem de “Constituição” do projeto e de cada funcionalidade.[^3][^29][^54]
- Documentação como fonte de verdade, seguindo princípios de docs‑as‑code (Markdown, version control, CI para documentação), para garantir que spec e código evoluam juntos.[^50][^15]
- Uso de acceptance criteria e exemplos concretos (incluindo contraexemplos) para cada requisito funcional e regra de negócio, frequentemente em estilo BDD.[^22][^14][^13]
- Contratos de API e schemas (OpenAPI, JSON Schema, Protobuf) como parte central da especificação, usados diretamente em function/tool calling ou structured outputs.[^66][^56][^20][^5]
- User stories e casos de uso integrados ao spec, servindo tanto para comunicação com stakeholders quanto para derivação de testes e prompts.[^49][^52]
- Testes derivados da especificação, seja manualmente, seja com apoio de IA (por exemplo, CURRANTE gera testes a partir de spec TOML, que são então revisados por humanos).[^30]
- Geração incremental de código, em que cada feature ou tarefa é implementada a partir de um spec específico, validada e apenas então integrada; frameworks como Spec Kit Agents formalizam esse ciclo multi‑fase.[^29][^5]

### Papel da IA na criação e interpretação de especificações e riscos envolvidos

LLMs podem atuar em várias etapas do ciclo de especificação:

- Auxiliar na redação de specs a partir de rascunhos de requisitos, issues, conversas com stakeholders ou documentos legados, aplicando templates e sugerindo seções obrigatórias.[^54][^3]
- Refinar specs: detectar ambiguidades, inconsistências, requisitos não testáveis, ou lacunas em requisitos de segurança, privacidade e performance.[^21][^20]
- Completar e reestruturar specs em formatos mais adequados a consumo por agentes (por exemplo, transformar prosa em tabelas, schemas, DSLs).[^56][^46]
- Interpretar specs para gerar código, testes, migrações, scripts e documentação de API.[^27][^28][^5]

Por outro lado, surgem riscos claros quando a IA é usada sem controles:

- Invenção de requisitos: se o spec é incompleto, modelos podem “preencher lacunas” sem deixar claro o que é requisito real e o que é inferência; pesquisas sobre ambiguidades de requisitos mostram que mesmo humanos erram, e LLMs amplificam esse problema.[^51][^44]
- Omissão de restrições: requisitos não funcionais e restrições (limites de throughput, latência, privacidade, regulamentação) são frequentemente subespecificados, e IA tende a ignorá‑los a menos que sejam codificados explicitamente na especificação (daí o foco em Constitutional SDD).[^20][^21]
- Interpretação ambígua: sem estruturas claras (tags, tabelas, schemas), o modelo pode confundir exemplos com requisitos, misturar escopo confirmado com ideias futuras, ou aplicar decisões arquiteturais obsoletas.[^62][^67][^55]
- Desalinhamento com stakeholders: se specs gerados por IA não forem validados por pessoas de negócio, há risco de a “fonte de verdade” ser um documento que nunca foi realmente acordado, o que contraria princípios de requirements engineering.[^6][^49]

Por isso, a literatura e as recomendações institucionais enfatizam o papel de humanos na revisão e aprovação das especificações, sobretudo em domínios críticos (finanças, saúde, sistemas de segurança).[^26][^52][^20]


## 4. Como criar boas especificações para SDD

### Modelo prático de especificação ideal

Combinando normas de requisitos (ISO/IEC/IEEE 29148), práticas de SRS, BDD, DDD, ADRs e recomendações de SDD, um modelo de especificação para software moderno poderia incluir as seguintes seções mínimas:

1. **Objetivo do sistema ou funcionalidade**  
Resumo conciso do que a feature ou sistema faz, por que existe e qual problema resolve. Idealmente 2–4 frases, orientadas a valor de negócio.[^7][^6]

2. **Contexto de negócio**  
Descrição do domínio, processos afetados, métricas de sucesso (KPIs) e stakeholders principais. Pode incluir diagramas simples (context diagram, mapas de processo) quando útil.[^16][^49]

3. **Usuários e personas**  
Perfis de usuários (internos e externos), necessidades, limitações e linguagem que utilizam, alinhados com a linguagem ubíqua de DDD.[^15][^16]

4. **Escopo**  
Lista do que está incluído na funcionalidade, preferencialmente em bullets, vinculados a objetivos de negócio específicos.[^6]

5. **Fora de escopo**  
Itens explicitamente excluídos, para reduzir suposições da IA e do time.[^7][^6]

6. **Requisitos funcionais**  
Requisitos numerados (RF‑01, RF‑02 …), cada um:
   - Formulado de forma clara e verificável.
   - Com referência a critérios de aceitação associados.[^7][^6]

7. **Requisitos não funcionais**  
Categorias como performance, disponibilidade, segurança, privacidade, usabilidade, observabilidade etc., todos expressos com métricas sempre que possível (por exemplo, latência P95, SLAs, volumetria planejada).[^68][^6][^7]

8. **Critérios de aceitação**  
Para cada requisito funcional (ou grupo), critérios objetivos que determinam “pronto”, preferencialmente em formato BDD ou checklists observáveis.[^22][^13]

9. **Regras de negócio**  
Regras declarativas (“se … então …”), invariantes de domínio, políticas de tarifação, prazos, limites; idealmente associadas a entidades ou contextos de DDD.[^16]

10. **Fluxos principais**  
Cenários “happy path” (por exemplo, fluxo de compra), descritos em passos ou diagramas de sequência simples. Úteis para orientar tanto testes quanto prompts estruturados.[^14][^15]

11. **Fluxos alternativos e exceções de negócio**  
Fluxos para erros de domínio (saldo insuficiente, permissões insuficientes) e para variações válidas. Devem ser claramente marcados para evitar que a IA os trate como casos “marginais” irrelevantes.[^52][^49]

12. **Tratamento de erros técnicos**  
Estratégia de tratamento de erros em nível de infraestrutura (timeouts, falhas de rede, fallbacks, retries, circuit breakers), códigos de erro e mensagens padrão.[^68]

13. **Permissões e papéis**  
Matriz de autorização (RBAC/ABAC), mapeando operações a papéis e privilégios, alinhada a requisitos de segurança e privacidade.[^6][^7]

14. **Dados de entrada e saída**  
Estrutura dos dados (campos, tipos, constraints), validações de fronteira, formatos de serialização (JSON, XML, protobuf, etc.). Idealmente em tabelas ou schemas.[^56][^6]

15. **Contratos de API**  
Especificações OpenAPI/gRPC/GraphQL, com endpoints, métodos, parâmetros, códigos de resposta e exemplos de requests/responses. Esses contratos são cruciais para function/tool calling e structured outputs.[^66][^56]

16. **Dependências**  
Sistemas externos, serviços internos, filas, bancos, bibliotecas de terceiros; incluir contratos e SLAs dependentes.[^68]

17. **Restrições técnicas**  
Tecnologias obrigatórias ou proibidas, limitações de plataforma, compatibilidade (por exemplo, versões de SDK, banco de dados existente).[^69][^68]

18. **Requisitos de segurança**  
Controles mínimos (autenticação, autorização, criptografia, logging seguro), ameaças relevantes, requisitos derivando de constituições de segurança (se existir), frameworks de referência (OWASP, CWE, etc.).[^21][^20]

19. **Requisitos de privacidade**  
Coleta, armazenamento e processamento de dados pessoais; minimização de dados; retenção; consentimento; requisitos de anonimização ou pseudonimização, mapeados a regulamentações aplicáveis (LGPD, GDPR etc.).[^7][^6]

20. **Requisitos de observabilidade**  
Logs, métricas, traces, dashboards desejados, KPIs de negócio e técnicos que devem ser instrumentados.[^68]

21. **Requisitos de performance e escalabilidade**  
Metas de throughput, latência, limites de uso, estratégias de escalonamento, testes de carga previstos.[^68]

22. **Rastreabilidade entre requisito, implementação e teste**  
Mecanismo (tabelas, matrizes, tags em código e testes) que liga requisitos e regras de negócio a módulos, commits e casos de teste. Essencial para SDD e traceability.[^43][^45]

23. **Exemplos concretos e casos extremos**  
Conjunto de exemplos com dados realistas e casos extremos (valores de fronteira, inputs inválidos, cenários de stress). Podem ser usados diretamente para geração de testes via IA.[^46][^30]

24. **Decisões arquiteturais**  
ADRs resumidos ou referências a ADRs com decisões chave (estilo arquitetural, padrões, decisões de tecnologia e trade‑offs). Devem ser curtas e rastreáveis ao spec.[^24][^26]

25. **Critérios de pronto (Definition of Done)**  
Condições para considerar a feature pronta: cobertura de testes, revisão de código, aprovação de segurança, documentação atualizada, monitoria em produção etc.[^70][^26]

### Linguagem natural vs estruturas formais

Boas especificações para SDD tendem a combinar:

- **Linguagem natural estruturada**: usada para contexto de negócio, objetivos, narração de fluxos, explicação de trade‑offs e ADRs. Deve ser clara, concisa e evitar ambiguidade; a estrutura (títulos, listas, seções) ajuda LLMs a navegar.[^71][^15]
- **Estruturas formais ou semi‑formais**:
  - **Tabelas** para requisitos, regras de negócio, matrizes de permissão, campos de dados e rastreabilidade.[^6][^7]
  - **Schemas** (JSON Schema, OpenAPI, Protobuf, GraphQL SDL) para definir contratos de API e formatos de dados consumidos por tool/function calling e structured outputs.[^66][^56]
  - **Exemplos codificados** (casos BDD, exemplos de request/response, payloads de eventos) para few‑shot prompting e geração de testes.[^13][^22][^46]
  - **Pseudocódigo e DSLs** para algoritmos complexos, quando for útil dar ao modelo uma referência de lógica sem ficar preso a detalhes de linguagem.[^17]
  - **Diagramas** (C4, sequência, estados) armazenados como código (Mermaid, PlantUML) para facilitar sincronização com repositório e geração automatizada.[^72][^50]
  - **Testes especificados** (ex.: suites de teste de aceitação em Gherkin, casos de teste em TOML/YAML como no CURRANTE) para tornar os requisitos executáveis.[^23][^30]

A combinação ideal para SDD com IA é aquela em que tudo o que precisa ser lido por humanos permanece legível, mas tudo o que precisa ser verificado ou consumido por ferramentas é estruturado de forma explícita e machine‑readable.


## 5. Recomendações de empresas e fontes oficiais

### Google

Google publica uma série de materiais relevantes, embora não sob o rótulo “SDD”:

- **Software Engineering at Google – capítulo “Documentation is Like Code”** destaca que documentação deve ser tratada como código: versionada, revisada, testada e com donos claros, dividindo‑a em tipos (tutoriais, documentação conceitual, de referência) e incentivando design docs como artefatos de alinhamento antes de grandes mudanças.[^69][^71]
- **Google Engineering Practices (eng‑practices)** documenta guidelines de code review e práticas gerais de engenharia, enfatizando clareza, pequenas mudanças, justificativa de design e rastreabilidade de decisões nos CLs; design docs aparecem como parte do processo interno, ainda que o template não esteja formalmente publicado ali.[^70]
- **Google Testing Blog** discute diferentes abordagens para test planning, incluindo o “10 Minute Test Plan” e o “Inquiry Method for Test Planning”, que priorizam foco em atributos, componentes e capacidades em vez de documentos extensos e estáticos; a mensagem central é que planejamento importa mais que o artefato, e documentos devem ser vivos e pragmáticos.[^73][^74]
- **Google Cloud Architecture Framework** fornece diretrizes sobre operacionalidade, segurança, confiabilidade e otimização de performance e custo, com perguntas de design e recomendações que podem ser transcritas em seções de especificação (requisitos não funcionais, decisões de arquitetura, observabilidade).[^68]
- **Documentação de prompt design para Gemini** recomenda prompts estruturados, delimitação clara entre background, instruções e exemplos, uso de tags e delimitações (`##`, `<instruction>` etc.), foco em tarefas específicas, concisão e uso de schemas para function calling e structured outputs.[^75][^58][^59]

Esses materiais se alinham ao SDD na medida em que:

- Reforçam o papel de design docs como passo anterior ao código, capturando escopo, trade‑offs e critérios de sucesso.[^71]
- Recomendam documentação versionada e próxima do código (docs‑as‑code), essencial para specs que devem guiar agentes de IA de forma confiável.[^50][^71]
- Enfatizam requisitos não funcionais (segurança, confiabilidade, observabilidade) como parte central de qualquer design, coerente com “Constitutional SDD”.[^20][^68]
- Para Gemini, enfatizam prompts claros, estruturados, com exemplos e delimitações bem definidas — essencialmente mini‑specs para tarefas pontuais.[^58][^59]

### Anthropic

Anthropic mantém documentação oficial extensa sobre prompt engineering e uso de Claude:

- O whitepaper “Effective Prompt Engineering – 6 Techniques” destaca: fornecer contexto, dar exemplos, especificar formato de saída, quebrar tarefas em passos, pedir para o modelo “pensar antes de responder” e definir papel/persona do modelo.[^60]
- Tutoriais oficiais de prompt engineering recomendam uso de tags XML (`<instructions>`, `<data>`, `<examples>`, `<thinking>`), prompts estruturados e separação clara entre dados e instruções, além de cadeias de prompts para tarefas complexas.[^55][^57]
- Documentação de structured outputs mostra como forçar JSON válido com schemas, facilitando uso de Claude como componente em pipelines que exigem outputs estruturados.[^76][^55]
- Guias sobre contexto longo orientam colocar documentos grandes antes das instruções e usar tags para identificar cada documento, pedindo que o modelo cite trechos relevantes antes de responder — essencialmente um padrão de grounding e verificação que ecoa SDD.[^77]

Essas recomendações se relacionam diretamente com SDD porque:

- Tratam prompts como especificações: claras, estruturadas, com exemplos e formats predefinidos.[^60][^57]
- Incentivam uso de schemas e outputs estruturados, que são a base de specs machine‑readable que agentes podem consumir e validar.[^76][^55]
- Recomendam separar raciocínio interno (<thinking>) de respostas, o que permite usar SDD não apenas para código, mas também para justificar decisões e construir artefatos de documentação.[^57]

### OpenAI

A documentação oficial da OpenAI fornece recomendações explícitas que podem ser lidas como guidelines para especificações consumíveis por IA:

- O guia de prompt engineering recomenda: colocar instruções no início, separar instruções de contexto com delimitadores, ser específico e detalhado quanto a contexto, resultado, formato e estilo, demonstrar formatos desejados com exemplos, reduzir ambiguidade e iterar prompts.[^61][^78][^62]
- A funcionalidade de **Structured Outputs / JSON mode** permite garantir que outputs sigam exatamente schemas JSON fornecidos, especialmente quando usada com `strict: true`; isso transforma parte da especificação (schemas) em contrato rígido para o modelo, evitando variações e facilitando testes.[^79][^56][^66]
- Documentação de function/tool calling mostra como definir funções/ ferramentas com schemas e deixar que o modelo decida quando chamá‑las, conectando especificações de API diretamente ao comportamento da IA.[^80][^81][^82]
- A documentação de Agents e Agents SDK enfatiza configuração de agentes com instruções estáticas, ferramentas, guardrails, handoffs e outputType (para outputs tipados), bem como uso de avaliações e tracing para monitorar aderência a expectativas.[^64][^83][^84]

No contexto de SDD, essas práticas suportam:

- Especificações estruturadas para tarefas e APIs (JSON Schema, OpenAPI) que o modelo deve seguir exatamente.[^56][^66]
- Divisão clara entre instruções de alto nível (system/developer messages), contexto de usuário e dados de entrada, similar a separar spec de requisitos de inputs específicos de uma chamada.[^85][^86][^61]
- Instrumentação e evals para medir quão bem o modelo cumpre critérios de aceitação derivados de uma especificação.[^87]


## 6. Boas práticas (“o que fazer”) em SDD

A seguir, práticas recomendadas categorizadas por fase, sintetizando literatura acadêmica, normas e recomendações institucionais.

### Antes de escrever código

- **Clarificar escopo e objetivos**: definir claramente o problema, objetivos de negócio, escopo e fora de escopo antes de qualquer geração de código, em linha com ISO/IEC/IEEE 29148 e boas práticas de SRS.[^7][^6]
- **Escrever especificações verificáveis**: formular requisitos em termos observáveis e testáveis (valores, condições, tolerâncias), evitando termos vagos como “rápido” ou “intuitivo”.[^43][^6]
- **Definir critérios de aceitação objetivos**: associar critérios a cada requisito funcional, de preferência em formato BDD ou checklists mensuráveis.[^22][^13]
- **Declarar explicitamente o que está fora de escopo**: minimizar inferências indevidas por IA e por desenvolvedores, reduzindo risco de escopo oculto.[^6]
- **Incluir exemplos e contraexemplos**: para entradas típicas, casos extremos e cenários proibidos; isso auxilia tanto LLMs quanto testers humanos.[^30][^46]
- **Endereçar NFRs e riscos desde o início**: incorporar requisitos de segurança, privacidade, performance, confiabilidade e observabilidade já na especificação, especialmente em domínios regulados.[^20][^6][^68]
- **Selecionar o nível adequado de rigor**: decidir entre spec‑first, spec‑anchored ou spec‑as‑source conforme risco, tamanho do projeto e maturidade da equipe, conforme frameworks de SDD.[^4]

### Durante a geração ou implementação (com IA ou sem)

- **Usar prompts estruturados que apontem para o spec**: separar claramente seções de contexto, instruções, spec e formato de saída, indicando ao modelo para seguir apenas o que está no spec e pedir clarificações em caso de lacunas.[^61][^58][^55]
- **Passar contratos e schemas diretamente ao modelo**: usar JSON Schema/OpenAPI/grpc/proto como parte do prompt ou via structured outputs/tool calling, para reduzir interpretações ambíguas.[^81][^66][^56]
- **Gerar código incrementalmente por feature**: cada ciclo deveria focar em uma especificação de feature específica, com branch separado e critérios de saída claros, como recomendado por cursos de SDD e agentes.[^3][^29][^5]
- **Deixar a IA implementar, não decidir requisitos**: usar a IA para geração de código e testes a partir de specs, evitando prompts vagos que transfiram decisão de negócio para o modelo.[^88][^5][^20]

### Durante revisão humana

- **Revisar a especificação antes do código**: revisar, priorizar e eventualmente ajustar o spec com stakeholders e especialistas técnicos antes de gerar ou revisar código.[^49][^6]
- **Realizar code reviews orientados a spec**: avaliar se o código atende a cada requisito e critério de aceitação, em vez de apenas revisar estilo; isso está alinhado com práticas de code review do Google e de engenharia moderna.[^70][^71]
- **Checar divergências entre spec e código**: usar ferramentas de rastreabilidade e testes baseados em spec para identificar incoerências, atualizando spec ou código conforme necessário.[^89][^43]
- **Registrar decisões em ADRs**: documentar mudanças significativas de arquitetura ou trade‑offs, referenciando requisitos e specs afetados.[^24][^26]

### Durante testes

- **Derivar testes diretamente da especificação**: mapear critérios de aceitação, regras de negócio e casos extremos para casos de teste; frameworks como BDD e ferramentas como CURRANTE seguem explicitamente essa estratégia.[^13][^30]
- **Automatizar testes de regressão baseados em spec**: manter suites automatizadas que validem invariantes de domínio, contratos de API e NFRs mensuráveis.[^23][^5][^13]
- **Usar a IA para sugerir testes adicionais, mas com verificação humana**: IA pode propor mais casos a partir do spec, mas a seleção final deve ser humana, especialmente para riscos críticos.[^46][^30]

### Durante manutenção

- **Versionar especificações junto com o código**: specs devem viver no mesmo repositório, sob controle de versão, com revisões e histórico tão rigorosos quanto o código.[^50][^71]
- **Manter rastreabilidade entre requisitos, código e testes**: atualizar links sempre que requisitos mudarem, usando tags, matrizes ou ferramentas especializadas.[^89][^43]
- **Evoluir documentação como living documentation**: automatizar geração de visões (relatórios, diagramas), derivadas de specs e testes, para manter documentação atualizada.[^15][^14][^23]

### Durante uso com IA generativa

- **Criar templates consistentes para prompts e specs**: usar estruturas repetíveis para tarefas (por exemplo, “Plan → Implement → Validate” de SDD), facilitando automação e reuso.[^54][^57][^3]
- **Separar instruções estáticas (constituições) de contexto dinâmico**: manter documentos de constituição (princípios de segurança, padrões arquiteturais) que são sempre incluídos nos prompts, complementados por specs de feature e contexto da tarefa.[^64][^29][^20]
- **Aplicar guardrails e evals**: medir continuamente aderência às especificações (por testes, evals e tracing), ajustando prompts e specs conforme necessário.[^87][^64][^5][^20]


## 7. Más práticas (“o que não fazer”) e riscos

Principais más práticas a evitar em SDD, sobretudo com IA:

- **Especificações vagas ou superficiais**: requisitos genéricos (“deve ser rápido”, “fácil de usar”) sem métricas ou critérios de aceitação; isso leva a interpretações arbitrárias, tanto por IA quanto por humanos.[^43][^6]
- **Requisitos não testáveis**: requisitos que não podem ser verificados objetivamente (por exemplo, “interface bonita” sem critérios) minam a possibilidade de usar testes como oráculos derivados do spec.[^13][^6]
- **Deixar a IA inferir regras de negócio críticas**: prompts vagos ou incompletos para domínios regulados ou financeiros, confiando que o modelo “faça o certo” sem uma especificação clara; estudos de segurança e requisitos ambíguos mostram que isso tende a introduzir falhas sutis.[^90][^51][^20]
- **Misturar escopo confirmado com ideias futuras no mesmo spec**: documentos que misturam roadmap aspiracional com requisitos aprovados levam IA e desenvolvedores a implementarem funcionalidades não priorizadas ainda.[^29][^71]
- **Omitir requisitos não funcionais, segurança e privacidade**: tratar apenas funcionalidades e deixar NFRs “implicitos” é particularmente perigoso em SDD, pois agentes de IA tendem a otimizar apenas a funcionalidade aparente.[^20][^6][^68]
- **Não versionar documentação**: specs em wikis não versionadas, documentos fora do repositório e sem histórico previsível; isso rompe rastreabilidade e dificulta auditoria de decisões.[^52][^50]
- **Criar especificações longas sem estrutura**: documentos densos, sem seções, tabelas ou modelos repetíveis, são difíceis de navegar para humanos e LLMs, aumentando o risco de interpretações erradas.[^15][^50]
- **Confiar cegamente no código gerado pela IA**: uso de IA como única validação, sem testes adequados, revisões humanas ou verificação contra o spec; trabalhos em segurança e guidelines ressaltam necessidade de validação independente.[^82][^5][^20]
- **Transformar especificação em mera burocracia**: produzir documentos apenas para cumprir checklist, sem usá‑los de fato como entrada para desenvolvimento, testes e IA; literatura em living documentation critica esse padrão.[^14][^15]
- **Tratar documentação como substituto de comunicação com stakeholders**: specs não substituem conversas, discovery contínuo e feedback; normas de requirements enfatizam atividades de elicitação e validação com stakeholders.[^49][^6]
- **Usar fontes informais não verificadas como base factual**: usar opiniões de blog posts não institucionais para codificar regras de negócio ou práticas sem checagem cruzada, especialmente críticas em domínios regulados.[^52][^6]


## 8. Workflow recomendado de SDD com IA

Com base em frameworks propostos (SDD, Spec Kit Agents, Constitutional SDD, CURRANTE) e em práticas de empresas, um fluxo prático de SDD com IA pode ser estruturado em etapas:

1. **Descoberta do problema**  
   - Entrevistas com stakeholders, análise de processos, estudo de métricas existentes.[^49][^52]
   - Saída: conjunto inicial de objetivos de negócio e hipóteses.

2. **Levantamento de requisitos**  
   - Refinamento de requisitos funcionais e não funcionais, incluindo segurança e privacidade, com técnicas de elicitação e priorização.[^7][^6]
   - Saída: rascunho de SRS/SRS‑lite para a funcionalidade.

3. **Escrita da especificação (spec)**  
   - Consolidação do spec de feature, em formato estruturado (Markdown + schemas + exemplos), usando eventualmente IA para auxiliar na formatação e completude, mas com revisão humana.[^3][^29][^30]
   - Se necessário, atualização da “Constituição” do projeto (documento de princípios de segurança, arquitetura e políticas globais).[^21][^20]

4. **Revisão da especificação**  
   - Revisão com stakeholders de negócio, arquitetos, segurança e QA; ajustes até consenso mínimo.
   - Saída: spec versão 1.0, aprovada e versionada no repositório.[^26][^6]

5. **Decomposição em tarefas**  
   - Quebra do spec em tarefas implementáveis (user stories, tasks técnicas), vinculadas a seções do spec e ADRs planejados.[^24][^26]

6. **Geração de testes a partir do spec**  
   - Uso de IA para sugerir casos de teste com base nos critérios de aceitação, regras de negócio e casos extremos; curadoria por QA/desenvolvedores.[^30][^13]
   - Implementação (manual ou assistida por IA) de suites automatizadas (unitárias, de integração, de contrato, testes de aceitação BDD).

7. **Geração de código (implementação)**  
   - Agentes de IA são instruídos com: constituição, spec de feature, contratos de API, exemplos de testes e guidelines de projeto.[^64][^5]
   - Código é gerado incrementalmente, com feedback contínuo dos testes.
   - Em domínios críticos, combina‑se IA com TDD tradicional: desenvolvedor escreve testes explícitos antes de aceitar código de IA.[^9][^10]

8. **Revisão humana**  
   - Code review focado em aderência ao spec, qualidade de design, segurança, legibilidade e manutenção.[^18][^70]
   - Ajustes no código ou no spec conforme discrepâncias sejam identificadas.

9. **Validação contra critérios de aceitação**  
   - Execução dos testes de aceitação e validação de critérios não funcionais (load tests, security testing).[^13][^20]
   - Se falhar, volta‑se para refinamento de spec, testes ou implementação.

10. **Atualização da especificação**  
   - Atualizar o spec para refletir exatamente o que foi entregue (por exemplo, restrições adicionais descobertas na implementação) e registrar ADRs para decisões significativas.[^24][^15]

11. **Manutenção contínua**  
   - A cada mudança, seguir o mesmo ciclo em menor escala: atualizar spec, testes e código, mantendo rastreabilidade e documentação viva.[^50][^15]

### Adaptação do fluxo por contexto

- **Startups e times ágeis**:  
  - Usar SDD de forma leve: specs curtos mas estruturados por feature; focar em requisitos centrais, fluxos principais, critérios de aceitação e NFRs críticos.[^15][^50]
  - Evitar excesso de formalismo, mas manter versionamento e rastreabilidade mínima.

- **Projetos enterprise**:  
  - Adotar níveis maiores de rigor (spec‑as‑source ou spec‑anchored), com integração a processos de governança, arquitetura corporativa e requisitos (ISO/IEC/IEEE 29148).[^6][^68]
  - Forte uso de documentação viva, ADRs e SDD com “constituições” para segurança e compliance.[^53][^26][^20]

- **Produtos regulados (saúde, financeiro, governo)**:  
  - SDD é especialmente valioso, desde que combinado com requisitos formais, rastreabilidade completa e validações independentes.[^53][^52][^20]
  - Requisitos devem ser fundamentados em normas específicas (por exemplo, IEC 62304, regulamentos bancários) e incorporados à constituição de SDD.[^53]

- **Projetos solo ou pequenos side‑projects**:  
  - Aplicar SDD de forma pragmática: specs curtos, mas claros; uso pesado de IA para rascunhar e refinar especificações; foco em contratos de API e critérios de aceitação.

- **Ambientes com agentes de IA orquestrados**:  
  - Utilizar patterns como Spec Kit Agents: fases explícitas (Specify, Plan, Tasks, Implement), hooks de grounding para repositório e validação automatizada.[^5][^64]
  - Specs e constituições tornam‑se insumo obrigatório para todos os agentes.


## 9. Template recomendado de especificação SDD

Abaixo, um template resumido de spec SDD, em formato adequado para uso humano e por IA.

```markdown
# [ID-FEATURE] Título da funcionalidade

## 1. Objetivo e contexto de negócio
- Objetivo:
- Problema de negócio que resolve:
- KPIs de sucesso:

## 2. Usuários e personas
- Persona 1: [nome] – [características / objetivos]
- Persona 2: ...

## 3. Escopo
- Incluído:
  - ...
- Fora de escopo:
  - ...

## 4. Requisitos funcionais
- RF-01: [descrição clara e verificável]
- RF-02: ...

## 5. Requisitos não funcionais
- Performance: ...
- Segurança: ...
- Privacidade: ...
- Confiabilidade: ...
- Observabilidade: ...

## 6. Regras de negócio
| ID | Regra | Condição | Resultado |
|----|-------|----------|-----------|
| RN-01 | ... | ... | ... |

## 7. Fluxos principais
### 7.1 Fluxo "Happy Path" – [nome]
1. ...
2. ...

### 7.2 Fluxos alternativos
- FA-01: [condição] → [descrição]

## 8. Tratamento de erros
- Erros de negócio:
- Erros técnicos:

## 9. Permissões e papéis
| Operação | Papel | Permissão |
|----------|-------|-----------|

## 10. Dados de entrada e saída
### 10.1 Estrutura de dados
| Campo | Tipo | Obrigatório | Validação | Observações |
|-------|------|------------|-----------|-------------|

### 10.2 Exemplos
```json
// Exemplo de request
{}
```

## 11. Contratos de API
- Especificação OpenAPI/gRPC anexa em `api/feature-x.yaml`.
- Endpoints:
  - [método] /caminho – descrição

## 12. Dependências
- Sistemas externos:
- Serviços internos:

## 13. Restrições técnicas
- Linguagens / frameworks obrigatórios:
- Restrições de compatibilidade:

## 14. Requisitos de segurança
- Autenticação:
- Autorização:
- Criptografia:
- Auditoria:

## 15. Requisitos de privacidade
- Dados pessoais tratados:
- Base legal:
- Retenção e descarte:

## 16. Requisitos de observabilidade
- Logs necessários:
- Métricas:
- Dashboards:

## 17. Requisitos de performance
- Latência alvo (P95):
- Throughput esperado:

## 18. Critérios de aceitação
- CA-01 (para RF-01): ...
- CA-02: ...

## 19. Exemplos e casos extremos
- Caso normal:
- Caso de fronteira:
- Caso inválido:

## 20. Rastreabilidade
| ID Req | Código (módulo/arquivo) | Testes | ADR relacionada |
|--------|-------------------------|--------|-----------------|

## 21. Decisões arquiteturais
- ADR-001: [título] – link

## 22. Critérios de pronto (Definition of Done)
- [ ] Todos os critérios de aceitação aprovados
- [ ] Cobertura de testes >= X%
- [ ] Requisitos de segurança e privacidade verificados
- [ ] Observabilidade implementada
- [ ] Spec atualizado e revisado
```

Esse template pode ser adaptado por domínio, removendo seções opcionais para features de baixo risco e expandindo partes regulatórias quando necessário.


## 10. Conclusão crítica

### SDD é realmente novo?

Os conceitos fundadores de SDD — especificações como fonte de verdade, contratos executáveis, geração de código a partir de modelos ou specs — existem há décadas em requisitos, DbC e MDD. O que é novo é:[^17][^18][^49][^6]

- A combinação sistemática desses conceitos com LLMs e agentes de IA como principais executores de implementação e teste.[^27][^4][^5]
- A infraestrutura de tooling (Spec Kit, CURRANTE, Spec Kit Agents, constitutions de segurança) que operacionaliza specs como artefatos centrais em workflows multi‑agentes.[^48][^29][^5]
- A formulação explícita de SDD como resposta ao “vibe coding” (prompts soltos, sem contrato), alvo de críticas tanto na indústria quanto na academia.[^28][^27][^5]

Portanto, SDD é melhor entendido como uma recombinação de práticas consolidadas com um novo foco: tornar specs consumíveis por IA, e IA responsável por executar intent codificado nessas specs.

### Em quais contextos SDD mais vale a pena?

- Projetos com uso intenso de IA para geração de código e testes, especialmente quando há múltiplos agentes e repositórios grandes.[^29][^5]
- Sistemas com requisitos de segurança, privacidade ou compliance rigorosos, onde “Constitutional SDD” pode codificar regras não negociáveis.[^21][^20]
- Ambientes enterprise com múltiplos times e longa vida útil, nos quais a rastreabilidade de requisitos, decisões e implementação é crítica.[^26][^43][^6]
- APIs públicas, SDKs e plataformas, onde contratos claros e estáveis são fundamentais e podem ser facilmente transformados em specs machine‑readable.[^56][^6]

### Em quais contextos SDD pode ser exagerado?

- Prototipagem rápida de baixa criticidade (por exemplo, scripts internos, provas de conceito), onde o custo de specs detalhadas supera o valor do artefato e onde mudanças são frequentes e pouco estruturadas.[^53][^49]
- Projetos muito pequenos ou descartáveis, nos quais um spec leve (algumas seções) e bons testes podem ser suficientes, sem overhead de constituições e matrizes de rastreabilidade.

### Como adotar SDD sem criar burocracia?

- Começar pequeno: aplicar SDD a features críticas ou módulos centrais, não ao sistema inteiro de uma vez.[^4][^15]
- Definir níveis de spec: por exemplo, specs minimalistas para features simples e specs completos para domínios críticos, usando o framework de níveis de rigor (spec‑first vs spec‑anchored).[^4]
- Automatizar o máximo possível: geração de docs a partir de código e testes, validações automáticas de compliance com spec em CI/CD, geração de relatórios de rastreabilidade.[^23][^5][^15]
- Usar IA como assistente de documentação, mas sempre com revisão humana e ligação direta com código e testes.[^46][^30]

### Como equilibrar documentação, agilidade e geração de código com IA?

- Tratar specs como código: pequenos incrementos, revisão por pares, versionamento, integração contínua e refatoração constante.[^71][^50]
- Integrar SDD ao ciclo ágil: usar specs como parte da Definition of Ready e Definition of Done, em vez de etapas separadas e pesadas.[^52][^53]
- Focar em documentação que é realmente usada: tudo o que não alimenta diretamente implementação, testes, operações ou IA é candidato a ser reduzido ou eliminado.
- Usar SDD para disciplinar o uso de IA: em vez de proibir IA, fornecer specs claras e guardrails bem definidos para que IA atue como executor confiável de intent, não como autor de requisitos.

Em conclusão, SDD oferece um caminho promissor para tornar desenvolvimento com IA mais previsível, seguro e rastreável, desde que adotado com senso crítico: nem como silver bullet, nem como burocracia adicional, mas como uma evolução pragmática de boas práticas de especificação e documentação para a era dos agentes de IA.


***

## Bibliografia (organizada por tipo de fonte)

### Fontes acadêmicas
- Piskala, D. B. “Spec‑Driven Development: From Code to Contract in the Age of AI Coding Assistants.” arXiv:2602.00180, 2026.[^4]
- Marri, S. R. “Constitutional Spec‑Driven Development: Enforcing Security by Construction in AI‑Assisted Code Generation.” arXiv:2602.02584, 2026.[^20]
- Taghavi, P., Bhavani, S. “Spec Kit Agents: Context‑Grounded Agentic Workflows.” arXiv:2604.05278, 2026.[^5]
- Rosa, G. et al. “Understanding Specification‑Driven Code Generation with LLMs: An Empirical Study Design.” arXiv:2601.03878, 2026.[^30]
- Requirements Traceability: A Systematic Literature Review. ACM/IEEE.[^45]
- Gotel, O., Finkelstein, A. “An analysis of the requirements traceability problem.” IEEE, 1994.[^42]
- ISO/IEC/IEEE 29148 – Systems and software engineering — Life cycle processes — Requirements engineering.[^6]
- Behaviour Driven Development: A Systematic Mapping Study. Journal of Systems and Software, 2023.[^13]
- Guidelines to Prompt Large Language Models for Code Generation.[^46]

### Fontes oficiais/institucionais
- ISO/IEC/IEEE 29148:2018 – Systems and software engineering — Life cycle processes — Requirements engineering.[^6]
- IEEE 830-1998 – Recommended Practice for Software Requirements Specifications.[^65]
- Google Cloud Architecture Framework – Overview and principles.[^68]
- Google Testing Blog – “The 10 Minute Test Plan” e “The Inquiry Method for Test Planning”.[^74][^73]
- Google Engineering Practices Documentation (eng‑practices).[^70]
- Software Engineering at Google – Capítulo “Documentation Is Like Code”.[^71]
- Anthropic – “Effective Prompt Engineering – 6 Techniques”.[^60]
- Anthropic – Prompt Engineering Interactive Tutorial.[^57]
- Anthropic – Structured Outputs / JSON results docs.[^55][^76]
- OpenAI – “Best practices for prompt engineering with the OpenAI API”.[^61]
- OpenAI – Prompt engineering guide.[^62]
- OpenAI – Structured Outputs / JSON mode documentation.[^79][^66][^56]
- OpenAI – Function calling docs.[^80][^81][^82]
- OpenAI – Agents API and Agents SDK documentation.[^83][^84][^64]

### Documentação técnica e guias de empresas
- DeepLearning.AI – Curso “Spec‑Driven Development with Coding Agents” e repositório associado.[^91][^3][^29]
- IBM – Artigos sobre Spec‑Driven Development com IBM Bob e IA‑assisted engineering.[^35][^92][^38]
- Thoughtworks – Materiais sobre spec‑driven development como prática de engenharia em 2025.[^93][^36]
- Google Gemini – documentação de prompt design strategies e best practices.[^59][^58]

### Normas e padrões relevantes
- ISO/IEC/IEEE 29148:2018 – Requirements engineering.[^6]
- Padrões de documentação “docs‑as‑code” e living documentation, conforme livros “Living Documentation” e “Specification by Example”.[^14][^15]
- Práticas de ADR (Architecture Decision Records) conforme descritas por Nygard e Fowler.[^25][^26][^24]

---

## References

1. [Spec-Driven Development: o que é e como usar com IA - SoftDesign](https://www.softdesign.com.br/blog/spec-driven-development/) - Em resumo, Spec-Driven Development (SDD) é uma abordagem em que especificações estruturadas (specs) ...

2. [Spec Driven Development: When Architecture Becomes Executable](https://www.infoq.com/articles/spec-driven-development/) - Spec-Driven Development inverts traditional architecture by making specifications executable and aut...

3. [Spec-Driven Development with Coding Agents - DeepLearning.AI](https://www.deeplearning.ai/courses/spec-driven-development-with-coding-agents) - Spec-driven development is the disciplined alternative: write a clear markdown spec defining what to...

4. [Spec-Driven Development:From Code to Contract in the Age of AI ...](https://arxiv.org/abs/2602.00180) - Spec-driven development (SDD) inverts the traditional workflow by treating specifications as the sou...

5. [[2604.05278] Spec Kit Agents: Context-Grounded Agentic Workflows](https://arxiv.org/abs/2604.05278) - Title:Spec Kit Agents: Context-Grounded Agentic Workflows ; Subjects: Software Engineering (cs.SE); ...

6. [ISO/IEC/IEEE 29148:2018 - Systems and software engineering](https://www.iso.org/standard/72089.html) - ISO/IEC/IEEE 29148:2018 Systems and software engineering — Life cycle processes — Requirements engin...

7. [Software requirements specification - Wikipedia](https://en.wikipedia.org/wiki/Software_requirements_specification) - The ISO/IEC/IEEE 29148 standard "Systems and software engineering —Life cycle processes — Requiremen...

8. [A Dissection of the Test-Driven Development Process: Does It Really Matter to Test-First or to Test-Last?](https://ieeexplore.ieee.org/document/7592412/)

9. [What Do We (Really) Know about Test-Driven Development? | IEEE Software](https://dl.acm.org/doi/10.1109/MS.2018.2801554) - Test-driven development (TDD) involves more than just testing before coding. This article examines h...

10. [Driving Software Quality: How Test-Driven Development Impacts Software Quality](https://ieeexplore.ieee.org/document/4012627/)

11. [Spec + TDD: The Combination That Actually Produces Shippable AI ...](https://www.augmentcode.com/guides/spec-tdd-shippable-ai-generated-code) - Learn how Spec + TDD produces shippable AI-generated code: the five-phase workflow, the four failure...

12. [Behavior Driven Development: A Systematic Literature Review](https://ieeexplore.ieee.org/document/10210040) - Behavior Driven Development (BDD) is a widely adopted agile methodology for software development tha...

13. [Behaviour driven development: : A systematic mapping study: Journal of Systems and Software: Vol 203, No C](https://dl.acm.org/doi/abs/10.1016/j.jss.2023.111749) - Abstract Context: Behaviour Driven Development (BDD) uses scenarios written in semi-structured natur...

14. [Specification by Example](https://www.oreilly.com/library/view/specification-by-example/9781617290084/kindle_split_020.html) - Chapter 11. Evolving a documentation system In chapter 3, I introduced the concept of living documen...

15. [Living Documentation: Continuous Knowledge Sharing by Design ...](https://www.oreilly.com/library/view/living-documentation-continuous/9780134689418/) - Use an Approach Inspired by Domain-Driven Design to Build Documentation That Evolves to Maximize Val...

16. [Domain-Driven Design: Tactical Patterns](https://www.fratepietro.com/2020/domain-driven-design-tactical/) - Learn DDD tactical patterns: entities, value objects, aggregates, domain services, repositories, and...

17. [arXiv:cs/0703012v1  [cs.SE]  2 Mar 2007](https://arxiv.org/pdf/cs/0703012.pdf)

18. [Design by Contract](https://www.eiffel.com/values/design-by-contract/) - Design by Contract™ is a method for building software in which the expected behavior of every compon...

19. [Design by contract](https://en.wikipedia.org/wiki/Design_by_contract) - Design by contract (DbC), also known as contract programming, programming by contract and design-by-...

20. [Enforcing Security by Construction in AI-Assisted Code Generation](https://arxiv.org/abs/2602.02584) - We present Constitutional Spec-Driven Development, a methodology that embeds non-negotiable security...

21. [Spec-Driven Security: constitution.md Workflow Guide - Mad Devs](https://maddevs.io/writeups/practical-security-layer-for-spec-driven-development/) - [From constitution.md to Compliance: A Practical Security Layer for SDD] · Constitutional spec-drive...

22. [Writing Great Specifications](https://www.oreilly.com/library/view/writing-great-specifications/9781617294105/OEBPS/Text/kindle_split_016.html) - Chapter 7. Living documentation This chapter covers Understanding living documentation Helping new t...

23. [serenity-bdd.github.io/docs/reporting/living-documentation.md at master · serenity-bdd/serenity-bdd.github.io](https://github.com/serenity-bdd/serenity-bdd.github.io/blob/master/docs/reporting/living-documentation.md) - Serenity BDD Documentation. Contribute to serenity-bdd/serenity-bdd.github.io development by creatin...

24. [Architecture Decision Record - Martin Fowler](https://martinfowler.com/bliki/ArchitectureDecisionRecord.html) - An Architecture Decision Record (ADR) is a short document that captures and explains a single decisi...

25. [Architecture decision record (ADR) - GitHub](https://github.com/architecture-decision-record/architecture-decision-record) - An architecture decision record (ADR) is a document that captures an important architectural decisio...

26. [Maintain an architecture decision record (ADR) - Microsoft Learn](https://learn.microsoft.com/en-us/azure/well-architected/architect-role/architecture-decision-record) - Learn about the benefits of creating an architecture decision record in the design process to docume...

27. [Spec-Driven Development: From Code to Contract in the Age of AI ...](https://arxiv.org/html/2602.00180v1) - When an AI coding assistant is asked to add a feature, it must guess what the developer wants from a...

28. [How spec-driven development improves AI coding quality](https://developers.redhat.com/articles/2025/10/22/how-spec-driven-development-improves-ai-coding-quality) - Learn how to implement spec coding, a structured approach to AI-assisted development that combines h...

29. [https-deeplearning-ai/sc-spec-driven-development-files: All ...](https://github.com/https-deeplearning-ai/sc-spec-driven-development-files) - Spec-Driven Development with Agentic Coding Assistants. This repository contains the companion code ...

30. [Understanding Specification-Driven Code Generation with LLMs](https://arxiv.org/abs/2601.03878) - Abstract page for arXiv paper 2601.03878: Understanding Specification-Driven Code Generation with LL...

31. [[PDF] Specification-Driven Design with Eiffel and Agents for Teaching ...](http://www.cse.yorku.ca/~jonathan/publications/2004/Colognet04.pdf)

32. [Design by Contract](https://se.inf.ethz.ch/~meyer/publications/old/dbc_chapter.pdf) - de B Meyer · Citado por 90 — Design by contract. 1.8.1 Causes for exceptions. In the execution of an...

33. [What is Spec-Driven Development? - IBM](https://www.ibm.com/think/topics/spec-driven-development) - Spec-driven development (SDD) is a software methodology in which a detailed specification of impleme...

34. [A Spec-First Approach to AI-Native Engineering - Microsoft Developer](https://developer.microsoft.com/blog/spec-driven-development-ai-native-engineering) - AI has made software delivery faster, but speed alone does not guarantee better outcomes. As teams a...

35. [Spec-driven Development with IBM Bob](https://heidloff.net/article/spec-driven-development-ibm-bob/) - This post demonstrates how Spec-driven Development addresses these challenges and how it can be util...

36. [Spec-driven development - Thoughtworks](https://x.com/thoughtworks/status/1997924176833794396)

37. [Getting started with Spec Driven Development in ... - Google Codelabs](https://codelabs.developers.google.com/codelabs/getting-started-with-spec-driven-development-in-antigravity?hl=en&authuser=1) - In this hands-on lab you will learn how to use Antigravity to create applications using Google Antig...

38. [IBM/iac-spec-kit: AI-assisted workflows for translating ...](https://github.com/IBM/iac-spec-kit) - For the original Spec-Driven Development methodology, see the GitHub Spec Kit documentation. Generat...

39. [ISO/IEC/IEEE 29148:2011 - IEC Webstore](https://webstore.iec.ch/en/publication/11975) - ISO/IEC/IEEE 29148:2011 contains provisions for the processes and products related to the engineerin...

40. [RT_PAP.PDF](http://csis.pace.edu/~ogotel/research/GOTEL93%20An%20Analysis%20of%20the%20Requirements%20Traceability%20Problem.pdf)

41. [[PDF] An initial model of requirements traceability : an empirical study](https://upload.wikimedia.org/wikipedia/commons/8/8f/An_initial_model_of_requirements_traceability_-_an_empirical_study_(IA_initialmodelofre00rame).pdf)

42. [An analysis of the requirements traceability problem - IEEE Xplore](https://ieeexplore.ieee.org/iel2/948/7229/00292398.pdf) - Requirements traceability (RT) is recognised as a concern in an increasing number of standards and g...

43. [[PDF] A systematic literature review of pre-requirements specification ...](https://dirkriehle.com/wp-content/uploads/2024/01/s00766-023-00412-z.pdf)

44. [How Requirements Quality Makes (or Breaks) Traceability Link ...](https://arxiv.org/html/2606.11834v1)

45. [Requirements Traceability: A Systematic Literature Review](https://dl.acm.org/doi/10.1145/3672608.3707952) - An empirical study on the state-of-the-art methods for requirement-to-code traceability link recover...

46. [Guidelines to Prompt Large Language Models for Code Generation](https://huggingface.co/papers/2601.13118) - Understanding Specification-Driven Code Generation with LLMs: An Empirical Study Design (2026); Unde...

47. [DeepLearning.AI Courses](https://www.deeplearning.ai/courses) - Spec-Driven Development with Coding Agents. Short Course. JetBrains ... Short Course. DeepLearning.A...

48. [Spec Kit Agents: Context-Grounded Agentic Workflows - arXiv](https://arxiv.org/html/2604.05278v1)

49. [[PDF] The Requirements Engineering Handbook - Libros UCLV](https://libros.uclv.edu.cu/books/download?id=727)

50. [Documentation as Code: How Agile Development ... - SCAND](https://scand.com/company/blog/documentation-as-code-agile-one-source-of-truth/) - Documentation as Code is a practice of writing and keeping documentation in the same way developers ...

51. [[PDF] Evaluating Code Model Robustness to Ambiguous, Contradictory ...](https://homepages.dcc.ufmg.br/~figueiredo/disciplinas/papers/icse2026larbi.pdf) - Empirical research on requirements quality: a systematic mapping study. ... Deceptprompt: Exploiting...

52. [[PDF] IREB Handbook Foundation Level - GASQ](https://www.gasq.org/files/content/gasq/downloads/certification/IREB/IREB%20FL/cpre_foundationlevel_handbook_en_v1.0.pdf)

53. [What is living documentation for medical device software?](https://bluefruit.co.uk/processes/what-living-documentation-medical-device-software/) - How can we make documentation generation for medical device software a straightforward and efficient...

54. [Spec-Driven Development with Coding Agents - DeepLearning.AI](https://learn.deeplearning.ai/courses/spec-driven-development-with-coding-agents/lesson/bvlyk483/set-up-your-environment)

55. [Structured outputs - Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) - Get validated JSON results from agent workflows

56. [Structured Outputs](https://platform.openai.com/docs/guides/structured-outputs?api-mode=responses) - Understand how to ensure model responses follow specific JSON Schema you define.

57. [GitHub - anthropics/prompt-eng-interactive-tutorial: Anthropic's Interactive Prompt Engineering Tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) - Anthropic's Interactive Prompt Engineering Tutorial - anthropics/prompt-eng-interactive-tutorial

58. [Prompt design strategies | Gemini API | Google AI for Developers](https://ai.google.dev/gemini-api/docs/prompting-strategies) - - Unplug your Google Wifi router and wait for 30 seconds. - Plug your Google Wifi router back in and...

59. [Prompt design for Gemini Nano | ML Kit - Google for Developers](https://developers.google.com/ml-kit/genai/prompt/android/prompt-design) - This page describes best practices for formatting prompts for Gemini Nano. For more general prompt e...

60. [[PDF] 6 Techniques for Effective Prompt Engineering - Anthropic](https://www-cdn.anthropic.com/62df988c101af71291b06843b63d39bbd600bed8.pdf)

61. [Best practices for prompt engineering with the OpenAI API](https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api) - How to give clear and effective instructions to OpenAI models

62. [Prompt engineering | OpenAI API](https://platform.openai.com/docs/guides/prompt-engineering) - Learn strategies and tactics for better results using large language models in the OpenAI API.

63. [Specification-Driven Code Generation with LLMs - CodeNote.net](https://codenote.net/en/posts/specification-driven-code-generation-llm-empirical-study/) - This article introduces the paper “Understanding Specification-Driven Code Generation with LLMs: An ...

64. [OpenAI Agents API documentation](https://platform.openai.com/docs/guides/agents) - Learn how to build agents with the OpenAI API.

65. [IEEE 830-1998 - IEEE SA](https://standards.ieee.org/standard/830-1998.html) - IEEE Recommended Practice for Software Requirements Specifications ; Status: Superseded Standard ; S...

66. [Introducing Structured Outputs in the API](https://openai.com/index/introducing-structured-outputs-in-the-api/) - Today we're introducing Structured Outputs in the API, a new feature designed to ensure model-genera...

67. [OpenAI Harmony Response Format](https://developers.openai.com/cookbook/articles/openai-harmony) - The gpt-oss models were trained on the harmony response format for defining conversation structures,...

68. [Learn about Google Cloud with the new Architecture Framework guide](https://cloud.google.com/blog/products/cloud-migration/new-google-cloud-architecture-framework-guide) - You can learn more about the Google Cloud Architecture Framework here and contact us for additional ...

69. [[PDF] Software Engineering at Google - GitHub](https://raw.githubusercontent.com/abseil/abseil.github.io/cd13b21daa6ec74155548241241693198c1b1264/resources/swe_at_google.2.pdf) - ... Software Engineering at Google by Titus Winters, Tom Manshreck, and Hyrum ... Design Docs. 195. ...

70. [Google Engineering Practices Documentation | eng-practices](https://google.github.io/eng-practices/) - Google Engineering Practices Documentation. Google has many generalized engineering practices that c...

71. [Documentation Is Like Code - Software Engineering at Google](https://abseil.io/resources/swe-book/html/ch10.html)

72. [GitHub - comsysto/livingdoc: Living Documentation Lab](https://github.com/comsysto/livingdoc) - Living Documentation Lab. Contribute to comsysto/livingdoc development by creating an account on Git...

73. [The 10 Minute Test Plan](https://testing.googleblog.com/2011/09/10-minute-test-plan.html) - On this topic, I will be posting a Google Testing Blog article on a very different approach to autho...

74. [2016 - Google Testing Blog](https://testing.googleblog.com/2016/)

75. [Google Gemini: Prompt engineering techniques for more accurate ...](https://www.datastudios.org/post/google-gemini-prompt-engineering-techniques-for-more-accurate-responses) - Gemini’s performance depends heavily on how instructions are framed and delivered. By applying struc...

76. [Strukturierte Ausgaben - Claude Platform Docs](https://platform.claude.com/docs/de/build-with-claude/structured-outputs) - Erhalte validierte JSON-Ergebnisse aus Agenten-Workflows

77. [Use Xml Tags To Structure...](https://www.aiwithgrant.com/guides/anthropic-prompt-engineering-overview) - Everything from Anthropic's prompt engineering docs in one progressive guide. Clear instructions, mu...

78. [Prompt engineering best practices for ChatGPT - OpenAI Help Center](https://help.openai.com/en/articles/10032626-prompt-engineering-best-practices-for-chatgpt) - Learn how to craft effective prompts to get the best out of ChatGPT

79. [Structured Outputs - OpenAI API](https://platform.openai.com/docs/guides/json-mode) - Understand how to ensure model responses follow specific JSON Schema with OpenAI's Structured Output...

80. [Handling function calls](https://developers.openai.com/api/docs/guides/function-calling) - Learn how function calling enables large language models to connect to external data and systems.

81. [Function calling - OpenAI API](https://platform.openai.com/docs/guides/function-calling/function-calling) - Learn how function calling enables large language models (LLMs) to fetch data, take action, and conn...

82. [Function Calling in the OpenAI API](https://help.openai.com/en/articles/8555517-function-calling-in-the-openai-api) - What is function calling and how does it work in the OpenAI API?

83. [Quickstart | OpenAI API](https://developers.openai.com/api/docs/guides/agents/quickstart) - Build your first agent with the OpenAI Agents SDK, add tools and handoffs, and understand where to g...

84. [Agent definitions](https://developers.openai.com/api/docs/guides/agents/define-agents) - Learn how to define an agent's instructions, model, tools, and local context in the OpenAI Agents SD...

85. [System and Developer Roles in messages and Instructions ...](https://community.openai.com/t/system-and-developer-roles-in-messages-and-instructions-in-responses-create/1370516/12) - GPT-5.2 uses a “developer” message. Sending “system” and having it work as “developer” is simply a c...

86. [What goes in the system vs developer role - API](https://community.openai.com/t/what-goes-in-the-system-vs-developer-role/1347594) - Both the system and developer are specified in the api reference for the role type and the docs ment...

87. [Evals | OpenAI API Reference](https://developers.openai.com/api/reference/resources/evals/)

88. [Prompt engineering for business performance - Anthropic](https://www.anthropic.com/news/prompt-engineering-for-business-performance) - Anthropic is an AI safety and research company that's working to build reliable, interpretable, and ...

89. [[PDF] Tracing Requirements and Source Code During Software ...](http://archiv.ub.uni-heidelberg.de/volltextserver/15996/1/Dissertation-Alexander-Delater.pdf)

90. [[PDF] Preserving Requirements-Driven Development in the Age of AI](https://iscap.us/proceedings/2025/pdf/6416.pdf)

91. [Course Summary / Additional walkthrough material](https://community.deeplearning.ai/t/course-summary-additional-walkthrough-material/891884) - Hey everyone, Regarding the short course Spec-Driven Development with Coding Agents: We’ve heard tha...

92. [From Vibecoding to Engineering with AI: Spec-Driven ...](https://community.ibm.com/community/user/events/event-description?CalendarEventKey=0cf0dbaa-4408-482d-8ad2-019e3cf1591a&CommunityKey=f1c2cf2b-28bf-4b68-8570-b239473dcbbc&Home=%2Fcommunity%2Fuser%2Fcommunity) - IBM Community logo. This will take you to the ... From Vibecoding to Engineering with AI: Spec-Drive...

93. [Spec-driven development - Thoughtworks - Medium](https://thoughtworks.medium.com/spec-driven-development-d85995a81387) - Unpacking one of 2025’s key new AI-assisted engineering practices

