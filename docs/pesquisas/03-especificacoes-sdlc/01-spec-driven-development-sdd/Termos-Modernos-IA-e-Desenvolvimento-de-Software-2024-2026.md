# Termos Modernos, IA e Desenvolvimento de Software (2024–2026)

## Resumo executivo

O período 2024–2026 consolidou uma mudança real, mas ainda instável semanticamente, na forma como software é construído com IA generativa. Não existe uma "metodologia" unificada: há um punhado de práticas com definição relativamente estável (agentic coding, coding agents, AI pair programming), termos guarda-chuva que diferentes empresas usam com sentidos distintos (specification-driven development, context engineering), um termo de origem clara e uso já deturpado da intenção original (vibe coding), e vários rótulos de marketing sem definição consistente (AI-native software development, autonomous software engineering). A evidência científica é mista e por vezes contraditória: estudos de campo mostram ganhos de produtividade em métricas de output (PRs, commits), enquanto um RCT rigoroso da METR encontrou desaceleração de 19% em tarefas reais de desenvolvedores experientes em bases de código que já conheciam. Paralelamente, o benchmark mais citado do setor, SWE-bench Verified, foi declarado inválido pela própria OpenAI em fevereiro de 2026 por contaminação de dados e testes malformados.[^1][^2][^3][^4][^5][^6][^7][^8]

## Glossário e tabela de termos

| Termo | Definição resumida | Origem/data | Classificação |
|---|---|---|---|
| AI-assisted software development | Guarda-chuva amplo para qualquer uso de IA generativa no ciclo de desenvolvimento | Uso difuso desde 2022 (Copilot) | Industrial, consolidado como categoria genérica |
| AI coding assistants | Ferramentas que sugerem/completam código (Copilot, Cursor, Codeium) | 2021 (Copilot) em diante | Industrial, consolidado |
| AI-native software development | Software projetado assumindo IA como componente arquitetural central | Marketing recente, sem definição formal única | Marketing, ambíguo |
| Agentic coding / coding agents | Sistemas de IA que planejam, executam múltiplas etapas e usam ferramentas (terminal, testes, arquivos) autonomamente, não só autocompletam | ~2024–2025, popularizado por Claude Code, Devin, Cursor Agent | Industrial, emergente, com uso técnico consistente |
| Software engineering agents | Sinônimo técnico/acadêmico de coding agents, usado em benchmarks (SWE-bench) | 2023–2024, literatura acadêmica | Acadêmico + industrial, emergente |
| Specification-driven development (SDD) | Especificação torna-se artefato executável central; código deriva da spec | Termo pré-IA reaproveitado; popularizado por GitHub Spec Kit (set/2025) [^9] | Ambíguo — reaproveitamento de termo antigo, sem consenso único |
| Prompt-driven development | Construir software primariamente via prompts em linguagem natural | ~2023, uso informal | Meme/comunidade, sem definição formal |
| Context engineering | Disciplina de projetar todo o ambiente informacional (memória, RAG, ferramentas, histórico) que o modelo vê, não apenas o prompt | Popularizado em 2025 por praticantes como Tobi Lütke (Shopify) e comunidade de agentes [^10] | Industrial, emergente, crescente formalização |
| Agentic software engineering | Engenharia de software realizada majoritariamente por agentes autônomos com supervisão humana | 2025–2026 | Industrial/acadêmico, emergente |
| Vibe coding | Programar "confiando nas vibes", aceitando código gerado sem revisão linha a linha, guiando por resultados/prompts | Cunhado por Andrej Karpathy, fevereiro de 2025 [^11] | Consolidado quanto à origem; uso ampliado e criticado |
| AI pair programming | Metáfora de par-programação humano-IA, análoga a pair programming clássico | Popularizado com GitHub Copilot (2021) | Industrial, consolidado |
| Autonomous software engineering | Cenário aspiracional de agentes completando tarefas de engenharia fim a fim sem supervisão | 2024–2026, mais aspiracional que realizado | Marketing/experimental, ambíguo |
| Repository-level agents | Agentes que operam sobre todo um repositório (não só um arquivo/trecho) | 2024, literatura de benchmarks (SWE-bench) | Acadêmico, emergente |
| Multi-agent software development | Múltiplos agentes especializados colaborando (planejador, codificador, revisor) | 2025–2026 | Experimental, emergente |
| Requirements-to-code workflows | Pipelines que convertem requisitos em NL diretamente em código executável | Conceito antigo (model-driven dev) reaproveitado | Industrial, ambíguo |
| Test generation with LLMs | Geração automática de testes unitários/integração via LLM | 2023–2026, tema acadêmico ativo | Acadêmico + industrial, emergente |
| AI-generated code review | Revisão automatizada de PRs por IA (CodeRabbit, Copilot Review, Claude Code Review) | GA a partir de abril/2025 [^12] | Industrial, consolidado como categoria de produto |
| AI-assisted debugging/testing | Uso de IA para localizar bugs, gerar casos de teste, executar TDD/BDD assistido | 2023–2026 | Industrial, emergente |
| Natural-language-to-code | Tradução direta de linguagem natural para código executável | Conceito clássico (desde anos 1980) revivido pela IA generativa | Acadêmico, consolidado como linha de pesquisa |
| Executable specifications | Especificações formais o suficiente para gerar/verificar código automaticamente | Termo técnico pré-IA (formal methods), reaproveitado por SDD [^13] | Acadêmico, ambíguo no novo contexto |
| AI-assisted TDD/BDD | Uso de IA para gerar testes antes do código (TDD) ou cenários Gherkin (BDD) | 2024–2026 | Industrial, experimental |

