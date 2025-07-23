# 🔗 MCP Integration Guide - Model Context Protocol + N8N

## 🎯 **O QUE É MCP (Model Context Protocol)**

### **📋 Conceitos Fundamentais**
```yaml
mcp_definition:
  purpose: "Protocolo para integração entre LLMs e ferramentas externas"
  developer: "Anthropic"
  version: "1.0"
  architecture: "Client-Server com JSON-RPC"

key_benefits:
  - standardized_communication: "Protocolo padrão para tools"
  - security: "Sandboxed execution environment"
  - scalability: "Multiple tools per server"
  - flexibility: "Custom tool development"
  - reliability: "Error handling e retry logic"
```

### **🏗️ Arquitetura MCP**
```mermaid
graph LR
    A[LLM Client] --> B[MCP Protocol]
    B --> C[MCP Server]
    C --> D[Business Tools]
    D --> E[External APIs]
    
    F[N8N Workflow] --> B
    C --> G[Database]
    C --> H[File System]
```

---

## 🔧 **MCP SERVER DEVELOPMENT**

### **📦 Estrutura Básica do Servidor**
```python
# mcp_server_template.py
from fastmcp import FastMCP, Context
from typing import List, Dict, Any
import asyncio
import json

# Inicializar servidor MCP
mcp = FastMCP("Business Integration Server")

@mcp.tool()
async def analyze_api_documentation(
    context: Context,
    documentation_url: str,
    analysis_depth: str = "standard"
) -> Dict[str, Any]:
    """
    Analisa documentação de API e extrai informações estruturadas
    
    Args:
        documentation_url: URL da documentação API
        analysis_depth: Nível de análise (basic|standard|detailed)
    
    Returns:
        Estrutura de integração com endpoints, complexidade, relacionamentos
    """
    
    # 1. Fetch documentation
    doc_content = await fetch_documentation(documentation_url)
    
    # 2. AI Analysis
    analysis_result = await ai_analyze_content(doc_content, analysis_depth)
    
    # 3. Structure data
    structured_data = {
        "integration_name": analysis_result.get("name"),
        "provider": analysis_result.get("provider"),
        "category": classify_integration_category(analysis_result),
        "endpoints": analysis_result.get("endpoints", []),
        "authentication": analysis_result.get("auth_method"),
        "complexity_score": calculate_complexity(analysis_result),
        "story_points": estimate_story_points(analysis_result),
        "dependencies": discover_dependencies(analysis_result),
        "quality_score": assess_documentation_quality(doc_content)
    }
    
    return structured_data

@mcp.tool()
async def sync_erp_data(
    context: Context,
    source_erp: str,
    target_erp: str,
    entity_type: str,
    batch_size: int = 100
) -> Dict[str, Any]:
    """
    Sincroniza dados entre diferentes ERPs
    
    Args:
        source_erp: ERP origem (omie|nibo|sap)
        target_erp: ERP destino (omie|nibo|sap) 
        entity_type: Tipo de entidade (clients|products|orders)
        batch_size: Tamanho do lote para processamento
    
    Returns:
        Resultado da sincronização com estatísticas
    """
    
    # 1. Get source data
    source_data = await get_erp_data(source_erp, entity_type, batch_size)
    
    # 2. Transform data
    transformed_data = await transform_data(
        source_data, 
        source_erp, 
        target_erp, 
        entity_type
    )
    
    # 3. Sync to target
    sync_result = await sync_to_target_erp(
        target_erp, 
        entity_type, 
        transformed_data
    )
    
    return {
        "sync_id": sync_result.get("id"),
        "source_records": len(source_data),
        "synced_records": sync_result.get("synced_count"),
        "failed_records": sync_result.get("failed_count"),
        "errors": sync_result.get("errors", []),
        "execution_time": sync_result.get("duration"),
        "next_sync_available": sync_result.get("next_available_at")
    }

# Configuração do servidor
if __name__ == "__main__":
    import uvicorn
    
    # Health check endpoint
    @mcp.get("/health")
    async def health_check():
        return {
            "status": "healthy",
            "server": "Business Integration MCP",
            "version": "1.0.0",
            "tools": [
                "analyze_api_documentation",
                "sync_erp_data",
                "build_integration_graph",
                "assess_integration_quality"
            ]
        }
    
    # Executar servidor
    uvicorn.run(
        "mcp_server:mcp.app",
        host="localhost",
        port=3001,
        reload=True
    )
```

