# Modelo operacional de build, release, segurança e governança para equipe de engenharia

## 1. Premissas e lacunas de contexto

- Os relatórios consultados tratam majoritariamente de segurança de cadeia de suprimentos (NIST SSDF, NIST SP 800-161r1, SP 800-204D, guias de SLSA, melhores práticas de CI/CD, controle de artefatos e SBOM).[^1][^2][^3][^4][^5][^6][^7]
- Não há informações específicas sobre: tipo de produto, stack tecnológica, sistema operacional alvo, ferramenta de CI/CD, sistema de repositório, forma de distribuição, tamanho de equipe, frequência de release, criticidade, perfil de clientes, necessidade de manter versões antigas, restrições de instalação ou requisitos de compliance. Essas são lacunas explícitas que impactam decisões de arquitetura de build, segurança e rollout.
- O modelo proposto é, portanto, parametrizável: para cada decisão crítica são apresentados cenários alternativos, dependências de contexto e perguntas para refinar com um desenvolvedor sênior.

## 2. Resumo executivo

- A proposta organiza o ciclo completo da ideia à operação em torno de um pipeline CI/CD automatizado, com foco em integridade de artefatos, versionamento disciplinado, governança de mudanças e segurança de cadeia de suprimentos (inspirado em NIST SSDF, NIST SP 800-161r1 e SLSA nível 1–2).[^3][^5][^6][^7]
- O modelo mínimo viável prioriza: uma política simples de branches, build reproduzível em ambiente efêmero de CI, geração e retenção de artefatos com versionamento claro, checklist de release, rollback rápido (via flags ou reimplantação) e observabilidade básica (logs estruturados, métricas de erro e tempo de resposta).[^2][^8][^9][^3]
- O modelo futuro eleva maturidade em: segurança da cadeia (SBOM, provenance, assinatura de artefatos, atestação), ambientes e anéis de rollout, governança formal de mudanças, métricas de fluxo e qualidade, políticas de acesso com MFA e princípio de menor privilégio em toda a toolchain.[^4][^5][^6][^7][^1][^2][^3]
- As recomendações são compatíveis tanto com aplicações web/SaaS quanto com desktop ou plugins, desde que sejam adaptadas quanto a forma de distribuição (instalador, marketplace, container registry, etc.) e janelas de manutenção.

## 3. Diagnóstico das necessidades (a partir das referências)

### 3.1 Necessidades recorrentes identificadas

- **Integridade do build e dos artefatos**: necessidade de builds totalmente automatizados, reproduzíveis, com scripts versionados, executados em ambientes efêmeros e isolados; geração de metadados de proveniência (provenance) para permitir decisões de risco.[^6][^7][^2][^3]
- **Proteção da cadeia de suprimentos e dependências**: controle centralizado de componentes de terceiros, geração e uso de SBOM (SPDX/CycloneDX), integração com ferramentas de Software Composition Analysis (SCA) para vulnerabilidades e licenças.[^7][^1][^3][^4]
- **Governança de repositórios e CI/CD**: forte política de versionamento, uso de controle de versão para todo código e scripts de build, autenticação forte e princípio de menor privilégio para acesso a repositórios, servidores de build e registries.[^5][^2][^3][^6][^7]
- **Processos de release e rollback disciplinados**: uso de feature flags e rollout progressivo (anéis, porcentagens) para desacoplar deploy de release e permitir rollback rápido sem reverter binários sempre que possível.[^8][^10][^11][^9]
- **Resposta a vulnerabilidades e incidentes**: necessidade de processos formais para identificação, correção e comunicação de vulnerabilidades (incluindo coordenação com clientes e fornecedores).[^1][^2][^4][^7]
- **Observabilidade e métricas**: monitoramento contínuo de deploys, uso de métricas de erro, performance e fluxo (lead time, taxa de falha de change) para orientar melhoria contínua.[^8][^5][^6]

### 3.2 Divergências e conflitos entre fontes

