<p align="center">
 
  <img src="/img/fabric.png" alt="Microsoft Fabric" width="90"/>
  &nbsp;&nbsp;&nbsp;
  <img src="/img/ai-foundry.png" alt="Foundry" width="90"/>
</p>


# 🧠 AI Fabric Hackathon  

## 🎯 Objetivos do Hackathon

Ao final deste hackathon, os participantes serão capazes de:

- Preparar, transformar e enriquecer dados financeiros, de varejo e transacionais usando **Microsoft Fabric**, aplicando o modelo **medallion** para estruturar camadas de valor analítico.  
- Ingerir dados de sistemas core, fontes externas e APIs por meio de **pipelines, notebooks e conectores nativos do Fabric**.  
- Projetar **modelos semânticos** robustos que facilitem o consumo de dados por analistas, auditores e sistemas de inteligência.  
- Monitorar e otimizar o consumo de capacidade no **Fabric**, aplicando métricas-chave para governança operacional e eficiência de recursos.  
- Construir **agentes de inteligência artificial** com **AI Foundry** para análise preditiva, detecção de fraude e geração de insights financeiros.  
- Orquestrar fluxos multiagente e processos de dados, habilitando automação inteligente em cenários bancários e de seguros.
- Visualizar **insights estratégicos** com **Power BI no Microsoft Fabric**, habilitando painéis interativos para decisões baseadas em dados.
  
**Bônus**
- Aplicar **controles de segurança e governança** de dados sensíveis, configurando papéis, permissões e políticas em workspaces do Fabric.   
- Integrar **Microsoft Purview** para rastreabilidade, classificação e conformidade regulatória, fortalecendo a governança de dados em ambientes regulados.  




# Agenda


| Dia  | Atividade                                                                  | Tipo    |
|------|----------------------------------------------------------------------------|---------|
| Dia 1 | Preparação de dados (estruturação, limpeza, perfilamento)                 | Desafio |
| Dia 1 | Ingestão de dados de fontes internas e externas                           | Desafio |
| Dia 1 | Transformação de dados com notebooks e pipelines                          | Desafio |
| Dia 1 | Enriquecimento de dados e criação de modelo semântico                     | Desafio |
| Dia 1 | Mesa-redonda: Q&A com especialistas e participantes                       | Desafio |
| Dia 1 | Encerramento e resumo do dia                                              | Encerramento |
| Dia 2 | Construção de agente em AI Foundry para análise preditiva                 | Desafio |
| Dia 2 | Orquestração multiagente com pipelines e triggers                         | Desafio |
| Dia 2 | Segurança no Fabric: papéis, objetos, workspaces (opcional)              | Desafio |
| Dia 2 | Sessão de valor: Q&A sobre adoção, impacto e próximos passos             | Encerramento |
| Dia 2 | Encerramento e entrega de reconhecimentos                                 | Encerramento |


# Arquitetura
![Arquitetura](img/architecture.png)


# 📖 História do Caso de Uso
 
## "Contoso e a Inteligência de Dados Multissetorial em Ação"
 
**Contoso**, uma organização com presença nos setores **financeiro e comercial**, enfrenta o desafio de consolidar informações provenientes de múltiplas fontes para habilitar análises confiáveis, automação inteligente e experiências conversacionais baseadas em dados. No contexto deste hackathon, os participantes assumem o papel de **equipe técnica** encarregada de construir uma solução moderna sobre **Microsoft Fabric**, colocando à prova suas habilidades em um ambiente realista e multissetorial.
 
### 🗃️ Fontes de Dados
O cenário começa com três conjuntos de dados em formato **JSON**, ingeridos a partir de um banco de dados NoSQL **Cosmos DB**:
 
• **Conjunto de score de crédito:** informações de clientes, comportamento de pagamento e perfil financeiro

• **Conjunto de produtos de varejo:** dados sobre disponibilidade, valor comercial, categoria e marca de produtos de varejo

• **Conjunto de transações:** compras de clientes, canais de compra, taxas de juros, localização



![Modelo de Dados](img/container_schemas.png)


 
### 🎯 Objetivo Principal
Transformar, limpar e estruturar os datasets em um **modelo enriquecido** que sirva como base para a criação de **agentes de inteligência artificial**. Para isso, os participantes aplicarão o **modelo medallion** (Bronze → Silver → Gold), assegurando a qualidade, a rastreabilidade e o valor analítico da informação. O foco do exercício é orientado a objetivos, portanto não existe uma única solução e incentiva-se o uso de diferentes abordagens para concluí-lo.
 
