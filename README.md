# 🤖 Agente de IA para Triagem e Aprovação de Reembolsos

Automação inteligente e autônoma construída na plataforma **n8n** para gerenciar solicitações de reembolso do início ao fim — sem intervenção humana na maioria dos casos.

> **Versão atual:** O gatilho do fluxo é um **e-mail recebido via Gmail**. Ao receber um e-mail de pedido de reembolso, o agente lê a mensagem, cruza os dados com a planilha da empresa e toma a decisão sozinho.

---

## 📌 O que esse projeto faz?

Quando um cliente envia um e-mail pedindo reembolso, o agente:

1. **Lê e interpreta** o e-mail recebido (extrai nome, produto, motivo e sentimento do cliente)
2. **Busca o cliente** na planilha do Google Sheets pelo e-mail do remetente
3. **Toma uma decisão** com base em regras de negócio:
   - ✅ **Dentro do prazo (≤ 7 dias):** reembolso aprovado automaticamente
   - ⚠️ **Fora do prazo + alto valor (> R$ 3.000):** escalado para gerência via Telegram
   - ❌ **Fora do prazo + baixo valor:** rejeição educada por e-mail
   - 🔥 **Cliente agressivo (sentimento "Muito Negativo"):** atendimento suspenso, alerta crítico para gerência via Telegram
   - ❓ **Cliente não encontrado na base:** e-mail solicitando mais informações
4. **Responde ao cliente** com um e-mail formatado e personalizado de acordo com o cenário

---

## 🗺️ Arquitetura do Fluxo

```
[Gmail Trigger]
      │
      ▼
[AI Agent] ──usa──> [Google Sheets Tool] (busca o cliente)
      │
      ▼
[Structured Output Parser] → JSON estruturado
      │
      ▼
[If: cliente encontrado?]
  ├── NÃO → [Email: solicitar mais informações]
  └── SIM
        │
        ▼
  [If: dentro do prazo? (≤ 7 dias)]
    ├── SIM → [Email: Reembolso Aprovado ✅]
    └── NÃO
          │
          ▼
    [If: gastou > R$ 3.000?]
      ├── SIM → [Email: caso em análise] + [Telegram: alerta gerência ⚠️]
      └── NÃO
            │
            ▼
      [If: sentimento POSITIVO / NEUTRO / NEGATIVO?]
        ├── SIM → [Email: reembolso negado ❌]
        └── NÃO (MUITO_NEGATIVO) → [Email: suspenso] + [Telegram: alerta crítico 🔥]
```

---

## 🚀 Como instalar e rodar

### Pré-requisitos
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado

### Passo a passo

**1. Clone o repositório**
```bash
git clone https://github.com/stefano3938/fluxo-de-automacao-reembolso.git
cd fluxo-de-automacao-reembolso
```

**2. Suba o n8n com Docker**
```bash
docker compose up -d
```

**3. Acesse o n8n no navegador**
```
http://localhost:5678
```

**4. Crie sua conta de administrador local** (só aparece na primeira vez)

**5. Importe o fluxo**
- No menu lateral esquerdo, clique em **Workflows**
- Clique em **Add Workflow**
- Clique nos três pontinhos (⋯) no canto superior direito
- Selecione **Import from File** e escolha o arquivo `fluxo_atualizado.json`

---

## 🔑 Configurando as Credenciais

Após importar o fluxo, você precisará conectar suas próprias contas nos nós. Clique em cada nó indicado para configurar.