### **🔧 Configuração para N8N**
```json
{
  "mcp_server_config": {
    "name": "business-integration-mcp",
    "url": "http://localhost:3001",
    "timeout": 30000,
    "retry_attempts": 3,
    "tools": [
      {
        "name": "analyze_api_documentation",
        "description": "Analyze API docs and extract integration info",
        "parameters": {
          "documentation_url": {
            "type": "string",
            "required": true,
            "description": "URL of API documentation"
          },
          "analysis_depth": {
            "type": "string",
            "enum": ["basic", "standard", "detailed"],
            "default": "standard"
          }
        }
      }
    ]
  }
}
```

---

## 🔄 **INTEGRAÇÃO N8N + MCP**

### **📊 Workflow Pattern: MCP Tool Call**
```json
{
  "workflow_name": "AI API Analysis via MCP",
  "nodes": [
    {
      "name": "Webhook Trigger",
      "type": "n8n-nodes-base.webhook",
      "parameters": {
        "path": "analyze-api",
        "httpMethod": "POST"
      }
    },
    {
      "name": "Validate Input",
      "type": "n8n-nodes-base.function",
      "parameters": {
        "functionCode": "// Validate required fields\nconst input = items[0].json;\n\nif (!input.documentation_url) {\n  throw new Error('documentation_url is required');\n}\n\nif (!input.documentation_url.startsWith('http')) {\n  throw new Error('documentation_url must be a valid URL');\n}\n\nreturn items;"
      }
    },
    {
      "name": "Call MCP Server",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "http://localhost:3001/call",
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "method": "analyze_api_documentation",
          "params": {
            "documentation_url": "={{ $json.documentation_url }}",
            "analysis_depth": "={{ $json.analysis_depth || 'standard' }}"
          }
        },
        "options": {
          "timeout": 60000,
          "retry": {
            "enabled": true,
            "maxRetries": 3
          }
        }
      }
    },
    {
      "name": "Process MCP Response",
      "type": "n8n-nodes-base.function",
      "parameters": {
        "functionCode": "// Process MCP server response\nconst response = items[0].json;\n\nif (response.error) {\n  throw new Error(`MCP Error: ${response.error.message}`);\n}\n\nconst result = response.result;\n\n// Enrich with metadata\nresult.analyzed_at = new Date().toISOString();\nresult.workflow_id = $workflow.id;\nresult.execution_id = $execution.id;\n\nreturn [{ json: result }];"
      }
    },
    {
      "name": "Save to Database",
      "type": "n8n-nodes-base.postgres",
      "parameters": {
        "operation": "insert",
        "table": "integrations",
        "columns": "name,provider,category,complexity_score,story_points,quality_score,raw_data,created_at",
        "values": "={{ $json.integration_name }},={{ $json.provider }},={{ $json.category }},={{ $json.complexity_score }},={{ $json.story_points }},={{ $json.quality_score }},={{ JSON.stringify($json) }},NOW()"
      }
    },
    {
      "name": "Update Neo4j Graph",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "http://localhost:7474/db/data/cypher",
        "headers": {
          "Content-Type": "application/json",
          "Authorization": "Basic bmVvNGo6MTA5UkhTTDZyODFG"
        },
        "body": {
          "query": "MERGE (i:Integration {name: $name}) SET i.provider = $provider, i.category = $category, i.complexity_score = $complexity_score, i.story_points = $story_points, i.updated_at = datetime()",
          "params": {
            "name": "={{ $json.integration_name }}",
            "provider": "={{ $json.provider }}",
            "category": "={{ $json.category }}",
            "complexity_score": "={{ $json.complexity_score }}",
            "story_points": "={{ $json.story_points }}"
          }
        }
      }
    },
    {
      "name": "Return Success Response",
      "type": "n8n-nodes-base.respondToWebhook",
      "parameters": {
        "responseBody": {
          "success": true,
          "integration": "={{ $json.integration_name }}",
          "complexity_score": "={{ $json.complexity_score }}",
          "story_points": "={{ $json.story_points }}",
          "analysis_id": "={{ $json.execution_id }}"
        }
      }
    }
  ],
  "error_handling": {
    "global_error_handler": {
      "name": "Handle Errors",
      "type": "n8n-nodes-base.function",
      "parameters": {
        "functionCode": "// Log error details\nconst error = $input.first().error;\nconsole.error('Workflow error:', error);\n\n// Return error response\nreturn [{\n  json: {\n    success: false,\n    error: error.message,\n    timestamp: new Date().toISOString(),\n    workflow_id: $workflow.id,\n    execution_id: $execution.id\n  }\n}];"
      }
    }
  }
}
```

