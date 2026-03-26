# QGIS MCP — Manual de Usuario
**Autor:** Lucho Ferrer (Asociación QGIS Perú/Asociación QGIS España) | El Laboratorio de Lucho.  
**Fecha:** Marzo 2026 | QGIS 3.44 + Claude Desktop (Sonnet 4.6)

---

## Introducción

Este manual describe cómo usar QGIS MCP en la práctica profesional. Asume que la instalación ya está completa (ver *qgis_mcp_general_guide.pdf*) y que el estado del servidor en Claude Desktop → Ajustes → Desarrollador muestra **running** 🟢.

QGIS MCP no reemplaza tu criterio técnico como especialista GIS — lo amplifica. Puedes delegar en Claude las tareas repetitivas (cargar capas, aplicar simbología estándar, exportar mapas, correr algoritmos) y enfocarte en la interpretación y el análisis.

---

## Flujo de trabajo básico

Antes de cada sesión de trabajo, verifica que todo está activo:

```
1. Abre QGIS 3.44
   → El plugin MCP arranca automáticamente el servidor (puerto 9876)
   
2. Abre Claude Desktop
   → Ajustes → Desarrollador → qgis: running 🟢
   
3. Abre un chat nuevo y escribe el primer ping:
   "Ping QGIS and tell me the version"
   
4. Confirmas respuesta con herramienta real (ícono 🔨) → listo para trabajar. De no aparecer, no te preocupes. El servidor debe estar en Produccion y eso es lo importante.
```

---

## Reglas generales de uso

**Sé específico con las rutas.** Claude no adivina rutas — proporciónale siempre la ruta completa y existente.

```
❌  "Carga la capa de uso de suelo"
✅  "Carga la capa vectorial en G:\Lucho\workshop_mcp\uso_suelo_2025.shp con nombre 'Uso de Suelo'"
```

**Desglosa las tareas complejas.** Para flujos largos, enumera los pasos explícitamente para que Claude los ejecute en orden y te reporte cada resultado.

**Valida antes de continuar.** Después de cada bloque de acciones, pide a Claude que liste las capas activas para confirmar el estado del proyecto.

**Usa `execute_code` con criterio.** Es la herramienta más poderosa pero también la más riesgosa — permite correr cualquier código PyQGIS directamente en tu sesión de QGIS. Revisa siempre el código antes de aprobar su ejecución.

---

## Referencia rápida de instrucciones

| Qué quieres hacer | Cómo pedírselo a Claude |
|---|---|
| Verificar conexión | `"Ping QGIS"` |
| Ver capas activas | `"List all layers in the current project"` |
| Crear proyecto nuevo | `"Create a new project and save it at [ruta completa]"` |
| Cargar shapefile | `"Add vector layer [ruta] named [nombre]"` |
| Cargar raster | `"Add raster layer [ruta] named [nombre]"` |
| Zoom a capa | `"Zoom to layer [nombre]"` |
| Ejecutar algoritmo | `"Run processing algorithm [nombre_algoritmo] on layer [nombre] with params [...]"` |
| Exportar mapa | `"Render map to [ruta.png] at [DPI] DPI"` |
| Guardar proyecto | `"Save the project"` |
| Código PyQGIS | `"Execute this PyQGIS code: [código]"` |

---

---

# EJEMPLO 1 — Preparar el proyecto

Escribe en Claude Desktop:

```
Tengo acceso a las herramientas QGIS. Ejecuta los siguientes pasos en orden y 
confírmame cada uno antes de pasar al siguiente:

1. Ping para verificar la conexión.
2. Crea un nuevo proyecto QGIS y guárdalo en:
   "G:\Lucho\workshop_mcp\proyecto_1.qgz"
3. Carga las siguientes capas vectoriales:
   - "G:\Lucho\workshop_mcp\uso_suelo_2025.shp" → nombre: "Uso de Suelo"
   - "G:\Lucho\workshop_mcp\ccpp.shp" → nombre: "Centros_poblados"
4. Zoom a la capa "Trazo Ducto".
5. Lista todas las capas del proyecto para confirmar que cargaron correctamente.
```

**Respuesta esperada de Claude:** Confirmación de cada paso con las capas listadas al final.

---

# Ejemplo 2 — Simbología temática

```
Aplica la siguiente simbología a las capas del proyecto:

1. Capa "Centros_poblados": Simbología categorizada por el campo "ubigeo"
   usando la paleta de colores "Spectral" con 5 clases.

```

---

## Buenas prácticas

### Nomenclatura de archivos

Establece una convención clara antes de iniciarle una sesión a Claude:

```
Usa la siguiente nomenclatura para todos los archivos que generes:
- Vectoriales: [CODIGO_PROYECTO]_[TEMA]_[FECHA_AAAAMMDD].gpkg
- Rasters: [CODIGO_PROYECTO]_[TEMA]_[RESOLUCION]m_[FECHA].tif
- Mapas: Mapa_[TEMA]_[ESCALA]_[FECHA].png
- Proyecto QGIS: [CODIGO_PROYECTO]_[ETAPA]_v[VERSION].qgz

Ejemplo: EIA_GN2026_SensibilidadAmbiental_20260326.gpkg
```

### Control de versiones del proyecto

```
Al finalizar cada sesión de trabajo, ejecuta:
1. Guarda el proyecto con "Save Project"
2. Ejecuta este código PyQGIS para registrar el estado:

from qgis.core import QgsProject
import datetime

proyecto = QgsProject.instance()
capas = [l.name() for l in proyecto.mapLayers().values()]
log = f"""
=== RESUMEN DE SESIÓN QGIS MCP ===
Fecha: {datetime.datetime.now().strftime('%Y-%m-%d %H:%M')}
Proyecto: {proyecto.fileName()}
Capas activas ({len(capas)}):
""" + "\n".join(f"  - {c}" for c in sorted(capas))
print(log)
```

---

## Referencias y recursos

- Repositorio QGIS MCP: https://github.com/nkarasiak/qgis-mcp
- Documentación PyQGIS: https://docs.qgis.org/3.40/en/docs/pyqgis_developer_cookbook/
- Comunidades: [QGIS Oficial](https://qgis.org/) | [QGIS Perú](https://qgis.pe/) | [QGIS España](https://www.qgis.es/) | [QGIS Brasil](https://qgisbrasil.org/)
