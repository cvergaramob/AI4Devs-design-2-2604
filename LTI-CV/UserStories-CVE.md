# Product Backlog — Plataforma ATS LTI-CVE
**LTI-CVE · Mayo 2026**

---

## Contexto del producto

**LTI-CVE** es un Applicant Tracking System (ATS) de nueva generación diseñado para empresas de 50 a 5.000 empleados. Su North Star es: *"Contratar tan fluido para el equipo interno como hacer un pedido online, y tan transparente para el candidato como rastrear un envío."*

El producto articula su propuesta de valor en tres ejes diferenciales: la experiencia del candidato como ventaja competitiva, la adopción del hiring manager como condición de diseño, y la inteligencia artificial como motor de ejecución real. Este backlog representa el conjunto completo de historias de usuario priorizadas para llevar el producto desde cero hasta su primera versión lista para design partners.

**Roles del sistema:**

| Rol | Descripción |
|---|---|
| Candidato | Usuario externo que aplica a vacantes |
| Recruiter / TA Manager | Usuario interno que gestiona el pipeline diariamente |
| Hiring Manager (HM) | Usuario interno con baja frecuencia de uso; toma decisiones sobre candidatos |
| TA Lead / VP People | Usuario estratégico; consume analytics y reportes |
| Administrador | Configura el sistema, integraciones y permisos |
| Agencia de reclutamiento | Usuario externo con acceso controlado; envía candidatos y hace seguimiento de fees |

---

## 1. Historias de Usuario

Las historias se organizan por épica. Cada épica agrupa historias relacionadas con un mismo diferenciador estratégico del producto.

---

### Épica E1 — Experiencia del Candidato

> **Diferenciador atacado:** D1 — Candidate experience como producto central
> **Pains del mercado:** Pain #1 (abandono de formularios, 96% de impacto) · Pain #3 (comunicación deficiente, 85%)

---

#### US-01a — Formulario de aplicación con campos mínimos y progreso adaptativo

**Historia:**
Como candidato externo, quiero completar el formulario de aplicación con solo los campos estrictamente necesarios para la vacante, adaptado a mi dispositivo, para no abandonar el proceso por fricción innecesaria.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: formulario adaptado a móvil*
- **Given** que accedo a una vacante desde un dispositivo móvil (viewport < 768px),
- **When** hago clic en "Aplicar ahora",
- **Then** el formulario presenta máximo 8 campos obligatorios en pantalla completa sin scroll horizontal, el botón "Siguiente" es visible sin hacer scroll, y el progreso se persiste en la DB (`Application.stage = "draft"`) cada 30 segundos.

*Escenario 2 — Edge case: retoma de sesión interrumpida*
- **Given** que completo el 70% del formulario y cierro el navegador sin enviar,
- **When** regreso al enlace de la vacante desde el mismo navegador dentro de los 7 días siguientes,
- **Then** el banner "Retomar aplicación guardada" aparece en la parte superior del formulario con todos los campos previos cargados en menos de 2 segundos, incluyendo el archivo CV si fue adjuntado.

*Escenario 3 — Error: campo obligatorio ausente*
- **Given** que intento enviar el formulario sin completar el campo "Email" (obligatorio),
- **When** hago clic en "Enviar",
- **Then** el campo Email muestra un borde rojo y el mensaje "Este campo es obligatorio" directamente debajo del campo en menos de 500ms, el scroll se posiciona en el primer campo con error, y ningún otro campo pierde su valor.

**Estimación de complejidad:** M

---

#### US-01b — Importación de perfil desde LinkedIn y parser de CV

**Historia:**
Como candidato externo, quiero importar mi perfil de LinkedIn o subir mi CV para que los campos del formulario se rellenen automáticamente, para no tener que reescribir información que ya existe en mi perfil.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: importación desde LinkedIn*
- **Given** que hago clic en "Importar desde LinkedIn" y autorizo el acceso OAuth,
- **When** el sistema procesa mi perfil,
- **Then** en menos de 8 segundos los campos nombre, apellido, últimos 3 empleos, formación y habilidades se pre-rellenan, y cada campo muestra el icono de LinkedIn indicando origen automático.

*Escenario 2 — Happy path: carga y parsing de CV*
- **Given** que subo un archivo PDF o DOCX de hasta 5MB,
- **When** el parser procesa el documento,
- **Then** en menos de 10 segundos los campos se pre-rellenan y el sistema indica con un badge cuáles fueron completados automáticamente versus cuáles requieren revisión manual.

*Escenario 3 — Error: CV no parseable*
- **Given** que el CV subido es una imagen escaneada o un archivo corrupto,
- **When** el procesamiento falla,
- **Then** el sistema muestra el mensaje "No pudimos leer este archivo. Por favor sube un PDF con texto seleccionable o completa los campos manualmente", sin bloquear el envío del formulario.

**Estimación de complejidad:** M

---

#### US-03 — Portal de seguimiento en tiempo real para el candidato

**Historia:**
Como candidato que ha enviado una aplicación, quiero acceder a un portal personal que me muestre en qué etapa está mi candidatura y cuándo esperar novedades, para no tener que contactar al recruiter para conocer el estado de mi proceso.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: acceso inmediato tras aplicar*
- **Given** que he enviado una aplicación exitosamente y recibo el email de confirmación,
- **When** hago clic en "Ver mi candidatura",
- **Then** accedo en menos de 1,5 segundos a una página que muestra nombre de la vacante, etapa actual con label legible (no el código interno), nombre del recruiter responsable y un mensaje de "Qué esperar a continuación", sin formulario de login.

*Escenario 2 — Edge case: etapa actualizada*
- **Given** que el recruiter avanzó mi candidatura de `screening` a `hm_review` hace 2 horas,
- **When** accedo al portal con mi `portal_token`,
- **Then** la etapa mostrada es "En revisión por el equipo", la fecha del último cambio es visible y el historial muestra la secuencia de etapas recorridas con fechas.

*Escenario 3 — Error: candidatura rechazada*
- **Given** que mi candidatura fue marcada como `rejected`,
- **When** accedo al portal,
- **Then** el portal muestra un mensaje de cierre empático, el motivo genérico si fue configurado por el recruiter, y el botón "Unirme al talent pool de [empresa]" que registra `Candidate.talent_pool = true`.

**Estimación de complejidad:** M

---

#### US-04 — Notificaciones proactivas por WhatsApp y email en cada cambio de estado

**Historia:**
Como candidato con una aplicación activa, quiero recibir una notificación por WhatsApp y email cada vez que mi candidatura avanza de etapa, para estar siempre informado sin tener que consultar el portal activamente.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: notificación en cambio de etapa*
- **Given** que mi candidatura está en estado `screening`,
- **When** el recruiter avanza mi candidatura a `hm_review`,
- **Then** en menos de 2 minutos recibo un email con asunto "Tu candidatura para [nombre vacante] avanzó de etapa" y, si WhatsApp está habilitado, un mensaje por ese canal, ambos conteniendo un enlace a mi portal de seguimiento.

*Escenario 2 — Happy path: notificación de rechazo*
- **Given** que mi candidatura fue rechazada,
- **When** el recruiter registra el rechazo en el sistema,
- **Then** recibo un email con tono empático que incluye el nombre de la vacante y el motivo genérico si fue configurado, sin revelar el campo `rejection_note` interno, y no recibo comunicaciones posteriores sobre esa vacante.

*Escenario 3 — Edge case: deduplicación ante cambios rápidos*
- **Given** que mi candidatura cambia de etapa dos veces en menos de 30 minutos,
- **When** el sistema procesa ambos cambios,
- **Then** el Deduplication Guard envía únicamente la notificación del estado más reciente, con `Notification.status = "skipped"` para la primera, evitando mensajes duplicados o contradictorios.

**Estimación de complejidad:** M

---

### Épica E2 — HM Inbox y Adopción del Hiring Manager

> **Diferenciador atacado:** D2 — HM Inbox: decisión en 30 segundos desde el móvil
> **Pain del mercado:** Pain #2 (baja adopción de hiring managers, 90% de impacto)

---

#### US-05 — Acceso al HM Inbox por magic link sin login

**Historia:**
Como hiring manager, quiero acceder a la tarjeta de un candidato directamente desde el email sin iniciar sesión en el ATS, para poder tomar una decisión en 30 segundos desde mi móvil sin aprender a usar ningún sistema.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: apertura exitosa del magic link*
- **Given** que soy un hiring manager que recibe el email de alerta con el magic link,
- **When** hago clic en el enlace desde cualquier dispositivo,
- **Then** en menos de 1 segundo se valida el token (existe en `magic_links`, `expires_at > NOW()`, `used_at IS NULL`) y se muestra la tarjeta del candidato con los tres botones de acción (Avanzar / Rechazar / Necesito más información), sin formulario de login.

*Escenario 2 — Error: magic link expirado*
- **Given** que han pasado más de 72 horas desde que el MagicLink fue generado,
- **When** el HM hace clic en el enlace,
- **Then** el sistema muestra "Este enlace ha expirado. Pide al recruiter que te envíe uno nuevo." sin exponer datos del candidato ni de la vacante, y retorna HTTP 410 Gone.

*Escenario 3 — Error: magic link ya utilizado*
- **Given** que el HM ya usó el magic link y tomó una decisión (`MagicLink.used_at IS NOT NULL`),
- **When** intenta usar el mismo enlace,
- **Then** el sistema muestra "Este enlace ya fue utilizado. Inicia sesión para revisar este candidato." con un enlace al login estándar, retorna HTTP 410 Gone, y no registra un segundo uso en `AuditLog`.

**Estimación de complejidad:** M

---

#### US-06 — Tarjeta de candidato con resumen de IA para decisión rápida

**Historia:**
Como hiring manager, quiero ver una tarjeta del candidato con un resumen generado por IA, el score de compatibilidad con el rol y el CV adjunto, para tomar una decisión informada sin necesidad de leer el CV completo ni coordinar reuniones con el recruiter.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: contenido mínimo de la tarjeta*
- **Given** que accedo al HM Inbox mediante un magic link válido,
- **When** la tarjeta del candidato termina de cargar,
- **Then** en menos de 2 segundos veo: foto o avatar, nombre completo, resumen IA de máximo 5 líneas (o "Perfil en análisis..." si `ai_summary = null`), score de fit con etiqueta (Alto ≥ 0,70 · Medio 0,40–0,69 · Bajo < 0,40) y enlace al CV en PDF con URL firmada válida.