- **Profundidade de requisitos de segurança**: documentos NIST/SLSA assumem, em geral, cenários de alta criticidade e órgãos governamentais, recomendando níveis elevados de atestação, segmentação, hermeticidade de build e auditoria contínua.[^2][^3][^4][^5][^6][^1]
- **Pragmatismo vs. maturidade alta**: materiais focados em feature flags e trunk-based development enfatizam velocidade, simplicidade de fluxo (branch único/trunk) e pragmatismo, aceitando que nem todas as organizações chegarão a SLSA 3–4 em curto prazo.[^10][^11][^9][^8]
- **Rolagem vs. rollback binário**: documentos de feature flags favorecem rollback por toggle sempre que possível, enquanto guias de cadeia de suprimentos enfatizam também a capacidade de reimplantar versões anteriores assinadas e confiáveis.[^9][^3][^2][^8]

### 3.3 Conciliação proposta

- Para equipes pequenas e criticidade baixa ou média, adotar inicialmente um subconjunto de práticas alinhado a SLSA 1–2 e SSDF `Foundational`, garantindo automação, provenance básico, controle de acesso e uso de SCA, sem exigir hermeticidade total de builds ou atestações complexas.[^3][^5][^6][^7]
- Para criticidade alta (por exemplo, software usado em infraestrutura crítica ou com requisitos regulatórios fortes), planejar desde o início a evolução para práticas próximas de SLSA 3 e SSDF `Sustaining`/`Enhancing` (ambiente de build isolado, atestação robusta, SBOM integrado ao processo de compra e atualização).[^4][^5][^6][^1][^2][^3]

## 4. Glossário oficial da equipe

- **Build engineering**: disciplina responsável por desenhar, manter e automatizar o processo de compilação, empacotamento e geração de artefatos binários a partir do código-fonte, incluindo scripts, pipelines e ambientes de build.[^2][^3]
- **Release engineering**: disciplina responsável por planejar, coordenar e executar releases para ambientes de teste, homologação e produção, incluindo versionamento, canais, rollout, rollback e comunicação com stakeholders.[^5][^1]
- **Artefato**: resultado de um processo de build (binários, pacotes, imagens de container, instaladores, bibliotecas, SBOMs, manifests de deploy) armazenado em um repositório ou registry confiável.[^12][^7][^3]
- **Provenance**: conjunto de metadados que descreve como um artefato foi produzido (fonte, commit, ambiente de build, dependências, parâmetros), usado para avaliar integridade e risco.[^6][^7][^3]
- **SBOM (Software Bill of Materials)**: inventário dos componentes de software (bibliotecas, dependências, versões, licenças), usado para análise de vulnerabilidades e conformidade.[^7][^1][^3][^4]
- **CI (Continuous Integration)**: prática de integrar mudanças de código com frequência em um branch principal, executando automaticamente builds e testes para detectar problemas cedo.[^10][^9][^5]
- **CD (Continuous Delivery/Deployment)**: prática de manter o software sempre em estado implantável, automatizando deploys até ambientes de produção ou quase-produção, com gates de aprovação quando necessário.[^9][^5]
- **Feature flag**: mecanismo de configuração que permite ativar ou desativar funcionalidades em tempo de execução, desacoplando deploy de release e permitindo rollout progressivo e rollback rápido.[^11][^8][^10][^9]
- **Cadeia de suprimentos de software**: conjunto de processos, ferramentas e fornecedores envolvidos na construção, empacotamento, distribuição e operação do software, incluindo dependências de terceiros e serviços SaaS.[^1][^3][^4][^5][^6][^7]
- **SLSA**: framework de maturidade para integridade de artefatos (Supply-chain Levels for Software Artifacts), definindo níveis de garantia sobre source, build e distribuição.[^3][^6][^7]
- **SSDF (NIST)**: Secure Software Development Framework, conjunto de práticas de segurança aplicadas ao ciclo de vida de desenvolvimento.[^13][^14][^5][^6][^1]

## 5. Fluxo da ideia até a operação

### 5.1 Descrição em alto nível

