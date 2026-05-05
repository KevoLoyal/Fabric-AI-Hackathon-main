# Solution - Desafio 5: Orquestração Multiagente

Guia passo a passo para habilitar um multiagente a partir do AI Foundry integrado com o Data Agent do Microsoft Fabric e outros agentes. Esta solução inclui um **Router Agent** que orquestra o fluxo de forma inteligente: emite uma única tag (`[SALES]`, `[MARKET]`, `[CREDIT]` ou `[CROSS]`) para direcionar a consulta ao agente correto, ou ativar todos os agentes especializados mais o Strategy Advisor quando a pergunta cruza múltiplos domínios.

### Pré-requisitos 🎯
Antes de começar, certifique-se de ter:  
✅ Acesso ao Microsoft Foundry com permissões para criação de agentes e workflows  
✅ Sales Operations Analyst (Contoso-Virtual-Analyst) já criado e funcional  


**Verificação de Pré-requisitos**
- Criação de um Agente Conversacional no AI Foundry com Integração ao Microsoft Fabric - (ver `04-Solution.md`).
- Navegue até **Foundry Portal** → **Agents**
- Verifique que existe: **Contoso-Virtual-Analyst** (Sales Operations Analyst ou o Agente que você criou conectado ao Fabric)
- Vá em **Tools** e confirme que **Bing Search** está disponível como ferramenta

  **Protótipo da solução**

   ![New Foundry](/img/multi-flujo.png)


## Passos

### 1 - Criar Credit Risk Analyst Data Agent no Fabric (você pode criar outro conforme modelou o seu cenário)

**Criar novo data agent de Credit**

- Siga o mesmo procedimento que usamos para criar o primeiro data agent em *03-Solution.md*
- Em vez de conectá-lo ao subconjunto de dados de **gold.business_operations**, vincule este novo agente a outra tabela, neste exemplo **gold.credit_score**
- Inclua instruções relevantes para o modelo
- Valide com perguntas e respostas e publique-o.

  Aqui está um exemplo de instruções para os dados deste exemplo:


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


✅ **Resultado esperado:** Agora temos dois agentes no Fabric: um orientado a *retail/vendas (business_operations)* e outro orientado a *score de crédito/clientes (credit_scores)*

 ![New Foundry](/img/fabric-two-agents.png)


---

### 1.2 Modificar o system prompt do agente existente de vendas (Contoso-Sales-Analyst)

Em preparação para o fluxo multiagente, vamos ajustar o system-prompt do primeiro agente que tínhamos criado no Foundry, conectado ao `Contoso-Sales Agent de Fabric`. Isso vai permitir que este agente opere de forma mais organizada com os demais agentes especializados.

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

### 2 - Criar Agente no AI Foundry conectado ao Data Agent Credit Risk Analyst (Risco de Crédito)

Criar Novo Agente
  - Repita os mesmos passos que seguimos ao criar o nosso primeiro agente em *04-Solution.md*, incluindo a conexão com o novo Fabric Data Agent e as configurações, instruções e validações.
  - Name: **Contoso-Credit-Risk-Analyst**
  - Para as instruções (system prompt), estamos usando algo semelhante ao primeiro agente; adapte-o às suas necessidades e ao seu cenário.

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

✅ **Resultado esperado:** Agora temos dois agentes no AI Foundry conectados, por sua vez, a dois data agents no Fabric. Em cenários reais, dependendo do domínio de dados, pode-se consolidar um data agent no nível do modelo semântico (com várias tabelas), mas em ambientes multissetoriais convém manter uma separação por área de negócio.

![New Foundry](/img/credit-risk-agent.png)

---

### 3 - Criar um Agente no AI Foundry para Market Research (Pesquisa)

- Repita os mesmos passos que seguimos ao criar o nosso primeiro agente em *04-Solution.md*, incluindo a conexão com o novo Fabric Data Agent e as configurações, instruções e validações.
- Name: **Contoso-Market-Research-Analyst**
- Para as instruções (system prompt), estamos utilizando o seguinte conjunto de instruções; adapte-o às suas necessidades e ao seu cenário.

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
  - Em **Tools**, clique em **Add** → **+ Add a new tool**
  - Selecione **Grouding with Bing Search**
  - Configuração:
    **Connection**: Configuramos uma nova conexão com o Bing Search → **Connect to a new resource** → **Create a new resource** e aqui vamos criar um novo recurso do Bing Search dentro do nosso tenant do Azure para que possa ser utilizado pelo agente. Preenchemos as opções de acordo com o nosso ambiente e o Grupo de Recursos utilizado e deixamos as demais opções no padrão.

    ![New Foundry](/img/bing-search-agent.png)

    Quando o recurso estiver disponível, voltamos ao nosso agente no Foundry e vinculamos o recurso à conexão

    ![New Foundry](/img/bing-search-tool2.png)

    
    **Count**: Número de resultados de busca que o Bing retornará. Recomenda-se 5 para buscas rápidas e concisas, 10 se a análise for mais profunda e forem necessárias mais fontes para comparar
    **Set language**: Idioma dos resultados; para espanhol, deixe em **es**
    **Market**: Região de mercado para resultados localizados; podemos manter em **es-mx**
    **Freshness**: Filtro de atualização/atualidade dos resultados no formato *YYYY-MM-DD*. Pode ser deixado em branco.

 