### 1. 📧 Gmail (Trigger + Envio de E-mails)
- **Nós afetados:** `Gmail Trigger`, todos os nós `Send a message`
- **Tipo de credencial:** `Gmail OAuth2`
- Autentique com a conta do Google que **receberá** os pedidos de reembolso
- > ⚠️ **Importante:** Para que a credencial não expire em 7 dias, publique o app no [Google Cloud Console](https://console.cloud.google.com/) ou use uma conta de serviço.

### 2. 📊 Google Sheets (Banco de Dados)
- **Nó afetado:** `Get row(s) in sheet in Google Sheets`
- **Tipo de credencial:** `Google Sheets OAuth2 API`
- Aponte para uma planilha sua com as seguintes colunas obrigatórias:

| Coluna | Descrição |
|---|---|
| `email_cliente` | E-mail usado na compra |
| `nome_cliente` | Nome completo do cliente |
| `total_gasto_cliente` | Valor total gasto (número) |
| `data_ultima_compra` | Data da última compra (formato `YYYY-MM-DD`) |

### 3. 🤖 Modelo de IA (LLM)
- **Nó afetado:** `OpenAI Chat Model`
- **Tipo de credencial:** `OpenAI API`
- O fluxo foi desenvolvido usando o modelo **Gemma 4** via API compatível com OpenAI
- Você pode trocar por qualquer modelo compatível:
  - **OpenAI:** GPT-4o, GPT-4o-mini (basta inserir sua API Key)
  - **Local (Ollama):** Llama 3, Gemma, Mistral — sem custo de API
  - **Groq, Together AI, etc.:** qualquer provedor com endpoint no formato OpenAI

### 4. 📲 Telegram (Alertas para Gerência)
- **Nós afetados:** `Send a text message`, `Send a text message1`
- **Tipo de credencial:** `Telegram API`
- **Como configurar:**
  1. No Telegram, fale com o [@BotFather](https://t.me/BotFather) e crie um bot com `/newbot`
  2. Copie o **Token** gerado e cole na credencial do nó
  3. Descubra seu **Chat ID** falando com o [@userinfobot](https://t.me/userinfobot)
  4. Insira o Chat ID nos nós do Telegram

---

## 🧪 Como testar

**1.** Com o fluxo importado e as credenciais configuradas, **ative o workflow** pelo toggle no canto superior direito do n8n.

**2.** Envie um e-mail para a conta do Gmail configurada no `Gmail Trigger` pedindo um reembolso. Exemplo:

> *"Olá, comprei o Curso de Marketing Digital há 3 dias e gostaria de solicitar o reembolso. Obrigado."*

**3.** O Gmail Trigger verifica novos e-mails a **cada minuto**. Em até 60 segundos, o fluxo será disparado.

**4.** Experimente cenários diferentes:
- E-mail de uma conta **que existe** na sua planilha vs. uma que **não existe**
- Tom **educado** vs. **agressivo** no corpo do e-mail
- Datas de compra **dentro e fora** do prazo de 7 dias

---

## 🛡️ Segurança: Por que o e-mail do remetente é a fonte de verdade?

O agente foi instruído com uma **regra de segurança explícita**: ele usa **exclusivamente o endereço do remetente** (campo técnico do e-mail) para buscar o cliente na planilha — e ignora completamente qualquer e-mail mencionado dentro do corpo da mensagem. Isso evita ataques de engenharia social onde um cliente mal-intencionado escreve "solicito reembolso para o e-mail outrocliente@email.com" tentando disparar uma ação em nome de outra pessoa.

---

## 🗂️ Estrutura do Repositório

```
.
├── fluxo_atualizado.json   # Fluxo do n8n para importar
├── docker-compose.yaml     # Infraestrutura Docker para rodar o n8n
└── README.md               # Este arquivo
```

---

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Papel no projeto |
|---|---|
| [n8n](https://n8n.io/) | Plataforma de automação (low-code) |
| Google Gmail | Trigger de entrada + envio de respostas |
| Google Sheets | Banco de dados dos clientes |
| LLM (Gemma / GPT / Ollama) | Extração de dados e análise de sentimento |
| Telegram Bot API | Alertas em tempo real para a equipe |
| Docker | Infraestrutura para rodar o n8n localmente |

---

## 📄 Licença

Este projeto é de código aberto para fins de estudo e portfólio. Fique à vontade para clonar, adaptar e usar como base para seus próprios projetos.