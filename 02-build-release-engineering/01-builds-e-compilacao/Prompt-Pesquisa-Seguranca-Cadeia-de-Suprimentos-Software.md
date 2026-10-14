<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Realize uma pesquisa aprofundada sobre segurança da cadeia de suprimentos de software, com foco em builds, dependências, pipelines, artefatos e distribuição.

Considere informações, padrões e documentação disponíveis até agosto de 2026.
OBJETIVO
Identificar ameaças e controles necessários para garantir que o código revisado seja o mesmo código compilado, assinado e distribuído.
Investigue:
software supply chain;
dependency confusion;
typosquatting;
pacotes comprometidos;
dependências transitivas;
lockfiles;
SCA;
SAST;
DAST;
secret scanning;
license scanning;
SBOM;
SPDX;
CycloneDX;
provenance;
SLSA;
Sigstore;
assinatura de artefatos;
code signing;
checksums;
proteção de branches;
isolamento de runners;
runners efêmeros;
least privilege;
OpenID Connect;
rotação de credenciais;
proteção dos arquivos de pipeline;
revisão de dependências;
resposta a CVEs;
revogação de releases.
Priorize:
Padrões oficiais.
Órgãos governamentais e instituições de segurança.
Documentação oficial dos projetos citados.
Estudos técnicos reconhecidos.
Não use marketing de fornecedor como única base.
Investigue:
Como um invasor pode comprometer a build.
Como uma dependência comprometida entra no produto.
Como proteger credenciais.
Como evitar alteração não autorizada no pipeline.
Como comprovar origem do artefato.
Como gerar e utilizar uma SBOM.
Como assinar e verificar artefatos.
Como responder a uma dependência vulnerável.
Como revogar uma release.
Como aplicar controles proporcionais ao risco.
FORMATO DA RESPOSTA
Entregue:
Resumo executivo.
Modelo de ameaças.
Fluxo seguro.
Controles preventivos.
Controles detectivos.
Controles corretivos.
Estratégia mínima para equipe pequena.
Estratégia para produto empresarial.
Checklist de pipeline.
Checklist de artefato.
Checklist de resposta a comprometimento.
Mapa risco → controle.
Tabela de fontes e limitações.
Inclua citações junto às afirmações.

Segue um panorama direto das recomendações, depois você pode abrir o relatório em anexo para o conteúdo completo.

## Resumo executivo

Para garantir que o código revisado é o mesmo que é compilado, assinado e distribuído, você precisa combinar três blocos: (1) governança e proteção de código (SSDF, OWASP), (2) integridade de build e artefato (SLSA, proveniência, assinatura) e (3) visibilidade e resposta (SBOM, SCA/SAST/DAST, processos de resposta a CVEs e revogação de releases). Órgãos como NIST e CISA, junto com a comunidade OpenSSF/OWASP, já oferecem padrões claros (SSDF, SLSA, SBOM com SPDX/CycloneDX, Sigstore) que permitem rastrear e atestar, de forma criptográfica, a cadeia completa: commit → pipeline → artefato → deploy.[^1_1][^1_2][^1_3][^1_4][^1_5][^1_6]

No seu contexto (time técnico enxuto com stack moderna e forte uso de CI/CD), um baseline pragmático é: pinagem de dependências com lockfiles + proxy de pacotes, runners efêmeros com OIDC para credenciais de nuvem, branch protection forte (commits assinados, CODEOWNERS para workflows), geração automática de SBOM + proveniência SLSA em builds de release, e assinatura de artefatos com Sigstore/cosign.[^1_7][^1_8][^1_9]

Abaixo um resumo dos principais blocos solicitados; o relatório anexo traz o modelo de ameaças detalhado, fluxos e tabelas.

***

## Modelo de ameaças (visão condensada)

Principais riscos ao longo da cadeia de suprimentos, com foco em como o atacante entra:

- **Código/SCM**: takeover de conta de dev, commit malicioso, alteração de arquivos de pipeline (.github, .gitlab-ci.yml) para pular checagens ou exfiltrar segredos.[^1_10][^1_11]
- **Dependências**:
    - dependency confusion (pacote público com mesmo nome de pacote interno e versão maior);[^1_12][^1_8]
    - typosquatting (nomes parecidos com libs populares);[^1_12]
    - pacotes upstream comprometidos ou abandonados;[^1_5]
    - dependências transitivas vulneráveis.[^1_5]
