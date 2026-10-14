# DECISÕES — [PROJETO]

> propósito: registro de toda decisão do projeto: o que foi decidido, por quem e por quê
> ler quando: início de todo chat — ler as entradas `ACTIVE` e `INFERRED`
> atualizar quando: no momento em que o usuário decide (GF-06); anexar ao fim, com número = maior DEC existente + 1; NUNCA apagar entrada
> status possíveis: ACTIVE | INFERRED | SUPERSEDED | REJECTED
> decidido_por possíveis: usuário | código existente (a confirmar) | usuário (confirmou decisão herdada em YYYY-MM-DD) | usuário (v2)
> substituição: nova entrada com `substitui: DEC-xxx`; na antiga, só o `status` muda para `SUPERSEDED`
> confirmação de herdada (D-12): na mesma entrada, só `status` e `decidido_por` mudam

## Modelo de entrada

Ao criar o arquivo no projeto: copiar só o título e o cabeçalho acima. O modelo abaixo não é uma entrada.

```text
## DEC-[NNN] — [título curto]

- data: YYYY-MM-DD
- status: ACTIVE
- unidade: [ONBOARDING | PRD-001 | BUILD001_S01_T01 | ...]
- pergunta: [o que foi perguntado | instrução direta do usuário]
- opcoes: [A) ...; B) ... | —]
- decisao: [o que foi decidido, em uma frase]
- decidido_por: usuário
- motivo: [por quê]
- consequencias: [o que muda; restrições futuras]
- substitui: [DEC-xxx | —]
- adr: [`.memory/adrs/ADR-xxxx.md` | —]
```
