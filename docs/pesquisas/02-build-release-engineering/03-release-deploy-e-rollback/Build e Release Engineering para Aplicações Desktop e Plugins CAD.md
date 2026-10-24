## Resumo Executivo

Build e release engineering para aplicações desktop e plugins CAD envolve uma cadeia com múltiplos pontos de falha específicos do ecossistema Windows: compatibilidade binária entre versões do host (AutoCAD, ZWCAD, SketchUp, Revit, Rhino etc.), arquitetura de runtime (.NET Framework vs .NET moderno vs C++ nativo), empacotamento (MSI/MSIX/EXE/ZIP), assinatura de código e reputação de reconhecimento (SmartScreen), estratégias de atualização/rollback e rastreabilidade via telemetria e crash reporting.[^1][^2][^3]

Como nenhum dado concreto foi fornecido sobre a stack real, sistema operacional alvo, versões suportadas ou forma de distribuição da TotalCAD, este relatório apresenta **cenários alternativos** cobrindo os principais hosts CAD relevantes ao contexto do usuário (ZWCAD, AutoCAD, SketchUp, Enscape/Vectorworks/Rhino/ArchiCAD), com base em documentação oficial e da comunidade de desenvolvedores, e sinaliza explicitamente onde a informação é desconhecida ao invés de presumi-la.

## Cenários por Host CAD (dados confirmados via documentação oficial)

Nenhum campo de contexto (stack, SO, host, versões suportadas, forma de distribuição, tipo de plugin, clientes) foi preenchido na consulta. Por isso, a análise é organizada por host, com dados reais coletados de fontes oficiais.

### AutoCAD (.NET managed)

A Autodesk publica uma tabela oficial de compatibilidade entre versão do AutoCAD, versão do "Managed .NET runtime toolkit" e versão do .NET Framework/.NET moderno exigida:[^3]

| AutoCAD | Versão interna | Toolkits .NET suportados | Runtime .NET |
|---|---|---|---|
| AutoCAD 2026 | 25.1 | AutoCAD 2026, 2025 | .NET 8.0 |
| AutoCAD 2025 | 25.0 | AutoCAD 2025 | .NET 8.0 |
| AutoCAD 2024 | 24.3 | AutoCAD 2024–2021 | .NET Framework 4.8 |
| AutoCAD 2023 | 24.2 | AutoCAD 2023–2021 | .NET Framework 4.8 |
| AutoCAD 2022 | 24.1 | AutoCAD 2022–2021 | .NET Framework 4.8 |

A partir do AutoCAD 2025 houve migração de .NET Framework para .NET (Core) 8.0, quebrando compatibilidade binária com plugins compilados para versões anteriores — o que o próprio fórum de desenvolvedores da Autodesk confirma ao discutir a limitação de o host não conseguir carregar plugins compilados para uma versão de .NET inferior à sua própria. Desde o AutoCAD 2025, o mecanismo Autoloader passou a exigir o atributo `RuntimeRequirements` com `SeriesMin`/`SeriesMax` no `PackageContents.xml`; sem isso, o Autoloader assume compatibilidade total e pode carregar um plugin incompatível, causando falha em runtime em vez de bloqueio antecipado. Um exemplo oficial:[^4][^5][^3]

```xml
<RuntimeRequirements OS="Win64" SeriesMin="R25.0" SeriesMax="R25.0" />
```

Historicamente, versões mais antigas do AutoCAD também permitiam fixar a versão do CLR via arquivo `.config` da aplicação host, usando `<supportedRuntime version="v2.0.50727"/>` para forçar um runtime específico quando múltiplas versões coexistiam na máquina.[^6]

### ZWCAD (.NET/C++ nativo, ZRX)

