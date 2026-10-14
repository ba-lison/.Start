# Padrão Orchestrator-Worker com Profundidade Rígida 1 e Prevenção de Degradação de Contexto

## Princípio central

O padrão Orchestrator-Worker com profundidade rígida igual a 1 significa que apenas o orquestrador (nível 0) pode invocar subagentes (nível 1), e nenhum subagente recebe a ferramenta de invocação (`Task`/`Agent`), impedindo qualquer recursão. Essa restrição é exatamente o comportamento nativo do Claude Code quando `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` é definido como `1`: no limite de profundidade, o subagente perde acesso à ferramenta de agente e deve executar o trabalho delegado sozinho, retornando apenas um resumo. A justificativa de design é forçar orquestração explícita no nível superior em vez de hierarquias emergentes difíceis de depurar e auditar.[^1][^2][^3]

## Por que a degradação de contexto ocorre

O principal modo de falha em sistemas orquestrador-trabalhador não é o desempenho dos workers, mas o esgotamento da janela de contexto do orquestrador. Isso acontece quando o orquestrador lê a saída bruta e completa de cada subagente em vez de um resumo compacto — cada rodada de delegação acumula texto irrelevante, diluindo o "espaço de atenção" disponível para a síntese final. A mitigação consensual entre as fontes é: workers sempre retornam JSON estruturado e compacto; o orquestrador nunca relê a saída bruta do worker; e cada agente deve receber apenas o contexto mínimo suficiente para sua subtarefa específica, seguindo uma estrutura em camadas (contexto global → local → da tarefa).[^4][^5][^6][^7][^8]

## Melhores práticas para o contrato de profundidade 1

| Prática | Descrição | Fonte |
|---|---|---|
| Fan-out limitado | Definir um número máximo de workers por requisição (tipicamente 5-8); acima disso, considerar variante hierárquica em vez de aumentar profundidade | [^5][^4] |
| Escopo estreito por worker | Um worker, uma responsabilidade única e limitada (ex.: não mais do que cabe em uma revisão de PR) | [^6][^7] |
| Saída somente estruturada | Workers retornam apenas JSON/schema tipado; nunca texto livre longo | [^5][^6] |
| Sem comunicação entre workers | Toda coordenação passa exclusivamente pelo orquestrador; workers não se conhecem | [^5][^9] |
| Modelo mais barato para workers | Reservar o modelo mais caro/capaz apenas para orquestração e síntese | [^4][^7] |
| Papel explícito | Cada worker recebe instrução clara de que é um subagente, sem ferramenta de invocação de agentes | [^10][^1] |
| Resposta única ao usuário | Somente o orquestrador comunica-se com o usuário final; workers são "pesquisadores", não "respondedores" | [^10] |
| Orçamento de execução | Limites de tokens, passos (`max_steps`) e tempo por worker, com timeout de 30-60s por chamada | [^9][^6] |
| Tratamento de erro estruturado | Falha retorna objeto de erro estruturado para o orquestrador decidir (retry, fallback, escalonamento humano), nunca propagação livre | [^11][^7] |
| Observabilidade | Registrar o plano do orquestrador, cada dispatch e cada resumo de worker como eventos discretos e auditáveis | [^4][^12] |

## Schema de dispatch (entrada da tarefa)

O dispatch deve conter apenas o necessário para a subtarefa isolada, evitando repassar histórico de conversa completo (princípio de contexto mínimo suficiente):[^7][^8]

```json
{
  "task_id": "string (UUID único da subtarefa)",
  "orchestrator_id": "string (identificador da sessão pai, nível 0)",
  "depth": 1,
  "agent_role": "string (ex.: 'code-reviewer', 'api-researcher')",
  "objective": "string (descrição objetiva e verificável da meta)",
  "inputs": {
    "files": ["array de caminhos/URLs relevantes, somente os necessários"],
    "context_snippet": "string (contexto local mínimo, não histórico completo)",
    "constraints": ["array de restrições, ex.: 'não modificar schema de DB'"]
  },
  "tools_allowed": ["array de ferramentas permitidas — nunca incluir Task/Agent"],
  "output_schema_ref": "string (referência ao schema de reporte esperado)",
  "success_criteria": ["array de critérios verificáveis de conclusão"],
  "budget": {
    "max_steps": "integer",
    "max_tokens": "integer",
    "timeout_seconds": "integer (ex.: 30-60)"
  }
}
```

Esse formato espelha as recomendações de "communication protocols" para handoffs multiagente, que exigem IDs de tarefa, papel do agente, objetivo, contexto e metadados explícitos, além do requisito de que o worker nunca receba a ferramenta de invocação de subagentes, garantindo profundidade fixa em 1.[^12][^3][^1]

## Schema de reporte sintético (máximo 150 palavras)

O reporte de retorno deve ser estritamente estruturado e compacto — nunca texto livre extenso — para não inflar o contexto do orquestrador:[^5][^6]