*Escenario 2 — Happy path: decisión Avanzar*
- **Given** que estoy revisando la tarjeta de un candidato en el HM Inbox,
- **When** hago clic en "Avanzar",
- **Then** `Application.stage` cambia de `hm_review` a `interview`, `MagicLink.used_at` recibe el timestamp en la misma transacción, el recruiter recibe una notificación push en menos de 30 segundos, y la tarjeta desaparece de mi bandeja.

*Escenario 3 — Edge case: solicitud de más información*
- **Given** que hago clic en "Necesito más información" y escribo mi pregunta en el campo de texto,
- **When** envío la nota,
- **Then** `Application.stage` permanece en `hm_review`, el recruiter recibe una notificación con el texto exacto de la nota y el nombre del HM, y la tarjeta sigue visible con el label "Información solicitada".

**Estimación de complejidad:** M

---

#### US-07 — Recordatorios automáticos al HM sin respuesta

**Historia:**
Como recruiter, quiero que el sistema envíe automáticamente un recordatorio al hiring manager si no responde en 24 horas, para no tener que hacer seguimiento manual y evitar que el pipeline se paralice esperando su feedback.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: recordatorio automático a las 24 horas*
- **Given** que se envió una alerta al HM con un candidato para revisar,
- **When** pasan exactamente 24 horas sin que `MagicLink.used_at` tenga valor ni la Application haya cambiado de `hm_review`,
- **Then** el sistema encola un job en BullMQ que genera un nuevo MagicLink (token de 256 bits, `expires_at = NOW() + 72h`) y envía un email al HM con asunto "Recordatorio: tienes un candidato pendiente de revisión — [nombre vacante]" sin intervención del recruiter.

*Escenario 2 — Happy path: alerta al recruiter a las 48 horas*
- **Given** que han pasado 48 horas desde la alerta inicial al HM sin ninguna respuesta,
- **When** el job de 48h se ejecuta,
- **Then** el recruiter responsable de esa Application recibe una notificación in-app y por email con nombre del HM, nombre del candidato, nombre de la vacante y tiempo transcurrido, con un botón "Ver candidato" que enlaza directamente a la Application.

*Escenario 3 — Edge case: cancelación del recordatorio si el HM responde*
- **Given** que el HM tomó una decisión antes de cumplirse las 24 horas,
- **When** el sistema evalúa si encolar el job de recordatorio,
- **Then** el job está cancelado (`BullMQ removeJob`) y no se envía ninguna comunicación adicional al HM para esa Application en ese ciclo.

**Estimación de complejidad:** S

---

### Épica E3 — Coordinación de Entrevistas

> **Diferenciador atacado:** D3 — IA que ejecuta acciones, no que sugiere
> **Pain del mercado:** Pain #8 (burnout de equipos de TA por falta de automatización, 45% de impacto)

---

#### US-08a — Consulta de disponibilidad de entrevistadores y propuesta de slots

**Historia:**
Como recruiter, quiero que el sistema consulte automáticamente el calendario de los entrevistadores y me proponga tres horarios disponibles, para eliminar el intercambio manual de emails de coordinación.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: propuesta de slots basada en disponibilidad real*
- **Given** que selecciono una candidatura, el tipo de entrevista "técnica" y dos entrevistadores,
- **When** hago clic en "Buscar disponibilidad",
- **Then** en menos de 5 segundos el sistema retorna exactamente 3 slots disponibles en los próximos 5 días hábiles sin solapamiento con eventos existentes en Google Calendar u Outlook de los entrevistadores.

*Escenario 2 — Edge case: disponibilidad insuficiente*
- **Given** que solo uno de los dos entrevistadores tiene disponibilidad en los próximos 5 días hábiles,
- **When** el sistema no puede encontrar 3 slots comunes,
- **Then** muestra un aviso con el nombre del entrevistador sin disponibilidad y sugiere ampliar a 10 días o cambiar uno de los participantes, sin enviar ninguna comunicación al candidato.

*Escenario 3 — Error: token de calendario expirado*
- **Given** que el token OAuth de Google Calendar de un entrevistador ha expirado,
- **When** el sistema intenta consultar su calendario,
- **Then** marca ese entrevistador con un badge "Calendario no sincronizado", solicita al entrevistador que reconecte su calendario, y no bloquea la coordinación con los demás participantes.

**Estimación de complejidad:** M

---

#### US-08b — Confirmación de horario por el candidato y creación de evento en calendarios

**Historia:**
Como candidato y como recruiter, quiero que el candidato pueda seleccionar su horario preferido desde el email y que el sistema cree automáticamente el evento en todos los calendarios, para que la entrevista quede agendada sin intervención adicional del recruiter.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: confirmación y creación de evento*
- **Given** que el candidato recibe el email con 3 slots propuestos y hace clic en su horario preferido,
- **When** confirma la selección,
- **Then** en menos de 30 segundos se crean eventos en los calendarios de todos los participantes con el enlace de videollamada generado automáticamente si el tipo es `video`, y `Interview.status` pasa a `confirmed`.

*Escenario 2 — Edge case: reprogramación por el candidato*
- **Given** que el candidato quiere reprogramar después de haber confirmado,
- **When** hace clic en "Cambiar horario" del email de confirmación,
- **Then** accede a 3 slots actualizados según la disponibilidad vigente sin contactar al recruiter, y los eventos de calendario anteriores son actualizados automáticamente.

*Escenario 3 — Error: candidato no responde en 48 horas*
- **Given** que el candidato no responde a la propuesta de slots en 48 horas,
- **When** el sistema detecta la falta de respuesta,
- **Then** envía un recordatorio automático al candidato y una alerta al recruiter con el nombre del candidato y el tiempo transcurrido desde la propuesta.

**Estimación de complejidad:** M

---

### Épica E4 — Copiloto de IA y Automatización

> **Diferenciador atacado:** D3 — IA que ejecuta acciones · D1 — Candidate experience
> **Pains del mercado:** Pain #8 (burnout TA, 45%) · Pain #3 (comunicación deficiente, 85%)

---

#### US-09a — Parser de CV con extracción de datos estructurados

**Historia:**
Como sistema ATS, necesito extraer automáticamente datos estructurados del CV del candidato al recibir una nueva aplicación, para que el perfil del candidato esté normalizado y disponible para scoring y búsqueda desde el momento de la aplicación.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: extracción exitosa de CV en PDF*
- **Given** que un candidato envía una aplicación con un PDF de texto seleccionable,
- **When** el job de parsing se ejecuta en la cola BullMQ,
- **Then** en menos de 15 segundos `Candidate.parsed_experience`, `Candidate.parsed_education` y `Candidate.parsed_skills` están populados con JSONB válido según el schema definido, con al menos el nombre de la última empresa, el cargo y el período de la última experiencia extraídos correctamente.

*Escenario 2 — Edge case: CV con formato complejo*
- **Given** que el CV es un DOCX con tablas de dos columnas,
- **When** el parser procesa el documento,
- **Then** extrae correctamente nombre, email, última empresa y cargo, y registra en `AuditLog action: "cv_parse_partial"` con `new_values: {unextracted_sections: ["education"]}`.

*Escenario 3 — Error: CV escaneado sin capa de texto*
- **Given** que el CV subido es una imagen JPG escaneada,
- **When** el parser intenta procesarlo,
- **Then** registra `parsed_experience = null`, `parsed_skills = []`, genera una `Notification` de tipo `cv_parse_failed` para el recruiter asignado, y `Application` continúa en estado `applied` sin bloqueo del flujo.

**Estimación de complejidad:** M

---

#### US-09b — Generación de score de fit y resumen IA por candidatura

**Historia:**
Como recruiter, quiero que el sistema genere automáticamente un score de compatibilidad y un resumen en lenguaje natural de cada candidato contra la vacante, para priorizar qué perfiles revisar primero sin leer cada CV completo de forma manual.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: generación automática al recibir aplicación*
- **Given** que el parser completó la extracción de datos de un candidato (US-09a),
- **When** el sistema procesa la candidatura contra los requisitos de la vacante,
- **Then** en menos de 60 segundos `Application.ai_fit_score` (valor 0,00–1,00) y `Application.ai_summary` (máximo 5 líneas) están disponibles en la tarjeta del candidato en el Kanban del recruiter.

*Escenario 2 — Edge case: datos insuficientes para scoring*
- **Given** que el CV tiene menos de 2 experiencias laborales estructuradas,
- **When** el sistema intenta generar el score,
- **Then** registra `ai_fit_score = null` y muestra en la tarjeta el badge "Score no disponible — datos insuficientes", sin bloquear el flujo manual del recruiter.

*Escenario 3 — Gobernanza de IA: el score no bloquea decisiones humanas*
- **Given** que un candidato tiene `ai_fit_score = 0.25` y el recruiter decide avanzarlo manualmente,
- **When** el recruiter confirma la acción de avance,
- **Then** el sistema registra en `AuditLog`: `action: "stage_change"`, `user_id: <recruiter_id>`, `old_values: {stage: "screening", ai_fit_score: 0.25}`, `new_values: {stage: "hm_review"}`, sin mostrar ningún mensaje de advertencia al recruiter.

**Estimación de complejidad:** M

---

#### US-10 — Redacción y envío automático de comunicaciones personalizadas

**Historia:**
Como recruiter, quiero que el sistema genere y envíe automáticamente los emails de rechazo, confirmación de entrevista y seguimiento de candidatos, para mantener una comunicación constante y empática sin dedicar tiempo manual a redactar cada mensaje.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: email de rechazo generado automáticamente*
- **Given** que un recruiter marca una candidatura como rechazada seleccionando el motivo genérico,
- **When** confirma la acción de rechazo,
- **Then** el sistema envía automáticamente un email al candidato con un mensaje personalizado (nombre del candidato, nombre de la vacante, tono empático) sin revelar el motivo interno detallado, en menos de 5 minutos tras el rechazo.

*Escenario 2 — Edge case: revisión antes de enviar*
- **Given** que el recruiter ha activado la opción "Revisar antes de enviar",
- **When** se genera un email automático de cualquier tipo,
- **Then** el recruiter recibe el borrador del email para aprobarlo o editarlo, con un plazo máximo de 2 horas antes del envío automático.