A ZWSOFT adota um modelo de "travamento de versão" mais estrito que a Autodesk: o SDK de cada versão do ZWCAD é hard-coded para exigir correspondência exata entre o `AssemblyVersion`/`ProductVersion` da DLL `ZwSoft.ZwAPI.dll` compilada e a versão do executável `ZwCAD.exe` em runtime — divergências causam falha de carregamento silenciosa. Desde 2022, o ZWCAD abandonou totalmente o suporte a 32-bit, exigindo builds x64. O runtime exigido não é o .NET SDK, mas o **.NET Desktop Runtime** (ex.: 6.0), instalado separadamente; instalar apenas o .NET 8 sem o Desktop Runtime correspondente causa `PlatformNotSupportedException`. O registro do plugin ocorre via chave de usuário `HKEY_CURRENT_USER\Software\ZWSOFT\ZWCAD\<versão>\ACAD\ZRX`, apontando para o caminho absoluto da DLL — não há dependência do registro global do Windows. Essa fonte é um fórum de desenvolvedores/CSDN, não documentação primária da ZWSOFT — deve ser tratada como indicativa e validada contra o SDK oficial da ZWSOFT.[^7][^8][^9]

### SketchUp (Ruby API + C Extensions)

A documentação oficial do Ruby API do SketchUp afirma retrocompatibilidade da API Ruby pura, mas exige recompilação de extensões C/C++ a cada mudança de versão do interpretador Ruby embutido. Histórico de versões do Ruby por release do SketchUp:[^10][^11]

| SketchUp | Versão Ruby |
|---|---|
| 2014–2016 | Ruby 2.0 |
| 2017–2020 | Ruby 2.2 → 2.5 |
| 2021 | Ruby 2.7.1/2.7.2 |
| 2022–2023 | Ruby 2.7.x (`x64-msvcrt-ruby270`) |
| 2024–2025 | Ruby 3.2 (`x64-ucrt-ruby320`) |

Desenvolvedores relatam necessidade de compilar dois binários separados (uma DLL por família Ruby) e escolher em runtime qual carregar via detecção de `RUBY_PLATFORM`/versão do SketchUp. A prática recomendada oficial inclui checar `Sketchup.version` programaticamente na inicialização da extensão e bloquear/alertar o usuário se a versão mínima não for atendida. Desde SketchUp 2022, binários universais macOS (Apple Silicon + Intel) são suportados nativamente pelo C SDK.[^12][^13][^11]

### Enscape (plugin multi-host)

A Chaos/Enscape documenta oficialmente matriz de compatibilidade por host, versão e política de suporte com prazo definido ("soft support"):[^14]

| Host | Versões suportadas (jul/2026) | Soft support |
|---|---|---|
| Revit | 2023, 2024, 2025, 2026, 2027 | 2 anos |
| SketchUp | 2024, 2025, 2026 | 1 ano |
| Rhino | 7.0, 8.3+ | 2 anos |
| ArchiCAD | 27, 28, 29 | — |
| Vectorworks | 2022–2025 | — |

O instalador único do Enscape para Windows detecta e instala componentes para todos os hosts compatíveis presentes na máquina, dispensando instaladores dedicados por host — abordagem relevante para produtos com múltiplos plugins CAD simultâneos. Revit LT é explicitamente não suportado por restrição da própria Autodesk sobre plugins de terceiros em edições LT.[^15]

## Fluxo Build → Pacote → Assinatura → Distribuição → Instalação

1. **Build** — compilar por combinação (arquitetura × runtime × versão do host), com verificação automatizada de correspondência de versão entre SDK e binário do host, como no exemplo de script PowerShell do ecossistema ZWCAD que compara `ProductVersion` do SDK e do executável antes de empacotar.[^7]
2. **Empacotamento** — MSI para instalação corporativa gerenciada (GPO/SCCM/Intune) e suporte nativo a propriedades públicas (`INSTALLDIR`, `ALLUSERS`); MSIX quando distribuição via Microsoft Store ou MDM moderno é viável; EXE/ZIP para instalação simples ou side-load manual.[^2][^16][^17]
3. **Assinatura** — Authenticode via `signtool sign /fd SHA256 /tr <URL-TSA> /td SHA256`, sempre com timestamping RFC 3161, aplicada a todo executável, DLL e instalador antes da distribuição.[^18][^19]
4. **Distribuição** — hospedagem HTTPS versionada, canal de update próprio ou catálogo corporativo (Intune/SCCM); se via Microsoft Store, o publisher deve possuir certificado encadeado à Microsoft Trusted Root Program.[^2]
5. **Instalação** — silenciosa via `msiexec /i pacote.msi /quiet /norestart /l*v log.txt` com log verboso obrigatório para diagnóstico pós-falha em ambientes sem interface.[^16][^20][^21]

## Matriz de Compatibilidade — Modelo Genérico

