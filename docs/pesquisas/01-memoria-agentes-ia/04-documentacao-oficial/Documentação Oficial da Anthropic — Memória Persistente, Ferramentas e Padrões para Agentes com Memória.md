# Documentação Oficial da Anthropic: Memória Persistente, Ferramentas e Padrões para Agentes com Memória

> **Fontes:** Exclusivamente documentação oficial da Anthropic (platform.claude.com/docs, support.claude.com, modelcontextprotocol.io, anthropic.com/engineering) e anúncios oficiais. Acesso em: julho de 2026.

***

## 1. O que a Anthropic Documenta Oficialmente sobre Memória Persistente

Por padrão, Claude **não possui memória entre sessões**. Cada nova conversa começa com um contexto completamente limpo. A Anthropic documenta oficialmente três camadas distintas de persistência de memória, cada uma com propósitos e mecanismos diferentes:[^1]

1. **Memória dentro de uma sessão (context window):** Claude lembra tudo o que aconteceu na conversa atual, mas esse contexto é perdido quando a sessão encerra.
2. **Memória gerenciada pelo usuário (claude.ai):** A partir de 2024, Claude.ai passou a oferecer um sistema de memória automática para usuários. Em março de 2026, esse recurso foi ativado para todos os planos, incluindo o gratuito. O sistema opera em dois modos: *busca de conversas anteriores* (disponível em planos pagos) e *geração automática de memórias* a partir do histórico de conversas.[^2][^1]
3. **Memória para desenvolvedores (API):** Dois mecanismos principais — a *Memory Tool* (disponível na Messages API) e os *Memory Stores* do Managed Agents (em beta).[^3][^4]

***

## 2. Existe Ferramenta Oficial de Memória?

**Sim.** A Anthropic documenta oficialmente **duas ferramentas distintas de memória**, com escopos diferentes:

### Memory Tool (Messages API — GA)
A *Memory Tool* (`memory_20250818`) é uma ferramenta **geralmente disponível** (sem necessidade de beta header) na Messages API. Ela permite que Claude armazene e recupere informações entre conversas através de operações em arquivos. **Importante: a ferramenta opera no lado do cliente.** Claude apenas solicita operações de arquivo; a aplicação do desenvolvedor é responsável por executá-las e controlar onde os dados são armazenados.[^3]

### Memory Stores (Managed Agents — Beta)
Os *Memory Stores* são um recurso em **beta** exclusivo da infraestrutura Managed Agents, acessível através do header `managed-agents-2026-04-01`. Diferentemente da Memory Tool, os Memory Stores são gerenciados pela própria infraestrutura da Anthropic, com persistência server-side e versionamento automático.[^4]

***

## 3. Como Cada Ferramenta Funciona

### Memory Tool — Funcionamento Detalhado
Quando a Memory Tool está habilitada, Claude automaticamente verifica o diretório `/memories` antes de iniciar uma tarefa. Durante o trabalho, armazena o que aprende em arquivos e os recupera em conversas posteriores.[^3]

**Comandos suportados:**
- `view` — visualiza diretório ou conteúdo de arquivo
- `create` — cria novo arquivo de memória
- `str_replace` — substitui texto em arquivo existente
- `insert` — insere texto em linha específica
- `delete` — remove arquivo ou diretório
- `rename` — renomeia ou move arquivo

O fluxo típico é: Claude faz uma chamada de ferramenta → a aplicação executa a operação no armazenamento que ela controla → retorna o resultado em um bloco `tool_result`.[^3]

**Adição automática ao system prompt:** Quando a Memory Tool está presente na requisição, a API automaticamente adiciona instruções ao system prompt orientando Claude a sempre verificar o diretório de memória antes de qualquer ação.[^3]

### Memory Stores (Managed Agents) — Funcionamento Detalhado
Um *memory store* é uma coleção de documentos de texto com escopo de workspace, otimizada para Claude. Quando anexado a uma sessão, é montado como diretório dentro do sandbox da sessão em `/mnt/memory/`. O agente lê e escreve nele com as mesmas ferramentas de arquivo que usa para o restante do filesystem.[^4]

**Ciclo de vida:**
1. Criar o store com `name` e `description`
2. Opcionalmente semear com conteúdo de referência
3. Anexar ao criar uma sessão via `resources[]`
4. O agente opera nos arquivos normalmente; escrita em `/mnt/memory/` é persistida automaticamente
5. Cada mudança cria uma *memory version* imutável para auditoria

***

## 4. Onde as Memórias Ficam Armazenadas

