# Backlog de Pesquisas da V2

> **Objetivo:** listar o que ainda precisa de fundamentação antes de transformar o SOP V2 em regra definitiva.
> **Status:** primeira rodada concluída. Sínteses salvas em `docs.v2/05-pesquisas-v2/`; questões que dependem de escolha humana permanecem abertas.

---

## 1. Diagnóstico da biblioteca atual

A biblioteca já possui boa cobertura conceitual sobre:

- builds determinísticas e reproduzíveis;
- branching e integração contínua;
- pipelines, ambientes e promoção de artefatos;
- versionamento e releases;
- deploy progressivo e feature flags;
- rollback e resposta a incidentes;
- segurança da cadeia de suprimentos;
- migrações de banco de dados;
- métricas DORA e governança;
- PRD, SDD, documentação e memória de agentes.

Esses assuntos não estão completamente ausentes. A lacuna é a ligação entre teoria e operação: faltam estudos específicos que determinem como um agente de código deve criar, manter, validar e transferir esses registros entre sessões sem gerar burocracia ou histórico inconsistente.

---

## 2. Pesquisas prioritárias — núcleo do processo e histórico

### P01 — Memória operacional mínima de um projeto de software

**Perguntas:**

- Qual é a separação correta entre estado atual, histórico, decisões, backlog e evidências?
- O que deve ficar em `PROJECT_CONTEXT`, `DECISIONS`, ADRs, `CHANGELOG`, Tasks, handoffs e Releases?
- Qual informação deve ser sobrescrita e qual deve ser somente anexada?
- Como evitar duplicação e contradição entre esses arquivos?
- Como arquivar histórico sem perder rastreabilidade?

**Fontes desejadas:** OpenAI/Codex, Anthropic/Claude Code, Google/DeepMind, Microsoft Research, literatura de knowledge management, arc42 e Diátaxis.

**Entregável:** especificação da memória operacional canônica e templates mínimos.

### P02 — Continuidade segura entre sessões de agentes de código

**Perguntas:**

- Qual contexto precisa ser persistido antes de encerrar uma sessão?
- Como um novo agente verifica o estado em vez de confiar cegamente no handoff?
- Quais dados tornam um handoff reproduzível e auditável?
- Quando continuar uma sessão e quando iniciar outra?
- Há evidência para `1 Task = 1 chat`, ou isso deve ser uma heurística adaptativa?
- Como tratar compactação de contexto, sessão interrompida e handoff incompleto?

**Fontes desejadas:** documentação oficial OpenAI/Codex e Anthropic/Claude Code; artigos de agentes de longa duração; SWE-agent, OpenHands e pesquisas acadêmicas sobre long-horizon agents.

**Entregável:** protocolo de início, encerramento, recuperação e transferência de sessão.

### P03 — Planejamento hierárquico para agentes: Projeto → Build → Sprint → Task

**Perguntas:**

- Build e Sprint devem ser conceitos separados neste sistema?
- Como decompor uma entrega sem criar Tasks pequenas demais ou grandes demais?
- Como representar dependências, critérios de pronto e escopo negativo?
- Como lidar com descoberta de trabalho, bugs e mudança de escopo?
- Como identificar automaticamente que uma Task não cabe em uma sessão?

**Fontes desejadas:** Scrum.org, Kanban University, Team Topologies, Shape Up/Basecamp, literatura de project planning e estudos de agentes de software.

**Entregável:** taxonomia simples, regras de decomposição e template `BUILDxxx.md`.

### P04 — Registro de mudanças e decisões sem duplicação

**Perguntas:**

- O que pertence a commit, handoff, changelog, release notes, ADR e contexto?
- Como relacionar Task → commit → decisão → Build → Release?
- Quando uma decisão merece ADR?
- Como substituir uma decisão antiga sem apagar sua história?
- Como gerar changelog automaticamente sem perder significado humano?

**Fontes desejadas:** Architecture Decision Records de Michael Nygard; Thoughtworks; Keep a Changelog; Conventional Commits; documentação Git e GitHub/GitLab.

**Entregável:** modelo de rastreabilidade e política de atualização dos arquivos.

### P05 — Estado, lock e recuperação de execução interrompida

**Perguntas:**