## Specification-Driven Development: origem e reapropriação

O termo "specification-driven development" já existia antes da IA generativa, associado a métodos formais e desenvolvimento orientado a modelos, mas seu uso maciço atual decorre do GitHub Spec Kit, lançado como toolkit open source em setembro de 2025. A proposta do GitHub inverte a lógica clássica: em vez da especificação ser descartada após o "trabalho real" de codificar, ela se torna um artefato executável que gera diretamente a implementação, seguindo o fluxo Specify → Plan → Tasks → Implement. Até maio de 2026, o Spec Kit acumulava mais de 90 mil estrelas no GitHub e suporte a mais de 30 agentes de codificação, incluindo Copilot, Claude Code, Gemini CLI e Codex.[^14][^15][^9][^16][^17][^18]

Diferentes organizações usam a expressão com sentidos distintos: a Microsoft/GitHub enfatiza "version control for your thinking" e decisões arquiteturais explícitas; a AWS Kiro e outras ferramentas convergem no mesmo padrão de quatro artefatos (requirements.md, design.md, tasks.md, AGENTS.md), mas com ênfase em gates de revisão humana; extensões comunitárias como AIDE adicionam camadas de "visão" e "roadmap" separadas do núcleo spec-plan-tasks. Um artigo acadêmico recente (arXiv, 2026) trata SDD como um princípio geral — "o código é o detalhe de implementação da especificação, não o contrário" — e cita estudos preliminares (ainda nascentes) sugerindo redução de até 50% em erros quando specs refinadas por humanos guiam agentes. Não existe, porém, consenso terminológico firme: o próprio ecossistema mistura SDD com "spec-first" e "prompt-driven development" de forma intercambiável em textos de blog, sem definição normativa única.[^19][^18][^20][^13]

## Context Engineering: origem, popularização e diferença de Prompt Engineering

Context engineering ganhou tração em 2025 como reação à percepção de que "prompt engineering" era insuficiente para sistemas de agentes com múltiplas etapas, memória e ferramentas. A formulação mais citada — "a arte de fornecer todo o contexto necessário para que a tarefa seja plausivelmente resolvível pelo LLM" — é atribuída a Tobi Lütke (Shopify). Enquanto prompt engineering trata de "como fraseio a instrução" em uma única interação, context engineering trata de "o que o modelo precisa saber agora", abrangendo documentos recuperados (RAG), memória persistente, saídas de ferramentas, histórico de conversa e metadados estruturados. A relação entre os dois é hierárquica e não de substituição: prompt engineering é tratado como subconjunto de context engineering na maioria das fontes analisadas.[^21][^22][^23][^10][^24][^25]