| Mecanismo | Localização | Controle |
|-----------|-------------|----------|
| Memory Tool | Onde o **desenvolvedor implementar** (filesystem, banco de dados, cloud storage, arquivos criptografados) | Total controle do desenvolvedor |
| Memory Stores (Managed Agents) | **Infraestrutura Anthropic** (workspace-scoped, montada em `/mnt/memory/`) | Anthropic gerencia; acesso via API |
| CLAUDE.md (Claude Code) | Arquivo no **repositório do projeto** ou `~/.claude/CLAUDE.md` | Controlado pelo time/usuário |
| Memória claude.ai | **Servidores Anthropic** (associada à conta do usuário) | Controlado pelo usuário via UI |
| Prompt Caching | **Memória em tempo de execução** (KV cache, não persistido em disco) | Anthropic; TTL de 5min ou 1h |

A Memory Tool é explicitamente *client-side*: "Memory lives entirely in your application". Os Memory Stores do Managed Agents ficam server-side na infraestrutura Anthropic, com isolamento por workspace.[^3]

***

## 5. Como Memórias São Criadas, Atualizadas e Recuperadas

### Memory Tool
- **Criação:** Claude emite chamada `create` para `/memories/nome-do-arquivo.md`; o handler do desenvolvedor cria o arquivo no armazenamento escolhido.
- **Atualização:** Claude usa `str_replace` (substitução cirúrgica de texto) ou `insert` (inserção em linha específica). O comando `str_replace` suporta precondição de hash SHA256 para evitar conflitos de escrita concorrente.[^3]
- **Recuperação:** Claude emite `view /memories` para listar o diretório, depois `view /memories/arquivo.md` para ler o conteúdo. O sistema retorna o conteúdo com números de linha.
- **Sincronização entre sessões:** A memória é a mesma quando se usa a mesma entrada `tools` e o handler serve o mesmo store.[^3]

### Memory Stores (Managed Agents)
- **Criação:** `memories.create` cria memória em path específico. Não sobrescreve — para alterar existente, usa `memories.update`.[^4]
- **Atualização:** `memories.update` modifica conteúdo ou renomeia (move) por ID. Suporta `content_sha256` como precondição de concorrência otimista.[^4]
- **Recuperação:** `memories.retrieve` retorna conteúdo completo; `memories.list` lista com filtro por `path_prefix`.[^4]
- **Auditoria:** Cada mutação cria uma `memory version` imutável (`memver_...`). As versões são mantidas por 30 dias; versões recentes são preservadas independente da idade.[^4]
- **Rollback:** Não há endpoint de restore dedicado; para reverter, recupera a versão desejada e reescreve com `memories.update`.[^4]

***

## 6. Limitações e Riscos Documentados

### Memory Tool
- **Sem armazenamento nativo:** O desenvolvedor deve implementar toda a infraestrutura de armazenamento.[^3]
- **Path traversal:** Um path malicioso como `/memories/../../secrets.env` pode acessar arquivos fora do diretório permitido. O handler deve validar todos os paths.[^3]
- **Tamanho de arquivo:** Sem controle nativo de tamanho; o desenvolvedor deve implementar limites e paginação.[^3]
- **Expiração:** Sem expiração automática; o desenvolvedor deve implementar limpeza periódica.[^3]
- **Dados sensíveis:** Claude normalmente recusa escrever dados sensíveis, mas para garantias mais fortes deve-se adicionar validação no handler.[^3]

### Memory Stores (Managed Agents)
- **Limite de 2.000 memórias por store:** Quando atingido, escrita em novas memórias falha (memórias existentes permanecem legíveis).[^4]
- **Limite de 100 kB por memória:** Individual memories são limitadas a ~25k tokens.[^4]
- **Máximo de 8 stores por sessão**.[^4]
- **Prompt injection em stores read-write:** Se o agente processa input não confiável, uma injeção bem-sucedida pode escrever conteúdo malicioso no store, que futuras sessões lerão como memória confiável. A recomendação é usar `read_only` para material de referência.[^4]
- **Memory stores não podem ser adicionados a sessões em execução** — apenas durante a criação.[^4]
- **Versões são mantidas por apenas 30 dias** — exportar via API para histórico mais longo.[^4]
- **Não elegível para ZDR ou HIPAA BAA** (Managed Agents é stateful by design).[^4]

### Riscos de Segurança Comprovados (Memory Poisoning)
Pesquisadores da Cisco divulgaram, em março de 2026, uma vulnerabilidade de *memory poisoning* no Claude Code onde scripts maliciosos podiam modificar arquivos MEMORY.md e CLAUDE.md para injetar instruções persistentes maliciosas. A Anthropic corrigiu na versão v2.1.50, removendo arquivos de memória do usuário do system prompt. A Anthropic esclareceu que "o principal do usuário na máquina é considerado totalmente confiável" e que usuários são responsáveis por valar dependências introduzidas no ambiente.[^5]

