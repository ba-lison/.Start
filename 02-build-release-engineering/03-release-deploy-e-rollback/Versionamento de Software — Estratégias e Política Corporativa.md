# Versionamento de Software: Estratégias, Padrões e Política Corporativa

## Resumo Executivo

Versionamento de software é o sistema de identificação única de builds, releases e artefatos que permite rastrear mudanças, comunicar compatibilidade e coordenar dependências entre times, clientes e sistemas automatizados. As duas famílias dominantes são o Semantic Versioning (SemVer), que codifica compatibilidade de API em MAJOR.MINOR.PATCH, e o Calendar Versioning (CalVer), que codifica a data de lançamento (ex.: YYYY.MM). Nenhum dos dois é universalmente superior: SemVer funciona bem quando existe uma API pública clara e consumidores que fazem pinning de dependências, enquanto CalVer é preferível quando o cadenciamento (cadência de lançamento, previsibilidade de suporte) é mais relevante para o usuário do que a natureza da mudança, como em distribuições de sistema operacional e ferramentas com releases contínuas.[^1][^2][^3][^4][^5]

Uma política corporativa coerente deve separar claramente quatro conceitos que frequentemente se confundem: a versão pública/comercial (voltada ao marketing e ao usuário), a versão de build/artefato (identificação técnica interna, frequentemente um número monotônico ou hash de commit), a versão de assembly/pacote (usada por sistemas de binding, como o `AssemblyVersion` do .NET), e a versão de schema/formato de arquivo (compatibilidade de dados). Cada camada de software — aplicação interna, produto desktop distribuído, plugin, biblioteca/API — exige regras específicas de incremento, canais de release (stable/beta/dev/nightly/canary) e políticas de depreciação, detalhadas nas seções abaixo.[^6][^7]

## Comparação entre Modelos de Versionamento

| Modelo | Estrutura | O que comunica | Quando é adequado | Limitações |
|---|---|---|---|---|
| Semantic Versioning (SemVer) | MAJOR.MINOR.PATCH[-prerelease][+build] | Compatibilidade de API (breaking, aditiva, correção)[^1] | Bibliotecas, SDKs, APIs, pacotes com gerenciador de dependências (npm, NuGet, Maven)[^8] | Exige API pública bem definida; ambíguo para aplicações sem "API" clara; não informa cadência ou data[^1] |
| Calendar Versioning (CalVer) | YYYY.MM[.patch] ou YY.MM.DD | Momento do lançamento, cadência[^3] | Distribuições (Ubuntu), ferramentas com releases regulares e previsíveis, produtos SaaS de ciclo fixo[^4] | Não comunica compatibilidade nem magnitude da mudança; duas versões consecutivas podem ter breaking changes sem aviso na própria versão[^3] |
| Sequencial/incremental (build number) | Inteiro crescente (build 4521) ou N por commit | Ordem cronológica de builds, não semântica[^9] | CI/CD interno, builds nightly, rastreabilidade de artefato a commit[^9] | Não comunica compatibilidade nem significado ao usuário final |
| Híbrido (SemVer + build metadata/CalVer) | Ex.: 3.4.0+20260610.a1b2c3 | Combina semântica com rastreabilidade temporal[^10] | Produtos comerciais com necessidade de auditoria e suporte | Maior complexidade de parsing e comunicação |

O SemVer define regras estritas: a versão MAJOR aumenta com mudanças incompatíveis na API pública; MINOR aumenta ao adicionar funcionalidade retrocompatível; PATCH aumenta com correções de bugs retrocompatíveis. Uma vez publicada, uma versão nunca pode ser modificada — qualquer alteração exige nova versão. O CalVer, por outro lado, não define semântica de mudança: apenas padroniza os tokens de data (`YYYY`, `YY`, `MM`, `0M`, `DD`, `0D`), deixando a decisão de quando incrementar a cargo do projeto.[^2][^3][^1]

## Exemplos Válidos e Inválidos

**SemVer válido:**
- `1.9.0 → 1.10.0 → 1.11.0` — incremento numérico correto, sem zeros à esquerda.[^1]
- `1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-beta < 1.0.0-rc.1 < 1.0.0` — ordem de precedência de pré-lançamentos.[^10]
- `2.1.0+20260610.a1b2c3` — metadados de build após o `+`, ignorados na comparação de precedência.[^10]

**SemVer inválido:**
- `1.02.0` — zero à esquerda no MINOR não é permitido.[^1]
- Alterar o conteúdo da versão `2.3.1` já publicada sem incrementar o número — viola a imutabilidade exigida pela especificação.[^1]
- Usar `1.0` sem o componente PATCH quando a política interna exige três dígitos completos.

**CalVer válido:** `2026.08`, `26.08.1` (ano-mês, com patch opcional).[^3]
**CalVer inválido:** misturar convenções, por exemplo `2026.8.15rc` sem definir o significado de `rc` na convenção do projeto, criando ambiguidade de parsing.[^3]