---

## 🤖 **ADVANCED MCP PATTERNS**

### **🔄 Pattern 1: Intelligent Data Sync**
```python
@mcp.tool()
async def intelligent_data_sync(
    context: Context,
    sync_config: Dict[str, Any]
) -> Dict[str, Any]:
    """
    Sincronização inteligente usando AI para resolver conflitos
    """
    
    # 1. Get data from both sources
    source_data = await get_data(sync_config["source"])
    target_data = await get_data(sync_config["target"])
    
    # 2. AI-powered conflict resolution
    conflicts = detect_conflicts(source_data, target_data)
    
    if conflicts:
        # Use LLM to resolve conflicts
        resolution_strategy = await ai_resolve_conflicts(
            conflicts, 
            sync_config.get("business_rules", {})
        )
        
        # Apply resolution
        resolved_data = apply_resolution(
            source_data, 
            target_data, 
            resolution_strategy
        )
    else:
        resolved_data = merge_data(source_data, target_data)
    
    # 3. Execute sync
    sync_result = await execute_sync(resolved_data, sync_config)
    
    return {
        "sync_status": "completed",
        "records_processed": len(resolved_data),
        "conflicts_resolved": len(conflicts),
        "ai_decisions": len(resolution_strategy) if conflicts else 0,
        "execution_time": sync_result["duration"]
    }

async def detect_conflicts(source_data, target_data):
    """Detecta conflitos entre datasets"""
    conflicts = []
    
    for source_record in source_data:
        target_record = find_matching_record(source_record, target_data)
        
        if target_record:
            differences = compare_records(source_record, target_record)
            if differences:
                conflicts.append({
                    "source": source_record,
                    "target": target_record,
                    "differences": differences,
                    "conflict_type": classify_conflict_type(differences)
                })
    
    return conflicts

async def ai_resolve_conflicts(conflicts, business_rules):
    """Usa AI para resolver conflitos automaticamente"""
    resolution_strategies = []
    
    for conflict in conflicts:
        # Prepare context for AI
        ai_context = {
            "conflict": conflict,
            "business_rules": business_rules,
            "conflict_history": get_similar_conflict_resolutions(conflict)
        }
        
        # Get AI recommendation
        ai_prompt = f"""
        Analyze this data conflict and suggest resolution:
        
        Conflict: {conflict['differences']}
        Business Rules: {business_rules}
        Similar Cases: {ai_context['conflict_history']}
        
        Provide resolution strategy with confidence score.
        """
        
        ai_response = await call_llm(ai_prompt)
        
        resolution_strategies.append({
            "conflict_id": conflict["id"],
            "strategy": ai_response["strategy"],
            "confidence": ai_response["confidence"],
            "reasoning": ai_response["reasoning"]
        })
    
    return resolution_strategies
```

