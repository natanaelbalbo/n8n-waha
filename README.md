<<<<<<< HEAD
# Integração n8n + WhatsApp (Waha) para Resumo de Mensagens

Este repositório contém a configuração necessária para integrar o n8n com o WhatsApp utilizando o Waha como middleware, com o objetivo de criar um sistema de resumo de mensagens do WhatsApp.

## Pré-requisitos

- Docker e Docker Compose instalados
- Conhecimento básico de Docker e n8n
- Um número de telefone com WhatsApp ativo

## Estrutura do Projeto

O projeto consiste em três arquivos principais:

1. `docker-compose.yml` - Configuração dos containers Docker
2. `.env` - Variáveis de ambiente para configuração
3. `Dockerfile.waha` - Dockerfile para personalização do Waha (opcional)

## Configuração Passo a Passo

### 1. Configuração do Ambiente

Clone este repositório e certifique-se de que os arquivos estão corretamente configurados:

#### docker-compose.yml

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    ports:
      - "5679:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - N8N_LISTEN_ADDRESS=0.0.0.0
      - N8N_EDITOR_BASE_URL=http://localhost:5679
      - N8N_SECURE_COOKIE=false
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_RUNNERS_ENABLED=true
    volumes:
      - /home/natanael-figueredo-balbo/n8n:/home/node/.n8n
    restart: always
    networks:
      - waha-network

  waha:
    image: devlikeapro/waha
    container_name: waha
    ports:
      - "3001:3000"
    environment:
      - WAHA_PORT=3000
      - WAHA_LOG_LEVEL=info
      - WHATSAPP_DEFAULT_ENGINE=${WHATSAPP_DEFAULT_ENGINE}
      - WHATSAPP_HOOK_EVENTS=${WHATSAPP_HOOK_EVENTS}
      - WHATSAPP_HOOK_URL=${WHATSAPP_HOOK_URL}
      - NODE_OPTIONS=--no-deprecation
      - WAHA_DASHBOARD_ENABLED=${WAHA_DASHBOARD_ENABLED}
      - WAHA_DASHBOARD_USERNAME=${WAHA_DASHBOARD_USERNAME}
      - WAHA_DASHBOARD_PASSWORD=${WAHA_DASHBOARD_PASSWORD}
    volumes:
      - /home/natanael-figueredo-balbo/waha:/tmp/
    restart: always
    depends_on:
      - n8n
    networks:
      - waha-network
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  waha_data:

networks:
  waha-network:
    driver: bridge
```

#### .env

```bash
# n8n
N8N_ENCRYPTION_KEY=suachaveunica123456789
GENERIC_TIMEZONE=America/Sao_Paulo

# WAHA
# Porta interna do Waha dentro do container
WAHA_PORT=3000
WHATSAPP_DEFAULT_ENGINE=NOWEB
WHATSAPP_HOOK_EVENTS=message
# URL do webhook do n8n
# Usando host.docker.internal e a porta externa 5679
WHATSAPP_HOOK_URL=http://host.docker.internal:5679/webhook-test/c3dc7c4f-4f55-4121-bac8-edc68b70f18c/waha
WAHA_DASHBOARD_ENABLED=true
WAHA_DASHBOARD_USERNAME=waha
WAHA_DASHBOARD_PASSWORD=waha
```

### 2. Iniciando os Containers

Execute o seguinte comando para iniciar os containers:

```bash
docker-compose up -d
```

### 3. Configurando o n8n

1. Acesse o n8n em: http://localhost:5679
2. Instale o node do Waha:
   - Vá para "Settings" > "Community Nodes"
   - Pesquise e instale: `@devlikeapro/n8n-nodes-waha`
   - Reinicie o n8n quando solicitado

### 4. Criando um Fluxo no n8n com Trigger do Waha

1. Crie um novo fluxo no n8n
2. Adicione um nó "Waha Trigger"
3. Configure o trigger:
   - **Authentication**: None (ou configure se necessário)
   - **Host**: http://localhost:3001 (URL do Waha)
   - **Events**: message (ou outros eventos que deseja capturar)
   - **Webhook URL**: Copie a URL gerada (será algo como `http://localhost:5679/webhook-test/ID-ÚNICO/waha`)

4. Atualize o arquivo `.env` com a URL do webhook gerada:
   ```
   WHATSAPP_HOOK_URL=http://host.docker.internal:5679/webhook-test/ID-ÚNICO/waha
   ```

5. Reinicie o container do Waha:
   ```bash
   docker-compose restart waha
   ```

### 5. Conectando o WhatsApp

1. Acesse o dashboard do Waha em: http://localhost:3001
2. Inicie uma sessão clicando em "Start Session"
3. Escaneie o QR code com seu WhatsApp
4. Aguarde a conexão ser estabelecida

### 6. Testando a Integração

1. Envie uma mensagem para o número do WhatsApp conectado
2. Verifique se o trigger do n8n é acionado
3. Continue construindo seu fluxo no n8n para processar as mensagens recebidas

## Solução de Problemas

### Problema: O webhook não está funcionando

1. **Verifique a URL do webhook**:
   - Certifique-se de que a URL no arquivo `.env` está correta
   - Use `host.docker.internal` em vez de `localhost` ou IP específico
   - Use a porta externa do n8n (5679)

2. **Verifique a configuração de rede**:
   - Certifique-se de que os containers estão na mesma rede
   - Verifique se o `extra_hosts` está configurado para o container do Waha

3. **Verifique os logs**:
   ```bash
   docker logs waha | grep webhook
   ```

4. **Reinicie os containers**:
   ```bash
   docker-compose down && docker-compose up -d
   ```

### Problema: Erro de permissão ao parar containers

