# **Desafio 2 – Transformação intermediária, análise exploratória (Silver) e Preparação Gold 🔧📊**

## **Objetivo e solução passo a passo 🧭**

### **Objetivo 🎯**
Transformar os dados Bronze, realizar análise exploratória na Silver e, por fim, construir a camada Gold. 

---

## **Solução passo a passo 🪜**

Nesta abordagem, decidiu-se trabalhar três produtos de dados: **Credit Scores** com medallion + segmentação de clientes com ML (KMeans), **Products** com medallion + segmentação de produtos por valor comercial com ML (KMeans), e **Business Operations**, que combina tabelas de transactions e products para modelar uma análise operacional de compras de clientes e produtos. O exercício não pretende impor uma modelagem dimensional estrita; em vez disso, convida a trabalhar com diferentes abordagens. O Fabric, como ferramenta, adapta-se tanto a abordagens de modelagem dimensional quanto a abordagens de desnormalização comuns em ambientes de Big Data.

A seguir, a estrutura utilizada:

![Enfoque](/img/approach.png)




### **Conjunto de score de crédito 🧩**  
(Este é um exemplo de como trabalhar o cenário financeiro de credit; pode ser feito com outras abordagens)

- Criar novo Dataflow Gen2 ou Notebook para a camada Silver
- Configurar a origem nas tabelas Bronze.
- Aplicar transformações intermediárias 
- Criar coluna de score de crédito, derivadas
- Segmentar clientes por perfil de crédito 
- Salvar no Lakehouse como uma tabela silver
- Criar um novo Notebook para a camada Gold de score de crédito
- Configurar a origem nas tabelas Silver.
- Identificar o cluster com maior média de score 
- Filtrar os clientes pertencentes a esse cluster
- Armazenar os dados na camada Gold
- Resultado: subconjunto de clientes com perfil de crédito alto 

---

# **CAMADA SILVER - SCORE DE CRÉDITO**

# **Exemplo - Criação de tabelas silver - Score de Crédito 🧮**


## **Carregar tabela bronze com dados financeiros 🧾**

```python

df_fin = spark.sql("SELECT * FROM Contoso_Lakehouse.bronze.credit_score") 
```

---

## **Derivar coluna score_estimado com base no comportamento de pagamento e uso de crédito 💳**

```python

from pyspark.sql.functions import col, when, udf 
from pyspark.sql.types import StringType 

df_fin = df_fin.withColumn("score_estimado", 
    when(col("Payment_Behaviour") == "High_spent_Small_value_payments", 650)
    .when(col("Payment_Behaviour") == "Low_spent_Large_value_payments", 750)
    .when(col("Payment_Behaviour") == "High_spent_Large_value_payments", 800)
    .when(col("Payment_Behaviour") == "Low_spent_Small_value_payments", 600)
    .otherwise(620)
)
```
---

## **Penalização por pagamentos em atraso ⏰**

```python
df_fin = df_fin.withColumn("score_estimado", 
    col("score_estimado") - (col("Num_of_Delayed_Payment") * 5)
)
```

---

## **Penalização por alto uso de crédito 📉**

```python
df_fin = df_fin.withColumn("score_estimado", 
    when(col("Credit_Utilization_Ratio") > 0.8, col("score_estimado") - 20)
    .otherwise(col("score_estimado"))
)
```

---

## **Limitar score entre 300 e 850 ⚙️**

```python
df_fin = df_fin.withColumn("score_estimado", 
    when(col("score_estimado") < 300, 300)
    .when(col("score_estimado") > 850, 850)
    .otherwise(col("score_estimado"))
)
```

---

## **Filtrar registros válidos para clustering 🧹**

```python
df_fin_clean = df_fin.filter(col("score_estimado").isNotNull()) 

```

---

## **Padronizar colunas para lower case**

```python
df_fin_clean = df_fin_clean .toDF(*[c.lower() for c in df_fin_clean.columns])
```
---
## **Filtrar clientes sem nulos/na + idades e quantidade de empréstimos corretas**
```python
df_fin_clean = df_fin_clean.dropna().filter(
col("age").between(1, 120) & 
(col("num_of_loan") >= 0) &
(col("credit_history_age") != "NA"))
```
---

## **💾 Salvar tabela silver preparada**

```python
df_fin_clean.write.option("overwriteSchema", "true").mode("overwrite").saveAsTable("Contoso_Lakehouse.silver.credit_score") 

```

---


# **CAMADA GOLD - SCORE DE CRÉDITO**

# **Exemplo - Segmentação de clientes por score de crédito 🧮**


## **📌 Importar funções necessárias**

```python
from pyspark.sql.functions import col, when, udf 
from pyspark.sql.types import StringType 
from pyspark.ml.feature import VectorAssembler 
from pyspark.ml.clustering import KMeans 
```


