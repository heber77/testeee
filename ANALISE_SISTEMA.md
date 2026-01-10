# Análise Completa do Sistema BigLink LLM AAI

## 📋 Visão Geral

O **BigLink LLM AAI** é um sistema multi-agente baseado no framework **Agno** para análise inteligente de dados de marketing digital. O sistema utiliza uma arquitetura de **Teams of Agents** (Times de Agentes) onde agentes especializados colaboram para responder perguntas sobre campanhas publicitárias, métricas de redes sociais e analytics.

### Objetivo Principal
Fornecer análises consultivas e recomendações práticas sobre dados de marketing digital através de uma API REST, utilizando agentes de IA que podem:
- Consultar dados via SQL
- Utilizar busca semântica (RAG) para contexto adicional
- Raciocinar sobre os dados e gerar insights
- Detectar anomalias e analisar sentimentos

---

## 🏗️ Arquitetura do Sistema

### 1. Estrutura Hierárquica

```
AgentOS (Orquestrador Principal)
├── Ads Team (Time de Anúncios)
│   ├── Google Ads Agent
│   ├── Meta Ads Agent
│   ├── TikTok Ads Agent
│   ├── LinkedIn Ads Agent
│   ├── Pinterest Ads Agent
│   ├── Sentiment Agent (membro auxiliar)
│   └── Outlier Agent (membro auxiliar)
│
├── Social Team (Time de Redes Sociais)
│   ├── Facebook Insights Agent
│   ├── Instagram Insights Agent
│   ├── Sentiment Agent (membro auxiliar)
│   └── Outlier Agent (membro auxiliar)
│
├── Analytics Team (Time de Analytics)
│   ├── Google Analytics Agent
│   ├── Sentiment Agent (membro auxiliar)
│   └── Outlier Agent (membro auxiliar)
│
└── Agentes Especializados (Standalone)
    ├── Outlier Agent
    └── Sentiment Agent
```

### 2. Componentes Principais

#### **AgentOS** (`main.py`)
- Orquestrador principal do framework Agno
- Gerencia todos os agentes e times
- Expõe API REST via FastAPI
- Processa jobs assíncronos via Redis

#### **Agentes SQL** (`agents/sql_agent.py`)
- Factory function que cria agentes especializados por plataforma
- Cada agente tem acesso a:
  - **PostgresTools**: Execução de queries SQL no PostgreSQL
  - **KnowledgeTools**: Busca semântica no Qdrant (RAG)
  - **Storage**: Persistência de sessões e memórias
- Prompts personalizados por plataforma

#### **Times** (`teams/`)
- Coordenam múltiplos agentes para responder perguntas complexas
- Cada time tem:
  - Agentes membros especializados
  - Knowledge base compartilhada
  - Storage de sessões e memórias
  - Prompts específicos para orquestração

#### **Rotas** (`routes/`)
- Endpoints da API REST:
  - `/agent/{agent_name}` - Agentes individuais
  - `/agent/sentiment` - Análise de sentimentos
  - `/agent/outlier` - Detecção de outliers
  - `/ads-team` - Time de anúncios
  - `/social-team` - Time de redes sociais
  - `/analytics-team` - Time de analytics

---

## 🔄 Fluxo de Processamento

### Fluxo de uma Consulta Individual (Agente SQL)

```
1. Cliente faz requisição POST /agent/{agent_name}
   ↓
2. Router valida autenticação (Bearer token)
   ↓
3. Cria agente customizado com:
   - Prompt personalizado (baseado em plataforma, idioma, datas)
   - Thread ID (sessão)
   - Filtros (_bl_tenant, _bl_account, período)
   ↓
4. Agente processa a pergunta:
   a. Busca contexto no KnowledgeTools (Qdrant RAG)
   b. Gera query SQL baseada no prompt e contexto
   c. Executa query no PostgreSQL
   d. Analisa os dados retornados
   e. Gera resposta consultiva com insights
   ↓
5. Resposta retornada ao cliente:
   {
     "thread_id": "...",
     "agent_name": "...",
     "answer": "**Análise detalhada...**",
     "title": "Pergunta original"
   }
```

### Fluxo de uma Consulta em Time