**Build/versão de assembly (.NET) válido:** `AssemblyVersion 2.0.0.0` fixo entre patches, com `AssemblyFileVersion 2.0.0.4521` variando por build — separa binding de rastreabilidade.[^11][^6]
**Inválido:** incrementar `AssemblyVersion` a cada build automatizado (`1.0.0.*`), o que força recompilação de todos os consumidores e quebra binding em cascata.[^12][^13]

## Árvore de Decisão para Escolha de Estratégia

1. O artefato expõe uma API pública consumida por terceiros via gerenciador de dependências (npm, NuGet, Maven, PyPI)? → **Use SemVer** com regras estritas de MAJOR/MINOR/PATCH.[^8][^1]
2. O produto é lançado em cadência fixa e previsível (ex.: a cada 6 meses), e o valor para o usuário está na data/atualidade, não na natureza da mudança? → **Use CalVer**.[^4][^3]
3. É um artefato de build interno/CI, nunca exposto diretamente ao cliente final? → **Use número sequencial monotônico ou hash de commit**, associado à versão pública apenas como metadado.[^9]
4. É uma API HTTP/REST exposta a clientes externos? → **Combine SemVer conceitualmente com versionamento na URI ou header** (ex.: `/v1/`, `Accept: application/vnd.api.v2+json`), reservando o MAJOR para mudanças que quebram contrato.[^14][^15][^8]
5. É um schema de dados ou formato de arquivo? → **Versione via identificador explícito** (`$id`, `schemaVersion`) seguindo semântica aditiva vs. quebra, com testes de compatibilidade automatizados em CI.[^16][^17]
6. Precisa de rastreabilidade forense entre versão pública, commit e artefato binário? → **Adote versionamento em três camadas**: versão comercial (SemVer/CalVer), versão de build (sequencial ou hash), versão de assembly/pacote (fixa, muda só em breaking changes).[^6][^11]

## Quando Aumentar MAJOR, MINOR ou PATCH

Segundo a especificação oficial, dado `MAJOR.MINOR.PATCH`: incrementa-se MAJOR ao introduzir mudanças incompatíveis na API pública; incrementa-se MINOR ao adicionar funcionalidade de forma retrocompatível; incrementa-se PATCH ao corrigir bugs de forma retrocompatível, sendo bug fix definido como uma correção de comportamento interno incorreto. Uma consequência prática pouco intuitiva é que remover uma funcionalidade — mesmo que "pequena" — é sempre um breaking change e exige MAJOR, independentemente do esforço de implementação. Adicionar um campo obrigatório novo em uma API também é breaking (exige MAJOR), enquanto adicionar um campo opcional é aditivo (MINOR).[^2][^8][^16][^1]

## Limitações Práticas do SemVer

O SemVer pressupõe que exista uma "API pública" claramente declarada, o que é ambíguo em aplicações desktop, produtos SaaS internos ou sistemas sem contrato formal. Ele não comunica nada sobre a data de lançamento, cadência ou urgência da atualização, o que é problema em cenários onde compliance ou auditoria exige rastreabilidade temporal. Julgamentos sobre "o que é breaking" são subjetivos: uma mudança de comportamento sutil pode ser tecnicamente retrocompatível na assinatura mas quebrar clientes que dependiam do comportamento antigo, tornando a classificação MAJOR/MINOR/PATCH uma questão de julgamento humano, não uma regra mecânica pura. Além disso, o SemVer não define nada sobre metadados de build determinísticos ou sobre como lidar com múltiplas linhas de suporte simultâneas (LTS vs. current), exigindo convenções adicionais.[^8][^16][^3][^1]

## Quando Calendar Versioning é Mais Adequado

CalVer é preferível quando (a) o projeto tem cadência de release fixa e regular, tornando a data mais informativa que a semântica de mudança, como no Ubuntu, que lança a cada seis meses com LTS a cada dois anos; (b) o público-alvo avalia "quão atualizado" está um pacote mais do que "o que quebrou" (ferramentas de infraestrutura, imagens de sistema); (c) o projeto não tem uma API pública singular e coesa, dificultando a aplicação de regras de breaking change do SemVer. A desvantagem central é que CalVer, por si, não impede nem sinaliza breaking changes — isso precisa ser comunicado por outro canal (changelog, notas de release).[^18][^4][^3]

## Distinguindo Número de Build de Versão Comercial

A versão comercial (ou "versão do produto") é a identificação voltada ao usuário e ao marketing — geralmente SemVer ou CalVer, estável e memorável (ex.: "ZWCAD 2026"). O número de build é um identificador técnico interno, tipicamente monotônico ou derivado de timestamp/CI (ex.: build 4521, ou `20260610.3`), usado para rastrear exatamente qual compilação está em execução, independentemente de ter sido promovida a release pública. Na prática .NET, essa separação é formalizada: `AssemblyInformationalVersion`/`Version` mapeia para a "Product Version" visível ao usuário, enquanto `AssemblyFileVersion` é a versão interna de build exibida nas propriedades do arquivo do Windows, podendo ser incrementada a cada deploy sem alterar a versão comercial. A regra prática recomendada é manter o `AssemblyVersion` fixo (mudando somente em breaking changes) e usar o `FileVersion`/build number para cada compilação, evitando exceções de binding em cascata.[^7][^13][^9][^11][^12][^6]

