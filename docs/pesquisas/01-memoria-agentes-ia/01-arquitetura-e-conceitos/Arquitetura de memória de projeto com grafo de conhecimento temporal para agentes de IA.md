# Arquitetura de memória de projeto com grafo de conhecimento temporal para agentes de IA

## Visão geral

Memória de projeto para agentes de IA em engenharia de software precisa capturar não só **o que** existe (arquivos, módulos, funções), mas **como e quando** tudo muda: decisões arquiteturais, bugs, PRs, commits, issues, dependências, integrações externas e owners, com histórico e revogações. Grafos de conhecimento temporais (temporal knowledge graphs, TKGs) são adequados porque combinam estrutura explícita (entidades e relações) com metadados temporais e proveniência.[^1][^2][^3][^4]

Esta arquitetura se inspira em trabalhos recentes como Zep/Graphiti (memória temporal para agentes), Neo4j/GraphRAG e iniciativas de GraphRAG em bancos como FalkorDB, que combinam grafo + vetores + pesquisa textual.[^5][^4][^6]

## Quando grafo é melhor que vector search

Vetores respondem à pergunta “o que é semanticamente parecido com isso?”, enquanto um grafo responde “o que é estruturalmente relacionado com isso, e como, ao longo do tempo?”. Em uma memória de projeto, há vários tipos de consultas onde o grafo domina:[^7]

- **Raciocínio multi-hop e estrutural**: “Quais serviços são impactados se eu mudar este módulo?”, “Que bugs foram introduzidos por este commit e em quais áreas frágeis?”. Isso é naturalmente modelado como travessia de caminhos no grafo (módulo → dependências → serviços → issues → bugs).[^8][^7]
- **Consultas agregadas e de impacto**: “Quantos bugs de regressão surgiram 30 dias após a adoção da decisão arquitetural X?”, “Qual é o módulo com maior densidade de bugs críticos por linha de código?”. Vetores não codificam hierarquias nem cardinalidades; o grafo permite agregações explícitas.[^7]
- **Desambiguação e tipo de entidade**: diferenciar arquivo `billing.py` em `service-billing` vs `service-reporting`, ou PRs com o mesmo título em repositórios distintos, depende de tipos e relações explícitas (Projeto, Repositório, Arquivo, PR).[^8][^7]
- **Relações causais e temporais**: “Este bug apareceu depois de qual commit?”, “qual decisão substituiu a anterior?”, “qual estado o grafo tinha em 2024‑10‑01?”. Isso exige validade temporal e bitemporalidade, não só semelhança semântica.[^1][^5]
- **Auditabilidade e explicabilidade**: você quer saber exatamente **por que** o agente sugeriu um rollback ou uma refatoração, com o caminho de evidência (commits → PRs → discussão → decisão → incidentes).[^4][^9]

Benchmarks recentes de GraphRAG mostram que, em tarefas de raciocínio multi-documento e agregação sobre relacionamentos, grafos recuperam contexto relevante muito mais frequentemente que RAG apenas com vetores.[^9][^7]

## Quando vector search é melhor que grafo

Vector search continua sendo o melhor primitivo para várias tarefas da memória de projeto:

- **Texto não estruturado e heterogêneo**: descrições de PRs, issues, discussões em Slack, ADRs, docs de arquitetura, logs de incidentes — tudo pode ser indexado em um único espaço de embedding sem schema.[^10]
- **Onboarding e perguntas abertas**: “Me explica o módulo de billing”, “resuma as mudanças no último trimestre”. Essas consultas são respondidas bem com recuperação de trechos textuais + LLM, sem necessidade de percorrer caminhos complexos.[^10]
- **Cold start**: é possível começar rápido só com índice vetorial dos repositórios, enquanto o grafo é preenchido incrementalmente.
- **Fuzzy match e vocabulário variado**: embeddings lidam melhor com sinônimos (“payment”, “billing”, “cobrança”) do que um grafo com labels estritos, especialmente em linguagens diferentes.[^10]

