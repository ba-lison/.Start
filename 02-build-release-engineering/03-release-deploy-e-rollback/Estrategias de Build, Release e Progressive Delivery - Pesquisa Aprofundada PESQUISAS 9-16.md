# Estratégias de Build, Release e Progressive Delivery

## 1. Resumo Executivo

Este relatório consolida oito frentes de pesquisa (9 a 16) sobre estratégias de release, deployment progressivo, build e distribuição de software desktop/plugins CAD, migrações de banco de dados, observabilidade e resposta a incidentes, segurança da cadeia de suprimentos, governança de release e métricas de engenharia, encerrando com um modelo operacional sintetizado.

Os achados centrais são: (1) deployment (mecanismo técnico de colocar código em execução) é conceitualmente distinto de release (ato de tornar uma funcionalidade visível/disponível a usuários) e de experimentação de produto (uso de variantes para aprender), um princípio articulado por praticantes de continuous delivery e consolidado na literatura de feature flags; (2) o padrão "expand and contract" (também chamado "parallel change") é a abordagem consensual da indústria para evoluir esquemas de banco de dados sem downtime, documentado por múltiplas fontes técnicas independentes com fases equivalentes de expandir, migrar/backfill e contrair; (3) para plugins CAD Windows, a compatibilidade binária entre versões do host (ex.: AutoCAD) muda a cada release anual e exige recompilação, conforme documentação oficial da Autodesk; (4) segurança de cadeia de suprimentos de software tem framework de referência oficial (NIST SSDF SP 800-218) e modelo de níveis de garantia de build (SLSA), mas nem todo material disponível sobre SLSA é uma norma formal — muito é literatura de fornecedores e da comunidade explicando a especificação; (5) métricas DORA (frequência de deploy, lead time, taxa de falha de mudança, tempo de recuperação) têm base empírica em pesquisa anual da Google/DORA, mas a robustez estatística e a generalização entre contextos organizacionais são limitadas e não implicam causalidade direta entre práticas e desempenho de negócio.[^1][^2][^3][^4][^5][^6][^7][^8][^9][^10][^11]

Como a query cobre 8 sub-pesquisas amplas e a organização do usuário (TotalCAD) não especificou stack completa, ferramenta de CI/CD, ou critérios de criticidade para todos os campos solicitados na Pesquisa 16, este relatório apresenta cenários alternativos e declara explicitamente as lacunas de informação em vez de presumir escolhas específicas de produto.

## 2. Definições e Terminologia

| Termo | Definição | Fonte |
|---|---|---|
| Deployment | Ato técnico de colocar uma nova versão de código em execução em um ambiente (pode estar oculta do usuário) | [^1] |
| Release | Ato de tornar uma funcionalidade visível/acessível a um conjunto de usuários, potencialmente após o deployment já ter ocorrido | [^1] |
| Blue-green deployment | Duas cópias de produção (blue/green); tráfego é roteado de uma para outra numa troca abrupta, permitindo rollback rápido por reversão de roteamento | [^12][^13] |
| Canary release | Rollout gradual da nova versão para um subconjunto pequeno de usuários antes de expandir para o restante, monitorando sinais de erro | [^14][^15] |
| Feature flag | Mecanismo para ativar/desativar comportamento em tempo de execução sem novo deploy, distinto de branch de código | [^1] |
| Kill switch | Tipo de feature flag operacional, geralmente permanente, para desligar rapidamente uma funcionalidade problemática em produção | [^16][^17] |
| Expand and contract | Padrão de migração de schema que separa uma mudança destrutiva em fases aditivas (expand), de transição (migrate/backfill) e de remoção (contract) | [^2][^3][^5] |
| SLSA | Framework de níveis de garantia de proveniência e integridade de build de software (Supply-chain Levels for Software Artifacts) | [^4] |
| SSDF | Secure Software Development Framework, publicado pelo NIST como SP 800-218, com práticas de segurança ao longo do SDLC | [^11][^9] |
| DORA metrics | Quatro métricas (frequência de deploy, lead time, change failure rate, tempo de recuperação) originadas da pesquisa anual "Accelerate"/State of DevOps do DORA/Google | [^6][^7] |

Dependência declarada: a terminologia de "release por cliente/região/dispositivo" e "deployment desacoplado de release" não possui uma norma oficial única; a análise a seguir baseia-se em práticas documentadas de continuous delivery (Fowler, ThoughtWorks, e literatura de feature-flag vendors), tratadas aqui como consenso profissional/experiência de indústria, não como norma formal.

## 3. Funcionamento Técnico das Estratégias de Release

### Big bang release
Todo o sistema é substituído de uma vez, sem fases intermediárias. Não requer infraestrutura adicional além do ambiente de produção único, mas oferece rollback difícil (normalmente restauração completa de backup) e nenhum controle de exposição gradual. É adequado a sistemas simples, sem SLA de disponibilidade, ou a mudanças que não podem ser parcialmente aplicadas (ex.: migração de dados totalmente incompatível). Inadequado a produtos críticos ou com múltiplos consumidores simultâneos.

### Recreate deployment
A versão antiga é totalmente encerrada antes que a nova seja iniciada (comum em ambientes com poucas réplicas ou orquestradores simples como Kubernetes "Recreate" strategy). Implica downtime proporcional ao tempo de boot da aplicação; rollback exige o mesmo processo invertido. Adequado a ambientes de baixo tráfego ou jobs em lote; inadequado a serviços com exigência de disponibilidade contínua.

### Rolling deployment
Instâncias são substituídas em lotes sucessivos enquanto o restante do fleet permanece ativo, mantendo disponibilidade parcial durante a transição. Requer balanceador de carga e health checks para retirar instâncias de forma segura; rollback é possível revertendo o rollout, mas pode haver período com duas versões coexistindo no tráfego — exigindo compatibilidade de sessão e de schema entre versões (regra N/N-1).[^18]

### Blue-green deployment
Dois ambientes de produção completos (blue e green) coexistem; o roteador/balanceador direciona 100% do tráfego para um deles enquanto o outro recebe a nova versão e é validado antes do switch. Rollback é quase instantâneo (reverter o roteamento), mas o custo de infraestrutura dobra durante a transição, e mudanças de banco de dados exigem estratégia própria — ou o banco é compartilhado e versionado de forma compatível, ou é replicado via CDC (padrão "blue-green database", usado por serviços gerenciados como Amazon Aurora/RDS). Sessões com estado (sticky sessions, caches locais) precisam de replicação ou de design stateless para não serem perdidas na troca.[^12][^19][^13][^20][^21]

### Canary release
Uma pequena fração do tráfego (ou de usuários) é direcionada à nova versão; se as métricas de erro permanecem estáveis, o percentual aumenta progressivamente. Requer roteamento por peso, observabilidade granular por versão e capacidade de análise automática ou manual de canary. É reversível rapidamente (reduzir peso a zero), mas exige que ambas as versões coexistam e sejam compatíveis com o mesmo schema de dados. Adequado a APIs web e serviços de alto tráfego; para desktop/plugins, "canário" normalmente significa canal de distribuição (ex.: canal beta) em vez de roteamento de tráfego em tempo real.[^15][^14]

