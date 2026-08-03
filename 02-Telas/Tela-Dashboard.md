---
aliases: ["Dashboard", "Home", "Tela inicial"]
tags: [tela]
rota: "/"
codigo: ["FinancialControll2.0/src/pages/Home/index.jsx", "FinancialControll2.0/src/components/Loader/index.jsx"]
atualizado: 2026-08-02
---

# Tela · Dashboard

**Rota:** `/` · **Componente:** `src/pages/Home`

A tela inicial e o **produto em si**: todo o resto do sistema existe para
alimentar esta visão. Ela responde uma pergunta só —
**"meu mês está aderente ao plano 60/20/10/10?"**

---

## O que ela entrega

| Seção | Responde |
|---|---|
| **KPIs** | Salário, comprometido, saldo livre, % em Contas |
| **Quitação do mês** | Quanto já **saiu do bolso** e o que ainda vence |
| **Medidores** | Cada bucket está dentro do limite? (gasto × limite) |
| **Donut + legenda** | Como o dinheiro se distribui **de fato** × como deveria |
| **Gastos por tag** | *Em que* o dinheiro foi, independente de bucket |

Layout: `InsightGrid` de duas colunas (medidores + distribuição lado a lado),
colapsando para uma coluna abaixo de **1024px**.

---

## O que o usuário consegue fazer

- **Filtrar mês/ano** (default: atual), com navegação de mês
- **Criar lançamento sem sair da tela** — botão ⚡ (`QuickAddButton`) abre o
  [[QuickAddModal]]. Ao salvar, `onCreated(tipo)` refaz o fetch **daquele tipo**,
  atualizando o resumo na hora.
- **Pular direto para a tela da finança** — cada medidor e cada linha da legenda
  do donut tem um **👁** que leva a Contas / Investimentos / Opcionais /
  Metas. Antes disso, ver que "Investimentos está atrás da meta" exigia ir em
  Finanças → Investimentos na mão.
- Navegar para as telas de detalhe pelos links dos estados vazios

---

## De onde vêm os dados

```
useAccounts(1) ─┐
useAccounts(2) ─┼─→ sumActiveInPeriod(accounts, ano, mes) → medidores/donut
useAccounts(3) ─┤   accountActiveInPeriod + account.tags  → gastos por tag
useAccounts(4) ─┘   accountActiveInPeriod + isPaidInPeriod
                    + account.diaVencimento              → quitação do mês
financeService.fetchSalary() ────────────────────────────→ KPIs e limites
```

> **Os quatro hooks buscam SEM filtro**, uma vez cada. A soma por período é
> client-side. É o mesmo caminho que as telas de despesa usam —
> ver [[ADR-005-Filtro-de-periodo-no-frontend]].
>
> **Consequência:** se o Dashboard e a listagem discordarem, alguém
> reimplementou o filtro em vez de usar `accountActiveInPeriod`.

---

## `BUDGET` — a configuração central

```js
{ tipo: 1, label: "Contas",        pct: 60, kind: "teto", color: "#3987e5", route: "/Contas"      }
{ tipo: 2, label: "Investimentos", pct: 20, kind: "meta", color: "#199e70", route: "/Investments" }
{ tipo: 3, label: "Opcionais",     pct: 10, kind: "teto", color: "#c98500", route: "/Optional"    }
{ tipo: 4, label: "Metas",         pct: 10, kind: "meta", color: "#d55181", route: "/metas"       }
```

**`kind` é a regra que mais importa:** `teto` fica vermelho ao **exceder**;
`meta` fica verde ao **atingir**. A mesma barra em 80% lê-se de formas opostas.
Ver [[Tipo]].

**Cores de status** (`STATUS`: `good #3ddc84`, `warning #fab219`,
`critical #ff6b6b`) **nunca** são reutilizadas como cor de série — senão um
bucket ficaria verde por acaso e leria como aprovação.

**`route`** alimenta o 👁 das duas seções. Como `buckets` faz spread de `BUDGET`,
basta estar aqui para chegar tanto no medidor quanto na legenda — não existe um
segundo mapa tipo→rota para sair de sincronia.

---

## O 👁 de atalho

Um `<Link>` (`EyeLink`) — o mesmo componente nos medidores e na legenda do
donut, em posições diferentes. Ícone SVG inline com `currentColor`; o projeto
não tem biblioteca de ícones. Ver [[Nav]] para o padrão de SVG.

- **No medidor:** no **cabeçalho** (`MeterHead`), à direita, colado no valor
  gasto — não no rodapé. O `MeterHead` é `space-between` entre o nome e o valor;
  com o 👁 como terceiro filho direto, o valor iria para o **centro**. Por isso
  valor e 👁 vivem juntos dentro de `MeterHeadRight`.
- **Na legenda:** no **fim** da linha, depois da comparação Real × Plano. A
  legenda é um **grid de colunas fixas** — o 👁 acrescentou uma 5ª coluna
  (`2.6rem`), e o `LegendHeadRow` precisa de uma célula vazia
  (`<span aria-hidden="true" />`) para o cabeçalho não desalinhar do corpo.

Abaixo de 460px o "Plano" some da legenda, mas **o 👁 fica**: ele é atalho, não
informação.

> **`MeterFootRight` é resquício.** O 👁 nasceu no rodapé do medidor, agrupado
> com o "% da fatia" pelo mesmo motivo de `space-between` acima. Com o 👁 no
> cabeçalho, esse wrapper embrulha um filho só e pode sair — o rodapé
> (`StatusPill` + hint) já se resolve com dois filhos diretos.

