## Resumo executivo

Uma build é considerada confiável quando produz o mesmo artefato — ou um artefato funcionalmente equivalente — a partir do mesmo código-fonte, independentemente de quem, onde ou quando a executa. O Reproducible Builds Project define formalmente: "a build é reproduzível se, dado o mesmo código-fonte, ambiente de build e instruções de build, qualquer parte pode recriar cópias idênticas bit a bit de todos os artefatos especificados". Essa propriedade sustenta segurança de cadeia de suprimentos (detecção de backdoors e adulteração), auditoria (correspondência verificável entre fonte e binário) e depuração de incidentes (capacidade de recriar exatamente o estado que gerou um artefato em produção).[^1][^2][^3][^4]

Na prática, "determinismo", "reprodutibilidade", "hermeticidade" e "repetibilidade" são propriedades relacionadas mas distintas, com graus diferentes de rigor. Determinismo é a propriedade mais forte (bit a bit idêntico); reprodutibilidade pode aceitar equivalência funcional; hermeticidade descreve isolamento do ambiente de execução; repetibilidade descreve apenas a capacidade de executar os mesmos passos de forma consistente. Casos reais de builds diferentes a partir do mesmo commit derivam majoritariamente de cinco categorias de causas: timestamps, caminhos de sistema de arquivos, locale, aleatoriedade/memória não inicializada e diferenças de sistema/ambiente. Frameworks como SLSA formalizam níveis de garantia de proveniência e build, ligando build reproduzível a controles de segurança de cadeia de suprimentos, SBOM e assinatura digital.[^5][^6][^7][^8][^9]

## Glossário e diferenças entre os conceitos

| Conceito | Definição técnica | Nível de garantia | Fonte |
|---|---|---|---|
| Deterministic build | Build que, dado exatamente o mesmo input, sempre produz o mesmo output byte a byte, incluindo metadados, timestamps e ordenação de arquivos | Máximo — verificação por hash criptográfico (SHA-256) | [^6][^5] |
| Reproducible build | Build que, dado o mesmo código-fonte, ambiente e instruções, permite que qualquer parte recrie cópias bit a bit idênticas dos artefatos especificados; a definição formal do Reproducible Builds Project exige identidade bit a bit, mas algumas literaturas usam "reprodutível" para equivalência funcional | Alto (formal) a moderado (uso coloquial) | [^2][^6] |
| Hermetic build | Build cujo resultado é determinado inteiramente pelos inputs declarados explicitamente — nenhuma influência de estado do host, variáveis de ambiente não declaradas, relógio do sistema ou rede | Propriedade de isolamento; pré-requisito para determinismo, mas não é sinônimo dele | [^10][^11][^12] |
| Repeatable build | Build cujos passos são scriptados e executados de forma consistente na mesma ordem, mas sem garantir controle total de todos os inputs (ex.: ainda pode usar dependências não fixadas) | Nível básico | [^5] |
| Clean build | Build executada a partir de um estado limpo, sem artefatos ou caches de builds anteriores, forçando reexecução completa de todos os passos | Técnica de verificação, não propriedade final | [^13] |
| Incremental build | Build que reutiliza artefatos intermediários de builds anteriores para acelerar o processo, recompilando apenas o que mudou | Otimização de performance; risco de introduzir estado obsoleto | — |
| Isolated build environment | Ambiente (container, VM, sandbox) que restringe o acesso do processo de build a recursos do host, rede e sistema de arquivos além do declarado | Mecanismo de aplicação de hermeticidade | [^12][^14] |
| Dependency pinning | Fixação de versões exatas (não intervalos) de dependências diretas e transitivas | Prática de mitigação | [^15][^16] |
| Lockfiles | Arquivos gerados automaticamente (package-lock.json, yarn.lock, pnpm-lock.yaml, Cargo.lock) que registram a árvore de dependências exata resolvida, incluindo hashes de integridade | Mecanismo técnico de dependency pinning | [^15][^17][^18] |
| Toolchain pinning | Fixação de versões exatas de compiladores, linkers, SDKs e ferramentas de build | Prática de mitigação | [^11] |
| Environment pinning | Fixação de variáveis de ambiente, locale, timezone e outras condições do sistema operacional usadas durante a build | Prática de mitigação | [^2][^9] |
| Source Date Epoch | Variável de ambiente padronizada (timestamp UNIX) usada por ferramentas de build para substituir a data/hora "atual" embutida em artefatos | Padrão técnico formal (especificação) | [^19] |
| Build cache | Armazenamento de resultados intermediários ou finais de builds anteriores para reutilização | Otimização; pode introduzir resultados incorretos se a chave de cache não capturar todos os inputs relevantes | [^20] |
| Binary comparison | Comparação byte a byte (ou estruturada) de dois artefatos para verificar identidade | Método de verificação (ex.: diffoscope) | [^21] |
| Checksum | Valor hash (ex.: SHA-256) usado para verificar integridade e identidade de um artefato | Mecanismo de verificação | [^2][^13] |
| Provenance | Metadados que documentam como, onde, quando e por quem um artefato foi construído, incluindo fonte, configuração e plataforma de build | Base para atestação de cadeia de suprimentos (ex.: SLSA, in-toto) | [^8][^22] |

