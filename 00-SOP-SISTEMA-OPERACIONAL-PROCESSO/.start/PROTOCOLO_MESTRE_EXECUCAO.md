# PROTOCOLO MESTRE DE EXECUÇÃO

> propósito: como um projeto anda do início à entrega no processo .start
> ler quando: início do projeto; dúvida sobre fluxo, unidade, estado ou fechamento
> versao_kit: 3.0.0

## 1. Princípios

1. O repositório é a fonte de verdade; o chat nunca é a única memória (GF-01).
2. O usuário decide; o agente pergunta, recomenda e registra (GF-06).
3. Estado atual, decisões, eventos, evidências e Releases ficam separados (`.start/FORMATO-LLM.md` §5).
4. Um chat executa uma unidade de trabalho e termina com handoff (GF-10).
5. Toda unidade tem objetivo, limites e verificação proporcional ao risco.
6. Todo projeto tem Git local; toda unidade concluída gera commit local (GF-11).
7. Ações externas, destrutivas ou de risco exigem autorização (GF-07).
8. Só se declara estado comprovado (GF-04).
9. Nenhum trabalho no produto antes de `.memory/PROJECT_GUARDRAILS.md` estar `APPROVED`.

## 2. Arquivos do processo

No projeto:

```text
AGENTS.md    entrada única; a ferramenta carrega a cada chat
.start/      kit do processo; somente leitura; versionado com o projeto; atualizado trocando a pasta inteira
.memory/     memória do projeto; escrita somente pelo orquestrador
VERSION      versão do produto
```

No kit (`.start/`):

- `INICIO.md` — roteador de modos; primeiro arquivo de todo chat
- `GUARDRAILS-E-CIRCUIT-BREAKERS.md` — GF, decisão, GP, rodadas, Git, parada
- `QUESTIONARIO-GUARDRAILS.md` — as 7 perguntas de guardrails
- `CONTRATO-SUBAGENTE.md` — regras e bloco obrigatório para subagentes
- `FORMATO-LLM.md` — como escrever na `.memory/`
- `MODO-NOVO.md`, `MODO-ADOTAR.md`, `MODO-AJUSTAR.md`, `MODO-CONTINUAR.md`
- `FASE-01` a `FASE-05` — procedimento de cada fase
- `TEMPLATES/` — formatos mínimos (índice em `TEMPLATES/INDEX.md`)
- `CHANGELOG-START.md` — mudanças do kit e instruções de migração
- `VERSION` — versão do kit

## 3. Perfis

| Perfil | Uso | Controles |
| :--- | :--- | :--- |
| `SIMPLE` | projeto pessoal ou local | histórico, testes proporcionais, pacote e hash quando houver entrega |
| `STANDARD` | produto compartilhado ou publicado | CI, review, artefato retido, segurança e homologação |
| `CRITICAL` | produção, dados sensíveis, alto impacto | segregação, aprovações, proveniência, observabilidade e recuperação ensaiada |

O perfil é decisão do usuário (Fase 01) e fica em `.memory/PROJECT_CONTEXT.md`. Recomendar `SIMPLE` salvo risco evidente.

## 4. Estrutura da `.memory/`

```text
.memory/
├── 00_INDEX.md               entrada: estado, última e próxima unidade, mapa
├── PROJECT_GUARDRAILS.md     regras aprovadas do projeto
├── PROJECT_CONTEXT.md        fotografia atual
├── DECISIONS.md              decisões (só anexar)
├── CHANGELOG.md              mudanças entregues
├── DELIVERY.md               comandos reais, Git, versão e entrega
├── DIAGNOSTICO*.md           só em ADOTAR e AJUSTAR (histórico, não muda)
├── prd/  sdd/  adrs/
├── BUILDS/  SPRINTS/
├── TASKS/
│   ├── INDEX.md              todas as unidades e seus estados
│   ├── PENDENCIAS_FUTURAS.md
│   ├── BUILDxxx_Syy_Tzz.md   definição de cada Task
│   ├── HANDOFF_[UNIDADE].md  handoff final da unidade
│   ├── HANDOFF_[UNIDADE]_P[n].md   handoff de cada pausa (PAUSED)
│   └── BLOQUEIO_[UNIDADE].md
├── RELEASES/
├── _rascunho/                temporário de subagentes; apagado no fechamento
└── _arquivo/                 trechos arquivados
```

## 5. Unidades de trabalho e fluxo

Cada unidade ocupa um chat (GF-10).

| Unidade | ID | Procedimento |
| :--- | :--- | :--- |
| Onboarding: modo, guardrails e contexto | `ONBOARDING` | `MODO-NOVO` ou `MODO-ADOTAR` + `FASE-01` |
| PRD | `PRD-xxx` | `FASE-02` |
| Plano: SDD, DELIVERY, Build, Sprints e Tasks | `PLANO-BUILDxxx` | `FASE-03` |
| Task (GP-01 = `Task`) | `BUILDxxx_Syy_Tzz` | `FASE-04` |
| Fechamento de Sprint (GP-01 = `Task`) | `BUILDxxx_Syy_CLOSE` | `FASE-04` |
| Sprint inteira (GP-01 = `Sprint`) | `BUILDxxx_Syy` | `FASE-04`, seção "Sprint por chat" |
| Fechamento de Build e Release | `BUILDxxx_CLOSE` | `FASE-05` |
| Ajuste do processo | `AJUSTE-YYYY-MM-DD` | `MODO-AJUSTAR` |

