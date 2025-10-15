# 🐳 Guia - Usando Docker Puro (sem docker-compose)

## 🎯 Sim, funciona perfeitamente sem docker-compose!

O `docker-compose` é apenas uma ferramenta de conveniência. Você pode usar Docker puro com os comandos `docker build` e `docker run`.

---

## 🚀 Passo a Passo

### **1. Build da Imagem**

```bash
cd D:\Downloads\podeapagar\HectorAI

docker build -t hectorai:latest .
```

**Opções úteis:**
```bash
# Build sem usar cache (força rebuild completo)
docker build --no-cache -t hectorai:latest .

# Build com tag específica
docker build -t hectorai:v2.0 .
```

---

### **2. Run do Container**

#### **Opção A: Modo Básico (com .env file)**

```bash
docker run -it --rm \
  --name hectorai \
  --env-file .env \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

#### **Opção B: Passando variáveis manualmente**

```bash
docker run -it --rm \
  --name hectorai \
  -e OPENAI_API_KEY="sk-sua-chave-aqui" \
  -e USER="seu_usuario_oracle" \
  -e SENHA="sua_senha_oracle" \
  -e DSN="host.docker.internal:1521/XEPDB1" \
  -e CHROMA_DIR="/app/chroma/chroma_db/" \
  -e ATTO_DIR="/app/AgentsAPI/rag/atto" \
  -e STREAMLIT_PORT=8501 \
  -e SKIP_CHROMA_REBUILD=false \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

#### **Opção C: Modo Desenvolvimento Rápido (pula recriação SQL)**

```bash
docker run -it --rm \
  --name hectorai \
  --env-file .env \
  -e SKIP_CHROMA_REBUILD=true \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

#### **Opção D: Com Volumes Persistentes**

```bash
docker run -it --rm \
  --name hectorai \
  --env-file .env \
  -v "%cd%\data\chroma_db:/app/chroma/chroma_db" \
  -v "%cd%\data\atto:/app/AgentsAPI/rag/atto" \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

**Nota:** No Linux/Mac, use `$(pwd)` em vez de `%cd%`

#### **Opção E: Modo Background (daemon)**

```bash
docker run -d \
  --name hectorai \
  --env-file .env \
  --restart unless-stopped \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

---

## 📋 Explicação dos Parâmetros

### **Flags Essenciais:**

| Flag | Descrição |
|------|-----------|
| `-it` | Modo interativo + TTY (ver logs em tempo real) |
| `--rm` | Remove container ao parar (não deixa lixo) |
| `--name hectorai` | Nome do container |
| `-d` | Roda em background (daemon mode) |
| `-p 8000:8000` | Mapeia porta 8000 (API) |
| `-p 8501:8501` | Mapeia porta 8501 (Streamlit) |

### **Variáveis de Ambiente:**

| Flag | Descrição |
|------|-----------|
| `--env-file .env` | Carrega variáveis do arquivo .env |
| `-e VARIAVEL=valor` | Define variável individual |

### **Volumes:**

| Flag | Descrição |
|------|-----------|
| `-v host:container` | Monta volume para persistir dados |

### **Restart Policy:**

| Flag | Descrição |
|------|-----------|
| `--restart unless-stopped` | Reinicia automaticamente |
| `--restart always` | Sempre reinicia |
| `--restart on-failure` | Reinicia apenas se falhar |

---

## 🎛️ Comandos Úteis

### **Ver containers rodando:**
```bash
docker ps
```

### **Ver logs em tempo real:**
```bash
docker logs -f hectorai
```

### **Ver logs das últimas 100 linhas:**
```bash
docker logs --tail 100 hectorai
```

### **Parar container:**
```bash
docker stop hectorai
```

### **Reiniciar container:**
```bash
docker restart hectorai
```

### **Remover container:**
```bash
# Para e remove
docker rm -f hectorai
```

### **Entrar no container (debug):**
```bash
docker exec -it hectorai bash
```

### **Ver uso de recursos:**
```bash
docker stats hectorai
```

### **Inspecionar container:**
```bash
docker inspect hectorai
```

### **Ver imagens disponíveis:**
```bash
docker images
```

### **Remover imagem:**
```bash
docker rmi hectorai:latest
```

---

## 🔧 Cenários Práticos

### **Cenário 1: Desenvolvimento Local (Interativo)**
```bash
# Build
docker build -t hectorai:latest .

# Run (ver logs em tempo real)
docker run -it --rm \
  --name hectorai \
  --env-file .env \
  -e SKIP_CHROMA_REBUILD=true \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

### **Cenário 2: Produção (Background)**
```bash
# Build
docker build -t hectorai:latest .

# Run em background com restart automático
docker run -d \
  --name hectorai \
  --env-file .env \
  --restart unless-stopped \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest

# Acompanhar logs
docker logs -f hectorai
```

### **Cenário 3: Teste Rápido**
```bash
# Build e run em uma linha
docker build -t hectorai:latest . && \
docker run -it --rm \
  --name hectorai \
  --env-file .env \
  -e SKIP_CHROMA_REBUILD=true \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

### **Cenário 4: Com Volumes (Dados Persistentes)**
```bash
# Windows (PowerShell)
docker run -it --rm `
  --name hectorai `
  --env-file .env `
  -v "${PWD}\data\chroma_db:/app/chroma/chroma_db" `
  -v "${PWD}\data\atto:/app/AgentsAPI/rag/atto" `
  -p 8000:8000 `
  -p 8501:8501 `
  hectorai:latest

# Linux/Mac (Bash)
docker run -it --rm \
  --name hectorai \
  --env-file .env \
  -v "$(pwd)/data/chroma_db:/app/chroma/chroma_db" \
  -v "$(pwd)/data/atto:/app/AgentsAPI/rag/atto" \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

### **Cenário 5: Rebuild Forçado**
```bash
# Para container existente
docker stop hectorai 2>/dev/null
docker rm hectorai 2>/dev/null

