---
name: skill-exemplo
description: >
  Template em branco para criar uma nova skill de automação da DevOZ. NÃO é uma
  skill funcional — copie esta pasta, renomeie e preencha. Uma boa description é
  em 3ª pessoa e lista as frases/gatilhos que devem ativar a skill ("use sempre
  que ... mesmo sem citar X").
---

# Nome da skill (substitua)

> Skills são instruções PARA o Claude executar, não documentação para humano ler.
> Escreva no imperativo ("Extraia...", "Rode...", "Valide..."), enxuto.

## Quando usar
Liste os gatilhos: que pedido do usuário deve disparar esta skill.

## Pré-requisitos
- Acessos/conectores necessários.
- Variáveis de ambiente (nunca segredos no repo — use `.env` / secrets do CI).

## Passo a passo
1. Primeiro passo objetivo.
2. Segundo passo (qual script rodar, com quais argumentos).
3. Como validar o resultado.

## Scripts
Código executável em `./scripts/`. Aponte caminhos relativos.

## Página de contexto (Confluence)
Linke aqui a página da base que explica o "porquê/política" deste processo,
quando houver. Regra: contexto na página, execução na skill — sem cópia.
