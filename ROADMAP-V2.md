# 🚒 DESPAX-CAD Frutillar — Roadmap v2

**Estado:** Aprobado · 26 abr 2026
**Pre-requisito:** Bloques 1 (Bitácora) y 2 (Modales Narrativos) instalados, probados y estables.

---

## 📋 Contexto

Después de finalizar el desarrollo y testing de los bloques actuales (Bitácora estructurada + Modales Narrativos: Relato, Primera Impresión, Pre-Informe, 6-7, 6-9, 6-10), avanzamos a este Roadmap v2.

Las ideas surgen de:
- Necesidades operacionales del Cuerpo de Bomberos de Frutillar
- Análisis comparativo con el sistema CAD del Cuerpo de Bomberos de Fresia (referencia regional)
- Brechas detectadas durante el desarrollo de los bloques 1 y 2

---

## 🎯 Bloques aprobados — orden de prioridad

### **🥇 Prioridad ALTA**

#### V2.1 — Panel resumen del cuerpo (dashboard)
Vista nueva tipo dashboard que muestra el estado completo del cuerpo de un vistazo.

**Componentes:**
- Card grande por cada compañía (1ª a 6ª + Comandancia) con:
  - Cantidad de **voluntarios en cuartel** ahora (vía respuestas WSP CallMeBot)
  - Cantidad de **maquinistas disponibles**
  - Cantidad de **carros disponibles** (con desglose: verde/0-9/0-8/en-emerg)
  - Estado actual: Disponible / En emergencia / Apoyo
- Totales globales del cuerpo arriba: "X volun en línea · Y carros disponibles · Z emergencias activas"
- Auto-refresh cada 30s

**Datos:**
- Voluntarios → suma de respuestas WSP positivas + estados de presencia
- Maquinistas → subset de voluntarios con flag `esMaquinista: true`
- Carros → estado actual del CAD

**Ubicación UI:** Pestaña nueva "RESUMEN" en el header principal, entre "DESPACHO" y "BITÁCORA".

---

#### V2.2 — Oficial a cargo de la emergencia (OBAC)
Captura del oficial responsable de cada emergencia, con cambio de mando registrado.

**Funcionalidad:**
- Selector de oficial al despachar (lista de oficiales activos del cuerpo)
- Selector visible en la card de emergencia activa para **cambiar el OBAC durante la emergencia**
- Cada cambio queda registrado en bitácora con hora y motivo opcional
- En el parte/PDF aparece la cadena de mando completa: "OBAC inicial: X · Asume mando: Y a las HH:MM"
- El sistema sugiere automáticamente el oficial de mayor jerarquía presente (Capitán > Teniente > Ayudante > Subalterno)

**Datos requeridos:**
- Flag `esOficial: true` y `jerarquia: 'capitan'|'teniente'|'ayudante'|'subalterno'` en cada voluntario
- Orden jerárquico configurable por compañía

**Bitácora:** evento nuevo tipo `obac_cambio` color púrpura oscuro.

---

#### V2.3 — Capas del mapa
Botones de toggle estilo Fresia para encender/apagar capas en el mapa.

**Capas a implementar:**
- 🏠 **Cuarteles** — las 6 cías + comandancia (ya hay datos en COMPANIAS)
- 🚒 **Material mayor** — ubicación actual de carros (requiere GPS — fase 2)
- 💧 **Grifos** — ya existen en el sistema
- 📍 **Esquinas** — cruces nombrados (ingresar manualmente o importar de OSM)
- 🗺️ **Sectores** — polígonos SECTORES_GEO (ya existen)
- 🏥 **Puntos críticos** — hospitales, colegios, comisarías, gasolineras (ingresar manualmente)
- ⚠️ **Riesgo forestal** — zonas de interfaz urbano-rural (polígonos manuales)

**UI:** Panel lateral colapsable en el mapa con checkboxes por capa.

---

#### V2.4 — Estado "Sin Conductor / Sin Maquinista"
Estado nuevo del carro: mecánicamente OK pero sin maquinista disponible.

