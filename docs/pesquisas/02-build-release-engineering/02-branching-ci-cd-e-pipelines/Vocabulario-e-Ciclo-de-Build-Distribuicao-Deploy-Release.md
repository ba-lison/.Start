## Resumo executivo

O ciclo que leva código-fonte até o usuário final envolve uma cadeia de termos frequentemente usados como sinônimos, mas que descrevem etapas distintas e independentes entre si: **build** (transformar código em artefato executável), **package/bundle** (empacotar o artefato e suas dependências), **publish** (armazenar o artefato versionado em um repositório), **deploy** (colocar o artefato em execução em um ambiente) e **release** (tornar uma funcionalidade disponível para usuários finais). A confusão nasce porque, historicamente, essas ações ocorriam simultaneamente — quando um deploy em produção coincidia sempre com a exposição da funcionalidade ao usuário. Práticas modernas de entrega contínua e feature flags desacoplaram essas etapas: hoje é comum haver deploy sem release ("dark launch"), release sem novo deploy (apenas ativação de uma flag) e build sem distribuição (nightly builds internos).[^1][^2][^3][^4]

Jez Humble e Martin Fowler formalizaram a distinção entre **Continuous Integration**, **Continuous Delivery** e **Continuous Deployment**: CI garante integração frequente do código; Continuous Delivery garante que o software esteja sempre em estado "liberável" (releasable) para produção; Continuous Deployment automatiza a liberação de toda mudança aprovada para produção, sem intervenção humana. Termos como **rollback**, **roll-forward**, **hotfix**, **patch**, **update** e **upgrade** descrevem estratégias e granularidades de correção após um release, e seus significados variam conforme o produto — aplicação web, app mobile, biblioteca, firmware embarcado ou instalador desktop possuem ciclos de release com peculiaridades próprias, especialmente quanto a versionamento (SemVer), lojas de aplicativos, tracks de staged rollout e mecanismos de rollback A/B em dispositivos embarcados.[^5][^6][^7][^8][^9][^10]

## Glossário completo

**Source code (código-fonte):** conjunto de instruções escritas por humanos em uma linguagem de programação, ainda não transformado em forma executável; é a entrada (input) do processo de build.[^11]

**Compile (compilar):** tradução de código-fonte para código objeto ou linguagem intermediária/de máquina por um compilador; é uma etapa dentro do processo de build, não o processo completo.[^12]

**Build (construir):** ato de combinar código-fonte e suas dependências para produzir um produto executável (artefato), incluindo compilação, linkedição e, frequentemente, empacotamento e testes. Um "build" também designa o resultado desse processo (um build específico, numerado ou versionado).[^13][^12]

**Build system (sistema de build):** ferramenta/programa que automatiza a construção de software a partir de um grafo de dependências entre arquivos de entrada e saída, decidindo o que precisa ser reconstruído. Exemplos: Make, CMake, Ninja, Bazel, Gradle, MSBuild. Um sistema de build define **targets** e resolve dependências estáticas ou dinâmicas entre eles.[^14][^13]

**Build target (alvo de build):** unidade nomeada que o sistema de build sabe produzir — um arquivo, biblioteca, executável ou meta "phony" (sem saída em disco, usada para agrupar outros targets). Em Bazel/CMake, um target é declarado com suas fontes e dependências explícitas.[^15][^14]

**Build configuration (configuração de build):** conjunto de parâmetros (arquitetura de CPU, modo debug/release, flags de compilador, variáveis de ambiente) que determina como um target é construído; a mesma base de código pode gerar artefatos distintos conforme a configuração. Em Bazel, configurações são identificadas por hash e não possuem label próprio — só existem durante a fase de análise.[^16]

**Package (pacote):** unidade de distribuição específica de uma linguagem ou ecossistema (um `.jar`, `.whl`, `.tar.gz`, um pacote npm) que agrupa código compilado/interpretado, metadados e dependências, geralmente publicado em um gerenciador de pacotes.[^17][^11]

**Bundle (pacote agregado):** resultado de um processo de "bundling" que consolida múltiplos módulos/arquivos-fonte (e por vezes assets) em um número reduzido de arquivos de saída, tipicamente para o navegador (ex.: Webpack, Rollup, esbuild); "bundle" enfatiza a fusão de módulos, enquanto "package" enfatiza a distribuição via gerenciador.[^17]

**Binary (binário):** arquivo executável compilado, em código de máquina, resultante direto da compilação/linkedição — pode ser um artefato completo em si (ex.: um `.exe`) ou parte de um pacote maior.[^11]

**Artifact (artefato):** conceito guarda-chuva para qualquer saída versionada e imutável produzida por um passo de build, destinada a ser consumida por um passo de deploy — pode ser um binário, uma imagem de contêiner, um pacote de linguagem, um bundle estático ou uma imagem de SO. "O artefato é o contrato entre CI e CD": CI termina seu trabalho ao publicar o artefato; CD começa a partir dele.[^1][^11]

**Publish (publicar):** ação de enviar o artefato compilado para um repositório/registro (Artifactory, npm registry, Docker Hub, Maven repository) sob uma versão imutável; após publicado, os bytes do artefato nunca mudam.[^18][^1]

