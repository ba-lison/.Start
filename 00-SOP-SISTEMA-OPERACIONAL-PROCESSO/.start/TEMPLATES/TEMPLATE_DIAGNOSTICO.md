# DIAGNÓSTICO — [PROJETO]

> propósito: fotografia do projeto no momento em que o processo .start foi adotado ou ajustado
> destino: `.memory/DIAGNOSTICO.md` (ADOTAR) ou `.memory/DIAGNOSTICO-AJUSTE-YYYY-MM-DD.md` (AJUSTAR)
> ler quando: durante o onboarding ou o ajuste; depois, só se uma unidade citar
> atualizar quando: só para completar um diagnóstico parcial (pausa na Parte 0); imutável depois do fechamento do ONBOARDING ou do AJUSTE. O progresso fica em `parte_atual` de `.memory/00_INDEX.md`
> modo: ADOTAR | AJUSTAR
> data: YYYY-MM-DD
> commit_inspecionado: `[hash | sem git]`

## O que encontrei

### Estrutura e stack

- [fato] — `[caminho]`

### Como roda e como verifica

- setup: `[comando]` — fonte: `[arquivo]`
- testes: `[comando]` — fonte: `[arquivo]`
- build: `[comando]` — fonte: `[arquivo]`
- CI: [workflow | nenhum] — `[caminho]`

### Git

- existe: [sim | não]
- branch atual:
- `HEAD`:
- remotos:
- mudanças não commitadas: [quantidade de arquivos, sem conteúdo]
- últimos commits: [5 linhas curtas]
- tags:

### Documentação existente

- `[caminho]` — [do que trata]
- changelog ou ADRs próprios: [não | `caminho`]

### Instruções de agente existentes

Cada regra em 1 linha. Na pergunta 7 do questionário o usuário decide se cada uma vira GX ou fica só como referência (GF-05).

- `AGENTS.md`: [não existe | existe — regras: ...]
- `CLAUDE.md`: [não existe | existe — regras: ...]
- outras (`.cursorrules`, `.cursor/rules/`, `.agents/rules/`, `GEMINI.md`, `.github/copilot-instructions.md`): [lista]
- conflitos com os guardrails fixos: [nenhum | lista]

### Riscos visíveis

- arquivo de segredo aparente (identificado pelo nome, sem abrir): [nenhum | caminhos — versionado: sim/não]
- áreas sem teste ou frágeis: [lista]
- TODO/FIXME: [quantidade]

### Decisões embutidas no código

Cada uma vira entrada `INFERRED` em `.memory/DECISIONS.md` para confirmação.

- [tema]: [o que o código mostra] — evidência: `[arquivo]`

### Versão anterior do processo (só AJUSTAR)

- versão encontrada: [2.x com `docs/` | 3.x]
- mudanças a aplicar (de `.start/CHANGELOG-START.md`): [lista]

## O que falta para trabalhar no processo .start

- [lacuna]

## Plano em partes

- Parte 1 — [nome] — [n] perguntas
- Parte 2 — [nome] — [n] perguntas
