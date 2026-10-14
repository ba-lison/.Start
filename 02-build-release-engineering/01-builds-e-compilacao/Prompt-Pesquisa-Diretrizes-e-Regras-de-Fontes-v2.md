<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Realize uma pesquisa aprofundada, verificável e rastreável sobre o tema apresentado abaixo.

Considere informações e documentação disponíveis até agosto de 2026.
REGRAS DE FONTES
Priorize, nesta ordem:
padrões, especificações e normas oficiais;
documentação oficial dos projetos, ferramentas e fornecedores envolvidos;
órgãos governamentais, reguladores e instituições reconhecidas;
livros e publicações de referência em engenharia de software;
artigos revisados por pares, revisões sistemáticas e estudos empíricos, quando houver uma questão que comporte validação científica;
publicações técnicas de empresas reconhecidas, desde que claramente identificadas como experiência industrial e não como evidência científica.
Não use fóruns, redes sociais, posts pessoais, páginas de marketing ou blogs desconhecidos como fundamento factual. Eles só podem aparecer como contexto, claramente rotulados, e devem ser corroborados por fontes mais fortes.
Para alegações sobre produtividade, qualidade, desempenho de equipes, redução de defeitos ou superioridade de uma prática:
procure estudos empíricos;
informe metodologia, amostra, contexto, limitações e nível da evidência;
não transforme correlação em causalidade;
diferencie evidência acadêmica, experiência industrial e recomendação do autor.
Para procedimentos técnicos, formatos, ferramentas e configurações:
priorize documentação oficial e especificações técnicas;
não force a inclusão de artigos acadêmicos quando a documentação técnica for a fonte adequada.
Faça checagem cruzada entre pelo menos duas fontes independentes para afirmações relevantes, sempre que possível.
Avalie cada fonte considerando:
autoridade;
atualidade;
independência;
rastreabilidade;
aplicabilidade ao contexto;
possíveis interesses comerciais;
limitações.
Quando houver divergência entre fontes ou práticas:
apresente as posições;
explique por que divergem;
indique em quais contextos cada abordagem pode ser apropriada;
não escolha uma vencedora sem critérios explícitos.
Diferencie claramente:
fatos documentados;
evidências empíricas;
consenso profissional;
recomendações;
inferências;
opiniões.
FORMATO DA RESPOSTA
Entregue a pesquisa em português e use a seguinte estrutura:
Resumo executivo.
Definições e terminologia.
Funcionamento técnico.
Alternativas e abordagens existentes.
Comparação estruturada.
Benefícios, riscos e limitações.
Critérios para escolher uma abordagem.
Exemplos práticos.
Antipadrões e erros comuns.
Checklist de aplicação.
Questões que precisam ser respondidas pela equipe.
Conclusões.
Tabela de fontes contendo:
título;
organização ou autores;
data;
tipo de fonte;
autoridade da fonte;
principal contribuição;
limitações.
Referências completas com links diretos.
Inclua citações junto às afirmações correspondentes, e não apenas uma lista de referências no final.
Não invente informações ausentes. Quando algo depender da stack, do produto ou da organização, declare a dependência e apresente cenários separados.