*Escenario 3 — Happy path: confirmación de entrevista automática*
- **Given** que un candidato ha confirmado su horario de entrevista,
- **When** el sistema registra la confirmación,
- **Then** se envía automáticamente al candidato un email con todos los detalles (fecha, hora, formato, enlace de videollamada, nombre de los entrevistadores) sin intervención del recruiter.

**Estimación de complejidad:** M

---

### Épica E5 — Analytics e Inteligencia de Negocio

> **Diferenciador atacado:** D4 — Analytics accionables por rol, sin exportar a Excel
> **Pain del mercado:** Pain #5 (reporting limitado y dashboards no accionables, 72% de impacto)

---

#### US-11a — Dashboard de métricas operativas para recruiter

**Historia:**
Como recruiter, quiero ver en un panel mis métricas de pipeline activo (candidatos por etapa, tiempo promedio en cada etapa, tasa de respuesta de HMs) sin exportar a Excel, para identificar cuellos de botella en mis vacantes asignadas.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: acceso al dashboard*
- **Given** que accedo a la sección "Mis métricas" con rol `recruiter`,
- **When** selecciono el período "Último mes",
- **Then** veo en menos de 3 segundos: número de candidatos activos por etapa, tiempo promedio de permanencia en cada etapa (en días) y tasa de respuesta de HMs (% con decisión en < 48h).

*Escenario 2 — Edge case: filtro por vacante*
- **Given** que tengo 5 vacantes activas,
- **When** selecciono una vacante específica en el filtro,
- **Then** todas las métricas se recalculan para esa vacante únicamente, incluyendo el funnel de conversión etapa por etapa.

*Escenario 3 — Error: vacante sin datos suficientes*
- **Given** que una vacante fue creada hace menos de 3 días y no tiene candidatos,
- **When** accedo al dashboard filtrado por esa vacante,
- **Then** el sistema muestra el panel con valores en cero y el mensaje "Aún no hay datos suficientes para este período", sin errores de carga.

**Estimación de complejidad:** S

---

#### US-11b — Dashboard estratégico para TA Lead y VP People

**Historia:**
Como TA Director, quiero un dashboard con métricas estratégicas de reclutamiento (time-to-hire global, coste por contratación, NPS de candidato, source of hire) con alertas automáticas cuando un KPI supera el umbral configurado, para demostrar el ROI del equipo al C-suite con datos en tiempo real.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: acceso al dashboard estratégico*
- **Given** que accedo con rol `ta_lead` a la sección Analytics,
- **When** selecciono "Último trimestre",
- **Then** veo time-to-hire promedio en días, fuente de contratación con porcentaje por canal, NPS promedio de candidatos del período y coste estimado por contratación, todo calculado en menos de 5 segundos.

*Escenario 2 — Happy path: alerta automática por KPI fuera de rango*
- **Given** que el time-to-hire promedio supera los 45 días durante dos semanas consecutivas,
- **When** el sistema calcula las métricas del período activo,
- **Then** el TA Director recibe un email de alerta con el valor actual, el umbral configurado, las 3 vacantes que más contribuyen a la desviación y un enlace al dashboard filtrado.

*Escenario 3 — Edge case: filtro por recruiter*
- **Given** que aplico un filtro por nombre de recruiter,
- **When** el dashboard recalcula,
- **Then** todas las métricas muestran únicamente candidaturas con `assigned_recruiter_id` igual al recruiter seleccionado, incluyendo su time-to-hire individual y tasa de completitud de scorecards.

**Estimación de complejidad:** M

---

### Épica E6 — Integraciones y Ecosistema

> **Diferenciadores atacados:** D5 — Integración HRIS bidireccional profunda · D7 — Pricing transparente
> **Pains del mercado:** Pain #4 (integración HRIS incompleta, 78%) · Pain #6 (pricing opaco, 65%)

---

#### US-12a — Career site propio con vacantes activas y formulario de aplicación

**Historia:**
Como recruiter, quiero que las vacantes publicadas en el ATS aparezcan automáticamente en un career site personalizable de la empresa, para que los candidatos puedan encontrarlas y aplicar directamente sin intermediarios.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: publicación en career site*
- **Given** que publico una vacante en estado `published` en el ATS,
- **When** accedo al career site público de la organización (`empresa.ats.io/careers`),
- **Then** la vacante aparece listada en menos de 2 minutos con título, departamento, ubicación y tipo de empleo, y el botón "Aplicar" lleva al formulario definido en US-01a.

*Escenario 2 — Edge case: vacante cerrada*
- **Given** que una vacante pasa a estado `closed`,
- **When** el sistema actualiza el estado,
- **Then** la vacante desaparece del career site en menos de 5 minutos y cualquier URL directa muestra "Esta posición ya no está disponible".

*Escenario 3 — Edge case: personalización de marca*
- **Given** que el admin sube el logo y colores de la empresa en la configuración,
- **When** un candidato accede al career site,
- **Then** ve el logo de la empresa en el header, el color primario en los botones y el nombre de la empresa en el título, sin ninguna mención a la marca del ATS.

**Estimación de complejidad:** M

---

#### US-12b — Publicación en job boards externos (LinkedIn e Indeed)

**Historia:**
Como recruiter, quiero publicar mis vacantes en LinkedIn e Indeed con un solo clic desde el ATS, para multiplicar el alcance de cada vacante sin entrar a cada plataforma por separado.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: publicación simultánea*
- **Given** que una vacante está en estado `published` y los tokens de LinkedIn e Indeed están vigentes,
- **When** selecciono ambos canales y hago clic en "Distribuir",
- **Then** en menos de 5 minutos el sistema muestra el estado de cada publicación (✅ Publicado) y registra el `job_board_id` externo de cada plataforma en el ATS.

*Escenario 2 — Error: fallo parcial en un canal*
- **Given** que el token de Indeed ha expirado pero el de LinkedIn está vigente,
- **When** intento publicar en ambos,
- **Then** LinkedIn se publica correctamente, el sistema muestra "⚠️ Indeed: token expirado — reconectar integración" y el botón de reintento para Indeed está disponible sin necesidad de reconfigurar LinkedIn.

*Escenario 3 — Happy path: trazabilidad de fuente*
- **Given** que un candidato aplica desde la publicación de Indeed,
- **When** su `Application` se crea en el ATS,
- **Then** `Candidate.source = "indeed"` y este valor es visible en la tarjeta del candidato y contribuye a las métricas de "source of hire" en US-11b.

**Estimación de complejidad:** M

---

#### US-13a — Integración HRIS unidireccional con BambooHR al contratar

**Historia:**
Como administrador, quiero que cuando un candidato sea marcado como contratado sus datos se transfieran automáticamente a BambooHR, para eliminar el trabajo manual de 3–5 horas por contratación que actualmente realiza el equipo de RRHH.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: sincronización automática al contratar*
- **Given** que `Application.stage` cambia a `hired` y BambooHR está configurado con credenciales válidas,
- **When** el sistema procesa el evento,
- **Then** en menos de 15 minutos se crea en BambooHR un registro de empleado con nombre, apellido, email, cargo, departamento y fecha de incorporación, y el ATS registra `sync_status: "success"` en `AuditLog` con el ID del empleado creado.

*Escenario 2 — Error: fallo de sincronización*
- **Given** que la API de BambooHR devuelve error 401 por credenciales inválidas,
- **When** el sistema intenta la sincronización,
- **Then** reintenta 3 veces con backoff exponencial (1s, 2s, 4s), notifica al admin con email detallando el candidato y el error, registra `sync_status: "failed"` en `AuditLog`, y no revierte el estado `hired` de la Application.

*Escenario 3 — Edge case: configuración de la integración*
- **Given** que soy admin y accedo a "Integraciones > BambooHR",
- **When** ingreso el subdominio y la API key y hago clic en "Verificar conexión",
- **Then** el sistema realiza un GET a la API de BambooHR y muestra "✅ Conexión exitosa — X empleados encontrados" o el mensaje de error específico en menos de 10 segundos.

**Estimación de complejidad:** M

---

#### US-13b — Integración HRIS con Workday *(Fase 3)*

**Historia:**
Como administrador de una organización con más de 500 empleados, quiero configurar la sincronización bidireccional con Workday para que todos los datos del candidato contratado se transfieran automáticamente al HRIS enterprise, para eliminar el trabajo manual post-contratación y garantizar la consistencia de datos en el sistema de RRHH.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: sincronización completa con Workday*
- **Given** que `Application.stage` cambia a `hired` y Workday está configurado con las credenciales SOAP/REST de la organización,
- **When** el sistema procesa el evento,
- **Then** en menos de 30 minutos se crea un registro de trabajador en Workday con todos los campos mapeados según la configuración de la organización, incluyendo compensación, código de posición y entidad legal.

*Escenario 2 — Edge case: mapeo de campos personalizado*
- **Given** que soy admin y necesito ajustar el mapeo entre campos del ATS y campos de Workday,
- **When** accedo al configurador visual de integración,
- **Then** puedo modificar la correspondencia campo a campo sin editar código, y el sistema valida que los campos obligatorios de Workday tienen un mapeo asignado antes de guardar.

*Escenario 3 — Error: fallo de autenticación Workday*
- **Given** que las credenciales de Workday han expirado,
- **When** el sistema intenta sincronizar un candidato contratado,
- **Then** el admin recibe una alerta inmediata con el detalle del error y los datos no sincronizados, y la Application permanece en estado `hired` sin ser revertida.

**Estimación de complejidad:** M

---

### Épica E7 — Configuración, Compliance y Administración

> **Diferenciadores atacados:** D6 — Workflows 100% configurables sin código
> **Pains del mercado:** Pain #7 (workflows rígidos, 55% de impacto) · Compliance GDPR (bloqueador contractual)

---

#### US-00 — Autenticación base y gestión de roles de usuarios internos *(Historia nueva — habilitadora)*

