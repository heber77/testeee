# Estrutura de Arquivos - LLM Clients e utils

Lista de arquivos criados ou modificados para implementação dos clients LLM (OpenAI, Anthropic, Groq) e utilitários de proteção (Rate Limiter e Circuit Breaker).

## 📁 Estrutura de Diretórios

```
jedai-ms-extracao-dados/
├── src/
│   └── jedai_ms_extracao_dados/
│       ├── clients/
│       │   ├── __init__.py (modificado)
│       │   ├── llm_base.py (criado)
│       │   ├── openai_llm.py (criado)
│       │   ├── anthropic_llm.py (criado)
│       │   └── groq_llm.py (criado)
│       └── shared/
│           ├── config.py (modificado)
│           └── utils/
│               ├── __init__.py (modificado)
│               ├── rate_limiter.py (criado)
│               └── circuit_breaker.py (criado)
│
└── tests/
    └── unit/
        └── clients/
            ├── test_llm_base.py (criado)
            ├── test_openai_llm.py (criado)
            ├── test_anthropic_llm.py (criado)
            └── test_groq_llm.py (criado)
```