O grau de formalização acadêmica ainda é baixo — a maior parte da literatura disponível é de blogs de vendors (Elastic, Redis, Glean, deepset) e não papers revisados por pares, o que sugere um termo em fase de consolidação industrial, não ainda um conceito com taxonomia acadêmica estável. Isso o classifica como emergente/industrial, com uso consistente entre fontes, mas sem base de pesquisa formal equivalente à de "prompt engineering".[^26][^27][^28][^24][^21]

## Vibe Coding: origem, evolução e riscos

O termo foi cunhado por Andrej Karpathy, cofundador da OpenAI e ex-líder de IA da Tesla, em fevereiro de 2025, descrevendo a prática de "se entregar totalmente às vibes, abraçar exponenciais e esquecer que o código existe". No sentido original, Karpathy referia-se explicitamente a projetos descartáveis ("throwaway projects") em que o desenvolvedor aceita código gerado sem revisão linha a linha, confiando em resultados e prompts de acompanhamento. A expressão evoluiu de conceito de nicho para rótulo popular aplicado (incorretamente, segundo críticos) a fluxos de produção reais, o que motivou o contraste explícito com Spec-Driven Development em materiais do GitHub e da comunidade — "vibe coding cada nova feature" é citado como o antimodelo que a SDD busca substituir.[^29][^18][^11][^30]

As evidências de risco acumuladas em 2025–2026 são concretas e mensuráveis: um estudo da Tenzai (dezembro de 2025) encontrou vulnerabilidades de SSRF em 5 de 5 agentes de codificação testados na mesma funcionalidade; a IBM documentou que 64% das credenciais expostas em aplicações vibe-coded permaneciam ativas em janeiro de 2026; e uma nota de pesquisa da Cloud Security Alliance (abril de 2026), sobre 2,23 milhões de amostras de código gerado por IA em 16 modelos, encontrou 19,7% com ao menos um pacote de dependência inexistente (risco de "slopsquatting"). Análises da Veracode de março de 2026 mostraram taxa de aprovação em testes de segurança estagnada em torno de 55%, sem melhoria mesmo com modelos mais avançados. O status do termo permanece informal — não há definição normativa por nenhuma organização de padrões, apenas o uso consistente da definição original de Karpathy nas fontes analisadas.[^30]

## Evidência científica sobre produtividade, qualidade e segurança

Os resultados quantitativos disponíveis são heterogêneos e dependem fortemente de metodologia e contexto:

- **Estudo de campo com Microsoft e Accenture (MIT, publicado em pubpub, março de 2024)**: 1.974 desenvolvedores, ganho de 12,92% a 21,83% em PRs/semana na Microsoft e 7,51% a 8,69% na Accenture — evidência observacional/experimental sugestiva, não RCT puro descrito como definitivo pelos autores.[^4]
- **RCT da METR (julho de 2025)**: 16 desenvolvedores open-source experientes, 246 tarefas reais, randomizadas entre "IA permitida" e "IA proibida"; resultado: desaceleração de 19% (IC 95%: +2% a +39%), apesar da expectativa prévia de 24% de aceleração. Este é o estudo controlado mais rigoroso disponível e contradiz a narrativa de ganho universal de produtividade.[^6][^31][^32][^7]
- **Estudo de continuação da METR (fevereiro de 2026)**: a própria METR abandonou o desenho do experimento por efeitos de seleção — desenvolvedores evitavam participar de tarefas "sem IA" — tornando os novos resultados (variando de -18% a +9%) estatisticamente não confiáveis.[^33][^6]
- **Estudo arXiv (setembro de 2025) sobre atividade de desenvolvedores com/sem Copilot**: não encontrou mudança estatisticamente significativa em métricas baseadas em commits após adoção, embora usuários de Copilot já fossem mais ativos antes da adoção (viés de seleção).[^34]
- **AMCIS 2024 / Ziegler et al. (Communications of the ACM, 2024)**: relatam melhorias observadas em throughput, tempo de ciclo e defeitos em estudo de caso, mas com metodologia de survey/case study, não RCT.[^35]

A distinção metodológica é crucial: apenas o estudo da METR é um experimento controlado randomizado; os demais são estudos observacionais, relatórios comerciais (Veracode, IBM) ou pesquisas de opinião/autorrelato — e resultados de autorrelato sistematicamente sobrestimam o ganho real, segundo a própria METR.[^7][^8]

## Benchmarks: o que medem e por que falharam