Para cargas de trabalho que são majoritariamente Q&A sobre documentação estática, com pouca necessidade de raciocínio sobre dependências e tempo, um vector store bem configurado é suficiente.[^11]

## Como combinar grafo e vector search

Arquiteturas modernas de memória para agentes (Graphiti/Zep, FalkorDB GraphRAG, Neo4j GraphRAG) convergem para um modelo híbrido:[^6][^5][^4]

1. **Ingestão**
   - Extrair entidades/relacionamentos de código, PRs, issues, etc. com LLMs e parsers estáticos.
   - Criar nós/arestas no grafo com metadados temporais e proveniência.
   - Indexar texto bruto (corpos de PR, descrições de issues, ADRs) em um índice vetorial.

2. **Consulta (retrieval)**
   - Dado um objetivo de agente, primeiro usar vetores para achar nós/documentos relevantes (ex.: PRs e ADRs semanticamente ligados à query).[^3][^10]
   - A partir desses nós, executar travessias de grafo para:
     - Expandir dependências (upstream/downstream),
     - Navegar relações temporais/causais,
     - Computar agregações (contagem de bugs, regressões, etc.).[^4][^6]
   - Fundir resultados de vetor, BM25 e grafo em um ranking único, como faz Graphiti.[^5][^3]

3. **Contexto para LLM**
   - O LLM recebe: trechos de texto (via vetor) + subgrafo projetado (caminhos explicativos e metadados) para fundamentar a resposta.

Graphiti implementa exatamente esse padrão: grafo temporal sobre Neo4j/FalkorDB/Neptune, com índice vetorial e BM25 integrados, e API de busca híbrida com ranqueamento único.[^3][^5][^4]

## Modelo de dados: entidades

Uma ontologia prática para memória de projeto de software, em cima de um grafo de propriedade (Neo4j/FalkorDB):

### Nós principais

- `Project`
- `Repo`
- `File`
- `Module` (agrupa arquivos ou namespaces)
- `Function` / `Class`
- `Commit`
- `PullRequest` (PR)
- `Issue`
- `Bug` (instância de problema técnico, muitas vezes derivada de uma issue)
- `Decision` (ADR, RFC, comentário de revisão que virou decisão)
- `TestPattern` (estratégia de teste, suíte, convenção)
- `FragileArea` (áreas conhecidas como frágeis)
- `Dependency` (biblioteca ou serviço interno)
- `ExternalService` (API externa, SaaS)
- `Command` (CLI, script, comando de build/deploy)
- `Owner` (pessoa ou time)
- `Memory` (fato abstrato, para memórias que não se encaixam nas entidades acima)

### Propriedades comuns

- `id` (chave natural ou UUID)
- `created_at`, `updated_at` (tempo de ingestão)
- `valid_from`, `valid_to` (intervalo de validade no mundo — bitemporalidade pode incluir `recorded_from`, `recorded_to` se necessário).[^1][^5]
- `source` (arquivo, URL, origem, hash do commit)
- `revoked` (bool) / `superseded_by` (referência a outra entidade)

### Relações principais

- `(Repo)-[:CONTAINS]->(File)`
- `(File)-[:DECLARES]->(Module)`
- `(Module)-[:CONTAINS]->(Function)`
- `(Function)-[:CALLS {first_seen, last_seen}]->(Function)`
- `(Commit)-[:MODIFIES]->(File)`
- `(Commit)-[:INTRODUCES]->(Bug)`
- `(Bug)-[:REPORTED_AS]->(Issue)`
- `(Issue)-[:FIXED_BY]->(PullRequest)`
- `(PullRequest)-[:MERGES]->(Commit)`
- `(Module)-[:DEPENDS_ON {valid_from, valid_to}]->(Dependency)`
- `(ServiceA:Module)-[:CALLS]->(ServiceB:ExternalService)`
- `(Owner)-[:OWNS]->(Module|File|Service)`
- `(Decision)-[:APPLIES_TO]->(Module|Service|ArchitectureArea)`
- `(Decision)-[:SUPERSEDED_BY]->(Decision)`
- `(FragileArea)-[:LOCATED_IN]->(Module|File)`
- `(TestPattern)-[:COVERS]->(Module|Function)`
- `(Command)-[:RUNS_AGAINST]->(Service|Env)`
- `(Memory)-[:ABOUT]->(File|Module|Issue|Decision|Bug|Owner)`

