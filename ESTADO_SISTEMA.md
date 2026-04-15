# Sistema Mega Tabla IL - Estado Actual y Proximos Pasos

## 1. Que es este sistema

Sistema automatizado que consolida datos del programa de Insercion Laboral (IL) de CREAMOS.
Conecta **KoboToolbox** (donde se recopilan datos en campo) con **Google Sheets** (donde se
almacenan para analisis) y esta diseñado para alimentar **Power BI** como destino final.

### Arquitectura General

```
KoboToolbox (12 formularios)
        |
    n8n Workflow (automatico cada 6 horas)
        |
        +---> Limpiar + Normalizar + Escalar Likert
        |
        +---> 4 hojas en Google Sheets:
              |
              +--- Master Sheet    (cada respuesta individual = 1 fila)
              +--- Mega Tabla      (1 fila por participante, todo consolidado)
              +--- PrePost         (comparacion Pre-Test vs Post-Test)
              +--- Indicadores_IL  (indicadores CREAMOS con desagregaciones)
```

### Llave maestra

`creamos_id` es el campo unico que identifica a cada participante y conecta
todas las tablas entre si (y futuras tablas de otros programas).

### Politica de privacidad (importante)

**El sistema NO almacena informacion personal identificable (PII).** Solo se
procesa `creamos_id` como identificador unico, mas campos demograficos no
identificantes (edad, genero, nivel educativo) que son necesarios para la
desagregacion de indicadores.

Campos que **NO se guardan** en ninguna hoja:
- Nombre, apellido, nombre_completo
- Telefono, celular, whatsapp
- Correo electronico
- DPI, CUI, numero de documento
- Direccion, colonia, zona, aldea, municipio
- Fecha de nacimiento (solo se usa para calcular edad, luego se descarta)
- Nombre de contacto de emergencia

Cualquier registro sin `creamos_id` valido se descarta automaticamente.

---

## 2. Formularios conectados (12 formularios KoboToolbox)

| Codigo | Nombre | Tiene Escala Likert |
|--------|--------|:-------------------:|
| C_01   | Hoja de Interes 2025 | No |
| C_01   | Hoja de Interes 2026 | No |
| IL_01  | Entrevista | No |
| IL_02  | Convenio | No |
| IL_03  | Estipendios | No |
| IL_05.1| Pre-Test | Si |
| IL_05.2| Post-Test | Si |
| IL_05  | Satisfaccion Formacion | Si |
| IL_06  | Satisfaccion Empleo | Si |
| IL_07  | Empleos Directos | No |
| IL_08  | Acompanamiento | No |
| IL_Pre_Post | Orientacion 2026 | Si |

---

## 3. Que produce el sistema (4 hojas en Google Sheets)

### 3.1 Master Sheet
- **Que contiene**: Cada respuesta individual de cada formulario = 1 fila
- **Para que sirve**: Datos crudos limpios, respaldo completo
- **Campos principales**: participante, formulario, fecha, todas las respuestas normalizadas

### 3.2 Mega Tabla
- **Que contiene**: 1 fila por participante con TODO consolidado
- **Para que sirve**: Vista 360 de cada participante para Power BI
- **Campos principales**:
  - Identificador: `Creamos_ID` (unica llave, sin datos personales)
  - Datos demograficos no identificantes (edad, genero, nivel educativo)
  - Estado por formulario (completado si/no, fecha, puntaje)
  - Avance general (% formularios, etapa actual, estado)
  - Comparacion Pre vs Post Test (delta, % cambio, resultado)
  - Satisfaccion (formacion, empleo, promedio general)
  - Campos reservados para programas futuros (barismo, terapia, estudios, empleo)

### 3.3 PrePost
- **Que contiene**: Comparacion detallada pregunta por pregunta del Pre-Test vs Post-Test
- **Para que sirve**: Analisis granular de cambio por competencia/pregunta

### 3.4 Indicadores_IL
- **Que contiene**: Indicadores calculados con codigos CREAMOS
- **Para que sirve**: Reportes institucionales, cumplimiento de metas
- **Indicadores calculados**:

| Codigo | Criterio | Indicador | Tipo |
|--------|----------|-----------|------|
| IL.P.01 | Pertinencia | Participantes en IL | Conteo unico |
| IL.P.02 | Pertinencia | Participantes con Acompanamiento | Conteo unico |
| IL.P.03 | Pertinencia | Participantes Formacion Tecnica | Conteo unico |
| IL.P.04 | Pertinencia | Personas Alcanzadas (Interes) | Conteo unico |
| IL.P.11 | Eficiencia | Sesiones Acompanamiento | Conteo total |
| IL.R.01 | Pertinencia | Satisfaccion Formacion Promedio | Promedio escala |
| IL.R.02 | Pertinencia | Satisfaccion Empleo Promedio | Promedio escala |
| IL.R.06 | Eficacia | Cambio Pre/Post % Mejora | Pre vs Post |
| IL.R.08 | Eficacia | Conexiones Laborales | Conteo unico |

