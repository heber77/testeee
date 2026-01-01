# Como o Extrator de Planilhas Funciona

## 🔍 Por que o texto parece "sem formatação" no terminal?

O texto **ESTÁ formatado**, mas em formato **Markdown**! O terminal do Windows PowerShell não renderiza Markdown, então você vê o "código bruto" com os pipes `|`.

### O que você vê no terminal:
```
| Conceito | Descrição | Exemplo Prático | Aplicação |
|---|---|---|---|
| Supervised Learning | Aprendizado supervisionado... | ... | ... |
```

### Como ficaria renderizado (Markdown):
| Conceito | Descrição | Exemplo Prático | Aplicação |
|---|---|---|---|
| Supervised Learning | Aprendizado supervisionado usa dados rotulados para treinar modelos | Classificação de emails como spam/não-spam | Sistemas de recomendação, detecção de fraudes |

---

## 📋 Como o Extrator Funciona (Passo a Passo)

### 1. **Entrada** (`ExtractorInput`)
```python
input_data = ExtractorInput(source_uri="caminho/para/planilha.xlsx")
```

### 2. **Validação Inicial**
- ✅ Verifica se o arquivo existe
- ✅ Verifica se o formato é suportado (`.xlsx`, `.xls`, `.csv`, `.ods`)

### 3. **Extração Estrutural** (método principal)

#### 3.1. Carregamento da Planilha
```python
# Usa pandas para ler a planilha
excel_file = pd.ExcelFile(file_path)
```

#### 3.2. Processamento de Cada Aba
Para cada aba na planilha:

**a) Filtragem de Abas Administrativas:**
```python
# Exclui abas como: backup, temp, old, test, tmp, _old
if sheet_name.lower() in ["backup", "temp", "old", "test", "tmp", "_old"]:
    continue  # Pula esta aba
```

**b) Filtragem de Colunas Administrativas:**
```python
# Remove colunas como: id, timestamp, uuid, hash, version, etc.
relevant_columns = [
    col for col in df.columns
    if not self._is_admin_column(str(col).lower())
]
```

**c) Remoção de Linhas Vazias:**
```python
# Remove linhas completamente vazias
df_filtered = df_filtered.dropna(how="all")
```

**d) Limite de Linhas:**
```python
# Limita a 1000 linhas por aba (evita textos muito longos)
max_rows = 1000
for idx, row in df_filtered.head(max_rows).iterrows():
    # Processa linha...
```

#### 3.3. Conversão para Markdown
```python
# Constrói tabela em formato Markdown
text_parts = [
    f"Aba: {sheet_name}",           # Nome da aba
    "",                              # Linha vazia
    f"Tabela: {sheet_name}",         # Título da tabela
    f"| {' | '.join(columns)} |",   # Cabeçalho
    "|---|---|---|",                 # Separador
    f"| {row_values} |",             # Cada linha de dados
    # ...
]
```

### 4. **Formatação dos Valores**
```python
# Remove .0 de números inteiros
if isinstance(val, float) and val.is_integer():
    formatted_values.append(str(int(val)))  # 10.0 → "10"
else:
    formatted_values.append(str(val))
```

### 5. **Saída** (`ExtractorOutput`)
```python
ExtractorOutput(
    extracted_text="Aba: Conceitos\n\nTabela: Conceitos\n| ...",  # Texto em Markdown
    extraction_type=ContentCategory.INSTRUCTIONAL,
    format_type=InstructionalFormatType.SHEET,
    metadata=SheetMetadata(
        pages=1,
        worksheets=["Conceitos"],
        total_rows=10,
        total_columns=4,
        extraction_method="pandas",
        file_extension=".xlsx"
    )
)
```

---

## 🎯 Estrutura do Texto Extraído

O texto segue este padrão:

```
Aba: [Nome da Aba]

Tabela: [Nome da Aba]
| Coluna 1 | Coluna 2 | Coluna 3 |
|---|---|---|
| Valor 1 | Valor 2 | Valor 3 |
| Valor 4 | Valor 5 | Valor 6 |
```

