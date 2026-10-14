# Como usar o kit .start num projeto

> Versão do kit: ver `.start/VERSION` (3.0.0). Mudanças e migração: `.start/CHANGELOG-START.md`.

## O que é o kit

A pasta `.start/` (dentro desta pasta) é tudo que vai para dentro de um projeto: roteador de entrada, guardrails, questionário, modos, fases e templates. Ela é lida pelo agente e nunca é editada dentro do projeto.

O agente cria no projeto:

- `AGENTS.md` na raiz — a entrada única, lida pela ferramenta de IA a cada chat;
- `.memory/` — toda a memória do projeto: guardrails aprovados, contexto, decisões, PRD, SDD, Builds, Sprints, Tasks, handoffs e Releases.

## Instalar

1. Copie a pasta `00-SOP-SISTEMA-OPERACIONAL-PROCESSO/.start/` para a raiz do projeto. Fica `seu-projeto/.start/`.
2. Abra um chat novo na ferramenta de IA, com o projeto como pasta de trabalho.
3. Primeira mensagem:

```text
Leia .start/INICIO.md e siga.
```

4. O agente identifica o caso e pede sua confirmação:

| O agente encontra | Modo | O que acontece |
| :--- | :--- | :--- |
| pasta vazia | NOVO | guardrails → contexto → PRD no próximo chat |
| projeto com código, sem `.memory/` | ADOTAR | diagnóstico só de leitura → "o que encontrei / o que falta / plano em partes" → uma pergunta por vez |
| projeto com `docs/` da versão 2, ou `.memory/` antiga ou incompleta | AJUSTAR | auditoria → migração e correções por partes, com sua autorização |
| `.memory/` em dia | CONTINUAR | lê o último handoff e propõe a próxima unidade |

5. Todo onboarding (NOVO e ADOTAR) começa pelo questionário de guardrails: 7 perguntas, uma por vez, cada uma com o padrão e a recomendação. No AJUSTAR, o questionário vem depois da migração dos arquivos.
6. No fim de cada chat o agente entrega o handoff e o prompt do próximo chat. Daí em diante, basta abrir um chat novo e colar o prompt (ou só escrever "Continue").
7. A pasta `.start/` entra no commit do onboarding: o kit fica versionado junto com o projeto.

## Padrões que você pode mudar no questionário

- unidade por chat: Task;
- troca de chat antes do fim: quando a ferramenta compactar o contexto ou o chat passar de 40 mensagens suas;
- até 10 subagentes por chat; subagente nunca cria subagente (fixo);
- relatório de subagente com até 500 caracteres;
- até 3 rodadas por objetivo, contando erro ou acerto e somando agente, subagentes e chats;
- uma pergunta por vez; nenhuma decisão sem você (fixo);
- commit local automático; push, PR e deploy só com sua autorização;
- áreas protegidas, idioma e regras extras.

## Atualizar o kit num projeto

Substitua a pasta `.start/` do projeto pela versão nova e diga `Leia .start/INICIO.md e siga.` O agente entra em AJUSTAR, mostra o que mudou e commita a troca do kit.

## O que NÃO copiar

Não copie a pasta `.Start` inteira. As pesquisas confundiriam o agente, que poderia tratá-las como regra do projeto.

## Ferramentas

Todo o conteúdo fica no `AGENTS.md` e na `.memory/`; não existe versão de regras por ferramenta. Codex, Cursor e OpenCode leem o `AGENTS.md` direto. Claude Code (2.1.277 ou mais novo) também, quando não há `CLAUDE.md`; nos outros casos, e no Antigravity, o agente cria um ponteiro de uma linha. Detalhes em `.start/TEMPLATES/TEMPLATE_PONTEIROS.md`.

## Onde está cada coisa

- roteador: `.start/INICIO.md`
- regras: `.start/GUARDRAILS-E-CIRCUIT-BREAKERS.md`
- questionário: `.start/QUESTIONARIO-GUARDRAILS.md`
- subagentes: `.start/CONTRATO-SUBAGENTE.md`
- formato dos arquivos: `.start/FORMATO-LLM.md`
- fluxo completo: `.start/PROTOCOLO_MESTRE_EXECUCAO.md`
- exemplo preenchido: `05-exemplo-projeto-simple/` (na base de conhecimento)
