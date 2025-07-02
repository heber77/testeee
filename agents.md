# Documentação dos Agentes

O módulo de agentes é responsável por processar perguntas dos usuários e gerar respostas através da análise de dados armazenados em bancos de dados. O sistema utiliza Large Language Models (LLMs) para interpretar perguntas em linguagem natural e convertê-las em consultas SQL apropriadas.

### Estrutura do Módulo

- **ConnectorAgent**: Classe principal para agentes individuais de cada plataforma de ads
- **SupervisorAdsAgent**: Agente supervisor para coordenar múltiplos conectores (documentado em `supervisor_agent.md`)

## ConnectorAgent

A classe `ConnectorAgent` é responsável por processar perguntas relacionadas a uma plataforma específica (Meta Ads, Google Ads, TikTok Ads, LinkedIn Ads, Google Analytics, Pinterest Ads).

### Atributos da Classe

| Atributo | Tipo | Descrição |
|----------|------|-----------|
| `settings` | `Module` | Módulo de configurações carregado dinamicamente contendo parâmetros específicos da plataforma |
| `retriever` | `ExampleRetriever` | Instância responsável por recuperar exemplos de consultas SQL|

### Dependências Externas

A classe utiliza as seguintes instâncias globais:
- `postgres_manager`: Gerenciador de conexões PostgreSQL
- `langfuse_manager`: Gerenciador de observabilidade e logging de LLMs

### Construtor

```python
def __init__(self, settings_path: str)
```

**Parâmetros:**
- `settings_path` (str): Caminho para o arquivo de configurações (.py) da plataforma específica

**Funcionalidade:**
- Carrega dinamicamente as configurações da plataforma
- Inicializa o retriever de exemplos baseado nas configurações

### Métodos Privados

#### `_load_settings(path: str)`

**Descrição:** Carrega dinamicamente um módulo de configurações a partir de um caminho de arquivo.

**Parâmetros:**
- `path` (str): Caminho completo para o arquivo de configurações

**Retorno:** 
- `Module`: Módulo Python carregado dinamicamente

**Funcionalidade:**
- Utiliza `importlib.util` para carregar módulos dinamicamente
- Permite configurações flexíveis por plataforma sem alteração de código

### Métodos Públicos

#### `get_language_name(language_code: str) -> str`

**Descrição:** Converte códigos de idioma em nomes por extenso em português.

**Parâmetros:**
- `language_code` (str): Código do idioma (ex: "en", "pt-br", "es")

**Retorno:**
- `str`: Nome do idioma em português (ex: "inglês", "português do Brasil", "espanhol")

**Mapeamentos Suportados:**
- `"en"` → `"inglês"`
- `"pt-br"` → `"português do Brasil"`
- `"es"` → `"espanhol"`
- Outros códigos retornam o valor original

#### `get_platform_name() -> str`

**Descrição:** Identifica o nome da plataforma baseado no arquivo de configurações carregado.

**Retorno:**
- `str`: Nome da plataforma (ex: "meta_ads", "google_ads", "tiktok_ads")

**Mapeamentos Suportados:**
- `meta_ads.py` → `"meta_ads"`
- `google_ads.py` → `"google_ads"`
- `linkedin_ads.py` → `"linkedin_ads"`
- `tiktok_ads.py` → `"tiktok_ads"`
- `google_analytics.py` → `"google_analytics"`
- `pinterest_ads.py` → `"pinterest_ads"`

**Funcionalidade:**
- Extrai o nome do arquivo de configurações
- Registra informação de log sobre a plataforma identificada

#### `classify_question_type(question: str, llm) -> bool`

**Descrição:** Classifica se uma pergunta é conceitual ou de dados utilizando um LLM.

**Parâmetros:**
- `question` (str): Pergunta do usuário a ser classificada
- `llm`: Instância do Large Language Model para classificação

**Retorno:**
- `bool`: `True` se a pergunta for conceitual, `False` se for pergunta de dados

**Funcionalidade:**
- Perguntas conceituais acionam busca web para contexto adicional
- Utiliza prompt específico do settings (`QUESTION_CLASSIFICATION_PROMPT`)

#### `get_sql_query(response: dict) -> str | None`

**Descrição:** Extrai a consulta SQL executada da resposta do agente.

**Parâmetros:**
- `response` (dict): Resposta completa do agente contendo mensagens e tool calls

**Retorno:**
- `str`: Query SQL executada pelo agente
- `None`: Se nenhuma query SQL foi encontrada

**Funcionalidade:**
- Percorre as mensagens em ordem reversa para encontrar a última query
- Busca especificamente por tool calls do tipo "sql_db_query"
- Extrai e decodifica os argumentos JSON da query

#### `get_answer(response: dict) -> str`

**Descrição:** Extrai a resposta final do agente.

**Parâmetros:**
- `response` (dict): Resposta completa do agente

