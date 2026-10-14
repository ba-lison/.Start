# MODO ADOTAR — projeto que já existe, sem o processo .start

> propósito: trazer um projeto existente para o processo sem estragar nada
> ler quando: `.start/INICIO.md` indicou ADOTAR e o usuário confirmou
> unidade: `ONBOARDING`

## Retomada

SE este modo foi retomado (`.start/INICIO.md`, regra 1): ir direto à `parte_atual` de `.memory/00_INDEX.md`. NUNCA refazer a Parte 0 nem a Preparação. SE `parte_atual: 0`: completar o diagnóstico parcial salvo, sem refazer o que já está nele.

## Regras deste modo

1. A Parte 0 é SOMENTE LEITURA: nenhum arquivo criado ou alterado.
2. NUNCA mover, renomear, apagar ou reformatar arquivo existente, inclusive `docs/`, `README.md` e instruções de outras ferramentas. Arquivos novos só em `.memory/`; `AGENTS.md`, ponteiros e `.gitignore` só com autorização.
3. NUNCA corrigir código ou documentação durante o diagnóstico. Problema encontrado vira pendência.
4. NUNCA copiar documentação existente para `.memory/`: apontar o caminho.
5. Segredo: identificar pelo nome ou padrão do arquivo, sem abrir o conteúdo; registrar só o caminho e avisar o usuário (GF-03).
6. Commits deste modo: na branch atual; só os arquivos criados ou alterados por este modo, mais `.start/`. NUNCA `git add -A` ou `git add .` (GF-11).
7. SE um arquivo que este modo precisa alterar (`AGENTS.md`, `CLAUDE.md`, `.gitignore`) tem mudança do usuário sem commit: não alterar e perguntar: A) você commita a sua mudança antes; B) pular essa alteração agora e registrar pendência. Registrar a resposta como DEC.
8. Toda autorização pedida neste modo vira DEC (GUARDRAILS §3.4, item 3).

## Parte 0 — Diagnóstico (somente leitura)

Inspecionar e anotar, sempre com o caminho de origem:

1. estrutura: pastas até 2 níveis e tamanho aproximado;
2. stack: manifestos (`package.json`, `*.csproj`, `*.sln`, `pyproject.toml`, `requirements.txt`, `Cargo.toml`, `go.mod`, `pom.xml`, `build.gradle` e equivalentes);
3. comandos reais: scripts de setup, lint, teste, build e pacote;
4. testes: onde estão, como rodam e se há cobertura visível;
5. CI/CD: `.github/workflows/`, `.gitlab-ci.yml`, `azure-pipelines.yml` e equivalentes;
6. Git: existe? Branch atual, `HEAD`, remotos, quantidade de mudanças não commitadas (sem conteúdo), últimos 5 commits e tags;
7. documentação: `README.md`, `docs/`, ADRs e changelog próprios;
8. instruções de agente: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.cursor/rules/`, `.agents/rules/`, `GEMINI.md`, `.github/copilot-instructions.md`, `.windsurfrules` — cada regra encontrada, em 1 linha, e conflitos com os GF;
9. riscos: arquivo de segredo aparente, versionado ou não (pelo nome: `.env`, `*.pem`, `*.key`, `id_rsa`, `credentials*`); áreas sem teste; arquivos muito grandes; quantidade de TODO/FIXME;
10. decisões embutidas: linguagem, framework, banco, arquitetura e padrões visíveis, cada uma com a evidência;
11. versão: `VERSION`, tags e versão nos manifestos.

Mostrar ao usuário exatamente neste formato:

```text
DIAGNÓSTICO — [projeto]

O QUE ENCONTREI
- [fato] — [caminho]

O QUE FALTA PARA TRABALHAR NO PROCESSO .start
- [lacuna]

PLANO EM PARTES
Parte 1 — Guardrails (7 perguntas)
Parte 2 — Contexto do projeto (até 7 perguntas)
Parte 3 — Decisões que já estão no código ([n] temas para confirmar)
Parte 4 — Estado atual e pendências ([n] perguntas)
Parte 5 — Fechamento e próxima unidade

