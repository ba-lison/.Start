# Modos de Entrada (`NOVO`, `ADOTAR`, `CONTINUAR`, `MIGRAR`)

> Ler quando: após detectar o modo no Passo 0 do `SKILL.md`.

---

## 1. `MODO NOVO` — Projeto Começando do Zero

### Fluxo no Trilho `LITE` (Rápido / Ponytail)
1. **Entrevista Relâmpago + Guardrails em 1 Mensagem:**
   - Se o usuário ainda não descreveu o que quer construir, pergunte de forma curta (numa única mensagem):
     1. O que vamos construir e qual resultado comprova que deu certo?
     2. Qual stack/ambiente (ou quer recomendação)?
     3. Alguma restrição ou ajuste nos guardrails padrão (máx. 3 rodadas, commit local por entrega, push só com autorização)?
   - Se o usuário já passou o contexto no prompt inicial, mostre direto o **Mini-Escopo + Guardrails Padrão** e peça um único **"Aprovo"**.
2. **Preparação Pós-"Aprovo":**
   - Se não houver Git: inicialize `git init` e garanta `.gitignore` protegendo `.env*`, `_backup_original/` e dependências pesadas.
   - Crie apenas os **3 arquivos enxutos**:
     - `.memory/00_INDEX.md` (a partir de `templates/lite/00_INDEX_LITE.md`, já preenchido com `status: APPROVED`, objetivo, escopo, fora de escopo e critérios de aceite);
     - `.memory/DECISIONS.md` (`templates/core/DECISIONS.md`, registrando `DEC-001` de escopo/stack + guardrails);
     - `.memory/HANDOFF.md` (`templates/lite/HANDOFF_LITE.md`).
   - Configure a ponte agnóstica na raiz (`AGENTS.md` com bloco de `templates/core/AGENTS_BLOCK.md` + `CLAUDE.md` contendo `@AGENTS.md`, respeitando `GF-12`).
3. **Execução Imediata Permitida:**
   - No `LITE`, assim que o escopo inicial estiver aprovado, **você já pode implementar a primeira entrega neste mesmo chat** (respeitando o Circuit Breaker de 3 rodadas), testar, atualizar `.memory/00_INDEX.md` + `.memory/HANDOFF.md` e fechar com 1 commit limpo (`GF-11`).

### Fluxo no Trilho `FULL` (SDD Completo)
1. **Unidade `ONBOARDING` (1 Bloco de Guardrails + Fase 01):**
   - Apresente a tabela de `GP-01..GP-10` com os padrões recomendados (`references/guardrails-e-decisoes.md` §2) + as perguntas de contexto da Fase 01 (`references/fases-full.md` §1) que ainda faltarem.
   - **Proibido escrever código do produto durante o `ONBOARDING`.**
2. **Criação Estruturada Pós-Aprovação:**
   - Garanta `git init` + `.gitignore` seguro.
   - Crie `.memory/00_INDEX.md` (`templates/core/00_INDEX.md`), `.memory/PROJECT_GUARDRAILS.md` (`status: APPROVED`), `.memory/PROJECT_CONTEXT.md`, `.memory/DECISIONS.md`, `.memory/CHANGELOG.md`, `.memory/TASKS/INDEX.md` e `.memory/TASKS/PENDENCIAS_FUTURAS.md`.
   - Configure `AGENTS.md` + `CLAUDE.md` (`@AGENTS.md`) sem sobrescrever regras existentes sem permissão.
3. **Fechamento do `ONBOARDING`:**
   - Defina próxima unidade = `PRD-001` (ou `PLANO-BUILD001` se o requisito já veio 100% fechado em documento validado), crie `.memory/TASKS/HANDOFF_ONBOARDING.md`, faça 1 commit local limpo (`chore(memory): conclui onboarding [ONBOARDING]`), entregue o prompt do próximo chat e **PARE**.

---

## 2. `MODO ADOTAR` — Repositório de Código Já Existente (Sem `.memory/`)