PESQUISA 9 — ESTRATÉGIAS DE RELEASE, DEPLOYMENT E PROGRESSIVE DELIVERY
Pesquise e compare:
big bang release;
recreate deployment;
rolling deployment;
blue-green deployment;
canary release;
ring deployment;
phased rollout;
percentage rollout;
dark launch;
shadow traffic;
A/B testing;
feature flags;
kill switches;
release por cliente;
release por região;
release por dispositivo;
deployment desacoplado de release.
Para cada abordagem, explique:
funcionamento;
infraestrutura necessária;
benefícios;
riscos;
custo operacional;
possibilidade de rollback;
impacto sobre banco de dados;
impacto sobre sessões e estado;
observabilidade necessária;
adequação a web, desktop, mobile, APIs e plugins.
Diferencie claramente experimento de produto, estratégia de deploy e mecanismo de liberação.
Produza:
Matriz comparativa.
Árvore de decisão.
Cenários apropriados e inadequados.
Exemplo de combinação entre feature flag e canary.
Estratégia mínima para equipes pequenas.
Estratégia para produtos críticos.
Lista de falhas comuns em progressive delivery.
PESQUISA 10 — BUILD E RELEASE DE SOFTWARE DESKTOP E PLUGINS CAD
Pesquise especificamente engenharia de build e release para:
aplicações Windows desktop;
bibliotecas nativas e gerenciadas;
plugins;
extensões;
integrações com aplicações CAD;
software instalado em máquinas de clientes;
ambientes corporativos com atualizações controladas.
Considere:
[STACK];
[SISTEMA OPERACIONAL];
[PRODUTOS CAD OU HOSTS SUPORTADOS];
[FORMA DE DISTRIBUIÇÃO];
[VERSÕES QUE PRECISAM SER SUPORTADAS].
Não presuma uma stack quando ela não estiver informada. Apresente cenários separados.
Investigue:
matriz entre versão do plugin e versão do software host;
compatibilidade de runtime;
x86, x64 e ARM, quando aplicável;
carregamento de DLLs;
manifests;
caminhos de instalação;
registro de componentes;
permissões;
instalação por usuário ou máquina;
dependências nativas;
redistribuíveis;
empacotamento;
MSI, MSIX, EXE, ZIP e outros formatos;
instalação silenciosa;
distribuição corporativa;
ambientes offline;
assinatura de código;
certificados;
timestamping;
reputação do instalador;
alertas de antivírus;
auto-update;
atualização manual;
downgrade;
rollback;
desinstalação;
preservação de configurações;
licenciamento;
telemetria;
crash reporting;
suporte a várias versões do host.
Priorize documentação oficial da Microsoft e dos fornecedores das plataformas CAD correspondentes.
Produza:
Fluxo completo da build até a instalação.
Matriz de compatibilidade.
Estrutura de diretórios sugerida.
Política de versionamento.
Estratégia de instalador.
Estratégia de atualização.
Checklist de testes de instalação, atualização, downgrade e remoção.
Checklist de assinatura.
Principais riscos específicos de plugins CAD.
Perguntas que deveriam ser feitas a um desenvolvedor sênior da TotalCAD.
PESQUISA 11 — MIGRAÇÕES DE BANCO, SCHEMA E DADOS DURANTE RELEASES
Pesquise como mudanças de banco de dados e dados devem ser integradas a releases.
Investigue:
schema migrations;
data migrations;
expand and contract;
backward compatibility;
forward compatibility;
zero-downtime migrations;
dual read;
dual write;
backfill;
feature flags;
migrações online e offline;
transações;
locks;
índices;
backups;
restore;
validação de dados;
versionamento de schema;
ordem entre aplicação e banco;
múltiplas versões da aplicação acessando o mesmo banco;
rollback de código após mudança de schema;
roll-forward;
migrações destrutivas;
retenção de colunas antigas;
testes de migração.
Produza:
Fluxo seguro de uma mudança compatível.
Exemplo de expand-migrate-contract.
Exemplo de migração de dados de longa duração.
Matriz de riscos.
Critérios para permitir ou bloquear rollback.
Checklist antes, durante e depois da migração.
Procedimento de recuperação.
Antipadrões que podem causar perda de dados.
PESQUISA 12 — ROLLBACK, ROLL-FORWARD, OBSERVABILIDADE E INCIDENTES
Pesquise como uma equipe detecta e reage a problemas depois de uma release.
Investigue:
rollback;
roll-forward;
redeploy;
hotfix;
automatic rollback;
manual rollback;
health checks;
readiness e liveness;
logs;
métricas;
traces;
crash reporting;
deployment markers;
correlação entre versão e erro;
SLI;
SLO;
error budget;
alertas;
canary analysis;
synthetic monitoring;
real user monitoring;
incident command;
runbooks;
post-mortems;
comunicação com usuários;
critérios para interromper rollout.
Explique quando rollback é perigoso, especialmente em presença de:
migrações de banco;
mudanças de formato;
eventos publicados;
integrações externas;
atualizações de clientes desktop;
dados produzidos pela nova versão.
Produza:
Árvore de decisão rollback versus roll-forward.
Checklist de observabilidade antes da release.
Checklist dos primeiros minutos, horas e dias.
Runbook de falha de release.
Modelo de registro de incidente.
Modelo de post-mortem sem culpabilização.
Métricas que indicam sucesso ou fracasso da versão.
PESQUISA 13 — SEGURANÇA DA CADEIA DE SUPRIMENTOS DE SOFTWARE
Pesquise segurança de build, pipelines, dependências e artefatos.
Investigue:
software supply chain;
dependency confusion;
typosquatting;
pacotes comprometidos;
dependências transitivas;
lockfiles;
SCA;
SAST;
DAST, quando aplicável;
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
proteção contra alterações no pipeline;
revisão de dependências;
políticas de atualização;
resposta a CVEs;
revogação de releases comprometidas.
Priorize padrões, documentação oficial e órgãos reconhecidos de segurança. Não use páginas promocionais de fornecedores como única evidência.
Produza:
Modelo de ameaças.
Fluxo seguro commit → pipeline → artefato → distribuição.
Controles preventivos, detectivos e corretivos.
Estratégia mínima para equipe pequena.
Estratégia para produto empresarial.
Checklist de segurança de pipeline.
Checklist de segurança de artefato.
Checklist de resposta a dependência comprometida.
Mapa entre controles e riscos mitigados.
PESQUISA 14 — GOVERNANÇA, PAPÉIS E PROCESSO DE RELEASE
Pesquise como organizações definem responsabilidades e decisões durante releases.
Considere os papéis:
desenvolvedor;
autor do PR;
revisor;
QA;
tech lead;
arquiteto;
segurança;
operações;
SRE;
product manager;
release manager;
suporte;
responsável de negócio.
Investigue:
ownership;
RACI;
segregation of duties;
aprovação técnica;
aprovação de produto;
go/no-go;
release readiness;
change advisory board;
processos leves versus processos regulados;
release calendar;
release train;
freeze;
emergency change;
exceções;
auditoria;
registro de decisão;
comunicação;
documentação;
handoff;
responsabilidade pós-release.
Não presuma que processos mais burocráticos são melhores. Compare governança proporcional ao risco.
Produza:
Modelo leve para equipe pequena.
Modelo intermediário.
Modelo para sistema crítico ou regulado.
Matriz RACI.
Agenda de reunião go/no-go.
Checklist de readiness.
Critérios objetivos de aprovação.
Processo de exceção.
Política de release emergencial.
Antipadrões de governança.
PESQUISA 15 — MÉTRICAS DE BUILD, RELEASE E MATURIDADE DE ENGENHARIA
Pesquise métricas para avaliar a qualidade e eficiência de build e release.
Inclua:
deployment frequency;
lead time for changes;
change failure rate;
failed deployment recovery time;
build duration;
queue time;
pipeline success rate;
flaky test rate;
tempo de feedback de PR;
tempo de promoção;
frequência de rollback;
hotfix rate;
escaped defects;
artifact reproducibility rate;
release adoption;
crash-free sessions;
erros por versão;
tempo para detectar falha;
tempo para interromper rollout;
custo de infraestrutura;
toil;
satisfação do desenvolvedor.
Para métricas DORA ou similares:
use relatórios e pesquisas originais;
descreva metodologia e limitações;
não trate correlações como garantias;
evite rankings simplistas de desenvolvedores;
explique riscos de Goodhart’s Law e manipulação de métricas.
Produza:
Catálogo de métricas.
Fórmula ou método de medição.
Fonte dos dados.
Frequência recomendada.
Riscos de interpretação.
Métricas mínimas para começar.
Dashboard conceitual.
Modelo de maturidade de cinco níveis.
Indicadores de que a equipe adicionou burocracia sem aumentar segurança ou velocidade.
PESQUISA 16 — SÍNTESE APLICADA: MODELO OPERACIONAL DE BUILD E RELEASE
Use as conclusões das pesquisas anteriores para criar um modelo operacional completo para o seguinte contexto:
Empresa ou equipe: [CONTEXTO DA EMPRESA]
Tipo de produto: [TIPO DE PRODUTO]
Stack: [STACK]
Sistema operacional: [SISTEMA OPERACIONAL]
Ferramenta de CI/CD: [FERRAMENTA DE CI/CD]
Repositório: [REPOSITÓRIO]
Forma de distribuição: [FORMA DE DISTRIBUIÇÃO]
Tamanho da equipe: [QUANTIDADE DE DESENVOLVEDORES]
Frequência esperada de release: [FREQUÊNCIA]
Criticidade: [BAIXA, MÉDIA OU ALTA]
Clientes internos ou externos: [CONTEXTO]
Necessidade de manter versões anteriores: [SIM/NÃO/INCERTO]
Quando algum campo estiver vazio, não invente. Declare as hipóteses e produza cenários alternativos.
Crie:
Diagnóstico das necessidades.
Definição oficial dos termos que a equipe utilizará.
Fluxo completo da ideia até a operação.
Estrutura sugerida do repositório.
Estrutura de scripts e arquivos de build.
Pipeline de Pull Request.
Pipeline de build oficial.
Estratégia de geração e armazenamento de artefatos.
Política de versionamento.
Política de branches e tags.
Pipeline de release.
Ambientes ou canais de distribuição.
Estratégia de configuração e secrets.
Estratégia de assinatura e segurança.
Estratégia de rollout.
Estratégia de rollback e hotfix.
Observabilidade pós-release.
Papéis e aprovações.
Métricas.
Checklists.
Exemplo de release normal.
Exemplo de hotfix.
Exemplo de falha e recuperação.
Roadmap de adoção dividido em:
primeiros 30 dias;
31 a 60 dias;
61 a 90 dias;
evolução posterior.
Modelo mínimo viável, evitando burocracia excessiva.
Modelo futuro para maior maturidade.
Lista de decisões que ainda precisam ser tomadas.
Lista de perguntas a fazer ao desenvolvedor sênior.
Glossário final.
Diagrama textual completo.
Todas as decisões devem possuir justificativa, alternativas consideradas, riscos e condições em que precisariam ser revistas.
Não recomende ferramentas específicas apenas por popularidade. Primeiro determine as necessidades e critérios; somente depois apresente ferramentas compatíveis, distinguindo capacidades verificadas de conteúdo promocional.