### Ring deployment
Extensão do canary em anéis concêntricos (ex.: anel 0 = equipe interna, anel 1 = early adopters, anel 2 = geral), popularizado por práticas de deployment de sistemas operacionais e enterprise software com atualização em ondas. Requer segmentação de população de usuários e canais de distribuição distintos; adequado a software desktop/enterprise com ciclos de update controlados, exatamente o cenário de plugins CAD corporativos.

### Phased rollout / percentage rollout
Aumenta gradualmente a porcentagem de usuários/instâncias expostas à nova versão ao longo do tempo, geralmente com portas de decisão (gates) em cada etapa. É a mecânica operacional por trás de canary e ring; a diferença é semântica — "phased" enfatiza etapas de negócio/tempo, "percentage" enfatiza o controle numérico de exposição.

### Dark launch e shadow traffic
Dark launch implanta a nova versão em produção sem exposição a usuários reais (ex.: endpoint oculto, feature flag sempre desativada para usuários finais). Shadow traffic (ou "traffic mirroring") duplica requisições reais para a nova versão sem retornar sua resposta ao usuário, permitindo comparar comportamento sob carga real sem risco. Exige infraestrutura de mirror de tráfego e comparação de resultados (diffing); não afeta usuários, mas pode gerar efeitos colaterais indesejados se a nova versão realizar escritas (mutações) durante o shadow.[^3][^22]

### A/B testing
Mecanismo de experimentação de produto — não deployment — que divide usuários entre variantes para medir impacto em métricas de negócio, com análise estatística de significância. Depende de feature flags ou sistemas de experimentação, e é fundamentalmente diferente de canary: canary busca minimizar risco técnico; A/B testing busca aprender sobre comportamento do usuário. Ambos podem usar a mesma infraestrutura de flags.[^1]

### Feature flags
Estruturas de controle que alternam comportamento em runtime sem novo deploy, permitindo desacoplar deployment de release. Documentação de fornecedores recomenda avaliação server-side, defaults seguros (geralmente "off") e datas de expiração para flags de release, para evitar acúmulo de dívida técnica. Kill switches são um subtipo permanente, usado para desligar funcionalidades problemáticas rapidamente.[^16][^17][^1]

### Release por cliente / região / dispositivo
Formas de segmentação de rollout: por cliente (tenant-by-tenant, comum em SaaS multi-tenant e plugins corporativos com licenciamento por cliente), por região (útil para requisitos regulatórios/latência) e por dispositivo/plataforma (ex.: liberar antes para Windows x64 e depois ARM). Requerem sistemas de targeting de flags ou de distribuição por canal, e observabilidade segmentada pela mesma dimensão usada no rollout.

### Deployment desacoplado de release
Princípio segundo o qual o código pode estar em produção (deployed) sem estar visível/ativo (released) — normalmente via feature flags. Isso permite testar em produção com risco controlado e separar o cronograma de engenharia do cronograma de produto/negócio.[^1]

## 4. Comparação Estruturada — Matriz

| Abordagem | Infra necessária | Rollback | Impacto BD | Impacto sessão/estado | Observabilidade mínima | Web | Desktop | Mobile | API | Plugin CAD |
|---|---|---|---|---|---|---|---|---|---|---|
| Big bang | Nenhuma extra | Difícil (restore) | Alto risco | Alto risco | Logs básicos | Sim | Sim | Sim | Sim | Sim |
| Recreate | Orquestrador simples | Redeploy versão anterior | Médio | Downtime elimina o problema | Health checks | Sim | N/A | N/A | Sim | N/A |
| Rolling | LB + health checks | Reverter rollout | Requer compatibilidade N/N-1 | Requer sessão distribuída | Métricas por instância | Sim | Parcial | N/A | Sim | N/A |
| Blue-green | Ambiente duplicado | Instantâneo (switch) | Requer replicação/CDC ou schema compatível | Requer réplica de estado | Métricas por ambiente | Sim | Parcial (auto-update com canal) | Parcial | Sim | Parcial |
| Canary | Roteamento por peso | Rápido (reduzir peso) | Requer compatibilidade de schema | Requer coexistência de versões | Métricas por versão em tempo real[^14] | Sim | Limitado | Sim | Sim | Limitado |
| Ring | Canais/anéis de usuários | Moderado (reverter anel) | Requer compatibilidade | Requer compatibilidade | Métricas por anel | Parcial | Sim | Sim | Parcial | Sim |
| Phased/percentage | Flags ou canais | Rápido | Depende do flag | Depende do flag | Métricas por %  | Sim | Sim | Sim | Sim | Sim |
| Dark launch/shadow | Mirror de tráfego | Trivial (não exposto) | Cuidado com escritas duplicadas | Nenhum (oculto) | Diffing de respostas[^3] | Sim | Limitado | Limitado | Sim | Limitado |
| A/B testing | Sistema de experimentação | Encerrar experimento | Neutro | Neutro | Métricas de negócio + estatística | Sim | Sim | Sim | Parcial | Limitado |
| Feature flag/kill switch | Serviço de flags | Instantâneo (toggle)[^17] | Neutro (se bem projetado) | Neutro | Alertas de toggle | Sim | Sim | Sim | Sim | Sim |
| Release por cliente/região/dispositivo | Sistema de targeting | Por segmento | Depende de multi-tenancy | Depende de isolamento | Métricas segmentadas | Sim | Sim | Sim | Sim | Sim |
| Deploy desacoplado de release | Feature flags | Toggle | Neutro | Neutro | Alertas de toggle | Sim | Sim | Sim | Sim | Sim |

## 5. Benefícios, Riscos e Limitações

Estratégias graduais (canary, ring, percentage rollout) reduzem o "blast radius" de defeitos, mas aumentam a complexidade operacional: exigem observabilidade por versão e podem manter múltiplas versões da aplicação acessando o mesmo banco simultaneamente, o que exige que o schema seja compatível com ambas (regra N/N-1). Blue-green oferece rollback extremamente rápido, mas seu custo de infraestrutura (ambiente duplicado) e sua dificuldade com bancos de dados stateful são limitações documentadas mesmo em serviços gerenciados de nuvem. Feature flags desacoplam risco de deploy do risco de release, mas a documentação de fornecedores adverte que flags sem data de expiração acumulam dívida técnica e tornam-se um risco de segurança e manutenção por si só.[^20][^21][^23][^18][^1]

## 6. Critérios para Escolher uma Abordagem

- Criticidade do sistema e tolerância a downtime: sistemas críticos favorecem blue-green ou canary com rollback rápido; sistemas internos de baixo risco podem usar recreate ou big bang.
- Natureza do estado (stateless vs. stateful): serviços stateless adotam blue-green/canary com facilidade; serviços com estado exigem estratégia de replicação de dados (CDC) ou compatibilidade de sessão.
- Tipo de cliente (web, API, desktop, mobile, plugin): entrega de desktop/plugin frequentemente usa canais de distribuição (ring deployment por canal beta/estável) em vez de controle de tráfego em tempo real, pois o binário é instalado localmente.
- Maturidade de observabilidade: canary análise automática exige métricas por versão em tempo real; sem essa capacidade, rollout percentual manual com checkpoints é mais seguro.
- Compatibilidade de schema entre versões: se o schema não suportar coexistência N/N-1, estratégias que mantêm duas versões simultâneas (rolling, canary, blue-green sem separação de banco) tornam-se arriscadas até que o expand-and-contract seja aplicado primeiro.[^23][^18]