1. **Descoberta/Ideação**: registro da ideia como item de backlog com contexto, impacto esperado, riscos de segurança e critérios de aceite.
2. **Planejamento técnico**: definição de escopo, design de arquitetura, requisitos de segurança e plano de rollout/rollback (incluindo se haverá feature flag).
3. **Implementação**: desenvolvimento em branch de feature ou diretamente no trunk (com flag), com commits pequenos e frequentes, seguindo padrões de código seguro e validações locais.
4. **Integração contínua (CI)**: abertura de Pull Request dispara pipeline de PR (build + testes + checagens de segurança básicas). Merge exige revisão e aprovação conforme políticas.
5. **Build oficial**: merge em branch principal ou branch de release dispara pipeline de build oficial, geração de artefatos, SBOM, assinatura e publicação em registry/repositório de artefatos.
6. **Release**: pipeline de release promove artefatos para ambientes/canais (dev, staging, produção, etc.), aplica migrações, configurações e feature flags conforme plano de rollout.
7. **Operação**: monitoramento de métricas, logs e eventos de segurança; suporte a incidentes, hotfixes e rollback conforme necessário.
8. **Melhoria contínua**: análise de métricas de fluxo, qualidade e segurança para ajustar políticas, pipelines e práticas.

### 5.2 Diagrama textual (visão completa)

- [IDEIA] → [BACKLOG PRIORIZADO]
- [BACKLOG PRIORIZADO] → (Planejamento técnico) → [HISTÓRIA/EPIC PLANEJADA]
- [HISTÓRIA/EPIC PLANEJADA] → (Criação de branch ou flag) → [TRABALHO EM ANDAMENTO]
- [TRABALHO EM ANDAMENTO] → (Commit e push) → [PULL REQUEST ABERTO]
- [PULL REQUEST ABERTO] → (Pipeline de PR: build + testes + SCA + lints) → [RESULTADO DE CI]
- [RESULTADO DE CI] → (Aprovação de revisão + checks verdes) → [MERGE EM MAIN/RELEASE]
- [MERGE EM MAIN/RELEASE] → (Pipeline de build oficial: build limpo + testes + SBOM + assinatura + publicação) → [ARTEFATO PUBLICADO]
- [ARTEFATO PUBLICADO] → (Pipeline de release: deploy para dev/staging + smoke tests) → [VALIDAÇÃO EM AMBIENTE]
- [VALIDAÇÃO EM AMBIENTE] → (Rollout progressivo: flags/anéis/porcentagens) → [RELEASE EM PRODUÇÃO]
- [RELEASE EM PRODUÇÃO] → (Monitoramento: métricas, logs, alertas) → [OPERAÇÃO NORMAL OU INCIDENTE]
- [INCIDENTE] → (Rollback via flag ou reimplante de versão anterior + hotfix se necessário) → [ESTADO ESTÁVEL]
- [ESTADO ESTÁVEL] → (Post-mortem, ajustes de pipeline/políticas) → [CICLO DE MELHORIA]

## 6. Estrutura sugerida do repositório

> Conclusão derivada de: necessidade de versionar scripts de build, manifestos de CI, políticas e documentação de segurança recomendadas por NIST e SLSA.[^5][^6][^7][^2][^3]

Exemplo genérico (monorepo simplificado):

```text
/
  docs/
    architecture.md
    security-model.md
    release-process.md
  src/
    ...código da aplicação...
  tests/
    ...testes automatizados...
  build/
    scripts/
    configs/
  ci/
    pipelines/
    templates/
  deploy/
    manifests/
    envs/
  .github/ ou .gitlab/ ou ci/
    workflows.yml / pipelines.yml
  SECURITY.md
  CONTRIBUTING.md
  CHANGELOG.md
  LICENSE
```

- **Justificativa**: separar claramente código, testes, scripts de build e manifests de deploy facilita governança, auditoria e reuso.[^2][^3][^5]
- **Alternativas**: múltiplos repositórios (por serviço), estrutura orientada a módulos por domínio, repositórios separados para infraestrutura.
- **Benefícios**: legibilidade, localização previsível de scripts e políticas, base para automação e auditors.[^3][^2]
- **Riscos**: reorganização pode causar atrito inicial; necessidade de manter documentação atualizada.
- **Custo**: esforço pontual de reestruturação e documentação.
- **Condições de revisão**: crescimento da base de código, mudança para microserviços, requisitos regulatórios específicos.

Perguntas abertas:

- O projeto atual usa monorepo ou múltiplos repos? (impacta organização de `build/`, `deploy/`)
- Há repositório separado para infraestrutura (ex.: IaC)?

