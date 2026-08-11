# Mikrotik MCP — Integração com n8n via Docker

Este guia explica como rodar o servidor MCP do MikroTik como container Docker com transporte SSE, pronto para integrar com o n8n.

## Arquitetura

```
┌─────────────────┐     HTTP/SSE      ┌─────────────────────┐     SSH      ┌──────────────┐
│    n8n           │ ──────────────►   │  mikrotik-mcp:8000  │ ──────────►  │  MikroTik    │
│  (AI Agent)      │   porta 3000     │  (Docker Container) │  porta 22   │  Router      │
└─────────────────┘                   └─────────────────────┘              └──────────────┘
```

## Pré-requisitos

- Docker e Docker Compose instalados
- Acesso SSH ao seu roteador MikroTik
- n8n rodando (pode ser em outro container/stack)

## Quick Start

### 1. Configurar credenciais

Edite o arquivo `docker-compose.yml` e ajuste as variáveis do seu MikroTik:

```yaml
environment:
  - MIKROTIK_HOST=192.168.88.1      # IP do seu MikroTik
  - MIKROTIK_USERNAME=admin          # Usuário SSH
  - MIKROTIK_PASSWORD=sua_senha      # Senha SSH
  - MIKROTIK_PORT=22                 # Porta SSH
```

### 2. Build da imagem

```bash
docker build -t mikrotik-mcp-sse .
```

### 3. Subir o container

```bash
docker compose up -d
```

### 4. Verificar se está rodando

```bash
# Health check
curl http://localhost:3000/health
# Deve retornar: OK

# Teste SSE (Ctrl+C para sair)
curl -N http://localhost:3000/sse
```

## Conectando no n8n

1. Abra o n8n
2. Vá em **Settings → MCP Servers** (ou no nó de AI Agent)
3. Selecione transporte **SSE**
4. URL: `http://IP_DO_SEU_LXC:3000/sse`

> **Nota**: Se o n8n e o MCP estão no mesmo host Docker mas em stacks diferentes,
> use o IP da máquina host (não `localhost`, pois cada container tem seu próprio
> namespace de rede).

As ferramentas de gerenciamento do MikroTik aparecerão automaticamente no Agent.

## Deploy via Portainer

1. No Portainer, vá em **Stacks → Add Stack**
2. Dê um nome (ex: `mikrotik-mcp`)
3. Cole o conteúdo de `docker-compose.yml`
4. Ajuste as variáveis de ambiente
5. Clique em **Deploy the stack**

## Variáveis de Ambiente

| Variável | Padrão | Descrição |
|----------|--------|-----------|
| `MIKROTIK_MCP__TRANSPORT` | `sse` | Transporte MCP: `sse`, `streamable-http`, ou `stdio` |
| `MIKROTIK_MCP__HOST` | `0.0.0.0` | IP de bind do servidor MCP |
| `MIKROTIK_MCP__PORT` | `8000` | Porta interna do servidor MCP |
| `MIKROTIK_MCP__ALLOWED_HOSTS` | *(vazio)* | Hosts permitidos. Use `*` para desabilitar proteção |
| `MIKROTIK_HOST` | `192.168.88.1` | IP do roteador MikroTik |
| `MIKROTIK_USERNAME` | `admin` | Usuário SSH |
| `MIKROTIK_PASSWORD` | *(vazio)* | Senha SSH |
| `MIKROTIK_PORT` | `22` | Porta SSH |
| `MIKROTIK_INVENTORY_FILE` | *(vazio)* | Caminho para arquivo YAML de inventário multi-device |
| `MIKROTIK_INVENTORY` | *(vazio)* | Inventário inline em YAML/JSON |

## Multi-Device (Vários MikroTiks)

Para gerenciar múltiplos roteadores, crie um `inventory.yaml`:

```yaml
- title: Router Principal
  host: 192.168.88.1
  username: admin
  password: senha1
  port: 22

- title: Router Filial
  host: 10.0.0.1
  username: admin
  password: senha2
  port: 22
  region: SP
  tags: [filial, producao]
```

Descomente as linhas de volume no `docker-compose.n8n.yml`:

```yaml
volumes:
  - ./inventory.yaml:/config/inventory.yaml:ro
environment:
  - MIKROTIK_INVENTORY_FILE=/config/inventory.yaml
```

## Troubleshooting

### Container não sobe
```bash
docker logs mcp_mikrotik
```

### n8n não conecta
- Verifique se a porta 3000 está acessível: `curl http://IP:3000/health`
- Confirme que `MIKROTIK_MCP__ALLOWED_HOSTS=*` está setado
- Verifique a rede Docker (containers em networks diferentes não se comunicam)

### Erro de autenticação SSH
- Confirme as credenciais do MikroTik
- Verifique se o usuário tem permissão SSH no MikroTik
- Teste: `ssh admin@192.168.88.1` a partir do container

### Erro "Invalid Host header" (HTTP 421)
- Defina `MIKROTIK_MCP__ALLOWED_HOSTS=*` ou adicione o hostname/domínio correto
