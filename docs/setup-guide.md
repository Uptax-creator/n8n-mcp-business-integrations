# 🚀 Setup Guide - N8N + MCP Business Integrations

## 📋 **PRÉ-REQUISITOS**

### **🖥️ Sistema**
```yaml
requirements:
  os: "Windows 10+, macOS 10.15+, Ubuntu 18.04+"
  node: "16.0+ (recomendado 18.x)"
  python: "3.8+ (para MCP servers)"
  memory: "4GB RAM mínimo, 8GB recomendado"
  storage: "2GB disponível"
```

### **🔧 Ferramentas Necessárias**
```bash
# Node.js e npm
curl -fsSL https://nodejs.org/dist/v18.17.0/node-v18.17.0-linux-x64.tar.xz

# Python 3.8+
python --version

# Git
git --version
```

---

## 🎯 **INSTALAÇÃO PASSO A PASSO**

### **1️⃣ Instalar N8N**

#### **Opção A: Instalação Global (Recomendada)**
```bash
# Instalar N8N globalmente
npm install -g n8n

# Verificar instalação
n8n --version
```

#### **Opção B: Docker (Alternativa)**
```bash
# Executar N8N via Docker
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

### **2️⃣ Configurar MCP Servers**

#### **Instalar Dependências Python**
```bash
# Criar ambiente virtual
python -m venv mcp-env
source mcp-env/bin/activate  # Linux/macOS
# mcp-env\Scripts\activate   # Windows

# Instalar packages MCP
pip install fastmcp anthropic requests aiohttp
```

#### **Clonar MCP Servers Existentes**
```bash
# Clonar servidor Omie (se disponível)
git clone https://github.com/seu-user/omie-mcp-server.git
cd omie-mcp-server
pip install -r requirements.txt

# Iniciar servidor MCP
python omie_mcp_server.py
```

### **3️⃣ Clonar Este Repositório**
```bash
# Clonar workflows
git clone https://github.com/Uptax-creator/n8n-mcp-business-integrations.git
cd n8n-mcp-business-integrations
```

---

## ⚙️ **CONFIGURAÇÃO INICIAL**

### **1️⃣ Iniciar N8N**
```bash
# Primeira execução
n8n start

# N8N será iniciado em: http://localhost:5678
# Criar conta admin na primeira execução
```

### **2️⃣ Configurar Credenciais**

#### **Credenciais API Omie**
```json
{
  "name": "Omie API",
  "type": "httpHeaderAuth", 
  "data": {
    "name": "Authorization",
    "value": "Bearer SEU_TOKEN_OMIE"
  }
}
```

#### **Credenciais MCP Server**
```json
{
  "name": "MCP Local Server",
  "type": "httpBasicAuth",
  "data": {
    "user": "admin",
    "password": "sua_senha_mcp"
  }
}
```

### **3️⃣ Importar Workflows**

#### **Via Interface N8N**
1. Acesse N8N: http://localhost:5678
2. Clique em **"Settings"** > **"Import/Export"**
3. Selecione **"Import workflow"**
4. Escolha arquivo `.json` da pasta `workflows/`
5. Clique **"Import"**

#### **Via Linha de Comando**
```bash
# Importar workflow específico
n8n import:workflow --input=./workflows/01-omie-data-sync.json

