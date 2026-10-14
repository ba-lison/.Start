# Matriz de Decisões do SOP V2

## Síntese

As pesquisas sustentam um processo menor que a proposta inicial. O histórico deve ser rígido; a execução deve ser proporcional ao risco. A V2 não precisa simular uma organização corporativa para ser confiável.

| Tema | Situação anterior | Decisão recomendada |
| :--- | :--- | :--- |
| Sessão | Uma Task por chat como lei absoluta | Manter como padrão local; permitir encerramento antecipado |
| Sprint | Confundida com Build | Obrigatória; agrupador funcional dentro da Build |
| Build | Marco/Sprint/versão misturados | Separar plano, execução da CI e versão pública |
| Histórico | Muitos arquivos sem fronteira clara | Separar estado, decisão, evento, evidência e Release |
| Handoff | Resumo amplo | Delta curto, verificável e com ponteiros |
| Contexto | Limite fixo de linhas | Utilidade, índice, arquivamento e carregamento sob demanda |
| Backup | Cópia `.backup` obrigatória | Git/worktree como padrão; cópia apenas sem Git |
| Lock | Fonte de verdade operacional | Marcador opcional; sempre verificar workspace/Git |
| TDD | Obrigatório em toda Task | Aplicar quando útil; verificação proporcional sempre |
| Tentativas | Três falhas absolutas | Parada por risco/ausência de progresso, teto configurável |
| Subagentes | Até cinco, profundidade fixa | Somente para subtarefas independentes; limite da ferramenta |
| Aprovação | Antes de toda modificação | Exigir em mudança destrutiva, externa, ambígua ou de alto risco |
| Delivery | Checklist corporativo universal | Template curto por perfil, com `N/A` justificado |
| CI/CD | Teoria na biblioteca | Comandos reais e gates no `DELIVERY.md` |
| Artefato | Caminho genérico | Versão + commit + hash + teste do artefato |
| Deploy | Etapa implícita | Estados comprováveis; `READY_FOR_DEPLOY` quando faltar autoridade |
| Segurança | Tudo ou nada | Baseline mínimo + controles por risco |
| DORA | Potencial obrigação universal | Métricas automáticas e aplicáveis; nunca medir indivíduo |
| Portabilidade | Regras misturadas à ferramenta | Núcleo agnóstico + adaptadores Codex/Claude/Gemini |

## Núcleo mínimo proposto

Para qualquer projeto:

```text
PROJECT_CONTEXT.md
DECISIONS.md
CHANGELOG.md
DELIVERY.md
prd/
sdd/
BUILDS/
TASKS/
RELEASES/
```

Para cada Task:

```text
especificação → mudança → verificação → handoff → histórico → próxima ação
```

Para cada Release:

```text
commit → build limpa → artefato → teste do artefato
→ entrega/deploy → verificação → estado final
```

## Próximas alterações documentais

1. Reescrever o Protocolo Mestre com as decisões aprovadas.
2. Reescrever Fase 03, separando Sprint, Build e versão.
3. Reescrever Fase 04, removendo TDD e três falhas como absolutos.
4. Criar Fase 05 para fechamento, entrega e operação.
5. Revisar guardrails para usar perfis de risco.
6. Criar templates mínimos: contexto, delivery, Build, índice de Tasks, handoff e Release.
7. Criar auditor read-only da consistência documental.

## Decisões adotadas no baseline operacional

- troca de chat obrigatória após cada Task de implementação;
- ao menos uma Sprint obrigatória por Build, com toda Task vinculada a uma Sprint;
- commits locais conforme política do projeto; push, PR, publicação e deploy dependem de autorização;
- perfis `SIMPLE`, `STANDARD` e `CRITICAL` com verificação proporcional;
- núcleo documental mínimo mantido também em projetos pequenos, usando conteúdo curto e `N/A` justificado.

## Revisão V3 — 2026-09-22

Decisões do usuário que levaram ao kit `.start` 3.0.0 (detalhes e migração em `00-SOP-SISTEMA-OPERACIONAL-PROCESSO/.start/CHANGELOG-START.md`):

| Tema | V2 | V3 |
| :--- | :--- | :--- |
| Guardrails | princípios + pergunta aberta | fixos (GF) + parâmetros com padrão (GP) + questionário obrigatório, uma pergunta por vez |
| Tentativas | teto configurável, sem valor | GP-04 = 3 rodadas por objetivo (erro ou acerto contam), mantendo parada antecipada por falta de progresso e parada imediata por risco |
| Subagentes | limite da ferramenta | profundidade 1 fixa; GP-02 = 10 por chat; GP-03 = relatório de até 500 caracteres; contrato no próprio pedido |
| Decisões | registrar decisão duradoura | nenhuma decisão sem o usuário (lista D-01 a D-12); registro no momento da resposta |
| Memória | `docs/` | `.memory/`, com `00_INDEX.md` como entrada |
| Portabilidade | núcleo agnóstico + adaptadores | `AGENTS.md` único; ponteiro de uma linha só onde a ferramenta exigir |
| Entrada | apontar para o Protocolo Mestre | roteador `INICIO.md` com modos NOVO, ADOTAR, AJUSTAR e CONTINUAR |

Os números da V3 não contradizem esta matriz: ela recusou limites universais sem configuração, e a V3 os torna padrões que cada projeto confirma ou muda.
