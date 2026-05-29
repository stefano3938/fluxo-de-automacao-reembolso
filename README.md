# 🤖 Agente de IA para Triagem e Aprovação de Reembolsos

Este projeto consiste em uma automação inteligente e autônoma construída na plataforma **n8n**. O objetivo principal é otimizar e automatizar o processo de atendimento ao cliente focado em solicitações de reembolso.

O fluxo atua como um "funcionário digital" que recebe os dados de um formulário preenchido pelo cliente, processa essas informações através de uma Inteligência Artificial e toma decisões de negócios baseadas em regras pré-estabelecidas, cruzando dados com o banco de dados da empresa.

## ⚙️ Como o fluxo funciona?

1. **Entrada de Dados:** O sistema recebe um alerta (via Webhook) toda vez que um cliente preenche o formulário solicitando reembolso.
2. **Busca de Dados (Google Sheets):** O e-mail do cliente é utilizado para buscar o seu histórico de compras na base de dados da empresa.
3. **Análise de IA:** O agente de Inteligência Artificial processa o comentário deixado no formulário para analisar o **sentimento** do cliente (Positivo, Neutro, Negativo ou Muito Negativo) e calcula automaticamente se o pedido está dentro do prazo de garantia de 7 dias.
4. **Tomada de Decisão (Roteamento):**
   * **Aprovação Automática:** Se estiver no prazo, o reembolso é aprovado.
   * **Escalonamento Financeiro:** Compras acima de R$ 3.000,00 fora do prazo são enviadas para análise manual da gerência.
   * **Gestão de Crise:** Se o cliente for classificado com sentimento "Muito Negativo" (agressivo, ameaças de processo/Procon), o atendimento automatizado é suspenso e a gerência é acionada imediatamente.
5. **Comunicação:** O sistema dispara e-mails atualizando o cliente sobre o status e envia alertas críticos via Telegram para a equipe responsável.

![Arquitetura do Fluxo](workflow.png)

---

## 🚀 Como instalar e rodar (Usando Docker)

Para facilitar os testes e garantir que o projeto rode em qualquer sistema operacional sem conflitos, a infraestrutura foi empacotada utilizando **Docker**.

### Pré-requisitos
* Ter o [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado na sua máquina.

### Passo a Passo
1. Faça o clone ou baixe os arquivos deste repositório para o seu computador.
2. Abra o terminal na pasta onde os arquivos foram baixados e rode o comando abaixo para iniciar o servidor do n8n em segundo plano:
   ```bash
   docker compose up -d
   ```
3. Abra o seu navegador e acesse: `http://localhost:5678`
4. Crie uma conta de administrador local (apenas para o seu ambiente).
5. No menu esquerdo, vá em **Workflows** > **Add Workflow** > Clique nos três pontinhos no canto superior direito > **Import from File**.
6. Selecione o arquivo `fluxo.json` contido neste repositório.

---

## 🔑 Configuração de Integrações e Credenciais

Para que o agente de IA funcione na sua máquina, será necessário configurar as credenciais dos serviços que ele utiliza. Dentro do n8n, você poderá autenticar cada nó de acordo com as suas próprias contas.

### 1. Banco de Dados (Google Sheets)
* **Como foi usado:** Atua como o "banco de dados" da empresa para verificar o valor gasto pelo cliente e a data da última compra.
* **Como configurar:** No nó *Google Sheets*, crie uma nova credencial do tipo **Google Sheets OAuth2 API**. Autentique com sua conta do Google e aponte o nó para uma planilha sua que contenha colunas como `nome_cliente`, `email_cliente`, `total_gasto_cliente` e `data_ultima_compra`.

### 2. Inteligência Artificial (Modelo de Linguagem)
* **Como foi usado:** O "cérebro" do agente. Ele lê as reclamações, extrai as informações estruturadas e faz a análise de sentimento.
* **Como configurar:** O projeto original foi desenhado utilizando o modelo **Gemma-4** rodando via OpenAI node. No entanto, o fluxo é totalmente agnóstico. Ao abrir o nó `OpenAI Chat Model`, você pode inserir uma **API Key da OpenAI** (para usar o GPT-4o ou GPT-3.5) ou configurar as credenciais para rodar modelos locais (como o Llama 3 ou Gemma através do Ollama).

### 3. Comunicação com Cliente (Gmail)
* **Como foi usado:** Para o envio de notificações dinâmicas (Aprovação, Análise de Gerência ou Recusa).
* **Como configurar:** No nó *Gmail*, crie uma credencial **Gmail OAuth2**. Faça login com sua conta do Google. *(Dica: Certifique-se de publicar o App no Google Cloud Console para que a credencial não expire em 7 dias).*

### 4. Alertas Internos (Telegram)
* **Como foi usado:** Utilizado para alertar a equipe de suporte ou gerência, em tempo real, sobre casos sensíveis (clientes que gastaram muito dinheiro ou clientes agressivos que exigem atenção humana urgente).
* **Como configurar:** Crie um bot no Telegram falando com o `@BotFather`, pegue o Token gerado e crie a credencial no nó do Telegram. Insira o `Chat ID` do seu grupo de suporte na configuração do nó.

---

## 🧪 Como simular o funcionamento

Como a interface do formulário (front-end) não está inclusa para manter a aplicação leve, você pode simular o envio de um cliente disparando um teste diretamente para o Webhook do n8n.

1. No n8n, abra o fluxo e clique em **Test Workflow** (para deixar o nó do Webhook ouvindo).
2. Abra o terminal e rode o comando abaixo (ou use ferramentas como Postman/Insomnia):

   ```bash
   curl -X POST http://localhost:5678/webhook-test/3f475ea5-100f-4c97-a651-8a03265e868c \
   -H "Content-Type: application/json" \
   -d '{
     "event": "reimbursement_request",
     "data": {
       "fullName": "Seu Nome",
       "email": "email_cadastrado_na_sua_planilha@gmail.com",
       "product": "Produto de Teste",
       "comments": "Odiei o produto, quero meu dinheiro de volta agora ou vou processar a empresa!"
     }
   }'
   ```
*Experimente mudar o teor do texto no campo `comments` para ver a IA alterando o roteamento da automação!*