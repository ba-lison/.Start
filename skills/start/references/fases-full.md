# Fases 01 a 05 — Procedimento Completo do Trilho `FULL` (SDD)

> Ler quando: projeto operando no trilho **`FULL`** (`PRD`, `SDD`, `DELIVERY`, `Build`, `Sprints`, `Tasks` e `Release`). Não carregar no trilho `LITE`.

Hierarquia obrigatória no `FULL`: **`Projeto → Build → Sprint → Task → Handoff`**.
Fluxo de unidades (1 unidade por chat por padrão em `GP-01 = Task`):
```text
ONBOARDING → PRD-001 → PLANO-BUILD001 → BUILD001_S01_T01 … → BUILD001_S01_CLOSE → BUILD001_CLOSE
```

---

## Fase 01 — Contexto Inicial e Perfil (`ONBOARDING`)

> Pré-condição: aprovação do bloco único de Guardrails (`PROJECT_GUARDRAILS.md` com `status: APPROVED`). Proibido codar o produto aqui.

1. Levante apenas o que ainda não estiver claro (em 1 mensagem objetiva):
   - Qual problema real o produto resolve e para quem?
   - Qual resultado mensurável/observável comprova sucesso?
   - Restrições duras (prazo, SO/plataforma, custo, stack obrigatória/proibida, dados sensíveis/integrações)?
   - Qual o **Perfil Operacional** do projeto (recomende sempre `SIMPLE` salvo risco real compartilhado/produção crítica):
     - **`SIMPLE`**: projeto pessoal, interno leve ou local (testes proporcionais + pacote/hash na entrega);
     - **`STANDARD`**: produto compartilhado/publicado (CI, code review, homologação e checagem de segurança);
     - **`CRITICAL`**: produção sensível, financeiro ou alto impacto (aprovações segregadas, migração reversível, smoke test e rollback ensaiado).
2. Grave `.memory/PROJECT_CONTEXT.md` (`templates/core/PROJECT_CONTEXT.md`) e registre decisões em `.memory/DECISIONS.md`.

---

## Fase 02 — Especificação de Produto (`PRD-xxx`)

> Pré-condição: `ONBOARDING` concluído. Define **o quê, para quem e como aceitar**, sem travar detalhes prematuros de código.

1. Crie `.memory/prd/PRD-xxx.md` (`templates/full/PRD.md`) com:
   - Problema, público-alvo e métricas de sucesso;
   - Requisitos funcionais numerados (`RF-001`, `RF-002`...) e não funcionais proporcionais (`RNF-001`...);
   - Cenários principais e critérios objetivos de aceite;
   - **Fora de escopo explícito** (o que NÃO entra nesta Build);
   - Riscos, dados sensíveis e dependências externas.
2. Resolva ambiguidades perguntando ao usuário (`GF-06`) e registrando `DEC-xxx`.
3. Peça aprovação explícita do `PRD-xxx`, registre a decisão, atualize índices/handoff (`HANDOFF_PRD-xxx.md`), faça o commit da unidade e defina próxima unidade = `PLANO-BUILDxxx`. **PARE.**

---

## Fase 03 — Arquitetura, `DELIVERY` e Decomposição (`PLANO-BUILDxxx`)

> Pré-condição: `PRD-xxx` aprovado (ou escopo equivalente aprovado via `DEC-xxx` no modo `ADOTAR`).

1. **SDD (`.memory/sdd/SDD-xxx.md` via `templates/full/SDD.md`):**
   - Mapeie quais `RF`/`RNF` são cobertos;
   - Defina componentes, fronteiras, contratos/APIs, modelo de dados/migrations e estratégia de testes;
   - Toda escolha estrutural (`D-02` a `D-04`) passa por aprovação do usuário e vira `DEC-xxx` (+ `ADR-NNNN.md` se duradoura).
2. **DELIVERY (`.memory/DELIVERY.md` via `templates/full/DELIVERY.md`):**
   - Registre comandos reais de setup, lint, typecheck, teste, build e empacotamento (marque com `hipótese:` o que ainda não pôde ser executado);
   - Defina política de branches/tags, artefato final e gates de deploy/rollback proporcionais ao Perfil (`SIMPLE`/`STANDARD`/`CRITICAL`).
