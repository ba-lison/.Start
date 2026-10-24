<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Realize uma pesquisa técnica aprofundada sobre arquitetura de pipelines de Continuous Integration.

Considere informações e documentação disponíveis até agosto de 2026.
OBJETIVO
Explicar como projetar um pipeline de CI rápido, seguro, confiável e capaz de produzir feedback útil para desenvolvedores.
Investigue:
triggers por commit;
triggers por Pull Request;
triggers por tag;
execução manual;
checkout;
restauração de dependências;
compilação;
lint;
análise estática;
testes unitários;
testes de integração;
testes end-to-end;
testes de contrato;
cobertura;
análise de segurança;
geração de SBOM;
empacotamento;
publicação de artefatos;
logs;
notificações;
runners;
agentes;
ambientes limpos;
cache;
paralelização;
matrizes de plataforma;
timeout;
retry;
testes flaky;
quality gates;
status checks;
branch protection.
Explique:
Diferença entre pipeline de Pull Request e pipeline de release.
Quais etapas devem falhar imediatamente.
Quais verificações podem inicialmente gerar apenas alertas.
Como reduzir o tempo de feedback.
Como dividir testes rápidos e testes completos.
Como impedir divergência entre execução local e CI.
Como proteger secrets.
Como aplicar menor privilégio.
Como evitar execução de código não confiável com credenciais.
Como tratar testes instáveis.
Como diagnosticar uma build que falha apenas no CI.
Como produzir um artefato oficial.
CRITÉRIOS DE FONTES
Priorize documentação oficial de plataformas de CI, padrões de segurança e literatura reconhecida de engenharia de entrega.
Não recomende uma ferramenta apenas por popularidade. Compare capacidades e limitações.
FORMATO DA RESPOSTA
Entregue:
Resumo executivo.
Pipeline mínimo.
Pipeline intermediário.
Pipeline para produto crítico.
Diagrama.
YAML ou pseudocódigo conceitual.
Orçamento de tempo por etapa.
Estratégias de cache e paralelização.
Estratégia para flaky tests.
Checklist de segurança.
Checklist de qualidade.
Antipadrões.
Tabela de fontes com autoridade e limitações.
Inclua citações junto às afirmações.

O relatório técnico completo sobre arquitetura de pipelines de CI foi entregue no documento gerado. Ele cobre todos os pontos solicitados: diferenças entre pipeline de PR e release, catálogo completo de estágios (triggers, checkout, dependências, build, lint, análise estática, testes em todas as camadas, cobertura, segurança, SBOM, empacotamento, publicação), pipelines em três níveis de maturidade com YAML conceitual, diagrama de fluxo, orçamento de tempo, estratégias de cache/paralelização/flaky tests, checklists de segurança e qualidade, antipadrões e tabela de fontes com autoridade e limitações.

Pontos-chave que merecem destaque:

- O vetor de ataque mais relevante hoje em dia é o **"pwn request"** via `pull_request_target` combinado com checkout/execução de código de PR de fork — isso expõe segredos e tokens de escrita a código não confiável. A partir de julho de 2026, `actions/checkout` v7 passou a bloquear esse padrão por padrão.
- Testes grandes (mais memória, mais bibliotecas) têm estatisticamente mais chance de serem flaky, segundo dados do Google Testing Blog — a estratégia correta é quarentena automática, não retry indiscriminado.
- SLSA nível 3 (builder isolado, provenance não falsificável) é o padrão de referência para "artefato oficial" em produtos críticos, complementado por SBOM em CycloneDX/SPDX.

Todo o conteúdo foi fundamentado em fontes primárias: documentação oficial do GitHub, OWASP Cheat Series, especificação SLSA/OpenSSF, artigos originais de Martin Fowler, Pact Docs e pesquisas de segurança (Orca Security, GitHub Security Lab).
<span style="display:none">[^1_1][^1_10][^1_11][^1_12][^1_13][^1_14][^1_15][^1_16][^1_17][^1_18][^1_19][^1_2][^1_20][^1_21][^1_22][^1_23][^1_24][^1_25][^1_26][^1_27][^1_28][^1_29][^1_3][^1_30][^1_31][^1_32][^1_33][^1_34][^1_35][^1_36][^1_37][^1_38][^1_39][^1_4][^1_40][^1_41][^1_42][^1_43][^1_44][^1_45][^1_5][^1_6][^1_7][^1_8][^1_9]</span>

