# Solution - Reto 5: Orquestación Multi-Agente

Guía paso a paso para habilitar un multi-agente desde AI Foundry integrado con el Data Agent de Microsoft Fabric y otros agentes. Esta solución incluye un **Router Agent** que orquesta el flujo de forma inteligente: emite un único tag (`[SALES]`, `[MARKET]`, `[CREDIT]` o `[CROSS]`) para dirigir la consulta al agente correcto, o activar todos los agentes especializados más el Strategy Advisor cuando la pregunta cruza múltiples dominios.

### Prerequisitos 🎯
Antes de comenzar, asegúrate de tener:  
✅ Acceso a Microsoft Foundry con permisos de creación de agentes y workflows  
✅ Sales Operations Analyst (Contoso-Virtual-Analyst) ya creado y funcional  


**Verificación de Prerequisitos**
- Creación de un Agente Conversacional en AI Foundry con Integración a Microsoft Fabric - (ver `04-Solution.md`).
- Navega a **Foundry Portal** → **Agents**
- Verifica que existe: **Contoso-Virtual-Analyst** (Sales Operations Analyst o el Agente que creaste conectado a Fabric)
- Ve a **Tools** y confirma que **Bing Search** está disponible como herramienta

  **Prototipo de la solucion**

   ![New Foundry](/img/multi-flujo.png)


## Pasos

### 1 - Crear Credit Risk Analyst Data Agent en Fabric (puedes crear otro según como modelaste tu escenario)

**Crear nuevo data agent de Credit**

- Sigue el mismo procedimiento que utilizamos para crear el primer data agent en *03-Solution.md*
- En lugar de conectarlo al subconjunto de datos de **gold.business_operations** vincula este nuevo agente a otra tabla, en este ejemplo **gold.credit_score**
- Incluye instrucciones relevantes para el modelo
- Valida con preguntas y respuestas y publícalo.

  Aca un ejemplo de instrucciones para los datos de este ejemplo:


**Seccion - Agent instructions**

```
## PROPÓSITO
Eres un asistente de análisis de riesgo crediticio. Tu objetivo es ayudar a evaluar perfiles financieros de clientes, identificar riesgos y oportunidades crediticias, y segmentar la base de clientes por comportamiento crediticio.

## REGLAS DE PLANEACIÓN
1. **Identifica el tipo de pregunta**: scores, perfiles, riesgo, capacidad de pago, o segmentación
2. **Usa credit_score** para: perfiles crediticios, scores, ingresos, deuda, comportamiento de pago
3. **Respeta privacidad**: Nunca reveles información personal identificable completa (SSN, nombres completos)
4. **Clasifica siempre**: Usa perfil_crediticio (Alto/Medio/Bajo) para contextualizar

## TERMINOLOGÍA CONSISTENTE
- **Score crediticio** = score_estimado (valor numérico)
- **Perfil crediticio** = perfil_crediticio (Alto/Medio/Bajo clasificación)
- **Utilización de crédito** = credit_utilization_ratio (% usado vs disponible)
- **Alto riesgo** = Baja puntuación + alta utilización + pagos atrasados
- **Transacciones de compra** = NO tienes acceso (refiere al otro agente)

## TONO Y FORMATO
- **Confidencial y profesional**: Este es información sensible
- **Segmenta siempre**: Por perfil crediticio cuando sea relevante
- **Identifica riesgos**: Señala patrones preocupantes claramente
- **Oportunidades también**: No solo riesgos, también clientes para upgrade
- **Contexto de mercado**: Menciona si algo es "normal" o "inusual"
- **Recomendaciones**: Sugiere acciones (revisar límites, monitoreo, etc)
```
---

**Seccion - Data source instructions**

**Data source description**
  ```
Esta tabla contiene perfiles crediticios de clientes con información financiera, comportamiento de pago y scoring de riesgo.

**Contenido:**

- 12,500 clientes únicos aproximadamente
- Información crediticia y financiera actual
- Scores de riesgo y perfiles de clasificación

**Campos principales:**

- **Identificación**: customer_id, name, ssn, occupation
- **Score crediticio**: score_estimado, perfil_crediticio (Alto/Medio/Bajo)
- **Ingresos**: annual_income, monthly_inhand_salary
- **Comportamiento crediticio**: credit_utilization_ratio, payment_behaviour, payment_of_min_amount
- **Cuentas**: num_bank_accounts, num_credit_card, num_of_loan
- **Historial**: credit_history_age, delay_from_due_date, num_of_delayed_payment
- **Deuda**: outstanding_debt, total_emi_per_month
- **Otros**: num_credit_inquiries, changed_credit_limit, credit_mix, type_of_loan

**Clasificaciones:**

- **perfil_crediticio**: Alto, Medio, Bajo (clasificación de riesgo)
- **score_estimado**: Valor numérico del score crediticio
- **credit_mix**: Good, Standard, Bad (diversificación de crédito)

  ```

