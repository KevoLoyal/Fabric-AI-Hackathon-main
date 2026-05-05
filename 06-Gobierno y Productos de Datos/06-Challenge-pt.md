# 🚀 Desafio 6: Governança de Dados com Purview + Fabric  

## 🎯 Objetivo  
Catalogar dados do Fabric no Purview e criar um Data Product governado com políticas de acesso e termos de glossário associados.

## 📋 Pré-requisitos  
- Conta do Microsoft Purview (mesmo tenant do Fabric)
- Workspace do Fabric com Lakehouse populado (exercícios anteriores)

- Permissões de **Admin** no workspace do Fabric
- Função **Data Governance Administrator** no Purview

## 🧠 Tarefas do Desafio

### 1. Configurar o Purview
- Acessar https://purview.microsoft.com
- Usar o Governance Domain padrão (ele tem o nome da sua conta Purview)
- **Não publicar o domain até o final**

### 2. Escanear o Fabric no Purview Data Map
**Configurar autenticação:**
- Criar Security Group com: Purview Managed Identity


**Configurar Fabric:**
- Habilitar "Admin API settings" em Fabric Tenant Settings para o Security Group
- Dar permissões de **Contributor** ao Security Group no seu workspace

**Executar scan:**
- Registrar o tenant do Fabric como source no Purview Data Map
- Criar scan usando (Managed Identity)
- Verificar que ele descubra: Lakehouse + Tabelas + Arquivos

### 3. Criar Termos de Glossário
No seu Governance Domain → Business concepts → Glossary terms:
- Criar 3 termos com definições de negócio:
  - "Cliente"
  - "Venda" 
  - "Produto"
- Os termos ficam no estado **Draft** (não publicados)

### 4. Criar Data Product
No seu Governance Domain → Business concepts → Data products:
- Criar data product: "Sales Insights Product"
- Adicionar description e use cases detalhados
- **Adicionar data assets**: Tabelas `customers` e `sales` da Lakehouse
- **Associar glossary terms**: Cliente, Venda, Produto
- Configurar **access policy**: Approval required, 365 days
- Adicionar documentation links
- O produto fica no estado **Draft**

### 5. Publicar
- **Publicar Governance Domain** (isso habilita a publicação de termos e produtos)
- **Publicar Data Product** (agora ele pode ser publicado)
- Verificar em **Discovery → Enterprise glossary** que os termos estejam visíveis
- Verificar em **Discovery → Data products** que o produto seja descobrível



## 🏁 Entregáveis
✅ Tenant do Fabric escaneado (Lakehouse + tabelas visíveis no Data Map)  
✅ 3 termos de glossário publicados  
✅ 1 Data Product publicado com 2+ assets e termos associados  
✅ Screenshot de linhagem de dados