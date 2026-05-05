# 🏆 Desafio 5: Orquestração de Agentes - Design de Fluxo Multiagente 

📖 Cenário  

No contexto da IA aplicada à automação de processos de negócio, a orquestração de agentes permite escalar a tomada de decisões por meio da colaboração entre agentes com papéis diferenciados. A Contoso, como empresa de varejo e financeira, precisa combinar múltiplas fontes de informação para tomar decisões estratégicas informadas. 

Neste desafio, você projetará e construirá um fluxo multiagente que coordene dados internos de **vendas**, perfis de crédito de **clientes** e **inteligência de mercado** externa para automatizar análises complexas de negócio e fornecer recomendações estratégicas baseadas em dados. 

Em cenários produtivos, naturalmente esses fluxos podem envolver não apenas análise, mas também ações e execuções; porém, para este cenário, o objetivo é conceituar a segmentação de papéis em fluxos agentivos, para que posteriormente possam ser implementadas soluções mais avançadas

---

### 🎯 Sua Missão  
Ao concluir este desafio, você será capaz de:  

✅ Definir **quatro agentes especializados** com responsabilidades claras (Vendas, Crédito, Pesquisa de Mercado, Sintetizador) além de um **agente orquestrador (Router)** que direcione o fluxo de forma inteligente.  
✅ Projetar um **fluxo orquestrado** colaborativo usando Foundry Workflows com ramificações **condicionais** e tratamento de **erros**.  
✅ Integrar fontes de dados internas **(Microsoft Fabric)** com ferramentas externas, por exemplo: **(Bing Search)**.  
✅ Validar cenários de negócio complexos que exigem síntese de múltiplos domínios de informação.  
✅ Documentar o desenho para sua **replicabilidade e escalabilidade**.  

---

### 🔗 Contexto
Este desafio é construído sobre o trabalho dos desafios anteriores:

**Desafios 1-3:** Você implementou o pipeline de dados

*Bronze*: Ingestão a partir do Cosmos DB  
*Silver*: Transformações com produtos, credit e transações  
*Gold*: Tabelas curadas, modeladas ou unificadas, exemplo: business_operations e segmentações de ML  

**Desafio 4:** Você configurou agentes especializados 

**Sales Operations Analyst (Contoso-Virtual/Sales-Analyst)** conectado a **business_operations** (ou às suas próprias tabelas)

**Desafio 5 (este)**: Projete a orquestração de agentes com inteligência externa
Como eles colaboram para responder perguntas complexas como:

```
"Como se comparam nossas vendas de produtos premium com as tendências do mercado?"
"Quais produtos recomendar a clientes de perfil alto considerando as tendências atuais da indústria?"
"Nossos preços na categoria Electronics são competitivos segundo o mercado?"
```

Essas perguntas exigem:

**Agente de Vendas** → Dados internos de transações e produtos  
**Agente de Crédito** → Perfis e capacidade de clientes  
**Agente de Pesquisa** → Tendências, concorrência, benchmarks externos  
**Orquestração** → Coordenar e sintetizar todas as fontes  

---

### 📊 Dados e Ferramentas Disponíveis

**Agentes Especializados**

0. Router Agent (a ser criado) 

- *Fonte*: Nenhuma (analisa o texto da query) 
- *Expertise*: Classificação da intenção do usuário e roteamento inteligente 
- *Comportamento*: Trabalha em silêncio, emite **uma única tag** — `[SALES]`, `[MARKET]`, `[CREDIT]` para consultas de um único domínio, ou `[CROSS]` quando a pergunta envolve múltiplos domínios 
- *Queries típicas*: Qualquer consulta do usuário — ele decide quem responde 

1. Sales Operations Agent (existente, criado no desafio anterior)

- *Fonte*: **gold.business_operations** (Microsoft Fabric). Ou a tabela que você usou para o seu Data Agent 
- Expertise: Revenue, canais, produtos, MSI, segmentação de tickets 
- Queries típicas: "¿Cuál es el revenue por canal?", "¿Top productos vendidos?" 

