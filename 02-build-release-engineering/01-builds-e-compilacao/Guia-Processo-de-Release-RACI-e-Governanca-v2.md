<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

## Resumo executivo

Um processo de release saudável combina três pilares: um dono claro por release, critérios objetivos de risco/readiness e automação forte para evitar filas de aprovação desnecessárias.[^1_1][^1_2][^1_3]
Boas práticas recomendam definir RACI (Responsible, Accountable, Consulted, Informed) com um único “A” por atividade, usar segregação de funções apenas onde o risco justifica e registrar todas as decisões em artefatos versionados (issue de release, change record, página de release) para auditoria.[^1_4][^1_5][^1_6][^1_7][^1_8]

Em times pequenos e pouco regulados, a maior parte da segurança vem de testes automatizados, políticas de branch e pipelines com gates, com um “release owner” combinando papéis de dev/tech lead/prod, e poucas aprovações humanas.[^1_9][^1_10][^1_3][^1_1]
Em empresas médias ou com produto distribuído a clientes, entra um release manager/engineer coordenando calendário, readiness checklist, go/no-go e comunicação, mas mantendo o fluxo leve e baseado em evidências (testes, métricas, rollback pronto).[^1_2][^1_11][^1_12][^1_13]

Em sistemas críticos ou ambientes regulados, o processo adiciona tiers de mudança, CAB, janelas de manutenção/freeze, SoD formal (dev não deploya sozinho em produção), trilhas de auditoria e processos de exceção/emergência bem definidos, ainda com foco em automação para não travar o negócio.[^1_14][^1_15][^1_16][^1_17]
Em todos os cenários, “mais aprovações” só aumentam segurança se forem baseadas em critérios objetivos (defeitos, cobertura, vulnerabilidades, SLOs, plano de rollback) e suportadas por ferramentas, em vez de e‑mails e comitês genéricos.[^1_18][^1_11][^1_12][^1_3][^1_9]

***

## Modelo leve

Focado em: equipe pequena, pouco regulada; produto interno não crítico; SaaS early-stage com baixo impacto regulatório.[^1_10][^1_3][^1_1]

### Características principais

- **Release owner único por release (geralmente tech lead ou dev sênior)**: responsável por empacotar a release, coordenar revisões e decidir o go/no-go, evitando “decisão sem dono”.[^1_11][^1_1][^1_2]
- **Pipelines com gates automáticos**: testes automatizados obrigatórios, análise estática, scans básicos de segurança e smoke em ambiente de staging, bloqueando deploy se falharem.[^1_12][^1_3][^1_9][^1_18]
- **RACI mínimo**:
    - Desenvolvedor / autor do PR: implementa, escreve testes, documenta mudanças.[^1_19][^1_1]
    - Revisor (outro dev): revisão de código, qualidade técnica, aderência a padrões.[^1_1][^1_19]
    - QA (quando existir): automatiza/registra casos críticos, valida regressão básica.[^1_9][^1_11]
    - Tech lead: dono técnico do serviço, muitas vezes também release owner.[^1_2][^1_1]
    - Product manager / responsável de negócio: prioriza e aceita escopo, mas não precisa aprovar cada deploy se risco for baixo.[^1_10][^1_1]


### Quem faz o quê (modelo leve)

- **Quem pode criar uma release**: desenvolvedor ou tech lead, via tag, branch de release ou promoção de build “verde” do pipeline.[^1_13][^1_3][^1_1]
- **Quem pode aprovar (go)**: release owner (geralmente tech lead) com base em checklist rápido e sinal de QA (quando houver), sem CAB.[^1_11][^1_9][^1_2]
- **Quem pode interromper (no-go / abortar rollout)**: tech lead, SRE/ops (se existir), ou on-call do serviço, com autonomia para pausar pipeline ou desligar feature flag.[^1_20][^1_3][^1_13]
- **Quem pode executar rollback**: SRE/ops ou dev on-call, via playbook automatizado (rollback de release, canary/blue‑green, roll back de feature flag).[^1_21][^1_3][^1_13]
- **Quem comunica clientes**: normalmente product manager ou suporte, usando notas de release simples e canal padrão (e‑mail, status page, WhatsApp para clientes B2B, etc.).[^1_2][^1_11][^1_10]
- **Quem acompanha métricas pós-release**: dev/tech lead e SRE (se houver), via dashboards padronizados (erros, latência, taxa de rollback, incidentes pós-release).[^1_3][^1_13][^1_20]
- **Quem encerra o incidente**: responsável on‑call (dev ou SRE) + suporte, garantindo RCA leve e atualização de backlog.[^1_20][^1_3]
- **Quem atualiza documentação**: autor do PR (doc técnica e change log) e product/UX (doc de usuário), com checklist no próprio PR/release issue.[^1_18][^1_3][^1_11]


