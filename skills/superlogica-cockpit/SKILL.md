---
name: superlogica-cockpit
description: >
  Como integrar e extrair dados de faturamento do Superlógica (DevOZ LTDA, BR) para o
  cockpit financeiro. Use SEMPRE que o assunto envolver Superlógica, faturamento/cobranças
  BR, boletos, assinaturas, ingestão de receita do Brasil, ou o script superlogica_ingest.py
  — mesmo que o usuário não cite "Superlógica" explicitamente, mas fale de cobranças BR,
  composição de boletos, st_sincro/ozid no Superlógica, ou carga incremental do BR.
metadata:
  type: reference
---

# Superlógica → Cockpit (faturamento BR, DevOZ LTDA)

Conhecimento prático para puxar e normalizar faturamento do Superlógica. Empresa
`empresa_id = 2`, fonte `'superlogica'`, moeda BRL (convertida p/ USD no fato).

## API
- Base: `https://api.superlogica.net/v2/financeiro`. Módulo = **Financeiro**.
- Auth por **headers**: `app_token` + `access_token` (a integração usa só esses dois).
  ⚠️ Tokens **expiram em 1 ano** — prever renovação e alertar em 401.
- Gerar tokens: superlogica.net/usuario → role até **Aplicativos** → **Novo App Token**.
- ⚠️ Datas da API são **MM/DD/AAAA** (m/d/Y), tanto em parâmetros (`dtInicio`/`dtFim`/
  `alteradasApos`) quanto em respostas. Decimais com ".".
- Secrets podem vir com `\n` no fim → sempre `.strip()` em tokens/env.

## Endpoints e parâmetros (o que importa)
- `/clientes` — campos: `id_sacado_sac` (id do cliente, = `id_cliente_ext` no fato),
  `st_nome_sac`, `st_cgc_sac` (CNPJ), `st_estado_sac`, **`st_sincro_sac` = ozid (UUID)**
  que cruza com o CRM/Zoho. País sempre BR.
- `/assinaturas` — `st_nome_pla` (produto), `st_nome_gpl` (grupo/plano nativo),
  `dt_contrato_plc`, `dt_cancelamento_plc`, `st_sincro_sac`/`_sac1`.
- `/cobranca` — a fonte da receita. Params essenciais:
  - **`exibirComposicaoDosBoletos=1`** → retorna os ITENS de cada boleto em
    `compo_recebimento` (um boleto agrupa mensalidade + setup + adicionais).
  - `filtrarpor=vencimento` (âncora oficial da série), `dtInicio`/`dtFim` (m/d/Y).
  - `status` (liquidadas/pendentes/canceladas/protestadas/remetidascartorio — iterar p/
    histórico total), `alteradasApos`/`alteradasAte` (carga incremental), `apenasColunasPrincipais=0`.
  - Paginação: `pagina` / `itensPorPagina` (**máx 200**).
- Campos de cobrança: `id_recebimento_recb` (id fatura), `id_sacado_sac` (cliente),
  `vl_total_recb`, `dt_vencimento_recb` (âncora), `dt_recebimento_recb` (pago),
  `fl_status_recb`. Itens: `nm_quantidade_comp`, `st_valor_comp`, `st_descricao_comp`, `id_composicao_comp`.

## Regras de transformação (transform_superlogica.py)
- **Grão = ITEM da cobrança** (não o boleto). `id_externo` = `id_composicao_comp`.
- ⚠️ **`st_valor_comp` é UNITÁRIO**: valor da linha = `nm_quantidade_comp × st_valor_comp`
  (bug histórico: linhas "Portas ×N" ficavam subcontadas).
- Classificação por `id_produto` via `db/seed_classificacao_br.csv`
  (colunas: id_produto, nome_original, linha_produto, segmento, tipo_servico, tratamento).
- `tratamento` ∈ receita/desconto/financeiro/fora; `tipo_receita` derivado do tipo de
  serviço (Assinatura+Adicional+Módulo = recorrente). MRR = receita recorrente.
- Casos especiais: id 106 "OZmap Outro" classificado POR CLIENTE (EAF = receita; DevOZ LLC
  / "Teste" = fora/intercompany). "Adesão-inativo" = Setup antigo (receita). "Mensalidade"
  id 999999982 = planos corporativos antigos (receita Corporativo).
- Câmbio: usd = brl / taxa_média_do_mês (ver skill cockpit-financeiro). Mês futuro = câmbio pendente → valor_usd nulo.

## Gotchas
- Para histórico completo, iterar todos os `status` (uma chamada por status).
- `--full` no `superlogica_ingest.py` = todos os status sem limite; `--since` adiciona
  `alteradasApos` (incremental diário). Em dia sem alterações o transform mantém o CSV anterior.
- Validação: MRR recorrente BR deve ficar ~1% do painel "MRR" do Superlógica (que é run-rate;
  o nosso é "faturado no mês por vencimento" — pequena diferença é esperada e aceita).

## Para incrementar esta skill
Acrescente aqui: novos `status` úteis, mudanças de campos da API, novas regras de
classificação por cliente/produto, e qualquer endpoint novo que passarmos a usar.

---

## Código-fonte (canônico)

O código executável desta skill **não** mora aqui — vive no repositório da aplicação do cockpit, com pipeline próprio (GitHub Actions). Esta skill é o ponteiro de "como rodar".

**Repo:** [https://github.com/rodrigo-kautzmann/cockpit-devoz](https://github.com/rodrigo-kautzmann/cockpit-devoz)

**Arquivos principais:**
- `etl/superlogica_ingest.py`, `etl/transform_superlogica.py`
- `db/seed_classificacao_br.csv`, `db/seed_plano_virtual.csv`
