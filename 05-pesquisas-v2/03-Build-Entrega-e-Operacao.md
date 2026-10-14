# Build, Entrega e Operação

## Resumo executivo

Release engineering deve começar junto com o projeto, não depois do código. O Google define a disciplina como o caminho completo do repositório até build, testes, empacotamento e deploy, com processos repetíveis, automatizados e auditáveis.[^1] Para projetos pequenos, isso não exige uma plataforma corporativa: exige comandos claros, dependências fixadas, artefato identificável, verificação real e um procedimento de entrega.

## P10 — `DELIVERY.md` por tipo de projeto

O contrato precisa ser curto e refletir o repositório. Campos universais:

- tipo de produto e destinatário;
- requisitos de ambiente;
- comandos de instalar, validar, buildar e empacotar;
- arquivos produzidos;
- versão e origem Git;
- destino da entrega;
- verificação pós-entrega;
- recuperação;
- autoridade para ações externas.

Perfis:

| Perfil | Acrescentar |
| :--- | :--- |
| Biblioteca/pacote | registry, compatibilidade, API pública |
| CLI/desktop/plugin | instalador, assinatura, SO/runtime, upgrade/uninstall |
| Web estático | preview, cache/CDN, smoke de URL |
| API/serviço | ambientes, migrações, health, observabilidade, rollback |
| Mobile | signing, store, canais, rollout e crash metrics |

`N/A` é válido somente com justificativa. O documento deve apontar para scripts e workflows, evitando duplicar YAML.

## P11 — Builds reproduzíveis

Uma build repetível usa as mesmas etapas; uma build reproduzível busca produzir saída equivalente a partir das mesmas entradas. O Google SRE recomenda builds reproduzíveis, automatizadas, testadas e identificáveis.[^1][^2]

### Níveis práticos

1. **Básico:** SDK documentado, lockfile, comando único e ambiente limpo.
2. **Controlado:** container/toolchain fixado, dependências verificadas e CI equivalente.
3. **Reproduzível:** duas execuções limpas comparadas; diferenças explicadas.
4. **Atestado:** proveniência assinada e materiais registrados.

Nem todo projeto precisa de igualdade bit a bit. O nível básico é obrigatório; os demais dependem do risco e distribuição.

## P12 — Pipeline mínimo e paridade local/CI

Pipeline mínimo:

```text
checkout
→ toolchain/dependências fixadas
→ lint/tipos
→ testes proporcionais
→ build limpa
→ teste do artefato
→ publicar artefato somente em evento autorizado
```

Regras:

- comandos centrais vivem em scripts executáveis pelo desenvolvedor e CI;
- PR valida; merge pode gerar candidato; tag ou aprovação publica;
- deploy deve ser idempotente ou protegido contra repetição;
- logs detalhados ficam na CI; Markdown guarda resumo e link/ID;
- falha de publicação nunca é descrita como Release concluída.

## P13 — Git para Tasks de agentes

Política padrão simples:

- branch principal protegida;
- uma branch ou worktree por mudança concorrente;
- commits pequenos e coerentes, não necessariamente um único commit por Task;
- Task ID no PR ou commit quando útil;
- PR para risco/repositório compartilhado; commit direto permitido em projeto pessoal conforme política;
- integração frequente e curta.

Trunk-based development favorece integração frequente; branches longas aumentam divergência. Worktrees são apropriados para agentes paralelos, pois isolam arquivos sem duplicar o repositório Git.[^3]

## P14 — Versionamento

Separar:

- `BUILD001`: identificador do plano ou execução interna;
- `run ID`: execução específica da CI;
- `v1.2.3`: versão pública do produto;
- commit SHA: origem imutável;
- hash do artefato: identidade do arquivo entregue.

SemVer é adequado quando consumidores dependem de uma API/contrato: major para incompatibilidade, minor para funcionalidade compatível e patch para correção compatível.[^4] Projetos internos podem usar CalVer ou versão simples, desde que a política seja inequívoca.

Atualizar versão na preparação da Release ou automatizar a partir de metadados aprovados. Conventional Commits pode alimentar automação, mas não deve obrigar bump baseado em mensagens incorretas.[^5]

## P15 — Artefatos e proveniência

Registro mínimo:

- nome, versão e plataforma;
- commit de origem;
- workflow/run;
- SHA-256;
- data e destino;
- resultado do smoke test.

SLSA define proveniência como atestado de quem construiu, como construiu e quais materiais foram usados.[^6] Para projeto pessoal local, hash + commit pode bastar. Para distribuição pública ou cadeia crítica, adicionar SBOM, assinatura e attestations.

Promover o mesmo artefato já testado é preferível a recompilar para cada ambiente.

## P16 — Homologação, entrega e deploy

Estados separados:

```text
BUILT → VERIFIED → RELEASE_CANDIDATE → APPROVED
→ DELIVERED/DEPLOYED → OBSERVED → RELEASED
```

- `BUILT`: arquivo existe.
- `VERIFIED`: testes do artefato passaram.
- `APPROVED`: autoridade aceitou promoção.
- `DELIVERED`: pacote chegou ao destino.
- `DEPLOYED`: ambiente foi alterado.
- `RELEASED`: validação pós-entrega concluída.

Sem acesso ou autorização: `READY_FOR_DEPLOY`. O agente deve relatar exatamente o último estado comprovado.

## P17 — Rollback, roll-forward e migrações

Rollback de binário só é seguro se versão anterior continuar compatível com dados e configuração. Para banco, preferir mudanças expand/contract:

1. adicionar estrutura compatível;
2. implantar código que convive com versões;
3. migrar/backfill;
4. remover estrutura antiga em Release posterior.

Migrações destrutivas exigem backup testado, janela, aceite e estratégia de recuperação. Em alguns casos roll-forward é mais seguro que reverter dados.

## P18 — Observabilidade e incidentes

Antes do deploy, definir sinais de saúde:

- disponibilidade/health;
- taxa de erro;
- latência ou duração relevante;
- saturação/recursos quando aplicável;
- métrica funcional crítica;
- crashes para desktop/mobile.

Canary é exposição parcial e temporária avaliada contra controle; reduz o impacto potencial e deve usar critérios ligados a SLOs.[^2] Projeto simples pode usar smoke test manual, janela curta e instrução de rollback.

Incidentes relevantes geram registro sem culpa: impacto, linha do tempo, causa contribuinte, detecção, resposta e ações com responsáveis. O Google SRE recomenda postmortems voltados a aprendizado, não punição.[^7]

## Decisões propostas

1. `DELIVERY.md` curto e baseado no tipo de projeto.
2. Nível básico de build controlada obrigatório; níveis avançados por risco.
3. Build, Sprint e versão pública são identificadores diferentes.
4. Todo artefato entregue liga versão, commit e hash.
5. Estados de Release comprováveis substituem “concluído” ambíguo.
6. Deploy externo exige autorização e verificação pós-deploy.
7. Rollback nunca é presumido seguro; dados e configuração entram no plano.

## Fontes

[^1]: Google SRE. [Release Engineering](https://sre.google/sre-book/release-engineering/).
[^2]: Google SRE Workbook. [Canarying Releases](https://sre.google/workbook/canarying-releases/).
[^3]: Git. [git-worktree](https://git-scm.com/docs/git-worktree).
[^4]: Semantic Versioning. [SemVer 2.0.0](https://semver.org/).
[^5]: Conventional Commits. [Specification 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).
[^6]: SLSA. [Provenance](https://slsa.dev/spec/v1.0/provenance).
[^7]: Google SRE. [Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/).

