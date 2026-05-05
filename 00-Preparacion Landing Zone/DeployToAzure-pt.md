
# 🚀 Implantar Landing Zone para o Hackathon

Este repositório permite criar uma **Landing Zone base** para preparar o ambiente de um hackathon de forma **rápida, consistente e repetível**.

O objetivo é que os participantes não percam tempo criando infraestrutura básica e possam se concentrar diretamente em **desenvolvimento, inovação e uso de dados**.

---

## 🟦 O que faz o botão **Deploy to Azure**?

Ao clicar no botão **Deploy to Azure**, será implantada automaticamente na sua assinatura do Azure uma **Landing Zone mínima**, que inclui:

- Um **Azure Cosmos DB** para armazenamento de dados estruturados e semiestruturados.
- Uma **Azure Storage Account** para blobs, arquivos e outros artefatos do hackathon.
- Recursos organizados em um **Resource Group dedicado**.

Toda a implantação é realizada usando **Infrastructure as Code (IaC)**, garantindo que todas as equipes partam do mesmo ponto.

---

## 🧱 Recursos que são implantados

### 📌 Fontes de dados
  - Ideal para cenários de:
    - Cosmos NoSQL
    - Storage Accounts


### 📦 Armazenamento

  - Usado para:
    - Carga de arquivos
    - Datasets do hackathon
  

---

## 🎯 Para que serve esta Landing Zone?

Esta Landing Zone foi pensada para:

- ✅ Preparar o ambiente base do hackathon em minutos  
- ✅ Evitar configurações manuais repetitivas  
- ✅ Garantir consistência entre todas as equipes  
- ✅ Reduzir erros de configuração  
- ✅ Facilitar a avaliação das soluções  



## 🚀 Como usar o botão

1. Clique no botão **Deploy to Azure**.
2. Entre com sua conta do Azure.
3. Selecione a **assinatura** e o **Resource Group**.
4. Preencha os parâmetros solicitados (se aplicável).
5. Confirme a implantação.

Em poucos minutos, o ambiente estará pronto.

---


[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmakanto32%2FHackathon-Mexico%2Fmain%2F00-Preparacion%2520Landing%2520Zone%2Fazuredeploy.json
)