3. Validar o agente com o grounding do Bing Search
  - Fazemos algumas perguntas sobre comportamento de mercado, por exemplo sobre produtos que fazem parte do catálogo retail"

```
¿Cuáles son las tendencias actuales del mercado para smartphones premium en 2024?
```

 - Quando o agente estiver corretamente configurado, nós o publicamos

✅ **Resultado esperado:** Agora temos um agente de research que faz *grounding* por meio do **Bing Search** para análise de mercado global.


![New Foundry](/img/grounding-bing.png)

---

### 4 - Criar um Agente no AI Foundry de Strategy Advisor (Sintetiza a informação)

Este agente vai assumir o papel de coordenador de tarefas e permite delegar as tarefas ao agente mais indicado para o que o usuário está consultando. Para este agente, não precisamos vincular nenhum tool, já que ele recebe dados de outros agentes via workflow, que vamos construir mais adiante.

- Repita os mesmos passos que seguimos ao criar nossos agentes anteriores no AI Foundry
- Name: **Contoso-Strategy-Advisor**
- Para as instruções (system prompt), estamos utilizando o seguinte conjunto de instruções; adapte-o às suas necessidades
- Finalize publicando o agente, ainda não o teste, pois ele não tem nenhum contexto vinculado

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

✅ **Resultado esperado:** Agora temos um agente coordenador criado que, por enquanto, não está vinculado a nenhum fluxo

![New Foundry](/img/supervisor.png)


---

### 4.5 - Criar o Router Agent (Orquestrador Silencioso) 

O Router Agent é o cérebro do fluxo. Ele analisa a consulta do usuário e decide silenciosamente para qual agente direcioná-la, emitindo **uma única tag** que o workflow intercepta com condições Power Fx. **Ele não responde diretamente ao usuário**.

**Criar o agente**
- Repita os passos de criação de agentes anteriores
- Name: **Contoso-Router-Agent**
- Não vincule nenhum tool — este agente trabalha apenas com o texto da query

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

**Validar o Router Agent**

Antes de integrá-lo ao workflow, teste o Router Agent no Playground com estas consultas e verifique se ele emite a tag correta:

| Consulta de prueba | Tag esperado |
|---|---|
| ¿Cuáles son nuestras ventas de laptop este año? | `[SALES]` |
| ¿Cuáles son las tendencias del mercado para laptops? | `[MARKET]` |
| ¿Cuántos clientes tenemos con perfil crediticio Alto? | `[CREDIT]` |
| ¿Cómo se comparan nuestros precios de laptop vs el mercado? | `[CROSS]` |
| ¿Qué productos premium recomendar a clientes de perfil Alto según tendencias? | `[CROSS]` |

✅ **Resultado esperado:** O Router Agent responde com uma única tag, sem texto adicional.

![New Foundry](/img/router-agent.png)

---

### 5 - Criar o Workflow Multiagente

**Configuração Inicial do Workflow**

Para construir um workflow multiagente, existem várias opções pro-code, como o uso de **Semantic Kernel** ou **AutoGen**, agora fundidos dentro do [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview). Recentemente, o AI Foundry anunciou um novo método chamado **Workflows** de baixo código, que permite realizar esses fluxos de forma visual, facilitando o processo.

Para este exercício, vamos utilizar esse método.

**Nota**: Antes de construir o fluxo, certifique-se de que você já construiu os 5 agentes: Router, Sales, Market Research, Credit Risk e Strategy Advisor.

**Navegar para Workflows**
- No portal do AI Foundry, estando dentro da opção **Build**, navegamos até o menu lateral → **Workflows**

  ![New Foundry](/img/workflow1.png)

- Clique em **Create** e, na lista suspensa, selecione **Sequential**. Vamos projetar o fluxo do zero, começando com o nó **Start**.

  ![New Foundry](/img/workflow2.png)