**Historia:**
Como administrador del sistema, quiero crear usuarios internos con roles diferenciados (recruiter, hiring_manager, ta_lead, viewer), invitarlos por email y que se autentiquen con email más contraseña o SSO Google, para que cada usuario acceda únicamente a las funciones correspondientes a su rol desde el primer día de uso.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: invitación y activación de usuario*
- **Given** que soy admin e invito a `recruiter@empresa.com` asignando rol `recruiter`,
- **When** el sistema procesa la invitación,
- **Then** se crea un registro en `users` con `role = "recruiter"` e `is_active = false`, y se envía un email con enlace de activación `https://app.lti.com/activate?token=<uuid>` válido por exactamente 48 horas.

*Escenario 2 — Happy path: login con SSO Google*
- **Given** que la organización tiene SSO Google configurado con dominio `@empresa.com` y el usuario hace clic en "Entrar con Google",
- **When** el callback OAuth2 retorna el `id_token` válido,
- **Then** el sistema crea la sesión JWT (access token 1h, refresh token 30d), registra en `AuditLog action: "login"` con `user_id`, `ip_address`, `user_agent` y `created_at`, y redirige al dashboard del rol correspondiente en menos de 3 segundos.

*Escenario 3 — Error: acceso no autorizado por rol*
- **Given** que un request llega con JWT de rol `viewer` al endpoint `PATCH /applications/:id/stage`,
- **When** el middleware RBAC evalúa el token,
- **Then** retorna HTTP 403 `{"error": {"code": "INSUFFICIENT_PERMISSIONS"}}` en menos de 200ms sin ejecutar ninguna lógica de negocio.

**Estimación de complejidad:** M

---

#### US-14a — Configuración de stages personalizados del pipeline sin código

**Historia:**
Como administrador, quiero añadir, renombrar y reordenar etapas del pipeline desde una interfaz visual, para adaptar el proceso de selección de mi empresa sin depender del equipo de IT.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: creación de stage personalizado*
- **Given** que accedo al configurador de pipeline como admin,
- **When** añado un stage "Test técnico" entre `screening` y `hm_review` con nombre, descripción y color,
- **Then** el nuevo stage aparece en el Kanban de todas las vacantes activas en menos de 30 segundos, la máquina de estados acepta la transición `screening → test_técnico → hm_review`, y las vacantes existentes con candidatos en `screening` no son afectadas retroactivamente.

*Escenario 2 — Edge case: eliminación de stage con candidatos activos*
- **Given** que intento eliminar un stage que tiene candidatos activos en él,
- **When** hago clic en "Eliminar etapa",
- **Then** el sistema muestra un aviso con el número de candidatos afectados y exige especificar a qué etapa moverlos antes de confirmar, sin pérdida de datos.

*Escenario 3 — Error: límite de stages del plan*
- **Given** que intento crear el stage número 11 (el límite del plan es 10 stages personalizados),
- **When** confirmo la creación,
- **Then** el sistema bloquea la acción y muestra "Has alcanzado el límite de 10 etapas personalizadas de tu plan Growth" con un enlace al comparador de planes.

**Estimación de complejidad:** M

---

#### US-14b — Triggers automáticos entre etapas del pipeline *(Fase 3)*

**Historia:**
Como administrador, quiero configurar acciones automáticas que se ejecuten cuando un candidato avanza a una etapa específica, para que el recruiter no tenga que recordar pasos manuales del proceso.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: trigger de envío de email al avanzar de etapa*
- **Given** que configuro el trigger "Al avanzar a Test técnico → Enviar email 'Instrucciones del test' al candidato",
- **When** un recruiter mueve un candidato a esa etapa,
- **Then** en menos de 2 minutos el candidato recibe el email con el template configurado y el evento queda en `Notification.status = "delivered"`.

*Escenario 2 — Error: template del trigger eliminado*
- **Given** que el template de email configurado en el trigger fue eliminado posteriormente,
- **When** un recruiter activa el trigger moviendo un candidato,
- **Then** el sistema ejecuta la transición de etapa correctamente pero registra `Notification.status = "failed"` con motivo "Template no encontrado" y notifica al admin con una alerta de configuración rota.

*Escenario 3 — Edge case: trigger de solicitud de scorecard*
- **Given** que configuro un trigger de tipo "Solicitar scorecard al entrevistador X al avanzar a Entrevista final",
- **When** el candidato llega a esa etapa,
- **Then** se genera automáticamente un `InterviewParticipant` con `scorecard_submitted = false` para el entrevistador configurado y se envía la solicitud de scorecard por email.

**Estimación de complejidad:** M

---

#### US-15 — Gestión de consentimiento GDPR y eliminación de datos de candidato

**Historia:**
Como administrador del sistema, quiero disponer de un mecanismo para eliminar los datos personales de un candidato a su solicitud y asegurar la purga automática de perfiles inactivos a los 12 meses, para cumplir con el GDPR y evitar riesgos de compliance para la organización.

**Criterios de aceptación (Given / When / Then):**

*Escenario 1 — Happy path: eliminación de datos a solicitud*
- **Given** que un candidato solicita el borrado de sus datos desde el portal personal,
- **When** el sistema procesa la solicitud,
- **Then** los campos `first_name`, `last_name`, `email`, `phone`, `linkedin_url` y `cv_url` del `Candidate` son reemplazados por `"[ANONIMIZADO]"` o `null`, el archivo CV es eliminado de S3/R2, y `AuditLog` registra `action: "gdpr_erasure"` con los campos anonimizados en `new_values`.

*Escenario 2 — Happy path: purga automática a 12 meses*
- **Given** que un candidato no tiene Applications activas y han transcurrido 12 meses desde `Candidate.created_at`,
- **When** el job de purga programado (cron diario a las 2:00 AM UTC) se ejecuta,
- **Then** el candidato es anonimizado automáticamente y el evento queda registrado en `AuditLog` sin notificación al recruiter.

*Escenario 3 — Error: intento de aplicar sin consentimiento GDPR*
- **Given** que un candidato intenta enviar el formulario sin marcar el checkbox de consentimiento,
- **When** hace clic en "Enviar",
- **Then** el endpoint retorna HTTP 422 `{"error": {"code": "GDPR_CONSENT_REQUIRED", "message": "El consentimiento es obligatorio para procesar tu aplicación"}}` y no crea ningún registro en `Candidate` ni en `Application`.

**Estimación de complejidad:** M

---

## 2. Metodología de Priorización

Usé **WSJF (Weighted Shortest Job First)** con el Pain Score del PRD como ancla de valor, introducido progresivamente en tres fases: Pain Score + MoSCoW para definir el Must Have del MVP (semanas 1–4), WSJF simplificado con Valor y Urgencia para la fase de validación (semanas 5–12), y WSJF completo sumando Habilitación Técnica a partir del mes 4. La escala es Fibonacci (1·2·3·5·8·10), el tamaño se normaliza como S=1·M=2·L=4 (todas las historias L fueron subdivididas antes de entrar al backlog), y las historias de compliance reciben Urgencia Temporal = 10 por ser bloqueadores contractuales independientemente de su Pain Score.

Elegí WSJF porque obliga a considerar simultáneamente el valor que entrega una historia y el costo de no hacerla ahora —algo que MoSCoW solo no captura—, y porque anclar el valor al Pain Score del PRD elimina el sesgo hacia lo técnicamente interesante frente a lo estratégicamente crítico. La introducción por fases responde a una limitación real: en las primeras semanas no tenemos datos suficientes para puntuar Habilitación Técnica de forma fiable, así que es mejor no fingir precisión donde no la hay.

---

## 3. Product Backlog Priorizado

### Fase 1 — MVP Must Have (Semanas 1–4)

> **Objetivo:** producto demostrable a design partners al finalizar la semana 4.

| Pos. | ID | Título | Épica | Valor | Urgencia | Tamaño | WSJF | MoSCoW | Sprint |
|---|---|---|---|---|---|---|---|---|---|
| 1 | US-07 | Recordatorios automáticos al HM | E2 | 9 | 3 | S(1) | **12,00** | M | 1 |
| 2 | US-00 | Autenticación base y roles | E7 | 10 | 10 | M(2) | **10,00** | M | 1 |
| 3 | US-01a | Formulario de aplicación mínimo | E1 | 10 | 10 | M(2) | **10,00** | M | 1 |
| 4 | US-05 | Magic link HM Inbox | E2 | 9 | 10 | M(2) | **9,50** | M | 1 |
| 5 | US-09a | Parser de CV estructurado | E4 | 10 | 8 | M(2) | **9,00** | M | 2 |
| 6 | US-03 | Portal de seguimiento candidato | E1 | 8 | 10 | M(2) | **9,00** | M | 2 |
| 7 | US-15 | GDPR: consentimiento y borrado | E7 | 5 | 10 | M(2) | **7,50** | M | 2 |
| 8 | US-06 | Tarjeta candidato HM Inbox | E2 | 9 | 5 | M(2) | **7,00** | M | 2 |
| 9 | US-04 | Notificaciones email y WhatsApp | E1 | 8 | 5 | M(2) | **6,50** | M | 2 |

**Dependencias de la Fase 1:**

- US-07 no puede iniciarse sin US-05: el recordatorio genera un nuevo `MagicLink` del mismo tipo que el original.
- US-00 no tiene dependencias: es la historia raíz del sistema.
- US-01a depende de US-00: la Application requiere `organization_id` del tenant extraído del JWT.
- US-05 depende de US-00: el `MagicLink.user_id` referencia un `User` con `role = "hiring_manager"`.
- US-09a depende de US-01a: el parser opera sobre `cv_url` almacenado al crear la Application.
- US-03 depende de US-01a: el portal requiere una Application existente con `portal_token`.
- US-15 depende de US-01a: el consentimiento GDPR es parte del formulario de aplicación.
- US-06 depende de US-05: la tarjeta solo es accesible mediante un MagicLink válido.
- US-04 depende de US-01a y US-03: las notificaciones se disparan sobre Applications existentes y enlazan al portal.

---

### Fase 2 — Validación (Semanas 5–12)

> **Objetivo:** refinar el producto con feedback de design partners y completar coordinación, IA, analytics e integraciones básicas.
> **Metodología:** `WSJF = (Valor + Urgencia) / Tamaño`

