---
aliases: ["DatePicker", "Select", "Controles customizados"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/Select/index.jsx", "FinancialControll2.0/src/components/DatePicker/index.jsx"]
atualizado: 2026-08-01
---

# DatePicker e Select

Dois componentes próprios que **substituem controles nativos** em todo o app.

| Componente | Substitui | Por quê |
|---|---|---|
| `Select` | `<select>` | O dropdown nativo não aceita o tema dark-glass |
| `DatePicker` | `<input type="date">` | **O calendário nativo não é estilizável por CSS** — nem com hacks |

> Esta é uma convenção do projeto, não preferência pontual: **controle nativo
> que não dá para estilizar é substituído.** Ver [[Convencoes]].

---

## Contrato compartilhado

Ambos seguem o mesmo desenho:

- **Gatilho + popup**
- Fecham no **clique fora** e no **Esc**
- `value` / `onChange` **diretos** — `onChange` recebe o **valor**, não um
  evento:

```jsx
<Select value={mes} onChange={setMes} options={[{ value, label }]} />
```

> **Atenção:** `onChange={setMes}` e não `onChange={e => setMes(e.target.value)}`.
> É a diferença mais comum ao trocar um `<select>` nativo por este componente —
> e falha silenciosamente (o state vira um objeto de evento).

---

## Onde aparecem

**Select:** filtros de mês do [[ExpenseBox]] e do [[Tela-Dashboard|Dashboard]],
tipo de recorrência, tipo de lançamento, período de pagamento na
[[Tela-Settings]], tipo por linha na [[Tela-Importar-Extrato]].

**DatePicker:** vencimento da fatura na [[Tela-Importar-Extrato]] — hoje o
principal caso de data explícita do sistema.

---

## Custo a ter em mente

São componentes próprios, então **acessibilidade e comportamento de teclado
são responsabilidade nossa** — não vêm de graça como no nativo. Navegação por
setas, `aria-*` e foco precisam ser verificados a cada mudança.

---

## Relacionadas
[[Convencoes]] · [[ExpenseBox]] · [[Tela-Importar-Extrato]] · [[Tela-Settings]]
