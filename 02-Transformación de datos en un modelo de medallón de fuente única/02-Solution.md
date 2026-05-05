# **Reto 2 – Transformación intermedia, análisis exploratorio (Silver) y Preparacion Gold 🔧📊**

## **Objetivo y solución paso a paso 🧭**

### **Objetivo 🎯**
Transformar los datos Bronze, realizar análisis exploratorio en Silver y finalmente construir la capa Gold. 

---

## **Solución paso a paso 🪜**

En este approcah se decidio trabajar tres productos de datos **Credit Scores** medallion + segmentacion de clientes con ML (KMeans), **Products** -medallion + segmentacion de productos por valor comercial con ML (KMeans), y **Business Operations** que combina tablas de transactions y products para modelar un analisis operacional de compras de clientes y productos. El ejercicio no pretende ejercer un modelado dimensional estricto y mas bien se invita a trabajar distintos enfoques, Fabric como herramienta se adapta a enfoqaues de modelado dimensional asi como enfoques de denormalizacion comunes en entornos de Big Data.

A continuacion la estructura utilizada:

![Enfoque](/img/approach.png)




### **Set de score crediticio🧩**  
(Esto es un ejemplo de como trabajar el escenario financiero de credit, se puede hacer con otros enfoques)

- Crear nuevo Dataflow Gen2 o Notebook para la capa Silver
- Configurar el origen en las tablas Bronze.
- Aplicar transformaciones intermedias 
- Crear columna de score crediticio, derivadas
- Segmentar clientes por perfil crediticio 
- Guardar en el Lakehouse como tabla una tabla silver
- Se crea un nuevo Notebook para la capa Gold de score crediticio
- Configurar el origen en las tablas Silver.
- Se identifica el cluster con mayor promedio de score 
- Se filtran los clientes pertenecientes a ese cluster
- Se almacenan los datos en la capa Gold
- Resultado: subconjunto de clientes con perfil crediticio alto 

---

# **CAPA SILVER - SCORE CREDITICIO**

# **Ejemplo -Creacion tablas silver - Score Crediticio  🧮**


## **Cargar tabla bronze con datos financieros 🧾**

```python

df_fin = spark.sql("SELECT * FROM Contoso_Lakehouse.bronze.credit_score") 
```

---

## **Derivar columna score_estimado basada en comportamiento de pago y uso de crédito 💳**

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

## **Penalización por pagos atrasados ⏰**

```python
df_fin = df_fin.withColumn("score_estimado", 
    col("score_estimado") - (col("Num_of_Delayed_Payment") * 5)
)
```

---

## **Penalización por alto uso de crédito 📉**

```python
df_fin = df_fin.withColumn("score_estimado", 
    when(col("Credit_Utilization_Ratio") > 0.8, col("score_estimado") - 20)
    .otherwise(col("score_estimado"))
)
```

---

## **Limitar score entre 300 y 850 ⚙️**

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

## **Homologar columnas a lower case**

```python
df_fin_clean = df_fin_clean .toDF(*[c.lower() for c in df_fin_clean.columns])
```
---
## **Filtrando clientes sin nulos/na + edades y cantidad de prestamos correctos**
```python
df_fin_clean = df_fin_clean.dropna().filter(
col("age").between(1, 120) & 
(col("num_of_loan") >= 0) &
(col("credit_history_age") != "NA"))
```
---

## **💾 Guardar tabla silver preparada**

```python
df_fin_clean.write.option("overwriteSchema", "true").mode("overwrite").saveAsTable("Contoso_Lakehouse.silver.credit_score") 

```

---


# **CAPA GOLD - SCORE CREDITICIO**

# **Ejemplo -Segmentacion de clientes por score crediticio 🧮**


## **📌 Importar funciones necesarias**

```python
from pyspark.sql.functions import col, when, udf 
from pyspark.sql.types import StringType 
from pyspark.ml.feature import VectorAssembler 
from pyspark.ml.clustering import KMeans 
```


## **Cargar tabla Silver con datos financieros preparados 🧾**

```python
df_fin = spark.sql("SELECT * FROM Contoso_Lakehouse.silver.credit_score")

```

---

## **Vectorizar columna score_estimado para ML 🤖**

```python
assembler = VectorAssembler(inputCols=["score_estimado"], outputCol="features") 
df_fin_vec = assembler.transform(df_fin_clean) 
```

---

## **Aplicar KMeans clustering para segmentar clientes 🧠**

```python
kmeans = KMeans(k=3, seed=42) 
model_fin = kmeans.fit(df_fin_vec) 
df_fin_clustered = model_fin.transform(df_fin_vec) 
```

---

## **Etiquetar perfiles crediticios según promedio de score por cluster 🏷️**

```python
cluster_scores = df_fin_clustered.groupBy("prediction") \
    .avg("score_estimado") \
    .orderBy("avg(score_estimado)", ascending=False) \
    .collect()
```

---

## **Crear mapa de etiquetas: Alto, Medio, Bajo 🗺️**

