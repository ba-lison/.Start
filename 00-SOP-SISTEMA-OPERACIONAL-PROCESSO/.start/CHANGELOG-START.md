# CHANGELOG DO KIT .start

> propósito: o que mudou em cada versão do kit e como migrar projetos
> ler quando: modo AJUSTAR; atualização do kit num projeto
> atualizar quando: toda nova versão do kit (anexar no topo)

## 3.0.0 — 2026-09-22

### Mudou

- Memória do projeto: `docs/` → `.memory/`. Evita misturar com a `docs/` própria de projetos existentes.
- Entrada única: `AGENTS.md` na raiz + `.start/INICIO.md` (roteador de modos). Ferramenta que não carrega `AGENTS.md` recebe um ponteiro de uma linha, sem conteúdo próprio.
- Guardrails: fixos (GF-01 a GF-14), parâmetros com padrão (GP-01 a GP-13) e questionário de 7 perguntas, uma por vez.
- Decisões: lista fechada do que é decisão (D-01 a D-12); registro no momento da resposta; toda autorização vira decisão; formato de entradas `## DEC-xxx` no lugar de tabela.
- Subagentes: profundidade 1 fixa; padrão de 10 por chat e relatório de até 500 caracteres; bloco de contrato obrigatório em todo pedido.
- Rodadas: padrão de 3 por objetivo, contando erro ou acerto e somando orquestrador, subagentes e chats; proibido redisparar para zerar a contagem.
- Troca de chat antes do fim (GP-13): quando a ferramenta compactar o contexto ou o chat passar de 40 mensagens do usuário.
- Unidades de trabalho: `ONBOARDING`, `PRD-xxx`, `PLANO-BUILDxxx`, Task, fechamento de Sprint, Sprint inteira (GP-01 = `Sprint`), fechamento de Build e `AJUSTE-YYYY-MM-DD` — uma por chat.
- Modos: NOVO, ADOTAR, AJUSTAR e CONTINUAR, com retomada na parte em que parou.
- Novo estado de unidade: `PAUSED`, com handoff `HANDOFF_[UNIDADE]_P[n].md`.
- Novos arquivos da memória: `00_INDEX.md`, `DIAGNOSTICO*.md`, `TASKS/BLOQUEIO_[UNIDADE].md`, `_rascunho/`, `_arquivo/`.
- A pasta `.start/` passa a ser versionada com o projeto.
- Templates renomeados: `TEMPLATE_MICRO_HANDOFF_TASK.md` → `TEMPLATE_HANDOFF.md`; `TEMPLATE_RELATORIO_CIRCUIT_BREAKER.md` → `TEMPLATE_BLOQUEIO.md`; `TEMPLATE_HANDOFF_RELEASE_BUILD.md` → `TEMPLATE_RELEASE.md`; `TEMPLATE_DECISION_LOG_ADR.md` → `TEMPLATE_ADR.md`.

### Decisões do usuário que originaram esta versão (2026-09-22)

- O agente pergunta sobre os guardrails no início de todo projeto, e o usuário decide o que o agente pode e não pode fazer.
- Unidade padrão por chat: Task.
- "Decisão" = lista definida; detalhe interno da Task aprovada não é decisão; na dúvida, perguntar.
- Limites padrão: 10 subagentes por chat, relatório de 500 caracteres, 3 rodadas por objetivo.
- Tudo que é relevante vai para `.memory/`, em formato focado em leitura por LLM, sem arquivo de conteúdo por ferramenta.
- Três casos de entrada: projeto do zero (foco principal), projeto já iniciado e ajuste de projeto que já usa o processo.

### Migração a partir da versão 2 (projeto com `docs/`)

Executada pelo `.start/MODO-AJUSTAR.md`, nesta ordem:

1. SE não há Git: `git init` e commit de base autorizado, sem arquivos de segredo.
2. Criar `.memory/00_INDEX.md` (`versao_start: 2.x`) e o `AGENTS.md`.
3. Mover com `git mv`, com autorização, para o mesmo nome dentro de `.memory/`:
   - `docs/PROJECT_CONTEXT.md`
   - `docs/PROJECT_GUARDRAILS.md`
   - `docs/DECISIONS.md`
   - `docs/CHANGELOG.md`
   - `docs/DELIVERY.md`
   - `docs/prd/`, `docs/sdd/`, `docs/adrs/`
   - `docs/BUILDS/`, `docs/SPRINTS/`, `docs/TASKS/`, `docs/RELEASES/`
4. NÃO mover nenhum outro arquivo de `docs/`.
5. Trocar, dentro dos arquivos movidos, os caminhos `docs/` → `.memory/`.
6. Converter `DECISIONS.md` de tabela para entradas `## DEC-xxx` e completar `TASKS/INDEX.md`, `PROJECT_CONTEXT.md` e `DELIVERY.md`.
7. Guardrails: arquivar o arquivo antigo em `.memory/_arquivo/` e fazer as 7 perguntas, com os valores da versão 2 como sugestão.
8. Fechamento: `versao_start: 3.0.0`, `modo_atual: CONTINUAR`, commit local do ajuste incluindo `.start/`.

## 2.x — até 2026-09-21

Versão anterior, com memória em `docs/`. O que mudou e como migrar: seção 3.0.0 acima.
