# 🏆 Desafio 2: Transformação Intermediária e Análise Exploratória no Microsoft Fabric (Camada Silver) 

📖 Cenário  
A Contoso busca **avaliar a qualidade de seus dados** antes de construir modelos preditivos e analíticos.  
Para isso, a equipe de dados deve **transformar e analisar os dados** que vêm da camada **Bronze**, gerando uma versão intermediária otimizada na camada **Silver**.  

---

### 🎯 Sua Missão  
Para concluir este desafio, você deverá:  

✅ Criar uma **tabela Silver** a partir dos dados limpos na **Bronze**.  
✅ Aplicar **transformações intermediárias** que melhorem a estrutura e a consistência dos dados.   
✅ Realizar uma **análise exploratória** usando técnicas de agrupamento e machine learning (ML). Exemplo: Segmentação de clientes por score.    
✅ Deixar os dados prontos para a etapa de **modelagem semântica (Gold)**.  

---

## 🚀 Passo 1: Construir Tabelas Silver a partir da camada Bronze  
💡 *Por quê?* A camada Silver serve como base para aplicar transformações e análises intermediárias, preparando os dados para a modelagem analítica posterior.  

1️⃣ Acesse o **Lakehouse** no seu workspace do Microsoft Fabric.  
2️⃣ Utilize um **notebook** (idealmente) com Spark dataframes para a camada silver, ou um **Dataflow Gen2** se preferir, para ler os dados da camada **Bronze**.  
3️⃣ Aplique limpezas adicionais, se necessário (por exemplo, correção de formatos ou padronização de nomes de colunas, derivação de novas colunas).  
4️⃣ Armazene essas versões mais curadas dos dados como tabelas **Silver**  

✅ **Resultado esperado:** Os dados da Bronze estão mais refinados e prontos na camada Silver para aplicar transformações/agregações mais avançadas da camada Gold.  

---

## 🚀 Passo 2: Aplicar Transformações Intermediárias
💡 *Por quê?* Essas transformações permitem gerar visões analíticas e facilitar os processos de modelagem e segmentação.  

1️⃣ Crie um **notebook do Fabric** para a camada **Gold**  
2️⃣ Aplique transformações que agreguem valor analítico, por exemplo:  
   - 📊 **Agrupamentos:** Identificar o **score de crédito mais alto por cliente**.  
   - 🏷️ **Perfis de produto:** Classificar produtos por categoria ou nível de vendas.  
3️⃣ Crie novas colunas ou métricas que sirvam para análises posteriores (por exemplo, média de compras ou níveis de risco).  

✅ **Resultado esperado:** Os dados da tabela silver agora contêm transformações úteis e estão prontos para análise exploratória, modelagem ou segmentação.  

---

## 🚀 Passo 3: Realizar uma Análise Exploratória com ML (Dataset de Credit Score)
💡 *Por quê?* As técnicas de **Machine Learning (ML)** permitem avaliar a distribuição e a similaridade entre os dados, ajudando a descobrir padrões. **Nota:** Se você não quiser trabalhar com ML e preferir realizar outra análise, pule esta etapa e substitua-a por sua própria preparação na camada Silver

Para Credit Score e Products

1️⃣ Use **funções de ML integradas** ou **bibliotecas PySpark MLlib** / **scikit-learn** no seu notebook.   
2️⃣ Implemente um algoritmo de **K-Means** ou outro que considere útil para agrupar registros em clusters:  
   - 🎯 Agrupe clientes ou produtos segundo características numéricas semelhantes.  
   - 🔍 Analise as relações entre variáveis dentro de cada cluster.  
3️⃣ Salve a versão final das tabelas no **Lakehouse [Silver}**  

✅ **Resultado esperado:** Você obtém uma segmentação dos seus dados de clientes e uma compreensão mais profunda do seu comportamento. Como foi mencionado anteriormente, se você preferir não trabalhar com ML, tente desenvolver uma análise que descubra novos insights a partir dos dados.

Para as outras tabelas de `productos`, `transacciones`, realize preparações e ajustes semelhantes que permitam trabalhar um modelo analítico na camada Gold.

---

## 🚀 Passo 4: Preparar a Tabela para a Modelagem Semântica (Camada Gold)  
💡 *Por quê?* A preparação final da tabela Gold é a etapa final antes de criar modelos analíticos ou dashboards de negócio.  

1️⃣ Ajuste nomes de colunas, tipos de dados e chaves primárias necessárias para a modelagem; elimine colunas desnecessárias.  
2️⃣ Salve a versão final das tabelas no **Lakehouse [Gold}** ou publique-a como fonte para a **camada Gold**.  

✅ **Resultado esperado:** Os dados estão prontos para serem consumidos na camada Gold por ferramentas de BI ou modelos de análise avançados.  

---

## 🏁 Pontos de Controle Finais  

✅ As tabelas Silver foram criadas corretamente a partir da Bronze?  
✅ Transformações intermediárias foram aplicadas (agrupamentos, cálculos, perfis)?  
✅ Um modelo de segmentação (KMeans) ou preparação de dados foi implementado e analisado?  
✅ Os dados estão prontos para uso na camada Gold?  
✅ As transformações e os resultados da análise exploratória foram documentados?  

---

## 📝 Documentação  

- [Notebook de Transformações e ML](https://learn.microsoft.com/es-es/fabric/data-engineering/how-to-use-notebook)  


💡 *Dica:* Mantenha um registro dos parâmetros e resultados dos seus modelos, pois eles serão fundamentais para o próximo desafio: **modelagem semântica**. 🚀  
