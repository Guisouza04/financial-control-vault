---
aliases: ["Endpoints", "API", "Contratos"]
tags: [backend]
codigo: ["FinancialControllBackend/app/api/financas.py", "FinancialControllBackend/app/api/security.py"]
atualizado: 2026-08-01
---

# Endpoints

Todas as rotas sob o prefixo **`/api`**. Base do front:
`http://localhost:8000/api`.

---

## `/api/security/*`

| Método | Rota | Payload | Auth |
|---|---|---|---|
| `POST` | `/signup` | `{ CPF, email, password, confirmPassword }` | ❌ |
| `POST` | `/login` | `{ username, password }` — **CPF ou e-mail** | ❌ |
| `PUT` | `/change-password` | `{ oldPassword, newPassword }` | ✅ |
| `PUT` | `/profile` | `{ nome?, apelido?, email? }` | ✅ |

> **Estes usam `camelCase`** (`confirmPassword`, `oldPassword`) e `CPF`
> maiúsculo — diferente do `snake_case` de `/financas`. Inconsistente, mas é o
> contrato atual.

**`/profile`** só grava os campos **não-nulos** — omitir preserva o valor atual.
Não valida e-mail duplicado (ver [[Modelo-de-Dados]]).

---

## `/api/financas/*` — lançamentos

| Método | Rota | Observação |
|---|---|---|
| `GET` | `/contas` | [[Tipo]] 1 |
| `GET` | `/investimentos` | [[Tipo]] 2 |
| `GET` | `/opcionais` | [[Tipo]] 3 |
| `GET` | `/metas` | [[Tipo]] 4 |
| `POST` | `/create` | |
| `PUT` | `/update/{id}` | |
| `DELETE` | `/delete/{id}` | `204` |
| `PUT` | `/payment-status/{id}` | Status **por competência** |

### Os `GET` devolvem TUDO do tipo

**Não filtram por período.** O front recebe todos os lançamentos do usuário
para aquele tipo e faz o recorte client-side.

Antes filtravam por `dt_create` — o que **escondia contas recorrentes fora do
mês de criação**. Ver [[ADR-005-Filtro-de-periodo-no-frontend]].

### Payload de create/update

```json
{
  "de_conta": "string",
  "vl_conta": "1234.56",
  "tipo": 1,
  "recorrencia": "UNICA" | "MENSAL" | "ANUAL",
  "dia_vencimento": 10,
  "data_inicio": "2026-07-10",
  "data_fim": "2026-12-10",
  "qtd_parcelas": 6,
  "tag_ids": [3, 7]
}
```

- **`vl_conta` como string** com ponto decimal
- **`tag_ids` substitui** o conjunto inteiro (não faz merge). Ver [[Tag]]

### Resposta

```json
{
  "id": 1, "de_conta": "Nome", "vl_conta": "150.00", "qtd_parcelas": 12,
  "tipo": 1, "conta_paga": "S", "dt_create": "2025-01-15T…",
  "pagamentos": ["2026-08", "2026-10"],
  "tags": [{ "id": 3, "nome": "Mercado", "cor": "#199e70" }]
}
```

### `payment-status`

```json
{ "competencia": "2026-08", "conta_paga": "S" | "N" }
```

`competencia` é validada por regex `\d{4}-\d{2}` no schema — formato errado dá
**422**. Ver [[Competencia-e-Pagamento]].

---

## `/api/financas/tags`

| Método | Rota | |
|---|---|---|
| `GET` | `/tags` | |
| `POST` | `/tags` | `201` |
| `PUT` | `/tags/{id}` | |
| `DELETE` | `/tags/{id}` | `204` — global |

Nome único por usuário (case-insensitive).

---

## `/api/financas/salary`

| Método | Rota | |
|---|---|---|
| `GET` | `/salary` | `{ salario: "8000.00" \| null, periodo_pagamento }` |
| `POST` | `/salary` | `{ salario, periodo_pagamento }` |

> ⚠️ **Ambos obrigatórios no POST.** `SalaryIn` não tem default — mandar só
> `salario` dá **422**. Ver [[Salario]].

`salario` é serializado como **string** com 2 casas na resposta.

---

## `/api/financas/import/*`

| Método | Rota | |
|---|---|---|
| `POST` | `/import/preview` | Faz parse, **não grava** |
| `POST` | `/import/commit` | Grava; devolve `{ created, skipped }` |

O conteúdo do `.ofx` vai como **texto no corpo** (`{ content: "..." }`), não
multipart — para não exigir `python-multipart`.

Ver [[Tela-Importar-Extrato]].

---

## Formato de erro — padronizado

`main.py` normaliza **toda** `HTTPException` para:

```json
{ "error": "mensagem" }
```

E erros de validação (422):

```json
{ "error": "Dados inválidos", "details": [ … ] }
```

> **Mantenha esse formato ao criar endpoints.** O front lê
> `err.response?.data?.error` — qualquer outro shape cai na mensagem genérica.

---

## CORS

```python
allow_origins=[settings.CORS_ORIGIN]   # UMA origem só
allow_credentials=True
```

Default: `http://localhost:5173/`.

> ⚠️ **Uma origem apenas, vinda do `.env`.** Se o Vite subir em outra porta
> (5174, quando a 5173 está ocupada), **toda chamada falha como "Network Error"**
> no axios — é o CORS barrando, não o backend fora do ar. Ver [[Ambiente-de-Dev]].

---

## Relacionadas
[[Modelo-de-Dados]] · [[Autenticacao]] · [[Camada-de-Dados]] · [[Ambiente-de-Dev]]