Fluxo:

```text
ONBOARDING → PRD-001 → PLANO-BUILD001 → BUILD001_S01_T01 … → BUILD001_S01_CLOSE
→ (próximas Sprints) → BUILD001_CLOSE → próxima Build ou encerramento
```

Hierarquia: Projeto → Build → Sprint → Task → Handoff.

- Build: marco completo de entrega. Toda Build tem pelo menos uma Sprint.
- Sprint: incremento funcional verificável. Toda Sprint termina com fechamento (`BUILDxxx_Syy_CLOSE`, ou o último passo do chat da Sprint quando GP-01 = `Sprint`).
- Task: trabalho atômico. Toda Task pertence a exatamente uma Sprint.
- Build, Sprint e versão pública não são sinônimos.

## 6. Início de todo chat

Seguir `.start/INICIO.md`. Em modo CONTINUAR, a ordem de leitura está em `.start/MODO-CONTINUAR.md`.

## 7. Ciclo de uma unidade

1. confirmar com o usuário: unidade, escopo, fora de escopo, verificação e autorizações;
2. inspecionar Git e workspace; proteger o trabalho existente;
3. SE houver decisão pendente para começar: perguntar antes (GF-06);
4. executar somente o escopo (rodadas: GUARDRAILS §5; subagentes: `.start/CONTRATO-SUBAGENTE.md`);
5. verificar conforme o risco (GUARDRAILS §6);
6. revisar o diff e classificar pendências;
7. fechar (seção 8).

## 8. Fechamento de toda unidade

Nesta ordem:

1. conferir se toda decisão do chat está em `.memory/DECISIONS.md`;
2. atualizar o conteúdo da unidade: `.memory/CHANGELOG.md` SE houve mudança entregue; `.memory/PROJECT_CONTEXT.md` SE a fotografia mudou; pendências em `.memory/TASKS/PENDENCIAS_FUTURAS.md`;
3. remover o lock, se houver;
4. commit principal (GP-06): implementação e testes, ou os documentos da unidade. No ONBOARDING e no AJUSTE, inclui `.start/`, `AGENTS.md` e ponteiros;
5. criar o handoff com o hash do commit principal e o essencial de `.memory/_rascunho/` (`.start/TEMPLATES/TEMPLATE_HANDOFF.md`): `.memory/TASKS/HANDOFF_[UNIDADE].md`; SE `PAUSED`: `.memory/TASKS/HANDOFF_[UNIDADE]_P[n].md`, com n = 1, 2, …. Depois, apagar `.memory/_rascunho/`;
6. atualizar `.memory/TASKS/INDEX.md` (estado, commit principal, handoff e a seção "Próxima unidade") e, quando for o caso, o resultado da Sprint ou da Build;
7. atualizar `.memory/00_INDEX.md`: última unidade (com o commit principal e o handoff), próxima unidade, `perfil`, `build_ativa` e `sprint_ativa`. SE a unidade é `ONBOARDING` ou `AJUSTE-*` E fechou `DONE`: também `modo_atual: CONTINUAR`, `parte_atual: —` e `versao_start` igual a `.start/VERSION`. Em `PAUSED` ou `BLOCKED`: `modo_atual` e `parte_atual` NÃO mudam, e a próxima unidade é a mesma;
8. commit local dos registros de fechamento;
9. informar no chat: resultado, commit principal, `HEAD` final, estado do workspace, decisões do chat, próxima unidade e o prompt do próximo chat;
10. PARE. NUNCA iniciar outra unidade neste chat.

Unidade `PAUSED` ou `BLOCKED`: os passos 1 a 10 valem. No passo 4, o commit principal leva o trabalho parcial, com `wip` na mensagem. No passo 7, vale a regra de `PAUSED`/`BLOCKED`. SE o bloqueio é o próprio commit (identidade Git ausente, hook, conflito): pular os passos 4 e 8, gravar `commit_principal: — (commit falhou: [motivo])`, preservar o workspace e perguntar ao usuário.

## 9. Estados

Unidades: `BACKLOG`, `READY`, `RUNNING`, `PAUSED`, `BLOCKED`, `DONE`, `CANCELLED`.

- `PAUSED`: chat encerrado antes do fim porque GP-13 disparou; retoma em novo chat pelo último handoff `_P[n]`.
- `BLOCKED`: depende de decisão, autorização ou correção externa.

Sprints: `PLANNED`, `RUNNING`, `PAUSED`, `BLOCKED`, `DONE`.

Builds e Releases: `PLANNED`, `BUILDING`, `VERIFIED`, `READY_FOR_RELEASE`, `READY_FOR_DEPLOY`, `RELEASED`, `REJECTED`, `ROLLED_BACK`, `HALT_BLOCKED`.

## 10. Critérios de conclusão

- Unidade `DONE`: critérios satisfeitos, verificação registrada, diff explicado, decisões registradas, handoff criado e commit local feito.
- Release `RELEASED`: somente com artefato ou implantação comprovada e validação pós-entrega. Sem acesso ou autorização: `READY_FOR_DEPLOY`.

## 11. Documentos normativos

- GUARDRAILS define limites e vence qualquer outro arquivo do kit em caso de conflito.
- MODOS e FASES definem o procedimento.
- TEMPLATES definem o formato mínimo.
- `.memory/DELIVERY.md` adapta o processo ao projeto.
- Pesquisas fundamentam decisões, mas não mandam.