## 7. Árvore de Decisão (Textual)

1. O sistema tolera downtime programado? Sim → recreate/big bang aceitável para simplicidade. Não → seguir.
2. Existe estado compartilhado entre versões (BD, sessão)? Não → blue-green ou canary direto. Sim → aplicar expand-and-contract antes de qualquer estratégia de múltiplas versões simultâneas.[^2][^3]
3. É possível medir saúde por versão em tempo real? Sim → canary com análise automática. Não → rollout percentual manual com portas de decisão humana.
4. O artefato é instalado localmente (desktop/plugin)? Sim → ring deployment por canal de distribuição + feature flags para funcionalidades internas. Não → prosseguir com canary/blue-green de infraestrutura.
5. A mudança é uma hipótese de produto (não apenas técnica)? Sim → combinar com A/B testing sobre a infraestrutura de flags já usada no rollout técnico.

## 8. Exemplo de Combinação Feature Flag + Canary

Um fluxo comum documentado na literatura de progressive delivery: a nova versão do serviço é implantada (deployment) atrás de uma feature flag desligada para 100% dos usuários; em seguida, o tráfego é roteado gradualmente para os novos pods via canary (por exemplo, 5%, 25%, 50%, 100%) enquanto a flag permanece controlando a visibilidade da funcionalidade dentro dessa fração de tráfego. Essa combinação separa dois riscos: o risco de infraestrutura (a nova versão do binário quebra) é mitigado pelo canary de tráfego; o risco de produto (a funcionalidade nova confunde ou prejudica usuários) é mitigado pela flag, que pode ser desligada instantaneamente sem reverter o deployment.[^14][^1]

## 9. Estratégia Mínima para Equipes Pequenas

Rolling deployment simples com health checks, um flag service leve (mesmo que seja uma tabela de configuração própria) para funcionalidades de risco, e rollback manual via redeploy da tag anterior. Não é necessário blue-green completo nem análise de canary automatizada quando a equipe não tem capacidade de operar dois ambientes ou pipelines de análise estatística — isso seria burocracia desproporcional ao risco.

## 10. Estratégia para Produtos Críticos

Canary automatizado com análise de métricas (canary analysis) integrada ao pipeline, blue-green para o ambiente de banco de dados quando aplicável, feature flags obrigatórios para toda funcionalidade nova com kill switch, e ring deployment para clientes desktop/plugin com canal beta antes do canal estável geral.

## 11. Falhas Comuns em Progressive Delivery

- Ignorar compatibilidade de schema entre versões coexistentes durante rollout gradual, causando erros silenciosos em uma fração do tráfego.[^18][^23]
- Deixar feature flags sem data de expiração, acumulando "dívida de flags" que dificulta entender o comportamento real do sistema.[^1]
- Tratar canary como suficiente sem observabilidade por versão, tornando impossível decidir se a nova versão está de fato saudável.
- Confundir experimento de produto (A/B testing) com estratégia de deploy (canary), aplicando critérios de decisão de negócio a um rollout técnico ou vice-versa.
- Realizar shadow traffic sem isolar efeitos colaterais de escrita, gerando dados duplicados ou inconsistentes.[^3]

***

# PESQUISA 10 — Build e Release de Software Desktop e Plugins CAD

**Declaração de dependência**: a query não especificou [STACK], [SISTEMA OPERACIONAL] (assume-se Windows dado o contexto ZWCAD/AutoCAD/SketchUp), [PRODUTOS CAD OU HOSTS SUPORTADOS] (assume-se AutoCAD/ZWCAD como referência por serem os mais documentados no ecossistema TotalCAD) e [FORMA DE DISTRIBUIÇÃO]. Os achados abaixo baseiam-se em documentação oficial da Autodesk (referência de mercado para plugins CAD) e da Microsoft, com cenários alternativos indicados.

## Matriz de Compatibilidade Plugin-Host (exemplo AutoCAD)

| Release AutoCAD | Compatibilidade binária | .NET exigido | Ferramentas |
|---|---|---|---|
| AutoCAD 2025 | Quebra compatibilidade com 2021–2024; recompilação obrigatória | .NET 8.0 (antes: .NET Framework 4.8) | VS 2022 17.10.4[^10] |
| AutoCAD 2026 | Compatível com 2025 e 2026 | .NET 8.0 | VS 2022 17.10.4[^24] |
| AutoCAD 2027 | Não compatível binariamente com 2025/2026; recompilação obrigatória | .NET 10.0 | VS 2026 v18.0[^8][^25] |

Esse padrão — quebra de compatibilidade binária a cada 1–2 anos, exigindo recompilação e atualização do target framework — é documentado oficialmente pela Autodesk e deve ser assumido como regra geral para hosts CAD proprietários com SDKs próprios (ObjectARX/.NET API), salvo declaração em contrário do fornecedor. Para ZWCAD, SketchUp, Enscape ou outros hosts do ecossistema do usuário, essa mesma matriz deve ser levantada separadamente na documentação oficial de cada fornecedor — não há evidência disponível nas fontes pesquisadas sobre a política exata de compatibilidade do ZWCAD ou SketchUp SDK, portanto essa lacuna é declarada explicitamente e não deve ser presumida.[^8][^24][^10]

## Fluxo Build → Instalação (Genérico Windows)

1. Compilação do plugin direcionado à versão(ões) do .NET/framework exigida(s) pelo host, com bibliotecas do host referenciadas como "Copy Local = False" (o host fornece os binários em runtime).[^8]
2. Empacotamento em um instalador (MSI, MSIX ou EXE bundler) que grava chaves de registro/manifests apontando o plugin para o host correto, e coloca os binários no caminho esperado pelo host (por usuário ou por máquina).
3. Assinatura de código do instalador e dos binários com certificado de assinatura de código válido, usando SignTool com timestamping (carimbo de tempo) para que a assinatura permaneça válida após a expiração do certificado.[^26][^27][^28]
4. Distribuição via canal apropriado (download direto, portal corporativo, ferramenta de gestão de patch) com suporte a instalação silenciosa para ambientes corporativos.
5. Registro do componente no host via `NETLOAD`/`APPLOAD` (ObjectARX/.NET) ou mecanismo equivalente do host, dependendo se o carregamento é automático (startup suite) ou manual.[^8]
6. Telemetria/crash reporting opcionalmente embutidos no plugin, respeitando privacidade e políticas corporativas de rede.

## Estratégia de Instalador

MSI oferece controle detalhado sobre chaves de registro, reparo e desinstalação, adequado a ambientes corporativos com ferramentas de gestão (GPO, SCCM/Intune). MSIX oferece isolamento e atualização automática nativa mas historicamente teve fricção com plugins que exigem escrita em pastas compartilhadas do sistema/registro global usadas por hosts CAD legados — a documentação da Microsoft evidencia que o MSIX via Microsoft Store oferece assinatura gratuita (o Store re-assina o pacote) e auto-update embutido, enquanto distribuição direta exige certificado próprio. EXE/ZIP bootstrapper oferecem flexibilidade máxima mas exigem que a lógica de instalação (registro, permissões, dependências nativas) seja implementada manualmente.[^29]

## Assinatura de Código, Certificados e Reputação

