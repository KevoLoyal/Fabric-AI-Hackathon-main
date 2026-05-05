# Solución Reto 00 - Preparación de la Landing Zone y Conexión de Datos (Microsoft Fabric)

Este documento describe, paso a paso, cómo completar el *Reto 0*: crear la zona de aterrizaje (landing zone) en Microsoft Fabric, provisionar Azure Cosmos DB con los JSON suministrados y conectar ambas plataformas.

Objetivos
- Provisionar Azure Cosmos DB (NoSQL) y cargar los datasets JSON.
- Crear un workspace y una Lakehouse en Microsoft Fabric con la estructura Bronze / Silver / Gold.
- Conectar Fabric a Cosmos DB y verificar la ingesta inicial.

Requisitos previos
- Permisos para crear recursos en Azure.
- Archivos JSON (por ejemplo `creditScore.json`, `products.json`, `transactions.json`).

Resultado esperado
- Cosmos DB con contenedores que contienen los JSON.
- Workspace en Fabric conectado a una Capacity.
- Lakehouse `Contoso_Lakehouse` con schemas: `bronze`, `silver`, `gold`.

---

## 1 - Crear Azure Cosmos DB (NoSQL)

**NOTA**  

Por conveniencia se recomienda utilizar el template de despliegue de artefactos de Azure [DeployToAzure](./DeployToAzure.md) el cual permite crear la Cosmos DB NoSQL y otros artefactos de Azure de manera automatizada. En caso de utilizar el template omitiremos este primer paso, caso contrario seguiremos el paso 1 de manera manual.

1. Accede al portal de Azure.
2. En el buscador de Azure escribe **Azure Cosmos DB** → +Crear → seleccionar API **Azure Cosmos DB for NoSQL** → Crear.
3. Rellena:
   - Workload Type: `Development/Testing`
   - Subscription: `tu subscripción de Azure`
   - Resource Group: `tu grupo de recursos`
   - Account name: `nombre de la cuenta a crear`
   - Location:  `Tu región default`
   - Availability Zones: `Disable`
   - Capacity mode: `Serverless`
   - Global Distribution: `Disable`
   - Networking: `All networks`
   - Backup Policy: `Dejar valor por defecto`
   - Key-based authentication: `Enable`
   - Encryption: `Service-managed key`
   - Tags: `Vacios`

Review + create
   
![New Foundry](/img/cosmos1.png)

4. Una vez creada la cuenta de Cosmos DB. En *Data Explorer* crea la base de datos con la opcion +New → New Database → Database id → `<nombre>` por ejemplo DB-Hackathon

![New Foundry](/img/cosmos2.png)  	
  
5.Crea un contenedor para cada set de datos (por ejemplo `products`, `credit`, `transactions`). Sobre el mismo botón de +New seleccionamos → New container → seleccionamos:
- Database id: `Use existing` y seleccionamos la base que acabamos de crear
- Container id: `products`
- Partition key: `/id`
- OK

**Repetimos este proceso para los otros set de datos de `credit` y `transactions`**
   
![New Foundry](/img/cosmos3.png)  	
   
6. Carga los archivos JSON (manualmente desde Data Explorer o usando Azure Data Factory para cargas repetibles). En Data Explorer repitiendo este mismo proceso para cada contenedor creado seleccionamos la opción → `Item` → `Upload Item` → seleccionamos el archivo JSON correspondiente para cada contenedor  → `Upload`

![New Foundry](/img/cosmos4.png)

7. Verificación
- En *Data Explorer* se ven documentos JSON en el contenedor.
  
![New Foundry](/img/cosmos5.png)

---

## 2 - Crear Workspace en Microsoft Fabric

1. Abre Microsoft Fabric → en la barra lateral izquierda selecciona `Workspaces` → `+New workspace` y crea un workspace llamado `ContosoData-Fabric` o el nombre que prefieras.
2. Una vez creado, busca la opción `Workspace settings`→ Workspace type` → `Edit` y asocia el nuevo Workspace al Capacity de Fabric (este paso no es necesario si tu organización ya tiene un Workspace asignado a un capacity que puedas usar).
3. Agrega tu usuario como Admin/Contributor al Workspace

Verificación
- El workspace aparece en el listado de espacios de trabajo y tienes permisos adecuados.

![Fabric](/img/wscreate.png)

---

## 3 - Crear Lakehouse y definir capas

1. En el workspace, crea una Lakehouse `Contoso_Lakehouse` y marca el `check` para utilizar schemas.
2. Una vez este nuestro Lakehouse, crea nuevos schemas con la siguiente convención:
	- `bronze*` para datos crudos
	- `silver*` para datos limpios
	- `gold*` para datos curados/consumo

Verificación
- Los schemas están visibles en el navegador del Lakehouse.

![Schemas](/img/schemas.png)



---

## 4 - Conectar Fabric a Azure Cosmos DB

1.  Dentro del Workspace creado selecciona `+New item` → Dataflow Gen2
2.  En el nuevo DataFlow gen 2 → Get data → More → buscas Azure Cosmos DBv2.
    
![Cosmos](/img/cosmoscon.png)

3.  En la config de conexion vamos a agregar:
   - Cosmos DB Endpoint: Vamos a colocar el endpoint de la cuenta de Cosmos DB, se puede obtener en el recurso de Cosmos DB en la opción Settings → Keys
   - Connection name: El nombre de nuestra conexión
   - Authentication kind: Account key
   - Acount key: El PRIMARY KEY de la cuenta de Cosmos DB. Se puede obtener en el recurso de Cosmos DB en la opción `Settings` → `Keys` → `PRIMARY KEY` 

![Cosmos](/img/cosmoscon1.png)

4. Testea y guarda la conexión.

Verificación
- La conexión se prueba y puedes ver containers/collections disponibles.

---