| Pos. | ID | Título | Épica | Valor | Urgencia | Tamaño | WSJF | MoSCoW | Sprint |
|---|---|---|---|---|---|---|---|---|---|
| 1 | US-11a | Dashboard métricas recruiter | E5 | 7 | 3 | S(1) | **10,00** | S | 3 |
| 2 | US-09b | Score IA y resumen candidato | E4 | 9 | 5 | M(2) | **7,00** | S | 3 |
| 3 | US-01b | Importación LinkedIn + parser | E1 | 10 | 3 | M(2) | **6,50** | S | 3 |
| 4 | US-08a | Consulta disponibilidad calendarios | E3 | 4 | 8 | M(2) | **6,00** | S | 4 |
| 5 | US-10 | Comunicaciones automáticas | E4 | 8 | 3 | M(2) | **5,50** | S | 4 |
| 6 | US-12a | Career site propio | E6 | 6 | 5 | M(2) | **5,50** | S | 4 |
| 7 | US-13a | Integración BambooHR | E6 | 8 | 3 | M(2) | **5,50** | C | 5 |
| 8 | US-11b | Dashboard estratégico TA Lead | E5 | 7 | 2 | M(2) | **4,50** | C | 5 |
| 9 | US-08b | Confirmación horario + calendarios | E3 | 4 | 5 | M(2) | **4,50** | S | 5 |
| 10 | US-14a | Stages personalizados sin código | E7 | 5 | 3 | M(2) | **4,00** | S | 6 |
| 11 | US-12b | Publicación LinkedIn / Indeed | E6 | 6 | 2 | M(2) | **4,00** | C | 6 |

**Dependencias de la Fase 2:**

- US-09b depende de US-09a: el score opera sobre los campos `parsed_experience` y `parsed_skills`.
- US-01b depende de US-09a: la importación de LinkedIn usa el mismo pipeline de normalización.
- US-08b depende de US-08a: la confirmación opera sobre los slots generados previamente.
- US-11b depende de US-11a: el dashboard estratégico comparte la capa de queries de aggregación.
- US-12b depende de US-12a: LinkedIn e Indeed enlazan al career site como destino de las aplicaciones.
- US-13a depende de US-06: la sincronización HRIS se dispara cuando `Application.stage = "hired"`.

---

### Fase 3 — Escala (Mes 4 en adelante)

> **Objetivo:** funcionalidades que requieren datos reales de clientes o acuerdos comerciales con proveedores enterprise.
> **Metodología:** `WSJF = (Valor + Urgencia + Habilitación Técnica) / Tamaño`

| ID | Título | Épica | Justificación de fase | WSJF F3 estimado |
|---|---|---|---|---|
| US-13b | Integración Workday | E6 | Requiere acuerdo comercial con Workday (API enterprise de acceso restringido) y datos reales de clientes Scale (> 500 empleados). El segmento objetivo del MVP usa principalmente BambooHR. | (8+1+8)/2 = **8,50** |
| US-14b | Triggers automáticos pipeline | E7 | Alta complejidad técnica en el motor de reglas. Requiere feedback de design partners sobre qué triggers son más demandados. Depende de US-14a. | (5+2+5)/2 = **6,00** |

---

### Métricas de éxito del MVP al finalizar la Fase 1

| Métrica | Objetivo | Fórmula de medición |
|---|---|---|
| NPS del candidato | > 50 | Encuesta post-aplicación vía `Notification` de tipo `nps_survey` al pasar a `hired` o `rejected` |
| Adopción de HMs | > 75% con ≥ 1 decisión en primeras 2 semanas | `COUNT DISTINCT user_id WHERE role='hiring_manager' AND action='hm_decision'` / total HMs activos |
| Reducción time-to-hire | ≥ 30% vs. baseline del cliente | `stage_updated_at (hired) - created_at (Application)` vs. dato histórico del cliente |
| Tasa de completion del formulario | > 80% | `COUNT(Applications con stage ≠ 'draft')` / `COUNT(Applications creadas)` |

---

## 4. Tickets Técnicos — US-01a

> **Historia de referencia:** US-01a — Formulario de aplicación con campos mínimos y progreso adaptativo
> **Sprint:** 1
> **Épica:** E1 — Experiencia del Candidato

---

### TICKET-01 · [Refinamiento] Definir campos obligatorios del formulario de aplicación y comportamiento en tablet

**Descripción:**
Antes de iniciar el desarrollo del formulario, el equipo necesita respuestas concretas a dos preguntas sin respuesta en la documentación actual:

1. **Campos del formulario:** el PRD establece un máximo de 8 campos obligatorios pero no los enumera. Solo se conocen `email` (del BDD) y `cv_url` (del modelo de datos). Es necesario definir la lista completa de campos obligatorios y opcionales, su orden de aparición, y si esta lista varía según el tipo de vacante o es fija para el MVP.

2. **Comportamiento en tablet:** el BDD especifica el comportamiento para mobile (viewport < 768px) y se asume desktop para el resto, pero el rango 768px–1024px no está definido. Necesita aclararse si tablet usa el layout mobile, el layout desktop, o uno propio.

3. **Mecanismo de identificación del borrador:** si el candidato no tiene cuenta, ¿cómo se vincula el borrador guardado en DB con el candidato al retomar? ¿Cookie de sesión, token en URL, email como identificador? La respuesta determina el diseño del endpoint de auto-save y la query de recuperación.

4. **Email de reanudación:** el escenario 2 de US-01a menciona "enlace de reanudación enviado por email". US-04 (notificaciones) está en Sprint 2. ¿El envío de este email es parte de US-01a o una dependencia de US-04? Si es parte de US-01a, necesita un template mínimo.

**Criterios de Aceptación:**
- El equipo de producto entrega la lista completa de campos (nombre, tipo, obligatorio/opcional) antes del día 2 del Sprint 1.
- Se define el comportamiento de layout para viewport 768px–1024px.
- Se define el mecanismo de identificación de borradores para candidatos anónimos.
- Se define si el email de reanudación es parte del alcance de US-01a o de US-04.

**Prioridad:** Crítica — bloquea TICKET-02, TICKET-03 y TICKET-04

**Estimación:** 2 puntos (sesión de refinamiento de 2–3 horas con PM + Tech Lead)

**Asignado a:** Product Manager + Tech Lead

**Etiquetas:** `refinamiento` `us-01a` `sprint-1` `bloqueador`

**Comentarios:** Este ticket debe cerrarse en el día 1 o 2 del sprint. Si no se resuelve, los tickets de frontend y backend no pueden comenzar.

**Dependencias:** Ninguna. Es el primer ticket a ejecutar.

---

### TICKET-02 · [Base de Datos] Crear tabla `applications` con soporte para estado `draft` y RLS por `organization_id`

**Descripción:**
Crear el schema de la tabla `applications` en PostgreSQL con todos los campos definidos en el modelo de datos del PRD (sección 9.1), habilitando Row-Level Security (RLS) por `organization_id` según la especificación de la sección 9.6. El estado `draft` debe ser un valor válido del ENUM `stage` y el punto de partida para toda nueva candidatura.

Incluye la creación de los índices definidos en el PRD:
```sql
CREATE INDEX idx_applications_org_job ON applications (organization_id, job_id);
CREATE INDEX idx_applications_org_stage ON applications (organization_id, stage);
CREATE INDEX idx_applications_org_recruiter ON applications (organization_id, assigned_recruiter_id);
```

Y el constraint de unicidad:
```sql
ALTER TABLE applications ADD CONSTRAINT uq_candidate_job UNIQUE (candidate_id, job_id);
```

También debe crearse la tabla `candidates` con los campos mínimos necesarios para US-01a: `id`, `organization_id`, `email`, `cv_url`, `gdpr_consent`, `gdpr_consent_at`, `portal_token`, `source`, `created_at`, `updated_at`. Los campos de parsing (`parsed_experience`, `parsed_education`, `parsed_skills`) se agregan en US-09a.

**Criterios de Aceptación:**
- La migración corre sin errores en entorno de desarrollo y staging.
- `Application.stage` acepta el valor `draft` y lo usa como default al crear una nueva candidatura.
- RLS habilitado: una query con `organization_id` de otro tenant no retorna filas de este tenant. Verificable con test de integración.
- El constraint `UNIQUE(candidate_id, job_id)` rechaza duplicados con error de DB controlado.
- Los tres índices existen y aparecen en `\d applications` de psql.
- La tabla `candidates` tiene el campo `portal_token` con constraint `UNIQUE`.

**Prioridad:** Alta

**Estimación:** 3 puntos

**Asignado a:** Backend

**Etiquetas:** `base-de-datos` `migraciones` `rls` `us-01a` `sprint-1`

**Comentarios:** Usar Drizzle ORM para definir el schema. La migración debe ser reversible (incluir `down`). Coordinar con TICKET-03 para que el endpoint de creación de Application esté alineado con el schema.

**Dependencias:** TICKET-01 (definición de campos), US-00 (la tabla `users` debe existir para las FK de `assigned_recruiter_id` y `created_by`)

---

### TICKET-03 · [Backend] Endpoint `POST /jobs/:jobId/applications` — Creación de candidatura con validación GDPR

**Descripción:**
Implementar el endpoint público (sin autenticación de usuario) `POST /jobs/:jobId/applications` que recibe los datos del formulario de aplicación y crea los registros correspondientes en `candidates` y `applications`.

Comportamiento requerido según el PRD (sección 12, módulo Pipeline):

- El endpoint es público (`🔓` en el contrato de API).
- El `organization_id` se extrae del `job_id` en la query (no viaja en el body).
- Si el candidato ya existe para esa organización (mismo email + `organization_id`), reutilizar el `Candidate` existente; no crear duplicado. El constraint `uq_candidate_email_org` maneja esto a nivel DB.
- Si la combinación `candidate_id + job_id` ya existe en `applications`, retornar HTTP 422 con `code: "DUPLICATE_APPLICATION"`.
- Si `gdpr_consent` no es `true` en el body, retornar HTTP 422 con `code: "GDPR_CONSENT_REQUIRED"` sin crear ningún registro.
- Al crear exitosamente: generar `portal_token` (token opaco de 32+ bytes), registrar `gdpr_consent_at = NOW()`, setear `Application.stage = "draft"`.
- Registrar en `AuditLog`: `action: "create"`, `entity_type: "application"`, `entity_id`, `organization_id`, con `form_duration_seconds` como metadata si el frontend lo envía.
- Retornar HTTP 201 con el `portal_token` y el `application_id` en la respuesta.

