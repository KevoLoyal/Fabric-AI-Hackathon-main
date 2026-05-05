# Solução Desafio 03 - Modelo Semântico, Data Agent e Dashboard de Valor

Guia passo a passo para usar dados da camada Gold no Microsoft Fabric, construir um modelo semântico, validar perguntas de negócio, criar um dashboard de valor e configurar um Data Agent.

### Objetivo 🎯
- Projetar um modelo semântico em modo `Direct Lake` usando tabelas da camada Gold que respondam a um cenário de negócio, criando medidas úteis e relacionamentos.  
- Construir um `Data Agent` no Microsoft Fabric conectando-o ao seu modelo semântico e fornecendo instruções ao LLM para responder de forma apropriada e com precisão semântica em linguagem natural.
- Desenvolver um `Dashboard interativo no Power BI` manualmente ou com a ajuda do Copilot para Power BI, incluindo visualizações de valor para o negócio.
- Validar as respostas a perguntas em linguagem natural a partir do `Copilot de Power BI` e do `Data Agent`

---

## Requisitos prévios

- Transformação intermediária, análise exploratória (Silver) e Preparação Gold (ver `02-Solution.md`).


## Passos

### 1 - Projeto do modelo semântico no Microsoft Fabric
Partindo do nosso `Lakehouse`, já com todas as camadas e dados atualizados, vamos criar um novo modelo semântico em modo [Direct Lake](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview):

1. Dentro do painel principal do Lakehouse, selecione `New semantic model`
2. No painel do novo modelo semântico, adicione um nome ao modelo, por exemplo: `Contoso-semantic-model` (observe que o Direct Lake já está habilitado por padrão).
3. Selecione o `Workspace` onde está o nosso `Lakehouse` e prossiga para selecionar as tabelas delta da nossa camada `Gold` que usaremos no modelo.  

**Nota**: Nesta solução que construímos, projetamos a camada Gold unindo *silver.transactions* + *silver.products* para obter **gold.business_operations**, o que nos permite abordar um cenário de *análise de compras por produtos*, mas também nos permite vinculá-la com *credit_score* para correlacionar **segmentação de clientes por score com padrões de compra**, portanto não estamos incluindo **gold.products** no modelo semântico para evitar redundância. Ainda assim, se outra estratégia de modelagem tiver sido seguida, é totalmente válido incluí-la.

O resultado é um novo modelo semântico a partir do qual passamos a projetar nossas medidas DAX e relacionamentos (se desenvolvemos tabelas dimensionais, elas também seriam válidas aqui)

![Semantic](/img/semantic.png)