SWE-bench mede a capacidade de um modelo/agente de resolver issues reais do GitHub, aplicando patches gerados em containers Docker e executando a suíte de testes original do repositório (critério FAIL_TO_PASS e PASS_TO_PASS). Em fevereiro de 2026, a OpenAI publicou uma análise declarando SWE-bench Verified "cada vez mais contaminado" e anunciou que deixaria de reportar essa métrica. Dois problemas centrais foram identificados:[^2][^36]

1. **Testes malformados**: 59,4% de uma amostra auditada continha testes excessivamente restritivos (exigindo nomes de função específicos nunca mencionados no problema) ou testes que checavam funcionalidades não especificadas.[^2]
2. **Memorização de treinamento**: todos os modelos de fronteira testados conseguiram reproduzir verbatim patches de referência ("gold patches"), indicando exposição prévia aos dados durante o treinamento — o que infla artificialmente os escores.[^37][^2]

A lacuna entre SWE-bench Verified (70–81% para modelos de fronteira) e seu substituto resistente à contaminação, SWE-bench Pro (23–34% para os mesmos modelos), é a evidência mais direta da inflação. Outros estudos independentes (LessLeak-Bench, arXiv 2512.10218; SWE-Bench Illusion, arXiv 2506.12286) confirmam padrões de memorização, com modelos apresentando desempenho de 3 a 6 vezes melhor em tarefas dentro do benchmark do que em repositórios externos equivalentes. Mesmo o substituto SWE-bench Pro não é imune: uma auditoria da DeepSWE encontrou taxa de erro de verificador de 32% (falsos positivos e negativos combinados), e o METR mostrou que metade dos PRs "aprovados" pelo SWE-bench nunca seria de fato aceita em merge para a branch principal. A lição metodológica geral: benchmarks construídos a partir de dados públicos (issues e PRs do GitHub) carregam risco estrutural de contaminação, e comparações entre agentes só são válidas quando o "scaffolding" (arquitetura do agente) é mantido constante — caso contrário, a pontuação reflete a engenharia do harness, não a capacidade do modelo subjacente.[^38][^36][^3][^1][^37]

## AI-generated code review e AI-assisted testing

Ferramentas de revisão de código por IA se tornaram uma categoria de produto madura em 2025–2026: GitHub Copilot Code Review atingiu disponibilidade geral em abril de 2025 e 1 milhão de usuários em um mês. Testes independentes (Greptile, julho de 2025; Tenki, 2026) mostram taxas de detecção de bugs reais bastante variáveis entre ferramentas — de 82% (Greptile) a 24,6% (GitHub Copilot) dependendo da metodologia de teste, evidenciando falta de padronização nas avaliações comerciais. Quanto à geração de testes por LLM, a literatura acadêmica trata o tema como ativo mas ainda emergente, sem consenso sobre cobertura ou qualidade equivalente à escrita humana; a maior parte da evidência disponível é de estudos de caso e relatórios de vendors, não RCTs.[^12][^39]

## Linha do tempo 2022–2026

| Período | Evento |
|---|---|
| 2021–2022 | Lançamento do GitHub Copilot; Karpathy declara "the hottest new programming language is English" (2023) [^11] |
| Nov/2023 | SWE-bench é publicado como benchmark acadêmico para avaliação de agentes de engenharia de software |
| Mar/2024 | Estudo MIT/Microsoft/Accenture sobre produtividade com Copilot (preprint) [^4] |
| 2024 | Surgimento e popularização de "coding agents" e "software engineering agents" na literatura acadêmica; SWE-bench Verified lançado como subconjunto validado por humanos |
| Fev/2025 | Andrej Karpathy cunha "vibe coding" [^11] |
| Fev–Jun/2025 | METR conduz RCT sobre produtividade com IA (resultado: -19%) [^31] |
| Abr/2025 | GitHub Copilot Code Review atinge disponibilidade geral [^12] |
| Jul/2025 | METR publica resultado do RCT (slowdown de 19%), amplamente divulgado pela Reuters [^32] |
| Meados 2025 | Popularização de "context engineering" como termo distinto de "prompt engineering" [^10] |
| Set/2025 | GitHub lança Spec Kit e populariza "Spec-Driven Development" no contexto de agentes [^9][^18] |
| Dez/2025 | Fundação da Agentic AI Foundation (Linux Foundation) e padrão AGENTS.md; MCP da Anthropic já lançado [^19] |
| Fev/2026 | OpenAI declara SWE-bench Verified inválido por contaminação; recomenda SWE-bench Pro [^2] |
| Fev/2026 | METR anuncia que seu experimento de continuação é estatisticamente não confiável por viés de seleção [^33] |
| 2026 (ao longo do ano) | Proliferação de ferramentas de "AI code review" (CodeRabbit, Greptile, Claude Code Review, Codex Security); consolidação de críticas de segurança ao vibe coding (Veracode, IBM, Cloud Security Alliance) [^30][^12] |