**Distribution (distribuição):** processo mais amplo de tornar o software disponível a quem vai instalá-lo ou executá-lo — pode envolver publicação em lojas de app, CDN, mídia física, ou repositórios internos; nem toda distribuição implica deploy imediato (ex.: instaladores desktop ficam disponíveis para download sem "rodar" em lugar algum controlado pelo fornecedor).

**Delivery (entrega):** no contexto de "Continuous Delivery", entrega significa que o software está pronto e verificado para ser colocado em produção a qualquer momento — a decisão de liberar é de negócio, não técnica.[^6][^19]

**Deployment (deploy/implantação):** processo de colocar um artefato executável (JAR, WAR, script, imagem de contêiner) em execução em um ambiente-alvo — servidor, cliente, dispositivo; é um evento técnico controlado por equipes de engenharia/operações.[^3][^18]

**Release (versão/liberação):** o momento em que usuários finais obtêm acesso a uma nova funcionalidade ou versão, servindo a um objetivo de negócio; pode envolver múltiplos deploys e nem sempre afeta todos os usuários simultaneamente; frequentemente conduzido por product/release management.[^3][^12]

**Promotion (promoção):** movimentação do mesmo artefato, sem reconstrução, através de uma cadeia de ambientes (dev → staging → produção); é uma operação de metadados (mudança de tag, rota, spec), não um novo build — princípio de "build once, deploy many".[^20][^1]

**Rollout:** ação de implantar gradualmente uma nova versão (ex.: rolling update no Kubernetes, staged rollout em lojas de apps) sem downtime, aumentando progressivamente a fração de usuários ou instâncias expostas.[^21][^22]

**Rollback:** reversão para a versão anterior conhecida como estável, desfazendo por completo uma atualização problemática; no Kubernetes, `kubectl rollout undo` restaura o template de uma revisão anterior.[^23][^24][^21]

**Roll-forward:** estratégia de corrigir um problema publicando uma nova versão com a correção, em vez de reverter à versão anterior — mantém as mudanças boas do release, mas depende de diagnosticar e corrigir antes de implantar.[^25][^23]

**Hotfix:** correção urgente e cirúrgica aplicada fora do ciclo normal de release para resolver um problema específico e crítico, geralmente preservando as demais mudanças já entregues.[^23][^25]

**Patch:** em versionamento semântico, incremento que corrige bugs de forma retrocompatível, sem alterar a API pública (terceiro número em X.Y.Z). Também usado genericamente para qualquer atualização pequena e corretiva.[^8][^26]

**Update (atualização):** termo genérico para qualquer nova versão distribuída a um usuário/dispositivo, podendo incluir correções, melhorias de segurança ou pequenas funcionalidades; em contexto de OTA (over-the-air) refere-se ao processo de entregar, verificar e instalar novo firmware remotamente.[^27]

**Upgrade:** normalmente denota uma mudança de versão maior, com adição de funcionalidades ou mudança de capacidades, por vezes exigindo ação do usuário ou migração de dados — a linha entre "update" e "upgrade" é convencional, não universal.

**Release candidate (RC):** build funcionalmente completo, considerado pronto para produção pendente uma rodada final de verificação; a mesma imagem binária é promovida por ambientes e testes de aceitação, sendo aprovada como release final ou rejeitada em favor de um novo RC. É "congelado": não recebe novos recursos, só correções pontuais.[^28][^29]

**Nightly build:** build automatizado e agendado (tipicamente noturno) que compila o estado atual do branch principal, usado para detectar regressões de integração precocemente; é descartável e geralmente não é distribuído a usuários externos.[^30][^31][^28]

## Diagrama textual do ciclo

```
[código-fonte] 
      │  (edição, commit)
      ▼
[compilação / build]  ──►  falha ──► volta ao código-fonte
      │  (sucesso: gera binário/pacote)
      ▼
[testes automatizados] (unitários, integração, segurança)
      │  (sucesso)
      ▼
[artefato] (imutável, versionado: jar, imagem docker, ipa/apk, instalador)
      │
      ▼
[armazenamento] (registro/repositório: Artifactory, npm registry, Docker Hub, App Store binário)
      │
      ▼
[promoção] (mesmo artefato avança por ambientes: dev → staging → pré-produção)
      │  (aprovação manual ou automática / release candidate)
      ▼
[deploy] (artefato colocado em execução no ambiente-alvo — pode estar "dark", atrás de feature flag)
      │
      ▼
[release] (decisão de negócio: flag ativada, rollout gradual, publicação em loja liberada aos usuários)
      │
      ▼
[operação] (monitoramento, observabilidade, rollback/roll-forward/hotfix se necessário)
```

Esse fluxo mostra por que build, distribuição, deploy e release são etapas conceitualmente separáveis: cada seta representa uma decisão distinta que pode ser adiada, paralelizada ou pulada dependendo da arquitetura (feature flags, staged rollout, canary, promoção por ambientes).[^20][^1][^3]

## Tabela comparativa entre os termos