Se você encontrar erros de permissão ao tentar parar os containers, use sudo:

```bash
sudo docker-compose down
sudo docker-compose up -d
```

Ou adicione seu usuário ao grupo docker:

```bash
sudo usermod -aG docker $USER
```
(Requer logout e login para ter efeito)

## Notas Importantes

1. **Segurança**: Este setup é para desenvolvimento. Para produção, considere adicionar autenticação e HTTPS.
2. **Persistência**: Os dados do n8n e Waha são persistidos nos volumes configurados.
3. **Portas**: 
   - n8n: acessível na porta 5679
   - Waha: acessível na porta 3001

## Estado Atual do Projeto

### O Que Foi Feito

1. **Configuração Básica**:
   - Configuração dos containers Docker para n8n e Waha
   - Configuração da rede Docker para comunicação entre os containers
   - Configuração do webhook para o Waha enviar mensagens para o n8n

2. **Resolução de Problemas**:
   - Correção do problema de conexão entre Waha e n8n usando `host.docker.internal`
   - Configuração do n8n para aceitar conexões externas
   - Desativação de cookies seguros para desenvolvimento local
   - Configuração do n8n para usar `localhost` como hostname para compatibilidade com Google OAuth

3. **Integração com Google Sheets** ✅:
   - Configuração do URI de redirecionamento OAuth para o Google
   - Ativação das APIs do Google Drive e Google Sheets
   - Configuração da autenticação OAuth com o Google
   - Conexão bem-sucedida com o Google Sheets

## O Que Falta Para Completar o Projeto

1. **Criação do Fluxo de Resumo no n8n**:
   - Criar um novo fluxo no n8n
   - Adicionar o nó "Waha Trigger" para capturar mensagens do WhatsApp
   - Adicionar um nó "Function" para processar e resumir as mensagens
   - Adicionar um nó "Google Sheets" para salvar os resumos em uma planilha
   - (Opcional) Adicionar nós para notificações ou outras integrações

3. **Exemplo de Fluxo para Resumo de Mensagens**:
   ```
   [Waha Trigger] → [Function (Resumo)] → [Google Sheets] → [Notificações (opcional)]
   ```

4. **Exemplo de Código para o Nó Function**:
   ```javascript
   // Exemplo de código para resumir mensagens
   const message = $input.all()[0].json.message.body;
   const sender = $input.all()[0].json.message.sender;
   const timestamp = new Date().toISOString();
   
   // Aqui você pode implementar um algoritmo de resumo
   // ou usar uma API de IA para resumir o texto
   const summary = message.length > 100 ? 
     message.substring(0, 100) + "..." : 
     message;
   
   return [
     {
       json: {
         timestamp,
         sender,
         original_message: message,
         summary
       }
     }
   ];
   ```

## Próximos Passos

1. Crie o fluxo de resumo de mensagens no n8n
2. Teste enviando mensagens para o WhatsApp conectado
3. Ajuste o algoritmo de resumo conforme necessário

## Referências

- [Documentação do n8n](https://docs.n8n.io/)
- [Documentação do Waha](https://waha.devlike.pro/docs/)
- [Node do Waha para n8n](https://www.npmjs.com/package/@devlikeapro/n8n-nodes-waha)
- [API do Google Sheets](https://developers.google.com/sheets/api)
- [Autenticação OAuth no n8n](https://docs.n8n.io/integrations/credentials/oauth2/)
=======
# Integração n8n + WAHA

## Situação Atual
- **n8n**: Plataforma de automação (porta 5678)
- **WAHA**: API HTTP para WhatsApp (porta 3000)

## Configuração Atual
- Docker Compose com n8n e WAHA
- WAHA configurado com engine NOWEB
- Tentativa de integração via webhooks

## Problema Identificado
- **Erro na comunicação via webhook**:
  - URL no `.env`: `WHATSAPP_HOOK_URL=http://n8n-aula:5678/webhook/`
  - URL incompleta, falta ID do webhook
  - Erro nos logs: `ERROR: POST request failed: Request failed with status code 404` ("The requested webhook is not registered")

## Causa Raiz
- WAHA envia eventos, mas n8n não reconhece o endpoint devido a ID incompleto ou webhook inativo

## O Que Está Faltando
1. **URL de Webhook Completa**:
   - Criar webhook no n8n
   - Exemplo: `http://n8n-aula:5678/webhook/abc123def456`
2. **Workflow Ativo no n8n**:
   - Workflow ativado (não apenas em teste)
   - Iniciar com nó Webhook (genérico, método POST)
3. **Configuração de Eventos**:
   - Verificar `WHATSAPP_HOOK_EVENTS=message` para incluir eventos desejados

## Próximos Passos
1. **No n8n**:
   - Criar novo workflow
   - Adicionar nó "Webhook" (não WAHA Trigger)
   - Configurar método POST
   - Ativar workflow
   - Copiar URL completa do webhook
2. **No arquivo `.env`**:
   - Atualizar: `WHATSAPP_HOOK_URL=http://n8n-aula:5678/webhook/SEU_ID_AQUI`
3. **Reiniciar containers**:
   ```bash
   docker-compose down && docker-compose up -d
   ```
4. **Testar integração**:
   - Enviar mensagem no WhatsApp
   - Verificar execução no n8n (aba "Executions")

## Observações Adicionais
- Webhook genérico é mais confiável que WAHA Trigger
- Workflow deve estar ativo, não em modo teste
- Logs do WAHA (`docker logs n8n-waha-waha-1`) são essenciais para diagnóstico

Esta configuração permite que o WAHA envie eventos ao n8n para mensagens recebidas no WhatsApp, possibilitando automações.
>>>>>>> 82bd0ccc52c3ba667f2f8acf6b93af172a5664c7