Cada relacionamento também carrega atributos temporais (`valid_from`, `valid_to`, `observed_at`) e de proveniência.[^12][^5]

## Modelagem do tempo e bitemporalidade

Seguindo o modelo temporal de Graphiti/Zep, cada fato pode ter pelo menos dois eixos de tempo:[^5][^1]

- **Tempo de validade (valid time)**: quando o fato era verdadeiro no domínio (ex.: período em que um módulo dependia de uma lib X).
- **Tempo de registro (transaction time / knowledge time)**: quando o sistema aprendeu/registrou o fato e quando descobriu que não é mais verdadeiro.

Uma implementação simples em grafo de propriedade:

- Para cada relação `:DEPENDS_ON`, propriedades:
  - `valid_from: datetime`
  - `valid_to: datetime|null` (null = ainda válido)
  - `recorded_from: datetime`
  - `recorded_to: datetime|null`

Isso permite queries como:

- “Estado do grafo em 2024‑10‑01” → filtra relações com `valid_from <= t < coalesce(valid_to, +inf)`.
- “O que o agente acreditava em 2024‑10‑01, mesmo que fosse errado?” → usa `recorded_*`.[^1][^5]

Neo4j tem tipos temporais nativos (Date, Time, DateTime, Duration) suportados em propriedades e funções Cypher, o que facilita esse modelo.[^13][^12]

## Atualizando o grafo a partir de commits e PRs

### Pipeline de ingestão

1. **Webhook do Git**
   - Recebe eventos de commit e PR (GitHub, GitLab, Bitbucket).

2. **Normalização**
   - Enriquecer com dados do provider (lista de arquivos modificados, diffs, reviewers, labels, etc.).

3. **Extração de entidades/relacionamentos**
   - Parser estático (Tree-sitter, ctags, AST) para extrair `Function`, `Class`, `Module`, chamadas de função, imports.
   - LLM opcional para extrair decisões arquiteturais, áreas frágeis, padrões de teste a partir de descrição do PR, ADRs e comentários.[^3][^4]

4. **Upsert no grafo**
   - Para cada commit/PR, criar nós `Commit`, `PullRequest`, ligar com `:MODIFIES`, `:MERGES`, `:FIXED_BY`, etc.
   - Atualizar ou criar relações temporais: novas dependências, owners, flags de fragilidade, decisões.
   - Invalidar fatos antigos via atualização de `valid_to` quando detectado conflito (por exemplo, `Decision` com `SUPERSEDED_BY`).[^5]

5. **Indexação vetorial**
   - Gerar embeddings para corpos de PR, descrições de issue, ADRs, comentários, docs.
   - Armazenar no mesmo back‑end (Neo4j, FalkorDB) ou em um vector store externo, mas com `id` apontando para nós/arestas do grafo.[^14][^6]

### Exemplo de pseudo‑código (ingestão de commit)

```python
# evento de webhook (GitHub)
commit = event["commit"]

files = git_api.list_files(commit["sha"])  # arquivos modificados

# 1) criar nó Commit
MERGE (c:Commit {sha: $sha})
  ON CREATE SET c.message = $msg, c.author = $author,
                c.created_at = datetime($ts),
                c.recorded_from = datetime()  # ingest time

# 2) para cada arquivo
for f in files:
  MERGE (file:File {path: f.path, repo: $repo})
  MERGE (c)-[:MODIFIES {recorded_from: datetime()}]->(file)

  # 3) extrair funções e dependências
  funcs, deps = parse_file(f)

  for fn in funcs:
    MERGE (fnNode:Function {full_name: fn.full_name})
    MERGE (file)-[:DECLARES]->(fnNode)

  for dep in deps:
    MERGE (depNode:Dependency {name: dep.name, version: dep.version})
    MERGE (file)-[r:DEPENDS_ON]->(depNode)
    ON CREATE SET r.valid_from = datetime($ts), r.recorded_from = datetime()
    ON MATCH SET r.valid_to = null  # ainda válido
```

