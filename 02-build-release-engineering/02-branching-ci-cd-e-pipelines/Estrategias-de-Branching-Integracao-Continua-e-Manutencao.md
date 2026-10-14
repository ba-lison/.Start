# Estratégias de Branching, Integração Contínua e Manutenção de Releases

## Resumo executivo

Não existe uma estratégia de branching universalmente "melhor" — a escolha correta depende da frequência de release desejada, do número de versões que precisam de suporte simultâneo, do tamanho da equipe e da maturidade de automação (CI/CD, feature flags, testes). O trunk-based development (TBD) é o padrão mais associado a alta performance de entrega segundo o programa de pesquisa DORA, que analisando dados de mais de 33.000 profissionais encontrou correlação estatisticamente significativa entre TBD e desempenho elite de entrega de software ao longo de uma década de pesquisa. A própria DORA define TBD operacionalmente como: três ou menos branches ativas no repositório, merge para trunk pelo menos uma vez por dia, e ausência de code freezes ou fases de integração dedicadas.[^1][^2]

Por outro lado, o GitFlow — criado por Vincent Driessen em 2010 para um mundo de releases agendadas e infrequentes — foi explicitamente desaconselhado pelo próprio autor para equipes que praticam entrega contínua, que recomendou GitHub Flow como alternativa mais simples. GitLab Flow ocupa uma posição intermediária, formalizando branches de ambiente (staging, pré-produção, produção) sem exigir toda a complexidade estrutural do GitFlow. A escolha entre esses modelos deve ser guiada pela frequência real de release: produtos com deploy contínuo (SaaS web) favorecem TBD/GitHub Flow; software distribuído com múltiplas versões em campo (embarcado, bibliotecas, produtos on-premise) frequentemente exige branches de manutenção e release branches, mesmo que isso implique maior atrito de merge.[^3][^4][^5][^6][^7][^8][^9]

Feature flags são o mecanismo que permite reconciliar TBD com funcionalidades incompletas, decompondo "deploy" de "release": o código pode ser mesclado ao trunk e implantado em produção enquanto permanece invisível a usuários até estar pronto. Estudos empíricos sobre merge conflicts mostram que aproximadamente 1 em cada 5 merges gera conflitos que exigem intervenção humana, e que código associado a conflitos de merge tem probabilidade até 26 vezes maior de conter bugs quando a resolução exige intervenção manual — um argumento quantitativo a favor de integrações menores e mais frequentes.[^10][^11][^12][^13]

## Diagramas de cada estratégia

### Trunk-based development (TBD)

```
main  ──●──●──●──●──●──●──●──●──●──→ (sempre deployável)
         \   \ /    \ /   \
          fb1 fb2    fb3   fb4   (branches de 1-2 dias, ou commit direto)
```
Commits diretos ou branches de vida curtíssima (menos de 24-48h) convergem constantemente para o trunk.[^14][^15]

### GitHub Flow

```
main ──●────────────●────────●──────●──→ (deploy a cada merge)
         \          / \      /
          feature-A    feature-B
```
Uma única branch permanente (main); features em branches, PR, revisão, merge, deploy imediato.[^16][^3]

### GitLab Flow (com branches de ambiente)

```
main ──●──●──●──●──●──●──●──→
                  |     |
staging ──────────●─────●──→
                        |
production ─────────────●──→
```
Commits fluem apenas "downstream" (main → staging → production); cada ambiente tem sua branch, garantindo que tudo seja testado em sequência antes de produção.[^9][^17]

### GitFlow

