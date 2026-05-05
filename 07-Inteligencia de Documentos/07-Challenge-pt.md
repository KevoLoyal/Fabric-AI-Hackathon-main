# 🏆 Desafio 2: Automação do Processamento de Documentos com Logic Apps 🚀  

## 📖 Objetivo  

Neste desafio você deverá:  

✅ Configurar uma **Azure Logic App** para processar arquivos PDF automaticamente  
✅ Habilitar a **Identidade Gerenciada** para um acesso seguro aos recursos  
✅ Atribuir **permissões** à Logic App para armazenamento e processamento com IA  
✅ Usar **Document Intelligence (Form Recognizer)** para analisar PDFs  
✅ Criar um **pipeline de ADF** para mover PDFs do **Fabric** para uma **Conta de Armazenamento**  
✅ Salvar os **JSON analisados** na **Conta de Armazenamento**  
✅ Verificar que os **arquivos processados** sejam salvos corretamente no **Storage Account**  

---

## 🚀 Passo 1: Criar uma Logic App  

💡 **Por quê?** A Logic App permitirá **automatizar** a detecção de novos arquivos PDF e ativar a análise com **Document Intelligence**.  

### 1️⃣ Criar uma Azure Logic App  
🔹 No **Azure Portal**, crie uma nova **Logic App (Consumption Plan)**.  

🔹 **Quais configurações você deve definir durante a criação?**  

✅ **Resultado**: É criada uma Logic App pronta para ser configurada.  

---

## 🚀 Passo 2: Habilitar a Identidade Gerenciada para a Logic App  

💡 **Por quê?** A Logic App precisa se autenticar de forma segura para interagir com o **Azure Storage** e o **Document Intelligence**.  

### 1️⃣ Ativar a Identidade Atribuída pelo Sistema  
🔹 Nas configurações da **Logic App**, habilite a opção **System-Assigned Identity**.  

🔹 **Onde você pode copiar o Object (Principal) ID para usá-lo mais adiante?**  

✅ **Resultado**: A Logic App conta com uma identidade para autenticação.  

---

## 🚀 Passo 3: Atribuir Permissões à Logic App  

💡 **Por quê?** A Logic App precisa de **acesso** para ler do **Blob Storage** e escrever na **Conta de Armazenamento**.  

### 1️⃣ Atribuir permissões IAM à Logic App  
🔹 Vá para **Storage Account e Document Intelligence** → **Access Control (IAM)**  

🔹 Atribua as seguintes **funções** à **Identidade Gerenciada** da Logic App:  
✅ **Storage Blob Data Contributor**  
✅ **Storage Account Contributor**  
✅ **Cognitive Services Contributor** (para Document Intelligence)  

🔹 **Por que essas funções são importantes para o processamento de documentos?**  

✅ **Resultado**: A Logic App agora pode acessar o **Blob Storage** e os **serviços de IA**.  

---

## 🚀 Passo 4: Configurar a Logic App usando o Portal do Azure (Designer)  

💡 **Por quê?** É necessário **definir o fluxo de trabalho** para processar os PDFs recebidos.  

### 1️⃣ Criar o fluxo de trabalho no Logic App Designer  
🔹 Abra o **Logic App Designer** → Inicie com uma **Blank Logic App**  

🔹 **Gatilho (Trigger):**  
✅ Adicione **"When a blob is added or modified (properties only)"**  
✅ **Selecione a conta de armazenamento**: `(Seu Storage Account)`  
✅ **Selecione o contêiner**: `Seu-Contêiner`  
✅ Adicione uma **condição**: que seja ativada apenas para arquivos `.pdf`  

🔹 **Etapa de processamento:**  
✅ Adicione **"Analyze Document (Prebuilt-Invoice)"**  
✅ Forneça a **URL de armazenamento** do arquivo  

🔹 **Salvar a saída:**  
✅ Adicione **"Create Blob"** em **Azure Blob Storage**  
✅ **Selecione o contêiner**: `processed-json`  
✅ **Defina o nome do Blob:**  

```plaintext
analyzed-document-@{triggerOutputs()?['body']['name']}.json