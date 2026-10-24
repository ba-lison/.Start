## Resumo executivo

Gerenciamento de artefatos é a disciplina que governa tudo que uma pipeline de build produz — binários, pacotes, imagens de container, instaladores, símbolos de depuração e seus metadados — desde a criação até a promoção, distribuição, retenção e descarte seguro. O princípio central da entrega moderna é "build once, promote the same artifact": compilar uma única vez, armazenar o resultado com um identificador imutável e mover exatamente esses bytes por todos os ambientes, injetando apenas configuração em tempo de deploy. Frameworks como SLSA e in-toto formalizam como provar a origem (provenance) de um artefato, enquanto SBOMs em CycloneDX ou SPDX documentam sua composição. Assinatura digital via Sigstore/Cosign, checksums SHA-256 e políticas de retenção completam o ciclo de confiança, integridade e conformidade. Registries como OCI-compliant container registries e repositórios de pacotes (npm, PyPI, Artifactory) implementam mecanismos de revogação — yank, deprecate, unpublish — para tratar artefatos comprometidos sem quebrar consumidores existentes.[^1][^2][^3][^4][^5][^6][^7][^8][^9][^10][^11][^12][^13][^14]

## Definição de artefato

Um artefato é um blob imutável de dados produzido por um processo de build, endereçável por hash criptográfico — pode ser um binário, uma imagem de container, um diretório serializado, um arquivo firmware ou até um commit Git. A propriedade definidora não é o formato, mas a imutabilidade: uma vez publicado, o conteúdo nunca é reescrito; qualquer alteração gera uma nova versão com novo identificador. Isso distingue artefato de "release", que é o ato de promover um artefato específico, identificado por dígito ou hash, para um estado de disponibilidade (ex.: produção).[^5][^15][^16]

Categorias tratadas neste levantamento incluem:

- Artefatos intermediários: objetos, arquivos de cache de build, camadas de container geradas durante etapas de compilação, mas não destinados a distribuição direta.
- Artefatos finais: o produto de build empacotado e testado, pronto para promoção — executáveis, bibliotecas (jar, dll, so), pacotes (npm, Maven, PyPI, NuGet), imagens de container OCI, instaladores (msi, dmg, deb/rpm), arquivos ZIP/tar de distribuição.
- Artefatos de suporte: símbolos de depuração (pdb, dSYM), manifests, arquivos de configuração, checksums, SBOMs, assinaturas e atestações de provenance.[^17][^8][^18]

## Fluxo commit → build → artifact ID → teste → aprovação → promoção → release

O fluxo recomendado por múltiplas fontes de engenharia de release segue uma sequência linear e auditável, na qual apenas metadados mudam entre estágios — nunca o conteúdo binário:[^15]

1. **Commit**: código-fonte revisado e mesclado gera um SHA de commit imutável, ponto de origem rastreável no controle de versão.[^16]
2. **Build**: pipeline compila/empacota exatamente uma vez a partir desse commit, produzindo o artefato e sua provenance (builder, comandos, dependências).[^19][^20]
3. **Artifact ID**: o artefato recebe um identificador imutável — digest de conteúdo (sha256), tag semver ou coordenada de repositório — e é publicado no repositório apropriado.[^5][^15]
4. **Teste**: o artefato publicado, e não o código-fonte, é implantado no ambiente de teste; validações rodam sobre os bytes exatos que seguirão adiante.[^21][^1]
5. **Aprovação**: gate manual ou automatizado (scan de vulnerabilidades, aprovação humana, política) decide se o artefato avança.[^22]
6. **Promoção**: o mesmo digest é re-tagueado ou copiado (nunca reconstruído) para o repositório/ambiente seguinte — homologação, depois produção.[^23][^15]
7. **Release**: o artefato promovido é marcado como liberado, associado a uma tag de release e a notas de versão, tornando-se o alvo de rollback conhecido.[^5]

Toda a cadeia deve ser rastreável de volta ao commit, ao pull request, ao pipeline run e aos responsáveis, via provenance assinada (SLSA) ou atestações in-toto anexadas ao artefato.[^18][^24][^19]

## Por que "build once, promote the same artifact"

Recompilar o código para cada ambiente introduz risco porque duas execuções de build a partir do mesmo commit não garantem bytes idênticos: diferenças em cache de dependências, timestamps, versões transitórias de bibliotecas, variáveis de ambiente do agente de CI ou race conditions de rede podem gerar binários sutilmente diferentes daquilo que foi testado — o chamado "environmental drift" ou "contaminação cruzada de build". Isso quebra a premissa fundamental de qualquer teste: se o artefato que passou na homologação não é bit-a-bit igual ao que vai para produção, a validação realizada não garante nada sobre o que efetivamente será executado.[^25][^1][^21]