- `_ACTIVE_TASK.lock` é necessário quando Git já existe?
- O lock deve conter estado, tentativas e arquivos-alvo ou somente apontar para a Task?
- Cópia em `.backup/` é segura ou cria uma segunda fonte de verdade?
- Como recuperar uma sessão interrompida sem sobrescrever trabalho válido?
- Como lidar com dois agentes ou worktrees concorrentes?

**Fontes desejadas:** documentação Git, worktrees, sistemas transacionais, idempotência, durable execution e literatura de workflow engines.

**Entregável:** política de estado e recuperação com alternativas para projeto com e sem Git.

---

## 3. Pesquisas prioritárias — qualidade e verificação

### P06 — Estratégia de testes proporcional ao risco para agentes

**Perguntas:**

- Quando TDD é adequado e quando não é?
- Quais mudanças exigem teste unitário, integração, contrato, E2E, smoke ou somente inspeção?
- Como impedir testes placebo e alteração oportunista dos testes?
- Quem valida os testes criados pelo mesmo agente que implementou?
- Como preservar evidências sem copiar logs enormes para Markdown?

**Fontes desejadas:** Kent Beck, Martin Fowler, Google Testing Blog, Microsoft Engineering Fundamentals, OpenAI e Anthropic sobre verificação de coding agents, estudos SWE-bench.

**Entregável:** matriz mudança × risco × verificação e template de evidências.

### P07 — Definition of Ready, Definition of Done e gates

**Perguntas:**

- Quais condições mínimas permitem iniciar uma Task?
- Quando uma Task está concluída?
- Quando uma Sprint, Build e Release estão concluídas?
- Como evitar checklists universais excessivos?
- Como declarar exceções e risco aceito?

**Fontes desejadas:** Scrum Guide, Scrum.org, DORA, Google SRE e literatura de continuous delivery.

**Entregável:** gates mínimos por nível, com perfis simples e críticos.

### P08 — Circuit breaker e orçamento de tentativas

**Perguntas:**

- Existe fundamento para exatamente três falhas?
- O contador deve ser por Task, comando, hipótese ou classe de erro?
- Quais falhas devem interromper imediatamente?
- Quando escalar ao humano e quais evidências fornecer?
- Rollback automático é sempre seguro?

**Fontes desejadas:** sistemas distribuídos, Site Reliability Engineering, durable workflows, pesquisas sobre agentes autônomos e human-in-the-loop.

**Entregável:** política de tentativas, parada, escalonamento e recuperação baseada em risco.

### P09 — Revisão independente e uso de subagentes

**Perguntas:**

- Quando um subagente melhora qualidade e quando apenas aumenta custo/contexto?
- É útil separar autoria de implementação, teste e revisão?
- Como evitar alterações concorrentes conflitantes?
- Que evidência o subagente precisa devolver?
- O limite fixo de profundidade e quantidade possui fundamento?

**Fontes desejadas:** OpenAI, Anthropic, Google DeepMind, Microsoft Research, Meta e artigos acadêmicos de multi-agent software engineering.

**Entregável:** política simples de delegação, revisão e integração de resultados.

---

## 4. Pesquisas prioritárias — Build, entrega e operação

### P10 — Como instanciar `DELIVERY.md` por tipo de projeto

**Perguntas:**

- Quais campos são universais?
- Quais variam entre biblioteca, CLI, desktop, plugin CAD, web, API e aplicativo móvel?
- Como usar `N/A` sem mascarar lacunas?
- Como descobrir comandos reais a partir do repositório?
- Como manter o documento sincronizado com CI e scripts?

**Fontes desejadas:** Continuous Delivery, DORA, GitHub/GitLab/Azure DevOps e documentação dos ecossistemas de build.

**Entregável:** template curto de `DELIVERY.md` com perfis por tipo de projeto.

### P11 — Build reproduzível aplicada a stacks reais

**Perguntas:**

- Como verificar reprodutibilidade em .NET, Node, Python, Java, Rust e C/C++?
- Como controlar dependências, SDK, sistema operacional e timestamps?
- Quando builds bit a bit idênticas não são viáveis?
- Que nível mínimo é suficiente para projeto pessoal?

**Fontes desejadas:** Reproducible Builds, Nix, Bazel, SLSA, documentação oficial de cada runtime.

**Entregável:** guia operacional por stack e níveis de maturidade.

### P12 — Pipeline mínimo e paridade local/CI

**Perguntas:**

