# Extratores de Conteúdo

Este módulo contém os extratores responsáveis por extrair texto de diferentes formatos de arquivo, seguindo um contrato padrão.

## Estrutura

```
extractors/
├── __init__.py      # Exports dos extratores
├── base.py          # Classe base abstrata e contrato padrão
├── sheet.py         # Extrator de planilhas (Excel, CSV, etc)
└── README.md        # Esta documentação
```

## Contrato Padrão

Todos os extratores seguem o mesmo contrato definido em `base.py`:

```python
from jedai_ms_example.services.extractors.base import BaseExtractor, ExtractionResult

class ExtractionResult:
    extracted_text: str      # Texto extraído e normalizado
    metadata: dict[str, Any] # Informações sobre a extração
```

### Campos Obrigatórios da Metadata

- **`pages`** (int): Número de páginas/abas/seções extraídas
- **`language`** (str): Idioma do conteúdo (ex: "pt-BR", "en-US")
- **`source_type`** (str): Tipo do conteúdo - valores: `"instructional"` ou `"assessment"`
- **`instructional_type`** (str): Tipo específico quando `source_type` é `"instructional"` - valores: `"sheet"`, `"pdf"`, `"doc"`, `"image"`, `"video"`, `"csv"`, `"raw_text"`

### Campos Opcionais da Metadata

- **`extraction_method`** (str): Método usado ("pandas", "llm_fallback", "pypdf", etc)
- **`file_extension`** (str): Extensão do arquivo (".xlsx", ".pdf", etc)
- **Campos específicos por tipo**: Cada extrator pode adicionar campos específicos

### Regras do Contrato

1. **Consistência**: Todos os extratores retornam o mesmo formato
2. **Normalização**: Texto sem formatação binária, legível por humanos e LLMs
3. **Metadata Mínima**: Campos obrigatórios sempre presentes
4. **Extensibilidade**: Campos opcionais podem ser adicionados conforme necessário

## Extrator de Planilhas (sheet.py)

### Uso Básico

```python
import asyncio
from jedai_ms_example.services.extractors.sheet import SheetExtractor

async def main():
    # Criar extrator
    extractor = SheetExtractor()
    
    # Extrair texto
    result = await extractor.extract("caminho/para/planilha.xlsx")
    
    print(result.extracted_text)
    print(result.metadata)

asyncio.run(main())
```

### Formatos Suportados

- `.xlsx` (Excel moderno)
- `.xls` (Excel legado)
- `.csv` (Valores separados por vírgula)
- `.ods` (OpenDocument Spreadsheet)

### Funcionalidades

1. **Extração Estrutural**: Usa pandas/openpyxl para extrair dados
2. **Filtragem Automática**:
   - Remove colunas administrativas (ID, timestamp, uuid, hash, version, etc)
   - Exclui abas administrativas (backup, temp, old, test, tmp, _old)
   - Remove linhas vazias
   - Limita a 1000 linhas por aba
3. **LLM como Fallback**: Opcional, para casos complexos ou quando extração estrutural falha

### Exemplo de Retorno

```python
{
    "extracted_text": "Aba: Conceitos\n\nTabela: Conceitos\n| Conceito | Descrição | Exemplo Prático | Aplicação |\n|---|---|---|---|\n| Supervised Learning | ... | ... | ... |",
    "metadata": {
        "pages": 1,
        "language": "pt-BR",
        "source_type": "instructional",
        "instructional_type": "sheet",
        "worksheets": ["Conceitos"],
        "total_rows": 10,
        "total_columns": 4,
        "has_formulas": False,
        "extraction_method": "pandas",
        "file_extension": ".xlsx"
    }
}
```

### Com LLM (Opcional)

```python
# Com chave OpenAI configurada
extractor = SheetExtractor(openai_api_key="sua-chave")

# Forçar uso de LLM (pula extração estrutural)
result = await extractor.extract("planilha.xlsx", force_llm=True)

# Ou usar como fallback automático (quando texto muito curto ou erro)
result = await extractor.extract("planilha.xlsx", use_llm_fallback=True)
```

## Exemplo Real de Extração

### Exemplo: Planilha com Conceitos de Machine Learning

**Arquivo:** `teste_ml_conceitos.xlsx`

**Resultado da Extração:**

```
✅ EXTRAÇÃO CONCLUÍDA COM SUCESSO

📄 Arquivo: teste_ml_conceitos.xlsx
📊 Método: pandas
📑 Abas/Páginas: 1
📝 Tamanho do texto: 1822 caracteres
🌐 Idioma: pt-BR
📋 Tipo: instructional / sheet
```