# Importar múltiplos workflows
for file in ./workflows/*.json; do
  n8n import:workflow --input="$file"
done
```

---

## 🔧 **CONFIGURAÇÃO AVANÇADA**

### **1️⃣ Variáveis de Ambiente**
```bash
# Criar arquivo .env na pasta raiz do N8N
cat > ~/.n8n/.env << EOF
# Configurações N8N
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=sua_senha_segura

# Configurações MCP
MCP_SERVER_OMIE_URL=http://localhost:3001
MCP_SERVER_AI_URL=http://localhost:3002

# Configurações Database
DATABASE_TYPE=postgres
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=n8n_db
DATABASE_USER=n8n_user
DATABASE_PASSWORD=n8n_password

# Configurações de Execução
EXECUTIONS_PROCESS=main
EXECUTIONS_TIMEOUT=3600
EXECUTIONS_TIMEOUT_MAX=7200
EOF
```

### **2️⃣ Configuração PostgreSQL (Opcional)**
```bash
# Para produção, usar PostgreSQL
npm install pg

# String de conexão
DATABASE_POSTGRESDB_HOST=localhost
DATABASE_POSTGRESDB_PORT=5432
DATABASE_POSTGRESDB_DATABASE=n8n
DATABASE_POSTGRESDB_USER=n8n
DATABASE_POSTGRESDB_PASSWORD=n8n
```

### **3️⃣ Configuração SSL/HTTPS**
```bash
# Para produção com HTTPS
N8N_PROTOCOL=https
N8N_SSL_KEY=/path/to/private.key
N8N_SSL_CERT=/path/to/certificate.crt
```

---

## 🧪 **TESTE DE INSTALAÇÃO**

### **1️⃣ Verificar N8N**
```bash
# Acessar interface
curl http://localhost:5678/healthz
# Deve retornar: {"status":"ok"}
```

### **2️⃣ Testar MCP Servers**
```bash
# Testar servidor Omie
curl http://localhost:3001/health
# Deve retornar: {"status":"healthy","tools":["consultar_clientes",...]}

# Testar servidor AI
curl http://localhost:3002/health  
# Deve retornar: {"status":"healthy","tools":["extract_info",...]}
```

### **3️⃣ Executar Workflow de Teste**
```bash
# Via N8N CLI
n8n execute --id=workflow_id

# Ou criar workflow de teste simples
```

---

## 🐛 **TROUBLESHOOTING**

### **❌ Problemas Comuns**

#### **Erro: "N8N command not found"**
```bash
# Verificar instalação
npm list -g n8n

# Reinstalar se necessário
npm uninstall -g n8n
npm install -g n8n
```

#### **Erro: "MCP Server Connection Failed"**
```bash
# Verificar se servidor está rodando
ps aux | grep python | grep mcp

# Verificar porta
netstat -tulpn | grep :3001

# Verificar logs
tail -f mcp-server.log
```

#### **Erro: "Database Connection Failed"**
```bash
# Usar SQLite para desenvolvimento
rm ~/.n8n/database.sqlite
n8n start

# Verificar permissões PostgreSQL
pg_isready -h localhost -p 5432
```

### **🔍 Logs e Debugging**
```bash
# Logs N8N detalhados
N8N_LOG_LEVEL=debug n8n start

# Logs específicos de workflow
tail -f ~/.n8n/logs/n8n.log

# Debug MCP communication
export DEBUG=mcp:*
python mcp_server.py
```

---

## 📊 **MONITORAMENTO E MANUTENÇÃO**

### **1️⃣ Health Checks**
```bash
# Script de monitoramento
#!/bin/bash
echo "Checking N8N..."
curl -s http://localhost:5678/healthz || echo "N8N DOWN"

echo "Checking MCP Servers..."
curl -s http://localhost:3001/health || echo "Omie MCP DOWN"
curl -s http://localhost:3002/health || echo "AI MCP DOWN"
```

### **2️⃣ Backup Workflows**
```bash
# Exportar todos workflows
n8n export:workflow --output=./backup/

# Backup configurações
cp -r ~/.n8n ./backup/n8n-config-$(date +%Y%m%d)
```

### **3️⃣ Updates**
```bash
# Atualizar N8N
npm update -g n8n

# Atualizar MCP servers
cd omie-mcp-server
git pull origin main
pip install -r requirements.txt --upgrade
```

---

## 🚀 **PRÓXIMOS PASSOS**

Após a instalação bem-sucedida:

1. **📖 Leia**: [use-cases.md](use-cases.md) para entender cenários práticos
2. **🧪 Teste**: Workflows de exemplo na pasta `examples/`
3. **🔧 Customize**: Adapte workflows para suas necessidades
4. **📊 Monitore**: Configure alertas e métricas
5. **🤝 Contribua**: Compartilhe seus workflows personalizados

---

## 📞 **SUPORTE**

- **🐛 Problemas**: [GitHub Issues](https://github.com/Uptax-creator/n8n-mcp-business-integrations/issues)
- **❓ Dúvidas**: [GitHub Discussions](https://github.com/Uptax-creator/n8n-mcp-business-integrations/discussions)
- **📧 Email**: suporte@uptax-creator.com

**✅ Setup concluído! Você está pronto para automatizar integrações empresariais com N8N + MCP.**