## Como Identificar Builds Não Oficiais

Builds não oficiais (forks, compilações locais de desenvolvedor, builds de CI não assinados) devem ser marcados de forma inequívoca via metadados de build do SemVer após o `+`, que são ignorados na precedência mas ficam visíveis (ex.: `2.3.1+dev.a1b2c3.unofficial`). Boas práticas incluem: assinatura digital ausente ou chave de assinatura diferente da produção; sufixos explícitos como `-nightly`, `-canary`, `-local` no componente pré-release; ausência de tag Git correspondente em repositório oficial; e inclusão do hash de commit e timestamp de build para auditoria, prática comum em pipelines de CI e no esquema GitCalVer, que deriva versões diretamente do histórico Git (`YYYYMMDD.N`).[^9][^10]

## Canais de Release: Stable, Beta, Dev, Nightly, Canary

Navegadores como o Chrome formalizam múltiplos canais paralelos instaláveis simultaneamente: Canary (builds duas vezes ao dia, apenas testes automatizados), Dev, Beta (cerca de um mês de maturação antes de ir para Stable) e Stable (lançamentos a cada quatro semanas). Esse modelo de "trem de lançamento" (release train) é aplicável a qualquer produto com base de usuários heterogênea em tolerância a risco: cada canal usa a mesma base de código em estágios diferentes de maturidade, com promoção gradual entre eles. Termos como alpha, beta, preview, release candidate (RC), nightly e snapshot são convencionalmente ordenados por precedência crescente de maturidade: alpha < beta < RC < release estável, conforme a regra de precedência de pré-lançamento do SemVer (`1.0.0-alpha < 1.0.0-beta < 1.0.0-rc.1 < 1.0.0`).[^19][^10]

## LTS e Manutenção de Versões Anteriores

O modelo Long Term Support (LTS) do Ubuntu é a referência mais documentada: releases LTS ocorrem a cada dois anos, recebem cinco anos de manutenção de segurança padrão e podem ser estendidos a até quinze anos totais via add-ons pagos (ESM + Legacy). Releases intermediários (não-LTS) recebem apenas nove meses de suporte, incentivando usuários que valorizam estabilidade a migrar apenas entre LTS. O padrão geral extraído desse caso: reserve um subconjunto de releases (tipicamente uma a cada N ciclos) para suporte estendido, comunique claramente a data de fim de vida (EOL) desde o lançamento, e separe suporte em camadas (manutenção padrão vs. manutenção de segurança estendida) para permitir planos comerciais diferenciados.[^5][^20][^21][^22][^23][^4]

## Versionamento de APIs

Para APIs REST, as estratégias mais usadas são versionamento na URI (`/v1/recurso`), em cabeçalho HTTP customizado (`X-API-Version`), em parâmetro de query (`?version=2`) ou via media type (`Accept: application/vnd.api.v2+json`). Não há consenso absoluto: o versionamento na URI é o mais simples e testável manualmente, mas gera proliferação de URLs e rompe o princípio REST de que a URI deve ser um identificador estável do recurso; header/media-type é mais alinhado ao REST mas exige tooling mais sofisticado. Roy Fielding, criador do REST, é frequentemente citado defendendo que a melhor prática de versionamento de API é simplesmente "não versionar", preferindo evolução aditiva e negociação de conteúdo. Na prática majoritária, recomenda-se: escolher uma estratégia única e manter consistência, aplicar princípios de SemVer conceitualmente (MAJOR = breaking, MINOR = aditivo compatível), suportar no mínimo duas versões simultâneas (atual e anterior) com uma janela de transição de 6 a 24 meses, e usar cabeçalhos `Deprecation`/`Sunset` para avisar clientes.[^24][^25][^26][^15][^14][^8]

## Versionamento de Bibliotecas e Pacotes

Bibliotecas publicadas em gerenciadores de pacotes (npm, NuGet, Maven, PyPI) devem seguir SemVer estritamente, pois consumidores fazem "pinning" de intervalos de versão (ex.: `^2.1.0`) esperando que o comportamento definido pelas regras MAJOR/MINOR/PATCH seja respeitado. No ecossistema .NET, a versão de pacote NuGet é normalmente derivada do atributo `<Version>` do projeto, que por padrão também popula `AssemblyVersion`, `AssemblyFileVersion` e `AssemblyInformationalVersion`, salvo configuração explícita em contrário. Uma prática comum e recomendada é manter o `AssemblyVersion` fixo no nível MAJOR (ex.: `2.0.0.0`) mesmo quando o pacote NuGet avança para `2.3.7`, evitando exceções de binding em cascata para consumidores que referenciam a DLL diretamente.[^27][^8][^6]

## Versionamento de Plugins e Compatibilidade com o Host