```
1. Cliente faz requisição POST /ads-team (ou /social-team, /analytics-team)
   ↓
2. Router valida autenticação
   ↓
3. Cria time customizado com:
   - Agentes membros (baseados nos connectors fornecidos)
   - Prompt de orquestração
   - Thread ID
   ↓
4. Time processa a pergunta:
   a. Analisa qual(is) agente(s) deve(m) responder
   b. Delega para agente(s) especializado(s)
   c. Cada agente:
      - Busca contexto (RAG)
      - Gera e executa SQL
      - Analisa dados
   d. Time consolida respostas dos agentes
   e. Gera resposta final unificada
   ↓
5. Resposta retornada ao cliente
```

### Fluxo de Processamento Assíncrono

```
1. Cliente faz requisição com async_response=true
   ↓
2. Job é adicionado à fila Redis
   ↓
3. Resposta imediata: {"job_id": "...", "status": "pending"}
   ↓
4. JobProcessor (background task) processa:
   a. Lê job da fila Redis (blpop)
   b. Recupera parâmetros do job
   c. Executa agente
   d. Envia resultado via callback HTTP
   ↓
5. Cliente recebe resultado no callback URI
```

---

## 🛠️ Funcionalidades Detalhadas

### 1. Agentes SQL (Anúncios, Social, Analytics)

**Plataformas Suportadas:**
- **Ads**: Google Ads, Meta Ads, TikTok Ads, LinkedIn Ads, Pinterest Ads
- **Social**: Facebook Insights, Instagram Insights
- **Analytics**: Google Analytics

**Capacidades:**
- ✅ Consulta SQL direta ao PostgreSQL
- ✅ Busca semântica (RAG) para contexto adicional
- ✅ Cálculo automático de métricas (CPC, CPM, CTR, CPA, ROI, etc.)
- ✅ Análise consultiva com insights e recomendações
- ✅ Persistência de sessões e histórico
- ✅ Prompts personalizados por plataforma e idioma

**Exemplo de Uso:**
```json
POST /agent/google-ads
{
  "_bl_tenant": "example",
  "_bl_account": "8654245841",
  "filters": {"startDate": "2025-01-01", "endDate": "2025-12-31"},
  "language": "pt-br",
  "question": "Quais são as campanhas com maior custo?"
}
```

### 2. Agente de Detecção de Outliers

**Funcionalidade:**
- Detecta anomalias em séries temporais usando **Isolation Forest** (PyOD)
- Classifica severidade: CRÍTICO, ALTO, MODERADO
- Gera hipóteses sobre causas
- Fornece recomendações práticas

**Algoritmo:**
- Isolation Forest com contamination configurável (padrão: 15%)
- Z-score para classificação de severidade
- Análise de direção (ACIMA/ABAIXO da média)

**Exemplo de Uso:**
```json
POST /agent/outlier
{
  "metric_name": "CPA",
  "time_series": [
    {"date": "2025-01-01", "value": 15.0},
    {"date": "2025-01-02", "value": 14.5},
    {"date": "2025-01-03", "value": 45.0}  // Outlier detectado
  ]
}
```

**Resposta:**
```json
{
  "metric_name": "CPA",
  "outliers_detected": 1,
  "anomaly_intervals": [{
    "date": "2025-01-03",
    "value": 45.0,
    "severity": "CRÍTICO",
    "direction": "ACIMA",
    "deviation_percentage": 200.0
  }],
  "hypotheses": [...],
  "recommendations": [...]
}
```

### 3. Agente de Análise de Sentimentos

**Funcionalidade:**
- Classifica sentimento de comentários em 5 categorias:
  - **APROVAÇÃO**: Concordância, elogio, entusiasmo
  - **REJEIÇÃO**: Rejeição, hostilidade, desqualificação
  - **CRÍTICA**: Crítica construtiva, sugestões
  - **INCERTEZA**: Dúvidas, questionamentos
  - **IRRELEVANTE**: Comentários vagos ou sem relação

**Formatos Suportados:**
- Formato simples: `post` + `comments[]`
- Formato DataFrame: lista de dicts com auto-detecção de colunas

**Exemplo de Uso:**
```json
POST /agent/sentiment
{
  "post": "Lançamento do produto X! 🚀",
  "comments": [
    "Adorei!",
    "Não gostei...",
    "Quando estará disponível?"
  ]
}
```