## 7. Estrutura dos arquivos de build

### 7.1 Conceito no contexto

- "Estrutura de build" aqui significa: conjunto organizado de arquivos, scripts e configurações que definem como o código é transformado em artefatos (por exemplo: `build.gradle`, `pom.xml`, `Makefile`, `Dockerfile`, scripts de empacotamento, manifests de pipeline de build).[^12][^6][^5][^2][^3]

### 7.2 Proposta

- Diretório `build/` contendo:
  - `build/configs/`: parâmetros de build por ambiente (sem secrets), por exemplo `build.dev.yml`, `build.prod.yml`.
  - `build/scripts/`: scripts idempotentes e versionados, por exemplo `build.sh`, `package.sh`, `generate-sbom.sh`, `sign-artifacts.sh`.
  - `build/tools/`: wrappers para ferramentas de build específicas (Gradle, Maven, msbuild, etc.).

Decisão: **todo build oficial deve ser acionável via um único comando/script de orquestração**, por exemplo `./build/scripts/ci-build.sh`, que:

1. Faz checkout limpo do código.
2. Restaura dependências.
3. Executa build/compilação.
4. Executa testes.
5. Gera SBOM.
6. Empacota artefatos.
7. Assina artefatos (quando aplicável).

- **Justificativa**: scripts de build versionados e padronizados são recomendação central para garantir integridade e auditabilidade da cadeia de build.[^6][^7][^5][^2][^3]
- **Alternativas**: uso direto de pipelines declarativos no provedor de CI, com lógica embutida em YAML; uso de ferramentas de build multi-plataforma (Bazel, Pants).
- **Benefícios**: reprodutibilidade, menor risco de mudanças ad-hoc em agentes de CI, facilidade de migração entre ferramentas de CI.[^6][^3]
- **Riscos**: scripts mal mantidos podem acumular complexidade; duplicação de lógica se YAML de CI também contiver lógica.
- **Custo**: esforço de padronização dos scripts, treinamento da equipe.
- **Condições de revisão**: aumento de complexidade, adoção de nova plataforma de build ou linguagem.

Perguntas ao desenvolvedor sênior:

- Qual sistema de build é usado hoje (Gradle, Maven, msbuild, CMake, custom)?
- Existem scripts de build locais diferentes do build em CI?

## 8. Estrutura dos scripts

- Padrão recomendado de nomenclatura e organização:

```text
build/scripts/
  ci-build.sh
  local-build.sh
  package.sh
  generate-sbom.sh
  sign-artifacts.sh
  run-tests.sh
  verify-signature.sh
```

- Boas práticas derivadas de NIST e SLSA:
  - Scripts devem ser idempotentes, registrados em controle de versão e revisados como qualquer código.[^7][^2][^3][^6]
  - Não conter secrets embutidos; secrets vêm de mecanismos próprios de secret management.[^7][^2][^3]
  - Devem registrar logs suficientes para auditoria (com timestamps, parâmetros críticos, hashes de artefatos).

## 9. Pipeline de Pull Request

### 9.1 Objetivo

- Fornecer feedback rápido sobre qualidade e segurança das mudanças, antes de afetarem o branch principal.

### 9.2 Etapas mínimas

1. Checkout do código.
2. Restauração de dependências.
3. Build rápido (incremental quando suportado).
4. Execução de testes unitários.
5. Análise estática de código / linters.
6. SCA básica das dependências (quando viável em tempo de PR).[^4][^3][^7]
7. Verificação de formatação/padrões.

- **Justificativa**: SSDF e SLSA enfatizam integração de verificação de segurança e qualidade na pipeline de CI o mais cedo possível.[^1][^4][^5][^3][^6][^7]
- **Alternativas**: executar SCA e testes mais pesados apenas em nightly builds; usar análise estática assíncrona com comentários em PR.
- **Benefícios**: redução de bugs em produção, base para SLSA nível 2, criação de trilha de auditoria.[^5][^3]
- **Riscos**: pipelines lentas podem desincentivar pequenos PRs; necessidade de ajustar escopo dos checks.
- **Custo**: configuração de ferramentas, runners e integrações; tempo adicional de execução de CI.
- **Condições de revisão**: aumento de tempo de execução além de limite aceitável; crescimento da base de código.