Desde 2023, a indústria de certificação (CA/Browser Forum) exige armazenamento da chave privada em token de hardware seguro para novos certificados de assinatura de código. O timestamping é essencial: sem ele, a assinatura é considerada inválida quando o certificado expira; com o timestamp, a validade da assinatura é preservada indefinidamente, pois comprova que a assinatura ocorreu antes da expiração. A reputação do instalador junto a antivírus/SmartScreen tende a ser prejudicada por instaladores não assinados ou assinados com certificados recém-emitidos e baixo volume de downloads — mitigação documentada por fornecedores de CA é a assinatura consistente ao longo do tempo e o uso de EV quando disponível, embora esse ponto seja majoritariamente experiência de indústria e não uma norma formal.[^30][^28][^31][^26]

## Checklist de Assinatura

- Certificado de assinatura de código válido, armazenado em token/HSM conforme exigência do CA/Browser Forum.[^30]
- Uso de SignTool com `/tr` (URL do timestamp server) e `/td SHA256 /fd SHA256`.[^27][^32]
- Verificação pós-assinatura com `signtool verify /pa /v`.[^30]
- Confirmação de timestamp presente antes da expiração do certificado (retimestamping é possível enquanto o certificado ainda é válido).[^30]
- Repetição do processo de assinatura para cada artefato (DLL, EXE, MSI/MSIX) distribuído.

## Riscos Específicos de Plugins CAD

- Quebra de compatibilidade binária anual do host, exigindo recompilação e testes de regressão a cada nova versão suportada.[^10][^8]
- Conflitos de carregamento de DLL quando múltiplas versões do plugin ou de dependências nativas (ex.: bibliotecas C++ redistribuíveis) coexistem na mesma máquina cliente.
- Instalação por usuário vs. por máquina gerando comportamento inconsistente em ambientes multiusuário/RDS, comum em escritórios de arquitetura/engenharia.
- Falta de suporte a arquitetura ARM em hosts CAD legados, relevante à medida que clientes migram para hardware ARM.
- Perda de configurações/licenças durante atualização ou downgrade se o instalador não preservar corretamente os dados de usuário.

## Perguntas para um Desenvolvedor Sênior da TotalCAD

- Qual é a matriz oficial de compatibilidade binária ZWCAD/SketchUp/Enscape versão a versão, e como ela é rastreada internamente?
- O plugin precisa suportar instalação silenciosa para clientes corporativos com ferramentas de gestão de patch (SCCM/Intune)?
- Existe processo de assinatura de código automatizado no pipeline de release, com timestamping garantido?
- Como é tratada a coexistência de múltiplas versões do host na mesma máquina cliente (ex.: AutoCAD 2025 e 2026 simultâneos)?
- Qual é a política de suporte a versões antigas do host — quantas versões anteriores são mantidas ativamente?

***

# PESQUISA 11 — Migrações de Banco, Schema e Dados

## Fluxo Seguro (Expand-Migrate-Contract)

O padrão consensual, documentado por múltiplas fontes técnicas independentes, envolve três fases distintas em deploys separados: (1) **Expand** — adicionar a nova estrutura (colunas, tabelas, índices) sem remover nada, mantendo total compatibilidade com o código antigo; (2) **Migrate/backfill** — o código passa a escrever em ambas as estruturas (dual write) ou ler preferencialmente da nova com fallback para a antiga, enquanto dados históricos são copiados em lotes (batches) com throttling; (3) **Contract** — após confirmação de que nenhuma versão em produção depende da estrutura antiga, ela é removida. A regra geral é nunca combinar expand e contract no mesmo release, mesmo que os testes passem, pois isso remove a janela de segurança para rollback.[^33][^34][^22][^21][^5][^2][^3]

## Exemplo de Expand-Migrate-Contract

1. Deploy 1 (Expand): adicionar coluna nova nullable + índice criado concorrentemente (ex.: `CREATE INDEX CONCURRENTLY` em PostgreSQL para não bloquear escritas).[^33]
2. Deploy 2 (Migrate): código passa a escrever nas duas colunas (dual write) e lê da nova coluna com fallback para a antiga; backfill em lotes por chave, com checkpoints e taxa controlada.[^22][^23]
3. Verificação: comparação de leitura (shadow read) entre coluna antiga e nova em amostra de tráfego, monitorando divergências.[^23]
4. Deploy 3 (Switch): leitura passa a ser exclusivamente da nova coluna; escrita na antiga é interrompida.
5. Deploy 4 (Contract): remoção da coluna antiga, aplicada somente após uma janela de rollback (frequentemente uma semana ou mais, cobrindo um ciclo completo de deploy).[^23]

## Migração de Dados de Longa Duração

Para volumes grandes (documentado como regra prática: acima de ~1 milhão de linhas requer backfill em lote obrigatório), o backfill deve ser restartável, paginado por chave estável (não por offset), com throttling para não saturar I/O nem causar lag de replicação, e idempotente (uso de checks `IF EXISTS`).[^21][^35][^23]

## Matriz de Riscos

| Risco | Fase mais exposta | Mitigação |
|---|---|---|
| Lock de tabela bloqueando escritas | Expand (DDL) | `NOT VALID` + validação separada, índices concorrentes[^33] |
| Divergência entre dados antigos e novos | Migrate | Shadow reads com diffing antes de switch[^23] |
| Rollback de código após mudança de schema quebrar versão anterior | Qualquer fase se regra N/N-1 for violada | Testar explicitamente app N-1 contra schema novo[^21][^36] |
| Perda de dados em rollback destrutivo | Contract | Backup/snapshot antes de qualquer DDL destrutivo[^21] |
| Múltiplas versões da app lendo/escrevendo simultaneamente de forma incompatível | Rolling/canary com schema mutável | Manter compatibilidade dupla até 100% do fleet estar na nova versão[^18] |

## Critérios para Permitir ou Bloquear Rollback

Rollback de código é seguro enquanto a fase for "expand" ou "dual write" (o dado antigo ainda existe e é atualizado). Rollback torna-se perigoso ou impossível assim que a fase "contract" remove a estrutura antiga — a partir desse ponto, a filosofia recomendada por múltiplas fontes é "roll-forward", não rollback.[^35][^21]

## Checklist Antes/Durante/Depois

- **Antes**: inventariar todos os leitores/escritores do schema afetado (APIs, jobs, relatórios, integrações externas); confirmar backup/snapshot; validar que app N-1 funciona contra o schema alvo.[^34][^21]
- **Durante**: aplicar DDL com `lock_timeout` curto; monitorar lock wait time e replicação; executar backfill em lotes throttled com checkpoints.[^33][^23]
- **Depois**: validar paridade de dados (shadow read); manter estrutura antiga por uma janela de rollback definida antes do contract; documentar e arquivar histórico da migração.[^34][^23]

## Antipadrões que Podem Causar Perda de Dados

- Rodar "down scripts" de rollback automáticos sem testar se eles preservam dados históricos, podendo apagar dados criados após a migração.[^36]
- Combinar expand e contract em um único deploy porque os testes locais passaram.[^34]
- Ignorar a regra N/N-1, permitindo que uma versão antiga da aplicação tente ler colunas já removidas.
- Executar UPDATE massivo em uma única transação sem lotes, causando bloqueios longos ou timeouts que podem corromper estado parcial.

***

# PESQUISA 12 — Rollback, Roll-Forward, Observabilidade e Incidentes

## Árvore de Decisão Rollback vs. Roll-Forward