O princípio "build once, deploy everywhere" resolve isso ao selar o artefato no momento do build e tratar toda diferença entre ambientes como configuração injetada em runtime (variáveis, secrets, ConfigMaps), nunca como recompilação. Isso também reduz custo computacional (uma única compilação) e cria uma cadeia de custódia auditável: o mesmo digest que passou por testes é o que roda em produção, verificável por comparação de hash.[^3][^1][^25][^15]

| Aspecto | Rebuild por ambiente | Build once + promoção |
|---|---|---|
| Garantia de equivalência | Nenhuma — bytes podem diferir[^25] | Byte a byte idêntico, verificável por digest[^1] |
| Rastreabilidade de testes | Testes validam um binário diferente do de produção | Testes validam exatamente o artefato promovido[^21] |
| Custo de CI | Build repetido em cada estágio | Build único, reuso via cópia/tag[^23] |
| Rollback | Requer rebuild da versão anterior | Artefato anterior já armazenado, pronto para redeploy[^5] |
| Superfície de ataque | Maior — cada rebuild é oportunidade de injeção | Menor — builder único, hermético e auditável[^4] |

## Distinção entre artefato aprovado, rejeitado, expirado e liberado

Repositórios de artefatos maduros modelam o ciclo de vida como uma máquina de estados sobre o mesmo objeto imutável, nunca alterando o binário:

- **Aprovado**: passou pelos gates definidos (testes, scans, assinatura) e recebeu uma promoção de status/tag para o próximo estágio.[^20][^15]
- **Rejeitado**: falhou em um gate; permanece no repositório para auditoria/forense, mas nunca é promovido ou implantado.
- **Expirado**: atingiu o limite de uma política de retenção (tempo, contagem de versões) e é marcado para limpeza automática — em plataformas como JFrog Artifactory isso é implementado via "Retention Policies" com duração em dias, número máximo de versões e execução em dry-run antes da exclusão real.[^10][^26]
- **Liberado (released)**: promovido ao estágio final e associado publicamente a uma tag de release, notas e SBOM/atestação — o estado que consumidores externos devem consumir.[^5]

Estados como *yanked* (PyPI) ou *deprecated* (npm) formam uma quinta categoria — "revogado, mas não deletado" — usada para artefatos comprometidos, tratada na seção de revogação abaixo.[^11][^12]

## Comparação entre tipos de armazenamento

| Tipo de repositório | Uso típico | Padrão/Formato | Observações |
|---|---|---|---|
| Package registry (npm, PyPI, Maven, NuGet) | Bibliotecas e dependências versionadas | Coordenadas nome@versão, hash do pacote | Suporta yank/deprecate; instalação por versão pinada ainda funciona mesmo após yank[^11][^13] |
| Container registry (OCI) | Imagens de container, Helm charts, artefatos OCI genéricos | OCI Distribution Spec — blobs endereçados por digest, manifests, tags | API padronizada de push/pull/discovery/content management[^27][^28][^29] |
| Repositório de artefatos genérico (Artifactory, Nexus) | Binários, ZIPs, símbolos, múltiplos formatos num só sistema | Estrutura de repositório + políticas de retenção configuráveis | Suporta múltiplos tipos de pacote e cleanup automatizado[^10][^26] |
| Release assets (GitHub Releases) | Distribuição de binários/instaladores finais ao público | Upload manual/CI com digest SHA-256 automático desde 2025 | Digest imutável exposto via UI, REST e GraphQL[^30][^31] |
| Object storage genérico (S3, GCS, blob storage) | Armazenamento de longo prazo, símbolos, SBOMs, logs de build | Chave/bucket, sem semântica de pacote nativa | Requer camada própria de versionamento e política de ciclo de vida |
| Symbol server | Símbolos de depuração (pdb, dSYM) para crash analysis | Estrutura hash-indexada, cache downstream | Retenção geralmente ligada ao ciclo de vida do build associado[^17][^32] |

## Modelo de nomenclatura

Uma convenção de nomenclatura robusta combina identidade humana e identidade imutável, evitando referências mutáveis como `latest` ou `-SNAPSHOT` em produção:[^1][^15]

- **Nome do componente**: identifica logicamente o produto (`totalcad-license-service`).
- **Versão semântica**: `MAJOR.MINOR.PATCH` para releases estáveis (`2.4.1`), com sufixos padronizados para pré-lançamentos: `2.5.0-alpha.1`, `2.5.0-beta.2`, `2.5.0-rc.1`, seguindo o padrão adotado por ecossistemas Python/RPM (a=alpha, b=beta, c=rc).[^33]
- **Identificador de build/commit**: SHA curto do commit ou número de build monotônico, garantindo unicidade mesmo dentro da mesma versão semântica (`2.5.0-rc.1+a1b2c3d`).[^3][^23]
- **Digest de conteúdo**: para containers e blobs, a identidade canônica é o digest sha256, não a tag — tags são apenas apontadores mutáveis para digests imutáveis.[^28][^15]
- **Canal/estágio**: quando aplicável, um sufixo de canal (`nightly`, `beta`, `rc`, `stable`) comunica o nível de maturidade sem exigir recompilação.

