# FORMATO PARA LEITURA POR LLM

> propósito: como escrever os arquivos de `.memory/` e do kit para que qualquer LLM leia sem ambiguidade
> ler quando: antes de criar ou editar arquivo em `.memory/`
> vale para: Claude Code, Codex, Cursor, OpenCode, Antigravity e qualquer outro agente; não existe versão por ferramenta

## 1. Estrutura

1. Um assunto por arquivo. O nome do arquivo diz o conteúdo.
2. Todo arquivo começa com título `#` e um cabeçalho em linhas `> chave: valor`:
   - `propósito:` o que o arquivo guarda;
   - `ler quando:` em que momento ler;
   - `atualizar quando:` em que momento escrever;
   - `status:` ou `estado:`, quando houver.
3. `.memory/00_INDEX.md` lista todos os arquivos ativos com "ler quando". O agente carrega só o que a unidade precisa.
4. `AGENTS.md` fica com no máximo 12.000 caracteres (limite de arquivo de regras do Antigravity). O conteúdo detalhado fica em `.start/` e `.memory/`.
5. `00_INDEX.md` fica curto: estado, última unidade, próxima unidade e mapa.
6. Ao criar um arquivo a partir de template com a seção "Modelo de entrada": o arquivo nasce com tudo que vem antes dessa seção. O modelo não é copiado.

## 2. Escrita

1. Frases curtas e imperativas. Regras com DEVE, NUNCA ou PODE em maiúsculas.
2. Números explícitos: "no máximo 3", nunca "poucos" ou "quando fizer sentido".
3. Procedimento = passos numerados. Condição = `SE ...:` ou `SE ... →`. Parada = `PARE`.
4. IDs estáveis e nunca reutilizados: GF-xx, GP-xx, GX-xx, D-xx, D-Pxx (item de decisão do projeto), DEC-xxx, PEND-xxx, ADR-xxxx, RF-xxx, RNF-xxx, BUILDxxx, BUILDxxx_Syy, BUILDxxx_Syy_Tzz. Novo ID = maior existente + 1.
5. Registros em listas `chave: valor`. Tabela só para dados curtos: poucas colunas e poucas palavras por célula.
6. Caminhos relativos à raiz do projeto, exatos e entre crases: `.memory/DECISIONS.md`.
7. Datas no formato `YYYY-MM-DD`.
8. Um idioma para o texto (GP-12). Estados (de unidade, Sprint, Build, Release, decisão, pendência e progresso) sempre em inglês maiúsculo: `DONE`, `BLOCKED`, `APPROVED`, `PENDING`. Nomes de modo em português: `NOVO`, `ADOTAR`, `AJUSTAR`, `CONTINUAR`.
9. Sem emoji, sem HTML e sem diagrama como única fonte de uma informação. Única exceção: os marcadores do bloco `.start` no `AGENTS.md` — o bloco começa na linha que começa com `<!-- .start:inicio` e termina na linha `<!-- .start:fim -->`.
10. Fato e hipótese separados: hipótese começa com `hipótese:`.

## 3. O que é relevante (vai para `.memory/`)

| Informação | Arquivo |
| :--- | :--- |
| decisão | `DECISIONS.md` (+ ADR quando duradoura) |
| regra do projeto | `PROJECT_GUARDRAILS.md` |
| fotografia atual | `PROJECT_CONTEXT.md` |
| requisito, arquitetura, plano | `prd/`, `sdd/`, `BUILDS/`, `SPRINTS/`, `TASKS/` |
| comandos reais e entrega | `DELIVERY.md` |
| o que foi feito num chat + evidência resumida | `TASKS/HANDOFF_[UNIDADE].md` (pausa: `_P[n]`) |
| tentativa fracassada e bloqueio | `TASKS/BLOQUEIO_[UNIDADE].md` |
| mudança entregue | `CHANGELOG.md` |
| pendência | `TASKS/PENDENCIAS_FUTURAS.md` |
| estado e próxima unidade | `00_INDEX.md` |

## 4. O que nunca vai para `.memory/`

- segredo, senha, token, chave ou dado pessoal (GF-03);
- log bruto: registrar o comando, o resultado resumido e onde o log está;
- cópia de código: apontar o arquivo;
- cópia de documentação existente do projeto: apontar o caminho;
- conversa do chat;
- hipótese apresentada como fato.

## 5. Sobrescrever ou anexar

- fotografia (sobrescrever; o Git guarda o histórico): `00_INDEX.md`, `PROJECT_CONTEXT.md`, `DELIVERY.md`, `TASKS/INDEX.md`;
- histórico (só anexar; NUNCA apagar): `DECISIONS.md`, `CHANGELOG.md`, `TASKS/PENDENCIAS_FUTURAS.md` (item resolvido muda de estado, não some);
- imutável depois de fechado: handoffs, bloqueios, `DIAGNOSTICO*.md`, ADR aceito (outro ADR o substitui), registros de Release. Erro factual é corrigido só por uma seção anexada ao fim, `## Correção YYYY-MM-DD`, com o motivo; o texto original não é editado. Exceção: o bloqueio recebe a seção "Resposta" quando o usuário decide;
- temporário: `.memory/_rascunho/` (apagado antes do commit de fechamento).

Exceção de migração: trocar caminho `docs/` → `.memory/`, ou corrigir link relativo quebrado pela migração, é permitido em qualquer arquivo, inclusive imutável. A troca é registrada no `CHANGELOG.md` do projeto.

## 6. Arquivamento

Quando um arquivo ativo acumular conteúdo que não serve mais à próxima unidade: mover esse trecho para `.memory/_arquivo/[arquivo]-[YYYY-MM-DD].md` e deixar no original uma linha apontando para ele. Mover exige autorização (GF-02).
