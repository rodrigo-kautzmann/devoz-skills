---
name: ozmap-brand
description: >
  Diretrizes de marca da OZmap / DevOZ. Use esta skill SEMPRE que o usuário pedir para criar,
  gerar ou editar qualquer documento de saída (Word, PDF, apresentação, planilha, relatório,
  e-mail formatado, template). Mesmo que o pedido não mencione "marca" ou "OZmap" — se há um
  documento sendo produzido para a empresa, aplique estas diretrizes. Inclui paleta de cores
  oficial, tipografia, hierarquia e regras de aplicação da identidade visual.
---

# Diretrizes de Marca OZmap

Sempre que produzir qualquer documento em nome da OZmap / DevOZ, siga rigorosamente estas diretrizes. Elas vêm da Cartilha de Marca oficial da empresa (2023).

## Identidade da empresa

**DevOZ** é a empresa. **OZmap** é o produto principal (software de gestão e documentação de redes ópticas). Sede em Florianópolis, desde 2012.

---

## Paleta de Cores

Estas são as únicas cores permitidas em documentos OZmap:

| Papel               | HEX       | RGB              |
|---------------------|-----------|------------------|
| Verde principal     | `#00D256` | 0, 210, 86       |
| Preto               | `#000000` | 0, 0, 0          |
| Branco              | `#FFFFFF` | 255, 255, 255    |

**Regras:**
- Verde `#00D256` → destaques, cabeçalhos, linhas divisórias, bordas de tabela, bullets
- Preto `#000000` → títulos principais, textos de corpo
- Branco `#FFFFFF` → fundos, texto sobre verde
- Não use outras cores sem justificativa explícita do usuário
- Para fundos suaves, use uma versão clara do verde (ex.: `#E8FBF0`) com moderação

---

## Tipografia

### Fonte do nome da marca (títulos, papelaria)
- **Primária:** Maven Pro  
- **Fallback:** Calibri

### Fonte de materiais gráficos e corpo (use esta em documentos Office)
- **Primária:** Axiforma  
- **Fallback 1:** Poppins ← **use esta em docx/pptx/xlsx** (amplamente disponível)
- **Fallback 2:** Montserrat

### Hierarquia de uso
| Elemento                       | Fonte                   |
|-------------------------------|-------------------------|
| Nome da marca / logo text     | Maven Pro (ou Calibri)  |
| Títulos e subtítulos          | Poppins bold            |
| Blocos de texto / corpo       | Poppins regular         |
| Texto auxiliar / notas        | Poppins light ou italic |

**Em documentos Word (.docx), use Poppins em toda a hierarquia** (é o fallback oficial da Axiforma).

---

## Aplicação em Documentos Word / DOCX

### Estrutura padrão de página

```
┌─────────────────────────────────────────┐
│ CABEÇALHO: fundo #00D256                │
│  OZmap  |  [tipo do documento]          │
│ Faixa preta fina (#000000)              │
├─────────────────────────────────────────┤
│                                         │
│  TÍTULO PRINCIPAL  ← Poppins Bold       │
│  ─────────────────────── (linha verde)  │
│                                         │
│  Corpo do texto  ← Poppins Regular      │
│                                         │
│  ┌────────────────────────────────┐     │
│  │ Caixa destaque (borda verde)   │     │
│  └────────────────────────────────┘     │
│                                         │
├─────────────────────────────────────────┤
│ RODAPÉ: linha verde, texto cinza        │
│  © DevOZ | ozmap.com  — Página N        │
└─────────────────────────────────────────┘
```

### Parâmetros docx-js

```javascript
// Cores
const GREEN  = "00D256";
const BLACK  = "000000";
const WHITE  = "FFFFFF";
const GRAY   = "4A4A4A";   // texto secundário
const LIGHT  = "E8FBF0";   // fundo suave (usar com moderação)

// Tipografia
font: "Poppins"

// Título principal
size: 72-80, bold: true, color: BLACK

// Subtítulo / seção
size: 32-36, bold: true, color: BLACK

// Corpo
size: 24, color: GRAY

// Notas / rodapé
size: 18-20, color: GRAY
```

### Elementos obrigatórios em todo documento

1. **Cabeçalho** com fundo verde `#00D256`, texto branco "OZmap | [contexto]"
2. **Faixa preta fina** logo abaixo do cabeçalho
3. **Linha divisória verde** após o título principal (`BorderStyle.SINGLE`, size 16, color `00D256`)
4. **Rodapé** com linha verde no topo, texto `© [ano] OZmap | ozmap.com — Página N`

### O que NÃO fazer
- Não usar cores fora da paleta (ex.: azul, roxo, tons aleatórios)
- Não usar fontes como Arial, Calibri como primária em documentos de marca
- Não omitir o cabeçalho/rodapé em documentos formais
- Não rotacionar ou distorcer o nome da marca

---

## Aplicação em Apresentações (PPTX)

- Slide de capa: fundo preto ou verde, título em branco
- Slides internos: fundo branco, destaque verde para elementos-chave
- Paleta restrita: verde, preto, branco
- Fonte: Poppins em toda a apresentação

---

## Aplicação em Planilhas (XLSX)

- Cabeçalho da tabela: fundo `#00D256`, texto branco, Poppins Bold
- Linhas alternadas: branco e `#E8FBF0` (verde muito claro)
- Bordas: `#00D256` ou cinza suave
- Totais / destaques: bold, fundo `#00D256`

---

## Checklist antes de entregar qualquer documento

- [ ] Paleta correta: verde `#00D256`, preto, branco (sem outros tons)
- [ ] Fonte Poppins em todo o documento
- [ ] Cabeçalho com fundo verde e texto "OZmap"
- [ ] Rodapé com crédito "© [ano] OZmap | ozmap.com"
- [ ] Linha divisória verde após título principal
- [ ] Sem elementos visuais fora da identidade da marca

---

## Logo

Os logos estão no Google Drive. Ao criar documentos que precisam da logo:

1. Use o MCP do Google Drive para baixar o arquivo com o ID correto:
   - **Logo positivo** (fundo branco): Drive file ID `1MD7nQ8KbEzJ7Cf8-myI7lKuBVGe7dhHx` — `LOGO OZmap_POSITIVO.png`
   - **Logo negativo** (fundo verde/preto): Drive file ID `1azSTPESCoPIi_64vZ2sgkBvGCo9i5jXD` — `LOGO OZmap_NEGATIVO.jpg`

2. Para baixar, chame `mcp__e083b893-355f-4e33-8ec9-46740e7cf856__download_file_content` com o `fileId` correspondente e decode o base64 para salvar o arquivo de imagem.

3. **Regra de uso:**
   - Cabeçalho com fundo `#00D256` → use o logo **negativo** (branco)
   - Corpo/fundo branco → use o logo **positivo** (colorido)

4. Posicionamento padrão em documentos: canto superior esquerdo do cabeçalho, altura ~1cm, sem distorção.

5. Se o MCP do Drive não estiver disponível na sessão, mencione ao usuário que o logo pode ser inserido manualmente a partir da pasta: https://drive.google.com/drive/folders/11kGGNDadqmeLwnf-axUhtByD885FCy8n