3. **Decomposição Executável (`Build → Sprints → Tasks`):**
   - Crie `.memory/BUILDS/BUILDxxx.md` (`templates/full/BUILD.md`).
   - Crie ao menos 1 Sprint verificável em `.memory/SPRINTS/BUILDxxx_Syy.md` (`templates/full/SPRINT.md`), encerrada por `BUILDxxx_Syy_CLOSE`.
   - Quebre cada Sprint em Tasks atômicas que caibam folgadas em 1 chat: `.memory/TASKS/BUILDxxx_Syy_Tzz.md` (`templates/full/TASK.md`), especificando objetivos verificáveis (base do contador de 3 rodadas), arquivos tocados, critérios de aceite e verificação.
   - Liste todas as unidades em `.memory/TASKS/INDEX.md` (`templates/full/TASK_INDEX.md`).
4. Peça aprovação do plano completo (`SDD + DELIVERY + Tasks`), registre `DEC-xxx`, feche a unidade com handoff + commit local e defina a 1ª Task (`BUILDxxx_S01_T01`, ou `BUILDxxx_S01` se `GP-01 = Sprint`) como próxima. **PARE.**

---

## Fase 04 — Execução de `Tasks` e Fechamento de `Sprint`

### A. Modo Padrão (`GP-01 = Task` → `BUILDxxx_Syy_Tzz`)
1. Siga o `MODO CONTINUAR` (`references/modos-entrada.md` §3): confira Git, anuncie escopo/verificação e aguarde confirmação.
2. Respeite a política de branch (`GP-07`: `principal`, `branch_por_task` ou `worktree_por_task`).
3. Implemente **estritamente o escopo da Task** (`GF-08`), mostrando o contador `Rodada n/<GP-04> — objetivo: ...` a cada ciclo de validação.
4. Se usar subagentes, aplique `references/contrato-subagente.md` e confira pessoalmente o diff e os testes (`GF-04`).
5. Atualize `.memory/CHANGELOG.md` (se entregou funcionalidade visível), `.memory/TASKS/HANDOFF_BUILDxxx_Syy_Tzz.md`, `.memory/TASKS/INDEX.md` (`DONE`) e `.memory/00_INDEX.md`, faça 1 commit atômico (`GF-11`) e **PARE**.

### B. Modo Sprint por Chat (`GP-01 = Sprint` → `BUILDxxx_Syy`)
1. O chat atual atua como **Orquestrador da Sprint**. Cada Task (`T01`, `T02`...) é delegada em sequência (ou paralelo só se arquivos permitidos forem 100% disjuntos!) a subagentes com `references/contrato-subagente.md`.
2. Ao concluir cada Task, o orquestrador valida os testes/diff, atualiza `TASKS/INDEX.md` e faz 1 commit limpo daquela Task antes de disparar a próxima.
3. Ao final das Tasks, executa o **Fechamento da Sprint** no mesmo chat.

### C. Fechamento da Sprint (`BUILDxxx_Syy_CLOSE`)
1. Rode a verificação integrada do incremento completo da Sprint (build limpa + suíte de testes da Sprint).
2. Consolide pendências remanescentes em `.memory/TASKS/PENDENCIAS_FUTURAS.md`.
3. Marque a Sprint como `DONE` em `.memory/SPRINTS/BUILDxxx_Syy.md` e `TASKS/INDEX.md`, gere o handoff e faça o commit de fechamento. Somente uma Sprint `DONE` libera a próxima Sprint ou o `BUILDxxx_CLOSE`.

---

## Fase 05 — Fechamento da Build, Release e Operação (`BUILDxxx_CLOSE`)

> Pré-condição: todas as Sprints da `BUILDxxx` em estado `DONE`.

1. Confira cobertura de todos os `RF`/`RNF` prometidos no `PRD` da Build.
2. Rode a suíte ampla de testes + build limpa de produção + checagens de segurança/migração aplicáveis ao perfil.
3. Atualize versão do produto (`VERSION` / manifesto), consolide `.memory/CHANGELOG.md`, gere o pacote/artefato real quando aplicável, valide o artefato construído e calcule hash `SHA-256` (quando binário/pacote).
4. Crie `.memory/RELEASES/RELEASE_BUILDxxx.md` (`templates/full/RELEASE.md`) registrando estado comprovado:
   - **`RELEASED`**: somente após validação real do pacote entregue ou smoke-test pós-deploy autorizado (`GF-07`);
   - **`READY_FOR_DEPLOY`**: artefato construído e verificado localmente, aguardando autorização ou credencial de deploy.
5. Pergunte ao usuário o próximo passo (`PRD` da próxima Build, manutenção ou encerramento), atualize `00_INDEX.md` + handoff final e faça o commit de fechamento (e `git tag` local se previsto em `DELIVERY.md`). **PARE.**