**Criterios de Aceptación:**
- `POST` con `gdpr_consent: false` retorna HTTP 422 `{"error": {"code": "GDPR_CONSENT_REQUIRED"}}` y no crea ningún registro en DB. Verificable con test de integración.
- `POST` con un par `candidate_id + job_id` ya existente retorna HTTP 422 `{"error": {"code": "DUPLICATE_APPLICATION"}}`.
- `POST` válido retorna HTTP 201 con `portal_token` y `application_id` en menos de 500ms (medido sin el tiempo del parser de CV que corre asíncronamente).
- El `portal_token` generado tiene longitud mínima de 64 caracteres hexadecimales.
- El `AuditLog` contiene el evento `create` con `form_duration_seconds` si fue enviado en el body.
- RLS: el endpoint solo accede a datos del tenant correspondiente al `job_id`. Test de integración con `job_id` de otro tenant retorna HTTP 404.

**Prioridad:** Alta

**Estimación:** 5 puntos

**Asignado a:** Backend

**Etiquetas:** `backend` `api` `gdpr` `us-01a` `sprint-1`

**Comentarios:** El endpoint debe estar completamente funcional antes de que el frontend (TICKET-05) pueda integrarse. El `organization_id` nunca debe viajar en el body del request; siempre se resuelve server-side desde el `job_id`. Coordinar con el equipo de QA para los tests de los escenarios de error.

**Dependencias:** TICKET-02 (schema de DB), TICKET-01 (campos del formulario)

---

### TICKET-04 · [Backend] Endpoint `PATCH /applications/:id` — Auto-save de borrador cada 30 segundos

**Descripción:**
Implementar el endpoint que recibe actualizaciones parciales del formulario en progreso y persiste el estado del borrador en `applications`. Este endpoint es llamado automáticamente por el frontend cada 30 segundos mientras el candidato tiene el formulario abierto.

Comportamiento requerido:
- Solo opera sobre Applications en estado `stage = "draft"`. Si el estado es otro, retorna HTTP 409.
- Actualiza los campos del formulario enviados en el body (parcial, no reemplaza todo el registro).
- Actualiza `updated_at`.
- No registra en `AuditLog` (operación de alta frecuencia; registrar cada auto-save generaría ruido).
- El endpoint es público o requiere el mismo mecanismo de identificación del candidato anónimo definido en TICKET-01.
- Retorna HTTP 200 con `{"saved_at": "<timestamp>"}`.

**Criterios de Aceptación:**
- `PATCH` sobre una Application en estado `draft` actualiza los campos enviados y retorna HTTP 200 en menos de 200ms.
- `PATCH` sobre una Application en estado distinto de `draft` retorna HTTP 409.
- Campos no enviados en el body no son modificados (update parcial, no reemplazo).
- El campo `updated_at` refleja el timestamp del último auto-save. Verificable con query directa a DB en test de integración.
- El endpoint no genera entradas en `AuditLog`. Verificable con count de `audit_logs` antes y después del PATCH.

**Prioridad:** Alta

**Estimación:** 3 puntos

**Asignado a:** Backend

**Etiquetas:** `backend` `api` `auto-save` `us-01a` `sprint-1`

**Comentarios:** La frecuencia de 30 segundos es responsabilidad del frontend (TICKET-06). Este endpoint solo debe ser eficiente y seguro. Discutir con TICKET-01 el mecanismo de autenticación del candidato anónimo antes de implementar.

**Dependencias:** TICKET-02 (schema), TICKET-01 (mecanismo de identificación del borrador)

---

### TICKET-05 · [Backend] Endpoint `GET /applications/draft?token=<token>` — Recuperación de borrador para retoma

**Descripción:**
Implementar el endpoint que permite al candidato recuperar un borrador guardado previamente al regresar al formulario. Devuelve los datos del borrador si el token es válido y el borrador no ha expirado (menos de 7 días desde `updated_at`).

Comportamiento requerido:
- Recibe un token de identificación del borrador (mecanismo definido en TICKET-01).
- Si el token es válido y el borrador tiene `stage = "draft"` y `updated_at > NOW() - 7 days`: retorna HTTP 200 con los campos guardados y la URL del CV si fue subido.
- Si el borrador expiró (`updated_at < NOW() - 7 days`): retorna HTTP 410 con `{"expired": true, "cv_url": "<url_si_existe>"}` — preservar `cv_url` aunque el resto haya expirado.
- Si el token no existe: retorna HTTP 404.
- El endpoint no modifica ningún dato; es solo lectura.

**Criterios de Aceptación:**
- Borrador válido (< 7 días): HTTP 200 con todos los campos guardados presentes en la respuesta en menos de 200ms.
- Borrador expirado (> 7 días): HTTP 410 con `cv_url` preservado si el candidato había subido un CV.
- Token inexistente: HTTP 404.
- La `cv_url` retornada es una URL firmada de R2/S3 con TTL de 1 hora, nunca una URL pública. Verificable inspeccionando el valor retornado.
- El endpoint no modifica `updated_at` ni ningún otro campo. Verificable con query directa a DB antes y después del GET.

**Prioridad:** Alta

**Estimación:** 3 puntos

**Asignado a:** Backend

**Etiquetas:** `backend` `api` `borrador` `us-01a` `sprint-1`

**Comentarios:** La lógica de expiración de 7 días se calcula sobre `updated_at`, no sobre `created_at`, para que los candidatos que estuvieron activos recientemente no pierdan su progreso.

**Dependencias:** TICKET-02 (schema), TICKET-01 (mecanismo de identificación del borrador)

---

### TICKET-06 · [Backend] Upload de CV a Cloudflare R2 y generación de URL firmada

**Descripción:**
Implementar el endpoint o el flujo de servidor que recibe el archivo CV del candidato, lo almacena en Cloudflare R2 y retorna una URL firmada con TTL de 1 hora. El `cv_url` almacenado en la tabla `candidates` debe ser la ruta interna del objeto en R2, no la URL firmada (que se genera on-demand).

Comportamiento requerido:
- Aceptar multipart/form-data con el archivo CV.
- Almacenar el archivo en R2 bajo una ruta con el formato `cvs/{organization_id}/{candidate_id}/{timestamp}-{filename}`.
- Actualizar `Candidate.cv_url` con la ruta interna.
- Retornar una URL firmada con TTL de 1 hora para que el frontend pueda confirmar la carga al candidato.
- El endpoint solo acepta archivos (formatos y tamaño máximo según lo definido en TICKET-01).

**Criterios de Aceptación:**
- El archivo subido es accesible mediante la URL firmada retornada durante 1 hora. Verificable con GET a la URL inmediatamente después del upload.
- La misma URL firmada retorna HTTP 403 o 404 pasadas 2 horas. Verificable en test con mock de tiempo o TTL corto en staging.
- `Candidate.cv_url` en DB contiene la ruta interna (no la URL firmada). Verificable con query directa.
- Un archivo de formato no permitido retorna HTTP 422 con mensaje descriptivo del error (formato y tamaño máximo aceptados).
- La ruta del objeto en R2 incluye `organization_id` como prefijo, garantizando aislamiento entre tenants. Verificable inspeccionando la ruta almacenada.

**Prioridad:** Alta

**Estimación:** 3 puntos

**Asignado a:** Backend

**Etiquetas:** `backend` `storage` `r2` `us-01a` `sprint-1`

**Comentarios:** Usar el SDK de S3 compatible con R2 ya definido en el stack. Coordinar con TICKET-01 los formatos y tamaño máximo aceptados. En staging usar un bucket de R2 separado del de producción.

**Dependencias:** TICKET-01 (formatos y tamaño de CV), TICKET-02 (campo `cv_url` en `candidates`)

---

### TICKET-07 · [Frontend] Componente de formulario adaptativo con layout mobile y desktop

**Descripción:**
Implementar el componente React del formulario de aplicación con comportamiento responsivo según el viewport. En mobile (< 768px) el formulario debe presentar máximo 8 campos obligatorios en pantalla completa sin scroll horizontal, con el botón de avance siempre visible sin hacer scroll.

El formulario debe:
- Renderizar los campos definidos en TICKET-01 en el orden especificado.
- Adaptar el layout automáticamente según el viewport sin recargar la página.
- Mostrar un indicador de progreso visual (porcentaje o pasos completados).
- El botón "Siguiente" o "Enviar" debe estar siempre visible en mobile (posición fija o sticky).
- El formulario no requiere login ni registro previo del candidato.

**Criterios de Aceptación:**
- En viewport < 768px: ningún campo genera scroll horizontal. Verificable con Playwright en viewport 375px × 812px (iPhone 14).
- El botón de avance es visible sin hacer scroll en viewport < 768px. Verificable con test E2E que comprueba que el botón está dentro del viewport sin scroll.
- El formulario carga en < 2 segundos en conexión "Slow 3G" simulada con Chrome DevTools. Verificable con Lighthouse en modo mobile.
- Score de Lighthouse Performance > 85 en mobile. Verificable en pipeline de CI.
- Los campos opcionales están claramente diferenciados de los obligatorios visualmente (label o badge).

**Prioridad:** Alta

**Estimación:** 5 puntos

**Asignado a:** Frontend

**Etiquetas:** `frontend` `react` `responsive` `us-01a` `sprint-1`

**Comentarios:** Usar Tailwind CSS para el responsive. El componente debe ser agnóstico del mecanismo de auto-save; ese comportamiento se agrega en TICKET-08. Dependencia bloqueante de TICKET-01 para la lista de campos.

**Dependencias:** TICKET-01 (campos del formulario y comportamiento tablet)

---

### TICKET-08 · [Frontend] Auto-save del formulario cada 30 segundos con indicador de estado

**Descripción:**
Implementar la lógica de guardado automático en el componente del formulario. Cada 30 segundos, si el formulario tiene cambios no guardados, se llama al endpoint `PATCH /applications/:id` (TICKET-04) y se actualiza el indicador de estado visible para el candidato.

Comportamiento requerido:
- Timer de 30 segundos que se reinicia en cada guardado exitoso.
- El guardado solo se dispara si hay cambios respecto al último estado guardado (dirty state).
- Mientras se guarda: mostrar "Guardando..." o un spinner discreto.
- Tras guardar exitosamente: mostrar "Guardado" con timestamp.
- Si el guardado falla (error de red): mostrar "No se pudo guardar. Reintentando..." y reintentar en 10 segundos.
- Al cerrar la pestaña o navegar fuera: ejecutar un guardado final sincrónico si hay cambios pendientes (`beforeunload`).

