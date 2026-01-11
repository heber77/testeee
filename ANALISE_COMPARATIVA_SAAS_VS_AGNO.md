# Análise Comparativa: SaaS vs Agno

## Resumo Executivo

Este documento apresenta uma análise completa comparando o repositório **biglink-llm-aai-saas** (versão antiga) com o repositório **biglink-llm-aai-agno** (versão nova baseada no framework Agno). O objetivo é identificar todas as funcionalidades presentes no SaaS que ainda não foram implementadas no Agno, para guiar o trabalho futuro de migração completa.

---

## 1. Guardrails de Segurança

### 1.1 Guardrail de Comandos SQL ⚠️ **CRÍTICO - FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Descrição:**
O SaaS possui uma classe `QuerySQLDataBaseToolWithGuardrails` que herda de `QuerySQLDataBaseTool` e bloqueia comandos SQL perigosos antes da execução.

**Implementação no SaaS:**
```python
# biglink-llm-aai-saas/tools/connector_tools.py
class QuerySQLDataBaseToolWithGuardrails(QuerySQLDataBaseTool):
    def _process_query(self, query: str):
        forbidden_commands = [
            "update", "delete", "drop", "create", "rename", 
            "alter", "truncate", "insert", "merge", "grant",
            "revoke", "commit", "rollback"
        ]
        
        query_lower = query.lower()
        for command in forbidden_commands:
            if command in query_lower:
                raise ValueError(
                    f"Comando SQL não permitido detectado: '{command}'. "
                    "Apenas comandos SELECT são permitidos."
                )
        
        result = self.db.run_no_throw(query)
        return result
```

**Implementação no Agno:**
O Agno usa diretamente `PostgresTools` do framework Agno, que **não possui guardrails** para bloquear comandos perigosos. Os prompts instruem o LLM a gerar apenas SELECT, mas não há validação programática.

**Recomendação:**
Criar uma classe `ReadOnlyPostgresTools` que herda de `PostgresTools` e sobrescreve o método `_execute_query` para adicionar validação de comandos proibidos, similar ao SaaS.

**Arquivo a modificar:**
- `biglink-llm-aai-agno/agents/sql_agent.py` - Substituir `PostgresTools` por `ReadOnlyPostgresTools`

---

### 1.2 Guardrail de Idioma ⚠️ **CRÍTICO - FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Descrição:**
O SaaS possui um guardrail que detecta o idioma da resposta e traduz automaticamente se não estiver no idioma configurado pelo usuário.

**Implementação no SaaS:**
```python
# biglink-llm-aai-saas/agents/connector_agent.py (linha 332-335)
import langdetect

if language_code.replace("pt-br", "pt") not in langdetect.detect(answer):
    answer = translation_agent.translate(answer, target_lang=language_name)
```

**Implementação no Agno:**
O Agno confia apenas nos prompts para garantir o idioma correto. Não há validação programática nem tradução automática.

**Recomendação:**
1. Adicionar `langdetect` como dependência
2. Criar um agente de tradução similar ao SaaS (`agents/translation_agent.py`)
3. Adicionar validação de idioma após a geração da resposta em:
   - `routes/agent.py` (função `execute_agent`)
   - `routes/ads_team.py` (função `generate_response`)
   - `routes/social_team.py` (função `generate_response`)
   - `routes/analytics_team.py` (função `generate_response`)
   - `utils/job_processor.py` (função `run_agent`)

**Arquivos a criar/modificar:**
- `biglink-llm-aai-agno/agents/translation_agent.py` (novo)
- `biglink-llm-aai-agno/routes/agent.py`
- `biglink-llm-aai-agno/routes/ads_team.py`
- `biglink-llm-aai-agno/routes/social_team.py`
- `biglink-llm-aai-agno/routes/analytics_team.py`
- `biglink-llm-aai-agno/utils/job_processor.py`

---

## 2. Funcionalidades de API

### 2.1 Endpoints de Threads ❌ **FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Endpoints no SaaS:**
- `GET /thread/{tenant}/{account}/{connector}` - Obtém histórico de uma thread
- `DELETE /thread/{tenant}/{account}/{connector}` - Remove uma thread
- `GET /threads` - Lista todas as threads

**Implementação no Agno:**
O Agno usa o sistema de storage do Agno (PostgreSQL/SQLite) para gerenciar sessões, mas não expõe endpoints REST para gerenciar threads externamente.

**Recomendação:**
Criar um router `routes/threads_routes.py` similar ao SaaS, mas adaptado para usar o sistema de storage do Agno.

**Arquivos a criar:**
- `biglink-llm-aai-agno/routes/threads_routes.py` (novo)
- Modificar `main.py` para incluir o router

---

