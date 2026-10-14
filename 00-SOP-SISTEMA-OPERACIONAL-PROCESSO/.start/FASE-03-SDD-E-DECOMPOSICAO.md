# FASE 03 — SDD, DELIVERY E PLANO DA BUILD

> propósito: definir arquitetura, entrega e unidades executáveis antes da implementação
> ler quando: unidade `PLANO-BUILDxxx`
> pré-condição: PRD aprovado; em projeto adotado (ADOTAR), vale também o escopo da entrega aprovado e registrado numa DEC

## 1. SDD

Criar `.memory/sdd/SDD-xxx.md` (`.start/TEMPLATES/TEMPLATE_SDD.md`) com:

- requisitos do PRD cobertos;
- arquitetura e limites dos componentes;
- interfaces e dados;
- decisões e riscos;
- estratégia de verificação;
- implantação e migração, quando aplicável.

Toda escolha de arquitetura, dados, interface ou dependência é decisão (D-02 a D-04): perguntar com opções e recomendação, registrar e, se duradoura, criar ADR.

## 2. DELIVERY

Criar ou completar `.memory/DELIVERY.md` (`.start/TEMPLATES/TEMPLATE_DELIVERY.md`) com:

- perfil;
- comandos reais de setup, lint, teste, build e pacote — comando que não foi conferido fica marcado `hipótese:`;
- Git, branches e CI, coerentes com GP-06 a GP-09;
- versionamento;
- artefatos e destino;
- ambientes e autorização;
- deploy, verificação e recuperação;
- segurança, migrações e métricas aplicáveis;
- `N/A` com justificativa para o restante (marcar N/A é decisão, D-10).

## 3. Build, Sprints e Tasks

1. Criar `.memory/BUILDS/BUILDxxx.md`. Toda Build tem pelo menos uma Sprint em `.memory/SPRINTS/BUILDxxx_Syy.md`.
2. Cada Sprint tem:
   - objetivo funcional demonstrável;
   - Tasks ordenadas e suas dependências;
   - critérios de conclusão integrados;
   - verificação do incremento completo;
   - a unidade de fechamento `BUILDxxx_Syy_CLOSE`.
3. Cada Task, em `.memory/TASKS/BUILDxxx_Syy_Tzz.md` (`.start/TEMPLATES/TEMPLATE_TASK.md`), tem:
   - objetivo coeso e requisitos relacionados;
   - escopo e fora de escopo;
   - dependências;
   - decisões que precisam de resposta antes de começar;
   - objetivos verificáveis (base do contador de rodadas);
   - critérios de aceite;
   - verificação proporcional;
   - subagentes previstos, ou nenhum;
   - risco e autorizações necessárias.
4. Uma Task deve caber num chat, com margem para verificação e handoff (GF-10). Não há quantidade fixa de Sprints ou Tasks, mas a hierarquia `Build → Sprint → Task` é obrigatória.
5. SE GP-01 = `Sprint`: cada Sprint precisa caber num chat e ter no máximo GP-02 Tasks (cada Task usa pelo menos um subagente). SE não couber: perguntar ao usuário se divide a Sprint ou aumenta GP-02 (DEC).
6. Registrar todas as unidades em `.memory/TASKS/INDEX.md`, inclusive `BUILDxxx_Syy_CLOSE` e `BUILDxxx_CLOSE`. Com GP-01 = `Sprint`: a unidade é `BUILDxxx_Syy`, e suas Tasks ficam listadas logo abaixo dela.

## 4. Aprovação

Pedir a aprovação do plano (SDD, DELIVERY, Build, Sprints e Tasks) no formato de GUARDRAILS §3.3 e registrar como decisão.

## Gate

Avançar quando SDD, DELIVERY, Build e as Tasks iniciais estiverem aprovados, sem dependência oculta conhecida, e o fechamento da unidade tiver commit local documentado.
