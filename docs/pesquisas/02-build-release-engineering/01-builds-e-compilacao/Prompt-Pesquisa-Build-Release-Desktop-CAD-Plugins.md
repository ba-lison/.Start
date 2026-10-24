<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Realize uma pesquisa técnica aprofundada sobre engenharia de build e release para aplicações desktop e plugins de software CAD.

Considere informações e documentação disponíveis até agosto de 2026.
CONTEXTO A PREENCHER
Stack: [PREENCHER OU DECLARAR DESCONHECIDA]
Sistema operacional: [PREENCHER]
Software CAD ou host: [PREENCHER]
Versões suportadas: [PREENCHER]
Forma atual de distribuição: [PREENCHER OU DECLARAR DESCONHECIDA]
Tipo de plugin: [PREENCHER]
Clientes internos ou externos: [PREENCHER]
Quando um campo estiver desconhecido, não invente. Apresente cenários separados.
OBJETIVO
Explicar como estruturar build, compatibilidade, empacotamento, assinatura, instalação, atualização, suporte e distribuição de aplicações desktop e plugins CAD.
Investigue:
matriz de compatibilidade;
versão do plugin versus versão do host;
runtimes;
.NET Framework;
.NET moderno;
C++;
dependências nativas;
x86;
x64;
ARM;
carregamento de DLLs;
manifests;
caminhos de instalação;
diretórios por usuário;
diretórios por máquina;
registro de componentes;
permissões;
redistribuíveis;
isolamento de dependências;
conflitos de versões;
MSI;
MSIX;
EXE;
ZIP;
instalação silenciosa;
distribuição corporativa;
ambientes offline;
assinatura de código;
certificados;
timestamping;
reputação do instalador;
antivírus;
auto-update;
atualização manual;
downgrade;
rollback;
desinstalação;
preservação de configurações;
licenciamento;
telemetria;
crash reporting.
Priorize documentação oficial da Microsoft e dos fornecedores dos softwares CAD correspondentes.
Investigue:
Como produzir builds para várias versões do host.
Quando são necessários binários diferentes.
Como evitar conflitos de dependências.
Como testar instalação limpa.
Como testar atualização.
Como testar downgrade.
Como testar desinstalação.
Como distribuir atualizações em empresas com políticas restritivas.
Como assinar executáveis e instaladores.
Como rastrear qual versão está instalada em cada cliente.
Como recuperar símbolos e crash dumps.
Como manter compatibilidade sem acumular complexidade excessiva.
FORMATO DA RESPOSTA
Entregue:
Resumo executivo.
Fluxo build → pacote → assinatura → distribuição → instalação.
Matriz de compatibilidade.
Estrutura de diretórios.
Estratégia de build.
Estratégia de instalador.
Estratégia de atualização.
Estratégia de rollback.
Política de versionamento.
Checklist de testes.
Checklist de assinatura.
Principais riscos de plugins CAD.
Perguntas específicas para um desenvolvedor sênior da TotalCAD.
Tabela de fontes com autoridade e limitações.
Inclua citações junto às afirmações.

Preparei um relatório técnico aprofundado sobre engenharia de build e release para aplicações desktop e plugins CAD. Como nenhum campo de contexto foi preenchido (stack, SO, host específico, versões suportadas, forma de distribuição, tipo de plugin, clientes), estruturei a pesquisa por cenários reais documentados oficialmente para os principais hosts relevantes ao seu ecossistema — AutoCAD (.NET managed), ZWCAD (ZRX/travamento de versão), SketchUp (Ruby API + C Extensions) e Enscape (plugin multi-host) — em vez de presumir dados.

O relatório cobre: fluxo build→pacote→assinatura→distribuição→instalação, matriz de compatibilidade host×runtime×arquitetura, estrutura de diretórios (WinSxS, por usuário/máquina), estratégias de instalador (MSI/MSIX/EXE/ZIP), assinatura Authenticode com timestamping RFC 3161, a mudança recente da Microsoft sobre reputação SmartScreen e certificados EV, estratégia de atualização/rollback, symbol servers para crash dumps, checklists de teste e assinatura, principais riscos específicos de plugins CAD, e perguntas direcionadas para validação interna na TotalCAD — já que dados como stack exata, host prioritário e política de distribuição corporativa precisam ser confirmados por vocês antes de qualquer decisão de engenharia.
<span style="display:none">[^1_1][^1_10][^1_11][^1_12][^1_13][^1_14][^1_15][^1_16][^1_17][^1_18][^1_19][^1_2][^1_20][^1_21][^1_22][^1_23][^1_24][^1_25][^1_26][^1_27][^1_28][^1_29][^1_3][^1_30][^1_31][^1_32][^1_33][^1_34][^1_35][^1_36][^1_37][^1_38][^1_39][^1_4][^1_40][^1_41][^1_42][^1_43][^1_44][^1_45][^1_46][^1_47][^1_48][^1_49][^1_5][^1_50][^1_51][^1_52][^1_53][^1_54][^1_55][^1_56][^1_57][^1_58][^1_59][^1_6][^1_60][^1_7][^1_8][^1_9]</span>

<div align="center">⁂</div>

[^1_1]: https://learn.microsoft.com/en-us/windows/win32/seccrypto/time-stamping-authenticode-signatures

[^1_2]: https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/package-and-deploy/choose-distribution-path.md

[^1_3]: https://help.autodesk.com/cloudhelp/2026/RUS/AutoCAD-Customization/files/GUID-A6C680F2-DE2E-418A-A182-E4884073338A.htm

[^1_4]: https://forums.autodesk.com/t5/civil-3d-ideas/autocad-civil-3d-should-be-able-to-load-plugin-with-lower-net/idi-p/14050028

