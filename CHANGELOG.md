# Changelog 📌

Notas de versão do KiraGo para **assinantes** — o que mudou no produto, no painel e na API do dia a dia.

Cada release também pode ser consultada em [GitHub — Kira-Go](https://github.com/luiis716/Kira-Go/releases).

## [1.17] - 2026-11-07

### O que há de novo 🚀

**KiraGo 1.17 — Proxy pelo celular (app + QR) e motor WhatsApp mais estável**

#### Principais melhorias

- **KiraGo Proxy** — modo de proxy no painel (junto com Webshare e manual) para a sessão WhatsApp sair pelo **IP do celular** do cliente, com consentimento.
- **QR no painel** — o celular lê o QR e conecta sozinho (sem ficar digitando host/token).
- **EasyPanel / Portainer** — no HTTPS do subdomínio o app usa `wss://…/mobile-proxy/tunnel` (sem abrir porta TCP extra).
- **Instalar app** — seletor no painel da instância: **PWA** (notificações) ou **APK do Proxy**.
- **App Proxy** — visual KiraGo, várias instâncias no mesmo celular, nome/avatar do WhatsApp quando a sessão conectar.
- **App sempre ativo** — reconecta sozinho quando a rede volta; religa após reboot/kill; pede liberação de bateria só se o Android restringir.
- **Motor WhatsApp** — app state, recibos, keepalive/reconnect e histórico em chats de bot mais estáveis (cliente WA Web **v1052556849**).

---

### Novidades ✅

#### Proxy (painel)

- Novo modo **KiraGo Proxy — app no celular**, no mesmo fluxo do Webshare.
- Status ao vivo: celular online/offline, tráfego ↑/↓ e conexões ativas.
- QR de pareamento com host, porta/path, token e nome da instância.
- Em painéis abertos por HTTPS (EasyPanel/Portainer/Traefik), o QR usa WSS no mesmo domínio.
- Status mobile inclui nome da instância, nome/JID do WhatsApp e caminho do avatar (o app busca depois que a sessão logar).

#### App Android (KiraGo Proxy)

- Baixe em **Instalar app → KiraGo Proxy (APK)** (`/downloads/kirago-proxy.apk`).
- Visual alinhado ao painel KiraGo; lista de **várias instâncias** (QR adiciona sem derrubar as outras).
- Card com **nome da instância**, **nome do WhatsApp** e **avatar** (preenche quando o WhatsApp conectar).
- Tráfego em tempo real, WSS no subdomínio público e keepalive.
- **Auto-reconnect** se cair a rede do celular; restaura túneis se o Android matar o app ou após reiniciar o aparelho.
- Aviso de **segundo plano** só aparece se o Android restringir o app; some depois de permitir (evita desligar de madrugada).

#### Servidor / Docker

- Relay embutido no container quando o túnel aponta para `127.0.0.1` (um serviço só no EasyPanel).
- Variáveis opcionais: `KIRAGO_MOBILE_PROXY_*` (SOCKS, status, túnel, host público, embed).
- Endpoint público de túnel: `GET /mobile-proxy/tunnel` (WebSocket).
- API de sessão: `GET/POST /session/proxy/mobile`.

#### Conexão WhatsApp

- App state mais tolerante a erro em patch/sync de contatos.
- Mais formatos de recibo em grupo; logs melhores em decrypt/SKDM.
- Keepalive/reconnect sem sobrescrever disconnect concorrente; disconnect atômico no socket.
- Histórico em chats de bot com remetente (JID do chat) correto.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.17
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.17`, `update_available: false`).

No painel: instância → **Instalar app** → baixe o APK do Proxy → **Proxy** → modo app → leia o QR.

---

## [1.16] - 2026-11-01

### O que há de novo 🚀

**KiraGo 1.16 — Motor WhatsApp v1052556849 e vídeo em streaming**

#### Principais melhorias

- **Motor WhatsApp** atualizado — cliente alinhado ao WhatsApp Web **v1052556849**.
- **Vídeo** enviado pela API passa a incluir o sidecar de streaming, para o WhatsApp reproduzir enquanto o arquivo baixa.
- **Conexão** tenta de novo se o handshake não responde ou se o socket fecha no meio.

---

### Novidades ✅

#### Conexão WhatsApp

- Fingerprint do cliente na versão WA Web **v1052556849**.
- Handshake sem resposta, ou socket fechado durante a espera, entra na reconexão automática.

#### Mensagens

- Envio de vídeo no chat, no status, no cabeçalho interativo, no Chatwoot e no Typebot grava o `StreamingSidecar` gerado no upload.
- Número sem LID no WhatsApp passa a falhar com o erro de destinatário fora do WhatsApp.

#### Contatos e log

- Quando o mesmo contato chega duplicado, o nome guardado é o mais novo.
- O log deixa de imprimir token de mídia e de ligação.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.16
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.16`, `update_available: false`).

