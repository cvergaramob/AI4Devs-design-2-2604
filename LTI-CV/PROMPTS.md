# Historial de Prompts — Proyecto LTI-CVE
**IA:** Claude · **Modelo:** Claude Sonnet 4.6 (claude-sonnet-4-6)
**Fecha:** Mayo 2026

---

**Prompt 1**
Actúa como un Product Manager y Business Analyst con experiencia en productos SaaS y metodologías ágiles.

A partir de la descripción de producto adjunta en el archivo LTI-CVE.md genera User Stories que cumplan los criterios INVEST.

Para cada una incluye:
- Título descriptivo
- Historia en formato "Como [rol], quiero [acción], para [beneficio]"
- 3 criterios de aceptación en formato BDD (Dado que/Cuando/Entonces)
- Estimación de complejidad (S/M/L)
- Evaluación breve contra INVEST

Al finalizar identifica posibles épicas.

Salida esperada: archivo Markdown claro y legible.

---

**Prompt 2**
Como Product Manager Senior, analiza el contexto del producto descrito en el PRD y las User Stories generadas en el paso anterior, necesito que respondas:
1. ¿Qué metodología de priorización utilizarías para este producto y por qué?
   * Evalúa al menos las siguientes alternativas:
     * MoSCoW
     * WSJF (Weighted Shortest Job First)
     * Valor vs. Esfuerzo
     * Priorización basada en los pains identificados en el PRD
   * Recomienda una metodología principal para LTI y justifica la elección
2. Una vez aprobada la metodología propuesta, genera el Product Backlog priorizado.
No realices cambios en el documento hasta que validemos lo propuesto.

---

**Prompt 3**
La conclusión de ChatGPT a la misma consulta fue:
Para LTI utilizaría una estrategia híbrida:

1. Metodología principal: Priorización basada en los pains identificados en el PRD.
2. Metodología de apoyo para el MVP: MoSCoW.
3. Metodología de apoyo para roadmap post-MVP: WSJF.
Esta combinación mantiene al equipo enfocado en validar la propuesta de valor diferencial de LTI (Candidate Experience + HM Adoption + Automatización) antes de optimizar el backlog con métricas financieras más sofisticadas.

Tu tarea, actuando como Product Manager y Business Analyst Senior, es analizar ambas propuestas de forma exhaustiva y objetiva. Debes recomendar la mejor alternativa o una versión consolidada que combine los aspectos más sólidos de cada una.

---

**Prompt 4**
Actúa como un Prompt Engineer Senior especializado en Product Management y Agile.
Tomando como contexto los documentos utilizados y nuestra conversación previa tu tarea es diseñar el prompt que utilizarías para solicitar a una IA la creación de un Product Backlog completo y priorizado.

---

**Prompt 5**
Actúa como un Product Manager Senior y Business Analyst.
A partir de toda la documentación generada previamente (PRD, User Stories, análisis de priorización y backlog), genera un documento final consolidado, profesional y listo para entregar.
El documento debe:
* Unificar toda la información relevante en una única versión definitiva.
* Eliminar redundancias, repeticiones e inconsistencias.
* Mantener la trazabilidad entre las historias de usuario y su priorización.
* Utilizar una redacción clara, formal y consistente.
* Estar estructurado en formato Markdown.
Incluye únicamente las siguientes secciones:
# Historias de Usuario
Presenta todas las historias de usuario organizadas de forma clara y consistente, respetando el formato definido previamente.
# Metodología de Priorización
Explica la metodología de priorización utilizada, incluyendo los criterios aplicados y la justificación de su elección para este proyecto.
# Product Backlog Priorizado
Presenta el backlog completo ordenado según la priorización definida previamente.
Para cada historia incluye:
* ID
* Título
* Historia de usuario
* Resultado de verificación de aprobación de criterio INVEST
* Acceptance Criteria: con el formato Given/When/Then
* Prioridad
* Puntaje o clasificación de priorización correspondiente
* Dependencias relevantes (si existen)
El resultado debe ser un documento final coherente, consistente y directamente utilizable como entrega del proyecto.

---

**Prompt 6**
Evaluá el documento generado contra lo solicitado:
tu misión es preparar la documentación necesaria para empezar a implementar LTI:

1. Generar las User Stories. Puedes implementar tantas como quieras y puedas, el mínimo son 2. Utiliza lo aprendido sobre buenas prácticas de este capítulo para que contenga toda la información necesaria, y como consejo, usa una plantilla común para todas ellas (recuerda que dejamos un ejemplo de plantilla en la [sección de User Stories](https://training.lidr.co/posts/ai4devs-202604-%F0%9F%93%84-historias-de-usuarios-%F0%9F%94%B4-29-min)).
2. Arma el Backlog de producto con las User Stories, priorizándolas como consideres conveniente acorde a alguna metodología concreta. experimenta con diferentes formas de generar un prompt que te pueda genera tu back log basado en la documentación que has generado previamente. Entrega los diferentes prompts que usaste e indica cual prompt te dio mejores resultados. Entrega junto a los prompts tus conclusiones, por qué crees este prompt fue efectivo.

enumerá lo que no corresponde al alcance del documento. No modifiques nada hasta que tengas mi aprobación.

---

**Prompt 7**
sí, a todas

---

**Prompt 8**
Actúa como Product Manager Senior y Technical Lead.
Analiza la historia de usuario US-01a y descompónla en tickets técnicos listos para Sprint Planning.
Genera tickets con el nivel de detalle que utilizaría un equipo de desarrollo durante una reunión de planificación. Cada ticket debe representar una unidad de trabajo concreta, implementable y estimable.
Reglas

* Utiliza únicamente la información explícita de la historia de usuario, criterios de aceptación y documentación proporcionada.
* No inventes requerimientos, reglas de negocio, pantallas, endpoints, tablas, validaciones o integraciones no documentadas.
* Si falta información para definir un trabajo técnico, crea un ticket de tipo Refinamiento indicando qué debe aclararse.
* Evita tareas genéricas como "Desarrollar funcionalidad", "Implementar backend" o "Crear pantalla".
* Descompón la historia en tareas reales de análisis, backend, frontend, base de datos, integraciones, QA, seguridad o documentación según corresponda.
* Los tickets deben poder asignarse individualmente a un miembro del equipo.
* Mantén la trazabilidad con la historia de usuario original.
Formato requerido para cada ticket

* Título
* Descripción
* Criterios de Aceptación
* Prioridad
* Estimación
* Asignado a
* Etiquetas
* Comentarios
* Enlaces o dependencias relevantes
Ejemplo de un Ticket de trabajo bien formulado
Título: Implementación de Autenticación de Dos Factores (2FA)
Descripción: Añadir autenticación de dos factores para mejorar la seguridad del login de usuarios. Debe soportar aplicaciones de autenticación como Authenticator y mensajes SMS.
Criterios de Aceptación:

* Los usuarios pueden seleccionar 2FA desde su perfil.
* Soporte para Google Authenticator y SMS.
* Los usuarios deben confirmar el dispositivo 2FA durante la configuración.
Prioridad: Alta
Estimación: 8 puntos de historia
Asignado a: Equipo de Backend
Etiquetas: Seguridad, Backend, Sprint 10
Comentarios: Verificar la compatibilidad con la base de usuarios internacionales para el envío de SMS.
Enlaces: Documento de Especificación de Requerimientos de Seguridad
Antes de generar los tickets, resume brevemente:

1. Qué entiende de la historia.
2. Qué información es explícita.
3. Qué información falta o requiere refinamiento.
Genera únicamente tickets respaldados por la documentación disponible.

Salida esperada: Agrega los tickets generados al documento md

---

**Prompt 9**
Actúa como Product Manager Senior y Scrum Master.
Analiza los tickets de trabajo generados previamente y estima el esfuerzo de cada uno.
Selecciona la metodología de estimación que consideres más adecuada (Fibonacci, Planning Poker o Tallas de Camiseta) y utiliza la unidad de medida que mejor represente el esfuerzo estimado (horas o puntos de historia).
Para cada ticket indica:

* ID
* Título
* Estimación
* Justificación breve de la estimación

Salida esperada: incorporar la estimación solicitada al documento, presenta el resultado en formato tabular.

---

**Prompt 10**
generá un archivo PROMPTS.md con todo el historial de prompts utilizados en esta conversación, incluyendo este. No incluyas las respuestas. Indica la IA y el modelo utilizados.


