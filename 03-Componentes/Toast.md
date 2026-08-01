---
aliases: ["Toast", "Notificações"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/Toast/index.jsx", "FinancialControll2.0/src/context/toast.js"]
atualizado: 2026-08-01
---

# Toast

**`src/components/Toast`** + **`src/context/toast.js`**

Notificações globais. O `ToastProvider` é montado no `main.jsx` e renderiza a
fila; **as telas só consomem**:

```js
const toast = useToast();
toast.success("Salvo!");
toast.error("Deu ruim");
toast.warning("Preencha o campo");
toast.info("...");
toast.show(tipo, msg, { duration });   // duration: 0 = fixo, sem barra
toast.dismiss(id);
```

Duração padrão: **4s**.

> **Context e hook vivem em arquivo separado** (`src/context/toast.js`) do
> componente. Isso é deliberado: exportar hook e componente do mesmo arquivo
> quebra o **Fast Refresh** do Vite. Mesmo padrão em [[ConfirmDialog]].

---

## As três regras são do provider, não das telas

Nenhuma tela precisa implementar nada disso:

### 1 · Barra de tempo
`ToastProgress` encolhe de 100% a 0 durante a duração, na cor do tipo.
Usa `scaleX` linear — **roda no compositor**, sem reflow por frame.

### 2 · Dedupe com reset
Toasts são indexados por `tipo + mensagem` (`dedupeKey`).

Reacionar a **mesma** condição:
- reaproveita o toast já visível
- reinicia o timer
- remonta a barra (via `key={resetKey}`)
- **cancela a saída** se ele já estava saindo

Mensagens **diferentes** continuam empilhando.

> Os índices (`idByKey`/`keyById`) são **refs, não state** — `show` precisa
> consultá-los de forma síncrona. Trocar por state reintroduz o empilhamento
> de cópias.

### 3 · Saída animada
`slideOut` = `slideIn` invertido, mesma curva e duração (`TRANSITION_MS`,
exportado de `styles.js` e compartilhado com o provider).
`dismiss` marca `leaving` e só remove do DOM quando a animação termina.

---

## ⚠️ Ao testar animação no navegador

Aba em **segundo plano** (`document.visibilityState === "hidden"`) congela
animações CSS e estrangula timers. A barra aparece travada em `scaleX(1)` e as
medições de tempo saem distorcidas.

**Meça com a aba visível.** Já custou tempo de debug.

---

## Relacionadas
[[ConfirmDialog]] · [[Camada-de-Dados]] · [[Convencoes]]
