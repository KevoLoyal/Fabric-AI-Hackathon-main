
# 🏆 Desafio 4: Criação de um Agente Conversacional no AI Foundry com Integração ao Microsoft Fabric 🤖  

📖 Cenário  
A Contoso deseja que seus **analistas possam interagir com os dados usando linguagem natural**, sem necessidade de conhecimentos técnicos em T-SQL ou modelagem.  
O objetivo é criar um **agente no Azure AI Foundry** que consuma o **modelo semântico conectado ao Fabric por meio de um Data Agent**, permitindo obter respostas claras, compreensíveis e baseadas em dados confiáveis. 

Os agentes de dados do Fabric têm a possibilidade de ser expostos para outras ferramentas de IA da Microsoft, como Copilot Studio ou AI Foundry. Dessa forma, podemos orquestrar fluxos multiagente em que, dependendo da tarefa, um agente especializado assume o controle do processo (especialistas) e outros coordenam o trabalho (supervisores). Neste desafio, vamos expor nosso data agent do Fabric dentro de outro agente no Foundry, que será impulsionado por um LLM da OpenAI (GPT). Esse LLM atuará como um roteador para o data agent do Fabric, recuperando as informações dos dados e devolvendo-as ao usuário. 

A imagem a seguir mostra como é o fluxo de trabalho deste cenário.

 ![Foundry-Fabric](/img/foundry-data-agent.png)

---