### 2.2 Endpoints de Exemplos ❌ **FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Endpoints no SaaS:**
- `GET /get_examples/{connector}` - Obtém todos os exemplos de um conector
- `POST /search_examples` - Busca semântica nos exemplos
- `POST /insert_example` - Insere novo exemplo
- `POST /update_example` - Atualiza exemplo existente

**Implementação no Agno:**
O Agno usa Qdrant para RAG, mas não expõe endpoints REST para gerenciar exemplos.

**Recomendação:**
Criar um router `routes/examples_routes.py` similar ao SaaS, adaptado para usar o sistema Qdrant do Agno.

**Arquivos a criar:**
- `biglink-llm-aai-agno/routes/examples_routes.py` (novo)
- Modificar `main.py` para incluir o router

---

### 2.3 Endpoint de Monitoramento (Prometheus) ❌ **FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Endpoint no SaaS:**
- `GET /metrics` - Expõe métricas do Prometheus

**Métricas no SaaS:**
- `generate_response_time` (Summary)
- `generate_response_count` (Counter)
- `generate_response_by_agent_time` (Summary)
- `generate_response_by_agent_count` (Counter)
- `generate_response_social_time` (Summary)
- `generate_response_social_count` (Counter)
- `run_agent_step_time` (Summary com labels: step_name)
- `connector_agent_step_time` (Summary com labels: step_name, api, env)

**Implementação no Agno:**
O Agno não possui integração com Prometheus.

**Recomendação:**
1. Adicionar `prometheus_client` como dependência
2. Criar `routes/monitoring_routes.py` com endpoint `/metrics`
3. Adicionar métricas nos endpoints principais (similar ao SaaS)
4. Adicionar métricas de tempo por etapa nos handlers

**Arquivos a criar/modificar:**
- `biglink-llm-aai-agno/routes/monitoring_routes.py` (novo)
- `biglink-llm-aai-agno/routes/agent.py` (adicionar métricas)
- `biglink-llm-aai-agno/routes/ads_team.py` (adicionar métricas)
- `biglink-llm-aai-agno/routes/social_team.py` (adicionar métricas)
- `biglink-llm-aai-agno/routes/analytics_team.py` (adicionar métricas)
- `biglink-llm-aai-agno/utils/job_processor.py` (adicionar métricas)
- `main.py` (incluir router)

---

## 3. Funcionalidades de Processamento

### 3.1 Reformulação de Perguntas com Histórico ✅ **IMPLEMENTADO (via Agno)**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ✅ Implementado (via `add_history_to_context`)

**Descrição:**
O SaaS possui uma função `reformulate_question_with_history` que reformula a pergunta considerando o histórico da conversa usando um LLM.

**Implementação no Agno:**
O Agno usa o sistema nativo do Agno (`add_history_to_context=True`) que automaticamente inclui o histórico no contexto do agente. Não há necessidade de reformulação explícita, pois o LLM recebe o histórico completo.

**Status:** ✅ Funcionalidade equivalente presente

---

### 3.2 Classificação de Perguntas (Conceitual vs Dados) ❌ **FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Descrição:**
O SaaS classifica perguntas como "conceituais" (definições, funcionamento) ou "dados" (métricas, performance) para decidir se deve usar busca na web.

**Implementação no SaaS:**
```python
# biglink-llm-aai-saas/agents/connector_agent.py
def classify_question_type(self, question: str, llm) -> bool:
    # Usa LLM com structured output para classificar
    # Retorna True se for conceitual, False se for de dados
```

**Implementação no Agno:**
O Agno não possui classificação de perguntas. Não há distinção entre perguntas conceituais e de dados.

**Recomendação:**
Adicionar classificação de perguntas antes de executar o agente para otimizar o uso de recursos (busca web apenas para perguntas conceituais).

**Arquivos a criar/modificar:**
- `biglink-llm-aai-agno/agents/sql_agent.py` (adicionar método de classificação)
- `biglink-llm-aai-agno/routes/agent.py` (usar classificação)

---

### 3.3 Busca na Web para Perguntas Conceituais ❌ **FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Descrição:**
O SaaS possui uma ferramenta `WebSearchAdsTool` que busca informações conceituais na web usando OpenRouter (modelo `openai/gpt-4o-mini-search-preview`).

**Implementação no SaaS:**
- `tools/connector_tools.py` - Classe `WebSearchAdsTool`
- Usa OpenRouter com modelo de busca
- Busca informações sobre plataformas de anúncios antes de executar o agente

**Implementação no Agno:**
O Agno não possui busca na web.

**Recomendação:**
Adicionar ferramenta de busca web similar ao SaaS, integrada ao sistema de tools do Agno.

**Arquivos a criar:**
- `biglink-llm-aai-agno/tools/web_search.py` (novo)

---