A relação hierárquica central é: hermeticidade é condição de isolamento que, quando plena, torna a build determinística; determinismo implica reprodutibilidade formal (bit a bit); mas reprodutibilidade em sentido amplo pode ser satisfeita sem determinismo total, aceitando "equivalência funcional". Um estudo da VMware Open Source Blog propõe três definições operacionais: builds repetíveis controlam apenas os passos, builds "rebuildable" controlam todos os inputs explícitos, e builds binariamente reproduzíveis (também chamadas deterministic/hermetic) controlam todos os estados do processo.[^6][^12][^5]

## Modelo de ameaças e falhas

O modelo de ameaças para builds não determinísticas cobre três categorias de risco: integridade da cadeia de suprimentos, auditabilidade e confiabilidade operacional.

- **Adulteração no pipeline (supply-chain tampering)**: um atacante insere código malicioso durante a compilação (não no repositório), produzindo um binário diferente do que o código-fonte revisado sugere; sem reprodutibilidade, isso é indetectável por qualquer parte externa.[^3][^1]
- **Comprometimento de dependências transitivas**: pacotes substituídos, removidos do registro ou atualizados silenciosamente alteram a árvore de dependências entre builds, mesmo sem mudança no código do projeto.[^16][^18]
- **Poluição de cache**: cache com chave incompleta (que não reflete todos os inputs reais) retorna artefatos obsoletos ou incorretos, mascarando bugs ou vulnerabilidades corrigidas.[^20]
- **Builder não isolado**: builds executadas em máquinas de desenvolvedores com acesso à rede e ao sistema de arquivos do host permitem vazamento de segredos, dependência de versões de ferramentas não declaradas e resultados não reprodutíveis por terceiros.[^23][^22]
- **Falsificação de proveniência**: provenance gerada e assinada pelo próprio processo de build (não por serviço isolado) pode ser forjada por quem controla o script de build.[^22]
- **Falta de auditabilidade forense**: sem builds reproduzíveis, investigar um incidente de segurança exige confiar no binário fornecido, pois não é possível recriar de forma independente o estado exato que gerou o artefato comprometido.[^3]

## Exemplo de build não reproduzível

Um projeto Go compilado com um Makefile típico:

```makefile
build:
	go build -ldflags "-X main.BuildTime=$(shell date +%s)" -o app ./cmd/app
```

Esse comando embute o timestamp de compilação (data atual real) diretamente no binário via `ldflags`, além de depender da versão do Go instalada no PATH do host, sem lockfile de dependências fixado e sem controle de locale ou de ordenação de arquivos no processo de build. Rodar esse comando em duas máquinas diferentes — ou na mesma máquina em momentos diferentes — produzirá dois binários com hashes SHA-256 distintos, mesmo com o código-fonte idêntico. Esse padrão é o motivo raiz mais frequente de builds não reproduzíveis identificado em estudos empíricos com pacotes Debian, respondendo por centenas de casos catalogados na categoria "Build Timestamp".[^9]

## Exemplo de correção

```makefile
SOURCE_DATE_EPOCH := $(shell git log -1 --format=%ct)
export SOURCE_DATE_EPOCH

build:
	go build -trimpath \
	  -ldflags "-X main.BuildTime=$(SOURCE_DATE_EPOCH) -buildid=" \
	  -o app ./cmd/app
```

A correção fixa o timestamp ao commit (via `SOURCE_DATE_EPOCH` derivado do log do Git, conforme a especificação oficial), usa `-trimpath` para remover caminhos absolutos do sistema de arquivos local embutidos no binário, e neutraliza o build ID não determinístico do Go. Complementarmente, a versão exata do toolchain Go deve ser fixada (via `go.mod`/`toolchain` directive ou `asdf`/`mise`), as dependências devem estar travadas em `go.sum`, e a build deve ocorrer dentro de um container com imagem pinada por digest (não por tag), eliminando variação de sistema operacional, arquitetura e bibliotecas do sistema. A verificação final compara o SHA-256 do binário produzido em duas execuções independentes usando `diffoscope` para inspecionar diferenças remanescentes, caso existam.[^19][^10][^21]

