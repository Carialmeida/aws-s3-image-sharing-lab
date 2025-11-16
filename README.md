# Working with Amazon S3 – Complete Hands-On Lab (Markdown Version)

Este documento descreve um laboratório completo utilizando Amazon S3 para criar um bucket compartilhado com um usuário externo e configurar notificações automáticas via SNS sempre que o conteúdo do bucket for modificado. Todo o fluxo foi projetado para simular um ambiente real de colaboração entre um time interno e uma empresa de mídia contratada.

## Cenário Completo

Um café contrata uma empresa de mídia para fornecer fotos dos produtos. Para isso, a AWS fornece um ambiente onde:

- O usuário externo `mediacouser` pode **visualizar, enviar, atualizar e excluir** imagens dentro da pasta `images/` de um bucket S3.
- Todas as alterações dentro da pasta `images/` geram **notificações automáticas** enviadas via Amazon SNS para o administrador.
- A administração de permissões é feita por IAM através de um grupo chamado `mediaco`.

O fluxo geral funciona da seguinte maneira:

1. O administrador cria um bucket S3 chamado `cafe-xxxnnn` (nome único).
2. O administrador envia imagens iniciais para a pasta `images/`.
3. O usuário externo `mediacouser` acessa o console ou CLI para gerenciar imagens.
4. Quando objetos são enviados ou removidos, o S3 dispara eventos para o tópico SNS `s3NotificationTopic`.
5. O tópico envia e-mails ao administrador alertando sobre a mudança.

Este laboratório ensina como configurar todo esse fluxo completamente via console e AWS CLI.

## 📊 Arquitetura do Laboratório (Diagrama)

```mermaid
flowchart LR
    A[mediacouser<br>AWS IAM User] -->|Upload/Update/Delete Images| B[S3 Bucket<br>cafe-xxxnnn/images/]
    B -->|Event Trigger<br>ObjectCreated/ObjectRemoved| C[S3 Event Notification]
    C --> D[SNS Topic<br>s3NotificationTopic]
    D --> E[Administrador recebe Email]

---

## 2️⃣ Versão em Inglês do README (compacta)

```markdown
## 🌎 English Version – Summary

This lab demonstrates how to build an Amazon S3 file-sharing workflow with proper IAM permissions and automated notifications via SNS.  
You will:

- Create an S3 bucket (`cafe-xxxnnn`)
- Upload initial images using AWS CLI
- Review IAM group policies for `mediaco`
- Test permissions for the external user `mediacouser`
- Configure S3 → SNS event notifications
- Validate object creation and deletion events
- Ensure unauthorized actions are denied

This simulates a real-world collaboration where an external media company manages product images securely through AWS S3.


---

# Acesso inicial à AWS

1. Inicie seu laboratório e aguarde o status **Lab ready**.
2. Abra o Console AWS a partir do botão **AWS**.
3. Vá até **Details → Show** para visualizar:
   - AccessKey
   - SecretKey
   - Região: `us-west-2`
4. Copie essas credenciais para configurar o AWS CLI em um host EC2.

---

# Conectando ao CLI Host (EC2) e configurando o AWS CLI

1. No console AWS, abra **EC2 → Instances**.
2. Selecione a instância **CLI Host**.
3. Clique em **Connect → EC2 Instance Connect → Connect**.
4. No terminal, configure o AWS CLI:

```bash
aws configure
# AWS Access Key ID → copie do painel do lab
# AWS Secret Access Key → copie do painel do lab
# Default region name → us-west-2
# Default output format → json
