# Histórico, Memória e Continuidade para Agentes de Código

## Resumo executivo

Um processo com histórico confiável precisa separar cinco coisas: instruções permanentes, estado atual, decisões, eventos e evidências. Misturá-las em um único “arquivo de memória” causa repetição, contradição e crescimento sem limite. A recomendação é manter poucos arquivos canônicos, usar Git como registro técnico e produzir handoff curto como índice verificável, não como cópia integral do contexto.

A continuidade não deve depender de o próximo agente confiar no texto do agente anterior. O novo agente precisa conferir o repositório, a Task, o estado do Git e os comandos de verificação. A OpenAI descreve ambientes legíveis, documentação, testes, observabilidade e feedback loops como infraestrutura essencial para trabalho confiável de agentes; também registra que tentativas rígidas de modelar todo o trabalho como uma máquina de estados foram limitantes.[^1][^2]

## P01 — Memória operacional mínima

### Modelo recomendado

| Informação | Fonte canônica | Política |
| :--- | :--- | :--- |
| Como trabalhar | `AGENTS.md` ou protocolo apontado por ele | Atualizado por versão; curto |
| O que o projeto é agora | `PROJECT_CONTEXT.md` | Sobrescrever com histórico no Git |
| Por que decisões foram tomadas | `DECISIONS.md` + `adrs/` | Anexar; decisões antigas viram superseded |
| O que está planejado | PRD, SDD, Build e Tasks | Estado explícito e IDs estáveis |
| O que aconteceu | `CHANGELOG.md` | Append por entrega relevante |
| Evidência de uma sessão | Handoff | Imutável após fechamento, salvo correção declarada |
| O que foi distribuído | Release record + artefato | Imutável e relacionado a commit/hash |
| Verdade técnica do código | repositório e testes | Nunca duplicar integralmente em memória |

### Regra de não duplicação

- `PROJECT_CONTEXT` contém a fotografia atual, não uma cronologia.
- `CHANGELOG` contém mudanças relevantes, não comandos e logs completos.
- ADR contém contexto e consequência de uma decisão, não cada escolha local de implementação.
- Handoff contém delta, evidência e ponteiros, não resumos de todos os documentos.
- Git contém o histórico detalhado dos arquivos; Markdown explica significado e intenção.

Documentação de arquitetura deve ser útil e mantida conforme o sistema muda; o Google Well-Architected destaca que histórico de mudanças fornece contexto e que qualidade documental não é quantidade de texto.[^3]

## P02 — Continuidade entre sessões

### Pacote mínimo de entrada

1. protocolo/instruções aplicáveis;
2. Task atual;
3. último handoff relacionado;
4. `PROJECT_CONTEXT` e decisões ativas;
5. estado real do Git e do workspace;
6. comandos de verificação definidos no repositório.

### Verificação no início

O agente deve conferir:

- se os caminhos citados existem;
- se a branch/commit correspondem ao handoff;
- se há mudanças não registradas;
- se a Task anterior está realmente concluída;
- se o teste citado ainda executa;
- se existe trabalho em andamento ou conflito.

O handoff é uma hipótese assinada pelo estado anterior, não uma autoridade superior ao repositório. Divergências devem ser apresentadas antes de modificar arquivos.

### Quando trocar de chat

Não existe fonte oficial que prove que toda Task deve equivaler exatamente a um chat. Codex suporta trabalhos longos e múltiplos agentes; a Anthropic recomenda persistir progresso antes de compactações de contexto.[^4][^5] Portanto, `1 Task = 1 chat` deve ser uma política local para previsibilidade, não uma alegação científica.

Decisão proporcional sugerida:

- trocar de sessão ao concluir ou bloquear uma Task;
- trocar antes se o contexto estiver degradado;
- não forçar troca durante uma unidade curta ainda íntegra;
- nunca usar o chat como única fonte de histórico.

## P04 — Mudanças e decisões

### Cadeia de rastreabilidade

```text
Requisito PRD
→ decisão/ADR quando necessário
→ Build
→ Task
→ commit ou PR
→ handoff com evidência
→ Release record
→ versão e artefato
```

### Quando criar ADR

Criar ADR para decisão que:

- afeta mais de um componente ou Build;
- impõe restrição futura;
- possui alternativas razoáveis;
- altera segurança, dados, compatibilidade ou operação;
- seria difícil compreender apenas pelo diff.

Decisão local reversível deve ficar no commit/handoff. Uma decisão substituída não é apagada: recebe status `Superseded by ADR-xxxx`.

Conventional Commits pode estruturar commits e sinalizar mudanças incompatíveis, mas não substitui changelog nem decisão arquitetural.[^6] O changelog deve ser curado para humanos e organizado por versão.

## P05 — Lock e recuperação

### Avaliação

Um `_ACTIVE_TASK.lock` pode indicar intenção, mas não garante exclusão mútua real e pode ficar órfão. A cópia indiscriminada em `.backup/` cria outra fonte de verdade e pode sobrescrever arquivos novos durante rollback.

### Política recomendada

- Projeto Git: usar branch/worktree e commits como recuperação primária.
- Lock: opcional; guardar Task, workspace/worktree, commit base, data e status.
- Backup por cópia: apenas para projeto sem Git ou arquivo externo não versionável.
- Nunca restaurar automaticamente sobre mudanças não classificadas.
- Recuperação começa com inspeção e comparação, não com sobrescrita.

Worktrees isolam árvores de trabalho para agentes paralelos; hooks podem executar validações no contexto do worktree, mas não devem ser a única barreira, pois hooks locais não são distribuídos automaticamente.[^7]

## P25 — Tamanho e envelhecimento do contexto

### Princípios

- manter arquivos ativos pequenos e orientados à decisão;
- arquivar detalhes e usar índices;
- carregar referências sob demanda;
- apontar para código e evidências, sem copiá-los;
- atualizar a fotografia atual e preservar seu histórico no Git;
- testar recuperação: um novo agente deve conseguir explicar estado e próxima ação.

ReadAgent, do Google DeepMind, mostra uma abordagem de memórias episódicas resumidas com retorno seletivo ao texto original, reforçando a ideia de resumo navegável mais fonte íntegra, em vez de contexto sempre crescente.[^8]

## Decisões propostas para o SOP

1. Git é a fonte de recuperação primária quando disponível.
2. Handoff é obrigatório, curto e verificável.
3. Estado atual, decisões, eventos e evidências ficam separados.
4. `1 Task = 1 chat` permanece uma política simples, mas deve permitir encerramento antecipado por degradação.
5. O novo agente verifica o estado real antes de confiar no handoff.
6. Limites de linhas fixos são substituídos por critérios de utilidade e arquivamento.

## Fontes

[^1]: OpenAI. [Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/). 2026.
[^2]: OpenAI. [An open-source spec for Codex orchestration: Symphony](https://openai.com/index/open-source-codex-orchestration-symphony/). 2026.
[^3]: Google Cloud. [Well-Architected Framework](https://docs.cloud.google.com/architecture/framework). Acesso em 2026.
[^4]: OpenAI. [Introducing the Agents API](https://openai.com/index/introducing-the-agents-api/). 2026.
[^5]: Anthropic. [Prompting best practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prompt-templates-and-variables). Acesso em 2026.
[^6]: Conventional Commits. [Specification 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).
[^7]: Git. [git-worktree documentation](https://git-scm.com/docs/git-worktree) e [githooks](https://git-scm.com/docs/githooks).
[^8]: Google DeepMind. [A Human-Inspired Reading Agent with Gist Memory of Very Long Contexts](https://deepmind.google/research/publications/74917/). 2024.

