<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/readme/hero-grafite.svg">
  <img src="assets/readme/hero-papel.svg" alt=".Start · Processo e memória para agentes de IA" width="100%">
</picture>

<h1 align="center">.Start</h1>

<p align="center">Processo de engenharia e memória de projeto para agentes de IA, em dois trilhos proporcionais ao trabalho.</p>

<p align="center">
  <img alt="formato: skill" src="https://img.shields.io/badge/formato-skill-4C6972?style=flat-square&amp;labelColor=202725">
  <img alt="idioma: pt-BR" src="https://img.shields.io/badge/idioma-pt--BR-5C6554?style=flat-square&amp;labelColor=202725">
  <a href="https://github.com/ba-lison/.Start/blob/main/LICENSE"><img alt="licença: MIT" src="https://img.shields.io/badge/licen%C3%A7a-MIT-5C6554?style=flat-square&amp;labelColor=202725"></a>
</p>

<p align="center">
  <a href="#começar-a-usar">Começar a usar</a> &nbsp; / &nbsp;
  <a href="#demonstração-e-recursos">Ver recursos</a> &nbsp; / &nbsp;
  <a href="#documentação-e-ajuda">Documentação</a>
</p>

---

## Sobre o projeto

O .Start é uma skill para orientar agentes de IA durante projetos de software. Ela organiza o diagnóstico inicial, o escopo, as decisões e a continuidade entre conversas, mantendo o motor do processo nas skills do agente e os registros específicos do produto em .memory/ dentro do repositório.

Na versão 4, a antiga pasta extensa copiada para cada projeto deu lugar a uma skill modular: referências e modelos são lidos quando necessários. O método atende desde scripts, automações e plugins curtos até produtos com vários módulos e etapas de entrega. A proposta é manter decisões visíveis, conferir o estado real do Git e registrar o ponto de retomada sem carregar todo o processo em cada conversa.

O agente primeiro identifica se vai iniciar um projeto, adotar o processo em um código existente, continuar uma memória ativa ou migrar uma estrutura antiga. A partir desse diagnóstico, recomenda um fluxo de trabalho e apresenta o escopo para confirmação.

## Demonstração e recursos

### Dois trilhos

| Trilha | Indicada para | Memória do projeto |
| --- | --- | --- |
| **LITE** | Scripts, automações, CLIs simples, plugins curtos, POCs e tarefas pequenas. | Três arquivos: <code>00_INDEX.md</code>, <code>DECISIONS.md</code> e <code>HANDOFF.md</code>. |
| **FULL** | SaaS, produtos médios ou grandes, vários módulos, equipes ou operação de maior risco. | Estrutura modular para contexto, guardrails, PRD, SDD, builds, sprints, tarefas e releases. |

No LITE, o escopo e os guardrails são confirmados em um bloco curto e a implementação pode seguir na mesma conversa após a aprovação. No FULL, o trabalho avança por unidades, como onboarding, PRD, plano de build, tarefas e release. Ambos registram decisões e handoffs; o tamanho da estrutura acompanha o trabalho.

### Fluxo de entrada

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/readme/demo-arquitetura-grafite.svg">
  <img alt="Diagnóstico inicial e escolha entre os trilhos LITE e FULL" src="assets/readme/demo-arquitetura-papel.svg" width="100%">
</picture>

Os modos de entrada cobrem projeto novo (<code>NOVO</code>), código existente sem memória (<code>ADOTAR</code>), projeto com <code>.memory/</code> ativa (<code>CONTINUAR</code>) e estruturas legadas (<code>MIGRAR</code>). Em cada caso, o agente segue a sequência e os modelos correspondentes, em vez de carregar todos os arquivos de apoio de uma vez.

## Começar a usar

### Instalar e iniciar

A forma rápida indicada no repositório usa a CLI <code>skills</code>:

    npx skills add ba-lison/.Start

Também é possível instalar manualmente copiando a pasta <code>skills/start</code> para o diretório de skills do agente. O [guia de uso](https://github.com/ba-lison/.Start/blob/main/docs/COMO-USAR.md) mostra os caminhos para Claude Code, Codex, Cursor, Antigravity/Gemini e OpenCode, incluindo a opção de compartilhamento agnóstico por <code>.agents/skills</code>.

Abra a pasta do projeto de software no agente e acione <code>/start</code>; no Codex, use <code>$start</code>. Em linguagem natural, também é possível pedir para iniciar ou continuar um projeto com a skill. O primeiro passo é um diagnóstico somente de leitura. O agente resume o que encontrou, recomenda LITE ou FULL e pede confirmação antes de criar ou alterar arquivos. Ao adotar um repositório, a inspeção inclui código, instruções existentes e estado do Git. Ao continuar, confere o repositório e o último handoff.

Após a confirmação, os arquivos de memória ficam no próprio projeto. Eles registram o contexto e as decisões daquele produto; a instalação da skill fornece o método. O .Start orienta o agente e não cria isolamento de arquivos ou uma barreira técnica contra alterações: permissões e proteções continuam dependendo do ambiente usado.

## Documentação e ajuda

### Guias e referências

- [Instalação, uso e convivência com workspaces](https://github.com/ba-lison/.Start/blob/main/docs/COMO-USAR.md)
- [Instruções principais da skill](https://github.com/ba-lison/.Start/blob/main/skills/start/SKILL.md)
- [Modos de entrada](https://github.com/ba-lison/.Start/blob/main/skills/start/references/modos-entrada.md)
- [Guardrails e decisões](https://github.com/ba-lison/.Start/blob/main/skills/start/references/guardrails-e-decisoes.md)
- [Etapas do trilho FULL](https://github.com/ba-lison/.Start/blob/main/skills/start/references/fases-full.md)
- [Exemplo de memória FULL preenchida](https://github.com/ba-lison/.Start/blob/main/docs/exemplos/full-sdd/README.md)
- [Biblioteca de pesquisas](https://github.com/ba-lison/.Start/blob/main/docs/pesquisas/README.md)

Os materiais de pesquisa mantêm as fontes consultadas para fundamentar as regras. Para relatar um problema ou sugerir uma melhoria, use as issues do repositório: https://github.com/ba-lison/.Start/issues.

### Créditos e materiais de terceiros

Os materiais de pesquisa mantêm a atribuição a seus autores e fontes originais.

## Autoria e licença

Desenvolvido por [Alison Cruz](https://github.com/ba-lison) · **BALISON**.

Distribuído sob a [licença MIT](https://github.com/ba-lison/.Start/blob/main/LICENSE).

---

<p align="center">
  <a href="https://github.com/ba-lison">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="assets/readme/assinatura-grafite.svg">
      <img src="assets/readme/assinatura-papel.svg" alt="Alison · BALISON" width="280" height="28">
    </picture>
  </a>
  <br>
  <sub>Tecnologia com traço humano.</sub>
</p>
