# KiraGo

KiraGo é uma API REST para automação de WhatsApp Web (multidevice).

Este repositório público existe para **documentação de uso** e **notas de versão**.

## Aviso importante

O WhatsApp pode banir números por uso indevido. Não use para SPAM, disparos em massa ou qualquer violação dos Termos do WhatsApp. Use por sua conta e risco.

## Configuração (.env)

Exemplo completo (mesmo formato do `.env.sample`):

```env
# .env
# Configurações server
KIRAGO_PORT=8080
KIRAGO_ADDRESS=0.0.0.0

# Token do KiraGo Admin
KIRAGO_ADMIN_TOKEN=1234ABCD

# Licença Key de uso
KIRAGO_LICENSE_KEY=LIC-XXXX-XXXX-XXXX-XXXX

# Chave de criptografia para dados sensíveis (32 bytes para AES-256)
KIRAGO_GLOBAL_ENCRYPTION_KEY=your_32_byte_encryption_key_here

# Chave HMAC global para assinatura de webhook (mínimo de 32 caracteres)
KIRAGO_GLOBAL_HMAC_KEY=your_global_hmac_key_here_minimum_32_chars

# URL global do webhook
KIRAGO_GLOBAL_WEBHOOK=https://example.com/webhook

# Webhook retry configuration (optional)
WEBHOOK_RETRY_ENABLED=true
WEBHOOK_RETRY_COUNT=5
WEBHOOK_RETRY_DELAY_SECONDS=30

# "json" ou "formulário" para o padrão
WEBHOOK_FORMAT=json

# Configuração da sessão KiraGo
SESSION_DEVICE_NAME=KiraGo

# Configuração do banco de dados
DB_USER=kirago
DB_PASSWORD=kirago
DB_NAME=kirago
DB_HOST=db
DB_PORT=5432
DB_SSLMODE=false
TZ=America/Sao_Paulo

# Configuração do RabbitMQ (opcional)
RABBITMQ_URL=amqp://kirago:kirago@localhost:5672/%2F
RABBITMQ_QUEUE=whatsapp_events
```

### Variáveis opcionais (avançado)

```env

# Repetir automaticamente o webhook (se você usar o RabbitMQ)
WEBHOOK_RETRY_ENABLED=true
WEBHOOK_RETRY_COUNT=5
WEBHOOK_RETRY_DELAY_SECONDS=30
WEBHOOK_ERROR_QUEUE_NAME=webhook_errors
```

## Endpoints (principais)

- Sessão: `/session/connect`, `/session/qr`, `/session/status`, `/session/disconnect`, `/session/logout`
- Webhook: `/webhook` (GET/POST/DELETE/PUT)
- Envio de mensagens: `/chat/send/text`, `/chat/send/image`, `/chat/send/audio`, `/chat/send/video`, `/chat/send/document`, `/chat/send/sticker`, `/chat/send/location`, `/chat/send/contact`, `/chat/send/poll`
- Interativos: `/chat/send/buttons`, `/chat/send/list`, `/chat/send/carousel`, `/chat/send/product-carousel`
- Utilitários: `/user/check`, `/user/info`, `/user/avatar`, `/chat/markread`, `/chat/react`

## Documentação no servidor

Quando disponível no seu servidor:

- Swagger/OpenAPI: `GET {BASE_URL}/api`
- Dashboard: `GET {BASE_URL}/dashboard`
- QR/Login: `GET {BASE_URL}/login`

## Autenticação

Envie sempre o header `token`:

- Header: `token: SEU_TOKEN`
- Content-Type: `application/json`

## Exemplos rápidos (cURL)

### 1) Conectar e gerar QR

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"immediate":true}' \
  "{BASE_URL}/session/connect"
```

Depois busque o QR:

```bash
curl -s -X GET \
  -H "token: SEU_TOKEN" \
  "{BASE_URL}/session/qr"
```

### 2) Status da instância

```bash
curl -s -X GET \
  -H "token: SEU_TOKEN" \
  "{BASE_URL}/session/status"
```

### 3) Enviar texto

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"Phone":"5511999999999","Body":"Olá! Teste via KiraGo","Id":"msg-1"}' \
  "{BASE_URL}/chat/send/text"
```

### 4) Configurar webhook (receber eventos)

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"webhookurl":"https://seuapp.com/webhook","events":["Message","ReadReceipt","Connected"]}' \
  "{BASE_URL}/webhook"
```

Para receber tudo:

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"webhookurl":"https://seuapp.com/webhook","events":["All"]}' \
  "{BASE_URL}/webhook"
```

## Eventos de webhook (principais)

Alguns eventos úteis para monitoramento:

- Sessão: `Connected`, `Disconnected`, `ConnectFailure`, `KeepAliveTimeout`, `KeepAliveRestored`, `StreamError`, `StreamReplaced`, `LoggedOut`, `ClientOutdated`, `TemporaryBan`
- QR/Pareamento: `QR`, `QRTimeout`, `PairSuccess`, `PairError`
- Mensagens: `Message`, `UndecryptableMessage`, `MediaRetry`, `ReadReceipt`
- Presença: `Presence`, `ChatPresence`
- Sincronização: `HistorySync`, `AppStateSyncComplete`, `OfflineSyncCompleted`

## Licenças e atualizações

O KiraGo é licenciado. O acesso e as atualizações dependem do seu plano.

Em ambientes licenciados, **atualizações podem ser bloqueadas** quando a data do build (versão instalada) estiver fora da janela de atualizações do seu plano (ex.: build de 19/12 e plano com updates até 18/12 → bloqueia).

Se você renovar/atualizar o plano, o servidor faz uma nova verificação e passa a permitir builds mais recentes automaticamente.

## Notas de versão

As mudanças ficam em **Releases** (GitHub). Se você usa o KiraGo em produção, recomendo acompanhar as notas antes de atualizar.

## Como atualizar

Normalmente é só trocar a imagem Docker para a versão mais recente e subir novamente:

```bash
docker compose pull
docker compose up -d
```

## Suporte

Se precisar de ajuda, envie:

- Seu `BASE_URL`
- O endpoint que está usando
- O erro completo retornado pela API (sem expor seu token em público)
