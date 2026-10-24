<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Realize uma pesquisa aprofundada sobre builds determinísticas, reproduzíveis, herméticas e repetíveis.

Considere informações e documentação disponíveis até agosto de 2026.
OBJETIVO
Explicar como garantir que o mesmo código-fonte e a mesma configuração produzam resultados equivalentes ou idênticos, independentemente da máquina, horário ou pessoa que execute a build.
Investigue e diferencie:
deterministic build;
reproducible build;
hermetic build;
repeatable build;
clean build;
incremental build;
isolated build environment;
dependency pinning;
lockfiles;
toolchain pinning;
environment pinning;
source date epoch;
build cache;
binary comparison;
checksum;
provenance.
Analise as causas de builds diferentes a partir do mesmo commit:
timestamps;
versões de compiladores;
versões de SDKs;
dependências transitivas;
pacotes removidos ou substituídos;
locale;
timezone;
caminhos absolutos;
ordem não determinística;
sistema operacional;
arquitetura;
variáveis de ambiente;
arquivos gerados;
acesso à rede durante a build;
scripts dependentes da máquina local.
Investigue:
Como fixar versões de dependências e toolchains.
Quando utilizar containers, máquinas virtuais ou runners efêmeros.
Como validar a reprodução de uma build.
Como comparar binários e artefatos.
Como cache pode introduzir resultados incorretos.
Como builds reproduzíveis ajudam na auditoria, segurança e investigação de incidentes.
Limitações práticas para aplicações complexas.
Relação com SBOM, assinatura digital e provenance.
Diferenças de exigência entre projetos experimentais, comerciais e críticos.
CRITÉRIOS DE FONTES
Priorize projetos oficiais de reproducible builds, padrões de segurança, documentação oficial de ferramentas e literatura técnica ou acadêmica reconhecida.
Diferencie claramente:
propriedade técnica demonstrável;
recomendação de boas práticas;
experiência industrial;
evidência empírica.
FORMATO DA RESPOSTA
Entregue:
Resumo executivo.
Glossário e diferenças entre os conceitos.
Modelo de ameaças e falhas.
Exemplo de build não reproduzível.
Exemplo de correção.
Níveis progressivos de maturidade.
Estratégia mínima para equipe pequena.
Estratégia intermediária.
Estratégia para software crítico.
Procedimento para testar reprodutibilidade.
Checklist técnico.
Antipadrões.
Tabela de fontes com autoridade, contribuição e limitações.
Inclua citações junto a todas as afirmações técnicas importantes.

Concluí a pesquisa aprofundada sobre builds determinísticas, reproduzíveis, herméticas e repetíveis. O relatório cobre:

- **Diferenciação conceitual precisa**: determinismo (byte a byte) vs reprodutibilidade (formal ou funcional) vs hermeticidade (isolamento) vs repetibilidade (consistência de passos), com base nas definições oficiais do Reproducible Builds Project e do Bazel.
- **16 causas-raiz de builds não determinísticas** identificadas em estudo empírico sobre pacotes Debian, categorizadas em cinco grupos (Build, Filesystem, Locale, Memory, System).
- **Exemplo prático** de build não reproduzível (timestamp embutido via Makefile/Go) e sua correção com `SOURCE_DATE_EPOCH`, `-trimpath` e toolchain pinning.
- **Cinco níveis de maturidade progressiva**, do ad hoc ao nível auditável com proveniência SLSA assinada.
- **Estratégias diferenciadas** para equipe pequena, intermediária e software crítico (SLSA Build L3).
- **Procedimento de teste de reprodutibilidade**, checklist técnico e lista de antipadrões.
- **Relação entre reprodutibilidade, SBOM (CycloneDX/SPDX), proveniência (SLSA/in-toto) e assinatura digital (Sigstore/Cosign)**.
- **Tabela de fontes** classificando cada referência por autoridade (propriedade técnica demonstrável, padrão oficial, evidência empírica acadêmica ou experiência/recomendação industrial) e suas limitações.

O relatório distingue claramente afirmações com base em especificações oficiais (ex.: SOURCE_DATE_EPOCH, definição do Reproducible Builds Project, SLSA) de recomendações de boas práticas de blogs técnicos e de evidências empíricas de estudos acadêmicos sobre adoção industrial e causas reais de não reprodutibilidade.
<span style="display:none">[^1_1][^1_10][^1_11][^1_12][^1_13][^1_14][^1_15][^1_16][^1_17][^1_18][^1_19][^1_2][^1_20][^1_21][^1_22][^1_23][^1_24][^1_25][^1_26][^1_27][^1_28][^1_29][^1_3][^1_30][^1_31][^1_32][^1_33][^1_34][^1_35][^1_36][^1_37][^1_38][^1_39][^1_4][^1_40][^1_41][^1_42][^1_43][^1_44][^1_45][^1_5][^1_6][^1_7][^1_8][^1_9]</span>