| Termo | O que responde | Quem controla tipicamente | Repetível sem novo build? |
|---|---|---|---|
| Build | "O código compila e gera um artefato?" | Engenharia/CI | Não — é a origem do artefato[^12] |
| Package/Bundle | "Como o artefato é empacotado para consumo?" | CI/build tooling | Não, ocorre dentro do build[^17] |
| Publish | "O artefato está armazenado e versionado?" | CI | Não — ocorre uma vez por artefato[^1] |
| Promotion | "Este artefato já validado está avançando de ambiente?" | CD/release manager | Sim — reaproveita o mesmo artefato[^1] |
| Deploy | "A versão X está rodando no ambiente Y?" | Engenharia/operações/SRE | Sim — múltiplos deploys sem novo release visível[^2] |
| Release | "Os usuários podem acessar?" | Produto/negócio | Sim — sem novo deploy, via feature flag[^3] |
| Rollback | "Voltamos à última versão estável?" | Operações | Reaproveita artefato anterior[^21] |
| Roll-forward | "Corrigimos avançando com nova versão?" | Engenharia | Não — exige novo build corretivo[^25] |
| Hotfix | "Corrigimos um problema pontual fora do ciclo normal?" | Engenharia | Geralmente exige build pequeno e dedicado[^23] |
| Patch (SemVer) | "Corrigimos bug sem quebrar API?" | Engenharia/versionamento | Novo build com incremento no terceiro dígito[^8] |

## Diferenças por tipo de produto

| Contexto | Build | Distribuição/Deploy | Release |
|---|---|---|---|
| Aplicação web | Bundle JS/CSS, artefato de contêiner | Deploy em servidores/K8s, promoção entre ambientes[^20] | Feature flag ativada; rollout blue-green/canário[^3] |
| API/microsserviço | Build gera imagem de contêiner versionada | Deploy via orquestrador (K8s rollout)[^21] | Endpoint ativo para consumidores; versionamento de API |
| Aplicação desktop (instalador) | Build gera executável/instalador (.exe, .dmg, .msi) | Distribuição via download/CDN; nem sempre há "deploy" controlado pelo fornecedor | Release = disponibilização pública do instalador; usuário decide quando "atualizar" |
| Mobile app | Build gera .ipa/.apk/.aab | Upload à loja e enfileiramento em "tracks" (internal, alpha, beta, production)[^9][^32] | Staged/phased rollout controla % de usuários que recebem a versão[^22][^33] |
| Biblioteca/SDK | Compilação + empacotamento (jar, wheel, npm tarball) | Publish no gerenciador de pacotes (npm, Maven Central, PyPI) | Release = nova versão SemVer publicamente consumível por outros projetos[^8] |
| Plugin | Build gera binário compatível com host (CAD, browser, IDE) | Distribuição via marketplace do host ou instalação manual | Release = versão aprovada/listada no marketplace |
| Software CAD (ex.: ZWCAD/SketchUp) | Build de instalador multiplataforma, com dependências nativas e módulos de licenciamento | Distribuição via instalador/atualizador; deploy geralmente manual pelo usuário/TI corporativa | Release = versão numerada com notas; hotfixes para bugs críticos de licenciamento |
| Software embarcado/firmware | Build de imagem binária para arquitetura específica | Deploy via OTA com partição A/B; verificação criptográfica antes de rollout[^10][^34] | Release = disponibilização da imagem para a frota; rollback automático se healthcheck falhar[^34] |
| Produto entregue por instalador | Build gera pacote instalável assinado | Distribuição via site/loja; sem controle direto sobre "quando" o cliente instala | Release = data de disponibilização pública; patches subsequentes via updater |

## Diferença entre build, package, publish, deploy e release

Build transforma código em artefato; package/bundle empacota esse artefato com suas dependências em formato consumível; publish armazena esse pacote versionado e imutável em um repositório; deploy coloca esse pacote em execução em um ambiente-alvo; release é a decisão — técnica ou de negócio — de expor a funcionalidade correspondente aos usuários finais. A separação crucial, enfatizada por engenheiros de plataformas como LaunchDarkly e Resend, é que **deploy é um evento técnico** ("o código está rodando em produção?") enquanto **release é um evento de produto/negócio** ("os usuários conseguem ver isso?").[^35][^12][^1][^3]

## Como pode existir deploy sem release

Ao usar feature flags, uma equipe pode implantar código em produção mantendo-o inativo ou invisível — a técnica chamada **"dark launch"** ou **"deploy dark"**. O código está fisicamente presente e rodando nos servidores de produção, mas nenhum usuário o acessa até que a flag seja ativada, dias ou semanas depois, sem necessidade de novo pipeline de deploy. Um exemplo documentado no ecossistema Salesforce mostra uma funcionalidade que foi deployada em abril e só liberada a clientes em setembro seguinte.[^36][^37][^38][^3]

## Como pode existir release sem um novo deploy

Se o código de uma funcionalidade já foi deployado atrás de uma flag, "liberar" essa funcionalidade é apenas alternar o valor da flag em um painel — uma operação de configuração, sem redeploy, sem pipeline, sem novo artefato. Essa desconexão também permite reverter um release problemático instantaneamente desligando a flag, sem precisar reimplantar a versão anterior.[^4][^36][^35]

