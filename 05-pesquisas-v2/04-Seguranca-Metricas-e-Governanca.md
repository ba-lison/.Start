# Segurança, Métricas, Governança e Portabilidade

## Resumo executivo

O SOP deve oferecer um baseline pequeno e aumentar controles conforme risco. NIST SSDF organiza práticas em preparar a organização, proteger software, produzir software bem protegido e responder a vulnerabilidades; também recomenda registrar requisitos, riscos, decisões e proveniência.[^1] Isso não implica exigir SBOM, assinatura, DORA e SLO completos em todo projeto pessoal.

## P19 — Segurança proporcional

### Baseline obrigatório

- não versionar segredos;
- `.env.example` sem valores reais;
- dependências com lockfile quando suportado;
- origem oficial de pacotes;
- revisão de mudanças em autenticação, autorização, criptografia e dados;
- teste negativo para controles críticos;
- permissões mínimas na CI;
- registrar vulnerabilidade conhecida e risco aceito.

### Controles condicionais

- SAST, dependency scanning e secret scanning para repositório publicado/compartilhado;
- SBOM para software distribuído ou exigência contratual;
- assinatura/proveniência para distribuição pública ou alto risco;
- threat model para dados sensíveis, exposição pública ou fronteiras de confiança complexas.

O NIST orienta preservar releases e dados de proveniência, enquanto SLSA formaliza attestations de build.[^1][^2]

## P20 — Métricas para projeto pequeno

DORA estuda capacidades e desempenho de entrega. Métricas não devem servir para avaliar produtividade individual nem gerar coleta manual pesada.[^3]

Conjunto mínimo sugerido:

| Métrica | Uso |
| :--- | :--- |
| Tempo de ciclo da Task | Detectar unidades grandes ou bloqueios |
| Taxa de Tasks reabertas | Detectar qualidade insuficiente |
| Builds/Releases falhas | Avaliar estabilidade da entrega |
| Tempo para recuperar | Avaliar capacidade de correção |
| Frequência de entrega | Observar fluxo, sem meta artificial |

Coletar a partir de Git/CI quando possível. Se não houver produção contínua, deployment frequency e MTTR podem ser `N/A`.

## P21 — Evolução do SOP

O próprio processo precisa de histórico:

- versão no cabeçalho;
- `CHANGELOG-SOP.md`;
- proposta/racional para mudança relevante;
- período experimental na `docs.v2`;
- teste em projeto piloto;
- aprovação humana;
- instruções de migração;
- possibilidade de retorno à versão anterior.

Regras devem declarar se são `MUST`, `SHOULD` ou `MAY`, evitando linguagem absoluta acidental.

## P22 — Auditoria automática

Validações úteis:

- links e caminhos quebrados;
- IDs duplicados;
- Task `DONE` sem handoff/evidência;
- Build fechada com Task aberta;
- Release sem commit, versão ou hash;
- decisão superseded sem sucessora;
- `DELIVERY.md` apontando para comandos inexistentes;
- versão divergente entre arquivo, tag e artefato;
- segredo provável em Markdown/log;
- documento órfão não listado no índice.

Começar com script local de leitura e CI opcional. O auditor não deve corrigir automaticamente decisões semânticas.

## P23 — Portabilidade entre agentes

Separar núcleo e adaptadores:

```text
SOP independente de fornecedor
├── adaptador Codex: AGENTS.md, skills, sandbox/worktrees
├── adaptador Claude Code: CLAUDE.md, hooks, subagents
├── adaptador Gemini/ADK
└── adaptador genérico: prompt + arquivos Markdown + Git
```

Google ADK publica documentação em `llms.txt` e MCP para consumo por coding agents, exemplo útil de documentação legível por máquina.[^4] Regras específicas de ferramenta não devem contaminar o núcleo.

OpenAI e Anthropic possuem documentação diretamente aplicável a instruções, ferramentas, contexto, subagentes e verificação.[^5][^6] DeepMind oferece pesquisa relevante sobre memória e avaliação, mas não é fonte principal para Git/CI. A documentação pública da DeepSeek deve ser usada somente quando cobrir concretamente o comportamento operacional do agente; o nome do laboratório não basta como evidência.

## P24 — Prompt injection e memória

Arquivos recuperados, páginas web, issues e logs devem ser tratados como dados não confiáveis. Riscos:

- instrução maliciosa persistida em handoff;
- segredo copiado para contexto ou log;
- documento externo apresentado como regra local;
- alteração de protocolo por conteúdo pesquisado;
- comando perigoso escondido em arquivo.

Controles:

- somente arquivos canônicos aprovados definem regras;
- conteúdo externo recebe origem e não vira instrução;
- registrar fonte de fatos importados;
- não persistir tokens, dados pessoais ou saídas sensíveis;
- revisar mudanças em protocolo/AGENTS/skills;
- sandbox, permissões e logs para ações do agente.

A OpenAI descreve sandbox, políticas de aprovação, limites de rede e telemetria como controles complementares para coding agents.[^7]

## P25 — Custo e manutenção do contexto

Critérios operacionais:

- boot carrega apenas protocolo curto, estado, decisões ativas e Task;
- pesquisas são consultadas sob demanda;
- handoff aponta para evidência em vez de copiá-la;
- arquivos ativos têm dono e gatilho de atualização;
- documentos históricos são indexados e arquivados;
- inconsistência detectada bloqueia conclusão, não necessariamente toda leitura.

## Decisões propostas

1. Segurança em perfis: básico, distribuído e crítico.
2. Métricas mínimas e automáticas; DORA completa apenas quando aplicável.
3. O SOP recebe versionamento e changelog próprios.
4. Criar auditor read-only antes de mais templates.
5. Núcleo permanece agnóstico a fornecedor.
6. Conteúdo externo nunca ganha autoridade de instrução por estar em Markdown.
7. Documentação DeepSeek/DeepMind entra apenas quando diretamente relevante e verificável.

## Fontes

[^1]: NIST. [Secure Software Development Framework (SSDF)](https://csrc.nist.gov/projects/ssdf).
[^2]: SLSA. [Supply-chain Levels for Software Artifacts](https://slsa.dev/).
[^3]: Google Cloud DORA. [DevOps capabilities](https://docs.cloud.google.com/architecture/devops) e [DORA research](https://cloud.google.com/devops).
[^4]: Google Agent Development Kit. [Code with AI](https://google.github.io/adk-docs/tutorials/coding-with-ai/).
[^5]: OpenAI. [Model guidance](https://developers.openai.com/api/docs/guides/latest-model).
[^6]: Anthropic. [Prompting best practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prompt-templates-and-variables).
[^7]: OpenAI. [Running Codex safely at OpenAI](https://openai.com/index/running-codex-safely/). 2026.