### Contexto

- **Equipe pequena e pouco regulada** → este modelo é o default, com foco em automação e um release owner bem definido.[^1_1][^1_10]
- **Produto interno** → comunicação mais simples, menos formalidade, mas mesma disciplina de rollback e métricas para evitar impactos em equipes internas.[^1_3][^1_9]

***

## Modelo intermediário

Focado em: empresa de porte médio; software distribuído a clientes; sistemas com impacto financeiro moderado, mas sem regulação pesada (por exemplo, SaaS B2B típico, e‑commerce médio).[^1_10][^1_1][^1_2]

### Características principais

- **Release manager/engineer**: coordena calendário de releases, readiness, comunicação, execução do corte (cutover) e pós-release, mantendo previsibilidade.[^1_13][^1_2][^1_10]
- **Release calendar + release train leve**: cadência definida (por exemplo, 1× por semana ou quinzenal), com branch freeze curto e janela de deploy conhecida.[^1_16][^1_14][^1_10]
- **Readiness checklist e go/no-go formal, mas enxutos**: 8–15 itens com evidências, gates de QA, segurança, rollback e suporte, com aprovação de poucos papéis chave (Release Owner, QA Lead, Product, Ops).[^1_12][^1_9][^1_18][^1_11]


### Papéis chave

- **Desenvolvedor / autor do PR**: idem modelo leve.[^1_19][^1_1]
- **Revisor**: garante padrões de código, design técnico e riscos localizados.[^1_19][^1_1]
- **QA**: dono da estratégia de testes por release, consolida evidências, assina readiness em nome de QA.[^1_9][^1_11][^1_1]
- **Tech lead / arquiteto**: avalia impactos em arquitetura, integrações, dados; participa de go/no-go em mudanças de alto risco.[^1_5][^1_1][^1_2]
- **Segurança**: supervisiona resultados de scans SAST/DAST, vulnerabilidades críticas e controles de dados.[^1_6][^1_7][^1_12]
- **Operações/SRE**: confirma readiness de infraestrutura, monitoramento, runbooks e plano de rollback.[^1_13][^1_20][^1_3]
- **Product manager / responsável de negócio**: aprova readiness de negócio (treinamento, comunicação, feature flags, impacto em clientes).[^1_11][^1_1][^1_2]
- **Release manager**: coordena tudo isso, dono do release calendar, da release issue e do go/no-go meeting.[^1_2][^1_13][^1_10]


### Quem faz o quê (modelo intermediário)

- **Criar release**: release manager ou release engineer gera registro de release (issue, ticket, change), agrupa tickets/features, referencia build/tag e agenda janela.[^1_18][^1_13][^1_2]
- **Aprovar (go)**:
    - Normal: Release Owner (pode ser Release Manager ou Tech Lead) + QA Lead + Product + Ops/SRE, com assinaturas registradas na issue de release.[^1_22][^1_18][^1_11][^1_2]
    - Alto risco: pode incluir Security e, opcionalmente, um mini‑CAB técnico/comercial.[^1_17][^1_22][^1_12]
- **Interromper rollout**: SRE/ops, Release Manager e incident commander on‑call têm autoridade explícita para pausar ou reverter, seguindo runbook.[^1_21][^1_20][^1_13]
- **Rollback**: SRE/ops executa, dev/QA valida funcionalmente e QE/QA documenta motivo; release manager registra decisão no change record.[^1_21][^1_20][^1_13]
- **Comunicação a clientes**: Product + Suporte; release manager coordena release notes, status page e avisos de indisponibilidade planejada.[^1_11][^1_10][^1_2]
- **Métricas pós-release**: SRE/ops monitora SLOs, incidentes e rollback; release manager consolida e reporta KPI de release (por exemplo, DORA + taxa de incidentes pós-release).[^1_20][^1_3][^1_13][^1_10]
- **Encerrar incidente**: incident commander (geralmente SRE ou gestor de operações) + owner do serviço, com post‑mortem enxuto ligado à issue de release.[^1_3][^1_20]
- **Atualizar documentação**:
    - Técnica/runbooks: dev/tech lead + SRE,
    - Usuário/negócio: product/suporte,
    - Registro formal de release: release manager na wiki/confluence ou ferramenta equivalente.[^1_18][^1_13][^1_3][^1_11]


