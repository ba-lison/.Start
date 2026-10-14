# FASE 01 — CONTEXTO E DECISÕES INICIAIS

> propósito: transformar a ideia (NOVO) ou o projeto existente (ADOTAR) em contexto suficiente para especificar
> ler quando: Parte 2 do onboarding
> unidade: `ONBOARDING` (a mesma do questionário de guardrails)
> pré-condição: `.memory/PROJECT_GUARDRAILS.md` com `status: APPROVED`

## Regras

1. NUNCA implementar o produto nesta fase.
2. Uma pergunta por vez (GP-05).
3. Perguntas 1 a 4 são de levantamento (GUARDRAILS §3.3): abertas, sem opções. As respostas vão para `.memory/PROJECT_CONTEXT.md`; só viram DEC se trouxerem uma decisão do §3.1 (exemplo: "tem que ser em Python" é D-04).
4. Perguntas 5 e 6 são de escolha: formato do GUARDRAILS §3.3, com opções e recomendação; a resposta vira DEC na hora.
5. Em ADOTAR: partir do rascunho feito com o diagnóstico e só confirmar o que já foi deduzido.

## Perguntas (nesta ordem; pular o que já foi respondido)

1. Qual problema o projeto resolve, e para quem?
2. Qual resultado mostra que deu certo?
3. Quais restrições existem (prazo, plataforma, custo, tecnologia obrigatória ou proibida)?
4. Há dados sensíveis, integrações externas ou ambiente especial?
5. Há alternativas reais de abordagem? SE houver: apresentar as opções com recomendação (D-08). SE não houver: pular. NUNCA inventar alternativas só para cumprir a pergunta.
6. Qual o perfil: `SIMPLE`, `STANDARD` ou `CRITICAL`? Recomendar, com o motivo.

## Saída

- `.memory/PROJECT_CONTEXT.md` (`.start/TEMPLATES/TEMPLATE_PROJECT_CONTEXT.md`);
- decisões em `.memory/DECISIONS.md`; decisão complexa também em ADR (`.memory/adrs/`);
- dúvidas e riscos explícitos em `.memory/PROJECT_CONTEXT.md`.

## Gate

Avançar somente quando:

- [ ] `.memory/PROJECT_GUARDRAILS.md` está `APPROVED`;
- [ ] problema, sucesso esperado e limites estão claros o suficiente para o PRD;
- [ ] o perfil foi decidido pelo usuário;
- [ ] as decisões da fase estão registradas.

Incerteza que não bloqueia pode ficar registrada como dúvida. Depois do gate: fechamento conforme o arquivo do modo.
