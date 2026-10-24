# Como usar o `.Start` (v4.0.0)

> Versão atual: `4.0.0` (Skill agnóstica em `skills/start/`). Para consultar o formato antigo em pasta (`.start/` v3.0.0), acesse a tag Git `v3.0.0`.

## O que mudou na v4.0

Na versão 3, era preciso copiar 39 arquivos (`.start/`) para dentro de cada repositório de código e responder 7 perguntas em mensagens separadas mesmo em scripts pequenos.

Na **v4.0**, o `.Start` virou uma **Skill agnóstica modular (`skills/start/`)**:
- O motor mora nas skills do seu agente — no seu projeto entra **apenas a pasta `.memory/`** com os dados reais do produto.
- Os módulos de referência e templates são lidos **sob demanda** (sem despejar dezenas de arquivos no contexto inicial).
- Você escolhe entre dois trilhos logo na largada:
  - **`LITE` (Rápido / Ponytail):** apenas 3 arquivos em `.memory/` (`00_INDEX.md`, `DECISIONS.md` e `HANDOFF.md`), aprovação em 1 tela e código liberado na mesma sessão. Ideal para scripts, automações, plugins curtos, POCs e features pequenas.
  - **`FULL` (SDD Completo):** ciclo completo para produtos médios/grandes (`ONBOARDING` → `PRD` → `SDD + DELIVERY + Sprints/Tasks` → execução com subagentes → `Release`), com guardrails aprovados em 1 bloco único e 1 commit limpo por entrega.

---

## 1. Instalar a Skill

### Opção A — Instalação rápida via CLI (`skills`)
```bash
npx skills add ba-lison/.Start
```

### Opção B — Instalação manual agnóstica (Zero duplicação)
Copie `skills/start` para `.claude/skills/start` (no projeto ou na sua pasta global `~/.claude/skills/start`) e aponte `.agents/skills` para ela:

**Windows (PowerShell — sem Admin):**
```powershell
New-Item -ItemType Directory -Force -Path ".agents" | Out-Null
cmd /c mklink /J ".agents\skills" ".claude\skills"
```

**macOS / Linux:**
```bash
mkdir -p .agents && ln -s ../.claude/skills .agents/skills
```

*(Compatível com Claude Code, Codex, Cursor, Antigravity/Gemini, OpenCode e `Brain.OS`).*

---

## 2. Usar no Dia a Dia

Abra o agente na pasta do seu projeto de código e chame:

```text
/start
```
*(No Codex: `$start` · Ou em linguagem natural: `"Vamos iniciar este projeto com a skill start"` / `"Continue o projeto"`).*

O agente faz um diagnóstico **100% somente-leitura** antes de tocar em qualquer arquivo e detecta o cenário sozinho:

| Cenário encontrado | Modo | O que o agente faz |
| :--- | :--- | :--- |
| Pasta limpa | **`NOVO`** | Recomenda `LITE` ou `FULL`, confirma guardrails em 1 tela e cria `.memory/` + `AGENTS.md` |
| Código existente sem `.memory/` | **`ADOTAR`** | Diagnostica stack, comandos reais, Git e riscos sem mexer no código existente |
| `.memory/` já ativa | **`CONTINUAR`** | Confere `git status`/`HEAD` contra o último handoff e anuncia a próxima entrega |
| Projeto antigo (`.start/` v3 ou `docs/` v2) | **`MIGRAR`** | Migra com `git mv` e política *Zero Delete* (arquivos antigos preservados em `_backup_original/`) |

---

## 3. Skills Complementares Recomendadas (Uso Standalone)

A skill `start` funciona 100% sozinha, mas ganha ainda mais eficiência combinada com estas duas skills abertas:

| Skill | Repositório | Como ajuda no fluxo do `.Start` |
| :--- | :--- | :--- |
| **Ponytail** | [`DietrichGebert/ponytail`](https://github.com/DietrichGebert/ponytail) | Mantém o agente em modo sênior enxuto: menor código que resolve o problema completo, zero abstração prematura e declaração explícita de riscos ao final de cada entrega. |
| **Graphify** | [`Graphify-Labs/graphify`](https://github.com/Graphify-Labs/graphify) | Mapeia relações e arquitetura de código via AST (`--no-viz`), acelerando o diagnóstico somente-leitura no modo `ADOTAR` em repositórios médios e grandes sem queimar janela de contexto. |

---

## 4. Convivência com `Brain.OS`

Se você usa o `Brain.OS` para organizar o workspace da empresa:
- `iniciar-brain` cuida da **raiz do cérebro/operação** (`_contexto/`, `marca/`, `_raw/`).
- `start` cuida da **engenharia dentro da pasta de cada projeto de código** (ex: `projetos/meu-produto/`). Se você chamar `/start` na raiz do cérebro por engano, a própria skill detecta e pergunta em qual subpasta de projeto deve iniciar.
