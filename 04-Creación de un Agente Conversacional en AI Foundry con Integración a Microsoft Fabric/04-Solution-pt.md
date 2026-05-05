# Solução Desafio 04 - Criação de um Agente Conversacional no AI Foundry com Integração ao Microsoft Fabric


Guia passo a passo para habilitar um agente conversacional no AI Foundry integrado com o Data Agent do Microsoft Fabric

### Objetivo 🎯
- Projetar um agente conversacional no AI Foundry integrado ao Microsoft Fabric.  
- Conectar o agente a um Data Agent associado ao modelo semântico/tabelas Gold.  
- Configurar intents e prompts orientados a perguntas reais de negócio.  
- Validar que o agente responda em linguagem natural, sem mostrar código nem sintaxe técnica.  
- (Opcional) Publicar o agente para uso de analistas dentro do Copilot, Power BI ou AI Foundry.  


---

## Requisitos prévios

- Modelo semântico, Data Agent e Dashboard de valor (Gold) - (ver `03-Solution.md`).


## Passos

### 1 - Criar o Agente no Foundry

1. Para criar um agente a partir do Foundry, precisamos ter acesso a um recurso do Foundry dentro de um projeto (pré-requisitos). Acesse o recurso do Azure AI Foundry a partir da assinatura do Azure ou faça login com seu usuário autorizado em [AI Foundry](https://ai.azure.com/). Ative a nova experiência do Foundry, pois ela oferece um ambiente mais simples e intuitivo.


 ![New Foundry](/img/new_foundry.png)

2. Selecione seu projeto → no menu de boas-vindas → **Start building** → **Create agent** → em **Agent Name**, atribua um nome descritivo e único, por exemplo: `Contoso-Virtual-Analyst`.


![Foundry-Start](/img/foundry-start.png)

3. Dentro do menu do agente → **Playground** → selecione o modelo que criamos como parte dos pré-requisitos (**gpt-4o**) e clique em **Save**. Outros modelos conversacionais podem ser usados, se já estiverem habilitados no recurso.

✅ **Resultado esperado:** O agente está criado e configurado para interação conversacional.  


![Foundry-Agent](/img/foundry-agent.png)


### 2 - Conectar o Agente ao Data Agent do Fabric

1️. Na seção **Tools** (também pode ser pelo Knowledge) → **+ Add a new tool** → **Fabric Data Agent** → **Add tool**, configure o **Data Agent** criado no desafio anterior do Fabric.


![Foundry-Agent](/img/fabric-tool.png)


2. Na janela pop-up, devemos configurar uma nova conexão do tipo Fabric Data Agent. Para isso, precisamos preencher as seguintes informações:

   - **Name**: Um nome descritivo para a conexão
   - **Workspace ID**: Aqui vai o ID do Workspace onde o Data Agent está hospedado. Com o Data Agent aberto, corresponde à sequência alfanumérica que está no início da URL da web (1)
   - **Artifact ID**: Aqui vai o ID do artefato (Data Agent). Com o Data Agent aberto, corresponde à segunda sequência alfanumérica que está na URL da web (2)
  
Imagem de referência para validar `Workspace ID` e `Artifact ID` do Data Agent


![Foundry-Agent](/img/workspace-artifact.png)
     

3. Verifique novamente no Fabric se o Data Agent está vinculado ao **modelo semântico Gold** ou às tabelas que precisamos para que ele realize seu trabalho, o que inclui tabelas como:  
   - `gold.business_operations`  
   - `gold.credit_score`
   - `modelo semantico`

4. Salve a configuração da conexão.  

✅ **Resultado esperado:** O agente do Foundry está vinculado ao Data Agent.


![Foundry-Agent](/img/fabric-tools.png)


### 3 - Definir Intents e Prompts Orientativos  

1. Adicione instruções que permitam ao agente entender o que ele deve fazer. Essa configuração, muitas vezes chamada de **System Prompt**, permite ao agente entender como deve atuar, que tarefas deve executar e como deve formatar as respostas (tom etc.). Neste caso, devemos orientá-lo para o Data Agent do Fabric. 

Em **Instructions**, passamos a inserir nossas instruções de forma clara, concisa e estruturada, e salvamos novamente a configuração do **Agente**. Aqui está um exemplo:

```
# Rol y Contexto
Eres un asistente experto en análisis operacional que tiene acceso a 
datos de transacciones y productos de la empresa Contoso.

# Fuente de Datos
Tienes acceso a datos actualizados del Data Agent de Fabric llamado 'Contoso Data Agent' que contiene datos de:
- business_operations (tablas de transacciones y productos)

# Comportamiento Esperado
1. Siempre consulta los datos antes de responder preguntas factuales
2. Si no encuentras información en los datos, indícalo claramente
3. Cita las fuentes específicas cuando uses información de los datos
4. Mantén un tono [profesional y técnico según necesites]

# Restricciones
- No inventes información que no esté en los datos
- Siempre valida fechas y números antes de reportarlos

# Formato de Respuesta
- Usa tablas para datos numéricos
- Incluye contexto cuando sea relevante
- Sé conciso pero completo
```


![Foundry-Agent](/img/sys-prompt.png)


### 4 - Definir Intents e Prompts Orientativos  

1. Agora tente criar `intents` ou consultas que reflitam as necessidades analíticas da Contoso. Para isso, abrimos o ícone de engrenagem ⚙️ no canto superior direito da janela do chat.
   No menu, podemos preencher o seguinte:

   - **Display name**: para que os usuários identifiquem o agente com um nome familiar
   - **Description**: Uma descrição opcional do que o agente faz e como usá-lo
   - **Starter Prompts**: Exemplos de intents orientativos para o agente sobre o contexto do agente. Eles aparecerão como sugestões para o usuário.

```
 “¿Qué productos tienen mayor tasa de devolución?”  
 “¿Qué categoría tiene más productos valiosos?”  
 “¿Cuál es el valor comercial total por marca?” 
```

Salvamos a configuração

![Foundry-Agent](/img/starter-prompts.png)


### 5 - Validar o Agente com Perguntas Reais 
     
1. Na barra de chat, vamos proceder a fazer perguntas para validar as respostas geradas. Podemos selecionar perguntas da lista de `starter prompts` ou usar nossas próprias consultas.


✅ **Resultado esperado:** O agente entende as perguntas de negócio e responde de forma contextual. Se faltar precisão, podemos ajustar as instruções e testar novamente.


![Foundry-Agent](/img/output-prompt.png)


### 6 - Publicar e Habilitar o Agente  

Quando o nosso agente estiver validado e demonstrar um comportamento adequado e preciso, o próximo passo é publicá-lo para que possa ser exportado ou publicado nos diferentes canais disponíveis (Teams, M365 Copilot etc.)

1. No canto superior direito, encontraremos um botão **Publish**. Isso permitirá habilitar o agente nos canais do Teams e M365 Copilot, de onde nossos usuários corporativos poderão consumi-lo sem necessidade de acesso direto ao Foundry ou ao Microsoft Fabric. A partir daí, também podemos configurar o acesso e os grupos que o utilizarão.
  
![Foundry-Agent](/img/publish.png)


2. (Opcional) Se tivermos acesso, podemos avançar e publicar o agente para o M365 e o Teams. Para isso, precisamos ter habilitado o serviço **Azure Bot Service**, que atua como middleware entre o agente e a camada de frontend (Teams, 365).
   Para isso, precisamos concluir as configurações solicitadas no menu e prosseguir no ecossistema do 365 para sua respectiva validação.


![Foundry-Agent](/img/optional-365.png)

3. Depois de publicado localmente no Foundry, podemos ver o agente em `Preview`, que é basicamente uma visualização simulada a partir de um aplicativo final.

![Foundry-Agent](/img/preview.png)

![Foundry-Agent](/img/preview2.png)


✅ **Resultado esperado:** O agente está ativo e disponível para consultas em linguagem natural.