## Relação com práticas clássicas (TDD/BDD/DDD)

As novas práticas não substituem TDD, BDD e DDD — elas os reinterpretam sob supervisão de agentes. No Spec Kit, a fase "Tasks" é explicitamente comparada a um "processo de test-driven development para o agente de IA", em que cada tarefa deve ser implementável e testável isoladamente antes de avançar. "AI-assisted TDD/BDD" propõe que o agente gere os testes antes da implementação (mantendo a lógica clássica de TDD) ou cenários Gherkin a partir de especificações em linguagem natural (BDD), mas a validação empírica dessas variantes ainda é escassa e majoritariamente anedótica. Domain-Driven Development (DDD) aparece de forma indireta: specs executáveis e "constitution files" do Spec Kit cumprem função similar à linguagem ubíqua do DDD, formalizando conceitos de domínio que o agente deve respeitar.[^9][^19]

## Ferramentas associadas

| Categoria | Ferramentas |
|---|---|
| Coding agents / IDEs agentivos | Claude Code, Cursor, GitHub Copilot (modo agente), OpenAI Codex CLI, AWS Kiro, Windsurf, Devin [^19] |
| Spec-driven development | GitHub Spec Kit (Specify CLI), AIDE (extensão comunitária) [^14][^20] |
| AI code review | GitHub Copilot Code Review, CodeRabbit, Greptile, Claude Code Review, Cursor Bugbot, SonarQube, Qodo [^12][^39][^40] |
| Padrões de contexto/agentes | AGENTS.md (padrão aberto, governado pela Agentic AI Foundation), Model Context Protocol (MCP, Anthropic) [^19] |
| Benchmarks | SWE-bench / SWE-bench Verified (descontinuado pela OpenAI), SWE-bench Pro, SWE-rebench, SWE-MERA [^2][^41][^42] |

## Riscos conhecidos

Os riscos documentados concentram-se em quatro categorias recorrentes: falhas de controle de acesso (vulnerabilidades SSRF em 100% dos agentes testados em um estudo); credenciais expostas e não revogadas (64% permaneciam ativas); dependências alucinadas ("slopsquatting", 19,7% de amostras); e o "paradoxo de produtividade", em que ganhos individuais de output (20–40% mais código/commits) não se traduzem em velocidade de entrega organizacional, sendo absorvidos por maior tempo de revisão de código (até 91% mais longo). Adicionalmente, a validade dos benchmarks usados para justificar investimento em agentes está comprometida por contaminação e scaffolding não padronizado.[^3][^43][^30][^38]

## Tendências e questões em aberto

A tendência dominante é o deslocamento de "autocomplete" para "agentes com escopo de repositório e ciclos de revisão humana obrigatórios" (human-in-the-loop), refletido tanto no Spec Kit quanto nos padrões AGENTS.md. Ao mesmo tempo, cresce o ceticismo acadêmico e institucional (METR, OpenAI) quanto a métricas de produtividade e capacidade que antes eram aceitas sem crítica. Questões em aberto incluem: (1) se ganhos de produtividade individuais medidos por autorrelato refletem impacto real em entrega de software; (2) se existe uma definição estável e não-comercial para "specification-driven development" à medida que mais vendors adotam o rótulo; (3) se "context engineering" desenvolverá uma base acadêmica formal comparável à de prompt engineering ou permanecerá majoritariamente um termo de prática de mercado; e (4) como construir benchmarks de avaliação de agentes resistentes à contaminação e ao gaming de scaffolding, dado que mesmo o substituto SWE-bench Pro já apresenta taxa de erro de verificação de 32%.[^19][^3]

