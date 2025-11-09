# Análise: Migração Híbrida - Apenas Agentes para Agno

## 📋 Sumário Executivo

Esta análise avalia a viabilidade de migrar **apenas os agentes** para o Agno, mantendo o resto da arquitetura (FastAPI, Langfuse, Qdrant, Supabase, ThreadManager, etc.).

**Recomendação:** ✅ **VIÁVEL, mas com ressalvas importantes**

---

## 🎯 Abordagem Proposta

### O Que Seria Migrado
- ✅ **Apenas os 3 agentes:** SQLAgent, OptimizationAgent, AnomalyAgent
- ✅ **Lógica de agentes:** Substituir LangChain/LangGraph por Agno

### O Que Seria Mantido
- ✅ **FastAPI:** Rotas e estrutura atual
- ✅ **Langfuse:** Observabilidade e tracing
- ✅ **Qdrant + Supabase:** RAG e exemplos
- ✅ **ThreadManager:** Gerenciamento de threads customizado
- ✅ **Tools customizadas:** Guardrails SQL, etc.
- ✅ **ML/Análise:** PyOD, Pandas, visualizações

---

## 🔍 Análise de Compatibilidade

### 1. **Uso Standalone dos Agentes Agno**

#### Como Funciona Atualmente
```python
# optimization_routes.py
async def ask_optimization_agent(body: AskOptimizationAgentRequest):
    agent = OptimizationAgent()  # Instância da classe customizada
    response = await agent.ask(...)
    return response
```

#### Como Funcionaria com Agno
```python
from agno.agent import Agent
from agno.models.openai import OpenAI

# Criar agente Agno standalone
agno_agent = Agent(
    name="OptimizationAgent",
    model=OpenAI(id="gpt-4.1-nano"),
    # ... configurações
)

# Usar diretamente nas rotas
async def ask_optimization_agent(body: AskOptimizationAgentRequest):
    response = await agno_agent.run(...)
    return response
```

**✅ Viabilidade:** **ALTA** - Agno permite criar agentes standalone sem AgentOS

---

### 2. **Integração com FastAPI Atual**

#### Compatibilidade
- ✅ **Agentes Agno são objetos Python normais** - podem ser instanciados e usados diretamente
- ✅ **Métodos assíncronos** - Agno suporta async/await
- ✅ **Retorno de dados** - Agentes retornam objetos Python que podem ser serializados

#### Exemplo de Integração
```python
# api/optimization_routes.py (adaptado)
from agno.agent import Agent
from agno.models.openai import OpenAI
from utils.langfuse import LangfuseManager

# Criar wrapper que mantém interface atual
class OptimizationAgentAgno:
    def __init__(self):
        self.agno_agent = Agent(
            name="OptimizationAgent",
            model=OpenAI(id="gpt-4.1-nano"),
            # Integrar com ThreadManager customizado
            # Integrar com Langfuse
        )
    
    async def ask(self, question, _bl_tenant, _bl_account, ...):
        # Usar ThreadManager atual
        thread_id = self.thread_manager.get_or_create_thread(...)
        
        # Executar agente Agno
        response = await self.agno_agent.run(question)
        
        # Integrar com Langfuse
        langfuse_handler.trace(...)
        
        return response
```

**✅ Viabilidade:** **ALTA** - Integração direta possível

---

### 3. **Compatibilidade com Langfuse**

#### Desafio
- ⚠️ Agno tem seu próprio sistema de observabilidade (Agno Telemetry)
- ⚠️ Não há integração nativa com Langfuse

#### Soluções Possíveis

**Opção A: Wrapper com Callbacks**
```python
from agno.agent import Agent
from langfuse.callback import CallbackHandler

class OptimizationAgentAgno:
    def __init__(self):
        self.agno_agent = Agent(...)
        self.langfuse_handler = LangfuseManager().get_handler(...)
    
    async def ask(self, question, ...):
        # Executar com callback do Langfuse
        # Agno pode não suportar callbacks externos diretamente
        # Precisaria interceptar chamadas
        pass
```

**Opção B: Tracing Manual**
```python
async def ask(self, question, ...):
    # Iniciar trace no Langfuse
    trace = langfuse_client.trace(...)
    
    # Executar agente Agno
    response = await self.agno_agent.run(question)
    
    # Registrar no Langfuse manualmente
    trace.generation(...)
    trace.end()
```