**Retorno:**
- `str`: Conteúdo da última mensagem (resposta final)

#### `extract_prompt_from_response(response: dict) -> str`

**Descrição:** Reconstrói toda a interação realizada pelo agente para debugging.

**Parâmetros:**
- `response` (dict): Resposta completa do agente

**Retorno:**
- `str`: Prompt completo formatado com todas as interações

**Funcionalidade:**
- Reconstrói cronologicamente todas as interações
- Formata diferentes tipos de mensagens (system, tool, ai, human)
- Útil para debugging e análise de comportamento do agente

#### `ask(data: dict) -> tuple[str, str, str]`

**Descrição:** Método principal que processa uma pergunta e retorna a resposta completa.

**Parâmetros:**
- `data` (dict): Dicionário contendo todos os dados da requisição

**Estrutura do `data`:**
```python
{
    "_bl_tenant": str,           # Identificador do tenant
    "connector": [               # Lista de conectores (apenas 1 para agentes individuais)
        {
            "_bl_account": str,  # Identificador da conta
            "provider": str      # Nome do provedor da plataforma
        }
    ],
    "question": str,             # Pergunta do usuário
    "thread_history": str,       # Histórico da conversa (opcional)
    "filters": {                 # Filtros de data (opcional)
        "startDate": str,        # Data inicial (YYYY-MM-DD)
        "endDate": str          # Data final (YYYY-MM-DD)
    },
    "language": str             # Código do idioma (opcional)
}
```

**Retorno:**
- `tuple[str, str, str]`: Tupla contendo (sql_query, answer_html, answer_md)
  - `sql_query`: Query SQL executada (ou "{}" se nenhuma)
  - `answer_html`: Resposta formatada em HTML
  - `answer_md`: Resposta formatada em Markdown

**Fluxo de Processamento:**

1. **Extração de Parâmetros:**
   - Extrai tenant, account e configurações do data
   - Aplica valores padrão do settings quando necessário

2. **Configuração do Ambiente:**
   - Estabelece conexão com banco de dados PostgreSQL
   - Inicializa o LLM com modelo e temperatura configurados

3. **Processamento do Histórico:**
   - Reformula a pergunta considerando histórico da conversa
   - Utiliza função `reformulate_question_with_history` quando aplicável

4. **Classificação da Pergunta:**
   - Determina se é pergunta conceitual ou de dados
   - Para perguntas conceituais, realiza busca web adicional

5. **Preparação das Ferramentas:**
   - Configura SQLDatabaseToolkit com ferramentas personalizadas
   - Substitui ferramentas padrão por versões seguras com guardrails:
     - `sql_db_list_tables` → `make_sql_db_list_tables_tool`
     - `sql_db_query` → `QuerySQLDataBaseToolWithGuardrails`
     - `sql_db_query_checker` → `QuerySQLCheckerToolWithNullsLast`

6. **Criação do Agente:**
   - Utiliza `create_react_agent` do LangGraph
   - Aplica system message específico da plataforma

7. **Preparação do Contexto:**
   - Formata instruções do sistema com parâmetros específicos
   - Adiciona contexto web para perguntas conceituais
   - Inclui exemplos de consultas SQL relevantes

8. **Execução:**
   - Invoca o agente com mensagens estruturadas
   - Registra interação no Langfuse para observabilidade

9. **Extração de Resultados:**
   - Extrai query SQL executada
   - Processa resposta para formatos HTML e Markdown
   - Remove quebras de linha do HTML para compatibilidade

**Tratamento de Erros:**
- Captura e registra exceções no log
- Re-propaga exceções para tratamento em nível superior
- Mantém rastreabilidade completa de erros

## Exemplo de Uso

```python
# Inicialização do agente
meta_agent = ConnectorAgent("./settings/meta_ads.py")

# Preparação dos dados
data = {
    "_bl_tenant": "exemplo_tenant",
    "connector": [{
        "_bl_account": "123456789",
        "provider": "meta-ads"
    }],
    "question": "Quais campanhas tiveram melhor performance no último mês?",
    "filters": {
        "startDate": "2024-01-01",
        "endDate": "2024-01-31"
    },
    "language": "pt-br"
}

# Execução
sql_query, answer_html, answer_md = meta_agent.ask(data)
```

## Considerações Técnicas

### Performance
- Utiliza busca web apenas para perguntas conceituais
- Implementa cache através do ExampleRetriever
- Processamento assíncrono através de thread pools

### Segurança
- Ferramentas SQL com guardrails para prevenir queries maliciosas
- Validação de parâmetros através das configurações
- Logging completo para auditoria

### Observabilidade
- Integração com Langfuse para rastreamento de LLM
- Logging estruturado em todos os pontos críticos
- Extração completa de prompts para debugging

### Flexibilidade
- Configurações dinâmicas por plataforma
- Suporte a múltiplos idiomas
- Adaptação automática a diferentes esquemas de banco 