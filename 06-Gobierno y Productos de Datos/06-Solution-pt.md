# ✅ Solução Desafio 6: Governança de Dados e Data Products (Purview + Fabric)

## 🎯 Objetivo
Implementar um framework de governança de dados usando o **Microsoft Purview Unified Catalog** para catalogar, classificar e publicar **Data Products** que agrupem ativos do **Microsoft Fabric** (Lakehouse) com políticas de acesso, documentação e qualidade de dados.

---

## 📋 Pré-requisitos

### **Acessos necessários:**
- Assinatura ativa do Azure
- Conta do Microsoft Purview (mesmo tenant do Fabric)
- Workspace do Fabric com Lakehouse criado nos exercícios anteriores
- Permissões:
  - **Fabric Admin** ou **Contributor** no workspace
  - **Data Governance Administrator** no Purview
  - Função **Data Product Owner** no Purview

### **Configurações prévias:**
- Lakehouse com dados de vendas e clientes (dos exercícios 1-5)
- Acesso de contributor no Fabric para o Managed Identity do Purview


---

## 🔧 PARTE 1: Configuração do Microsoft Purview

### **1.1 Criar conta do Purview (se não existir)**

**Portal UI**
1. Azure Portal → **Create Resource** → Buscar "Microsoft Purview"
2. Preencha:
   - **Subscription**: `<your-subscription`
   - **Resource Group**: `<your-resource_group>`
   - **Purview account name**: `contoso-retail-purview`
   - **Loacation**: East US 2 (preferencialmente para ter os workloads do Unified Catalog)
   - As demais opções deixamos no padrão
     
4. **Review + Create**

![Purview](/img/purview-account.png)

---

### **1.2 Acessar o Portal do Microsoft Purview**

1. Navegue até: **https://purview.microsoft.com**
2. Selecione sua conta do Purview: `contosoretail-purview`
3. Verifique se aparecem as soluções:
   - **Unified Catalog** (para Data Products)
   - **Data Map** (para escaneamentos)
   - **Information Protection**

  
![Purview](/img/purview-account2.png)   

---

### **1.3 Criar Governance Domain**

**Por quê?** Os Data Products devem pertencer a um Governance Domain publicado.

1. No Purview Portal → **Unified Catalog** → **Catalog management** → **Governance domains**
2. Clique em **New governance domain**:
   - **Name**: `ContosoRetailDomain`
   - **Description**: "Domain for retail sales and customer data products"
   - **Type**: Data Domain
   - **Parent**: Vazio
   - **Owner**: Atribua seu usuário
   - **Custom Attributes**: Vazio
3. **Create**, mas **NÃO publique ainda** (será publicado depois de criar os Data Products)


![Purview](/img/purview-account3.png) 



---

## 🗺️ PARTE 2: Registrar e Escanear o Fabric como Fonte


### **1. Configurar Security Group**

1. Azure Portal → **Microsoft Entra ID** → **Groups** → **New group**:
   - **Group type**: Security
   - **Name**: `sg-purview-fabric-readers`
   - **Description**: "Security group for Purview to scan Fabric"
   - **Members**: 
     - Purview Managed Identity (procure pelo nome da sua conta Purview)
2. **Create**

![Purview](/img/purview-account7.png)

---

### **2. Habilitar Admin APIs no Fabric**

1. Fabric Portal → **Settings** (⚙️) → **Admin portal** → **Tenant settings**
2. Procure: **"Admin API settings"**
3. Habilite as seguintes opções:
   - ☑️ **Service principals can access read-only admin APIs**
   - ☑️ **Enhance admin APIs responses with detailed metadata**
   - ☑️ **Enhance admin APIs responses with DAX and mashup expressions**
4. Em **"Apply to"** → Selecione **Specific security groups** → Adicione `sg-purview-fabric-readers`
5. **Apply**

⏱️ **IMPORTANTE: ESPERAR 15 minutos** antes de continuar com o registro do scan.

![Purview](/img/purview-account8.png)

---

### **2.4 Dar permissões ao Managed Identity do Purview no Workspace do Fabric**