### 📊 Modelo Semântico e Métricas-Chave
Uma vez estruturados os dados na **camada Gold**, será projetado um **modelo semântico em Power BI**, que permitirá correlacionar métricas-chave como, por exemplo:
 
• Score médio por segmento  
• Valor comercial por categoria  
• Taxa de devolução por marca  
• Tendências mensais de risco ou vendas  
• Performance por canal  
• Análise de MSI (Meses sem Juros)  
• Métodos de pagamento  

 
### 🤖 Agentes Conversacionais
Utilizando **AI Foundry**, os participantes criarão **agentes** capazes de interagir com os dados por meio de **linguagem natural**, sem exibir código técnico, resolvendo desafios de automação e orquestrando fluxos multiagente com **modelos de linguagem de grande escala (LLMs)**. Esses agentes estarão conectados aos modelos semânticos por meio de **Data Agents**, permitindo consultas conversacionais como:
 
• *"Qual segmento tem o maior score médio?"*  
• *"Quais produtos têm a maior taxa de devolução?"*  
• *"Existe relação entre score e valor de compra?"*  
• *"Como os clientes compram de acordo com seu perfil de crédito?"*  
• *"Quais categorias de produtos cada perfil de crédito prefere?"*  
 
### 📈 Visualização e Insights
Por fim, os **insights gerados** serão visualizados em **painéis interativos no Power BI**, facilitando a tomada de decisão baseada em dados tanto para **analistas financeiros** quanto **comerciais**. Este caso exemplifica uma adoção realista e escalável do **Microsoft Fabric** em ambientes híbridos, onde a **inteligência de dados** se torna uma vantagem competitiva para a Contoso, impulsionando a inovação, a eficiência operacional e a democratização da análise.
 
---
 
# 🎯 Resumo dos Desafios - Do Insight à Decisão
 
## 🏆 Desafio 00: Configuração da Landing Zone e Preparação de Dados
 
**📖 Cenário:** A Contoso deve preparar o ambiente de trabalho no Microsoft Fabric, conectando dados armazenados no Azure Cosmos DB e estabelecendo uma landing zone estruturada em camadas.
 
### 🎯 Objetivos-Chave:
- ✅ Criar Azure Cosmos DB NoSQL e carregar datasets JSON (financeiro, varejo, transações)
- ✅ Configurar workspace no Microsoft Fabric com estrutura em camadas
- ✅ Estabelecer conexão entre Cosmos DB e Fabric
- ✅ Criar Lakehouse com arquitetura medallion (Bronze, Silver, Gold)
- ✅ Explorar e validar a estrutura dos dados JSON
 
### 🚀 Entregáveis:
- Cosmos DB configurado com contêineres de dados
- Workspace do Fabric com Lakehouse estruturado por camadas
- Documentação do fluxo de dados planejado
 
---
 
## 🏆 Desafio 01: Ingestão de Dados do Cosmos DB para o Microsoft Fabric (Camada Bronze)
 
**📖 Cenário:** Consolidar dados operacionais da Contoso no Microsoft Fabric por meio da ingestão do Azure Cosmos DB para a camada Bronze, aplicando limpeza básica.
 
### 🎯 Objetivos-Chave:
- ✅ Implementar ingestão com Dataflows Gen2 a partir do Cosmos DB
- ✅ Aplicar limpeza básica (valores nulos, colunas desnecessárias, normalização)
- ✅ Validar carga e estrutura de dados na camada Bronze
- ✅ Preparar dados para transformações avançadas
 
### 🚀 Entregáveis:
- Dataflow Gen2 funcional com transformações básicas
- Tabela Bronze com dados limpos e estruturados
- Validação da integridade dos dados ingeridos
 
---
 
## 🏆 Desafio 02: Transformação Intermediária e Análise Exploratória (Camada Silver)
 
**📖 Cenário:** Avaliar a qualidade dos dados e criar uma versão intermediária otimizada na camada Silver, aplicando transformações avançadas e análise exploratória com Machine Learning.
 
