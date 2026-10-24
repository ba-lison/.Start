<!-- start:inicio v4.0.0 — bloco gerenciado pela skill start. Não edite os marcadores. -->
## Ciclo de Engenharia (`.Start` · `.memory/`)

Este repositório opera com memória determinística em `.memory/` (skill `start`). Regras válidas para qualquer agente de IA (**Claude Code, Codex, Cursor, Antigravity, OpenCode**):

1. **Ao abrir qualquer chat (antes de editar código):** leia `.memory/00_INDEX.md`, confira o estado real do Git (`git status --porcelain` e `git log -n 3 --oneline`) e siga a skill `start` (modo `CONTINUAR`). O estado real do repositório vence qualquer handoff antigo.
2. **Decisões são humanas:** NUNCA decida sozinho escopo, requisito, arquitetura, banco/schema, API/UI pública, dependências novas, segurança, custos ou exclusão de arquivos. Pergunte com opções + recomendação e registre imediatamente em `.memory/DECISIONS.md`.
3. **Circuit Breaker anti-loop:** no máximo `3` rodadas (tentar + verificar) por objetivo, somando orquestrador, subagentes e chats. Repetiu o mesmo erro sem hipótese nova ou esgotou as 3 tentativas → **PARE**, registre bloqueio e pergunte ao usuário.
4. **Subagentes em profundidade 1:** subagente nunca chama subagente, nunca decide escopo/arquitetura, nunca faz commit e devolve relatório de até `500` caracteres.
5. **Segurança & Git limpo:** NUNCA abra ou grave segredos (`.env*`, `*.key`, `*.pem`, tokens). NUNCA apague ou sobrescreva arquivos pré-existentes sem autorização. Faça 1 commit local atômico ao concluir a unidade; `push`, PR e deploy exigem autorização explícita.
6. **Fechamento obrigatório:** atualize `.memory/00_INDEX.md` + handoff da sessão antes de encerrar.
<!-- start:fim -->
