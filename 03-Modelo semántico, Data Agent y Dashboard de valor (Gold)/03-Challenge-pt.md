# 🏆 Desafio 3: Modelo Semântico, Data Agent e Dashboard de Valor no Microsoft Fabric (Camada Gold) 

📖 Cenário  
A Contoso busca **habilitar análises de negócio sobre dados confiáveis**.  
A equipe de dados deve construir um **modelo semântico**, criar um **Data Agent conectado ao modelo** e projetar um **dashboard de valor** no Power BI que permita responder perguntas-chave do negócio.  

**Antes de começar, conclua os Desafios 0-2. Certifique-se de ter as tabelas Silver e Gold preparadas**

---

### 🎯 Sua Missão  
Ao concluir este desafio, você será capaz de:  

✅ Projetar um **modelo semântico** vinculado à camada **Gold** com medidas, relacionamentos e dimensões conforme a necessidade.  
✅ Criar um **Data Agent** no Microsoft Fabric conectado a esse modelo.  
✅ Construir um **dashboard interativo no Power BI** com visualizações de valor.  
✅ Validar que o modelo responda corretamente a perguntas de negócio por meio do Copilot ou do Power BI.  

---

## 🚀 Passo 1: Projetar o Modelo Semântico  
💡 *Por quê?* O modelo semântico permite representar medidas, dimensões e relacionamentos de negócio de forma que os usuários possam consultar e analisar os dados com facilidade.  

1️⃣ No **Power BI ou Microsoft Fabric**, projete o **modelo semântico Gold** incluindo:  
   - 🔹 **Dimensões:** `Brand`, `Category`, `perfil_producto` (derivada: por exemplo, categorizar por Price > 100 como 'Premium'), `Availability`. Se preferir, você pode usar as tabelas desnormalizadas da camada Gold em vez de dimensões separadas.
   - 📏 **Medidas-chave:** Exemplos (você pode criar suas próprias medidas) 
     - `precio_total = SUM([Price])` (converta Price para numérico se estiver como string)  
     - `productos_disponibles = COUNTIF([Availability] = "backorder")` (ajuste conforme os valores reais no JSON)
       
2️⃣ Valide se as medidas e os relacionamentos estão configurados corretamente.  
3️⃣ Se tiver múltiplas tabelas (products, credit_score, transactions), crie os relacionamentos usando as chaves correspondentes  

✅ **Resultado esperado:** O modelo semântico Gold está completo e reflete a lógica de negócio da Contoso.  

---

## 🚀 Passo 2: Validar o Modelo com Perguntas de Negócio  
💡 *Por quê?* Validar o modelo garante que as consultas em linguagem natural no Copilot ou Power BI retornem respostas precisas.  

1️⃣ A partir do modelo semântico, crie um novo relatório e ative o Copilot; depois faça perguntas sobre o contexto dos dados no **Copilot Power BI**, por exemplo:  
   - 💬 “Qual categoria tem o preço médio mais alto?”  
   - 💬 “Qual é o preço total por marca?”  
   - 💬 “Quantos produtos estão em backorder?”  
   - 💬 “Qual perfil de produto gera mais receita?” (com base na medida derivada)
     
2️⃣ Se alguma resposta não estiver correta, ajuste as medidas ou os relacionamentos no modelo.  

✅ **Resultado esperado:** O modelo responde de forma precisa e coerente às perguntas de negócio.  

---

## 🚀 Passo 3: Projetar um Relatório/Dashboard no Power BI  
💡 *Por quê?* O dashboard permite visualizar métricas-chave e comunicar insights de negócio de forma eficaz.  

1️⃣ No **Power BI (dentro do Fabric ou Power BI Desktop)**, crie um novo relatório conectado ao seu modelo Gold (você pode usar o que já estava aberto anteriormente).    
2️⃣ Inclua novas visualizações como:  
   - 📊 **Preço médio por categoria (de products.json).**  
   - 💰 **Produtos por marca e estoque disponível.**  
   - 📈 **Tendências de preços por categoria.**
     
3️⃣ Personalize cores, títulos e formatação para melhorar a apresentação.  
4️⃣ Publique o dashboard no **workspace correspondente**.  

Opcionalmente, você pode usar o *Copilot do Power BI* para ajudar a criar conteúdo para o relatório

✅ **Resultado esperado:** O relatório/dashboard está publicado e pronto para responder perguntas sobre o negócio.

---

## 🚀 Passo 4: Criar um Data Agent Conectado ao Modelo  
💡 *Por quê?* Um **Data Agent** no Fabric permite que os usuários consultem os dados por meio de linguagem natural, potencializando o uso do **Copilot**.  

1️⃣ No Microsoft Fabric, crie um novo item **Data Agent** e conecte-o ao seu **modelo semântico Gold**.  
2️⃣ Vincule-o a tabelas como `gold.products`, `gold.business_operations` e `gold.credit_score` (criadas no Passo 1).  
3️⃣ Configure as instruções do agente para influenciar o raciocínio do modelo LLM. Forneça orientação sobre como responder e o contexto de colunas, métricas e agregações.  
4️⃣ Teste consultas em linguagem natural para validar que o agente responde adequadamente.  
   Opcional: Você pode vincular o Data Agent às suas tabelas Gold da Lakehouse e testar o comportamento dele. Eventualmente, poderíamos construir diferentes Data Agents para áreas distintas (Products Data Agent, Credit Score Agent, etc.)
   

✅ **Resultado esperado:** O Data Agent está conectado ao modelo e permite realizar consultas interativas e estratégicas sobre o contexto do negócio. 

---

## 🏁 Pontos de Controle Finais  

✅ O modelo semântico foi projetado com medidas-chave, relacionamentos ou dimensões adequadas? 
✅ O modelo responde corretamente a perguntas de negócio no Copilot ou Power BI?  
✅ O Data Agent conectado ao modelo foi criado e testado?  
✅ O dashboard está publicado e funcionando corretamente?  

**Valide que as medidas funcionem importando uma amostra dos JSON no Fabric ou gerando um dataset sintético com novos dados.**

---

## 📝 Documentação  

- [Modelo Semântico Gold (Power BI)](https://learn.microsoft.com/es-es/fabric/data-warehouse/semantic-models)  
- [Atualizar Modelo Semântico](https://learn.microsoft.com/es-es/power-bi/connect-data/data-pipeline-templates)
- [Criar Data Agent](https://learn.microsoft.com/es-es/fabric/data-science/how-to-create-data-agent)
- [Como unir tabelas no Fabric](https://learn.microsoft.com/en-us/fabric/data-engineering/tutorial-build-lakehouse)

💡 *Dica:* Documente os relacionamentos, medidas e fontes de dados utilizadas, pois esse modelo servirá como base para a criação de **copilotos empresariais** e **análises preditivas avançadas**. 🚀  
