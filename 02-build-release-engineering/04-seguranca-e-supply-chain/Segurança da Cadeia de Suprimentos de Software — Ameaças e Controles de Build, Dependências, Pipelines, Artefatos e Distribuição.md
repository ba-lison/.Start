# Segurança da cadeia de suprimentos de software

## 1. Resumo executivo

A cadeia de suprimentos de software hoje é um dos vetores de ataque mais críticos, a ponto de o OWASP Top 10:2025 introduzir **Software Supply Chain Failures** como categoria própria, cobrindo falhas em dependências, builds, pipelines e distribuição. NIST, CISA e a comunidade OpenSSF responderam com frameworks como **SSDF** (SP 800‑218), **SLSA**, padrões de **SBOM** (SPDX, CycloneDX) e mecanismos modernos de **proveniência e assinatura de artefatos** (Sigstore, in‑toto), que permitem rastrear e atestar de forma criptográfica desde o commit até o binário instalado.[^1][^2][^3][^4][^5]

O objetivo central desta referência é responder a uma pergunta: **como garantir que o código revisado é exatamente o mesmo código que foi compilado, assinado e distribuído**, apesar de ameaças como dependency confusion, typosquatting, pacotes comprometidos, abuso de runners de CI/CD e vazamento de credenciais. Para isso, mapeiam‑se ameaças típicas por etapa (código, dependências, build, pipeline, artefatos, distribuição) e propõem‑se controles preventivos, detectivos e corretivos baseados em padrões oficiais e documentação de projetos.[^6][^7][^8]

A abordagem recomendada combina: (1) um **modelo de ameaças** alinhado a OWASP e NIST; (2) um **fluxo de build seguro** com SLSA e proveniência assinada; (3) controles de aplicação de política em SCM e CI/CD (proteção de branches, runners efêmeros, least privilege, OIDC, proteção de arquivos de pipeline); (4) visibilidade via SBOM + SCA/SAST/DAST/secret scanning/license scanning; e (5) processos de resposta a CVEs e revogação de releases, inclusive suporte a revogação de confiança via assinatura, checksums e frameworks como TUF.[^2][^9][^10]

## 2. Modelo de ameaças da cadeia de suprimentos

### 2.1 Escopo e superfícies de ataque

A categoria “Software Supply Chain Failures” do OWASP define falhas em componentes, ferramentas de build, pipelines e infra de distribuição, indo além de “componentes com vulnerabilidades conhecidas”. NIST SSDF organiza práticas em quatro grupos — **Prepare the Organization, Protect the Software, Produce Well‑Secured Software, Respond to Vulnerabilities** — que ajudam a estruturar controles ao longo de todo o ciclo.[^11][^7][^3][^4]

Superfícies principais para o modelo de ameaças:

- **Código fonte e SCM**: commits maliciosos, comprometimento de contas de desenvolvedores, alteração de workflows/pipelines, bypass de revisão de código.
- **Dependências diretas e transitivas**: dependency confusion, typosquatting, pacotes comprometidos ou abandonados, licenças incompatíveis.[^12][^13]
- **Sistema de build / CI/CD**: runners compartilhados, vazamento de segredos, alteração de steps, execução de código não revisado com credenciais privilegiadas.[^14][^15]
- **Artefatos e registries**: injeção de malware em imagens, troca de binário após build, ausência de assinatura/verificação, registries expostos.[^16][^17]
- **Distribuição e atualização**: canais de update sem verificação de integridade/autenticidade, CDN ou mirror comprometido, ausência de revogação de releases.[^8][^10]

### 2.2 Como um invasor compromete a build

- **Alvo: pipeline/runner**: Ataques a runners persistentes permitem que backdoors plantados em um job sobrevivam e infectem jobs futuros, inclusive builds de produção. Runners com acesso amplo à rede e a credenciais de nuvem permitem movimento lateral.[^15][^14]
- **Alvo: arquivos de pipeline**: Modificação de workflows (por exemplo, GitHub Actions YAML, GitLab CI, Azure Pipelines) para:
  - incluir steps de exfiltração;
  - pular testes ou verificações de assinatura;
  - alterar parâmetros de build para apontar para branches ou dependências maliciosas.[^18][^15]
- **Alvo: credenciais**: Uso de segredos expostos em código, variáveis de ambiente ou logs para assumir contas de CI, registries ou nuvem, permitindo alterarações em imagens, buckets ou releases.
- **Alvo: infraestrutura de build**: Comprometimento de builders e servidores gera proveniência falsificável e artefatos trojanizados, como visto em incidentes de supply chain amplamente analisados por órgãos governamentais.[^5][^8]

### 2.3 Como uma dependência comprometida entra no produto

