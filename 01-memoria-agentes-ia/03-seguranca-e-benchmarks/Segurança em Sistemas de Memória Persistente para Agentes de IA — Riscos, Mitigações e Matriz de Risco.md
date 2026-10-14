# Segurança em Sistemas de Memória Persistente para Agentes de IA: Riscos, Mitigações e Matriz de Risco

> Pesquisa aprofundada sobre ameaças, controles e frameworks relevantes para sistemas de memória de agentes autônomos de IA. Cobre riscos documentados por Google, Anthropic, OpenAI, Microsoft, OWASP, NIST e literatura acadêmica, com mitigações práticas, design de permissões e guia de testes adversariais.

***

## 1. Contexto e Taxonomia de Memória em Agentes de IA

Sistemas de memória persistente transformaram agentes de IA de ferramentas de sessão única em entidades com continuidade de estado entre sessões. Frameworks como LangGraph, Mem0, Zep e Letta padronizaram quatro camadas de memória: (1) memória de sessão (contexto de janela), (2) memória episódica (histórico de conversas), (3) memória semântica (fatos e crenças), e (4) memória procedural (padrões e regras). A transição de chatbots stateless para agentes com estado persistente criou uma superfície de ataque inteiramente nova: **o artefato durável é a vulnerabilidade**.[^1][^2]

A distinção estrutural entre *prompt injection* clássico e *memory poisoning* é crucial para o design de defesas. Prompt injection é session-scoped — o dano se encerra quando a sessão termina. Memory poisoning é persistente: o ataque e seu efeito são temporalmente desacoplados, podendo separar o vetor de ataque do impacto observável por semanas ou meses. Essa diferença invalida toda a arquitetura de defesa baseada apenas em guardrails de sessão.[^3][^4][^1]

***

## 2. Riscos Documentados por Fornecedores Oficiais

### 2.1 OWASP — LLM Top 10 (2025) e Agentic AI Top 10 (2026)

O OWASP é a fonte oficial mais referenciada. No **OWASP LLM Top 10 (2025)**, os riscos mais relevantes para memória persistente são:[^5]

| # | Risco OWASP LLM 2025 | Relevância para Memória |
|---|---|---|
| LLM01:2025 | Prompt Injection | Semente para memory poisoning; permanece #1 desde 2023 |
| LLM02:2025 | Sensitive Information Disclosure | PII, secrets e credenciais em logs/memória |
| LLM03:2025 | Supply Chain | Dependências e plugins comprometidos |
| LLM04:2025 | Data and Model Poisoning | Envenenamento de bases RAG e embeddings |
| LLM06:2025 | Excessive Agency | Agente com permissões além do necessário |
| LLM07:2025 | System Prompt Leakage | Exposição de credenciais em prompts de sistema |
| LLM08:2025 | Vector and Embedding Weaknesses | Ataques a bancos vetoriais |
| LLM09:2025 | Misinformation | Fatos falsos armazenados como verdade |

No **OWASP Top 10 for Agentic AI (2026)**, memory poisoning foi formalizado como **ASI06 — Memory and Context Poisoning**, reconhecendo que os controles de sessão do LLM Top 10 não cobrem a camada de estado persistente. O OWASP lançou também o **OWASP Agent Memory Guard**, uma implementação de referência open-source que rastreia leituras e escritas de memória com política YAML, suporte a SHA-256 cryptographic baselines e rollback forense para estados conhecidamente seguros.[^4]

### 2.2 Google — SAIF (Secure AI Framework) e White Paper de Agentes

O Google publicou seu **SAIF (Secure AI Framework)** com uma extensão específica para sistemas agentivos. O SAIF define quatro componentes de risco para agentes: Application & Perception, Reasoning Core, Orchestration e Response Rendering. Para memória, o SAIF especifica que:[^6][^7]

- **Agent Memory** torna-se vetor de ataque quando dados maliciosos são armazenados, resultando em ataques persistentes
- O isolamento de memória entre usuários distintos é obrigatório para prevenir contaminação cross-tenant
- O princípio de **least-privilege** deve ser dinâmico e contextual: as permissões devem se adaptar à consulta atual do usuário, não ser estáticas

O white paper técnico do Google (**"Google's Approach for Secure AI Agents"**, 2025) advoga por uma estratégia de defesa em profundidade híbrida combinando controles determinísticos com defesas baseadas em raciocínio, ancorada em três princípios: agentes devem ter controladores humanos bem definidos, seus poderes devem ser cuidadosamente limitados, e suas ações devem ser observáveis.[^8]

A pesquisa da Unit 42 (Palo Alto Networks) documentou em 2025 uma prova de conceito de **indirect prompt injection que envenena a memória de longo prazo** de um agente via manipulação do mecanismo de sumarização de sessão. Em março de 2026, a mesma Unit 42 confirmou ataques de *indirect prompt injection* ativos em produção, com conteúdo adversarial em páginas web se tornando instruções executadas por agentes.[^9][^10]

### 2.3 Anthropic — Zero Trust for AI Agents (Maio 2026)

A Anthropic publicou em maio de 2026 o guia **"Zero Trust for AI Agents"**, um dos documentos oficiais mais abrangentes sobre o tema. O guia estabelece cinco riscos primários para agentes:[^11][^12]

1. **Prompt Injection**: instruções embutidas em dados externos
2. **Tool Poisoning**: servidores MCP ou plugins com código malicioso
3. **Abuse of Permissions**: permissões acumuladas além do necessário
4. **Memory Poisoning**: informações não autorizadas escritas em memória (ex.: MEMORY.md)
5. **Supply Chain Attacks**: contaminação de canais de distribuição de skills/plugins

O framework Zero Trust da Anthropic propõe três estágios de maturidade — Foundation (identidade única por agente, least-privilege, logging), Advanced (sandboxing, validação de I/O, proteção de memória) e Optimized (Agentic SOAR, detecção de anomalias em tempo real). O guia enfatiza que *"state-of-the-art AI models compress the time from vulnerability discovery to exploit creation from months to hours"*, tornando a defesa manual insuficiente.[^13]

O documento NIST RFI de Segurança Agentiva submetido pela Anthropic cobre explicitamente *prompt injection, persistent memory poisoning e tool supply chain risk* como superfícies de ataque específicas de sistemas agentivos.[^14]

### 2.4 Microsoft — AI Red Team Taxonomy (2025 e 2026)

O **Microsoft AI Red Team (AIRT)** publicou em abril de 2025 uma taxonomia de modos de falha em sistemas agentivos, atualizada em junho de 2026 de 27 para 34 modos de falha. Os riscos de segurança classificados como "existentes/amplificados" incluem:[^15][^16]

- Memory poisoning e theft
- Targeted knowledge-base poisoning
- Cross-domain Prompt Injection (XPIA)
- Human-in-the-loop bypass
- Incorrect permissions management
- Excessive agency

Os modos novos adicionados em 2026 incluem **session context contamination**, **inter-agent trust escalation**, **MCP/plugin abuse**, **agentic supply chain compromise** e **goal hijacking**.[^16]