## Causas de builds diferentes a partir do mesmo commit

Pesquisa empírica sobre pacotes Debian identificou uma taxonomia de 16 causas-raiz agrupadas em cinco categorias: Build, Filesystem, Locale, Memory e System. As causas mais relevantes na prática incluem:[^9]

| Causa | Mecanismo | Mitigação típica |
|---|---|---|
| Timestamps | Data/hora de compilação embutida em metadados, logs ou headers de arquivo, gerando binários diferentes mesmo com fonte idêntica | `SOURCE_DATE_EPOCH` e "clamping" de timestamps [^19][^9] |
| Versões de compiladores/SDKs | Diferentes versões geram código-objeto, otimizações ou metadados de debug distintos | Toolchain pinning, containers com versão fixa [^10] |
| Dependências transitivas | Resolução dinâmica de versões (sem lockfile) traz versões diferentes entre execuções | Lockfiles com hashes de integridade [^15][^16] |
| Pacotes removidos/substituídos | Dependência descontinuada no registro força resolução para versão alternativa | Mirrors internos, vendoring, cache de artefatos [^16] |
| Locale | Ordenação de strings, formatação de números e mensagens variam por configuração regional do sistema | Fixar `LC_ALL=C`/`en_US.UTF-8` explicitamente [^9] |
| Timezone | Conversões de data/hora embutidas variam conforme fuso do host | Fixar `TZ=UTC` no ambiente de build [^9] |
| Caminhos absolutos | Caminho do diretório de build embutido em símbolos de debug ou logs | Uso de caminhos relativos, `-trimpath`, builds em diretório canônico | 
| Ordem não determinística | Iteração sobre diretórios, hash maps ou listas de arquivos sem ordenação explícita | Ordenação canônica (ex.: `sort`) antes de empacotar [^9] |
| Sistema operacional/arquitetura | Diferenças de ABI, bibliotecas do sistema e comportamento do linker | Containers/VMs com imagem base fixa por arquitetura [^10] |
| Variáveis de ambiente | Flags de compilador ou comportamento condicionado por env vars não declaradas | Hermeticidade — bloquear/normalizar env vars não essenciais [^11][^14] |
| Arquivos gerados (build IDs, UUIDs) | Identificadores aleatórios inseridos automaticamente por ferramentas | Desabilitar ou fixar geração de IDs (ex.: `-buildid=` no Go) | 
| Acesso à rede durante a build | Download dinâmico de dependências pode trazer versões diferentes entre execuções | Bloquear rede durante execução (network sandboxing) em builds hermeticamente isoladas [^22] |
| Scripts dependentes da máquina local | Scripts que consultam `uname`, hostname, ou binários instalados fora do controle do build | Declarar todas as ferramentas como inputs explícitos (hermeticidade) [^10] |

## Fixação de versões, containers e ambientes efêmeros

Fixar dependências exige lockfiles que registrem versão exata, origem e hash de integridade de cada pacote — direto e transitivo —, permitindo que instalações subsequentes reproduzam exatamente a mesma árvore. Toolchains (compiladores, SDKs, interpretadores) devem ser fixados por versão exata e, idealmente, por hash de binário, tratando-os como parte do código-fonte para efeitos de hermeticidade — essa é justamente a definição do Bazel para builds herméticos, que trata ferramentas como inputs de build, baixando cópias e gerenciando-as em árvores de arquivos controladas.[^10][^12][^15][^17][^18]

Containers, VMs e runners efêmeros devem ser usados quando o objetivo é isolamento reproduzível entre execuções: eles eliminam dependência de estado residual do host (pacotes previamente instalados, variáveis de ambiente de sessões anteriores, arquivos temporários). Runners efêmeros — instâncias de CI descartadas após cada job — são especialmente recomendados quando builds precisam atingir níveis mais altos de SLSA, pois evitam contaminação cruzada entre builds sucessivas na mesma máquina. Ferramentas como Bazel e Nix implementam sandboxing que restringe cada ação de build a exatamente os inputs declarados, bloqueando acesso a diretório home, bibliotecas do sistema não declaradas e rede.[^11][^7][^12][^14][^23]

## Validação de reprodução e comparação de artefatos