2. Credit Risk Agent (a ser criado)

- *Fonte*: **gold.credit_scores** (Microsoft Fabric) 
- *Expertise*: Perfis de crédito (Baixo, Médio, Alto, Premium), scores, capacidade de pagamento 
- *Queries típicas*: "¿Cuántos clientes Premium?", "¿Score promedio por perfil?" 

3. Market Research Agent (a ser criado)

- *Fonte*: **Bing Search API**
- *Expertise*: Tendências da indústria, análise da concorrência, benchmarks de mercado 
- *Queries típicas*: "¿Tendencias en productos premium?", "¿Precios de competidores?" 

4. Strategy Advisor Agent (a ser criado)

- Sintetiza e consolida as informações dos demais agentes para apresentá-las ao usuário

**Ferramentas de Orquestração**

Foundry Workflows

- Visual designer para construção de fluxos 
- Invoke Agent nodes para chamar agentes especializados 
- Set Variable nodes para passar contexto entre agentes 
- If/Else nodes para ramificações condicionais 
- YAML e Code views para controle avançado 


## 🚀 Passo 1: Definir Papéis e Responsabilidades dos Agentes  
💡 *Por quê?* Uma separação clara de responsabilidades reduz o acoplamento e facilita a escalabilidade.  

**Agente de Vendas Internas (Sales Operations Analyst)**  
  - Analisa dados transacionais históricos 
  - Calcula métricas de revenue, volume, canais 
  - Identifica padrões de compra e produtos top 
  - Segmenta por tickets (Low, Medium, High) 

*Entradas*: Queries sobre vendas, produtos, canais, MSI 
*Saídas*: Métricas quantitativas, rankings, distribuições 

**Agente de Risco de Crédito (Credit Risk Analyst)**  
  - Avalia perfis de crédito dos clientes 
  - Identifica segmentos por capacidade de pagamento 
  - Analisa distribuição de scores e ratios financeiros 
  - Recomenda estratégias de financiamento  

**Agente de Pesquisa de Mercado (Market Research Analyst)**
  - Busca tendências atuais da indústria 
  - Obtém informações de concorrentes 
  - Encontra benchmarks e padrões de mercado 
  - Fornece contexto externo para decisões internas 

*Entradas*: Keywords sobre produtos, categorias, mercados 
*Saídas*: Insights de tendências, dados da concorrência, contexto de mercado 

**Agente Sintetizador (Strategy Advisor)**
  - Recebe outputs dos agentes especializados 
  - Identifica correlações entre dados internos e externos 
  - Sintetiza informações em recomendações acionáveis 
  - Apresenta insights de forma estruturada 

*Entradas*: Outputs dos 3 agentes especializados + query original do usuário 
*Saídas*: Análise integrada com recomendações estratégicas 

✅ **Resultado esperado:** Definição clara dos 4 papéis com suas entradas, saídas e critérios de sucesso.

---

### 🚀 Passo 2: Projetar a Orquestração e o Fluxo Colaborativo  
💡 *Por quê?* A orquestração define o “quem, quando e como” entre agentes e garante rastreabilidade.  

**Arquitetura do Fluxo**  

1️⃣ Gatilho de início: Query do usuário via Playground  
2️⃣ **Router Agent**: analisa a consulta e emite **uma única tag** — `[SALES]`, `[MARKET]`, `[CREDIT]`, ou `[CROSS]`  
3️⃣ Roteamento condicional:  

![Multi](/img/multi-flujo.png)  

4️⃣ Condições e ramificações:  
- Se o router emitir `[SALES]` → apenas o Sales Agent responde  
- Se o router emitir `[MARKET]` → apenas o Market Research Agent responde  
- Se o router emitir `[CREDIT]` → apenas o Credit Risk Agent responde  
- Se o router emitir `[CROSS]` → os 3 agentes especializados são executados em sequência e o Strategy Advisor sintetiza  