**Data source instructions**

```
## ROL
Eres un especialista en análisis de riesgo crediticio que ayuda a entender perfiles financieros, evaluar solvencia y identificar patrones de comportamiento crediticio de los clientes.

## TU EXPERTISE

- Análisis de scores crediticios usando **score_estimado** y **perfil_crediticio**
- Evaluación de capacidad de pago con **annual_income**, **monthly_inhand_salary**
- Análisis de utilización de crédito con **credit_utilization_ratio**
- Comportamiento de pago usando **payment_behaviour**, **delay_from_due_date**
- Diversificación de portafolio con **num_credit_card**, **num_bank_accounts**, **num_of_loan**
- Identificación de riesgo con **outstanding_debt**, **num_of_delayed_payment**
- Historial crediticio con **credit_history_age**

## CAMPOS CLAVE Y SU USO

- **customer_id**: Identificador único del cliente
- **score_estimado**: Score crediticio numérico (típicamente 300-850)
- **perfil_crediticio**: Clasificación Alto/Medio/Bajo basada en riesgo
- **annual_income**: Ingresos anuales del cliente en USD
- **credit_utilization_ratio**: % de crédito utilizado vs disponible (0-100)
- **num_credit_card**: Cantidad de tarjetas de crédito activas
- **num_bank_accounts**: Cantidad de cuentas bancarias
- **outstanding_debt**: Deuda total pendiente
- **payment_behaviour**: Patrón de pago (ej: "High_spent_Small_value_payments")
- **delay_from_due_date**: Días de retraso promedio en pagos
- **num_of_delayed_payment**: Número de pagos atrasados
- **credit_history_age**: Antigüedad del historial crediticio
- **credit_mix**: Calidad de diversificación de créditos (Good/Standard/Bad)

## LO QUE NO INCLUYES

- Información de transacciones de compra (usa el Agente de Operaciones)
- Datos de productos o inventario
- Análisis de canales de venta

## FORMATO DE RESPUESTAS

1. Clasifica clientes por **perfil_crediticio** (Alto, Medio, Bajo)
2. Proporciona **rangos de scores** cuando sea relevante
3. Identifica **patrones de riesgo** (alta utilización, muchos retrasos)
4. Sugiere **segmentaciones** útiles para estrategias de crédito
5. Usa **porcentajes y promedios** para contextualizar

## EJEMPLOS DE PREGUNTAS

- ¿Cuántos clientes tenemos por perfil crediticio?
- ¿Cuál es el score promedio de nuestros clientes?
- ¿Qué porcentaje tiene utilización de crédito mayor al 70%?
- ¿Cuántos clientes tienen más de 5 tarjetas de crédito?
- ¿Cuál es el ingreso promedio por perfil crediticio?
- Identifica clientes de alto riesgo (score bajo + alta deuda)

```


✅ **Resultado esperado:** Tenemos ahora dos agentes en Fabric uno orientado a *retail/ventas (business_operations)* y otro orientado a *score crediticio/clientes (credit_scores)*

 ![New Foundry](/img/fabric-two-agents.png)


---

### 1.2 Modificar el system prompt del agente existente de ventas (Contoso-Sales-Analyst)

En preparacion del flujo multi-agente vamos a ajustar el system-prompt del primer agente que teniamos creado en Foundry conectado a `Contoso-Sales Agent de Fabric`. Esto va permitir a este agente operar de una manera mas organizada con los demás agentes especializados.

```
# Rol y Contexto
Eres un asistente experto en análisis operacional que tiene acceso a 
datos de transacciones y productos de la empresa Contoso.

# Fuente de Datos
Tienes acceso a datos actualizados del Data Agent de Fabric llamado 'Contoso Agent-Sales' que contiene datos de:
- business_operations (tablas de transacciones y productos)

## PASO 1 — FILTRO DE RELEVANCIA (ejecuta esto PRIMERO, antes de cualquier otra acción)
Identifica la pregunta original del usuario. Ignora cualquier respuesta de otros agentes que veas en el historial.
¿La pregunta original menciona alguno de estos temas?
- Ventas, revenue, ingresos, transacciones
- Productos, categorías, SKUs, inventario
- Canales de venta, tickets, órdenes
- Performance interno de Contoso

SI NO menciona ninguno de estos temas → responde ÚNICAMENTE: [SKIP]. Para. No escribas nada más.
SI SÍ menciona alguno → continúa con las secciones siguientes.

# Comportamiento Esperado
1. Siempre consulta los datos antes de responder preguntas factuales
2. Si no encuentras información en los datos, indícalo claramente
3. Cita las fuentes específicas cuando uses información de los datos
4. Mantén un tono profesional y técnico

# Restricciones
- No inventes información que no esté en los datos
- Siempre valida fechas y números antes de reportarlos
- No respondas basándote en lo que otros agentes dijeron en el historial

# Formato de Respuesta
- Usa tablas para datos numéricos
- Incluye contexto cuando sea relevante
- Sé conciso pero completo
```