[^1_5]: https://blog.autodesk.io/autocad-2025-update-your-packagecontentsxml-with-runtimerequirements/

[^1_6]: https://keanw.com/2006/10/configuring_aut_1.html

[^1_7]: https://ask.csdn.net/questions/9336109

[^1_8]: https://www.zwsoft.com/support/zwcad-devdoc

[^1_9]: https://www.zwsoft.com/support/zw3d-devdoc

[^1_10]: https://forums.sketchup.com/t/will-my-plugins-be-compatible-with-2018/67170

[^1_11]: https://ruby.sketchup.com/file.ReleaseNotes.html

[^1_12]: https://forums.sketchup.com/t/supporting-several-su-versions-on-mac-osx-for-a-plugin/283445

[^1_13]: https://forums.sketchup.com/t/sketchup-2025-c-sdk-compatibility/319117

[^1_14]: https://docs-chaos.atlassian.net/wiki/spaces/enscape/pages/842039327/System+Requirements

[^1_15]: https://www.sketchupvray.com/233253.html

[^1_16]: https://learn.microsoft.com/en-us/windows/win32/msi/standard-installer-command-line-options

[^1_17]: https://silentinstall.org/msiexec

[^1_18]: https://my-ssl.com/learn/code-signing-timestamping

[^1_19]: https://blog.didierstevens.com/2015/11/24/authenticode-and-timestamping-and-sha256/

[^1_20]: https://www.advancedinstaller.com/silent-install-exe-msi-applications.html

[^1_21]: https://stackoverflow.com/questions/8560166/silent-installation-of-an-msi-package

[^1_22]: https://en.wikipedia.org/wiki/Side-by-side_assembly

[^1_23]: http://rxwen.blogspot.com/2009/12/windows-side-by-side-assemblies.html

[^1_24]: https://store.ptsource.eu/tips/sidebysideassemblies

[^1_25]: https://stackoverflow.com/questions/39052943/how-to-compile-ruby-c-extension-on-ruby-2-2-1-for-sketchup-plugin

[^1_26]: https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/smartscreen-reputation

[^1_27]: https://www.todesktop.com/blog/posts/windows-apps-psa-ev-certs-do-not-grant-immediate-reputation-anymore

[^1_28]: https://news.ycombinator.com/item?id=48078194

[^1_29]: https://codesigningstore.com/code-signing-time-stamping-best-practices

[^1_30]: https://techcommunity.microsoft.com/discussions/microsoftdefenderthreatintelligence/smartscreen-reputation-issue-for-ev-code-signed-windows-application/4537122/replies/4538443

[^1_31]: https://www.cantabilesoftware.com/blog/visual-studio-crash-dumps---a-guide-for-plugin-developers-5b2a666d94e2/

[^1_32]: https://learn.microsoft.com/en-us/windows/win32/dxtecharts/debugging-with-symbols

[^1_33]: https://flylib.com/books/en/4.441.1.22/1/

[^1_34]: https://randomascii.wordpress.com/2013/03/09/symbols-the-microsoft-way/

[^1_35]: https://www.codeproject.com/Tips/449993/Windows-Symbols-and-Dump-Analysis-Quick-Steps

[^1_36]: https://learn.microsoft.com/en-us/visualstudio/debugger/using-dump-files?view=visualstudio

[^1_37]: https://support.globalsign.com/code-signing/code-signing/code-signing-windows-7-8-and-10

[^1_38]: https://stackoverflow.com/questions/1969360/a-way-to-load-dll-from-central-repository

[^1_39]: https://www.signfiles.com/manuals/MicrosoftAuthenticode.pdf

[^1_40]: https://learn.microsoft.com/en-us/windows/win32/msi/installation-of-win32-assemblies

[^1_41]: https://stackoverflow.com/questions/4774284/private-side-by-side-assembly-in-1-dll-is-it-the-same-as-usual-dll

[^1_42]: https://docs.keyfactor.com/signserver/latest/code-signing-with-authenticode-signatures

[^1_43]: https://www.sectigo.com/knowledge-base/detail/Time-Stamp-Server-Stamping-Protocols-for-Digital-Signatures-Code-Signing

[^1_44]: https://ruby.sketchup.com/

[^1_45]: https://github.com/SketchUp/sketchup-live-c-api

[^1_46]: https://www.suapp.com/ruby/file.ReleaseNotes.html

[^1_47]: https://stackoverflow.com/questions/17450330/msiexec-what-is-the-difference-between-qn-and-quiet

[^1_48]: https://learn.microsoft.com/en-in/answers/questions/5884757/microsoft-defender-smartscreen-blocking-the-instal

[^1_49]: https://support.sectigo.com/PS_KnowledgeDetailPageFaq?Id=kA01N000000zFJx

[^1_50]: https://learn.microsoft.com/pt-br/windows/win32/msi/standard-installer-command-line-options

[^1_51]: https://learn.microsoft.com/fr-fr/windows/win32/msi/standard-installer-command-line-options

[^1_52]: https://stackoverflow.com/questions/77101110/smartscreen-still-gives-warning-with-ev-code-signing-certificate

[^1_53]: https://resources.flexera.com/web/media/documents/silent_installs.pdf

[^1_54]: https://stackoverflow.com/questions/2237017/post-mortem-crash-dump-debugging-without-having-the-exact-version-of-a-windows-d

[^1_55]: https://designexpress.eu/support/en/enscape/enscape-en/download-enscape

[^1_56]: https://www.youtube.com/watch?v=lpBazIozkeY

[^1_57]: https://www.youtube.com/watch?v=MOUlwOvc0fo

[^1_58]: https://download.archsupply.com/enscape/

[^1_59]: https://www.one-learn.fr/tutos-guides/utiliser-enscape

[^1_60]: https://www.damaswiki.net/node/121963