| Dimensão | Pergunta a validar | Fonte de verdade |
|---|---|---|
| Versão do host vs versão do plugin | O host aceita binário compilado contra SDK mais antigo/novo? | Doc do fabricante do host (ex.: tabela .NET da Autodesk[^3], `RuntimeRequirements`[^5]) |
| Arquitetura (x86/x64/ARM) | O host é 32-bit, 64-bit ou nativo ARM64? | Requisitos de sistema do host — desconhecido para ZWCAD/SketchUp sem confirmação do usuário |
| Runtime (.NET Framework, .NET moderno, C++/CRT) | Qual CLR ou CRT o host hospeda internamente? | Doc oficial do host — variável por versão (ver AutoCAD 2024 vs 2025[^3]) |
| Manifest/Autoloader | O host exige manifest declarando compatibilidade? | `PackageContents.xml` (AutoCAD)[^5] |

Como o software CAD-alvo específico da TotalCAD não foi informado, cada linha desta matriz deve ser preenchida separadamente por produto (ZWCAD, SketchUp, Enscape, outros Trimble) antes de qualquer decisão de build.

## Estrutura de Diretórios (referência Windows)

Sem confirmação da stack real, apresentam-se os padrões documentados pela Microsoft e pelos fabricantes CAD:

- Instalação por máquina (`%ProgramFiles%`) exige elevação administrativa e é a prática padrão para MSI corporativo.[^16]
- Instalação por usuário (`%LocalAppData%` ou perfil do usuário) evita necessidade de privilégio administrativo, comum em extensões carregadas via registro de usuário (ex.: chave `HKEY_CURRENT_USER` do ZWCAD).[^7]
- Assemblies compartilhados versionados (side-by-side) residem em `%SystemRoot%\WinSxS`, isolando versões conflitantes da mesma DLL por meio de manifest com hash exclusivo por versão.[^22][^23]
- Aplicações que dependem de CRT/Assemblies do Visual C++ compilados com SxS precisam do manifest complementar (`*.manifest`) e podem ser afetadas pela chave de registro `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\SideBySide\Winners`, que decide qual versão "vence" mesmo contrariando o manifest do app.[^24]

## Estratégia de Build

- Definir builds distintos sempre que o host mudar de runtime major (ex.: AutoCAD 2024 .NET Framework 4.8 → AutoCAD 2025 .NET 8.0 exige recompilação, não apenas reconfiguração).[^3]
- Automatizar checagem de correspondência de versão SDK↔host antes da assinatura, evitando publicar binário incompatível de forma silenciosa.[^7]
- Para plugins com componentes nativos (C/C++), compilar variantes distintas por versão do runtime do host embutido, como no caso do Ruby C Extension do SketchUp, que exige recompilação por versão do Ruby (2.7 vs 3.2).[^13][^25][^12]
- Declarar limites explícitos de compatibilidade em manifest sempre que o mecanismo do host suportar (`RuntimeRequirements SeriesMin/SeriesMax` no AutoCAD).[^5]

## Estratégia de Instalador

| Formato | Melhor uso | Assinatura exigida | Auto-update nativo |
|---|---|---|---|
| MSI | Distribuição corporativa via GPO/SCCM/Intune | Recomendada | Não — responsabilidade do app |
| MSIX | Apps novos, WinUI, Store | Grátis se via Store (Microsoft re-assina) | Sim, nativo |
| EXE (bootstrapper) | Apps legados com lógica de instalação própria | Obrigatória para Store e para evitar SmartScreen | Não |
| ZIP | Distribuição manual/offline sem instalador | Opcional, mas recomendada nos binários internos | Não |

A submissão de instalador MSI/EXE à Microsoft Store, mesmo sem empacotamento MSIX, exige certificado encadeado à cadeia raiz confiável da Microsoft — certificado autoassinado não é aceito.[^2]

## Estratégia de Atualização e Rollback