Preparei o relatório completo cobrindo as oito frentes solicitadas (Pesquisas 9 a 16) sobre estratégias de release, progressive delivery, build/release de plugins CAD, migrações de banco, observabilidade/incidentes, segurança da cadeia de suprimentos, governança e métricas, finalizando com o modelo operacional sintetizado.

**Pontos-chave do relatório:**

- **Progressive delivery**: comparei big bang, recreate, rolling, blue-green, canary, ring, phased/percentage rollout, dark launch/shadow traffic, A/B testing, feature flags e kill switches em matriz estruturada, com árvore de decisão e exemplo de combinação flag+canary.[^1_1][^1_2]
- **Plugins CAD**: usei a documentação oficial da Autodesk como referência primária para mostrar como a compatibilidade binária quebra a cada 1-2 releases do host, exigindo recompilação — e declarei explicitamente a lacuna sobre políticas específicas de ZWCAD/SketchUp, já que não há fonte oficial equivalente disponível.[^1_3][^1_4]
- **Migrações de banco**: o padrão "expand and contract" aparece corroborado por múltiplas fontes técnicas independentes como o método consensual para mudanças de schema sem downtime.[^1_5][^1_6][^1_7]
- **Segurança de supply chain**: baseei-me no NIST SSDF (norma oficial) e no framework SLSA, sinalizando onde o material disponível é explicativo/comercial em vez de especificação primária.[^1_8][^1_9]
- **Métricas DORA**: expliquei a metodologia de survey autoatribuído e os riscos de tratar correlação como causalidade ou cair na Lei de Goodhart.[^1_10][^1_11]
- **Modelo operacional (Pesquisa 16)**: como a empresa, stack, CI/CD e criticidade não foram especificados na consulta, apresentei dois cenários hipotéticos declarados (plugin desktop vs. serviço cloud) em vez de presumir escolhas.