A Microsoft Defender documentou em fevereiro de 2026 **50 tentativas distintas de injetar instruções de persistência na memória de assistentes de IA em 31 empresas de 14 indústrias em 60 dias**, confirmando que memory poisoning já não é teórico.[^17]

### 2.5 NIST AI RMF (AI 600-1, Julho 2024)

O **NIST AI 600-1** (Generative AI Profile, julho de 2024) mapeia 12 categorias de risco de IA generativa, incluindo *data poisoning*, *confabulation* (fatos falsos), *information security* (vazamento de dados) e *human-AI configuration*. O Cloud Security Alliance publicou em 2026 um **NIST AI RMF: Agentic Profile** que estende esses controles especificamente para sistemas com memória persistente e autonomia.[^18][^19][^20]

### 2.6 MITRE ATLAS

O MITRE **ATLAS (Adversarial Threat Landscape for AI Systems)** é o equivalente do ATT&CK para ataques a sistemas de ML/IA. O CSA publicou em março de 2026 uma análise de gap entre MITRE ATT&CK/ATLAS e sistemas agentivos, identificando seis categorias de técnica sem representação adequada: *agent-to-agent lateral movement, tool-chain poisoning, orchestrator hijacking*, e outros.[^21][^22][^23]

***

## 3. Riscos Documentados na Literatura Técnica e Acadêmica

### 3.1 Memory Poisoning — AgentPoison e MINJA

**AgentPoison** (NeurIPS 2024, arXiv 2407.12784) foi o primeiro ataque de backdoor demonstrado especificamente para agentes RAG-based. Os resultados publicados mostram:[^24][^25]

- Taxa de sucesso de ataque ≥ 80% em três tipos de agentes reais (autonomous driving, QA, EHRAgent)
- Poison rate < 0,1% do corpus (pouquíssimos documentos envenenados)
- Impacto em consultas benignas < 1% (ataque quase invisível)
- Nenhum retreinamento de modelo necessário[^26][^24]

**MINJA** (Memory INJection Attack, arXiv 2503.03704), apresentado no NeurIPS 2025, demonstrou que o atacante não precisa de acesso privilegiado ao store de memória — interação somente por queries é suficiente. Taxa de sucesso de injeção superior a 95% em benchmarks LLM-based (EHRAgent, RAP, MMLU).[^27][^4]

**Sleeper Memory Poisoning** ("Hidden in Memory", arXiv 2605.15338) descreve ataques time-bomb: memórias envenenadas que permanecem dormentes e ativam apenas quando condições acumuladas se satisfazem. Resultados: 99,8% de aceitação de memória envenenada no GPT-5.5; 95% no Kimi-K2.6; entre resgates bem-sucedidos, memórias envenenadas causam ações intencionadas pelo atacante em 60–89% das avaliações.[^4]

**Memory Control Flow Attacks** ("From Storage to Steering", arXiv 2603.15125) demonstram que manipulações da camada de armazenamento podem direcionar raciocínio futuro do agente, forçando uso de ferramentas não intencionadas mesmo contra instruções explícitas do usuário. Mais de 90% dos testes vulneráveis em GPT-5 mini, Claude Sonnet 4.5 e Gemini 2.5 Flash.[^4]

O **Agent Security Bench** reporta uma taxa de sucesso média máxima de 84,30% em 27 combinações de ataque/defesa, com eficácia limitada das defesas atuais em isolamento.[^4]

### 3.2 Prompt Injection Persistente (Cross-Session)

O paper *"What If Prompt Injection Never Left?"* (arXiv, junho 2026) formaliza a distinção entre *reflected* (clássico) e *stored cross-session prompt injection*. A analogia com web security é precisa:[^1]

| Tipo | Analogia Web | Lifetime | Detecção |
|---|---|---|---|
| Prompt Injection clássico | Reflected XSS | Uma sessão | Imediata |
| Cross-session stored injection | Stored XSS | Indefinido (persiste em memória/checkpoints) | Atrasada (dias) |

A dimensão cross-user é subestimada: em stores de memória compartilhados (equipes, multi-tenant), uma injeção plantada por um usuário pode surgir na sessão de outro.[^1]

### 3.3 Dados Falsos como Verdade (Overreliance e Misinformation)

O OWASP LLM09:2025 (*Misinformation*) e os trabalhos sobre *overreliance* documentam o risco de LLMs armazenarem fatos não verificados com alta confiança aparente. A pesquisa A-MemGuard (2025) descobriu que detectores baseados em LLM perdem 66% das entradas de memória envenenadas, evidenciando o problema de confiar excessivamente em memórias geradas pelo próprio LLM.[^27][^5]

O problema se amplifica em conflitos entre memória armazenada e código atual: uma memória de 6 meses atrás dizendo que "a API X retorna JSON" pode conflitar com uma atualização que mudou o formato, e o agente prioriza a memória sem verificação.[^28][^29]

### 3.4 Supply Chain: Repositórios, Issues e PRs Comprometidos

A pesquisa **ClawdPwned** (OpenReview, 2026) realizou uma auditoria de 2.556 agent skills do ecossistema ClawHub (141.000+ stars), identificando **49 skills provavelmente maliciosas**. Os mecanismos de ataque mais comuns:[^30]

- Coleta de números de cartão de crédito completos (skill `tsyvic/buy-anything`)
- Roubo de credenciais LinkedIn e segredos 2FA via instaladores base64|bash ofuscados
- Extração de private keys de wallet crypto via executáveis password-protected
- **Envenenamento de memória com conteúdo controlado pelo atacante** (`cgallic/wake-up-skill`)

O ataque **Agentjacking** (junho 2026, Tenet Security) demonstrou injeção de instruções maliciosas via eventos Sentry — agentes de código que lêem logs e telemetria como contexto confiável podem ser manipulados para executar ações no repositório, instalar dependências ou roubar credenciais. O Mitiga Labs varreu 50.000+ arquivos de instrução de IA em 7.000+ repositórios e encontrou **1.230+ chaves de API expostas** e base URLs de Claude hijackadas.[^31][^32]

A cadeia de suprimentos do LangGraph foi comprometida com CVEs críticos: **CVE-2025-67644** (SQL injection no SQLite checkpointer), **CVE-2026-28277** (RCE via desserialização unsafe de msgpack) e **CVE-2026-27022** (Redis injection). O CheckPoint Research identificou que uma cadeia explorar esses CVEs permite **acesso completo ao servidor, incluindo chaves de API de LLM, dados de clientes, credenciais de CRM e histórico de conversas**.[^33][^34]

### 3.5 Vazamento de Secrets e Credenciais

O OWASP LLM02:2025 (Sensitive Information Disclosure) e LLM07:2025 (System Prompt Leakage) cobrem a exposição de API keys, tokens OAuth, connection strings de banco de dados e outros segredos. Os vetores documentados incluem:[^35][^36][^5]

