# Changelog 📌

Notas de versão do KiraGo para **assinantes** — o que mudou no produto, no painel e na API do dia a dia.

Cada release também pode ser consultada em [GitHub — Kira-Go](https://github.com/luiis716/Kira-Go/releases).

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