Na prática, isso seria gerado em Cypher parâmetrizado via driver Python/Node.

## Detecção de contradições e revogação de memórias

Com bitemporalidade, contradições não são resolvidas apagando fatos, mas marcando‑os como **revogados** ou **substituídos**, preservando histórico.[^1][^5]

### Tipos de contradições comuns

- **Decisões arquiteturais conflitantes**: duas `Decision` diferentes para o mesmo `Module` com políticas incompatíveis.
- **Propriedades exclusivas**: um `Owner` antigo e um novo para o mesmo módulo, onde só um pode estar ativo.
- **Flags de fragilidade**: `FragileArea` marcada como resolvida, mas novos incidentes mostram que ainda é frágil.

### Estratégia

1. **Regra de unicidade lógica** (ex.: “no máximo um owner ativo por módulo”): expressa como constraint de aplicação.
2. Ao inserir novo fato que viola a regra:
   - Encontrar fatos antigos ainda válidos.
   - Encerrar `valid_to` (e `recorded_to`) desses fatos.
   - Criar relação `:SUPERSEDED_BY` ou marcar `revoked = true`.

Pseudo‑código Cypher:

```cypher
// Novo owner para o módulo
MATCH (m:Module {id: $module_id})
MATCH (newOwner:Owner {id: $owner_id})

// encerrar owners antigos
MATCH (oldOwner:Owner)-[r:OWNS]->(m)
WHERE r.valid_to IS NULL
SET r.valid_to = datetime($now), r.recorded_to = datetime($now),
    r.revoked = true

// criar relação nova
MERGE (newOwner)-[r2:OWNS {
  valid_from: datetime($now),
  recorded_from: datetime($now)
}]->(m)
```

Graphiti implementa lógica semelhante: ao inserir novo conhecimento, ele verifica conflitos semânticos e temporais e invalida fatos antigos, mantendo histórico.[^3][^5]

## Recuperação de contexto para uma tarefa

### Fluxo de alto nível para um agente de manutenção

1. **Entender a tarefa**
   - Prompt do usuário: “refatorar o módulo de billing para reduzir acoplamento com o serviço de notas fiscais”.
   - LLM extrai entidades: `Module=billing`, `ExternalService=nota_fiscal`, tipo de objetivo.

2. **Localização do subgrafo relevante**
   - Consulta vetorial para encontrar ADRs, PRs, issues e docs que mencionam billing/nota fiscal.
   - A partir dos nós correspondentes, travessia de grafo:
     - `(Module)-[:DEPENDS_ON]->(Dependency|ExternalService)`
     - `(Module)-[:CONTAINS]->(File|Function)`
     - `(Module)-[:OWNS]-(Owner)`
     - `(Module)<-[:LOCATED_IN]-(FragileArea)`

3. **Filtragem temporal**
   - Considerar apenas fatos válidos neste momento (ou num ponto específico no passado, se for análise de regressão).

4. **Construção do contexto**
   - Selecionar:
     - Lista de dependências externas com `valid_from` recentes.
     - Decisões arquiteturais vigentes.
     - Histórico de bugs críticos em `FragileArea` relacionadas.
     - Padrões de teste aplicáveis.

5. **Geração de resposta**
   - Fornecer ao LLM um pacote contendo:
     - Subgrafo (serializado em JSON ou natural language chain-of-thought amigável),
     - Trechos textuais relevantes (ADRs, PRs),
     - Metadados (owners, datas, severidades).