### Contexto

- **Empresa de porte médio / software distribuído a clientes** → este modelo equilibra governança com fluxo, evitando CAB para tudo, mas exigindo evidência objetiva e registro da decisão.[^1_1][^1_10][^1_3][^1_2]

***

## Modelo regulado

Focado em: sistema crítico (financeiro, saúde, governo); ambiente regulado (SOX, PCI‑DSS, HIPAA, GDPR etc.); forte exigência de auditoria e segregação de funções.[^1_7][^1_6][^1_14][^1_17]

### Características principais

- **Segregation of duties (SoD) formal**: nenhum indivíduo pode iniciar, aprovar e executar sozinho uma mudança crítica de ponta a ponta; funções de iniciador, aprovador, executor e verificador são claramente separadas e reforçadas por RBAC/políticas na ferramenta de CI/CD.[^1_23][^1_6][^1_7]
- **Change tiers, CAB e janelas de mudança**: mudanças categorizadas (standard, normal, expedited, emergency) com trilhas de aprovação e janelas de manutenção/freeze; CAB ou autoridade de mudança avaliam riscos e conflitos para mudanças de maior impacto.[^1_24][^1_14][^1_16][^1_17]
- **Calendário e freezes publicados com antecedência**: freeze total/parcial/segmentado em períodos de alto risco (fechamento contábil, campanhas, picos de demanda, início de período letivo), com processo explícito de exceção.[^1_15][^1_25][^1_26][^1_16][^1_17]
- **Auditabilidade total**: cada release/change possui registro imutável de quem aprovou, quando, com qual evidência de testes, segurança, risco e rollback.[^1_7][^1_12][^1_13][^1_20][^1_2][^1_11]


### Papéis

Além dos papéis do modelo intermediário, entram:

- **Change/Release authority (CAB ou Change Manager)**: autoridade formal que aprova mudanças de alto risco/impacto, registra decisão e condições (por exemplo, “GO condicionado a X”).[^1_22][^1_14][^1_17]
- **Compliance/auditoria**: participa como “Consulted” ou “Informed” para validar alinhamento com normas e revisa amostras de changes, incluindo emergenciais.[^1_6][^1_7][^1_20]


### Quem faz o quê (modelo regulado, high‑level)

- **Criar release/change record**: geralmente Release Manager ou Change Initiator (pode ser dev lead ou product), mas sempre num sistema de ITSM (ServiceNow, Jira SM etc.).[^1_14][^1_16][^1_23][^1_24]
- **Aprovar**:
    - Standard (baixo risco, pré-aprovada): aprovação automática + validação técnica local (tech lead).[^1_16][^1_17]
    - Normal/major: Change Manager/CAB + business owner + áreas técnicas chave (QA, Security, Ops), cada uma com responsabilidade definida.[^1_17][^1_22][^1_14]
- **Interromper / rollback**: incident commander/SRE/ops têm autoridade imediata; posteriormente, CAB revisa decisão e impacto.[^1_16][^1_21][^1_20]
- **Comunicar clientes/reguladores**: product + business owner +, quando necessário, compliance/assessoria, seguindo playbooks específicos (notificações de incidente, manutenção programada).[^1_17][^1_16][^1_20][^1_2]
- **Encerrar incidente e atualizar documentação/compliance**: incident commander + service owner + compliance, com post‑mortem formal e evidências anexadas ao change record.[^1_17][^1_20][^1_3]


### Contexto

- **Sistema crítico / ambiente regulado / software instalado em clientes corporativos** → exige este modelo ou variações, com foco em trilha de auditoria, SoD e gestão de riscos.[^1_15][^1_6][^1_7][^1_14][^1_17]