Plugins (extensões para ZWCAD, SketchUp, AutoCAD, etc.) precisam declarar compatibilidade explícita com a versão do software host, tipicamente via um intervalo de versões suportadas no manifesto do plugin. A prática recomendada combina: (a) versão própria do plugin em SemVer, refletindo mudanças na sua própria API/funcionalidade; (b) uma declaração separada de compatibilidade de host (ex.: "compatível com Host v2024–v2026"), atualizada a cada release major do host; e (c) testes de regressão automatizados contra cada versão de host suportada, análogo ao que se recomenda para múltiplas versões de API em CI/CD. Quando o host introduz breaking changes na sua própria API de extensão, isso deve disparar obrigatoriamente um MAJOR bump no plugin, mesmo que o código do plugin em si não tenha mudado, pois sua compatibilidade externa mudou.[^28][^24]

## Versionamento de Schemas e Formatos de Arquivo

Para schemas JSON/dados, a prática recomendada é incorporar a versão no identificador (`$id`) do schema, seguindo semântica: MAJOR para mudanças que quebram compatibilidade (remoção de campo, renomeação, mudança de tipo, novo campo obrigatório), MINOR para adições opcionais retrocompatíveis, e PATCH para correções não funcionais (documentação, exemplos). Regra central: nunca modificar mutuamente um schema já publicado — qualquer alteração, mesmo pequena, gera uma nova versão imutável, e checagens automatizadas de diff semântico devem rodar em CI para impedir bumps incorretos. Uma alternativa mais simples adotada por algumas organizações é usar apenas um inteiro monotônico de versão major por schema, evitando a ambiguidade do SemVer completo quando o volume de mudanças menores é baixo. Formatos de arquivo binários (ex.: `.dwg`, `.skp`) seguem lógica similar: a versão do formato deve ser embutida no cabeçalho do arquivo, permitindo que versões futuras do software detectem e, idealmente, mantenham capacidade de leitura retroativa (backward-compatible read) mesmo sem suporte de escrita retroativa.[^17][^16]

## Compatibilidade Retroativa, Futura e Breaking Changes

Compatibilidade retroativa (backward compatibility) significa que uma versão nova consegue processar dados/chamadas produzidos por versões anteriores; compatibilidade futura (forward compatibility) significa que uma versão antiga consegue, ao menos parcialmente, processar dados de versões futuras (geralmente ignorando campos desconhecidos). A regra prática mais citada para preservar ambas é a "adição segura": adicionar apenas campos opcionais, nunca remover ou renomear campos existentes, e nunca restringir tipos ou validações já publicadas (ex.: `additionalProperties: false` no root quebra forward compatibility). Uma mudança é definida como breaking sempre que um cliente/consumidor existente, sem alterações no seu código, deixaria de funcionar corretamente após a atualização — isso inclui remoção de endpoint, mudança de tipo de retorno, e adição de campo obrigatório.[^29][^16][^8]

## Depreciação: Comunicação e Prazos

A depreciação deve ser anunciada com bastante antecedência — a literatura recomenda entre 6 e 24 meses de aviso, dependendo da criticidade do sistema — e comunicada por múltiplos canais simultâneos: documentação, changelog, cabeçalhos HTTP dedicados (`Deprecation`, `Sunset`), e contato direto com clientes de alto impacto. Durante o período de transição, é recomendado manter no mínimo duas versões ativas simultaneamente (a atual e a anterior), monitorar ativamente o uso da versão depreciada para identificar clientes que ainda não migraram, e fornecer guias de migração explícitos com exemplos de código antes e depois. Em schemas JSON modernos (2019-09+), a depreciação pode ser formalizada tecnicamente com a palavra-chave `deprecated: true`, frequentemente combinada com um `$ref` apontando para a versão substituta.[^25][^14][^24][^17][^8]

## Impedindo Colisão de Versão entre Artefatos Diferentes

Para impedir que dois artefatos distintos recebam a mesma string de versão, a política deve amarrar a geração de versão a um processo determinístico e único: (a) tags Git imutáveis vinculadas 1:1 a um commit específico, nunca reaplicadas após o push; (b) inclusão de metadados de build (hash de commit, timestamp) no componente `+build` do SemVer, que distingue artefatos mesmo com a mesma versão base "core" (ex.: `2.3.1+a1b2c3` vs `2.3.1+d4e5f6`); (c) pipelines de CI/CD que rejeitam publicação se a versão-alvo já existir no registro de pacotes (comportamento padrão em npm, NuGet e a maioria dos registries, que tratam versões publicadas como imutáveis); e (d) para builds internos, um número sequencial global gerado pelo próprio sistema de CI, nunca definido manualmente por um desenvolvedor, eliminando risco de duplicação por erro humano.[^10][^9][^1]

## Como Versionar Correções de Segurança

Correções de segurança devem ser tratadas como PATCH no SemVer sempre que não alterarem a API pública, mas frequentemente exigem tratamento de prioridade acelerada (backport para todas as linhas de suporte ativas, incluindo LTS). No formato "Keep a Changelog", existe uma categoria dedicada `Security` para documentar exatamente essas correções separadamente de `Fixed` genérico, aumentando a visibilidade para equipes de compliance. Em produtos com múltiplas linhas de suporte simultâneas (ex.: LTS + current), a prática recomendada é fazer backport da correção de segurança para todas as versões suportadas, publicando um PATCH release em cada linha (ex.: `24.04.4`, `22.04.5`) em vez de forçar upgrade de MAJOR.[^30][^31][^32][^33][^4][^1]

