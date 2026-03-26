# QGIS MCP — Guia Geral de Instalação
**Autor:** Lucho Ferrer (Associação QGIS Peru / Associação QGIS Espanha) | El Laboratorio de Lucho.   
**Data:** Março de 2026 | QGIS 3.44 + Windows 11

---

## O que é o QGIS MCP?

O QGIS MCP conecta o Claude AI ao QGIS Desktop por meio do **Model Context Protocol (MCP)**. Ele permite que o Claude controle o QGIS diretamente: carregar camadas, executar algoritmos de processamento, aplicar simbologia, renderizar mapas e rodar código PyQGIS — tudo a partir de linguagem natural.

```
Você escreve no Claude Desktop
        ↓
  Servidor MCP (Python, roda em C:\qgis-mcp)
        ↓  socket TCP porta 9876
  Plugin QGIS MCP (roda dentro do QGIS)
        ↓
  PyQGIS executa as ações
```

> **⚠️ IMPORTANTE:** O fluxo principal funciona com o **Claude Desktop** (app instalada). Para usar via Antigravity ou API, consulte a seção final.

---

## Requisitos

- Windows 10/11
- QGIS 3.28 ou superior (testado na versão 3.44)
- [Claude Desktop](https://claude.ai/download) instalado
- Git instalado
- Python 3.12+ (incluído no QGIS/OSGeo4W)

---

## INSTALAÇÃO PASSO A PASSO

### PASSO 1 — Instalar o gerenciador de pacotes `uv`

Abra o **PowerShell** e execute:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Feche e reabra o PowerShell. Adicione `uv` ao PATH permanentemente:

```powershell
[System.Environment]::SetEnvironmentVariable("Path", "C:\Users\lferrer\.local\bin;" + [System.Environment]::GetEnvironmentVariable("Path", "User"), "User")
```

Verifique:

```powershell
uv --version
# Deve exibir: uv 0.11.x
```

---

### PASSO 2 — Clonar o repositório QGIS MCP

```powershell
git clone https://github.com/nkarasiak/qgis-mcp.git C:\qgis-mcp
```

---

### PASSO 3 — Preparar o pacote Python

```powershell
cd C:\qgis-mcp
```

Reescreva o `pyproject.toml` com `package = true` (necessário para que o `uv` instale o entry point):

```powershell
$toml = @"
[project]
name = "qgis-mcp"
version = "0.2.1"
description = "QGIS integration through the Model Context Protocol"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "mcp[cli]>=1.20.0",
]

[project.scripts]
qgis-mcp-server = "qgis_mcp.server:main"

[project.optional-dependencies]
dev = ["pytest>=7.0", "pytest-asyncio>=0.23"]

[tool.uv]
package = true

[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "W", "I", "UP", "B", "SIM", "RUF"]
ignore = ["E501"]

[tool.pytest.ini_options]
asyncio_mode = "auto"
"@
[System.IO.File]::WriteAllText("C:\qgis-mcp\pyproject.toml", $toml, [System.Text.UTF8Encoding]::new($false))
```

Instalar dependências:

```powershell
C:\Users\lferrer\.local\bin\uv.exe sync
```

Verificar se o servidor inicia:

```powershell
C:\Users\lferrer\.local\bin\uv.exe run qgis-mcp-server
# Deve exibir: QgisMCPServer starting up (will connect to QGIS at localhost:9876 on first call)
# Ctrl+C para parar
```

---

### PASSO 4 — Instalar o plugin no QGIS

```powershell
Copy-Item -Recurse "C:\qgis-mcp\qgis_mcp_plugin" "C:\Users\lferrer\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\qgis_mcp_plugin"
```

Em seguida, no QGIS:
1. **Complementos → Gerenciar e instalar complementos**
2. Aba **"Instalados"** → procure **QGIS MCP** → marque a caixa ✅
3. Aparecerá um botão MCP (ícone de corrente verde) na barra de ferramentas
4. Clique no botão → Porta `9876` → **Auto-start on startup** ✅ (o servidor inicia automaticamente)

---

### PASSO 5 — Configurar o Claude Desktop

Crie o arquivo de configuração **sem BOM** (crítico no Windows):

```powershell
New-Item -ItemType Directory -Force "C:\Users\lferrer\AppData\Roaming\Claude"

$json = @"
{
  "mcpServers": {
    "qgis": {
      "command": "C:/Users/lferrer/.local/bin/uv.exe",
      "args": [
        "run",
        "--directory",
        "C:/qgis-mcp",
        "qgis-mcp-server"
      ]
    }
  }
}
"@
[System.IO.File]::WriteAllText("C:\Users\lferrer\AppData\Roaming\Claude\claude_desktop_config.json", $json, [System.Text.UTF8Encoding]::new($false))
```

> **Observação:** Usar `[System.Text.UTF8Encoding]::new($false)` é crítico — evita o BOM (marca invisível que corrompe o JSON no Claude Desktop).

---

### PASSO 6 — Verificar a conexão

1. Abra o QGIS 3.44 (o plugin inicia o servidor automaticamente na porta 9876)
2. Feche o Claude Desktop completamente: `taskkill /F /IM "Claude.exe"`
3. Abra o Claude Desktop pelo menu Iniciar
4. Vá a **Configurações → Desenvolvedor** → o servidor `qgis` deve mostrar o status **running** 🟢
5. No chat, escreva:

```
Ping QGIS to check connection, then tell me the QGIS version installed
```

O Claude responderá usando ferramentas reais (você verá o ícone 🔨 na barra do chat).

---

## FERRAMENTAS DISPONÍVEIS (51 no total)

| Ferramenta | Descrição |
|---|---|
| `ping` | Verificar conexão com o QGIS |
| `get_qgis_info` | Versão do QGIS, plugins instalados |
| `create_new_project` | Criar novo projeto .qgz |
| `load_project` | Abrir projeto existente |
| `get_project_info` | Informações do projeto ativo |
| `add_vector_layer` | Carregar camada vetorial (SHP, GPKG, etc.) |
| `add_raster_layer` | Carregar camada raster (TIF, etc.) |
| `get_layers` | Listar todas as camadas do projeto |
| `remove_layer` | Remover camada por ID |
| `zoom_to_layer` | Zoom para a extensão de uma camada |
| `get_layer_features` | Extrair feições de uma camada vetorial |
| `execute_processing` | Executar algoritmos do Processador |
| `save_project` | Salvar o projeto |
| `render_map` | Exportar o mapa como imagem PNG |
| `execute_code` | ⚠️ Executar código PyQGIS arbitrário |

---

## SOLUÇÃO DE PROBLEMAS

### Erro: `is not valid JSON` ao abrir o Claude Desktop
**Causa:** O PowerShell salvou o JSON com BOM (marca invisível).  
**Solução:** Usar sempre `[System.Text.UTF8Encoding]::new($false)` para gravar o arquivo.

### O servidor aparece como `failed` no Claude Desktop
**Causa:** O Claude Desktop não consegue encontrar o `uv.exe` ou o diretório do projeto.  
**Solução:** Usar caminhos absolutos no config e reiniciar o Claude com `taskkill /F /IM "Claude.exe"`.

### `ModuleNotFoundError: No module named 'qgis_mcp'`
**Causa:** O pacote não está instalado no ambiente virtual.  
**Solução:** Adicionar `package = true` em `[tool.uv]` do `pyproject.toml` e rodar `uv sync`.

### O plugin não aparece no QGIS
**Causa:** A pasta do plugin não está no diretório correto.  
**Solução:** Verificar com `dir "C:\Users\lferrer\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\"`.

### `execute_code` retorna erro
**Causa:** O código PyQGIS tem um erro de sintaxe ou referência incorreta.  
**Solução:** Testar o código primeiro no console Python do QGIS antes de enviá-lo via MCP.

---

## AVISOS DE SEGURANÇA

- ⚠️ `execute_code` permite executar qualquer código Python na sua máquina. Use com cautela.
- ⚠️ Não use o MCP em projetos com dados confidenciais de clientes sem revisar a política de uso de IA da sua organização.
- ⚠️ O servidor MCP deve estar ativo somente quando necessário. O plugin tem **Auto-start** — você pode desativá-lo se preferir iniciá-lo manualmente.

---

## REFERÊNCIAS

- Repositório principal: https://github.com/nkarasiak/qgis-mcp
- Plugin no QGIS.org: https://plugins.qgis.org/plugins/qgis_mcp_plugin/
- Documentação MCP da Anthropic: https://docs.anthropic.com/en/docs/agents-and-tools/mcp
- Repositório original (jjsantos01): https://github.com/jjsantos01/qgis_mcp
- Comunidades: [QGIS Oficial](https://qgis.org/) | [QGIS Perú](https://qgis.pe/) | [QGIS España](https://www.qgis.es/) | [QGIS Brasil](https://qgisbrasil.org/)