Graphiti e Zep implementam APIs que fazem grande parte desse fluxo: dado um “assunto” (subject), retornam um contexto híbrido (grafos + texto) em uma única chamada de baixa latência.[^15][^5]

## Evitando overengineering

É fácil cair em um design de grafo mega‑genérico e impraticável. Estratégias para manter o sistema enxuto:

- **Comece pelas queries, não pelo modelo**: liste as 10–20 perguntas mais valiosas (impacto no suporte, incidentes, produtividade de devs). Modele só o necessário para respondê‑las.[^11][^7]
- **Adote um núcleo de ontologia pequeno**: Project, Repo, File, Module, Function, Commit, PR, Issue, Decision, Bug, Owner, Dependency, ExternalService, FragileArea, TestPattern. Adicione entidades conforme surgirem novas queries reais.
- **Desnormalize onde fizer sentido**: não transforme cada detalhe em nó; mantenha metadados simples em propriedades.
- **Automatize a ingestão**: se uma relação não pode ser inferida automaticamente de código/PRs/issues, pense bem antes de exigir anotação manual.
- **Mantenha vetores como escape hatch**: para queries que não exigem raciocínio estrutural, use vector RAG padrão e não force passagem pelo grafo.
- **Itere a partir de pilotos**: comece com um único repositório crítico (ex.: serviço de billing) e expanda conforme o ganho se comprovar.

Experiências de equipes que migraram de vector‑only para GraphRAG relatam que a maior parte do valor vem de poucos tipos de relação bem modelados e de uma boa estratégia de atualização, não de ontologias perfeitas.[^16][^9]

## Exemplo de queries (Cypher)

### 1. Dependências impactadas por um módulo

```cypher
MATCH (m:Module {name: "billing"})-[:DEPENDS_ON]->(dep)
WHERE coalesce(r.valid_to, datetime("9999-12-31")) > datetime()
RETURN dep
```

### 2. Bugs de regressão após uma decisão arquitetural

```cypher
MATCH (d:Decision {id: $decision_id})-[:APPLIES_TO]->(m:Module)
MATCH (m)<-[:LOCATED_IN]-(fa:FragileArea)
MATCH (fa)<-[:INTRODUCES]-(c:Commit)-[:INTRODUCES]->(b:Bug)
WHERE c.created_at > d.valid_from
RETURN b, c, m
ORDER BY c.created_at DESC
```

### 3. Estado do grafo em um ponto no tempo

```cypher
WITH datetime($as_of) AS t
MATCH (m:Module)-[r:DEPENDS_ON]->(dep)
WHERE r.valid_from <= t AND (r.valid_to IS NULL OR r.valid_to > t)
RETURN m, dep, r
```

### 4. Áreas frágeis de um serviço externo

```cypher
MATCH (s:ExternalService {name: $service})<-[:CALLS]-(m:Module)
MATCH (fa:FragileArea)-[:LOCATED_IN]->(m)
RETURN m, fa
ORDER BY fa.severity DESC
```

### 5. Contexto para tarefa de refatoração

```cypher
// encontrar módulo
MATCH (m:Module {name: $module})

// dependências atuais
OPTIONAL MATCH (m)-[depRel:DEPENDS_ON]->(dep)
WHERE depRel.valid_to IS NULL

// bugs recentes
OPTIONAL MATCH (m)<-[:LOCATED_IN]-(fa:FragileArea)
OPTIONAL MATCH (fa)<-[:INTRODUCES]-(c:Commit)-[:INTRODUCES]->(b:Bug)
WHERE c.created_at > datetime() - duration("P90D")

RETURN m, collect(DISTINCT dep) AS deps,
       collect(DISTINCT b) AS recent_bugs,
       collect(DISTINCT fa) AS fragile_areas
```

## Pseudo‑código de arquitetura de serviço

### Componentes