1. A mudança envolveu apenas código, sem alteração destrutiva de schema, formato de evento ou dado já persistido pela nova versão? Sim → rollback de código é seguro.
2. Houve migração destrutiva de schema (fase "contract" já aplicada)? Sim → rollback de código pode quebrar; priorizar roll-forward com hotfix.[^21][^35]
3. A nova versão já publicou eventos/mensagens em novo formato para sistemas externos ou filas assíncronas? Sim → reverter o produtor pode deixar consumidores incompatíveis; avaliar compatibilidade de formato antes de decidir.
4. Clientes desktop já atualizaram automaticamente e passaram a gravar arquivos/dados em novo formato local? Sim → rollback do servidor não resolve; requer estratégia de compatibilidade retroativa no formato de dados do cliente.

## Quando Rollback é Perigoso

Rollback é especialmente arriscado na presença de: migrações de banco já contraídas (dados antigos removidos); mudanças de formato de dados já persistidos pela nova versão; eventos já publicados em filas/tópicos consumidos por outros serviços; integrações externas que já processaram dados no novo formato; clientes desktop que já se atualizaram automaticamente; e dados produzidos exclusivamente pela nova versão que a versão antiga não sabe interpretar. Em todos esses casos, a literatura tende a recomendar "forward-only" — corrigir com um hotfix roll-forward em vez de reverter.[^35][^21]

## Checklist de Observabilidade Antes da Release

- Health checks de readiness/liveness configurados e testados.
- Dashboards de métricas por versão (deployment markers correlacionando versão a taxa de erro).
- Alertas configurados com thresholds definidos antes do rollout, não ajustados durante.
- Logging estruturado permitindo correlação entre versão, request e erro.

## Checklist dos Primeiros Minutos/Horas/Dias

- **Minutos**: monitorar taxa de erro, latência e health checks das primeiras instâncias/canary; comparar com baseline.
- **Horas**: acompanhar métricas de negócio afetadas pela funcionalidade nova; validar ausência de vazamento de memória ou degradação gradual.
- **Dias**: revisar métricas de adoção, crash-free sessions (para desktop/mobile), e taxa de rollback/hotfix associada à versão.

## Runbook de Falha de Release (Esqueleto)

1. Detectar (alerta automático ou canary analysis reprovado).
2. Interromper o rollout imediatamente (pausar percentual, não avançar).
3. Classificar severidade e decidir rollback vs. roll-forward conforme árvore de decisão acima.
4. Executar ação escolhida e validar recuperação via health checks/métricas.
5. Comunicar status a stakeholders/usuários conforme severidade.
6. Abrir post-mortem sem culpabilização dentro do prazo definido pela organização.

## Modelo de Post-Mortem Sem Culpabilização (Esqueleto)

Título e data do incidente; linha do tempo objetiva de eventos; impacto (usuários afetados, duração, métricas de negócio); causa raiz técnica (não pessoal); fatores contribuintes; o que funcionou bem na resposta; ações de acompanhamento com responsável e prazo — sem atribuir culpa individual, seguindo prática consensual de post-mortems blameless da indústria de SRE.

## Métricas que Indicam Sucesso ou Fracasso da Versão

Taxa de erro pós-release comparada ao baseline, crash-free sessions (aplicável a desktop/mobile), tempo até detecção de falha, taxa de rollback/hotfix associada à versão, e adoção da versão nova pelos usuários ao longo do tempo.

***

# PESQUISA 13 — Segurança da Cadeia de Suprimentos de Software

## Padrões e Referências Oficiais

O NIST SP 800-218 (SSDF) é a referência governamental primária para práticas seguras ao longo do ciclo de desenvolvimento, cobrindo governança (PO), design (PW) e integridade de release (PS), incluindo a exigência de coletar e proteger dados de proveniência (ex.: SBOM) para cada release. O SLSA é um framework — não uma norma governamental — que define níveis crescentes de garantia sobre a integridade do processo de build, culminando em proveniência assinada criptograficamente e gerada por plataformas de build isoladas e hardened. É importante notar que boa parte do conteúdo detalhado sobre SLSA disponível publicamente vem de blogs técnicos e fornecedores de segurança explicando a especificação, não da especificação oficial em si — a fonte primária deve ser consultada em slsa.dev para os requisitos formais exatos.[^4][^37][^11]

## Fluxo Seguro Commit → Pipeline → Artefato → Distribuição

1. Commit assinado, branch protegida exigindo revisão (segregation of duties).
2. Pipeline executado em runner isolado/efêmero, com credenciais de curta duração via OIDC (evitando segredos estáticos de longa duração).
3. Build gera SBOM (formatos SPDX ou CycloneDX) e proveniência assinada (ex.: via Sigstore/Cosign).
4. Artefato assinado com certificado de code signing e timestamp antes de publicação.
5. Distribuição para consumidores que verificam assinatura e proveniência antes de instalar/executar.

## Controles Preventivos, Detectivos e Corretivos

| Categoria | Controles |
|---|---|
| Preventivo | Lockfiles, revisão de dependências, proteção de branch, least privilege, runners efêmeros, OIDC em vez de segredos estáticos |
| Detectivo | SCA (análise de composição de software), SAST, secret scanning, monitoramento de CVEs |
| Corretivo | Rotação de credenciais, revogação de releases comprometidas, patch de dependência vulnerável, resposta a CVE documentada |

## Estratégia Mínima para Equipe Pequena

Lockfiles obrigatórios, scanning de dependências automatizado no PR (SCA), proteção de branch principal, e assinatura de código para artefatos distribuídos — sem exigir SLSA nível 3 completo, que demanda infraestrutura de build hardened dedicada, desproporcional a times pequenos.

## Estratégia para Produto Empresarial

Geração de SBOM e proveniência assinada em cada release (alinhado a SLSA nível 2-3), secret scanning contínuo, runners efêmeros e isolados, revisão obrigatória de dependências novas, e processo formal de resposta a CVE com capacidade de revogar releases comprometidas.

## Checklist de Segurança de Pipeline

- Branch protection com revisão obrigatória.
- Runners efêmeros e isolados por execução.
- Credenciais via OIDC/least privilege, sem segredos de longa duração no pipeline.
- Scanning automático de dependências (SCA) e segredos (secret scanning) em cada PR.

## Checklist de Segurança de Artefato

- SBOM gerado e anexado ao release.
- Proveniência de build assinada e verificável.
- Artefato assinado com certificado de code signing válido e timestampado.[^26]
- Checksums publicados para verificação de integridade.

## Checklist de Resposta a Dependência Comprometida

- Identificar todos os artefatos que consomem a dependência afetada (via SBOM).
- Avaliar exploitabilidade real (VEX) antes de tratar toda CVE como crítica.
- Aplicar patch ou pin de versão segura, revalidar build.
- Revogar/republicar releases já distribuídas se a exploração for confirmada em produção.

***

# PESQUISA 14 — Governança, Papéis e Processo de Release

## Modelo Leve (Equipe Pequena)

Autor do PR e revisor único aprovam a mudança técnica; não há CAB formal; o "go/no-go" é uma decisão informal do tech lead baseada em checklist mínimo (testes passando, sem alertas críticos ativos). Processo proporcional ao baixo risco de mudanças pequenas e frequentes.

## Modelo Intermediário