## 10. Pipeline de build oficial

### 10.1 Etapas principais

1. Disparo por merge em `main` ou branch de release.
2. Checkout limpo em ambiente efêmero e isolado.[^2][^3][^5]
3. Execução de `ci-build.sh` (build completo + testes automatizados).[^3][^6]
4. Geração de SBOM e execução de SCA.[^4][^1][^7][^3]
5. Empacotamento dos artefatos (binários, instaladores, imagens, manifests).
6. Assinatura dos artefatos, quando aplicável (code signing).[^7][^2][^3]
7. Upload em repositório de artefatos/registry com metadados de provenance.[^6][^3][^7]
8. Publicação de resultados (logs, relatórios de testes, hashes) e notificação.

## 11. Estratégia de artefatos

- Recomendação: um repositório centralizado de artefatos (Nexus, Artifactory, GitHub Packages, container registry, etc.) com:
  - Artefatos imutáveis referenciados por versão e hash.
  - Metadados: commit, branch, data, ambiente, SBOM, resultado de testes.

- **Justificativa**: documentação de cadeia de suprimentos enfatiza inventário, armazenamento seguro e rastreabilidade de artefatos.[^1][^4][^2][^3][^6][^7]
- **Alternativas**: artefatos apenas em registries de container; distribuições diretas via marketplace.
- **Benefícios**: reprodutibilidade, suporte a rollback, auditoria de releases.[^3][^6]
- **Riscos**: custo de storage; necessidade de política de retenção.
- **Custo**: configuração de registry e credenciais, monitoramento de uso.
- **Condições de revisão**: aumento de volume; mudança na forma de distribuição.

Perguntas:

- Tipo de artefato principal (imagem de container, instalador desktop, plugin)?
- Há exigência de assinatura específica por plataforma (por exemplo, assinatura de driver, assinatura de app store)?

## 12. Política de versionamento

### 12.1 Proposta base (SemVer adaptado)

- Utilizar versionamento semântico (`MAJOR.MINOR.PATCH`), com possíveis sufixos de pré-release (`-alpha`, `-beta`, `-rc`), alinhado com práticas de distribuição de bibliotecas e aplicações.[^7]

- **Decisões dependentes de contexto**:
  - Se o produto é API pública, SemVer estrito é mais crítico.
  - Para aplicações desktop internas, foco maior em rastreabilidade operacional do que em compatibilidade pública.

- **Justificativa**: facilita comunicação de compatibilidade, riscos de quebra e políticas de suporte.[^7]
- **Alternativas**: versionamento por data (`YYYY.MM.DD`), por build incremental, combinação de ambos.

## 13. Política de branches

### 13.1 Modelo mínimo viável

- Branches principais:
  - `main`: sempre estável e implantável.
  - Branches de feature curtas, integradas frequentemente (trunk-based development).[^11][^10][^9]

- Branches opcionais (dependendo de criticidade e cadência):
  - `release/x.y`: congelamento para versões de longo suporte ou cadência fixa.
  - `hotfix/x.y.z`: correções emergenciais em produção.

- **Justificativa**: trunk-based com feature flags é recomendado para manter CI real e reduzir complexidade de merge.[^8][^10][^11][^9]
- **Alternativas**: GitFlow completo; múltiplos branches de manutenção de longo prazo.
- **Benefícios**: menor tempo de integração, base sempre próxima do estado de produção, simplificação do pipeline.[^10][^9]
- **Riscos**: exige disciplina em feature flags e testes; branches longos se tornam difíceis.
- **Condições de revisão**: aumento de criticidade, requisitos de LTS, múltiplas versões suportadas.

## 14. Política de tags

- Tags imutáveis no repositório para cada build/release oficial, seguindo a mesma convenção de versão (`vMAJOR.MINOR.PATCH`).

- **Justificativa**: tags são âncoras para provenance, auditoria e reconstrução de artefatos.[^6][^3][^7]
- **Alternativas**: tags adicionais para milestones internos, labels por ambiente.

## 15. Pipeline de release

### 15.1 Etapas típicas

