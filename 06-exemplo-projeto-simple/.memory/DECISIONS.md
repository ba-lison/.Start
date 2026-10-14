# DECISÕES — CLI de saudação

> propósito: registro de toda decisão do projeto: o que foi decidido, por quem e por quê
> ler quando: início de todo chat — ler as entradas `ACTIVE` e `INFERRED`
> atualizar quando: no momento em que o usuário decide (GF-06); anexar ao fim, com número = maior DEC existente + 1; NUNCA apagar entrada
> status possíveis: ACTIVE | INFERRED | SUPERSEDED | REJECTED
> decidido_por possíveis: usuário | código existente (a confirmar) | usuário (confirmou decisão herdada em YYYY-MM-DD) | usuário (v2)
> substituição: nova entrada com `substitui: DEC-xxx`; na antiga, só o `status` muda para `SUPERSEDED`
> confirmação de herdada (D-12): na mesma entrada, só `status` e `decidido_por` mudam

## DEC-001 — Guardrails do projeto aprovados (v1)

- data: 2026-09-12
- status: ACTIVE
- unidade: ONBOARDING
- pergunta: APROVAÇÃO — Guardrails deste projeto
- opcoes: A) Aprovo; B) Quero mudar
- decisao: guardrails aprovados. Respostas: 1) regras fixas mantidas; 2) uma Task por chat, troca de chat no padrão, ferramentas Codex e Cursor; 3) até 10 subagentes, relatório de 500 caracteres; 4) 3 rodadas; 5) uma pergunta por vez, sem itens extras de decisão; 6) Git no padrão; 7) sem áreas protegidas, pt-BR, sem regras extras
- decidido_por: usuário
- motivo: projeto de exemplo, sem necessidade de regra extra
- consequencias: valem os valores de `.memory/PROJECT_GUARDRAILS.md` v1
- substitui: —
- adr: —

## DEC-002 — Perfil SIMPLE

- data: 2026-09-12
- status: ACTIVE
- unidade: ONBOARDING
- pergunta: PERGUNTA 6/6 — Perfil do projeto
- opcoes: A) SIMPLE; B) STANDARD; C) CRITICAL
- decisao: perfil SIMPLE
- decidido_por: usuário
- motivo: projeto local, sem dados sensíveis nem usuários externos
- consequencias: verificação proporcional de baixo risco; CI e deploy podem ser N/A
- substitui: —
- adr: —

## DEC-003 — Escopo da primeira versão limitado à saudação

- data: 2026-09-12
- status: ACTIVE
- unidade: PRD-001
- pergunta: PERGUNTA 1/2 — Localização (outros idiomas) entra na v0.1.0?
- opcoes: A) Fora de escopo; B) Incluir inglês
- decisao: localização fica fora de escopo da v0.1.0
- decidido_por: usuário
- motivo: manter o exemplo mínimo
- consequencias: saída sempre em português; nova língua exige novo PRD
- substitui: —
- adr: —

## DEC-004 — PRD-001 aprovado

- data: 2026-09-12
- status: ACTIVE
- unidade: PRD-001
- pergunta: APROVAÇÃO — PRD-001
- opcoes: A) Aprovo; B) Quero mudar
- decisao: PRD-001 aprovado com RF-001 e RF-002
- decidido_por: usuário
- motivo: requisitos verificáveis e fora de escopo explícito
- consequencias: libera a unidade PLANO-BUILD001
- substitui: —
- adr: —

## DEC-005 — Node.js com saída textual

- data: 2026-09-12
- status: ACTIVE
- unidade: PLANO-BUILD001
- pergunta: PERGUNTA 1/2 — Runtime da CLI
- opcoes: A) Node.js; B) Python
- decisao: usar Node.js, com saída textual no terminal
- decidido_por: usuário
- motivo: exemplo mínimo e portátil
- consequencias: comandos `npm` no `.memory/DELIVERY.md`
- substitui: —
- adr: —

## DEC-006 — Plano da BUILD001 aprovado, com itens N/A

- data: 2026-09-12
- status: ACTIVE
- unidade: PLANO-BUILD001
- pergunta: APROVAÇÃO — SDD-001, DELIVERY e BUILD001
- opcoes: A) Aprovo; B) Quero mudar
- decisao: plano aprovado; CI, deploy, banco, feature flags, DORA e SBOM marcados N/A
- decidido_por: usuário
- motivo: exemplo local, sem serviço e sem distribuição pública
- consequencias: libera BUILD001_S01_T01; entrega é pacote local
- substitui: —
- adr: —

## DEC-007 — Próxima entrega em aberto

- data: 2026-09-14
- status: ACTIVE
- unidade: BUILD001_CLOSE
- pergunta: PERGUNTA — Próxima unidade depois da v0.1.0
- opcoes: A) Planejar BUILD002 agora (PRD-002); B) Encerrar por ora e decidir depois
- decisao: encerrar por ora; PRD-002 só quando o usuário pedir
- decidido_por: usuário
- motivo: o exemplo já demonstra o fluxo completo
- consequencias: `.memory/00_INDEX.md` fica sem próxima unidade definida
- substitui: —
- adr: —
