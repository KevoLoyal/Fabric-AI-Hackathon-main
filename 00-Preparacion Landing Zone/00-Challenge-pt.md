# 🏆 Desafio 0: Configuração da Landing Zone e Preparação de Dados no Microsoft Fabric  
📖 Cenário  
A Contoso Retail carregou três conjuntos de dados no formato **JSON**:  
- Um **financeiro**, com informações de **score de crédito** de clientes.  
- Dois de **varejo**, com dados de **transações e produtos**.  

Sua missão é **preparar o ambiente de trabalho no Microsoft Fabric**, conectando os dados armazenados no **Azure Cosmos DB** e estabelecendo uma **zona de aterrissagem (landing zone)** estruturada em camadas para iniciar o processo de transformação.  

---

### 🎯 Sua Missão  
Ao concluir este desafio, você será capaz de:  

✅ Criar Cosmos DB NoSQL e enviar os arquivos para os contêineres.
✅ Configurar um **workspace** no Microsoft Fabric para a gestão de dados.  
✅ Conectar o **Azure Cosmos DB** como fonte de dados.  
✅ Explorar e compreender a estrutura dos arquivos JSON financeiros e de varejo.  
✅ Criar uma **Lakehouse** com estrutura em camadas (**Bronze**, **Silver**, **Gold**).  
✅ Definir e documentar o fluxo de dados entre as camadas.  

---
## 🚀 Passo 1: Criar o Azure Cosmos DB para NoSQL 
💡 *Por quê?* O Cosmos DB servirá como a fonte dos dados que serão ingeridos a partir do Fabric 

1️⃣ Acesse o portal do **Microsoft Azure** e crie um banco de dados Cosmos DB para NoSQL. Para maior conveniência, recomendamos executar o template de implantação que permitirá acelerar este processo de criação de artefatos do Azure, como mostrado no arquivo [DeployToAzure](./DeployToAzure.md)  
🔹 Atribua um nome descritivo (por exemplo, `ContosoData-Source`).  
🔹 Crie o contêiner e atribua um nome identificável.  
🔹 Envie o dataset em formato JSON

✅ **Resultado esperado:** Você terá um Cosmos DB com um contêiner contendo as informações prontas para serem ingeridas a partir do Fabric.

## 🚀 Passo 2: Criar um Workspace no Microsoft Fabric  
💡 *Por quê?* O workspace é o ambiente centralizado onde datasets, dataflows, pipelines e notebooks são gerenciados.  

1️⃣ Acesse o **Microsoft Fabric** e crie um novo workspace para o projeto da Contoso.  
🔹 Atribua um nome descritivo (por exemplo, `ContosoData-Fabric`).  
🔹 Certifique-se de que ele esteja atribuído a uma **Fabric Capacity** (se você já a tiver configurada, pode ignorar esta etapa).  

✅ **Resultado esperado:** Você terá um workspace dedicado para todos os recursos do Fabric.  

---

## 🚀 Passo 3: Conectar ao Azure Cosmos DB  
💡 *Por quê?* Estabelecer essa conexão permite que o Fabric acesse e ingira diretamente os dados JSON a partir do Cosmos DB.  

1️⃣ No seu workspace do Fabric, crie uma nova **conexão de dados** com o **Azure Cosmos DB**.  
🔹 Forneça o **endpoint** e a **chave de acesso** corretos.  
🔹 Verifique se as permissões estão configuradas adequadamente.  

✅ **Resultado esperado:** Seu workspace estará conectado ao Cosmos DB e pronto para a ingestão de dados.  

---

## 🚀 Passo 4: Criar uma Lakehouse e Definir a Estrutura em Camadas  
💡 *Por quê?* A Lakehouse é a base da arquitetura de dados e permite separar as etapas de processamento.  

1️⃣ No Fabric, crie uma **Lakehouse** (ative o suporte para schemas) chamada `Contoso_Lakehouse`.  
2️⃣ Dentro da Lakehouse, defina a seguinte estrutura de schemas:  
   - 🥉 **Bronze:** Dados brutos e sem processamento, ingeridos diretamente do Cosmos DB.  
   - 🥈 **Silver:** Dados limpos, normalizados e consistentes.  
   - 🥇 **Gold:** Dados curados e prontos para análise ou visualizações.  

✅ **Melhor prática:** Mantenha uma convenção clara de nomes para schemas e tabelas que facilite o rastreamento do fluxo de dados.  

✅ **Resultado esperado:** Sua Lakehouse contará com uma base estruturada que dará suporte às transformações e à análise de dados.  

---

## 🏁 Pontos de Controle Finais  

✅ O Cosmos DB e os contêineres foram criados corretamente?  
✅ O workspace no Microsoft Fabric foi criado corretamente e está conectado ao Cosmos DB?  
✅ A estrutura dos datasets JSON no Cosmos foi validada?  
✅ A Lakehouse está organizada com as camadas Bronze, Silver e Gold?  
✅ A estratégia de fluxo de dados entre as camadas foi documentada?  

---

💡 **Próximos Passos:**  
Com a **landing zone configurada**, você está pronto para avançar para o próximo desafio, no qual começará com a **ingestão, limpeza e transformação de dados** dentro do Fabric. 🚀  

---

**📄 Documentação**
- [Criação do Cosmos DB](https://learn.microsoft.com/es-es/azure/cosmos-db/nosql/quickstart-portal)
- [Permitir IP pública no Firewall](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops&tabs=IP-V4)
- [Criação de workspace no Fabric](https://learn.microsoft.com/es-es/fabric/data-warehouse/tutorial-create-workspace)
- [Criação de lakehouse no Fabric](https://learn.microsoft.com/es-es/fabric/data-engineering/tutorial-build-lakehouse)
- [Criar Pipeline](https://learn.microsoft.com/es-mx/fabric/data-factory/create-first-pipeline-with-sample-data)