### 🎯 Objetivos-Chave:
- ✅ Criar tabelas Silver com transformações intermediárias
- ✅ Aplicar agrupamentos e métricas analíticas (score de crédito por cliente, perfis de produto, transações por canal)
- ✅ Implementar análise exploratória com clustering K-Means ou outra modelagem de preferência (pode ser uma análise não preditiva caso ML não seja utilizado)
- ✅ Preparar dados para modelagem semântica na Gold
 
### 🚀 Entregáveis:
- Tabelas Silver com transformações, predições e métricas de negócio
- Análise de clustering com insights de segmentação (ou análise equivalente)
- Dados otimizados prontos para a camada Gold
 
---
 
## 🏆 Desafio 03: Modelo Semântico, Data Agent e Dashboard de Valor (Camada Gold)
 
**📖 Cenário:** Habilitar análise de negócios por meio de um modelo semântico robusto, Data Agent conversacional e dashboard interativo para responder perguntas-chave do negócio.
 
### 🎯 Objetivos-Chave:
- ✅ Projetar modelo semântico Gold com medidas e relacionamentos relevantes. Podem ser implementados modelos normalizados ou desnormalizados.
- ✅ Criar Data Agent conectado ao modelo semântico ou às tabelas Gold do Lakehouse
- ✅ Desenvolver dashboard em Power BI com visualizações de valor
- ✅ Validar respostas a perguntas de negócio por meio do Copilot
 
### 🚀 Entregáveis:
- Modelo semântico com medidas-chave (exemplo: valor_comercial_total, produtos_disponiveis)
- Data Agent funcional para consultas em linguagem natural
- Dashboard do Power BI publicado com métricas estratégicas
 
---
 
## 🏆 Desafio 04: Criação de Agente Conversacional no AI Foundry
 
**📖 Cenário:** Permitir que analistas interajam com dados usando linguagem natural, criando um agente no Azure AI Foundry integrado ao modelo semântico do Fabric.
 
### 🎯 Objetivos-Chave:
- ✅ Projetar agente conversacional no AI Foundry integrado ao Fabric
- ✅ Conectar o agente ao Data Agent associado ao modelo semântico Gold
- ✅ Configurar intents e prompts orientados a perguntas reais de negócio
- ✅ Validar respostas em linguagem natural sem código técnico
- ✅ Publicar o agente para uso por analistas
 
### 🚀 Entregáveis:
- Agente conversacional funcional no AI Foundry conectado ao Data Agent do Fabric
- Configuração de intents para perguntas de negócio frequentes
- Integração completa com o modelo semântico do Fabric
- Validação de respostas em linguagem natural
 
---
 
## 🏆 Desafio 05: Orquestração Multiagente e Fluxos Colaborativos
 
**📖 Cenário:** Projetar e documentar um fluxo multiagente que coordene ingestão, análise e execução para automatizar tarefas complexas e adaptar-se dinamicamente a cenários em constante mudança.
 
### 🎯 Objetivos-Chave:
- ✅ Definir três agentes especializados [Sales Analyst, Credit Analyst, Research Analyst] e um sintetizador [Strategy Advisor]. Você pode defini-los de acordo com o cenário que propôs.
- ✅ Projetar fluxo orquestrado
- ✅ Simular cenários de negócio e validar o comportamento dos agentes
- ✅ Documentar o desenho para replicabilidade e escalabilidade
 
### 🚀 Entregáveis:
- Arquitetura de três agentes com papéis definidos e um agente sintetizador
- Fluxo orquestrado
- Simulação de cenários de negócio
- Documentação completa do desenho multiagente
 
---
 
## 📚 Recursos e Documentação
 
### 🔗 Links de Referência:
- [Documentação do Microsoft Fabric](https://learn.microsoft.com/es-es/fabric/)
- [Azure AI Foundry](https://learn.microsoft.com/es-es/azure/ai-foundry/)
- [Power BI Embedded](https://learn.microsoft.com/es-es/power-bi/)
- [Azure Cosmos DB](https://learn.microsoft.com/es-es/azure/cosmos-db/)
 
### 🎯 Próximos Passos:
Com esses desafios concluídos, você terá construído uma solução completa que vai **do insight à decisão**, implementando:
- ✅ Pipeline de dados completo com arquitetura medallion
- ✅ Modelo semântico robusto para análise de negócios
- ✅ Agentes conversacionais para democratização de dados
- ✅ Orquestração inteligente e dinâmica para automação de processos