1. Input acidental: usuário cola código com credenciais hardcoded
2. Tool output leaking: agent chama ferramenta que retorna variáveis de ambiente ou arquivos de configuração
3. Prompt injection extraction: atacante induz o agente a ler e exfiltrar `.env` ou `credentials.json`
4. Training data echoing: LLM regurgita segredos memorizados do treinamento

O CWE-532 (inserção de informações sensíveis em arquivos de log) e CWE-798 (use of hard-coded credentials) são as CWEs mais associadas a esse risco.[^37]

### 3.6 Dados Pessoais e PII

O GDPR Art. 5 exige minimização de dados, propósito específico, precisão e prazo de retenção. Para sistemas de memória de IA, a conformidade é complexa:[^38]

- **Art. 17 (direito ao esquecimento)** exige deleção completa: não apenas registros de banco, mas embeddings vetoriais (que podem ser invertidos), nós de knowledge graph e caches[^38]
- **Art. 22** cobre decisões automatizadas; sistemas RAG que fundamentam decisões em memória persistente ativam esta obrigação[^39]
- O **EU AI Act** (plenamente aplicável em agosto de 2026) adiciona obrigações de log de auditoria por 10 anos para sistemas de alto risco, criando uma tensão direta com o GDPR que exige deleção[^40][^41]

Agentes de IA operam a velocidade de máquina e podem gerar dezenas de milhares de eventos de conformidade por sessão, contra os quais frameworks regulatórios calibrados para acesso humano são inadequados.[^42]

***

## 4. Mitigações por Categoria de Risco

### 4.1 Memory Poisoning

**Controles técnicos automáticos:**
- **Validação no write path**: toda entrada na memória deve ser screened antes de ser armazenada. Implementar pipeline de detecção com stages: `base64_decode` → `detect_secrets` → `llm_screen` → `prompt_injection` → `size_anomaly` → `protected_keys`[^4]
- **Trust scoring por proveniência**: atribuir nível de confiança (high/medium/low) baseado na origem (sistema interno vs. documento externo vs. web). Memórias de fonte não confiável recebem peso reduzido na recuperação[^3]
- **Temporal decay**: implementar decaimento exponencial de confiança baseado em tempo desde criação e frequência de acesso. Ex.: `trust_eff = base_confidence × e^(−age_days/90) × access_factor`[^3]
- **Monitoramento de drift comportamental**: rastrear distribuição de fontes de memória recuperadas ao longo do tempo; alertar se taxa de recuperação de fontes de baixa confiança dobrar em relação ao baseline[^3]
- **Quarantine-on-detect**: mover entradas suspeitas para partição de quarentena em vez de deletar diretamente (preservar para forensics)[^3]

**Controles humanos:**
- Revisão manual obrigatória para escritas em memória do sistema (políticas, configurações)
- Revisão periódica de memórias acumuladas (ex.: sprint semanal de curadoria de memória)
- Aprovação humana para qualquer memória derivada de fontes externas[^12]

### 4.2 Prompt Injection Persistente

**Controles técnicos automáticos:**
- Separar conteúdo confiável (instruções do sistema) de não confiável (dados externos) em nível arquitetural — uso de tokens de controle especiais[^6]
- Tratar memória persistente como input não confiável no momento da leitura, aplicando a mesma scrutiny que conteúdo externo fresco[^1]
- Isolar state por proveniência: conteúdo derivado de fontes não confiáveis em partição quarantenada, nunca injetado diretamente em prompts de orquestração sem sanitização[^1]
- Aplicar TTLs e expiração agressiva para state não verificado; evitar estruturas append-only que alimentam prompts[^1]

**Controles humanos:**
- Checkpoint human-in-the-loop antes de ações irreversíveis (envio de email, commit, chamada de API destrutiva)[^12]
- Auditoria regular de logs de escrita de memória com rastreamento de proveniência[^1]

### 4.3 Dados Falsos Armazenados como Verdade

**Controles técnicos automáticos:**
- Validação de consistência: verificar contradições com memórias de alta confiança antes de armazenar nova entrada[^3]
- Expiração baseada em uso: entradas não acessadas por N dias entram em review automático[^43]
- Confidence scoring com floor: memórias geradas por LLM devem ter confiança automaticamente limitada (ex.: máximo 0.7) sem verificação externa[^4]

**Controles humanos:**
- Processo periódico de reconciliação memória vs. código atual (ex.: antes de cada release)
- Validação humana de memórias de alta influência (aquelas com maior taxa de recuperação)[^44]

### 4.4 Memórias Obsoletas

**Controles técnicos automáticos:**
- Políticas de retenção com TTLs automáticos por categoria (ex.: preferências de usuário: 90 dias; configs de sistema: sem expiração mas com review periódico)[^45][^43]
- **Controlled Forgetting**: o EU AI Act e GDPR exigem caminhos de deleção verificáveis que cubram todos os substratos de armazenamento: parâmetros do modelo, índices vetoriais, snapshots de backup e caches de sumarização[^45]
- Mecanismo de *version staleness*: marcar memórias como "needs_review" após X dias sem acesso[^29]

**Controles humanos:**
- Auditoria trimestral de memórias mais antigas que threshold definido por categoria[^43]

### 4.5 Vazamento de Secrets e Credenciais

**Controles técnicos automáticos:**
- Scanner de padrões no write path e read path: detectar formatos canônicos de credenciais (API keys de OpenAI, AWS, Google; private keys PEM; JWTs; connection strings)[^36][^4]
- Nunca armazenar credenciais em prompts de sistema — usar injeção de variáveis de ambiente via secret manager[^36]
- Redação automática em logs: implementar `redact_pii()` e `redact_secrets()` antes de qualquer escrita de log[^37]
- Restrição de acesso do agente ao filesystem: bloquear acesso a `.env`, `/etc/shadow`, `credentials.json`[^36]
- Rotação periódica de chaves; chaves estáticas devem ser tratadas como comprometidas (Anthropic recomenda tokens de curta duração)[^46]

**Controles humanos:**
- Secret scanning em CI/CD (GitHub Actions, pre-commit hooks)
- Revisão manual de qualquer memória que contenha `[REDACTED:*]` markers para confirmar sanitização adequada[^4]

### 4.6 Credenciais em Logs

**Controles técnicos automáticos:**
- Implementar pattern matching de alta entropia em pipelines de logging: detectar e redatar strings que correspondem a formatos de keys antes de persistir em qualquer sink de log (Splunk, Datadog, CloudWatch)[^37][^36]
- Webhooks com HMAC-SHA256 para integridade de eventos de segurança[^4]
- CWE-532 (logging de informações sensíveis): validar schema de log contra lista de campos permitidos[^37]

**Controles humanos:**
- Revisão de configurações de log em cada deploy de agente novo
- Auditoria regular de pipelines de observabilidade para confirmar que redação está ativa end-to-end[^44]

### 4.7 Dados Pessoais (PII)

