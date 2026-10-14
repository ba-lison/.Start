# FASE 05 — FECHAMENTO, ENTREGA E OPERAÇÃO

> propósito: comprovar que o conjunto funciona, gerar a entrega real, registrar o resultado e definir a próxima ação
> ler quando: unidade `BUILDxxx_CLOSE`
> pré-condição: todas as Sprints da Build fechadas

## Fechamento da Build

Em chat próprio, nesta ordem:

1. conferir Tasks e requisitos;
2. confirmar que todas as Sprints estão formalmente fechadas;
3. executar a suíte ampla, proporcional ao perfil;
4. executar build limpa;
5. conferir migração, segurança e compatibilidade aplicáveis;
6. atualizar `VERSION`, consolidar changelog e pendências (`.start/PROTOCOLO_MESTRE_EXECUCAO.md` §8, passos 1 a 3);
7. commit principal — é o commit de origem do artefato (§8, passo 4) — e tag local, quando prevista em `.memory/DELIVERY.md`;
8. gerar o artefato a partir desse commit, testar o artefato (não apenas o código) e calcular o SHA-256;
9. criar `.memory/RELEASES/RELEASE_BUILDxxx.md` (`.start/TEMPLATES/TEMPLATE_RELEASE.md`) com commit de origem, tag e SHA-256;
10. entregar (seções "Entrega sem deploy" ou "Deploy") e gravar no registro da Release o estado final (`RELEASED`, `READY_FOR_DEPLOY` ou outro);
11. pós-entrega: atualizar `.memory/PROJECT_CONTEXT.md` (entrega atual) e as pendências; registrar incidente e recuperação, quando houver;
12. perguntar ao usuário a próxima unidade (D-01; DEC): próxima Build, correção, monitoramento ou encerramento;
13. seguir o §8 a partir do passo 5. O registro da Release, o contexto e os índices entram no commit dos registros de fechamento.

## Entrega sem deploy

Validar pacote, instalação ou uso, hash e destino. Só então marcar `RELEASED`.

## Deploy

Executar somente se `.memory/DELIVERY.md` definir ambiente, comandos, gates, autorização e recuperação, e com autorização conforme GP-07:

```text
artefato imutável → homologação → verificação → aprovação
→ produção → smoke/health → observação → concluir ou recuperar
```

Sem autorização ou acesso: marcar `READY_FOR_DEPLOY`.
