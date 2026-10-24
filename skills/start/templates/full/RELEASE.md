# RELEASE — `BUILD[NNN]` (`vX.Y.Z`)

> propósito: comprovante auditável de build, artefato, verificação e entrega (imutável após fechado)
> ler quando: auditoria de entrega, deploy ou rollback
> data: YYYY-MM-DD
> estado: [READY_FOR_DEPLOY | RELEASED | ROLLED_BACK]

## Rastreabilidade
- **Versão:** `vX.Y.Z`
- **Commit de origem / Tag:** `[hash] / [tag | —]`
- **Artefato gerado:** `[caminho ou URL | código-fonte]`
- **SHA-256 do pacote (quando aplicável):** `[hash | N/A]`

## Verificações Executadas
- **Testes:** `[comando]` → `[resultado]`
- **Build limpa:** `[comando]` → `[resultado]`
- **Validação pós-entrega / Smoke test:** `[resultado real | pendente de autorização de deploy]`

## Rollback Rápido
- `[comando/procedimento para voltar à versão anterior segura]`
