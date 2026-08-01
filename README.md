---
aliases: ["Leia-me", "Como usar o vault"]
tags: [meta]
atualizado: 2026-08-01
---

# Vault do Financial Control

Este vault é a **fonte de verdade sobre o sistema** — não sobre o código.
O código responde *o quê*; o vault responde **o porquê, o para quem e o que não pode**.

Ele existe para uma coisa: **decidir melhor antes de implementar**. Antes de
refatorar, criar feature ou mudar regra, a resposta deve estar aqui — e se não
estiver, ela passa a estar depois.

> Comece por [[Mapa-do-Sistema|🗺️ Mapa do Sistema]].

---

## Para quem é cada coisa

| Quem | Usa o vault para |
|---|---|
| **Gui** | Lembrar por que uma regra é como é; decidir o que construir; validar se uma ideia nova conflita com o modelo atual |
| **Claude** | Ler as notas relevantes **antes** de escrever código; não reinventar decisão já tomada; não quebrar regra invisível no código |

---

## Regras de manutenção

Um vault desatualizado é pior que nenhum vault — ele mente com autoridade.
Por isso:

1. **Mudou regra de negócio → atualiza a nota de domínio**, no mesmo trabalho.
   Não é tarefa separada, é parte do "pronto".
2. **Decisão com trade-off → vira ADR** em `05-Decisoes`. Se daqui a 6 meses
   alguém puder perguntar "por que não fizeram do jeito óbvio?", precisa de ADR.
3. **Nota que descreve código** cita o arquivo (`src/utils/recurrence.js`), mas
   **não cola o código**. Código muda; a intenção dura mais.
4. **Toda nota diz o que a coisa NÃO faz.** É a metade mais esquecida e a que
   mais evita retrabalho.
5. **Descobriu na prática (bug real, uso real) → registra.** O aprendizado caro
   é o que mais vale documentar.

---

## Convenções

**Nomes de arquivo em ASCII**, sem acento nem espaço (`Recorrencia.md`,
`Competencia-e-Pagamento.md`). O título de exibição vem do `aliases` no
frontmatter e do `# H1`. Isso evita problema de encoding no Windows e no Git,
sem perder a grafia correta na leitura.

**Frontmatter padrão:**

```yaml
---
aliases: ["Nome Acentuado", "Sinônimo"]
tags: [dominio]        # dominio | tela | componente | backend | adr | guia | limite
codigo: ["src/utils/recurrence.js"]   # arquivos que implementam isto
atualizado: 2026-08-01
---
```

**Tags de tipo de nota:**

| Tag | Significa |
|---|---|
| `dominio` | Conceito de negócio — vale independente de tecnologia |
| `tela` | O que o usuário vê e consegue fazer |
| `componente` | Peça de UI reutilizável |
| `backend` | API, modelo de dados, migrações |
| `adr` | Decisão registrada, com alternativas e consequências |
| `guia` | "Quero fazer X — por onde começo" |
| `limite` | O que o sistema deliberadamente não faz |

**Links:** use `[[Arquivo|Texto exibido]]`. Linkar liberalmente é bom — um link
para nota que ainda não existe marca um buraco a preencher, não um erro.

---

## Estrutura

```
Vault/
├── Mapa-do-Sistema.md      ← hub central, comece aqui
├── 01-Dominio/             ← regra de negócio e vocabulário
├── 02-Telas/               ← o que cada tela entrega
├── 03-Componentes/         ← peças de UI + camada de dados do front
├── 04-Backend/             ← API, modelo, migrações, auth
├── 05-Decisoes/            ← ADRs (por que é assim)
├── 06-Guias/               ← como mexer sem quebrar
└── 07-Limites/             ← o que o sistema não faz + pendências
```

---

## Abrindo no Obsidian

`Abrir pasta como cofre` → aponte para `Projects/FinancialControll/Vault`.

Plugins que valem a pena aqui (nenhum é obrigatório):

- **Dataview** — listas automáticas (ex.: "todas as notas com `tags: limite`").
- **Graph view** (nativo) — enxergar o acoplamento entre conceitos. Um nó de
  domínio muito conectado é um bom candidato a merecer cuidado extra em refactor.

---

## O que este vault NÃO é

- **Não é changelog.** Histórico é do Git.
- **Não é cópia do `CLAUDE.md`.** O `CLAUDE.md` é o cartão de referência rápida
  (comandos, convenções, armadilhas); o vault é a profundidade. Quando os dois
  divergirem, **o vault vence** e o `CLAUDE.md` é corrigido.
- **Não é documentação de API gerada.** Contratos ficam aqui só quando a forma
  do dado carrega uma decisão de negócio.