## Como Versionar Releases Específicas para um Cliente (Custom Builds)

Builds customizados para um cliente específico devem preservar rastreabilidade à linha de release pública da qual derivam, usando metadados de build do SemVer para identificar o cliente/branch (ex.: `3.4.0+client-acme.b7c9`), nunca criando um novo esquema de numeração paralelo que perca a relação com o produto base. Essa prática permite que a equipe de suporte identifique rapidamente, a partir da string de versão, qual código-base e qual customização estão em execução, sem ambiguidade quanto à precedência (metadados de build não afetam a ordenação semântica).[^10]

## Manutenção de Duas Versões Simultâneas

Manter duas linhas simultâneas (ex.: v1 legado e v2 atual, ou LTS e current) exige branches de manutenção dedicados no controle de versão, pipelines de CI/CD separados que testam ambas as linhas a cada mudança relevante, e uma política explícita de "quais tipos de mudança são permitidos em cada linha" — tipicamente apenas correções de bugs e segurança na linha antiga, e funcionalidade completa na linha atual. O caso do Ubuntu demonstra esse padrão em escala: interim releases recebem apenas 9 meses de suporte enquanto LTS recebe 5+ anos, com políticas de backport de segurança claramente documentadas e datas de fim de vida publicadas com anos de antecedência.[^20][^32][^24][^28]

## Conventional Commits e Geração Automatizada de Release Notes

A especificação Conventional Commits define um formato leve para mensagens de commit — `<tipo>[escopo opcional]: <descrição>` — com tipos padronizados como `feat` (nova funcionalidade) e `fix` (correção de bug), e permite indicar breaking changes via `!` após o tipo/escopo ou uma seção de rodapé `BREAKING CHANGE:`. Essa convenção se integra diretamente ao SemVer: ferramentas de release automatizado (como `semantic-release`) analisam o histórico de commits e decidem automaticamente se o próximo release deve ser MAJOR, MINOR ou PATCH, com base nos tipos de commit encontrados desde a última tag. Tipos comuns incluem `feat` → mapeado para categoria "Added" do changelog e bump MINOR; `fix` → categoria "Fixed" e bump PATCH; `refactor`/`perf` → categoria "Changed"; `docs`/`style`/`test`/`chore` → geralmente excluídos do changelog voltado ao usuário final.[^34][^35][^36][^37][^8]

## Relação entre Versão, Commit e Artefato

Uma política robusta deve garantir rastreabilidade bidirecional: a partir de uma versão publicada, deve ser possível identificar exatamente o commit Git que a gerou (via tag imutável) e o artefato binário resultante (via hash/checksum registrado no pipeline de build). Esquemas como GitCalVer formalizam essa relação de forma determinística, derivando `YYYYMMDD.N` diretamente da data de commit e da posição sequencial dentro do dia, eliminando a necessidade de numeração manual. Na prática corporativa, isso é implementado associando ao processo de build: tag Git → CI dispara build → build gera artefato com metadados embutidos (versão, commit hash, timestamp) → artefato é publicado com checksum registrado, criando uma cadeia auditável completa.[^9][^1]

## Política para Aplicação Interna

Para aplicações de uso interno (ferramentas de suporte, dashboards internos, integrações com Supabase/Firebase), recomenda-se:
- Usar CalVer ou versão sequencial simples (ex.: `2026.08.3`), já que não há consumidores externos fazendo pinning de dependência.[^3]
- Automatizar o deploy continuamente (CI/CD) sem necessidade de canais beta/stable formais, mas mantendo tags Git para cada deploy em produção para rastreabilidade.[^9]
- Documentar mudanças relevantes em um changelog interno simplificado, priorizando rastreabilidade sobre comunicação externa polida.
- Registrar o hash de commit no ambiente de execução (variável de ambiente ou endpoint de health-check) para diagnóstico rápido.

## Política para Software Desktop Distribuído a Clientes

Para produtos desktop (ex.: integrações com ZWCAD, SketchUp), recomenda-se:
- Versão comercial em formato ano ou SemVer simplificado, exibida ao usuário (ex.: "Produto 2026" ou "5.2.0"), separada da versão de build interna.[^11]
- Separar `AssemblyVersion` (fixo, muda apenas em breaking changes de plugin/API interna) de `FileVersion`/build number (incrementado a cada compilação), seguindo o padrão .NET consolidado.[^13][^6]
- Manter canais Stable e Beta/Preview distintos e instaláveis em paralelo quando houver base de early adopters, inspirado no modelo de canais do Chrome.[^19]
- Definir política de LTS se o produto tiver clientes corporativos que exigem estabilidade prolongada, com suporte de segurança estendido claramente documentado, seguindo o modelo Ubuntu (5 anos padrão, extensível).[^4][^20]
- Assinar digitalmente todos os builds oficiais e rejeitar/alertar sobre builds sem assinatura válida como forma de identificar builds não oficiais.[^10]