- **CI/CD e runners**: runners persistentes utilizados por jobs de diferentes níveis de confiança; vazamento de segredos via env/logs; jobs de PR rodando com credenciais de produção; alteração não autorizada do pipeline.[^1_9][^1_13]
- **Artefatos/registries**: troca de imagem/binário depois do build; registry sem controle de acesso; ausência de assinatura.[^1_14][^1_15]
- **Distribuição/updates**: canal de atualização sem verificação de assinatura; falta de mecanismo de revogação/rollback de releases.[^1_16][^1_17]

***

## Fluxo seguro (alto nível)

Um fluxo de build alinhado a SSDF + SLSA que você pode usar como referência:

1. **Código e revisão**
    - Branches principais (`main`, `release`) protegidos com commits assinados, revisão obrigatória e CODEOWNERS para pipelines e scripts de deploy.[^1_11][^1_10]
    - SAST + secret scanning + lint de workflows em cada PR.[^1_2][^1_18]
2. **Dependências**
    - Manifest + lockfile obrigatório em todos os projetos;
    - uso de registry/proxy interno (Artifactory/Nexus/Verdaccio) como única origem;
    - SCA + license scanning para gerar SBOM (SPDX ou CycloneDX) a cada build de release.[^1_18][^1_19]
3. **Build**
    - Rodar em runner efêmero isolado (VM/pod one‑shot) com rede mínima e sem chaves estáticas;[^1_13][^1_9]
    - gerar artefato, SBOM e **proveniência SLSA** como attestation.[^1_3][^1_6]
4. **Assinatura e publicação**
    - Assinar artefato e atestações (SBOM, proveniência) com Sigstore/cosign (idealmente “keyless” via OIDC);[^1_15][^1_7]
    - publicar em registry seguro com controle de acesso.
5. **Distribuição e deploy**
    - Pipelines de deploy (e admission controller em k8s) verificam assinaturas e proveniência;[^1_14][^1_15]
    - monitorar SBOMs em produção contra novos CVEs e manter playbooks de resposta e revogação de releases.[^1_17][^1_16]

***

## Controles preventivos (essenciais)

### Código, branches e pipelines

- Branch protection forte: commits assinados, revisões obrigatórias, CODEOWNERS para `.github`, `.gitlab-ci.yml`, scripts de build/deploy.[^1_10][^1_11]
- Proibir push direto em `main`/`release`; apenas via PR revisado.
- Pinagem de actions/plugins por SHA de commit, nunca `@vX` ou `@latest`.[^1_10]


### Dependências

- Lockfiles obrigatórios + política de não atualizar dependência sem PR e review (incluindo diff de changelog/SECURITY.md).[^1_20][^1_8]
- Proxy de dependências como único caminho para npm/pip/maven/nuget; sem acesso direto do runner à internet para registries públicos.[^1_8]
- Escopos/namespaces internos (`@suaorg/*`, `com.suaorg.*`) roteados apenas para registry privado; não há fallback para público; mitigação forte de dependency confusion.[^1_21][^1_8]
- Política para novos pacotes externos (first‑use) exigir aprovação SCA + verificação de reputação.


### Runners, isolamento e credenciais

- Runners **efêmeros por padrão**; evitar runners persistentes compartilhados.[^1_22][^1_9]
- Separar runners por nível de confiança (PRs, build interno, release, deploy) e por escopo de rede (sem acesso a produção em jobs de build/test).[^1_9][^1_13]
- Bloquear acesso desnecessário a metadata de nuvem (IMDS), abrir só o mínimo de egress (SCM, registry, repositórios de dependência).[^1_13]
- Usar **OIDC + Workload Identity** para acessar GCP/AWS/Azure em vez de chaves estáticas; tokens curtos, escopo mínimo por job.[^1_23]
- Guardar qualquer segredo residual em vault e não em variables/plaintext; rotation periódica.


### Assinatura, SBOM e proveniência

- Gerar **SBOM** (SPDX ou CycloneDX) em todo build de release.[^1_24][^1_19]
- Gerar **proveniência SLSA** (in‑toto) descrevendo commit, builder, parâmetros de build e dependências resolvidas.[^1_6][^1_14]
- Assinar artefatos e atestações com Sigstore/cosign; preferir keyless (Fulcio + OIDC + Rekor) para fugir de key management manual.[^1_7][^1_15]

***

## Controles detectivos