---

**Configurar Nós do Workflow**

**Nó Start**: Já está criado. Adicione uma nota opcional para documentar o fluxo.

---

**Adicionar Nós: Set Variable — Capturar Query do Usuário**

Adicionamos dois nós de variável no início para preservar a query original e restaurá-la antes de cada agente.

- Clique em **+** → **Set variable**
  - **Variable name**: `UserQuestion`
  - **Value**: `=System.LastMessageText`
- Clique em **+** → **Set variable**
  - **Variable name**: `LatestMessage`
  - **Value**: `=UserMessage(Local.UserQuestion)`

*Por que duas variáveis?* `Local.UserQuestion` guarda o texto puro da query original e não é modificado em nenhum momento do fluxo. `Local.LatestMessage` é a variável "ativa" que cada agente recebe como input e sobrescreve com a sua resposta. Antes de invocar cada agente especializado, fazemos um `restore` — isto é, reatribuímos `Local.LatestMessage` a partir de `Local.UserQuestion` — para garantir que cada agente receba a pergunta original do usuário e não a resposta do agente anterior.

![New Foundry](/img/workflow3.png) ![New Foundry](/img/workflow3.1.png)

---

**Adicionar Nó: Invoke Router Agent**

Este é o primeiro agente que é executado. Ele trabalha em silêncio e a sua tag direciona o fluxo.

- Clique em **+** → **Invoke agent**
- Configure:
  - **Select an agent**: `Contoso-Router-Agent`
  - **Conversation context**: `System.ConversationId`
  - **Input message**: `=Local.LatestMessage`
  - **Automatically include agent response**: ❌ **Desativado** (trabalha em silêncio)
  - **Save agent output message as**: `LatestMessage` → é salvo como `Local.LatestMessage`
- Pressione **Done**

![New Foundry](/img/workflow4.png)

---

**Adicionar Nó: ConditionGroup — Roteamento**

Este é o nó central. Ele avalia a tag emitida pelo Router e direciona o fluxo.

- Clique em **+** → **If/Else**

*Nota sobre o padrão `restore`:* Dentro de cada ramo, antes de invocar o agente especializado, sempre adicionamos um nó **Set variable** que reatribui `Local.LatestMessage = UserMessage(Local.UserQuestion)`. Isso é necessário porque, depois do Router Agent, `Local.LatestMessage` contém a resposta do Router (as tags), não a query do usuário. Sem o restore, o agente especializado receberia `[SALES]` como input em vez da pergunta original.

**Condition 1 — `[SALES]`:**
```
=!IsBlank(Find("[SALES]", Upper(Last(Local.LatestMessage).Text)))
```
Ações (ramo YES):
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
Ações (ramo YES):
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
Ações (ramo YES):
- **Set variable**: `LatestMessage` = `=UserMessage(Local.UserQuestion)`
- **Invoke agent**: `Contoso-Credit-Risk-Analyst` → output: `Local.LatestMessage`, autoSend: false
- **End Conversation**

 ![New Foundry](/img/ifelse6.png)
 ![New Foundry](/img/ifelse7.png)
 ![New Foundry](/img/ifelse8.png)
 

**elseActions — `[CROSS]` (ou qualquer caso não capturado):**

Quando o Router emite `[CROSS]`, nenhuma condição individual é verdadeira e o fluxo cai aqui. Os 3 agentes são executados em sequência e o Strategy Advisor sintetiza.

- **Set variable**: `LatestMessage` = `=UserMessage(Local.UserQuestion)`
- **Invoke agent**: `Contoso-Sales-Analyst` → output: `Local.sales_output` (guardamos o output em uma variável), autoSend: false
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

**Salvar o Workflow**
- Clique em **Save** e dê o nome **Workflow-Multi-Agente-Contoso**

![New Foundry](/img/workflow9.png)

---

**Opção alternativa: YAML**

Se você preferir carregar o workflow diretamente via YAML em vez de construí-lo nó a nó, abra a aba **YAML** no canvas e cole o conteúdo do arquivo [workflow-multi-agente-contoso](/workflow-multi-agente-contoso.yaml) incluído neste repositório. Depois volte para a aba **Visualizer** para verificar a estrutura e salve.


### 6 - Testar e Validar o Workflow Multiagente
Neste passo, vamos validar o fluxo com perguntas que coloquem à prova o roteamento inteligente do Router Agent.

**Preview do Workflow**

Cenário 1:

**Query de um único domínio — apenas Sales**
- Clique em **Preview** na parte superior
- Isso abrirá uma janela de chat
- Adicionamos uma pergunta de teste e executamos

  ```
  ¿Cuál es el top 5 de productos más vendidos durante el 2024?
  ```

Fluxo esperado:

✅ Router Agent analisa a consulta → emite `[SALES]`  
✅ Condition `[SALES]` é verdadeira → Sales Agent é executado  
⏭️ Market Research e Credit Risk são IGNORADOS  
⏭️ Strategy Advisor é IGNORADO (não é caso `[CROSS]`)  
✅ Resposta direta do Sales Agent ao usuário  

![New Foundry](/img/workflow11.png)

---

Cenário 2:

**Query cross-domain — Sales + Market + Strategy Advisor**
- Clique em **Preview**
- Adicionamos uma pergunta de teste e executamos

  ```
  ¿Cómo se comparan nuestras ventas de smartphone con las tendencias del mercado?
  ```

Fluxo esperado:

✅ Router Agent → emite `[CROSS]`  
⏭️ Nenhuma condição individual é verdadeira → cai em `elseActions`  
✅ Sales Agent é executado (dados internos de smartphones)  
✅ Market Research é executado (tendências externas de mercado)  
✅ Credit Risk é executado  
✅ Strategy Advisor sintetiza e responde ao usuário  

![New Foundry](/img/workflow12.png)

---

Cenário 3:

**Query de perfil de clientes — apenas Credit**
- Clique em **Preview**
- Adicionamos uma pergunta de teste e executamos

  ```
  ¿Cuántos clientes tenemos con perfil crediticio alto?
  ```

Fluxo esperado:

✅ Router Agent → emite `[CREDIT]`  
✅ Condition `[CREDIT]` é verdadeira → Credit Risk Agent é executado  
⏭️ Sales, Market e Strategy Advisor são IGNORADOS  
✅ Resposta direta do Credit Risk Agent ao usuário  

![New Foundry](/img/workflow13.png)

No menu **traces**, também é possível analisar o fluxo de cada nó para fins de troubleshooting. Você verá claramente quais nós foram executados e quais foram ignorados em cada cenário.

### Recomendações

- O Router Agent pode ser refinado continuamente ajustando seu system prompt com mais exemplos de classificação (few-shot examples). Quanto mais exemplos específicos do domínio da Contoso você incluir, mais preciso será o roteamento.
- Uma recomendação adicional é estender as capacidades desse fluxo com ramificações que considerem retentativas quando o Router não emitir tags reconhecidas, além de outros fluxos secundários para casos de borda.
- Como vimos, este fluxo é determinístico, mas inteligente no roteamento inicial. Fica o convite opcional para construir um fluxo do tipo **Group Chat**, que permita um manejo ainda mais autônomo da colaboração entre agentes.

### 🎓 Bônus: Explorando Group Chat Workflow (Opcional)

**Por que Group Chat é diferente?**

No Sequential, você foi o diretor de orquestra — definiu cada etapa do fluxo. No **Group Chat**, os agentes formam uma mesa redonda em que um Manager Agent (IA) decide dinamicamente quem deve falar e quando.

Analogia:

**Sequential** = Diretor de orquestra (você) que diz a cada músico quando tocar
**Group Chat** = Mesa redonda de especialistas que decidem entre si quem contribui


**Quando considerar Group Chat**

Group Chat é útil quando:

✅ Os agentes precisam negociar ou debater  
✅ A ordem de participação depende do contexto  
✅ Você precisa de escalonamento dinâmico (Tier 1 → Tier 2 → Specialist)  
✅ Você quer que a IA decida a colaboração  

**Exemplos de casos de uso ideais:**

- Customer Support Escalation: Agente básico → Especialista → Manager (conforme a complexidade)  
- Medical Diagnosis: Múltiplos especialistas discutem sintomas e chegam a um consenso  
- Legal Case Analysis: Advogados debatem a estratégia de forma colaborativa  
- Product Design: Designer, Engineer, PM iteram dinamicamente  

### 📚 Recursos Adicionais

[Orchestrating Multi-Agent Conversations with Microsoft Foundry Workflows](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/orchestrating-multi-agent-conversations-with-microsoft-foundry-workflows/4472329)   
[Multi-Agent Orchestration Patterns](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/building-a-digital-workforce-with-multi-agents-in-azure-ai-foundry-agent-service/4414671)  
[Agent Framework Examples](https://github.com/microsoft/agent-framework)  
[Building No-Code Agentic Workflows with Microsoft Foundry](https://medium.com/data-science-collective/building-no-code-agentic-workflows-with-microsoft-foundry-52ad377ad644)  