- **Dependency confusion**: O atacante registra num registry público um pacote com o mesmo nome de um pacote interno, mas com versão maior; a ferramenta de build resolve para o pacote público, que executa código malicioso durante instalação com os privilégios do build.[^13][^12]
- **Typosquatting**: Criação de pacotes com nomes muito parecidos com bibliotecas populares (`reqeusts` vs `requests`, etc.), explorando erros de digitação de desenvolvedores; pesquisas documentam alta eficácia desse vetor.[^19][^12]
- **Pacotes comprometidos/upstream hijack**: Compromisso da conta do maintainer, takeover de projeto abandonado ou injeção de backdoor em commit upstream, como reconhecido por guias de CISA e NIST ao tratar de componentes de terceiros.[^3][^8]
- **Dependências transitivas**: Mesmo que as dependências diretas sejam revisadas, uma dependência de terceiro pode trazer uma transitiva maliciosa ou vulnerável, que entra silenciosamente no produto; OWASP enfatiza inventário contínuo de dependências diretas e transitivas como controle essencial.[^20][^3]

### 2.4 Como proteger credenciais no contexto de supply chain

- Organização de NIST e CISA recomendam **credenciais de curta duração**, autenticação federada e minimização de segredos armazenados em repositórios e runners.[^21][^8]
- O uso de **OpenID Connect (OIDC) com Workload Identity Federation** permite que jobs de CI obtenham tokens temporários para provedores de nuvem sem armazenar chaves estáticas, como demonstrado em tutoriais oficiais do GitLab+GCP.[^22]
- Guias de hardening de runners enfatizam **least privilege**, separação de contas de serviço, bloqueio de acesso a metadados de nuvem e uso de runners efêmeros para reduzir superfície de vazamento de segredos.[^14][^15]

### 2.5 Como evitar alteração não autorizada no pipeline

- NIST SSDF e CISA orientam a proteger arquivos de build e delivery contra acesso não autorizado e tampering, com controles de acesso, revisão obrigatória e registro de alterações.[^11][^8]
- Práticas de code signing e branch protection recomendam exigir **commits assinados, revisão obrigatória e proteção específica para diretórios sensíveis** (como `.github/workflows`, `.gitlab-ci.yml`) usando CODEOWNERS e regras de branch.[^23][^18]

### 2.6 Como comprovar origem do artefato

- **Proveniência SLSA** define um documento assinado descrevendo onde, quando, como e por qual builder um artefato foi produzido, usando o formato de attestation do in‑toto.[^16][^5]
- A combinação de SLSA + Sigstore/in‑toto permite atestar que certo digest foi produzido por um pipeline específico, com commit e inputs específicos, e registrar isso em logs de transparência.[^17][^2]

## 3. Fluxo de build seguro (alto nível)

### 3.1 Padrões de referência (SSDF, SLSA, OWASP)

- **NIST SP 800‑218 (SSDF)** recomenda práticas de preparação organizacional, proteção de software, produção de software bem protegido e resposta a vulnerabilidades, servindo como base para políticas internas e requisitos de fornecedores.[^7][^11]
- **SLSA (Supply‑chain Levels for Software Artifacts)** define níveis progressivos de integridade de build, desde documentação básica de processo (nível 1) até builds herméticos com revisão em dupla (nível 4).[^2][^16]
- **OWASP Top 10:2025** enfatiza inventário de componentes, pinagem de versões, SBOM, verificação de assinaturas/proveniência e tratamento de updates de terceiros como parte integral do modelo de ameaças.[^4][^3]

### 3.2 Fluxo seguro proposto (do commit ao release)

1. **Código e revisão**
   - Branches protegidos com commits assinados, revisões obrigatórias, CODEOWNERS para workflows, scripts de deploy e infra de build.[^18][^23]
   - SAST, secret scanning e linting de pipeline rodando em cada PR.[^20][^11]
2. **Gestão de dependências**
   - Uso de manifests e lockfiles (`package-lock.json`, `requirements.txt` fixo, `pom.lock`, etc.), repositórios privados/proxy e políticas de roteamento que impedem dependency confusion.[^13][^3]
   - Execução de SCA + license scanning sobre a árvore de dependências, gerando SBOM (SPDX/CycloneDX) e anotando vulnerabilidades/licenças.
3. **Build em runner efêmero**
   - Build automatizado em runner isolado e efêmero (VM ou pod que processa um único job), com credenciais mínimas obtidas via OIDC.[^22][^15][^14]
   - Build produz artefatos versionados (binários, imagens) e gera **proveniência SLSA** e **SBOM** como artefatos auxiliares.[^5][^2]
4. **Assinatura e publicação de artefatos**
   - Artefatos e suas atestações (SBOM, proveniência) são assinados com Sigstore/cosign (idealmente assinatura “keyless” via OIDC) e enviados ao registry ou storage de releases.[^9][^17]
   - Checksums são publicados (e.g., `SHA256SUMS`) para consumo manual e por scripts/gestores de pacote.[^10]
