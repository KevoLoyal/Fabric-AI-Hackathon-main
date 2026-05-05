# Solución Reto 01 - Ingesta desde Cosmos DB a Microsoft Fabric (Capa Bronze) + Limpieza Básica

Guía paso a paso para ingerir datos desde Azure Cosmos DB hacia la capa Bronze de la Lakehouse en Microsoft Fabric, aplicar limpieza inicial y validar resultado.

Objetivo
- Ingerir datos desde Cosmos DB usando Dataflow Gen2 o pipeline y aplicar transformaciones básicas (nulos, columnas, formatos).

Requisitos previos
- Conexión a Cosmos DB desde Fabric (ver `00-Solution.md`).

## Pasos

### 1 - Crear Dataflow Gen2

1. En Fabric, Data → New → Dataflow Gen2.
2. Selecciona **Azure Cosmos DB** como fuente.
3. Rellena endpoint y key (o selecciona la conexión ya creada).
4. Selecciona la colección/contenedor que contiene `products` , `creditScore`, `transactions`.

### 2 - Diseñar transformaciones básicas

Dentro del diseñador de Dataflow:
- Elimina columnas no necesarias.  
- Normaliza formatos: convertir fechas, cadenas (trim, lower), normalizar decimales, cambia tipos de datos.  
- Reemplaza o marca valores nulos (por ejemplo, `unknown` o valores por defecto).  
- Filtra registros corruptos o incompletos (si aplica).  

![DF](/img/dfgen2.png)

Consejo: agrega pasos de validación intermedios y usa muestras pequeñas para probar transformaciones.

### 3 - Destino: Lakehouse Bronze

1. Configura el sink/destino como la Lakehouse `Contoso_Lakehouse` → buscar esquema → crear tabla. Ejemplo `[bronze].[sales]`. Recuerda activar la opcion de **advance options** : `Navigate using full hierarchy` que permite navegar sobre la estructura jerarquica de `schemas/tablas`.  Repetir con los otros datasets
2. Ejecuta el Dataflow en modo de validacion y luego en producción.

 ![DF](/img/dfgen22.png)
 ![DF](/img/dfgen23.png)  
 ![DF](/img/dfgen24.png)  
 ![DF](/img/dfgen25.png)  
 ![DF](/img/dfgen26.png)  
 
### 4 - Verificar y documentar

1. Abre la Lakehouse y revisa cada una de las tablas. Ejemplo `[bronze].[sales]`.
2. Verifica conteo de registros, columnas esperadas y formatos de columna (fechas, numéricos).
3. Guarda un registro de la ejecución (logs) y captura de pantalla.

 ![Tablas bronze](/img/tablas_bronze.png)

 ---

## Validaciones clave
- Validacion origen vs destino
- No hay columnas que deban eliminarse accidentalmente.
- Nulos y errores manejados y formatos normalizados.

## Siguientes pasos sugeridos (opcionales)
- Automatizar con un pipeline (schedule) para ingestas periódicas.
- Implementar pruebas de calidad de datos (Data Quality checks) antes de mover a Silver.

