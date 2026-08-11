# MikroTik MCP for n8n

Fork do [mikrotik-mcp](https://github.com/jeff-nasseri/mikrotik-mcp) configurado para rodar como container Docker com transporte **SSE**, pronto para integração com **n8n**.

## O que muda neste fork

- **Transporte SSE por padrão** — o container já sobe em modo SSE (HTTP), sem precisar configurar
- **Docker Compose pronto** — aponte o Portainer para este repo e deploy
- **Log configurável** — variável `LOG_LEVEL` para debug sem rebuild
- **Documentação em PT-BR** — instruções completas em [README-N8N.md](README-N8N.md)

## Arquitetura

```
┌─────────────────┐     HTTP/SSE      ┌─────────────────────┐     SSH      ┌──────────────┐
│    n8n           │ ──────────────►   │  mikrotik-mcp:8000  │ ──────────►  │  MikroTik    │
│  (AI Agent)      │   porta 3000     │  (Docker Container) │  porta 22   │  Router      │
└─────────────────┘                   └─────────────────────┘              └──────────────┘
```

## Quick Start

### 1. Deploy via Portainer (recomendado)

1. **Stacks → Add Stack**
2. **Repository**: `https://github.com/Foninhoiuri/mcp-mikrotik-for-n8n.git`
3. **Compose path**: `docker-compose.yml`
4. Ajuste as variáveis de ambiente (IP, usuário, senha do MikroTik)
5. **Deploy the stack**

### 2. Deploy manual

```bash
git clone https://github.com/Foninhoiuri/mcp-mikrotik-for-n8n.git
cd mcp-mikrotik-for-n8n

# Edite o docker-compose.yml com suas credenciais
docker compose up -d
```

### 3. Conectar no n8n

1. No n8n, vá em **Settings → MCP Servers**
2. Transporte: **SSE**
3. URL: `http://IP_DO_SEU_SERVIDOR:3000/sse`

As ferramentas do MikroTik aparecem automaticamente no AI Agent.

## Variáveis de Ambiente

| Variável | Padrão | Descrição |
|----------|--------|-----------|
| `MIKROTIK_HOST` | `192.168.88.1` | IP do roteador MikroTik |
| `MIKROTIK_USERNAME` | `admin` | Usuário SSH |
| `MIKROTIK_PASSWORD` | *(vazio)* | Senha SSH |
| `MIKROTIK_PORT` | `22` | Porta SSH |
| `MIKROTIK_MCP__TRANSPORT` | `sse` | Transporte: `sse`, `streamable-http`, `stdio` |
| `MIKROTIK_MCP__ALLOWED_HOSTS` | *(vazio)* | Use `*` para rede local |
| `LOG_LEVEL` | `INFO` | Nível de log: `DEBUG`, `INFO`, `WARNING`, `ERROR` |

## Ferramentas disponíveis

Todas as 173+ tools do projeto original, incluindo:

- 🔥 **Firewall** — regras de filter e NAT
- 🌐 **DNS** — registros estáticos e configurações
- 📡 **DHCP** — leases e configuração de servidor
- 🔒 **WireGuard** — peers e interfaces VPN
- 🖧 **Interfaces** — VLANs, bridges, ethernet
- 📊 **Queues** — controle de banda
- 🗂️ **Backup** — export e backup do sistema
- 👥 **Usuários** — gerenciamento de contas
- 📋 **Logs** — consulta de logs do sistema
- 🛣️ **Rotas** — tabela de roteamento

## Documentação completa

📚 Veja [README-N8N.md](README-N8N.md) para instruções detalhadas incluindo:
- Deploy via Portainer
- Configuração multi-device (vários MikroTiks)
- Troubleshooting

## Créditos

Baseado no projeto [mikrotik-mcp](https://github.com/jeff-nasseri/mikrotik-mcp) por [Jeff Nasseri](https://github.com/jeff-nasseri).

## Licença

MIT License — veja [LICENSE](LICENSE).