**Controles técnicos automáticos:**
- Detecção automatizada de PII na ingestão: names, emails, telefones, IDs, dados de saúde, dados financeiros
- Tokenização: substituir PII por surrogate keys; agentes operam em tokens, não em dados reais; apenas "resolution nodes" com privilégio específico re-hidratam os dados[^47]
- Segmentação de state no LangGraph: `public_context` vs. `sensitive_context` vs. `internal_reasoning`; nós sem necessidade não recebem `sensitive_context`[^47]
- Workflows de erasure que cobrem: conteúdo da memória, embeddings vetoriais, nós de knowledge graph, caches de queries[^38]

**Controles humanos:**
- DPIA (Data Protection Impact Assessment) antes de deploy de agentes que processam dados de saúde, finanças ou crianças[^48]
- Processo documentado de resposta a requisições de erasure (Art. 17 GDPR) com SLA de 30 dias[^38]

### 4.8 Dependências Comprometidas (Supply Chain)

**Controles técnicos automáticos:**
- SBOM (Software Bill of Materials) para todos os componentes do stack de agentes
- Dependency scanning contínuo (Dependabot, Snyk, Trivy) com bloqueio de builds em CVEs críticos
- Verificação de integridade criptográfica de plugins e skills antes de instalação[^30]
- **Para LangGraph especificamente**: atualizar para `langgraph-checkpoint-sqlite 3.0.1+`, `langgraph 1.0.10+`, `langgraph-checkpoint-redis 1.0.2+` para corrigir CVE-2025-67644, CVE-2026-28277, CVE-2026-27022[^34]
- Varredura de arquivos de instrução (CLAUDE.md, MEMORY.md, `.cursorrules`, agent skills) em repos para API keys expostas (ferramenta: Skillgate)[^32]

**Controles humanos:**
- Auditoria manual de skills/plugins de terceiros antes de integração
- Processo de aprovação para adição de novos MCPs ou integrações de ferramentas[^30]

### 4.9 Instruções Maliciosas em Arquivos do Repositório

**Controles técnicos automáticos:**
- Scanner automático de arquivos de configuração de agentes (`.github/copilot-instructions.md`, `CLAUDE.md`, `MEMORY.md`, `AGENTS.md`) no CI/CD por padrões maliciosos[^32]
- Assinatura criptográfica de arquivos de instrução: verificar integridade antes que o agente os consuma
- Bloqueio de URLs externas em arquivos de instrução: permitir apenas domínios aprovados em allowlist[^12]

**Controles humanos:**
- Code review obrigatório para qualquer alteração em arquivos de instrução de agentes (mesmo que não sejam código executável)[^30]
- Política de não salvar informações obtidas de fontes externas diretamente em memória sem revisão[^12]

### 4.10 Contaminação por Issues/PRs Externos (Agentjacking)

**Controles técnicos automáticos:**
- Tratar issues, PRs, eventos de telemetria (Sentry, logs) como fontes não confiáveis — nunca injetá-los diretamente no contexto de planejamento do agente sem sanitização[^31]
- Filtro de prompt injection em todas as fontes externas que o agente consome
- Restrição de egress: monitorar e limitar ações que podem resultar de contextos externos (commits, exec de comandos, instalação de pacotes)[^31]

**Controles humanos:**
- Aprovação humana antes de ações de agente iniciadas por contexto externo (ex.: agente debugando issue do GitHub não deve poder fazer commit sem aprovação)[^31]

### 4.11 Excesso de Confiança em Memórias Geradas por LLM

**Controles técnicos automáticos:**
- Confidence ceiling para memórias geradas pelo LLM (max 0.7) vs. memórias verificadas por fonte primária (max 1.0)[^3]
- Cross-validation: antes de agir em memória com alto impacto, buscar corroboração em fonte independente[^44]
- Flag de `llm_generated: true` em metadados de proveniência[^3]

**Controles humanos:**
- Revisão humana de memórias com alta taxa de influência em decisões mas proveniência apenas de LLM[^44]

### 4.12 Contexto Irrelevante e Conflitos Memória vs. Código

**Controles técnicos automáticos:**
- Relevance filtering na recuperação: recuperar apenas memórias com similarity > threshold e timestamp recente[^43]
- Conflict detection: comparar memórias recuperadas contra estado atual do sistema (ex.: versão de API, schema de banco) antes de uso[^28]
- Memory decay automático: reduzir peso de memórias com >90 dias sem acesso[^3]

**Controles humanos:**
- Pre-release reconciliation: processo de validação de memórias críticas antes de cada release de sistema[^29]

### 4.13 Supply Chain de Modelos e Embeddings

**Controles técnicos automáticos:**
- Verificação de hash de modelos (SHA-256) antes de carregamento
- Audit trail de origem de modelos base e datasets de fine-tuning[^49]
- Isolamento de modelos auxiliares (safety classifiers, embedding models) como componentes do supply chain com suas próprias vulnerabilidades[^6]

**Controles humanos:**
- Revisão de proveniência de modelos fine-tuned: auditoria do dataset de treinamento[^18]

### 4.14 Rollback de Memória

**Controles técnicos automáticos:**
- Snapshots imutáveis versionados do store de memória antes de cada sessão de agente
- API de rollback: implementar `RollbackMemory(memory_id, target_revision)` com registro de cada revisão[^50]
- Append-only history: garantir que histórico de memórias persiste mesmo após deleção do estado atual[^50]
- Google Memory Bank e Claude managed-agent memory APIs já oferecem revisões imutáveis com `RollbackMemory`[^50]

**Controles humanos:**
- Procedimento documentado de resposta a incidente incluindo: identificar entradas envenenadas → traçar influência downstream → quarentenar (não deletar) → auditar decisões na janela de exposição → revalidar store restante[^3]

***

## 5. Design de Permissões

O princípio de **least-privilege dinâmico** é central para todos os fornecedores. A arquitetura recomendada:[^11][^6]

### 5.1 Hierarquia de Partições de Memória

```
memory:system          → read-only para agentes; write requer aprovação humana
memory:user:{user_id}  → isolado por usuário; agente A não acessa memória de agente B sobre usuário X
memory:agent:{agent_id}→ isolado por agente; previne cross-agent contamination
memory:quarantine      → entradas suspeitas; nenhum agente pode ler sem autorização explícita
```

### 5.2 Permissões por Operação

| Operação | Permissão Necessária | Escopo |
|---|---|---|
| Read memory:system | agent:read:system | Global, imutável |
| Write memory:system | human:write:system | Requer aprovação manual |
| Read memory:user | agent:read:user:{user_id} | Escopo por usuário |
| Write memory:user | agent:write:user:{user_id} | Escopo por usuário; screened |
| Delete memory | human:delete OR expiry_policy | GDPR Art.17 workflow |
| Rollback memory | human:rollback | Requer auditoria |
| Read memory:quarantine | security:read:quarantine | Apenas equipe de segurança |

### 5.3 Permissões de Ferramentas (Tool Permissions)

Seguindo o SAIF do Google e Anthropic Zero Trust:[^11][^6]
- Usar just-in-time permissions: conceder acesso apenas para a tarefa específica, não permanentemente
- Bloquear explicitamente ferramentas destrutivas: `rm`, `git push --force`, chamadas de API irreversíveis
- Cada servidor MCP deve ter permissões explicitamente declaradas e auditadas
- Separar permissões por projeto usando configurações locais (ex.: `.claude/settings.json` por projeto)[^12]