1. Seleção do artefato aprovado (via versão/tag).
2. Deploy em ambiente de pré-produção/staging.
3. Execução de smoke tests e testes automatizados críticos.
4. Aprovação manual (quando necessário) com base em evidências (testes, métricas, segurança).[^5]
5. Rollout progressivo em produção (por ambiente, anéis de usuários ou porcentagem).[^9][^8][^10]
6. Monitoramento reforçado após release (janela de observabilidade).

- **Justificativa**: NIST SP 800-204D recomenda gates e automação integrada com métricas e segurança em pipelines de CI/CD.[^5]

## 16. Ambientes ou canais

- Possíveis ambientes/canais:
  - `dev`: uso de desenvolvimento interno.
  - `staging` / `homolog`: ambiente próximo de produção.
  - `prod`: produção.
  - Canais de distribuição: `internal`, `beta`, `GA`.

- **Decisões dependentes de contexto**:
  - Produtos desktop podem ter canais de atualização separados (beta/estável) e instaladores distintos.
  - SaaS pode usar anéis por tenant, país, plano, etc.[^8][^10][^9]

## 17. Estratégia de configuração

- Configurações separadas por ambiente, com princípio de doze fatores quando aplicável:
  - Uso de variáveis de ambiente e arquivos de configuração versionados sem secrets.
  - Definição clara de valores default seguros.[^2][^3][^7]

## 18. Estratégia de secrets

- Secrets nunca no repositório; uso de vault/serviço de secrets.
- MFA e rotação periódica de credenciais; registro de acesso.[^4][^2][^3][^7]

## 19. Estratégia de assinatura

- Assinar binários, instaladores ou imagens de container.
- Proteger chaves de assinatura em HSMs ou serviços equivalentes, com uso controlado e multi-pessoa para operações sensíveis.[^2][^3][^7]

## 20. Estratégia de segurança da cadeia

- Ações mínimas alinhadas a SSDF/SLSA:
  - Controle de acesso forte a repositórios e CI/CD (MFA, least privilege).[^3][^6][^2][^7]
  - Build automatizado e reproduzível em ambiente efêmero.[^5][^6][^2][^3]
  - Geração de SBOM e uso de SCA.[^1][^4][^3][^7]
  - Assinatura de artefatos e verificação de integridade em deploy.[^2][^3][^7]
  - Monitoramento e resposta a vulnerabilidades conhecidas.[^4][^1][^7][^2]

## 21. Estratégia de rollout

- Preferencialmente baseada em feature flags e anéis de rollout:
  - Anel 0: equipe interna.
  - Anel 1: clientes beta selecionados.
  - Anel 2: parcela (porcentagem) da base.
  - Anel 3: 100% da base.[^11][^10][^9][^8]

- Para desktop/plugins: uso de canais de atualização (beta/estável), waves regionais ou por tipo de cliente.

## 22. Estratégia de rollback

- Nível 1: rollback lógico via feature flag (desligar feature).[^10][^11][^9][^8]
- Nível 2: rollback de deploy usando artefatos anteriores (reimplantar versão anterior assinada).[^3][^7][^2]

- **Condições de uso**:
  - Rollback via flag exige que o estado de dados seja compatível com o caminho "flag off".
  - Rollback de artefato exige suporte a versões anteriores mantidas e migrações de dados reversíveis ou neutras.

## 23. Processo de hotfix

1. Detecção de incidente e severidade.
2. Avaliação de possibilidade de mitigação via feature flag.
3. Se necessário, criação de branch `hotfix/x.y.z` a partir da tag de produção.
4. Implementação mínima da correção, com testes direcionados.
5. Pipeline de PR e build oficial dedicados.
6. Release imediato com priorização de rollback se algo falhar.
7. Posterior merge da correção de volta para `main` e branches de release relevantes.

## 24. Observabilidade

- Mínimo recomendado:
  - Logs estruturados com correlação a versão do artefato, ambiente e usuário/tenant quando aplicável.
  - Métricas de disponibilidade, latência, erros por tipo e por versão.[^8][^6][^5]
  - Dashboards de saúde de release, com alertas automáticos.

## 25. Papéis e aprovações

- Papéis sugeridos (em equipes pequenas podem ser cumulativos):
  - **Dev responsável pela change**.
  - **Revisor de código** (outro dev).
  - **Responsável de release** (pode ser dev ou eng. de release).
  - **Responsável de segurança** (pode ser função parcial).

