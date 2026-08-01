---
aliases: ["ADR-004", "Controles customizados"]
tags: [adr]
atualizado: 2026-08-01
---

# ADR-004 · Componentes próprios no lugar de controles nativos

## Contexto

O app tem identidade visual forte: tema **dark-glass** sobre fundo roxo
(`--background: #1E0033`).

Dois controles nativos não colaboram:

- **`<select>`** — o dropdown é desenhado pelo SO. Dá para estilizar o gatilho,
  mas a lista de opções não.
- **`<input type="date">`** — **o calendário nativo não é estilizável por CSS**.
  Não há pseudo-elemento, não há hack; em cada navegador ele é diferente e
  nenhum combina com o tema.

## Decisão

**Controle nativo que não pode ser estilizado é substituído por componente
próprio.** Hoje: `Select` e `DatePicker`.

Ambos com o mesmo contrato: gatilho + popup, fecham no clique fora e no Esc,
`value`/`onChange` diretos.

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|
| Aceitar o visual nativo | Quebra a identidade justamente nos pontos mais usados (filtro de mês está em toda tela) |
| Biblioteca de UI (MUI, Radix, etc.) | Peso e opinião de estilo desproporcionais — o app tem 2 controles problemáticos, não um design system inteiro |
| Hacks de CSS no nativo | Não existem para o calendário; e o que existe para `<select>` quebra entre navegadores |

## Consequências

**Boas**
- Visual consistente em 100% da UI
- Comportamento previsível entre navegadores
- Sem dependência nova

**Custos**
- **Acessibilidade é nossa.** Navegação por teclado, `aria-*` e foco não vêm de
  graça — precisam ser verificados a cada mudança
- Sem suporte nativo de mobile (o seletor de data nativo do celular é bom)
- Mais código para manter

**A preservar** ⚠️
> **`onChange` recebe o VALOR, não um evento.**
> ```jsx
> <Select value={mes} onChange={setMes} />          ✅
> <Select onChange={e => setMes(e.target.value)} /> ❌
> ```
> Este é o erro mais comum ao migrar um `<select>` nativo — e **falha em
> silêncio**: o state vira um objeto de evento.

---

## Relacionadas
[[DatePicker-e-Select]] · [[Convencoes]]