## Bibliografia

- GitHub Spec Kit — repositório oficial: https://github.com/github/spec-kit[^14]
- GitHub Spec Kit — documentação: https://github.github.com/spec-kit/[^15]
- GitHub Blog — "Spec-driven development with AI" (set/2025): https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/[^9]
- Microsoft Developer Blog — "Diving Into Spec-Driven Development With GitHub Spec Kit" (set/2025): https://developer.microsoft.com/blog/spec-driven-development-spec-kit/[^18]
- Fora Soft — "Spec-Driven Development: The 2026 AI Agent Playbook" (ago/2026): https://www.forasoft.com/blog/article/spec-driven-agentic-engineering[^19]
- arXiv — "From Code to Contract in the Age of AI Coding Assistants": https://arxiv.org/html/2602.00180v1[^13]
- Wikipedia — "Vibe coding" (mar/2025): https://en.wikipedia.org/wiki/Vibe_coding[^11]
- codebymelendez.com — "Vibe Coding in Production: The Real Risks" (ago/2026): https://codebymelendez.com/en/insights/vibe-coding-produccion[^30]
- keepmyprompts.com — "Context Engineering vs Prompt Engineering" (2026): https://www.keepmyprompts.com/en/blog/context-engineering-vs-prompt-engineering-2026[^10]
- deepset.ai — "Context Engineering: The Next Frontier Beyond Prompt Engineering" (jan/2026): https://www.deepset.ai/blog/context-engineering-the-next-frontier-beyond-prompt-engineering[^24]
- OpenAI — "Why SWE-bench Verified no longer measures frontier coding capabilities" (fev/2026): https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/[^2]
- theorydelta.com — "SWE-bench Contamination Benchmark Trust Collapse" (mai/2026): https://www.theorydelta.com/findings/swe-bench-contamination-benchmark-trust-collapse/[^1]
- codesota.com — "Is SWE-bench Verified Contaminated?" (fev/2026): https://www.codesota.com/news/swe-bench-contamination-debate[^38]
- niteagent.com — "The SWE-bench Reckoning of 2026" (jun/2026): https://niteagent.com/blog/swe-bench-reckoning-2026/[^3]
- arXiv — "The SWE-Bench Illusion" (2506.12286): https://arxiv.org/html/2506.12286v1[^37]
- METR — "Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity" (jul/2025), PDF: https://metr.org/Early_2025_AI_Experienced_OS_Devs_Study-paper.pdf[^31]
- METR — "We are Changing our Developer Productivity Experiment Design" (fev/2026): https://metr.org/blog/2026-02-24-uplift-update/[^33]
- Reuters — "AI slows down some experienced software developers, study finds" (jul/2025): https://www.reuters.com/business/ai-slows-down-some-experienced-software-developers-study-finds-2025-07-10/[^32]
- Cui, Demirer, Jaffe, Musolff, Peng, Salz — "The Productivity Effects of Generative AI" (MIT PubPub, mar/2024): https://mit-genai.pubpub.org/pub/v5iixksv/download/pdf[^4]
- arXiv — "Developer Productivity With and Without GitHub Copilot" (2509.20353): https://arxiv.org/html/2509.20353v2[^34]
- Ziegler et al. — "Measuring GitHub Copilot's Impact on Productivity", Communications of the ACM 67(3), 2024[^35]
- dev.to — "The Best AI Code Review Tools of 2026" (fev/2026): https://dev.to/heraldofsolace/the-best-ai-code-review-tools-of-2026-2mb3[^12]
- techsy.io — "12 Best AI Code Review Tools in 2026" (abr/2026): https://techsy.io/en/blog/best-ai-code-review-tools[^39]

---

## References