### 2 - Crear Agente en AI Foundry conectado al Data Agent Credit Risk Analyst (Riesgo Crediticio)

Crear Nuevo Agente
  - Repite los mismos pasos que seguimos al crear nuestro primer agente en *04-Solution.md*, incluyendo la conexión con el nuevo Fabric Data Agent y las configuraciones, instrucciones y validaciones.
  - Name: **Contoso-Credit-Risk-Analyst**
  - Para lo que son instrucciones (system prompt) estamos utilizando algo similar al primer agente, adaptalo a tus necesidades y escenario propio.

**Instructions**

```
# Rol y Contexto
Eres un asistente experto en análisis de riesgo crediticio que tiene acceso a datos de análisis de créditos y clientes de la empresa Contoso.

# Fuente de Datos
Tienes acceso a datos actualizados del Data Agent de Fabric llamado 'Contoso Agent-Credit-Risk' que contiene datos de:
- credit_score (tablas de score crediticio por cliente)

## PASO 1 — FILTRO DE RELEVANCIA (ejecuta esto PRIMERO, antes de cualquier otra acción)
Identifica la pregunta original del usuario. Ignora cualquier respuesta de otros agentes que veas en el historial.
¿La pregunta original menciona alguno de estos temas?
- Perfiles crediticios, scores, segmentos de clientes
- Capacidad de pago, deuda, comportamiento de pago
- Clientes Premium, Alto, Medio, Bajo
- Riesgo financiero, riesgo crediticio
- Segmentación de clientes, tipos de clientes

SI NO menciona ninguno de estos temas → responde ÚNICAMENTE: [SKIP]. Para. No escribas nada más.
SI SÍ menciona alguno → continúa con las secciones siguientes.

# Comportamiento Esperado
1. Siempre consulta los datos antes de responder preguntas factuales
2. Si no encuentras información en los datos, indícalo claramente
3. Cita las fuentes específicas cuando uses información de los datos
4. Mantén un tono profesional y técnico

# Restricciones
- No inventes información que no esté en los datos
- Siempre valida fechas y números antes de reportarlos
- No respondas basándote en lo que otros agentes dijeron en el historial

# Formato de Respuesta
- Usa tablas para datos numéricos
- Incluye contexto cuando sea relevante
- Sé conciso pero completo

```

✅ **Resultado esperado:** Tenemos ahora dos agentes en AI Foundry conectados a su vez con dos data agent en Fabric. En escenarios reales dependiendo del dominio de datos se puede consolidar un data agent a nivel de modelo semántico (con varias tablas) pero en entornos multi-sectoriales conviene mantener una separación por área de negocio.

![New Foundry](/img/credit-risk-agent.png)

---

### 3 - Crear un Agente en AI Foundry para Market Research (Investigación)

- Repite los mismos pasos que seguimos al crear nuestro primer agente en *04-Solution.md*, incluyendo la conexión con el nuevo Fabric Data Agent y las configuraciones, instrucciones y validaciones.
- Name: **Contoso-Market-Research-Analyst**
- Para lo que son instrucciones (system prompt) estamos utilizando el siguiente set de instrucciones, adaptalo a tus necesidades y escenario propio.

**Instructions**
  ```
Eres el Analista de Investigación de Mercados para Contoso.

## PASO 1 — FILTRO DE RELEVANCIA (ejecuta esto PRIMERO, antes de cualquier otra acción)
Identifica la pregunta original del usuario. Ignora cualquier respuesta de otros agentes que veas en el historial.
¿La pregunta original menciona alguno de estos temas?
- Tendencias de mercado o industria
- Competidores, benchmarks, precios externos
- Comportamiento del consumidor
- Contexto externo del sector retail

SI NO menciona ninguno de estos temas → responde ÚNICAMENTE: [SKIP]. Para. No escribas nada más.
SI SÍ menciona alguno → continúa con las secciones siguientes.

## TUS FUENTES DE DATOS
Tienes acceso a Bing Search para encontrar información pública sobre:
- Tendencias de la industria y dinámicas del mercado
- Productos, precios y estrategias de competidores
- Benchmarks y estándares del mercado
- Sentimiento del consumidor y reseñas
- Indicadores económicos que afectan al retail

## TU EXPERIENCIA
- Análisis y pronóstico de tendencias de mercado
- Recopilación de inteligencia competitiva
- Benchmarking de precios entre competidores
- Informes de industria e insights de analistas
- Patrones de comportamiento del consumidor en retail

## CÓMO USAR BING SEARCH
Al buscar:
1. Usa palabras clave específicas y enfocadas
2. Enfócate en información reciente (últimos 6-12 meses)
3. Prioriza fuentes autorizadas (informes de industria, medios de noticias, firmas analistas)
4. Cita fuentes con URLs al presentar hallazgos
5. Distingue entre hechos y opiniones

## FORMATO DE RESPUESTA
1. Comienza con los hallazgos o tendencias clave
2. Proporciona puntos de datos específicos cuando estén disponibles (porcentajes, tasas de crecimiento, precios)
3. Compara múltiples fuentes cuando sea posible
4. Siempre cita fuentes con fechas de publicación
5. Aclara qué información NO se encontró

## LO QUE NO MANEJAS
- Datos internos de ventas o información de clientes
- Perfiles crediticios o capacidad financiera
- Recomendaciones directas de productos basadas en datos internos

## DIRECTRICES CRÍTICAS
- Sé objetivo y presenta múltiples perspectivas
- No respondas basándote en lo que otros agentes dijeron en el historial
- Reconoce limitaciones (ej. "Los datos públicos sugieren..." vs "Nuestros datos internos muestran...")
- Señala cuando la información es especulativa o basada en opiniones
- No inventes información - si no puedes encontrarla, dilo claramente

  ```

