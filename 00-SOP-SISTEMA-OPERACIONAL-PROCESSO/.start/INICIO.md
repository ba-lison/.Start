# INÍCIO — roteiro de entrada do processo .start

> propósito: decidir o modo do chat e o primeiro passo
> ler quando: no começo de todo chat, antes de qualquer ação
> versao_kit: ver `.start/VERSION`

## Regra zero

Até a confirmação do Passo 3, NÃO crie nem altere nenhum arquivo.

## Passo 1 — Ler as regras

1. Leia `.start/GUARDRAILS-E-CIRCUIT-BREAKERS.md` (regras fixas GF, o que é decisão, parâmetros GP, rodadas).
2. SE existe `.memory/PROJECT_GUARDRAILS.md`: leia. Os valores dele só valem com `status: APPROVED` (GUARDRAILS §1).
3. SE existe `.memory/00_INDEX.md`: leia.

## Passo 2 — Detectar o modo

Responda olhando o projeto, sem alterar nada:

- M0: existe a pasta `.memory/`?
- M1: existe `.memory/00_INDEX.md`?
- M2: existe `docs/PROJECT_GUARDRAILS.md`, ou `docs/PROJECT_CONTEXT.md` contendo `Perfil:` (versão 2 do processo)?
- M3: existem arquivos ou pastas além de `.start/`, `.git/`, `.gitignore`, `AGENTS.md`, `CLAUDE.md`, `.agents/` e um `README.md` com até 20 linhas e sem bloco de código?
- M4: `versao_start` em `.memory/00_INDEX.md` é igual a `.start/VERSION` E `status` em `.memory/PROJECT_GUARDRAILS.md` é `APPROVED`?

Aplique a primeira regra verdadeira:

1. SE M1 = sim E `modo_atual` em `00_INDEX.md` é `NOVO`, `ADOTAR` ou `AJUSTAR` → retomada desse modo, na `parte_atual`.
2. SE M1 = sim E M4 = sim → CONTINUAR → `.start/MODO-CONTINUAR.md`.
3. SE M1 = sim E M4 = não → AJUSTAR → `.start/MODO-AJUSTAR.md`.
4. SE M0 = sim E M1 = não → onboarding interrompido antes do índice: mostrar ao usuário o que existe em `.memory/` e perguntar se retoma NOVO ou ADOTAR, sem recriar os arquivos que já existem.
5. SE M2 = sim → AJUSTAR (migração da versão 2) → `.start/MODO-AJUSTAR.md`.
6. SE M3 = sim → ADOTAR → `.start/MODO-ADOTAR.md`.
7. Nos demais casos → NOVO → `.start/MODO-NOVO.md`.

## Passo 3 — Confirmar com o usuário

SE o modo é CONTINUAR: pule este passo. A confirmação acontece no Passo 3 de `.start/MODO-CONTINUAR.md`.

Nos outros casos, diga em até 3 linhas o que encontrou e o modo:

```text
Encontrei: [fatos curtos, com caminho].
Modo: [NOVO | ADOTAR | AJUSTAR | retomada de <modo> na Parte <n>] — [o que isso significa em 1 frase].
Confirma?
```

- NUNCA avance sem a confirmação.
- SE o usuário discordar: pergunte qual é o caso e siga o modo escolhido. É decisão (D-11): registre assim que `.memory/DECISIONS.md` existir (GUARDRAILS §3.4, item 9).

## Passo 4 — Seguir o arquivo do modo

- Início normal: siga o arquivo do modo do começo ao fim. Ele diz o que criar, quando perguntar e quando parar.
- Retomada (regra 1): leia o último handoff indicado em `.memory/00_INDEX.md` (se for `PAUSED`, a seção "Se PAUSED: onde parei"), rode as conferências do Passo 2 de `.start/MODO-CONTINUAR.md` (Git e workspace contra o handoff) e vá direto à `parte_atual`. NUNCA refaça a Parte 0 nem a Preparação.

## Primeiro chat de um projeto

Na primeira vez, nenhuma ferramenta carrega o processo sozinha. O usuário abre o chat com:

```text
Leia .start/INICIO.md e siga.
```

O onboarding cria o `AGENTS.md` na raiz. A partir daí a ferramenta carrega as regras a cada chat (ponteiros por ferramenta: `.start/TEMPLATES/TEMPLATE_PONTEIROS.md`).