1. [Swe Bench Contamination Benchmark Trust Collapse](https://www.theorydelta.com/findings/swe-bench-contamination-benchmark-trust-collapse/) - Source evidence shows SWE-bench Verified was abandoned by OpenAI in Feb 2026 after audit found 59.4%...

2. [Why SWE-bench Verified no longer measures frontier ...](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/) - SWE-bench Verified is increasingly contaminated and mismeasures frontier coding progress. Our analys...

3. [The SWE-bench Reckoning of 2026: Contamination, Verifier Errors, and What Really Measures Coding Agent Ability](https://niteagent.com/blog/swe-bench-reckoning-2026/) - A research deep-dive into the 2026 SWE-bench crisis: OpenAI abandoned Verified after 59% flawed test...

4. [The Productivity Effects of Generative AI: Evidence from a Field Experiment with GitHub Copilot](https://mit-genai.pubpub.org/pub/v5iixksv/download/pdf)

5. [GitHub Copilot and Developer Productivity](https://arxiv.org/html/2606.00438v1)

6. [METR Scraps Its AI Productivity Study: What the Redesign Reveals ...](https://agentmarketcap.ai/blog/2026/04/05/metr-redesigns-developer-productivity-experiment) - METR scrapped its developer productivity experiment after selection bias made results unreliable. He...

7. [On METR's AI Coding RCT](https://www.lesswrong.com/posts/m2QeMwD7mGKH6vDe2/on-metr-s-ai-coding-rct)

8. [AI tools slow down experienced developers, study shows](https://www.linkedin.com/posts/metr-evals_we-ran-a-randomized-controlled-trial-to-see-activity-7349133398998732801-MDF6) - We ran a randomized controlled trial to see how much AI coding tools speed up experienced open-sourc...

9. [Spec-driven development with AI: Get started with a new ...](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/) - Developers can use their AI tool of choice for spec-driven development with this open source toolkit...

10. [Context Engineering vs Prompt Engineering (2026)](https://www.keepmyprompts.com/en/blog/context-engineering-vs-prompt-engineering-2026) - Context engineering is the evolution of prompt engineering, not its replacement. Data from 9,649 exp...

11. [Vibe coding - Wikipedia](https://en.wikipedia.org/wiki/Vibe_coding)

12. [The Best AI Code Review Tools of 2026](https://dev.to/heraldofsolace/the-best-ai-code-review-tools-of-2026-2mb3) - Early AI code review tools had a math problem. For every real bug they caught, they flagged nine...

13. [From Code to Contract in the Age of AI Coding Assistants](https://arxiv.org/html/2602.00180v1)

14. [github/spec-kit: 💫 Toolkit to help you get started with ...](https://github.com/github/spec-kit) - 💫 Toolkit to help you get started with Spec-Driven Development - github/spec-kit

15. [Spec Kit Documentation](https://github.github.com/spec-kit/)

16. [What is Spec-Driven Development?](https://github.github.com/spec-kit/concepts/sdd.html)

17. [GitHub Spec Kit Deep Dive — Spec-Driven Development, the Constitution, /speckit.* Slash Commands, and the Specify CLI for Taming AI Coding Agents](https://dev.to/x4nent/github-spec-kit-deep-dive-spec-driven-development-the-constitution-speckit-slash-commands-1m71) - As AI coding agents become routine, the most common failure mode is *vibe coding*: you start with a ...

18. [Diving Into Spec-Driven Development With GitHub Spec Kit](https://developer.microsoft.com/blog/spec-driven-development-spec-kit/) - Developers are increasingly relying on AI agents to both build new software as well as extend capabi...

19. [Spec-Driven Development: The 2026 AI Agent Playbook - Fora Soft](https://www.forasoft.com/blog/article/spec-driven-agentic-engineering) - Spec-driven development turns AI coding agents from chaos into shipped software: the 4 artefacts, re...

20. [Community Walkthroughs | Spec Kit Documentation - GitHub Pages](https://github.github.com/spec-kit/community/walkthroughs.html)

21. [Context engineering vs. prompt engineering | Elasticsearch ...](https://www.elastic.co/search-labs/blog/context-engineering-vs-prompt-engineering) - Prompt engineering focuses on how you communicate with the model. Context engineering focuses on wha...

22. [Context Engineering vs Prompt Engineering | DataHub](https://datahub.com/blog/context-engineering-vs-prompt-engineering/) - Prompt engineering encodes knowledge at write time. Context engineering retrieves it at runtime from...

23. [Context Engineering vs Prompt Engineering | by Mehul Gupta](https://medium.com/data-science-in-your-pocket/context-engineering-vs-prompt-engineering-379e9622e19d) - How Context Engineering different from Prompt Engineering

24. [Context Engineering: The Next Frontier Beyond Prompt ...](https://www.deepset.ai/blog/context-engineering-the-next-frontier-beyond-prompt-engineering) - This article explores how designing the right informational environment is the key to building sophi...

25. [Context engineering](https://timesofindia.indiatimes.com/toi-blogs/digital-life/context-engineering/articleshow/133344806.cms) - We have spoken about prompt engineering many times. What is context engineering now?

26. [Context engineering vs prompt engineering: the difference](https://redis.io/blog/context-engineering-vs-prompt-engineering/) - The line is clean: prompt engineering is about how you instruct the model; context engineering is ab...

27. [Context Engineering vs. Prompt Engineering Explained](https://intuitionlabs.ai/articles/context-engineering-vs-prompt-engineering-ai) - Analyze the shift from prompt engineering to context engineering in AI. Learn how curating knowledge...

28. [Context engineering vs. prompt engineering: Key ...](https://www.glean.com/perspectives/context-engineering-vs-prompt-engineering-key-differences-explained) - The Glean Team | Context engineering vs prompt engineering: Prompt engineering crafts AI inputs whil...

29. [GitHub Spec-Kit: From Vibe Coding to Spec-Driven Development](https://dev.to/petersaktor/github-spec-kit-from-vibe-coding-to-spec-driven-development-1pgd) - AI coding agents are improving rapidly. Tools like Copilot, Codex, and Claude can generate large...

30. [Vibe Coding in Production: The Real Risks](https://codebymelendez.com/en/insights/vibe-coding-produccion) - What happens when AI-generated code reaches production without anyone who actually understands it re...

31. [[PDF] Measuring the Impact of Early-2025 AI on Experienced ... - METR](https://metr.org/Early_2025_AI_Experienced_OS_Devs_Study-paper.pdf)

32. [AI slows down some experienced software developers, study finds](https://www.reuters.com/business/ai-slows-down-some-experienced-software-developers-study-finds-2025-07-10/) - Contrary to popular belief, using cutting-edge artificial intelligence tools slowed down experienced...

33. [We are Changing our Developer Productivity Experiment ...](https://metr.org/blog/2026-02-24-uplift-update/) - Our second developer productivity study faces selection effects from wider AI adoption, prompting us...

34. [Developer Productivity With and Without GitHub Copilot](https://arxiv.org/html/2509.20353v2)

35. [The impact of GitHub Copilot on developer productivity from a software engineering body of knowledge perspective](https://aisel.aisnet.org/cgi/viewcontent.cgi?article=1096&context=amcis2024)

36. [SWE-bench: The Benchmark That Broke Software Engineering AI ...](https://www.snackonai.com/p/swe-bench-the-benchmark-that-broke-software-engineering-ai-then-got-broken-itself) - SWE-bench is the most important benchmark in AI software engineering, and it has a contamination pro...

37. [The SWE-Bench Illusion: When State-of-the-Art LLMs ...](https://arxiv.org/html/2506.12286v1)

38. [Is SWE-bench Verified Contaminated? OpenAI Shifts to ...](https://www.codesota.com/news/swe-bench-contamination-debate) - OpenAI stops reporting Verified scores. Agent scaffolding inflates results by 12+ points. The benchm...

39. [12 Best AI Code Review Tools in 2026 (Ranked) - TECHSY](https://techsy.io/en/blog/best-ai-code-review-tools) - Vendor-neutral ranking of 12 AI code review tools. Every catch rate names its benchmark, repos, samp...

40. [AI Code Review and the Best AI Code Review Tools in 2026](https://www.qodo.ai/blog/ai-code-review/) - AI handles the tedious checks, while humans apply judgment to the messy, context-dependent decisions...

41. [[PDF] SWE-rebench: An Automated Pipeline for Task Collection and ...](https://openreview.net/pdf?id=nMpJoVmRy1)

42. [SWE-MERA: A Dynamic Benchmark for Agenticly ...](https://aclanthology.org/2025.emnlp-demos.30.pdf)

43. [Faq Section](https://www.softwareseni.com/what-the-research-actually-shows-about-ai-coding-assistant-productivity/) - Research reveals AI coding tools increase individual output 20-40% but slow task completion 19%. Lea...

