# MODO NOVO — projeto do zero

> propósito: iniciar um projeto sem conteúdo próprio no processo .start
> ler quando: `.start/INICIO.md` indicou NOVO e o usuário confirmou
> unidade: `ONBOARDING`

## Retomada

SE este modo foi retomado (`.start/INICIO.md`, regra 1): ir direto à `parte_atual` de `.memory/00_INDEX.md`. NUNCA refazer a Parte 0.

## Parte 0 — Preparar

O projeto não tem conteúdo próprio: criar os arquivos abaixo não afeta nada existente. Só é preciso perguntar onde está indicado.

1. Criar, a partir de `.start/TEMPLATES/` (nome do projeto: o nome da pasta, até a Fase 01 definir outro):
   - `.memory/00_INDEX.md` com `modo_atual: NOVO`, `parte_atual: 1`, `versao_start` igual a `.start/VERSION` e próxima unidade `ONBOARDING`;
   - `.memory/PROJECT_GUARDRAILS.md` com `status: DRAFT`;
   - `.memory/DECISIONS.md` e `.memory/TASKS/PENDENCIAS_FUTURAS.md`, só com título e cabeçalho;
   - `.memory/CHANGELOG.md`;
   - `.memory/TASKS/INDEX.md`, com a unidade `ONBOARDING` em `RUNNING`.
2. `AGENTS.md` na raiz:
   - SE não existe: criar a partir de `.start/TEMPLATES/TEMPLATE_AGENTS.md`;
   - SE já existe: NÃO sobrescrever; seguir `.start/MODO-ADOTAR.md`, Preparação, item 4.
3. Ponteiro da ferramenta atual, SE ela precisar (`.start/TEMPLATES/TEMPLATE_PONTEIROS.md`). SE o arquivo do ponteiro já existe: seguir `.start/MODO-ADOTAR.md`, Preparação, item 5.
4. SE não há Git: avisar e rodar `git init` (GF-11). Assim, uma pausa no meio do onboarding já pode ser commitada.
5. Dizer ao usuário, em 1 linha, o que foi criado.

## Parte 1 — Guardrails

Seguir `.start/QUESTIONARIO-GUARDRAILS.md` (7 perguntas e aprovação).

## Parte 2 — Contexto do projeto

1. Atualizar `parte_atual: 2` em `.memory/00_INDEX.md`.
2. Seguir `.start/FASE-01-BRAINSTORM-E-MEMORIA.md`.

## Parte 3 — Fechamento da unidade ONBOARDING

1. Definir a próxima unidade: `PRD-001`, arquivo `.memory/prd/PRD-001.md` (a criar). Acrescentar a linha em `.memory/TASKS/INDEX.md`.
2. Seguir `.start/PROTOCOLO_MESTRE_EXECUCAO.md` §8. O passo 7 grava `modo_atual: CONTINUAR`, `parte_atual: —` e `versao_start`.
3. PARE.

## NUNCA neste modo

- escrever código do produto;
- escolher stack, arquitetura ou ferramenta sem perguntar;
- pular o questionário ou aprovar os guardrails no lugar do usuário.
