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