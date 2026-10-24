# Padrões de Arquitetura para Circuit Breakers e Anti-Looping em Agentes de Código

## Visão geral do problema

Agentes de IA que orquestram execução de código sofrem de um modo de falha específico chamado "self-healing doom loop": o agente detecta um erro, tenta corrigi-lo automaticamente, gera um novo erro (às vezes causado pela própria correção), e reentra no ciclo de correção indefinidamente, consumindo tokens, tempo e, em casos críticos, aplicando ações destrutivas repetidas. A mitigação estrutural mais eficaz combina três camadas complementares: uma máquina de estados formal (circuit breaker), políticas de parada determinísticas por escopo (passo, agente, stack) e critérios de rollback ligados a idempotência, encerrando com escalonamento estruturado para um humano.[^1][^2][^3][^4]

## Máquina de estados formal (Circuit Breaker)

O padrão consolidado, herdado de sistemas distribuídos e adaptado para agentes, define três estados centrais:[^5][^6]

```
stateDiagram-v2
[*] --> Closed
Closed --> Open: failures >= threshold
Open --> HalfOpen: cooldown elapsed
HalfOpen --> Closed: probe succeeds
HalfOpen --> Open: probe fails
```

| Estado | Comportamento | Efeito prático |
|---|---|---|
| Closed (normal) | Ações fluem livremente; contadores de falha monitoram condições em background | Operação padrão do agente[^7] |
| Open (disparado) | Um limiar foi cruzado; agente é bloqueado e ações consequentes são impedidas imediatamente | Nenhuma chamada é tentada, sem gasto de tokens; erro `CircuitOpenError` retornado[^5][^7] |
| Half-Open (sondagem) | Uma quantidade limitada de chamadas de teste é permitida para validar recuperação antes de reabrir totalmente | Sucesso fecha o circuito; falha retorna a Open[^5][^6] |

Um ponto crítico é que o estado Open **não deve reabrir automaticamente por timer** em cenários de alto risco — a reabertura completa deve depender de reautorização humana explícita, distinta apenas da sondagem controlada em Half-Open. Em arquiteturas de produção, essa lógica é aplicada em camadas independentes — por ferramenta, por provedor e por capacidade — de modo que a falha de uma dependência não bloqueie ferramentas não relacionadas.[^7][^8][^5]

Threshold de disparo variam conforme o perfil de latência da ferramenta protegida:

| Tipo de ferramenta | Limiar de falha | Cooldown |
|---|---|---|
| APIs rápidas (busca, clima) | 3 falhas | 30 s[^5] |
| Ferramentas lentas (scraping, compilação) | 2 falhas | 120 s[^5] |
| Executores de código | 2 falhas | 60 s[^5] |

Arquiteturas orientadas a AWS Step Functions implementam essa lógica via estados `Catch`, roteando execuções falhas para um caminho de quarentena em vez de retry indefinido, com eventos do EventBridge disparando alertas quando limiares são excedidos.[^9]

## Políticas de parada determinísticas (máximo de 3 tentativas)

A política de parada deve ser aplicada de forma diferenciada por tipo de erro, não uniformemente:[^3][^10]

- Erros transitórios de infraestrutura (timeout, rate limit, 503): retry com backoff exponencial e jitter, limite de 3 a 5 tentativas.[^10][^3]
- Erros de validação de saída do LLM (formato incorreto, alucinação, violação de schema): não repetir a mesma chamada; construir prompt de correção explícito informando o que estava errado, permitindo apenas uma tentativa de correção antes de escalar.[^3]
- Falhas de execução de ferramenta: classificação por ferramenta entre retriable (timeout) e não retriable (violação de constraint única).[^3]
- Autenticação (401): zero retries automáticos — reautenticar primeiro, depois uma única tentativa.[^10]

Um esquema de escalonamento hierárquico em três níveis evita alarmes falsos por falhas isoladas ao mesmo tempo em que contém loops sistêmicos:[^1]

1. **Nível de passo (step):** 3 falhas consecutivas na mesma operação → interrompe apenas essa operação e grava contexto em log de saída (`outbox`).
2. **Nível de agente:** 5 falhas totais em um ciclo → interrompe o agente inteiro.
3. **Nível de stack:** 2 agentes paralisados simultaneamente → alerta o operador humano imediatamente.[^1]