- Silent update reaproveita os mesmos parâmetros de instalação silenciosa (`/quiet`, `/qn`), com log obrigatório (`/l*v`) para diagnosticar falhas sem interface visível ao usuário.[^21][^16]
- MSP (patch) permite atualização incremental via `msiexec /update pacote.msp /quiet`.[^16]
- Para rollback/downgrade, o Windows Installer bloqueia downgrade direto por padrão (major upgrade requer desinstalação/reinstalação da versão anterior ou lógica de versionamento explícita no MSI); testar esse cenário isoladamente é necessário porque o comportamento não é uniformemente documentado por todos os fabricantes.
- Preservação de configurações do usuário deve ser testada explicitamente durante upgrade/downgrade/desinstalação, pois arquivos de configuração podem residir em diretórios por usuário e não são necessariamente removidos ou versionados pelo instalador MSI padrão.

## Assinatura de Código e Reputação (SmartScreen)

A publicação via Microsoft Store elimina totalmente avisos do SmartScreen, pois o app é assinado pelo certificado da própria Microsoft. Fora da Store, mesmo com certificado válido (OV ou EV), o SmartScreen ainda exibirá aviso de "app não reconhecido" até que reputação suficiente se acumule — a Microsoft removeu em 2024 o benefício de reputação instantânea que certificados EV historicamente concediam. Pontos centrais confirmados por documentação oficial da Microsoft:[^26][^27][^28]

- Reputação é vinculada ao hash do arquivo e à identidade do publisher; trocar de certificado ou publisher reseta a reputação acumulada.[^26]
- Não existe mecanismo de submissão manual de reputação para endpoints de consumidor — o acúmulo é orgânico, via volume de downloads e execuções limpas.[^27][^26]
- Timestamping RFC 3161 é obrigatório para que a assinatura permaneça válida após a expiração do certificado.[^29][^1][^18]
- Assinar consistentemente todos os artefatos (instalador, EXE embutidos, DLLs, MSI) com a mesma identidade de publisher acelera o acúmulo de reputação entre versões.[^30]

## Rastreamento de Versão Instalada e Diagnóstico

Rastrear qual versão está instalada por cliente depende de telemetria própria (não documentado nas fontes oficiais consultadas) combinada com metadados do MSI (ProductVersion, UpgradeCode) consultáveis via registro do Windows Installer. Para crash reporting:

- Gerar PDBs em builds de release (`/Zi`, `/DEBUG` no linker do MSVC) e arquivá-los junto ao binário correspondente por build.[^31]
- Manter um symbol server próprio (via `symstore.exe`) indexado por build/versão, permitindo que o WinDBG/Visual Studio resolvam automaticamente símbolos de qualquer dump recebido de cliente.[^32][^33][^34][^31]
- Configurar `_NT_SYMBOL_PATH` apontando para o symbol server interno e, em cascata, para o Microsoft Symbol Server público.[^35][^32]
- Minidumps contêm apenas registradores, pilha e trace — para diagnósticos completos de estado de memória, capturar dump com heap (Visual Studio) ou dump completo (`Task Manager > Create Dump File`).[^36][^31]

## Checklist de Testes

- Instalação limpa em máquina sem versões anteriores do plugin nem do host.
- Instalação com múltiplas versões do host coexistindo, validando isolamento por manifest/registro.
- Atualização direta de N-1 para N, validando preservação de configurações e licença.
- Downgrade de N para N-1, validando bloqueio esperado ou comportamento documentado.
- Desinstalação completa, validando remoção de chaves de registro, arquivos por usuário e por máquina.
- Instalação silenciosa via linha de comando em ambiente sem GUI, validando log e código de retorno.
- Instalação offline sem acesso à internet, validando ausência de dependência de download de redistribuíveis em runtime.
- Verificação de assinatura pós-build (`signtool verify /pa /v`) antes de qualquer distribuição.[^18]

## Checklist de Assinatura

- Assinar todo executável, DLL carregada dinamicamente e instalador (MSI/EXE) com o mesmo certificado/publisher.[^30]
- Sempre incluir timestamp RFC 3161 (`/tr` + `/td SHA256`) em cada assinatura.[^19][^18]
- Verificar a assinatura antes da publicação (`signtool verify /pa /v`) como etapa final do pipeline.[^18]
- Falhar o build automaticamente se o timestamping falhar, ao invés de publicar artefato assinado sem timestamp.[^18]
- Manter URL de TSA alternativa configurada para contingência de indisponibilidade.[^18]

## Principais Riscos de Plugins CAD