Inclui QA como gate adicional antes de produção, um release manager informal que coordena o calendário de releases (release train com cadência fixa), e comunicação obrigatória de mudanças relevantes ao suporte antes do deploy.

## Modelo para Sistema Crítico ou Regulado

Change Advisory Board (CAB) ou equivalente avalia mudanças de maior risco; aprovação técnica (arquitetura/segurança) e aprovação de produto/negócio são segregadas; auditoria e registro formal de decisões são obrigatórios; existe processo de exceção documentado para mudanças emergenciais.

## Matriz RACI (Exemplo Simplificado)

| Atividade | Dev | Revisor | QA | Tech Lead | SRE/Ops | Release Manager | Product |
|---|---|---|---|---|---|---|---|
| Codificar mudança | R | C | I | A | I | I | I |
| Revisar PR | I | R | I | A | I | I | I |
| Aprovar go/no-go | I | I | C | A | C | R | C |
| Executar release | I | I | I | C | R | A | I |
| Monitorar pós-release | C | I | I | C | R | A | I |

(R=Responsável, A=Aprovador, C=Consultado, I=Informado)

## Antipadrões de Governança

Adicionar aprovações sequenciais sem critério de risco (mesma burocracia para hotfix trivial e mudança estrutural de banco); CAB que aprova sem entender detalhes técnicos, tornando-se apenas um gate cerimonial; ausência de processo de exceção, forçando burlar o processo formal em emergências reais, o que corrói a confiança no processo.

***

# PESQUISA 15 — Métricas de Build, Release e Maturidade de Engenharia

## Métricas DORA — Metodologia e Limitações

As quatro métricas DORA (deployment frequency, lead time for changes, change failure rate, failed deployment recovery time) originam-se da pesquisa anual "Accelerate"/State of DevOps, conduzida via survey com autoatribuição pelos respondentes, não medição direta instrumentada — uma limitação metodológica relevante para interpretação de benchmarks. Elite performers reportam lead time inferior a um dia e change failure rate próximo de 5%, enquanto baixo desempenho reporta ciclos de um a seis meses. Essas correlações não implicam causalidade garantida entre adoção de práticas específicas e esses resultados — a pesquisa é observacional e sujeita a viés de autosseleção dos respondentes.[^6][^7]

## Catálogo de Métricas (Resumo)

| Métrica | Fórmula/Método | Fonte de dados | Risco de interpretação |
|---|---|---|---|
| Deployment frequency | Contagem de deploys/janela de tempo | Pipeline CI/CD | Otimizar só a frequência sem qualidade gera Goodhart's Law |
| Lead time for changes | Tempo do commit ao deploy em produção | VCS + pipeline | Pode mascarar filas ocultas antes do commit |
| Change failure rate | % de deploys que geram incidente/rollback | Incident tracker + deploy log | Definição de "falha" varia entre equipes |
| Recovery time | Tempo entre detecção e restauração | Observabilidade + incident tracker | Depende de qualidade de detecção, não só de resposta |
| Crash-free sessions | % sessões sem crash | Crash reporting (desktop/mobile) | Sensível a amostragem de telemetria |
| Flaky test rate | % de testes com resultado inconsistente | CI | Ignorar pode mascarar falhas reais |

## Riscos de Goodhart's Law

Otimizar métricas isoladamente (ex.: aumentar deployment frequency sem controle de change failure rate) pode incentivar deploys triviais ou fragmentação artificial de mudanças apenas para melhorar o número, sem ganho real de valor ou segurança — um risco amplamente reconhecido na literatura de métricas de engenharia e explicitamente citado como armadilha a evitar.[^7]

## Modelo de Maturidade (Cinco Níveis, Síntese Baseada em DORA)

1. Ad hoc: deploys manuais, sem métricas, alta taxa de falha.
2. Repetível: pipeline básico de CI, deploys frequentes mas sem observabilidade por versão.
3. Definido: métricas DORA coletadas, rollback documentado, testes automatizados consistentes.
4. Gerenciado: canary/feature flags padronizados, SLO definidos, observabilidade correlacionando versão e erro.
5. Otimizado: análise de canary automatizada, SBOM/proveniência de build, cultura de post-mortem blameless madura, métricas usadas para decisão, não para avaliação punitiva de indivíduos.

## Indicadores de Burocracia sem Ganho de Segurança/Velocidade

Aumento do número de aprovações sem redução mensurável de change failure rate; lead time crescente sem correlação com queda de incidentes; equipes contornando o processo formal em emergências (sinal de que o processo não serve à realidade operacional).

***

# PESQUISA 16 — Síntese Aplicada: Modelo Operacional

**Declaração de dependência crítica**: os campos [CONTEXTO DA EMPRESA], [TIPO DE PRODUTO], [STACK], [SISTEMA OPERACIONAL], [FERRAMENTA DE CI/CD], [REPOSITÓRIO], [FORMA DE DISTRIBUIÇÃO], [QUANTIDADE DE DESENVOLVEDORES], [FREQUÊNCIA], [CRITICIDADE] e [NECESSIDADE DE MANTER VERSÕES ANTERIORES] não foram preenchidos explicitamente na consulta. Com base no perfil profissional fornecido (TotalCAD, suporte a ZWCAD/SketchUp/Enscape/plugins CAD, uso de Google Cloud/Vertex AI/Supabase/Firebase), este modelo apresenta **dois cenários hipotéticos declarados como hipóteses, não fatos**, para ilustrar como o modelo se adapta.

## Cenário A — Hipótese: Plugin CAD Desktop para Clientes Externos

Suposições declaradas: produto = plugin/extensão Windows para hosts CAD (AutoCAD/ZWCAD/SketchUp); equipe pequena (poucos desenvolvedores); criticidade média (afeta produtividade do cliente, não vidas); clientes externos; necessidade de suportar múltiplas versões do host simultaneamente = sim (dado o padrão da indústria CAD).

### Diagnóstico das Necessidades
Necessidade central: gerenciar a matriz de compatibilidade plugin-host, assinatura de código confiável, e um canal de distribuição/atualização que minimize atrito para clientes não técnicos.

### Fluxo Completo (Ideia → Operação)
Ideia/ticket → branch de feature → PR com revisão → build automatizado contra a matriz de versões do host suportadas → assinatura de código → testes de instalação/atualização/downgrade → publicação em canal beta (ring 1) → validação → promoção a canal estável (ring 2) → monitoramento de crash reporting → suporte via ticket/WhatsApp.

### Estrutura de Repositório (Sugerida)
Monorepo por produto, com pastas separadas por versão-alvo do host quando a API diverge significativamente; pipeline de build parametrizado pela versão do host.

### Política de Versionamento
Versionamento semântico do plugin, desacoplado da numeração de versão do host, com uma tabela de compatibilidade explícita documentada e testada em CI.

### Estratégia de Rollout
Ring deployment por canal (interno → beta → estável), combinado com feature flags internas para funcionalidades experimentais dentro do próprio plugin, dado que canary de tráfego em tempo real não se aplica a software instalado localmente.

### Estratégia de Rollback/Hotfix
Como o software é instalado localmente, "rollback" significa fornecer instalador da versão anterior e permitir downgrade controlado, preservando configurações/licenças do cliente — não reversão de tráfego de servidor.

### Observabilidade Pós-Release
Crash reporting embutido, telemetria de versão em uso por cliente, e correlação de tickets de suporte com versão do plugin instalada.