**⚠️ Viabilidade:** **MÉDIA** - Requer trabalho adicional para integração

---

### 4. **Compatibilidade com ThreadManager Customizado**

#### Desafio
- ⚠️ Agno tem sistema de memória próprio (`add_history_to_context=True`)
- ⚠️ ThreadManager atual usa Supabase + OpenAI Threads API

#### Solução
```python
class OptimizationAgentAgno:
    def __init__(self):
        self.thread_manager = ThreadManager()  # Manter atual
        self.agno_agent = Agent(
            name="OptimizationAgent",
            # NÃO usar db do Agno
            # Usar ThreadManager customizado
        )
    
    async def ask(self, question, thread_id, ...):
        # Usar ThreadManager atual
        thread_id, is_new = self.thread_manager.get_or_create_thread(...)
        history = self.thread_manager.get_thread_history(thread_id)
        
        # Converter histórico para formato Agno
        agno_messages = self._convert_history_to_agno(history)
        
        # Executar agente com histórico
        response = await self.agno_agent.run(
            question,
            messages=agno_messages  # Passar histórico manualmente
        )
        
        # Salvar resposta no ThreadManager
        self.thread_manager.add_message_to_thread(thread_id, response.content)
        
        return response
```

**✅ Viabilidade:** **ALTA** - Pode usar ThreadManager atual, apenas adaptar formato

---

### 5. **Compatibilidade com RAG (Qdrant + Supabase)**

#### Desafio
- ⚠️ Agno tem sistema de Knowledge próprio
- ⚠️ Sistema atual usa Qdrant + Supabase customizado

#### Solução
```python
from biglink_utils.qdrant import ExampleRetriever

class OptimizationAgentAgno:
    def __init__(self):
        self.example_retriever = ExampleRetriever(...)  # Manter atual
        self.agno_agent = Agent(...)
    
    async def ask(self, question, ...):
        # Buscar exemplos com sistema atual
        examples = await self.example_retriever.sql_examples(question)
        
        # Inserir exemplos no contexto do agente Agno
        context = self._format_examples_for_agno(examples)
        
        # Executar agente com contexto
        response = await self.agno_agent.run(
            question,
            context=context  # Adicionar exemplos ao contexto
        )
        
        return response
```

**✅ Viabilidade:** **ALTA** - Pode manter RAG atual, apenas adaptar formato

---

### 6. **Compatibilidade com Tools Customizadas**

#### Desafio
- ⚠️ Agno tem sistema de tools próprio
- ⚠️ Tools atuais: SQLDatabaseToolkit, QuerySQLDataBaseToolWithGuardrails

#### Solução
```python
from agno.tools import Tool
from tools.sql_agent_tools import QuerySQLDataBaseToolWithGuardrails

# Criar wrapper para tool customizada
class SQLToolAgno(Tool):
    def __init__(self, db):
        self.sql_tool = QuerySQLDataBaseToolWithGuardrails(db)
    
    async def run(self, query: str):
        return self.sql_tool.run(query)

# Usar no agente
agno_agent = Agent(
    tools=[SQLToolAgno(db)]
)
```

**✅ Viabilidade:** **ALTA** - Pode criar wrappers para tools existentes

---

### 7. **Compatibilidade com ML/Análise (PyOD, Pandas)**

#### Análise
- ✅ **AnomalyAgent:** Usa PyOD para detecção de anomalias
- ✅ **Processamento:** Pandas para manipulação de dados
- ✅ **Visualizações:** Matplotlib para gráficos

#### Solução
```python
class AnomalyAgentAgno:
    def __init__(self):
        self.agno_agent = Agent(...)  # Apenas para geração de SQL
        # Manter lógica ML separada
    
    async def ask(self, question, ...):
        # 1. Usar Agno para gerar SQL
        sql_query = await self.agno_agent.run(question)
        
        # 2. Executar query (lógica atual)
        df = self.run_sql_query(db, sql_query)
        
        # 3. Detecção de anomalias (lógica atual - PyOD)
        df = self.detect_anomalies(df, metric)
        
        # 4. Visualização (lógica atual)
        chart_data = self.prepare_chart_data(df, metric)
        
        # 5. Análise final com Agno
        final_response = await self.agno_agent.run(
            f"Analise estas anomalias: {df.to_json()}"
        )
        
        return {
            "answer": final_response.content,
            "chart_data": chart_data
        }
```