**Criterios de Aceptación:**
- A los 30 segundos de inactividad con cambios sin guardar, el endpoint `PATCH` es llamado. Verificable con test unitario que mockea el timer y verifica la llamada al endpoint.
- Si no hay cambios desde el último guardado, el `PATCH` no se llama. Verificable con spy en el endpoint.
- El indicador de estado cambia de "Guardando..." a "Guardado [hora]" tras respuesta HTTP 200 del endpoint.
- Si el endpoint retorna error de red, el formulario muestra el mensaje de reintento y reintenta en 10 segundos. Verificable con mock de error de red.
- Al cerrar el navegador con cambios pendientes, se ejecuta el guardado final. Verificable con test E2E que intercepta el evento `beforeunload`.

**Prioridad:** Alta

**Estimación:** 3 puntos

**Asignado a:** Frontend

**Etiquetas:** `frontend` `react` `auto-save` `ux` `us-01a` `sprint-1`

**Comentarios:** El timer debe ser un custom hook (`useAutoSave`) para facilitar el testing unitario. Coordinar con TICKET-04 el contrato del endpoint antes de implementar.

**Dependencias:** TICKET-04 (endpoint de auto-save), TICKET-07 (componente base del formulario)

---

### TICKET-09 · [Frontend] Banner de retoma de borrador y recuperación de progreso guardado

**Descripción:**
Implementar la lógica que detecta si existe un borrador guardado para el candidato al acceder al formulario de una vacante, y muestra el banner "Retomar aplicación guardada" con la opción de continuar o empezar desde cero.

Comportamiento requerido:
- Al cargar el formulario: llamar al endpoint `GET /applications/draft?token=<token>` (TICKET-05) con el token de identificación del candidato (mecanismo definido en TICKET-01).
- Si existe borrador válido (HTTP 200): mostrar el banner en la parte superior del formulario con los campos pre-cargados en menos de 2 segundos desde la carga de la página.
- Si el borrador expiró (HTTP 410): no mostrar el banner; si hay `cv_url` en la respuesta, pre-cargar el CV silenciosamente.
- Si no existe borrador (HTTP 404): cargar el formulario vacío normalmente.
- El botón "Empezar desde cero" descarta el borrador y limpia todos los campos excepto el CV si estaba guardado.

**Criterios de Aceptación:**
- Con borrador válido: el banner aparece y los campos están cargados en menos de 2 segundos desde la carga del formulario. Verificable con test E2E que mide el tiempo desde `DOMContentLoaded` hasta que el banner es visible.
- Con borrador expirado: no aparece el banner; si había CV, el campo de CV muestra el archivo previo. Verificable con mock de respuesta HTTP 410 con `cv_url`.
- Sin borrador: el formulario carga vacío sin banner. Verificable con mock de respuesta HTTP 404.
- "Empezar desde cero" limpia todos los campos. Verificable con test E2E que verifica que los inputs están vacíos tras el clic.
- El banner no bloquea la interacción con el formulario mientras carga.

**Prioridad:** Alta

**Estimación:** 3 puntos

**Asignado a:** Frontend

**Etiquetas:** `frontend` `react` `ux` `borrador` `us-01a` `sprint-1`

**Comentarios:** Este ticket tiene una dependencia dura de TICKET-01 (mecanismo de identificación del candidato anónimo) y de TICKET-05 (endpoint de recuperación). No puede completarse hasta que ambos estén definidos e implementados.

**Dependencias:** TICKET-05 (endpoint de recuperación), TICKET-07 (componente base), TICKET-01 (mecanismo de token)

---

### TICKET-10 · [Frontend] Validación inline de campos con feedback de error sin pérdida de datos

**Descripción:**
Implementar la validación del formulario con feedback inline sobre cada campo. La validación debe ejecutarse al intentar enviar el formulario (no en tiempo real mientras el candidato escribe). El feedback debe aparecer en menos de 500ms y no puede provocar pérdida de datos en ningún campo válido.

Comportamiento requerido según el BDD:
- Al hacer clic en "Enviar" con campos obligatorios vacíos: borde rojo y mensaje "Este campo es obligatorio" directamente debajo de cada campo incompleto.
- El scroll de la página debe posicionarse automáticamente en el primer campo con error.
- Los campos con valor válido no se limpian ni pierden su contenido.
- El mensaje de error desaparece cuando el campo recibe un valor válido.
- La validación ocurre en el cliente antes de llamar al backend; el backend valida nuevamente server-side (TICKET-03).

**Criterios de Aceptación:**
- Al hacer clic en "Enviar" con el campo Email vacío: aparece el borde rojo y el mensaje "Este campo es obligatorio" debajo del campo Email en menos de 500ms. Verificable con test de Playwright que mide el tiempo con `performance.now()`.
- El scroll se posiciona en el primer campo con error. Verificable con test E2E que verifica la posición del viewport.
- Un campo con valor válido no pierde su contenido cuando otro campo falla la validación. Verificable llenando 7 de 8 campos y verificando que los 7 mantienen su valor tras el intento de envío.
- El mensaje de error desaparece cuando el campo recibe un valor. Verificable con test E2E que escribe en el campo con error y verifica que el mensaje desaparece.
- No se llama al endpoint `POST /jobs/:jobId/applications` si la validación del cliente falla. Verificable con spy en el fetch.

**Prioridad:** Alta

**Estimación:** 3 puntos

**Asignado a:** Frontend

**Etiquetas:** `frontend` `react` `validacion` `ux` `us-01a` `sprint-1`

**Comentarios:** Usar una librería de validación de formularios compatible con React (ej. React Hook Form) si ya está en el stack, o implementar validación propia. El mensaje "Este campo es obligatorio" es literal según el BDD; no cambiar la redacción sin aprobación de PM.

**Dependencias:** TICKET-07 (componente base del formulario), TICKET-01 (lista de campos y cuáles son obligatorios)

---

### TICKET-11 · [QA] Tests de integración y E2E para los tres escenarios de US-01a

**Descripción:**
Diseñar e implementar la suite de tests que cubre los tres escenarios de aceptación de US-01a más los casos de error documentados en la historia.

**Tests de integración (backend):**
- `POST /jobs/:jobId/applications` sin `gdpr_consent` → HTTP 422 `GDPR_CONSENT_REQUIRED`
- `POST /jobs/:jobId/applications` con par `candidate_id + job_id` duplicado → HTTP 422 `DUPLICATE_APPLICATION`
- `POST /jobs/:jobId/applications` válido → HTTP 201 con `portal_token` de 64+ caracteres
- `PATCH /applications/:id` sobre borrador expirado → HTTP 409
- `GET /applications/draft?token=` con borrador de 8 días → HTTP 410 con `cv_url` preservado
- Intento de acceso con `organization_id` de otro tenant → HTTP 403 o HTTP 404

**Tests E2E (Playwright):**
- Escenario 1: formulario en viewport 375px carga en < 2s, sin scroll horizontal, botón visible sin scroll
- Escenario 2: candidato llena 70% del formulario, cierra el navegador, regresa dentro de 7 días → banner y campos cargados en < 2s
- Escenario 3: intento de envío con Email vacío → borde rojo + mensaje en < 500ms + scroll al campo + otros campos conservan valores

**Performance:**
- Lighthouse en mobile con conexión "Slow 3G": Performance > 85

**Criterios de Aceptación:**
- El 100% de los tests de integración listados pasa en CI.
- El 100% de los tests E2E listados pasa en CI con Playwright en Chromium y Firefox.
- El score de Lighthouse Performance > 85 pasa en CI como check obligatorio.
- Los tests de integración no requieren servicios externos reales (usar mocks de R2 y DB de testing).

**Prioridad:** Alta

**Estimación:** 5 puntos

**Asignado a:** QA

**Etiquetas:** `qa` `testing` `e2e` `integracion` `us-01a` `sprint-1`

**Comentarios:** Los tests E2E deben ejecutarse contra el entorno de staging, no contra producción. El test de Lighthouse debe correr en un paso separado del pipeline para no bloquear el merge por variaciones de red.

**Dependencias:** TICKET-03, TICKET-04, TICKET-05, TICKET-07, TICKET-08, TICKET-09, TICKET-10 (todos deben estar implementados antes de ejecutar la suite completa)

---

### Estimación de tickets — US-01a

**Metodología:** Planning Poker · Escala Fibonacci · Unidad: puntos de historia
**Referencia de calibración:**

| Puntos | Referencia para este equipo |
|---|---|
| 1 | Cambio de configuración o query simple ya definida |
| 2 | Análisis o refinamiento con output concreto y acotado |
| 3 | Endpoint o componente con lógica clara y un solo camino de error |
| 5 | Endpoint o componente con múltiples casos, integración externa o lógica de negocio compuesta |
| 8 | Trabajo que cruza múltiples capas o tiene incertidumbre técnica significativa |
| 13 | Spike necesario o dependencia externa no controlada por el equipo |

---