```python
cluster_map = {} 
for i, row in enumerate(cluster_scores): 
    cluster_map[row["prediction"]] = ["Alto", "Medio", "Bajo"][i] 
```

---

## **UDF para asignar etiqueta ⚡**

```python
def map_cluster(pred): 
    return cluster_map.get(pred, "Desconocido") 

map_udf = udf(map_cluster, StringType()) 
df_segmentado = df_fin_clustered.withColumn("perfil_crediticio", map_udf(col("prediction"))) 
```

---

## **Contar clientes por perfil (opcional para validación) 📊**

```python
df_segmentado.groupBy("perfil_crediticio").count().orderBy("count", ascending=False).show() 
```

---

## **Filtrar clientes por perfil🥇**
**NOTA**: Este paso puede ajustarse si se desea analizar todos los tiers o solo los de perfil alto (valioso)

```python
#df_gold_fin = df_segmentado.filter(col("perfil_crediticio") == "Alto") 
#display(df_gold_fin)
df_gold_fin = df_segmentado
```

---

## **Eliminar columnas innecesarias de ML**

```python
drop_columns = ["features", "prediction"]
df_gold_fin = df_gold_fin.drop(*drop_columns)
```

---

## **Guardar tabla Gold con clientes de mejor perfil crediticio 💾**

```python
df_gold_fin.write.option("mergeSchema", "true").mode("overwrite").saveAsTable("Contoso_Lakehouse.gold.credit_score") 

```

---

### **Set de Retail por Producto🧩**  
(Esto es un ejemplo de como trabajar el escenario de score, se puede hacer con otros enfoques)

- Crear nuevo Dataflow Gen2 o Notebook para la capa Silver
- Configurar el origen en las tablas Bronze.
- Aplicar transformaciones intermedias 
- Crear columnas derivadas
- Segmentar clientes por perfil crediticio 
- Guardar en el Lakehouse como tabla una tabla silver
- Se crea un nuevo Notebook para la capa Gold de score crediticio
- Configurar el origen en las tablas Silver.
- Se identifica el cluster con mayor promedio de score 
- Se filtran los clientes pertenecientes a ese cluster
- Se almacenan los datos en la capa Gold
- Resultado: subconjunto de clientes con perfil crediticio alto 


# **CAPA SILVER - RETAIL**

# **Ejemplo -Creacion tablas silver - Retail  🧮**

---

## **📌 Importar funciones necesarias**

```python
from pyspark.sql.functions import col, when, udf 
from pyspark.sql.types import StringType 

```

---

## **Cargar tabla Silver con catálogo de productos retail 🧾**

```python
df_retail = spark.read.table("productos_silver") 
```

---

## **🧮 Derivar columna valor_comercial = Price × Stock**

```python
df_retail = df_retail.withColumn("valor_comercial", col("Price") * col("Stock")) 
```

---

## **🧮 Derivar columna disponibilidad_binaria**

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

## **Homologar columnas a lower case**

```python
df_fin_clean = df_fin_clean .toDF(*[c.lower() for c in df_fin_clean.columns])
```
---

## **💾 Guardar tabla silver preparada**

```python
df_retail_clean.write.option("overwriteSchema", "true").mode("overwrite").saveAsTable("Contoso_Lakehouse.silver.products") 

```

---



# **CAPA GOLD - RETAIL**

# **Ejemplo -Segmentacion de productos🧮**

---

## **📌 Importar funciones necesarias**

```python
from pyspark.sql.functions import col, when, udf 
from pyspark.sql.types import StringType 
from pyspark.ml.feature import VectorAssembler 
from pyspark.ml.clustering import KMeans 
```
---

## **📊 Vectorizar columnas para ML**

```python
assembler = VectorAssembler(inputCols=["valor_comercial", "disponible"], outputCol="features") 
df_retail_vec = assembler.transform(df_retail_clean) 
```

---

## **🤖 Aplicar KMeans clustering para segmentar productos**

```python
kmeans = KMeans(k=3, seed=42) 
model_retail = kmeans.fit(df_retail_vec) 
df_retail_clustered = model_retail.transform(df_retail_vec) 
```

---

## **🏷️ Etiquetar productos según valor comercial promedio por cluster**

```python
cluster_scores = df_retail_clustered.groupBy("prediction") \
    .avg("valor_comercial") \
    .orderBy("avg(valor_comercial)", ascending=False) \
    .collect()
```

---

## **Crear mapa de etiquetas: Valioso, Medio, Bajo 🗺️**

```python
cluster_map = {} 
for i, row in enumerate(cluster_scores): 
    cluster_map[row["prediction"]] = ["Valioso", "Medio", "Bajo"][i] 
```

---

## **UDF para asignar etiqueta ⚡**

```python
def map_cluster(pred): 
    return cluster_map.get(pred, "Desconocido") 

map_udf = udf(map_cluster, StringType()) 
df_segmentado = df_retail_clustered.withColumn("perfil_producto", map_udf(col("prediction"))) 
```