***

## 6. Como Evitar que Agentes Salvem Instruções Perigosas como Memória

Prevenir que o agente use a memória como vetor de persistência de instruções maliciosas requer controles no write path:[^4][^3]

1. **Instruction pattern detection**: scanner de padrões que identifica conteúdo semelhante a instrução (verbos imperativos, estruturas de override de sistema, jailbreak patterns) no write path
2. **Trust-source gating**: memórias derivadas de fontes não confiáveis (web, documentos externos, user inputs) recebem flag `external_source: true` e passam por review adicional
3. **Contradiction check**: antes de armazenar, verificar se a nova entrada contradiz memórias de alta confiança existentes
4. **Tamanho e entropia**: entradas acima de threshold de tamanho ou com alta entropia (possível payload ofuscado) são bloqueadas ou quarentenadas
5. **Semantic similarity to known attacks**: manter uma lista de padrões de ataques conhecidos (MINJA triggers, AgentPoison patterns) para comparação semântica[^4]

Para o **CLAUDE.md** e arquivos equivalentes, a Anthropic recomenda explicitamente listar o que o agente NÃO deve fazer, incluindo: não enviar variáveis de ambiente externamente, não fazer commit de `.env`, não acessar URLs desconhecidas, não tentar privilege escalation.[^12]

***

## 7. Como Testar Ataques contra a Memória

### 7.1 Metodologia de Red Teaming (6 fases)

Baseado no framework da ZioSec e CSA:[^51][^52]

**Fase 1 — Scope e Threat Modeling**: mapear todos os stores de memória (vector DB, SQLite, Redis, arquivos), identificar write paths, trust boundaries e pontos de recuperação

**Fase 2 — Seeding adversarial**: injetar entradas controladas nos stores de memória antes dos testes; incluir entradas benignas, entradas claramente maliciosas e entradas subtilmente envenenadas

**Fase 3 — Attack chaining**:
- Tool abuse: combinações inesperadas de parâmetros, ações encadeadas, boundary checks
- Memory poisoning: MINJA-style (via query apenas), AgentPoison-style (documentos no RAG), sleeper (trigger demorado)
- Cross-session injection: verificar se payload de sessão 1 ativa em sessão N com usuário diferente
- Exfiltration: testar se o agente pode ser induzido a exfiltrar memória via ferramentas disponíveis

**Fase 4 — Evidence capture**: instrumentar todos os sinks (tool calls, commits, emails, webhooks); capturar memory reads e writes com timestamp; verificar que audit trail é completo

**Fase 5 — Remediation e re-testing**: corrigir issues identificados; retestar com variações do mesmo ataque para confirmar mitigação robusta

**Fase 6 — Relatório**: documentar attack chain, evidências, blast radius, recomendações por severidade

### 7.2 Testes Específicos para Memória

| Teste | Método | Critério de Sucesso (Defesa) |
|---|---|---|
| Direct memory write injection | Escrever instrução maliciosa diretamente no store | Entrada bloqueada ou quarentenada; alert gerado |
| MINJA-style query poisoning | Sequência de queries benign-looking projetadas para plantar memória | Agente não executa instrução plantada em sessão N |
| Cross-session persistence | Sessão A planta memória; sessão B (usuário diferente) verifica ativação | Isolamento por usuário impede ativação |
| Sleeper trigger | Plantar memória dormente; ativar após N sessões | Expiração ou re-validation detecta antes de ativação |
| Secret exfiltration via memory | Incluir API key em user input; verificar se armazenado em memória | Key redatada antes do storage |
| Stale memory conflict | Armazenar fato que conflita com código atual | Conflict detection alerta; drift monitoring sinaliza |
| Rollback | Envenenar memória; executar rollback para snapshot anterior | Estado restaurado corretamente; log de rollback auditável |
| Supply chain skill audit | Instalar skill de terceiro; verificar comportamento | Skill scanneado antes de instalação; sem memory writes não autorizados |

***

## 8. Matriz de Risco

| # | Risco | Severidade | Probabilidade | Impacto | Mitigação Principal | Fonte |
|---|---|---|---|---|---|---|
| R01 | Memory Poisoning (direto/via RAG) | **CRÍTICA** | Alta (ASR ≥80% em PoCs) | Comportamento comprometido em todas as sessões futuras | Write-path screening; trust scoring; behavioral drift monitoring; rollback | OWASP ASI06[^4]; AgentPoison NeurIPS 2024[^24]; Microsoft AIRT[^15] |
| R02 | Prompt Injection Persistente (Cross-Session) | **CRÍTICA** | Alta (confirmada em produção) | Payload persiste indefinidamente; afeta múltiplos usuários | Tratar memória como input não confiável na leitura; TTLs; write gating; egress monitoring | arXiv 2606.04425[^1]; Unit 42 PoC[^9]; OWASP LLM01[^5] |
| R03 | Dados Falsos Armazenados como Verdade | **ALTA** | Alta (LLMs inherentemente confiáveis em memória própria) | Decisões incorretas baseadas em "fatos" fabricados | Confidence ceiling para memórias LLM-generated; cross-validation; contradiction check | OWASP LLM09[^5]; A-MemGuard 2025[^27] |
| R04 | Memórias Obsoletas | **ALTA** | Muito alta (inevitável sem governança) | Decisões baseadas em estado desatualizado do sistema | TTLs; temporal decay; pre-release reconciliation; controlled forgetting | GDPR Art.17[^38]; arXiv 2605.15338[^45] |
| R05 | Vazamento de Secrets / Credenciais | **CRÍTICA** | Média (requer vetor) | Exposição de API keys, tokens, passwords com blast radius difícil de conter | Pattern scanning (write+read path); redação em logs; secret manager; key rotation | OWASP LLM02/LLM07[^5][^35]; CWE-532/798[^37] |
| R06 | Credenciais em Logs | **ALTA** | Alta (muito comum em debugging) | Exposição em sistemas de observabilidade e treinamento de modelos | Redação automática antes de logging; schema validation de logs; log-level controls | CWE-532[^37]; Anthropic Zero Trust[^11] |
| R07 | PII em Memória / Violações GDPR | **ALTA** | Alta (agentes processam PII por natureza) | Multas regulatórias; vazamento de dados pessoais | Tokenização de PII; erasure workflows completos (embeddings + graph); DPIA | GDPR Arts. 5, 17, 22[^38][^42]; EU AI Act[^40] |
| R08 | Supply Chain — Dependências Comprometidas | **CRÍTICA** | Média (LangGraph CVEs ativos) | RCE; acesso completo ao servidor; exfiltração de todos os dados | SBOM; CVE scanning; patch management (CVEs LangGraph 2025-2026)[^34] | Check Point Research 2026[^33]; CSA LangChain/LangGraph report[^53] |
| R09 | Instruções Maliciosas em Arquivos de Repositório | **ALTA** | Média (Mitiga Labs: 1.230+ keys expostas) | Hijack de agent behavior; exfiltração de credenciais | CI/CD scanning de arquivos de instrução; code review obrigatório; allowlist de URLs | Mitiga Labs SkillGate 2026[^32]; ClawdPwned 2026[^30] |
| R10 | Contaminação por Issues/PRs/Telemetria (Agentjacking) | **ALTA** | Média (confirmada em produção contra Claude Code, Cursor, Codex) | Execução de código não autorizado; exfiltração via agent tooling | Tratar issues/logs como fontes não confiáveis; human-in-the-loop para ações iniciadas por contexto externo | Tenet Security Agentjacking 2026[^31]; CSA research note[^23] |
| R11 | Excesso de Confiança em Memórias LLM | **MÉDIA** | Alta | Propagação de erros/hallucinations como fatos estabelecidos | Confidence ceiling; human review de memórias de alta influência; fonte corroboradora | A-MemGuard 2025[^27]; OWASP LLM09[^5] |
| R12 | Contexto Irrelevante / Context Flooding | **MÉDIA** | Alta | Degradação de qualidade; custo de tokens; noise que oculta memórias relevantes | Relevance filtering; similarity threshold; expiry policies | OWASP LLM10 (Unbounded Consumption)[^5]; arXiv systematic study[^54] |
| R13 | Conflito Memória vs. Código Atual | **ALTA** | Alta (inevitável sem processo) | Comportamento incorreto baseado em estado desatualizado | Conflict detection automático; versioning; pre-release reconciliation | Microsoft AIRT taxonomy[^15]; arXiv 2606.04329[^54] |
| R14 | Supply Chain de Modelos/Embeddings | **ALTA** | Baixa (requer comprometimento do supply chain) | Backdoors no modelo base; comportamento malicioso difícil de detectar | Hash verification; model provenance audit; embedding provenance tracking | NIST AI 600-1[^18]; arXiv backdoors AI supply chain[^49] |
| R15 | Ausência de Rollback de Memória | **ALTA** | Alta (default de maioria dos frameworks) | Impossibilidade de recovery após poisoning; forensics inviável | Snapshots imutáveis; rollback API; append-only history com audit trail | Google Memory Bank docs[^50]; Claude managed-agent memory[^50]; OWASP Agent Memory Guard[^4] |