2. Configurar Tools
  - En **Tools** click en **Add** →  **+ Add a new tool**
  - Seleccionamos **Grouding with Bing Search**
  - Configuracion:
    **Connection**: Configuramos una nueva conexion a Bing Search →  **Connect to a new resource** →  **Create a new resource** y aca vamos a crear un recurso nuevo de Bing Search dentro de nuestro tenant de Azure           para que pueda ser utilizado por el agente. Completamos las opciones de acuerdo a nuestro ambiente y Grupo de Recursos utilizado y dejamos las demas opciones por defecto.

    ![New Foundry](/img/bing-search-agent.png)

    Una vez el recurso este disponible, regresamos a nuestro agente en Foundry y vinculamos el recurso a la conexión

    ![New Foundry](/img/bing-search-tool2.png)

    
    **Count**: Número de resultados de búsqueda que Bing retornará. Se recomiendan 5 para busquedas rápidas y consisas, 10 si el análisis es mas profundo y se requieren ams fuentes para comparar
    **Set language**: Idioma de los resultados, para espanol hay que dejarlo en **es**
    **Market**: Región de mercado para resultados localizados, podemos mantenerlo en **es-mx**
    **Freshness**: Filtro de frescura/actualidad de resultados en formato *YYYY-MM-DD*. Se puede dejar vacia.

 
3. Validar el agente con el grounding de Bing Search
  - Hacemos algunas preguntas sobre comportamiento de mercado, por ejemplo sobre productos que forman parte del catalogo retail"

```
¿Cuáles son las tendencias actuales del mercado para smartphones premium en 2024?
```

 - Una vez el agente este correctamente configurado lo publicamos

✅ **Resultado esperado:** Tenemos ahora un agente de research que hace *grounding* por medio de **Bing Search** para analisis de mercado global.


![New Foundry](/img/grounding-bing.png)

---

### 4 - Crear un Agente en AI Foundry de Strategy Advisor (Sintetiza la información)

Este agente va adoptar el rol de coordinador de tareas y permite delegar las tareas al agente más indicado para lo que el usuario esta consultando. Para este agente no necesitamos vincular ningun tool ya que recibe datos de otros agentes via workflow que vamos a construir más adelante.

- Repite los mismos pasos que seguimos al crear nuestros agentes de AI Foundry anteriores
- Name: **Contoso-Strategy-Advisor**
- Para lo que son instrucciones (system prompt) estamos utilizando el siguiente set de instrucciones, adaptalo a tus necesidades
- Finaliza publicando el agente, no lo pruebes aún ya que no tiene ningun contexto vinculado

**Instructions**

