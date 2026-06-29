---
name: zoho-crm-cockpit
description: >
  Como cruzar o Zoho CRM (Deals/Potentials) com o faturamento do cockpit: reconciliação
  de clientes ativos, churn, país de operação. Use SEMPRE que envolver Zoho, CRM, deals,
  stage Rodando/Churn, campo ozid/domain, COQL, reconciliação CRM × faturamento, ou os
  scripts zoho_ingest.py/zoho_token.py — mesmo sem citar "Zoho", se falar de clientes
  "rodando", churn confirmado pelo CRM, ou cruzar deals com Xero/Superlógica por ozid.
metadata:
  type: reference
---

# Zoho CRM → Cockpit (reconciliação e churn)

O CRM é usado para **desempatar** e **validar** o que o faturamento mostra (não substitui).
Conector MCP do Zoho existe no chat; o pipeline (GitHub Actions) usa a **API direta** via
OAuth (não tem o MCP).

## Modelo de dados (módulo Deals / Potentials)
- **`ozid`** (UUID) = chave que cruza com faturamento: Superlógica `st_sincro_sac` e
  Xero `AccountNumber`. ⭐ Cruzar SEMPRE por ozid; fallback `domain` (CRM) = slug do Xero
  (texto entre parênteses no nome do cliente). **NUNCA cruzar por nome do deal** — nomes
  divergem (ex.: CRM "4net" = nosso "4everplug C.A (cuatronet)"), inflam divergências ~3x.
- **`Stage`**: ativo = **"Rodando"**; churn = **"Churn"**. Outros: Fechado Ganho/Perdido,
  Inativo, Negociando, etc. Fora do BR, ativo é praticamente tudo "Rodando".
- **`Country_Code`** = país da empresa/faturação; **`Country_Operation`** = país onde a rede
  opera (o que vale para "clientes por país"). Hoje Country_Operation está quase todo vazio.
  CRM usa ISO-3 (BRA/VEN/COL); o dashboard usa ISO-2 → precisa de-para.
- O plano no CRM NÃO é confiável — confiar em ozid + Stage.

## COQL (API) — limites e armadilhas
- Toda query precisa de um `WHERE` (senão "missing clause: where").
- `in (...)` aceita no **máx 100** valores. Agregações/`group by count(id)` NÃO funcionam
  bem pelo conector — para distintos de Stage use `select Stage ... group by Stage`.
- Paginação por `limit offset,linhas` (máx ~200/página); iterar enquanto `more_records`.
- `getFields` é gigante (vai p/ arquivo temp; usar Grep). Saídas grandes truncam no chat.

## OAuth (pipeline)
- **Self Client** em api-console.zoho.com → scopes
  `ZohoCRM.modules.deals.READ,ZohoCRM.coql.READ` → gera **code** (curto) → trocar por
  **refresh token** (não expira). DC `.com` → `accounts.zoho.com` / `www.zohoapis.com`.
- Helper `etl/zoho_token.py` (lê self_client.json do Downloads, troca e grava no .env sem
  expor segredo). Guia completo: `SETUP_ZOHO.md`. Header de chamada:
  `Authorization: Zoho-oauthtoken <access_token>`.
- Secrets: `ZOHO_CLIENT_ID/SECRET/REFRESH_TOKEN` (+ ACCOUNTS_URL/API_URL se não for `.com`).

## Como o cockpit usa (churn canônico)
- `etl/zoho_ingest.py` puxa `ozid, Stage, Deal_Name` de todos os deals com ozid e regrava
  `core.crm_deal_status` (truncate+insert transacional). Calcula **`is_churn`** na ingestão
  via `CHURN_STAGES = {"churn"}` (normaliza minúsc/trim) — ÚNICO lugar p/ ajustar a regra.
- `core.dim_cliente_ozid` mapeia (empresa_id, id_cliente_ext) → ozid:
  BR id_sacado_sac→st_sincro_sac; LLC ContactID→AccountNumber. Populado por
  `etl/build_dim_cliente.py` (lê _amostras dos ingests).
- A view `vw_clientes_ativos_cadencia` faz o **desempate**: na zona de folga (dúvida de
  churn), se `crm_is_churn` → cliente sai dos ativos; quem fatura no ritmo permanece.
- Ambos os passos são **não-críticos** no pipeline (se o Zoho falhar, a carga financeira segue).

## Para incrementar esta skill
Acrescente: novos stages que signifiquem churn (em CHURN_STAGES), uso de Country_Operation
quando preenchido, novos campos do deal, e padrões de cruzamento que descobrirmos.

---

## Código-fonte (canônico)

O código executável desta skill **não** mora aqui — vive no repositório da aplicação do cockpit, com pipeline próprio (GitHub Actions). Esta skill é o ponteiro de "como rodar".

**Repo:** [https://github.com/rodrigo-kautzmann/cockpit-devoz](https://github.com/rodrigo-kautzmann/cockpit-devoz)

**Arquivos principais:**
- `etl/zoho_ingest.py`, `etl/zoho_token.py`, `etl/build_dim_cliente.py`