**Lógica:**
- Estado nuevo: `sin-conductor` (color azul, igual que Fresia)
- El sistema detecta automáticamente: si un carro tiene maquinistas asignados pero ninguno está marcado como "disponible", el carro pasa a `sin-conductor`
- En el sugerido de despacho, los carros `sin-conductor` aparecen al final con indicador "⚠️ sin maquinista — llamar a [lista de maquinistas]"
- Botón rápido "📞 Llamar maquinistas" abre el modal de WhatsApp directo a maquinistas suplentes
- Estados completos del carro: `disponible` | `0-9` | `0-8` | `en-emergencia` | `sin-conductor`

**Bitácora:** evento `cambio_disponibilidad` cuando el carro entra/sale de este estado.

---

### **🥈 Prioridad MEDIA**

#### V2.5 — Citaciones y agenda del cuerpo
Espacio para que la central registre toda la actividad programada del cuerpo.

**Funcionalidad:**
- Vista calendario (mes/semana/día)
- Tipos de evento:
  - Reunión de compañía
  - Ejercicio
  - Acto oficial (aniversario, funeral, ceremonia)
  - Trabajo voluntario
  - Capacitación
  - Otros
- Por cada evento: título, fecha/hora, compañías involucradas, lugar, descripción
- Al generar el **informativo diario** (parte), se incluyen automáticamente las citaciones del día/semana en una sección dedicada
- Notificación WSP automática a voluntarios de las cías citadas (configurable: días antes del evento)
- Vista "Próximos eventos" siempre visible en el dashboard

**Ubicación UI:** Pestaña "CITACIONES" en el header.

---

#### V2.6 — Modo "Alarma General"
Flujo dedicado para emergencias mayores que requieren convocatoria total.

**Funcionalidad:**
- El botón "Alarma General" actual abre modal con:
  - Motivo (ej: "Incendio estructural mayor", "Forestal grado 3", "Catástrofe")
  - Compañías a alertar (default: todas)
- Al confirmar:
  - Citación automática WSP a **todos los voluntarios del cuerpo** (no solo cía despachada)
  - Tonos de atención automáticos a todos los cuarteles (cuando exista PTT/integración tonos)
  - Estado especial "🚨 ALARMA GENERAL ACTIVA" visible en todo el sistema (banner rojo en header)
  - Log especial de respuestas de voluntarios (quién acude, tiempo respuesta)
- Solo el superadmin / admin-cuerpo puede activar y desactivar

---

#### V2.7 — Prueba de equipos diaria (checklist matutino)
Sistema de verificación matutina de carros, alimenta automáticamente el estado 0-9/0-8.

**Funcionalidad:**
- Cada mañana, el oficial de guardia o conductor abre el checklist por carro
- Items configurables por tipo de carro (Bomba, Rescate, Apoyo, Escala):
  - Combustible OK
  - Aceite OK
  - Agua del estanque OK
  - Equipos de respiración (cantidad y presión)
  - Mangueras revisadas
  - Radio operativa
  - Kit de primeros auxilios
  - Equipos especiales según tipo
- Resultado:
  - Todo OK → carro queda automáticamente en **disponible (0-9)**
  - Algo falla → operadora marca el carro como **0-8** con motivo del check fallido
- Histórico auditable (útil para inspecciones de la Superintendencia)
- Reporte mensual exportable

---

### **🥉 Prioridad BAJA**

#### V2.8 — Estadísticas operacionales
Dashboard de análisis con métricas clave del cuerpo.

**Métricas:**
- Tiempo promedio de respuesta por sector
- Tipos de emergencia más frecuentes por sector
- Carros más usados / menos usados
- Voluntarios con más participación
- Heatmap de emergencias en el mapa (densidad)
- Comparativa mes a mes / año a año
- Exportar a PDF/Excel para informes municipales/SBC

**Ubicación UI:** Pestaña "ESTADÍSTICAS" (ya existe pero está vacía).

---

#### V2.9 — Mejoras menores acumuladas
Conjunto de pequeñas mejoras de UX que se pueden hacer en cualquier momento.