5. **Distribuição e deploy com verificação**
   - Sistemas de deploy ou clientes verificam assinaturas e proveniência antes de aceitar artefatos; políticas de admissão (por exemplo, admission controller em Kubernetes) rejeitam imagens não assinadas ou sem proveniência esperada.[^17][^16]
   - Monitoramento contínuo de vulnerabilidades contra SBOMs em produção, com processos definidos para resposta a CVEs e potencial revogação de releases.[^8][^10]

## 4. Controles preventivos

### 4.1 Proteção de branches e code signing

- GitHub, GitLab e plataformas similares oferecem **branch protection** para exigir:
  - revisões obrigatórias;
  - commits assinados (GPG, SSH ou S/MIME);
  - status checks obrigatórios (builds, testes, SAST, SCA).[^23][^18]
- Boas práticas de code signing incluem chave dedicada por desenvolvedor, uso de hardware MFA (YubiKey, FIDO2) para proteger chaves e rotação periódica dos pares de chaves.[^23]

### 4.2 Gestão de dependências, lockfiles e SCA

- **Software Composition Analysis (SCA)** é definido pela OWASP como análise automatizada de componentes de terceiros para identificar vulnerabilidades e riscos de licenciamento. Ferramentas como OWASP Dependency‑Check e Dependency‑Track ajudam a mapear dependências e CVEs.[^24][^20]
- **Lockfiles** (e.g. `package-lock.json`, `Pipfile.lock`) garantem que builds reproduzam a mesma árvore de dependências, reduzindo variação não controlada em ambientes diferentes.[^25][^13]
- Para mitigar **dependency confusion**:
  - usar escopos/namespaces (@org/pkg, groupIds internos) e roteamento explícito para registries internos, evitando fallback para registries públicos para nomes internos;[^26][^13]
  - pré‑registrar nomes internos em registries públicos como squat defensivo, e monitorar se nomes internos surgem publicamente.[^13]
- Para mitigar **typosquatting**: políticas de revisão de novos pacotes, listas de bloqueio para padrões suspeitos e uso de proxys que aprovam explicitamente first‑use de pacotes externos.[^12][^19]

### 4.3 Runners isolados, efêmeros e least privilege

- Guias especializados e de fornecedores destacam **ephemeral first** como princípio, ou seja, cada runner deve processar um job (ou conjunto muito estreito de jobs) e ser destruído em seguida, evitando contaminação cruzada.[^27][^14]
- Recomendações incluem:
  - runners auto‑registrados com tokens de curta duração;
  - execução sob usuários sem privilégios, sem acesso a Docker privilegiado ou root, com Pod Security Standards “restricted” em Kubernetes;[^15][^27]
  - network egress estritamente necessário (GitHub, registry interno, provedores de pacote), bloqueando o resto.[^15]
- Runner groups devem separar jobs de PR, builds internos e deploys de produção, evitando que código não confiável chegue a runners com acesso a segredos de produção.[^28][^14]

### 4.4 Least privilege, OIDC e rotação de credenciais

- NIST e CISA recomendam **princípio do menor privilégio** tanto para contas humanas quanto de serviço, incluindo CI/CD, com revisão de permissões e monitoramento contínuo.[^11][^8]
- **OIDC + Workload Identity Federation** substitui chaves de longo prazo por tokens de acesso temporários obtidos pela identidade do job de CI, como mostrado no fluxo GitLab→GCP, onde um JWT da pipeline é trocado por um token de acesso restrito.[^22]
- Rotação de credenciais deve ser automatizada sempre que possível, com caducidade definida e processos de revogação em incidentes.

### 4.5 SBOM, SPDX, CycloneDX e proveniência SLSA

- CISA define SBOM como inventário formal e legível por máquina de componentes, dependências e relacionamentos, enfatizando abrangência e clareza de onde relações não podem ser articuladas.[^10]
- **SPDX** é padrão aberto mantido pela Linux Foundation para representação de SBOMs, incluindo informações de pacotes, licenças, checksums e relacionamentos; a versão 2.3 adiciona campos focados em segurança (datas de build/release, novos algoritmos de hash, referências de segurança).[^29][^30][^31]
- **CycloneDX** é outro padrão de SBOM (mantido pelo projeto OWASP), com foco em segurança, suportando hashes, identidades (Package URL, CPE, SWID), licenças, relações de dependência e serviços remotos; a especificação 1.6 define formato JSON para BOMs versionados e serializados.[^32][^33]
- **Proveniência SLSA** (predicate `https://slsa.dev/provenance/v1`) descreve o buildDefinition (buildType, parâmetros, dependências resolvidas) e runDetails (builder.id, metadados) para permitir que consumidores verifiquem se o artefato foi construído conforme expectativas.[^2][^5]

