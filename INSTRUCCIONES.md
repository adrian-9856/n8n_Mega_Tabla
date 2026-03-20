# Mega Tabla IL - Sistema de Consolidacion para Power BI

## Vision General

```
participante_id = LLAVE MAESTRA que conecta todo
         |
    +----+----+----+----+----+
    |         |         |         |         |
 MegaTabla  Barismo  Terapia  Estudios  Empleo
 (IL forms) (futuro) (futuro) (futuro)  (futuro)
```

Cada participante tiene UN `participante_id` unico. Este ID conecta la Mega Tabla
con cualquier otro programa (barismo, terapia, estudios, empleo, cohortes, etc.)
para que Power BI pueda hacer relaciones entre tablas.

## Flujo del Workflow

```
Cada 6 horas
    |
Config Formularios (12 UIDs de Kobo + Token)
    |
Loop por cada formulario
    |
Descargar de KoboToolbox API
    |
Expandir registros individuales
    |
Limpiar + Escala Likert → numeros
    |
    +---> Guardar Master Sheet (cada respuesta = 1 fila)
    |
    +---> Crear Mega Tabla → Guardar Mega Tabla Sheet
    |     (1 fila por participante, todo consolidado)
    |
    +---> Crear PrePost → Guardar PrePost Sheet
          (comparacion detallada Pre vs Post Test)
```

## Columnas de la Mega Tabla (optimizadas para Power BI)

### BLOQUE 1 - Datos del Participante (Dimension)
| Columna | Descripcion | Uso en Power BI |
|---------|-------------|-----------------|
| `participante_id` | **LLAVE MAESTRA** - conecta con todas las tablas | Relacion/Clave |
| `nombre_completo` | Nombre del participante | Etiqueta |
| `edad` | Edad | Filtro/Segmentacion |
| `genero` | Genero | Filtro/Segmentacion |
| `zona` | Zona geografica | Filtro/Mapa |
| `colonia` | Colonia | Filtro/Mapa |
| `nivel_educativo` | Nivel de estudios | Filtro/Segmentacion |
| `fecha_primer_contacto` | Primera vez que aparece | Eje temporal |
| `fecha_ultimo_contacto` | Ultima actividad | Eje temporal |
| `total_formularios` | Cuantos formularios lleno | KPI |

### BLOQUE 2 - Estado por Formulario IL (Si/No + Fecha + Puntaje)
Para cada formulario, columnas con prefijo `il_`:

| Formulario | Columnas generadas | Tiene Escala |
|------------|-------------------|:------------:|
| Hoja Interes 2025 | `il_interes_2025_ok`, `_fecha` | No |
| Hoja Interes 2026 | `il_interes_2026_ok`, `_fecha` | No |
| Entrevista | `il_entrevista_ok`, `_fecha` | No |
| Convenio | `il_convenio_ok`, `_fecha` | No |
| Estipendios | `il_estipendios_ok`, `_fecha` | No |
| Pre-Test | `il_pre_test_ok`, `_fecha`, `_puntaje`, `_promedio`, `_num_preguntas` | Si |
| Post-Test | `il_post_test_ok`, `_fecha`, `_puntaje`, `_promedio`, `_num_preguntas` | Si |
| Satisfaccion Formacion | `il_sat_formacion_ok`, `_fecha`, `_puntaje`, `_promedio`, `_num_preguntas` | Si |
| Satisfaccion Empleo | `il_sat_empleo_ok`, `_fecha`, `_puntaje`, `_promedio`, `_num_preguntas` | Si |
| Empleos Directos | `il_empleos_ok`, `_fecha` | No |
| Acompanamiento | `il_acompanamiento_ok`, `_fecha` | No |
| Orientacion 2026 | `il_orient_2026_ok`, `_fecha`, `_puntaje`, `_promedio`, `_num_preguntas` | Si |

### BLOQUE 3 - Resumen de Avance (KPIs para dashboards)
| Columna | Descripcion | Uso en Power BI |
|---------|-------------|-----------------|
| `pct_avance_formularios` | % de formularios completados (0-100) | Gauge/KPI |
| `etapas_completadas` | Lista: Inscripcion, Ingreso, Proceso, etc. | Tooltip |
| `num_etapas` | Numero de etapas completadas | KPI |
| `estado_participante` | Inscrito / Ingresado / En Proceso / Egresado | Filtro principal |

### BLOQUE 4 - Comparacion Pre-Test vs Post-Test
| Columna | Descripcion | Uso en Power BI |
|---------|-------------|-----------------|
| `test_tiene_pre` | Si/No | Filtro |
| `test_tiene_post` | Si/No | Filtro |
| `test_tiene_ambos` | Si/No | Filtro |
| `test_pre_puntaje` | Puntaje Pre-Test | Barra/Comparacion |
| `test_post_puntaje` | Puntaje Post-Test | Barra/Comparacion |
| `test_delta` | Diferencia Post - Pre | KPI |
| `test_pct_cambio` | % de cambio | KPI |
| `test_resultado` | Mejoro / Bajo / Sin cambio / Pendiente | Filtro/Color |

### BLOQUE 5 - Satisfaccion
| Columna | Descripcion |
|---------|-------------|
| `sat_formacion_promedio` | Promedio satisfaccion formacion (1-5) |
| `sat_empleo_promedio` | Promedio satisfaccion empleo (1-5) |
| `sat_promedio_general` | Promedio general de satisfaccion |

### BLOQUE 6 - Campos Reservados (para futuras conexiones)
| Columna | Para que sirve |
|---------|----------------|
| `programa_barismo` | Se llenara cuando se conecte programa barismo |
| `programa_terapia` | Se llenara cuando se conecte programa terapia |
| `programa_estudios` | Se llenara cuando se conecte programa estudios |
| `programa_empleo` | Se llenara cuando se conecte programa empleo |
| `cohorte` | Cohorte del participante |
| `ultima_actualizacion` | Fecha de ultima sincronizacion |

## Como importar en n8n

1. Abrir n8n
2. Ir a **Workflows** → **Import from File**
3. Seleccionar `n8n_mega_tabla_workflow.json`
4. **IMPORTANTE**: Crear una hoja llamada **"MegaTabla"** en tu Google Sheet
5. Verificar credenciales de Google Sheets
6. Activar el workflow

## Como conectar con Power BI

1. En Power BI Desktop: **Obtener datos** → **Google Sheets**
2. Conectar la hoja **MegaTabla**
3. El campo `participante_id` es la **clave** para relacionar con otras tablas
4. Cuando agregues mas programas (barismo, terapia, etc.), crea nuevas hojas
   con `participante_id` como columna comun y Power BI las relacionara

### Modelo de datos sugerido para Power BI:
```
                    MegaTabla (IL)
                         |
                   participante_id
                         |
    +--------+-----------+-----------+--------+
    |        |           |           |        |
 Barismo  Terapia    Estudios    Empleo   Cohortes
```