## Política para Plugin

Para plugins do ecossistema CAD/BIM:
- Versão própria do plugin em SemVer, refletindo mudanças na sua funcionalidade e API de extensão exposta a terceiros (se houver).[^1]
- Manifesto declarando explicitamente o range de versões do host suportadas (ex.: Host >= 2024, <= 2026), atualizado a cada release major do host.
- MAJOR bump obrigatório no plugin sempre que o host quebrar sua API de extensão, independentemente de mudança no código do plugin.
- Testes de regressão automatizados contra cada versão de host ativamente suportada, integrados ao pipeline de CI.[^24]

## Política para Biblioteca ou API

Para bibliotecas e APIs (incluindo integrações Vertex AI, Google Cloud, Supabase):
- SemVer estrito para bibliotecas publicadas em gerenciadores de pacote, com API pública explicitamente declarada em documentação.[^8][^1]
- Para APIs HTTP, versionamento na URI (`/v1/`) para breaking changes major, evitando versionar cada mudança minor/patch na URL, e uso de cabeçalhos `Deprecation`/`Sunset` para avisos.[^14][^8]
- Suporte mínimo de duas versões MAJOR simultâneas, com janela de transição documentada (recomendação de mercado: 6–24 meses).[^25][^14]
- Automação de detecção de breaking changes via diff de especificação OpenAPI/schema em CI, bloqueando merges que introduzam quebra sem bump de MAJOR correspondente.[^16][^8]

## Modelo de Changelog

A convenção "Keep a Changelog" é o padrão de fato: arquivo `CHANGELOG.md` na raiz do repositório, ordem cronológica inversa, com seção `[Unreleased]` no topo e categorias fixas por versão:[^31][^30]

```
# Changelog
Todas as mudanças notáveis deste projeto serão documentadas neste arquivo.
O formato é baseado em Keep a Changelog, e este projeto adere ao Semantic Versioning.

## [Unreleased]
### Added
- Nova funcionalidade em desenvolvimento

## [2.3.0] - 2026-08-01
### Added
- Suporte a exportação IFC 4.3
### Changed
- Melhoria de desempenho no importador DWG (~3x)
### Deprecated
- Endpoint /v1/legacy-import será removido em 2027-02
### Fixed
- Corrigido erro de offset de fuso horário na sincronização de licenças
### Security
- Corrigida vulnerabilidade de validação de token de ativação
```

As seis categorias oficiais são `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed` e `Security`, usando apenas as que se aplicam a cada release.[^30][^31]

## Modelo de Release Notes

Release notes, diferentemente do changelog técnico, são voltadas ao usuário final e devem ser mais narrativas:

```
# Produto X — Versão 5.2.0 (01/08/2026)

## Destaques
- Nova integração com Vertex AI para geração automática de descrições de projeto.
- Desempenho de renderização Enscape até 40% mais rápido em GPUs RTX.

## Correções importantes
- Resolvido problema de licenciamento de rede que causava desconexões intermitentes.

## Avisos de depreciação
- O módulo de licenciamento legado (v1) será descontinuado em fevereiro de 2027. Migre para o módulo v2 antes dessa data.

## Compatibilidade
- Requer ZWCAD 2025 ou superior.
- Não recomendado atualizar em ambientes com plugins de terceiros não certificados para esta versão.
```

## Política de Compatibilidade e Depreciação

1. Toda mudança que quebra contrato existente (API, schema, formato de arquivo, comportamento documentado) exige MAJOR bump e aviso de depreciação prévio de, no mínimo, 6 meses (bibliotecas internas) a 24 meses (APIs públicas críticas).[^14][^25]
2. Anúncios de depreciação devem ocorrer simultaneamente em: release notes, changelog, documentação técnica e, quando aplicável, cabeçalhos HTTP (`Deprecation`, `Sunset`).[^29][^14]
3. Durante a janela de depreciação, a versão antiga permanece funcional e recebe apenas correções de segurança, nunca novas funcionalidades.[^28][^24]
4. Monitoramento ativo de uso da versão depreciada é obrigatório para identificar clientes de alto impacto que ainda não migraram.[^24][^8]
5. Guias de migração com exemplos de código "antes/depois" devem acompanhar todo anúncio de breaking change.[^29][^24]

## Checklist para Criação de Política Corporativa de Versionamento