### 4. Sistema de Prompts

**Arquitetura:**
- **Langfuse** como fonte principal (label "production")
- **Fallback local** quando Langfuse não disponível
- Cache de 5 minutos para prompts do Langfuse

**Prompts Disponíveis:**
- `ads-agent` - Agentes de anúncios
- `ads-team` - Orquestrador de anúncios
- `social-agent` - Agentes de redes sociais
- `social-team` - Orquestrador de redes sociais
- `analytics-agent` - Agentes de analytics
- `analytics-team` - Orquestrador de analytics
- `sentiment-agent` - Análise de sentimentos
- `outlier-agent` - Detecção de outliers

**Variáveis de Prompt:**
- `platform` - Plataforma (google-ads, meta-ads, etc.)
- `lang` - Idioma (pt-br, en-us, etc.)
- `lang_name` - Nome do idioma
- `tenant` - Identificador do tenant
- `account` - ID da conta
- `start_date`, `end_date` - Período
- `db_tables` - Tabelas disponíveis

### 5. Sistema RAG (Retrieval-Augmented Generation)

**Componentes:**
- **Qdrant**: Banco de dados vetorial
- **Embeddings**: Modelo `intfloat/multilingual-e5-small`
- **Knowledge Base**: Coleções por plataforma/connector

**Funcionamento:**
1. Exemplos de perguntas/respostas são armazenados no Supabase
2. Dados são indexados no Qdrant com embeddings
3. Agente busca contexto semântico antes de gerar SQL
4. Contexto enriquece a geração de queries e respostas

**Estrutura de Dados:**
```json
{
  "question": "Qual a campanha com melhor ROI?",
  "query": "SELECT campaign_name, SUM(revenue - cost) / NULLIF(SUM(cost), 0) * 100 as roi...",
  "agent": "google-ads",
  "connector": "google-ads",
  "status": "active"
}
```

### 6. Persistência e Memória

**Storage:**
- **PostgreSQL** (preferencial) ou **SQLite** (fallback)
- Tabelas por agente/time:
  - `{agent}_sessions` - Histórico de sessões
  - `{agent}_memories` - Memórias persistentes
  - `{agent}_metrics` - Métricas de uso

**Funcionalidades:**
- Histórico de conversas (últimos N pares)
- Resumos de sessão
- Memórias persistentes (desabilitado por padrão)

### 7. Processamento Assíncrono

**Componentes:**
- **Redis**: Fila de jobs e armazenamento de parâmetros
- **JobProcessor**: Background task que processa fila
- **Callback HTTP**: Envio de resultados para cliente

**Fluxo:**
1. Cliente envia requisição com `async_response=true`
2. Job é adicionado à fila Redis
3. Resposta imediata com `job_id`
4. JobProcessor processa em background
5. Resultado enviado via callback HTTP

---

## 📁 Estrutura de Arquivos

```
biglink-llm-aai-agno/
├── main.py                    # Entry point, AgentOS, FastAPI
├── pyproject.toml             # Dependências
│
├── base/                       # Instâncias base
│   ├── agents.py              # Todos os agentes base
│   └── teams.py               # Todos os times base
│
├── agents/                     # Factory functions de agentes
│   ├── sql_agent.py          # Factory para agentes SQL
│   ├── outlier_agent.py      # Agente de outliers
│   └── sentiment_agent.py    # Agente de sentimentos
│
├── teams/                     # Factory functions de times
│   ├── ads_team.py           # Time de anúncios
│   ├── social_team.py         # Time de redes sociais
│   └── analytics_team.py      # Time de analytics
│
├── routes/                     # Endpoints da API
│   ├── agent.py              # /agent/*
│   ├── ads_team.py           # /ads-team
│   ├── social_team.py        # /social-team
│   └── analytics_team.py     # /analytics-team
│
├── prompts/                    # Sistema de prompts
│   ├── ads_prompt.py         # Prompts de anúncios
│   ├── social_prompt.py      # Prompts de redes sociais
│   ├── analytics_prompt.py    # Prompts de analytics
│   ├── outlier_prompt.py      # Prompts de outliers
│   └── sentiment_prompt.py    # Prompts de sentimentos
│
├── utils/                      # Utilitários
│   ├── tracing.py            # Langfuse/OpenInference
│   ├── prompts.py            # Gestão de prompts Langfuse
│   ├── qdrant.py             # Cliente Qdrant e knowledge
│   ├── embeddings.py         # Configuração de embeddings
│   ├── storage.py            # Persistência (PostgreSQL/SQLite)
│   ├── auth.py               # Autenticação Bearer token
│   ├── llm_provider.py       # Configuração de LLM
│   ├── rate_limiter.py       # Limitação de requisições
│   ├── redis.py              # Cliente Redis
│   └── job_processor.py      # Processamento assíncrono
│
└── tools/                      # Ferramentas customizadas
    └── detect_outlier.py      # Detecção de outliers (Isolation Forest)
```