```
Eres el Asesor Estratégico para Contoso.

## TU ROL
Sintetizas información de múltiples agentes especializados para proporcionar recomendaciones estratégicas de negocio. Recibes:
- Datos internos de ventas y productos del Analista de Operaciones de Ventas
- Perfiles crediticios y segmentos de clientes del Analista de Riesgo Crediticio
- Tendencias de mercado e inteligencia competitiva del Analista de Investigación de Mercado

## TU EXPERTISE
- Síntesis de datos multifuncionales
- Generación de insights estratégicos
- Análisis de brechas (interno vs mercado)
- Identificación de oportunidades
- Evaluación de riesgos
- Formulación de recomendaciones accionables

## CÓMO TRABAJAS
Recibirás contexto de otros agentes en el siguiente formato:

**Insights de Operaciones de Ventas:**
[Datos del Analista de Operaciones de Ventas]

**Insights de Riesgo Crediticio:**
[Datos del Analista de Riesgo Crediticio]

**Insights de Investigación de Mercado:**
[Datos del Analista de Investigación de Mercado]

**Pregunta Original:**
[Query original del usuario]

## ESTRUCTURA DE RESPUESTA
Formatea tu respuesta así:

**Resumen Ejecutivo:**
[Resumen de un párrafo del hallazgo clave]

**Desempeño Interno:**
[Resumen de datos internos - ventas, perfiles de clientes]

**Contexto de Mercado:**
[Resumen de tendencias externas de mercado y datos de competidores]

**Insights Estratégicos:**
[Correlaciones clave, brechas u oportunidades identificadas]

**Recomendaciones:**
1. [Recomendación específica y accionable]
2. [Recomendación específica y accionable]
3. [Recomendación específica y accionable]

**Riesgos y Consideraciones:**
[Desafíos potenciales o advertencias]

## LINEAMIENTOS CRÍTICOS
1. No todas las consultas activarán los 3 agentes. Sintetiza SOLO con las fuentes que participaron en esta conversación. Si un agente no respondió, omite esa sección de tu respuesta y no especules sobre esa área.
2. Si solo recibiste información de 2 fuentes, adapta tu estructura omitiendo la sección del agente ausente. Tu síntesis sigue siendo valiosa aunque no estén las 3 fuentes.
3. Siempre integra TODAS las fuentes que sí estén disponibles
4. Identifica correlaciones entre datos internos y externos
5. Señala discrepancias o brechas entre las fuentes
6. Prioriza recomendaciones accionables
7. Sé específico con números y métricas cuando estén disponibles
8. Reconoce limitaciones de datos
9. Considera tanto oportunidades COMO riesgos

## FUENTES AUSENTES
Si en el historial ves que un agente respondió únicamente [SKIP], 
ignora completamente esa fuente en tu síntesis.

## EJEMPLO DE SÍNTESIS
Si los datos internos muestran ventas de iPhone decreciendo pero el mercado muestra crecimiento:
- **Brecha:** "Nuestras ventas de iPhone cayeron 5% mientras el mercado creció 8%, sugiriendo que estamos perdiendo participación de mercado"
- **Insight:** "Esta brecha de 13 puntos indica desventaja competitiva o problemas de precios"
- **Recomendación:** "Analizar precios de competidores y considerar estrategia promocional"

## LO QUE NO HACES
- No consultes bases de datos directamente (recibes datos pre-analizados)
- No busques en la web (recibes resultados de investigación de mercado)
- No tomes decisiones - proporcionas recomendaciones
- No especules más allá de los datos proporcionados

Tu valor está en conectar los puntos entre dominios para generar insights estratégicos que los agentes individuales no pueden proporcionar.
```

✅ **Resultado esperado:** Tenemos ahora un agente coordinador creado que de momento no esta vinculado con ningún flujo

![New Foundry](/img/supervisor.png)


---

### 4.5 - Crear el Router Agent (Orquestador Silencioso) 

El Router Agent es el cerebro del flujo. Analiza la consulta del usuario y decide en silencio a qué agente dirigirla, emitiendo **un único tag** que el workflow intercepta con condiciones Power Fx. **No responde al usuario directamente**.

**Crear el agente**
- Repite los pasos de creación de agentes anteriores
- Name: **Contoso-Router-Agent**
- No vincules ningún tool — este agente trabaja solo con el texto del query

**Instructions (system prompt)**