### **🕸️ Pattern 2: Graph Relationship Discovery**
```python
@mcp.tool()
async def discover_integration_relationships(
    context: Context,
    integration_data: List[Dict[str, Any]]
) -> Dict[str, Any]:
    """
    Descobre relacionamentos entre integrações usando AI + Graph analysis
    """
    
    # 1. Analyze each integration
    analyzed_integrations = []
    for integration in integration_data:
        analysis = await ai_analyze_integration_context(integration)
        analyzed_integrations.append(analysis)
    
    # 2. Find potential relationships
    relationships = []
    for i, int1 in enumerate(analyzed_integrations):
        for j, int2 in enumerate(analyzed_integrations[i+1:], i+1):
            relationship = await ai_discover_relationship(int1, int2)
            if relationship["confidence"] > 0.7:
                relationships.append({
                    "source": int1["name"],
                    "target": int2["name"],
                    "type": relationship["type"],
                    "confidence": relationship["confidence"],
                    "reasoning": relationship["reasoning"]
                })
    
    # 3. Build graph structure
    graph_data = build_graph_structure(analyzed_integrations, relationships)
    
    # 4. Update Neo4j
    await update_neo4j_graph(graph_data)
    
    return {
        "integrations_analyzed": len(analyzed_integrations),
        "relationships_discovered": len(relationships),
        "graph_nodes": len(graph_data["nodes"]),
        "graph_edges": len(graph_data["edges"]),
        "discovery_confidence": calculate_average_confidence(relationships)
    }

async def ai_analyze_integration_context(integration):
    """Analisa contexto de uma integração usando AI"""
    
    ai_prompt = f"""
    Analyze this integration and extract key context:
    
    Integration: {integration}
    
    Extract:
    1. Primary business domain
    2. Data entities handled
    3. Integration patterns used
    4. Dependency indicators
    5. Common workflow usage
    """
    
    ai_response = await call_llm(ai_prompt)
    
    return {
        **integration,
        "business_domain": ai_response["domain"],
        "data_entities": ai_response["entities"],
        "integration_patterns": ai_response["patterns"],
        "dependencies": ai_response["dependencies"],
        "workflow_context": ai_response["workflow_usage"]
    }

async def ai_discover_relationship(int1, int2):
    """Descobre relacionamento entre duas integrações"""
    
    ai_prompt = f"""
    Analyze these two integrations and determine their relationship:
    
    Integration 1: {int1}
    Integration 2: {int2}
    
    Determine:
    1. Relationship type (DEPENDS_ON, FEEDS_INTO, COMPATIBLE_WITH, CONFLICTS_WITH, REPLACES)
    2. Confidence level (0.0 to 1.0)
    3. Reasoning for the relationship
    4. Suggested workflow patterns
    """
    
    ai_response = await call_llm(ai_prompt)
    
    return {
        "type": ai_response["relationship_type"],
        "confidence": ai_response["confidence"],
        "reasoning": ai_response["reasoning"],
        "workflow_suggestions": ai_response["workflow_patterns"]
    }
```

---

## 📊 **MCP PERFORMANCE OPTIMIZATION**

### **⚡ Caching Strategies**
```python
from functools import lru_cache
import redis
import json
import hashlib

# Redis para cache distribuído
redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_key(func_name: str, *args, **kwargs) -> str:
    """Gera chave única para cache"""
    key_data = f"{func_name}:{str(args)}:{str(sorted(kwargs.items()))}"
    return hashlib.md5(key_data.encode()).hexdigest()

def cached_mcp_tool(ttl: int = 3600):
    """Decorator para cache de ferramentas MCP"""
    def decorator(func):
        async def wrapper(*args, **kwargs):
            # Gerar chave de cache
            cache_key_str = cache_key(func.__name__, *args, **kwargs)
            
            # Tentar buscar no cache
            cached_result = redis_client.get(cache_key_str)
            if cached_result:
                return json.loads(cached_result)
            
            # Executar função
            result = await func(*args, **kwargs)
            
            # Salvar no cache
            redis_client.setex(
                cache_key_str, 
                ttl, 
                json.dumps(result, default=str)
            )
            
            return result
        return wrapper
    return decorator

# Exemplo de uso
@mcp.tool()
@cached_mcp_tool(ttl=1800)  # Cache por 30 minutos
async def get_api_documentation(
    context: Context,
    api_url: str
) -> Dict[str, Any]:
    """Cached API documentation retrieval"""
    # Expensive operation - fetch and parse documentation
    return await expensive_api_documentation_fetch(api_url)
```

