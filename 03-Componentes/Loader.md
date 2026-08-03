---
aliases: ["Loader", "Carregando", "Spinner"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/Loader/index.jsx"]
atualizado: 2026-08-02
---

# Loader

**`src/components/Loader/index.jsx`**

O indicador de carregamento do sistema — quatro bolas pulsando em onda, na cor
`--Complementar`. **É o único**: nenhuma tela deve escrever "Carregando…" solto
de novo.

```jsx
<Loader />                            // só as bolas
<Loader label="Lendo arquivo…" />     // texto opcional, abaixo delas
```

---

## Onde está

| Tela | Onde | Rótulo |
|---|---|---|
| [[Tela-Dashboard\|Dashboard]] | `LoaderArea`, centrado na sobra da tela | — |
| [[ExpenseBox]] (Contas · Investimentos · Opcionais) | `LoaderArea`, centrado no card | — |
| [[Tela-Metas\|Metas]] | `Loading`, no lugar da lista | — |
| [[Tela-Importar-Extrato\|Importar Extrato]] | dentro da `DropZone` | "Lendo arquivo…" |

**Botões de submit ficam de fora de propósito** ("Salvando…", "Entrando…",
"Importando…"). O botão já é pequeno, já fica `disabled`, e o verbo diz **o que**
está acontecendo — coisa que quatro bolas não dizem. Trocar o texto por
animação perderia informação e ainda mudaria a largura do botão no meio do
clique.

---

## O loader é ADIADO — senão ele pisca

**`src/hooks/useDeferredLoading.js`**

```js
const showLoader = useDeferredLoading(loading);   // 500ms carência · 600ms mínimos
```

Trocar de aba nas finanças costuma resolver dentro da carência. O loader
aparecia e sumia em poucos frames, e era exatamente esse **lampejo** que
incomodava — não a espera.

**São DUAS travas, e a segunda é a que faltava:**

1. **Carência (500ms)** — nada aparece antes disso.
2. **Permanência mínima (600ms)** — tendo aparecido, o loader só sai depois
   desse tempo, mesmo que os dados já tenham chegado. Só com a carência, um
   carregamento pouco acima dela mostrava o loader por dois ou três frames: a
   espera sumia, o lampejo não.

O resultado é binário na percepção: ou o loader não aparece, ou fica tempo
suficiente para ser lido como intencional.

- **Durante a carência a tela não fica vazia de moldura:** o card/área continua
  em pé, só sem conteúdo. O que se vê é a moldura permanecendo e o conteúdo
  trocando.
- **O loader entra com fade** (`softEnter`, 0.35s, sem deslocamento — ele já se
  mexe sozinho). Aparecer de estalo é metade da sensação de pisca, mesmo com o
  tempo de tela correto.
- **Cuidado ao mexer no hook:** o ramo de `loading` sai cedo se o loader já está
  visível. Reagendar ali reescreveria o instante de entrada e esticaria a
  permanência mínima a cada render.
- **A chegada dos dados é animada** (`contentEnter`, em
  `src/styles/animations.js`) — 0.24s de fade + 6px de subida, para tirar o corte
  seco da troca. Aplique **condicional** (`${(p) => p.$ready && contentEnter}`)
  quando o elemento existe nos dois estados: é a troca de classe que dispara a
  animação. Em elementos que só existem no estado carregado (a grade de metas,
  por exemplo), pode ser fixa — o mount deles **é** a chegada dos dados.

Usam: [[ExpenseBox]] (as três telas de despesa), [[Tela-Metas|Metas]] e
[[Tela-Dashboard|Dashboard]].

---

## Decisões

- **`label` é opcional e existe por acessibilidade.** Com ele, o texto visível é
  o que o leitor de tela anuncia (`role="status"` + `aria-live="polite"`, bolas
  em `aria-hidden`). Sem ele, o wrapper cai num `aria-label="Carregando"`
  próprio — **o anúncio nunca some, mesmo sem texto na tela**. Passar os dois
  duplicaria.
- **A onda é feita de atrasos, não de keyframes diferentes.** Cada bola começa
  0.3s depois da anterior; o anel que se expande sai 0.9s depois da **sua**
  bola, então acompanha o encolhimento, não o crescimento. Mexer num delay
  isolado desmonta a onda.
- **O anel usa roxo translúcido** (`--loader-ring`), não a cor cheia: sólido,
  ele vira um disco opaco de 20px piscando sobre o vidro.
- **`prefers-reduced-motion` desliga a animação** e deixa as bolas paradas em
  opacidade decrescente. Pulso infinito é exatamente o que essa preferência
  pede para cortar.

> **Centrar é responsabilidade de quem usa.** O componente não se posiciona — as
> telas envolvem num wrapper (`LoaderArea` / `Loading`) com `place-items: center`
> e alguma altura, porque a sobra disponível é diferente em cada uma (o
> Dashboard tem a tela toda; o `ExpenseBox`, o card; Metas, o vão da lista).

---

## Relacionadas
[[Tela-Dashboard]] · [[ExpenseBox]] · [[Tela-Metas]] · [[Tela-Importar-Extrato]]
