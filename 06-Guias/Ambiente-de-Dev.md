---
aliases: ["Ambiente de Dev", "Como rodar", "Setup"]
tags: [guia]
atualizado: 2026-08-01
---

# Ambiente de Dev

Dois repositórios, um projeto:

```
Projects/FinancialControll/
├── FinancialControll2.0/       ← frontend (React + Vite)
├── FinancialControllBackend/   ← backend (FastAPI + Postgres)
└── Vault/                      ← este vault
```

---

## Backend

```bash
cd FinancialControllBackend
cp .env.example .env      # se ainda não existir
docker compose up -d
```

Sobe dois containers:

| Container | Porta | |
|---|---|---|
| `financial_postgres` | 5432 | Postgres 16-alpine, volume `financial_pgdata` |
| `financial_api` | **8000** | FastAPI |

> **O `alembic upgrade head` roda no start do container** (está no `command`
> do compose). Não é preciso migrar à mão.

A API espera o Postgres ficar *healthy* antes de subir (`depends_on` +
`healthcheck`).

**Docs interativas:** `http://localhost:8000/docs`
**Health:** `GET http://localhost:8000/` → `{ "status": "ok" }`

---

## Frontend

```bash
cd FinancialControll2.0
npm install
npm run dev      # http://localhost:5173
```

| Comando | |
|---|---|
| `npm run dev` | Vite dev server |
| `npm run build` | Build de produção |
| `npm run preview` | Serve o build |
| `npm run lint` | ESLint (React Hooks + React Refresh) |

> **Não há framework de teste configurado.** Só ESLint. Verificação é manual,
> no navegador.

---

## ⚠️ A armadilha número 1: porta do Vite e CORS

O backend libera **uma única origem** (`CORS_ORIGIN`, default
`http://localhost:5173`).

**Se a 5173 estiver ocupada, o Vite sobe na 5174** — e aí **toda** chamada de
API falha como **"Network Error"** no axios.

> **Isso parece backend fora do ar, mas é CORS barrando.** Antes de investigar
> o backend:
> 1. Confira em que porta o Vite subiu (aparece no terminal)
> 2. Se não for 5173, mate o processo que ocupa a porta **ou** ajuste
>    `CORS_ORIGIN` no `.env` e recrie o container

Ver [[Endpoints]].

---

## Outras armadilhas

**`baseURL` hardcoded.** `src/services/api.js` aponta para
`http://localhost:8000/api` **no código** — não há variável de ambiente.
Mudar a porta do backend exige editar o arquivo.

**Segredo JWT default.** Sem `.env`, sobe com
`"troque-este-segredo-em-producao"`. Ver [[Autenticacao]].

**Testar animação de [[Toast]] com a aba visível.** Aba em segundo plano
congela animações CSS e estrangula timers.

---

## Antes de um release do backend

🔒 Rode a skill **`/verify-requirements`**. Ela varre o `requirements.txt`
procurando incompatibilidades conhecidas.

**Só prossiga se retornar OK.** Existe por causa do caso `bcrypt 4.x` +
`passlib 1.7.4`, que quebrou o login silenciosamente.
**Mantenha `bcrypt==3.2.2` pinado enquanto o `passlib` for `1.7.4`.**

---

## Dados de teste

- `FinancialControllBackend/seed.py`
- `FinancialControllBackend/samples/exemplo_fatura.ofx` — para a
  [[Tela-Importar-Extrato|importação]]

---

## Relacionadas
[[Endpoints]] · [[Autenticacao]] · [[Migracoes]] · [[Convencoes]]