```json
{
  "task_id": "string (mesmo ID do dispatch)",
  "status": "success | partial | failed",
  "summary": "string (<=150 palavras, síntese objetiva do resultado)",
  "key_findings": ["array de até 5 bullets factuais, cada um <=20 palavras"],
  "artifacts": ["array de referências a arquivos/diffs gerados, não conteúdo bruto"],
  "confidence": "float (0-1)",
  "blockers": ["array de impedimentos, vazio se nenhum"],
  "next_action_recommendation": "string opcional (<=30 palavras)",
  "error": {
    "type": "string opcional",
    "message": "string opcional"
  }
}
```

O campo `summary` é o único texto livre permitido e deve respeitar rigorosamente o limite de 150 palavras; o orquestrador consome apenas este JSON — nunca logs, saídas de ferramentas intermediárias ou raciocínio bruto do worker. Essa disciplina de "resumo compacto, nunca texto completo" é citada de forma consistente como o principal antídoto contra a explosão de custo e a degradação de contexto no orquestrador.[^6][^4][^5][^7]

## Guarda-corpos adicionais para profundidade rígida

Além do schema, a profundidade 1 deve ser reforçada estruturalmente, não apenas por convenção de prompt: a ferramenta de invocação de agentes (`Agent`/`Task`) deve ser omitida da lista `tools_allowed` de todo worker, e o ambiente de execução deve validar essa omissão antes do dispatch — replicando o comportamento nativo do Claude Code, em que no limite de profundidade a ferramenta `Agent` é retirada da caixa de ferramentas do subagente. Também é recomendável capar a concorrência de workers simultâneos (ex.: 20 no caso do Claude Code) como eixo independente da profundidade, já que os dois limites — concorrência e profundidade — são controlados por variáveis separadas e devem ser configurados em conjunto para sistemas de produção.[^13][^2][^3][^14]
</content>

---

## References

1. [Sub-Agent Architecture and Isolation | FlorianBruniaux/claude ...](https://deepwiki.com/FlorianBruniaux/claude-code-ultimate-guide/3.9-sub-agent-architecture-and-isolation) - This document explains Claude Code's sub-agent execution model: how the `Task` tool spawns isolated ...

2. [four days, three subagent policies · vibe code 24/7](https://www.vibecode247.com/news/claude-code-subagent-guardrails-depth-cap/) - Between July 21 and July 24, Anthropic shipped two Claude Code releases that rewrote the rules on ne...

3. [Create custom subagents - Claude Code Docs](https://code.claude.com/docs/en/sub-agents)

4. [Orchestrator-Workers Multi-Agent Pattern (2026 Guide)](https://vantaige.io/blog/orchestrator-workers-multi-agent-pattern-2026) - Multi-agent orchestration explained: how orchestrator-workers delegates to specialist agents, when t...

5. [Multi-Agent Orchestration Patterns: A Practical Guide](https://www.glukhov.org/ai-systems/architecture/multi-agent-orchestration-patterns/) - Six proven multi-agent orchestration patterns for production AI systems: orchestrator-worker, sequen...

6. [Multi-Agent — Implementing the Orchestrator Worker Pattern](https://dev.to/hiroki-kameyama/multi-agent-implementing-the-orchestrator-x-worker-pattern-38gd) - Introduction Through Chapter 6 (Fine-tuning), we focused on improving a single AI system....

7. [Multi-Agent Orchestration: 4 Patterns That Actually Work](https://www.heyuan110.com/posts/ai/2026-02-26-multi-agent-orchestration/) - Learn 4 proven multi-agent orchestration patterns — Pipeline, Router, Orchestrator-Worker, and Evalu...

8. [Multi-Agent Orchestration Patterns: Production Guide for ...](https://explainx.ai/blog/multi-agent-orchestration-patterns-guide-2026) - Learn the five core multi-agent orchestration patterns — orchestrator/worker, pipeline, fan-out, deb...

9. [Multi-Agent Orchestration: How to Coordinate AI Agents at](https://gurusup.com/blog/multi-agent-orchestration-guide) - Multi-agent orchestration coordinates specialized AI agents. This guide covers centralized and decen...

10. [Multi-agent orchestration patterns and best practices](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/multi-agent-patterns) - Best practices for multi-agent orchestration · 1. Single response principle · 2. Subagent instructio...

11. [Multi-Agent Patterns: Orchestrators, Workers, and Pipelines](https://aiagentsblog.com/blog/multi-agent-patterns/) - Three architecture patterns for building multi-agent systems with Claude — when to use orchestrators...

12. [Multi-Agent Orchestration: Patterns and Best Practices - GitHub](https://github.com/enuno/claude-command-and-control/blob/main/docs/best-practices/04-Multi-Agent-Orchestration.md) - Claude command & AI agent creation best practices and templates - enuno/claude-command-and-control

13. [Creare subagent personalizzati - Claude Code Docs](https://code.claude.com/docs/it/sub-agents)

14. [Claude Code Subagent Concurrency Limits Clarified — Two Axes ...](https://note.com/docsfirst/n/ndfe0419c51c4?hl=en) - On August 7, 2026, details regarding concurrency limits were added to the official Claude Code subag...

