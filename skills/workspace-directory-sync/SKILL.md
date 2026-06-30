---
name: workspace-directory-sync
description: >
  Sincroniza o Diretório (Quem é Quem) da intranet OZmap a partir do Google Workspace
  Admin Directory. Use quando o assunto for atualizar/automatizar o diretório de pessoas,
  popular "quem é quem", refletir entradas/saídas/mudanças de cargo na intranet, ou o
  script sync_directory.py. Roda diariamente via GitHub Actions.
---

# workspace-directory-sync

Skill autossuficiente: lê o **Google Workspace Admin Directory**, normaliza
`Department -> área` (via `scripts/area_map.json`), resolve manager (e-mail->nome) e
**reescreve a tabela de identidade** da página Diretório no Confluence.

## Quando usar
- Atualizar o Diretório da intranet com o estado atual das pessoas.
- Investigar/ajustar o mapeamento Department->área.
- Rodar manualmente uma sincronização (ou em modo simulação).

## Princípios
- A tabela gerada cobre só **identidade** (pessoa, cargo, pilar, área, reporta a).
- **Não** toca em "dono de quê / backup" — isso vive nos perfis individuais.
- Domínio oficial de identidade: **@ozmap.com**.
- A página é uma **view gerada** — ninguém edita a tabela à mão.

## Como rodar
```
pip install -r scripts/requirements.txt
# simulação (não grava):
DRY_RUN=1 python scripts/sync_directory.py
# valendo:
python scripts/sync_directory.py
```
Variáveis de ambiente e provisionamento: ver `SETUP.md`.

## Agendamento
GitHub Actions: `.github/workflows/directory-sync.yml` — diário 05:00 BRT, com
`workflow_dispatch` (inclui opção dry-run) para rodar sob demanda.

## Código-fonte (canônico)
Esta skill é autossuficiente — o código vive aqui em `scripts/`:
- `sync_directory.py` — leitura + transformação + escrita no Confluence
- `area_map.json` — mapeamento Department->área (editável por People)
- `requirements.txt`

## Página de contexto (intranet)
Governança: "Como gerenciamos dados de pessoas". O Diretório é alimentado por esta skill.