```
main       ──●────────────────●──────────●──→ (releases taggeadas)
              \                /\        /
release/1.0    ● ── ● ── ● ──●  \      /
              /                  \    /
develop    ──●──●──●──●──●──●──●───●──●──→
              \    /        \
feature/x      ●──●          hotfix/y ──●──● (parte de main)
```
Cinco tipos de branch (main, develop, feature/*, release/*, hotfix/*) com regras rígidas de fluxo entre elas.[^6][^16]

### Stacked changes (stacked diffs)

```
main ──●
         \
          A(base) ──●
                       \
                        B(dependente de A) ──●
                                                \
                                                 C(dependente de B) ──●
```
Cada mudança pequena é revisada isoladamente e faz rebase automático quando a base é alterada, permitindo revisões granulares sem esperar a feature completa.[^18][^19]

## Matriz comparativa

| Dimensão | Trunk-Based | GitHub Flow | GitLab Flow | GitFlow | Feature branches longas |
|---|---|---|---|---|---|
| Branch de trabalho diário | Trunk ou branches <1-2 dias[^15] | Feature branch curta[^3] | Feature branch curta + branches de ambiente[^3] | feature/* a partir de develop[^6] | Branch por feature, semanas[^6] |
| Integração contínua real | Sim, nativa[^14][^20] | Sim, alta[^3] | Sim, com gates por ambiente[^4] | Fraca — isolamento é o objetivo[^6] | Não — CI só valida ao final[^6] |
| Builds oficiais | A partir do trunk, a qualquer commit[^2] | A partir de main a cada merge[^3] | A partir de tags/production[^21] | A partir de release/* e main[^8] | Ao mesclar a branch, tarde | 
| Releases programadas | Possível via tags e flags[^22] | Requer tags adicionais[^3] | Suporte nativo (release/environment branches)[^3] | Suporte nativo (release/*)[^6] | Manual, acoplado ao merge |
| Releases sob demanda | Ideal — trunk sempre deployável[^15][^2] | Bom | Bom, com staging como gate | Fraco — processo pesado | Fraco |
| Funcionalidades incompletas | Feature flags obrigatórios[^10][sic][^12] | Feature flags recomendados | Feature flags ou branch de ambiente | Isoladas em feature branch até "pronta" | Isoladas, risco de "merge hell" |
| Hotfixes | Fix curto direto no trunk + deploy imediato | hotfix branch curta a partir de main | hotfix/* mesclado a production e propagado a montante[^9] | hotfix/* mesclado a main e develop[^6] | Difícil, exige branch dedicada |
| Backports/cherry-pick | Raramente necessário (uma única linha) | Raro | Ocasional entre ambientes | Comum entre release/support branches[^7] | Comum e propenso a erro |
| Múltiplas versões em manutenção | Difícil — não é o propósito | Difícil | Possível com branches de release | Suportado por design[^6] | Suportado, mas caótico |
| Risco de conflitos | Baixo, conflitos pequenos e frequentes[^23][^24] | Baixo-médio | Médio | Alto — merges grandes e tardios[^6] | Alto — "merge hell"[^24][^25] |
| Complexidade organizacional exigida | Alta disciplina, CI robusto, testes automatizados[^2] | Baixa-média | Média-alta | Média (mas processo pesado) | Baixa exigência, alto custo oculto |
| Frequência de release viável | Múltiplas por dia | Diária/contínua | Diária a semanal | Semanas a meses | Meses |

## Árvore de decisão

```
O produto pode ser deployado continuamente (sem obrigação
regulatória/contratual de versões fixas em campo)?
│
├── SIM → Existe exigência de gates de ambiente (staging, aprovação, compliance)?
│         │
│         ├── NÃO → Trunk-Based Development ou GitHub Flow
│         │         (branches curtas + feature flags)
│         │
│         └── SIM → GitLab Flow com branches de ambiente
│                   (main → staging → production)
│
└── NÃO → É necessário manter múltiplas versões publicadas
          simultaneamente (ex.: v1.x e v2.x em produção)?
          │
          ├── SIM → GitFlow ou GitLab Flow com release branches
          │         + branches de manutenção/support por versão
          │
          └── NÃO, mas há ciclos de release formais e não-diários
                    → GitLab Flow com release branches (sem 'develop' 
                    permanente, cortando release/* de main)
```

## Como cada abordagem trata os aspectos operacionais

**Pull Requests e revisão.** Em TBD e GitHub Flow, PRs são pequenos e de vida curta, o que estudos apontam reduzir a taxa de conflitos de merge em comparação a merges sem PR — um estudo de larga escala com 66 sistemas reais encontrou 5,02% de cenários de merge com conflito, com uso de PRs associado a menos conflitos. Em GitFlow, o PR típico é a mesclagem de uma feature branch inteira, potencialmente após semanas de divergência.[^26][^27]

**Integração contínua e builds oficiais.** TBD é considerado pré-requisito para CI verdadeira: a documentação da Atlassian afirma que se builds e testes são automatizados mas os desenvolvedores trabalham em branches isoladas e longas, a integração contínua "não está atingindo seu potencial". GitLab documenta que a divisão main/develop do GitFlow torna a entrega e integração contínuas mais difíceis, e que a criação de branches como hotfix e release pode tornar a integração complexa.[^4][^14]

**Feature flags e funcionalidades incompletas.** Em TBD, feature flags são o mecanismo padrão para impedir que trabalho incompleto bloqueie releases: o código é mesclado e implantado, mas permanece desligado em produção até estar pronto — decompondo deploy de release. O risco documentado é acúmulo de flags obsoletas: o tempo de vida pretendido de uma flag de TBD é de dias, mas na prática se estende a meses por falta de mecanismo automático de limpeza, gerando dívida técnica equivalente ou maior que branches longas.[^11][^28][^29][^12][^10]

**Hotfixes e como evitar que desapareçam do trunk.** Em GitFlow, o hotfix nasce de main e deve ser mesclado de volta tanto em main quanto em develop, para não "desaparecer" da branch principal de desenvolvimento. Em TBD, esse risco praticamente não existe porque só há uma linha principal: o fix entra no trunk e já está presente para todo trabalho futuro. O ponto crítico organizacional é garantir que toda correção aplicada em uma release/support branch seja sistematicamente propagada de volta (merge forward) para main — a ausência dessa disciplina é a causa mais comum de "hotfix perdido".[^8][^6]

**Backports e cherry-pick.** O cherry-pick é a ferramenta recomendada quando um defeito precisa ser corrigido em duas versões simultaneamente: corrige-se em main, testa-se, e usa-se `git cherry-pick <sha>` para portar o commit para a branch de suporte da versão antiga, resolvendo conflitos manualmente e retestando antes de re-tag. Fontes recomendam usar cherry-pick com moderação, pois seu uso frequente indica um problema de fluxo — idealmente as correções fluem por merge "para frente" (release → main) e não o contrário.[^7][^8]

**Múltiplas versões em manutenção.** Branches de manutenção/suporte (`support/1.x`, `*-stable`) são o padrão para produtos com várias versões vivas em campo; cada correção crítica é cherry-picked/testada e retaggeada por versão. TBD por si só não resolve esse cenário — ele assume, em geral, uma única versão em produção.[^30][^7]

**Conflitos e sincronização entre branches.** Quanto mais uma branch diverge do trunk, maior a chance e a severidade dos conflitos: um estudo com quase 3.000 repositórios encontrou que operações de refatoração estão envolvidas em 22% dos conflitos de merge, e outro estudo mostrou que aproximadamente 1 em cada 5 merges gera conflitos, com código conflitante 2 vezes mais propenso a bugs (chegando a 26x quando exige resolução manual). Rebases também não estão imunes: conflitos textuais surgem em 24,3%-26,2% dos rebases estudados.[^31][^13][^32]

**Tags e versionamento.** Em todos os modelos, tags imutáveis marcam pontos de release e seguem versionamento semântico (MAJOR.MINOR.PATCH). GitLab Flow recomenda que tags sejam criadas por desenvolvedores (não pelo CI automaticamente) e que releases sejam baseadas em tags — uma release por tag.[^21][^7][^8]

**Pipelines e branches protegidas.** Branches protegidas (main, release/*, *-stable) controlam quem pode fazer merge/push direto, previnem exclusão acidental e determinam quais pipelines têm acesso a variáveis e runners protegidos, sendo a base do controle de segurança em GitLab CI/CD. GitLab também oferece ambientes protegidos e "deployment freezes" como camadas adicionais de controle sobre quem pode implantar e quando.[^33][^34][^30]

**Rollback.** Em TBD com feature flags, o "rollback" mais rápido é apenas desligar a flag, sem necessidade de reverter deploy. Em modelos com branches de release/tags, o rollback típico é fazer deploy da tag/versão anterior estável.[^10][^11]

## Exemplo de desenvolvimento normal (Trunk-Based + feature flag)

```bash
git checkout main
git pull
git checkout -b feature/novo-checkout      # branch de 1-2 dias
# adicionar feature flag ANTES de implementar a lógica
# código roda em produção, mas fica "apagado" atrás da flag
git add . && git commit -m "novo checkout atrás de flag (off por padrão)"
git push origin feature/novo-checkout
# abrir PR pequeno, revisão rápida, merge no mesmo dia
git checkout main && git merge feature/novo-checkout
# deploy automático via CI/CD; flag continua off em produção
```
Esse fluxo é descrito consistentemente pela literatura de TBD com feature flags como o padrão que permite "mesclar diariamente, implantar diariamente, sem risco".[^12][^11]

## Exemplo de release (GitLab Flow com release branch)

```bash
# quando "main" acumula funcionalidades suficientes para uma release
git checkout main
git checkout -b release/2.5
# code freeze: só correções de bug e ajustes de documentação entram aqui
git checkout -b fix/ajuste-import release/2.5
# ... corrige e merge de volta em release/2.5
git checkout release/2.5
git tag -a v2.5.0 -m "Release v2.5.0"
git push origin release/2.5 --tags
# após validação em staging/produção, mesclar de volta em main
git checkout main
git merge release/2.5
```
Esse padrão de "code freeze → correções pontuais → merge de volta em main" é descrito como prática recomendada para reduzir conflitos futuros.[^35][^8]

## Exemplo de hotfix (correção emergencial em produção)

```bash
git checkout main
git checkout -b hotfix/falha-pagamento
# corrigir o bug crítico
git commit -am "corrige falha crítica no gateway de pagamento"
git push origin hotfix/falha-pagamento
# PR revisado com urgência, merge direto em main
git checkout main
git merge hotfix/falha-pagamento
git tag -a v2.5.1 -m "Hotfix: falha de pagamento"
git push origin main --tags
# se houver develop (GitFlow) ou release/support ativos, propagar também:
git checkout develop && git merge hotfix/falha-pagamento
```
No GitFlow, a regra explícita é que o hotfix deve ser mesclado tanto em main quanto em develop para não se perder; em TBD, essa etapa extra não existe porque há apenas uma linha principal.[^6]

## Exemplo de backport (mesmo defeito em duas versões suportadas)

```bash
# fix já mesclado e testado em main
git log --oneline -1   # sha do commit de correção: abc1234

# aplicar na branch de suporte da versão antiga ainda em manutenção
git checkout support/1.x
git cherry-pick abc1234
# resolver conflitos manuais se a base de código antiga diferir
git add .
git cherry-pick --continue
npm test   # ou suíte equivalente
git tag -a v1.5.3 -m "Backport: correção de segurança de v2.0.1"
git push origin support/1.x --tags
```
Esse padrão de cherry-pick + re-teste + retag é o fluxo documentado para portar correções entre versões de manutenção paralelas.[^7]

## Modelo de política de branches (template)

```markdown
# Política de Branches — <Projeto>

## Branch permanente
- `main`: sempre deployável; protegida; merge somente via PR aprovado + CI verde.

## Branches de trabalho
- `feature/<ticket>`: vida máxima de 2 dias; deve conter feature flag se incompleta.
- Proibido: branches sem PR associado; commits diretos em `main` sem revisão automatizada.

## Releases
- Tags semânticas (`vMAJOR.MINOR.PATCH`) criadas manualmente por responsável de release.
- Release branch (`release/x.y`) somente quando: (a) o produto exige testes manuais/regulatórios
  prolongados antes do lançamento, ou (b) múltiplas versões precisam coexistir em campo.
- Code freeze em release branch: apenas correções de bug, sem novas features.

## Hotfixes
- Branch curta a partir de `main` (ou da tag em produção).
- Deploy imediato após merge + tag de patch version.
- Obrigatório: propagar para `develop`/`release` ativos, se existirem.

## Manutenção de múltiplas versões
- Branches `support/<versão>` para cada linha ainda sob contrato de suporte.
- Correções via cherry-pick documentado (issue linkada ao commit original).
- Máximo de N versões simultâneas em suporte (definir N conforme SLA).

## Feature flags
- Toda feature incompleta mesclada ao trunk deve estar sob flag, off por padrão.
- Toda flag tem: owner, data de expiração, ticket de limpeza criado na mesma PR.

## Branches protegidas
- `main`, `release/*`, `support/*`: push direto bloqueado; force-push bloqueado; aprovação obrigatória.
```

## Antipadrões

- **Feature branches de vida longa sem integração intermediária** — geram "merge hell", divergência acumulada e feedback tardio, além de dificultarem a identificação de juniores com dificuldades.[^25][^6]
- **GitFlow aplicado a produtos de deploy contínuo** — cria burocracia desnecessária (develop + release + hotfix) quando não há múltiplas versões em campo nem releases agendadas; o próprio criador recomenda contra isso.[^5]
- **Cherry-pick como rotina, não excepção** — uso frequente de cherry-pick entre branches indica que mudanças não fluem pelo processo normal de merge, gerando históricos divergentes e duplicação de commits.[^8][^7]
- **Feature flags nunca removidas** — acumulam-se além do tempo de vida pretendido, tornando-se dívida técnica pior que branches longas por gerarem caminhos de código não testados.[^29][^11]
- **Merge de main para release em ambos os sentidos sem regra clara** — misturar merges bidirecionais entre release e main aumenta risco de reintroduzir código não aprovado para a release.[^8]
- **Ausência de branches protegidas em main/release** — permite push direto e force-push, comprometendo rastreabilidade e histórico de commits.[^36][^33]
- **Code freeze prolongado como substituto de release branch bem definida** — a DORA identifica a ausência de code freezes como característica de equipes de alta performance; freezes longos são sinal de processo de release inadequado.[^2]

## Riscos de branches longas

Branches de vida longa aumentam a probabilidade e a severidade dos conflitos de merge porque o código diverge continuamente do trunk; a métrica da DORA associa idade média de branch acima de 5-14 dias a desempenho de entrega inferior, contra menos de 1 dia em equipes de TBD. Estudos empíricos mostram que aproximadamente 1 em 5 merges gera conflito, e que conflitos que exigem intervenção manual estão associados a até 26 vezes mais bugs, tornando o "merge day" um momento de risco concentrado quando branches são longas. Branches longas também atrasam feedback: problemas de integração só aparecem semanas após o início do trabalho, tornando-os mais caros de corrigir e dificultando estimativas de prazo.[^13][^25][^1][^6]

## Requisitos organizacionais para trunk-based development

TBD exige: (1) suíte de testes automatizados robusta e rápida, cobrindo build, testes unitários e de integração a cada commit; (2) disciplina de commits pequenos e frequentes — a própria comunidade define branches de TBD como devendo durar no máximo alguns dias; (3) uso sistemático de feature flags para desacoplar deploy de release; (4) cultura de propriedade coletiva do código, já que todos trabalham essencialmente na mesma linha; e (5) pipelines de CI/CD maduros capazes de validar e implantar a cada merge. A pesquisa DORA reforça que TBD funciona melhor combinado com ausência de fases de integração dedicadas e no máximo três branches ativas simultaneamente.[^37][^24][^14][^12][^2][^10]

## Relação entre branching e frequência de release

A frequência de release e a estratégia de branching estão diretamente acopladas: dados da DORA associam TBD a até 3 vezes mais frequência de deployment e 2 vezes menor tempo médio de restauração de serviço (MTTR) comparado a equipes com feature branches longas. Modelos com release branches (GitFlow, GitLab Flow com releases) tendem a ciclos de semanas a meses, adequados quando o software é distribuído (não SaaS) ou exige testes de certificação prolongados. Em geral, quanto mais branches ativas e mais longas, menor a cadência sustentável de release, porque cada release exige sincronização manual acumulada entre linhas divergentes.[^16][^1][^2][^6][^8]

## Como identificar a estratégia usada em um repositório existente

Sinais observáveis no histórico e na configuração do repositório permitem inferir a estratégia adotada:

- Presença de branch `develop` permanente além de `main` sugere GitFlow.[^6]
- Branches `release/*` e `hotfix/*` recorrentes e de vida curta (dias) reforçam GitFlow.[^6]
- Branches de ambiente nomeadas `staging`, `pre-prod`, `production` com fluxo unidirecional indicam GitLab Flow.[^17][^9]
- Idade média das branches no momento do merge (via `git log` e datas de criação) abaixo de 1-2 dias é indicativo forte de TBD.[^1]
- Ausência de qualquer branch de longa duração além de `main`, com PRs pequenos e frequentes, sugere GitHub Flow ou TBD puro.[^3]
- Presença de flags de feature no código-fonte (bibliotecas como Unleash, LaunchDarkly, Togglz) reforça hipótese de TBD com feature flags.[^38][^10]
- Tags semânticas únicas por commit de release, sem branches de release dedicadas, sugere GitHub Flow ou TBD com tagging direto.[^21]
- Configuração de branches protegidas e regras de CI condicionadas a `CI_COMMIT_BRANCH == default` vs. tags separadas ajuda a mapear o fluxo de deploy real.[^39][^33]

## Checklist para avaliar a estratégia de um projeto

- [ ] Quantas branches de longa duração (>1 semana) existem hoje no repositório?
- [ ] Qual a idade média das branches de feature no momento do merge?
- [ ] Existe suíte de testes automatizados suficiente para validar cada commit no trunk?
- [ ] Funcionalidades incompletas são geridas por feature flags ou por branches isoladas?
- [ ] Há processo documentado e testado para hotfix chegar a todas as versões relevantes?
- [ ] Cherry-picks são exceção documentada ou prática rotineira sem rastreabilidade?
- [ ] Quantas versões estão simultaneamente em manutenção/suporte?
- [ ] Tags seguem versionamento semântico e são imutáveis?
- [ ] Branches protegidas (main, release, support) bloqueiam push direto e force-push?
- [ ] Pipelines de CI cobrem builds oficiais, testes e deploy de forma automatizada por branch/tag?
- [ ] Existe processo de rollback rápido (flag off, redeploy de tag anterior)?
- [ ] Feature flags têm dono, data de expiração e ticket de limpeza?

## Perguntas para fazer ao tech lead

- Qual é a cadência real de release hoje, e ela é limitada pela estratégia de branching ou por outros fatores (QA manual, aprovações)?
- Quantas versões do produto precisam receber correções simultaneamente, e por quanto tempo cada versão fica em suporte?
- Como uma funcionalidade incompleta é impedida de bloquear uma release programada?
- Qual o processo formal para garantir que um hotfix aplicado em produção não se perca ao longo do tempo?
- Existe uma política escrita de branches, ou o fluxo é tribal knowledge?
- Quantas vezes por semana ocorrem conflitos de merge que exigem intervenção manual significativa?
- Feature flags são monitoradas e removidas após rollout completo, ou se acumulam indefinidamente?
- Branches protegidas e regras de CI/CD estão alinhadas com quem realmente deveria poder fazer deploy em produção?

## Tabela de fontes e nível de evidência

| Fonte | Tipo | Nível de evidência |
|---|---|---|
| DORA (dora.dev) — capacidades e relatórios anuais[^2][^1] | Programa de pesquisa contínuo, >33.000 respondentes | Alto — dados survey longitudinais, mas correlacional, não causal |
| Estudo sobre ciclo de vida de conflitos de merge (Univ. Saarland)[^26][^27] | Estudo empírico, 66 repositórios reais, 78.740 cenários de merge | Alto — amostra grande de projetos open-source, mas limitado a contexto Java/específico |
| Estudo sobre conflitos de merge e qualidade de código (EMSE)[^13] | Estudo empírico, 36.122 merge commits | Alto — dados quantitativos robustos, generalização limitada ao conjunto de projetos estudado |
| Estudo sobre rebases e conflitos textuais[^31] | Estudo empírico | Médio-alto — foco específico em rebase, amostra não detalhada nesta pesquisa |
| Estudo sobre refatoração e conflitos (SANER)[^32] | Estudo empírico, ~3.000 repositórios Java | Alto — amostra grande, mas restrita à linguagem Java |
| Documentação oficial GitLab (protected branches, CI/CD, GitLab Flow)[^33][^34][^9] | Documentação de fornecedor | Alto para "como funciona"; fonte interessada quanto a recomendações de adoção |
| Documentação Atlassian sobre TBD[^14] | Documentação de fornecedor | Médio-alto — conteúdo técnico correto, mas com viés comercial (venda de ferramentas) |
| Post original de Vincent Driessen sobre GitFlow e sua nota de retratação[^5][^6] | Fonte primária do autor do modelo | Muito alto — testemunho direto do criador |
| Blogs técnicos (Aviator, Harness, Boldare, Graphite, etc.)[^40][^41][^18] | Conteúdo de marketing técnico | Médio — útil para consenso de práticas, mas sem dados primários próprios |
| AWS Prescriptive Guidance sobre trunk strategy[^23] | Documentação de fornecedor cloud | Alto para práticas recomendadas, viés comercial moderado |

---

## References

1. [Trunk-Based Development and DORA Metrics: The Research-Backed Connection](https://koalr.com/blog/trunk-based-development-dora) - Why TBD teams have 3× higher deployment frequency, 2× lower MTTR, and 1.8× lower change failure rate...

2. [Capabilities: Trunk-based development](https://dora.dev/capabilities/trunk-based-development/) - DORA is a long running research program that seeks to understand the capabilities that drive softwar...

3. [GitHub Flow vs. GitLab Flow - fueint.com](https://fueint.com/blog/github-gitlab) - Compare GitHub Flow and GitLab Flow to choose the best branching strategy for your team, from simple...

4. [gitlab-flow/README.md at master · jadsonjs/gitlab-flow](https://github.com/jadsonjs/gitlab-flow/blob/master/README.md) - Shows the Gitlab flow workflow. Contribute to jadsonjs/gitlab-flow development by creating an accoun...

5. [uRadical - GitFlow Is Pantomime](https://uradical.io/latest-news/gitflow-is-pantomime) - GitFlow is a branching model designed for a world that no longer exists. Here is a precise account o...

6. [Trunk-Based Development vs Git Flow](https://dev.to/acestus/trunk-based-development-vs-git-flow-2kb5) - What Is Git Flow? Git Flow was introduced by Vincent Driessen in 2010 as a branching model for git. ...

7. [Git Release Branching and Hotfixes: Managing Versions in ...](https://geekworkbench.com/blog/technical/git-release-branching-hotfixes) - Master release branching and hotfix strategies in Git. Learn version branches, emergency fixes, back...

8. [Shipping software: Release Flow & Branching Strategies - Medium](https://hector-reyesaleman.medium.com/shipping-software-release-flow-branching-strategies-cf09cb0d369a) - Shipping software is more than just merging code into main or master. A release is a quite “Magical”...

9. [Introduction to GitLab Flow](https://docs.gitlab.co.jp/ee/topics/gitlab_flow.html) - GitLabドキュメント（Community Edition, Enterprise Edition, Omnibusパッケージ, GitLab Runner）

10. [Implement trunk-based development using feature flags](https://docs.getunleash.io/guides/trunk-based-development) - Explore different trunk-based development strategies · Create feature flags for incomplete features ...

11. [Trunk-Based Development: Why Feature Flags Make It Actually Work](https://www.featureflow.com/blog/trunk-based-development-feature-flags) - Committing to main daily sounds scary. Feature flags are what make it safe — a practitioner's guide.

12. [Feature Flags in Trunk-Based Development](https://www.harness.io/blog/trunk-based-development) - Explore how feature flags enhance trunk-based development, reducing merge conflicts and boosting dep...

13. [An empirical investigation into merge conflicts and their effect on software quality](https://epiclab.github.io/publications/emse20-brindescu.pdf)

14. [Trunk-based Development](https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development) - Trunk-based development is a version control management practice where developers merge small, frequ...

15. [Trunk Based Development: Introduction](https://trunkbaseddevelopment.com/) - A portal on this practice

16. [Choosing the Right Git Workflow for Your Project: GitFlow ...](https://medium.com/@vava_silva/choosing-the-right-git-workflow-for-your-project-gitflow-github-flow-and-gitlab-flow-6a12cb438ccb) - Git is a powerful version control system widely used by developers to manage and track changes in th...

17. [GitLab Workflow | Tower Help (Windows)](https://www.git-tower.com/help/guides/workflows/gitlab-workflow/windows) - Learn about Tower's GitLab Workflow template, featuring environment-based branching with staging and...

18. [How do stacked diffs work - Graphite](https://graphite.com/guides/how-do-stacked-diffs-work) - Constructing a stacked diff. To create a stacked diff, developers start with a base branch and make ...

19. [Stacked Diffs - by Neo Kim - The System Design Newsletter](https://newsletter.systemdesign.one/p/stacked-diffs) - #95: A Code Review Workflow That Will Make You a 10X Software Engineer

20. [What Is TBD? Trunk-Based Development & Its Role in CI/CD](https://www.splunk.com/en_us/blog/learn/tbd-trunk-based-development.html) - Trunk-based development is a popular way to control source code when developing apps. Learn how TBD ...

21. [GitLab Flow Pipelines avancés](https://www.lirmm.fr/~fmichel/ens/cours/GitLab_Flow.pdf)

22. [Trunk-based development - Optimizely](https://www.optimizely.com/optimization-glossary/trunk-based-development) - Trunk-based development (TBD) is a software development strategy where engineers merge smaller chang...

23. [Advantages and disadvantages of the Trunk strategy](https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-git-branch-approach/advantages-and-disadvantages-of-the-trunk-strategy.html) - Trunk-based development offers several advantages that can improve the development process, streamli...

24. [Trunk-based Development Definition, Benefits & Practices](https://www.boldare.com/blog/what-is-trunk-based-development/) - One of the main trunk-based development benefits is the reduced complexity of merging different bran...

25. [What is Trunk Based Development? A Different Approach to the ...](https://www.freecodecamp.org/news/what-is-trunk-based-development/) - The Software Development Lifecycle (SDLC) is different at every company. The version control system ...

26. [Investigating the Merge Conflict Life-Cycle Taking the ...](https://www.se.cs.uni-saarland.de/theses/GustavoDoValeDiss.pdf)

27. [Empirical Software Engineering manuscript No.](https://www.se.cs.uni-saarland.de/publications/docs/VCA23.pdf)

28. [How do I position feature flags when moving from Gitflow to Trunk-Based? — Muhammet Şafak](https://www.muhammetsafak.com.tr/en/just-ask/gitflow-vs-trunk-based-development-and-feature-flags/) - Gitflow's long-lived branches drift; that's why merges hurt. Trunk-Based merges small changes to mai...

29. [Feature Flags and Trunk-Based Development: The Cleanup Problem Nobody Talks About](https://flagshark.com/blog/trunk-based-development-feature-flag-cleanup/) - Trunk-based development encourages more feature flags — but who cleans them up? The missing piece in...

30. [GitLab CI : branches protégées, environnements et ...](https://blog.stephane-robert.info/docs/pipeline-cicd/gitlab/securite/branches-protegees/) - Branches protégées, tags protégés, environnements de déploiement, approval gates, deployment freezes...

31. [Understanding Merge Conflicts and Resolutions in](https://lqchen.github.io/ISSRE20_Merge.pdf)

32. [Are Refactorings to Blame? An Empirical Study of ...](https://users.encs.concordia.ca/~nikolaos/publications/SANER_2019.pdf)

33. [Protected branches](https://docs.gitlab.com/user/project/repository/branches/protected/) - The permission to merge or push to protected branches defines whether or not a user can run CI/CD pi...

34. [CI/CD pipelines](https://docs.gitlab.com/ci/pipelines/) - Configuration, automation, stages, schedules, and efficiency.

35. [Branching strategies](https://docs.gitlab.com/user/project/repository/branches/strategies/) - Expand your Git branching strategy when you outgrow the basic approach.

36. [Protected branches · Project · User · Aide · GitLab](https://www-apps.univ-lehavre.fr/forge/help/user/project/protected_branches.md) - Forge de L'université du Havre

37. [Explaining Trunk Based Development](https://www.travis-ci.com/blog/explaining-trunk-based-development/) - Trunk-based development is one of the most widely used branching methodologies. It helps teams colla...

38. [Feature Flags in Java: The Missing Piece That Makes Trunk-Based Development Work](https://www.javacodegeeks.com/2026/07/feature-flags-in-java-the-missing-piece-that-makes-trunk-based-development-work.html) - How to implement feature flags in Spring Boot 4 with Togglz and Unleash, covering release, ops, and ...

39. [Using child pipelines to continuously deploy to five ...](https://about.gitlab.com/blog/using-child-pipelines-to-continuously-deploy-to-five-environments/) - Learn how to manage continuous deployment to multiple environments, including temporary, on-the-fly ...

40. [What is Trunk-Based Development? | Harness Glossary](https://www.harness.io/harness-devops-academy/trunk-based-development) - Trunk-Based Development is a software development model prioritizing continuous integration by commi...

41. [What is Trunk-Based Development: Advantages ... - Aviator](https://www.aviator.co/blog/trunk-based-development/) - Trunk-based development is a popular VCM practice that helps reduce the challenges associated with b...