### Riscos Gerais de Prompt Injection em Agentes
A Anthropic reconhece formalmente que prompt injection representa um dos maiores desafios de segurança para agentes baseados em browser e computer use. Mesmo com treinamento de resistência e classifiers automáticos, a taxa de sucesso de ataques não é zero. Para Computer Use, a documentação oficial lista múltiplos riscos incluindo: latência elevada, alucinação de coordenadas, falha de seleção de ferramentas, e vulnerabilidades a prompt injection via conteúdo em páginas web.[^6][^7]

***

## 7. Como Isso Se Aplica a Projetos de Software

### Padrão Multi-sessão para Desenvolvimento de Software (Memory Tool)
A documentação oficial descreve explicitamente o padrão de "recovery mechanism" para projetos de software:

1. **Sessão inicializadora:** A primeira sessão configura arquivos de memória antes de qualquer trabalho substantivo — log de progresso, checklist de features, referência a script de inicialização.[^3]
2. **Sessões subsequentes:** Cada nova sessão abre os arquivos de memória para restaurar o estado do projeto sem re-explorar o codebase.[^3]
3. **Atualização ao final de sessão:** Antes de encerrar, a sessão atualiza o log de progresso com o que foi completado e o que falta.[^3]

**Princípio chave:** Trabalhar em uma feature por vez; marcar como completa somente após verificação end-to-end.[^3]

### Claude Code e CLAUDE.md
Para Claude Code, a memória de projeto funciona através do arquivo `CLAUDE.md`:[^8]

| Tipo de Memória | Localização | Propósito |
|----------------|-------------|-----------|
| Project memory | `./CLAUDE.md` | Instruções compartilhadas pelo time |
| User memory | `~/.claude/CLAUDE.md` | Preferências pessoais para todos os projetos |
| Project memory (local) | `./CLAUDE.local.md` | Preferências pessoais por projeto (deprecado) |

Todos os arquivos de memória são automaticamente carregados no contexto quando o Claude Code é iniciado. CLAUDE.md suporta importação de outros arquivos com sintaxe `@path/to/file`.[^8]

### Integração Memory Tool + Context Editing
A documentação oficial recomenda combinar a Memory Tool com Context Editing para workflows de longa duração. O Context Editing remove automaticamente resultados de ferramentas antigos do contexto, e Claude recebe aviso automático para salvar informações importantes em memória antes que sejam removidas. Isso permite workflows que excederiam os limites da janela de contexto.

***

## 8. Diferença Entre Projects, Arquivos, MCP, Tools e Memória Externa Própria

### Tabela Comparativa: Recursos Oficiais de "Memória" para Projetos de Software

| Recurso | Finalidade | Persistência | Escopo | Limitações | Aplicabilidade para "Memória de Projeto" |
|---------|-----------|--------------|--------|-----------|------------------------------------------|
| **Projects (claude.ai)** [^9][^10] | Workspace com histórico de chats, knowledge base e instruções customizadas | Permanente até deletar | Usuário/time (planos pagos permitem colaboração) | Interface apenas; sem acesso API programático; RAG somente em planos pagos | ✅ Ideal para times usando claude.ai como interface; não para integração de código |
| **Memory Tool** (`memory_20250818`) [^3] | Agente armazena/recupera conhecimento acumulado entre sessões via arquivos | Definida pelo desenvolvedor | Por agente/usuário/projeto (como o dev configurar) | Developer responsável por toda infraestrutura; risco de path traversal | ✅ Excelente para desenvolvimento de software; total controle; compatível com ZDR |
| **Memory Stores (Managed Agents)** [^4] | Store server-side para agentes com versioning e auditoria | 30 dias para versões; store até arquivar/deletar | Workspace-scoped; até 8 stores/sessão | Beta; não elegível para ZDR/HIPAA; limite de 2.000 memórias e 100kB/memória | ✅ Ideal para agentes autônomos de longa duração em infraestrutura Anthropic |
| **CLAUDE.md / Claude Code** [^8] | Instruções persistentes de projeto para Claude Code | Permanente (arquivo em disco/repositório) | Projeto (CLAUDE.md) ou usuário (~/.claude/CLAUDE.md) | Específico para Claude Code; risco de memory poisoning via repositórios maliciosos | ✅ Perfeito para projetos de código; versionado com o repositório |
| **MCP (Model Context Protocol)** [^11][^12] | Protocolo padrão aberto para conectar Claude a sistemas externos (dados, ferramentas, workflows) | Definida pelo servidor MCP | Qualquer — depende do servidor implementado | Requer implementação de servidor MCP; autenticação OAuth; timeouts de 5min | ✅ Para conectar a bases de dados, sistemas de arquivos, APIs externas com memória |
| **Prompt Caching** [^13] | Cache de prefixos de prompt para reduzir custo e latência | Ephemeral: 5 min (padrão) ou 1h | Por workspace (isolado desde fev/2026) | Não é memória semântica; apenas cache de tokens; mínimo de tokens exigido | ⚠️ Não é memória persistente real; reduz custo em conversas longas |
| **Context Editing**  | Remove seletivamente conteúdo antigo do contexto (tool results, thinking blocks) | N/A — gerenciamento de contexto ativo | Por sessão | Beta; invalida cache ao limpar; não preserva informação removida | ✅ Complementar à Memory Tool para sessões longas de desenvolvimento |
| **Memória Externa Própria (implementação do desenvolvedor)** | Banco de dados vetorial, SQL, key-value store externo | Definida pelo desenvolvedor | Qualquer estrutura desejada | 100% responsabilidade do desenvolvedor; sem abstração oficial | ✅✅ Maior flexibilidade; usar com Memory Tool como interface para Claude |
| **Artifacts (claude.ai)** [^14][^15] | Conteúdo substancial gerado por Claude (código, documentos, apps interativos) | Até publicar/deletar; armazenamento persistente de 20MB por artifact (planos pagos) | Por usuário; pode ser compartilhado publicamente | 20MB por artifact; somente texto; apps AI-powered em beta | ⚠️ Para outputs de projeto, não para memória operacional do agente |