<div align="center">⁂</div>

[^1_1]: https://reproducible-builds.org/

[^1_2]: https://reproducible-builds.org/docs/definition/

[^1_3]: https://link.springer.com/article/10.1007/s11219-022-09607-z

[^1_4]: https://ar5iv.labs.arxiv.org/html/2104.06020

[^1_5]: https://blogs.vmware.com/opensource/2022/07/12/what-makes-a-build-reproducible-part-1/

[^1_6]: https://nemorize.com/roadmaps/hermetic-builds/lessons/reproducibility-vs-determinism

[^1_7]: https://cloudsmith.com/blog/slsa-a-route-to-tamper-proof-builds-and-secure-software-provenance

[^1_8]: https://slsa.dev/spec/v0.1/levels

[^1_9]: https://mcis.cs.queensu.ca/publications/2024/emse_rahul.pdf

[^1_10]: https://bazel.build/basics/hermeticity

[^1_11]: https://www.hermetiq.com/blog/what-is-hermeticity

[^1_12]: https://bazel.virtuslab.com/book/2~3~1/

[^1_13]: https://www.cleanstart.com/knowledge-hub/what-are-reproducible-builds

[^1_14]: https://nemorize.com/roadmaps/hermetic-builds/lessons/bazel-mental-model

[^1_15]: https://safeguard.sh/resources/blog/package-lock-json-explained

[^1_16]: https://www.runxbuild.com/blog/should-package-lock-json-be-committed/

[^1_17]: https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json/

[^1_18]: https://www.lihuanyu.com/en/posts/2022/frontend-dependencies-lockfile-reproducible-builds/

[^1_19]: https://reproducible-builds.org/specs/source-date-epoch/

[^1_20]: https://tweag.io/blog/2022-09-15-hermetic-bazel/

[^1_21]: https://diffoscope.org/

[^1_22]: https://www.systemshardening.com/articles/cicd/slsa-build-provenance/

[^1_23]: https://secure-pipelines.com/ci-cd-security/slsa-levels-explained-practical-compliance-checklist/

[^1_24]: https://devsecopsschool.com/blog/reproducible-builds/

[^1_25]: https://martinheinz.dev/blog/56

[^1_26]: https://docs.sigstore.dev/cosign/signing/overview/

[^1_27]: https://github.com/sigstore/cosign

[^1_28]: https://www.interlynk.io/resources/cyclonedx-vs-spdx-sbom-format

[^1_29]: https://www.youtube.com/watch?v=prhTq3H_BAU

[^1_30]: https://devsecopsatlas.com/guides/artifact-signing-cosign-sigstore

[^1_31]: https://en.wikipedia.org/wiki/Reproducible_builds

[^1_32]: https://lutpub.lut.fi/bitstream/handle/10024/135304/MariaGlukhova_ToolsForEnsuringReproducibleBuildsForOpenSourceSoftware.pdf?sequence=2

[^1_33]: https://www.youtube.com/watch?v=4dGQr35yUTY

[^1_34]: https://fossa.com/blog/three-pillars-reproducible-builds/

[^1_35]: https://mesonbuild.com/Reproducible-builds.html

[^1_36]: https://reproducible-builds.org/docs/plans/

[^1_37]: https://www.youtube.com/watch?v=vXlrTsR_to8\&list=PLxNYxgaZ8Rsc3auKhtfIB4qXAYf7whEux

[^1_38]: https://secportal.io/blog/slsa-framework-explained

[^1_39]: https://slsa.dev/blog/2025/07/slsa-e2e

[^1_40]: https://www.sonarsource.com/resources/library/slsa/

[^1_41]: https://edu.chainguard.dev/open-source/sigstore/cosign/cosign-manual-way/

[^1_42]: https://docs.sigstore.dev/cosign/signing/other_types/

[^1_43]: https://edu.chainguard.dev/open-source/sigstore/cosign/an-introduction-to-cosign/

[^1_44]: https://docs.sigstore.dev/quickstart/quickstart-cosign/

[^1_45]: https://docs.sigstore.dev/cosign/verifying/verify/