1. Fabric Portal → Navegue até o seu Workspace (ex. `ContosoRetailWorkspace`)
2. Clique em → **Manage access**
3. **Add people or groups**
4. Procure sua Managed Identity MSI: adicione o grupo `sp-purview-fabric-readers`, que contém a Managed Identity
5. Atribua a função: **Contributor** ou **Admin**
6. **Add**


---

### **2.6 Registrar o Fabric Tenant no Purview Data Map**

1. Purview Portal → **Data Map** → **Data Sources** → **Register**
2. Selecione: **Microsoft Fabric** (same tenant)
3. Clique em **Continue**
4. **Register source**:
   - **Name**: `fabric-contoso-tenant`
   - **Fabric Tenant ID**: (preenchido automaticamente - seu tenant ID do Microsoft Entra - você encontra em Azure Portal → Microsoft Entra ID → Overview)
   - **Domain**: Crie um domínio de governança ou escolha o que está no padrão
   - **Select a collection**: Crie uma nova collection no Purview ou selecione alguma existente
5. **Register**

![Purview](/img/purview-account10.png)

---

### **2.7 Criar Scan do Fabric**

1. Na sua source `fabric-contoso-tenant` → Clique em **New scan**
2. **Name**: `scan-contoso-lakehouse`
3. **Personal workspaces**: Se quiser incluir ou excluir Workspaces pessoais (deixe em exclude)
4. **Connect via integration runtime**:
   - Selecione **Azure AutoResolveIntegrationRuntime**