## Como pode existir build sem distribuição

Nightly builds e builds de CI executados a cada commit são gerados continuamente, mas frequentemente nunca chegam a usuários externos — servem apenas para detectar regressões de integração o quanto antes ("integrate early, integrate often"). Da mesma forma, artefatos intermediários (arquivos-objeto, builds de branch de feature) são produzidos e descartados sem nunca alcançar um repositório de distribuição pública.[^29][^31][^28]

## Como feature flags separam deployment de release

Feature flags atuam como um interruptor em tempo de execução avaliado em menos de 1 milissegundo, decidindo se um trecho de código roda ou não para determinado usuário/contexto, sem exigir redeploy. Isso rompe o acoplamento tradicional entre "código no servidor" e "código visível ao usuário": deployment passa a significar apenas entrega segura e contínua de código, enquanto release se torna uma decisão isolada, controlada por produto, com granularidade de percentuais de usuários, cohorts ou testes A/B. Riscos associados incluem "flag debt" — acúmulo de flags obsoletas que deveriam ser removidas após o release estabilizar.[^36][^35][^3]

## Por que o significado dos termos varia entre empresas e stacks

Diferentes ferramentas formalizam os mesmos conceitos com nomes distintos: o "target configuration" do Bazel não é o mesmo que "build configuration" no sentido genérico; "release" em npm/SemVer designa uma versão publicada no registro, enquanto em Kubernetes "release" pode se referir a uma revisão de Helm chart, e em lojas de apps designa uma entrada dentro de uma "track". A própria distinção deploy/release não é universalmente aceita: alguns times tratam ambos como sinônimos quando não usam feature flags, e a literatura aponta explicitamente essa confusão histórica entre Continuous Delivery e Continuous Deployment, mesmo entre autores reconhecidos. Cada organização adapta o vocabulário ao seu stack (Java/Maven usa "snapshot" vs "release repository"; mobile usa "tracks"; embarcados usam "partição A/B"), o que impede um glossário universal sem contexto explícito.[^19][^39][^9][^32][^34][^18][^16][^8]

## O que um desenvolvedor pode estar querendo dizer ao falar em "estrutura de build"

O termo é ambíguo e pode se referir a: (1) a arquitetura do build system (Makefiles, arquivos Bazel/CMake, pipeline de CI); (2) a organização dos build targets e suas dependências; (3) as configurações de build disponíveis (debug/release, arquiteturas, flags); (4) a hierarquia de diretórios de saída dos artefatos; ou (5) as etapas do pipeline (compilar, testar, empacotar, assinar). Sem contexto adicional, é necessário perguntar qual dessas dimensões o interlocutor tem em mente.[^15][^16][^13]

## O que um desenvolvedor pode estar querendo dizer ao falar em "estrutura de release"

Pode significar: (1) a estratégia de branching e tagging usada para releases (trunk-based, release branches, git-flow); (2) o esquema de versionamento adotado (SemVer, CalVer); (3) o processo de promoção entre ambientes e aprovações (release candidates, staging, gates); (4) a política de rollout (staged, canary, blue-green); ou (5) a governança organizacional de quem aprova e comunica um release (release notes, change management). Novamente, exige esclarecimento do escopo pretendido.[^40][^22][^18][^29][^21][^8]

## Exemplos reais ou conceituais

O Google Play Console ilustra deploy-sem-release-total: uma APK pode ser enviada à track "production" com staged rollout de apenas 5% dos usuários — todos os 5% já têm o artefato deployado, mas o "release" completo só ocorre quando o rollout atinge 100%. No lado embarcado, dispositivos IoT usam partição A/B: a nova imagem é baixada e fica pronta ("deploy" para uma partição inativa), mas só se torna a imagem ativa após o bootloader confirmar sucesso no boot — um rollback automático reverte silenciosamente se o healthcheck falhar. No Scala/Dotty, um release candidate é uma tag imutável promovida a release estável um ciclo depois, sem novo trabalho de desenvolvimento, apenas correções.[^22][^9][^10][^34][^40]

## Erros conceituais comuns

Tratar Continuous Delivery e Continuous Deployment como sinônimos é um erro recorrente mesmo em discussões técnicas — Continuous Delivery é a *capacidade* de deployar a qualquer momento; Continuous Deployment é *fazê-lo automaticamente a cada commit aprovado*. Outro erro é assumir que "deploy" sempre implica "release" — válido apenas em times sem feature flags ou canary releases. Também é comum confundir "package" (unidade de linguagem/ecossistema) com "bundle" (agregação de módulos para entrega), embora ambos sejam tipos de artefato. Por fim, alguns tratam "patch" e "hotfix" como idênticos, mas patch é uma classificação de versionamento (SemVer), enquanto hotfix descreve o *processo* urgente e fora do ciclo normal — um hotfix pode gerar um patch, mas nem todo patch é um hotfix.[^6][^19][^8][^3][^11][^23][^17]

## Terminologia interna recomendada para uma equipe