Boas práticas: nunca reutilizar um número de versão após publicação; tratar builds nightly com timestamp explícito (`2.5.0-nightly.20260806`); sempre referenciar deploys por digest, nunca por tag mutável.[^21][^15]

## Modelo de metadados

Cada artefato deve trafegar acompanhado de um conjunto de metadados que permita rastreabilidade completa e decisões automatizadas de política. Com base em SLSA, in-toto e requisitos de SBOM (CISA 2026), o conjunto mínimo recomendado é:

- **Identidade**: nome, versão, digest sha256, formato/tipo de artefato.[^9][^16]
- **Origem no controle de versão**: commit SHA, branch, tag, pull request associado.[^1]
- **Pipeline**: identificador da execução de build (build ID), builder/plataforma usada, timestamp de início/fim.[^19]
- **Dependências**: SBOM completo (CycloneDX ou SPDX) listando componentes diretos e transitivos com versão, fornecedor, licença e hash.[^8][^9]
- **Integridade**: checksum (sha256 no mínimo), assinatura digital e, quando aplicável, registro em log de transparência (Rekor).[^6][^7]
- **Provenance**: atestação SLSA/in-toto descrevendo como, onde e por quem o artefato foi construído, permitindo verificação antes do deploy.[^18][^19]
- **Responsável**: autor do commit, aprovador do merge/release, e — quando aplicável — assinatura de identidade (keyless signing via OIDC).[^7][^22]
- **Ciclo de vida**: estágio atual (nightly/beta/rc/stable), status de promoção, data de expiração conforme política de retenção.[^10]

## Ligação artefato ↔ commit, tag, PR, pipeline, dependências e responsável

A ligação é estabelecida por meio de atestações assinadas cujo "subject" é o digest do artefato e cujo "predicate" carrega os dados de contexto. O modelo in-toto formaliza isso: um "layout" define as etapas esperadas da supply chain (clone, build, teste, package), cada etapa gera um "link" assinado pela chave da função (functionary) que a executou, e o verificador reconstrói a cadeia inteira comparando os hashes de materiais e produtos entre etapas. Na prática:[^34][^35][^24][^18]

- O commit e o PR ficam registrados no metadado `buildDefinition`/`materials` da provenance SLSA.[^19]
- O pipeline (run ID, builder) é registrado em `runDetails`.[^19]
- Dependências são capturadas no SBOM anexado como atestação separada, referenciando o mesmo digest do artefato.[^8][^18]
- O responsável é capturado tanto pela identidade OIDC usada na assinatura keyless (ex.: workflow do GitHub Actions que assinou) quanto pelo autor/aprovador do commit e do release.[^36][^7]

Essas atestações costumam ser armazenadas como artefatos OCI adicionais, referenciados junto à imagem/pacote no mesmo registry, permitindo que o verificador busque tudo num único local antes do deploy.[^18]

## Como verificar integridade

A verificação de integridade combina três camadas independentes, cada uma cobrindo uma ameaça distinta:

1. **Checksum**: hash SHA-256 do arquivo, comparado byte a byte contra o valor publicado (ex.: arquivo `SHA256SUMS` ou digest anexado a um GitHub Release desde junho de 2025). Protege contra corrupção acidental ou adulteração em trânsito, mas não prova quem gerou o hash.[^30][^37]
2. **Assinatura digital**: cosign/Sigstore assina o digest do artefato com uma chave (ou certificado efêmero via OIDC no modo "keyless"), anexando a assinatura e o certificado ao próprio registry OCI. A verificação confirma que uma identidade específica (pessoa ou workflow de CI) assinou exatamente aquele digest.[^6][^7]
3. **Log de transparência**: o evento de assinatura é registrado com timestamp no Rekor, um log append-only e auditável, permitindo detectar assinaturas retroativas ou forjadas mesmo que a chave privada seja destruída após uso.[^38][^7]

O fluxo recomendado é sempre assinar e verificar por digest (`@sha256:...`), nunca por tag, pois tags são mutáveis e podem ser repontadas para outro conteúdo sem aviso. GPG sobre arquivos de checksum é uma alternativa amplamente usada em distribuições open source para dar uma segunda camada de confiança de origem.[^39][^36]