## **Carregar tabela Silver com dados financeiros preparados 🧾**

```python
df_fin = spark.sql("SELECT * FROM Contoso_Lakehouse.silver.credit_score")

```

---

## **Vetorizar coluna score_estimado para ML 🤖**

```python
assembler = VectorAssembler(inputCols=["score_estimado"], outputCol="features") 
df_fin_vec = assembler.transform(df_fin_clean) 
```

---

## **Aplicar clustering KMeans para segmentar clientes 🧠**

```python
kmeans = KMeans(k=3, seed=42) 
model_fin = kmeans.fit(df_fin_vec) 
df_fin_clustered = model_fin.transform(df_fin_vec) 
```

---

## **Rotular perfis de crédito conforme a média de score por cluster 🏷️**

```python
cluster_scores = df_fin_clustered.groupBy("prediction") \
    .avg("score_estimado") \
    .orderBy("avg(score_estimado)", ascending=False) \
    .collect()
```

---

## **Criar mapa de rótulos: Alto, Médio, Baixo 🗺️**

```python
cluster_map = {} 
for i, row in enumerate(cluster_scores): 
    cluster_map[row["prediction"]] = ["Alto", "Medio", "Bajo"][i] 
```

---

## **UDF para atribuir rótulo ⚡**

```python
def map_cluster(pred): 
    return cluster_map.get(pred, "Desconocido") 

map_udf = udf(map_cluster, StringType()) 
df_segmentado = df_fin_clustered.withColumn("perfil_crediticio", map_udf(col("prediction"))) 
```

---

## **Contar clientes por perfil (opcional para validação) 📊**

```python
df_segmentado.groupBy("perfil_crediticio").count().orderBy("count", ascending=False).show() 
```

---

## **Filtrar clientes por perfil 🥇**
**NOTA**: Esta etapa pode ser ajustada caso se deseje analisar todos os tiers ou apenas os de perfil alto (valioso)

```python
#df_gold_fin = df_segmentado.filter(col("perfil_crediticio") == "Alto") 
#display(df_gold_fin)
df_gold_fin = df_segmentado
```

---

## **Remover colunas desnecessárias de ML**

```python
drop_columns = ["features", "prediction"]
df_gold_fin = df_gold_fin.drop(*drop_columns)
```

---

## **Salvar tabela Gold com clientes de melhor perfil de crédito 💾**

```python
df_gold_fin.write.option("mergeSchema", "true").mode("overwrite").saveAsTable("Contoso_Lakehouse.gold.credit_score") 

```

---

### **Conjunto de Retail por Produto 🧩**  
(Este é um exemplo de como trabalhar o cenário de score; pode ser feito com outras abordagens)

- Criar novo Dataflow Gen2 ou Notebook para a camada Silver
- Configurar a origem nas tabelas Bronze.
- Aplicar transformações intermediárias 
- Criar colunas derivadas
- Segmentar clientes por perfil de crédito 
- Salvar no Lakehouse como uma tabela silver
- Criar um novo Notebook para a camada Gold de score de crédito
- Configurar a origem nas tabelas Silver.
- Identificar o cluster com maior média de score 
- Filtrar os clientes pertencentes a esse cluster
- Armazenar os dados na camada Gold
- Resultado: subconjunto de clientes com perfil de crédito alto 


# **CAMADA SILVER - RETAIL**

# **Exemplo - Criação de tabelas silver - Retail 🧮**

---

## **📌 Importar funções necessárias**

```python
from pyspark.sql.functions import col, when, udf 
from pyspark.sql.types import StringType 

```

---

## **Carregar tabela Silver com catálogo de produtos de retail 🧾**

```python
df_retail = spark.read.table("productos_silver") 
```

---

## **🧮 Derivar coluna valor_comercial = Price × Stock**

```python
df_retail = df_retail.withColumn("valor_comercial", col("Price") * col("Stock")) 
```

---

## **🧮 Derivar coluna disponibilidade_binaria**

```python
df_retail = df_retail.withColumn("disponible", 
    when(col("Availability") == "in_stock", 1).otherwise(0)
)
```

---

## **🧹 Filtrar registros válidos para clustering**

```python
df_retail_clean = df_retail.filter( 
    col("valor_comercial").isNotNull() & col("disponible").isNotNull()
)
```

---

## **Padronizar colunas para lower case**

```python
df_fin_clean = df_fin_clean .toDF(*[c.lower() for c in df_fin_clean.columns])
```
---

## **💾 Salvar tabela silver preparada**

```python
df_retail_clean.write.option("overwriteSchema", "true").mode("overwrite").saveAsTable("Contoso_Lakehouse.silver.products") 

```