4️⃣ Retroalimentação:  
- Variáveis passadas entre nós mantêm o contexto  
- Strategy Advisor recebe todos os insights anteriores  
- Logs e traces permitem debugging do fluxo  

5️⃣ Rastreabilidade:  
- Cada nó registra sua execução
- Variáveis são salvas em cada passo
- Workflow execution ID para acompanhamento completo

✅ **Resultado esperado:** Diagrama visual do fluxo no Foundry Workflows com nós conectados e condições claras.

---

### 🚀 Passo 3: Definir o Contrato de Mensagens e Esquemas de Dados  
💡  *Por quê?* Configurações claras garantem que cada agente cumpra seu papel de forma efetiva.  

**Configuração por Agente**

**Router Agent (Orquestrador):**

- *Model*: **gpt-4o**
- *Tools*: **None** (analisa o texto da query)
- *Instructions*: [Classifica a intenção do usuário e emite tags de roteamento internos]

**Sales Operations Analyst:**

- *Model*: **gpt-4o**
- *Tools*: **Fabric Data Agent [Contoso-Agent Sales] (business_operations)**
- *Instructions*: [Foco em métricas de vendas internas]

**Credit Risk Analyst:**

- *Model*: **gpt-4o**
- *Tools*: **Fabric Data Agent [Contoso-Agent Credit Risk] (credit_score)**
- *Instructions*: [Foco em perfis e capacidade de crédito]

**Market Research Analyst:**

- *Model*: **gpt-4o**
- *Tools*: **Bing Search**
- *Instructions*: [Foco em tendências e informação pública]

**Strategy Advisor:**

- *Model*: **gpt-4o**
- *Tools*: **None** (recebe variáveis dos outros agentes)
- *Instructions*: [Sintetiza e gera recomendações estratégicas]

✅ **Resultado esperado:** Especificação de configuração para cada agente com modelo, tools e instructions.

---

## Passo 4: Construir o Workflow no Foundry
💡 ¿Por qué? A implementação visual facilita debugging e compreensão do fluxo.

**Componentes do Workflow**

**Start Node:**
* Captura a mensagem do usuário em duas variáveis: `Local.UserQuestion` (texto plano) e `Local.LatestMessage` (mensagem formatada)

**Router Agent Node:**
* Invoca o Router Agent com a query do usuário
* Emite uma única tag de roteamento em silêncio (`Local.LatestMessage`)
* Não envia resposta ao usuário (`autoSend: false`)

**ConditionGroup Node:**
* Avalia a tag emitida pelo Router Agent
* Se a tag for `[SALES]`, `[MARKET]` ou `[CREDIT]` → invoca apenas esse agente e termina
* Se nenhuma condição individual for verdadeira (`[CROSS]`) → cai no `elseActions`

**elseActions (caso cross-domain):**
* Restaura `Local.LatestMessage` a partir de `Local.UserQuestion`
* Invoca Sales, Market Research e Credit Risk em sequência
* Cada agente acumula sua resposta em `Local.LatestMessage`
* O Strategy Advisor recebe o histórico completo e sintetiza

**End Node:**
* Retorna resposta final ao usuário
* Fecha o workflow

✅ Resultado esperado: Workflow funcional no Foundry com todos os nós conectados e configurados.

---

## 🚀 Passo 5: Validação de Cenários de Negócio 
💡 *Por quê?* A validação confirma que o desenho resolve casos reais.  

**Cenários de Teste**

**Cenário 1: Análise Comparativa de Mercado**
- **Query**: "¿Cómo se comparan nuestras ventas de iPhone vs las tendencias del mercado?"

Fluxo esperado:
Sales Analyst → Vendas internas de iPhone  
Market Research → Tendências de mercado do iPhone (Bing Search)  
Strategy Advisor → Comparação e análise de gaps  