**Items:**
- Búsqueda por esquina (calle + calle) en formulario de despacho — muy chileno
- "Tono de Atención" como botón rápido (alerta a un cuartel sin despachar emergencia)
- Atajos de teclado para operadora (Ctrl+N nueva emergencia, Esc cerrar modal, etc.)
- Modo oscuro/claro toggle
- Mejoras de accesibilidad (tamaño de fuente configurable, alto contraste)
- Vista compacta vs amplia para emergencias activas

---

### **🔍 Investigar antes de planificar**

#### V2.10 — Integración con SACFI / sistema nacional de bomberos
Si la Junta Nacional de Cuerpos de Bomberos está pidiendo reporte centralizado al cuerpo, sería un gran plus exportar partes en formato compatible.

**Acciones previas:**
- Confirmar con el cuerpo si están obligados a reportar a SACFI
- Conseguir documentación del formato de exportación
- Validar API si existe
- Estimar costo en función de complejidad del formato

---

## 🅿️ Parking lot — Ideas en revisión

### Push-to-Talk vía radio (PTT)
**Estado:** En la mira, sin trabajo activo aún.

**Por qué se aparcó:**
- Complejidad técnica alta (requiere gateway radio-IP físico)
- Implicancias regulatorias (frecuencias bomberos reguladas por SUBTEL)
- Coordinación con técnicos de radio del cuerpo
- Costo no trivial (USD 500-2000 en hardware + integración)

**Cuando se retome, evaluar:**
- Fase 1: PTT digital entre operadora y voluntarios con app abierta (sin radio real)
- Fase 2: Gateway radio-IP físico — proyecto separado con presupuesto propio

---

## 📊 Tabla resumen de prioridades

| # | Bloque | Valor | Costo | Prioridad |
|---|---|---|---|---|
| V2.1 | Panel resumen del cuerpo | 🔥🔥🔥 | Medio | ALTA |
| V2.2 | OBAC + cambio durante emergencia | 🔥🔥🔥 | Bajo | ALTA |
| V2.3 | Capas del mapa | 🔥🔥 | Bajo | ALTA |
| V2.4 | Estado "Sin Conductor" | 🔥🔥🔥 | Bajo | ALTA |
| V2.5 | Citaciones y agenda | 🔥🔥 | Medio | MEDIA |
| V2.6 | Modo Alarma General | 🔥🔥 | Bajo | MEDIA |
| V2.7 | Prueba de equipos diaria | 🔥🔥 | Medio | MEDIA |
| V2.8 | Estadísticas operacionales | 🔥 | Alto | BAJA |
| V2.9 | Mejoras menores | 🔥 | Bajo | BAJA |
| V2.10 | Integración SACFI | ? | ? | INVESTIGAR |
| 🅿️ | Push-to-Talk vía radio | 🔥🔥 | Alto | PARKING |

---

## 🎬 Próximos pasos

1. **Finalizar Bloques 1 y 2** — debuggeo y validación de los 7 tests del INSTALACION.md
2. **Cerrar Roadmap v1** — Bloques pendientes originales:
   - Modal 6-0 con conductor + tripulación → ✅ ya incluido en desarrollo actual
   - Claves rápidas en mapa (6-5, 12-6, etc.)
   - Despachar carros individuales / re-despacho superadmin
3. **Iniciar Roadmap v2** comenzando por bloques de prioridad ALTA en este orden:
   - V2.4 Estado "Sin Conductor" (más rápido, alto valor)
   - V2.2 OBAC (rápido, alto valor)
   - V2.3 Capas del mapa (rápido, valor visual inmediato)
   - V2.1 Panel resumen del cuerpo (mayor desarrollo, alto valor)

---

## 📝 Notas

- Este roadmap se mantiene vivo: cualquier idea nueva o ajuste se anota acá antes de empezar el siguiente bloque
- Los costos son estimaciones gruesas; cada bloque tendrá su propia estimación detallada al iniciar
- Validación operacional con la operadora del cuerpo es fundamental antes de cerrar cada bloque

---

**Documento generado:** 26 de abril 2026
**Cliente:** Cuerpo de Bomberos de Frutillar
**Proveedor:** Despax (despax.cl)