### 4.6 Assinatura de artefatos e code signing

- **Sigstore/cosign** implementa assinatura “keyless” baseada em identidade OIDC, onde Fulcio emite certificados de curta duração vinculando chave efêmera à identidade, e Rekor registra logs de transparência imutáveis para auditoria.[^9]
- Boas práticas de code signing incluem assinatura de commits, tags, pacotes, imagens de container e atestações (SBOM, proveniência), com verificação automatizada em CI e em admission controllers no momento do deploy.[^9][^17]
- Frameworks como **The Update Framework (TUF)** são usados para proteger a distribuição de updates (inclusive distribuição da raiz de confiança do Sigstore), com suporte a revogação e rotação de chaves.[^10][^9]

## 5. Controles detectivos

### 5.1 SAST, DAST e secret scanning

- NIST SSDF recomenda testar código executável para vulnerabilidades e conformidade com requisitos de segurança como parte do ciclo de desenvolvimento.[^11]
- **SAST** (Static Application Security Testing) analisa código fonte ou bytecode em busca de padrões de vulnerabilidade, enquanto **DAST** testa a aplicação em execução, simulando ataques externos; OWASP classifica ambos como componentes fundamentais em DevSecOps.[^3][^20]
- **Secret scanning** detecta chaves de API, tokens e credenciais acidentalmente commitados, reduzindo risco de comprometimento de infraestrutura e supply chain; muitos provedores de SCM e CI incluem scanners nativos ou integráveis.[^18][^11]

### 5.2 SCA, license scanning e monitoramento contínuo

- SCA deve ser contínuo, não apenas no build inicial: SBOMs em produção precisam ser correlacionados constantemente com bancos de dados de vulnerabilidades (NVD, advisories de fornecedores, KEV) e boletins licenciados.[^8][^10]
- License scanning baseado em SBOM/SPDX permite detectar componentes com licenças incompatíveis (por exemplo, copyleft forte em componentes não desejados), evitando riscos legais além dos de segurança.[^34][^31]

### 5.3 Monitoramento de registries e atividades suspeitas

- Para dependency confusion e typosquatting, recomenda‑se monitorar registries públicos por nomes internos recém‑publicados e padrões suspeitos, e auditar continuamente nomes internos vs. registros externos.[^26][^13]
- Logs de transparência (como Rekor) permitem monitorar eventos de assinatura inesperados vinculados à identidade da organização, detectando potenciais abusos da infraestrutura de assinatura.[^17][^9]

## 6. Controles corretivos

### 6.1 Resposta a CVEs em dependências

- NIST SSDF define um grupo específico de práticas para **Respond to Vulnerabilities (RV)**, cobrindo identificação, priorização, correção e prevenção de recorrências.[^11]
- CISA e ESF sugerem processos para:
  - inventariar software via SBOM;
  - usar SCA para identificar componentes vulneráveis;
  - avaliar risco com base em criticidade, exposição e contexto de uso;[^8][^10]
  - aplicar patches ou workarounds e registrar lições aprendidas.

### 6.2 Revogação de releases

- Frameworks de update seguros (como TUF) permitem marcar versões como comprometidas ou revogadas, de forma que clientes recusem updates assinados com chaves revogadas ou listados em metadados de revogação.[^9][^10]
- Em ambientes internos, revogação de release envolve:
  - marcar o build como comprometido em registros internos;
  - revogar ou girar chaves usadas para assiná‑lo;
  - atualizar repositórios/registries para bloquear downloads dessa versão;
  - acionar mecanismos de rollback nos sistemas afetados.[^8]

### 6.3 Tratamento de builds/artefatos comprometidos

- Proveniência SLSA e logs de transparência permitem investigar qual commit, builder e conjunto de dependências produziram um artefato comprometido, apoiando forense e erradicação.[^5][^17]
- Processos de incident response devem incluir etapas específicas para supply chain: isolar runners, revogar tokens de CI, suspender pipelines, auditar workflows e revisar permissões de contas de serviço.[^14][^8]

## 7. Estratégia mínima para equipe pequena

### 7.1 Prioridades práticas

Para uma equipe pequena, recomenda‑se priorizar controles com maior relação custo/benefício:

- **Pinagem de dependências e uso de lockfiles** em todos os projetos.[^25][^13]
- **Uso de repositório proxy de dependências** (Nexus, Artifactory lite, Verdaccio) com allow‑list e cache de pacotes aprovados.[^13]
- **Runners hospedados pelo provedor (GitHub/GitLab) ou self‑hosted efêmeros**, evitando runners persistentes.[^18][^15]
- **SAST/SCA e secret scanning** integrados ao CI básico (plugins nativos ou ferramentas open source).[^24][^20]
- **SBOM automático por build de release**, preferencialmente em CycloneDX ou SPDX, usando plugins de build.[^33][^29]
- **Assinatura mínima de artefatos de produção** com Sigstore/cosign (keyless), com verificação manual ou em deploy quando possível.[^17][^9]