- [ ] Definir, por tipo de artefato (app interna, desktop, plugin, biblioteca, API, schema), qual modelo de versionamento será usado (SemVer, CalVer, sequencial).[^3][^1]
- [ ] Declarar formalmente o que constitui "API pública" para cada componente, documentando o que está sujeito a garantias de compatibilidade.[^1]
- [ ] Separar explicitamente versão comercial, versão de build e versão de assembly/pacote, com regras de quando cada uma incrementa.[^6][^11]
- [ ] Adotar Conventional Commits e automação de release notes/versionamento a partir do histórico de commits.[^35][^37]
- [ ] Adotar Keep a Changelog como formato padrão de changelog técnico.[^31][^30]
- [ ] Definir política de canais de release (stable/beta/dev/nightly) e critérios de promoção entre eles.[^19]
- [ ] Definir política de LTS: quais releases recebem suporte estendido e por quanto tempo.[^20][^4]
- [ ] Estabelecer processo de backport de correções de segurança para todas as linhas de suporte ativas.[^32]
- [ ] Definir janela mínima de depreciação e canais de comunicação obrigatórios.[^25][^14]
- [ ] Implementar checagem automatizada de breaking changes em CI (diff de OpenAPI/schema) antes de permitir merge.[^16][^8]
- [ ] Garantir imutabilidade de versões publicadas em todos os registries e pipelines.[^1]
- [ ] Definir esquema de metadados de build para builds customizados/não oficiais, permitindo identificação e rastreabilidade.[^9][^10]
- [ ] Documentar compatibilidade explícita entre plugins e versões do software host, com testes automatizados por versão.[^24]
- [ ] Revisar e atualizar a política anualmente, incorporando lições de incidentes de compatibilidade.

## Tabela de Fontes: Autoridade e Limitações

| Fonte | Tipo | Autoridade | Limitações |
|---|---|---|---|
| semver.org / GitHub semver/semver[^1][^2][^38] | Especificação oficial | Alta — padrão de fato adotado por npm, Cargo, muitos ecossistemas | Não cobre CalVer, builds internos, ou versionamento de schema/API HTTP diretamente |
| calver.org[^3] | Especificação de convenção | Média-alta — referência convencional, não um órgão de padronização formal | Não define semântica de mudança, apenas formato de data |
| Conventional Commits (conventionalcommits.org)[^35][^36] | Especificação da comunidade | Alta — amplamente adotada em tooling de release automatizado | Define apenas dois tipos obrigatórios (`feat`, `fix`); extensões variam entre organizações |
| Keep a Changelog (keepachangelog.com)[^30][^31] | Convenção da comunidade | Alta — padrão de fato para changelogs open source | Não é um órgão de padronização formal; convenção, não especificação normativa |
| Microsoft Learn — AssemblyVersion/FileVersion[^6][^7] | Documentação oficial de plataforma | Alta — documentação primária da Microsoft | Específico ao ecossistema .NET; não generaliza a outras plataformas |
| Ubuntu/Canonical — release cycle e LTS[^4][^20][^23] | Documentação oficial de produto | Alta — fonte primária do fornecedor | Modelo específico de distribuição Linux; nem todos os detalhes se generalizam a outros produtos |
| Chromium — release channels[^19] | Documentação oficial de projeto | Alta — fonte primária do projeto Chromium | Modelo de navegador com alto volume de usuários; pode ser oneroso para times pequenos replicar integralmente |
| Artigos de blogs técnicos sobre API versioning (oneuptime, zernio, digitalapi, etc.)[^14][^24][^8][^25] | Conteúdo secundário/opinativo | Média — sintetizam práticas de mercado, mas não são especificações normativas | Refletem opinião e consenso de mercado, não um padrão único; algumas recomendações conflitam entre si (ex.: URI vs. header) |
| Stack Overflow / respostas de comunidade[^27][^12][^13] | Discussão de comunidade | Média — útil para casos práticos, mas não é fonte normativa | Respostas podem refletir opinião individual, não consenso oficial |

---

## References

