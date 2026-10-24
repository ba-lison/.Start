# Guardrails, Decisões (`D-01`..`D-12`) e Formato `.memory/`

> Ler quando: onboarding/aprovação de guardrails, dúvida se algo exige decisão do usuário, estouro de rodadas ou criação/edição de arquivos em `.memory/`.

---

## 1. O Que É Decisão Humana (`GF-06` — Perguntar Sempre)

Você **NUNCA** decide sozinho qualquer item abaixo:
- **`D-01`** Escopo, requisito, critério de aceite, prioridade ou ordem das entregas;
- **`D-02`** Arquitetura, estrutura de pastas/módulos, banco de dados, schema ou formato de arquivo;
- **`D-03`** Interface pública: contrato de API, CLI, tela/UI, mensagem visível ou nomenclatura de produto;
- **`D-04`** Dependência, biblioteca, SDK, serviço externo ou versão de runtime (adicionar, remover, trocar, subir major);
- **`D-05`** Segurança, autenticação, permissões, privacidade ou tratamento de dados sensíveis;
- **`D-06`** Custo financeiro, chamada externa paga ou ação irreversível;
- **`D-07`** Apagar, mover, renomear ou sobrescrever arquivo/conteúdo que já existia;
- **`D-08`** Escolha entre alternativas técnicas reais com trade-offs relevantes;
- **`D-09`** Desvio do escopo aprovado (`00_INDEX.md`, `PRD`, `SDD`, `Task` ou `DEC` ativa);
- **`D-10`** Aceitar risco conhecido, reduzir/pular verificação ou marcar etapa como `N/A`;
- **`D-11`** Mudar trilho (`LITE` ↔ `FULL`), modo ou parâmetro de guardrail;
- **`D-12`** Confirmar decisão deduzida de código legado no modo `ADOTAR` (`INFERRED` → `ACTIVE`).

### Quando NÃO Perguntar (Detalhe Interno de Implementação)
Siga direto sem interromper o usuário **somente** quando as 4 condições forem verdadeiras ao mesmo tempo:
1. Está dentro do escopo aprovado da unidade atual;
2. Segue convenção já existente no repositório;
3. Não altera contrato/interface visível fora do código;
4. É reversível sem custo.

### Formato Padrão de Pergunta e Registro Imediato (`DEC-xxx`)
```text
PERGUNTA — [tema curto]
Contexto: [1–2 linhas diretas do que está em jogo]
A) [opção recomendada] — [consequência prática]
B) [alternativa] — [consequência prática]
Recomendo: A, porque [motivo em 1 linha].
```
- Assim que o usuário responder (ou der uma instrução direta que configure `D-01`..`D-12`), **anexe imediatamente** em `.memory/DECISIONS.md` (`DEC-001`, `DEC-002`...) usando `templates/core/DECISIONS.md` e confirme em 1 linha (`Registrado como DEC-xxx.`).
- Decisão que substitui outra cria nova entrada `DEC-yyy` (`substitui: DEC-xxx`) e muda apenas o status da antiga para `SUPERSEDED` (**nunca apague entradas passadas**).
- No trilho `FULL`, decisões arquiteturais duradouras também ganham ADR curto em `.memory/adrs/ADR-NNNN.md` (`templates/full/ADR.md`).

---

## 2. Parâmetros Configuráveis (`GP-01` a `GP-10`) — Aprovação em 1 Bloco Único

Em vez de interrogar o usuário em 7 mensagens separadas, apresente os padrões recomendados **em um único bloco enxuto** durante o onboarding (`NOVO`, `ADOTAR` ou `MIGRAR`). O usuário aprova com `"Aprovo"` ou indica só o que quiser alterar:

| ID | Parâmetro | Padrão Recomendado | Opções Válidas |
| :--- | :--- | :--- | :--- |
| `GP-01` | Unidade por chat *(só `FULL`)* | `Task` | `Task` \| `Sprint` *(exige subagentes)* |
| `GP-02` | Máx. subagentes por chat | `10` | Inteiro `≥ 0` (`0` desativa subagentes) |
| `GP-03` | Tamanho máx. relatório subagente | `500` caracteres | Inteiro `≥ 200` |
| `GP-04` | Máx. rodadas por objetivo | `3` tentativas | Inteiro `1..5` |
| `GP-05` | Commit local ao fechar unidade | `automatico_ao_concluir` | `automatico_ao_concluir` \| `pedir_antes` |
| `GP-06` | Ações remotas (`push`/PR/deploy) | `pedir_sempre` | `pedir_sempre` \| `push_autorizado_em: <branch>` |
| `GP-07` | Branch de trabalho | `principal` *(pessoal)* ou `branch_por_task` *(time/PR)* | `principal` \| `branch_por_task` \| `worktree_por_task` |
| `GP-08` | Áreas protegidas extras | `nenhuma além de segredos` | Lista de pastas/arquivos intocáveis sem permissão |
| `GP-09` | Idioma dos registros | `pt-BR` | `pt-BR` \| `en` \| outro |
| `GP-10` | Gatilho de pausa de chat (`PAUSED`) | `compactacao_ou_40_msgs` | Compactação de contexto ou limite de msgs |