---

## [1.15] - 2026-10-21

### O que há de novo 🚀

**KiraGo 1.15 — Motor WhatsApp v1050728964, encaminhamento de canal e marco**

#### Principais melhorias

- **Motor WhatsApp** atualizado — cliente alinhado ao WhatsApp Web **v1050728964**.
- **Canal** — mensagens e o live update passam a trazer a contagem de encaminhamentos (`ForwardsCount`).
- **Webhook** `NewsletterMilestone` quando o canal atinge um marco (visualizações ou reação).
- **Envio privado** — se o hash de participantes não bater, o cache de dispositivos é invalidado e o envio segue para os aparelhos certos.
- **Conexão** — leitura de stanza mais tolerante e refresh de CAT só quando o callback está configurado.

---

### Novidades ✅

#### Conexão WhatsApp

- Fingerprint do cliente na versão WA Web **v1050728964**.
- Decode de stanza ignora erro de tipo na tag, em vez de derrubar o pacote.
- Falha de CAT inválido ou expirado só dispara refresh se houver callback.

#### Canais

- `ForwardsCount` nas mensagens do canal e no evento `NewsletterLiveUpdate`.
- Evento `NewsletterMilestone` (painel: “Marco” / “Marco do canal”), com tipo, valor e código de reação.

#### Mensagens

- Conversa privada: cache de dispositivos descartado quando o servidor recusa o hash de participantes, para o próximo envio alcançar os aparelhos atualizados.
- Número de fora do Brasil (DDI diferente de `55`, por exemplo `+971529846014`) é enviado mesmo quando a consulta ao WhatsApp não confirma o cadastro. Número com `55` continua bloqueado se o WhatsApp disser que não existe.

#### Grupos

- Na subida, se o banco antigo não tiver a coluna `sender_timestamp` em `whatsmeow_privacy_tokens`, a API cria a coluna e o índice. Sem isso, criar grupo falhava antes de chamar o WhatsApp.
- `POST /group/create` devolve o erro real quando a criação falha.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.15
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.15`, `update_available: false`).

---

## [1.14] - 2026-10-07

### O que há de novo 🚀

**KiraGo 1.14 — Motor WhatsApp v1049558099, blocklist com LID e erro de sync**

#### Principais melhorias

- **Motor WhatsApp** atualizado — cliente alinhado ao WhatsApp Web **v1049558099**.
- **Blocklist** — `GET /user/blocklist` e `POST /user/block` usam `dhash` e devolvem `Items` (LID + telefone) além de `JIDs`.
- **Webhook** `AppStateSyncError` quando a sincronização de app state falha.
- **Conexão** — correção de frame partido no socket e retry de recibo mais estável.
- **Bots Muse/WASA** — secret ativo e cifra ao enviar para o bot Hatch.

---

### Novidades ✅

#### Conexão WhatsApp

- Fingerprint do cliente na versão WA Web **v1049558099**.
- Frame de socket com tamanho certo quando a mensagem vem partida.
- Recibo de retry tratado de forma síncrona quando o ack síncrono está ligado.

#### Usuários

- Blocklist nova: cada item traz `LID`, `PN` e `Active`.
- `JIDs` continua na resposta (telefone quando existe, senão LID) para quem já lia a lista antiga.

#### Webhooks

- Evento `AppStateSyncError` (painel: “Erro de sync” / “Erro de sincronização”).

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.14
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.14`, `update_available: false`).