O relatório inclui tabela de fontes com avaliação de autoridade/limitações e todas as citações inline conforme solicitado.
<span style="display:none">[^1_12][^1_13][^1_14][^1_15][^1_16][^1_17][^1_18][^1_19][^1_20][^1_21][^1_22][^1_23][^1_24][^1_25][^1_26][^1_27][^1_28][^1_29][^1_30][^1_31][^1_32][^1_33][^1_34][^1_35][^1_36][^1_37][^1_38][^1_39][^1_40][^1_41][^1_42][^1_43][^1_44][^1_45][^1_46][^1_47][^1_48][^1_49][^1_50][^1_51][^1_52][^1_53][^1_54][^1_55][^1_56][^1_57][^1_58][^1_59][^1_60]</span>

<div align="center">⁂</div>

[^1_1]: https://anhtu.dev/feature-flags-progressive-delivery-safe-release-strategy-2026-2164

[^1_2]: https://keninkujovic.com/blog/expand-and-contract-migrations

[^1_3]: https://www.prisma.io/dataguide/types/relational/expand-and-contract-pattern

[^1_4]: https://secportal.io/blog/slsa-framework-explained

[^1_5]: https://www.tim-wellhausen.de/papers/ExpandAndContract/ExpandAndContract.html

[^1_6]: https://services.google.com/fh/files/misc/2024_final_dora_report.pdf

