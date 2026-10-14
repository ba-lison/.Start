# FASE 04 — EXECUÇÃO DE TASKS E HANDOFFS

> propósito: executar uma Task (ou o fechamento de uma Sprint) por chat, com verificação e registro
> ler quando: unidade `BUILDxxx_Syy_Tzz` ou `BUILDxxx_Syy_CLOSE`
> pré-condição: plano da Build aprovado

## Regra do chat

Um chat executa uma unidade (GP-01). Depois de `DONE`, `BLOCKED` ou `PAUSED`, o chat pode esclarecer a entrega, mas NUNCA começa outra unidade.

## Entrada

1. Seguir `.start/MODO-CONTINUAR.md`, passos 1 a 3.
2. Usar branch ou worktree conforme GP-08. Com `branch_por_task` ou `worktree_por_task`: a branch da Task nasce da branch principal local atualizada; o merge das branches da Sprint na principal acontece no fechamento da Sprint, com autorização (GF-07; DEC).
3. Criar `.memory/_ACTIVE_TASK.lock` só se `.memory/DELIVERY.md` adotar o lock.

## Execução

1. Somente o escopo (GF-08). Oportunidade fora da Task vira pendência.
2. Preservar mudanças existentes (GF-02).
3. Testes adequados: TDD quando ajuda, não como ritual.
4. Cada objetivo com contador de rodadas visível: `Rodada n/GP-04 — objetivo: [objetivo]` (GUARDRAILS §5).
5. Decisão no caminho: parar e perguntar (GF-06). Registrar na hora.
6. Subagentes: `.start/CONTRATO-SUBAGENTE.md`, com contador visível `Subagentes: n/GP-02`.
7. Contexto pesando: handoff `PAUSED` (GF-10).

## Saída

1. Revisar o diff e os resultados.
2. Fechar pelo `.start/PROTOCOLO_MESTRE_EXECUCAO.md` §8, com handoff em `.start/TEMPLATES/TEMPLATE_HANDOFF.md`.

## Fechamento da Sprint (`BUILDxxx_Syy_CLOSE`)

Em chat próprio (com GP-01 = `Sprint`: último passo do chat da Sprint):

1. verificação integrada do incremento;
2. consolidar os handoffs das Tasks;
3. registrar pendências;
4. atualizar o arquivo da unidade (`.memory/TASKS/BUILDxxx_Syy_CLOSE.md`) com os critérios marcados, a consolidação e as pendências; ele é o conteúdo do commit principal do fechamento;
5. fechar pelo `.start/PROTOCOLO_MESTRE_EXECUCAO.md` §8. No passo 6, o arquivo da Sprint recebe o estado (`DONE` ou `BLOCKED`) e o commit principal do fechamento.

Só uma Sprint fechada libera a seguinte ou o fechamento da Build.

## Sprint por chat (GP-01 = `Sprint`)

1. A unidade do chat é a Sprint: `BUILDxxx_Syy`. O handoff é `.memory/TASKS/HANDOFF_BUILDxxx_Syy.md` (pausa: `_P[n]`). As Tasks e o fechamento da Sprint fazem parte dessa mesma unidade; por isso não contam como "outra unidade" (GF-10).
2. O chat é o orquestrador da Sprint. Cada Task vai para um subagente, com o bloco do contrato; Tasks dependentes rodam em sequência.
3. O orquestrador confere cada relatório, faz o commit da Task, atualiza o arquivo da Task e `.memory/TASKS/INDEX.md` antes da próxima Task.
4. O limite GP-02 vale para a Sprint inteira. As rodadas continuam contadas por objetivo (GUARDRAILS §5).
5. O último passo do chat é o fechamento da Sprint (seção anterior).
6. SE a ferramenta não tem subagentes: avisar o usuário e voltar a uma Task por chat (D-11; DEC).

## Falha

Seguir GUARDRAILS §5 (rodadas) e §8 (parada). NUNCA rollback destrutivo automático.
