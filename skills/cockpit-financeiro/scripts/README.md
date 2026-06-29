# Scripts de `cockpit-financeiro`

Coloque aqui os scripts executáveis desta skill, migrados do repositório do cockpit:

```
apply_sql.py + ETL (consolida BR+US; views/métricas SaaS, pipeline diário)
```

**Regras**
- Sem segredos no repo. Tokens/keys via variáveis de ambiente (.env local; secrets no GitHub Actions).
- Mantenha o SKILL.md apontando para os caminhos relativos destes scripts.