***

## Matriz RACI (exemplo)

Exemplo simplificado para um serviço em modelo intermediário/regulado; adapte “A” e “R” conforme contexto. A recomendação de “um único A por atividade” vem de boas práticas RACI.[^1_8][^1_4][^1_5][^1_19]


| Atividade / Papel | Dev | Autor PR | Revisor | QA | Tech Lead | Arquiteto | Segurança | Operações | SRE | Product Manager | Release Manager | Suporte | Responsável de negócio |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| Definir escopo da release | C[^1_1] | R[^1_1] | C[^1_1] | C[^1_1] | C[^1_1] | C[^1_1] | I[^1_6] | I[^1_13] | I[^1_13] | R[^1_1] | A[^1_2] | I[^1_2] | A (negócio) em modelo regulado[^1_1] |
| Criar registro de release (issue/change) | I[^1_18] | R (se owner técnico)[^1_18] | I[^1_18] | I[^1_18] | C[^1_18] | C[^1_18] | I[^1_16] | C[^1_16] | C[^1_13] | C[^1_18] | A[^1_2] | I[^1_16] | C[^1_16] |
| Implementar mudanças | R[^1_1] | A (pelo próprio código)[^1_1] | C[^1_1] | I[^1_9] | C[^1_1] | C[^1_5] | I[^1_6] | I[^1_13] | I[^1_13] | I[^1_1] | I[^1_2] | I[^1_10] | I[^1_1] |
| Revisar código | I[^1_1] | R[^1_1] | A[^1_1] | I[^1_9] | C[^1_1] | C[^1_5] | I[^1_6] | I[^1_13] | I[^1_13] | I[^1_1] | I[^1_2] | I[^1_10] | I[^1_1] |
| Testes e evidências | C[^1_9] | C[^1_9] | C[^1_9] | R/A[^1_9][^1_11] | C[^1_9] | C[^1_5] | C (segurança app)[^1_6][^1_12] | I[^1_13] | C (performance)[^1_13][^1_3] | I[^1_1] | I[^1_2] | I[^1_10] | I[^1_1] |
| Aprovação técnica da release | C[^1_1] | C[^1_1] | C[^1_1] | C[^1_9] | A[^1_1][^1_2] | C[^1_5] | C[^1_6][^1_12] | C[^1_13] | C[^1_13] | I[^1_1] | R (facilita go/no-go)[^1_2][^1_22] | I[^1_10] | I[^1_1] |
| Aprovação de produto/negócio | I[^1_1] | I[^1_1] | I[^1_1] | I[^1_9] | C[^1_1] | C[^1_5] | I[^1_6] | I[^1_13] | I[^1_13] | A[^1_1][^1_11] | C[^1_2] | C[^1_10] | A em ambientes regulados[^1_22][^1_17] |
| Execução do deploy | I[^1_13] | I[^1_13] | I[^1_13] | I[^1_21] | C[^1_13] | C[^1_5] | I[^1_6] | A (se equipe de Ops) | [^1_13] | R/A (em times SRE‑heavy)[^1_13] | I[^1_2] | R (coordenação)[^1_2][^1_13] | I[^1_10] |
| Decisão go/no-go final | C[^1_9] | C[^1_9] | C[^1_9] | C[^1_22][^1_11] | C[^1_2] | C[^1_5] | C[^1_12] | C[^1_22] | C[^1_13] | C[^1_22][^1_11] | A (Release Owner)[^1_2][^1_11] | C[^1_10] | A/C em mudanças críticas de negócio[^1_22][^1_17] |
| Monitoramento pós-release | C[^1_20] | C[^1_20] | I[^1_20] | C[^1_20] | C[^1_13] | I[^1_5] | C (segurança em produção)[^1_6][^1_12] | R/A[^1_13][^1_20] | R/A[^1_13][^1_20] | I[^1_10] | C[^1_2] | R (suporte de primeira linha)[^1_5][^1_10] | I[^1_20] |
| Rollback | C[^1_21] | I[^1_21] | I[^1_21] | C (validação)[^1_21] | C[^1_13] | C[^1_5] | I[^1_6] | R/A[^1_13][^1_21] | R/A[^1_13][^1_21] | I[^1_10] | C[^1_2] | C (comunica clientes)[^1_11][^1_10] | I[^1_16] |
| Comunicação a clientes | I[^1_11] | I[^1_11] | I[^1_11] | I[^1_11] | I[^1_2] | I[^1_5] | I[^1_6] | C[^1_16] | I[^1_13] | R/A[^1_11][^1_10] | C[^1_2] | R (suporte)[^1_5][^1_10] | C/A em incidentes graves[^1_17][^1_20] |