- `ingestor-git`: escuta webhooks, normaliza eventos, chama `analyzer`.
- `analyzer`: extrai entidades/relacionamentos (parser + LLM), gera comandos Cypher.
- `graph-store`: Neo4j ou FalkorDB (via Bolt/Cypher).
- `vector-store`: índice vetorial integrado (FalkorDB, Graphiti) ou externo.
- `memory-service`: API de alto nível para agentes (ex.: `GET /context?task=...`).

Pseudo‑código de `GET /context`:

```python
def get_context(task: str) -> Context:
    # 1) extrair intenções e entidades
    entities = llm_extract_entities(task)

    # 2) busca vetorial para documentos relevantes
    vec_hits = vector_search(task, k=20)

    # 3) coletar IDs de nós associados
    node_ids = ids_from_vec_hits(vec_hits)

    # 4) query de subgrafo
    subgraph = neo4j_query(
        """
        MATCH (n)
        WHERE id(n) IN $node_ids
        OPTIONAL MATCH p=(n)-[*1..3]-(m)
        WHERE all(r IN relationships(p) WHERE
              r.valid_from <= datetime() AND
              (r.valid_to IS NULL OR r.valid_to > datetime()))
        RETURN nodes(p) AS nodes, relationships(p) AS rels
        """,
        {"node_ids": node_ids},
    )

    # 5) montar contexto para LLM
    return build_context_payload(task, vec_hits, subgraph)
```

Graphiti fornece uma API deste tipo out‑of‑the‑box, com fusão de ranking entre grafo, vetores e BM25.[^5][^3]

## Comparação de ferramentas

### Tabela resumida

| Ferramenta      | Tipo/base               | Suporte temporal | Vetores integrados | Uso típico em agentes | Observações |
|-----------------|-------------------------|------------------|--------------------|-----------------------|------------|
| Graphiti (Zep)  | Framework de grafo temporal sobre Neo4j/FalkorDB/Neptune[^5][^4] | Bitemporal em arestas, invalidação automática de fatos[^5][^1] | Sim, híbrido (vetor + BM25 + grafo)[^5][^3] | Memória de longo prazo, contexto dinâmico, multi‑grafo por sujeito[^3][^15] | Focado em agentes; open source (framework) + SaaS (Zep). |
| Zep             | Plataforma de memória com Graphiti por baixo[^2][^15] | Sim, temporal e cross‑session[^2] | Sim, híbrido | Memória enterprise multi‑aplicação | Oferece benchmarks DMR/LongMemEval, SOC2, etc. |
| Neo4j           | Banco de grafo de propriedade (LPG)[^12] | Temporal via propriedades e tipos DateTime/Duration, não nativo como TKG mas flexível[^12][^13] | Via plugins/integrações; eco‑sistema de GraphRAG[^8] | Knowledge graph, GraphRAG, consultas complexas | Ecossistema maduro, drivers, APOC, Graph Data Science. |
| FalkorDB        | Grafo in‑memory sobre Redis, GraphBLAS[^17][^18] | Temporal via modelagem de propriedades | Sim, vetor index nativo[^14][^19] | GraphRAG de baixa latência, agentic memory | Muito rápido (sparse matrices), Cypher, multi‑tenant. |
| RDF stores (Jena, Blazegraph, etc.) | RDF/OWL, triplestore | Suporte a tempo via OWL‑Time e ontologias temporais; consultas SPARQL com filtros temporais[^20][^21][^22] | Vetores normalmente externos | Cenários fortemente semânticos, interoperabilidade, web semântica | Mais rígidos, bom para ontologias ricas e inferência, menos focados em baixa latência para agentes. |

### Considerações de escolha

- **Se o foco é agente de engenharia de software com alta taxa de mudanças e baixa latência**: Graphiti sobre Neo4j ou FalkorDB é uma combinação forte; FalkorDB tende a melhor latência bruta, Neo4j traz tooling e ecosistema.[^23][^6][^4]
- **Se a organização já usa Neo4j**: Graphiti “encaixa” diretamente; GraphRAG da própria Neo4j pode ser base para queries avançadas.[^8][^4]
- **Se há requisitos fortes de semântica/ontologias formais (RDF/OWL)**: RDF store com OWL‑Time e ontologias temporais faz sentido, com um layer de adaptação para agentes.[^20][^22]