- Qual pipeline mínimo para cada tipo de projeto?
- Como garantir que os comandos locais equivalem aos jobs da CI?
- Como versionar e testar o próprio pipeline?
- Quais checks bloqueiam merge e quais apenas alertam?
- Como reexecutar com segurança e evitar efeitos duplicados?

**Fontes desejadas:** GitHub Actions, GitLab CI, Azure Pipelines, DORA e Continuous Delivery Foundation.

**Entregável:** modelos de pipeline mínimo, intermediário e crítico.

### P13 — Estratégia Git para Tasks executadas por agentes

**Perguntas:**

- Uma Task deve corresponder a branch, commit ou PR?
- Quando usar trunk-based development ou feature branches?
- Como nomear commits e relacioná-los aos handoffs?
- Como trabalhar com worktrees e agentes paralelos?
- Quando squash, merge commit ou rebase?

**Fontes desejadas:** Git oficial, GitHub, GitLab, trunkbaseddevelopment.com, DORA e estudos de integração contínua.

**Entregável:** política Git padrão simples e variantes justificadas.

### P14 — Versionamento de Build, produto e documentos

**Perguntas:**

- Qual relação entre `BUILD001`, Sprint e `v1.2.3`?
- Quando incrementar versão?
- Como versionar antes da primeira versão estável?
- Como tratar monorepos e múltiplos artefatos?
- Documentos e schema precisam de versões próprias?

**Fontes desejadas:** SemVer, CalVer, Conventional Commits, Release Please e documentação de package registries.

**Entregável:** política de versionamento sem ambiguidades.

### P15 — Artefatos, proveniência e retenção

**Perguntas:**

- O que precisa ser preservado para reproduzir ou auditar uma Release?
- Hash basta? Quando assinatura, SBOM e attestations são necessárias?
- Como promover o mesmo artefato entre ambientes?
- Onde guardar artefatos e por quanto tempo?

**Fontes desejadas:** SLSA, in-toto, Sigstore, SPDX, CycloneDX, NIST e registries oficiais.

**Entregável:** política proporcional de artefatos e proveniência.

### P16 — Homologação, entrega, deploy e pós-deploy

**Perguntas:**

- Qual diferença operacional entre pronto, homologado, entregue e implantado?
- Quem aprova cada transição?
- Quais smoke tests executar sobre o artefato e em produção?
- Quanto observar após deploy?
- Como registrar `READY_FOR_DEPLOY` sem declarar falso sucesso?

**Fontes desejadas:** Google SRE, DORA, AWS/Azure/Google architecture frameworks e literatura de continuous delivery.

**Entregável:** máquina de estados da Release e checklist proporcional.

### P17 — Rollback, roll-forward e migrações compatíveis

**Perguntas:**

- Quando rollback é mais arriscado que roll-forward?
- Como aplicar expand/contract em banco de dados?
- Como manter compatibilidade entre aplicação e schema?
- Como ensaiar backup e recuperação?
- Como tratar migrações irreversíveis?

**Fontes desejadas:** Google SRE, AWS Builders' Library, Azure Architecture Center, Martin Fowler e documentação oficial dos bancos/ferramentas.

**Entregável:** protocolo integrado de deploy, dados e recuperação.

### P18 — Observabilidade e incidentes após a entrega

**Perguntas:**

- Quais sinais mínimos confirmam que a Release está saudável?
- Como definir SLI, SLO e alertas simples?
- Quando encerrar a janela de observação?
- Como relacionar incidente, versão, rollback e ação corretiva?
- Como escrever postmortem sem culpa?

**Fontes desejadas:** Google SRE books, OpenTelemetry, Microsoft/AWS reliability guidance e incident.io/Atlassian como fontes industriais secundárias.

**Entregável:** template mínimo de observação e incidente.

---

## 5. Pesquisas complementares

### P19 — Segurança mínima proporcional ao projeto

Definir baseline pequeno: segredos, dependências, permissões da CI, varreduras, atualização e exceções. Usar NIST SSDF, OWASP SAMM/ASVS/SCVS e OpenSSF como fontes principais.

### P20 — Métricas úteis para projeto pessoal ou pequeno

Determinar quando DORA ajuda e quando vira burocracia. Investigar métricas mínimas de histórico: tempo de ciclo, taxa de falha, retrabalho, tempo bloqueado e frequência de entrega.