1. **Parte 0 — Diagnóstico 100% Somente-Leitura (Obrigatório Antes de Tocar em Nada):**
   Inspecione com caminhos exatos:
   - Estrutura de pastas (1º e 2º nível) e manifestos de stack;
   - Scripts reais de setup, lint, typecheck, teste e build;
   - CI/CD existente (`.github/workflows/`, etc.);
   - Estado do Git (branch, `HEAD`, arquivos sujos não commitados, últimos 5 commits, tags);
   - Documentação existente (`README.md`, `docs/`, ADRs) — **nunca mova nem copie docs existentes para `.memory/`**, apenas aponte o caminho;
   - Instruções existentes de agentes (`AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `GEMINI.md`) e eventuais conflitos;
   - Segredos aparentes pelo nome (`.env*`, `*.pem`, `*.key`, `credentials*` — **sem ler conteúdo**), verificando via `git ls-files` se algum vazou no índice do Git;
   - Decisões técnicas já visíveis no código (linguagem, framework, banco, arquitetura);
   - **Sinergia opcional com outras skills:** se existir `graphify-out/GRAPH_REPORT.md` no projeto (ou se o repositório for médio/grande e a skill `graphify` estiver disponível), consulte o grafo arquitetural (`--no-viz`) para poupar leitura bruta; durante a implementação de código, se a skill `ponytail` estiver instalada, mantenha a disciplina de menor mudança completa ativa.
2. **Apresentação do Diagnóstico + Proposta:**
   Mostre ao usuário:
   ```text
   DIAGNÓSTICO — [nome do projeto]
   O QUE ENCONTREI: [stack, comandos reais, Git, docs e alertas de segredo/sujeira]
   DECISÕES DEDUZIDAS DO CÓDIGO (a confirmar): [lista curta]
   TRILHO RECOMENDADO: [LITE | FULL] — [motivo]
   GUARDRAILS PADRÃO: [resumo em 3 linhas]
   O QUE É PRIORIDADE AGORA? [pergunta aberta para definir a próxima entrega]
   ```
   **Pare e aguarde confirmação antes de criar `.memory/` ou tocar em `AGENTS.md`/`CLAUDE.md`.**
3. **Preparação Segura Pós-Aprovação:**
   - **Se houver arquivos sujos do usuário no Git:** nunca misture com o commit de adoção nem rode `git reset`. Pergunte se o usuário prefere commitar antes ou se devemos commitar exclusivamente os arquivos novos de `.memory/` (`git add .memory ...`).
   - **Se `AGENTS.md` ou `CLAUDE.md` já existirem:** mostre exatamente o bloco `<!-- start:inicio -->` que será inserido no topo de `AGENTS.md` e garanta que nenhum conteúdo prévio seja perdido (se `CLAUDE.md` tiver regras próprias, proponha fundi-las no `AGENTS.md` deixando `CLAUDE.md` apenas com `@AGENTS.md`, salvando cópia original em `_backup_original/` se substituir).
   - **Se houver segredo rastreado no Git:** ofereça `git rm --cached <arquivo>` + inclusão no `.gitignore` + recomendação de rotacionar chave (`DEC-xxx`).
   - No `ADOTAR + LITE`: crie os 3 arquivos de `LITE` + confirme o escopo da tarefa atual (já pode executar se o usuário pedir).
   - No `ADOTAR + FULL`: salve `.memory/DIAGNOSTICO.md` (`templates/full/DIAGNOSTICO.md`), registre decisões herdadas confirmadas em `DECISIONS.md` (`ACTIVE`), feche `ONBOARDING` com handoff + commit local só dos arquivos do processo e **PARE**.

---

## 3. `MODO CONTINUAR` — Retomada Determinística do Projeto

1. **Leitura Enxuta na Ordem Exata:**
   - **Trilho `LITE`:** leia `.memory/00_INDEX.md`, `.memory/DECISIONS.md` (ativas) e `.memory/HANDOFF.md`.
   - **Trilho `FULL`:** leia `.memory/00_INDEX.md`, `.memory/PROJECT_GUARDRAILS.md`, `.memory/PROJECT_CONTEXT.md`, `.memory/DECISIONS.md` (`ACTIVE`), o último `HANDOFF` citado, o arquivo da próxima unidade (`PRD` / `SDD` / `Task` / `Sprint`) e a fase correspondente em `references/fases-full.md`.
2. **Conferência da Realidade no Git (`GF-01` — Sem Alterar Nada):**
   - Confira `git status --porcelain` e `git log -n 3 --oneline`.
   - Há commits novos feitos por fora do último handoff ou mudanças não commitadas no workspace? Se sim, relate em 2 linhas e pergunte como tratar antes de editar.
   - Se a unidade anterior ficou `PAUSED`, recupere de onde parou e o contador exato de rodadas já gastas por objetivo (`GF-09`).
3. **Anúncio de 5 Linhas e Confirmação:**
   ```text
   UNIDADE ATUAL: [ID / Objetivo]
   ESCOPO: [o que entra] · FORA DE ESCOPO: [o que não entra]
   VERIFICAÇÃO PREVISTA: [comando/teste real]
   DECISÕES OU AUTORIZAÇÕES PENDENTES: [nenhuma | lista]
   Posso executar?
   ```
   *(Se o usuário já mandou na mensagem inicial um pedido diferente da unidade prevista, pergunte se A) o pedido dele vira a unidade de agora ou B) entra como pendência).*
4. **Execução + Fechamento Limpo (`GF-11`):**
   - Execute apenas o escopo aprovado com contador de rodadas visível (`Rodada n/3`).
   - Rode a verificação real proporcional ao risco (`GF-04`).
   - Atualize `.memory/DECISIONS.md` (se houve decisão), `.memory/CHANGELOG.md` (no `FULL`, se entregou mudança), o handoff (`.memory/HANDOFF.md` no `LITE` ou `.memory/TASKS/HANDOFF_<UNIDADE>.md` no `FULL`), `.memory/TASKS/INDEX.md` (no `FULL`) e `.memory/00_INDEX.md`.
   - Remova `.memory/_rascunho/` se usou subagentes.
   - Faça **1 commit local único e limpo** (`<tipo>(<escopo>): <descrição> [UNIDADE]`) contendo os arquivos da unidade + `.memory/` atualizada.
   - Entregue resumo curto + hash do `HEAD` final + bloco de continuidade para o próximo chat. No trilho `FULL`, **PARE** após concluir a unidade do chat (`GP-01`).
   - **Se o contexto compactar ou bater o limite `GP-10` no meio da tarefa:** avise o usuário, salve handoff com status `PAUSED` anotando o passo exato e as rodadas usadas, faça commit `wip(<escopo>): pausa segura [UNIDADE]` (se autorizado/seguro) e **PARE**.

---

## 4. `MODO MIGRAR` — Projetos com `.start/` Antiga (v3) ou `docs/` (v2)

1. **Auditoria Somente-Leitura:**
   - Identifique se o projeto usa `.Start` v2 (`docs/PROJECT_GUARDRAILS.md` / `docs/PROJECT_CONTEXT.md`) ou `.Start` v3 (pasta física `.start/` com 39 arquivos + `.memory/` v3).
   - Mostre o que encontrou, proponha escolher o trilho (`LITE` ou `FULL`) e liste exatamente o plano de migração com **ZERO DELETE**.
2. **Execução Pós-"Aprovo" (`GF-02` — Zero Perda de Dados):**
   - **Vindo da v2 (`docs/`):** mova com `git mv` apenas os artefatos do processo (`PROJECT_*`, `DECISIONS.md`, `DELIVERY.md`, `prd/`, `sdd/`, `adrs/`, `BUILDS/`, `SPRINTS/`, `TASKS/`, `RELEASES/`) para `.memory/`, preservando intocada qualquer documentação própria em `docs/`. Atualize referências internas `docs/` → `.memory/` e converta tabela antiga de decisões para blocos `## DEC-xxx` (arquivando cópia bruta original em `_backup_original/`).
   - **Vindo da v3 (pasta `.start/` no projeto):** como o motor agora roda via Skill (`start`), proponha aposentar a cópia física `.start/` movendo-a intacta para `_backup_original/dot-start-v3/` (garantindo `_backup_original/` no `.gitignore` e removendo `.start/` do índice Git via `git rm --cached -r .start` somente após autorização explícita do usuário — zero arquivo apagado do disco!).
   - Atualize o bloco `<!-- start:inicio v4.0.0 -->` no `AGENTS.md`, valide `CLAUDE.md` (`@AGENTS.md`), marque `versao_start: 4.0.0` em `.memory/00_INDEX.md` e feche com 1 commit limpo de migração.
