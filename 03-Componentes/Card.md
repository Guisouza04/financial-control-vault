---
aliases: ["Card", "Cards"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/Card/index.jsx"]
atualizado: 2026-08-02
---

# Card

**`src/components/Card/index.jsx`**

O card dos dois hubs: [[Tela-Financas|Finanças]] (Contas, Investimentos,
Opcionais, Importar Extrato) e [[Tela-Settings|Configurações]] (Perfil, Dados,
Alterar Senha, Sair).

```jsx
<Cards
  name="Contas"
  subtitle="Despesas fixas do mês"
  hint="60% do plano"
  svgContent={<svg …/>}
/>
```

---

## Como ele se comporta

**Em repouso mostra só o ícone.** No hover o ícone cresce (30% → 65% da altura),
borra (`blur(7px)`) e flutua enquanto o texto aparece por cima; o card escala
1.04 com rotação de -1°.

`subtitle` e `hint` são **opcionais** — sem eles o card mostra só o nome. O
`hint` é a linha em destaque (`--Complementar`).

---

## Decisões

- **O texto só existe no hover — e isso precisou de duas saídas.**
  1. **Teclado:** `a:focus-visible &` revela o mesmo estado. A regra sobe um
     nível de propósito: o card **não** é focável, quem recebe foco é o
     `<Link>`/`<div>` em volta, que é *ancestral* — `:focus-within` no card não
     resolveria.
  2. **Toque:** em `@media (hover: none)` o efeito nunca dispararia e o card
     seria um ícone mudo. Lá ele volta ao layout empilhado (ícone em cima, texto
     embaixo, ambos visíveis) — o `position: static` no ícone é o que tira a
     sobreposição.
- **`overflow: hidden` no card não é enfeite:** o ícone cresce e borra; sem ele o
  borrão vaza pelas bordas arredondadas.
- **O CSS vence o `width`/`height` do `<svg>`.** Os ícones chegam com tamanho
  fixo no próprio elemento (60px, 80px…); é o `.img svg { height: 100% }` que os
  deixa crescer no hover. `width: auto` preserva a proporção de qualquer viewBox.
- **A prop `variant="preencherFill"` deixou de existir.** Ela escurecia o ícone
  quando o disco branco do desenho antigo clareava no hover — comportamento que
  este card não tem. Se você achar `variant` em alguma chamada, é resquício.
- **Do design original ficou só a estrutura**; cores e tipografia são as do
  sistema (vidro `--glass-bg`, `--text-muted`, `--Complementar`, escala em rem).

> **Os percentuais do `hint` em Finanças (60/20/10) vêm da divisão do
> [[Tela-Dashboard|Dashboard]].** Se o plano mudar, mudam aqui também — é texto
> escrito à mão, não derivado de `BUDGET`.

---

## Relacionadas
[[Tela-Financas]] · [[Tela-Settings]] · [[Tela-Dashboard]]
