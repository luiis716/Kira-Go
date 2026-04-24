# KiraGo

KiraGo é uma API REST para automação de WhatsApp Web (multidevice).

Este repositório público existe para **documentação de uso** e **notas de versão**.

## Aviso importante

O WhatsApp pode banir números por uso indevido. Não use para SPAM, disparos em massa ou qualquer violação dos Termos do WhatsApp. Use por sua conta e risco.

## Instalação (Docker)

Exemplo de `docker-compose.yml`:

```yml
services:
  kirago:
    image: SUA_IMAGEM_DO_KIRAGO_AQUI
    container_name: kirago
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "${KIRAGO_PORT:-8080}:8080"
```

Suba o serviço:

```bash
docker compose up -d
```

## Configuração (.env)

Crie um arquivo `.env` na mesma pasta do `docker-compose.yml`. Você pode copiar `kirago/.env.sample` como ponto de partida.

### Variáveis obrigatórias

| Variável | Descrição |
|---|---|
| `KIRAGO_ADMIN_TOKEN` | Token de administrador para rotas `/admin/*` |
| `KIRAGO_LICENSE_KEY` | Chave de licença — obrigatória para criar instâncias |
| `KIRAGO_GLOBAL_ENCRYPTION_KEY` | Chave AES-256 para dados sensíveis (exatamente 32 bytes) |
| `KIRAGO_GLOBAL_HMAC_KEY` | Chave HMAC global para assinar webhooks (mínimo 32 caracteres) |
| `DB_USER` | Usuário PostgreSQL (padrão: `postgres`) |
| `DB_PASSWORD` | Senha PostgreSQL |
| `DB_NAME` | Nome do banco (padrão: `kirago`) |
| `DB_HOST` | Host do banco (padrão: `kirago-db`) |
| `DB_PORT` | Porta do banco (padrão: `5432`) |

### Variáveis de servidor

| Variável | Padrão | Descrição |
|---|---|---|
| `KIRAGO_PORT` | `8080` | Porta de escuta |
| `KIRAGO_ADDRESS` | `0.0.0.0` | Endereço de bind |
| `KIRAGO_PUBLIC_URL` | — | URL pública da instância (usado em links externos) |
| `KIRAGO_SKIP_HOMEPAGE` | `false` | `true` redireciona `/` direto para `/dashboard` |
| `KIRAGO_STATIC_DIR` | auto | Caminho dos arquivos estáticos (dashboard, swagger) |
| `TZ` | — | Timezone (ex: `America/Sao_Paulo`) |
| `DB_SSLMODE` | `false` | Modo SSL do banco |

### Variáveis de sessão e dispositivo

| Variável | Padrão | Descrição |
|---|---|---|
| `SESSION_DEVICE_NAME` | `KiraGo` | Nome do dispositivo exibido no WhatsApp |

### Variáveis de webhook

| Variável | Padrão | Descrição |
|---|---|---|
| `KIRAGO_GLOBAL_WEBHOOK` | — | URL de webhook global (recebe eventos de todos os usuários) |
| `WEBHOOK_FORMAT` | `json` | Formato do payload: `json` ou `form` |
| `WEBHOOK_RETRY_ENABLED` | `true` | Ativa retry automático em falha de entrega |
| `WEBHOOK_RETRY_COUNT` | `5` | Número de tentativas |
| `WEBHOOK_RETRY_DELAY_SECONDS` | `30` | Intervalo entre tentativas (segundos) |
| `WEBHOOK_ERROR_QUEUE_NAME` | `webhook_errors` | Fila RabbitMQ para webhooks com falha |

### Variáveis de presença (WhatsApp)

| Variável | Padrão | Descrição |
|---|---|---|
| `WA_AUTO_PRESENCE_AVAILABLE` | `true` | Envia `PresenceAvailable` automaticamente ao conectar |
| `WA_AUTO_PRESENCE_OFF_SECONDS` | `20` | Segundos até enviar `PresenceUnavailable` após conectar |
| `WA_AUTO_PRESENCE_ON_INCOMING` | `false` | Envia presença ao receber mensagens |
| `WA_FORCE_ACTIVE_DELIVERY_RECEIPTS` | `false` | Força recibos de entrega ativos |
| `WA_HUMAN_CHAT_PRESENCE` | `true` | Envia `ChatPresence` (digitando) antes de enviar mensagens |
| `WA_HUMAN_CHAT_PRESENCE_DELAY_MS_MIN` | `0` | Delay mínimo (ms) antes de enviar com chat presence |
| `WA_HUMAN_CHAT_PRESENCE_DELAY_MS_MAX` | `0` | Delay máximo (ms) antes de enviar com chat presence |

