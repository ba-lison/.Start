# Exemplo mínimo — CLI de saudação

Exemplo documental de um projeto `SIMPLE` no processo .start 3.0.0. Não contém implementação; mostra como a memória se conecta, da primeira conversa à Release. Os hashes de commit são ilustrativos.

Fluxo demonstrado (uma unidade por chat):

```text
ONBOARDING → PRD-001 → PLANO-BUILD001 → BUILD001_S01_T01
→ BUILD001_S01_CLOSE → BUILD001_CLOSE → v0.1.0
```

Por onde começar a ler: `AGENTS.exemplo.md` → `.memory/00_INDEX.md` → `.memory/TASKS/INDEX.md`.

## Diferenças em relação a um projeto real

- Num projeto real, `AGENTS.exemplo.md` se chama `AGENTS.md`. Aqui ele foi renomeado para que agentes que trabalham nesta base de conhecimento não o carreguem como instrução.
- Num projeto real, a pasta `.start/` (o kit) fica nesta raiz, ao lado do `AGENTS.md`.