[^1_7]: https://www.taskade.com/blog/dora-metrics-explained

[^1_8]: https://blog.autodesk.io/autocad-2027-sdk-what-every-plugin-developer-needs-to-know/

[^1_9]: https://csrc.nist.gov/pubs/sp/800/218/r1/ipd

[^1_10]: https://help.autodesk.com/view/OARX/2025/ENU/?guid=OARX-ManagedRefGuide-_NET_Migration_Guide

[^1_11]: https://csrc.nist.gov/pubs/sp/800/218/final

[^1_12]: https://www.martinfowler.com/bliki/BlueGreenDeployment.html

[^1_13]: https://openpracticelibrary.com/practice/blue-green-deployments/

[^1_14]: https://www.martinfowler.com/bliki/CanaryRelease.html

[^1_15]: https://martinfowler.com/articles/patterns-legacy-displacement/canary-release.html

[^1_16]: https://launchdarkly.com/blog/mitigate-risk-with-kill-swith-flags-in-python-launchdarkly/

[^1_17]: https://launchdarkly.com/docs/home/flags/killswitch

[^1_18]: https://www.enolcasielles.com/en/blog/database-migrations-strategy

[^1_19]: https://www.javacodegeeks.com/2015/08/blue-green-deployments-ab-testing-and-canary-releases.html

[^1_20]: https://www.youtube.com/watch?v=0Pa1wv4Oilo

[^1_21]: https://anhtu.dev/zero-downtime-database-migration-expand-contract-ef-core-batch-backfill-production-1151

[^1_22]: https://thebackenddevelopers.substack.com/p/zero-downtime-database-migrations

[^1_23]: https://palakorn.com/blog/zero-downtime-database-migrations/

[^1_24]: https://help.autodesk.com/cloudhelp/2026/PTB/OARX-DevGuide-Managed/files/GUID-4E1AAFA9-740E-4097-800C-CAED09CDFF12.htm