# Remove imagem antiga
docker rmi hectorai:latest

# Build do zero
docker build --no-cache -t hectorai:latest .

# Run
docker run -it --rm \
  --name hectorai \
  --env-file .env \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

---

## 🐧 Scripts Auxiliares

### **Windows (PowerShell):**

Crie `start.ps1`:
```powershell
# start.ps1
Write-Host "🚀 Iniciando HectorAI..." -ForegroundColor Green

# Para container existente
docker stop hectorai 2>$null
docker rm hectorai 2>$null

# Build
Write-Host "📦 Building imagem..." -ForegroundColor Yellow
docker build -t hectorai:latest .

# Run
Write-Host "▶️  Iniciando container..." -ForegroundColor Yellow
docker run -it --rm `
  --name hectorai `
  --env-file .env `
  -p 8000:8000 `
  -p 8501:8501 `
  hectorai:latest
```

Execute:
```powershell
.\start.ps1
```

### **Linux/Mac (Bash):**

Crie `start.sh`:
```bash
#!/bin/bash
# start.sh

echo "🚀 Iniciando HectorAI..."

# Para container existente
docker stop hectorai 2>/dev/null
docker rm hectorai 2>/dev/null

# Build
echo "📦 Building imagem..."
docker build -t hectorai:latest .

# Run
echo "▶️  Iniciando container..."
docker run -it --rm \
  --name hectorai \
  --env-file .env \
  -p 8000:8000 \
  -p 8501:8501 \
  hectorai:latest
```

Dê permissão e execute:
```bash
chmod +x start.sh
./start.sh
```

---

## ⚡ Atalhos Rápidos

### **Build + Run (uma linha):**
```bash
# Windows (PowerShell)
docker build -t hectorai:latest . ; docker run -it --rm --name hectorai --env-file .env -p 8000:8000 -p 8501:8501 hectorai:latest

# Linux/Mac (Bash)
docker build -t hectorai:latest . && docker run -it --rm --name hectorai --env-file .env -p 8000:8000 -p 8501:8501 hectorai:latest
```

### **Stop + Remove + Rebuild + Run:**
```bash
# Windows (PowerShell)
docker stop hectorai ; docker rm hectorai ; docker build -t hectorai:latest . ; docker run -it --rm --name hectorai --env-file .env -p 8000:8000 -p 8501:8501 hectorai:latest

# Linux/Mac (Bash)
docker stop hectorai && docker rm hectorai && docker build -t hectorai:latest . && docker run -it --rm --name hectorai --env-file .env -p 8000:8000 -p 8501:8501 hectorai:latest
```

---

## 🆚 Docker vs Docker Compose

### **Docker Puro:**
✅ Mais controle direto  
✅ Não precisa instalar docker-compose  
✅ Bom para entender o que está acontecendo  
❌ Comandos mais longos  
❌ Mais difícil gerenciar múltiplos serviços  

### **Docker Compose:**
✅ Comandos mais curtos  
✅ Configuração declarativa (YAML)  
✅ Fácil gerenciar múltiplos serviços  
✅ Networking automático entre serviços  
❌ Precisa instalar docker-compose  
❌ Camada extra de abstração  

**Recomendação:**
- **Desenvolvimento solo**: Use Docker puro
- **Múltiplos serviços**: Use Docker Compose
- **Produção**: Depende da infraestrutura (Docker Swarm, Kubernetes, etc)

---

## 🔍 Troubleshooting

### **Erro: "port is already allocated"**
```bash
# Encontre o que está usando a porta
netstat -ano | findstr :8000

# Ou use outra porta
docker run -p 8080:8000 -p 8502:8501 ...
```

### **Erro: "no such file or directory (.env)"**
```bash
# Verifique se .env existe
ls -la .env

# Ou passe variáveis manualmente
docker run -e OPENAI_API_KEY="..." -e USER="..." ...
```

### **Container para sozinho**
```bash
# Veja os logs
docker logs hectorai

# Veja por que parou
docker inspect hectorai | grep -A 10 "State"
```

### **Problemas de conexão Oracle**
```bash
# Se Oracle está em localhost, use:
-e DSN="host.docker.internal:1521/XEPDB1"
```

---

## 📚 Arquivos Necessários

Para rodar com Docker puro, você precisa:

```
HectorAI/
├── Dockerfile                    ✅ Obrigatório
├── docker/
│   └── entrypoint.sh            ✅ Obrigatório
├── .env                          ✅ Obrigatório (ou passe variáveis via -e)
├── requirements.txt              ✅ Obrigatório
├── main.py                       ✅ Obrigatório
├── AgentsAPI/
│   └── rag/
│       └── atto_rag.json        ✅ Obrigatório
└── chroma/
    ├── ddl_vanna_oracle.txt     ✅ Obrigatório
    ├── documentation_bd_atual.txt ✅ Obrigatório
    └── sql_oracle.json          ✅ Obrigatório
```

---

## 🎉 Pronto!

Agora você pode usar **Docker puro** sem precisar do docker-compose! 

Escolha o comando que melhor se adapta ao seu cenário e seja feliz! 🚀

**Dica:** Salve seus comandos favoritos em um script (`.ps1` no Windows ou `.sh` no Linux/Mac) para facilitar o uso diário.