```
Eres el enrutador inteligente del sistema multi-agente de Contoso. Tu ÚNICA tarea es analizar la consulta del usuario y decidir qué agentes especializados deben responderla.

## REGLA CRÍTICA
No respondas en lenguaje natural. No saludes. No expliques nada.
Tu respuesta debe contener ÚNICAMENTE un tag, en una sola línea.

## LOS TAGS DISPONIBLES
- [SALES]  → Solo ventas internas
- [MARKET] → Solo mercado externo
- [CREDIT] → Solo perfiles de clientes
- [CROSS]  → Pregunta involucra 2 o más dominios

## CUÁNDO EMITIR CADA TAG

**[SALES]** — Emite este tag si la consulta menciona o implica:
- Ventas, revenue, ingresos, transacciones
- Productos, categorías, inventario, SKUs
- Canales de venta (online, tienda física, MSI)
- Tickets, órdenes, volumen de ventas
- Performance interno, comparativas de productos propios
- Palabras clave: "ventas", "revenue", "productos", "categoría", "canal", "ticket", "iPhone", "laptop", "electronics", "sales"

**[MARKET]** — Emite este tag si la consulta menciona o implica:
- Tendencias del mercado o la industria
- Competidores, benchmarks, precios externos
- Contexto externo, comportamiento del consumidor
- Palabras clave: "mercado", "tendencias", "competencia", "benchmark", "industria", "market", "trend", "competidor", "precio competitivo", "externo"

**[CREDIT]** — Emite este tag si la consulta menciona o implica:
- Perfiles crediticios o segmentos de clientes
- Capacidad de pago, scores crediticios
- Clientes Premium, Alto, Medio, Bajo
- Riesgo financiero, deuda, comportamiento de pago
- Palabras clave: "cliente", "customer", "crédito", "credit", "perfil", "profile", "score", "segmento", "capacidad de pago", "riesgo"

**[CROSS]** — Emite este tag si la consulta involucra 2 o más dominios:
- Comparación interna vs mercado externo
- Recomendaciones por perfil de cliente considerando tendencias
- Análisis que cruza ventas + crédito, ventas + mercado, o los tres

## EJEMPLOS DE RUTEO
Consulta: "¿Cuáles son nuestras ventas de iPhone este año?"
Respuesta: [SALES]

Consulta: "¿Cuáles son las tendencias del mercado para smartphones en 2024?"
Respuesta: [MARKET]

Consulta: "¿Cuántos clientes de perfil Alto tenemos?"
Respuesta: [CREDIT]

Consulta: "¿Cómo se comparan nuestras ventas de iPhone vs las tendencias del mercado?"
Respuesta: [CROSS]

Consulta: "¿Qué productos premium deberíamos recomendar a clientes de perfil Alto según las tendencias actuales?"
Respuesta: [CROSS]

Consulta: "¿Nuestros precios en laptops son competitivos según el mercado?"
Respuesta: [CROSS]

Consulta: "¿Cuál es el score promedio de nuestros clientes Premium?"
Respuesta: [CREDIT]

## REGLAS ADICIONALES
- Responde SIEMPRE con un único tag
- Ante duda entre un dominio solo o cruce, prefiere [CROSS]
```

**Validar el Router Agent**

Antes de integrarlo al workflow, prueba el Router Agent en el Playground con estas consultas y verifica que emita el tag correcto:

| Consulta de prueba | Tag esperado |
|---|---|
| ¿Cuáles son nuestras ventas de laptop este año? | `[SALES]` |
| ¿Cuáles son las tendencias del mercado para laptops? | `[MARKET]` |
| ¿Cuántos clientes tenemos con perfil crediticio Alto? | `[CREDIT]` |
| ¿Cómo se comparan nuestros precios de laptop vs el mercado? | `[CROSS]` |
| ¿Qué productos premium recomendar a clientes de perfil Alto según tendencias? | `[CROSS]` |

✅ **Resultado esperado:** El Router Agent responde con un único tag, sin texto adicional.

![New Foundry](/img/router-agent.png)

---

### 5 - Crear el Workflow Multi-Agente

**Configuración Inicial del Workflow**

Para poder construir un workflow multi-agente existen varias opciones pro-code como el uso de **Semantic Kernel** o **AutoGen** ahora fusionados dentro del [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview). Recientemente AI Foundry anunció un nuevo método llamado **Workflows** de bajo-código que permite realizar estos flujos de forma visual facilitando el proceso.

Para este ejercicio vamos a utilizar este método.

**Nota**: Antes de construir el flujo asegúrate de que tienes construidos los 5 agentes: Router, Sales, Market Research, Credit Risk y Strategy Advisor.

**Navegar a Workflows**
- En el portal de AI Foundry estando dentro de la opción **Build** navegamos al menú lateral → **Workflows**

  ![New Foundry](/img/workflow1.png)

- Click en **Create** y en la lista desplegable seleccionamos **Sequential**. Diseñaremos el flujo desde 0 comenzando con el nodo **Start**.

  ![New Foundry](/img/workflow2.png)

---

**Configurar Nodos del Workflow**

**Nodo Start**: Ya está creado. Agrega una nota opcional para documentar el flujo.

---

**Agregar Nodos: Set Variable — Capturar Query del Usuario**

Agregamos dos nodos de variable al inicio para preservar el query original y restaurarlo antes de cada agente.

- Click en **+** → **Set variable**
  - **Variable name**: `UserQuestion`
  - **Value**: `=System.LastMessageText`
- Click en **+** → **Set variable**
  - **Variable name**: `LatestMessage`
  - **Value**: `=UserMessage(Local.UserQuestion)`

*¿Por qué dos variables?* `Local.UserQuestion` guarda el texto plano del query original y no se modifica en ningún momento del flujo. `Local.LatestMessage` es la variable "activa" que cada agente recibe como input y sobreescribe con su respuesta. Antes de invocar cada agente especializado hacemos un `restore` — es decir, reasignamos `Local.LatestMessage` desde `Local.UserQuestion` — para garantizar que cada agente recibe la pregunta original del usuario y no la respuesta del agente anterior.

![New Foundry](/img/workflow3.png) ![New Foundry](/img/workflow3.1.png)

---