---



# **CAMADA GOLD - RETAIL**

# **Exemplo - Segmentação de produtos 🧮**

---

## **📌 Importar funções necessárias**

```python
from pyspark.sql.functions import col, when, udf 
from pyspark.sql.types import StringType 
from pyspark.ml.feature import VectorAssembler 
from pyspark.ml.clustering import KMeans 
```
---

## **📊 Vetorizar colunas para ML**

```python
assembler = VectorAssembler(inputCols=["valor_comercial", "disponible"], outputCol="features") 
df_retail_vec = assembler.transform(df_retail_clean) 
```

---

## **🤖 Aplicar clustering KMeans para segmentar produtos**

```python
kmeans = KMeans(k=3, seed=42) 
model_retail = kmeans.fit(df_retail_vec) 
df_retail_clustered = model_retail.transform(df_retail_vec) 
```

---

## **🏷️ Rotular produtos conforme o valor comercial médio por cluster**

```python
cluster_scores = df_retail_clustered.groupBy("prediction") \
    .avg("valor_comercial") \
    .orderBy("avg(valor_comercial)", ascending=False) \
    .collect()
```

---

## **Criar mapa de rótulos: Valioso, Médio, Baixo 🗺️**

```python
cluster_map = {} 
for i, row in enumerate(cluster_scores): 
    cluster_map[row["prediction"]] = ["Valioso", "Medio", "Bajo"][i] 
```

---

## **UDF para atribuir rótulo ⚡**

```python
def map_cluster(pred): 
    return cluster_map.get(pred, "Desconocido") 

map_udf = udf(map_cluster, StringType()) 
df_segmentado = df_retail_clustered.withColumn("perfil_producto", map_udf(col("prediction"))) 
```

---

## **🔍 Contagem por perfil (opcional para validação)**

```python
df_segmentado.groupBy("perfil_producto").count().orderBy("count", ascending=False).show() 
```

---

## **🥇 Filtrar produtos disponíveis**
**NOTA**: Esta etapa pode ser ajustada caso se deseje analisar todos os tiers ou apenas os produtos de perfil alto (valioso)

```python
#df_gold_retail = df_segmentado.filter(
#    (col("perfil_producto") == "Valioso") & (col("disponible") == 1)
#)
#display(df_gold_retail)
df_gold_retail = df_segmentado.filter(
    (col("disponible") == 1))
```

---

## **Remover colunas desnecessárias de ML**

```python
drop_columns = ["features", "prediction"]
df_gold_retail = df_gold_retail.drop(*drop_columns)

```

## **💾 Salvar tabela Gold com produtos valiosos**

```python
df_gold_retail.write.option("overwriteSchema", "true").mode("overwrite").saveAsTable("Contoso_Lakehouse.gold.products")
```

---


# **Preparação + Promoção para Gold (Business Operations - Transações) 🛍️🤖**


## **Solução passo a passo 🪜**

### **Conjunto de transações 🧩**  
(Este é um exemplo de como trabalhar o cenário de transactions; pode ser feito com outras abordagens)

- Criar novo Dataflow Gen2 ou Notebook para a camada Silver
- Configurar a origem nas tabelas Bronze.
- Aplicar transformações intermediárias 
- Extrações temporais básicas
- Segmentação simples de tickets
- Flags de financiamento
- Salvar no Lakehouse como uma tabela silver
- Criar um novo Notebook para a camada Gold de score de crédito
- Configurar a origem nas tabelas Silver.
- Identificar o cluster com maior média de score 
- Filtrar os clientes pertencentes a esse cluster
- Armazenar os dados na camada Gold
- Resultado: subconjunto de clientes com perfil de crédito alto 

---


# **CAMADA SILVER - TRANSACTIONS**

# **Exemplo - Criação de tabelas silver - Transactions 🧮**

---

## **📌 Importar funções necessárias**

```python
from pyspark.sql.functions import *
```
---

## **Ler Bronze**

```python
df_transactions = spark.read.table("Contoso_Lakehouse.bronze.transactions")

print(f"💳 Transactions en Bronze: {df_transactions.count():,}")
print("\n📋 Schema:")
df_transactions.printSchema()
```
---

## **Transformações Silver**

