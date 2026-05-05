# Guia de Pré-requisitos para o Hackathon  
### 🧩 Preparativos essenciais para participar com sucesso  

---

## ✅ Registro de Provedores de Recursos  
Certifique-se de que os seguintes provedores de recursos estejam registrados na sua assinatura do **Azure**:  
- `Microsoft.PolicyInsights`  
- `Microsoft.Cdn`  
- `Microsoft.StreamAnalytics`  

**Como registrá-los:**  
1. Vá até o **Portal do Azure** → Configurações da assinatura → *Provedores de recursos*.  
2. Selecione cada provedor e clique em **Registrar**.  

---

## ✅ Identidade e Autenticação  
**Entidade de Serviço e Autenticação:**  
- ID do cliente e segredo *(que não expirem antes do segundo dia do evento)*.  
- Os participantes devem ter seu **ID do cliente** e **segredo** disponíveis durante o hackathon.  

---

## ✅ Pré-requisitos do Microsoft Fabric  
**Opções de acesso:**  
- Criar uma **nova avaliação gratuita do Microsoft Fabric**, ou  
- Usar uma **capacidade do Fabric já provisionada** em sua assinatura do Azure.  

**Requisitos de configuração do Fabric:**  
- Pelo menos um membro designado como **administrador do Microsoft Fabric**.  
- Um **espaço de trabalho do Fabric** atribuído à equipe.  
- Capacidade de criar **Lakehouses** e **Modelos Semânticos** no Fabric.  
- Acesso ao **OneLake** (armazenamento do Fabric) para enviar arquivos.  

---

## ✅ Requisitos do Azure OpenAI  
**Cota de TPM para Modelos OpenAI:**  
- `text-embedding-ada-002`  
- `gpt-4o`  

Se a cota atual for menor que **100.000**, solicite um aumento antes do evento para garantir disponibilidade.  
> ⏱️ As aprovações normalmente levam 24 horas, portanto é fundamental concluir esta etapa com antecedência.  

**Passos recomendados:**  
- Verifique sua cota atual → [Guia de Cotas do Azure OpenAI](#)  
- Solicite aumento de cota → [Solicitação de Aumento de Cota](#)  

---

## ✅ Requisitos de Rede e Acesso  
Garanta acesso sem restrições às seguintes plataformas:  
- **Azure AI Foundry**  
- **Azure Data Factory**  
- **Document Intelligence Studio**  
- **Portal do Azure**  
- **Microsoft Fabric**  

---

## ✅ Requisitos do Visual Studio Code  
**Extensões necessárias no VS Code:**  
- Python 🐍  
- Azure Tools ☁️  
- Azure Semantic Kernel Tools 🧠  

---

## 🎯 O que Esperar  
- 💡 Desafios técnicos práticos  
- 🤝 Colaboração com profissionais da mesma área  
- 🧩 Resolução de problemas ao vivo e orientação especializada  
- 🚀 Oportunidade para aprimorar habilidades e gerar ideias com nossas equipes  

---

## 🚀 Lista Final de Verificação  
✔️ Conclua todos os pré-requisitos antes do evento  
✔️ Verifique seu acesso ao **Azure**, **Microsoft Fabric** e **serviços do OpenAI**  
✔️ Confirme que você pode acessar **todos os recursos e ferramentas necessários**  

---
# Requisitos Dia 2

# 🔐 Funções no Azure e seu Uso

| **Função no Azure** | **Uso Principal** |
|------------------|-------------------|
| **Owner ou Contributor** | Permite **criar e administrar recursos** como *AI Services*, *Azure ML*, *App Services*, entre outros. |
| **Cognitive Services Contributor** | Habilita a **configuração e administração de recursos do Cognitive Services**. |
| **Storage Blob Data Contributor** | Fornece **acesso completo ao armazenamento** utilizado pelo *AI Foundry* (leitura, gravação e exclusão de blobs). |
| **Azure OpenAI Contributor (se aplicável)** | Concede **acesso a modelos GPT, embeddings e outros serviços** do *Azure OpenAI*. |
| **Key Vault Administrator (opcional)** | Permite **gerenciar segredos, certificados e chaves de API** armazenados no *Azure Key Vault*. |

## 🎉 Nos vemos no Hackathon!  
Esperamos um evento **dinâmico e inspirador** e, acima de tudo, uma **experiência de aprendizado enriquecedora para todos**.  

Se você tiver perguntas ou precisar de ajuda, **não hesite em nos contatar**.  

> 🚀 Prepare-se para inovar, colaborar e construir soluções potencializadas por IA! 🎉