### Roadmap de Adoção
- 30 dias: implementar assinatura de código com timestamping e checklist de instalação/desinstalação testado manualmente.
- 60 dias: automatizar a matriz de compatibilidade no pipeline de build e introduzir canal beta formal.
- 90 dias: telemetria básica de crash/versão em produção.
- Evolução posterior: canary de canal com métricas automatizadas de adoção e taxa de erro por versão.

## Cenário B — Hipótese: Serviço Cloud/API (Vertex AI, Supabase, Firebase)

Suposições declaradas: produto = API/serviço backend; stack = Node/Python + Google Cloud; criticidade média-alta (integra com produtos pagos); clientes internos e externos.

### Estratégia de Rollout
Canary de tráfego com análise automatizada de métricas de erro, combinado a feature flags para funcionalidades novas de IA (ex.: integração Nano Banana/Vertex AI), permitindo desligar rapidamente uma integração problemática sem rollback completo do serviço.

### Estratégia de Migração de Dados
Expand-and-contract obrigatório para qualquer mudança de schema em Supabase/Firebase compartilhado entre múltiplas versões da API em rollout gradual.

### Observabilidade
Correlação de deployment markers com métricas de erro por versão, SLOs para latência de chamadas a APIs de IA externas (rate limiting/quota da Vertex AI como risco adicional a monitorar).

## Decisões que Ainda Precisam Ser Tomadas

- Confirmar qual ferramenta de CI/CD será usada oficialmente (GitHub Actions, outra) e se já existe pipeline de assinatura de código configurado.
- Definir explicitamente quantas versões anteriores de cada host CAD serão suportadas simultaneamente.
- Definir critérios objetivos de criticidade por produto (plugin vs. serviço cloud) para calibrar o nível de governança aplicável.
- Decidir se a filosofia de rollback será "sempre roll-forward" ou se rollback de código será permitido em quais condições exatas.

## Perguntas ao Desenvolvedor Sênior da TotalCAD

- Qual é o apetite de risco real da organização para adotar canary automatizado versus rollout percentual manual?
- Existe hoje algum SBOM ou inventário de dependências de terceiros nos plugins e serviços cloud?
- Como são tratadas atualmente licenças e configurações do cliente durante downgrade/desinstalação de plugins?
- Qual ferramenta de feature flag (se alguma) já está em uso ou é candidata, e ela oferece avaliação server-side?

## Glossário Final
Ver seção 2 (Definições e Terminologia) para o glossário consolidado de todas as pesquisas.

***

## Tabela de Fontes

| Título | Organização/Autores | Data | Tipo | Autoridade | Contribuição principal | Limitações |
|---|---|---|---|---|---|---|
| Blue Green Deployment | Martin Fowler | 2010 | Blog técnico de referência | Alta (referência histórica no setor) | Definição original do padrão blue-green | Não é norma oficial; ilustrativo |
| Canary Release | Martin Fowler | 2014/2021 | Blog técnico de referência | Alta | Definição de canary release | Conteúdo é "stub", conciso |
| Expand and Contract - A Pattern | Tim Wellhausen | s/d | Artigo técnico | Média-alta | Definição formal do padrão expand/contract | Página pessoal, não institucional |
| Prisma Data Guide — Expand and Contract | Prisma | s/d | Documentação de fornecedor | Média (documentação técnica de produto) | Passo a passo detalhado do padrão | Foco em produtos Prisma |
| AutoCAD 2027 SDK | Autodesk (blog.autodesk.io) | 2026 | Documentação oficial de fornecedor | Alta (fonte primária do fabricante) | Matriz de compatibilidade binária | Específico à Autodesk, não generalizável a ZWCAD/SketchUp |
| AutoCAD 2025 Developer/.NET Migration Guide | Autodesk (help.autodesk.com) | 2025 | Documentação oficial | Alta | Confirma quebra de compatibilidade 2025 | Idem |
| Time Stamping Authenticode Signatures | Microsoft Learn | 2022 | Documentação oficial | Alta | Mecanismo técnico de timestamping | Não cobre políticas de CA de terceiros |
| SignTool.exe | Microsoft Learn | 2023 | Documentação oficial | Alta | Referência de comando de assinatura | — |
| NIST SP 800-218 (SSDF v1.1) | NIST | 2022 | Norma governamental | Muito alta | Framework oficial de segurança no SDLC | Foco em práticas, não prescreve ferramentas |
| NIST SP 800-218 rev.1 (SSDF v1.2, draft) | NIST | 2025 | Norma governamental (rascunho) | Muito alta | Atualização de práticas | Ainda em processo de comentário público até jan/2026 |
| SLSA Framework Explained | secportal.io | 2026 | Publicação de fornecedor/blog de segurança | Média | Explicação prática de níveis SLSA | Não é a especificação oficial primária (slsa.dev) |
| DORA / Accelerate 2024 report | DORA/Google | 2024 | Relatório de pesquisa de indústria | Alta (mas survey autoatribuído) | Origem das métricas DORA e benchmarks | Metodologia de survey, viés de autosseleção |
| DORA Metrics Explained 2026 | Taskade | 2026 | Blog explicativo | Baixa-média | Síntese acessível dos benchmarks DORA | Fonte secundária, não original |
| Kill switch flags | LaunchDarkly | s/d | Documentação de fornecedor | Média (documentação de produto, viés comercial) | Definição prática de kill switch | Específico ao produto LaunchDarkly |
| Feature Flags & Progressive Delivery | anhtu.dev | 2026 | Blog técnico | Baixa-média | Boas práticas de flag naming/expiração | Blog pessoal, não institucional |
| Zero-Downtime Database Migrations (múltiplas fontes cruzadas) | Diversos autores técnicos independentes | 2025-2026 | Blogs técnicos | Média | Corroboração cruzada do padrão expand/contract | Nenhuma é norma oficial; convergência de múltiplas fontes independentes reforça confiabilidade do consenso |
| MSIX vs distribuição direta | Microsoft (windows-dev-docs, GitHub) | 2017 (atualizado) | Documentação oficial | Alta | Comparação de caminhos de distribuição Windows | Pode estar desatualizada em detalhes específicos |

---

## References

