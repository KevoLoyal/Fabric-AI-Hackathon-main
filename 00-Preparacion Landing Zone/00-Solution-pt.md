# Solução Desafio 00 - Preparação da Landing Zone e Conexão de Dados (Microsoft Fabric)

Este documento descreve, passo a passo, como concluir o *Desafio 0*: criar a zona de aterrissagem (landing zone) no Microsoft Fabric, provisionar o Azure Cosmos DB com os JSON fornecidos e conectar ambas as plataformas.

Objetivos
- Provisionar o Azure Cosmos DB (NoSQL) e carregar os datasets JSON.
- Criar um workspace e uma Lakehouse no Microsoft Fabric com a estrutura Bronze / Silver / Gold.
- Conectar o Fabric ao Cosmos DB e verificar a ingestão inicial.

Requisitos prévios
- Permissões para criar recursos no Azure.
- Arquivos JSON (por exemplo `creditScore.json`, `products.json`, `transactions.json`).

Resultado esperado
- Cosmos DB com contêineres contendo os JSON.
- Workspace no Fabric conectado a uma Capacity.
- Lakehouse `Contoso_Lakehouse` com schemas: `bronze`, `silver`, `gold`.

---

## 1 - Criar o Azure Cosmos DB (NoSQL)

**NOTA**  

Por conveniência, recomenda-se utilizar o template de implantação de artefatos do Azure [DeployToAzure](./DeployToAzure.md), que permite criar o Cosmos DB NoSQL e outros artefatos do Azure de forma automatizada. Caso utilize o template, omitiremos este primeiro passo; caso contrário, seguiremos o passo 1 manualmente.

1. Acesse o portal do Azure.
2. No campo de busca do Azure, digite **Azure Cosmos DB** → +Criar → selecionar a API **Azure Cosmos DB for NoSQL** → Criar.
3. Preencha:
   - Workload Type: `Development/Testing`
   - Subscription: `sua assinatura do Azure`
   - Resource Group: `seu grupo de recursos`
   - Account name: `nome da conta a ser criada`
   - Location: `sua região padrão`
   - Availability Zones: `Disable`
   - Capacity mode: `Serverless`
   - Global Distribution: `Disable`
   - Networking: `All networks`
   - Backup Policy: `deixar o valor padrão`
   - Key-based authentication: `Enable`
   - Encryption: `Service-managed key`
   - Tags: `vazias`

Review + create
   
![New Foundry](/img/cosmos1.png)

4. Depois que a conta do Cosmos DB for criada, em *Data Explorer* crie o banco de dados com a opção +New → New Database → Database id → `<nome>`, por exemplo DB-Hackathon

![New Foundry](/img/cosmos2.png)  	
  
5. Crie um contêiner para cada conjunto de dados (por exemplo `products`, `credit`, `transactions`). No mesmo botão +New, selecione → New container → selecione:
- Database id: `Use existing` e selecione a base que acabamos de criar
- Container id: `products`
- Partition key: `/id`
- OK

**Repetimos este processo para os outros conjuntos de dados de `credit` e `transactions`**
   
![New Foundry](/img/cosmos3.png)  	
   
6. Carregue os arquivos JSON (manualmente pelo Data Explorer ou usando o Azure Data Factory para cargas repetíveis). No Data Explorer, repetindo este mesmo processo para cada contêiner criado, selecione a opção → `Item` → `Upload Item` → selecione o arquivo JSON correspondente para cada contêiner → `Upload`

![New Foundry](/img/cosmos4.png)

7. Verificação
- Em *Data Explorer* é possível ver documentos JSON no contêiner.
  
![New Foundry](/img/cosmos5.png)

---

## 2 - Criar Workspace no Microsoft Fabric

1. Abra o Microsoft Fabric → na barra lateral esquerda selecione `Workspaces` → `+New workspace` e crie um workspace chamado `ContosoData-Fabric` ou o nome que preferir.
2. Depois de criado, procure a opção `Workspace settings` → `Workspace type` → `Edit` e associe o novo Workspace à Capacity do Fabric (esta etapa não é necessária se sua organização já tiver um Workspace atribuído a uma capacity que você possa usar).
3. Adicione seu usuário como Admin/Contributor no Workspace

Verificação
- O workspace aparece na lista de espaços de trabalho e você tem as permissões adequadas.

![Fabric](/img/wscreate.png)

---

## 3 - Criar Lakehouse e definir camadas

1. No workspace, crie uma Lakehouse `Contoso_Lakehouse` e marque o `check` para usar schemas.
2. Depois que a Lakehouse estiver criada, crie novos schemas com a seguinte convenção:
	- `bronze*` para dados brutos
	- `silver*` para dados limpos
	- `gold*` para dados curados/consumo

Verificação
- Os schemas estão visíveis no navegador da Lakehouse.

![Schemas](/img/schemas.png)



---

## 4 - Conectar o Fabric ao Azure Cosmos DB

1. Dentro do Workspace criado, selecione `+New item` → Dataflow Gen2
2. No novo DataFlow Gen2 → Get data → More → procure por Azure Cosmos DBv2.
    
![Cosmos](/img/cosmoscon.png)

3. Na configuração da conexão, vamos adicionar:
   - Cosmos DB Endpoint: Vamos informar o endpoint da conta do Cosmos DB, que pode ser obtido no recurso do Cosmos DB na opção Settings → Keys
   - Connection name: O nome da nossa conexão
   - Authentication kind: Account key
   - Acount key: A PRIMARY KEY da conta do Cosmos DB. Ela pode ser obtida no recurso do Cosmos DB na opção `Settings` → `Keys` → `PRIMARY KEY` 

![Cosmos](/img/cosmoscon1.png)

4. Teste e salve a conexão.

Verificação
- A conexão é testada e você consegue ver os containers/collections disponíveis.

---