## Como armazenar símbolos para investigação de falhas

Símbolos de depuração (PDB no Windows, dSYM no macOS/iOS) não devem viajar dentro do artefato de produção — aumentam superfície de exposição de código e tamanho de binário — mas precisam estar disponíveis para depuração pós-incidente. A prática estabelecida é um **symbol server** separado, indexado por hash/GUID do binário, ao qual o depurador consulta sob demanda e mantém um cache local ("downstream store") para evitar buscas repetidas. Plataformas de CI como Azure Pipelines publicam símbolos com o mesmo período de retenção do build que os gerou, garantindo que símbolos e binários envelheçam juntos e nunca fiquem disponíveis "órfãos" além do artefato que descrevem. Boas práticas incluem: gerar símbolos automaticamente a cada build, associar cada arquivo de símbolo ao mesmo identificador de build usado no binário, e restringir acesso ao symbol server como um recurso interno sensível (pode revelar detalhes de implementação).[^32][^40][^17]

## Política de imutabilidade

A imutabilidade é o alicerce que torna a promoção segura: um artefato publicado sob um identificador de versão/digest nunca deve ser sobrescrito — qualquer mudança de conteúdo exige um novo identificador. Práticas recomendadas:[^15][^5]

- Tratar toda tag mutável (`latest`, `-SNAPSHOT`) como não confiável para produção; usar apenas referências por digest ou versão fixa em deploys.[^15][^1]
- Configurar repositórios (Artifactory, registries OCI) para rejeitar `push --force` ou sobrescrita de uma versão já publicada.
- Verificar periodicamente que o que roda em produção corresponde exatamente ao artefato original comparando digests/assinaturas — captura modificações manuais que tenham contornado a pipeline.[^1]
- Tratar builds `nightly` como mutáveis por natureza apenas dentro do canal `nightly` (cada execução gera novo timestamp/digest), nunca reescrevendo um artefato nightly já publicado.

## Política de retenção

Retenção equilibra custo de armazenamento contra a necessidade de rollback e auditoria. Um modelo em camadas, observado em ferramentas como JFrog Artifactory, define regras por:

- **Duração (dias)**: artefatos mais antigos que N dias tornam-se candidatos a limpeza.[^26]
- **Número máximo de versões/tags retidas**: preserva sempre as últimas K versões, independentemente da idade, garantindo capacidade de rollback.[^26]
- **Escopo e tipo de pacote**: políticas diferenciadas por repositório e formato (Docker, Maven, npm etc.).[^26]
- **Padrões de inclusão/exclusão**: protege exceções (ex.: releases marcados como "GA" nunca são limpos automaticamente).
- **Execução em dry-run**: toda política deve ser simulada antes de execução real, evitando exclusões irreversíveis acidentais.[^26]

Diretrizes gerais por tipo de canal:
- **Nightly/CI**: retenção curta (dias), sem garantia de disponibilidade de longo prazo.
- **Beta/RC**: retenção intermediária, suficiente para o ciclo de testes de aceitação.
- **Release estável**: retenção longa ou permanente, associada a suporte contratual/regulatório e a requisitos de auditoria/compliance.
- **Símbolos**: acompanham a retenção do build associado, não uma política própria independente.[^32]

## Política de acesso

Controle de acesso deve seguir o princípio de menor privilégio em cada etapa da cadeia:

- **Publicação**: apenas o pipeline de CI autenticado (via identidade de serviço/OIDC) deve poder publicar em repositórios de release; publicação manual deve ser excepcional e auditada.[^22]
- **Promoção**: mudança de estágio (dev→staging→produção) deve ser uma operação de metadados protegida por aprovação/gate, distinta do direito de publicar artefatos novos.[^15]
- **Assinatura**: chaves de assinatura (ou identidade OIDC para keyless signing) devem ser restritas a workflows de release protegidos, nunca expostas a ambientes de desenvolvimento local.[^7][^22]
- **Leitura/consumo**: repositórios de produção devem aplicar autenticação e, idealmente, enforcement de admissão (ex.: Kyverno) que rejeita imagens sem assinatura válida antes do deploy.[^41]
- **Exclusão/retenção**: operações destrutivas (delete, purge) devem exigir permissão elevada distinta da permissão de push/pull cotidiana.

## Procedimento de auditoria

Um procedimento de auditoria eficaz trata cada estágio como uma verificação independente, na ordem em que uma falha detectada interrompe o fluxo antes de avançar:

1. Definir a expectativa antes de olhar o repositório: qual artefato, em qual papel, em qual formato deveria existir.[^42]
2. Consultar a lista de artefatos/assets via API do repositório e comparar nome, estado, tamanho e digest contra a expectativa.[^42]
3. Rebaixar (re-download) o artefato pelo mesmo caminho que um consumidor real usaria, nunca inspecionando a pasta de upload original.[^42]
4. Recalcular o hash SHA-256 do artefato obtido e compará-lo byte a byte com o digest publicado/assinado.[^37][^42]
5. Verificar a assinatura e, quando disponível, a entrada correspondente no log de transparência (Rekor).[^38][^7]
6. Registrar o resultado de cada verificação num "ledger" de auditoria — não apenas um flag binário de "aprovado", mas o estado de cada checagem individual (planejado / upload confirmado / re-baixado / hash verificado / assinatura verificada).[^42]
7. Em caso de qualquer falha (red), isolar se o problema está no artefato ou no verificador antes de repetir o teste, evitando mascarar um comprometimento real.

## Antipadrões

- **Recompilar por ambiente**: cada rebuild pode gerar bytes diferentes daqueles testados, quebrando a garantia de que "o que foi testado é o que roda em produção".[^25][^21]
- **Deploy por tag mutável** (`latest`, `-SNAPSHOT`): a referência pode apontar para conteúdo diferente entre a validação e o deploy real.[^1][^15]
- **Ausência de checksum/assinatura em artefatos de distribuição pública**: impossibilita ao consumidor confirmar integridade e origem.[^30][^6]
- **Retenção "infinita" sem política**: acúmulo descontrolado de custo de armazenamento e dificuldade de localizar a versão correta em auditorias.
- **Excluir em vez de revogar** (deletar releases/pacotes já consumidos por terceiros): quebra builds de downstream; a prática correta é yank/deprecate, preservando o artefato para quem já depende dele mas evitando adoção nova.[^13][^11]
- **Misturar configuração de ambiente dentro do artefato**: torna impossível reutilizar o mesmo binário entre ambientes, forçando rebuilds.[^15][^1]
- **Símbolos de depuração publicados junto ao binário de produção**: expõe detalhes internos e aumenta a superfície de ataque sem necessidade.[^17]
- **Assinar por tag em vez de por digest**: sujeito a repontamento posterior da tag para outro conteúdo, invalidando a garantia da assinatura.[^36]

## Como tratar nightly, beta, release candidate e release estável

Cada canal de maturidade deve ser tratado como um fluxo de promoção paralelo, com políticas de retenção e visibilidade distintas, mas usando a mesma mecânica de build único + promoção:

- **Nightly**: gerado automaticamente a cada execução da branch principal, com timestamp no identificador (`-nightly.20260806`); retenção curta; destinado a testes internos contínuos, nunca a distribuição externa.
- **Beta**: build estabilizado o suficiente para testes externos limitados; versão com sufixo `-beta.N`; feedback de usuários early-adopters coletado antes de avançar.
- **Release candidate (RC)**: candidato final a release, com sufixo `-rc.N`; a expectativa é que, salvo bugs críticos, o mesmo artefato RC seja promovido a release estável sem qualquer recompilação — apenas removendo o sufixo e reetiquetando o mesmo digest.[^33][^15]
- **Release estável**: versão semântica limpa (`MAJOR.MINOR.PATCH`), retenção longa, assinada e acompanhada de SBOM e notas de release completas; é o artefato que efetivamente foi testado como RC, promovido sem alteração de bytes.[^5][^15]

## Como revogar um artefato comprometido

A revogação de um artefato já distribuído segue o princípio de "não deletar, marcar como não confiável", preservando a disponibilidade para quem já depende da versão, mas impedindo novo consumo:

- **PyPI (yank)**: uma release inteira pode ser marcada como "yanked" — instaladores ignoram automaticamente essa versão em resoluções normais (`pip install pacote`), mas ainda a instalam se explicitamente fixada (`pacote==versão`), acompanhada de uma razão opcional exibida aos usuários. A regra prática é "yankar quase sempre, deletar quase nunca".[^11][^13]
- **npm (deprecate/unpublish)**: `npm deprecate` emite um aviso visível a cada download sem quebrar builds existentes; `npm unpublish` só é permitido dentro de critérios estritos (poucas dependências, poucos downloads, janela de 72 horas) para evitar quebras em massa no ecossistema.[^12]
- **Container registries**: revogação passa por invalidar a assinatura/atestação associada ao digest comprometido (ou emitir uma atestação de revogação) e bloquear novas implantações via política de admissão que rejeita aquele digest específico.[^41][^18]
- **Registries genéricos (Artifactory/Nexus)**: marcação de status "compromised"/bloqueio de download, aliado a alerta automático para consumidores conhecidos via metadados de auditoria de quem consumiu a versão.
- Em todos os casos, a resposta a incidente deve incluir: identificar todos os consumidores conhecidos daquele digest (via logs/atestações), emitir e assinar uma nova versão corrigida, e comunicar a revogação através dos mesmos canais de metadados que anunciaram a versão original (changelog, SBOM atualizado, aviso no registry).[^13][^22]