> **Dica:** Em alguns aparelhos, manter a instância "online" pode suprimir notificações no celular. Se isso acontecer, mantenha `WA_AUTO_PRESENCE_ON_INCOMING=false` (padrão). Se ainda persistir, teste `WA_AUTO_PRESENCE_AVAILABLE=false`.

### Variáveis de proxy

| Variável | Padrão | Descrição |
|---|---|---|
| `KIRAGO_PROXY_FAILOPEN` | `false` | Ao detectar falha no proxy, desativa-o e reconecta direto pela VPS |
| `KIRAGO_PROXY_FAILOPEN_COOLDOWN_SECONDS` | `300` | Cooldown (segundos) entre tentativas de fail-open por sessão |

### Variáveis de RabbitMQ

| Variável | Padrão | Descrição |
|---|---|---|
| `RABBITMQ_URL` | — | URL de conexão (`amqp://...`) |
| `RABBITMQ_QUEUE` | — | Fila para eventos WhatsApp |
| `RABBITMQ_TYPEBOT_QUEUE` | `typebot_outbox` | Fila para mensagens do Typebot |

### Variáveis de log

| Variável | Padrão | Descrição |
|---|---|---|
| `KIRAGO_EVENT_LOG` | `false` | Persiste logs de eventos no banco, acessíveis em `GET /logs/events` |

---

## Documentação no servidor

Após subir a instância:

| URL | Descrição |
|---|---|
| `{BASE_URL}/dashboard` | Painel de gerenciamento |
| `{BASE_URL}/api` | Swagger / OpenAPI |
| `{BASE_URL}/docs` | Documentação de uso |
| `{BASE_URL}/login` | Tela de login por token |

## Autenticação

Todas as rotas (exceto `/health`) exigem o header:

```
token: SEU_TOKEN
Content-Type: application/json
```

## Endpoints (resumo)

### Sessão
| Método | Rota | Descrição |
|---|---|---|
| POST | `/session/connect` | Iniciar conexão / gerar QR |
| GET | `/session/qr` | Obter QR Code atual |
| GET | `/session/status` | Status da sessão |
| POST | `/session/disconnect` | Desconectar |
| POST | `/session/logout` | Logout completo |
| POST | `/session/pairphone` | Parear por número de telefone |
| GET/POST | `/session/history` | Solicitar sincronização de histórico |

### Webhook
| Método | Rota | Descrição |
|---|---|---|
| POST | `/webhook` | Configurar webhook |
| GET | `/webhook` | Obter configuração atual |
| PUT | `/webhook` | Atualizar configuração |
| DELETE | `/webhook` | Remover webhook |

### Envio de mensagens
| Método | Rota | Descrição |
|---|---|---|
| POST | `/chat/send/text` | Texto |
| POST | `/chat/send/image` | Imagem |
| POST | `/chat/send/audio` | Áudio |
| POST | `/chat/send/video` | Vídeo |
| POST | `/chat/send/document` | Documento |
| POST | `/chat/send/sticker` | Sticker |
| POST | `/chat/send/gif` | GIF |
| POST | `/chat/send/location` | Localização |
| POST | `/chat/send/contact` | Contato |
| POST | `/chat/send/poll` | Enquete |
| POST | `/chat/send/buttons` | Botões interativos |
| POST | `/chat/send/list` | Lista interativa |
| POST | `/chat/send/carousel` | Carrossel |
| POST | `/chat/send/product-carousel` | Carrossel de produtos |
| POST | `/chat/send/order` | Detalhes de pedido |
| POST | `/chat/send/edit` | Editar mensagem enviada |
| POST | `/chat/send/presence` | Enviar presença |

### Ações no chat
| Método | Rota | Descrição |
|---|---|---|
| POST | `/chat/react` | Reagir a mensagem |
| POST | `/chat/markread` | Marcar como lido |
| POST | `/chat/delete` | Deletar mensagem |
| POST | `/chat/presence` | Typing / ChatPresence |
| GET | `/chat/history` | Histórico do chat |
| POST | `/chat/archive` | Arquivar/desarquivar chat |