1. [Feature Flags & Progressive Delivery: Safe Release Strategy ...](https://anhtu.dev/feature-flags-progressive-delivery-safe-release-strategy-2026-2164) - Deep dive into Feature Flags and Progressive Delivery — reducing production incidents by 73%. Rollou...

2. [Zero-Downtime Migrations: Expand and Contract Explained](https://keninkujovic.com/blog/expand-and-contract-migrations) - How to ship schema changes with zero downtime using the expand and contract pattern. A practical wal...

3. [Using the expand and contract pattern | Prisma's Data Guide](https://www.prisma.io/dataguide/types/relational/expand-and-contract-pattern) - In this article, we introduce the expand and contract pattern to help migrate data and clients to a ...

4. [SLSA Framework Explained: Build Provenance, Levels, and ...](https://secportal.io/blog/slsa-framework-explained) - A practical SLSA guide for AppSec, product security, and GRC teams. Covers SLSA levels, build proven...

5. [Expand and Contract - A Pattern to Apply Breaking ...](https://www.tim-wellhausen.de/papers/ExpandAndContract/ExpandAndContract.html) - The Expand and Contract pattern provides a way to implement breaking changes to a system in a safe m...

6. [Accelerate](https://services.google.com/fh/files/misc/2024_final_dora_report.pdf)

7. [DORA Metrics Explained (2026): Four Keys & Benchmarks](https://www.taskade.com/blog/dora-metrics-explained)

8. [AutoCAD 2027 SDK: What Every Plugin Developer Needs to ...](https://blog.autodesk.io/autocad-2027-sdk-what-every-plugin-developer-needs-to-know/) - Discover the key updates in AutoCAD 2027's ObjectARX SDK, including compatibility, development envir...

9. [Secure Software Development Framework (SSDF) Version 1.2](https://csrc.nist.gov/pubs/sp/800/218/r1/ipd) - Few software development life cycle (SDLC) models explicitly address software security in detail, so...

10. [AutoCAD 2025 Developer and ObjectARX Help | .NET Migration ...](https://help.autodesk.com/view/OARX/2025/ENU/?guid=OARX-ManagedRefGuide-_NET_Migration_Guide)

11. [NIST Special Publication (SP) 800-218, Secure Software Development Framework (SSDF) Version 1.1: Recommendations for Mitigating the Risk of Software Vulnerabilities](https://csrc.nist.gov/pubs/sp/800/218/final) - Few software development life cycle (SDLC) models explicitly address software security in detail, so...

12. [Blue Green Deployment](https://www.martinfowler.com/bliki/BlueGreenDeployment.html) - Blue-green deployment allows you to upgrade production software without downtime. You deploy the new...

13. [Blue Green Deployments - Open Practice Library](https://openpracticelibrary.com/practice/blue-green-deployments/) - Practices that empower teams to collaborate and deliver iteratively

14. [Canary Release - Martin Fowler](https://www.martinfowler.com/bliki/CanaryRelease.html) - A canary release occurs when you roll out a new version of some software to a small subset of your u...

15. [Canary Release](https://martinfowler.com/articles/patterns-legacy-displacement/canary-release.html) - The canary release technique can be especially useful for legacy replacement as it allows a gradual ...

16. [Creating A Feature Flag](https://launchdarkly.com/blog/mitigate-risk-with-kill-swith-flags-in-python-launchdarkly/) - One of the best ways to keep innovating with speed while mitigating risk is by implementing feature ...

17. [Kill switch flags](https://launchdarkly.com/docs/home/flags/killswitch) - This topic explains how you can use kill switch or circuit breaker flags to shut off functionality.

18. [Database Migrations. The Expand-Contract Pattern](https://www.enolcasielles.com/en/blog/database-migrations-strategy) - In this article I share my experience on how I manage database migrations in high-availability envir...

19. [Blue-green Deployments, A/B Testing, and Canary Releases - Java Code Geeks](https://www.javacodegeeks.com/2015/08/blue-green-deployments-ab-testing-and-canary-releases.html) - A lot of teams I talk to recently are very interested in “DevOps” (whatever that means… seems to mea...

20. [Blue-green deployments | Postgres.FM 071 | #PostgreSQL #Postgres podcast](https://www.youtube.com/watch?v=0Pa1wv4Oilo) - [ 🇬🇧_🇺🇸 Check out the subtitles – we now edit them, ChatGPT+manually! You can also try YouTube's aut...

21. [Zero-Downtime Database Migration: Expand-Contract, EF Core ...](https://anhtu.dev/zero-downtime-database-migration-expand-contract-ef-core-batch-backfill-production-1151) - Zero-downtime database migration strategies for .NET 10 and SQL Server: Expand-Contract pattern, EF ...

22. [Zero-Downtime Database Migrations: Expand/Contract, Triggers, and ...](https://thebackenddevelopers.substack.com/p/zero-downtime-database-migrations) - The Day I Learned That “ALTER TABLE” Is a Four-Letter Word

23. [Zero-Downtime Database Migrations: The Expand/Contract ...](https://palakorn.com/blog/zero-downtime-database-migrations/) - How to change a production schema without taking traffic down — expand/contract, backfills, dual-wri...

24. [AutoCAD 2026 Developer and ObjectARX](https://help.autodesk.com/cloudhelp/2026/PTB/OARX-DevGuide-Managed/files/GUID-4E1AAFA9-740E-4097-800C-CAED09CDFF12.htm)

25. [Informazioni sulla compatibilità delle applicazioni | Autodesk](https://help.autodesk.com/cloudhelp/2027/ITA/AutoCAD-Customization/files/GUID-D54B0935-1638-4F97-8B37-1EC3635A1E71.htm) - Ogni volta che viene rilasciata una nuova release dei prodotti basati su AutoCAD, verificare che le ...

26. [Time Stamping Authenticode Signatures - Win32 apps](https://learn.microsoft.com/en-us/windows/win32/seccrypto/time-stamping-authenticode-signatures) - Time stamping allows Authenticode signatures to be verifiable even after the certificates used for s...

27. [SignTool.exe (Sign Tool) - .NET Framework](https://learn.microsoft.com/en-us/dotnet/framework/tools/signtool-exe) - Learn about SignTool.exe, the Sign Tool. This command-line tool digitally signs files, verifies sign...

28. [How can I be sure my Windows desktop application is signed with a timestamp? - Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/1684036/how-can-i-be-sure-my-windows-desktop-application-i) - We use DigiCert to create certificates for our Windows desktop apps. The certificate expires in June...

29. [Msix Direct Download -- Isv...](https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/package-and-deploy/choose-distribution-path.md) - Conceptual and overview content for developing Windows apps - MicrosoftDocs/windows-dev-docs

30. [The Ultimate Guide to Microsoft SignTool - Code Signing Store](https://codesigningstore.com/the-ultimate-guide-to-microsoft-signtool) - Explore the ins and outs of Microsoft SignTool. Discover advanced tricks to help you sign Windows ap...

31. [About Code Signing and Timestamps | Microsoft Community Hub](https://techcommunity.microsoft.com/discussions/msix-discussions/about-code-signing-and-timestamps/382906) - Code Singing certificates may include a url for a timestampling service. This is known to be used by...

32. [Code Signing in Windows Using Microsoft Signtool](https://support.globalsign.com/code-signing/code-signing/code-signing-windows-7-8-and-10)

33. [Zero-downtime database migrations: the expand-contract ...](https://matthewpalma.dev/blog/zero-downtime-database-migrations-expand-contract-pattern) - Ship relational schema changes without maintenance windows using expand-contract phases, backfills, ...

34. [Expand and Contract Database Migrations in... | Edilec Blog](https://edilec.com/blog/clodev-11028/database-changes-continuous-delivery-expand-migrate-contract/) - Ship backward-compatible schema changes through expand, migrate, verify, switch, and contract phases...

35. [Expand and Contract Schema Migration](https://www.zero-downtime-schema.com/zero-downtime-schema-evolution-patterns/expand-and-contract-methodology/) - Rename, split, or drop columns on live high-traffic tables using the expand-migrate-contract pattern...

36. [Database Migration in CI/CD -  Expand & Contract Pattern](https://www.youtube.com/watch?v=gpLy95UZTIU) - 🚨 Are your database migrations crashing production or wiping out historical data during rollbacks? I...

37. [SSDF](https://csrc.nist.gov/files/pubs/sp/800/218/final/docs/nist.sp.800-218.ssdf-table.xlsx)