4. Uma vez dentro do nosso modelo semântico, passamos a estabelecer relacionamentos entre as tabelas correspondentes. Na barra de tarefas, selecione `Manage relationships` → `+ New relationship`. 
5. No menu de novo relacionamento, vamos selecionar uma tabela, por exemplo: `business_operations` (união de transactions + products)` e verificar o campo-chave `customer_id`.
6. Selecione a outra tabela a ser unida, neste caso `credit_score`, e verifique o campo-chave `customer_id`; configure a cardinalidade de acordo com a modelagem que você fez na camada Gold, e as demais opções podem ficar no padrão.
7. Clique em `Save` → `Close`

Já temos nossas tabelas relacionadas no modelo 

![Semantic](/img/relationship.png)

O próximo passo é construir algumas medidas DAX que nos permitam facilitar o desenvolvimento de dashboards e relatórios, além de serem utilizadas pelos `Data Agents`

8. Selecionando a tabela `business_operations` do modelo, procure na barra de tarefas a opção `New measure` 
9. Na barra de fórmulas, insira a medida DAX que deseja construir e pressione `Enter`. A medida é adicionada ao modelo
10. Repita o mesmo processo com as medidas que deseja incluir. Aqui estão alguns exemplos de medidas para esta solução específica:

**Total Revenue**
```
Total Revenue = SUM(business_operations[amount])
```
**Ticket Promedio**
```
Avg Ticket = AVERAGE(business_operations[amount])
```
**Total Transacciones**
```
Total Transactions = COUNTROWS(business_operations)
```
**Clientes Activos**
```
Active Customers = DISTINCTCOUNT(business_operations[customer_id])
```
**Revenue por perfil crediticio**
```
Revenue by Credit Profile = 
CALCULATE(
    [Total Revenue],
    USERELATIONSHIP(business_operations[customer_id], credit_score[customer_id])
)
```
**% MSI (Meses sin Intereses)**
```
% MSI = 
DIVIDE(
    COUNTROWS(FILTER(business_operations, business_operations[is_msi] = TRUE)),
    COUNTROWS(business_operations),
    0
)
```
```
Revenue by Credit Profile = 
CALCULATE(
    [Total Revenue],
    USERELATIONSHIP(business_operations[customer_id], credit_score[customer_id])
)
```

**Revenue YTD (Year-to-Date)**
```
Revenue YTD = 
TOTALYTD(
    [Total Revenue],
    business_operations[transaction_date]
)
```

O resultado é um modelo com relacionamentos e medidas prontas para serem utilizadas.

![Model-ready](/img/model-ready.png)


### 2 - Validar o Modelo com Perguntas de Negócio
Antes de passar para a construção do dashboard e dos relatórios, vamos validar com a ajuda do Copilot-PBI se o modelo nos fornece informações de contexto de negócio em linguagem natural sobre os dados:

1. No menu principal do Fabric, sobre o item do modelo semântico, procure `Create report` e, no modo de edição, ative o ícone do Copilot; depois faça perguntas sobre o contexto dos dados no Copilot Power BI. **Nota:** Certifique-se de que o modelo semântico tenha o recurso `Q&A - Turn on Q&A to ask natural language questions about your data` ativado para que o Copilot possa trabalhar perguntas sobre o modelo. Isso é habilitado em `settings` do seu modelo semântico.

![QA](/img/qa_settings.png)

2. Uma vez ativado, passamos a fazer perguntas no menu de chat. Aqui estão alguns exemplos:

💬 “¿Qué categoría tiene el precio promedio más alto?”  
💬 "¿Cuál es el total de transacciones?"  
💬 “¿Qué perfil de producto genera más ingresos?” (basado en la medida derivada)  

3. Se alguma resposta não estiver correta, ajuste as medidas ou os relacionamentos no modelo.

✅ Resultado esperado: O modelo responde de forma precisa e coerente às perguntas de negócio. Inclusive, posso começar a construir visuais com esses outputs.

![PBI Copilot](/img/pbi-copilot.png)



### 3 - Projeto de um dashboard de valor
O desenho do dashboard é um aspecto muito próprio do contexto organizacional e dos requisitos de cada organização. Com essa variedade em mente, o Power BI oferece muitas opções para customizar e construir painéis corporativos que respondam a perguntas de negócio e facilitem a tomada de decisão. Para este dashboard específico, pensou-se em um dashboard com vários relatórios; aqui está o diagrama UX de referência:


![Executive Summary](/img/executive_dash.png)


Com essa ideia em mente, podemos proceder para construir um dashboard manualmente, configurando cada aspecto dos relatórios. 


![Executive Summary](/img/pbi-executive.png)


Para este exercício, também podemos contar com o apoio do Copilot para que, com base nos dados, ele nos sugira conteúdo para um novo relatório, criando os visuais automaticamente. No menu principal do relatório, tente o seguinte:

- Crie um novo relatório sobre o modelo semântico
- No menu do relatório, ative a opção `Copilot` e pressione a opção `Suggest content for a new report page`
- Deixe o LLM (modelo subjacente) analisar o contexto do modelo semântico e gerar as opções que considera mais adequadas.
- Quando responder, selecione alguma das recomendações e clique em `+ Create` para observar como é gerado um relatório criado com inteligência artificial generativa.

✅ Resultado esperado: Você tem um novo relatório criado pelo Copilot; agora pode validá-lo e repetir com novas sugestões para completar o seu dashboard.

![Cop](/img/copilot_pbi2.png)
![Cop2](/img/copilot_pbi3.png)

### 4 - Criação de um Data Agent 
Passamos agora a criar o nosso Data Agent. 
1. No nosso **Workspace do Fabric**, vamos criar um novo item: `→ New item → Data agent`.
2. Dê um nome ao seu agente, por exemplo: `Contoso-Business Operations Agent`. Neste exemplo, queremos um agente que trabalhe o escopo de transações e produtos (retail)
3. No painel do novo Data Agent, configuramos:
   - **Data Source**: Clique em `+ Data source` ou `Add data source` → procure o nosso Lakehouse → `Add` e selecione a nossa tabela `[gold.business_operations]`. Também podemos conectá-lo a modelos semânticos ou outros repositórios de dados nativos do Fabric.
  
![Cop](/img/data_agent_ops.png)

   - **Setup**: Aqui, o objetivo é fornecer instruções claras e concisas ao modelo LLM que roda nos bastidores, para que, usando o modelo semântico, ele saiba interpretar os conceitos de negócio e evitemos ambiguidades que provoquem imprecisões nas respostas. Aqui está um exemplo de instruções para *agent instructions* (system prompt), assim como para a seção de *Data source instructions*:


**Sección - Agent instructions**

```
## PROPÓSITO
Eres un asistente de análisis de datos especializado en operaciones comerciales retail. Tu objetivo es ayudar a usuarios de negocio a entender el desempeño de ventas, productos, canales y comportamiento transaccional.