### **🔄 Connection Pooling**
```python
import aiohttp
import asyncio
from aiohttp_session import setup
from aiohttp_session.cookie_storage import EncryptedCookieStorage

class MCPConnectionPool:
    """Pool de conexões para otimizar performance"""
    
    def __init__(self, max_connections: int = 100):
        self.max_connections = max_connections
        self.connector = None
        self.session = None
    
    async def initialize(self):
        """Inicializa pool de conexões"""
        self.connector = aiohttp.TCPConnector(
            limit=self.max_connections,
            limit_per_host=20,
            keepalive_timeout=30
        )
        
        self.session = aiohttp.ClientSession(
            connector=self.connector,
            timeout=aiohttp.ClientTimeout(total=30)
        )
    
    async def close(self):
        """Fecha pool de conexões"""
        if self.session:
            await self.session.close()
        if self.connector:
            await self.connector.close()
    
    async def make_request(self, method: str, url: str, **kwargs):
        """Faz requisição usando pool"""
        if not self.session:
            await self.initialize()
        
        async with self.session.request(method, url, **kwargs) as response:
            return await response.json()

# Instância global do pool
connection_pool = MCPConnectionPool()

@mcp.tool()
async def optimized_api_call(
    context: Context,
    api_endpoint: str,
    method: str = "GET",
    **kwargs
) -> Dict[str, Any]:
    """API call otimizada com connection pooling"""
    
    try:
        result = await connection_pool.make_request(
            method, 
            api_endpoint, 
            **kwargs
        )
        return result
        
    except asyncio.TimeoutError:
        raise Exception(f"API call timeout: {api_endpoint}")
    except Exception as e:
        raise Exception(f"API call failed: {str(e)}")
```

---

## 🔐 **SECURITY BEST PRACTICES**

### **🗝️ Authentication & Authorization**
```python
import jwt
import bcrypt
from datetime import datetime, timedelta
from functools import wraps

class MCPSecurityManager:
    """Gerenciamento de segurança para MCP servers"""
    
    def __init__(self, secret_key: str):
        self.secret_key = secret_key
        self.authorized_clients = set()
    
    def generate_token(self, client_id: str, permissions: List[str]) -> str:
        """Gera JWT token para cliente"""
        payload = {
            "client_id": client_id,
            "permissions": permissions,
            "exp": datetime.utcnow() + timedelta(hours=24),
            "iat": datetime.utcnow()
        }
        return jwt.encode(payload, self.secret_key, algorithm="HS256")
    
    def verify_token(self, token: str) -> Dict[str, Any]:
        """Verifica e decodifica JWT token"""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=["HS256"])
            return payload
        except jwt.ExpiredSignatureError:
            raise Exception("Token expired")
        except jwt.InvalidTokenError:
            raise Exception("Invalid token")
    
    def require_permission(self, required_permission: str):
        """Decorator para verificar permissões"""
        def decorator(func):
            @wraps(func)
            async def wrapper(context: Context, *args, **kwargs):
                # Extrair token do contexto
                auth_header = context.headers.get("Authorization", "")
                if not auth_header.startswith("Bearer "):
                    raise Exception("Missing or invalid authorization header")
                
                token = auth_header.split(" ")[1]
                payload = self.verify_token(token)
                
                # Verificar permissão
                if required_permission not in payload.get("permissions", []):
                    raise Exception(f"Insufficient permissions: {required_permission}")
                
                # Adicionar info do cliente ao contexto
                context.client_id = payload["client_id"]
                context.permissions = payload["permissions"]
                
                return await func(context, *args, **kwargs)
            return wrapper
        return decorator

# Instância global do security manager
security = MCPSecurityManager("your-secret-key-here")

@mcp.tool()
@security.require_permission("read:integrations")
async def get_integration_data(
    context: Context,
    integration_id: str
) -> Dict[str, Any]:
    """Ferramenta protegida que requer autenticação"""
    
    # Log access
    logger.info(f"Client {context.client_id} accessed integration {integration_id}")
    
    # Implementar lógica da ferramenta
    return await fetch_integration_data(integration_id)
```