### 3.4 QuerySQLCheckerToolWithNullsLast ❌ **FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Descrição:**
O SaaS possui uma versão customizada do `QuerySQLCheckerTool` que garante que cláusulas `DESC` sempre tenham `NULLS LAST` no PostgreSQL.

**Implementação no SaaS:**
```python
# biglink-llm-aai-saas/tools/connector_tools.py
class QuerySQLCheckerToolWithNullsLast(QuerySQLCheckerTool):
    def _add_nulls_last(self, query: str) -> str:
        pattern = re.compile(r"\bDESC\b(?!\s+NULLS\s+LAST)", re.IGNORECASE)
        return pattern.sub(lambda m: m.group(0) + " NULLS LAST", query)
```

**Implementação no Agno:**
O Agno não possui esta customização. O framework Agno usa `PostgresTools` padrão.

**Recomendação:**
Se necessário, criar uma versão customizada do checker de SQL do Agno (se o framework permitir extensão).

---

## 4. Funcionalidades de Observabilidade

### 4.1 Métricas Detalhadas por Etapa ❌ **FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Descrição:**
O SaaS possui métricas Prometheus detalhadas por etapa do processamento:
- `run_agent_step_time` com labels `step_name`
- `connector_agent_step_time` com labels `step_name`, `api`, `env`

**Etapas medidas no SaaS:**
- Validação e Configuração Inicial
- Gestão de Threads
- Geração de Título
- Recuperação de Histórico
- Execução do Agente
- Finalização da Thread
- Instanciamento do banco de dados
- Reformulação da mensagem
- Classificação de necessidade de busca web
- Execução de busca web
- Configuração do executor do agente
- Interação com o Langfuse
- Preparação de prompt
- Recuperação de exemplos
- Execução do agente
- Recuperação de query e resposta
- Tradução da resposta

**Implementação no Agno:**
O Agno possui logs detalhados (adicionados recentemente), mas não possui métricas Prometheus por etapa.

**Recomendação:**
Adicionar métricas Prometheus por etapa similar ao SaaS para monitoramento detalhado.

---

### 4.2 Cálculo de Custo (OpenAI Callback) ❌ **FALTANDO**

**Status no SaaS:** ✅ Implementado  
**Status no Agno:** ❌ Não implementado

**Descrição:**
O SaaS calcula e retorna o custo das inferências usando `get_openai_callback()` do LangChain.

**Implementação no SaaS:**
```python
# biglink-llm-aai-saas/api/agents_routes.py
from langchain_community.callbacks import get_openai_callback

with get_openai_callback() as cb:
    response = await run_agent(body, agent_name)
response.cost = cb.total_cost
return response
```

**Implementação no Agno:**
O Agno não calcula nem retorna custos.

**Recomendação:**
Adicionar cálculo de custo nas respostas, se necessário para o front-end.

---

## 5. Funcionalidades de Autenticação

### 5.1 Autenticação HTTP Basic ❌ **DIFERENTE**

**Status no SaaS:** ✅ HTTP Basic Auth  
**Status no Agno:** ✅ Bearer Token (OS_SECURITY_KEY)

**Descrição:**
- **SaaS:** Usa HTTP Basic Authentication (`API_USER` e `API_PASS`)
- **Agno:** Usa Bearer Token (`OS_SECURITY_KEY`)

**Status:** ✅ Funcionalidade equivalente presente (diferentes métodos)

---

## 6. Funcionalidades de Gerenciamento de Threads

### 6.1 ThreadManager com Supabase ❌ **DIFERENTE**

**Status no SaaS:** ✅ ThreadManager customizado com Supabase  
**Status no Agno:** ✅ Sistema de storage do Agno (PostgreSQL/SQLite)

**Descrição:**
- **SaaS:** Usa `biglink_utils.threads.ThreadManager` que gerencia threads no Supabase
- **Agno:** Usa sistema nativo do Agno com PostgreSQL/SQLite

**Status:** ✅ Funcionalidade equivalente presente (diferentes implementações)

**Nota:** O SaaS possui endpoints REST para gerenciar threads, o Agno não (ver seção 2.1).

---

## 7. Funcionalidades de RAG e Exemplos

### 7.1 Gerenciamento de Exemplos via API ❌ **FALTANDO**

**Status no SaaS:** ✅ Endpoints REST para CRUD de exemplos  
**Status no Agno:** ❌ Apenas leitura via RAG

**Descrição:**
O SaaS permite inserir, atualizar e buscar exemplos via API REST. O Agno apenas lê exemplos do Qdrant durante o RAG.

**Recomendação:**
Implementar endpoints REST para gerenciar exemplos (ver seção 2.2).

---

## 8. Funcionalidades de Multi-Agent (Supervisor)

### 8.1 Supervisor Agents ❌ **DIFERENTE**

**Status no SaaS:** ✅ Supervisor Agents (LangGraph)  
**Status no Agno:** ✅ Teams (Agno Teams)