Use esta matriz como base e simplifique para o modelo leve e endureça para o regulado (por exemplo, separando claramente “executor” e “aprovador”).[^1_4][^1_5][^1_8][^1_6][^1_7]

***

## Agenda de go/no-go

Boas práticas recomendam um encontro curto, time‑boxed, focado em evidências, usando a issue de release ou checklist como agenda única.[^1_22][^1_12][^1_9][^1_18][^1_11]

Exemplo de agenda (30–45 minutos):

1. **Contexto da release (5 min)**
    - Release ID, janela, tipo (normal/hotfix/major) e serviços impactados.[^1_18][^1_11]
    - Risco agregado (ex.: score automático ou classificação baixa/média/alta).[^1_12]
2. **Status dos gates de readiness (10–15 min)**
    - QA: resumo de testes executados, defeitos críticos/altos em aberto.[^1_9][^1_12][^1_11][^1_18]
    - Segurança: resultados de scans, vulnerabilidades críticas.[^1_6][^1_12]
    - Ops/SRE: readiness de ambiente, monitoramento, rollback testado.[^1_13][^1_9][^1_11][^1_18]
    - Suporte/Negócio: comunicação preparada, treinamento e hypercare.[^1_10][^1_11]
3. **Riscos remanescentes e mitigação (5–10 min)**
    - Top 3 riscos com respectivos owners e planos.[^1_12]
    - Condições de GO condicional (por exemplo, “GO se feature flag permanecer off para 90% dos clientes”).[^1_3][^1_11][^1_12]
4. **Decisão formal (5 min)**
    - Votação/assinatura dos papéis requeridos (Release Owner, QA, Product, Ops/SRE, Security quando aplicável).[^1_22][^1_11][^1_12]
    - Resultado registrado como GO / NO-GO / GO condicional, com timestamp e justificativa.[^1_22][^1_11][^1_18]
5. **Próximos passos (5 min)**
    - Confirmar quem estará on‑call, canal de war room e expectativas de monitoramento pós‑deploy.[^1_20][^1_13][^1_3]

***

## Checklist de readiness

Derivado de checklists usados em práticas modernas de release readiness e change management.[^1_9][^1_11][^1_12][^1_3][^1_18]

Checklist objetivo (adapte por risco):

- **Artefato**
    - Build/tag da release é único, assinado/imutável e rastreável até commits e tickets.[^1_11][^1_12][^1_13][^1_3]
- **Qualidade**
    - Pipeline CI passou sem falhas em testes unitários, integração e smoke.[^1_12][^1_9][^1_18]
    - Não há defeitos bloqueadores em aberto; defeitos altos têm plano claro de mitigação.[^1_9][^1_11][^1_12]
- **Testes de regressão e NFR**
    - Regressão automatizada com taxa de sucesso mínima (por exemplo, ≥95%); falhas conhecidas documentadas.[^1_12][^1_9]
    - Performance e disponibilidade testadas em ambiente representativo (p95/p99 dentro de limites).[^1_13][^1_3][^1_12]
- **Segurança**
    - Scans SAST/DAST/SCA executados; zero vulnerabilidades críticas abertas ou aceitação formal devidamente registrada.[^1_7][^1_6][^1_12]
- **Dados e migrações**
    - Migrações de banco revisadas, com dry‑run bem-sucedido e plano de reversão/downgrade.[^1_21][^1_11][^1_9]
- **Infraestrutura e observabilidade**
    - Dashboards, logs e alertas configurados e vinculados ao release.[^1_3][^1_18][^1_13]
    - Health checks e smoke pós-deploy definidos.[^1_21][^1_20][^1_3]
- **Rollback**
    - Procedimento documentado, testado, com RTO estimado (por exemplo, rollback em <15 minutos).[^1_11][^1_21][^1_3][^1_9][^1_12]
