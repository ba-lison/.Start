# FASE 02 — PRD E CRITÉRIOS DE PRODUTO

> propósito: definir o que será entregue, para quem, por quê e como comprovar valor, sem prescrever implementação
> ler quando: unidade `PRD-xxx`
> pré-condição: onboarding concluído (`modo_atual: CONTINUAR` em `.memory/00_INDEX.md`)

## Conteúdo mínimo do PRD

- problema e usuários;
- resultado e métricas de sucesso;
- requisitos funcionais;
- requisitos não funcionais proporcionais;
- cenários e critérios de aceite;
- dados, privacidade e riscos relevantes;
- fora de escopo;
- dependências e dúvidas;
- versão, status e aprovação.

## Procedimento

1. Criar `.memory/prd/PRD-xxx.md` (`.start/TEMPLATES/TEMPLATE_PRD.md`).
2. Dar IDs estáveis aos requisitos (RF-xxx, RNF-xxx).
3. Ambiguidade ou conflito: perguntar, uma pergunta por vez (GUARDRAILS §3.3), e registrar (§3.4).
4. Deixar o fora de escopo explícito.
5. Pedir o aceite do escopo:

```text
APROVAÇÃO — PRD-xxx
A) Aprovo
B) Quero mudar: diga o item
```

6. Registrar a aprovação como decisão.
7. Fechar a unidade (`.start/PROTOCOLO_MESTRE_EXECUCAO.md` §8). Próxima unidade: `PLANO-BUILDxxx`.

## Gate

Avançar quando os requisitos prioritários forem verificáveis, o fora de escopo estiver explícito e a aprovação estiver registrada. Implementação continua proibida até o plano ser aprovado.