- Aprovações:
  - PR: pelo menos uma revisão técnica.
  - Release para produção: aprovação de responsável de release; em alta criticidade, dupla aprovação (técnica e de negócio).[^1][^5][^2]

## 26. Métricas

- Métricas de fluxo (inspiradas em DORA, não explicitamente referenciadas, mas alinhadas com recomendações de NIST sobre monitoramento de processo):
  - Lead time de mudança.
  - Frequência de deploy.
  - Taxa de falha de change (releases que exigem rollback/hotfix).
  - Tempo médio de restauração (MTTR).[^6][^5]

- Métricas de segurança:
  - Número de vulnerabilidades críticas abertas por release.
  - Tempo para corrigir vulnerabilidades.

- Métricas de flags:
  - Número de flags ativas e tempo médio de vida (para evitar "flag debt").[^8]

## 27. Checklist de release

Checklist mínimo antes de promover release para produção:

- [ ] Build oficial executado com sucesso a partir de commit/tag conhecida.
- [ ] Tests automatizados críticos aprovados.
- [ ] SBOM gerado e SCA sem vulnerabilidades críticas não tratadas.
- [ ] Artefatos assinados e verificação de assinatura bem-sucedida.
- [ ] Plano de rollout definido (anéis, flags, janelas de manutenção).
- [ ] Plano de rollback claro (flag ou reimplante da versão anterior) e testado quando possível.
- [ ] Observabilidade configurada (dashboards e alertas específicos para a release).[^4][^1][^5][^6][^7][^2][^3]

## 28. Exemplos narrativos

### 28.1 Exemplo de release normal

1. Feature finalizada em feature branch com flag desativada em produção.
2. PR aprovado, pipeline de PR passa.
3. Merge em `main` dispara build oficial; artefato `v1.4.0` é gerado, SBOM e assinatura ok.
4. Pipeline de release implanta `v1.4.0` em `staging`, executa smoke tests.
5. Responsável de release aprova promoção para produção.
6. Rollout progressivo: 5% dos usuários, depois 25%, 50%, 100%, com monitoramento.[^9][^10][^8]

### 28.2 Exemplo de hotfix

1. Erro crítico em produção identificado (ex.: crash em fluxo específico).
2. Flag não resolve porque bug é em caminho crítico sem flag.
3. Criado branch `hotfix/1.4.1` a partir da tag `v1.4.0`.
4. Correção implementada, testes direcionados e pipeline de PR.
5. Build oficial gera artefato `v1.4.1`, assinado e publicado.
6. Deploy direto em produção (com validação rápida em staging, se possível).
7. Monitoramento reforçado e posterior merge da correção em `main`.

### 28.3 Exemplo de falha e recuperação

1. Durante rollout de `v1.5.0`, aumento de erros 500 após liberar para 25% dos usuários.
2. Alertas disparam; responsável de release pausa rollout.
3. Flag da nova funcionalidade é desligada; métricas retornam ao normal.
4. Post-mortem identifica bug em caminho protegido pela flag.
5. Fix é desenvolvido e liberado em `v1.5.1`, repetindo rollout progressivo.

## 29. Roadmap de adoção

### 29.1 Primeiros 30 dias

- Mapear estado atual de repositórios, scripts de build, CI/CD e práticas de release.
- Migrar scripts de build críticos para diretório versionado (`build/scripts`).
- Padronizar pipeline de PR com build + testes básicos + lints.
- Definir política de branches mínima (`main` + feature branches).
- Documentar checklist de release básico.

### 29.2 Dias 31 a 60

- Implementar pipeline de build oficial com geração de artefatos centralizada.
- Introduzir SBOM e SCA em builds oficiais.
- Definir política de versionamento e tags.
- Introduzir feature flags em pelo menos um fluxo crítico.
- Configurar métricas básicas de observabilidade ligadas a versões.

### 29.3 Dias 61 a 90

- Introduzir assinatura de artefatos (quando aplicável).
- Começar rollouts progressivos em produção.
- Refinar processo de hotfix e rollback, incluindo testes de exercício de rollback.
- Estabelecer papéis de aprovação de release.
- Iniciar revisão periódica de flags e vulnerabilidades.