---

## [1.13] - 2026-09-22

### O que há de novo 🚀

**KiraGo 1.13 — Motor WhatsApp no tip do fork + link preview com imagem em canais**

#### Principais melhorias

- **Motor WhatsApp** atualizado — cliente alinhado ao WhatsApp Web **v1047769893**.
- **Link preview** — Open Graph com User-Agent (Mercado Livre etc. sem 403); thumbnail via MMS (`MediaLinkThumbnail`); em `@newsletter` usa `UploadNewsletter` + `MediaHandle`.
- **Bots / WASA / pair** — root secrets, decrypt de bot e refresh de QR/ADV do upstream.
- **Deps** do motor atualizadas (`go.mau.fi/util`, crypto/net).

---

### Novidades ✅

#### Conexão WhatsApp

- Fingerprint do cliente na versão WA Web **v1047769893**.
- Melhorias herdadas do tip: pair `companion_reg_refresh`, handshake com payload nil, `SendResponse.Chat`.

#### Mensagens e mídia

- **`POST /chat/send/text` + `LinkPreview: true`**:
  - busca Open Graph com User-Agent de navegador;
  - gera thumbnail JPEG e sobe via `MediaLinkThumbnail`;
  - em canal (`@newsletter`), upload newsletter + `MediaHandle` para a capa aparecer.
- Download de sticker pack com nil-check; tratamento de mídia/bot mais estável.

#### Grupos