***

## 9. O que É Documentado vs. o que Precisa Ser Implementado pelo Desenvolvedor

### O que a Anthropic Documenta e Fornece Prontos
- **Especificação completa da Memory Tool** com comandos, formatos de retorno, tratamento de erros e helpers de SDK para Python, TypeScript, C#, Java[^3]
- **Memory Stores API** com CRUD completo, versionamento e redação de versões sensíveis[^4]
- **Prompt Caching** com cache automático e explícito, TTL de 5min e 1h[^13]
- **Context Editing** para limpeza de tool results e thinking blocks
- **Compaction server-side** para sumarização automática de conversas longas
- **MCP specification e SDKs** em Python e TypeScript[^11]
- **Computer Use Tool** com sandbox de referência em Docker[^6]
- **CLAUDE.md** para memória de projeto no Claude Code[^8]
- **Padrões documentados** para agentes multi-sessão de desenvolvimento de software[^3]
- **Referência de implementação** com `BetaLocalFilesystemMemoryTool` para uso imediato com filesystem local[^3]

### O que o Desenvolvedor Precisa Implementar
- **Handler de armazenamento para a Memory Tool** — onde e como os arquivos de memória são guardados (filesystem, banco de dados, S3, etc.)[^3]
- **Validação de path traversal** — nenhuma proteção automática é fornecida; o desenvolvedor deve implementar[^3]
- **Limites de tamanho de arquivo** — controle e paginação de arquivos grandes[^3]
- **Expiração de memórias** — deletar periodicamente memórias não acessadas[^3]
- **Filtragem de dados sensíveis** — validação antes de gravar no handler[^3]
- **Lógica de retrieval semântica** — a Memory Tool é baseada em filesystem; busca semântica/vetorial requer implementação própria (ou uso de MCP conectando a banco vetorial)[^3]
- **Autenticação e isolamento por usuário/projeto** — o handler deve mapear `/memories` para o store correto por contexto[^3]
- **Infraestrutura MCP** — se usar MCP, desenvolver o servidor MCP que expõe memória/ferramentas[^11]
- **Estratégia de backup e retenção** para Memory Stores além de 30 dias[^4]
- **Lógica de consolidação ("dreaming")** — para condensar memórias fragmentadas (disponível em preview no Managed Agents, mas a estratégia é do desenvolvedor)[^4]

***

## 10. Fontes Oficiais Referenciadas

