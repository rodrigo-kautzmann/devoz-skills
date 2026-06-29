---
name: xero-cockpit
description: >
  Como integrar e extrair faturamento do Xero (DevOZ LLC, EUA, USD) para o cockpit
  financeiro. Use SEMPRE que envolver Xero, faturas/invoices da LLC, OAuth do Xero,
  line items, contatos/AccountNumber, status de fatura (AUTHORISED/PAID/VOIDED), país
  de cliente LatAm, ou os scripts xero_auth.py/xero_ingest.py — mesmo sem citar "Xero"
  explicitamente, se falar de faturamento US/LLC, ozid no AccountNumber, ou reconciliação Xero.
metadata:
  type: reference
---

# Xero → Cockpit (faturamento LLC, DevOZ LLC, USD)

Empresa `empresa_id = 1`, fonte `'xero'`, moeda USD nativa (cambio_aplicado = 1.0).
Fonte de verdade da LLC = Xero (Chargebee fica de fora). Clientes majoritariamente LatAm.

## OAuth (Accounting API)
- App **Web** em developer.xero.com. **Scopes GRANULARES** (apps pós-mar/2026):
  `accounting.invoices.read accounting.contacts.read accounting.settings.read offline_access`.
  ⚠️ NÃO usar o amplo `accounting.transactions.read` (dá `invalid_scope`).
  No parâmetro `scope`, separar com **%20** (não "+").
- O **refresh token ROTACIONA a cada uso**. Persistir o novo em `core.app_config`
  (chave do token) — `xero_ingest.py` lê/grava de lá (suporta PG*/DATABASE_URL).
- `xero_auth.py` = OAuth interativo único (gera o primeiro refresh token).
- Secrets necessários: `XERO_CLIENT_ID`, `XERO_CLIENT_SECRET`, `XERO_TENANT_ID`.
  ⚠️ `invalid_grant` = bootstrap token já consumido; refresh token local não vale após o bootstrap.
- Sempre `.strip()` em tokens/env (vêm com `\n`). Retry em 429/timeout (`_open` já faz).

## Dados (transform_xero.py)
- **Grão = LINE ITEM** da fatura. `id_externo` = `LineItemID` (fallback `InvoiceID:idx` —
  evitar, pode duplicar se a ordem mudar; preferir sempre LineItemID).
- Datas no formato **`/Date(ms+offset)/`** → parsear o epoch ms. Âncora do mês = **DueDate**
  (vencimento), igual ao BR.
- Status válidos: **AUTHORISED / PAID**. DRAFT é ignorado.
- ⚠️ **VOIDED / DELETED**: NÃO descartar silenciosamente — emitir linha **neutra**
  (tratamento='fora', status='cancelado', valor 0) com o MESMO `id_externo`, p/ o upsert
  sobrescrever a linha ativa antiga (senão o status fica defasado pra sempre fora da janela).
  Const `ANULADAS = {VOIDED, DELETED}`.
- `LineAmount` já é líquido de desconto de linha → valor_origem = valor_usd.
- Classificação por **AccountCode** via `db/seed_classificacao_xero.csv`. Conta **4000**
  (balde legado 2021-25) classifica POR DESCRIÇÃO (`classify_4000`). Contas legadas:
  4025/4022/4715→OZmob, etc.
- **País** vem do **cadastro do contato** (endereço), não da entidade. Enriquecer via
  `/Contacts` (`xero_ingest.py --contacts`); usar `includeArchived`. Override manual em
  `db/pais_override.csv` quando o contato não tem país.

## Chaves de cruzamento (CRM/Zoho)
- ⚠️ **`AccountNumber` do contato Xero = ozid (UUID)** — chave que cruza com o CRM.
  Cobre ~71% dos clientes LLC ativos; os 29% sem AccountNumber = higiene de cadastro.
- `id_cliente_ext` no fato = **ContactID** (UUID), que é DIFERENTE do AccountNumber/ozid.
  Por isso existe `core.dim_cliente_ozid` mapeando ContactID→AccountNumber (ver skill zoho).

## Ingestão
- `xero_ingest.py` — `get_invoices` ACCREC order Date DESC; `--modified-since` (incremental
  diário) ou full (sem flag, reconciliação semanal); `--contacts`; `--accounts` (nomes de conta).
- Validação: total LLC bate com o **P&L do Xero** (~0,9%).

## Para incrementar esta skill
Acrescente: novas contas/AccountCodes e suas classificações, mudanças de scope/OAuth,
novos campos de fatura usados, e ajustes na regra de país.