### **🛡️ Input Sanitization**
```python
from typing import Any, Dict, List, Union
import re
import html

class InputSanitizer:
    """Sanitização de inputs para ferramentas MCP"""
    
    @staticmethod
    def sanitize_url(url: str) -> str:
        """Sanitiza URLs para prevenir ataques"""
        if not url.startswith(('http://', 'https://')):
            raise ValueError("Invalid URL protocol")
        
        # Remove caracteres perigosos
        sanitized = re.sub(r'[<>"\']', '', url)
        
        # Validar formato básico
        if not re.match(r'^https?://[^\s/$.?#].[^\s]*$', sanitized):
            raise ValueError("Invalid URL format")
        
        return sanitized
    
    @staticmethod
    def sanitize_string(text: str, max_length: int = 1000) -> str:
        """Sanitiza strings de entrada"""
        if len(text) > max_length:
            raise ValueError(f"String too long (max {max_length})")
        
        # HTML escape
        sanitized = html.escape(text)
        
        # Remove caracteres de controle
        sanitized = re.sub(r'[\x00-\x1f\x7f-\x9f]', '', sanitized)
        
        return sanitized
    
    @staticmethod
    def sanitize_json(data: Dict[str, Any]) -> Dict[str, Any]:
        """Sanitiza objetos JSON recursivamente"""
        if isinstance(data, dict):
            return {
                key: InputSanitizer.sanitize_json(value) 
                for key, value in data.items()
                if key not in ['__proto__', 'constructor', 'prototype']
            }
        elif isinstance(data, list):
            return [InputSanitizer.sanitize_json(item) for item in data]
        elif isinstance(data, str):
            return InputSanitizer.sanitize_string(data)
        else:
            return data

def sanitized_mcp_tool(func):
    """Decorator para sanitização automática de inputs"""
    @wraps(func)
    async def wrapper(context: Context, *args, **kwargs):
        # Sanitizar argumentos
        sanitized_kwargs = {}
        for key, value in kwargs.items():
            if isinstance(value, str) and key.endswith('_url'):
                sanitized_kwargs[key] = InputSanitizer.sanitize_url(value)
            elif isinstance(value, str):
                sanitized_kwargs[key] = InputSanitizer.sanitize_string(value)
            elif isinstance(value, (dict, list)):
                sanitized_kwargs[key] = InputSanitizer.sanitize_json(value)
            else:
                sanitized_kwargs[key] = value
        
        return await func(context, *args, **sanitized_kwargs)
    return wrapper

# Exemplo de uso
@mcp.tool()
@sanitized_mcp_tool
@security.require_permission("write:integrations")
async def create_integration(
    context: Context,
    name: str,
    documentation_url: str,
    configuration: Dict[str, Any]
) -> Dict[str, Any]:
    """Ferramenta sanitizada e protegida"""
    # Inputs já foram sanitizados pelo decorator
    return await process_create_integration(name, documentation_url, configuration)
```

---

## 📊 **MONITORING E DEBUGGING**

### **📈 Metrics Collection**
```python
import time
import psutil
from prometheus_client import Counter, Histogram, Gauge, start_http_server

# Métricas Prometheus
tool_calls_total = Counter('mcp_tool_calls_total', 'Total tool calls', ['tool_name', 'status'])
tool_duration = Histogram('mcp_tool_duration_seconds', 'Tool execution time', ['tool_name'])
active_connections = Gauge('mcp_active_connections', 'Active connections')
memory_usage = Gauge('mcp_memory_usage_bytes', 'Memory usage')

def metrics_middleware(func):
    """Middleware para coleta de métricas"""
    @wraps(func)
    async def wrapper(context: Context, *args, **kwargs):
        tool_name = func.__name__
        start_time = time.time()
        
        try:
            result = await func(context, *args, **kwargs)
            
            # Métricas de sucesso
            tool_calls_total.labels(tool_name=tool_name, status='success').inc()
            tool_duration.labels(tool_name=tool_name).observe(time.time() - start_time)
            
            return result
            
        except Exception as e:
            # Métricas de erro
            tool_calls_total.labels(tool_name=tool_name, status='error').inc()
            tool_duration.labels(tool_name=tool_name).observe(time.time() - start_time)
            raise
        
        finally:
            # Atualizar métricas de sistema
            memory_usage.set(psutil.virtual_memory().used)
    
    return wrapper

# Iniciar servidor de métricas
start_http_server(8000)

@mcp.tool()
@metrics_middleware
async def monitored_tool(context: Context, param: str) -> Dict[str, Any]:
    """Ferramenta com monitoramento automático"""
    return await process_tool_logic(param)
```