1. [Semantic Versioning 2.0.0 | Semantic Versioning](https://semver.org/) - Semantic Versioning Specification (SemVer) · Software using Semantic Versioning MUST declare a publi...

2. [Versionamento Semântico 2.0.0](https://semver.org/lang/pt-BR/spec/v2.0.0.html) - Especificação de Versionamento Semântico (SemVer) · Software usando Versionamento Semântico DEVE dec...

3. [Calendar Versioning — CalVer](https://calver.org/calendar_versioning.html) - CalVer is a software versioning convention that is based on your project's release calendar, instead...

4. [Ubuntu release cycle](https://ubuntu.com/about/release-cycle) - Overview of the Ubuntu release cycle - maintenance, support and security coverage, lifetime, upgrade...

5. [How to Understand Ubuntu Release Cycles: LTS vs Interim ...](https://oneuptime.com/blog/post/2026-03-02-ubuntu-release-cycles-lts-vs-interim/view) - A clear explanation of Ubuntu's release cycle, the difference between LTS and interim releases, supp...

6. [.net - What are differences between AssemblyVersion ...](https://stackoverflow.com/questions/64602/what-are-differences-between-assemblyversion-assemblyfileversion-and-assemblyin) - There are three assembly version attributes. What are differences? Is it ok if I use AssemblyVersion...

7. [Use AssemblyVersion and AssemblyFileVersion attributes](https://learn.microsoft.com/en-us/troubleshoot/developer/dotnet/framework/general/assembly-version-assembly-file-version) - This article describes how to use AssemblyVersion and AssemblyFileVersion attributes in the Assembly...

8. [8 API Versioning Best Practices for Developers in 2026](https://zernio.com/blog/api-versioning-best-practices) - Discover top API versioning best practices for scalable and maintainable APIs. Learn about URL, head...

9. [Specification - gitcalver—calendar versioning from git history](https://gitcalver.org/spec) - GitCalVer specification—version format, algorithm, output formats, and edge cases

10. [Semantic Versioning 2.0.0 | Octopus blog](https://octopus.com/blog/semver2) - Octopus 3.4 includes support for Semantic Versioning 2.0.0.

11. [Creating a .NET Assembly, the importance of versioning](https://community.dynamics.com/blogs/post/?postid=65db6310-10f8-4eb5-afb7-bfdb09590ed7) - But basically, the file version is merely the version show in File Details, while the assembly versi...

12. [How best to use File Version and Assembly Version?](https://stackoverflow.com/questions/14770/how-best-to-use-file-version-and-assembly-version/1723652) - In .NET there are two version numbers available when building a project, File Version and Assembly V...

13. [Version Properly using AssemblyVersion and AssemblyFileVersion](https://jaylee.org/archive/2010/07/18/version-properly-using-assemblyversion-and-assemblyfileversion.html)

14. [How to Version REST APIs Effectively](https://oneuptime.com/blog/post/2026-01-26-rest-api-versioning/view) - A practical guide to implementing REST API versioning strategies that keep your clients happy while ...

15. [Versioning a REST API - Baeldung](https://www.baeldung.com/rest-versioning) - Versioning a REST API - version identifier in the URI or version media types?

16. [JSON Schema Versioning & Evolution Guide (2026)](https://zerodatatools.com/blog/json-schema-versioning-guide/) - A complete guide to JSON schema versioning and evolution. Learn how to manage backwards compatibilit...

17. [Schema Evolution and Versioning - Sourcemeta One](https://one.sourcemeta.com/guide/evolution/)

18. [Ubuntu - Wikipedia](https://en.wikipedia.org/wiki/Ubuntu)

19. [Chrome Release Channels](https://www.chromium.org/getting-involved/chrome-release-channels/)

20. [Ubuntu](https://endoflife.date/ubuntu) - Check end-of-life, release policy and support schedule for Ubuntu.

21. [supportperiod](https://wiki.ubuntu.com/supportperiod)

22. [Ubuntu Releases](https://releases.ubuntu.com/)

23. [Ubuntu LTS Support Extended to 15 Years](https://linuxblog.io/ubuntu-lts-support-extended-15-years/) - If you’re like me and value a rock-solid Linux setup that you don’t have to reinstall every couple o...

24. [Backend REST API Versioning: A Deep Dive Into ...](https://dev.to/mrgio7/backend-rest-api-versioning-a-deep-dive-into-strategies-nightmares-and-best-practices-22f0) - Published by Giorgi Bakashvili, Senior Software Engineer. Connect with me on LinkedIn. If you think....

25. [REST API Versioning: Definition, Strategies & Best Practices](https://www.digitalapi.ai/blogs/rest-versioning-definition-best-practices-pros-cons-and-when-to-use) - Master REST API versioning in 2026 URI, Header, Query & Media Type strategies explained. Pros, cons ...

26. [What’s the Best Strategy for Versioning a RESTful API?](https://medium.com/@edgar/whats-the-best-strategy-for-versioning-a-restful-api-29b90dbbcf5f) - There are several approaches used across the industry today for versioning RESTful APIs, the most co...

27. [What is the difference between Version, AssemblyVersion, and FileVersion?](https://www.reddit.com/r/dotnet/comments/u4a16a/what_is_the_difference_between_version/)

28. [API Versioning: Strategies & Best Practices - xMatters](https://www.xmatters.com/blog/api-versioning-strategies) - Learn API versioning strategies and best practices. Explore implementation techniques, popular tools...

29. [JSON Migration & Versioning — Evolving Schemas Without ...](https://devtoolsbuilder.com/learn/json/migration-versioning) - Version JSON APIs and evolve schemas without breaking existing clients.

30. [Keep a Changelog Format Explained - ReleasePad](https://www.releasepad.io/blog/keep-a-changelog/) - Keep a Changelog just shipped version 2.0.0 — its first major revision since 2017. The six categorie...

31. [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) - Don’t let your friends dump git logs into changelogs.

32. [Releases](https://wiki.ubuntu.com/Releases%20)

33. [List of releases - Ubuntu project documentation](https://ubuntu.com/project/docs/release-team/list-of-releases/) - Releases are published by the Release Team in a time-based fashion (rather than a feature-based one)...

34. [Changelog Best Practices: How to Publish Release Notes](https://unmarkdown.com/blog/changelog-best-practices) - A complete guide to changelog best practices covering the Keep a Changelog standard, Conventional Co...

35. [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) - A specification for adding human and machine readable meaning to commit messages

36. [Conventional Commits Specification - Wikipedia](https://en.wikipedia.org/wiki/Conventional_Commits_Specification)

37. [Conventional Commits | Cheatsheets](https://ashababnoor.github.io/cheatsheets/conventional-commits) - Reference for the Conventional Commits specification — commit types, scopes, breaking changes, and m...

38. [semver/semver: Semantic Versioning Specification](https://github.com/semver/semver) - Semantic Versioning Specification. Contribute to semver/semver development by creating an account on...