## Plano de implementação em fases

### Fase 0 – Descoberta

- Levantar queries críticas (suporte, SRE, devs): impacto, regressões, dependências, ownership.
- Escolher stack inicial: por exemplo, Graphiti + Neo4j (ou FalkorDB se latência for prioridade extrema).

### Fase 1 – MVP de grafo + vetor

- Implementar ingestão de:
  - Repos (estrutura de arquivos/módulos).
  - Commits e PRs.
  - Issues e bugs.
- Construir modelo de dados mínimo com: Project, Repo, File, Module, Function, Commit, PR, Issue, Bug, Dependency, Owner.
- Indexar texto de PRs/issues/ADRs num vector store.
- Criar 5–10 queries Cypher que respondam às perguntas de maior valor.

### Fase 2 – Tempo e decisões

- Adicionar propriedades temporais (`valid_from`, `valid_to`, `recorded_from`, `recorded_to`) em relações chave (`DEPENDS_ON`, `OWNS`, `INTRODUCES`, `FIXED_BY`, `APPLIES_TO`).
- Ingerir ADRs/decisões arquiteturais como nós `Decision` conectados a módulos/serviços.
- Implementar lógica de revogação/substituição de decisões e owners.

### Fase 3 – Integração com agentes

- Construir serviço `memory-service` com endpoint `/context` que implementa o pipeline híbrido (vetor + grafo).
- Integrar com agentes (Claude/Gemini/Vertex AI, etc.) via MCP ou APIs equivalentes.
- Usar Graphiti se desejar evitar implementação manual da fusão de ranking e da lógica temporal.

### Fase 4 – Otimização e expansão

- Medir latência, precisão de contexto e impacto em tarefas reais (tempo de resolução de bugs, quantidade de regressões).
- Estender o modelo com `FragileArea`, `TestPattern`, `ExternalService`, `Command` etc. conforme surgirem novas necessidades.
- Adicionar métricas e dashboards (ex.: “hotspots” de fragilidade e de dívida técnica).

Essa abordagem leva a uma memória de projeto realmente útil para agentes, que consegue responder perguntas complexas sobre o sistema ao longo do tempo, sem sacrificar a velocidade e sem cair em overengineering.

---

## References