O procedimento padrão de validação combina rebuild limpo e comparação binária. O checklist recomendado pela comunidade de reproducible builds inclui: pinar todas as dependências, documentar o ambiente de build, definir `SOURCE_DATE_EPOCH`, executar build limpa (remoção de artefatos/caches anteriores), gerar hash do artefato, limpar e reconstruir, e comparar os hashes das duas execuções. Quando os hashes diferem, a ferramenta `diffoscope` — mantida pelo próprio Reproducible Builds Project — realiza comparação recursiva de arquivos, arquivos comprimidos e diretórios, decompondo formatos binários em representações legíveis para identificar exatamente qual byte ou seção diverge. No ecossistema Bazel, o mesmo princípio é aplicado via `--execution_log_binary_file`, que registra todas as ações de build para comparação entre duas execuções em ambientes distintos, revelando fontes de não hermeticidade.[^13][^21][^20]

## Como o cache pode introduzir resultados incorretos

Sistemas de cache de build (local ou remoto/distribuído) assumem que uma "chave de cache" captura de forma completa todos os inputs relevantes de uma ação. Quando essa chave está incompleta — por exemplo, não incluindo uma variável de ambiente que afeta o resultado, uma versão de ferramenta não declarada ou um arquivo de configuração externo —, o sistema reutiliza um artefato de uma execução anterior que na realidade correspondia a um input diferente, produzindo um resultado incorreto sem qualquer aviso de erro. Esse problema é mitigado justamente pela hermeticidade: quando o build system declara exaustivamente todos os inputs reais de uma ação (Bazel chama isso de "input hermeticity" — os inputs registrados são de fato os inputs usados), a chave de cache reflete corretamente as dependências, tornando o cache seguro e permitindo cacheamento remoto e paralelismo confiáveis. Builds não herméticos com cache habilitado são a causa mais comum de "funciona na minha máquina, mas falha em produção" e de bugs que desaparecem após um `clean build` completo.[^12][^14][^20]

## Auditoria, segurança e investigação de incidentes

Builds reproduzíveis fornecem uma "certeza de que o software é genuíno e não foi adulterado", pois permitem que qualquer parte, incluindo autores originais, distribuidores independentes ou auditores terceirizados, recompile o código-fonte e compare o resultado bit a bit com o binário distribuído. Essa capacidade sustenta três funções de segurança: detecção de inserção maliciosa de código durante o processo de build (não apenas no repositório fonte); suporte a investigação forense de incidentes, ao permitir recriar exatamente o artefato que estava em produção no momento de um compromisso, sem depender de logs incompletos; e fornecimento de evidência auditável para conformidade regulatória em setores críticos. Um estudo empírico sobre adoção corporativa constatou que empresas usam reproducible builds especialmente em domínios de segurança e safety-critical, valorizando-os por rastreabilidade e colaboração no desenvolvimento, embora a proposta de valor de negócio ainda seja considerada intangível por muitas organizações.[^2][^4][^1][^3]

## Limitações práticas para aplicações complexas

Aplicações com toolchains heterogêneos (múltiplas linguagens, geradores de código, plugins de terceiros), builds que dependem de serviços externos legítimos (ex.: fetch de modelos de ML, dados de configuração dinâmica) e sistemas com paralelismo massivo enfrentam dificuldade real em atingir determinismo bit a bit, pois cada camada de ferramenta introduz potencial não determinismo (ordenação de threads, otimizações do compilador sensíveis a hardware, geração de UUIDs/build IDs automáticos). Além disso, builds totalmente herméticas exigem investimento significativo em infraestrutura (sandboxing, cache remoto, gerenciamento próprio de toolchains) que pode não se justificar para times pequenos ou protótipos. A literatura recomenda escolher reprodutibilidade funcional (equivalência de comportamento) em vez de determinismo total quando ferramentas do ecossistema não suportam bem SOURCE_DATE_EPOCH ou quando a velocidade de desenvolvimento tem prioridade sobre verificação criptográfica estrita.[^24][^5][^6][^3][^9]

## Relação com SBOM, assinatura digital e proveniência

