# Resumo comparativo MAIC – SINKT

## From MOOC to MAIC: Reshaping Online Teaching and Learning through LLM-driven Agents

O conceito central do MAIC é construir uma série de agentes impulsionados por LLMs para dar suporte tanto ao ensino quanto à aprendizagem no ambiente educacional online.
Pode ser usado para criar e ministrar cursos de forma automática e personalizada, onde cada aluno tem sua própria "sala de aula" com vários agentes de IA que ensinam, tiram dúvidas, gerenciam a interação e simulam colegas.
- Read stage: lê slides (texto + imagem) e estrutura o conhecimento.
- Plan stage: cria ações pedagógicas (ShowFile, ReadScript, AskQuestion).
- Criação de agentes: Teacher, Assistant, Classmates e Manager.
- Sala multiagente: ambiente "1 aluno + N agentes".
- Controle de sessão: módulo que organiza quem fala e quando.
- Avaliação: qualidade dos roteiros, engajamento e resultados de aprendizagem.

## SINKT: A Structure-Aware Inductive Knowledge Tracing Model with Large Language Model

Um modelo de Knowledge Tracing (KT) capaz de prever se um aluno acertará a próxima questão, mesmo quando a questão ou o conceito nunca apareceram antes, algo que os modelos tradicionais não conseguem fazer.
Pode ser usado para diagnosticar o conhecimento do estudante, atualizar seu nível de domínio e prever desempenho, funcionando como um motor de avaliação contínua dentro de sistemas de tutoria inteligente (ITS).
- Grafo conceito–questão: gerado por LLMs com relações semânticas e estruturais.
- Textual Encoder (PLM): entende o texto de questões e conceitos.
- Structural Encoder (GAT): propaga conhecimento no grafo.
- Student State (GRU): modela o estado cognitivo do aluno ao longo do tempo.
- Inductive KT: prevê questões novas (Q_train ≠ Q_test).
- Automated Pipeline: ITS atualiza conceitos e questões automaticamente.

## Comparativo MAIC – SINKT

1. Finalidade
MAIC: criar e gerenciar cursos online com sistemas LLM multiagentes.
SINKT: prever o conhecimento do aluno e sua probabilidade de acerto em próximas questões (KT).
2. Sistema
MAIC: sistema de ensino completo (preparo → ensino → avaliação).
SINKT: módulo analítico dentro de um ITS.
3. Inovações
MAIC: multiagentes + ações pedagógicas estruturadas.
SINKT: KT indutivo com LLM + grafos conceito-questão.

## O que cada modelo se propõe a resolver:

MAIC (gera cursos em pouco tempo, cria agentes autônomos e personaliza a experiência para cada estudante).
Resolve o gargalo dos MOOCs:
- cursos caros e lentos para produzir
- conteúdo fixo e padronizado
- pouca interação

SINKT (permite KT indutivo, usando LLMs para construir grafos e embeddings semânticos).
Resolve limitações dos modelos de KT tradicionais:
- não lidam com questões novas
- dependem de IDs estáticos
- não capturam relações semânticas e estruturais entre conceitos

## Arquitetura:

### MAIC - Arquitetura multiagente

MAIC opera em dois grandes fluxos:
- Fluxo de preparação do curso
1. Read Stage
- LLM multimodal lê slides e extrai
- conteúdo textual
- conteúdo visual
- estrutura hierárquica
- Gera descrições e "knowledge sections"

2. Plan Stage
- Converte conteúdo em ações pedagógicas:
- ReadScript
- ShowFile
- AskQuestion
- Gera agentes com papéis específicos (Teacher, Assistant, Classmates, Manager)
- Fluxo de aprendizagem em ambiente multiagente
- Ambiente "1 estudante + N agentes"
- Controlador de sessão decide quem fala, o que fala, quando fala
- Simulação de colegas (questionador, anotador, etc.)

### SINKT — Arquitetura de Knowledge Tracing

Grafo conceito-questão gerado via LLM
- conceito → conceito
- conceito → questão
- questão → conceito
Textual Information Encoder (PLM)
- extrai semântica de textos de questões e conceitos
Structural Information Encoder (GAT)
- propaga relações do grafo usando Graph Attention Networks
Student State Encoder (GRU)
- modela o domínio do aluno ao longo do tempo
Interaction Predictor
- combina estado do aluno + questão + conceitos
- prevê probabilidade de o aluno acertar a próxima questão

## Objetivo Pedagógico

MAIC - Ensino automático 
- Gera aulas completas, roteiros, agentes, interações.
- Resolve a parte instrucional.
SINKT - Diagnóstico de conhecimento
- Mede domínio conceitual.
- Previsão de desempenho individual.
- Resolve a parte avaliativa/analítica.

## MAIC

- LLM multimodal para leitura de slides.
- LLMs para geração de scripts.
- Agentes conversacionais impulsionados por LLM.

## SINKT

- LLMs constroem grafo conceito-questão.
- PLMs geram embeddings textuais.
- KT baseado em grafos + GRU.
- Uso de LLM é analítico e semântico, não conversacional.

## Estrutura de dados

MAIC
- Árvore de conhecimento extraída dos slides.
- Scripts e ações pedagógicas sequenciais.
- Diálogos multiagente.
SINKT
- Grafo com múltiplos tipos de arestas.
- Representações vetoriais de conceitos e questões.
- Sequências temporais armazenadas na GRU.

MAIC e SINKT são complementares, apesar da possibilidade de serem usados de forma isolada:
- MAIC automatiza o ensino
Cria aulas completas, interativas, personalizadas e escaláveis.
- SINKT automatiza a avaliação 
Detecta domínio conceitual e orienta o próximo passo do aprendizado.
Juntos, eles podem formar um ITS extremamente sofisticado:
MAIC ensina → SINKT mede → MAIC adapta → SINKT monitora (loop).