- **Build:** artefato numerado, gerado por CI a partir de um commit específico, imutável.
- **Release candidate:** build promovido a candidato, testado, sem novas features, aguardando aprovação.
- **Deploy:** ato técnico de colocar um artefato em execução em um ambiente nomeado (dev/staging/prod).
- **Release:** decisão registrada (com data e responsável) de expor uma funcionalidade a usuários, associada a flags e ao público-alvo.
- **Hotfix:** processo emergencial documentado, sempre seguido de um patch formal e retroativo ao branch de release.
- **Rollback/roll-forward:** ambos devem ter runbooks específicos e critérios de decisão pré-definidos (tempo estimado de correção vs. criticidade do incidente).[^25][^23]

## Checklist para identificar em qual estágio uma alteração se encontra

- O código foi mesclado ao branch principal? → etapa de **build**.
- Existe um artefato versionado, imutável, publicado em um registro? → etapa de **publish**.
- O artefato passou por promoção entre ambientes sem reconstrução? → etapa de **promotion**.
- O artefato está rodando em um ambiente de destino (mesmo que atrás de flag)? → etapa de **deploy**.
- Usuários finais (todos ou uma fração) já conseguem interagir com a funcionalidade? → etapa de **release**.
- Houve necessidade de reverter para uma versão anterior por falha? → **rollback**.
- Uma correção pontual foi publicada fora do ciclo normal? → **hotfix**.

## Perguntas a fazer quando alguém usa "estrutura de build" ou "estrutura de release"

Para "estrutura de build": Qual ferramenta/build system está em uso (Make, Bazel, Gradle, MSBuild)? A dúvida é sobre organização de targets, dependências, ou sobre configurações (debug/release, plataformas)? É sobre o pipeline de CI (etapas) ou sobre o layout de diretórios de saída?

Para "estrutura de release": A referência é à estratégia de branching/tagging, ao esquema de versionamento (SemVer?), ao processo de aprovação/promoção, à política de rollout (canário, staged, blue-green) ou à governança de comunicação (release notes, changelog)?

## Conclusão: o que um desenvolvedor sênior pode estar incluindo ao falar em "estrutura de build" e "estrutura de release"

Ao mencionar **"estrutura de build"**, um desenvolvedor sênior pode estar se referindo a qualquer combinação de: a escolha e configuração do build system (Make/CMake/Bazel/Gradle); a definição e hierarquia dos build targets e suas dependências estáticas/dinâmicas; as configurações de build (arquitetura, modo debug/release, variáveis de compilação); a organização dos artefatos gerados (binários, pacotes, bundles) e onde ficam armazenados; e as etapas do pipeline de CI que antecedem a publicação (compilação, testes, empacotamento, assinatura de artefatos).[^14][^16][^13][^1][^11][^15]

Ao mencionar **"estrutura de release"**, pode estar incluindo: a estratégia de branching e versionamento (trunk-based vs. release branches, SemVer); o fluxo de promoção de artefatos entre ambientes e os critérios de aprovação (release candidates, gates de qualidade); a política de rollout aos usuários (staged rollout, canary, blue-green, feature flags); os mecanismos de contingência (rollback automático, roll-forward, hotfix); e a governança organizacional — quem decide, comunica e documenta um release (release notes, change management, calendário de releases). Como esses termos carregam significados específicos por ferramenta, empresa e tipo de produto, a prática recomendada é sempre esclarecer explicitamente qual dessas dimensões está sendo discutida antes de assumir uma interpretação única.[^39][^29][^40][^19][^21][^8][^22][^3][^20][^25]

## Tabela de fontes