- **SCA contínuo** atrelado à SBOM de cada release: correlacionar com NVD, advisories de fornecedores, KEV/CISA.[^1_16][^1_17]
- **SAST + DAST** integrados à pipeline, pelo menos em branches principais e releases.[^1_2][^1_5]
- **Secret scanning** sempre ligado (GitHub/GitLab, mais scanners adicionais se necessário) com playbook de rotação imediata.[^1_2][^1_10]
- Monitorar registries públicos por nomes internos (para detectar dependency confusion) e logs de transparência (Rekor) por assinaturas inesperadas com sua identidade.[^1_8][^1_7]

***

## Controles corretivos (quando algo deu errado)

- Processo formal de resposta a vulnerabilidades (RV no SSDF): detectar CVE → mapear impacto via SBOM → priorizar (criticidade + exposição) → mitigar (update, patch, feature flag) → registrar preventivos.[^1_16][^1_2]
- Mecanismo de **revogação de releases**:
    - marcar builds comprometidos em catálogo interno;
    - revogar/rotacionar chaves de assinatura;
    - bloquear download/uso da versão nos registries;
    - automatizar rollback/rollforward em produção.[^1_17][^1_7]
- Para comprometimento de pipeline/runner: destruir runners suspeitos, revogar tokens e chaves de CI, congelar pipelines críticos, auditar workflows e permissões antes de retomar.[^1_9][^1_13]

***

## Estratégia mínima para equipe pequena

Para um time enxuto, focar em:

1. **Dependency + build determinístico**
    - lockfiles;
    - repositório/proxy de dependências;
    - SCA básico (OWASP Dependency‑Check/Track).[^1_18][^1_8]
2. **CI/CD com higiene**
    - usar runners hospedados do provedor ou self‑hosted efêmeros;
    - branch protection + reviews + secret scanning;
    - tokens de CI com permissões mínimas; OIDC onde disponível.[^1_23][^1_10]
3. **SBOM + assinatura só em produção**
    - gerar SBOM + checksums + assinatura apenas para builds marcados como release;[^1_24][^1_7]
    - verificação manual ou automatizada antes de deploy.
4. **Playbook simples de resposta a CVEs**
    - planilha ou sistema simples mapeando: componente → versão → onde está em produção → plano de atualização.

***

## Estratégia para produto empresarial

Para um produto com exigência enterprise/compliance:

- Alinhar SDLC a **NIST SSDF** (incluindo práticas de “Protect the Software” e “Respond to Vulnerabilities”), e usar isso como linguagem comum com clientes corporativos.[^1_4][^1_2]
- Mirar **SLSA nível 3** para builds de produção: builder endurecido, proveniência não falsificável, revisão em dupla, isolamento forte da plataforma de build.[^1_3][^1_14]
- Exigir de fornecedores internos/terceiros:
    - SBOM em formato padrão;
    - evidências de práticas SSDF/SLSA;
    - processos claros de resposta a vulnerabilidades.[^1_25][^1_16]
- Ter uma função formal (AppSec/Supply Chain Security) responsável por políticas, ferramentas (SCA, SBOM, assinatura, runners), integração com GRC e incident response.[^1_17][^1_16]

***

## Checklists e mapa risco → controle

O relatório em anexo inclui:

- **Checklist de pipeline** (branch protection, settings de workflows, isolamento/ephemeral dos runners, uso de OIDC, proteção de arquivos de pipeline).[^1_13][^1_10]
- **Checklist de artefato** (SBOM, proveniência SLSA, assinatura com Sigstore, políticas de armazenamento e validação em deploy).[^1_6][^1_7]
- **Checklist de resposta a comprometimento** (detecção, contenção, erradicação, recuperação, uso de SBOM/proveniência na forense).[^1_6][^1_16]
- **Mapa risco → controle** relacionando dependency confusion, typosquatting, pacote upstream comprometido, pipeline/runner comprometido, troca de artefato, canal de update, vazamento de credenciais e falhas na resposta a CVEs aos controles específicos mais eficazes.[^1_8][^1_17]

***

Se quiser, posso adaptar esses blocos para um template concreto de GitHub Actions/GitLab CI (com exemplos de `permissions`, `id-token`, cosign, geração de SBOM CycloneDX e proveniência SLSA) ou para um playbook interno de TotalCAD com foco em plugins/instaladores Windows.
<span style="display:none">[^1_26][^1_27][^1_28][^1_29][^1_30][^1_31][^1_32][^1_33][^1_34][^1_35][^1_36][^1_37][^1_38][^1_39][^1_40][^1_41][^1_42][^1_43][^1_44][^1_45]</span>