## Checklist para avaliar um repositório de artefatos existente

- Publica identificadores imutáveis (digest/hash) além de tags mutáveis para cada artefato?
- Impõe suporte nativo ao formato relevante (OCI para containers, protocolo específico para pacotes de linguagem)?[^29][^28]
- Gera e expõe checksums automaticamente no momento do upload/publicação?[^30]
- Suporta ou se integra a assinatura digital (Cosign/Sigstore) e verificação antes do deploy?[^6][^7]
- Permite anexar/armazenar SBOM e atestações de provenance junto ao artefato?[^18]
- Oferece política de retenção configurável, com execução em dry-run antes de exclusão real?[^26]
- Distingue estados de ciclo de vida (aprovado, rejeitado, expirado, liberado) sem recorrer à exclusão física?[^15]
- Permite yank/deprecate em vez de apenas delete para revogação de versões comprometidas?[^12][^11]
- Registra controle de acesso granular por operação (push, promote, sign, delete)?
- Mantém trilha de auditoria consultável (quem publicou, quem promoveu, quando, com qual pipeline)?
- Suporta replicação/federação entre regiões/ambientes sem duplicar builds?
- Armazena símbolos de depuração vinculados ao build de origem, com retenção equivalente?[^32]

## Tabela de fontes com avaliação de qualidade e limitações

| Fonte | Tipo | Qualidade/Autoridade | Limitações |
|---|---|---|---|
| SLSA specification (slsa.dev)[^2][^19][^16] | Padrão técnico open, mantido por OpenSSF/comunidade | Alta — especificação formal de níveis e provenance | Foco majoritariamente em builds de containers/binários; adoção prática ainda desigual entre organizações |
| OCI Distribution Spec (github.com/opencontainers)[^27][^28] | Especificação oficial da Open Container Initiative | Alta — padrão de indústria para registries | Cobre apenas a camada de distribuição, não políticas de retenção/acesso, que ficam a cargo de cada implementação |
| Sigstore/Cosign docs (docs.sigstore.dev, GitHub sigstore/cosign)[^6][^7][^36] | Documentação oficial de projeto CNCF | Alta — ferramenta amplamente adotada para assinatura | Documentação técnica de uso; não substitui análise independente de segurança criptográfica |
| PyPI Yanking / npm Unpublish Policy (docs.pypi.org, docs.npmjs.com)[^11][^12] | Documentação oficial de registries de pacotes | Alta — política oficial e vinculante do registry | Específica a esses ecossistemas; não generaliza diretamente a outros gerenciadores de pacote |
| GitHub Changelog sobre digests de release assets[^30][^31] | Anúncio oficial de produto | Alta para o fato específico, mas é conteúdo de fornecedor | Reflete apenas a implementação do GitHub, não um padrão universal de releases |
| JFrog Artifactory docs (docs.jfrog.com)[^10][^26] | Documentação oficial de produto comercial | Média-alta — describe funcionalidade real, mas é material de fornecedor | Não deve ser única evidência de "best practice" geral; usado aqui apenas para ilustrar mecanismo concreto de retenção |
| in-toto / CNCF (sbomify.com, slsa.dev/blog, emergentmind.com)[^14][^18][^35][^24] | Documentação de projeto CNCF graduado + análises técnicas de terceiros | Alta para o framework oficial; média para blogs de terceiros | Alguns artigos secundários simplificam conceitos; a especificação formal do in-toto é a referência primária |
| CycloneDX/SPDX comparativos (cyclonedx.org, interlynk.io, arxiv.org)[^8][^43][^44][^45] | Mistura de documentação oficial de projeto e análise acadêmica/comercial | Média-alta — cruzamento de múltiplas fontes independentes confirma consistência | Comparativos de blogs comerciais podem ter viés de produto; mitigado por citar também paper acadêmico (arxiv) |
| Microsoft Symbol Server / Azure Pipelines docs (learn.microsoft.com)[^17][^32][^40] | Documentação oficial de fornecedor | Alta para mecanismo específico da Microsoft | Prática específica ao ecossistema Windows/.NET; conceito geral (symbol server) transferível, detalhes de implementação não |
| Artigos de blog sobre "build once, promote" (minimumcd.org, glossary.deployment.to, oneuptime.com)[^1][^5][^15] | Conteúdo educacional independente/comunidade DevOps | Média — consistente entre múltiplas fontes independentes, reforçando confiabilidade do princípio | Não são padrões formais; valor está na convergência de múltiplas fontes não-vinculadas descrevendo a mesma prática |