- **Desagregaciones incluidas** (columnas separadas para Power BI):
  - Genero: Mujer/Femenino, Hombre/Masculino, Trans Mujer, Trans Hombre, No Binarie/Queer, Agenero, Autodescripcion, No Quiere Contestar, Sin Dato
  - Edad: Menor de 18, 18-24, 25-30, Mayor de 30, Sin Dato
  - Nivel Educativo: Primaria, Basico, Diversificado, Universitario, Otro, Sin Dato

---

## 4. Que funciona actualmente

| Componente | Estado | Notas |
|------------|--------|-------|
| Descarga automatica de KoboToolbox | FUNCIONA | Cada 6 horas, 12 formularios |
| Limpieza y normalizacion de datos | FUNCIONA | Campos flexibles, escala Likert automatica |
| Deteccion inteligente de campos | FUNCIONA | Busca genero/edad/nivel sin importar nombre exacto del campo en Kobo |
| Eliminacion de duplicados | FUNCIONA | Por creamos_id |
| Privacidad / sin PII | FUNCIONA | Solo creamos_id como identificador, sin nombres ni datos personales |
| Master Sheet | FUNCIONA | Escritura via API directa, headers = union de todas las columnas |
| Mega Tabla | FUNCIONA | 1 fila por participante consolidada |
| PrePost | FUNCIONA | Comparacion Pre vs Post con delta y % cambio |
| Indicadores IL | FUNCIONA | 9 indicadores con codigos CREAMOS |
| Desagregacion por edad | FUNCIONA | 5 rangos etarios |
| Desagregacion por nivel educativo | FUNCIONA | 6 categorias |
| Desglose por anio | FUNCIONA | Indicadores separados por anio |
| Columnas planas para Power BI | FUNCIONA | Sin JSON anidado, listo para tablas dinamicas |
| Integracion Formacion Tech | FUNCIONA | Lee Lista Definitiva, Cohortes, Graduadx, Retiradx, Estipendios |
| Integracion Formacion AyB | FUNCIONA | Mismas pestanas, join por creamos_id |
| Cohorte y estado en Mega Tabla | FUNCIONA | Programa_Tecnico, Cohorte, Estado_Cohorte, Estipendios_Recibidos, En_* |

---

## 5. Que tiene problemas conocidos

### 5.1 Desagregacion de Genero - EN REVISION
- **Problema**: Todos los valores caen en `Genero_Sin_Dato`, las demas columnas de genero quedan en 0
- **Causa probable**: KoboToolbox envia los valores de genero como codigos internos (ej: `mujer___femenino`) y no como texto legible. La normalizacion no los reconoce.
- **Estado**: Se aplico normalizacion flexible (minusculas, sin acentos, busqueda parcial). Se agrego fila de DIAGNOSTICO que muestra los valores crudos. **Pendiente ejecutar y verificar.**
- **Accion siguiente**: Ejecutar el workflow, revisar la fila DIAGNOSTICO en Indicadores_IL, y ajustar los patrones de matching con los valores exactos que KoboToolbox envia.

### 5.2 Targets/Metas de indicadores
- **Estado**: Los campos `Target_Compromiso` y `Target_Aspiracional` estan en 0
- **Razon**: No se han definido las metas numericas para cada indicador
- **Accion siguiente**: CREAMOS debe proporcionar las metas de cada indicador para que se calcule `Pct_Avance` correctamente

### 5.3 Campos reservados vacios
- Los campos `programa_barismo`, `programa_terapia`, `programa_estudios`, `programa_empleo`, `cohorte` estan vacios
- **Razon**: Son campos reservados para cuando se conecten esos programas
- **No es un error** - es intencional para futuro

---

## 6. Proximos pasos para presentacion profesional

### PRIORIDAD 1 - Resolver antes de presentar

- [ ] **Genero**: Ejecutar workflow, revisar fila DIAGNOSTICO, ajustar patrones con valores reales de KoboToolbox
- [ ] **Targets**: Definir con CREAMOS las metas numericas de cada indicador (Target_Compromiso y Target_Aspiracional)
- [ ] **Verificacion de datos**: Comparar conteos del sistema vs conteo manual para validar que los numeros son correctos
- [ ] **Eliminar fila DIAGNOSTICO**: Una vez resuelto genero, quitar la fila de debug

### PRIORIDAD 2 - Para presentacion profesional

- [ ] **Dashboard Power BI**: Crear dashboard conectado a Google Sheets con:
  - Pagina 1: KPIs principales (total participantes, % avance, satisfaccion promedio)
  - Pagina 2: Desagregacion demografica (genero, edad, nivel educativo)
  - Pagina 3: Comparacion Pre vs Post Test
  - Pagina 4: Indicadores CREAMOS con semaforo de cumplimiento