A regra formal recomendada para o arquivo de instruções do agente (system prompt / AGENTS.md) é explícita: rastrear falhas consecutivas por passo; após 3 falhas no mesmo passo, gravar contexto e parar; nunca repetir indefinidamente; e sinalizar o problema em vez de amplificá-lo. Antes de definir qualquer política de retry, é necessário decidir explicitamente o escopo do que está sendo repetido, pois a análise de segurança muda completamente conforme o nível — passo, agente ou stack.[^11][^4][^1]

## Critérios de rollback automático

O critério central é a idempotência: toda chamada de ferramenta que altera estado deve carregar uma chave de idempotência gerada uma única vez por ação lógica e reutilizada em todas as tentativas de retry, evitando efeitos duplicados. A partir daí, o guia de engenharia de confiabilidade de agentes define três políticas de recuperação de efeito colateral, escolhidas conforme a natureza da ação:[^2][^4]

- **Reconciliar:** não desfazer a ação, mas ler o estado atual do mundo e convergir para o estado pretendido — abordagem mais robusta quando existe uma fonte externa autoritativa, pois funciona independentemente de quanto da tentativa anterior já foi aplicado.[^4]
- **Recusar (refuse):** para efeitos que não podem ser desfeitos ou reconciliados — um e-mail já lido, um pagamento já compensado, uma mensagem já publicada — a política correta é não tentar retry automático algum; o passo deve escalar diretamente para revisão humana.[^4]
- **Re-estabelecer o mundo antes de continuar:** ao retomar uma tarefa pausada, é necessário reconferir o estado (outros atores podem ter agido durante a pausa) e reavaliar explicitamente o orçamento de tempo restante, decidindo como política — não como acidente de implementação — se o prazo é estendido.[^4]

Para ações classificadas como G2/G3 (consequenciais ou irreversíveis), o rollback automático só deve ocorrer dentro do wrapper da ferramenta quando a falha for de transporte em uma ferramenta idempotente; falhas semânticas devem permitir que o próprio agente redecida, pois ele possui contexto que o wrapper não tem; e qualquer efeito colateral não compensável deve escalar em vez de retry. Um registro central de ferramentas ("tool registry") deve documentar explicitamente a categoria de risco de cada ferramenta para orientar essa decisão automaticamente.[^4]

## Formato de log estruturado para escalonamento

Quando o circuito dispara, o registro deve capturar dados suficientes para diagnóstico sem reprodução manual do problema. Um formato JSON de referência amplamente citado é:[^10][^1]

```json
{
  "type": "circuit_break",
  "step": "fetch_market_data",
  "failure_count": 3,
  "last_error": "timeout after 30s",
  "context": "Mid-cycle on portfolio rebalance. Position unchanged.",
  "recommended_action": "Check API credentials and rate limits"
}
```


Além do payload de disparo, boas práticas de auditoria exigem registrar separadamente o evento de reabertura — quem reautorizou, quando e por quê — mantendo trilha de decisão distinta da trilha de falha. Em plataformas baseadas em AWS Step Functions, recomenda-se logging de execução no nível `ALL` para o CloudWatch Logs, capturando todas as transições de estado, entradas/saídas e eventos de erro, com políticas de retenção alinhadas a requisitos de compliance.[^7][^9]

A notificação de escalonamento em si — não apenas o log — deve ser ativa (PagerDuty, Slack, e-mail, SMS) e conter contexto suficiente para uma decisão humana em até 30 segundos: consulta original, tipo e mensagem de erro, número de tentativas de retry, quais estratégias de fallback foram tentadas, timestamps e identificadores relevantes (IDs de sessão, de ordem, etc.). A UX de escalonamento deve incluir explicitamente qual ação está sendo proposta, por que o agente deseja executá-la, quais alternativas foram consideradas e o que ocorre caso seja rejeitada — uma notificação genérica do tipo "agente quer executar ação, aprovar?" é considerada insuficiente.[^3][^10]

## Gatilhos de escalonamento para engenheiro humano

