# KiraGo

KiraGo é uma API REST para automação de WhatsApp Web (multidevice).

Este repositório reúne **documentação de uso** para assinantes e **notas de cada versão** ([CHANGELOG](./CHANGELOG.md)). Aqui você encontra como configurar, integrar e o que mudou em cada atualização da sua assinatura — não é um guia de deploy para desenvolvedores.

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
| `KIRAGO_GLOBAL_WEBHOOK` | — | URL que recebe eventos de **todas** as instâncias do servidor |
| `KIRAGO_GLOBAL_WEBHOOK_EVENTS` | — | Nomes dos eventos separados por vírgula ([lista e significado de cada um](#eventos-de-webhook--o-que-é-cada-um)). **Vazio = todos** |
| `WEBHOOK_FORMAT` | `json` | Formato do payload: `json` ou `form` |
| `WEBHOOK_RETRY_ENABLED` | `true` | Ativa retry automático em falha de entrega (webhook por instância) |
| `WEBHOOK_RETRY_COUNT` | `5` | Número de tentativas |
| `WEBHOOK_RETRY_DELAY_SECONDS` | `30` | Intervalo entre tentativas (segundos) |
| `WEBHOOK_ERROR_QUEUE_NAME` | `webhook_errors` | Fila RabbitMQ para webhooks com falha |

**Webhooks por instância** não usam variável de ambiente: configure no painel (**Gerenciar Webhooks**) ou pela API `GET/POST /webhooks`. O webhook global do `.env` é independente e continua recebendo eventos de **todas** as instâncias.

Exemplo de webhook global no `.env` (lista de eventos na seção [Eventos de webhook](#eventos-de-webhook--o-que-é-cada-um)):

```env
KIRAGO_GLOBAL_WEBHOOK=https://seu-servidor.com/hook/global
KIRAGO_GLOBAL_WEBHOOK_EVENTS=Message,GroupMessage,Connected,Disconnected,ReadReceipt
```

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

#### Pools Webshare (admin)

Na **lista de instâncias** (login admin), abra **Pools Webshare** para cadastrar uma ou mais contas [Webshare](https://www.webshare.io/):

| Campo | Descrição |
|---|---|
| **Nome** | Identificação no painel (ex.: *Conta principal*, *Revenda SP*) |
| **API key** | Chave da conta Webshare desse pool |
| **Esquema** | `http` ou `socks5` |

Cada instância escolhe **qual pool** usar no modal **Proxy** — não é mais necessário colar a API key em cada instância se os pools já estiverem cadastrados pelo admin.

**API (admin):** `GET/POST /admin/proxy/webshare/pools`, `PUT/DELETE /admin/proxy/webshare/pools/{id}`.

#### Proxy no painel (por instância)

No dashboard, abra a instância → **Proxy**:

| Modo | O que você faz |
|---|---|
| **Webshare (pool)** | Selecione um **pool nomeado** cadastrado pelo admin (ou informe API key direto, se preferir). O KiraGo escolhe um proxy da lista Webshare automaticamente. |
| **Manual** | Informe host, porta, usuário e senha (SOCKS5 ou HTTP). |

**Compartilhar pool (legado):** instâncias ainda podem referenciar pool de outra instância via `source_user_id`; o modo recomendado é usar **pools nomeados** no admin.

**Trocar IP:** use *forçar rotação* no modal para pegar outro proxy do pool selecionado.

**Fail-open:** com `KIRAGO_PROXY_FAILOPEN=true`, falha no proxy desativa-o temporariamente e a sessão reconecta direto pela VPS (ver tabela de variáveis acima).

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

## Versão e novidades no painel

- A **versão instalada** aparece na navbar do dashboard (ao lado do botão **API**).
- Quando sai uma versão mais nova, o painel mostra um **aviso** no badge de versão.
- Ao entrar no dashboard, pode abrir um **modal com as novidades** da atualização (link para detalhes no GitHub).
- O histórico completo de cada versão está no [CHANGELOG.md](./CHANGELOG.md).

> As atualizações do software na sua instalação dependem da **assinatura ativa** e do canal pelo qual você recebe o KiraGo (hospedagem gerenciada, imagem fornecida pelo suporte, etc.). O aviso no painel informa que existe versão mais nova; a aplicação na sua infraestrutura segue o processo do seu plano.

## Autenticação

Todas as rotas (exceto `/health` e páginas estáticas do painel) exigem o header:

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

### Webhook (por instância)

Cada instância pode ter **vários webhooks**. Cada um tem nome, URL, lista de eventos e flag `active`. Eventos recebidos no WhatsApp são enviados para **todas** as URLs ativas que assinam aquele tipo.

| Método | Rota | Descrição |
|---|---|---|
| GET | `/webhooks` | Listar todos os webhooks da instância |
| POST | `/webhooks` | Criar webhook (`name`, `webhookurl`, `events`, `active`) |
| PUT | `/webhooks/{id}` | Atualizar webhook (parcial ou completo) |
| DELETE | `/webhooks/{id}` | Excluir webhook |

**Legado (webhook único “Principal”):**

| Método | Rota | Descrição |
|---|---|---|
| GET | `/webhook` | Obter webhook Principal |
| POST | `/webhook` | Criar/atualizar Principal |
| PUT | `/webhook` | Atualizar Principal |
| DELETE | `/webhook` | Remover Principal |

No painel: card **Webhooks** na instância → **Gerenciar Webhooks** (lista + formulário).

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
| POST | `/session/proxy` | Configurar proxy manual de saída |
| POST | `/session/proxy/webshare` | Ativar Webshare (`pool_id`, `enable`, `force_rotate`, …) |
| GET | `/session/proxy/pool-config` | Listar pools Webshare disponíveis e pool atual da instância |
| POST | `/session/proxy/test` | Testar proxy |
| GET/POST | `/session/skip` | Configurar eventos ignorados |

### Admin
| Método | Rota | Descrição |
|---|---|---|
| GET/POST | `/admin/users` | Listar / criar usuários |
| PUT/DELETE | `/admin/users/{id}` | Editar / remover usuário |
| GET/POST | `/admin/proxy/webshare/pools` | Listar / criar pools Webshare nomeados |
| PUT/DELETE | `/admin/proxy/webshare/pools/{id}` | Editar / remover pool |
| GET/POST | `/admin/proxy/webshare` | Legado — primeiro pool da lista |

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

## Eventos de webhook — o que é cada um

O webhook envia um **POST** para a sua URL sempre que algo acontece no WhatsApp. Você escolhe **quais tipos de aviso** quer receber — cada tipo tem um **nome fixo** (coluna **Evento**).

Use esses nomes:

- no **webhook global**: `KIRAGO_GLOBAL_WEBHOOK_EVENTS=Message,Connected,...` no `.env`
- no **webhook da instância**: painel (checkboxes) ou `POST /webhook` com `"events": ["Message", "Connected"]`
- para **receber tudo**: deixe `KIRAGO_GLOBAL_WEBHOOK_EVENTS` vazio (global) ou use `"All"` (instância)

Abaixo, **o que cada evento seria** na prática — o que você está assinando para receber no seu sistema.

### Mensagens

| Evento | O que é este evento |
|---|---|
| `Message` | Aviso de **mensagem em chat individual** (texto, áudio, imagem, vídeo, documento, sticker, localização, etc.) — recebida ou enviada pelo número conectado. |
| `GroupMessage` | Aviso de **mensagem em grupo** — o mesmo tipo de conteúdo da `Message`, mas em conversa de grupo. |
| `UndecryptableMessage` | Aviso de que **chegou uma mensagem**, mas o KiraGo **não conseguiu ler/descriptografar** o conteúdo (útil para log e suporte). |
| `Receipt` | Aviso de **confirmação de entrega/leitura** (equivalente ao `ReadReceipt`; nome legado na API). |
| `ReadReceipt` | Aviso de que o contato **visualizou/leu** sua mensagem no chat individual (tique azul). |
| `GroupReadReceipt` | Aviso de que **alguém leu** uma mensagem em um grupo. |
| `MediaRetry` | Aviso de **nova tentativa** de baixar um arquivo de mídia que tinha falhado antes. |

### Grupos e contatos

| Evento | O que é este evento |
|---|---|
| `GroupInfo` | Aviso de que **algo mudou no grupo**: nome, descrição, foto, lista de participantes ou regras do grupo. |
| `JoinedGroup` | Aviso de que o número conectado **entrou em um grupo novo**. |
| `Picture` | Aviso de que a **foto de perfil** de um contato ou de um grupo foi alterada. |
| `BlocklistChange` | Aviso de que um número foi **bloqueado ou desbloqueado** na lista de bloqueados. |
| `Blocklist` | Envio da **lista completa** de números bloqueados (sincronização da blocklist). |

### Conexão da instância

| Evento | O que é este evento |
|---|---|
| `Connected` | Aviso de que a instância **está online** e conectada ao WhatsApp. |
| `Disconnected` | Aviso de que a instância **caiu** (internet, reinício, desconexão temporária, etc.). |
| `ConnectFailure` | Aviso de que **não foi possível conectar** (tentativa de login/conexão falhou). |
| `LoggedOut` | Aviso de que a sessão foi **encerrada pelo WhatsApp** (ex.: desvinculou o aparelho no celular). |
| `ClientOutdated` | Aviso de que o WhatsApp considera o **cliente desatualizado** — pode exigir atualização do KiraGo. |
| `TemporaryBan` | Aviso de **restrição temporária** no número (banimento temporário do WhatsApp). |
| `StreamError` | Aviso de **erro na conexão em tempo real** com os servidores do WhatsApp. |
| `StreamReplaced` | Aviso de que **outro login** com o mesmo número tomou o lugar desta sessão. |
| `KeepAliveTimeout` | Aviso de que a conexão **parou de responder** por um tempo (timeout). |
| `KeepAliveRestored` | Aviso de que a conexão **voltou ao normal** depois de um timeout. |

### QR Code e pareamento

| Evento | O que é este evento |
|---|---|
| `QR` | Envio do **QR Code** (imagem/dados) para vincular o WhatsApp Web. |
| `QRTimeout` | Aviso de que o **QR expirou** e é preciso gerar outro. |
| `PairSuccess` | Aviso de que o **pareamento por código/número** deu certo. |
| `PairError` | Aviso de **falha** no pareamento por código/número. |
| `QRScannedWithoutMultidevice` | Aviso de que alguém escaneou o QR em um celular **sem suporte a multidevice** (pareamento pode falhar). |

### Presença (online / digitando)

| Evento | O que é este evento |
|---|---|
| `Presence` | Aviso de mudança de **status do contato** (online, offline, etc.) em nível geral. |
| `ChatPresence` | Aviso de que o contato está **digitando** ou **gravando áudio** em um chat específico. |

### Sincronização de dados

| Evento | O que é este evento |
|---|---|
| `HistorySync` | Envio de **mensagens antigas** em lote (histórico ao conectar ou ao pedir sincronização). |
| `AppState` | Aviso de mudança em **dados internos** do WhatsApp (contatos, listas, configurações sincronizadas). |
| `AppStateSyncComplete` | Aviso de que a **sincronização desses dados terminou**. |
| `OfflineSyncCompleted` | Aviso de que mensagens **pendentes offline** foram sincronizadas após voltar à internet. |
| `OfflineSyncPreview` | Aviso com **resumo do que ainda falta** sincronizar antes de concluir o offline sync. |

### Chamadas

| Evento | O que é este evento |
|---|---|
| `CallOffer` | Aviso de **chamada de voz ou vídeo recebida**. |
| `CallAccept` | Aviso de que a chamada foi **atendida**. |
| `CallTerminate` | Aviso de que a chamada **terminou** (desligou). |
| `CallOfferNotice` | Aviso/lembrete de chamada (notificação de chamada sem necessariamente atender). |
| `CallRelayLatency` | Dados técnicos de **latência** da chamada (uso avançado / diagnóstico). |

### Privacidade e perfil

| Evento | O que é este evento |
|---|---|
| `PrivacySettings` | Aviso de alteração nas **configurações de privacidade** (quem vê foto, status, etc.). |
| `PushNameSetting` | Aviso de que o **nome exibido** do seu número no WhatsApp mudou. |
| `UserAbout` | Aviso de que o **recado/sobre** de um contato foi atualizado. |
| `IdentityChange` | Aviso de que um contato **mudou de aparelho** (segurança/chave de identidade alterada). |
| `CATRefreshError` | Aviso de **erro interno** ao renovar credenciais de autenticação com o WhatsApp. |

### Canais (Newsletter)

| Evento | O que é este evento |
|---|---|
| `NewsletterJoin` | Aviso de que o número **passou a seguir** um canal do WhatsApp. |
| `NewsletterLeave` | Aviso de que o número **deixou de seguir** um canal. |
| `NewsletterMuteChange` | Aviso de que um canal foi **silenciado ou reativado**. |
| `NewsletterLiveUpdate` | Aviso de **nova publicação ou atualização ao vivo** em um canal que você segue. |

### Meta / contas Business

| Evento | O que é este evento |
|---|---|
| `FBMessage` | Aviso de mensagem recebida pela integração **Facebook / Meta** (cenários Business). |

### Receber todos de uma vez

| Evento | O que é este evento |
|---|---|
| `All` | Atalho para **inscrever em todos os eventos** da lista (no webhook por instância). No global, prefira deixar `KIRAGO_GLOBAL_WEBHOOK_EVENTS` vazio. |

### Exemplos — o que colocar no webhook global

| Se você quer… | Coloque em `KIRAGO_GLOBAL_WEBHOOK_EVENTS` |
|---|---|
| Só conversas privadas e leitura | `Message,ReadReceipt` |
| Atendimento + saber se caiu a conexão | `Message,ReadReceipt,Connected,Disconnected` |
| Incluir grupos | `Message,GroupMessage,GroupReadReceipt` |
| Monitorar login e QR | `Connected,Disconnected,LoggedOut,QR,QRTimeout,PairSuccess` |
| Tudo | *(deixe vazio)* |

---

## Webhook: instância × global

| | Webhook(s) da instância | Webhook global |
|---|---|---|
| **Escopo** | Uma instância (várias URLs) | Todas as instâncias do servidor |
| **Quantidade** | Ilimitado via `GET/POST /webhooks` | Uma URL no `.env` |
| **Como configurar** | Painel **Gerenciar Webhooks** ou API `/webhooks` | `.env`: `KIRAGO_GLOBAL_WEBHOOK` + `KIRAGO_GLOBAL_WEBHOOK_EVENTS` |
| **Nomes dos eventos** | [Tabela acima](#eventos-de-webhook--o-que-é-cada-um) | Mesma tabela |
| **Receber tudo** | `"events": ["All"]` em um ou mais webhooks | `KIRAGO_GLOBAL_WEBHOOK_EVENTS` vazio |
| **Disparo** | POST para cada URL ativa que assina o evento | POST único no servidor global |
| **Pausar sem apagar** | `"active": false` no webhook | — |
| **Payload** | Dados do evento | Igual + `userID` e `instanceName` |
| **HMAC** | Por instância | `KIRAGO_GLOBAL_HMAC_KEY` |
| **Retry automático** | Sim (por URL na outbox) | Não |
| **API legada** | `POST /webhook` = webhook **Principal** | — |

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

### Listar webhooks da instância

```bash
curl -s -X GET \
  -H "token: SEU_TOKEN" \
  "{BASE_URL}/webhooks"
```

### Criar webhook (ex.: CRM + N8N)

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"name":"N8N","webhookurl":"https://n8n.exemplo.com/webhook/abc","events":["Message","GroupMessage"],"active":true}' \
  "{BASE_URL}/webhooks"
```

### Configurar webhook Principal (legado)

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"webhookurl":"https://seuapp.com/webhook","events":["Message","ReadReceipt","Connected"]}' \
  "{BASE_URL}/webhook"
```

### Ativar pool Webshare em uma instância

```bash
curl -s -X POST \
  -H "token: SEU_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"enable":true,"pool_id":"ID_DO_POOL_NO_ADMIN"}' \
  "{BASE_URL}/session/proxy/webshare"
```

### Cadastrar pool Webshare (admin)

```bash
curl -s -X POST \
  -H "Authorization: SEU_KIRAGO_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"name":"Conta principal","api_key":"SUA_WEBSHARE_API_KEY","scheme":"http"}' \
  "{BASE_URL}/admin/proxy/webshare/pools"
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

## Licença e atualizações (assinatura)

O KiraGo é licenciado por chave (`KIRAGO_LICENSE_KEY`). Com a **assinatura ativa**, você mantém acesso à API e às **atualizações** dentro da janela do seu plano.

- **Dentro da janela:** você pode instalar as versões novas conforme o canal da sua assinatura (atualização automática, imagem enviada pelo suporte, etc.).
- **Fora da janela:** o servidor continua funcionando na versão instalada; novas releases podem exigir renovação para aplicar.
- **No painel:** o badge de versão avisa quando existe release mais recente — consulte o [CHANGELOG](./CHANGELOG.md) para ver o que mudou.

Se você hospeda por conta própria e sua assinatura inclui atualização de imagem, siga as orientações do suporte ou do canal onde recebe o KiraGo (não é necessário build manual para uso normal).

## Notas de versão

Cada release está documentada em [CHANGELOG.md](./CHANGELOG.md). Releases publicadas: [GitHub — Kira-Go](https://github.com/luiis716/Kira-Go/releases).

**Versão atual (documentada):** [1.7](./CHANGELOG.md#17---2026-06-10) — observação por instância, múltiplos webhooks, pools Webshare no admin e painel renovado.

## Suporte

Se precisar de ajuda, informe:

- Seu `BASE_URL`
- O endpoint utilizado
- O erro retornado pela API (sem expor o token)
