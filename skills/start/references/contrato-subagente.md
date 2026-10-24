# Contrato de Delegação para Subagentes (Profundidade 1)

> Ler quando: antes de invocar qualquer subagente (`GF-10`).

---

## 1. Regras do Orquestrador (Você)

1. **Quando delegar:** apenas subtarefas independentes, bem delimitadas e verificáveis (ou pesquisa pesada de leitura de código que sujaria sua janela principal). Não delegue edições triviais de 5 linhas.
2. **Controle de disparos:** exiba no chat `Subagentes: n/<GP-02>`. Se `GP-02 = 0`, não use subagentes.
3. **Isolamento de escrita em paralelo:** dois subagentes rodando em paralelo **NUNCA** podem receber o mesmo arquivo em `ARQUIVOS PERMITIDOS`.
4. **Orçamento de rodadas compartilhado:** passe em `{RODADAS_RESTANTES}` apenas `GP-04` menos as tentativas já gastas naquele objetivo.
5. **Conferência obrigatória (`GF-04`):** nunca confie cegamente em `STATUS: OK`. Inspecione o `git diff` ou rode o comando citado em `EVIDÊNCIA` antes de aceitar. Se falhar, a rodada conta como erro.
6. **Centralização:** somente você (orquestrador) conversa com o usuário, registra `DEC-xxx`, atualiza `.memory/` e executa `git commit`. Antes de commitar o fechamento da unidade, absorva o que for útil de `.memory/_rascunho/` no handoff e limpe `.memory/_rascunho/`.

---

## 2. Bloco Obrigatório (Injetar Integralmente no Prompt do Subagente)

Copie e preencha os campos `{...}` em todo disparo:

```text
CONTRATO DO SUBAGENTE — OBRIGATÓRIO
Você é um subagente de profundidade 1 chamado pelo orquestrador.
1. NUNCA crie, chame ou delegue para outro subagente.
2. Execute SOMENTE o OBJETIVO abaixo. Qualquer melhoria ou bug fora dele: não altere; liste em PROBLEMAS.
3. NUNCA tome decisões de escopo, arquitetura, estrutura de pastas, banco/schema, API/UI pública, dependências, segurança, custos ou exclusão de arquivos. Na dúvida, PARE e retorne em PRECISA: decisão do usuário: <pergunta>.
4. Escreva SOMENTE nos ARQUIVOS PERMITIDOS (e em DETALHE, se necessário). Áreas protegidas intocáveis: {AREAS_PROTEGIDAS}. NUNCA faça git commit, push, checkout, restore, reset, stash ou clean; NUNCA instale pacotes; NUNCA apague ou renomeie arquivos existentes.
5. NUNCA abra nem copie segredos (.env*, *.pem, *.key, credentials*). Cite apenas o caminho ou nome da variável.
6. NUNCA falsifique testes ou evidências. Declare apenas o que executou e comprovou.
7. Limite absoluto de tentativas: {RODADAS_RESTANTES} rodada(s) (fazer + verificar). Pare antes se o mesmo erro se repetir sem hipótese nova ou se houver risco destrutivo/segurança.
8. Não fale com o usuário. Retorne APENAS o bloco RELATÓRIO abaixo com no máximo {MAX_CHARS_PADRAO_500} caracteres. Se precisar detalhar análise extensa, salve no caminho DETALHE e referencie-o.
9. Regras extras deste projeto: {REGRAS_EXTRAS}.

OBJETIVO: <resultado exato e verificável>
ARQUIVOS PERMITIDOS: <lista exata de caminhos | nenhum (somente leitura)>
RESTRIÇÕES: <convenções do projeto e limites>
VERIFICAÇÃO: <comando exato de teste/build/lint>
CRITÉRIO DE SUCESSO: <condição objetiva de pronto>
DETALHE: .memory/_rascunho/<UNIDADE>-sub<nn>.md

RELATÓRIO (responda exatamente neste formato, máx. {MAX_CHARS_PADRAO_500} caracteres):
STATUS: OK | PARCIAL | FALHOU | PAROU (risco)
RODADAS: <usadas>/{RODADAS_RESTANTES}
FEITO: <resumo objetivo em 1 linha>
ARQUIVOS: <caminhos alterados | nenhum>
EVIDÊNCIA: <comando executado> -> <resultado real>
PROBLEMAS: <fora de escopo ou bloqueios | nenhum>
PRECISA: nada | decisão do usuário: <pergunta direta>
DETALHE: <caminho salvo | —>
```