### 7.2 Trade‑offs e simplificações

- Em vez de buscar SLSA 3 ou 4 de imediato, almejar **SLSA 1–2**: builds totalmente automatizados e geração de proveniência assinada pela plataforma de build.[^16][^2]
- Focar em poucas políticas claras: branches protegidos, revisões obrigatórias, proibição de segredos em repositórios e uso de OIDC em vez de chaves estáticas quando suportado.[^22][^23]

## 8. Estratégia para produto empresarial

### 8.1 Alinhamento a SSDF, SLSA e requisitos regulatórios

- Organizações maiores tendem a ser cobradas por conformidade com **NIST SSDF**, memorandos OMB (como M‑22‑18) e exigências de SBOM/Secure Development em contratos governamentais.[^7][^8]
- Em contexto empresarial, a meta razoável é buscar **SLSA nível 3 para builds de produção**, com builders endurecidos, proveniência não falsificável e revisão em dupla para alterações de código e de configuração de pipeline.[^35][^2]

### 8.2 Programas estruturados de supply chain security

- Implantar **programa formal de Software Supply Chain Security**, cobrindo:
  - governança (políticas, ownership de componentes, aceitação de risco);
  - plataforma centralizada de CI/CD e registries com hardening padronizado;
  - SBOMs centralizadas por produto/versão com correlação automática a vulnerabilidades;[^10][^8]
  - requisitos de fornecedores para atestarem conformidade com SSDF/SBOM ao fornecerem software.[^6][^21]

### 8.3 Integração com GRC, compras e resposta a incidentes

- Guias NIST/CISA sugerem ampliar avaliações de risco de fornecedor para incluir práticas de desenvolvimento seguro (SSDF), SBOMs e processos de resposta a vulnerabilidades.[^21][^8]
- Processos de incident response corporativos devem prever cenários de supply chain (compromisso de dependency, pipeline, registry) e exercícios de tabletop específicos.

## 9. Checklists operacionais

### 9.1 Checklist de pipeline (CI/CD)

- [ ] Branch `main`/`release` protegido com:
  - [ ] commits assinados obrigatórios;[^23]
  - [ ] ao menos uma revisão obrigatória;[^18]
  - [ ] CODEOWNERS para diretórios sensíveis (`.github`, `.gitlab-ci.yml`, scripts de deploy).[^23]
- [ ] Workflows/pipelines:
  - [ ] definem `permissions` mínimos para tokens (ex.: `contents: read` por padrão);[^18]
  - [ ] pinam actions/plugins externos por SHA de commit, não por `latest` ou tag flutuante;[^18]
  - [ ] usam runners efêmeros para jobs com segredos ou artefatos de produção;[^14][^15]
  - [ ] separam jobs de PR (código de terceiros) de jobs de release/deploy.[^28]
- [ ] Runners:
  - [ ] são efêmeros (uma job por VM/pod) quando possível;[^27][^14]
  - [ ] rodam como usuário sem privilégios, com acesso de rede mínimo;[^15]
  - [ ] não são usados para repositórios públicos quando têm acesso a recursos internos;[^14]
  - [ ] usam OIDC/workload identity em vez de chaves estáticas para acesso a nuvem.[^22]
- [ ] Arquivos de pipeline:
  - [ ] não contém segredos em texto claro;[^11]
  - [ ] são protegidos por branch protection e CODEOWNERS;[^23]
  - [ ] passam por revisão de segurança ao alterar lógica de build/deploy.[^18]

### 9.2 Checklist de artefato

- [ ] Cada build de release gera:
  - [ ] artefato versionado (imagem, pacote, binário);
  - [ ] **SBOM** em formato SPDX ou CycloneDX;[^29][^33]
  - [ ] **proveniência SLSA** descrevendo commit, builder, parâmetros e dependências;[^5]
  - [ ] checksums (por exemplo, SHA‑256) publicados.[^30][^32]
- [ ] Assinatura:
  - [ ] artefato assinado com Sigstore/cosign (preferencialmente keyless via OIDC);[^9]
  - [ ] SBOM e proveniência publicados como atestações assinadas associadas ao artefato.[^17]
- [ ] Armazenamento:
  - [ ] artefatos e metadados armazenados em registry/repos seguro com controle de acesso;[^8]
  - [ ] política de retenção e arquivamento de builds antigos definida;[^11]
  - [ ] logs de transparência (Rekor ou similar) monitorados para assinaturas inesperadas.[^9]
- [ ] Consumo/deploy:
  - [ ] pipelines de deploy verificam assinaturas e proveniência antes do rollout;[^17]
  - [ ] admission controllers rejeitam imagens sem assinatura válida ou proveniência esperada.[^16]

