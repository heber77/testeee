# Narrativa/Roteiro da Demo - Validação E2E

**Responsável**: Héber (LLM Pleno)

Este documento define a narrativa padrão da validação (roteiro da demo) para garantir que o valor pedagógico e técnico fique claro em menos de 10 minutos.

---

## Objetivo

Demonstrar que todo o pipeline Jedai funciona end-to-end, da ingestão de conteúdo até a inferência do modelo SINKT, em menos de 10 minutos.

### Estrutura da Demo

**Pipeline:**
> Validação completa do pipeline Jedai. Todas as etapas: ingestão de conteúdo, extração de dados, validação semântica no grafo, geração de eventos, construção do dataset e treino/inferência do modelo SINKT.

**Comando:**
```bash
make validate_e2e
```

---

#### Checkpoint 1: Extração

**O que acontece:**
- Ingestão de conteúdos (PDFs, vídeos, imagens, textos, avaliações)
- Extração de dados de cada tipo
- Persistência em `content_extractions`

**O que mostrar no terminal:**
```
================================================================================
CHECKPOINT 1/5: EXTRAÇÃO
================================================================================
✅ Extração: SUCESSO
   Tempo: ...
   Conteúdos ingeridos: 25
   Extrações geradas: 25
   Tipos processados: pdf, video, image, raw_text, assessment
   Dados persistidos: ✅
```

**Extração:**
> Validamos que conseguimos ingerir conteúdos reais e extrair dados deles. Aqui temos X conteúdos processados, incluindo PDFs, vídeos, imagens, textos e avaliações. Todos foram persistidos corretamente.


**Campos obrigatórios no log:**
- `contents_ingested`: Número de conteúdos ingeridos
- `extractions_generated`: Número de extrações geradas
- `types_processed`: Lista de tipos processados (ex: ["pdf", "video", "image", "raw_text", "assessment"])
- `data_persisted`: Boolean indicando se dados foram persistidos

---

#### Checkpoint 2: Grafo

**O que acontece:**
- Carregamento de conceitos do curso
- Validação de `concept_refs`
- Verificação de ciclos e referências inválidas

**O que mostrar no terminal:**
```
================================================================================
CHECKPOINT 2/5: GRAFO
================================================================================
✅ Grafo: SUCESSO
   Tempo: ...
   Conceitos carregados: 18
   concept_refs válidos: ✅
   Sem ciclos críticos: ✅
   Sem referências inválidas: ✅
   Curso: course_v1
```

**Grafo:**
> Validamos a integridade semântica. O grafo de conhecimento do curso tem X conceitos, todos com referências válidas, sem ciclos problemáticos. Isso garante que os dados são pedagogicamente coerentes.

**Campos obrigatórios no log:**
- `concepts_loaded`: Número de conceitos carregados
- `concept_refs_valid`: Boolean indicando se concept_refs são válidos
- `no_critical_cycles`: Boolean indicando ausência de ciclos críticos
- `no_invalid_references`: Boolean indicando ausência de referências inválidas
- `course_version_id`: ID da versão do curso

---

#### Checkpoint 3: Eventos

**O que acontece:**
- Processamento de avaliações
- Geração de eventos avaliativos
- Persistência no Jedai MS Progresso

**O que mostrar no terminal:**
```
================================================================================
CHECKPOINT 3/5: EVENTOS
================================================================================
✅ Eventos: SUCESSO
   Tempo: ...
   Avaliações processadas: 15
   Eventos gerados: 280
   Eventos persistidos: ✅
```

**Eventos:**
> Validamos que as avaliações geram eventos reais no progresso do aluno. Processamos X avaliações que resultaram em Y eventos, cada um registrando uma interação do aluno com um conceito. Todos foram persistidos no sistema de progresso.

**Campos obrigatórios no log:**
- `assessments_processed`: Número de avaliações processadas
- `events_generated`: Número de eventos gerados
- `events_persisted`: Boolean indicando se eventos foram persistidos

---

#### Checkpoint 4: Dataset

**O que acontece:**
- Carregamento de eventos do Progresso
- Construção de sequências temporais
- Validação do dataset

**O que mostrar no terminal:**
```
================================================================================
CHECKPOINT 4/5: DATASET
================================================================================
✅ Dataset: SUCESSO
   Tempo: ...
   Alunos no dataset: 12
   Eventos válidos: 280
   Sequências temporais: ✅
   Curso: course_v1
```

**Dataset:**
> O pipeline consumiu os eventos do Progresso e montou sequências temporais para X alunos. Cada sequência representa a jornada de aprendizado de um aluno ao longo do tempo, ordenada cronologicamente.

**Campos obrigatórios no log:**
- `students_in_dataset`: Número de alunos no dataset
- `valid_events`: Número de eventos válidos
- `temporal_sequences_built`: Boolean indicando se sequências foram construídas
- `course_version_id`: ID da versão do curso

**Importante:** Se um evento tem múltiplos `concept_refs` (ex: ["K01", "K02"]), criar múltiplas entradas na sequência, uma para cada conceito.

---

#### Checkpoint 5: SINKT

**O que acontece:**
- Treino do modelo SINKT
- Inferência para alunos existentes e novos
- Geração de relatório final

**O que mostrar no terminal:**
```
================================================================================
CHECKPOINT 5/5: SINKT
================================================================================
✅ SINKT: SUCESSO
   Tempo: ...
   Treino executado: ✅
   Épocas: 3
   Inferência realizada: ✅
   Relatório final gerado: ✅
```

**SINKT:**
> Treino do modelo SINKT com as sequências e inferências. O modelo aprendeu padrões de aprendizado e agora pode prever a probabilidade de um aluno acertar um conceito. O relatório final consolida todas as métricas.

**Campos obrigatórios no log:**
- `training_executed`: Boolean indicando se treino foi executado
- `epochs`: Número de épocas executadas
- `inference_successful`: Boolean indicando se inferência foi realizada
- `final_report_generated`: Boolean indicando se relatório foi gerado

**Importante:** Incluir exemplo de inferência para aluno novo (cold start) com 1-3 eventos.

---

#### Resumo Final

**O que mostrar no terminal:**
```
================================================================================
RESUMO DA VALIDAÇÃO E2E
================================================================================
Status geral: SUCCESS
Tempo total: ...

Checkpoints:
  ✅ Extração
  ✅ Grafo
  ✅ Eventos
  ✅ Dataset
  ✅ SINKT

Relatório gerado: jedai_e2e_validation_report.json
```

**Resumo:**
> "Todos os 5 checkpoints passaram com sucesso. O pipeline completo funcionou end-to-end, desde a ingestão de conteúdo até a inferência do modelo. O relatório final documenta todas as métricas e pode ser auditado.

---

### Tratamento de Erros

Se um checkpoint falhar:

**O que mostrar:**
```
❌ Grafo: ERRO
   Erro: concept_refs inválidos encontrados: K99, K100
   Tempo: ...

Validação interrompida. Corrija os erros e tente novamente.
```

**Regra:** Parar execução imediatamente, não continuar com checkpoints seguintes.

---

## Nota

As tarefas específicas de cada colega estão descritas na task G1 original. Este documento foca apenas na narrativa/roteiro da demo para guiar a implementação e garantir clareza pedagógica e técnica.

