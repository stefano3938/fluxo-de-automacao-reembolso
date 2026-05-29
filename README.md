# fluxo-de-automacao-reembolso
# 🤖 Agente de IA para Triagem e Aprovação de Reembolsos

Este projeto é uma automação inteligente construída no **n8n**. Ele atua como um agente autônomo que recebe solicitações de reembolso de clientes, cruza os dados com uma base no Google Sheets, analisa o sentimento da reclamação usando IA e toma decisões de negócios (aprovar, encaminhar para gerência ou tratar como cliente crítico).

![Arquitetura do Fluxo](workflow.png)

## 💼 Regras de Negócio Implementadas
* **Análise de Sentimento:** A IA (OpenAI/Gemma) lê a reclamação e classifica o humor do cliente.
* **Cálculo de Prazo:** Verifica automaticamente se a compra está dentro dos 7 dias de garantia.
* **Triagem Financeira:** Compras acima de R$ 3.000,00 são escaladas para análise humana.
* **Comunicação Multicanal:** Dispara e-mails personalizados (Gmail) baseados no status e envia alertas críticos para a gerência via Telegram.

## 🚀 Como rodar este projeto em 1 minuto

Para facilitar os testes, este projeto está empacotado com Docker. Você não precisa instalar bancos de dados ou configurar o n8n manualmente.

### Pré-requisitos
* Ter o [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado na sua máquina.

### Passo a Passo
1. Faça o clone ou baixe este repositório.
2. Abra o terminal na pasta do projeto e rode o comando:
   ```bash
   docker compose up -d