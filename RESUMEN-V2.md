# DESPAX-CAD v2 · Resumen de cambios y propuestas

**Cuerpo de Bomberos de Frutillar · Central de Alarmas**
**Fecha:** 27 abril 2026
**Versión:** v2 (post Bloques 1, 2, 5 + Roadmap v2 parcial)

---

## 📑 Índice

1. [Cambios implementados](#1-cambios-implementados)
2. [Propuestas — Perspectiva de Operadora](#2-propuestas--perspectiva-de-operadora-de-cad)
3. [Propuestas — Perspectiva Comandante / Superintendente](#3-propuestas--perspectiva-comandante--superintendente)
4. [Backlog priorizado](#4-backlog-priorizado-por-valor--esfuerzo)

---

## 1. Cambios implementados

### 1.1 Modelo de datos · per carro (Bloque 5)
- **Antes:** estado de las emergencias se trackeaba por dispatch unit (`B-1`, `RX-1`...) — equivalía a "cía"
- **Ahora:** `e.clavesEstado[carroId]` indexed por carro real (`B1`, `BX1`, `R1`, etc.)
- **Source of truth:** `e.carros[]` (array de IDs). `e.companias[]` se mantiene como derived para retrocompatibilidad con WSP/parte
- **Consecuencias:**
  - Múltiples carros de una misma cía pueden estar en una emergencia (ej. B1 + BX1 + R1 de la 1ª)
  - Cada carro tiene su propia secuencia 6-0 → 6-3 → 6-9 → 6-10
  - Cierre de emergencia preserva carros marcados 0-8 (fuera de servicio) entre emergencias

### 1.2 Modales narrativos (Bloque 2 + extensiones V2)

| Modal | Trigger | Captura |
|---|---|---|
| 📞 **Relato del llamado** | Bloqueante pre-despacho | Texto libre + chips configurables por código (10-X) |
| 🚒 **Modal 6-0** | Click "Marcar 6-0" en un carro | Conductor + Oficial a cargo + Cantidad de tripulación + Notas |
| 👁️ **Primera Impresión** | Auto al primer 6-3 de cualquier carro | Observaciones desde afuera, skippeable |
| 📋 **Pre-Informe** | Botón en card emergencia tras 6-3 | Diagnóstico + estado (bajo/desarrollo/refuerzo/crítico) + heridos + evacuados + apoyo (auto re-despacho si pide) |
| ✅ **6-7 Controlada** | Botón en carro slot designado | Comentario opcional + warning si falta pre-informe |
| ↩️ 6-9 | Click en carro | Disponibilidad: 0-9 (disponible) o 0-8 (fuera de servicio) |
| 🏠 6-10 | Click en carro | KM de llegada + checkbox "mantener fuera de servicio" |
| ➕ **Re-despacho** | Botón "+ Despachar más recursos" en card | Selector de carros adicionales con sort por especialidad + motivo obligatorio |

### 1.3 Layout y navegación

**Cambios visuales mayores:**
- Panel **Cías/Carros/Grifos/Dotación** removido del despacho (movido a Resumen)
- **Bitácora movida arriba del mapa** (con altura dinámica: 60px sin emergencia, 280px durante despacho)
- **Emergencias Activas** pasa al lateral derecho (donde estaba bitácora)
- **Botón Alarma General** sale del topbar y aparece dentro de cada card de emergencia (rojo prominente)
- Tiles del mapa cambiados a CartoDB Dark Matter (no requiere Referer HTTP, funciona desde `file://`)

**Nuevas tabs en el header:**
- 📊 **Resumen** — dashboard con KPIs operativos
- 📋 **Bitácora** rediseñada como feed cronológico

### 1.4 Resumen del despacho por emergencia (visual)

Encima de cada emergencia activa:
- **Semáforo** (🔴 rojo → 🟡 amarillo → 🟢 verde) según progreso operativo
- 4 KPIs visuales: 🚒 Despachados / 🛣️ En camino / 📍 En lugar / 👥 Voluntarios
- Verde se queda fijo (sin pulso) cuando se marca 6-7

### 1.5 Tab Resumen — KPIs macro

| KPI | Detalle |
|---|---|
| 🔵 Llamados hoy | Con delta % vs ayer |
| 🟣 Llamados ayer | Histórico día anterior |
| 🔴 Activas ahora | IDs de emergencias en curso |
| 🟢 Carros en servicio | Desglose disponible / 0-8 / en-emerg / mantención |
| 🟦 Cías disponibles | X / 6 + alerta sin conductor |
| 🟡 Voluntarios en cuartel | Total + cuántos conductores |
| 🔵 Tiempo prom. respuesta | Despacho → primer 6-3 (calculado en runtime) |
| 🟣 Última actividad | Hace cuánto fue el último llamado |

Sumado a:
- Estado de servicios (Firebase / Radio VHF / WSP / sync voluntarios)
- Turno actual (operador logueado + tiempo de turno)
- Estado del Cuerpo (cías mini con presentes + flag sin-conductor)
- Distribución de tipos de llamados hoy (mini bar chart)
- Otras actividades del cuerpo (salidas no-emergencia desde PWAs)
- Auto-refresh cada 30 segundos

### 1.6 Tab Bitácora — feed cronológico

- Filtros de período: Hoy / 3 días / 7 días / Mes / Todo
- Búsqueda por ID, dirección, tipo, zona, reportante
- Resumen del período (5 cards con totales)
- Feed unificado: emergencias 🚨 (con badge activa/cerrada) + actividades no-emergencia 📅 (con tipo: instrucción, ejercicio, reunión, acto, social, t. voluntario)
- Click → abre modal detalle con bitácora estructurada
- Auto-cerrar bitácora al cerrar emergencia
- Cerradas-de-hoy minimizadas en panel "Activas" del despacho

### 1.7 Voluntarios + presencia + conductor por carro (V2.4)

**En PWAs de cías (compania-1.html..6.html):**
- Toggle "🏠 Estoy en cuartel" por voluntario en la lista
- Modal de editar voluntario expande: por cada carro de la cía, checkbox "habilitado a conducir"
- Counter en dashboard: "X presentes en cuartel · Y conductores presentes"
- Persistencia a Firestore (`voluntarios` collection con campos `presenteEnCuartel`, `presenciaTimestamp`, `conductorCarros[]`)

**En CAD:**
- Subscribe a `voluntarios` via onSnapshot (live sync con PWAs)
- Modal 6-0 filtra dropdown de conductor por carro específico
- Detección "sin conductor presente" con warnings:
  - En tab Carros del panel
  - En sugerido del despacho (al final, con chip "⚠ SIN CONDUCTOR")
  - En modal re-despacho (sort al final)
  - En cards de cía del Resumen
- Helper `_carroSinConductor(carroId)` evalúa por carro específico, no solo por cía

### 1.8 Mantenedor de etiquetas rápidas (admin.html)

- Sección nueva en admin: "🏷️ Etiquetas Rápidas"
- CRUD de chips por código de emergencia (10-0, 10-1, ..., 10-7, DEF)
- Persistencia en `config_cad/relato_chips`
- CAD lee al iniciar (con fallback a defaults hardcoded si Firestore vacío)
- Validaciones: máx 80 caracteres, sin duplicados case-insensitive

### 1.9 Validaciones del formulario de despacho

- Reportante y teléfono ahora **obligatorios**
- Prefijo **+56** fijo en el campo teléfono (operador solo escribe los dígitos)
- Validación: mínimo 8 dígitos
- Indicadores visuales en labels (asterisco rojo)

### 1.10 Bug fixes y mejoras técnicas

| Bug | Fix |
|---|---|
| Despacho 6-9 → 6-10 no aparecía | Lógica de `ultimoIdx` salta s67 cuando carro no es slot designado |
| `serverTimestamp is not defined` en bitácora | Imports del módulo Firebase expuestos al script clásico vía `window` |
| HTML tags `<strong>` se mostraban literales en bitácora | Función `bitFormat` con whitelist (`strong`, `b`, `em`, `i`, `br`) |
| Cuadro vacío entre Cías y Bitácora | Comentario HTML malformado tenía `-->` interno; se resolvía prematuramente y renderizaba `═══...═══` como texto |
| Mapa con tiles desplazados/negros | ResizeObserver sobre `#map` + cascada de `invalidateSize` (100/400/900/1800ms) |
| OSM 403 desde `file://` | Tiles cambiados a CartoDB (no requiere Referer) |
| Bitácora panel quedaba minimizado sin botón visible para abrir | CSS de header colapsado: padding reducido, botón centrado más grande |
| Toggle "Solicitar apoyo" del Pre-informe no funcionaba | Race entre toggle nativo del label + `chk.checked = !chk.checked` JS |
| Patches de bitácora se aplicaban antes del módulo Firebase | Polling (`aplicarPatchesBitacora`) que espera a que window.fbX esté disponible |

---

## 2. Propuestas — Perspectiva de Operadora de CAD

> "Estoy sola en el turno, suena el teléfono, hay un incendio, mientras escribo el llamado entra otra emergencia... ¿qué necesito que el sistema haga por mí?"

### 2.1 Quick wins (alto valor, esfuerzo bajo)

#### 🔔 Notificaciones audibles configurables
- **Tono de alerta** cuando llega un nuevo llamado (configurable por tipo: 10-0 distinto que 10-2)
- **Tono de recordatorio** si no hay 6-3 a los 5 min de despachado (ya parpadea en pantalla, pero no suena)
- **Tono de cierre** cuando se cierra una emergencia
- **Volumen máximo** override durante turno nocturno

#### 📞 Llamada en un click
- En el campo Teléfono del reportante: ícono `📞 Llamar` que abre `tel:+56...`
- En tabla de despacho de carros: si tienen GPS, "📍 Cómo llegar" abre Google Maps con ruta
- Click en número WSP → copia al portapapeles

#### ⌨️ Atajos de teclado
- `Ctrl+N` — nueva emergencia (limpia el form y enfoca dirección)
- `Ctrl+L` — ver lista de activas (foco al panel derecho)
- `Esc` — cerrar modal abierto
- `F1` — ayuda contextual
- `Ctrl+/` — listado de atajos

#### 💾 Plantillas de despacho rápido
- "Plantillas favoritas" guardadas: ej. "Fuego basural Frutillar Alto" prellena código + zona + cías + relato
- Al guardar una plantilla: aprende del despacho actual ("Guardar como plantilla")
- Crea/elimina/edita desde un mini panel

#### 🔍 Búsqueda inversa de carros
- En el form: "Buscar carro disponible que tenga..." → tipo (Bomba/Rescate/Escala) + zona
- Sugerencia en tiempo real mientras se escribe
- Útil cuando el operador no recuerda qué cía tiene qué tipo

#### 📋 Notas pegajosas / quick-notes
- Mini-card encima del mapa donde el operador escribe lo que necesita recordar (ej. "AndresAguirre: cumple turno 22:00")
- Persisten entre cambios de tab pero no entre sesiones (o opcional persistir)
- Útil para coordinación sin contaminar la bitácora oficial

### 2.2 Mediano esfuerzo (transforma la operación)

#### 🤝 Handoff de turno
- Al hacer logout: pop-up "Resumen del turno" con todo lo que pasó
- "Notas para el siguiente operador" → texto libre que el siguiente lee al loguearse
- Cierre de turno con firma digital (timestamp + operador) — útil para SBC
- Lista de "cosas pendientes" que el siguiente debe seguir
- Email opcional con resumen al supervisor

#### 📍 Geolocalización del reportante
- Botón "Pedir ubicación" que envía un SMS al reportante con un link `https://maps.google.com/?ll=X,Y` (requiere que el reportante haga click)
- O integración con SOSafe / aplicaciones de emergencia que ya hacen esto
- Si recibe → se rellena automáticamente lat/lng del despacho

#### 🚦 Sugerencia de carros por especialidad + distancia
- Hoy: tabla `TABLA_DESPACHO` matchea código → cías
- Mejora: factorizar también distancia (lat/lng del cuartel a la dirección reportada)
- Si hay GPS de carros: usar la distancia real
- Mostrar tiempo estimado de llegada por cía

#### 🌡️ Modo "todo a la vista" durante emergencia crítica
- Cuando se marca pre-informe `crítico` → activar modo full-screen
- Esconde tabs (Bitácora, Estadísticas, etc.) — solo Despacho visible
- Bitácora gigante a la derecha
- Mapa centrado en la emergencia con zoom alto
- Solo se desactiva cuando el operador lo permite

#### 🔄 Multi-incidente (relacionar emergencias)
- En el form de nueva emergencia: "¿Esta emergencia está relacionada con otra activa?" → selector
- Si vinculada → comparten relato + se notifica como "incidente complejo"
- Útil cuando un mismo accidente tiene 3 reportes

#### 📱 Modo móvil para tablet
- Layout responsive ya parcialmente implementado
- Optimizar para tablets de 10" (algunos operadores usan)
- Touch-friendly: targets más grandes, swipes

### 2.3 Esfuerzo grande (visión a largo plazo)

#### 🎙️ Push-to-Talk integrado (ya en parking lot del roadmap)
- Operador presiona botón → habla a la radio del cuartel destino
- Voluntarios responden por radio normal
- Requiere gateway radio-IP físico

#### 🤖 Asistente IA de despacho
- Ya hay chat IA flotante (Groq Llama). Mejorarlo:
- Sugiere despachos según el relato escrito
- Detecta palabras clave que cambian la prioridad ("personas atrapadas", "niños", "embarazada")
- Genera el parte automáticamente al cerrar

#### 📷 Recibir fotos del reportante
- Modal con QR / link → reportante escanea → sube fotos desde su teléfono
- Las fotos van a la bitácora con timestamp
- Útil para evaluar magnitud de incendios estructurales

---

## 3. Propuestas — Perspectiva Comandante / Superintendente

> "Soy responsable del cuerpo. ¿Cómo sé si funcionamos bien? ¿Cómo justifico ante la municipalidad? ¿Cómo planifico?"

### 3.1 Reporting y compliance

#### 📊 Reportes mensuales/anuales (V2.8 del roadmap original)
- Exportable a PDF con membrete del cuerpo
- Incluye: total emergencias, tipo, sectores, tiempos respuesta, voluntarios participantes, costos estimados
- Configurable: por mes, trimestre, año
- Auto-generado el día 1 de cada mes y enviado por email a las autoridades

#### 🇨🇱 Integración SACFI (V2.10 del roadmap)
- Junta Nacional de Cuerpos de Bomberos de Chile pide reporte centralizado
- Exportar partes en formato compatible (XML/JSON según especifique)
- Investigar API si existe
- Cumplimiento normativo

#### 📜 Auditoría completa
- "¿Quién despachó qué emergencia?" — log de operadores con timestamp
- "¿Quién marcó 6-7 a las XX:XX?" — quien y cuándo modificó cada campo
- "¿Por qué no se despachó la 5ª en este caso?" — registro de decisiones
- Útil para responder consultas legales, demandas, peritajes

### 3.2 Análisis y planificación estratégica

#### 🌡️ Heatmap de emergencias
- Mapa con calor por sector según frecuencia de llamados
- Filtros por tipo (incendios estructurales / forestales / médicas)
- Sirve para: dónde abrir nueva cía, dónde poner grifos, dónde reforzar capacitación

#### 📈 Tendencias operacionales
- "¿Subieron los 10-1 (accidentes) este invierno vs el anterior?"
- "¿Qué cía atendió más esta semana?"
- "¿Tiempo de respuesta promedio mejoró/empeoró respecto a hace 6 meses?"
- Comparativos automáticos en el dashboard ejecutivo

#### 🔮 Predicción de demanda
- Días/horas de mayor actividad histórica
- "Esperar más llamados los viernes-sábados entre 22-02 hs"
- Sugerencia: ajustar staffing del CAD según predicción

### 3.3 Personal y voluntariado

#### 👨‍🚒 Performance de voluntarios
- "¿Quiénes acuden más a las emergencias?"
- "¿Quiénes nunca responden?"
- "¿Quién es el conductor más activo de B1?"
- Gamification opcional: ranking mensual (con cuidado para no generar conflictos)
- Útil para: reconocimientos, promociones, capacitaciones dirigidas

#### 📚 Gaps de capacitación
- Análisis: "Esta cía tarda más en marcar 6-3 vs el promedio" → ¿necesita instrucción?
- Detección automática: ciertos tipos de emergencias se manejan peor que otros
- Recomendar capacitaciones específicas

#### 🎂 Calendario humano
- Cumpleaños, aniversarios de ingreso, ascensos
- Notificación al cuerpo
- Reconocimientos públicos
- Bonus: 25/50 años de servicio (medallas SBC)

### 3.4 Recursos materiales

#### 🚒 Gestión de mantenimiento de carros
- Alerta cuando un carro se acerca a fechas de mantención (cada X km)
- Histórico de fallas: "BX1 ha tenido 3 fallas en 2025 → evaluar reemplazo"
- Calendarización de revisiones técnicas
- Costos acumulados por carro

#### ⛽ Combustible
- KM recorridos por mes / por emergencia
- Costo estimado de combustible
- Comparativo entre carros (¿el RX1 consume mucho más?)

#### 🧰 Inventario de equipos
- Pies de manguera, BAB (botellas auto-respiración), trajes nivel I/II/III
- Stock por cía
- Alertas de vencimiento (BAB requiere prueba hidrostática cada 5 años)
- Asignación a emergencias (qué se usó dónde)

### 3.5 Coordinación inter-institucional

#### 🤝 Integración con Carabineros y SAMU
- Cuando se despacha 10-1 (accidente vehicular): notificar a Carabineros
- Cuando se despacha 10-2 (médica): notificar a SAMU
- Vía API o WSP automatizado
- Cierre conjunto de incidentes

#### 🏛️ Reporte a la municipalidad
- Vista pública del CAD: "X emergencias atendidas este mes en Frutillar"
- Mapa público de cuarteles y grifos (transparencia)
- Estado de servicio (cuántos carros disponibles AHORA, sin nombres ni datos sensibles)

#### 🌐 Mutual aid (apoyo entre cuerpos)
- Cuando una emergencia escala más allá de Frutillar: notificar a Llanquihue, Puerto Varas, etc.
- Catálogo de cuerpos vecinos y sus contactos
- Tracking de "deudas" mutuas (Frutillar fue 3 veces a Puerto Varas, viceversa 2)

### 3.6 Governance del sistema

#### 🔐 Permisos granulares
- Hoy: 5 roles (superadmin, admin-cuerpo, operadora, admin-cía, oficial, voluntario)
- Mejora: matriz de permisos editable desde admin
- "El admin-cía no debe ver emergencias de OTRAS cías"
- "Solo superadmin puede modificar la tabla de despacho"

#### 📝 Histórico de cambios de configuración
- Quién cambió qué en la tabla de despacho, cuándo, por qué
- Versionar la configuración del CAD
- Permitir rollback

#### 💼 Multi-cuerpo
- Hoy: solo Frutillar
- Cambio: que el sistema soporte N cuerpos (Frutillar, Fresia, Llanquihue, etc.)
- Cada cuerpo con su propia base de datos / config / personal
- Vista nacional para SBC (si llega a interesarles)

---

## 4. Backlog priorizado (por valor / esfuerzo)

### 🥇 Top 10 (recomiendo hacer en próximo bloque)

| # | Feature | Esfuerzo | Valor | Persona |
|---|---|---|---|---|
| 1 | Tono de alerta sonora al recibir llamado | Bajo | Alto | Operadora |
| 2 | Llamada un-click al reportante (`tel:`) | Bajo | Alto | Operadora |
| 3 | Atajos de teclado básicos (Ctrl+N, Esc) | Bajo | Medio | Operadora |
| 4 | Plantillas de despacho rápido | Medio | Alto | Operadora |
| 5 | Handoff de turno con notas | Medio | Alto | Operadora |
| 6 | Heatmap de emergencias en mapa | Medio | Alto | Comandante |
| 7 | Reporte mensual exportable PDF | Medio | Alto | Comandante |
| 8 | Auditoría: log de cambios por operador | Bajo | Alto | Comandante |
| 9 | Alerta de mantención de carros por KM | Bajo | Medio | Comandante |
| 10 | Multi-incidente (relacionar emergencias) | Alto | Medio | Operadora |

### 🥈 Visión a 6 meses
- Push-to-Talk via gateway radio-IP
- Predicción de demanda con histórico
- Integración Carabineros/SAMU
- Performance de voluntarios + gamification

### 🥉 Visión a 12 meses
- Multi-cuerpo (Frutillar + otros)
- Vista pública municipal
- Asistente IA con análisis del relato
- Integración SACFI nacional

---

## 📌 Notas de cierre

Este documento es vivo: cualquier feedback de operadores reales o del comandante debe agregarse acá antes de empezar el siguiente bloque.

**Validación recomendada antes de seguir:**
- Una jornada de prueba con la operadora real usando el sistema actual
- Sesión de 30 min con el comandante para mostrarle el dashboard y ver qué prioriza
- Recoger 3 dolores específicos de cada uno → eso baja al backlog priorizado

**Repositorio:** https://github.com/javierlama123-boop/CentralBomberos
**Cliente:** Cuerpo de Bomberos de Frutillar
**Proveedor:** Despax (despax.cl)
