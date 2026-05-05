# Solução Desafio 01 - Ingestão do Cosmos DB para o Microsoft Fabric (Camada Bronze) + Limpeza Básica

Guia passo a passo para ingerir dados do Azure Cosmos DB para a camada Bronze da Lakehouse no Microsoft Fabric, aplicar limpeza inicial e validar o resultado.

Objetivo
- Ingerir dados do Cosmos DB usando Dataflow Gen2 ou pipeline e aplicar transformações básicas (nulos, colunas, formatos).

Requisitos prévios
- Conexão com o Cosmos DB a partir do Fabric (ver `00-Solution.md`).

## Passos

### 1 - Criar Dataflow Gen2

1. No Fabric, Data → New → Dataflow Gen2.
2. Selecione **Azure Cosmos DB** como fonte.
3. Preencha o endpoint e a key (ou selecione a conexão já criada).
4. Selecione a coleção/contêiner que contém `products`, `creditScore`, `transactions`.

### 2 - Projetar transformações básicas

Dentro do designer do Dataflow:
- Remova colunas desnecessárias.  
- Normalize formatos: converter datas, strings (trim, lower), normalizar decimais, alterar tipos de dados.  
- Substitua ou marque valores nulos (por exemplo, `unknown` ou valores padrão).  
- Filtre registros corrompidos ou incompletos (se aplicável).  

![DF](/img/dfgen2.png)

Dica: adicione etapas intermediárias de validação e use amostras pequenas para testar as transformações.

### 3 - Destino: Lakehouse Bronze

1. Configure o sink/destino como a Lakehouse `Contoso_Lakehouse` → buscar schema → criar tabela. Exemplo `[bronze].[sales]`. Lembre-se de ativar a opção de **advance options**: `Navigate using full hierarchy`, que permite navegar pela estrutura hierárquica de `schemas/tabelas`. Repita com os outros datasets
2. Execute o Dataflow em modo de validação e depois em produção.

 ![DF](/img/dfgen22.png)
 ![DF](/img/dfgen23.png)  
 ![DF](/img/dfgen24.png)  
 ![DF](/img/dfgen25.png)  
 ![DF](/img/dfgen26.png)  
 
### 4 - Verificar e documentar

1. Abra a Lakehouse e revise cada uma das tabelas. Exemplo `[bronze].[sales]`.
2. Verifique a contagem de registros, as colunas esperadas e os formatos das colunas (datas, numéricos).
3. Salve um registro da execução (logs) e uma captura de tela.

 ![Tablas bronze](/img/tablas_bronze.png)

 ---

## Validações-chave
- Validação origem vs destino
- Não há colunas que devam ter sido removidas acidentalmente.
- Nulos e erros tratados e formatos normalizados.

## Próximos passos sugeridos (opcionais)
- Automatizar com um pipeline (schedule) para ingestões periódicas.
- Implementar testes de qualidade de dados (Data Quality checks) antes de mover para Silver.