| Documento | URL | Tipo |
|-----------|-----|------|
| Memory Tool Documentation | `platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool` | Docs oficiais API |
| Managed Agents Memory | `platform.claude.com/docs/en/managed-agents/memory` | Docs oficiais API (Beta) |
| Managed Agents Overview | `platform.claude.com/docs/en/managed-agents/overview` | Docs oficiais API |
| Prompt Caching | `platform.claude.com/docs/en/build-with-claude/prompt-caching` | Docs oficiais API |
| Context Editing | `platform.claude.com/docs/en/build-with-claude/context-editing` | Docs oficiais API |
| Computer Use Tool | `platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool` | Docs oficiais API (Beta) |
| Artifacts (claude.ai) | `support.claude.com/en/articles/9487310-what-are-artifacts-and-how-do-i-use-them` | Help Center oficial |
| Projects (claude.ai) | `support.claude.com/en/articles/9517075-what-are-projects` | Help Center oficial |
| Introducing MCP | `anthropic.com/news/model-context-protocol` | Anúncio oficial |
| MCP Specification | `modelcontextprotocol.io/specification/2025-06-18` | Spec oficial |
| MCP Introduction | `modelcontextprotocol.io/introduction` | Docs oficiais MCP |
| Building Effective AI Agents | `anthropic.com/engineering/building-effective-agents` | Blog de Engenharia |
| Managed Agents Engineering | `anthropic.com/engineering/managed-agents` | Blog de Engenharia |
| Prompt Injection Defenses | `anthropic.com/news/prompt-injection-defenses` | Artigo oficial |
| CLAUDE.md (Claude Code) | `code.claude.com/docs/en/memory` | Docs oficiais Claude Code |
| Models Overview | `platform.claude.com/docs/en/about-claude/models/overview` | Docs oficiais API |

> **Nota:** Todos os documentos acima foram acessados em julho de 2026. Recursos marcados como "Beta" podem ter comportamentos refinados entre versões. O header `managed-agents-2026-04-01` é necessário para todos os endpoints de Managed Agents.[^4]

---

## References

1. [Claude Memory — How to Make Claude Remember and Manage It](https://usingclaude.com/en/guides/features/claude-memory-guide) - How Claude's memory works — search past chats, auto memory, turning it on/off, managing it, project ...

2. [Claude Memory 2026: Complete Guide — What It Stores, How to ...](https://lumichats.com/blog/claude-memory-2026-complete-guide-how-to-use) - Anthropic activated memory for all Claude users (free and Pro) in March 2026. Claude now remembers y...

3. [Memory tool - Claude Docsdocs.claude.com › docs › agents-and-tools › tool-use › memory-tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) - Let Claude store and retrieve information across conversations by implementing the memory tool's fil...

4. [Claude Memory Architecture Explained: How Persistent ...](https://mer.vin/2026/05/claude-memory-architecture-explained-how-persistent-context-makes-coding-agents-reliable/) - Most agent failures in long-running workflows come from context drift, not model weakness. The pract...

5. [Disclosure](https://blogs.cisco.com/ai/identifying-and-remediating-a-persistent-memory-compromise-in-claude-code) - We recently discovered a method to compromise Claude Code’s memory and maintain persistence beyond o...

6. [Computer use tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool) - Claude API Documentation.

7. [Mitigating the risk of prompt injections in browser use - Anthropic](https://www.anthropic.com/news/prompt-injection-defenses) - Anthropic is an AI safety and research company that's working to build reliable, interpretable, and ...

8. [jsonl/en__docs__claude-code__memory.jsonl ...](https://huggingface.co/datasets/ratanon/claude-code/blob/main/jsonl/en__docs__claude-code__memory.jsonl) - We’re on a journey to advance and democratize artificial intelligence through open source and open s...

9. [Collaborate with Claude on Projects - Anthropic](https://www.anthropic.com/news/projects) - Claude Pro and Team users can now organize chats into Projects. Projects bring together internal kno...

10. [Claude Projects Feature Guide: Complete Setup Tutorial (2025)](https://aionx.co/claude-ai-reviews/claude-projects-feature-guide/) - Learn how to use Claude Projects effectively. Complete guide covering setup, custom instructions, kn...

11. [Introducing the Model Context Protocol - Anthropic](https://www.anthropic.com/news/model-context-protocol) - The Model Context Protocol (MCP) is an open standard for connecting AI assistants to the systems whe...

12. [Specification - Model Context Protocol](https://modelcontextprotocol.io/specification/2025-06-18)

13. [Prompt caching with Claude](https://www.anthropic.com/news/prompt-caching?s=09) - Prompt caching, which enables developers to cache frequently used context between API calls, is now ...

14. [What are artifacts and how do I use them? - Claude Help Center](https://support.claude.com/en/articles/9487310-what-are-artifacts-and-how-do-i-use-them)

15. [Build and Host AI-Powered Apps with Claude - No Deployment Neededwww.anthropic.com › news › claude-powered-artifacts](https://www.anthropic.com/news/claude-powered-artifacts?subjects=alignment) - Create interactive AI apps directly in Claude. Users pay for their own API usage, you pay nothing. B...