- Quebra silenciosa de compatibilidade quando o host migra de runtime major (ex.: .NET Framework para .NET moderno) sem que o Autoloader bloqueie o carregamento antecipadamente.[^4][^5][^3]
- Conflito de versão de CRT/dependência nativa via SxS, onde o "vencedor" registrado no sistema pode sobrepor a versão especificada no manifest do app.[^24]
- Reputação de SmartScreen resetada a cada nova versão de binário assinado, gerando avisos recorrentes mesmo com certificado válido.[^27][^26]
- Falta de matching exato entre SDK e binário do host em produtos com "travamento de versão" (ZWCAD), causando falhas silenciosas em campo.[^7]
- Recompilação obrigatória de extensões nativas por mudança de versão de linguagem embutida no host (Ruby no SketchUp), exigindo múltiplos binários por versão.[^25][^12][^13]

## Perguntas Específicas para Desenvolvedor Sênior da TotalCAD

- Qual é o host CAD prioritário para o próximo release (ZWCAD, AutoCAD, SketchUp, ou múltiplos simultâneos) e qual é o runtime exigido por cada versão suportada?
- A distribuição atual é via instalador próprio, Microsoft Store, ou side-load manual — e isso deve mudar diante das restrições de SmartScreen sem MSIX/Store?
- Existe symbol server interno hoje, ou os crash dumps são analisados sem PDBs arquivados por build?
- Qual é a política de suporte "soft support" da TotalCAD por versão de host (análogo ao modelo Enscape de 1–2 anos)?
- Os clientes são majoritariamente corporativos com GPO/SCCM restritivo, exigindo MSI silencioso e log verboso, ou majoritariamente usuários finais tolerantes a EXE interativo?
- Há dependência de componentes nativos (C++/CRT) no plugin que exija variantes binárias por arquitetura (x86/x64/ARM64) além do .NET managed?

## Tabela de Fontes com Autoridade e Limitações

| Fonte | Autoridade | Limitações |
|---|---|---|
| Microsoft Learn — Standard Installer command-line options[^16] | Documentação oficial Microsoft | Cobre apenas MSI, não instaladores EXE customizados |
| Microsoft Learn — Time Stamping Authenticode[^1] | Documentação oficial Microsoft | Foco técnico, sem orientação de política de release |
| Microsoft Learn — SmartScreen reputation[^26] | Documentação oficial Microsoft, atualizada 2026 | Não detalha thresholds exatos de reputação (proprietários) |
| Microsoft Learn — Managed .NET Compatibility (AutoCAD)[^3] | Documentação oficial Autodesk | Específica ao AutoCAD; não cobre ZWCAD/SketchUp |
| Blog Autodesk.io — RuntimeRequirements[^5] | Blog oficial de engenharia Autodesk | Não é doc de referência formal, pode mudar sem aviso de versão |
| Fórum Autodesk (Civil 3D ideas)[^4] | Comunidade de desenvolvedores | Opiniões de parceiros, não posição oficial garantida |
| CSDN (ZWCAD ZRX)[^7] | Comunidade chinesa de desenvolvedores | Não é documentação primária da ZWSOFT; requer validação cruzada |
| ZWSOFT — Application Development Support[^8] | Página oficial ZWSOFT | Nível de detalhe baixo, sem tabela de compatibilidade explícita |
| SketchUp Ruby API Release Notes[^11] | Documentação oficial Trimble/SketchUp | Foco em Ruby puro; C extensions cobertas em repositórios GitHub separados |
| Fóruns SketchUp Developers[^12][^13] | Comunidade oficial hospedada pela Trimble | Relatos de experiência individual, não garantia de suporte |
| Chaos Docs — Enscape System Requirements[^14] | Documentação oficial Chaos/Enscape | Específica ao produto Enscape, não generalizável a outros plugins |
| Wikipedia — Side-by-side assembly[^22] | Enciclopédia colaborativa | Conteúdo tecnicamente correto mas não citável como fonte primária em decisões críticas |
| Random ASCII / CodeProject / Cantabile blog (symbol servers)[^34][^35][^31] | Blogs técnicos especializados | Boas práticas de comunidade, não normas oficiais Microsoft |
| ToDesktop blog — EV certs reputation[^27] | Blog de fornecedor de tooling | Interpretação de terceiro sobre mudança de política Microsoft, não fonte primária |

---

## References