- **Suporte e operações**
    - Runbooks atualizados; rota de hypercare definida.[^1_20][^1_13][^1_11]
    - On‑call ciente da janela de deploy.[^1_13][^1_20]
- **Negócio e comunicação**
    - Release notes atualizadas, plano de comunicação para clientes/usuários internos pronto.[^1_2][^1_10][^1_11]
    - Treinamentos ou FAQs necessários preparados.[^1_10][^1_11]

***

## Critérios objetivos

Frameworks recentes defendem decisões de go/no-go baseadas em indicadores mensuráveis de risco e readiness em vez de “sensação de segurança”.[^1_3][^1_9][^1_11][^1_12]

Exemplo simplificado de critérios (adaptável por criticidade):


| Métrica | GO | Revisão manual | NO-GO |
| :-- | :-- | :-- | :-- |
| Defeitos bloqueadores em aberto | 0[^1_9][^1_12] | – | >0[^1_9][^1_12] |
| Vulnerabilidades críticas (CVSS ≥ 9) | 0[^1_6][^1_12] | – | >0 sem mitigação aprovada[^1_6][^1_12] |
| Pass rate de regressão | ≥95%[^1_9][^1_12] | 90–94% com justificativa | <90%[^1_9][^1_12] |
| Cobertura de testes em novo código | ≥80%[^1_12] | 70–79% com plano de aumento | <70%[^1_12] |
| Tempo estimado de rollback | ≤15 min[^1_9][^1_11][^1_3] | 16–30 min com mitigação extra | >30 min sem plano alternativo[^1_11][^1_3] |
| Mudanças em DB/esquema | Testadas e reversíveis[^1_9][^1_11][^1_21] | Reversão parcial, aceita com CAB | Irreversíveis sem plano/validação[^1_21] |
| Impacto em clientes/regulação | Baixo, sem obrigações novas[^1_10][^1_17] | Médio, com plano de comunicação | Alto, sem mitigação ou aprovação regulatória[^1_17] |

Este tipo de tabela ajuda a evitar que cada release vire discussão subjetiva, especialmente em sistemas críticos e ambientes regulados.[^1_11][^1_12][^1_3]

***

## Processo de exceção

Exceções são necessárias para lidar com oportunidades/obrigatoriedades fora do fluxo normal (por exemplo, mudança em janela de freeze, escopo extra em cima da hora), mas devem ter um fluxo padronizado.[^1_25][^1_26][^1_15][^1_16][^1_17]

Elementos recomendados:

1. **Critérios claros para o que é exceção**
    - Mudanças agendadas em período de freeze.[^1_25][^1_15][^1_17]
    - Mudanças de risco maior que o permitido para a janela (por exemplo, alteração estrutural de DB em véspera de pico).[^1_15][^1_16]
2. **Workflow mínimo**
    - Solicitação com justificativa de negócio, risco, impacto e alternativas.[^1_15][^1_16][^1_17]
    - Aprovação por change authority + business owner (e, se crítico, segurança/compliance).[^1_25][^1_15][^1_17]
    - Controles extras: smoke ampliado, monitoramento dedicado, on‑call reforçado e plano de rollback explícito.[^1_16][^1_15][^1_21][^1_20]
3. **Registro e auditoria**
    - Change record marcado como “freeze exception” ou equivalente, com logs de aprovação anexados.[^1_25][^1_15][^1_16]
    - Revisão posterior (por exemplo, em CAB) para avaliar se a exceção valeu o risco.[^1_14][^1_17][^1_20]

***

## Processo emergencial

Emergência é diferente de exceção: é resposta a incidente em curso ou vulnerabilidade crítica, em que não agir é pior do que o risco da mudança.[^1_14][^1_15][^1_16][^1_17][^1_21]

Elementos típicos (alinhados a ITIL e práticas DevOps modernas):

1. **Gatilhos de emergência**
    - Incidente de severidade alta (indisponibilidade, perda de dados, violação de segurança).[^1_16][^1_17][^1_20]
    - Vulnerabilidade crítica explorada ou facilmente explorável com exigência de patch imediato.[^1_6][^1_17][^1_12]