| Título | Autor/Organização | Data | Tipo | Autoridade | Contribuição | Limitações |
|---|---|---|---|---|---|---|
| Definitions Build, Deploy and Artifact Store[^18] | University of Colorado OIT | s.d. | Documentação institucional | Média (guia interno de padrões, não padrão da indústria) | Define deployment, artifact store e convenções Maven | Específico de uma organização; não é padrão universal |
| CI/CD & Deployment Glossary — Artifact[^1][^20] | glossary.deployment.to | s.d. | Glossário técnico especializado | Média-alta (conteúdo técnico consistente, sem afiliação institucional clara) | Define ciclo build→publish→promote→retire; distingue artifact de container image | Fonte não institucional; rotulada como glossário de blog técnico |
| Feature Flags as a Deployment Strategy (vídeo)[^36] | Canal YouTube técnico (não identificado institucionalmente) | 2026-05-22 | Vídeo educacional | Média (contexto técnico, mas não corrobora citando fontes primárias) | Explica "deploy dark" e tipos de feature flags | Rotulado como conteúdo de blog/vídeo; corroborado por LaunchDarkly |
| How we separate deployments from releases[^35] | Resend (empresa) | 2025-05-27 | Publicação técnica corporativa | Alta (experiência industrial documentada) | Define deploy vs release na prática de uma empresa real | Reflete prática de uma empresa específica, não universal |
| Deploy vs Release[^2] | Substack técnico (Dele) | 2026-01-01 | Blog/newsletter técnica | Baixa-média (rotulado como blog pessoal) | Cheat sheet prático deploy/release/promote | Fonte não institucional; usada apenas como reforço de conceito já corroborado |
| Deployment and release strategies[^3] | LaunchDarkly (documentação oficial) | s.d. | Documentação oficial de produto | Alta (fornecedor líder de feature flags) | Definição canônica de deployment vs release e dark launching | Possível viés comercial pró-feature-flags |
| What is Build Artifact?[^11] | devopsschool.org | 2026-02-21 | Artigo técnico educacional | Média | Tabela de termos relacionados a artifact (package, binary, release) | Fonte comercial educacional, não peer-reviewed |
| Flag Debt Is Real[^4] | devopsness.com (blog) | 2026-07-06 | Blog técnico | Baixa-média | Reforça separação deploy (técnico) vs release (negócio) | Rotulado como blog; usado apenas de forma corroborativa |
| Common terms in CI/CD[^12] | Medium (autor individual) | 2025-04-07 | Blog/artigo técnico | Baixa-média | Definições de build, deploy, release, release build | Fonte não institucional; conteúdo consistente com fontes mais fortes |
| Alpha vs Beta Apps and Nightly vs Production Builds[^28][^30] | Gist/Luciq.ai | 2019/2025 | Artigo técnico | Média | Define nightly build e release candidate | Não é padrão oficial; terminologia de mercado consolidada |
| What is a release candidate?[^29] | glossary.deployment.to | s.d. | Glossário técnico | Média-alta | Distingue nightly build de release candidate | Mesma limitação do glossário geral |
| Android internal_concepts.md[^15] | Google (AOSP/Android Open Source Project) | s.d. | Documentação oficial de engenharia | Alta | Define build target, module, phony goal no contexto Soong/Bazel | Específico ao ecossistema Android/Bazel |
| The Ninja build system (manual)[^14] | Ninja Build (projeto open source oficial) | s.d. | Documentação oficial de ferramenta | Alta | Define build target, build statement, default target | Específico à ferramenta Ninja |
| Bazel build configurations[^16] | Fuchsia.dev (Google) | s.d. | Documentação oficial de projeto | Alta | Define build configuration formalmente no contexto Bazel | Específico ao Bazel; conceito não idêntico em outros build systems |
| Build Systems: Bazel vs Make[^13] | Nalys Technical Blog | s.d. | Blog técnico corporativo | Média | Define build system, target, dependências estáticas/dinâmicas | Blog corporativo, não peer-reviewed |
| Continuous Integration[^5] | Martin Fowler (autor reconhecido) | 2024-01-17 | Artigo de autor reconhecido em engenharia de software | Alta | Define deployment pipeline e relação CI/CD | Artigo de opinião de especialista, sem revisão por pares formal |
| Continuous Delivery (bliki)[^6] | Martin Fowler | 2013-05-29 | Artigo de autor reconhecido | Alta | Distinção formal Continuous Delivery vs Continuous Deployment | Mesmo texto repetido/citado amplamente por terceiros |
| Continuous Delivery (livro)[^41] | Jez Humble & Dave Farley | s.d. | Livro técnico de referência | Alta (referência canônica da indústria) | Origem do conceito formal de Continuous Delivery | Não consultado em profundidade nesta pesquisa, apenas referenciado |
| What is Continuous Delivery and common misconceptions[^19] | Blog técnico (deors.wordpress.com) | 2024-01-10 | Blog técnico especializado | Média | Discute more_precisamente a relação CD/Continuous Deployment citando Humble/Farley | Blog pessoal; corrobora fontes primárias |
| CD Foundation Glossary[^7] | Continuous Delivery Foundation (CDF, sob Linux Foundation) | 2020-10-20 | Glossário oficial de fundação open source | Alta | Definições oficiais de Continuous Delivery e Continuous Deployment citando fontes primárias | Repositório comunitário, mas com governança institucional |
| Rolling Updates and Rollbacks in Kubernetes[^21] | Medium/Google Cloud (autor associado) | 2023-04-16 | Artigo técnico | Média-alta | Define rollout e rollback no contexto Kubernetes | Não é documentação oficial, mas alinhado a ela |
| Perform a Rollback on a DaemonSet[^24] | Kubernetes.io (documentação oficial) | 2021-02-10 | Documentação oficial de projeto open source | Alta | Define rollback e revision no Kubernetes | Específico ao Kubernetes |
| Decide on Hot-fix or Rollback Strategy[^25] | CloudPosse (documentação de referência) | 2024-10-31 | Documentação técnica corporativa | Média-alta | Compara hotfix, rollback e roll-forward | Documentação de uma empresa/framework específico |
| Rollback vs Hotfix[^23] | Bugnet (blog de produto) | 2026-03-09 | Blog corporativo de produto | Média | Define claramente rollback vs hotfix | Possível viés de produto (Bugnet) |
| Semantic Versioning 2.0.0[^8] | semver.org (especificação oficial mantida por Tom Preston-Werner e comunidade) | s.d. | Especificação técnica oficial | Alta (padrão de fato da indústria) | Define MAJOR.MINOR.PATCH formalmente | Especificação não é ISO/IEEE, mas amplamente adotada |
| Software versioning — Wikipedia[^42] | Wikipedia (comunidade) | 2004 (última rev. recente) | Enciclopédia colaborativa | Média | Contextualiza SemVer entre outros esquemas de versionamento | Fonte tercerizada; deve ser corroborada (feito via semver.org) |
| Release app updates with staged rollouts[^22] | Google Play Console (documentação oficial) | s.d. | Documentação oficial de plataforma | Alta | Define staged rollout, tracks de release no Android | Específico ao Google Play |
| APKs and Tracks — Google Play Developer API[^9] | Google (documentação oficial de API) | s.d. | Documentação oficial de API | Alta | Define tracks (alpha/beta/production) e staged rollout via API | Específico ao ecossistema Android |
| How to minimize impact of a faulty release with staged rollout[^33] | Bitrise (empresa de CI/CD mobile) | 2024-02-26 | Publicação técnica corporativa | Média-alta | Compara staged rollout Google Play vs phased release Apple | Conteúdo de fornecedor de CI/CD, mas factual e verificável |
| Secure OTA firmware updates with rollback[^27][^10] | Siliconlogix / Embedcrest (empresas de engenharia embarcada) | 2026 | Publicação técnica corporativa | Média-alta | Define OTA update, rollback e partição A/B em embarcados | Conteúdo comercial, mas tecnicamente consistente e corroborado |
| Google Play Tracks: Statuses, Rollout and Visibility[^32] | aso.dev (blog técnico especializado) | 2026-01-04 | Blog técnico | Média | Detalha diferença entre app status, release status e track status | Blog especializado, não documentação oficial, mas alinhado a ela |
| npm vs Webpack[^17] | StackShare (comunidade de comparação de ferramentas) | 2021-09-28 | Comparação técnica de plataforma | Média | Diferencia bundling de package management | Plataforma de comparação comunitária, não acadêmica |

