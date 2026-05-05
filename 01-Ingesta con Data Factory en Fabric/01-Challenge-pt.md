# 🏆 Desafio 1: Ingestão de Dados do Cosmos DB para o Microsoft Fabric (Camada Bronze) + Limpeza Básica  

📖 Cenário  
A Contoso precisa consolidar seus **dados operacionais e financeiros** no **Microsoft Fabric**.  
A equipe de dados deve realizar a **ingestão do Azure Cosmos DB** para a camada **Bronze** e aplicar uma **limpeza mínima inicial** para preparar os dados antes de avançar para as próximas fases de transformação.  

---

### 🎯 Sua Missão  
Ao concluir este desafio, você será capaz de:  

✅ Ingerir os dados do **Azure Cosmos DB** para o **Microsoft Fabric** usando **Dataflows Gen2**.  
✅ Aplicar **limpeza básica** que inclua, por exemplo:  
- Tratamento de valores nulos ou vazios.  
- Remoção de colunas desnecessárias.  
- Normalização de formatos básicos (datas, texto, etc.)
  
✅ Gerar uma camada semi-bruta/bruta dentro da **Bronze** da Lakehouse.  

---

## 🚀 Passo 1: Criar um Dataflow Gen2 para a Ingestão a partir do Cosmos DB (Vocês podem usar outros métodos de ingestão)
💡 *Por quê?* Os **Dataflows Gen2** permitem realizar a ingestão e a transformação inicial dos dados sem necessidade de código, conectando facilmente fontes externas como o Cosmos DB à sua Lakehouse. Opcional: Esta ingestão também pode ser realizada a partir de um **Pipeline** com atividades de cópia, **Notebooks**, ou até mesmo com a opção de **Mirroring do Cosmos DB**, que permite expor os contêineres do Cosmos no Fabric

1️⃣ No **Microsoft Fabric**, crie um novo **Dataflow Gen2** dentro do seu workspace.  
2️⃣ Selecione **Azure Cosmos DB** como fonte de dados.  
3️⃣ Informe as credenciais de conexão (endpoint e chave de acesso).  
4️⃣ Conecte-se ao contêiner que contém os dados de **produtos**, **credit score** e **transações**.  
5️⃣ Defina como destino o schema [bronze] da sua **Lakehouse** para armazenar os dados ingeridos (certifique-se de ativar o flag TRUE de *`navigate full hierarchy`* nas opções avançadas da conexão com a Lakehouse; isso permite navegar pelos schemas).  

✅ **Resultado esperado:** Os dados JSON do Cosmos DB estão disponíveis nas queries do Dataflow Gen2

## 🚀 Passo 2: Aplicar Limpezas Básicas no Dataflow Gen2  
💡 *Por quê?* Esta etapa melhora a qualidade dos dados, garantindo consistência e usabilidade para análises posteriores. É normal que algumas organizações façam limpezas na Bronze; outras ingerem os dados totalmente brutos para prepará-los depois.  
1️⃣ Edite seu **Dataflow Gen2** para adicionar etapas de transformação:  
   - 🧹 **Remover colunas desnecessárias** que não agregam valor analítico.  
   - 🩹 **Substituir ou remover valores nulos ou vazios.**  
   - 🕒 **Normalizar formatos básicos** (por exemplo, campos de data ou texto em minúsculas).
      
2️⃣ Salve e execute o Dataflow para aplicar as transformações.  
3️⃣ Publique os resultados na **camada Bronze** da sua Lakehouse.  

✅ **Resultado esperado:** As tabelas “Bronze” contêm dados limpos e prontos para sua transformação na camada Silver.  

---

## 🚀 Passo 3: Validar a Carga e a Estrutura dos Dados  
💡 *Por quê?* Validar a ingestão garante que os dados estejam completos e coerentes antes de iniciar a limpeza.  

1️⃣ Acesse sua **Lakehouse** a partir do painel do Fabric.  
🔹 Verifique se as tabelas delta criadas contêm os campos esperados.  
🔹 Confirme que não existam erros de formato ou registros incompletos.  

✅ **Resultado esperado:** A estrutura base dos dados foi validada corretamente.  

---


## 🏁 Pontos de Controle Finais  

✅ A ingestão do Cosmos DB por meio de Dataflows Gen2 foi concluída?  
✅ As limpezas básicas foram aplicadas corretamente?  
✅ Os dados resultantes estão armazenados e acessíveis na camada Bronze?  
✅ Os passos realizados e as evidências visuais foram documentados?  

---

## 📝 Documentação  


- [Criação de Dataflow Gen2](https://learn.microsoft.com/es-mx/fabric/data-factory/create-first-dataflow-gen2)