### 9.3 Checklist de resposta a comprometimento

- **Detecção e contenção**
  - [ ] Mecanismos de alerta para:
    - builds falhando em verificações de assinatura/proveniência;
    - novos pacotes internos surgindo em registries públicos;[^13]
    - credenciais vazadas (secret scanning, alertas do provedor de SCM).[^18]
  - [ ] Procedimentos para isolar runners suspeitos (desregistrar, destruir VMs/pods).[^15]
- **Erradicação**
  - [ ] Uso de SBOMs e proveniência para identificar quais versões e componentes foram afetados.[^5][^10]
  - [ ] Revogação/rotação de chaves e tokens potencialmente expostos.[^11]
  - [ ] Correção de dependências vulneráveis (update, patch, mitigação) e rebuild de artefatos.[^8]
- **Recuperação**
  - [ ] Rollback ou rollout de versões corrigidas, com verificações adicionais de assinatura.[^9]
  - [ ] Atualização de documentação, playbooks e treinamento com lições aprendidas.[^11]

## 10. Mapa risco → controle

### 10.1 Tabela de mapeamento

| Risco principal | Exemplos de cenário | Controles recomendados |
| --- | --- | --- |
| Dependency confusion | Pacote interno resolvido a partir de registry público com versão maior; execução de script malicioso na instalação.[^12][^13] | Namespaces/escopos internos, roteamento explícito para registries privados, lockfiles, proxy de dependências com allow‑list, monitoramento de nomes internos em registries públicos.[^13][^26] |
| Typosquatting | Instalação acidental de `reqeusts` em vez de `requests`, pacote malicioso publica dados ou abre backdoor.[^12][^19] | Revisão manual de novos pacotes, scanning de reputação, lista de bloqueio de padrões suspeitos, proxy de dependências com aprovação humana para first‑use.[^3][^13] |
| Pacote upstream comprometido | Conta do maintainer hackeada, versão nova introduz malware; dependência transitiva traz componente vulnerável.[^3][^8] | SCA contínuo, SBOM por release, políticas de atualização controlada, verificação de assinatura de pacotes, revisão de changelogs de bibliotecas críticas.[^20][^10] |
| Comprometimento de pipeline/runner | Modificação de workflow, runner persistente contaminado, vazamento de segredos via logs.[^14][^15] | Runners efêmeros, isolamento de rede, branch protection + CODEOWNERS para arquivos de pipeline, least privilege, secret scanning, OIDC com credenciais de curta duração.[^18][^22] |
| Troca de artefato após build | Binário ou imagem alterada manualmente no registry ou storage.[^16][^17] | Assinatura de artefatos, proveniência SLSA, registries com controle de acesso estrito, verificação de assinatura em pipelines de deploy e em clientes.[^9][^5] |
| Distribuição/update comprometido | Canal de update serve versão maliciosa; cliente não verifica integridade/autenticidade.[^8][^10] | Uso de frameworks de update seguros (ex.: TUF), assinatura de metadados de update, revogação de chaves comprometidas, verificação de assinatura no cliente.[^9][^10] |
| Exposição de credenciais | Segredos em código, logs, runners com acesso amplo a metadados de nuvem.[^11][^15] | Secret scanning, cofres de segredos, OIDC/workload identity, network policies bloqueando IMDS, rotação de credenciais.[^22][^14] |
| Falhas em resposta a CVEs | Equipe desconhece vulnerabilidades críticas em componentes em produção.[^8][^3] | SBOMs centralizados, SCA contínuo, playbooks de resposta a vulnerabilidades, métricas de SLA para correção.[^10][^11] |

## 11. Tabela de fontes e limitações