| ID | Tipo | Título resumido | Estimación (pts) | Justificación |
|---|---|---|---|---|
| TICKET-01 | Refinamiento | Definir campos, tablet y mecanismo de borrador | **2** | Sesión de refinamiento acotada con PM y Tech Lead. El output es un documento de decisiones, no código. Cuatro preguntas concretas con respuestas binarias o de lista corta. Sin incertidumbre técnica, solo de negocio. |
| TICKET-02 | Base de Datos | Schema `applications` y `candidates` con RLS | **5** | El schema está bien definido en el PRD (sección 9), pero la implementación cubre dos tablas, RLS con política de tenant, tres índices compuestos y dos constraints de unicidad. La combinación de capas (Drizzle ORM + PostgreSQL RLS + migración reversible) sube la estimación de 3 a 5. |
| TICKET-03 | Backend | `POST /jobs/:jobId/applications` con validación GDPR | **5** | Endpoint público con lógica de negocio compuesta: resolución de `organization_id` desde `job_id`, creación condicional de `Candidate` (upsert por email+org), validación GDPR, generación de `portal_token`, escritura en `AuditLog` con metadata de duración. Múltiples caminos de error documentados (422 × 2 casos, 404). No hay integración externa, pero la lógica transaccional justifica 5. |
| TICKET-04 | Backend | `PATCH /applications/:id` — auto-save de borrador | **3** | Endpoint de actualización parcial sobre un registro en estado conocido. Lógica simple: verificar `stage = "draft"`, aplicar patch parcial, actualizar `updated_at`. Un solo camino de error (409). Sin integraciones externas. El riesgo principal es el mecanismo de autenticación del candidato anónimo, que depende de TICKET-01. |
| TICKET-05 | Backend | `GET /applications/draft` — recuperación de borrador | **3** | Endpoint de solo lectura con tres caminos de respuesta bien definidos (200, 410, 404) y lógica de expiración por `updated_at`. La generación de URL firmada de R2 on-demand añade una integración, pero es la misma utilidad implementada en TICKET-06, que puede reutilizarse. Complejidad baja una vez resuelto TICKET-01. |
| TICKET-06 | Backend | Upload de CV a R2 y URL firmada | **5** | Integración con Cloudflare R2 vía SDK S3-compatible: multipart/form-data parsing, validación de formato y tamaño (pendiente de TICKET-01), escritura en R2 con ruta estructurada por tenant, actualización de `Candidate.cv_url`, generación de URL firmada con TTL. Es el primer punto de contacto con el servicio externo de storage; la configuración de credenciales y el manejo de errores de R2 añaden incertidumbre que justifica 5 sobre 3. |
| TICKET-07 | Frontend | Componente de formulario adaptativo mobile/desktop | **8** | El componente más complejo del frontend: layout responsivo para dos breakpoints (con posible tercero según TICKET-01), renderizado condicional de campos según definición de PM, estado del formulario compartido con TICKET-08 y TICKET-09, restricción de Lighthouse Performance > 85 en 3G. La combinación de responsividad estricta, performance y la dependencia de campos aún no definidos sube la estimación a 8. |
| TICKET-08 | Frontend | Auto-save cada 30s con indicador de estado | **3** | Custom hook `useAutoSave` con timer, dirty state tracking y retry logic. La lógica está bien especificada (30s, reintentar en 10s, `beforeunload`). El indicador de estado es UI sencilla. El riesgo principal es la coordinación con TICKET-07 para el estado compartido, pero el patrón es conocido. |
| TICKET-09 | Frontend | Banner de retoma y recuperación de borrador | **3** | Lógica de inicialización del formulario con tres ramas (200 / 410 / 404) y renderizado condicional del banner. La complejidad está en la coordinación con TICKET-07 (pre-carga de campos) y en el caso de CV preservado en borrador expirado. Sin integraciones externas propias; consume TICKET-05. |
| TICKET-10 | Frontend | Validación inline sin pérdida de datos | **3** | Validación en el evento submit (no en tiempo real), feedback por campo, scroll al primer error, preservación de valores. Patrón estándar con librerías de formularios React. El requisito de < 500ms y la preservación de valores en campos válidos son verificables pero no añaden complejidad de implementación sustancial. |
| TICKET-11 | QA | Suite de tests E2E e integración para US-01a | **8** | Cubre seis tests de integración de backend y tres escenarios E2E con Playwright (incluyendo medición de tiempos con `performance.now()`, viewport específico, mock de red y `beforeunload`). El test de Lighthouse en CI añade configuración de pipeline. La cobertura de casos de error de RLS y el mock de R2 en staging son los puntos de mayor esfuerzo. |
| | | **Total** | **54 pts** | |

---

## Experimento de Prompting: Generación del Product Backlog LTI-CVE

### Contexto del ejercicio

El objetivo fue generar un Product Backlog priorizado para el sistema **LTI-CVE** (Candidate Experience) utilizando distintas estrategias de prompting, iterando entre herramientas (Claude y ChatGPT) y comparando resultados para llegar al mejor output posible.

---

### Prompts utilizados

#### Prompt 1 — Análisis de metodología de priorización

> Como Product Manager Senior, analiza el contexto del producto descrito en el PRD y las User Stories generadas en el paso anterior, necesito que respondas:
>
> 1. ¿Qué metodología de priorización utilizarías para este producto y por qué?
>    * Evalúa al menos las siguientes alternativas:
>       * MoSCoW
>       * WSJF (Weighted Shortest Job First)
>       * Valor vs. Esfuerzo
>       * Priorización basada en los pains identificados en el PRD
>    * Recomienda una metodología principal para LTI y justifica la elección
> 2. Una vez aprobada la metodología propuesta, genera el Product Backlog priorizado. No realices cambios en el documento hasta que validemos lo propuesto.

**Herramienta:** Claude + ChatGPT (mismo prompt ejecutado en ambas)

---

#### Prompt 2 — Síntesis y triangulación entre IAs

> La conclusión de ChatGPT a la misma consulta fue: Para LTI utilizaría una estrategia híbrida:
>
> 1. Metodología principal: Priorización basada en los pains identificados en el PRD.
> 2. Metodología de apoyo para el MVP: MoSCoW.
> 3. Metodología de apoyo para roadmap post-MVP: WSJF. Esta combinación mantiene al equipo enfocado en validar la propuesta de valor diferencial de LTI (Candidate Experience + HM Adoption + Automatización) antes de optimizar el backlog con métricas financieras más sofisticadas.
>
> Tu tarea, actuando como Product Manager y Business Analyst Senior, es analizar ambas propuestas de forma exhaustiva y objetiva. Debes recomendar la mejor alternativa o una versión consolidada que combine los aspectos más sólidos de cada una.

**Herramienta:** Claude

---

#### Prompt 3 — Generación del Product Backlog completo (prompt principal)

> Genera el Product Backlog completo y priorizado para LTI-CVE siguiendo estrictamente las instrucciones de esta sección.
>
> **Paso 1 — Partición de historias L**
> Antes de puntuar, identifica todas las User Stories marcadas como tamaño L en el Documento 2. Para cada una:
> - Decide si puede subdividirse en historias más pequeñas manteniendo valor entregable de forma independiente (criterio INVEST: cada sub-historia debe ser valiosa por sí sola, no solo un trozo técnico).
> - Si puede subdividirse: genera las sub-historias con el mismo formato que las originales (título, historia Como/Quiero/Para, 3 criterios BDD, estimación, evaluación INVEST breve).
> - Si no puede subdividirse: mantenla como L y justifica por qué en una línea.
>
> **Paso 2 — Puntuación WSJF**
> Para cada historia del backlog (incluyendo las sub-historias del Paso 1), genera una tabla con los siguientes campos:
>
> | ID | Título corto | Pain(s) atacado(s) | Valor (Pain Score) | Urgencia | Tamaño | WSJF Fase 2 | MoSCoW |
>
> WSJF Fase 2 = (Valor + Urgencia) / Tamaño. Redondea a dos decimales.
> MoSCoW: clasifica cada historia en M / S / C / W considerando que el MVP debe ser demostrable a design partners en las primeras 6 semanas con un equipo de 4–5 personas. Sé estricto: si dudas entre M y S, elige S.
>
> **Paso 3 — Backlog priorizado por fase**
> Genera tres secciones ordenadas por WSJF descendente:
>
> *Sección A — MVP Must Have (Fase 1, semanas 1–4)*
> Solo historias clasificadas como M en MoSCoW. Para cada historia incluye:
> - ID y título
> - Épica a la que pertenece
> - Historia completa (Como / Quiero / Para)
> - Los 3 criterios de aceptación BDD
> - Puntuación WSJF con desglose de componentes
> - Dependencias de otras historias del backlog (IDs)
> - Definición de Done específica para esta historia (3–5 puntos concretos, no genéricos; deben ser verificables por el QA sin ambigüedad)
> - Sprint sugerido dentro de las primeras 4 semanas (Sprint 1 o Sprint 2)
>
> *Sección B — Backlog de validación (Fase 2, semanas 5–12)*
> Historias S y C ordenadas por WSJF. Para cada historia incluye:
> - ID y título
> - Épica
> - Historia completa
> - Puntuación WSJF con desglose
> - Dependencias
> - Sprint sugerido (Sprint 3 al Sprint 6)
>
> *Sección C — Backlog de escala (Fase 3, mes 4+)*
> Historias W y cualquier historia que requiera datos de clientes para validar su valor. Para cada historia incluye:
> - ID y título
> - Épica
> - Justificación de por qué pertenece a esta fase (1–2 líneas)
> - WSJF estimado Fase 3 (incluyendo Habilitación Técnica)
>
> **Paso 4 — Mapa de dependencias**
> Genera un listado estructurado de dependencias entre historias en formato: "[ID historia dependiente] no puede iniciarse sin [ID historia habilitadora] porque [razón técnica o de negocio en una línea]."
> Agrupa las dependencias por cadena crítica: identifica qué historia, si se retrasa, tiene el mayor efecto en cascada sobre el resto del backlog.
>
> **Paso 5 — Resumen ejecutivo del backlog**
> Genera una tabla resumen con:
> - Total de historias por fase y por épica
> - Capacidad estimada necesaria en sprints de 2 semanas (asumiendo un equipo de 4 personas con velocidad inicial de 20 puntos de historia por sprint, donde S = 2 pts, M = 3 pts, L = 5 pts)
> - Las 3 historias con mayor WSJF del backlog completo (los "no negociables")
> - Las 3 historias con mayor riesgo de retraso según el mapa de dependencias
> - Métricas de éxito del MVP al finalizar la Fase 1 (derivadas de los OKRs implícitos en el PRD: NPS candidato > 50, adopción HM > 75%, time-to-hire -30%)

**Herramienta:** Claude

---

### ¿Cuál prompt dio mejores resultados y por qué?

El Prompt 3 produjo el mejor resultado porque aplica metaprompting de forma deliberada: descompone la tarea en pasos secuenciales, prescribe el formato de salida y ancla las decisiones a restricciones operativas concretas (tamaño de equipo, duración del MVP, velocidad de sprint). Eso elimina la ambigüedad que normalmente genera variabilidad en el output. Los Prompts 1 y 2 no son pasos previos menores — definen la metodología de priorización que el Prompt 3 ya asume validada. Sin esa base, el modelo hubiera tenido que tomar decisiones metodológicas por su cuenta, y el backlog resultante sería menos confiable.

---