Reprodutibilidade, SBOM (Software Bill of Materials) e assinatura digital atuam em camadas complementares da cadeia de confiança de software. O SBOM (nos formatos CycloneDX, mantido pela OWASP e padronizado como ECMA-424, ou SPDX, padronizado como ISO/IEC 5962) documenta exaustivamente componentes, versões, licenças e hashes de todas as dependências de um artefato, servindo de base factual sobre o que compõe o software. A proveniência, formalizada no framework SLSA (Supply-chain Levels for Software Artifacts), atesta como, onde e a partir de qual fonte um artefato foi construído; o SLSA define quatro níveis de build (L0 a L3), do build sem controles (L0) ao builder isolado e endurecido com proveniência não falsificável (L3). A assinatura digital, tipicamente via Sigstore/Cosign, vincula criptograficamente um artefato (por seu hash) a uma identidade verificável, registrando o evento de assinatura em um log de transparência (Rekor) para auditoria pública. A combinação das três práticas — SBOM (o que é), proveniência (como foi feito) e assinatura (quem afirma isso e quando) — mais reprodutibilidade (verificação independente de que o binário corresponde à fonte declarada) forma a base técnica dos controles de integridade de cadeia de suprimentos modernos.[^7][^8][^25][^26][^27][^28][^29][^22]

## Níveis progressivos de maturidade

| Nível | Características | Ferramentas típicas |
|---|---|---|
| 0 — Ad hoc | Build manual, sem script, sem lockfile, sem documentação de ambiente | Nenhuma |
| 1 — Repetível | Build scriptada (Makefile/CI), mas dependências não fixadas por hash e ambiente não isolado | Makefile, script shell |
| 2 — Rebuildable | Dependências fixadas via lockfile, toolchain versionado, mas ainda executando no host sem isolamento total | package-lock.json, go.sum, asdf |
| 3 — Hermético/reproduzível | Build isolada em container/VM/sandbox, SOURCE_DATE_EPOCH definido, verificação de hash entre execuções | Docker digest-pinned, Bazel, Nix, diffoscope [^10][^21] |
| 4 — Auditável com proveniência | Build em runner efêmero hospedado, proveniência assinada automaticamente, SBOM gerado e publicado, verificação por terceiros | SLSA L2/L3, cosign, in-toto, slsa-verifier [^23][^8][^27] |

## Estratégia mínima para equipe pequena

Para times pequenos, a prioridade custo-benefício é: (1) commitar lockfiles de todas as linguagens usadas e rodar instalação em modo "frozen"/"ci" (ex.: `npm ci`, `pip install --require-hashes`); (2) fixar a versão do toolchain principal via arquivo de versão (`.tool-versions`, `go.mod`); (3) rodar a build em container com imagem base fixada por digest, não por tag; (4) exportar `SOURCE_DATE_EPOCH` a partir do timestamp do commit Git; e (5) gerar hash SHA-256 do artefato final em cada execução de CI para detectar divergências ao longo do tempo. Esse conjunto atinge o nível 2-3 de maturidade sem exigir infraestrutura dedicada de build hermético.[^18][^19][^16]

## Estratégia intermediária

Equipes de médio porte devem adicionar: sandboxing de build (Bazel, Nix ou containers com rede bloqueada durante a compilação) para impedir acesso não declarado a recursos externos; runners de CI efêmeros (descartados após cada job) em vez de agentes persistentes; geração automática de SBOM a cada build usando ferramentas como Syft ou CycloneDX-CLI; e testes periódicos de reprodutibilidade (rebuild independente em ambiente diferente e comparação de hash) como parte do pipeline de release, não apenas ad hoc.[^28][^12][^22]

## Estratégia para software crítico

Para software crítico (infraestrutura, segurança, saúde, financeiro), a exigência sobe para atingir SLSA Build L3: builder hospedado e isolado, sem credenciais persistentes, sem acesso à rede exceto por proxy controlado, proveniência gerada por serviço separado do processo de build (não auto-gerada), e assinatura keyless via Sigstore com registro em log de transparência público (Rekor). Adicionalmente, deve-se implementar revisão de duas pessoas em todas as mudanças de código-fonte (two-person review, alinhado ao antigo SLSA nível 4) e, quando viável tecnicamente, builds totalmente herméticas com verificação de reprodutibilidade bit a bit por partes independentes antes da liberação. SBOM completo (incluindo dependências transitivas) deve ser publicado e assinado junto com o artefato para permitir análise de vulnerabilidades contínua.[^8][^26][^30][^23][^22][^28]

## Procedimento para testar reprodutibilidade