### 29.4 Evolução posterior

- Avaliar adoção de práticas alinhadas a SLSA 3 (builds mais herméticos, atestação mais forte).
- Integrar atestação de build, SBOM e sinais de segurança em decisões automatizadas de deploy.
- Evoluir métricas de fluxo e qualidade para orientar melhorias de processo.

## 30. Modelo mínimo viável vs. modelo futuro

### 30.1 Modelo mínimo viável

- CI com pipeline de PR (build + testes básicos + lint).
- Build oficial automatizado com script único, geração de artefatos reprodutíveis.
- Repositório central de artefatos (mesmo que simples).
- Política de branches simplificada (trunk-based + feature branches curtos).
- Versionamento claro (SemVer) e tags para releases.
- Rollback básico via reimplante da última versão estável.
- Observabilidade essencial (logs estruturados, métricas de erro por versão).

### 30.2 Modelo de maturidade futura

- Adoção parcial ou total de práticas SLSA 2–3 (provenance autenticado, builds isolados e com atestação forte).[^6][^7][^3]
- Integração plena de SBOM, SCA, análise estática e dinâmic

---

## References

1. [Enduring Security Framework](https://csrc.nist.gov/csrc/media/Presentations/2024/enduring-security-framework/images-media/20240918_SSCA_ESF%20Software%20Supply%20Chain_MODIFIED_PP-24-3646%20%20-%20%20Read-Only.pdf)

2. [Securing The Software Supply Chain for Suppliers](https://media.defense.gov/2022/Oct/31/2003105368/-1/-1/0/securing_the_software_supply_chain_suppliers.pdf)

3. [[PDF] OpenSSF SLSA & NIST SSDF: Emerging Software Supply Chain ...](https://www.socallinuxexpo.org/sites/default/files/presentations/Google%20SLSA%20&%20NIST%20SSDF%20(Frameworks%20Presentation).pdf)

4. [[PDF] Software Security in Supply Chains](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=958682)

5. [SP 800-204D, Strategies for the Integration of Software Supply ...](https://csrc.nist.gov/pubs/sp/800/204/d/final) - The predominant application architecture for cloud-native applications consists of multiple microser...

6. [All about that Base(line): How Cybersecurity Frameworks ...](https://slsa.dev/blog/2022/07/slsa-foundational-framework) - In coordination with Aaron Bacchi, Emmy Eide, Melba Lopez, Brandon Lum, and Moshe Zioni

7. [Supply Chain Security Frameworks (NIST SSDF, SLSA) and Best ...](https://dohost.us/index.php/2025/11/01/supply-chain-security-frameworks-nist-ssdf-slsa-and-best-practices/) - Supply Chain Security Frameworks (NIST SSDF, SLSA) and Best Practices In today's interconnected digi...

8. [Feature Flags: 12 Best Practices (With Code Examples)](https://designrevision.com/blog/feature-flags-best-practices) - The complete guide to feature flags best practices for SaaS teams. Covers implementation patterns, f...

9. [Implement trunk-based development using feature flags](https://docs.getunleash.io/guides/trunk-based-development) - Implement trunk-based development with Unleash feature flags: merge faster, keep trunk deployable, c...

10. [Trunk-based development is not a Git trick. It is what ...](https://emmanuelvalverderamos.substack.com/p/trunk-based-development-is-not-a) - Many teams say they do CI and CD. But they don't

11. [Feature Flags in Trunk-Based Development | Matthew Thomas](https://matthewthomas.cloud/posts/tbd-feature-flagging/) - Exploring how feature flags fit into trunk-based development — the challenges they introduce, and pr...

12. [Managing Build Artifacts Using Maven and Nexus in CI/CD Workflows](https://www.questjournals.org/jses/papers/Vol9-issue-12/09128094.pdf)

13. [Software Supply Chain Security Guidance | NIST](https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-supply-chain-security-guidance) - ... best practices, and other guidelines to enhance software supply chain security. Those guidelines...

14. [Software Security in Supply Chains: Enhanced Vendor Risk Assessments](https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-supply-chain-security-guidance-21) - The EO creates higher standards for software verification techniques and other software supply chain...