### **🔍 Debug Logging**
```python
import logging
import json
import traceback
from datetime import datetime

# Configurar logging estruturado
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger("mcp_server")

class StructuredLogger:
    """Logger estruturado para MCP"""
    
    @staticmethod
    def log_tool_call(tool_name: str, context: Context, params: Dict[str, Any]):
        """Log de chamada de ferramenta"""
        log_data = {
            "event": "tool_call",
            "tool_name": tool_name,
            "client_id": getattr(context, 'client_id', 'anonymous'),
            "params": params,
            "timestamp": datetime.utcnow().isoformat()
        }
        logger.info(json.dumps(log_data))
    
    @staticmethod
    def log_tool_result(tool_name: str, result: Dict[str, Any], duration: float):
        """Log de resultado de ferramenta"""
        log_data = {
            "event": "tool_result",
            "tool_name": tool_name,
            "result_size": len(str(result)),
            "duration_seconds": duration,
            "timestamp": datetime.utcnow().isoformat()
        }
        logger.info(json.dumps(log_data))
    
    @staticmethod
    def log_error(tool_name: str, error: Exception, context: Context):
        """Log de erro"""
        log_data = {
            "event": "tool_error",
            "tool_name": tool_name,
            "error_type": type(error).__name__,
            "error_message": str(error),
            "client_id": getattr(context, 'client_id', 'anonymous'),
            "stack_trace": traceback.format_exc(),
            "timestamp": datetime.utcnow().isoformat()
        }
        logger.error(json.dumps(log_data))

def debug_logging(func):
    """Decorator para logging detalhado"""
    @wraps(func)
    async def wrapper(context: Context, *args, **kwargs):
        tool_name = func.__name__
        start_time = time.time()
        
        # Log da chamada
        StructuredLogger.log_tool_call(tool_name, context, kwargs)
        
        try:
            result = await func(context, *args, **kwargs)
            
            # Log do resultado
            duration = time.time() - start_time
            StructuredLogger.log_tool_result(tool_name, result, duration)
            
            return result
            
        except Exception as e:
            # Log do erro
            StructuredLogger.log_error(tool_name, e, context)
            raise
    
    return wrapper
```

---

## 🎯 **TESTING MCP TOOLS**

