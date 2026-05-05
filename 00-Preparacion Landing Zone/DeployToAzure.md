
# 🚀 Deploy Landing Zone for Hackathon

Este repositorio permite crear una **Landing Zone base** para preparar el ambiente de un hackathon de forma **rápida, consistente y repetible**.

El objetivo es que los participantes no pierdan tiempo creando infraestructura básica y puedan enfocarse directamente en **desarrollo, innovación y uso de datos**.

---

## 🟦 ¿Qué hace el botón **Deploy to Azure**?

Al hacer clic en el botón **Deploy to Azure**, se desplegará automáticamente en tu suscripción de Azure una **Landing Zone mínima**, que incluye:

- Un **Azure Cosmos DB** para almacenamiento de datos estructurados y semiestructurados.
- Un **Azure Storage Account** para blobs, archivos y otros artefactos del hackathon.
- Recursos organizados bajo un **Resource Group dedicado**.

Todo el despliegue se realiza usando **Infrastructure as Code (IaC)**, garantizando que todos los equipos parten del mismo punto.

---

## 🧱 Recursos que se despliegan

### 📌Fuentes de datos
  - Ideal para escenarios de:
    - Cosmos NoSQL
    - Storage Accounts


### 📦 Almacenamiento

  - Usado para:
    - Carga de archivos
    - Datasets del hackathon
  

---

## 🎯 ¿Para qué sirve esta Landing Zone?

Esta Landing Zone está pensada para:

- ✅ Preparar el ambiente base del hackathon en minutos  
- ✅ Evitar configuraciones manuales repetitivas  
- ✅ Asegurar consistencia entre todos los equipos  
- ✅ Reducir errores de configuración  
- ✅ Facilitar la evaluación de las soluciones  



## 🚀 Cómo usar el botón

1. Haz clic en el botón **Deploy to Azure**.
2. Inicia sesión con tu cuenta de Azure.
3. Selecciona la **suscripción** y el **Resource Group**.
4. Completa los parámetros solicitados (si aplica).
5. Confirma el despliegue.

En pocos minutos, el ambiente estará listo.

---


[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmakanto32%2FHackathon-Mexico%2Fmain%2F00-Preparacion%2520Landing%2520Zone%2Fazuredeploy.json
)
