# MODO AJUSTAR — projeto que já usa o processo, em versão antiga ou incompleta

> propósito: alinhar a `.memory/` (ou a `docs/` da versão 2) à versão atual do kit, por partes e com autorização
> ler quando: `.start/INICIO.md` indicou AJUSTAR e o usuário confirmou
> unidade: `AJUSTE-YYYY-MM-DD`

## Retomada

SE este modo foi retomado (`.start/INICIO.md`, regra 1): ir direto à `parte_atual` de `.memory/00_INDEX.md`. NUNCA refazer a Parte 0 nem a Preparação. SE `parte_atual: 0`: completar a auditoria parcial salva, sem refazer o que já está nela.

## Regras deste modo

1. A Parte 0 é SOMENTE LEITURA.
2. Mover, renomear ou converter arquivo existente exige autorização (GF-02, GF-07). Mostrar a lista exata antes. Toda autorização vira DEC (GUARDRAILS §3.4, itens 3 e 9).
3. Mover com `git mv`, para preservar o histórico.
4. Converter formato preservando todo o conteúdo: nada é apagado; conteúdo substituído vai para `.memory/_arquivo/`.
5. Commits deste modo: na branch atual; só os arquivos deste ajuste, mais `.start/`.
6. Ordem obrigatória: migração → conversão → guardrails. Assim o questionário não cria arquivos que a migração vai mover.
7. Pausa antes da Parte 1 (versão 2): o handoff vai na raiz, em `.memory/HANDOFF_AJUSTE-YYYY-MM-DD_P[n].md`, e NÃO se cria `.memory/TASKS/`. SE `.memory/TASKS/` já existir na hora do `git mv`: mover `docs/TASKS/` arquivo por arquivo. Na Parte 0 (somente leitura), SE o chat precisar parar: pedir autorização para criar `.memory/00_INDEX.md` (`parte_atual: 0`) e salvar a auditoria parcial, sem commit.

## Parte 0 — Auditoria (somente leitura)

1. Versão: comparar `versao_start` de `.memory/00_INDEX.md` com `.start/VERSION`. Sem `.memory/` e com `docs/PROJECT_GUARDRAILS.md` ou `docs/PROJECT_CONTEXT.md`: versão 2. Ler em `.start/CHANGELOG-START.md` o que mudou entre as versões.
2. Conferir:
   - Git existe? Há mudanças não commitadas?
   - `.memory/00_INDEX.md` existe e aponta última e próxima unidade;
   - `.memory/PROJECT_GUARDRAILS.md` está no formato GF/GP, com todos os blocos `ANSWERED` e `status: APPROVED`;
   - `.memory/DECISIONS.md` está no formato de entradas `## DEC-xxx`, todas com `status`;
   - `.memory/TASKS/INDEX.md` bate com os arquivos; toda unidade `DONE` tem handoff e commit;
   - Sprints e Builds fechadas batem com suas unidades;
   - `AGENTS.md` tem o bloco `.start` da versão atual; os ponteiros de GP-11 existem;
   - restos: `_ACTIVE_TASK.lock` órfão, `.memory/_rascunho/` não apagado.
3. Mostrar ao usuário neste formato:

```text
AUDITORIA — [projeto]
VERSÃO: [encontrada] → [versão do kit]

O QUE ENCONTREI
- [fato] — [caminho]

O QUE PRECISA AJUSTAR
- [item] — [por quê]

PLANO EM PARTES
Parte 1 — Migração de estrutura ([n] arquivos a mover)   (só versão 2)
Parte 2 — Conversão de formatos ([n] arquivos)
Parte 3 — Guardrails ([n] perguntas)
Parte 4 — Correções de consistência ([n] itens, um por vez)
Parte 5 — Fechamento

Posso começar?
```

## Preparação (depois do "sim")

1. SE o projeto não tem Git: avisar e rodar `git init` (GF-11). Propor um commit de base com os arquivos atuais, SEM os arquivos de segredo (listados à parte) e SEM `.start/` (que entra no commit do ajuste). Pedir autorização. SE o usuário recusar: a Parte 1 move com `mv` comum, com autorização, e o histórico começa no commit do ajuste.
2. `.memory/00_INDEX.md`:
   - SE não existe: criar com `modo_atual: AJUSTAR`, `parte_atual: 1`, `versao_start` igual à versão encontrada (`2.x` para a versão 2), última unidade = a última do `TASKS/INDEX.md` antigo (ou `nenhuma`) e próxima unidade = `AJUSTE-YYYY-MM-DD`;
   - SE existe: trocar só `modo_atual: AJUSTAR` e `parte_atual: 1`.
3. Salvar a auditoria em `.memory/DIAGNOSTICO-AJUSTE-YYYY-MM-DD.md` (`.start/TEMPLATES/TEMPLATE_DIAGNOSTICO.md`).
4. `AGENTS.md`:
   - SE não existe: criar a partir de `.start/TEMPLATES/TEMPLATE_AGENTS.md`;
   - SE existe com o bloco `.start` (linha que começa com `<!-- .start:inicio`): trocar só o bloco, até `<!-- .start:fim -->`;
   - SE existe sem o bloco: inserir no topo, com autorização (como `.start/MODO-ADOTAR.md`, Preparação, item 4).