---

## Quitação do mês — a única seção que olha o PAGAMENTO

Todo o resto da tela soma o **comprometido** (pago ou não). Esta seção é a única
que consulta o status, via `isPaidInPeriod` — a **mesma** função que pinta
"Paga"/"Pendente" no [[ExpenseBox]]. Se as duas telas discordarem, alguém
reimplementou a regra. Ver [[Competencia-e-Pagamento]].

Fica **logo abaixo dos KPIs**, antes dos medidores: é a única parte acionável da
tela ("o que eu preciso pagar"); o resto é leitura do plano.

**Dois painéis num card:**

| Painel | Conteúdo |
|---|---|
| Esquerda | `R$ pago de R$ total`, contagem de lançamentos, **barra empilhada** pago · atrasado · a vencer, legenda com valores |
| Direita | **Próximos vencimentos** — pendentes do mês ordenados por dia, 6 na lista + "+N além destes" |

### Decisões

- **Inclui os quatro tipos, metas junto.** O "comprometido" dos KPIs também as
  inclui, e um aporte não feito é dinheiro que ainda precisa sair. Numa
  [[Meta|meta]], "pago" lê-se "aporte registrado".
- **Atraso é `data de vencimento < hoje`** — comparação com a data real, não com
  o mês do filtro. Vale igual para mês passado (tudo que ficou pendente aparece
  atrasado), corrente e futuro (nada atrasado). Nenhum caso especial por mês.
- **`Math.min(diaVencimento, últimoDiaDoMês)`.** Vencimento 31 em fevereiro
  precisa cair no dia 28/29; sem o clamp o `Date` rolaria para março e a conta
  apareceria como "a vencer" num mês que já acabou.
- **Sem `diaVencimento` (legado, importação) nunca é atraso.** Não dá para
  afirmar atraso sem data: a linha vai para o fim da lista com "—" no dia e badge
  neutro "Sem data".
- **Ordenação é só por dia.** Dentro de um mesmo mês, dia menor já significa mais
  urgente — os atrasados sobem sozinhos, sem regra extra.
- **"A vencer" usa um neutro**, não a paleta de `STATUS` nem uma cor de bucket:
  não pagou ainda não é bom nem ruim, é o estado normal. Verde e vermelho ficam
  reservados a pago/atrasado, e **sempre** com marcador e rótulo ao lado (✓, ⚠) —
  cor sozinha não identifica nada para quem não distingue as duas.
- **Barra empilhada com 2px de folga** entre segmentos: encostados, verde e
  vermelho viram uma faixa só na leitura periférica.

### Estados

| Situação | O que aparece |
|---|---|
| Nada pendente | "✓ Tudo quitado neste mês — nada pendente" |
| Nenhum lançamento no mês | Estado vazio com link para Finanças |

---

## Gastos por tag — a seção com a pegadinha

Ranking horizontal do comprometido por [[Tag|tag]] no mês, somando **todos os
tipos**.

Como um lançamento pode ter **várias** tags, **o valor conta em cada uma** —
então **a soma do ranking pode passar do total comprometido**. Isso é esperado
e está avisado no subtítulo. Não "conserte" dividindo o valor entre as tags:
isso responderia uma pergunta diferente (e menos útil).

- Sem tag → bucket neutro **"Sem tag"** (`#7a7a85`, nunca colide com tag real)
- Barras escaladas pela maior fatia
- Cores vêm da própria tag
- Sem lançamento tagueado no mês → estado vazio com link para Finanças

---

## Enquanto carrega

`loading` é o **ou** de cinco fontes (salário + os quatro `useAccounts`): a tela
só aparece com tudo pronto, senão os medidores nasceriam com denominador errado
e "corrigiriam" sozinhos na frente do usuário.

Nesse meio-tempo, o [[Loader]] **sozinho, sem card nem texto em volta**, centrado
na sobra da tela pelo `LoaderArea` (`flex: 1` + `place-items: center`). O
`min-height: 50vh` é para o mobile, onde o `Content` tem altura automática e não
há sobra para esticar — sem ele o loader nasceria colado no cabeçalho.

---

## Sem salário configurado

Estado de primeira classe, não erro. Ver [[Salario]].

- Aviso (`InlineNotice`) com link para [[Tela-Settings|/Settings]] (era `/dados`,
  antes de a tela de Dados ser absorvida por Configurações)
- **Medidores ficam sem limite** — não há denominador
- **Donut continua funcionando** — proporção entre buckets não depende do salário

---

## O que a tela NÃO faz

- **Não compara meses.** Não há série temporal, tendência ou "vs. mês anterior".
- **Não projeta.** Nada de previsão de saldo futuro.
- **Não permite editar** lançamento — só criar (⚡). Editar é nas telas de detalhe.
  A seção de quitação **mostra** o que vence, mas não deixa marcar como pago:
  isso é ação da tela da finança (o toggle pede confirmação).
- **Não usa o salário histórico.** Usa o salário **atual** para qualquer mês
  consultado. Ver [[Salario]].
- **Não filtra por tag.** A seção de tags é leitura, não filtro.

---

## Relacionadas
[[Tipo]] · [[Salario]] · [[Tag]] · [[QuickAddModal]] · [[Camada-de-Dados]] · [[ADR-005-Filtro-de-periodo-no-frontend]]