**Descrição:**
- **SaaS:** Usa LangGraph com `SupervisorAdsAgent` e `SupervisorSocialAgent`
- **Agno:** Usa sistema de Teams do Agno (`ads_team`, `social_team`, `analytics_team`)

**Status:** ✅ Funcionalidade equivalente presente (diferentes arquiteturas)

**Nota:** O SaaS possui exemplos específicos para supervisor no Qdrant (`supervisor`, `supervisor-final-answer`, `supervisor-social`, `supervisor-social-final-answer`). O Agno pode precisar adaptar os exemplos para o formato de Teams.

---

## 9. Funcionalidades de Prompt Management

### 9.1 Gerenciamento de Prompts via Langfuse ✅ **IMPLEMENTADO**

**Status no SaaS:** ✅ Prompts gerenciados no Langfuse  
**Status no Agno:** ✅ Prompts gerenciados no Langfuse

**Descrição:**
Ambos os sistemas usam Langfuse para gerenciar prompts, com fallback local.

**Status:** ✅ Funcionalidade equivalente presente

---

## 10. Resumo de Funcionalidades Faltantes

### 🔴 **CRÍTICO - Segurança**
1. ❌ **Guardrail de Comandos SQL** - Bloquear DELETE, UPDATE, INSERT, etc.
2. ❌ **Guardrail de Idioma** - Detectar e traduzir respostas automaticamente

### 🟡 **IMPORTANTE - API e Monitoramento**
3. ❌ **Endpoints de Threads** - GET, DELETE, LIST threads
4. ❌ **Endpoints de Exemplos** - CRUD de exemplos via API
5. ❌ **Endpoint de Monitoramento** - `/metrics` para Prometheus
6. ❌ **Métricas Prometheus** - Métricas detalhadas por etapa
7. ❌ **Cálculo de Custo** - Retornar custo das inferências

### 🟢 **DESEJÁVEL - Otimizações**
8. ❌ **Classificação de Perguntas** - Conceitual vs Dados
9. ❌ **Busca na Web** - Para perguntas conceituais
10. ❌ **QuerySQLCheckerToolWithNullsLast** - Garantir NULLS LAST em DESC

---

## 11. Priorização Recomendada

### Fase 1 - Segurança (CRÍTICO)
1. Guardrail de Comandos SQL
2. Guardrail de Idioma

### Fase 2 - Funcionalidades Core (IMPORTANTE)
3. Endpoints de Threads
4. Endpoints de Exemplos
5. Endpoint de Monitoramento (Prometheus)
6. Métricas Prometheus básicas

### Fase 3 - Otimizações (DESEJÁVEL)
7. Classificação de Perguntas
8. Busca na Web
9. Métricas Prometheus detalhadas por etapa
10. Cálculo de Custo
11. QuerySQLCheckerToolWithNullsLast

---

## 12. Notas de Implementação

### 12.1 Guardrail de SQL
- Criar classe `ReadOnlyPostgresTools` que herda de `PostgresTools`
- Sobrescrever método de execução de queries
- Validar comandos proibidos antes da execução
- Manter compatibilidade com a API do Agno

### 12.2 Guardrail de Idioma
- Adicionar dependência `langdetect`
- Criar `TranslationAgent` similar ao SaaS
- Integrar validação após geração de resposta
- Considerar usar modelo mais barato para tradução (ex: `gpt-5-nano` como no SaaS)

### 12.3 Endpoints de Threads
- Adaptar `ThreadManager` do SaaS para usar storage do Agno
- Ou criar wrapper sobre o sistema de storage do Agno
- Manter compatibilidade com formato de resposta do SaaS

### 12.4 Endpoints de Exemplos
- Adaptar funções do `biglink_utils.qdrant` para usar sistema Qdrant do Agno
- Manter compatibilidade com formato de resposta do SaaS
- Considerar filtros por `api='agno'` vs `api='saas'`

### 12.5 Prometheus
- Adicionar `prometheus_client` como dependência
- Criar router de monitoramento
- Adicionar métricas nos endpoints principais
- Considerar métricas por etapa (similar ao SaaS)

---

## 13. Conclusão

O repositório Agno possui uma base sólida e arquitetura moderna usando o framework Agno. No entanto, faltam funcionalidades críticas de segurança (guardrails SQL e idioma) e funcionalidades importantes de API e monitoramento que existem no SaaS.

A priorização recomendada é:
1. **Primeiro:** Implementar guardrails de segurança (SQL e idioma)
2. **Segundo:** Implementar endpoints de gerenciamento (threads, exemplos)
3. **Terceiro:** Implementar monitoramento (Prometheus)
4. **Quarto:** Implementar otimizações (classificação, busca web)

Esta análise serve como guia para a equipe trabalhar na migração completa do SaaS para o Agno.