### Download de mídia
| Método | Rota | Descrição |
|---|---|---|
| POST | `/chat/downloadimage` | Imagem |
| POST | `/chat/downloadvideo` | Vídeo |
| POST | `/chat/downloadaudio` | Áudio |
| POST | `/chat/downloaddocument` | Documento |

### Usuário / contatos
| Método | Rota | Descrição |
|---|---|---|
| POST | `/user/check` | Verificar se número existe no WhatsApp |
| GET | `/user/info` | Informações do usuário |
| GET | `/user/avatar` | Foto de perfil |
| GET | `/user/contacts` | Lista de contatos |
| GET | `/user/lid` | Obter LID do usuário |
| GET/POST | `/user/privacy` | Configurações de privacidade |
| GET/POST | `/user/blocklist` | Lista de bloqueados |

### Status / Story
| Método | Rota | Descrição |
|---|---|---|
| POST | `/status/text` | Publicar story de texto |
| POST | `/status/image` | Publicar story de imagem |
| POST | `/status/video` | Publicar story de vídeo |
| POST | `/status/audio` | Publicar story de áudio |
| POST | `/status/sticker` | Publicar story de sticker |
| POST | `/status/set` | Definir texto de status pessoal |

### Grupos
| Método | Rota | Descrição |
|---|---|---|
| GET | `/group/list` | Listar grupos |
| GET | `/group/info` | Informações do grupo |
| POST | `/group/create` | Criar grupo |
| POST | `/group/leave` | Sair do grupo |
| POST | `/group/join` | Entrar via link |
| POST | `/group/participants` | Adicionar/remover participantes |
| POST | `/group/name` | Renomear grupo |
| POST | `/group/topic` | Alterar descrição |
| POST | `/group/photo` | Alterar foto |
| GET | `/group/invitelink` | Obter link de convite |
| GET | `/group/inviteinfo` | Info do link de convite |

### Newsletter (Canais WhatsApp)
| Método | Rota | Descrição |
|---|---|---|
| GET | `/newsletter/list` | Listar newsletters seguidos |
| POST | `/newsletter/create` | Criar newsletter |
| GET | `/newsletter/info` | Info do newsletter |
| POST | `/newsletter/follow` | Seguir |
| POST | `/newsletter/unfollow` | Deixar de seguir |
| POST | `/newsletter/mute` | Silenciar/ativar |

### CRM
| Método | Rota | Descrição |
|---|---|---|
| POST | `/session/crm/config` | Configurar integração CRM |
| GET | `/session/crm/config` | Obter configuração |
| DELETE | `/session/crm/config` | Remover configuração |
| GET | `/session/crm/token` | Obter token do CRM |
| POST | `/session/crm/sync-history` | Sincronizar histórico de mensagens para o CRM |

### Chatwoot
| Método | Rota | Descrição |
|---|---|---|
| POST | `/session/chatwoot/config` | Configurar Chatwoot |
| GET | `/session/chatwoot/config` | Obter configuração |
| DELETE | `/session/chatwoot/config` | Remover configuração |
| POST | `/session/chatwoot/sync-history` | Sincronizar histórico |
| POST | `/chatwoot/webhook` | Webhook recebido do Chatwoot |

### Typebot
| Método | Rota | Descrição |
|---|---|---|
| POST | `/session/typebot/config` | Configurar Typebot |
| GET | `/session/typebot/config` | Obter configuração |
| DELETE | `/session/typebot/config` | Remover configuração |
| POST | `/typebot/start` | Iniciar fluxo |
| POST | `/typebot/continue` | Continuar fluxo |

### Configurações avançadas por sessão
| Método | Rota | Descrição |
|---|---|---|
| POST/GET/DELETE | `/session/s3/config` | Configuração de S3 |
| POST | `/session/s3/test` | Testar conexão S3 |
| POST/GET/DELETE | `/session/rabbitmq/config` | Configuração de RabbitMQ |
| POST | `/session/rabbitmq/test` | Testar conexão RabbitMQ |
| POST/GET/DELETE | `/session/hmac/config` | Configuração de HMAC por sessão |
| POST/GET/DELETE | `/session/proxy` | Configurar proxy de saída |
| POST | `/session/proxy/test` | Testar proxy |
| GET/POST | `/session/skip` | Configurar eventos ignorados |

### Admin
| Método | Rota | Descrição |
|---|---|---|
| GET/POST | `/admin/users` | Listar / criar usuários |
| PUT/DELETE | `/admin/users/{id}` | Editar / remover usuário |

