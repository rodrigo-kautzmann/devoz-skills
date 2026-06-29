# devoz-skills

Repositório central de **skills e automações da DevOZ** — o conhecimento *executável* da empresa (o "como rodar" de cada processo automatizado pelo Claude), versionado e independente de pessoas ou computadores pessoais.

## Por que existe
Skill em cache local ou no laptop de alguém é frágil: sai a pessoa, troca a máquina, some o "como rodar". Aqui é a **fonte canônica**: versionada (histórico + revisão por PR), publicada como plugin pra todos instalarem a mesma versão.

## Como se encaixa na Base de Conhecimento
Três camadas, uma fonte de verdade por camada:

| Camada | Guarda | Onde |
|--------|--------|------|
| **Este repo** | Código + SKILL.md das automações | Git da DevOZ |
| **Plugin** | Este repo empacotado e publicado | Marketplace/plugin |
| **Confluence** (catálogo 🤖 Skills & Automações) | Índice pra humano achar/entender | wiki BDCR |

Regra de ouro: **passo a passo executável vive aqui; contexto/porquê vive na página do Confluence.** Sem cópia entre os dois — um aponta pro outro.

## Estrutura
```
devoz-skills/
├── .claude-plugin/plugin.json   # manifesto
├── skills/
│   ├── <skill>/SKILL.md         # instruções pro Claude (imperativo, enxuto)
│   └── <skill>/scripts/         # código executável da skill
└── README.md
```

## Skills incluídas
- **cockpit-financeiro** — consolida receita BR+US (MRR, churn, NRR); ETL/pipeline.
- **xero-cockpit** — faturamento Xero (DevOZ LLC, US).
- **superlogica-cockpit** — faturamento Superlógica (DevOZ LTDA, BR).
- **zoho-crm-cockpit** — cruza Zoho CRM × faturamento (ativos, churn, país).
- **ozmap-brand** — diretrizes de marca OZmap/DevOZ para documentos.
- **skill-exemplo** — template em branco para novas skills.

## Como adicionar uma skill
1. Copie `skills/skill-exemplo/` e renomeie (kebab-case).
2. Preencha o `SKILL.md` (description em 3ª pessoa com gatilhos; corpo imperativo).
3. Coloque o código em `scripts/`. **Nunca** comite segredos — use `.env` local e secrets do CI.
4. Registre a skill no catálogo do Confluence e linke da página da área dona.
5. Suba a versão em `plugin.json` (semver) e abra PR.

## Código do cockpit (apontado, não copiado)
As skills da família cockpit (xero, superlógica, zoho, cockpit-financeiro) **não** carregam o código aqui. O código é uma aplicação coesa (orquestrador, módulos compartilhados, schema de banco, pipeline no GitHub Actions) e vive no seu repo canônico: <https://github.com/rodrigo-kautzmann/cockpit-devoz>. Cada `SKILL.md` tem a seção **Código-fonte (canônico)** linkando os arquivos exatos. Assim não há duas fontes de verdade.

Skills genuinamente autossuficientes (sem app própria) podem, sim, carregar o código em `skills/<nome>/scripts/`.

## Segredos
Nenhum token/credencial no repo. Use variáveis de ambiente localmente e secrets do GitHub Actions no CI (Supabase `DATABASE_URL`, OAuth Xero/Zoho, API Superlógica).