**✅ Viabilidade:** **ALTA** - Lógica ML pode ser mantida separada

---

## 📊 Comparação: Abordagem Híbrida vs. Migração Completa

| Aspecto | Migração Completa | Migração Híbrida (Apenas Agentes) |
|---------|-------------------|-----------------------------------|
| **Esforço** | 3-5 meses | 4-8 semanas |
| **Risco** | Alto | Médio |
| **Complexidade** | Alta | Média |
| **Manutenção** | Tudo novo | Híbrido (novo + antigo) |
| **Flexibilidade** | Limitada ao Agno | Mantém flexibilidade |
| **Custos** | Redução significativa | Redução parcial |
| **Features** | Pode perder algumas | Mantém todas |

---

## ⚙️ Trabalho de Migração Híbrida Estimado

### Fase 1: Setup e Testes (1 semana)
- [ ] Instalar Agno
- [ ] Criar POC com um agente simples
- [ ] Testar integração básica com FastAPI
- [ ] Avaliar compatibilidade de modelos

**Esforço:** 20-30 horas

### Fase 2: Wrappers e Integrações (2 semanas)
- [ ] Criar wrappers para ThreadManager
- [ ] Integrar com Langfuse (tracing manual)
- [ ] Adaptar RAG (Qdrant) para contexto Agno
- [ ] Criar wrappers para tools customizadas
- [ ] Adaptar guardrails SQL

**Esforço:** 40-60 horas

### Fase 3: Migração dos Agentes (2-3 semanas)
- [ ] Migrar SQLAgent
- [ ] Migrar OptimizationAgent
- [ ] Migrar AnomalyAgent
- [ ] Manter lógica ML separada (PyOD)
- [ ] Adaptar integração entre agentes

**Esforço:** 60-80 horas

### Fase 4: Testes e Ajustes (1-2 semanas)
- [ ] Testes de integração
- [ ] Ajustes de prompts
- [ ] Validação de performance
- [ ] Documentação

**Esforço:** 20-40 horas

### **Total Estimado: 6-8 semanas (1.5-2 meses)**

---

## ✅ Vantagens da Abordagem Híbrida

### 1. **Menor Risco**
- ✅ Migração gradual (um agente por vez)
- ✅ Pode reverter se necessário
- ✅ Mantém infraestrutura estável

### 2. **Menor Esforço**
- ✅ 6-8 semanas vs. 3-5 meses
- ✅ Não precisa migrar infraestrutura
- ✅ Mantém features customizadas

### 3. **Flexibilidade**
- ✅ Mantém controle sobre componentes críticos
- ✅ Pode escolher o que migrar
- ✅ Não fica preso ao Agno completamente

### 4. **Manutenção de Features**
- ✅ Mantém Langfuse (observabilidade avançada)
- ✅ Mantém ThreadManager customizado
- ✅ Mantém RAG atual (Qdrant + Supabase)
- ✅ Mantém ML/Análise (PyOD)

---

## ⚠️ Desafios e Limitações

### 1. **Integração com Langfuse**
- ⚠️ Não há integração nativa
- ⚠️ Precisa tracing manual ou wrappers
- ⚠️ Pode perder algumas features automáticas

### 2. **Duplicação de Funcionalidades**
- ⚠️ Agno tem memória própria (não usada)
- ⚠️ Agno tem observabilidade própria (não usada)
- ⚠️ Pode gerar confusão na equipe

### 3. **Manutenção Híbrida**
- ⚠️ Precisa manter conhecimento de dois sistemas
- ⚠️ Debugging mais complexo (Agno + componentes atuais)
- ⚠️ Atualizações podem quebrar integrações

### 4. **Performance**
- ⚠️ Wrappers podem adicionar overhead
- ⚠️ Conversões de formato (histórico, contexto)
- ⚠️ Pode não ter todos os benefícios de performance do Agno

---

## 🎯 Recomendações Específicas

### ✅ **Fazer Migração Híbrida Se:**
1. Quer testar Agno sem compromisso total
2. Precisa manter infraestrutura atual
3. Quer migração gradual e segura
4. Tem 6-8 semanas disponíveis
5. Equipe pode lidar com sistema híbrido