- **No trilho `LITE`:** os valores aprovados ficam resumidos direto no cabeçalho de `.memory/00_INDEX.md` (sem criar `PROJECT_GUARDRAILS.md` separado, salvo se houver regras customizadas extensas).
- **No trilho `FULL`:** grave em `.memory/PROJECT_GUARDRAILS.md` (`templates/core/PROJECT_GUARDRAILS.md`) com `status: APPROVED` e consolide a aprovação numa entrada `DEC-xxx`.

---

## 3. Verificação Proporcional ao Risco (`GF-04`) e Circuit Breaker (`GF-09`)

- **Risco Baixo** (docs, copy, config simples, UI isolada): inspeção de diff + lint/typecheck ou teste rápido do módulo.
- **Risco Médio** (regra de negócio, endpoint, CLI, refatoração): testes direcionados + build/typecheck limpo + checagem de chamadores (`grep` em quem consome a função alterada).
- **Risco Alto** (autenticação, cobrança/dinheiro, migração de banco, segurança, release): suíte ampla + casos negativos/borda + build de artefato real + plano claro de rollback.

### Regra de Parada do Circuit Breaker (`GP-04`, padrão = `3`)
1. Ao atacar cada objetivo verificável (ex: *"testes de auth passam"*, *"build compila sem erros"*), exiba `Rodada 1/3 — objetivo: ...`.
2. Se delegar a subagente, passe apenas as `RODADAS_RESTANTES` (`GP-04` menos rodadas já gastas naquele objetivo). A sua conferência final do diff/teste faz parte da mesma rodada.
3. **Parada imediata antes do teto:** se o mesmo erro se repetir sem hipótese nova, ou se surgir risco destrutivo/permissão/segurança.
4. **Bateu o limite sem resolver:** crie `.memory/BLOQUEIO_<UNIDADE>.md` (no `FULL`: `.memory/TASKS/BLOQUEIO_<UNIDADE>.md`) usando `templates/core/BLOQUEIO.md`, preserve o código sem `git reset` destrutivo e pergunte ao usuário como seguir.

---

## 4. Regras de Escrita LLM-First para `.memory/`

1. **Cabeçalho obrigatório em todo arquivo de `.memory/`:**
   ```markdown
   # TÍTULO CLARO DO ARQUIVO
   > propósito: [o que guarda]
   > ler quando: [gatilho de leitura]
   > atualizar quando: [gatilho de escrita]
   ```
2. **Zero gordura:** frases imperativas curtas, caminhos exatos entre crases relativos à raiz do projeto (ex: `.memory/DECISIONS.md`), datas `YYYY-MM-DD`, estados em inglês maiúsculo (`APPROVED`, `READY`, `RUNNING`, `PAUSED`, `BLOCKED`, `DONE`).
3. **Fato vs. Hipótese:** nunca misture dedução com fato confirmado. Deduções não confirmadas levam prefixo explícito `hipótese:`.
4. **Política de mutação de cada arquivo:**
   - **Fotografia (sobrescrever mantendo enxuto; o Git guarda o histórico):** `00_INDEX.md`, `PROJECT_CONTEXT.md`, `DELIVERY.md`, `TASKS/INDEX.md`, e no `LITE` o `HANDOFF.md`.
   - **Histórico (somente anexar no fim; nunca apagar linhas):** `DECISIONS.md`, `CHANGELOG.md`, `PENDENCIAS_FUTURAS.md`.
   - **Imutável após fechado (no `FULL`):** `TASKS/HANDOFF_<UNIDADE>.md`, `BLOQUEIO_*.md`, `DIAGNOSTICO*.md`, `ADR-*.md` aceitos e `RELEASE_*.md` (correção posterior entra apenas como seção anexada `## Correção YYYY-MM-DD`).
5. **O que NUNCA entra em `.memory/`:** segredos/tokens (`GF-03`), dumps gigantes de logs brutos (registre apenas comando + resultado resumido), nem cópias inteiras de código-fonte (aponte o caminho do arquivo).
