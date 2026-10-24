# DELIVERY — Comandos Reais, Git e Entrega ([NOME DO PROJETO])

> propósito: registrar como rodar, verificar, empacotar, publicar e recuperar este projeto sem adivinhação
> ler quando: antes de rodar testes, builds, empacotamento ou release
> atualizar quando: Fase 03 ou quando mudar comando/pipeline real

## 1. Comandos Reais (Marcar `hipótese:` se ainda não testado)
- **Setup:** `[comando]`
- **Lint / Typecheck:** `[comando]`
- **Testes:** `[comando]`
- **Build:** `[comando]`
- **Empacotamento / Artefato:** `[comando | N/A]`

## 2. Git, Versionamento e Ambientes
- **Perfil:** `[SIMPLE | STANDARD | CRITICAL]`
- **Branch de trabalho:** `[principal | branch_por_task | worktree_por_task]`
- **Formato de commit:** `<tipo>(<escopo>): <descrição> [UNIDADE]`
- **Versionamento / Tags:** `[SemVer / arquivo VERSION / package.json]`

## 3. Deploy, Verificação Pós-Entrega e Rollback
- **Destino / Ambiente:** `[local | npm/PyPI | Vercel/Cloudflare/VPS | instalador desktop]`
- **Smoke test pós-entrega:** `[como validar que subiu funcionando]`
- **Procedimento de rollback:** `[comando ou passo exato para reverter sem perda de dados]`