Certifique-se de ter concluído - Modelo semântico, Data Agent e Dashboard de valor (Gold) - (ver `03-Solution.md`). Também, no nível de configurações, é importante validar os seguintes [requisitos](https://learn.microsoft.com/en-us/fabric/data-science/data-agent-foundry#prerequisites)

### 🎯 Sua Missão  
Ao concluir este desafio, você será capaz de:  

✅ Projetar um **agente conversacional no AI Foundry** integrado ao Microsoft Fabric.  
✅ Conectar o agente a um **Data Agent** associado ao modelo semântico Gold.  
✅ Configurar intents e prompts orientados a perguntas reais de negócio.  
✅ Validar que o agente responda em **linguagem natural**, sem exibir código nem sintaxe técnica.  
✅ Publicar o agente para uso de analistas dentro do **Copilot, Power BI ou AI Foundry**.  

---

## 🚀 Passo 1: Criar o Agente no AI Foundry  
💡 *Por quê?* O agente é a interface conversacional que permitirá aos analistas interagir diretamente com os dados do modelo semântico. A partir do AI Foundry, também temos a possibilidade de orquestrar fluxos multiagente, nos quais podemos expor nossos agentes do Fabric e combiná-los com outros agentes que executam tarefas diferentes, permitindo resolver cenários complexos e multidisciplinares.

1️⃣ Acesse seu recurso do **Azure AI Foundry** a partir da assinatura do Azure ou faça login com seu usuário autorizado em [AI Foundry](https://ai.azure.com/). De preferência, ative a nova experiência do Foundry.

![New Foundry](/img/new_foundry.png)


2️⃣ Selecione seu projeto → no menu de boas-vindas → **Start building** → **Create agent** → em **Agent Name**, atribua um nome descritivo e único, por exemplo: `Contoso-Virtual-Analyst`.  

![Foundry](/img/foundry-start.png)

3️⃣ Dentro do menu do agente → **Playground** → selecione o modelo que criamos como parte dos pré-requisitos (**gpt-4o**)

![Foundry](/img/foundry-agent.png)

✅ **Resultado esperado:** O agente está criado e configurado para interação conversacional.  



---

## 🚀 Passo 2: Conectar o Agente ao Data Agent do Fabric  
💡 *Por quê?* O Data Agent é o elo entre o AI Foundry e os dados governados no Microsoft Fabric.  

1️⃣ Na seção **Tools** ou **Knowledge** do agente, configure o **Data Agent** criado no desafio anterior do Fabric.  
2️⃣ Verifique se o Data Agent está vinculado ao **modelo semântico Gold** ou às tabelas que precisamos para que realize seu trabalho, o que inclui tabelas como:  
   - `gold.bsuiness_operations`  
   - `gold.credit_score`

3️⃣ Salve a configuração da conexão.  

✅ **Resultado esperado:** O agente pode acessar o modelo semântico e consultar os dados de forma controlada.  

---

## 🚀 Passo 3: Configurar o Comportamento do Agente  
💡 *Por quê?* Controlar o tom e o tipo de resposta garante uma experiência clara e livre de linguagem técnica.  

1️⃣ Na seção de **Instructions** das respostas, selecione:  
   - “Respostas em **linguagem natural**”.  
   - “**Ocultar código e sintaxe técnica**”.
   - “Não mostrar código nem sintaxe técnica **(como T-SQL)**”.
2️⃣ Ative a opção de **respostas explicativas**, para que o agente justifique suas respostas com frases como:  
> “Segundo os dados do modelo, o score médio no segmento alto é de 87 pontos.”  

✅ **Resultado esperado:** O agente comunica os achados em linguagem natural, sem mostrar código ou consultas.  

---

## 🚀 Passo 4: Definir Intents e Prompts Orientativos  
💡 *Por quê?* Os intents ajudam a treinar o agente para compreender as perguntas frequentes do negócio.  

1️⃣ Crie intents que reflitam as necessidades analíticas da Contoso.  
2️⃣ Exemplos sugeridos (adapte ao contexto dos dados):  

| **Intent / Tema** | **Prompt orientativo (pergunta do analista)** |
|--------------------|-----------------------------------------------|
| score_por_segmento | “Qual é o score médio por segmento?” |
| productos_con_devolucion | “Quais produtos têm maior taxa de devolução?” |
| productos_valiosos_por_categoria | “Qual categoria tem mais produtos valiosos?” |
| ventas_totales_por_marca | “Qual é o valor comercial total por marca?” |

✅ **Resultado esperado:** O agente entende as perguntas de negócio e responde de forma contextual.  

---

## 🚀 Passo 5: Validar o Agente com Perguntas Reais  
💡 *Por quê?* A validação permite confirmar que o agente compreende corretamente as consultas e correlações entre tabelas.  

1️⃣ Teste diretamente no **AI Foundry** com perguntas como as seguintes (ou conforme o cenário trabalhado no modelo de dados) 
   - “Que marca tem mais produtos disponíveis?”  
   - “Qual é a tendência mensal de risco?”  
   - “Qual perfil de produto gera mais receita?”

2️⃣ Verifique se as respostas:  
   - São **claras e sem código**.  
   - Entendem correlações entre entidades (por exemplo, *credit score*, *transactions* e *products*).  
   - Vêm de métricas do **modelo semântico conectado**.  

✅ **Resultado esperado:** O agente responde perguntas complexas de forma coerente e baseada nos dados do modelo.  

---

## 🚀 Passo 6: Publicar e Habilitar o Agente  
💡 *Por quê?* Publicar o agente o torna acessível para analistas e equipes de negócio dentro do ambiente do Fabric.  

1️⃣ Publique o agente a partir do **AI Foundry** .  
2️⃣ (Opcional) Se você tiver permissões de admin no seu tenant do M365, pode habilitá-lo para ser usado a partir do **Microsoft 365 Copilot, ou Microsoft Teams**.
3️⃣ Confirme que o agente está publicado. Você pode testá-lo em modo de teste para ver como ficaria dentro de um aplicativo.

✅ **Resultado esperado:** O agente está ativo e disponível para consultas em linguagem natural.  

---

## 🏁 Pontos de Controle Finais  

✅ O agente no AI Foundry foi criado e configurado corretamente?  
✅ Está conectado ao Data Agent e ao modelo semântico/tabelas Gold?  
✅ Foram definidos intents e prompts alinhados às necessidades do negócio?  
✅ O agente responde em linguagem natural sem mostrar código?  
✅ Está publicado e disponível?  

---

## 📝 Documentação  

-  [Configuração do Agente no AI Foundry](https://learn.microsoft.com/es-es/azure/ai-foundry/agents/environment-setup)  
-  [Conexão com o Data Agent do Fabric](https://learn.microsoft.com/es-es/azure/ai-foundry/agents/how-to/tools/fabric?pivots=portal)  
-  [Referência oficial - Criação de Agentes de Dados no Fabric](https://learn.microsoft.com/en-us/fabric/data-science/how-to-create-data-agent)  
  