---

## References

1. [Artifact - CI/CD & Deployment Glossary](https://glossary.deployment.to/artifact/) - A build artifact is the immutable, versioned package a CI pipeline produces - built once, deployed u...

2. [Deploy vs Release - by Dele - DevSecOps pour tous - Substack](https://thefrenchdevsecops.substack.com/p/deploy-vs-release) - Runtime vs user availability (and why feature flags changed everything)

3. [Deployment and release strategies](https://launchdarkly.com/docs/guides/infrastructure/deployment-strategies) - This guide explains the concepts behind deployment and release strategies made possible by feature f...

4. [Flag Debt Is Real And It...](https://www.devopsness.com/blog/feature-flags-decoupling-release-from-deploy)

5. [Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html) - Continuous Integration is a software development practice where each member of a team merges their c...

6. [Continuous Delivery](https://martinfowler.com/bliki/ContinuousDelivery.html) - Continuous Delivery is sometimes confused with Continuous Deployment. Continuous Deployment means th...

7. [glossary/definitions.md at main · cdfoundation/glossary](https://github.com/cdfoundation/glossary/blob/main/definitions.md) - Reference repository composed of timelines, lists and various other informative items related to the...

8. [Semantic Versioning 2.0.0](https://semver.org/) - Semantic Versioning spec and website

9. [APKs and Tracks | Google Play Developer API](https://developers.google.com/android-publisher/tracks) - Learn to add and modify APKs, draft releases, and name tracks, with a workflow example showing a sta...

10. [Delta Updates: Efficiency In...](https://www.rapidseasuite.com/blog/over-the-air-ota-update-frameworks) - A take on Over-The-Air (OTA) Update Frameworks, delta updates, rollback strategies and how RAPIDSEA ...

11. [What is Build Artifact? Meaning, Examples, Use Cases, and How to ...](https://devopsschool.org/blog/build-artifact/)

12. [Common terms in CI/CD](https://medium.com/@mojtaba.tavakolzadeh/common-terms-in-ci-cd-41c1ab90fb83) - The CI/CD process is entirely dependent on the logic of the organization or team and is not limited ...

13. [Build Systems: Bazel vs Make · Nalys Technical Blog](https://nalys-taas-projects.gitlab.io/internal/taas_blog/post/bazel_vs_make/) - Build systems define a set of targets that needs to be created. These targets can have some dependen...

14. [The Ninja build system](https://ninja-build.org/manual.html) - Ninja is yet another build system. It takes as input the interdependencies of files (typically sourc...

15. [Properties/Attributes](https://android.googlesource.com/platform/build/bazel/+/refs/heads/master/docs/internal_concepts.md)

16. [Bazel build configurations - Fuchsia.dev](https://fuchsia.dev/fuchsia-src/development/build/bazel_concepts/build_configurations)

17. [npm vs Webpack | What are the differences?](https://www.stackshare.io/stackups/npm-vs-webpack) - npm - The package manager for JavaScript.. Webpack - A bundler for javascript and friends.

18. [Definitions Build, Deploy and Artifact Store](https://oit.colorado.edu/sites/default/files/docs/Software%20Engineering%20Standards%20Guidelines-Development%20Projects.pdf)

19. [What is Continuous Delivery and some common ...](https://deors.wordpress.com/2024/01/11/what-is-continuous-delivery-and-some-common-misconceptions-about-it/) - Continuous deployment is therefore more properly defined as the ability to deploy changes to any env...

20. [CI/CD & Deployment Glossary](https://glossary.deployment.to/) - A plain-English glossary of CI/CD and deployment terms - clear, citable definitions of the concepts ...

21. [Rolling Updates and Rollbacks in Kubernetes ☸☸](https://medium.com/google-cloud/rolling-updates-and-rollbacks-in-kubernetes-7bec09da884f) - Everything about Rolling Updates and Rollbacks in Kubernetes :))

22. [Release app updates with staged rollouts - Play Console ...](https://support.google.com/googleplay/android-developer/answer/6346149?hl=en) - Set up and maintain a staged rollout · Open Play Console and go to the Production page if you want t...

23. [Rollback vs Hotfix | Bugnet Blog](https://bugnet.io/blog/rollback-vs-hotfix) - A rollback reverts to the previous known-good version, undoing a bad update; a hotfix ships a new fi...

24. [Perform a Rollback on a DaemonSet](https://kubernetes.io/docs/tasks/manage-daemon/rollback-daemon-set/) - This page shows how to perform a rollback on a DaemonSet. Before you beginYou need to have a Kuberne...

25. [Decide on Hot-fix or Rollback Strategy](https://docs.cloudposse.com/v1/layers/software-delivery/design-decisions/decide-on-hot-fix-or-rollback-strategy/) - Decide how to revert changes

26. [Semantic Versioning Explained: Rules, Benefits & Best Practices](https://talent500.com/blog/semantic-versioning-explained-guide/) - Learn what semantic versioning (SemVer) is, how it works, and why it’s essential for DevOps, CI/CD, ...

27. [Secure OTA firmware updates with rollback and lifecycle ...](https://www.siliconlogix.it/en/article/secure-ota-firmware-updates-with-rollback-for-embedded-devices) - An OTA firmware update is a remote update process that delivers, verifies and installs new firmware ...

28. [Alpha vs Beta Apps and Nightly vs Production Builds](https://gist.github.com/vadirajks/8041e83d46822800b70cc76c795e779e) - Release Candidate (RC) is the build released internally to check if any critical problems have gone ...

29. [What is a release candidate? Meaning + example](https://glossary.deployment.to/release-candidate/) - A release candidate (RC) is a build considered ready for production, pending a final round of verifi...

30. [Alpha vs Beta Apps and Nightly vs Production Builds](https://www.luciq.ai/blog/alpha-vs-beta-apps) - What is the difference between alpha vs. beta apps? And nightly vs. production builds? This post exp...

31. [软件版本管理RC、Nightly Build 、stable/alpha、beta、gamma](https://blog.csdn.net/qq_15821487/article/details/131652785) - 文章浏览阅读1.7k次。RC=Release Candidate,含义是"发布候选版"，它不是最终的版本，而是最终版(RTM=Release To Manufacture)之前的最后一个版本。广义上对...

32. [Google Play Tracks: Statuses, Rollout and Visibility](https://aso.dev/google-play/tracks/) - Overview of Google Play tracks (Internal, Closed, Open, Production), release statuses, staged rollou...

33. [How to minimize the negative impact of a faulty release ...](https://bitrise.io/blog/post/how-to-minimize-the-negative-impact-of-a-faulty-release-with-a-staged-rollout) - Select the production track and create a new release to initiate a staged rollout on Google Play. Af...

34. [How OTA Firmware Updates Work: Essential Guide for Firmware ...](https://www.embedcrest.com/blog/ota-firmware-updates) - Learn how over-the-air firmware updates work in embedded systems. A firmware development company gui...

35. [engineering/ How we separate deploy and release - Resend](https://resend.com/handbook/engineering/how-we-separate-deployments-from-releases) - Our approach to synchronizing changes and mitigating risk.

36. [Feature Flags as a Deployment Strategy: Deploy Dark, Release When Ready | ★ Bonus Episode](https://www.youtube.com/watch?v=GHWDil1_mUA) - The original series covers strategies where the *deployment mechanism itself* controls who sees what...

37. [Feature Flags: Deploy when you want, Release when you’re ready](https://medium.com/@mezhidani/feature-flags-deploy-when-you-want-release-when-youre-ready-f83569693b04) - Mastering Feature Flags

38. [Decouple Deployments from Releases with Feature Flags | Dreamforce 2023](https://www.youtube.com/watch?v=-jmWi111ED0) - Feature flags are an effective way to enable Continuous Delivery in Salesforce by allowing you to de...

39. [Distinction between continuous delivery and continuous deployment to production](https://softwareengineering.stackexchange.com/questions/361137/distinction-between-continuous-delivery-and-continuous-deployment-to-production) - According to Jez Humble, Continuous Delivery involves "ensuring our code is always in a deployable s...

40. [Release Procedure](https://nightly.scala-lang.org/docs/contributing/procedures/release.html)

41. [Continuous Delivery](https://martinfowler.com/books/continuousDelivery.html) - Continuous Delivery. Reliable Software Releases through Build, Test, and Deployment Automation ... C...

42. [Software versioning - Wikipedia](https://en.wikipedia.org/wiki/Software_versioning)