### Exemplo Real (do seu arquivo):

**Texto bruto (como está no JSON):**
```
Aba: Conceitos

Tabela: Conceitos
| Conceito | Descrição | Exemplo Prático | Aplicação |
|---|---|---|---|
| Supervised Learning | Aprendizado supervisionado usa dados rotulados para treinar modelos | Classificação de emails como spam/não-spam | Sistemas de recomendação, detecção de fraudes |
```

**Como ficaria renderizado (Markdown):**

| Conceito | Descrição | Exemplo Prático | Aplicação |
|---|---|---|---|
| Supervised Learning | Aprendizado supervisionado usa dados rotulados para treinar modelos | Classificação de emails como spam/não-spam | Sistemas de recomendação, detecção de fraudes |
| Unsupervised Learning | Aprendizado não supervisionado encontra padrões em dados sem rótulos | Agrupamento de clientes por comportamento | Segmentação de mercado, redução de dimensionalidade |

---

## 🔄 Fluxo Completo do Extrator

```
┌─────────────────┐
│  Planilha XLSX  │
│  (arquivo)      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Carrega com    │
│  pandas/Excel   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Filtra Abas    │
│  (remove backup,│
│   temp, etc)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Filtra Colunas │
│  (remove ID,    │
│   timestamp)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Remove Linhas  │
│  Vazias         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Converte para  │
│  Markdown       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ExtractorOutput│
│  + Metadata     │
└─────────────────┘
```

---

## 💡 Por que Markdown?

### Vantagens do formato Markdown:

1. **Legível para humanos**: Mesmo sem renderização, é fácil de ler
2. **Legível para LLMs**: LLMs entendem bem tabelas Markdown
3. **Estruturado**: Preserva a organização dos dados
4. **Padrão**: Markdown é um formato amplamente suportado
5. **Compatível**: Funciona bem com embeddings e processamento downstream

### Onde o Markdown será renderizado:

- ✅ **LLMs** (ChatGPT, Claude, etc.) - entendem perfeitamente
- ✅ **Sistemas de embeddings** - processam bem tabelas estruturadas
- ✅ **Interfaces web** - podem renderizar como HTML
- ✅ **Documentação** - GitHub, GitLab, etc.

---

## 🧪 Testando a Formatação

### Ver o texto completo formatado:

1. **Abrir o JSON gerado:**
```bash
# O arquivo exemplo_extracao_completo.json contém o texto completo
```

2. **Copiar o texto e colar em um editor Markdown:**
   - VS Code (com extensão Markdown Preview)
   - GitHub (criar um arquivo .md)
   - Qualquer editor Markdown online

3. **Ver renderizado:**
   - O texto aparecerá como tabela formatada!

---

## 📊 Exemplo Visual

### No Terminal (PowerShell):
```
| Conceito | Descrição | Exemplo Prático |
|---|---|---|
| Supervised Learning | Aprendizado supervisionado... | Classificação de emails... |
```

### Renderizado (Markdown):
| Conceito | Descrição | Exemplo Prático |
|---|---|---|
| Supervised Learning | Aprendizado supervisionado usa dados rotulados para treinar modelos | Classificação de emails como spam/não-spam |

---

## ✅ Conclusão

O extrator está funcionando **perfeitamente**! O texto está formatado em **Markdown**, que é:

- ✅ **Estruturado** (preserva tabelas)
- ✅ **Legível** (mesmo sem renderização)
- ✅ **Compatível** (funciona com LLMs e embeddings)
- ✅ **Padrão** (formato amplamente suportado)

O fato de parecer "sem formatação" no terminal é **normal** - o terminal não renderiza Markdown. Mas quando esse texto for usado por:
- LLMs (para processamento)
- Sistemas de embeddings (para vetorização)
- Interfaces web (para exibição)

Ele será **perfeitamente formatado** e **legível**! 🎯