2. **Fluxo de decisão rápido**
    - Incident commander (SRE/ops) declara necessidade de emergency change.[^1_16][^1_20]
    - Aprovação por ECAB (emergencial), geralmente um subconjunto enxuto de CAB + owner de negócio/serviço.[^1_26][^1_14][^1_15][^1_17]
    - Change executado fora das janelas normais, durante congelamento se necessário.[^1_14][^1_15][^1_25]
3. **Execução e rollback**
    - Runbooks específicos para hotfixes, com smoke imediato e plano de rollback.[^1_21][^1_20][^1_16]
    - Restrições de acesso e SoD ainda valem (por exemplo, quem aprova não é quem executa o script em produção, quando possível).[^1_7][^1_6][^1_17]
4. **Pós‑fato obrigatório**
    - Documentar change emergencial, evidências, motivo, decisão de rollback ou não, impactos e ações futuras.[^1_14][^1_20][^1_16]
    - Revisão em CAB e/ou com compliance para manter aderência regulatória.[^1_17][^1_20]

***

## Antipadrões

Com base nas práticas e guias estudados, alguns antipadrões recorrentes em governança de release:[^1_1][^1_6][^1_15][^1_2][^1_10][^1_12][^1_13][^1_3]

- **Aprovação por comitê gigante**: CAB com muitas pessoas, sem critérios objetivos, que vira gargalo, sem aumentar a qualidade da decisão.[^1_22][^1_17][^1_12]
- **Ninguém é “A” (Accountable)**: várias pessoas “responsáveis”, mas nenhuma com autoridade para decidir go/no-go, gerando atrasos e conflitos.[^1_5][^1_8][^1_4]
- **Segregação só no papel**: SoD definido em planilhas, mas pipeline/infra permitem que o mesmo usuário bypassa revisões e deploya direto em produção.[^1_23][^1_6][^1_7]
- **Aprovar sem evidência**: sign‑off de QA, segurança ou negócio sem link para testes, scans, planos de rollback ou análise de risco.[^1_18][^1_9][^1_12][^1_11]
- **Freeze “secreto” ou caótico**: janelas de congelamento mal comunicadas ou alteradas em cima da hora, gerando surpresa e exceções ad‑hoc.[^1_26][^1_15][^1_25][^1_16]
- **Rollback não testado**: playbooks de rollback existentes apenas na wiki, sem ensaios; na emergência, ninguém sabe se funciona ou quanto tempo leva.[^1_9][^1_21][^1_3][^1_11]
- **Confundir deployment com release**: expor feature para todos logo após o deploy, sem feature flags ou controle de exposição.[^1_10][^1_3]
- **Processo único para tudo**: aplicar o mesmo nível de CAB e documentação para uma correção de texto e para uma migração crítica de dados, gerando filas e incentivos para burlar o processo.[^1_17][^1_12][^1_16]

***

## Tabela de fontes e limitações

| Categoria de fonte | Exemplos | Limitações ao aplicar ao seu contexto |
| :-- | :-- | :-- |
| Guias de release management e RACI | Artigos sobre RACI/RASCI e responsabilidade em release management.[^1_4][^1_1][^1_5][^1_19][^1_8] | Abordam cenários genéricos; é preciso adaptar à estrutura real de papéis (por exemplo, quando não há QA dedicado ou CAB formal). |
| Práticas de readiness e go/no-go | Checklists e frameworks de release readiness, risk scoring e go/no-go.[^1_9][^1_22][^1_18][^1_11][^1_12] | Em geral escritos para SaaS ou ambientes enterprise anglo‑saxões; métricas e thresholds precisam ser calibrados para volume, risco e SLOs locais. |
| SoD, segurança e compliance | Materiais sobre segregação de funções em DevSecOps e órgãos públicos.[^1_6][^1_7][^1_14][^1_17] | Fortemente orientados a ambientes regulados; em times pequenos, é necessário balancear SoD com capacidade e automação para não paralisar o fluxo. |
| Calendário de mudanças e freeze | Documentos de change windows, freeze e janelas de manutenção.[^1_15][^1_25][^1_16][^1_23][^1_24][^1_26] | Alguns assumem ferramentas específicas (ServiceNow, Jira), culturas e calendários (acadêmicos/americanos) diferentes; datas e cadências devem ser redefinidas localmente. |
| Papéis de Release Manager/Engineer e SRE/Ops | Blueprints de responsabilidades e KPIs desses papéis.[^1_2][^1_13][^1_10][^1_20][^1_3] | Definições ideais podem ser mais amplas do que o que uma organização média consegue absorver; frequentemente é preciso combinar papéis em uma mesma pessoa. |
| Rollback, incidentes e pós-release | Checklists de rollback, incident handling e post-release analysis.[^1_21][^1_20][^1_3] | Nem sempre consideram restrições de softwares instalados em clientes (on‑prem), onde rollback depende do próprio cliente e conectividade. |