Cinco condições consolidadas determinam quando a escalada humana é obrigatória, independentemente do estado do circuit breaker:[^3]

- Ações irreversíveis de alto risco: exclusão, pagamento, publicação externa ou deploy em produção.
- Confiança abaixo de limiar: quando o raciocínio do agente expressa incerteza ("não tenho certeza", hedging) e o próximo passo é irreversível.
- Falhas consecutivas múltiplas: 3 ou mais falhas na mesma ferramenta com o agente ainda tentando executá-la indica que a abordagem, não apenas a execução, precisa de revisão humana.
- Contradição de contexto: quando informações no contexto do agente contradizem fatos previamente estabelecidos, geralmente sinalizando problema de qualidade de dados.
- Limite de política: a ação solicitada está fora do escopo definido para operação autônoma.[^3]

A escalação deve ser tratada como último recurso, disparada somente depois que retries, circuit breakers e fallbacks tiverem se esgotado — escalar cedo demais desperdiça tempo do operador humano, enquanto escalar tarde demais (ou nunca) arrisca falhas silenciosas em fluxos críticos. Um checklist operacional de validação do circuit breaker recomenda testar periodicamente se o disparo realmente ocorre e se o circuito permanece bloqueado, garantindo que a proteção funcione quando efetivamente necessária, e não apenas em teoria.[^7][^10]

---

## References

1. [The Circuit Breaker Pattern: Stop Your AI Agent From Amplifying ...](https://dev.to/askpatrick/the-circuit-breaker-pattern-stop-your-ai-agent-from-amplifying-failures-30d4) - Every experienced engineer knows the circuit breaker pattern for microservices. But most AI agent...

2. [Error Handling and Recovery Patterns for AI Agents](https://agent-works.ai/insights/agent-error-handling-recovery-patterns) - Idempotency keys, transient-vs-terminal errors, circuit breakers, and HITL escalation thresholds for...

3. [Reliable AI Agent Pipelines: Orchestration, Retries, Circuit ...](https://www.chaitanyaprabuddha.com/blog/reliable-ai-agent-pipelines-orchestration-retries) - Error taxonomy, retry strategies, circuit breakers, idempotent tool design, human-in-the-loop escala...

4. [Agent Reliability Engineering Design Guide - Retries, Loop ...](https://hidekazu-konishi.com/entry/agent_reliability_engineering_design_guide.html) - A vendor-neutral design guide for the reliability of the AI agent execution loop: termination condit...

5. [Agent Circuit Breaker - AgentPatterns.ai](https://www.agentpatterns.ai/agent-design/agent-circuit-breaker/) - Wrap external tools with per-tool failure-tracking state machines that block calls during degraded s...

6. [Circuit Breaker Pattern for AI Agents (2026) | Taskade AI](https://www.taskade.com/wiki/ai-agents/circuit-breaker) - The circuit breaker pattern lets an AI agent stop calling a failing tool, fail fast, serve a fallbac...

7. [The Circuit Breaker Pattern for AI Agents](https://dev.to/brennhill/the-circuit-breaker-pattern-for-ai-agents-11pl) - A circuit breaker for AI agents is an automatic control that pauses an agent the moment a measured c...

8. [Agent-Level Circuit Breakers: Per-Tool, Per-Provider, Per-Capability ...](https://appscale.blog/en/blog/microservices-pattern-agent-level-circuit-breakers-2026) - Production guide to agent-level circuit breakers: three-layer hierarchy (tool, provider, capability)...

9. [agentsec06-bp02.md](https://docs.aws.amazon.com/pt_br/wellarchitected/latest/agentic-ai-lens/agentsec06-bp02.md)

10. [Error Handling in AI Agents: Circuit Breakers, Retry & Recovery](https://preporato.com/blog/error-handling-resilience-patterns-agentic-ai-systems) - Master error handling and resilience patterns for agentic AI systems on the NVIDIA NCP-AAI exam. Cov...

11. [AGENTS.md Template for Circuit Breaker Design - Suhas Bhairav](https://suhasbhairav.com/ai-skills/agents-md-templates/circuit-breaker-design-agents-md-template) - Copyable AGENTS.md Template for circuit breaker design to govern AI coding agents and multi-agent or...

