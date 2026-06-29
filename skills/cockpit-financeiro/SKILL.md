---
name: cockpit-financeiro
description: >
  Arquitetura e operação do cockpit financeiro da DevOZ (consolida DevOZ LLC/Xero e DevOZ
  LTDA/Superlógica em USD; Supabase + Metabase; pipeline diário no GitHub Actions). Use
  SEMPRE que envolver o projeto cockpit: o fato fct_faturamento, views/métricas SaaS (MRR,
  churn, NRR, clientes ativos), pipeline/ETL, Supabase, Metabase, ou como estender o painel
  — mesmo sem citar "cockpit", se mexer em receita consolidada BR+US, MRR, ou nas views/cards.
metadata:
  type: reference
---

# Cockpit Financeiro DevOZ — visão geral

Consolida **DevOZ LLC** (EUA, Xero, USD) e **DevOZ LTDA** (BR, Superlógica, BRL) numa
visão única **em USD**, com séries mensais, atualização diária e dashboard online na nuvem.
Para detalhes de cada fonte, ver as skills [[xero-cockpit]], [[superlogica-cockpit]],
[[zoho-crm-cockpit]].

## Princípios (não violar)
- **NÃO usar campos pré-calculados** das fontes (nem `mrr` do Superlógica, nem relatórios
  agregados do Xero). Puxar o dado mais bruto (itens de fatura), normalizar e **calcular as
  métricas nós mesmos** nas views.
- **Âncora da série = data de VENCIMENTO** (guardar também fatura/competência).
- **Câmbio = média mensal BRL→USD** (BCB SGS série 1; `usd = brl / taxa_média_mês`).
- **Idempotência** por `unique(fonte, id_externo)` — reprocessar nunca duplica.
- Segredos só em `.env`/GitHub Secrets; nunca no chat. PG* p/ o banco (evita encoding de senha no URI).

## Modelo de dados (Supabase / Postgres, schema `core`)
- **`fct_faturamento`** — fato único DENORMALIZADO, grão = **item de fatura**, sempre em USD.
  Colunas-chave: fonte, id_externo, empresa_id (1=LLC, 2=LTDA), id_cliente_ext, cliente_nome,
  pais, produto_nome, plano_virtual, linha_produto, segmento, tipo_servico,
  tratamento(receita|desconto|financeiro|fora), tipo_receita(recorrente|nao_recorrente),
  mes_competencia (1º dia, =vencimento), data_*, status_pagamento, moeda_origem, valor_origem,
  cambio_aplicado, valor_usd, valor_pago_usd.
- `dim_cambio` (taxa média mensal), `app_config` (token rotativo Xero),
  `dim_cliente_ozid` + `crm_deal_status` (cruzamento CRM — ver skill zoho).
- ⚠️ O fato NÃO guarda ozid (guarda o id da fonte); o ozid vem via `dim_cliente_ozid`.
- Classificação 3D: **linha_produto** (OZmap/OZneutral/OZmatic/OZmob) + **segmento**
  (Básico/Corporativo, só OZmap) + **tipo_servico** (Assinatura/Adicional de capacidade/
  Módulo de software/Setup-Implantação/Serviço pontual/Treinamento/Importação).
  Recorrência DERIVADA do tipo. **MRR = receita recorrente**.

## Views (db/)
- `views_metricas.sql`: vw_receita_mensal[_global], vw_receita_por_plano/pais/linha/segmento/tipo,
  vw_mrr_cliente_mes, **vw_mrr_movimento** (novo/churn/expansão/contração; grão (empresa_id,
  id_cliente_ext)), vw_clientes_ativos (por mês), vw_clientes_por_pais.
- `views_saas.sql`: vw_top_clientes_mrr, vw_mrr_por_pais/linha/segmento, vw_receita_por_pais_linha,
  **vw_kpis_mensais** (MRR, ARR, ARPA, NRR%, GRR% [base = mês-calendário anterior], churn),
  vw_detalhe_faturamento (auditoria por mês; tem plano_classificacao e valor_aberto_usd).
- `views_clientes_ativos_cadencia.sql`: **cliente ativo por CADÊNCIA** — em vez de "faturou no
  mês", infere o intervalo típico (mediana dos gaps) e considera ativo se a última fatura
  (passada OU futura/pré-paga) está dentro de intervalo+1 mês de folga; na zona de folga,
  `crm_is_churn` desempata (sai). Resolve fim de mês + pré-pagos + churn. Esta família NÃO
  corta meses futuros (de propósito); as demais views cortam.
- Convenção: "receita líquida" inclui não-recorrente; "MRR" é só recorrente. Não comparar
  cegamente uma com a outra no painel.

## Pipeline (etl/)
- `run_daily.py` (orquestrador): câmbio → ingest Superlógica/Xero (incremental) → transforms
  → load_supabase → (soft) build_dim_cliente + zoho_ingest. Aceita `--full` (reconciliação,
  sem janelas). Passos "soft" não derrubam a carga se o CRM falhar.
- Automação: GitHub Actions — `daily.yml` (cron 09:00 UTC) e `weekly.yml` (dom 08:00 UTC,
  full p/ corrigir status defasado). `apply-sql.yml` = aplicar .sql sob demanda (manual).
- Conexão Supabase: **Session Pooler** host aws-1-sa-east-1.pooler.supabase.com:5432, user
  `postgres.icpypicdhxvocyruxchl` (com sufixo!), SSL require. RLS ligado (conexão direta ignora).

## Ferramentas de apoio (etl/)
- `apply_sql.py` — aplica arquivos .sql no banco (lê .env, psycopg2). Ex.:
  `python3 etl/apply_sql.py db/views_saas.sql`.
- `setup_cockpit.py` — UM comando: aplica views + cria/atualiza cards no Metabase (idempotente).
- `metabase_apply.py` — API do Metabase (list/show/dump/set-sql). ⚠️ versão nova usa formato
  MBQL: SQL nativo em `dataset_query.stages[0].native`; nativo = `query_type=='native'`.
- ⚠️ O **sandbox do Cowork não tem rede** — DB/Metabase/Zoho só conectam na máquina do
  usuário ou no GitHub Actions. Validar SQL localmente com `pglast` (parser Postgres).

## Como estender (padrão)
1. Editar/adicionar a view em `db/*.sql` (validar com pglast).
2. Aplicar: `python3 etl/apply_sql.py db/<arquivo>.sql` (ou `setup_cockpit.py` p/ views+cards).
3. Card no Metabase via `metabase_apply.py set-sql <id> --file ...` ou `setup_cockpit.py`.
4. Commit + push. Se mexer em `.github/workflows/`, o token precisa de escopo `workflow`.

## Para incrementar esta skill
Acrescente: novas views/métricas, mudanças de schema, pendências do REVISAO.md resolvidas
(materializar views, dim_produto, NRR 12m, coorte), e qualquer decisão de modelagem nova.