1. Fixar o commit exato do código-fonte a ser testado.
2. Executar a build em ambiente A (ex.: container/VM 1), registrando hash SHA-256 do artefato e, se disponível, log de execução detalhado (ex.: Bazel execution log).[^20]
3. Executar `clean` completo (remover cache, artefatos intermediários e diretórios de build).[^13]
4. Executar a mesma build em ambiente B, com sistema operacional, arquitetura ou timing diferente, se possível.
5. Comparar os hashes SHA-256 dos artefatos finais das duas execuções.
6. Se os hashes diferirem, usar `diffoscope` para identificar recursivamente a origem exata da divergência (timestamps, ordenação, metadados, caminhos).[^21]
7. Corrigir a causa raiz identificada (ex.: fixar SOURCE_DATE_EPOCH, normalizar locale/timezone, usar `-trimpath`) e repetir o ciclo até obter hashes idênticos ou aceitar divergência documentada como reprodutibilidade funcional.[^6][^9]
8. Registrar o resultado da verificação como parte da proveniência do artefato (ex.: atestação SLSA).[^8]

## Checklist técnico

- Lockfiles commitados e instalação em modo estrito (sem resolução dinâmica de versões)[^16][^18]
- Toolchain (compilador, SDK, runtime) fixado por versão exata, idealmente por hash[^10]
- `SOURCE_DATE_EPOCH` definido a partir do commit e aplicado a todos os artefatos com timestamp embutido[^19]
- Locale e timezone fixados explicitamente no ambiente de build (`LC_ALL`, `TZ`)[^9]
- Caminhos absolutos removidos ou normalizados (`-trimpath` ou equivalente)[^9]
- Ordenação determinística de arquivos/iterações garantida explicitamente no build script[^9]
- Build executada em container/VM com imagem pinada por digest, isolada de rede durante compilação[^12][^22]
- Cache de build com chave que captura todos os inputs reais (hermeticidade de inputs)[^12]
- Build limpa periódica para validar que resultados não dependem de estado de cache acumulado[^13]
- Verificação de hash entre execuções independentes, com `diffoscope` para diagnóstico de divergências[^21]
- SBOM gerado e publicado junto ao artefato (CycloneDX ou SPDX)[^28]
- Proveniência gerada por serviço separado do processo de build, no mínimo em nível SLSA L1-L2[^8]
- Artefato assinado digitalmente (Sigstore/Cosign) antes da publicação[^26][^27]
- Runners de CI efêmeros para builds que exigem nível SLSA L3[^23]

## Antipadrões

- Embutir a data/hora "atual" real (não do commit) em binários, logs de build ou metadados de pacote.[^19][^9]
- Instalar dependências sem lockfile ou permitir resolução de versões por intervalo (`^`, `~`) em builds de produção.[^16]
- Assinar provenance/atestações usando o mesmo processo/credenciais que executa o build (autoatestação sem isolamento).[^22]
- Confiar em cache de build sem validar que a chave de cache reflete 100% dos inputs reais da ação.[^12]
- Depender de ferramentas ou bibliotecas "que já estão instaladas" no host/CI, sem declará-las explicitamente como inputs.[^11][^10]
- Permitir acesso irrestrito à rede durante a etapa de compilação/linkagem, possibilitando download dinâmico e não controlado de dependências.[^22]
- Rodar builds de release em máquinas de desenvolvedor ou em runners persistentes reutilizados entre projetos.[^23]
- Ignorar diferenças de locale/timezone entre ambientes de desenvolvimento e CI, assumindo que "não deveria importar".[^9]
- Tratar reprodutibilidade como certificação pontual, sem reteste periódico ao longo da evolução do pipeline de build.[^3]

## Tabela de fontes: autoridade, contribuição e limitações