---

## References

1. [Immutable Artifacts](https://beyond.minimumcd.org/docs/migrate-to-cd/pipeline/immutable-artifacts/) - Instead of rebuilding for each environment, promote the same artifact through environments. The pipe...

2. [SLSA specification](https://slsa.dev/spec/v1.1/) - SLSA is a specification for describing and incrementally improving supply chain security, establishe...

3. [Build Once, Deploy Many | Ahmad Nurhidayat](https://ahmadnurhidayat.github.io/posts/cicd-build-once-deploy-many/) - The fundamental CI/CD principle of immutable artifacts — build your application once, then promote t...

4. [Introducing SLSA, an End-to-End Framework for Supply Chain Integrity](https://security.googleblog.com/2021/06/introducing-slsa-end-to-end-framework.html) - Posted Kim Lewandowski, Google Open Source Security Team & Mark Lodato, Binary Authorization for Bor...

5. [Artifact - CI/CD & Deployment Glossary](https://glossary.deployment.to/artifact/) - Why should build artifacts be immutable? Immutability is what lets you promote the same package from...

6. [Signing Software The Easy Way with Sigstore and Cosign](https://martinheinz.dev/blog/56) - Signing software artifacts has many obvious benefits such as code integrity or developer (author) au...

7. [Overview - Sigstore](https://docs.sigstore.dev/cosign/signing/overview/) - This document explains how identity-based, or “keyless” signing works in Sigstore. To learn more abo...

8. [What is an SBOM? — Software Bill of Materials Explained (CycloneDX, SPDX)](https://zerohunt.ai/learn/sbom-software-bill-of-materials/) - Definition, the two dominant formats (CycloneDX and SPDX), what NTIA minimum elements require, and h...

9. [Mapping CISA's 2026 SBOM Minimum Elements to ...](https://runsafesecurity.com/blog/sbom-minimum-elements-cyclonedx-spdx/) - A field-by-field guide mapping CISA's 2026 SBOM minimum elements to CycloneDX and SPDX, with exact p...

10. [Set the Retention Policy Configuration](https://docs.jfrog.com/artifactory/reference/setretentionpolicyconfiguration) - Sets the retention policies that were configured. If a Release Bundle meets one or more of the reten...

11. [Yanking](https://docs.pypi.org/project-management/yanking/)

12. [npm Unpublish Policy](https://docs.npmjs.com/policies/unpublish/) - If your package does not meet the unpublish policy criteria, we recommend deprecating the package. T...

13. [PyPI Package Yanking Policies Analyzed - Safeguard](https://safeguard.sh/resources/blog/pypi-package-yanking-policies-analysis) - A close analysis of PyPI's package yanking mechanism under PEP 592, when to yank versus delete, and ...

14. [in-toto and SLSA](https://slsa.dev/blog/2023/05/in-toto-and-slsa) - Taken together, a bundle of in-toto attestations and a layout can be used to comprehensively verify ...

15. [Build Once, Promote Everywhere: Stop Rebuilding Artifacts per Environment](https://oneuptime.com/blog/post/2026-07-28-build-once-promote-everywhere/view) - Build and verify one immutable artifact, then move its recorded identity through staging and product...

16. [Terminology](https://slsa.dev/spec/v1.0/terminology) - Before diving into the SLSA specification levels, we need to establish a core set of terminology and...

17. [Microsoft Public Symbol Server for Windows Debuggers](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/microsoft-public-symbols) - Use this guide to quickly set up your symbol server connection and start debugging. You'll learn how...

18. [The in-toto attestations and your supply chain security](https://www.crisci-solutions.co.uk/blog/software-attestations-and-the-supply-chain-security/) - Why in-toto attestations are critical to your supply chain security

19. [Build: Provenance](https://slsa.dev/spec/draft/build-provenance) - Description of SLSA build provenance specification for verifying where, when, and how something was ...

20. [Supply-chain Levels for Software Artifacts (SLSA)](https://docs.gitlab.com/ci/pipeline_security/slsa/) - Level 1: Provenance showing how the package was built · What entity built the package. · What build ...

21. [Artifact Promotion - Stop Rebuilding in CI/CD](https://www.youtube.com/watch?v=6QcCVj5KzS4) - Description:

🚀 Welcome back to QA Ops! In this episode, we tackle a critical architectural vulnerab...

22. [Artifact signing with Cosign and Sigstore - DevSecOpsAtlas](https://devsecopsatlas.com/guides/artifact-signing-cosign-sigstore) - A practical guide to artifact signing with Cosign and Sigstore, including container signatures, keyl...

23. [build once, prove provenance, version the mesh - nmadiraju](https://www.nmadiraju.com/posts/release-engineering-from-scratch/)

24. [What Is in-toto? Securing the Software Supply Chain End ...](https://sbomify.com/2024/08/14/what-is-in-toto/) - The in-toto attestation framework defines a standard envelope format for supply chain metadata. It w...

25. [13.2.1.5 Environment Rebuild Avoidance](https://www.codartium.com/environment-rebuild-avoidance-in-docker/) - Learn Environment Rebuild Avoidance with Docker-focused explanations, practical examples, and guidan...

26. [Configure Cleanup Policies for Federated Repositories | JFrog](https://docs.jfrog.com/artifactory/docs/how-to-configure-cleanup-policies-for-federated-repositories) - Learn how deletion events propagate when running retention policies on Federated repositories.

27. [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec) - The OCI Distribution Spec project defines an API protocol to facilitate and standardize the distribu...

28. [distribution-spec/spec.md at main · opencontainers/distribution-spec](https://github.com/opencontainers/distribution-spec/blob/main/spec.md) - OCI Distribution Specification. Contribute to opencontainers/distribution-spec development by creati...

29. [Supported formats | Artifact Registry - Google Cloudcloud.google.com › artifact-registry › docs › supported-formats](https://docs.cloud.google.com/artifact-registry/docs/supported-formats)

30. [Releases now expose digests for release assets](https://github.blog/changelog/2025-06-03-releases-now-expose-digests-for-release-assets/) - GitHub now automatically computes and displays SHA256 checksums (digests) for all uploaded release a...

31. [Expose Release Asset Digests [GA] · Issue #1136 · github/roadmap](https://github.com/github/roadmap/issues/1136) - Value Prop GitHub will now automatically compute and display checksums (SHA256 digests) for all uplo...

32. [Publish symbols for debugging - Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/artifacts/symbols?view) - Q: What is the duration for which symbols are retained? A: A symbol file has the same retention peri...

33. [Versioning¶](https://docs.pulpproject.org/en/2.12/nightly/dev-guide/policies/versioning.html)

34. [All Things in-Toto: Supply Chain Attestations, Policies an... Santiago Torres-Arias & Marcela Melara](https://www.youtube.com/watch?v=wuB--26-WpM) - Don't miss out! Join us at our next Flagship Conference: KubeCon + CloudNativeCon Europe in Paris fr...

35. [In-toto Attestations in Secure Pipelines](https://www.emergentmind.com/topics/in-toto-attestations) - Explore in-toto attestations—cryptographically secured metadata that ensure artifact integrity, prov...

36. [GitHub - sigstore/cosign: Code signing and transparency for containers and binaries](https://github.com/sigstore/cosign) - Code signing and transparency for containers and binaries - sigstore/cosign

37. [SHA-256 File Checksum – Verify Downloads Are Untampered](https://devtoolbox.vedaforge.dev/blog/sha256-file-checksum) - Verify file integrity with SHA-256 checksums. Confirm downloaded binaries match published hashes bef...

38. [Sigstore Quickstart with Cosign](https://docs.sigstore.dev/quickstart/quickstart-cosign/) - Sign and Verify with Cosign

39. [GitHub 下载文件 SHA256 校验完整教程：Windows / macOS / Linux 三步...](https://www.chonglangbiji.com/howto/github-sha256-verify/) - 从 GitHub Release 下载的 .exe/.dmg/.apk 不一定和发布者上传的完全一致。SHA256 校验能把下载的文件算出一个 64 位指纹，和发布者公布的指纹比对，一致才说明文件完好...

40. [When looking to free up disk space, don't forget your ...](https://devblogs.microsoft.com/oldnewthing/20230918-35/?p=108789) - They go all over the place.

41. [Signing and Verifying Container Images With Sigstore Cosign and Kyverno](https://www.youtube.com/watch?v=HLb1Q086u6M) - If we want to be certain that what we're running is what we built, we might need to sign container (...

42. [Your GitHub Release URL Opens Fine. That's Not Proof ...](https://dev.to/nomurasan/your-github-release-url-opens-fine-thats-not-proof-you-can-get-the-files-back-6jk) - A release URL proves the box exists, not that your GitHub Release assets are in it. A 5-step round-t...

43. [CycloneDX vs SPDX: Choosing an SBOM Format for ...](https://www.interlynk.io/resources/cyclonedx-vs-spdx-sbom-format) - Across that line, CycloneDX grew from a software SBOM format into a general bill of materials standa...

44. [SPDX vs CycloneDX](https://developer.harness.io/docs/software-supply-chain-assurance/how-to-guides/spdx-vs-cyclonedx/) - Comparison between SPDX and CycloneDX

45. [A Comparative Analysis of SPDX and CycloneDX](https://arxiv.org/html/2512.21781v2)