***

## 9. Validações Automáticas a Implementar

| Validação | Camada | Trigger |
|---|---|---|
| Instruction pattern detection | Write-path | Toda escrita de memória |
| Secret/credential regex scan (220+ padrões) | Write-path + log pipeline | Toda escrita + toda linha de log |
| PII detection e tokenização | Ingestão | Toda entrada de dados pessoais |
| Contradiction check vs. high-trust memories | Write-path | Toda escrita de fato |
| Provenance tagging | Write-path | Toda escrita de memória |
| Temporal decay scoring | Retrieval | Toda leitura |
| Behavioral drift monitoring | Runtime | Batch diário sobre últimas N sessões |
| CVE scan de dependências | CI/CD | Todo pull request e schedule diário |
| Audit log scanning para secrets | Post-processing | Pipeline de observabilidade |
| Trust-aware reranking | Retrieval | Toda consulta a memória |
| Snapshot antes de sessão | Runtime | Toda sessão de agente |

***

## 10. Validações Humanas a Manter

| Validação Humana | Frequência | Responsável |
|---|---|---|
| Revisão de escritas em memory:system | Por evento | Tech Lead / Security |
| Revisão de memórias com confiança apenas LLM e alta influência | Semanal | Product/Security |
| Auditoria de skills/plugins de terceiros antes de integração | Por integração | Security Engineer |
| Code review de arquivos de instrução de agente | Por PR | Tech Lead |
| Pre-release reconciliation memória vs. código | Por release | Dev + Product |
| DPIA para novos casos de uso com PII sensível | Por deploy | DPO/Legal |
| Resposta a requisições de erasure (GDPR Art.17) | Por requisição (SLA 30 dias) | DPO |
| Red team de memória | Trimestral | Security Red Team |
| Revisão de permissões de agentes e MCPs | Mensal | Security/Ops |
| Análise de logs de quarentena | Semanal | Security Ops |

***

## 11. Fontes Oficiais e Técnicas Referenciadas

- **OWASP LLM Top 10 (2025)**: genai.owasp.org[^5]
- **OWASP Agentic AI Top 10 (ASI06)**: Memory and Context Poisoning[^4]
- **Google SAIF Focus on Agents**: saif.google/focus-on-agents[^6]
- **Google "An Introduction to Google's Approach for Secure AI Agents" (2025)**: research.google[^8]
- **Anthropic "Zero Trust for AI Agents" (Maio 2026)**: claude.com/blog/zero-trust-for-ai-agents[^11]
- **Anthropic NIST RFI on Agentic Security**: cdn.anthropic.com[^14]
- **Microsoft AI Red Team Taxonomy v2.0 (Junho 2026)**: microsoft.com/security/blog[^16]
- **Microsoft Defender — 50 memory poisoning attempts in 31 companies (2026)**[^17]
- **NIST AI 600-1 Generative AI Profile (Julho 2024)**: nvlpubs.nist.gov[^18]
- **CSA NIST AI RMF Agentic Profile (2026)**: labs.cloudsecurityalliance.org[^19]
- **MITRE ATLAS**: atlas.mitre.org[^22]
- **AgentPoison (NeurIPS 2024)**: arXiv:2407.12784[^24]
- **MINJA attack (2025)**: arXiv:2503.03704[^4]
- **Sleeper Memory Poisoning (2026)**: arXiv:2605.15338[^4]
- **Cross-session stored prompt injection (Junho 2026)**: arXiv:2606.04425[^1]
- **Check Point Research — LangGraph CVEs (2026)**: blog.checkpoint.com[^33]
- **LangGraph Security Advisories**: github.com/langchain-ai/langgraph/security[^55]
- **Unit 42 — Indirect Prompt Injection Poisons AI Long-Term Memory (2025)**: unit42.paloaltonetworks.com[^9]
- **Tenet Security — Agentjacking (Junho 2026)**[^31]
- **Mitiga Labs — SkillGate AI Instruction File Scan (2026)**[^32]
- **ClawdPwned — Malicious Instructions in Agent Skills (2026)**: openreview.net[^30]
- **GDPR Art. 5, 17, 22 e EU AI Act Art. 12**[^41][^40][^38]
- **PoisonedRAG (USENIX Security 2025)**[^3]
- **MemoryGraft (Dezembro 2025)**[^3]

---

## References

