# QGIS MCP — Manual do Usuário
**Autor:** Lucho Ferrer (Associação QGIS Peru / Associação QGIS Espanha) | El Laboratorio de Lucho.  
**Data:** Março de 2026 | QGIS 3.44 + Claude Desktop (Sonnet 4.6)

---

## Introdução

Este manual descreve como usar o QGIS MCP na prática profissional. Pressupõe que a instalação já esteja concluída (veja *qgis_mcp_general_guide.pdf*) e que o status do servidor em Claude Desktop → Configurações → Desenvolvedor exiba **running** 🟢.

O QGIS MCP não substitui o seu critério técnico como especialista GIS — ele o amplifica. Você pode delegar ao Claude as tarefas repetitivas (carregar camadas, aplicar simbologia padrão, exportar mapas, rodar algoritmos) e focar na interpretação e análise.

---

## Fluxo de trabalho básico

Antes de cada sessão de trabalho, verifique se tudo está ativo:

```
1. Abra o QGIS 3.44
   → O plugin MCP inicia automaticamente o servidor (porta 9876)
   
2. Abra o Claude Desktop
   → Configurações → Desenvolvedor → qgis: running 🟢
   
3. Abra um novo chat e escreva o primeiro ping:
   "Ping QGIS and tell me the version"
   
4. Confirme a resposta com ferramenta real (ícone 🔨) → pronto para trabalhar. Caso não apareça, não se preocupe. O servidor deve estar em Produção e isso é o mais importante.
```

---

## Regras gerais de uso

**Seja específico com os caminhos.** O Claude não adivinha caminhos — forneça sempre o caminho completo e existente.

```
❌  "Carregue a camada de uso do solo"
✅  "Carregue a camada vetorial em G:\Lucho\workshop_mcp\uso_solo_2025.shp com o nome 'Uso do Solo'"
```

**Divida tarefas complexas.** Para fluxos longos, enumere as etapas explicitamente para que o Claude as execute em ordem e reporte cada resultado.

**Valide antes de continuar.** Após cada bloco de ações, peça ao Claude que liste as camadas ativas para confirmar o estado do projeto.

**Use `execute_code` com critério.** É a ferramenta mais poderosa, mas também a mais arriscada — permite rodar qualquer código PyQGIS diretamente na sua sessão do QGIS. Sempre revise o código antes de aprovar sua execução.

---

## Referência rápida de instruções

| O que você quer fazer | Como pedir ao Claude |
|---|---|
| Verificar conexão | `"Ping QGIS"` |
| Ver camadas ativas | `"List all layers in the current project"` |
| Criar novo projeto | `"Create a new project and save it at [caminho completo]"` |
| Carregar shapefile | `"Add vector layer [caminho] named [nome]"` |
| Carregar raster | `"Add raster layer [caminho] named [nome]"` |
| Zoom em camada | `"Zoom to layer [nome]"` |
| Executar algoritmo | `"Run processing algorithm [nome_algoritmo] on layer [nome] with params [...]"` |
| Exportar mapa | `"Render map to [caminho.png] at [DPI] DPI"` |
| Salvar projeto | `"Save the project"` |
| Código PyQGIS | `"Execute this PyQGIS code: [código]"` |

---

---

# EXEMPLO 1 — Preparar o projeto

Escreva no Claude Desktop:

```
Tenho acesso às ferramentas QGIS. Execute os seguintes passos em ordem e
confirme cada um antes de passar para o próximo:

1. Ping para verificar a conexão.
2. Crie um novo projeto QGIS e salve em:
   "G:\Lucho\workshop_mcp\projeto_1.qgz"
3. Carregue as seguintes camadas vetoriais:
   - "G:\Lucho\workshop_mcp\uso_solo_2025.shp" → nome: "Uso do Solo"
   - "G:\Lucho\workshop_mcp\ccpp.shp" → nome: "Centros_Populados"
4. Zoom para a camada "Traçado Duto".
5. Liste todas as camadas do projeto para confirmar que foram carregadas corretamente.
```

**Resposta esperada do Claude:** Confirmação de cada etapa com as camadas listadas ao final.

---

# Exemplo 2 — Simbologia temática

```
Aplique a seguinte simbologia às camadas do projeto:

1. Camada "Centros_Populados": Simbologia categorizada pelo campo "ubigeo"
   usando a paleta de cores "Spectral" com 5 classes.

```

---

## Boas práticas

### Nomenclatura de arquivos

Estabeleça uma convenção clara antes de iniciar uma sessão com o Claude:

```
Use a seguinte nomenclatura para todos os arquivos gerados:
- Vetoriais: [CODIGO_PROJETO]_[TEMA]_[DATA_AAAAMMDD].gpkg
- Rasters: [CODIGO_PROJETO]_[TEMA]_[RESOLUCAO]m_[DATA].tif
- Mapas: Mapa_[TEMA]_[ESCALA]_[DATA].png
- Projeto QGIS: [CODIGO_PROJETO]_[ETAPA]_v[VERSAO].qgz

Exemplo: EIA_GN2026_SensibilidadeAmbiental_20260326.gpkg
```

### Controle de versões do projeto

```
Ao finalizar cada sessão de trabalho, execute:
1. Salve o projeto com "Save Project"
2. Execute este código PyQGIS para registrar o estado:

from qgis.core import QgsProject
import datetime

projeto = QgsProject.instance()
camadas = [l.name() for l in projeto.mapLayers().values()]
log = f"""
=== RESUMO DA SESSÃO QGIS MCP ===
Data: {datetime.datetime.now().strftime('%Y-%m-%d %H:%M')}
Projeto: {projeto.fileName()}
Camadas ativas ({len(camadas)}):
""" + "\n".join(f"  - {c}" for c in sorted(camadas))
print(log)
```

---

## Referências e recursos

- Repositório QGIS MCP: https://github.com/nkarasiak/qgis-mcp
- Documentação PyQGIS: https://docs.qgis.org/3.40/en/docs/pyqgis_developer_cookbook/
- Comunidades: [QGIS Oficial](https://qgis.org/) | [QGIS Perú](https://qgis.pe/) | [QGIS España](https://www.qgis.es/) | [QGIS Brasil](https://qgisbrasil.org/)