1. [What Is a Temporal Knowledge Graph? Definition - Zep](https://www.getzep.com/ai-agents/temporal-knowledge-graph/) - A temporal knowledge graph records when each fact was true and where it came from. Standard vs tempo...

2. [Zep: A Temporal Knowledge Graph Architecture for Agent Memory](https://arxiv.org/abs/2501.13956) - We introduce Zep, a novel memory layer service for AI agents that outperforms the current state-of-t...

3. [Graphiti: Temporal Knowledge Graphs for Agentic Apps - Zep](https://blog.getzep.com/graphiti-knowledge-graphs-for-agents/) - Graphiti builds dynamic, temporally aware knowledge graphs that represent complex, evolving relation...

4. [Graphiti: Knowledge graph memory for an agentic world](https://neo4j.com/blog/developer/graphiti-knowledge-graph-memory/) - Zep AI's Graphiti framework introduces a flexible, real-time memory layer built on temporally aware ...

5. [Graphiti — Zep](https://www.getzep.com/platform/graphiti/) - Graphiti turns conversations, business data, and documents into temporal Context Graphs — entities, ...

6. [GitHub - FalkorDB/FalkorDB: A super fast Graph Database uses GraphBLAS under the hood for its sparse adjacency matrix graph representation. Our goal is to provide the best Knowledge Graph for LLM (GraphRAG).](https://github.com/falkordb/falkordb) - A super fast Graph Database uses GraphBLAS under the hood for its sparse adjacency matrix graph repr...

7. [Knowledge Graph vs. Vector Store: Choosing Your Retrieval Primitive](https://tianpan.co/blog/2026-04-18-knowledge-graph-vs-vector-store-retrieval-primitive) - Vector similarity and graph traversal answer different questions. Learn when vector stores fail on m...

8. [Knowledge Graph vs. Vector RAG: Optimization & analysis - Neo4j](https://neo4j.com/blog/developer/knowledge-graph-vs-vector-rag/) - Learn how graph and vector search systems can work together to improve retrieval-augmented generatio...

9. [GraphRAG vs. Vector RAG: When Knowledge Graphs Outperform ...](https://flur.ee/blog/graphrag-vs-vector-rag-when-knowledge-graphs-outperform-semantic-search) - Vector RAG fails when answers live in connections, not content. Seven scenarios where GraphRAG trave...

10. [A Deep-Dive Into Vector Search and Knowledge Graph for ...](https://www.superteams.ai/blog/vector-search-and-knowledge-graph-for-ai/) - Learn how to select vector search or knowledge graph for your AI application

11. [GraphRAG vs Vector Search: Do You Need a Knowledge Graph?](https://theroadtoenterprise.com/blog/graphrag-when-vector-search-not-enough) - Vector search can't do set intersection or hierarchy traversal. When GraphRAG is worth it, when Lazy...

12. [Temporal values - Cypher Manual - Neo4j](https://neo4j.com/docs/cypher-manual/current/values-and-types/temporal/) - Cypher has built-in support for handling temporal values, and the underlying database supports stori...

13. [datetime()](https://neo4j.com/docs/cypher-manual/current/functions/temporal/) - Cypher provides functions allowing for the creation and manipulation of values for each temporal typ...

14. [Released FalkorDB 4.0-a1 Vector Search Index & Bolt Protocol](https://www.falkordb.com/blog/released-falkordb-4-0-a1-vector-search-index-bolt-protocol/) - Discover the latest advancements in FalkorDB 4.0-a1, featuring the powerful Vector Search Index and ...

15. [Agent memory at enterprise scale — Zep](https://www.getzep.com) - Unlike other systems that only retrieve static documents, Zep uses a temporal knowledge graph to com...

16. [When Adding Documents Broke My Search System](https://dev.to/peter_wu_8ad6dcfb10c6f20d/how-i-built-a-rag-system-with-a-7-million-node-knowledge-graph-and-why-vector-search-alone-isnt-4f3e) - How I learned that vector search alone isn't enough, and built a knowledge graph that finds...

17. [The FalkorDB Design](https://docs.falkordb.com/design/) - FalkorDB: A High Performance In-Memory Graph Database

18. [FalkorDB/FalkorDB - DeepWiki](https://deepwiki.com/FalkorDB/FalkorDB) - This page provides a high-level introduction to FalkorDB: its purpose, key features, architectural d...

19. [What is FalkorDB?](http://ritzinmobileworld.blogspot.com/2025/09/what-is-falkordb.html) - • FalkorDB is a graph database built on top of Redis, designed for real-time AI/ML applications. • I...

20. [Time Ontology in OWL - W3C](https://www.w3.org/TR/owl-time/) - OWL-Time is an OWL-2 DL ontology of temporal concepts, for describing the temporal properties of res...

21. [Representing and Querying Validity Time in RDF and OWL](https://www.cs.ox.ac.uk/people/boris.motik/pubs/m10validity-time.pdf)

22. [Temporally Enhanced Ontologies in OWL: A Shared Conceptual Model and Reference Implementation](https://dl.acm.org/doi/10.1007/978-3-319-26190-4_3)

23. [The Next Layer After Karpathy’s Wiki: FalkorDB Graph + Vector Search for LLMs](https://www.youtube.com/watch?v=OUCij3cOkpY) - Karpathy’s LLM‑Wiki showed how powerful it is to have an always‑up‑to‑date wiki that LLMs can read i...

