# QGIS MCP — Guía General de Instalación 🇵🇪
**Autor:** Lucho Ferrer (Asociación QGIS Perú / Asociación QGIS España) | El Laboratorio de Lucho.   
**Fecha:** Marzo 2026 | QGIS 3.44 + Windows 11

---

## ¿Qué es QGIS MCP?

QGIS MCP conecta Claude AI con QGIS Desktop a través del **Model Context Protocol (MCP)**. Permite que Claude controle QGIS directamente: cargar capas, ejecutar algoritmos de procesamiento, aplicar simbología, renderizar mapas y correr código PyQGIS — todo desde lenguaje natural.

```
Tú escribes en Claude Desktop
        ↓
  Servidor MCP (Python, corre en C:\qgis-mcp)
        ↓  socket TCP puerto 9876
  Plugin QGIS MCP (corre dentro de QGIS)
        ↓
  PyQGIS ejecuta las acciones
```

> **⚠️ IMPORTANTE:** El flujo principal funciona con **Claude Desktop** (app instalada). Para usar desde Antigravity o API, ver la sección final.

---

## Requisitos

- Windows 10/11
- QGIS 3.28 o superior (probado en 3.44)
- [Claude Desktop](https://claude.ai/download) instalado
- Git instalado
- Python 3.12+ (incluido en QGIS/OSGeo4W)

---

## INSTALACIÓN PASO A PASO

### PASO 1 — Instalar el gestor de paquetes `uv`

Abre **PowerShell** y ejecuta:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Cierra y vuelve a abrir PowerShell. Agrega `uv` al PATH permanentemente:

```powershell
[System.Environment]::SetEnvironmentVariable("Path", "C:\Users\lferrer\.local\bin;" + [System.Environment]::GetEnvironmentVariable("Path", "User"), "User")
```

Verifica:

```powershell
uv --version
# Debe mostrar: uv 0.11.x
```

---

### PASO 2 — Clonar el repositorio QGIS MCP

```powershell
git clone https://github.com/nkarasiak/qgis-mcp.git C:\qgis-mcp
```

---

### PASO 3 — Preparar el paquete Python

```powershell
cd C:\qgis-mcp
```

Reescribir el `pyproject.toml` con `package = true` (necesario para que `uv` instale el entry point):

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

Instalar dependencias:

```powershell
C:\Users\lferrer\.local\bin\uv.exe sync
```

Verificar que el servidor arranca:

```powershell
C:\Users\lferrer\.local\bin\uv.exe run qgis-mcp-server
# Debe mostrar: QgisMCPServer starting up (will connect to QGIS at localhost:9876 on first call)
# Ctrl+C para detenerlo
```

---

### PASO 4 — Instalar el plugin en QGIS

```powershell
Copy-Item -Recurse "C:\qgis-mcp\qgis_mcp_plugin" "C:\Users\lferrer\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\qgis_mcp_plugin"
```

Luego en QGIS:
1. **Complementos → Administrar e instalar complementos**
2. Pestaña **"Instalados"** → busca **QGIS MCP** → marca la casilla ✅
3. Aparece un botón MCP (ícono de cadena verde) en la barra de herramientas
4. Clic en el botón → Puerto `9876` → **Auto-start on startup** ✅ (el servidor arranca automático)

---

### PASO 5 — Configurar Claude Desktop

Crear el archivo de configuración **sin BOM** (crítico en Windows):

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

> **Nota:** Usar `[System.Text.UTF8Encoding]::new($false)` es crítico — evita el BOM (marca invisible que rompe el JSON en Claude Desktop).

---

### PASO 6 — Verificar la conexión

1. Abre QGIS 3.44 (el plugin inicia el servidor automáticamente en puerto 9876)
2. Cierra Claude Desktop completamente: `taskkill /F /IM "Claude.exe"`
3. Abre Claude Desktop desde el menú Inicio
4. Ve a **Ajustes → Desarrollador** → el servidor `qgis` debe mostrar estado **running** 🟢
5. En el chat, escribe:

```
Ping QGIS to check connection, then tell me the QGIS version installed
```

Claude responderá usando herramientas reales (verás el ícono 🔨 en la barra del chat).

---

## HERRAMIENTAS DISPONIBLES (51 en total)

| Herramienta | Descripción |
|---|---|
| `ping` | Verificar conexión con QGIS |
| `get_qgis_info` | Versión de QGIS, plugins instalados |
| `create_new_project` | Crear nuevo proyecto .qgz |
| `load_project` | Abrir proyecto existente |
| `get_project_info` | Info del proyecto activo |
| `add_vector_layer` | Cargar capa vectorial (SHP, GPKG, etc.) |
| `add_raster_layer` | Cargar capa raster (TIF, etc.) |
| `get_layers` | Listar todas las capas del proyecto |
| `remove_layer` | Eliminar capa por ID |
| `zoom_to_layer` | Zoom a extensión de una capa |
| `get_layer_features` | Extraer features de una capa vectorial |
| `execute_processing` | Ejecutar algoritmos del Procesador |
| `save_project` | Guardar el proyecto |
| `render_map` | Exportar el mapa como imagen PNG |
| `execute_code` | ⚠️ Ejecutar código PyQGIS arbitrario |

---

## SOLUCIÓN DE PROBLEMAS

### Error: `is not valid JSON` al abrir Claude Desktop
**Causa:** PowerShell guardó el JSON con BOM (marca invisible).  
**Solución:** Usar siempre `[System.Text.UTF8Encoding]::new($false)` para escribir el archivo.

### El servidor aparece como `failed` en Claude Desktop
**Causa:** Claude Desktop no puede encontrar `uv.exe` o el directorio del proyecto.  
**Solución:** Usar rutas absolutas en el config y reiniciar Claude con `taskkill /F /IM "Claude.exe"`.

### `ModuleNotFoundError: No module named 'qgis_mcp'`
**Causa:** El paquete no está instalado en el entorno virtual.  
**Solución:** Agregar `package = true` en `[tool.uv]` del `pyproject.toml` y correr `uv sync`.

### El plugin no aparece en QGIS
**Causa:** La carpeta del plugin no está en el directorio correcto.  
**Solución:** Verificar con `dir "C:\Users\lferrer\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\"`.

### `execute_code` devuelve error
**Causa:** El código PyQGIS tiene un error de sintaxis o referencia incorrecta.  
**Solución:** Probar el código primero en la consola Python de QGIS antes de enviarlo vía MCP.

---

## ADVERTENCIAS DE SEGURIDAD

- ⚠️ `execute_code` permite ejecutar cualquier código Python en tu máquina. Úsalo con precaución.
- ⚠️ No uses MCP con proyectos que contengan datos confidenciales de clientes sin revisar la política de uso de IA en tu compañía o laburo.
- ⚠️ El servidor MCP solo debe estar activo cuando lo necesites. El plugin tiene **Auto-start** — puedes desactivarlo si prefieres iniciarlo manualmente.

---

## REFERENCIAS

- Repositorio principal: https://github.com/nkarasiak/qgis-mcp
- Plugin en QGIS.org: https://plugins.qgis.org/plugins/qgis_mcp_plugin/
- Documentación MCP de Anthropic: https://docs.anthropic.com/en/docs/agents-and-tools/mcp
- Repo original (jjsantos01): https://github.com/jjsantos01/qgis_mcp
- Comunidades: [QGIS Oficial](https://qgis.org/) | [QGIS Perú](https://qgis.pe/) | [QGIS España](https://www.qgis.es/) | [QGIS Brasil](https://qgisbrasil.org/)


---


<div align="center">

# QGIS MCP — Guia Geral de Instalação 🇧🇷
**Model Context Protocol para QGIS e Claude AI**

[![QGIS Version](https://img.shields.io/badge/QGIS-3.28%2B-green.svg)](https://qgis.org)
[![Python Version](https://img.shields.io/badge/Python-3.12%2B-blue.svg)](https://python.org)
[![Claude Desktop](https://img.shields.io/badge/Claude-Desktop-orange.svg)](https://claude.ai)

</div>

---

## 📍 O que é o QGIS MCP?

**QGIS MCP** conecta o Claude AI ao QGIS Desktop por meio do **Model Context Protocol (MCP)**. Permite que o Claude controle o QGIS diretamente via linguagem natural para:
- Carregar e gerenciar camadas (vetoriais e raster).
- Executar algoritmos de processamento.
- Aplicar simbologia.
- Renderizar mapas.
- Executar código PyQGIS.

### ⚙️ Arquitetura da Conexão

```mermaid
graph TD
    A[Você escreve no Claude Desktop] -->|Solicitação| B(Servidor MCP Python)
    B -->|Socket TCP Porta 9876| C(Plugin QGIS MCP no QGIS)
    C -->|Executa ações| D[PyQGIS]
```

> **⚠️ IMPORTANTE:** O fluxo principal funciona com o **Claude Desktop** (aplicativo instalado).

---

## 📋 Pré-requisitos

- **Sistema Operacional:** Windows 10/11
- **QGIS:** Versão 3.28 ou superior (testado na 3.44)
- **Claude:** [Claude Desktop](https://claude.ai/download) instalado
- **Controle de versão:** Git instalado
- **Python:** 3.12 ou superior (geralmente incluído no QGIS/OSGeo4W)

---

## 🚀 Instalação Passo a Passo

### PASSO 1 — Instalar o gerenciador de pacotes `uv`

Abra o **PowerShell** e execute o seguinte comando para instalar o `uv`:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Feche e reabra o PowerShell. Adicione `uv` ao PATH permanentemente:

```powershell
[System.Environment]::SetEnvironmentVariable("Path", "C:\Users\lferrer\.local\bin;" + [System.Environment]::GetEnvironmentVariable("Path", "User"), "User")
```

Verifique a instalação *(Deve exibir `uv 0.11.x` ou superior)*:
```powershell
uv --version
```

### PASSO 2 — Clonar o repositório QGIS MCP

```powershell
git clone https://github.com/nkarasiak/qgis-mcp.git C:\qgis-mcp
```

### PASSO 3 — Preparar o pacote Python

Entre no diretório clonado:
```powershell
cd C:\qgis-mcp
```

Reescreva o arquivo `pyproject.toml` (certifique-se de incluir `package = true` para que o `uv` instale o entry point). Você pode executar este script no PowerShell:

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

Instale as dependências e verifique o servidor:
```powershell
C:\Users\lferrer\.local\bin\uv.exe sync
C:\Users\lferrer\.local\bin\uv.exe run qgis-mcp-server
```
*(Deve exibir uma mensagem indicando que o servidor está iniciando e aguardará conexão na porta 9876. Use `Ctrl+C` para pará-lo).*

### PASSO 4 — Instalar o plugin no QGIS

Copie o plugin para o diretório de perfis do QGIS:
```powershell
Copy-Item -Recurse "C:\qgis-mcp\qgis_mcp_plugin" "C:\Users\lferrer\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\qgis_mcp_plugin"
```

Em seguida, dentro do QGIS:
1. Vá a **Complementos → Gerenciar e instalar complementos**.
2. Na aba **"Instalados"**, procure **QGIS MCP** e marque a caixa ✅.
3. Aparecerá um botão MCP *(ícone de corrente verde)* na barra de ferramentas.
4. Clique no botão → Porta `9876` → Marque **Auto-start on startup** ✅ (para iniciar automaticamente).

### PASSO 5 — Configurar o Claude Desktop

Crie o arquivo de configuração **sem BOM** (crítico no Windows para evitar erros no Claude):

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

### PASSO 6 — Verificar a conexão

1. Abra o QGIS (o plugin iniciará o servidor na porta 9876).
2. Feche completamente o Claude Desktop (se estiver aberto): `taskkill /F /IM "Claude.exe"`.
3. Abra o Claude Desktop.
4. Vá a **Configurações → Desenvolvedor**. O servidor `qgis` deve aparecer com status **running** 🟢.
5. No chat do Claude, teste escrevendo:
   > *"Ping QGIS to check connection, then tell me the QGIS version installed"*

---

## 🛠️ Ferramentas Disponíveis

O protocolo MCP habilita até **51 ferramentas**. As principais incluem:

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

## 🆘 Solução de Problemas

| Problema | Causa | Solução |
| --- | --- | --- |
| **Erro: `is not valid JSON` no Claude** | O PowerShell salvou o JSON com BOM (marca invisível). | Usar `[System.Text.UTF8Encoding]::new($false)` ao gravar o arquivo pelo PowerShell. |
| **Servidor como `failed` no Claude** | Claude não encontra `uv.exe` ou o diretório. | Usar caminhos absolutos no arquivo de config e reiniciar o Claude (`taskkill /F /IM "Claude.exe"`). |
| **`ModuleNotFoundError: No module named 'qgis_mcp'`** | Pacote não instalado no ambiente virtual. | Adicionar `package = true` em `[tool.uv]` do `pyproject.toml` e rodar `uv sync`. |
| **Plugin não aparece no QGIS** | Pasta em local incorreto. | Verificar se está no caminho correto em `AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\`. |
| **`execute_code` retorna erro** | Erro de sintaxe no PyQGIS ou referência incorreta. | Testar o código no console Python do QGIS antes de enviá-lo via MCP. |

---

## 🛡️ Avisos de Segurança

- ⚠️ A ferramenta `execute_code` permite executar **qualquer código Python** na sua máquina. Use com extrema cautela.
- ⚠️ Não use o MCP em projetos com dados confidenciais se isso violar as políticas de uso de IA da sua organização.
- ⚠️ Mantenha o servidor MCP ativo somente quando necessário. Você pode desativar o **Auto-start** do plugin se preferir um início manual.

---

## 🔗 Referências Úteis

- [Repositório principal do QGIS MCP](https://github.com/nkarasiak/qgis-mcp)
- [Plugin no QGIS.org](https://plugins.qgis.org/plugins/qgis_mcp_plugin/)
- [Documentação MCP da Anthropic](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)
- [Repositório original (jjsantos01)](https://github.com/jjsantos01/qgis_mcp)
- Comunidades: [QGIS Oficial](https://qgis.org/) | [QGIS Perú](https://qgis.pe/) | [QGIS España](https://www.qgis.es/) | [QGIS Brasil](https://qgisbrasil.org/)

---
*Guia desenvolvido por Lucho Ferrer (Associação QGIS Peru / Espanha) | El Laboratorio de Lucho.*