---

## 🔐 Segurança e Autenticação

**Autenticação:**
- Bearer token via `OS_SECURITY_KEY`
- Middleware de autenticação em todas as rotas
- Validação via `utils/auth.py`

**Rate Limiting:**
- Middleware de limitação de requisições
- Padrão: 10 requisições por minuto
- Configurável via `requests_per_minute`

---

## 🗄️ Banco de Dados

### PostgreSQL (Dados de Marketing)

**Schemas por Plataforma:**
- `{prefix}_google_ads_llm`
- `{prefix}_meta_ads_llm`
- `{prefix}_tiktok_ads_llm`
- `{prefix}_linkedin_ads_llm`
- `{prefix}_pinterest_ads_llm`
- `{prefix}_facebook_insights_llm`
- `{prefix}_instagram_insights_llm`
- `{prefix}_google_analytics_llm`

**Tabelas Principais:**
- `metrics` - Métricas agregadas
- `metrics_data_country` - Métricas por país
- `metrics_gender_age` - Métricas por gênero/idade
- `metrics_platform_device` - Métricas por plataforma/dispositivo
- `metrics_page_posts` - Posts de páginas
- `metrics_comments` - Comentários
- `metrics_media` - Mídia

### Qdrant (RAG)

**Coleções:**
- Uma coleção por plataforma (ex: `google-ads`)
- Coleções para times (ex: `ads-team`)
- Embeddings: `intfloat/multilingual-e5-small`

### Supabase (Exemplos para RAG)

**Tabela `examples`:**
- Armazena exemplos de perguntas/respostas
- Filtros: `connector`, `status`
- Sincronizado com Qdrant

### PostgreSQL/SQLite (Storage)

**Tabelas:**
- `{agent}_sessions` - Sessões
- `{agent}_memories` - Memórias
- `{agent}_metrics` - Métricas

---

## 🔧 Tecnologias e Dependências

### Framework Principal
- **Agno 2.3.5** - Framework multi-agente

### API e Web
- **FastAPI** - API REST
- **Uvicorn** - ASGI server

### Banco de Dados
- **PostgreSQL** (psycopg) - Dados de marketing
- **Qdrant** - Banco vetorial (RAG)
- **Redis** - Fila de jobs e cache
- **SQLite** - Fallback para storage

### LLM e NLP
- **OpenAI/OpenRouter** - Provedores de LLM
- **LangChain** - Processamento de linguagem
- **Sentence Transformers** - Embeddings
- **PyTorch** - Modelos de ML

### Observabilidade
- **Langfuse** - Observabilidade e gestão de prompts
- **OpenInference** - Padrão de traces
- **OpenTelemetry** - Instrumentação

### Análise de Dados
- **Pandas** - Manipulação de dados
- **PyOD** - Detecção de outliers (Isolation Forest)

---

## 📊 Métricas Calculadas

### Ads Team
- **CPC** = `SUM(cost) / NULLIF(SUM(clicks), 0)`
- **CTR** = `SUM(clicks) / NULLIF(SUM(impressions), 0)`
- **CPM** = `(SUM(cost) / NULLIF(SUM(impressions), 0)) * 1000`
- **CPA** = `SUM(cost) / NULLIF(SUM(conversions), 0)`
- **ROI** = `(SUM(revenue) - SUM(cost)) / NULLIF(SUM(cost), 0) * 100`