### P21 — Governança de mudanças no próprio SOP

Definir versão, changelog, aprovação, testes e migração dos documentos operacionais quando o protocolo mudar.

### P22 — Auditoria automática de consistência documental

Pesquisar validações para links quebrados, IDs duplicados, Task sem handoff, decisão sem status, Build sem Release, versão divergente e comandos inexistentes.

### P23 — Portabilidade entre agentes e ferramentas

Definir um núcleo independente de fornecedor e adaptadores para Codex, Claude Code, Gemini, agentes DeepSeek e ferramentas futuras.

### P24 — Privacidade, segurança e prompt injection em memória de projeto

Determinar quais arquivos são confiáveis, como marcar conteúdo externo, como evitar instruções maliciosas persistidas e como impedir vazamento de segredos nos handoffs.

### P25 — Custo, tamanho e envelhecimento do contexto

Definir limites baseados em evidência para documentos ativos, resumos, arquivamento, compactação e recuperação seletiva.

---

## 6. Política de fontes

### Nível A — fontes normativas e oficiais

- documentação oficial OpenAI/Codex;
- documentação oficial Anthropic/Claude Code;
- Google Cloud DORA e livros Google SRE;
- Google DeepMind, quando houver pesquisa diretamente relacionada;
- documentação oficial DeepSeek, quando houver material relevante;
- NIST, ISO/IEC acessível, IETF, CNCF, OpenSSF;
- documentação oficial Git, GitHub, GitLab, Azure DevOps e runtimes.

### Nível B — pesquisa acadêmica

- artigos revisados por pares;
- conferências de engenharia de software, sistemas e IA;
- papers com metodologia, limitações e resultados reproduzíveis;
- repositórios oficiais vinculados ao paper.

### Nível C — autores e organizações reconhecidas

- Martin Fowler e Thoughtworks;
- Kent Beck;
- Jez Humble e Dave Farley;
- Michael Nygard;
- Gene Kim, Nicole Forsgren e DORA;
- Charity Majors;
- Team Topologies;
- maintainers dos padrões e projetos citados.

### Restrições

- Prestígio da empresa não substitui relevância da fonte.
- OpenAI, Anthropic, DeepMind e DeepSeek devem ser usados para comportamento e operação de agentes quando possuírem documentação aplicável.
- Para Git, CI/CD, segurança e releases, padrões e especialistas de engenharia podem ser mais apropriados que laboratórios de IA.
- Blogs genéricos, conteúdo SEO e respostas sem autoria não devem fundamentar regras obrigatórias.
- Toda recomendação deve distinguir fato documentado, evidência empírica, prática industrial e decisão normativa deste SOP.

---

## 7. Ordem recomendada de execução

### Lote 1 — histórico e continuidade

1. P01 — memória operacional;
2. P02 — continuidade entre sessões;
3. P04 — mudanças e decisões;
4. P05 — lock e recuperação;
5. P25 — tamanho e envelhecimento do contexto.

### Lote 2 — planejamento e verificação

1. P03 — hierarquia de trabalho;
2. P06 — testes proporcionais;
3. P07 — gates;
4. P08 — circuit breaker;
5. P09 — subagentes.

### Lote 3 — entrega

1. P10 — `DELIVERY.md`;
2. P13 — Git;
3. P14 — versionamento;
4. P11 — builds;
5. P12 — pipelines;
6. P15 — artefatos;
7. P16 — entrega/deploy;
8. P17 — rollback/migrações;
9. P18 — observabilidade.

### Lote 4 — governança e automação

1. P19 — segurança proporcional;
2. P20 — métricas mínimas;
3. P21 — evolução do SOP;
4. P22 — auditoria automática;
5. P23 — portabilidade;
6. P24 — segurança da memória.

---

## 8. Critério de conclusão de cada pesquisa

Cada pesquisa deve gerar um arquivo Markdown contendo:

1. pergunta e escopo;
2. resumo executivo;
3. definições;
4. evidências e comparação entre fontes;
5. pontos de consenso e divergência;
6. limitações;
7. recomendação proporcional para projetos simples;
8. decisão sugerida para o SOP V2;
9. exemplos concretos;
10. referências com links, autor, organização e data de acesso.

Nenhuma pesquisa altera automaticamente o SOP. A mudança só ocorre após revisão e decisão registrada.