| Fonte / padrão | Tipo | Escopo principal | Limitações relevantes |
| --- | --- | --- | --- |
| NIST SP 800‑218 (SSDF)[^11][^7] | Padrão NIST | Práticas de desenvolvimento seguro e supply chain em alto nível | Não prescreve ferramentas específicas, exige mapeamento para contexto de cada organização. |
| CISA ESF & guias de supply chain[^36][^8][^10] | Órgão governamental (CISA/NSA) | Recomendações práticas para clientes, fornecedores e desenvolvedores, com foco em SBOM e gestão de vulnerabilidades | Orientado a mercado norte‑americano, parte das recomendações assume maturidade de processos e ferramentas nem sempre disponível em equipes pequenas. |
| OWASP Top 10 2025 e DevSecOps/SCA[^3][^20][^4] | Comunidade de segurança | Classificação de riscos (incluindo supply chain), definição de SCA e orientações DevSecOps | Ferramentas citadas são exemplos, não lista exaustiva; algumas práticas requerem adaptação para pipelines modernos (containers, serverless). |
| SLSA framework e SLSA Provenance[^2][^5][^16] | OpenSSF / padrão aberto | Níveis de integridade de build, requisitos de proveniência, formato de attestation | Versões do padrão ainda em evolução; alcançar níveis 3–4 pode ser oneroso para equipes pequenas ou legados complexos. |
| Sigstore (Fulcio, Rekor, cosign)[^9][^17] | Projeto OSS (Linux Foundation) | Assinatura keyless baseada em OIDC, logs de transparência, verificação de artefatos e atestações | Infraestrutura hospedada pública pode não atender requisitos de soberania de dados; operação self‑hosted adiciona complexidade operacional. |
| SPDX 2.3 & CycloneDX 1.6[^29][^30][^33][^32] | Padrões abertos (Linux Foundation, OWASP) | Formatos de SBOM amplamente adotados, com foco em licenças, segurança e interoperabilidade | Ferramentas de geração/consumo com níveis variados de maturidade; nem todo fornecedor entrega SBOMs completos ou atualizados. |
| Guias de hardening de runners e CI/CD[^14][^18][^15][^27][^28] | Melhores práticas técnicas | Segurança de runners, isolamento, OIDC, permissões de workflow, action pinning | Nem todos os recursos existem de forma idêntica em todas as plataformas de CI/CD; requer leitura cuidadosa da documentação de cada provedor. |
| Documentos e análises de dependency confusion/typosquatting[^12][^13][^26][^37] | Pesquisas técnicas e relatos de incidentes | Entendimento detalhado de dependency confusion, typosquatting e técnicas de exploração | Baseados em incidentes específicos (npm, PyPI, etc.); outros ecossistemas podem ter nuances diferentes de resolução e mitigação.

---

## References