<div align="center">⁂</div>

[^1_1]: https://martinfowler.com/articles/continuousIntegration.html

[^1_2]: https://martinfowler.com/articles/originalContinuousIntegration.html

[^1_3]: https://reclaim.cdh.ucla.edu/filedownload.ashx/papersCollection/0khyuU/Continuous_Integration_Martin_Fowler.pdf

[^1_4]: https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html

[^1_5]: https://slsa.dev/spec/v1.0/levels

[^1_6]: https://docs.github.com/en/enterprise-server@3.17/actions/reference/security/securely-using-pull_request_target

[^1_7]: https://github.blog/changelog/2025-11-07-actions-pull_request_target-and-environment-branch-protections-changes/

[^1_8]: https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/

[^1_9]: https://nhimg.org/articles/github-actions-pullrequesttarget-misuse-turns-forks-into-rce/

[^1_10]: https://docs.github.com/en/actions/reference/security/secure-use

[^1_11]: https://docs.pact.io/implementation_guides/python/docs/consumer

[^1_12]: https://qaskills.sh/blog/pact-contract-testing-guide-2026

[^1_13]: https://docs.pact.io/implementation_guides/javascript/docs/consumer

[^1_14]: https://docs.gitlab.com/ci/pipeline_security/slsa/

[^1_15]: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html

[^1_16]: https://owasp.org/www-project-devsecops-guideline/latest/01a-Secrets-Management

[^1_17]: https://testing.googleblog.com/2017/04/

[^1_18]: https://medium.com/@marthinpsrb/continuous-integration-by-martin-fowler-e43d124ddf55

[^1_19]: https://www.systemshardening.com/articles/cicd/slsa-build-provenance/

[^1_20]: https://infisical.com/blog/owasp-secrets-management-cheat-sheet

[^1_21]: https://testing.googleblog.com/2016/05/

[^1_22]: https://aquilax.ai/blog/owasp-secrets-management-environment-variables

[^1_23]: https://orca.security/resources/blog/pull-request-nightmare-github-actions-rce/

[^1_24]: https://www.sourcery.ai/vulnerabilities/yaml-github-actions-security-pull-request-target-code-checkout

[^1_25]: https://github.blog/changelog/2026-06-18-safer-pull_request_target-defaults-for-github-actions-checkout/

[^1_26]: https://safeguard.sh/concepts/slsa-provenance

[^1_27]: https://www.sonarsource.com/resources/library/slsa/

[^1_28]: https://www.wiz.io/academy/application-security/slsa-framework

[^1_29]: https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches

[^1_30]: https://docs.github.com/en/pull-requests/reference/status-checks

[^1_31]: https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-required-status-checks

[^1_32]: https://oneuptime.com/blog/post/2026-01-26-status-checks-github-actions/view

[^1_33]: https://synacktiv.com/publications/github-actions-exploitation-untrusted-input

[^1_34]: https://github.com/marketplace/actions/branch-protection

[^1_35]: https://github.com/orgs/community/discussions/167194

[^1_36]: https://github.com/marketplace/actions/required-status-check

[^1_37]: https://github.com/octokit/request-action/issues/234

[^1_38]: https://www.youtube.com/watch?v=YzZa8DumDm8

[^1_39]: https://secopslog.com/courses/supplychain-deep/sc-slsa/

[^1_40]: https://owasp.org/www-project-devsecops-guideline/latest/

[^1_41]: https://www.youtube.com/watch?v=tV40sLtJOmY

[^1_42]: https://www.appsecengineer.com/blog/advanced-secrets-management-in-devsecops

[^1_43]: https://github.com/orgs/community/discussions/179107

[^1_44]: https://github.com/confidential-containers/cloud-api-adaptor/issues/2214

[^1_45]: https://stackoverflow.com/questions/74193499/github-workflow-with-pull-request-target-as-trigger-event-security-risk