### **🧪 Unit Testing**
```python
import pytest
import asyncio
from unittest.mock import Mock, AsyncMock, patch

class TestMCPTools:
    """Testes para ferramentas MCP"""
    
    @pytest.fixture
    def mock_context(self):
        """Context mock para testes"""
        context = Mock()
        context.client_id = "test_client"
        context.permissions = ["read:integrations", "write:integrations"]
        context.headers = {"Authorization": "Bearer test_token"}
        return context
    
    @pytest.mark.asyncio
    async def test_analyze_api_documentation_success(self, mock_context):
        """Teste de análise de documentação API"""
        
        # Mock external dependencies
        with patch('mcp_server.fetch_documentation') as mock_fetch, \
             patch('mcp_server.ai_analyze_content') as mock_ai:
            
            # Setup mocks
            mock_fetch.return_value = "API documentation content"
            mock_ai.return_value = {
                "name": "Test API",
                "provider": "Test Provider",
                "endpoints": ["/users", "/orders"],
                "auth_method": "Bearer Token"
            }
            
            # Execute tool
            result = await analyze_api_documentation(
                mock_context,
                documentation_url="https://api.example.com/docs",
                analysis_depth="standard"
            )
            
            # Assertions
            assert result["integration_name"] == "Test API"
            assert result["provider"] == "Test Provider"
            assert len(result["endpoints"]) == 2
            assert result["complexity_score"] > 0
            
            # Verify mocks were called
            mock_fetch.assert_called_once_with("https://api.example.com/docs")
            mock_ai.assert_called_once()
    
    @pytest.mark.asyncio
    async def test_analyze_api_documentation_invalid_url(self, mock_context):
        """Teste com URL inválida"""
        
        with pytest.raises(ValueError, match="Invalid URL protocol"):
            await analyze_api_documentation(
                mock_context,
                documentation_url="ftp://invalid.com/docs",
                analysis_depth="standard"
            )
    
    @pytest.mark.asyncio
    async def test_sync_erp_data_success(self, mock_context):
        """Teste de sincronização ERP"""
        
        with patch('mcp_server.get_erp_data') as mock_get, \
             patch('mcp_server.sync_to_target_erp') as mock_sync:
            
            # Setup mocks
            mock_get.return_value = [
                {"id": 1, "name": "Client 1"},
                {"id": 2, "name": "Client 2"}
            ]
            mock_sync.return_value = {
                "id": "sync_123",
                "synced_count": 2,
                "failed_count": 0,
                "errors": [],
                "duration": "5.2s"
            }
            
            # Execute tool
            result = await sync_erp_data(
                mock_context,
                source_erp="omie",
                target_erp="nibo",
                entity_type="clients",
                batch_size=100
            )
            
            # Assertions
            assert result["sync_id"] == "sync_123"
            assert result["source_records"] == 2
            assert result["synced_records"] == 2
            assert result["failed_records"] == 0
```

### **🔄 Integration Testing**
```python
import aiohttp
import pytest
from testcontainers import compose

class TestMCPIntegration:
    """Testes de integração MCP + N8N"""
    
    @pytest.fixture(scope="class")
    def docker_compose(self):
        """Setup Docker Compose para testes"""
        with compose.DockerCompose("./docker-compose.test.yml") as compose_stack:
            yield compose_stack
    
    @pytest.mark.asyncio
    async def test_n8n_mcp_workflow(self, docker_compose):
        """Teste completo de workflow N8N + MCP"""
        
        # Wait for services to be ready
        await asyncio.sleep(10)
        
        # Test data
        test_payload = {
            "documentation_url": "https://jsonplaceholder.typicode.com/",
            "analysis_depth": "standard"
        }
        
        # Trigger N8N workflow
        async with aiohttp.ClientSession() as session:
            async with session.post(
                "http://localhost:5678/webhook/analyze-api",
                json=test_payload
            ) as response:
                
                assert response.status == 200
                result = await response.json()
                
                # Verify workflow execution
                assert result["success"] is True
                assert "integration" in result
                assert result["complexity_score"] > 0
    
    @pytest.mark.asyncio
    async def test_mcp_server_health(self, docker_compose):
        """Teste de health check do servidor MCP"""
        
        async with aiohttp.ClientSession() as session:
            async with session.get("http://localhost:3001/health") as response:
                assert response.status == 200
                
                health_data = await response.json()
                assert health_data["status"] == "healthy"
                assert "tools" in health_data
                assert len(health_data["tools"]) > 0
```

---

## 🏆 **PRODUCTION DEPLOYMENT**

### **🐳 Docker Configuration**
```dockerfile
# Dockerfile para MCP Server
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3001/health || exit 1

# Expose port
EXPOSE 3001

# Run server
CMD ["python", "mcp_server.py"]
```

### **📋 Docker Compose Production**
```yaml
version: '3.8'

services:
  mcp-server:
    build: .
    ports:
      - "3001:3001"
    environment:
      - MCP_ENV=production
      - LOG_LEVEL=INFO
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://user:pass@postgres:5432/mcp
    depends_on:
      - redis
      - postgres
    restart: unless-stopped
    
  redis:
    image: redis:7-alpine
    restart: unless-stopped
    
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: mcp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

**🎯 Este guia fornece uma base sólida para integrar MCP com N8N de forma profissional e escalável!**