5. **Credential**: Clique em **+ New**
   - **Name**: `cred-fabric-sp`
   - **Authentication method**: **Microsoft Purview MSI (system**
   - **Tenant ID**: (seu Microsoft Entra tenant ID)
   - **Collection**: A collection à qual o data source pertence 
   - **Create**
6. **Test connection** → Deve mostrar **Connection successful** ✅

![Purview](/img/purview-account11.png)


6. **Scope your scan**:
   - Na árvore de workspaces, expanda e selecione: `ContosoRetailWorkspace` ou seu Workspace
   
7. **Select a scan rule set**: 
   - Use o padrão: `Fabric`
   
8. **Set a scan trigger**:
   - **Once** (para este exercício)
   - Ou **Recurring** → Weekly (para ambientes de produção)

9. **Review your scan** → Verifique a configuração

10. **Save and run** 

⏱️ **O scan pode demorar de 5 a 15 minutos** dependendo do tamanho do seu Lakehouse.

![Purview](/img/purview-account12.png)


---

### **2.8 Verificar resultados do scan**

1. **Data Map** → **Sources** → `fabric-contoso-tenant` → Clique no nome
2. Vá até a aba **Scans** → Verifique se o status está como **Completed** ✅
3. Clique no nome do scan → **View details** 
4. Você deverá ver:
   - **Assets discovered**: Número de Lakehouses, Tables, Files encontrados
   - **Classifications applied**: Dados sensíveis detectados automaticamente
   - **Run time**: Duração do scan

**Exemplo de output esperado:**
```
Total assets discovered: 15
- Lakehouses: 1 (Contoso_Sales_Lakehouse)
- Tables: 3 (customers, sales, products)
- Files: 11 (parquet files)
Classifications applied: 8
- Personal.Email: 2 columns
- Personal.PhoneNumber: 1 column
- Personal.Location: 3 columns
```

---

## 📊 PARTE 3: Explorar Assets no Unified Catalog

### **3.1 Buscar Assets do Lakehouse**

1. Purview Portal → **Unified Catalog** → **Discovery** → **Data assets**
2. Nos filtros à esquerda:
   - **Source type**: Microsoft Fabric
   - **Collection**: ContosoData
3. Você deverá ver nos resultados:
   - Seu Lakehouse: `Contoso_Sales_Lakehouse`
   - Tabelas: `customers`, `sales`, `products`
   - Files: Arquivos parquet/delta individuais

![Purview](/img/purview-account13.png)

---

### **3.2 Revisar metadata de uma tabela**

1. Clique na tabela `gold.credit_score`
2. Explore as abas disponíveis:
   
   **Overview**:
   - Descrição
   - Owner/contacts
   - Collection
   - Source information
   
   **Schema**:
   - Colunas: nome, tipo de dado, descrição
   - Classificações aplicadas a cada coluna
   
   **Lineage**:
   - Origem dos dados (upstream)
   - Destinos onde são usados (downstream)
   - Nota: Pode estar vazio inicialmente até que você adicione pipelines
   
   **Properties**:
   - Metadata técnico (location, format, etc.)
   - Última modificação
   - Tamanho do asset

---

## 🏷️ PARTE 4: Glossário de Negócio e Data Products

### **4.1 Criar termos de glossário no Governance Domain**

**Documentação oficial:** [Create and manage glossary terms](https://learn.microsoft.com/purview/unified-catalog-glossary-terms-create-manage)

**Modelo atual:** Os termos de glossário são criados DENTRO de Governance Domains e associados a Data Products, NÃO diretamente a data assets individuais.


1. **Unified Catalog** → **Catalog management** → **Governance domains**
2. Clique no seu domain (nome da sua conta Purview por padrão)
3. Card **Glossary terms** → **View all** → **New term**

**Termo 1:**
```
Name: Cliente
Definition: Persona o entidad que realiza compras en Contoso Retail y está registrada en el CRM
Owner: [tu usuario]
Parent term: (ninguno)
Next → Next → Create
```

**Termo 2:**
```
Name: Venta  
Definition: Transacción comercial que incluye fecha, monto, productos y cliente asociado
Owner: [tu usuario]
Next → Next → Create
```

**Termo 3:**
```
Name: Producto
Definition: Artículo comercializable identificado por SKU único
Owner: [tu usuario]
Next → Next → Create
```

**Status:** Os 3 termos ficam em **Draft** (não publicados).

---

### **4.2 ⚠️ IMPORTANTE: Modelo de associação de termos**

**NO UNIFIED CATALOG:**
- ✅ Termos → são associados a **Data Products**
- ✅ Data Products → contêm **Data Assets**
- ❌ Termos NÃO são associados diretamente a data assets individuais

**Relação correta:**
```
Governance Domain
  └── Glossary Term: "Cliente"
       └── Data Product: "Sales Insights Product"
            └── Data Asset: customers table
```

#### **4.3: Vinculando termos de Glossário a partir de Data Products**

1. No seu data product `Sales Insights Product` → Seção **Glossary terms**
2. Clique no botão **+ (adicionar termos)** ao lado de "Glossary terms"
3. Abre-se um painel lateral de busca
4. Procure e selecione os termos:
   - ☑️ **Cliente**
   - ☑️ **Venta**
   - ☑️ **Producto**
5. Clique em **Add**

![Purview](/img/purview-account18.png)

---

## **4.3 Aplicar classificações (sensitivity labels) aos assets**

As classificações SIM são aplicadas diretamente a assets e colunas.

#### **A. Classificação automática (durante o scan)**
O Purview detecta automaticamente:
- Emails → `Personal.Email`
- Telefones → `Personal.PhoneNumber`
- Endereços → `Personal.Address`
- Localizações → `Personal.Location`

**Verificar classificações aplicadas:**
1. **Discovery** → **Data assets** → Procure a tabela `customers`
2. Aba **Schema** → você verá badges nas colunas classificadas

#### **B. Classificação manual**

1. Em **Discovery** → **Data assets** → Clique na tabela `credit_score`
2. Clique em **Edit**
3. Na seção **Schema**, para cada coluna:
   
   **Coluna `ssn`:**
   - Clique no ícone de lápis ao lado da coluna
   - **Classifications** → **+ Add classification**
   - Procure e selecione: `US Social Security Number`
   - **Apply**
4. **Save**

**Repita para outras tabelas sensíveis:**
- Tabela `transactions`: classificar colunas de cliente
- Tabela `products` ou `business_operations`: normalmente não requer classificação sensível, mas pode ser explorada


![Purview](/img/purview-account14.png)
  
---

## 🎁 PARTE 5: Criar e Publicar Data Product

### **5.1 Preparar o Governance Domain**

1. **Unified Catalog** → **Catalog management** → **Governance domains**
2. Clique em `ContosoRetailDomain`
3. Verifique se ele está em estado **Draft** (ainda não publicado); se não, você pode colocá-lo novamente em `Draft` para permitir alterações
4. Na seção **Business concepts** → Clique em **Go to data products**

---

### **5.2 Criar novo Data Product**

1. Clique em **New data product**
2. Preencha o formulário:

**Basic Information:**
```
Name: Sales Insights Product

Description: 
Este data product combina información de clientes y ventas para análisis de negocio. 
Proporciona una vista integrada que permite:
- Análisis de comportamiento de compra
- Segmentación de clientes por valor
- Identificación de tendencias de ventas
- Base para modelos predictivos


Data quality expectations:
- Actualización diaria
- Latencia máxima: 24 horas
- Completitud esperada: >95%

Type: Dashboard/Reports

Audience: Business User, Executive

Owner: [tu usuario]

Next:

Use cases:
- Dashboard ejecutivo de ventas mensuales
- Análisis de segmentación de clientes (RFM)
- Modelos predictivos de churn de clientes
- Reportes de cumplimiento de metas comerciales

Next:

Custom attributes: Vacio

```

3. **Create**

![Purview](/img/purview-account15.png)

---

### **5.3 Adicionar data assets ao produto**

1. No seu data product `Sales Insights Product` → Clique em **Add data assets** (na seção Assets)
2. No campo de busca:
   - **Search**: `credit_score`
   - Selecione a tabela `gold.credit_score` da sua Lakehouse
   - Clique em **Add**
3. Repita para adicionar:
   - Tabela `business_operations`
   - Tabela `gold.business_operations` (se existir)
   - Opcionalmente: Semantic Model do Power BI (se você tiver um publicado)

**Nota**: Você só pode adicionar assets que:
- Estejam no Data Map (já escaneados)
- Pertençam ao escopo do seu Governance Domain
- Você tenha permissões para visualizar


![Purview](/img/purview-account16.png)


---

### **5.4 Documentar o Data Product**

#### **A. Adicionar links externos**

1. No data product → Aba **Details**
2. Seção **Documentation** → Clique em **+ Add link**
3. **Add documentation link**:
```
   Display name: Especificación de Métricas de Ventas
   Link: https://contoso.sharepoint.com/sites/data/sales-metrics-spec
   Description: Documento con definiciones de KPIs y reglas de negocio
```
4. Clique em **Create**

![Purview](/img/purview-account17.png)



#### **B. Adicionar descrições aos assets**

1. Na seção **Data assets**, para cada asset adicionado:

   **Para a tabela `credit_score`:**
```
   Descripción: Tabla con información de clientes activos y sus atributos crediticios. 
   Incluye datos financieros, segmentación.
   Grain: Un registro por cliente único (customer_id)
   Actualización: Diaria a las 2:00 AM
```

   **Para a tabela `business_operations`:**
```
   Descripción: Tabla con transacciones históricas desde 2024.
   Contiene detalles de cada venta incluyendo productos, montos, descuentos y métodos de pago.
   Grain: Un registro por línea de venta (product__id)
   Actualización: Diaria a las 3:00 AM
```

---

### **5.5 Configurar políticas de acesso**

1. No data product → Clique em **Manage policies** (botão superior)
2. Aba **Access policies**:

**Configuração de tempo:**
```
Access time limit: 365 days (1 year)
Reason: Los usuarios necesitan acceso continuo para reportes recurrentes
```

**Workflow de aprovação:**
```
☑️ Approval required
Approvers: [Agrega tu usuario o un grupo de data stewards]

```

3. Clique em **Save**
   

5. (Opcional) Aba **Inherited policies**:
   - Aqui você verá políticas herdadas do Governance Domain
   - Por exemplo: políticas de data quality ou compliance

---

### **5.6 Publicar o Governance Domain**

⚠️ **IMPORTANTE**: Um Data Product só pode ser publicado se o seu Governance Domain for publicado primeiro.

1. Volte para **Catalog management** → **Governance domains**
2. Clique em `ContosoRetailDomain`
3. Verifique se ele possui:
   - ✅ Pelo menos um Data Product criado
   - ✅ Owner atribuído
   - ✅ Descrição completa
4. Clique em **Publish** (botão superior direito)

O status do domain mudará de **Draft** → **Published** ✅

---

### **5.7 Publicar o Data Product**

1. Vá para **Data products** → `Sales Insights Product`
2. Verifique se ele possui:
   - ✅ Pelo menos 1 data asset adicionado
   - ✅ Descrição e use cases completos
   - ✅ Owner atribuído
   - ✅ Políticas de acesso configuradas
3. Clique em **Publish** (botão superior)

O status do produto mudará para **Published** ✅

---





## 🎯 Resultado Final Alcançado

Ao concluir este exercício, você terá alcançado:

✅ **Catalogação automatizada**: assets do Fabric visíveis no Purview Data Map  
✅ **Data Product governado**: `Sales Insights Product` publicado com documentação completa  
✅ **Glossário de negócio**: termos de negócio vinculados a 12 assets  
✅ **Classificação de dados sensíveis**: colunas com etiquetas de privacidade aplicadas  
✅ **Linage de dados**: Rastreabilidade do Lakehouse até produtos de consumo  
✅ **Governança federada**: Workflow de solicitação e aprovação de acesso funcional  
✅ **Discoverability**: Data products pesquisáveis e consumíveis por toda a organização  

---

## 📚 Referências Oficiais

### **Documentação Core**
- [Purview + Fabric Integration Overview](https://learn.microsoft.com/en-us/fabric/governance/microsoft-purview-fabric)
- [Register and Scan Fabric Tenant (Same Tenant)](https://learn.microsoft.com/en-us/purview/register-scan-fabric-tenant)
- [Data Products in Unified Catalog](https://learn.microsoft.com/en-us/purview/unified-catalog-data-products)
- [Create and Manage Data Products](https://learn.microsoft.com/en-us/purview/unified-catalog-data-products-create-manage)

### **Tutoriais Passo a Passo**
- [Governance Tutorial - Publish Data Products](https://learn.microsoft.com/en-us/purview/section3-publish-data-products)
- [Sample Setup Walkthrough](https://learn.microsoft.com/en-us/purview/data-governance-setup-sample)
- [Get Started with Data Governance](https://learn.microsoft.com/en-us/purview/data-governance-get-started)

### **Configuração Avançada**
- [Data Quality for Fabric Lakehouse](https://learn.microsoft.com/en-us/purview/data-quality-for-fabric-data-estate)
- [Metadata and Lineage from Fabric](https://learn.microsoft.com/en-us/purview/data-map-lineage-fabric)
- [Microsoft Purview Hub in Fabric](https://learn.microsoft.com/en-us/fabric/governance/use-microsoft-purview-hub)

### **Permissões e Segurança**
- [Purview Permissions Overview](https://learn.microsoft.com/en-us/purview/catalog-permissions)
- [Access Policies for Data Products](https://learn.microsoft.com/en-us/purview/how-to-policies-data-owner-data-product)

---

## 🎓 Conceitos-Chave Aprendidos

### **O que é um Data Product no Purview?**
Um **Data Product** NÃO é apenas um dataset individual. É um **conceito de negócio** que:
- **Agrupa múltiplos assets relacionados** (tabelas, arquivos, relatórios) sob um caso de uso específico
- **Fornece contexto de negócio** (descrição, casos de uso, qualidade esperada)
- **Facilita a descoberta** usando linguagem de negócio, não técnica
- **Centraliza a governança** (uma política para todos os assets do produto)
- **Simplifica o acesso** (uma solicitação dá acesso a todos os assets)

### **Diferença: Data Map vs Unified Catalog**

| **Data Map** | **Unified Catalog** |
|---|---|
| Visão técnica dos assets | Visão de negócio dos products |
| Escaneamento automático de metadata | Curadoria manual de produtos |
| Orientado a data engineers | Orientado a data consumers |
| Catálogo de "o que existe" | Catálogo de "o que é útil" |

### **Fluxo de Governança no Purview + Fabric**
```
1. DISCOVERY (Data Map)
   ↓ Fabric assets → Purview scan → Data Map

2. CLASSIFICATION (Auto + Manual)
   ↓ Sensitive data → Labels applied → Compliance

3. CURATION (Unified Catalog)
   ↓ Business context → Glossary terms → Understanding

4. PRODUCTIZATION (Data Products)
   ↓ Group assets → Add context → Publish

5. ACCESS GOVERNANCE
   ↓ Request → Approval → Time-limited access

6. CONSUMPTION (Fabric Workspace)
   ↓ Discover product → Access data → Build solutions
```

---