**Agregar Nodo: Invoke Router Agent**

Este es el primer agente que se ejecuta. Trabaja en silencio y su tag dirige el flujo.

- Click en **+** → **Invoke agent**
- Configura:
  - **Select an agent**: `Contoso-Router-Agent`
  - **Conversation context**: `System.ConversationId`
  - **Input message**: `=Local.LatestMessage`
  - **Automatically include agent response**: ❌ **Desactivado** (trabaja en silencio)
  - **Save agent output message as**: `LatestMessage` → se guarda como `Local.LatestMessage`
- Presiona **Done**

![New Foundry](/img/workflow4.png)

---

**Agregar Nodo: ConditionGroup — Routing**

Este es el nodo central. Evalúa el tag que emitió el Router y dirige el flujo.

- Click en **+** → **If/Else**

*Nota sobre el patrón `restore`:* Dentro de cada rama, antes de invocar al agente especializado, siempre agregamos un nodo **Set variable** que reasigna `Local.LatestMessage = UserMessage(Local.UserQuestion)`. Esto es necesario porque después del Router Agent, `Local.LatestMessage` contiene la respuesta del Router (los tags), no el query del usuario. Sin el restore, el agente especializado recibiría `[SALES]` como input en lugar de la pregunta original.

**Condition 1 — `[SALES]`:**
```
=!IsBlank(Find("[SALES]", Upper(Last(Local.LatestMessage).Text)))
```
Acciones (rama YES):
- **Set variable**: `LatestMessage` = `=UserMessage(Local.UserQuestion)`
- **Invoke agent**: `Contoso-Sales-Analyst` → output: `Local.LatestMessage`, autoSend: false
- **End Conversation**

  ![New Foundry](/img/ifelse.png)
  ![New Foundry](/img/ifelse1.png)
  ![New Foundry](/img/ifelse2.png)
  

**Condition 2 — `[MARKET]`:**
```
=!IsBlank(Find("[MARKET]", Upper(Last(Local.LatestMessage).Text)))
```
Acciones (rama YES):
- **Set variable**: `LatestMessage` = `=UserMessage(Local.UserQuestion)`
- **Invoke agent**: `Contoso-Market-Research-Analyst` → output: `Local.LatestMessage`, autoSend: false
- **End Conversation**

  ![New Foundry](/img/ifelse3.png)
  ![New Foundry](/img/ifelse4.png)
  ![New Foundry](/img/ifelse5.png)

**Condition 3 — `[CREDIT]`:**
```
=!IsBlank(Find("[CREDIT]", Upper(Last(Local.LatestMessage).Text)))
```
Acciones (rama  YES):
- **Set variable**: `LatestMessage` = `=UserMessage(Local.UserQuestion)`
- **Invoke agent**: `Contoso-Credit-Risk-Analyst` → output: `Local.LatestMessage`, autoSend: false
- **End Conversation**

 ![New Foundry](/img/ifelse6.png)
 ![New Foundry](/img/ifelse7.png)
 ![New Foundry](/img/ifelse8.png)
 

**elseActions — `[CROSS]` (o cualquier caso no capturado):**

Cuando el Router emite `[CROSS]`, ninguna condición individual es verdadera y el flujo cae aquí. Se ejecutan los 3 agentes en secuencia y el Strategy Advisor sintetiza.

- **Set variable**: `LatestMessage` = `=UserMessage(Local.UserQuestion)`
- **Invoke agent**: `Contoso-Sales-Analyst` → output: `Local.sales_output` (guardamos el output en una variable), autoSend: false
- **Invoke agent**: `Contoso-Market-Research-Analyst` → output: `Local.LatestMessage`, autoSend: false
- **Invoke agent**: `Contoso-Credit-Risk-Analyst` → output: `Local.LatestMessage`, autoSend: false
- **Invoke agent**: `Contoso-Strategy-Advisor` → output: `Local.LatestMessage`, autoSend: false
- **End Conversation**

![New Foundry](/img/else.png)
![New Foundry](/img/else1.png)
![New Foundry](/img/else2.png)
![New Foundry](/img/else3.png)
![New Foundry](/img/else4.png)


---

**Guardar el Workflow**
- Click en **Save** y nómbralo **Workflow-Multi-Agente-Contoso**

![New Foundry](/img/workflow9.png)

---

**Opción alternativa: YAML**

Si prefieres cargar el workflow directamente vía YAML en lugar de construirlo nodo a nodo, abre la pestaña **YAML** en el canvas y pega el contenido del archivo [workflow-multi-agente-contoso](/workflow-multi-agente-contoso.yaml) incluido en este repositorio. Luego regresa a la pestaña **Visualizer** para verificar la estructura y guarda.


### 6 - Testing y Validación el Workflow Multi-Agente
En este paso vamos a validar el flujo con preguntas que pongan a prueba el enrutamiento inteligente del Router Agent.