1. [NIST Special Publication (SP) 800-218, Secure Software Development Framework (SSDF) Version 1.1: Recommendations for Mitigating the Risk of Software Vulnerabilities](https://csrc.nist.gov/pubs/sp/800/218/final) - Few software development life cycle (SDLC) models explicitly address software security in detail, so...

2. [SLSA • Security levels](https://slsa.dev/spec/v0.1/levels) - Provenance at SLSA 1 does not protect against tampering, but it offers a basic level of code source ...

3. [A03 Software Supply Chain Failures - OWASP Top 10:2025](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/) - A03:2025 Software Supply Chain Failures icon. Background. This was top-ranked in the Top 10 communit...

4. [Introduction - OWASP Top 10:2025](https://owasp.org/Top10/2025/0x00_2025-Introduction/) - OWASP Top 10:2025

5. [Provenance](https://slsa.dev/spec/v1.1/provenance) - Description of SLSA provenance specification for verifying where, when, and how something was produc...

6. [Software Supply Chain Security Guidance | NIST](https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-supply-chain-security-guidance) - Software is a critical component of the larger challenge of managing cybersecurity related to supply...

7. [Secure Software Development Framework | CSRC](https://csrc.nist.gov/projects/ssdf) - NIST has finalized SP 800-218A, Secure Software Development Practices for Generative AI and Dual-Use...

8. [Enduring Security Framework](https://csrc.nist.gov/csrc/media/Presentations/2024/enduring-security-framework/images-media/20240918_SSCA_ESF%20Software%20Supply%20Chain_MODIFIED_PP-24-3646%20%20-%20%20Read-Only.pdf)

9. [Overview](https://docs.sigstore.dev/cosign/signing/overview/) - This document explains how identity-based, or “keyless” signing works in Sigstore. To learn more abo...

10. [[PDF] Establishing a Common Software Bill of Materials (SBOM) - CISA](https://www.cisa.gov/sites/default/files/2024-10/SBOM%20Framing%20Software%20Component%20Transparency%202024.pdf)

11. [Secure Software Development Framework (SSDF) Version 1.1: - NIST](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf)

12. [Dependency Confusion: How I Hacked Into Apple, Microsoft ...](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610) - The Story of a Novel Supply Chain Attack

13. [Dependency Confusion Attack: Detection and Prevention | SecPortal](https://secportal.io/vulnerabilities/dependency-confusion) - Dependency confusion explained: how attackers hijack internal package names via public registries, w...

14. [Self-Hosted Runners Security Review Pack](https://www.product-security.expert/07-ci-cd-and-software-supply-chain/self-hosted-runners-security-review-pack) - When a workflow or pipeline job runs on a self-hosted runner, repository-defined code can often:

15. [Runner and Environment Security - CI/CD Pipeline Hardening](https://devops-daily.com/guides/pipeline-hardening/02-runner-environment-security) - Learn to secure CI/CD runners with isolation, hardening, and ephemeral environments to prevent compr...

16. [SLSA Provenance](https://safeguard.sh/concepts/slsa-provenance) - Signed attestations that prove how a build artifact was produced.

17. [Software Provenance with Sigstore & in-toto - beefed.ai](https://beefed.ai/en/software-provenance-sigstore-in-toto) - Practical guide to signing builds and recording attestations using Sigstore (cosign, fulcio, rekor) ...

18. [GitHub Actions Security: The Definitive Guide - Secure Pipelines](https://secure-pipelines.com/ci-cd-security/github-actions-security-definitive-guide/) - The definitive guide to GitHub Actions security. Covers workflow permissions, action pinning, secret...

19. [Software Supply Chain Attacks & Dependency Tampering](https://owasp.org/www-project-mcp-top-10/2025/MCP04-2025%E2%80%93Software-Supply-Chain-Attacks&Dependency-Tampering) - OWASP Top Ten · OWASP Application Security Verification Standard · OWASP GenAI Security Project · OW...

20. [OWASP DevSecOps Guideline - v-0.2 | OWASP Foundation](https://owasp.org/www-project-devsecops-guideline/latest/02d-Software-Composition-Analysis) - Software Component/Composition Analysis (SCA). Component Analysis is the process of automating appli...

21. [Software Security in Supply Chains: Enhanced Vendor Risk Assessments](https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-supply-chain-security-guidance-21) - The EO creates higher standards for software verification techniques and other software supply chain...

22. [Configure OpenID Connect with GCP Workload Identity ...](https://docs.gitlab.com/ci/cloud_services/google_cloud/) - This tutorial demonstrates authenticating to Google Cloud from a GitLab CI/CD job using a JSON Web T...

23. [Implementing Code Signing | Security Alliance](https://frameworks.securityalliance.org/devsecops/code-signing/) - Verify code integrity with GPG-signed commits and pull requests. Best practices for key management, ...

24. [www-project-devsecops-verification-standard/document/CODE-005-Software-Composition-Analysis-SCA.md at main · OWASP/www-project-devsecops-verification-standard](https://github.com/OWASP/www-project-devsecops-verification-standard/blob/main/document/CODE-005-Software-Composition-Analysis-SCA.md) - OWASP Foundation Web Respository. Contribute to OWASP/www-project-devsecops-verification-standard de...

25. [What Is Software Composition Analysis? SCA Guide 2026](https://www.augmentcode.com/guides/what-is-software-composition-analysis) - Learn what software composition analysis is, how SCA differs from SAST and DAST, and how reachabilit...

26. [Anatomy of an npm Dependency Confusion Attack](https://safeguard.sh/resources/blog/anatomy-of-an-npm-dependency-confusion-attack) - How dependency confusion attacks exploit npm registry resolution order, grounded in Alex Birsan's 20...

27. [GitHub Actions Runner Controller Security: Ephemeral ...](https://www.systemshardening.com/articles/cicd/actions-runner-controller-security/) - Actions Runner Controller (ARC) scales GitHub Actions self-hosted runners as Kubernetes pods. Each r...

28. [GitHub Actions Self-Hosted Runner Hardening](https://www.systemshardening.com/articles/cicd/github-actions-self-hosted-runner/) - Self-hosted runners execute arbitrary workflow code on infrastructure you own. Hardening the runner ...

29. [SPDX Specification 2.3.0](https://spdx.github.io/spdx-spec/v2.3/) - The Software Package Data Exchange® (SPDX®) Specification Version 2.3. Copyright © 2010-2022 Linux F...

30. [Annex I Differences from previous editions (Informative)](https://spdx.github.io/spdx-spec/v2.3/diffs-from-previous-editions/)

31. [The System Package Data Exchange (SPDX) specification ...](https://github.com/spdx/spdx-spec/) - The System Package Data Exchange™ (SPDX®) specification is an open standard designed to represent sy...

32. [CycloneDX Response to NTIA SBOM Guidance](https://www.ntia.gov/sites/default/files/publications/cyclonedx_-_2021.06.17_0.pdf)

33. [CycloneDX v1.6 JSON Reference](https://cyclonedx.org/docs/1.6/json/) - CycloneDX v1.6 JSON Reference

34. [What's new in SPDX 2.3?](https://www.chainguard.dev/unchained/whats-new-in-spdx-2-3) - SPDX 2.3 introduces new features and enhancements for improved software supply chain transparency an...

35. [Provenance & Standards - Confidential AI](https://confidential.ai/docs/attested-builds/provenance-standards) - How attested builds produce verifiable claims and fit into SLSA and in-toto supply-chain standards.

36. [Securing the Software Supply Chain](https://www.cisa.gov/resources-tools/resources/securing-software-supply-chain-recommended-practices-guide-customers-and) - Best practices for software customers on procuring and deploying secure software, with guidance for ...

37. [Исследователь скомпрометировал 35 компаний через новую ...](https://xakep.ru/2021/02/11/dependency-confusion/) - ИБ-эксперт Алекс Бирсан рассказал о новой проблеме, получившей название dependency confusion (путани...

