---
aliases: ["ExpenseBox"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/ExpenseBox/index.jsx", "FinancialControll2.0/src/components/PageDots/index.jsx"]
atualizado: 2026-08-02
---

# ExpenseBox

**`src/components/ExpenseBox/index.jsx`**

O componente mais importante do frontend. **Três telas inteiras são ele** —
[[Tela-Despesas|Contas, Investimentos e Opcionais]] diferem apenas na prop
`tipo`.

```jsx
<ExpenseBox tipo={1 | 2 | 3} />
```

> **Mudança de comportamento de listagem vai aqui.** Alterar uma das páginas
> muda uma só e cria divergência silenciosa entre as três.

---

## Responsabilidades

| Área | O que faz |
|---|---|
| **Dados** | Consome [[Camada-de-Dados\|useAccounts(tipo, ano, mes)]] |
| **Filtros** | Ano (input) · Mês (Select) · Status · Tag · **Busca** (atrás da 🔍) |
| **Tabela** | Nome (+chips +💳) · Valor · Parcela · Status · Ações |
| **Paginação** | Dinâmica, por altura disponível · bolinhas ([[ExpenseBox#Indicador de páginas (PageDots)\|PageDots]]) |
| **Modal** | Criar **e** editar — o mesmo |
| **Pagamento** | Toggle por competência, com confirmação |

---

## Paginação dinâmica

Itens por página são calculados pela **altura disponível** da tabela, via
`ResizeObserver` no `TableWrapper`. A tabela ocupa a altura da tela e enche de
linhas.

Layout: `flex` do `.frame` até o corpo, cabeçalho `sticky`, scroll no
`TableWrapper`.

> Não há constante de "itens por página". Redimensionar a janela muda a
> paginação. Se você mexer na altura de linha ou no header, a conta muda junto.

---

## Indicador de páginas (PageDots)

**`src/components/PageDots/index.jsx`** — entre "Anterior" e "Próxima", no lugar
do antigo texto "Página X de Y". Bolinhas na horizontal: a atual acesa com um
brilho radial roxo, as demais como poços escuros.

```jsx
<PageDots totalPages={n} currentPage={p} onChange={setPage} />
```

- **São `<input type="radio">` de verdade**, não divs. Um grupo de radio já traz
  navegação por ← → e leitura correta em leitor de tela sem código nenhum. O
  `name` vem de `useId()`: dois grupos com o mesmo `name` viram um só, e
  selecionar num apagaria a seleção do outro.
- **O brilho nunca é repintado — ele se move.** O gradiente fica sempre no
  `background-image`; apagado é o gradiente deslocado 16px (o tamanho do dot)
  para fora do círculo. `:checked ~ input` desloca para o lado oposto, e é isso
  que faz a luz *atravessar* a fileira em vez de piscar. Mexer no tamanho do dot
  exige mexer no deslocamento junto, senão o apagado vaza um resto de brilho.
- **Especificidade dobrada (`&&`) não é enfeite:** o `globalStyles` estiliza
  `input` e `input:focus` (padding, borda, anel). Com força igual, o vencedor
  dependeria da ordem de injeção do CSS e o foco global desenharia um retângulo
  por cima da bolinha.
- **Máximo de 7 bolinhas.** Acima disso vira janela deslizante centrada na
  página atual, com as pontas em `scale(.6)` sinalizando "continua para cá" — e
  aí (e **só** aí) aparece o contador `6 / 12`, porque as bolinhas deixaram de
  representar o total.

---

## Filtro de período — use os helpers

- Com `recorrencia` → `isActiveInPeriod` / `occurrenceLabel`
- **Legado** (sem `recorrencia`) → janela por `creationMonth` + `durationMonths`
- Sem filtro → mostra tudo

> **Nunca reimplemente essa regra.** `accountActiveInPeriod` de
> `src/utils/recurrence.js` encapsula os dois casos, e é o que garante que o
> [[Tela-Dashboard|Dashboard]] some exatamente o que esta tela lista.
> Ver [[Recorrencia]].

---

## Busca — é o ÚLTIMO elo da cadeia de filtros

Campo de texto na barra de filtros, incremental (filtra a cada tecla, sem
debounce — a filtragem é client-side sobre uma lista já em memória).

### Fica escondido atrás da 🔍

O campo **não** aparece de saída: quem o revela é o `SearchToggle`, um botão de
lupa ao lado do "Mês Atual". Buscar é uso eventual, e a barra de filtros (ano,
mês, ‹ ›, status, tag, "Mês Atual") já é longa — um campo grande e quase sempre
vazio custava uma linha inteira do `Toolbar`.

> **Fechar a lupa LIMPA o termo** (Esc dentro do campo também fecha). Sem isso, o
> usuário fecharia a busca com "mercado" digitado e ficaria olhando uma lista
> curta **sem nada na tela dizendo por quê** — um filtro ativo invisível. Aberta,
> a lupa fica com a borda de marca, para não parecer um botão qualquer.

**O campo se expande da lupa para o lado**, na mesma linha, até o fim da barra de
filtros — não é mais uma barra ocupando a largura toda. Lupa e campo vivem juntos
no `SearchArea`, que já ocupa a sobra da linha **com a busca fechada**: abrir só
preenche um vão que já existia, então nenhum filtro ao redor se mexe durante a
animação.

> **O `SearchInput` fica sempre montado** (a visibilidade é a prop `$open`, que
> anima `width`/`padding`/`opacity`). Desmontar no `false` cortaria a saída pela
> metade — o campo sumiria de estalo. Fechado ele sai da ordem de tabulação
> (`tabIndex={-1}` + `aria-hidden`) para não virar um alvo invisível de Tab, e
> `min-width: 0` é obrigatório: sem isso o tamanho intrínseco do `<input>` impede
> encolher até zero.

> **Aberto, o `SearchArea` pede `min-width: 22rem`.** Se a sobra da linha for
> menor, a área inteira (lupa junto) quebra para a linha de baixo e lá ocupa o vão
> todo — melhor que um campo de 4rem espremido no canto.

> **O botão "Adicionar" não pode descer de linha.** O `FilterContainer` tem
> `flex: 1 1 auto; min-width: 0` e o botão irmão `flex: 0 0 auto`: os filtros
> comem a sobra e quebram **dentro de si**. Sem isso, abrir a busca empurrava o
> "Adicionar" para uma segunda linha do `Toolbar`, onde ele reaparecia colado à
> **esquerda** — foi exatamente o que aconteceu quando a busca era fixa.

```js
filteredAccounts        // período + status + tag
  → visibleAccounts     // + busca  ← o que a tabela mostra
```

**A ordem importa.** A busca roda sobre o resultado dos outros filtros, nunca
sobre a lista crua: digitar "mercado" com Julho selecionado não pode trazer um
lançamento de Agosto. É a mesma regra em [[Tela-Metas]] e
[[Tela-Importar-Extrato]], e o util é compartilhado:

- **`src/utils/search.js`** — `normalizeText` (minúsculas + NFD sem acento, para
  "educacao" achar "Educação") e `matchesSearch(term, ...campos)` (todos os
  termos digitados precisam aparecer em algum campo; termo vazio passa tudo).

**Casa por nome do lançamento E por nome das tags** — a tag é a natureza do
gasto, então buscar "mercado" tem que achar tanto a conta chamada Mercado quanto
as etiquetadas com ela. Ver [[Tag]].

> **A busca NÃO altera o total.** `totalPayable` continua somando
> `filteredAccounts` — o quanto você deve no mês não muda porque você digitou uma
> palavra. Quando há busca ativa, a `TotalBar` diz isso em letra miúda, senão o
> número "não bate" com as linhas e parece bug. **Decisão de produto: não
> "conserte" isso somando `visibleAccounts`.**

Detalhes que o código já resolve e que quebram se removidos:
- `setCurrentPage(1)` no `onChange` — buscar estando na página 3 mostraria uma
  tabela vazia com resultados existindo na página 1.
- Linha de estado vazio quando a busca não acha nada — o `<tbody>` vazio faz a
  tabela parecer quebrada.
- O `SearchInput` é filho **direto** do `FilterContainer` (um `<input>`, sem
  wrapper): a regra `& > div { width: 18rem }` travaria qualquer wrapper. E é o
  **último** da linha, para ser ele a quebrar quando os filtros enchem a largura.

---

## O bloqueio do toggle em "Todos os meses"

Um `MENSAL` sem mês selecionado tem competência **ambígua**
(`occurrenceCompetencia` devolve `null`). O toggle **bloqueia** e pede para
selecionar um mês.

A [[Tela-Metas]] trata a mesma ambiguidade de forma **oposta** (mira o próximo
aporte pendente) — porque lá o usuário quer avançar, não quitar um mês
específico. Ver [[Competencia-e-Pagamento]].

---

## Modal único para criar e editar

Nome · Valor (máscara BRL) · Recorrência (Única/Mensal/Anual + campos
condicionais) · [[TagPicker]].

- Editar (✏️) pré-preenche via `deriveRecurrenceForm(account)`, que também
  converte registros legados
- **O início é preservado** na edição — dá para mudar o fim, converter
  única↔mensal, mas não reescrever a data de origem
- **Não passa `fimAno`** → o fim fica no ano do início. Só [[Tela-Metas]] passa.
  Ver [[Recorrencia]]

### O modal vai para o `<body>` por portal

O overlay é renderizado com `ModalPortal`
(`src/components/ModalPortal/index.jsx`, `createPortal` no `document.body`) —
**não** no lugar onde o JSX está.

> **Por quê:** o `Container` do ExpenseBox é uma superfície de vidro
> (`backdrop-filter`), e isso **cria um stacking context** e o faz virar o
> containing block de `position: fixed`. Renderizado lá dentro, o overlay ficava
> preso na caixa: o `z-index: 1000` do `.modalOverlay` só valia ali, e qualquer
> irmão da caixa que ganhasse um `transform` era pintado **por cima do modal** —
> foi o que aconteceu com o botão "Voltar", cujo `:hover` aplica
> `transform: scale(0.95)`. Não adianta subir o z-index do overlay nem baixar o
> do botão; o conserto é tirar o modal de dentro do contexto.
>
> Efeito colateral desejado: o overlay passou a cobrir a **viewport inteira**, e
> não só a área do card.

> Os outros modais do sistema ([[Tela-Metas|Metas]], [[Tela-Dados|Settings]],
> [[QuickAddModal]], `ConfirmDialog`) ficam em nível de página, sem ancestral de
> vidro, e por isso não têm o problema. Modal novo **dentro** de um card de vidro
> → use o `ModalPortal`.

---

## `TIPO_OPTIONS` exclui o tipo 4

[[Meta|Metas]] não são criáveis aqui. É deliberado — criar meta é ação da
página `/metas`.

---

## Reuso dos styled components

O [[QuickAddModal]] **importa os styled components daqui** em vez de duplicar.
Mexer nos estilos do modal afeta os dois.

---

## Relacionadas
[[Tela-Despesas]] · [[Recorrencia]] · [[Competencia-e-Pagamento]] · [[Tag]] · [[TagPicker]] · [[QuickAddModal]] · [[Camada-de-Dados]] · [[Loader]]
