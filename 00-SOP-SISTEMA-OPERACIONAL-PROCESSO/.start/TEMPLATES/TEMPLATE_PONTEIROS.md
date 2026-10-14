# PONTEIROS POR FERRAMENTA

> propósito: arquivos de uma linha para ferramentas que não carregam sozinhas o `AGENTS.md` da raiz
> ler quando: Preparação de todo modo (ferramenta atual); aprovação dos guardrails (demais ferramentas de GP-11)
> regra: ponteiro é UMA linha e NUNCA contém regras; todo o conteúdo fica no `AGENTS.md`
> verificado em: 2026-09-22 — ferramentas mudam; na dúvida, confirme na documentação da ferramenta

## Quem precisa de ponteiro

| Ferramenta | Carrega o `AGENTS.md` da raiz? | Ponteiro |
| :--- | :--- | :--- |
| Codex | sim | nenhum |
| Cursor | sim | nenhum |
| OpenCode | sim | nenhum |
| Claude Code 2.1.277 ou mais novo, sem `CLAUDE.md` no projeto | sim | nenhum |
| Claude Code com `CLAUDE.md` já existente, versão anterior, opção desligada em `/config`, ou via Bedrock/Vertex/telemetria desligada | não | `CLAUDE.md` |
| Antigravity | a documentação oficial cita `.agents/rules/` | `.agents/rules/start.md` |
| outra | ver a documentação | o arquivo que ela carrega, com o ponteiro genérico |

Na dúvida sobre versão ou configuração do Claude Code: criar o `CLAUDE.md` de ponteiro. Ele funciona em todas as versões.

## Quando criar

1. Na Preparação de qualquer modo: o ponteiro da ferramenta atual, SE ela precisar.
2. Na aprovação dos guardrails: os ponteiros das outras ferramentas de GP-11 que ainda não existem.
3. SE o arquivo do ponteiro já existe: pedir autorização uma única vez (GF-07) e registrar DEC.

## Conteúdo exato

Claude Code — `CLAUDE.md` novo, com exatamente esta linha:

```text
@AGENTS.md
```

Claude Code — `CLAUDE.md` que já existe: inserir `@AGENTS.md` como primeira linha, sem alterar o resto.

Antigravity — `.agents/rules/start.md`, com exatamente esta linha (SE a versão em uso já carrega o `AGENTS.md` da raiz, é dispensável):

```text
@/AGENTS.md
```

Outra ferramenta — no arquivo que ela carrega, esta linha (sintaxe de import `@` não é universal):

```text
Leia AGENTS.md na raiz do projeto e siga.
```

## Fontes

- Claude Code e `AGENTS.md` (versão 2.1.277, 2026-09-18): https://www.infoworld.com/article/4224410/claude-code-now-also-accepts-instructions-in-openais-agents-md-format.html
- Antigravity, regras e referências `@`: https://antigravity.google/docs/rules-workflows/
- Cursor: https://cursor.com/docs/rules
- OpenCode: https://opencode.ai/docs/rules/
- Padrão `AGENTS.md`: https://agents.md/