| Fonte | Autoridade | Contribuição principal | Limitações |
|---|---|---|---|
| Reproducible Builds Project (reproducible-builds.org) [^2][^1] | Projeto oficial colaborativo, referência canônica da indústria (propriedade técnica demonstrável) | Definição formal de build reproduzível; especificação SOURCE_DATE_EPOCH [^19] | Foco majoritário em distribuições Linux/pacotes open source; menos cobertura de ecossistemas proprietários |
| Bazel documentation (bazel.build) [^10] | Documentação oficial de ferramenta amplamente adotada (Google) — propriedade técnica demonstrável | Definição formal de hermeticidade e isolamento de build | Específico ao modelo de execução do Bazel; nem toda organização usa Bazel |
| SLSA / OpenSSF (slsa.dev) [^8][^7] | Padrão de segurança formal mantido pela comunidade OpenSSF sob a Linux Foundation — propriedade técnica demonstrável | Framework de níveis de proveniência e build; formalização de tracks de source/build | Terminologia mudou entre v0.1 e v1.0, gerando confusão sobre "nível 4"; requer adoção de tooling específico |
| Artigo acadêmico "Reproducible Builds: Increasing Integrity of Software Supply Chains" (ar5iv/arXiv) [^4] | Literatura acadêmica peer-reviewed | Definição rigorosa e histórico do caso Debian .buildinfo | Foco em contexto Debian; pode não generalizar a todos os ecossistemas de linguagem |
| Estudo empírico Empirical Software Engineering (mcis.cs.queensu.ca) [^9] | Publicação acadêmica com metodologia empírica — evidência empírica | Taxonomia de 16 causas-raiz de builds não reproduzíveis, com frequência quantificada | Amostra baseada em pacotes Debian; pode não cobrir causas específicas de outros ecossistemas (ex.: JVM, .NET) |
| Estudo de adoção empresarial (Springer, link.springer.com) [^3] | Publicação acadêmica peer-reviewed baseada em entrevistas — evidência empírica | Constatação de baixa adoção industrial apesar de valor reconhecido em domínios críticos | Baseado em entrevistas qualitativas; resultados não são estatisticamente generalizáveis |
| diffoscope.org [^21] | Ferramenta oficial mantida pelo Reproducible Builds Project — propriedade técnica demonstrável | Ferramenta padrão de comparação binária/estrutural de artefatos | Requer instalação e conhecimento técnico para interpretar saídas complexas |
| Sigstore/Cosign documentation (docs.sigstore.dev, GitHub) [^26][^27] | Documentação oficial de projeto sob a Open Source Security Foundation — propriedade técnica demonstrável | Mecanismo de assinatura keyless e verificação com log de transparência | Depende de infraestrutura pública (Fulcio/Rekor) ou hospedagem própria; nem todo pipeline suporta OIDC |
| VMware Open Source Blog [^5] | Blog corporativo técnico — recomendação de boas práticas / experiência industrial | Taxonomia prática (repeatable/rebuildable/binary reproducible) útil para comunicação com stakeholders não especialistas | Não é fonte acadêmica nem padrão oficial; terminologia própria pode não coincidir com uso formal |
| npm docs / safeguard.sh / runxbuild.com sobre lockfiles [^17][^15][^16] | Documentação oficial (npm) e blogs técnicos especializados — mistura de propriedade técnica e recomendação de boas práticas | Explicação de mecanismo de lockfile e hashes de integridade | Blogs de terceiros podem conter opiniões prescritivas não universalmente aplicáveis |
| CycloneDX/SPDX comparativos (interlynk.io) [^28] | Blog técnico especializado em SBOM — recomendação de boas práticas | Comparação de formatos de SBOM e casos de uso recomendados | Não é fonte oficial dos próprios padrões (OWASP/Linux Foundation), embora reflita consenso corrente |

---

## References