```python
df_transactions_enriched = (
    df_transactions
    
    # ============================================
    # FEATURES TEMPORALES (solo lo esencial)
    # ============================================
    .withColumn("year", year("transaction_date"))
    .withColumn("month", month("transaction_date"))
    .withColumn("quarter", quarter("transaction_date"))
    .withColumn("year_month", date_format("transaction_date", "yyyy-MM"))
    
    # ============================================
    # SEGMENTACIÓN SIMPLE
    # ============================================
    .withColumn("ticket_segment", 
        when(col("amount") < 500, "Low")
        .when(col("amount") < 1000, "Medium")
        .otherwise("High")
    )
    
    # ============================================
    # FLAGS DE FINANCIAMIENTO
    # ============================================
    .withColumn("is_msi", col("installments") > 1)
    .withColumn("is_credit", col("payment_method") == "Credit")
    
    # Solo transacciones aprobadas
    .filter(col("approval_status") == "Approved")
    
    # ============================================
    # SELECCIÓN FINAL (campos esenciales)
    # ============================================
    .select(
        # IDs
        "transaction_id",
        "customer_id",
        "product_id",
        
        # Transacción
        "transaction_date",
        "amount",
        "quantity",
        "payment_method",
        "installments",
        "channel",
        "store_location",
        
        # Tiempo
        "year",
        "month",
        "quarter",
        "year_month",
        
        # Segmentos
        "ticket_segment",
        "is_msi",
        "is_credit"
    )
)
```
---

## **Validações de Qualidade**

```python
total_transactions = df_transactions_enriched.count()
print(f"✅ Total de transações na Silver: {total_transactions:,}")

unique_customers = df_transactions_enriched.select("customer_id").distinct().count()
print(f"👥 Clientes únicos: {unique_customers:,}")

print("\n💰 Distribuição por Ticket Segment:")
df_transactions_enriched.groupBy("ticket_segment").count().orderBy("ticket_segment").show()

```
---

## **💾 Salvar tabela silver preparada**

```python
df_transactions_enriched.write \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("Contoso_Lakehouse.silver.transactions")

print(f"✅ Silver table created: silver.transactions")
print(f"   Records: {total_transactions:,}")
print(f"   Customers: {unique_customers:,}")
```

---

# **CAMADA GOLD - BUSINESS OPS**
Este notebook cria uma tabela Gold que combina:
- Transações (silver.transactions)
- Produtos (silver.products)

**Output**: `gold.business_operations` - Fact table para análise operacional


# **Exemplo - Business Operations - Combinando transações + produtos 🧮**

```python
from pyspark.sql.functions import *
```

---

## **Ler dados da Silver**

```python
# Transações enriquecidas
df_transactions = spark.read.table("Contoso_Lakehouse.silver.transactions")

# Produtos enriquecidos
df_products = spark.read.table("Contoso_Lakehouse.silver.products")

print(f"Transactions: {df_transactions.count():,} registros")
print(f"Products: {df_products.count():,} registros")
```

---

## **Criar tabela Gold**

```python

df_business_operations = (
    df_transactions
    
    # JOIN com produtos
    .join(
        df_products.select(
            "product_id",
            "product_name",
            "brand", 
            "category",
            "price",
        ),
        "product_id",
        "left"
    )
    
    # Seleção final (somente campos necessários)
    .select(
        # IDs
        "transaction_id",
        "customer_id",
        "product_id",
        
        # Produto
        "product_name",
        "brand", 
        "category",
        "price",
        
        
        # Transação
        "transaction_date",
        "amount",
        "quantity",
        "payment_method",
        "installments",
        "channel",
        "store_location",
        
        # Tempo
        "year",
        "month",
        "quarter",
        "year_month",
        
        # Segmentos
        "ticket_segment",
        "is_msi",
        "is_credit"
    )
)

```

---

## **Validações**

```python
total_records = df_business_operations.count()
print(f"✅ Total de registros na Gold: {total_records:,}")

# Verificar nulls críticos
null_checks = df_business_operations.select([
    count(when(col(c).isNull(), c)).alias(c) 
    for c in ["customer_id", "product_id", "amount", "transaction_date"]
])
print("\n📊 Nulls em campos críticos:")
null_checks.show()
```

---

## **Salvar tabela Gold**

```python

df_business_operations.write \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("Contoso_Lakehouse.gold.business_operations")

print(f"✅ Tabela gold.business_operations criada com sucesso")
print(f"   Registros: {total_records:,}")

```

---

## **Queries de Validação**

```python
# Top 10 produtos por revenue
print("🏆 Top 10 Produtos por Revenue:")
df_business_operations.groupBy("product_name", "category") \
    .agg(
        count("*").alias("units_sold"),
        sum("amount").alias("total_revenue")
    ) \
    .orderBy(col("total_revenue").desc()) \
    .limit(10) \
    .show(truncate=False)

# Revenue por canal
print("\n📺 Revenue por Canal:")
df_business_operations.groupBy("channel") \
    .agg(
        count("*").alias("transactions"),
        sum("amount").alias("revenue"),
        avg("amount").alias("avg_ticket")
    ) \
    .orderBy(col("revenue").desc()) \
    .show()
```

---