***

Se quiser, posso ajudar a transformar este modelo em documentos operacionais para o seu contexto (por exemplo, um template de issue de release para Jira/YouTrack, ou uma matriz RACI adaptada para um time específico – ZWCAD, SketchUp, Enscape etc.), incluindo exemplos de automação em CI/CD.
<span style="display:none">[^1_27][^1_28][^1_29][^1_30]</span>

<div align="center">⁂</div>

[^1_1]: https://www.exceediance.com/the-complete-release-management-guide/

[^1_2]: https://www.devopsschool.com/blog/release-manager-role-blueprint-responsibilities-skills-kpis-and-career-path/

[^1_3]: https://capgo.app/blog/release-management-process/

[^1_4]: https://www.itgov-docs.com/blogs/release-management/raci-matrix-for-release-management-template

[^1_5]: https://www.itsm-docs.com/blogs/release-management/release-and-deployment-management-raci-matrix-template

[^1_6]: https://www.devsecopsnow.com/segregation-of-duties/

[^1_7]: https://www.gsa.gov/system/files/CIO IL-22-01 DevSecOps Model_Separation%20of%20Duties%20(03-10-2022)%20(1).pdf

[^1_8]: https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/organize/raci-alignment

[^1_9]: https://www.tqsystems.io/blog/release-readiness-checklist

[^1_10]: https://www.apwide.com/enterprise-release-management/

[^1_11]: https://beefed.ai/en/release-readiness-checklist-go-no-go-templates

[^1_12]: https://beefed.ai/en/risk-based-go-no-go-framework-releases

[^1_13]: https://www.devopsschool.com/blog/release-engineer-role-blueprint-responsibilities-skills-kpis-and-career-path/

[^1_14]: https://www.cisco.com/c/dam/en_us/about/ciscoitatwork/downloads/ciscoitatwork/pdf/tt_ITIL_Change_and_Configuration_Management.pdf

[^1_15]: https://beefed.ai/en/change-freeze-windows-policy-scheduling-enforcement

[^1_16]: https://vantus.systems/process/change-windows-and-freeze-periods

[^1_17]: https://www.freshworks.com/itil/change-management/

[^1_18]: https://us.fitgap.com/stack-guides/defining-release-readiness-gates-to-prevent-last-minute-surprises

[^1_19]: https://www.wrixle.co.uk/journal/sample-raci-matrix-for-release-planning

[^1_20]: https://store.theartofservice.com/incident-management-in-release-and-deployment-management-dataset-publication-date-2024-01/

[^1_21]: https://www.scribd.com/document/968813102/Agile-Implementation-Backout-Plan-QE

[^1_22]: https://itiligence.co.uk/cab-support-go-no-go-criteria-sheet-lite/

[^1_23]: https://www.servicenow.com/docs/r/it-service-management/change-management/t_CreateBlkoutMaintSched.html

[^1_24]: https://support.atlassian.com/jira-service-management-cloud/docs/what-are-change-windows/

[^1_25]: https://harvard.service-now.com/ithelp/web browser?id=kb_article\&sys_id=a3cb7e250f947e802dfe5bd692050e91

[^1_26]: https://www.ilxgroup.com/uk/blog/guide-itsm-change-freezes

[^1_27]: https://www.scribd.com/document/703237287/Racie-Matrix

[^1_28]: https://www.slideteam.net/raci-matrix-for-release-management-process.html

[^1_29]: https://harvard.service-now.com/ithelp?id=kb_article\&sys_id=a3cb7e250f947e802dfe5bd692050e91

[^1_30]: https://aidevdayindia.org/blogs/enterprise-ai-governance-frameworks/pmo-guide-scaling-agentic-ai-agile-release-trains.html