5. SE existe `CLAUDE.md` sem `@AGENTS.md`: como `.start/MODO-ADOTAR.md`, Preparação, item 5. Ponteiro da ferramenta atual, SE ela precisar e ainda não existir.

## Parte 1 — Migração da versão 2 (`docs/` → `.memory/`)

1. `parte_atual: 1`. Pular esta parte SE a versão encontrada não for 2.
2. Mover somente os arquivos do processo listados em `.start/CHANGELOG-START.md`, versão 3.0.0. Outros arquivos de `docs/` ficam onde estão.
3. Mostrar numa única lista, e pedir uma única autorização (DEC):
   - os movimentos `origem → destino`;
   - as conversões da Parte 2;
   - o arquivamento do guardrails antigo em `.memory/_arquivo/` e a reescrita dele (Parte 3).
4. Mover com `git mv` (ou `mv`, se o usuário recusou o commit de base).
5. Trocar, dentro dos arquivos movidos, os caminhos `docs/` → `.memory/` e os links relativos quebrados (exceção de migração, `.start/FORMATO-LLM.md` §5).

## Parte 2 — Conversão de formatos

1. `parte_atual: 2`. SE a Parte 1 não rodou (ajuste de versão 3.x): mostrar a lista de conversões e pedir autorização (DEC).
2. Converter primeiro o `DECISIONS.md`: tabela → entradas `## DEC-xxx`, com o cabeçalho do template atual, mantendo IDs, datas, textos e ADRs. Status: `Active` → `ACTIVE`; `Superseded` → `SUPERSEDED`; `Rejected` → `REJECTED`; `Proposed` → perguntar ao usuário. Campos sem origem na versão 2: `— (não registrado na v2)`; `decidido_por: usuário (v2)`.
3. Registrar agora as DECs pendentes da Preparação e da Parte 1 (número = maior DEC existente + 1).
4. Converter o resto (conteúdo antigo preservado):
   - `TASKS/INDEX.md`: reescrever no template atual, preservando cada linha da versão 2 (ID, objetivo, estado, handoff); `Tipo` e `Commit principal` = `—` quando não registrados; acrescentar a linha `AJUSTE-YYYY-MM-DD`. NUNCA inventar linhas de unidades que não existiram.
   - `PROJECT_CONTEXT.md` e `DELIVERY.md`: são fotografias; reescrever no template atual, sem perder informação.
   - `TASKS/PENDENCIAS_FUTURAS.md`: acrescentar o cabeçalho do template; itens antigos ficam como estão.
5. Manter como histórico, sem alterar o que já existe (anexar é permitido): handoffs, Releases, PRD, SDD, `BUILDS/`, `SPRINTS/`, arquivos de Task e `CHANGELOG.md`.

## Parte 3 — Guardrails

1. `parte_atual: 3`.
2. Mapear cada regra do guardrails antigo:

| Regra da versão 2 | No kit atual | O que fazer |
| :--- | :--- | :--- |
| uma Task por chat | GP-01 | sugerir `Task` |
| Sprint obrigatória | PROTOCOLO | nada |
| preservar escopo e trabalho existente | GF-02, GF-08 | nada |
| testes proporcionais | GF-09 | nada |
| Git local e commits documentados | GF-11, GP-06 | sugerir `automatico_ao_concluir` |
| ações destrutivas ou externas exigem autorização | GF-07 | nada |
| não persistir segredos | GF-03 | nada |
| declarar só estados comprovados | GF-04 | nada |
| subagentes só quando houver benefício | GF-12, GP-02 | perguntar |
| push não autorizado por padrão | GP-07 | sugerir `pedir_sempre` |
| commits com Task ID | GP-09 | sugerir o formato atual |
| branch principal local | GP-08 | sugerir `principal` |
| regra adicional do usuário | GX | perguntar se continua |

3. Seguir `.start/QUESTIONARIO-GUARDRAILS.md` (regras 3 e 6: o arquivo antigo vai para `.memory/_arquivo/`, com a autorização da Parte 1 ou da Parte 2; as 7 perguntas usam as sugestões acima).
4. Aprovação como no questionário.

## Parte 4 — Correções de consistência

1. `parte_atual: 4`.
2. Uma correção por vez, com pergunta no formato de GUARDRAILS §3.3; cada resposta vira DEC. Exemplo: "A Task BUILD001_S01_T02 está DONE sem handoff. A) Criar handoff a partir do commit abc123 B) Voltar a Task para READY".

## Parte 5 — Fechamento

1. Anexar ao fim de `.memory/CHANGELOG.md` uma seção própria, `## Processo .start [versão] — YYYY-MM-DD`, com a lista de caminhos trocados na migração. SE o arquivo não tem cabeçalho: acrescentar o cabeçalho do template no topo, sem alterar o resto.
2. Definir a próxima unidade: a que estava em andamento antes do ajuste ou, SE não houver, perguntar (D-01; DEC). Atualizar `.memory/TASKS/INDEX.md`.
3. Seguir `.start/PROTOCOLO_MESTRE_EXECUCAO.md` §8. O passo 7 grava `modo_atual: CONTINUAR`, `parte_atual: —` e `versao_start` igual a `.start/VERSION`.
4. PARE.