## REGLAS DE PLANEACIÓN
1. **Identifica el tipo de pregunta**: ventas, productos, canales, temporal, o combinación
2. **Usa business_operations** para: revenue, transacciones, productos, canales, MSI, temporal
3. **No especules**: Si no tienes los datos, dilo claramente
4. **Prioriza precisión** sobre velocidad

## TERMINOLOGÍA CONSISTENTE
- **Revenue** = suma de amount (no "ingresos" o "ganancias")
- **MSI** = Meses Sin Intereses (financiamiento sin intereses)
- **Ticket promedio** = promedio de amount por transacción
- **Canal** = channel (Online, Store, Mobile App, Call Center)
- **Perfil crediticio** = NO tienes acceso (refiere al otro agente)

## TONO Y FORMATO
- **Profesional pero accesible**: Evita jerga técnica innecesaria
- **Números primero**: Siempre incluye cifras concretas
- **Contexto después**: Explica qué significan los números
- **Bullets para listas**: Usa viñetas para múltiples items
- **Comparaciones**: Siempre que sea posible ("40% más que...")
- **Insights accionables**: Termina con "qué hacer con esto"
```
---

**Sección - Data source descriptions**

**Data source descriptions**
```
Esta tabla contiene todas las transacciones de venta del año 2024, incluyendo información detallada de productos, canales de venta, métodos de pago y dimensiones temporales.
**Contenido:**
- Transacciones aprobadas
- Periodo: Enero a Diciembre 2024
- Revenue total
**Campos principales:**
- **IDs**: transaction_id, customer_id, product_id
- **Producto**: product_name, brand, category, price
- **Transacción**: transaction_date, amount, quantity
- **Pago**: payment_method, installments, is_msi, is_credit
- **Canal**: channel, store_location
- **Tiempo**: year, month, quarter, year_month
- **Segmentos**: ticket_segment (High)

```

**Data Source Instructions**

```
## ROL
Eres un analista de operaciones comerciales especializado en retail que ayuda a responder preguntas sobre ventas, productos, canales y comportamiento de compra.

## TU EXPERTISE
- Análisis de revenue usando el campo **amount**
- Performance por **channel** (Online, Store, Mobile App, Call Center)
- Top productos usando **product_name**, **brand**, **category**
- Adopción de MSI usando **is_msi** e **installments**
- Tendencias temporales con **transaction_date**, **year**, **month**, **quarter**
- Métodos de pago con **payment_method** e **is_credit**
- Segmentación con **ticket_segment**

## CAMPOS CLAVE Y SU USO
- **amount**: Para calcular revenue total, promedio, sumas
- **quantity**: Para contar unidades vendidas
- **channel**: Para comparar Online vs Store vs Mobile App vs Call Center
- **payment_method**: Credit, Debit, Cash
- **is_msi**: true = usa Meses Sin Intereses, false = pago único
- **installments**: número de cuotas (1 = pago único, >1 = financiado)
- **ticket_segment**: Low (<$500), Medium ($500-$1000), High (>$1000)
- **category**: Electronics, Kitchen Appliances, Fitness Equipment, etc.

## LO QUE NO INCLUYES
- Información de perfiles crediticios de clientes
- Productos del catálogo que no se han vendido
- Scores o análisis de riesgo

## FORMATO DE RESPUESTAS
1. Siempre incluye **números concretos** (revenue, unidades, %)
2. **Compara** cuando sea relevante ("Online genera 40% más que Store")
3. Identifica **top performers** (top 5-10)
4. Usa los **nombres exactos de los campos** en tus explicaciones

## EJEMPLOS DE QUERIES ÚTILES
- Revenue total: `SUM(amount)`
- Ticket promedio: `AVG(amount)`
- Transacciones por canal: `GROUP BY channel`
- Adopción MSI: `WHERE is_msi = true`
- Top productos: `GROUP BY product_name ORDER BY SUM(amount) DESC`
```

![Cop](/img/data_agent_ops2.png)


Opcionalmente, podemos fornecer queries de exemplo como parte do seu conjunto de instruções na seção `Example queries`. Quando tivermos isso pronto, podemos passar a testar o agente.


### 5 - Validação de itens com perguntas em linguagem natural
Vamos agora testar uma interação com o agente fazendo perguntas sobre o contexto desse subconjunto dos nossos dados

💬 ¿Cuánto revenue generamos en 2024?  
💬 ¿Qué canal genera más ventas?  
💬 ¿Cuáles son nuestros productos estrella?  
💬 ¿Qué categorías son las más rentables?  
💬 ¿Cómo prefieren pagar nuestros clientes?  

![Cop](/img/data_agent_ops3.png)

✅ Resultado esperado: Você tem um novo agente de dados dentro do Microsoft Fabric pronto para responder perguntas e realizar análises sobre um contexto dos dados da sua organização.