**Metadata Completa:**
```json
{
  "pages": 1,
  "language": "pt-BR",
  "source_type": "instructional",
  "instructional_type": "sheet",
  "worksheets": ["Conceitos"],
  "total_rows": 10,
  "total_columns": 4,
  "has_formulas": false,
  "extraction_method": "pandas",
  "file_extension": ".xlsx"
}
```

**Texto Extraído:**
```
Aba: Conceitos

Tabela: Conceitos
| Conceito | Descrição | Exemplo Prático | Aplicação |
|---|---|---|---|
| Supervised Learning | Aprendizado supervisionado usa dados rotulados para treinar modelos | Classificação de emails como spam/não-spam | Sistemas de recomendação, detecção de fraudes |
| Unsupervised Learning | Aprendizado não supervisionado encontra padrões em dados sem rótulos | Agrupamento de clientes por comportamento | Segmentação de mercado, redução de dimensionalidade |
| Neural Networks | Redes neurais são modelos inspirados no cérebro humano | Reconhecimento de imagens, processamento de linguagem natural | Visão computacional, tradução automática |
| Decision Trees | Árvores de decisão fazem escolhas baseadas em regras hierárquicas | Diagnóstico médico baseado em sintomas | Sistemas de apoio à decisão, análise de risco |
| Random Forest | Floresta aleatória combina múltiplas árvores de decisão | Previsão de preços de imóveis | Análise preditiva, classificação de dados |
| Gradient Boosting | Gradient boosting melhora modelos fracos iterativamente | Ranking de resultados de busca | Competições de ML, sistemas de ranking |
| K-Means Clustering | K-means agrupa dados em k clusters baseado em similaridade | Segmentação de clientes de e-commerce | Análise de mercado, organização de dados |
| Support Vector Machines | SVMs encontram o hiperplano ótimo para separar classes | Classificação de textos, reconhecimento facial | Processamento de sinais, bioinformática |
| Deep Learning | Aprendizado profundo usa redes neurais com múltiplas camadas | ChatGPT, sistemas de reconhecimento de voz | IA generativa, automação avançada |
| Reinforcement Learning | Aprendizado por reforço aprende através de tentativa e erro | Jogos como AlphaGo, robótica | Automação industrial, jogos inteligentes |
```

**Características Observadas:**
- ✅ Texto extraído em formato markdown legível
- ✅ Estrutura de tabela preservada
- ✅ Todos os campos obrigatórios presentes na metadata
- ✅ Tipo correto: `instructional` / `sheet`
- ✅ Método de extração identificado: `pandas`

### Testar com Arquivos Reais

Use o script temporário `test_extrator_real.py` na raiz do projeto:

```bash
python test_extrator_real.py caminho/para/seu/arquivo.csv
python test_extrator_real.py caminho/para/seu/arquivo.xlsx
```

O script irá:
- Extrair o texto do arquivo
- Mostrar metadata completa
- Salvar exemplo em `exemplo_extracao_real.txt`

## Criar Novo Extrator

Para criar um novo extrator, herde de `BaseExtractor`:

```python
from jedai_ms_example.services.extractors.base import BaseExtractor, ExtractionResult

class MeuExtrator(BaseExtractor):
    def supports(self, file_path: str) -> bool:
        """Verifica se suporta o arquivo."""
        return file_path.endswith(".meuformato")
    
    async def extract(self, file_path: str, **kwargs) -> ExtractionResult:
        """Extrai texto do arquivo."""
        # ... lógica de extração ...
        texto = "..."
        
        # Retornar no contrato padrão
        return ExtractionResult(
            extracted_text=texto,
            metadata={
                "pages": 1,
                "language": "pt-BR",
                "source_type": "instructional",
                "instructional_type": "meuformato",
                # ... outros campos opcionais
            }
        )
```

### Campos Específicos por Tipo (Opcional)

Cada extrator pode adicionar campos específicos na metadata:

**Planilhas:**
- `worksheets` (list[str]): Lista de nomes das abas
- `total_rows` (int): Número total de linhas
- `total_columns` (int): Número total de colunas
- `has_formulas` (bool): Se contém fórmulas

**PDF:**
- `total_pages` (int): Número total de páginas
- `has_images` (bool): Se contém imagens

**Documentos:**
- `paragraphs` (int): Número de parágrafos
- `has_tables` (bool): Se contém tabelas

**Imagens:**
- `image_format` (str): Formato (PNG, JPEG, etc)
- `dimensions` (dict): {"width": int, "height": int}

**Vídeos:**
- `duration` (float): Duração em segundos
- `has_transcription` (bool): Se tem transcrição

## Dependências

- `pandas`: Manipulação de dados
- `openpyxl`: Excel .xlsx
- `xlrd`: Excel .xls (legado)
- `openai`: LLM fallback (opcional)

## Validação

O contrato é validado automaticamente pelo Pydantic, garantindo tipos corretos e campos obrigatórios presentes.