1. [Reproducible Builds — a set of software development ...](https://reproducible-builds.org/) - Reproducible builds are a set of software development practices that create an independently-verifia...

2. [Definitions — reproducible-builds.org](https://reproducible-builds.org/docs/definition/) - A build is reproducible if given the same source code, build environment and build instructions, any...

3. [On business adoption and use of reproducible builds for open and closed source software](https://link.springer.com/article/10.1007/s11219-022-09607-z) - ## Abstract

Reproducible builds (R-Bs) are software engineering practices that reliably create bit-...

4. [Reproducible Builds: Increasing the Integrity of Software Supply Chains](https://ar5iv.labs.arxiv.org/html/2104.06020) - Although it is possible to increase confidence in Free and Open Source Software (FOSS) by reviewing ...

5. [What Makes a Build Reproducible, Part 1 - Open Source Blog](https://blogs.vmware.com/opensource/2022/07/12/what-makes-a-build-reproducible-part-1/) - Reproducible builds require things like a deterministic build script and caching of intermediate bui...

6. [Reproducibility vs Determinism - Hermetic Builds | Nemorize](https://nemorize.com/roadmaps/hermetic-builds/lessons/reproducibility-vs-determinism) - Distinguish between reproducible builds and deterministic builds, and why both matter. Free lesson f...

7. [What is the SLSA framework? A guide to SLSA levels and ...](https://cloudsmith.com/blog/slsa-a-route-to-tamper-proof-builds-and-secure-software-provenance) - The build track has four levels: SLSA Build Level 0 (“None”): No security controls. SLSA Build Level...

8. [SLSA • Security levels](https://slsa.dev/spec/v0.1/levels) - SLSA is organized into a series of levels that provide increasing integrity guarantees. This gives y...

9. [Empirical Software Engineering manuscript No.](https://mcis.cs.queensu.ca/publications/2024/emse_rahul.pdf)

10. [Hermeticity](https://bazel.build/basics/hermeticity) - When given the same input source code and product configuration, a hermetic build system always retu...

11. [What Is Hermeticity and Why It Matters for Bazel Builds](https://www.hermetiq.com/blog/what-is-hermeticity) - Hermeticity means a build's output is determined entirely by its declared inputs. Learn why it's the...

12. [2.3.1 Hermeticity - Bazel Knowledge Hub](https://bazel.virtuslab.com/book/2~3~1/) - A place to learn Bazel. Backed by 15+ years of experience, 100 projects delivered.

13. [What are Reproducible Builds?](https://www.cleanstart.com/knowledge-hub/what-are-reproducible-builds) - You pull a container image from your registry. How do you know the image in production matches the s...

14. [Bazel Mental Model - Hermetic Builds | Nemorize](https://nemorize.com/roadmaps/hermetic-builds/lessons/bazel-mental-model) - Understand Bazel's approach: explicit BUILD files, action-based execution, and remote caching. Free ...

15. [package-lock.json Use Explained: Why You Should Commit It](https://safeguard.sh/resources/blog/package-lock-json-explained) - What package-lock.json is, the real package lock json use cases, how integrity hashes protect instal...

16. [Should package-lock.json Be Committed?](https://www.runxbuild.com/blog/should-package-lock-json-be-committed/) - Should package-lock.json be committed to git? Yes for every project you actually deploy, with one ex...

17. [package-lock.json](https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json/) - In order to avoid processing the node_modules folder repeatedly, npm as of v7 uses a "hidden" lockfi...

18. [Dependency Updates Should Be...](https://www.lihuanyu.com/en/posts/2022/frontend-dependencies-lockfile-reproducible-builds/) - A practical look at dependency ranges, transitive dependencies, lockfiles, npm ci, pnpm frozen insta...

19. [SOURCE_DATE_EPOCH specification](https://reproducible-builds.org/specs/source-date-epoch/) - SOURCE_DATE_EPOCH. A UNIX timestamp, defined as the number of seconds, excluding leap seconds, since...

20. [How to keep a Bazel project hermetic?](https://tweag.io/blog/2022-09-15-hermetic-bazel/) - A build is hermetic if it is not affected by details of the environment where it is performed. Herme...

21. [diffoscope: in-depth comparison of files, archives, and ...](https://diffoscope.org/)

22. [SLSA Build Provenance: Verifying Supply Chain Integrity from ...](https://www.systemshardening.com/articles/cicd/slsa-build-provenance/) - SLSA (Supply-chain Levels for Software Artifacts) provides a graduated framework for proving that bu...

23. [SLSA Levels Explained: A Practical Compliance Checklist](https://secure-pipelines.com/ci-cd-security/slsa-levels-explained-practical-compliance-checklist/) - A practical checklist for SLSA compliance. Covers Build Levels 0-3, provenance requirements, tooling...

24. [What is Reproducible Builds? Meaning, Architecture ...](https://devsecopsschool.com/blog/reproducible-builds/) - ---

25. [Signing Software The Easy Way with Sigstore and Cosign](https://martinheinz.dev/blog/56) - Signing software artifacts has many obvious benefits such as code integrity or developer (author) au...

26. [Overview - Sigstore](https://docs.sigstore.dev/cosign/signing/overview/) - This document explains how identity-based, or “keyless” signing works in Sigstore. To learn more abo...

27. [GitHub - sigstore/cosign: Code signing and transparency for containers and binaries](https://github.com/sigstore/cosign) - Code signing and transparency for containers and binaries - sigstore/cosign

28. [CycloneDX vs SPDX: Choosing an SBOM Format for ...](https://www.interlynk.io/resources/cyclonedx-vs-spdx-sbom-format) - Across that line, CycloneDX grew from a software SBOM format into a general bill of materials standa...

29. [SBOM Standards & Formats Explained | SPDX vs CycloneDX (Which One Should You Use?)](https://www.youtube.com/watch?v=prhTq3H_BAU) - Presentation: https://workdrive.zohoexternal.in/file/0u4ck4e6366d63da541e19b145ff45eab12ea
_

In thi...

30. [Artifact signing with Cosign and Sigstore - DevSecOpsAtlas](https://devsecopsatlas.com/guides/artifact-signing-cosign-sigstore) - A practical guide to artifact signing with Cosign and Sigstore, including container signatures, keyl...