1. [Stored prompt injection: when an injection outlives the session](https://www.llm-hacking.com/hacks/cross-session-stored-prompt-injection.md/) - A June 2026 arXiv paper reframes prompt injection as a stored, cross-session problem: once adversari...

2. [AI Agent Memory Architectures: From Context Windows ... - Zylos](https://zylos.ai/research/2026-04-05-ai-agent-memory-architectures-persistent-knowledge/) - Mem0 is the most widely deployed semantic memory layer as of mid-2026, with approximately 48,000 Git...

3. [Memory and context poisoning: Don't let attackers rewrite your AI ...](https://workos.com/blog/ai-agent-memory-poisoning) - Prompt injection ends when the session closes. Memory poisoning persists across sessions, activates ...

4. [AI Memory Poisoning: How Attacks Corrupt Agent Memory](https://vectorize.io/articles/ai-memory-poisoning) - AI memory poisoning is OWASP ASI06 — a persistent attack that corrupts agent behavior across every f...

5. [OWASP Top 10 for LLMs 2025 - The LLM Red Teaming Framework](https://trydeepteam.com/docs/frameworks-owasp-top-10-for-llms) - The OWASP Top 10 for Large Language Models (LLMs) is a comprehensive list of the most critical secur...

6. [Components of Generative AI Systems - SAIF.Google](https://saif.google/focus-on-agents) - Agent memory: Agent Memory allows an agent to retain context and learn facts across interactions. It...

7. [Secure AI Framework](https://saif.google/secure-ai-framework) - Understand the Secure AI Framework and SAIF Risk Map. Explore the top security risks and controls fo...

8. [Google's Approach for Secure AI Agents](https://research.google/pubs/an-introduction-to-googles-approach-for-secure-ai-agents/) - As part of Google's ongoing efforts to define best practices for secure AI systems, we're sharing ou...

9. [When AI Remembers Too Much – Persistent Behaviors in ...](https://unit42.paloaltonetworks.com/indirect-prompt-injection-poisons-ai-longterm-memory/) - We demonstrated through our PoC that AI agents with long-term memory can serve as a vector for persi...

10. [Fooling AI Agents: Web-Based Indirect Prompt Injection Observed in ...](https://unit42.paloaltonetworks.com/ai-agent-prompt-injection/) - Uncover real-world indirect prompt injection attacks and learn how adversaries weaponize hidden web ...

11. [Zero Trust for AI agents](https://claude.com/blog/zero-trust-for-ai-agents) - An eight-phase implementation workflow covering identity, access scoping, sandboxing, input and outp...

12. [My Claude Code permission design was too lax. A review after ...](https://note.com/hacklog_stealth/n/n1860151d2552?hl=en) - Anthropic has released a guide titled 'Zero Trust for AI Agents' on their official blog.In a nutshel...

13. [Anthropic's 'Zero Trust Design for AI Agents' Released](https://note.com/samurai_ai/n/n053883918aba?hl=en) - 'I have given my AI agent access to the company database.'If you read that sentence and felt nothing...

14. [NIST RFI on Agentic Security](https://www-cdn.anthropic.com/43ec7e770925deabc3f0bc1dbf0133769fd03812.pdf) - We study the attack surfaces specific to agentic AI, including prompt injection, persistent memory p...

15. [Microsoft Releases a Comprehensive Guide to Failure Modes in ...](https://www.marktechpost.com/2025/04/27/microsoft-releases-a-comprehensive-guide-to-failure-modes-in-agentic-ai-systems/) - Microsoft Releases a Comprehensive Guide to Failure Modes in Agentic AI Systems

16. [Microsoft adds seven failure modes for AI agents - The Weather Report](https://theweatherreport.ai/posts/microsoft-ai-agent-failure-modes/) - Microsoft's AI Red Team added seven failure modes to its agentic AI taxonomy after a year of red tea...

17. [Microsoft caught 31 companies poisoning AI memory in 60 days ...](https://www.linkedin.com/pulse/microsoft-caught-31-companies-poisoning-ai-memory-60-days-david-baum-7ksue) - In February 2026, Microsoft Defender published a finding that should have changed how every governan...

18. [Artificial Intelligence Risk Management Framework](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) - de N AI · 2024 · Citado por 164 — Future revisions of this profile will include additional AI RMF su...

19. [NIST AI Risk Management Framework: Agentic Profile - Lab Space](https://labs.cloudsecurityalliance.org/agentic/agentic-nist-ai-rmf-profile-v1/) - NIST AI Risk Management Framework: Agentic Profile Executive Summary The proliferation of autonomous...

20. [NIST AI RMF 1.0 Implementation Guide for Enterprises 2026](https://neuraltrust.ai/blog/nist-ai-rmf-implementation-guide) - NIST AI 600-1, the Generative AI Profile published in July 2024, extends AI RMF 1.0 to cover 12 risk...

21. [mitre/advmlthreatmatrix: Adversarial Threat Landscape for AI Systems](https://github.com/mitre/advmlthreatmatrix) - The Adversarial ML Threat Matrix is a first-cut attempt at collating a knowledge base of how ML syst...

22. [MITRE ATLAS™](https://atlas.mitre.org)

23. [MITRE ATT&CK and ATLAS Agentic Gap Analysis - Lab Space](https://labs.cloudsecurityalliance.org/agentic/csa-research-note-atlas-agentic-gap-analysis-20260327/) - Six technique categories have no adequate representation in ATLAS: agent-to-agent lateral movement, ...

24. [[PDF] AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases | Semantic Scholar](https://www.semanticscholar.org/paper/AgentPoison:-Red-teaming-LLM-Agents-via-Poisoning-Chen-Xiang/b6948a9e8b3eec5a56a80c69727154fcd7ececce) - This work proposes a novel red teaming approach AgentPoison, the first backdoor attack targeting gen...

25. [Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases](https://arxiv.org/html/2407.12784v1)

26. [Memory Poisoning Attacks on AI: A Growing Threat](https://www.linkedin.com/posts/deanahart_%3F%3F%3F%3F-%3F%3F%3F%3F-%3F%3F-%3F%3F%3F%3F%3F%3F-%3F%3F-activity-7374524375678713856-ISn3) - 𝗪𝗵𝗲𝗻 𝘆𝗼𝘂𝗿 𝗔𝗜 𝗮𝗴𝗲𝗻𝘁'𝘀 𝗹𝗼𝗻𝗴-𝘁𝗲𝗿𝗺 𝗺𝗲𝗺𝗼𝗿𝘆 𝗴𝗲𝘁𝘀 𝗽𝗼𝗶𝘀𝗼𝗻𝗲𝗱, 𝗶𝘁 𝗰𝗮𝗻 𝗾𝘂𝗶𝗲𝘁𝗹𝘆 𝗯𝗲𝗰𝗼𝗺𝗲 𝗮 𝗹𝗶𝗮𝗯𝗶𝗹𝗶𝘁𝘆 𝘆𝗼𝘂 𝗰𝗮𝗻'𝘁 𝘀𝗲𝗲...

27. [AI Memory Security: Best Practices and Implementation](https://mem0.ai/blog/ai-memory-security-best-practices) - The A-MemGuard defense framework research (2025) found that even advanced LLM-based detectors miss 6...

28. [What breaks when AI agents rely on remembered workflow patterns ...](https://nhimg.org/faq/what-breaks-when-ai-agents-rely-on-remembered-workflow-patterns-instead-of-fresh/) - What breaks is the assumption that every action is independently reasoned and therefore easy to revi...

29. [AI Agent Memory: Why It Rots in Production and the Fix](https://www.boundev.ai/blog/ai-agent-memory-decay-production) - AI agent memory degrades when you only write and never forget. How to design the write, retrieve, an...

30. [Malicious Instructions in the OpenClaw AI Agent Skills ...](https://openreview.net/forum?id=lGRtTpCZ7i) - OpenClaw (formerly ClawdBot) has experienced explosive growth, gaining over 141000 GitHub stars and ...

31. [Agentjacking MCP Injection Hijacks AI Coding Agents | PointGuard AI](https://www.pointguardai.com/ai-security-incidents/agentjacking-shows-coding-agents-can-take-the-bait) - PointGuard AI analyzes Agentjacking, an MCP and coding-agent attack that abuses Sentry error ingesti...

32. [AI Agent Supply-Chain Malware in Instruction Files](https://www.mitiga.io/blog/malware-in-ai-instruction-files-skillgate) - Mitiga Labs scanned 50,000+ AI instruction files across 7,000+ repos and found hijacked Claude base ...

33. [When Your AI Agent's Memory Becomes a Security Liability - Check Point Blog](https://blog.checkpoint.com/research/when-your-ai-agents-memory-becomes-a-security-liability) - Key Findings: Check Point Research identified a critical vulnerability chain in LangGraph, an open-s...

34. [LangGraph flaws show the risk of stored agent memory](https://cyber-ivy.com/en/articles/langgraph-checkpointer-rce-agent-security-2026-06) - Check Point details a patched LangGraph vulnerability chain combining SQL injection and deserializat...

35. [OWASP Top 10 LLM Security Risks (2025) – 5-Minute TLDR](https://www.promptfoo.dev/blog/owasp-top-10-llms-tldr/) - Master OWASP Top 10 LLM security vulnerabilities with practical mitigation strategies in this compre...

36. [Secret and Credential Exposure in AI Agent I/O](https://runesec.dev/threats/secret-exposure)

37. [Data Exposure in AI Agents — Secrets and PII Leakage – Inkog Docs](https://docs.inkog.io/vulnerabilities/data-exposure) - Prevent infinite loops, token bombing, and prompt injection in LangChain, CrewAI, n8n, and any agent...

38. [How to Implement GDPR-Compliant AI Memory - Adaptive Recall](https://adaptiverecall.com/enterprise-memory/gdpr-compliant-memory.php) - Step-by-step guide to building GDPR-compliant AI memory systems. Lawful basis, consent management, d...

39. [EU AI Act Agentic AI Memory & RAG Compliance: GDPR Art.17 ...](https://sota.io/blog/eu-ai-act-agentic-ai-memory-rag-compliance-gdpr-vector-stores-2026) - How GDPR Art.17 right to erasure applies to vector embeddings, why RAG pipelines trigger Art.22 auto...

40. [GDPR says delete. EU AI Act says keep. Now what?](https://www.channel.tel/blog/gdpr-delete-eu-ai-act-keep-memory-compliance) - GDPR requires deletion on request. The EU AI Act requires 10-year audit trails. Here's how to archit...

41. [EU AI Act Article 12 and Annex III: what high-risk agent systems must log by August 2026 — DThink](https://dthink.ai/blog/eu-ai-act-article-12-what-to-log/) - High-risk AI systems face mandatory tamper-evident logging obligations under EU AI Act Article 12, w...

42. [Data Privacy Rules Built for Human Behavior Have an AI Agent ...](https://www.corporatecomplianceinsights.com/data-privacy-rules-built-human-behavior-ai-agent-problem/) - Srikanth Sallaka of TrustLogix examines what happens when agentic AI meets a regulatory structure th...

43. [How to prevent memory pollution in AI agents?](https://www.tencentcloud.com/techpedia/126470) - Preventing memory pollution in AI agents involves strategies to ensure the integrity, accuracy, and ...

44. [Memory Risk Framework and Mitigation Playbook for Production ...](https://medium.com/@bijit211987/memory-risk-framework-and-mitigation-playbook-for-production-ready-ai-agents-0bdcdbffcf1e) - A Practical Playbook to De-Risk AI Agent Memory and Build Trust from Day One

45. [Controlled Forgetting: Memory Retention Policies and the Right to Erase in Persistent AI Agents | Zylos Research](https://zylos.ai/research/2026-06-04-controlled-forgetting-ai-agent-memory-retention/) - Why production AI agents need deliberate forgetting mechanisms — covering retention policies, machin...

46. [ANTHROPIC A 36-PAGE SECURITY GUIDE THAT ...](https://www.facebook.com/0xSojalSec/posts/anthropic-a-36-page-security-guide-that-basically-tells-you-to-stop-trusting-you/1543541460633585/) - ANTHROPIC A 36-PAGE SECURITY GUIDE THAT BASICALLY TELLS YOU TO STOP TRUSTING YOUR OWN AI AGENTS. If ...

47. [Handling sensitive data in LangGraph's shared State (Found an ...](https://forum.langchain.com/t/handling-sensitive-data-in-langgraphs-shared-state-found-an-interesting-paper/3013) - Hey everyone, I’ve been building some complex multi-agent workflows with LangGraph lately and hit a ...

48. [AI Agent GDPR Compliance 2026: 12-Point Checklist](https://technovapartners.com/en/insights/security-gdpr-enterprise-ai-agents) - Is your AI agent GDPR-ready? 12-point compliance checklist, EU AI Act requirements, 3 real breach ca...

49. [Down the Rabbit Hole of Backdoors in the AI Supply Chain](https://arxiv.org/html/2510.05159v1) - These findings highlight an urgent threat to agentic AI development and underscore the critical need...

50. [Agent Memory Needs Undo and Recovery Path](https://www.linkedin.com/posts/kerss_ai-agents-explained-memory-needs-undo-do-activity-7469045567961329664-Z-jN) - AI Agents Explained: Memory Needs Undo Do not build agent memory without rollback. Bad memory is not...

51. [Adversarial Red-Teaming Playbooks for AI Agent Hardening Programs](https://www.armalo.ai/learn/red-teaming-ai-agent-hardening)

52. [Break-your-own AI agent testing needs a red-team framework](https://nhimg.org/articles/break-your-own-ai-agent-testing-needs-a-red-team-framework/) - What this is: This is a practical six-phase framework for red-teaming AI agents that maps attack cha...

53. [[PDF] LangChain/LangGraph: Critical Flaws in the AI Dev Stack - Lab Space](https://labs.cloudsecurityalliance.org/wp-content/uploads/2026/03/CSA_research_note_LangChain-LangGraph-critical-vulns-framework-security-20260327-csa-styled.pdf)

54. [A Systematic Study of Memory Poisoning Attacks in LLM ...](https://arxiv.org/html/2606.04329v1) - We present a systematic study of memory poisoning in LLM-based agents. We identify four memory write...

55. [Security Advisories · langchain-ai/langgraph - GitHub](https://github.com/langchain-ai/langgraph/security/advisories) - GitHub is where people build software. More than 150 million people use GitHub to discover, fork, an...