<div align="center">⁂</div>

[^1_1]: https://csrc.nist.gov/pubs/sp/800/218/final

[^1_2]: https://slsa.dev/spec/v0.1/levels

[^1_3]: https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/

[^1_4]: https://owasp.org/Top10/2025/0x00_2025-Introduction/

[^1_5]: https://slsa.dev/spec/v1.1/provenance

[^1_6]: https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-supply-chain-security-guidance

[^1_7]: https://csrc.nist.gov/projects/ssdf

[^1_8]: https://csrc.nist.gov/csrc/media/Presentations/2024/enduring-security-framework/images-media/20240918_SSCA_ESF%20Software%20Supply%20Chain_MODIFIED_PP-24-3646%20%20-%20%20Read-Only.pdf

[^1_9]: https://docs.sigstore.dev/cosign/signing/overview/

[^1_10]: https://www.cisa.gov/sites/default/files/2024-10/SBOM Framing Software Component Transparency 2024.pdf

[^1_11]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf

[^1_12]: https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610

[^1_13]: https://secportal.io/vulnerabilities/dependency-confusion

[^1_14]: https://www.product-security.expert/07-ci-cd-and-software-supply-chain/self-hosted-runners-security-review-pack

[^1_15]: https://devops-daily.com/guides/pipeline-hardening/02-runner-environment-security

[^1_16]: https://safeguard.sh/concepts/slsa-provenance

[^1_17]: https://beefed.ai/en/software-provenance-sigstore-in-toto

[^1_18]: https://secure-pipelines.com/ci-cd-security/github-actions-security-definitive-guide/

[^1_19]: https://owasp.org/www-project-mcp-top-10/2025/MCP04-2025–Software-Supply-Chain-Attacks\&Dependency-Tampering

[^1_20]: https://owasp.org/www-project-devsecops-guideline/latest/02d-Software-Composition-Analysis

[^1_21]: https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-supply-chain-security-guidance-21

[^1_22]: https://docs.gitlab.com/ci/cloud_services/google_cloud/

[^1_23]: https://frameworks.securityalliance.org/devsecops/code-signing/

[^1_24]: https://github.com/OWASP/www-project-devsecops-verification-standard/blob/main/document/CODE-005-Software-Composition-Analysis-SCA.md

[^1_25]: https://www.augmentcode.com/guides/what-is-software-composition-analysis

[^1_26]: https://safeguard.sh/resources/blog/anatomy-of-an-npm-dependency-confusion-attack

[^1_27]: https://www.systemshardening.com/articles/cicd/actions-runner-controller-security/

[^1_28]: https://www.systemshardening.com/articles/cicd/github-actions-self-hosted-runner/

[^1_29]: https://spdx.github.io/spdx-spec/v2.3/

[^1_30]: https://spdx.github.io/spdx-spec/v2.3/diffs-from-previous-editions/

[^1_31]: https://github.com/spdx/spdx-spec/

[^1_32]: https://www.ntia.gov/sites/default/files/publications/cyclonedx_-_2021.06.17_0.pdf

[^1_33]: https://cyclonedx.org/docs/1.6/json/

[^1_34]: https://www.chainguard.dev/unchained/whats-new-in-spdx-2-3

[^1_35]: https://confidential.ai/docs/attested-builds/provenance-standards

[^1_36]: https://www.cisa.gov/resources-tools/resources/securing-software-supply-chain-recommended-practices-guide-customers-and

[^1_37]: https://xakep.ru/2021/02/11/dependency-confusion/

[^1_38]: https://www.cisa.gov/resources-tools/resources/nist-sp-800-218-secure-software-development-framework-v11-recommendations-mitigating-risk-software

[^1_39]: https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-218.pdf

[^1_40]: https://www.securebydesignhandbook.com/docs/standards/us/nist-sp-800-218-ssdf-overview

[^1_41]: https://cloudsmith.com/blog/slsa-a-route-to-tamper-proof-builds-and-secure-software-provenance

[^1_42]: https://applicationsecurityauthority.com/software-composition-analysis

[^1_43]: https://www.rappler.com/technology/alex-birsan-dependency-confusion-exploit-hack-bug-bounties/

[^1_44]: https://www.youngju.dev/blog/devops/2026-03-05-devops-github-actions-self-hosted-runner-ops.en

[^1_45]: https://www.youtube.com/watch?v=orIhfkK_FgA