✅ Resultado esperado: Insight sobre posição de mercado com recomendações

**Cenário 2: Recomendação Segmentada**

- **Query**: "¿Qué productos premium recomendar a clientes de perfil Alto considerando tendencias actuales?"

Fluxo esperado:

Credit Risk → Identifica clientes de perfil Alto  
Sales Analyst → Produtos premium que eles compram  
Market Research → Tendências em produtos premium  
Strategy Advisor → Recomendações baseadas nas 3 fontes  

Resultado esperado: Lista de produtos recomendados com justificativa

**Cenário 3: Pricing Competitivo**

- **Query**: "¿Nuestros precios en laptops son competitivos según el mercado?"

Fluxo esperado:

Sales Analyst → Preços atuais de laptops  
Market Research → Preços de concorrentes (Bing Search)  
Strategy Advisor → Análise de gap e recomendações  

Resultado esperado: Análise de competitividade com sugestões de ajuste

✅ Resultado esperado: Execução bem-sucedida dos 3 cenários com screenshots e análise dos resultados.

---

## 🚀 Passo 6: Documentação e Melhoria Contínua 
💡 *Por quê?* O que não se mede, não se melhora.  

Elementos a Documentar

**Arquitetura:**

- Diagrama completo do workflow
- Descrição de cada agente e seu papel
- Decisões de design (por que este fluxo)

**Configurações:**

- System prompts de cada agente
- Tools configurados
- Variáveis e seu propósito

**Resultados:**

- Screenshots de execuções bem-sucedidas
- Análise de cenários de teste
- Métricas de performance (tempo de resposta)

**Aprendizados:**

- O que funcionou bem
- O que você melhoraria
- Próximos passos para produção

✅ Resultado esperado: Documento completo com arquitetura, configurações, resultados e aprendizados.

### 🏁 Pontos de Controle Finais

✅ Os 5 agentes (Router + 3 especializados + Advisor) estão definidos com seus papéis e responsabilidades claros?  
✅ O Router Agent emite tags corretas para diferentes tipos de consulta?  
✅ O fluxo ativa apenas os agentes necessários conforme o tipo de consulta?  
✅ O Strategy Advisor só é invocado quando 2 ou mais agentes participaram?  
✅ Os 3 agentes especializados estão configurados no Foundry com tools apropriados?  
✅ O workflow está construído visualmente no Foundry Workflows?  
✅ Os 3 cenários de negócio foram testados com sucesso?  
✅ A documentação inclui diagramas, configurações e resultados?  

---

### 💡 Dicas e Recomendações

**Design:**
- Comece simples (2 agentes) e expanda gradualmente  
- Use variáveis descritivas (sales_insights, market_data)  
- Documente decisões enquanto projeta  

**Implementação:**
- Teste cada agente individualmente antes de integrar  
- Use o Playground para validar prompts  
- Revise traces para debugging  

**Validação:**
- Escolha queries realistas que exijam múltiplos agentes  
- Analise se as respostas integram bem as fontes  
- Identifique gaps ou melhorias  

## 📝 Documentação  

-  [Build a Workflow in Microsoft AI Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/overview)
-  [Tools in Foundry Agent Service](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/concepts/workflow)
-  [Grounding with Bing Search](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools-classic/bing-grounding?view=foundry-classic)
-  [Orchestrating Multi-Agent Conversations with Microsoft Foundry Workflows](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/orchestrating-multi-agent-conversations-with-microsoft-foundry-workflows/4472329)
-  [Multi-Agent Orchestration Patterns](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/building-a-digital-workforce-with-multi-agents-in-azure-ai-foundry-agent-service/4414671)
-  [Agent Framework Examples](https://github.com/microsoft/agent-framework)
-  [Building No-Code Agentic Workflows with Microsoft Foundry](https://medium.com/data-science-collective/building-no-code-agentic-workflows-with-microsoft-foundry-52ad377ad644)
  