Para começar, vou criar a pasta .memory/ (nova, não mexe em nada existente),
salvar este diagnóstico em .memory/DIAGNOSTICO.md e [criar o AGENTS.md | propor um bloco no seu AGENTS.md].
Posso começar pela Parte 1?
```

SE o chat precisar parar ainda na Parte 0: pedir autorização para criar `.memory/00_INDEX.md` (`modo_atual: ADOTAR`, `parte_atual: 0`) e salvar o diagnóstico parcial em `.memory/DIAGNOSTICO.md`, sem commit (o Git ainda não foi tratado).

## Preparação (depois do "sim")

1. Criar `.memory/` como em `.start/MODO-NOVO.md`, Parte 0, item 1, com `modo_atual: ADOTAR` e `parte_atual: 1`.
2. Salvar `.memory/DIAGNOSTICO.md` (`.start/TEMPLATES/TEMPLATE_DIAGNOSTICO.md`).
3. Registrar como DEC a autorização para começar.
4. `AGENTS.md`:
   - SE não existe: criar a partir de `.start/TEMPLATES/TEMPLATE_AGENTS.md`;
   - SE existe: mostrar o bloco `.start` (de `.start/TEMPLATES/TEMPLATE_AGENTS.md`) que será inserido no topo e pedir autorização (GF-07; DEC). Não mexer no resto do arquivo.
5. SE existe `CLAUDE.md`: o Claude Code não carrega o `AGENTS.md` quando há `CLAUDE.md`. Pedir autorização para inserir `@AGENTS.md` na primeira linha (`.start/TEMPLATES/TEMPLATE_PONTEIROS.md`; DEC).
6. Ponteiro da ferramenta atual, SE ela precisar e ainda não existir.
7. SE o projeto não tem Git: avisar e rodar `git init` (GF-11). Em seguida, propor um commit de base com os arquivos atuais do projeto, SEM os arquivos de segredo (listados à parte). Pedir autorização (DEC). SE o usuário recusar: os commits do processo incluem só os arquivos do processo, e os arquivos do usuário ficam fora do Git até ele decidir.

## Parte 1 — Guardrails

Seguir `.start/QUESTIONARIO-GUARDRAILS.md`, mostrando as sugestões do diagnóstico. Exemplos: branch por Task se o projeto usa PR; proteger os arquivos de segredo e as partes frágeis (GP-10); na pergunta 7, as regras encontradas em instruções existentes (GF-05).

## Parte 2 — Contexto do projeto

1. `parte_atual: 2`.
2. Escrever um rascunho de `.memory/PROJECT_CONTEXT.md` com base no diagnóstico, marcando toda dedução com `hipótese:`.
3. Seguir `.start/FASE-01-BRAINSTORM-E-MEMORIA.md` (perguntas 1 a 6), partindo do rascunho: confirmar o que já foi deduzido e perguntar o que falta.
4. Perguntar também, como levantamento: o que não pode quebrar?
5. Remover a marca `hipótese:` só do que o usuário confirmou.

## Parte 3 — Decisões que já estão no código

1. `parte_atual: 3`.
2. Registrar cada decisão embutida em `.memory/DECISIONS.md` com `status: INFERRED` e `decidido_por: código existente (a confirmar)`.
3. Confirmar por tema, uma pergunta por vez (stack, dados, arquitetura, integrações, convenções). Exemplo: "O código usa Node 20 + TypeScript + PostgreSQL. Confirmo as 3 como decisões ativas? A) Sim B) Alguma mudou".
4. Confirmada: trocar na mesma entrada `status: ACTIVE` e `decidido_por: usuário (confirmou decisão herdada em YYYY-MM-DD)` (GUARDRAILS §3.4, item 6). Recusada: `status: REJECTED` e, se houver, nova decisão.

## Parte 4 — Estado atual e pendências

1. `parte_atual: 4`.
2. Perguntar, uma por vez: o que está em andamento; bugs conhecidos; o que é prioridade (D-01; DEC); o que o usuário quer entregar a seguir.
3. Registrar em `.memory/TASKS/PENDENCIAS_FUTURAS.md` e em `.memory/PROJECT_CONTEXT.md`.
4. SE o projeto já tem changelog ou ADRs próprios: perguntar qual é o canônico (D-02; DEC). A) O do projeto: `.memory/CHANGELOG.md` e `.memory/adrs/` só apontam para ele. B) O da `.memory/`: o do projeto fica como histórico.
5. SE há arquivo de segredo versionado: registrar PEND e perguntar (D-05; DEC). A) Tirar do Git (`git rm --cached`), incluir no `.gitignore` e trocar as chaves expostas. B) Manter e aceitar o risco. Recomendar A.
6. SE houver comandos reais confirmados: criar `.memory/DELIVERY.md` só com eles. O resto é completado na Fase 03.
7. Divergências encontradas no diagnóstico (GUARDRAILS §8): uma por vez, com pergunta e DEC.

## Parte 5 — Fechamento da unidade ONBOARDING

1. Perguntar a próxima unidade (D-01; DEC): `PRD-001` para uma entrega nova, ou `PLANO-BUILD001` se o escopo da próxima entrega já estiver claro — nesse caso, registrar o escopo aprovado numa DEC, que substitui o PRD (`.start/FASE-03-SDD-E-DECOMPOSICAO.md`). Acrescentar a linha em `.memory/TASKS/INDEX.md`.
2. Seguir `.start/PROTOCOLO_MESTRE_EXECUCAO.md` §8. O passo 7 grava `modo_atual: CONTINUAR`, `parte_atual: —` e `versao_start`.
3. PARE.

## SE o chat pesar

Projeto grande pode não caber num chat. Seguir GF-10: handoff `PAUSED` com a `parte_atual` registrada; o próximo chat retoma na mesma parte.