- `SetGroupDescription` alinhado ao fluxo de topic do protocolo atual.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.13
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.13`, `update_available: false`).

---

## [1.12] - 2026-09-07

### O que há de novo 🚀

**KiraGo 1.12 — Motor WhatsApp atualizado, botões native flow e assistente no painel**

#### Principais melhorias

- **Reconnect mais confiável** — erros de stream são tratados antes de reconectar, evitando sessão “presa”.
- **Noise socket** — context correto nos frames (mais estabilidade na conexão).
- **Histórico / ParseWebMessage** — identificação correta do próprio número (own ID).
- **Download de thumbnails** — ignora `mediaKey` inválida em mídia sem criptografia (menos falha ao baixar preview).
- **Grupos** — motivo de exclusão de membro fica opcional.
- **Protos** atualizados para o WhatsApp Web recente (v1046691727).
- **Botões (`/chat/send/buttons`)** alinhados à referência native flow (reply legado, CTAs, webview).
- **Assistente no dashboard** — bolinha com templates de JSON; com API key, também Groq/OpenAI/Anthropic.
- **Webhook** — posts de canal saem como `NewsletterMessage` (não mais como `Message` privado).
- **Canais** — envio para `@newsletter` em `/chat/send/*` deixa de falhar na validação de telefone.

---

### Novidades ✅

#### Conexão WhatsApp

- Stream error processado antes do auto-reconnect.
- Context correto no handshake/noise socket.
- Cliente alinhado à versão do protocolo (v1046691727).

#### Mensagens e mídia

- Correção do own ID ao interpretar mensagens web/histórico.
- Download de thumbnail sem chave de criptografia inválida.
- **`POST /chat/send/buttons`**:
  - reply puro (máx. 3) via `ButtonsMessage` legado (melhor render no iOS/Android);
  - CTAs (url/copy/call/webview/PIX) via `InteractiveMessage` direto;
  - validação: reply não mistura com CTA/PIX;
  - webview em botão URL (`ButtonWebview` / `WebviewPresentation` / `Type=webview`);
  - stanza `<native_flow v="9" name="mixed">` (sem `biz_bot` em contas pessoais).

#### Webhooks

- Posts de **canal/newsletter** (`@newsletter`) saem como evento dedicado `NewsletterMessage` (não mais como `Message` privado). Opção na setagem do webhook / push. Continuam: `NewsletterJoin` / `Leave` / `MuteChange` / `LiveUpdate`.
- Envio para `@newsletter` em `/chat/send/*` deixa de falhar na validação de telefone (`IsOnWhatsApp`).

#### Painel

- **Assistente KiraGo** (bolinha em `/dashboard/instances` e `/dashboard/instance`):
  - ligado por `KIRAGO_ASSISTANT_ENABLED=true`;
  - sem key: templates/FAQ (botões, webview, PIX, lista, carrossel…);
  - com `KIRAGO_ASSISTANT_API_KEY`: híbrido + LLM (`groq` | `openai` | `anthropic`).

#### Grupos

- Delete de participante aceita motivo opcional.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.12
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.12`, `update_available: false`).

---

## [1.11] - 2026-08-26

### O que há de novo 🚀

**KiraGo 1.11 — Motor WhatsApp mais estável, webhook com dados da instância e painel sem “modais fantasma”**

#### Principais melhorias

- **Motor WhatsApp atualizado** — fila de eventos por conexão (reconnect mais limpo), download de mídia sem chave inválida e protos alinhados ao WhatsApp Web recente.
- **Blocklist com LID** — `POST /user/block` usa o identificador LID do protocolo atual.
- **Webhook da instância** — além de `instanceName`, envia id, jid, telefone, status de conexão/login e nome do WhatsApp (`instance`).
- **Excluir e recriar instância** com o mesmo token deixa de abrir a tela vazia (cache de auth limpo de verdade).
- **Painel** — modal de licença não aparece mais atrás de “Excluir” nem ao clicar em Voltar.

---

### Novidades ✅

#### Conexão WhatsApp

- Handler queue criada por conexão — evita misturar eventos entre sessões após reconnect.
- Download de mídia ignora `mediaKey` quando a mídia não está criptografada (menos erro ao baixar).
- Protos e dependências do motor atualizados (Go 1.26 no build).

#### Usuários / privacidade

- Atualização da blocklist (`UpdateBlocklist`) passando a usar LID (+ `pn_jid` quando necessário).

#### Webhooks

- Payload enriquecido com dados da instância (`userID`, `instanceName`, `instanceJid`, `instancePhone`, `instanceConnected`, `instanceLoggedIn`, `instanceOwner` e objeto `instance`).
- O **token da instância não é enviado** no webhook.
- Vale para webhook da instância, webhook global e RabbitMQ.

#### Painel admin

- Delete de instância limpa token em cache, cliente WhatsApp e arquivos (recriar com o mesmo token funciona).
- Modal de licença só abre quando você clica nele — não “vaza” atrás de exclusão nem ao voltar da instância.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.11
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.11`, `update_available: false`).

---

## [1.10] - 2026-08-10

### O que há de novo 🚀

**KiraGo 1.10 — Motor WhatsApp atualizado: grupos, recado e estabilidade LID**

#### Principais melhorias

- **Motor WhatsApp atualizado** — compatibilidade alinhada à versão mais recente do WhatsApp Web.
- **Criação de grupos** mais alinhada ao protocolo atual (incluindo modo de adição de membros).
- **Recado / Sobre** (`POST /status/set/text`) atualizado para o fluxo novo do WhatsApp.
- **Mapeamento LID ↔ número** e tokens de privacidade mais estáveis no histórico.

---

### Novidades ✅

#### Conexão WhatsApp

- Versão do cliente alinhada à mais recente, para manter a sessão estável com o WhatsApp.

#### Grupos

- Criação de grupo com parâmetros atualizados e correção do modo de adicionar membros.

#### Perfil

- Alteração do texto de status (Sobre) compatível com o fluxo atual do WhatsApp.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.10
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.10`, `update_available: false`).

---

## [1.9] - 2026-07-27

### O que há de novo 🚀

**KiraGo 1.9 — Motor WhatsApp atualizado e notificações só no celular / app instalado**

#### Principais melhorias

- **Motor WhatsApp atualizado** — compatibilidade alinhada à versão mais recente do WhatsApp Web.
- **Notificações no painel** — o card de notificações aparece **somente** no celular (navegador) ou quando o KiraGo está instalado como app. No computador (navegador normal), a opção fica oculta.

---

### Novidades ✅

#### Painel — notificações

- Card **Notificações** visível apenas em:
  - **celular / tablet** (navegador), ou
  - **app instalado** (tela inicial).
- Em desktop sem o app instalado, a opção não é exibida (use o celular ou instale o app em **Configurações → Instalar app**).

#### Conexão WhatsApp

- Versão do cliente alinhada à mais recente, para manter a sessão estável com o WhatsApp.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.9
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.9`, `update_available: false`). No painel, use **Ctrl+Shift+R** para carregar a tela nova das notificações.

---

## [1.8.2] - 2026-07-22

### O que há de novo 🚀

**KiraGo 1.8.2 — Motor WhatsApp atualizado: criar grupo com LID, receipts e estabilidade**

#### Principais melhorias

- **Motor WhatsApp atualizado** — criar grupo mais estável quando o contato usa LID, evento de entrada em grupo mais preciso e melhor tratamento de confirmações de entrega.
- **Versão do cliente WhatsApp** alinhada à mais recente.

---

### Novidades ✅

#### Grupos

- **`POST /group/create`** — funciona melhor com participantes em formato LID (o sistema associa o número automaticamente).
- Evento webhook **`JoinedGroup`** — informação de endereço do grupo mais completa.

#### Entrega de mensagens

- Confirmações de entrega / retry mais confiáveis (menos mensagem “travada” ou reenvio inconsistente).

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.8.2
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.8.2`, `update_available: false`).

---

## [1.8.1] - 2026-07-16

### O que há de novo 🚀

**KiraGo 1.8.1 — Motor WhatsApp atualizado (LID), passkey no painel e `/user/check` com número PN**

#### Principais melhorias

- **Motor WhatsApp (whatsmeow) atualizado** — proto **v1043045417**, envio de DM sempre via LID, checagem de número mais estável e reject de chamada corrigido.
- **Passkey no painel** — se o WhatsApp pedir verificação após o QR, o modal aparece sozinho (autenticar / confirmar código). QR e código por telefone **não mudam**.
- **`POST /user/check`** — passa a devolver `PhoneNumber` (PN `@s.whatsapp.net`) quando o `JID` vier como `@lid`.

---

### Novidades ✅

#### Motor WhatsApp

- Proto / versão do cliente: **v1043045417**.
- **Envio de DM** — destino convertido para LID automaticamente (melhor entrega no modelo atual do WhatsApp).
- **`IsOnWhatsApp` / `/user/check`** — query LID, mapeamento LID↔PN e campo **`PhoneNumber`** na resposta.
- **Reject de chamada** — remove `tctoken` que podia falhar o rejeite (skip automático e `POST /call/reject`).
- Pareamento: `PairSuccess` / `PairError` podem incluir props do cliente (sem mudança obrigatória no seu webhook).

#### Painel — passkey

- Modal automático quando `status.passkey` está em `request` ou `confirmation`.
- Botão **Autenticar com passkey** (WebAuthn no navegador, HTTPS) e **Confirmar pareamento** com o código `XXXX-X`.
- Se `skipHandoffUX` vier ativo, a confirmação pode ser enviada sem mostrar o código.

#### API

- **`POST /user/check`** — `Users[].PhoneNumber` (opcional); `JID` pode ser `@lid`.
- Swagger (`/api`) atualizado com o exemplo LID/PN.

### Correções 🔧

- **Badge de atualização no painel** — compara a versão instalada com a do GitHub de forma mais estável (sem aviso falso de “nova versão” quando você já está atualizado).

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.8.1
# ou
docker pull ggdadds/kirago:latest
```

Reinicie o container e confira `GET /health` (`version` ≈ `1.8.1`, `update_available: false`). No painel, use **Ctrl+Shift+R** para carregar a tela nova do passkey.

---

## [1.8] - 2026-07-01

### O que há de novo 🚀

**KiraGo 1.8 — Pareamento passkey, motor WhatsApp atualizado e instância com pool Webshare em um passo**

#### Principais melhorias

- **Pareamento por passkey (WebAuthn)** — suporte ao novo fluxo do WhatsApp além do QR; API e webhooks para integrar no seu front-end.
- **Motor WhatsApp (whatsmeow) atualizado** — proto mais recente, pareamento QR mais estável e preparação para contas que exigem passkey.
- **Criar instância já com pool Webshare** — em um único `POST /admin/users` com `webshareConfig`; o sistema escolhe um IP livre do pool na hora.
- **Documentação Swagger (`/api`)** — endpoints de pools Webshare, proxy por instância e passkey documentados no painel API.

---

### Novidades ✅

#### Pareamento passkey

- **API** — `GET /session/passkey`, `POST /session/passkey/response`, `POST /session/passkey/confirm`.
- **Webhooks** — `PairPasskeyRequest`, `PairPasskeyConfirmation`, `PairPasskeyError` (assináveis como os demais eventos).
- **Status** — `GET /session/status` inclui campo `passkey` com a fase atual (`request`, `confirmation` ou vazio).
- Fluxo QR tradicional **continua igual**; passkey é caminho alternativo quando o WhatsApp solicitar.

#### Proxy Webshare — criação de instância

- **`POST /admin/users`** aceita `webshareConfig: { "enabled": true, "pool_id": "..." }` — vincula pool e proxy na criação (sem 3º passo).
- **Painel — Nova instância:** ao ativar proxy, escolha **Pool Webshare** (recomendado) ou **URL manual**; lista os pools cadastrados pelo admin.
- Não use `proxyConfig` e `webshareConfig` juntos na mesma requisição.

#### API / Swagger (`/api`)

- **Admin** — `GET/POST /admin/proxy/webshare/pools`, `PUT/DELETE /admin/proxy/webshare/pools/{id}`.
- **Sessão** — `GET /session/proxy/pool-config`, `POST /session/proxy/webshare`.
- **Passkey** — rotas de pareamento documentadas com exemplos.
- **`POST /admin/users`** — descrição e exemplo atualizados para `webshareConfig`.

#### Motor WhatsApp

- Atualização para whatsmeow com suporte a **passkeys** e proto **v1042386815**.
- Evento **`NotifyAccountReachoutTimelock`** — alerta de restrição/timelock da conta (webhook).
- Pareamento QR: ignora `connect success` antes do pairing concluir (mais estável).
- Log de `PairSuccess` passa a incluir **LID**.

---

### Correções 🔧

- **Chatwoot** — evita criar várias caixas de entrada ao receber mensagens (reutiliza inbox existente; não recria em erro de rede).
- **Botões com imagem** — título no header, `ViewOnce` desligado em botões (evita sumir título / travar WhatsApp Web).
- **Download de imagem em botões** — contorna bloqueio 403 de CDN no envio server-side.
- **Modal de licença** — PIX visível para renovação antecipada; QR some após confirmação do pagamento; modal com rolagem em telas menores.

### Melhorias gerais ♻️

- Helper interno `applyWebsharePoolToUser` — mesma lógica de atribuição de IP na criação da instância e em `POST /session/proxy/webshare`.
- README e exemplos de API alinhados ao fluxo pool → instância em um ou dois passos.

---

## [1.7] - 2026-06-10

### O que há de novo 🚀

**KiraGo 1.7 — Observação por instância, múltiplos webhooks, pools Webshare e painel renovado**

#### Principais melhorias

- **Observação interna** em cada instância (cliente, proxy, lembrete, etc.) — só no painel, não vai pro WhatsApp.
- **Vários webhooks por instância** — cada um com nome, URL, eventos e status ativo/pausado; o WhatsApp assina a **união** dos eventos de todos os webhooks ativos.
- **Pools Webshare nomeados (admin)** — cadastre várias contas Webshare no painel; cada instância escolhe qual pool usar no modal de Proxy.
- **Painel reorganizado** — lista de instâncias e página interna da instância; card **Webhooks** com resumo; modal **Gerenciar Webhooks** (lista, criar, editar, pausar, excluir).

---

### Novidades ✅

#### Observação por instância

- Chip **Nota** e prévia no card da lista quando houver texto.
- **Ferramentas → Observação** dentro da instância para editar.
- Campo opcional na aba **Essencial** ao criar nova instância.

#### Webhooks (multi)

- **API** — `GET/POST /webhooks` e `PUT/DELETE /webhooks/{id}` (header `token` da instância).
- Cada webhook: `name`, `url`, `events[]`, `active` (pausar sem apagar).
- **Compatibilidade:** rotas legadas `GET/POST/PUT/DELETE /webhook` continuam no webhook **Principal** (migrado automaticamente do campo único antigo).
- **Disparo:** cada evento vai para **todas** as URLs ativas que assinam aquele tipo; retry na outbox por URL.
- **Painel:** chips de eventos, atalhos (Mensagens, Conexão, Todos, Limpar), botões Salvar/Cancelar fixos no rodapé do modal.

#### Proxy — pools Webshare (admin)

- **API admin** — `GET/POST /admin/proxy/webshare/pools`, `PUT/DELETE /admin/proxy/webshare/pools/{id}`.
- Vários pools com **nome** e **API key**; esquema HTTP/SOCKS5 por pool.
- **Por instância:** `GET /session/proxy/pool-config` e `POST /session/proxy/webshare` com `pool_id`.
- **Painel admin:** modal **Pools Webshare** na lista de instâncias (criar, editar, trocar key, remover).
- Rotas legadas `GET/POST /admin/proxy/webshare` mantidas.

### Correções 🔧

- **Crash `concurrent map read and map write`** com várias requisições em paralelo no painel da instância.
- **Card Webhooks** não fica mais preso em *Carregando…* a cada poll de status.
- **Modal de webhooks** — rolagem e botões **Salvar** / **Adicionar** sempre visíveis.

### Melhorias gerais ♻️

- Documentação README atualizada (multi-webhook + pools).

---

## [1.6] - 2026-05-27

### O que há de novo 🚀

**KiraGo 1.6 — Mais estável no WhatsApp, painel mais completo e proxy Webshare integrado**

#### Principais melhorias

- **Grupos e mensagens mais estáveis** — promover, remover, adicionar participantes e enviar texto simples deixam de derrubar a sessão com tanta frequência.
- **Versão visível no painel** — número da versão na barra superior; aviso quando existe atualização mais nova; modal com novidades ao entrar no dashboard.
- **Proxy Webshare no painel** — configure só com a API key; o sistema escolhe o proxy; opção de compartilhar o pool entre instâncias sem repetir o mesmo IP ao mesmo tempo.
- **Avatares das instâncias** — fotos aparecem nos cards mesmo com proxy ativo; atualização automática após reiniciar o servidor.
- **Cards mais informativos** — selos (chips) mostram integrações ativas: Proxy, Webhook, Chatwoot, CRM, etc.

---

### Novidades ✅

- **Badge de versão** na navbar do dashboard (ao lado do botão API).
- **Aviso de atualização** quando uma versão mais nova está disponível (com link para as notas).
- **Modal de novidades** ao logar no painel (uma vez por sessão do navegador).
- **Proxy Webshare** no modal de Proxy de cada instância:
  - Modo **pool** (só API key) ou **manual** (host/porta/usuário).
  - **Compartilhar pool** com outras instâncias da mesma instalação.
  - **Forçar rotação** para trocar de IP no pool.
- **Avatares** carregados automaticamente nas instâncias conectadas.
- **Chips de integração** nos cards (Proxy, Webhook, Chatwoot, Typebot, CRM, RabbitMQ, S3, HMAC).
- **API de proxy Webshare** — `POST /session/proxy/webshare` e `GET /session/proxy/pool-config` (para automações que preferem API em vez do painel).
- **Motor WhatsApp atualizado** — melhor compatibilidade com o protocolo atual do WhatsApp Web.

### Correções 🔧

- **Desconexão ao gerenciar grupos** (adicionar, remover, promover, renomear, sair) — a sessão aguarda mais tempo a reconexão em vez de cair na hora.
- **Texto simples rejeitado pelo WhatsApp** — mensagens de texto sem link deixam de ser enviadas com tipo errado (`media/url`).
- **Avatares sumindo no painel** — especialmente quem usa proxy; imagens passam a carregar pelo servidor da instância.
- **Filtro de instâncias** no painel admin — não fica mais escondido atrás dos cards.
- **Modal de proxy** — interruptor não desliga sozinho ao abrir; lista de instâncias fonte do pool carrega corretamente.
- **Modal de novidades da versão** — rolagem e botões visíveis em telas menores.
- **Grupos com JID inválido** — uso de número sem `@g.us` em rotas de grupo passa a retornar erro claro em vez de derrubar a conexão.
- **Mensagem mais clara ao reconectar** — em alguns casos de instabilidade, a API responde pedindo para tentar de novo em alguns segundos (em vez de erro genérico).

### Melhorias gerais ♻️

- Painel mais rápido para identificar o que cada instância tem configurado (chips + avatares).
- Reconexão após falhas de rede mais tolerante em operações demoradas.

---

## [1.5] - 2026-05-04

### O que há de novo 🚀

**KiraGo 1.5 — Webhook global com filtro de eventos**

#### Principais melhorias

- **Filtro no webhook global** — envie só os eventos que importam (ex.: `Message`, `Connected`) para o seu endpoint central, em vez de receber tudo.
- **Documentação de eventos em português** — README com tabela de todos os eventos e o que cada um significa.

---

### Novidades ✅

- Variável `KIRAGO_GLOBAL_WEBHOOK_EVENTS` — lista de eventos separados por vírgula para o webhook global (vazio = todos, como antes).
- README com guia completo de eventos de webhook (instância × global).

### Exemplo (`.env`)

```env
KIRAGO_GLOBAL_WEBHOOK=https://seu-servidor.com/hook
KIRAGO_GLOBAL_WEBHOOK_EVENTS=Message,GroupMessage,Connected
```

## [1.4] - 2026-04-14

### O que há de novo 🚀

**KiraGo 1.4 — CRM, Typebot, GIF e painel renovado**

### Novidades ✅

- **CRM** por instância — URL + token; envio de mensagens recebidas/enviadas; sincronização de histórico em lotes.
- **Typebot** — fluxo automático 1:1 (mensagem → bot → resposta no WhatsApp); gatilhos e palavras de parada.
- **Envio de GIF** e melhorias em botões, carrossel, enquetes e mídias.
- **Dashboard** — layout mais moderno, pair phone e cards de instância reorganizados.

## [1.3] - 2026-01-03

### O que há de novo 🚀

**KiraGo 1.3 — Chatwoot, Typebot, logs no painel e webhook mais confiável**

### Novidades ✅

- **Chatwoot** — mensagens, reações, respostas e sincronização de histórico.
- **Typebot** — configuração pelo painel, gatilhos, sessão, fallback e mídia nas respostas.
- **Logs de webhook no painel** — consulte entregas e erros sem sair do dashboard.
- **Webhook com retry** — novas tentativas automáticas em falhas temporárias (`WEBHOOK_RETRY_*` no `.env`).

## [1.2] - 2025-12-24

### O que há de novo 🚀

**KiraGo 1.2 — Webhook pausável, eventos de grupo e painel renovado**

- Webhook **ativo/pausado** sem perder URL e eventos configurados.
- Eventos de **grupo** separados (`GroupMessage`, `GroupReadReceipt`, etc.).
- Painel da instância mais moderno; Swagger em PT‑BR com tema escuro.

## [1.1] - 2025-12-20

### O que há de novo 🚀

**KiraGo 1.1 — Painel e webhook mais fáceis de usar**

- Webhook com eventos em **checkboxes por categoria** (tema escuro).
- **Desconectar** funciona mesmo na tela de QR (antes de concluir o login).
- Correção no envio de mídia com S3/base64 conforme a configuração da instância.
- Licença e janela de atualizações validadas pela data do build do seu plano.