1. [Time Stamping Authenticode Signatures - Win32 apps](https://learn.microsoft.com/en-us/windows/win32/seccrypto/time-stamping-authenticode-signatures) - Always time-stamp your Authenticode signatures. Without a time stamp, the signature becomes invalid ...

2. [Msix Direct Download -- Isv...](https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/package-and-deploy/choose-distribution-path.md) - Conceptual and overview content for developing Windows apps - MicrosoftDocs/windows-dev-docs

3. [Совместимость управляемых компонентов .NET](https://help.autodesk.com/cloudhelp/2026/RUS/AutoCAD-Customization/files/GUID-A6C680F2-DE2E-418A-A182-E4884073338A.htm) - Управляемые приложения .NET часто связаны с корпоративными и сторонними приложениями, способными рас...

4. [AutoCAD/Civil 3D should be able to load plugin with lower .net version](https://forums.autodesk.com/t5/civil-3d-ideas/autocad-civil-3d-should-be-able-to-load-plugin-with-lower-net/idi-p/14050028) - Currently as it stands, AutoCAD/Civil 3D 2026 is on .net 8.0, and 2027 most likely will be on .net 1...

5. [AutoCAD 2025: Update Your PackageContents.xml with ...](https://blog.autodesk.io/autocad-2025-update-your-packagecontentsxml-with-runtimerequirements/) - <?xml encoding=”UTF-8″>By Madhukar Moogala If you are using PlugIn-Autoloader mechanism to install c...

6. [Configuring AutoCAD's use of the .NET Framework via acad ...](https://keanw.com/2006/10/configuring_aut_1.html)

7. [C#开发ZRX插件时，如何正确配置中望CAD开发环境？ - CSDN问答](https://ask.csdn.net/questions/9336109) - CSDN问答为您找到C#开发ZRX插件时，如何正确配置中望CAD开发环境？相关问题答案，如果想了解更多关于C#开发ZRX插件时，如何正确配置中望CAD开发环境？ 青少年编程 技术问题等相关问答，请访问...

8. [ZWCAD - Application Development Support](https://www.zwsoft.com/support/zwcad-devdoc) - ZWCAD supports .NET API, for example C#/C++, .NET/VB.NET ect., extending ZWCAD functionality and ach...

9. [ZW3D - Application Development Support](https://www.zwsoft.com/support/zw3d-devdoc) - ZW3D features a flexible development platform for specialized design and drafting applications. Its ...

10. [Will my plugins be compatible with 2018?](https://forums.sketchup.com/t/will-my-plugins-be-compatible-with-2018/67170) - I’ve made plugins that I have been using without problem in SketchUp 2016. I’m thinking about upgrad...

11. [File: Release Notes — SketchUp Ruby API Documentation](https://ruby.sketchup.com/file.ReleaseNotes.html) - What's new in SketchUp 2025.0.2. Update OpenSSL to 3.4.1. The version of OpenSSL in Ruby was updated...

12. [Supporting several SU versions on Mac OSX for a plugin](https://forums.sketchup.com/t/supporting-several-su-versions-on-mac-osx-for-a-plugin/283445) - Hey SU folks, I am working on an extension which relies on some C/C++ code, compiled in .so files an...

13. [SketchUp 2025 C SDK Compatibility - Developers](https://forums.sketchup.com/t/sketchup-2025-c-sdk-compatibility/319117) - I’m in the process of upgrading my extension to support SU2025 PBR materials. I successfully updated...

14. [System Requirements - Enscape - Chaos Docs](https://docs-chaos.atlassian.net/wiki/spaces/enscape/pages/842039327/System+Requirements)

15. [Enscape4.1支持哪一些SketchUp版本？](https://www.sketchupvray.com/233253.html) - Enscape 4.1 增效工具适用于以下主机应用程序：Revit 2022、2023、2024 和 2025SketchUp 2022、2023 与 2024Rhino 7.0&n...

16. [Microsoft Standard Installer command-line options](https://learn.microsoft.com/en-us/windows/win32/msi/standard-installer-command-line-options) - See standard command-line options for the Microsoft Standard Installer Msiexec.exe, the app you use ...

17. [The Silent Install options for the Windows installer (msiexec)](https://silentinstall.org/msiexec) - The Silent Install Builder is a great tool for the quick install of a set of software on multiple de...

18. [Code Signing Timestamps: How They Work](https://my-ssl.com/learn/code-signing-timestamping) - A timestamp proves your code was signed while its certificate was valid, so signatures survive expir...

19. [Authenticode And Timestamping And sha256](https://blog.didierstevens.com/2015/11/24/authenticode-and-timestamping-and-sha256/) - I have a couple of how-to posts on digital signatures, like this code signing post. Let me revisit t...

20. [Silently Install EXE and MSI setup applications (Unattended)](https://www.advancedinstaller.com/silent-install-exe-msi-applications.html) - Find out what a silent install is and how to silently install MSI and EXE setups (the unattended way...

21. [Silent installation of an MSI package](https://stackoverflow.com/questions/8560166/silent-installation-of-an-msi-package) - I have a MSI package that I need to install if the package is not already installed. Also I need to ...

22. [Side-by-side assembly - Wikipedia](https://en.wikipedia.org/wiki/Side-by-side_assembly)

23. [windows side by side assemblies](http://rxwen.blogspot.com/2009/12/windows-side-by-side-assemblies.html) - There are times when I deployed an application on a box different from the developing box, the appli...

24. [PTSource Wiki & Store | Tips Sidebysideassemblies](https://store.ptsource.eu/tips/sidebysideassemblies) - Windows Side-by-Side Assemblies (WinSxS).

25. [How to compile Ruby c extension on Ruby 2.2.1 for sketchup plugin?](https://stackoverflow.com/questions/39052943/how-to-compile-ruby-c-extension-on-ruby-2-2-1-for-sketchup-plugin) - I have used a Ruby C Extension for SketchUp Plugin. Ruby version is 1.9.3 and SketchUp2016. But I ha...

26. [SmartScreen reputation for Windows app developers](https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/smartscreen-reputation) - EV certificates no longer bypass SmartScreen. Years ago, signing files with an Extended Validation (...

27. [EV Certs do not grant immediate reputation anymore](https://www.todesktop.com/blog/posts/windows-apps-psa-ev-certs-do-not-grant-immediate-reputation-anymore) - EV code signing certificates no longer grant instant SmartScreen reputation. · New releases of EV-si...

28. [The extended validation code signing certificate you need to avoid ...](https://news.ycombinator.com/item?id=48078194)

29. [Code Signing Time Stamping - What Is It? Why It Matters?](https://codesigningstore.com/code-signing-time-stamping-best-practices) - Best Practices: Code Sign Time Stamping · Turn on time stamping in your signing tool. · Add timestam...

30. [SmartScreen reputation issue for EV Code Signed Windows application | Microsoft Community Hub](https://techcommunity.microsoft.com/discussions/microsoftdefenderthreatintelligence/smartscreen-reputation-issue-for-ev-code-signed-windows-application/4537122/replies/4538443) - Hello Microsoft Defender Threat Intelligence Team, We are the developer of a legitimate Windows desk...

31. [Mini Dumps Vs Full Crash...](https://www.cantabilesoftware.com/blog/visual-studio-crash-dumps---a-guide-for-plugin-developers-5b2a666d94e2/) - For any software used in live performance, rock solid stability is the number one priority. This has...

32. [Debugging with Symbols - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/dxtecharts/debugging-with-symbols) - This article provides a high level overview of how to best use symbols in your debugging process.

33. [Install the Operating System Symbols and Set Up a Symbol Store](https://flylib.com/books/en/4.441.1.22/1/) - Debugging Applications for MicrosoftВ® .NET and Microsoft WindowsВ® (Pro-Developer),2003, (isbn 0735...

34. [Symbols the Microsoft Way | Random ASCII - WordPress.com](https://randomascii.wordpress.com/2013/03/09/symbols-the-microsoft-way/) - Symbol servers allow developer tools on Windows to automatically find symbols (debug information). A...

35. [Windows Symbols and Dump Analysis (Quick Steps)](https://www.codeproject.com/Tips/449993/Windows-Symbols-and-Dump-Analysis-Quick-Steps) - For those who code

36. [Use dump files in the debugger - Visual Studio (Windows)](https://learn.microsoft.com/en-us/visualstudio/debugger/using-dump-files?view=visualstudio) - Open a dump file · To set symbol loading locations, select Set symbol paths. · To start debugging, s...