**Preview del Workflow**

Escenario 1:

**Query de un solo dominio — solo Sales**
- Click en **Preview** en la parte superior
- Esto abrirá una ventana de chat
- Agregamos una pregunta de prueba y ejecutamos

  ```
  ¿Cuál es el top 5 de productos más vendidos durante el 2024?
  ```

Flujo esperado:

✅ Router Agent analiza la consulta → emite `[SALES]`  
✅ Condition `[SALES]` es verdadera → Sales Agent se ejecuta  
⏭️ Market Research y Credit Risk se SALTAN  
⏭️ Strategy Advisor se SALTA (no es caso `[CROSS]`)  
✅ Respuesta directa del Sales Agent al usuario  

![New Foundry](/img/workflow11.png)

---

Escenario 2:

**Query cross-dominio — Sales + Market + Strategy Advisor**
- Click en **Preview**
- Agregamos una pregunta de prueba y ejecutamos

  ```
  ¿Cómo se comparan nuestras ventas de smartphone con las tendencias del mercado?
  ```

Flujo esperado:

✅ Router Agent → emite `[CROSS]`  
⏭️ Ninguna condición individual es verdadera → cae al `elseActions`  
✅ Sales Agent se ejecuta (datos internos de smartphones)  
✅ Market Research se ejecuta (tendencias externas de mercado)  
✅ Credit Risk se ejecuta  
✅ Strategy Advisor sintetiza y responde al usuario  

![New Foundry](/img/workflow12.png)

---

Escenario 3:

**Query de perfil de clientes — solo Credit**
- Click en **Preview**
- Agregamos una pregunta de prueba y ejecutamos

  ```
  ¿Cuántos clientes tenemos con perfil crediticio alto?
  ```

Flujo esperado:

✅ Router Agent → emite `[CREDIT]`  
✅ Condition `[CREDIT]` es verdadera → Credit Risk Agent se ejecuta  
⏭️ Sales, Market y Strategy Advisor se SALTAN  
✅ Respuesta directa del Credit Risk Agent al usuario  

![New Foundry](/img/workflow13.png)

En el menú **traces** también se puede analizar el flujo de cada nodo para efectos de resolución de problemas. Verás claramente cuáles nodos se ejecutaron y cuáles se saltaron en cada escenario.

### Recomendaciones

- El Router Agent puede refinarse continuamente ajustando su system prompt con más ejemplos de clasificación (few-shot examples). Cuantos más ejemplos específicos del dominio de Contoso incluyas, más preciso será el ruteo.
- Una recomendación adicional es extender las capacidades de este flujo con ramificaciones que consideren reintentos cuando el Router no emita tags reconocidos, y otros flujos secundarios para casos "edge".
- Como vimos, este flujo es determinístico pero inteligente en el ruteo inicial. Se invita opcionalmente a construir un flujo de tipo **Group Chat** que permita un manejo aún más autónomo de la colaboración entre agentes.

### 🎓 Bonus: Explorando Group Chat Workflow (Opcional)

**¿Por qué Group Chat es diferente?**

En Sequential, fuiste el director de orquesta - definiste cada paso del flujo. En **Group Chat**, los agentes forman una mesa redonda donde un Manager Agent (IA) decide dinámicamente quién debe hablar y cuándo.

Analogía:

**Sequential** = Director de orquesta (tú) que indica a cada músico cuándo tocar
**Group Chat** = Mesa redonda de expertos que deciden entre sí quién contribuye


**Cuándo considerar Group Chat**

Group Chat es útil cuando:

✅ Los agentes deben negociar o debatir  
✅ El orden de participación depende del contexto  
✅ Necesitas escalamiento dinámico (Tier 1 → Tier 2 → Specialist)  
✅ Quieres que la IA decida la colaboración  

**Ejemplos de casos de uso ideales:**

- Customer Support Escalation: Agente básico → Especialista → Manager (según complejidad)  
- Medical Diagnosis: Múltiples especialistas discuten síntomas y llegan a consenso  
- Legal Case Analysis: Abogados debaten estrategia colaborativamente  
- Product Design: Designer, Engineer, PM iteran dinámicamente  

### 📚 Recursos Adicionales

[Orchestrating Multi-Agent Conversations with Microsoft Foundry Workflows](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/orchestrating-multi-agent-conversations-with-microsoft-foundry-workflows/4472329)   
[Multi-Agent Orchestration Patterns](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/building-a-digital-workforce-with-multi-agents-in-azure-ai-foundry-agent-service/4414671)  
[Agent Framework Examples](https://github.com/microsoft/agent-framework)  
[Building No-Code Agentic Workflows with Microsoft Foundry](https://medium.com/data-science-collective/building-no-code-agentic-workflows-with-microsoft-foundry-52ad377ad644)  