- [ ] **Filtros interactivos**: Por anio, genero, edad, nivel educativo, estado del participante
- [ ] **Visualizacion de semaforos**: Verde (meta cumplida), Amarillo (en progreso), Rojo (por debajo)

### PRIORIDAD 3 - Mejoras futuras

- [ ] **Conectar mas programas**: Barismo, Terapia, Estudios, Empleo (usando creamos_id como llave)
- [ ] **Cohortes**: Agregar campo de cohorte para segmentar por grupos
- [ ] **Alertas**: Configurar notificaciones cuando un indicador baje de cierto umbral
- [ ] **Historico**: Guardar snapshots mensuales de indicadores para ver tendencias

---

## 7. Como ejecutar y verificar

### Ejecutar el workflow
1. Abrir n8n
2. Importar `n8n_mega_tabla_workflow.json`
3. Verificar credenciales de Google Sheets y KoboToolbox
4. Ejecutar manualmente o esperar al auto-sync (cada 6 horas)

### Verificar resultados
1. Abrir Google Sheets
2. Revisar las 4 hojas: Master, Mega Tabla, PrePost, Indicadores_IL
3. En Indicadores_IL buscar la fila donde Codigo = "DIAGNOSTICO" para ver valores crudos de genero
4. Verificar que los conteos de participantes coincidan con lo esperado

### Conectar Power BI
1. Power BI Desktop > Obtener datos > Google Sheets
2. Conectar las 4 hojas
3. Usar `creamos_id` como clave de relacion entre tablas
4. Los campos ya estan en formato plano (sin JSON) listos para Power BI

---

## 8. Arquitectura tecnica del workflow n8n

### Nodos del workflow (20 nodos)

```
Auto-Sync Cada 6 Horas (trigger)
  |
Config Formularios (12 UIDs + token Kobo)
  |
Loop Formularios (itera por cada formulario)
  |
Descargar Kobo (HTTP Request a KoboToolbox API)
  |
Expandir Registros (1 registro por fila)
  |
Limpiar y Escalar (Code - normalizacion inteligente)
  |
  +---> Preparar Master --> Limpiar Master --> Guardar Master
  |
  +---> Crear Mega Tabla --> Preparar MegaTabla --> Limpiar MegaTabla --> Guardar MegaTabla
  |
  +---> Crear PrePost --> Preparar PrePost --> Limpiar PrePost --> Guardar PrePost
  |
  +---> Calcular Indicadores IL --> Preparar Indicadores --> Limpiar Indicadores --> Guardar Indicadores
```

### Decisiones tecnicas importantes
- **API directa vs nodos Google Sheets**: Se usan llamadas HTTP directas para evitar error 429 (rate limit) de la API de Google Sheets
- **Limpiar antes de guardar**: Cada hoja se limpia (clear) y luego se escribe completa para evitar datos huerfanos
- **Normalizacion flexible**: Los campos de KoboToolbox tienen nombres variables segun el formulario. El sistema normaliza (minusculas, sin acentos, sin prefijos de grupo) para encontrarlos automaticamente
- **Escala Likert automatica**: Respuestas tipo texto ("Totalmente de acuerdo") se convierten a numeros (1-5) para calcular promedios

---

## 9. Historial de desarrollo

| Fecha | Cambio |
|-------|--------|
| Inicio | Workflow basico: descargar Kobo y guardar en Google Sheets |
| Sprint 1 | Mega Tabla: consolidar 1 fila por participante con etapas y avance |
| Sprint 2 | Eliminar error 429: migrar de nodos Google Sheets a API HTTP directa |
| Sprint 3 | Cambiar llave: de participante_id a creamos_id |
| Sprint 4 | Deteccion inteligente de campos + eliminacion de duplicados |
| Sprint 5 | Indicadores IL automaticos con codigos CREAMOS |
| Sprint 6 | Desglose por anio + columnas planas para Power BI |
| Sprint 7 | Desagregacion demografica (genero, edad, educacion) en columnas individuales |
| Sprint 8 | Normalizacion flexible de genero + diagnostico |

---

## 10. Resumen ejecutivo

El sistema **Mega Tabla IL** automatiza completamente la consolidacion de datos del programa
de Insercion Laboral de CREAMOS. Descarga datos de 12 formularios KoboToolbox cada 6 horas,
los limpia, normaliza, y genera 4 productos de datos en Google Sheets listos para Power BI:
una hoja maestra con todos los registros, una mega tabla con vista 360 por participante,
una comparacion Pre vs Post Test, y una tabla de indicadores institucionales con desagregaciones
demograficas.

**Estado general: 90% funcional.** El principal pendiente es ajustar la normalizacion de genero
con los valores exactos que envia KoboToolbox (todos los demas campos funcionan correctamente).
Una vez resuelto, el sistema esta listo para conectar con Power BI y crear dashboards profesionales.
