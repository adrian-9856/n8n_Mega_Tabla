# Mega Tabla - Workflow n8n para KoboToolbox

## Que hace este workflow

Descarga datos de **12 formularios** de KoboToolbox cada 6 horas y genera **3 hojas** en Google Sheets:

### 1. Master (todas las respuestas individuales)
- Cada fila = una respuesta de un formulario
- Incluye datos limpios + escala Likert convertida a numeros

### 2. MegaTabla (UNA fila por participante)
- **Consolida TODO** en una sola fila por persona
- Columnas:
  - `participante_id` - Identificador unico basado en el nombre
  - `nombre_completo`, `edad`, `genero`, `zona`, `colonia`, `nivel_educativo`
  - `total_formularios_completados` - Cuantos formularios lleno
  - Para CADA formulario:
    - `[formulario]_completado` → Si / No
    - `[formulario]_fecha` → Fecha de envio
    - `[formulario]_puntaje` → Puntaje total (solo si tiene escala)
    - `[formulario]_promedio` → Promedio escala (solo si tiene escala)
    - `[formulario]_total_preguntas` → Numero de preguntas de escala
  - Comparacion Pre-Test vs Post-Test:
    - `tiene_pre_test`, `tiene_post_test`, `tiene_ambos_tests`
    - `pre_puntaje`, `post_puntaje`, `delta_puntaje`
    - `pct_mejora` - Porcentaje de mejora
    - `resultado_test` - "Mejoro", "Bajo", "Sin cambio", "Pendiente Post-Test"
  - Resumen de satisfaccion (Formacion y Empleo)

### 3. PrePost_Comparacion (detalle Pre vs Post Test)
- Solo participantes con Pre-Test
- Comparacion detallada pregunta por pregunta

## Como importar en n8n

1. Abrir n8n
2. Ir a **Workflows** → **Import from File**
3. Seleccionar `n8n_mega_tabla_workflow.json`
4. **IMPORTANTE**: Crear una hoja llamada **"MegaTabla"** en tu Google Sheet
5. Verificar credenciales de Google Sheets
6. Activar el workflow

## Formularios incluidos

| Codigo | Nombre | Tiene Escala |
|--------|--------|:------------:|
| C_01 | Hoja de Interes 2025 | No |
| C_01 | Hoja de Interes 2026 | No |
| IL_01 | Entrevista | No |
| IL_02 | Convenio | No |
| IL_03 | Estipendios | No |
| IL_05.1 | Pre-Test | Si |
| IL_05.2 | Post-Test | Si |
| IL_05 | Satisfaccion Formacion | Si |
| IL_06 | Satisfaccion Empleo | Si |
| IL_07 | Empleos Directos | No |
| IL_08 | Acompanamiento | No |
| IL_Pre_Post | Orientacion 2026 | Si |

## Requisitos previos

- Crear la hoja **"MegaTabla"** en el Google Sheet destino
- Las columnas se crean automaticamente la primera vez que corre
- La columna clave para actualizar es `participante_id`