### Utilitários
| Método | Rota | Descrição |
|---|---|---|
| GET | `/health` | Health check |
| GET | `/logs/events` | Logs de eventos (requer `KIRAGO_EVENT_LOG=true`) |
| POST | `/call/reject` | Rejeitar chamada recebida |

---

## Integração CRM

O KiraGo suporta envio de eventos em tempo real para um CRM externo via HTTP POST no endpoint `/api/inbound` do CRM.

### O que é enviado

- **Mensagens 1:1** — texto, imagem, áudio, vídeo, documento, sticker, localização, reações, respostas e deleções
- **ReadReceipt** — confirmação de leitura
- **Presence** — status de presença do contato
- **HistorySync** — histórico de mensagens ao sincronizar (quando `history_sync: true`)

> Grupos e canais (newsletters) não são enviados ao CRM — apenas conversas individuais.

### Resolução de nome do contato

O nome enviado ao CRM segue a ordem: `FullName → FirstName → PushName → JID`. Para mensagens enviadas pela instância, o nome é buscado no store local de contatos.

### Sincronização de histórico

Use `POST /session/crm/sync-history` para sincronizar mensagens existentes:

```bash
curl -X POST "{BASE_URL}/session/crm/sync-history" \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "direction": "both",
    "include_media": true,
    "batch_size": 500,
    "max_messages": 0,
    "dry_run": false
  }'
```

Parâmetros:

| Campo | Tipo | Descrição |
|---|---|---|
| `direction` | string | `sent`, `received` ou `both` |
| `include_media` | bool | Incluir mídia no payload |
| `batch_size` | int | Mensagens por lote (padrão: 500) |
| `max_messages` | int | Limite total (0 = sem limite) |
| `dry_run` | bool | Simula sem enviar ao CRM |

---

## Eventos de webhook

Use `"events": ["All"]` para receber tudo, ou especifique os tipos desejados.

### Sessão e conexão
`Connected` · `Disconnected` · `ConnectFailure` · `LoggedOut` · `ClientOutdated` · `TemporaryBan` · `StreamError` · `StreamReplaced` · `KeepAliveTimeout` · `KeepAliveRestored`

### QR / pareamento
`QR` · `QRTimeout` · `PairSuccess` · `PairError` · `QRScannedWithoutMultidevice`

### Mensagens
`Message` · `GroupMessage` · `UndecryptableMessage` · `MediaRetry` · `ReadReceipt` · `GroupReadReceipt`

### Grupos e contatos
`GroupInfo` · `JoinedGroup` · `Picture` · `BlocklistChange` · `Blocklist`

### Presença
`Presence` · `ChatPresence`

### Sincronização
`HistorySync` · `AppState` · `AppStateSyncComplete` · `OfflineSyncCompleted` · `OfflineSyncPreview`

### Chamadas
`CallOffer` · `CallAccept` · `CallTerminate` · `CallOfferNotice` · `CallRelayLatency`

### Configurações e privacidade
`PrivacySettings` · `PushNameSetting` · `UserAbout` · `IdentityChange`

### Newsletter (Canais)
`NewsletterJoin` · `NewsletterLeave` · `NewsletterMuteChange` · `NewsletterLiveUpdate`

---

## Exemplos rápidos (cURL)

### Conectar e gerar QR

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"immediate":true}' \
  "{BASE_URL}/session/connect"
```

```bash
curl -s -X GET \
  -H "token: SEU_TOKEN" \
  "{BASE_URL}/session/qr"
```

### Enviar texto

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"Phone":"5511999999999","Body":"Olá via KiraGo","Id":"msg-1"}' \
  "{BASE_URL}/chat/send/text"
```

### Configurar webhook

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"webhookurl":"https://seuapp.com/webhook","events":["Message","ReadReceipt","Connected"]}' \
  "{BASE_URL}/webhook"
```

### Configurar CRM

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"base_url":"https://seucrm.com","api_token":"TOKEN_CRM","history_sync":true}' \
  "{BASE_URL}/session/crm/config"
```

---

## Licenças e atualizações

O KiraGo é licenciado por chave. O acesso à API e as atualizações dependem da licença ativa.

Atualizações podem ser bloqueadas quando o build instalado estiver fora da janela de atualizações do plano. Ao renovar, o servidor revalida automaticamente.

## Como atualizar

```bash
docker compose pull
docker compose up -d
```

## Suporte

Se precisar de ajuda, informe:

- Seu `BASE_URL`
- O endpoint utilizado
- O erro retornado pela API (sem expor o token)
