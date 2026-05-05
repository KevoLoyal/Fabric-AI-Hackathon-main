# 🏆 Reto 0: Configuración de la Zona de Aterrizaje y Preparación de Datos en Microsoft Fabric  
📖 Escenario  
Contoso Retail ha cargado trest conjuntos de datos en formato **JSON**:  
- Uno **financiero**, con información de **score crediticio** de clientes.  
- Dos de **retail**, con datos de **transacciones y productos**.  

Tu misión es **preparar el entorno de trabajo en Microsoft Fabric**, conectando los datos almacenados en **Azure Cosmos DB** y estableciendo una **zona de aterrizaje (landing zone)** estructurada en capas para iniciar el proceso de transformación.  

---

### 🎯 Tu Misión  
Al completar este reto podrás:  

✅ Crear Cosmos DB no SQL y subir los archivos a los contenedores.
✅ Configurar un **workspace** en Microsoft Fabric para la gestión de datos.  
✅ Conectar **Azure Cosmos DB** como fuente de datos.  
✅ Explorar y comprender la estructura de los archivos JSON financieros y de retail.  
✅ Crear una **Lakehouse** con estructura por capas (**Bronze**, **Silver**, **Gold**).  
✅ Definir y documentar el flujo de datos entre las capas.  

---
## 🚀 Paso 1: Crear Azure Cosmos DB para no SQL 
💡 *¿Por qué?* Cosmos DB nos servira como la fuente de los datos que seran ingestados desde Fabric 

1️⃣ Ingresa al portal de **Microsoft Azure** y crea una base de datos Cosmos DB para no SQL. Para mayor conveniencia recomendamos ejecutar el template de despliegue que permitira acelerar este proceso de creacion de artefactos de Azure como se muestra en el archivo [DeployToAzure](./DeployToAzure.md)  
🔹 Asigna un nombre descriptivo (por ejemplo, `ContosoData-Source`).  
🔹 Crea el contenedor y asigna un nombre identificable.  
🔹 Sube el dataset en formato JSON

✅ **Resultado esperado:** Tienes un Cosmos DB con un contenedor con la informacion lista para ser ingestada desde Fabric.

## 🚀 Paso 2: Crear un Workspace en Microsoft Fabric  
💡 *¿Por qué?* El workspace es el entorno centralizado donde se gestionan datasets, dataflows, pipelines y notebooks.  

1️⃣ Ingresa a **Microsoft Fabric** y crea un nuevo workspace para el proyecto de Contoso.  
🔹 Asigna un nombre descriptivo (por ejemplo, `ContosoData-Fabric`).  
🔹 Asegúrate de que esté asignado a una **Fabric Capacity** (si ya la tienes configurada, puedes omitir este paso).  

✅ **Resultado esperado:** Tienes un workspace dedicado para todos los recursos de Fabric.  

---

## 🚀 Paso 3: Conectar con Azure Cosmos DB  
💡 *¿Por qué?* Establecer esta conexión permite que Fabric acceda e ingiera directamente los datos JSON desde Cosmos DB.  

1️⃣ En tu workspace de Fabric, crea una nueva **conexión de datos** hacia **Azure Cosmos DB**.  
🔹 Proporciona el **endpoint** y la **clave de acceso** correctos.  
🔹 Verifica que los permisos estén configurados adecuadamente.  

✅ **Resultado esperado:** Tu workspace está conectado a Cosmos DB y listo para la ingesta de datos.  

---

## 🚀 Paso 4: Crear una Lakehouse y Definir la Estructura de Capas  
💡 *¿Por qué?* La Lakehouse es la base de la arquitectura de datos y permite separar las etapas de procesamiento.  

1️⃣ En Fabric, crea una **Lakehouse** (activa el soporte para schemas) llamada `Contoso_Lakehouse`.  
2️⃣ Dentro de la Lakehouse, define la siguiente estructura de schemas:  
   - 🥉 **Bronze:** Datos crudos y sin procesar, ingeridos directamente desde Cosmos DB.  
   - 🥈 **Silver:** Datos limpios, normalizados y consistentes.  
   - 🥇 **Gold:** Datos curados y listos para análisis o visualizaciones.  

✅ **Mejor práctica:** Mantén una convención clara de nombres para schemas y tablas que facilite el seguimiento del flujo de datos.  

✅ **Resultado esperado:** Tu Lakehouse cuenta con una base estructurada que soportará las transformaciones y el análisis de datos.  

---

## 🏁 Puntos de Control Finales  

✅ ¿Se creó correctamente el Cosmos DB y los contenedores?  
✅ ¿Se creó correctamente el workspace en Microsoft Fabric y está conectado a Cosmos DB?  
✅ ¿Se validó la estructura de los datasets JSON en Cosmos?  
✅ ¿Está la Lakehouse organizada con las capas Bronze, Silver y Gold?  
✅ ¿Se documentó la estrategia de flujo de datos entre las capas?  

---

💡 **Próximos Pasos:**  
Con la **zona de aterrizaje configurada**, estás listo para avanzar al siguiente reto, donde comenzarás con la **ingesta, limpieza y transformación de datos** dentro de Fabric. 🚀  

---

**📄 Documentacion**
- [Creacion Cosmos DB](https://learn.microsoft.com/es-es/azure/cosmos-db/nosql/quickstart-portal)
- [Permitir IP publica en Firewall](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops&tabs=IP-V4)
- [Creacion Fabric workspace](https://learn.microsoft.com/es-es/fabric/data-warehouse/tutorial-create-workspace)
- [Creacion Fabric lakehouse](https://learn.microsoft.com/es-es/fabric/data-engineering/tutorial-build-lakehouse)
- [Crear Pipeline](https://learn.microsoft.com/es-mx/fabric/data-factory/create-first-pipeline-with-sample-data)




