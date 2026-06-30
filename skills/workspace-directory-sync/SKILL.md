---
name: workspace-directory-sync
description: >
  Sincroniza o Diretório (Quem é Quem) da intranet OZmap a partir do Google Workspace
  Admin Directory. Use quando o assunto for atualizar/automatizar o diretório de pessoas,
  popular "quem é quem", refletir entradas/saídas/mudanças de cargo na intranet. Roda
  diariamente via GitHub Actions. O código vive no repo devoz-automations.
---

# workspace-directory-sync

Lê o **Google Workspace Admin Directory**, normaliza `Department -> área`, resolve manager
e **reescreve a tabela de identidade** da página Diretório no Confluence. Roda diário.

## Princípios
- A tabela gerada cobre só **identidade** (pessoa, cargo, pilar, área, reporta a).
- **Não** toca em "dono de quê / backup" — isso vive nos perfis individuais.
- Domínio oficial: **@ozmap.com**. A página é uma **view gerada** — não editar à mão.

## Código-fonte (canônico)
Esta skill é só a instrução. O **código + agendamento + secrets** moram em:
**https://github.com/rodrigo-kautzmann/devoz-automations**
- `scripts/sync_directory.py`, `scripts/area_map.json` (mapa Department→área, editável por People)
- `.github/workflows/directory-sync.yml` (cron diário + dry-run sob demanda)
- `docs/SETUP-directory-sync.md` (service account, token, secrets)

## Como rodar / setup
Ver o `devoz-automations`. Resumo: secrets no GitHub, depois Actions → directory-sync →
Run workflow (com dry_run pra simular).

## Página de contexto (intranet)
Governança "Como gerenciamos dados de pessoas"; o Diretório é alimentado por esta automação.
