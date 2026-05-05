# 📊 Datasets - Hackathon México

Esta carpeta contiene los datos necesarios para el Hackathon de IA y Fabric.

---

## 🚀 CARGAR DATOS A AZURE

### 1️⃣ Abre Azure Cloud Shell (haz clic en el botón)

[![Abrir Cloud Shell](https://learn.microsoft.com/en-us/azure/cloud-shell/media/embed-cloud-shell/launch-cloud-shell-1.png)](https://shell.azure.com/powershell)

---

### 2️⃣ Copia y pega este comando en Cloud Shell

```powershell
git clone https://github.com/DCSA-HackLabs/Hackathon-Mexico.git; cd Hackathon-Mexico; ./run-data-loader.ps1
```

<details>
<summary>📋 Click aquí para copiar el comando</summary>

```
git clone https://github.com/DCSA-HackLabs/Hackathon-Mexico.git; cd Hackathon-Mexico; ./run-data-loader.ps1
```

</details>

---

### 3️⃣ Ingresa tu Resource Group cuando te lo pida

```
📌 Ingresa el nombre de tu Resource Group: rg-fabric-challenge-TUEQUIPO
```

---

## ✅ ¡Listo!

El script cargará automáticamente:

| Archivo | Destino |
|---------|---------|
| 📄 `transactions.json` | **Cosmos DB** → FabricChallengeDB → Transactions |
| 📁 `Financial Data.zip` (PDFs) | **Storage Account** → documents-pdf |

---

## 🔄 Diagrama del proceso

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   📂 Datasets/                                                  │
│   ├── transactions.json  ──────►  🗄️ Cosmos DB                 │
│   │                               └── FabricChallengeDB         │
│   │                                   └── Transactions          │
│   │                                                             │
│   └── Financial Data.zip ──────►  📦 Storage Account           │
│       └── *.pdf                   └── documents-pdf/            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ❓ Problemas comunes

| Error | Solución |
|-------|----------|
| "No se encontró Cosmos DB" | Verifica el nombre del Resource Group |
| "Resource Group no existe" | Primero despliega los recursos con el template ARM |
| Error de autenticación | Ejecuta `az login` |

---

## 📞 Soporte

¿Problemas? Contacta al equipo de Microsoft durante el evento.