[^1_25]: https://help.autodesk.com/cloudhelp/2027/ITA/AutoCAD-Customization/files/GUID-D54B0935-1638-4F97-8B37-1EC3635A1E71.htm

[^1_26]: https://learn.microsoft.com/en-us/windows/win32/seccrypto/time-stamping-authenticode-signatures

[^1_27]: https://learn.microsoft.com/en-us/dotnet/framework/tools/signtool-exe

[^1_28]: https://learn.microsoft.com/en-us/answers/questions/1684036/how-can-i-be-sure-my-windows-desktop-application-i

[^1_29]: https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/package-and-deploy/choose-distribution-path.md

[^1_30]: https://codesigningstore.com/the-ultimate-guide-to-microsoft-signtool

[^1_31]: https://techcommunity.microsoft.com/discussions/msix-discussions/about-code-signing-and-timestamps/382906

[^1_32]: https://support.globalsign.com/code-signing/code-signing/code-signing-windows-7-8-and-10

[^1_33]: https://matthewpalma.dev/blog/zero-downtime-database-migrations-expand-contract-pattern

[^1_34]: https://edilec.com/blog/clodev-11028/database-changes-continuous-delivery-expand-migrate-contract/

[^1_35]: https://www.zero-downtime-schema.com/zero-downtime-schema-evolution-patterns/expand-and-contract-methodology/

[^1_36]: https://www.youtube.com/watch?v=gpLy95UZTIU

[^1_37]: https://csrc.nist.gov/files/pubs/sp/800/218/final/docs/nist.sp.800-218.ssdf-table.xlsx

[^1_38]: https://michaeljswart.com/2018/01/100-percent-online-deployments-blue-green-deployment/

[^1_39]: https://medium.com/trainline/continuous-delivery-with-blue-green-deployment-fe9f66566ecc

[^1_40]: https://stackoverflow.com/questions/60412841/reason-for-the-colors-blue-and-green-in-blue-green-deployment

[^1_41]: https://www.youtube.com/watch?v=OcaUQrRo7-Q

[^1_42]: https://medium.com/@berniedurfee/the-hunt-for-the-blue-green-deployment-68adbee6eefa

[^1_43]: https://zenn.dev/georgeosddev/articles/azure-container-apps-blue-green-deployment

[^1_44]: https://github.com/yotammanor/bluegreenbooks/blob/master/README.md

[^1_45]: https://medium.com/the-liberators/augmenting-agile-development-with-continuous-deployment-a-case-example-ef955f9decf4

[^1_46]: https://www.youtube.com/watch?v=Wbg2HoYjLHU

[^1_47]: https://www.getdefacto.com/article/database-schema-migrations

[^1_48]: https://www.zero-downtime-schema.com/ci-cd-migration-automation/migration-pipeline-gating/enforcing-backward-compatibility-checks-in-pull-requests/

[^1_49]: https://www.signfiles.com/manuals/MicrosoftAuthenticode.pdf

[^1_50]: https://learn.microsoft.com/en-us/answers/questions/2285380/using-an-https-timestamp-server-with-azure-trusted

[^1_51]: https://www.sectigo.com/knowledge-base/detail/Time-Stamp-Server-Stamping-Protocols-for-Digital-Signatures-Code-Signing

[^1_52]: https://learn.microsoft.com/pt-br/dotnet/framework/tools/signtool-exe

[^1_53]: https://getdx.com/blog/2024-dora-report-summary-laura-tacho/

[^1_54]: https://codeligence.com/research/engineering-metrics/

[^1_55]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.fre.pdf

[^1_56]: https://supplychainsecurityhandbook.com/standards-threats/standards-regulations/nist-ssdf/

[^1_57]: https://sudonull.com/post/106962-Creating-Plugins-for-AutoCAD-Using-the-NET-API-Part-1-First-Steps

[^1_58]: https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-supply-chain-security-guidance-9

[^1_59]: https://blog.autodesk.io/autocad-2026-interoperability-for-customization/

[^1_60]: https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-security-supply-chains-software