---

## **🔍 Conteo por perfil (opcional para validación)**

```python
df_segmentado.groupBy("perfil_producto").count().orderBy("count", ascending=False).show() 
```

---

## **🥇 Filtrar productos disponibles**
**NOTA**: Este paso puede ajustarse si se desea analizar todos los tiers o solo los productos de perfil alto (valioso)

```python
#df_gold_retail = df_segmentado.filter(
#    (col("perfil_producto") == "Valioso") & (col("disponible") == 1)
#)
#display(df_gold_retail)
df_gold_retail = df_segmentado.filter(
    (col("disponible") == 1))
```

---

## **Eliminar columnas innecesarias de ML**

```python
drop_columns = ["features", "prediction"]
df_gold_retail = df_gold_retail.drop(*drop_columns)

```

## **💾 Guardar tabla Gold con productos valiosos**

```python
df_gold_retail.write.option("overwriteSchema", "true").mode("overwrite").saveAsTable("Contoso_Lakehouse.gold.products")
```

---


# **Preparacion + Promoción a Gold (Business Operations - Transacciones) 🛍️🤖**


## **Solución paso a paso 🪜**

### **Set de transacciones🧩**  
(Esto es un ejemplo de como trabajar el escenario de transactions, se puede hacer con otros enfoques)

- Crear nuevo Dataflow Gen2 o Notebook para la capa Silver
- Configurar el origen en las tablas Bronze.
- Aplicar transformaciones intermedias 
- Extracciones temporales basicas
- Segmentacion simple de tickets
- Flags de financiamento
- Guardar en el Lakehouse como tabla una tabla silver
- Se crea un nuevo Notebook para la capa Gold de score crediticio
- Configurar el origen en las tablas Silver.
- Se identifica el cluster con mayor promedio de score 
- Se filtran los clientes pertenecientes a ese cluster
- Se almacenan los datos en la capa Gold
- Resultado: subconjunto de clientes con perfil crediticio alto 

---


# **CAPA SILVER - TRANSACTIONS**

# **Ejemplo -Creacion tablas silver - Transactions  🧮**

---

## **📌 Importar funciones necesarias**

```python
from pyspark.sql.functions import *
```
---

## **Leer Bronze**

```python
df_transactions = spark.read.table("Contoso_Lakehouse.bronze.transactions")

print(f"💳 Transactions en Bronze: {df_transactions.count():,}")
print("\n📋 Schema:")
df_transactions.printSchema()
```
---

## **Transformaciones Silver**

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

## **Validaciones de Calidad**

```python
total_transactions = df_transactions_enriched.count()
print(f"✅ Total transacciones en Silver: {total_transactions:,}")

unique_customers = df_transactions_enriched.select("customer_id").distinct().count()
print(f"👥 Clientes únicos: {unique_customers:,}")

print("\n💰 Distribución por Ticket Segment:")
df_transactions_enriched.groupBy("ticket_segment").count().orderBy("ticket_segment").show()

```
---

## **💾 Guardar tabla silver preparada**

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

# **CAPA GOLD - BUSINESS OPS**
Este notebook crea una tabla Gold que combina:
- Transacciones (silver.transactions)
- Productos (silver.products)

**Output**: `gold.business_operations` - Fact table para análisis operacional


# **Ejemplo - Business Operations - Combinando transacciones + productos🧮**

```python
from pyspark.sql.functions import *
```

---

## **Leer datos de Silver**

```python
# Transacciones enriquecidas
df_transactions = spark.read.table("Contoso_Lakehouse.silver.transactions")

# Productos enriquecidos
df_products = spark.read.table("Contoso_Lakehouse.silver.products")

print(f"Transactions: {df_transactions.count():,} registros")
print(f"Products: {df_products.count():,} registros")
```

---

## **Crear tabla Gold**

```python

df_business_operations = (
    df_transactions
    
    # JOIN con productos
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
    
    # Selección final (solo campos necesarios)
    .select(
        # IDs
        "transaction_id",
        "customer_id",
        "product_id",
        
        # Producto
        "product_name",
        "brand", 
        "category",
        "price",
        
        
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

## **Validaciones**

```python
total_records = df_business_operations.count()
print(f"✅ Total registros en Gold: {total_records:,}")

# Verificar nulls críticos
null_checks = df_business_operations.select([
    count(when(col(c).isNull(), c)).alias(c) 
    for c in ["customer_id", "product_id", "amount", "transaction_date"]
])
print("\n📊 Nulls en campos críticos:")
null_checks.show()
```

---

## **Guardar tabla Gold**

```python

df_business_operations.write \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("Contoso_Lakehouse.gold.business_operations")

print(f"✅ Tabla gold.business_operations creada exitosamente")
print(f"   Registros: {total_records:,}")

```

---

## **Queries de Validación**

```python
# Top 10 productos por revenue
print("🏆 Top 10 Productos por Revenue:")
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