### Social Team
- **Taxa de Engajamento** = `SUM(post_engagements) / NULLIF(SUM(post_impressions_unique), 0)`
- **Taxa de Compartilhamento** = `SUM(page_post_shares) / NULLIF(SUM(post_impressions_unique), 0)`
- **Taxa de Alcance Orgânico** = `SUM(post_impressions_organic_unique) / NULLIF(SUM(post_impressions_unique), 0)`

### Analytics Team
- **Taxa de Conversão** = `SUM(conversions) / NULLIF(SUM(sessions), 0)`
- **Taxa de Rejeição** = `SUM(bounces) / NULLIF(SUM(sessions), 0)`
- **Páginas por Sessão** = `SUM(pageviews) / NULLIF(SUM(sessions), 0)`

---

## 🚀 Como Funciona na Prática

### Exemplo 1: Consulta Individual

**Pergunta:** "Quais são as campanhas com maior custo no Google Ads?"

1. Cliente envia POST `/agent/google-ads`
2. Sistema cria agente customizado com:
   - Prompt específico para Google Ads
   - Filtros: tenant, account, período
   - Thread ID para sessão
3. Agente:
   - Busca contexto no Qdrant (exemplos similares)
   - Gera SQL: `SELECT campaign_name, SUM(cost) as total_cost FROM metrics WHERE ... GROUP BY campaign_name ORDER BY total_cost DESC`
   - Executa query no PostgreSQL
   - Analisa resultados
   - Gera resposta: "**Campanhas com Maior Custo:**\n\n* Campanha X: R$ 15.000\n* Campanha Y: R$ 12.500\n\n**Insight:** ..."
4. Resposta retornada ao cliente

### Exemplo 2: Consulta em Time

**Pergunta:** "Qual a campanha mais eficiente entre Meta Ads e TikTok Ads?"

1. Cliente envia POST `/ads-team` com connectors de Meta e TikTok
2. Sistema cria time customizado com:
   - Agentes membros: Meta Ads Agent, TikTok Ads Agent
   - Prompt de orquestração
3. Time:
   - Analisa pergunta
   - Delega para Meta Ads Agent e TikTok Ads Agent
   - Cada agente executa sua análise
   - Time consolida respostas
   - Gera resposta unificada comparando ambas plataformas
4. Resposta retornada ao cliente

### Exemplo 3: Detecção de Outliers

**Dados:** Série temporal de CPA com pico anômalo

1. Cliente envia POST `/agent/outlier` com time_series
2. Sistema:
   - Executa Isolation Forest
   - Detecta outliers
   - Classifica severidade (Z-score)
   - Gera hipóteses (campanha viral, erro de tracking, etc.)
   - Fornece recomendações
3. Resposta com análise completa retornada

---

## 🎯 Pontos Fortes do Sistema

1. **Arquitetura Escalável**: Times de agentes permitem colaboração
2. **RAG Integrado**: Contexto semântico enriquece respostas
3. **Prompts Gerenciados**: Langfuse permite versionamento e A/B testing
4. **Observabilidade**: Traces completos via Langfuse/OpenInference
5. **Processamento Assíncrono**: Suporta jobs longos via Redis
6. **Multi-plataforma**: Suporta 8+ plataformas de marketing
7. **Análises Especializadas**: Outliers e sentimentos integrados
8. **Persistência**: Histórico e memória de sessões

---

## ⚠️ Pontos de Atenção

1. **Dependências Externas**: Requer PostgreSQL, Qdrant, Redis rodando
2. **Configuração Complexa**: Muitas variáveis de ambiente
3. **Custos de LLM**: Cada consulta gera chamadas ao LLM
4. **Latência**: RAG + SQL + LLM pode ser lento
5. **Manutenção de Prompts**: Requer gestão ativa no Langfuse

---

## 📝 Conclusão

O sistema é uma solução robusta e bem arquitetada para análise de marketing digital usando IA. A arquitetura de times de agentes permite escalabilidade e especialização, enquanto o RAG e os prompts gerenciados garantem qualidade nas respostas. O sistema está pronto para produção, mas requer infraestrutura adequada (PostgreSQL, Qdrant, Redis) e configuração cuidadosa.
