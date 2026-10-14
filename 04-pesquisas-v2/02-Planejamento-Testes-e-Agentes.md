# Planejamento, Testes, Gates e Agentes

## Resumo executivo

O processo deve especificar resultados observáveis e deixar espaço para raciocínio técnico. A experiência publicada pela OpenAI indica que objetivos, ambientes bem documentados e feedback automatizado funcionam melhor do que uma máquina de estados excessivamente rígida.[^1] Regras universais como “sempre TDD”, “três falhas absolutas” e “até cinco subagentes” não possuem sustentação suficiente para todos os projetos.

## P03 — Projeto, Build, Sprint e Task

### Taxonomia recomendada

- **Projeto:** produto ou sistema duradouro.
- **Release:** versão disponibilizada a usuários.
- **Build:** execução/resultado técnico identificável; não deve ser sinônimo obrigatório de Sprint.
- **Sprint:** agrupamento funcional obrigatório neste SOP; a literatura não o exige universalmente, mas aqui toda Build possui pelo menos uma Sprint.
- **Task:** unidade de mudança com objetivo e verificação próprios.

Separar Build de Sprint evita confundir organização do trabalho com compilação. A pesquisa não encontrou obrigação universal de usar Sprints; entretanto, o baseline deste SOP adotou Sprint como camada obrigatória para manter uma hierarquia única. Mesmo em projeto simples, uma Build terá ao menos uma Sprint, com objetivo e conjunto de Tasks.

### Tamanho de Task

Uma Task está bem dimensionada quando:

- produz um resultado observável;
- possui poucos critérios de aceite coesos;
- pode ser revisada isoladamente;
- tem dependências conhecidas;
- permite rollback ou correção delimitada;
- cabe no contexto disponível com margem para testes.

Não há base para impor “2 a 5 Tasks por Sprint” universalmente. Use decomposição por coesão e risco.

## P06 — Testes proporcionais ao risco

### Matriz mínima

| Mudança | Verificação típica |
| :--- | :--- |
| Markdown/configuração simples | parser/linter, links ou inspeção |
| Função pura | unitário e tipos |
| Integração externa | contrato/integração com fixture |
| Fluxo do usuário | E2E direcionado |
| Build/pacote | build limpa + smoke do artefato |
| Migração | ensaio com schema/dados representativos |
| Segurança/autorização | testes negativos e revisão independente |
| Deploy | health checks, sinais operacionais e rollback pronto |

TDD é uma técnica útil, não obrigação para toda modificação. A orientação atual de modelos OpenAI recomenda calibrar testes ao impacto e evitar testes que apenas espelhem a implementação.[^2] O gate deve verificar comportamento ou propriedade relevante.

### Antiplacebo

- falhar o teste contra implementação sabidamente defeituosa quando viável;
- validar caminho feliz e falhas relevantes;
- não remover asserções para obter verde;
- registrar comando e resumo do resultado;
- preservar logs completos como artefato da CI, não no handoff.

## P07 — Ready, Done e gates

### Task Ready

- objetivo e não-objetivos claros;
- critérios observáveis;
- dependências resolvidas ou declaradas;
- comandos de verificação conhecidos;
- riscos e autorização externa identificados.

### Task Done

- critérios satisfeitos;
- verificações proporcionais passaram;
- diff revisado;
- histórico e decisões atualizados quando necessário;
- nenhuma mudança não explicada.

### Release Done

- conjunto de mudanças aprovado;
- build/artefato identificado;
- testes do artefato passaram;
- documentação de uso/migração atualizada;
- entrega ou deploy realmente confirmado;
- sinais pós-entrega saudáveis quando aplicável.

Gates devem variar por perfil: `simples`, `distribuído` e `produção crítica`. O DORA enfatiza capacidades e feedback rápido; métricas e controles não devem virar metas isoladas.[^3]

## P08 — Tentativas e circuit breaker

### Evidência e interpretação

Sistemas distribuídos usam retries para falhas transitórias, com limites, backoff e idempotência. Isso não justifica contar toda falha de lint, build e teste como uma única categoria nem executar rollback destrutivo na terceira falha.

### Política proposta

- falha destrutiva, de segurança ou de autorização: parar imediatamente;
- falha transitória conhecida: retry limitado com backoff;
- falha determinística: diagnosticar e mudar o código, sem retry cego;
- duas tentativas sem novo aprendizado: reavaliar plano;
- impasse persistente ou expansão de escopo: escalar com evidências;
- nunca sobrescrever trabalho automaticamente para “rollback”.

O contador pode ser registrado, mas o critério principal deve ser **progresso informacional e risco**, não o número três. O relatório deve conter comando, erro, hipóteses testadas, estado atual e opções.

## P09 — Subagentes e revisão independente

Subagentes ajudam quando há tarefas independentes, exploração paralela ou contexto especializado. Eles atrapalham em mudanças pequenas, edição concorrente do mesmo arquivo e problemas que exigem um único fio de raciocínio. A Anthropic recomenda escopos bem definidos e alerta para sobreuso; a OpenAI destaca worktrees como isolamento para trabalho paralelo.[^4][^5]

### Política recomendada

- delegar somente subtarefa independente e limitada;
- um proprietário integra o resultado;
- não permitir dois escritores no mesmo arquivo sem coordenação;
- usar worktree/branch quando houver escrita paralela;
- retorno inclui achado, evidência, arquivos e incerteza;
- revisão independente para risco alto, não para toda linha;
- quantidade e profundidade dependem da ferramenta, risco e capacidade disponível.

Um limite fixo de cinco agentes e 800 caracteres não deve ser guardrail universal. O limite deve ser configurável e o retorno tão curto quanto possível, mas completo para a decisão.

## Decisões propostas

1. Como decisão normativa deste SOP, Sprint é obrigatória; Build e Sprint deixam de ser sinônimos.
2. Task é dimensionada por coesão, verificabilidade e risco.
3. TDD deixa de ser universal; testes continuam obrigatoriamente proporcionais.
4. Circuit breaker usa risco e ausência de progresso, com teto configurável.
5. Subagentes são opcionais e usados apenas quando a decomposição beneficia o trabalho.
6. O fechamento verifica o artefato real, não apenas o código.

## Fontes

[^1]: OpenAI. [An open-source spec for Codex orchestration: Symphony](https://openai.com/index/open-source-codex-orchestration-symphony/). 2026.
[^2]: OpenAI. [Model guidance — testing and verification](https://developers.openai.com/api/docs/guides/latest-model). Acesso em 2026.
[^3]: Google Cloud DORA. [DevOps capabilities](https://docs.cloud.google.com/architecture/devops). Acesso em 2026.
[^4]: Anthropic. [Prompting best practices — subagent orchestration](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prompt-templates-and-variables). Acesso em 2026.
[^5]: OpenAI. [Introducing the Codex app](https://openai.com/index/introducing-the-codex-app/). 2026.