### ❌ **NÃO Fazer Se:**
1. Quer simplificar completamente
2. Precisa reduzir custos imediatamente
3. Não quer manter dois sistemas
4. Equipe não tem capacidade para híbrido
5. Quer todos os benefícios do Agno (precisa migração completa)

---

## 📝 Plano de Migração Híbrida Sugerido

### Etapa 1: POC (1 semana)
```python
# Criar POC com SQLAgent apenas
# Testar integração básica
# Validar viabilidade técnica
```

### Etapa 2: Migração Gradual (4-6 semanas)
```python
# Semana 1-2: SQLAgent
# Semana 3-4: OptimizationAgent  
# Semana 5-6: AnomalyAgent
```

### Etapa 3: Otimização (1-2 semanas)
```python
# Otimizar wrappers
# Melhorar integrações
# Documentar
```

---

## 🔧 Exemplo de Implementação

### SQLAgent com Agno (Híbrido)

```python
# agents/sql_agent_agno.py
from agno.agent import Agent
from agno.models.openai import OpenAI
from langchain_community.utilities import SQLDatabase
from tools.sql_agent_tools import QuerySQLDataBaseToolWithGuardrails
from utils.langfuse import LangfuseManager
from biglink_utils.postgres import PostgresManager
from settings import sql_agent_settings

class SQLAgentAgno:
    def __init__(self):
        self.settings = sql_agent_settings
        self.langfuse_manager = LangfuseManager()
        
        # Configurar banco de dados
        DB_URI = PostgresManager().get_db_uri()
        self.db = SQLDatabase.from_uri(DB_URI, schema=self.settings.DB_SCHEMA)
        
        # Criar tools customizadas
        sql_tool = QuerySQLDataBaseToolWithGuardrails(db=self.db)
        
        # Criar agente Agno
        self.agent = Agent(
            name="SQLAgent",
            model=OpenAI(id="gpt-5-nano"),
            tools=[self._create_agno_tool(sql_tool)],
            # Não usar db do Agno, manter controle manual
        )
    
    def _create_agno_tool(self, langchain_tool):
        """Wrapper para converter tool LangChain em Agno"""
        from agno.tools import Tool
        
        class SQLToolAgno(Tool):
            name = langchain_tool.name
            description = langchain_tool.description
            
            async def run(self, query: str):
                return langchain_tool.run(query)
        
        return SQLToolAgno()
    
    async def ask(self, user_message: str, _bl_tenant: str, _bl_account: str):
        # Buscar prompt do Langfuse (manter atual)
        try:
            system_prompt, langfuse_prompt = self.langfuse_manager.get_prompt(
                prompt_name="SQL_Agent_Prompt",
                variables={...}
            )
        except:
            system_prompt = self.settings.SQL_AGENT_SYSTEM_PROMPT.format(...)
        
        # Iniciar trace no Langfuse (manual)
        trace = self.langfuse_manager.client.trace(
            name="SQLAgent",
            metadata={"model": self.settings.MODEL}
        )
        
        try:
            # Executar agente Agno
            response = await self.agent.run(
                user_message,
                system=system_prompt
            )
            
            # Registrar no Langfuse
            trace.generation(
                name="SQLAgent",
                input=user_message,
                output=response.content,
                metadata={"prompt": langfuse_prompt.name if langfuse_prompt else None}
            )
            
            return {"answer": response.content}
            
        except Exception as e:
            trace.event(name="error", metadata={"error": str(e)})
            raise
        finally:
            trace.end()
```

---

## 📊 Conclusão

### Viabilidade: ✅ **ALTA para Abordagem Híbrida**

**Resumo:**
- ✅ **Tecnicamente viável** - Agno permite uso standalone
- ✅ **Menor risco** - Migração gradual possível
- ✅ **Menor esforço** - 6-8 semanas vs. 3-5 meses
- ✅ **Mantém features** - Infraestrutura atual preservada
- ⚠️ **Trabalho adicional** - Wrappers e integrações necessárias
- ⚠️ **Manutenção híbrida** - Dois sistemas para manter

**Recomendação Final:**
A abordagem híbrida é **viável e recomendada** se você quer:
1. Testar Agno sem compromisso total
2. Manter infraestrutura atual funcionando
3. Fazer migração gradual e segura
4. Preservar features customizadas importantes

É uma **excelente estratégia de transição** que permite avaliar o Agno enquanto mantém estabilidade.

---

**Data da Análise:** Janeiro 2025  
**Versão:** 1.0

