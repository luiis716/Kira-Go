# Changelog 📌

Formato inspirado no *Keep a Changelog*: cada versão lista mudanças em **Added / Changed / Fixed**.

## [1.5] - 2026-05-04 (pre-release)

### Release notes 🚀

**Atualização KiraGo (Webhook global + documentação) — Versão 1.5**

Release focada em **controle fino de eventos no webhook global** e em **documentação de eventos em português** no README, para operação e integração mais previsíveis.

#### Destaques

- **Filtro opcional no webhook global** — reduz ruído e carga no endpoint central quando você só precisa de um subconjunto de eventos (ex.: mensagens e sessão).
- **README expandido** — todos os tipos de evento suportados documentados com *o que dispara*, agrupados por categoria, mais tabela **instância × global** (escopo, filtro, HMAC, retry).

---

### Added ✅

- **Webhook global — filtro de eventos**
  - Variável de ambiente `KIRAGO_GLOBAL_WEBHOOK_EVENTS`: lista separada por **vírgula**, mesmos nomes de evento do webhook por instância (`Message`, `GroupMessage`, `Connected`, …).
  - Flag de linha de comando **`-globalwebhookevents`** com o mesmo formato (alternativa ao `.env`).
  - **Comportamento:** string **vazia** ou ausente = **sem filtro** (encaminha **todos** os eventos, como antes).
  - **Precedência:** se a flag **não** estiver vazia, ela vale; caso contrário usa-se `KIRAGO_GLOBAL_WEBHOOK_EVENTS` do ambiente.
  - Nomes de evento **desconhecidos** na lista são **ignorados** (apenas tipos válidos entram no filtro); conferir logs na subida para ver o filtro efetivo.

- **README**
  - Seção **Eventos de webhook** com tabelas por categoria (mensagens, grupos, sessão, QR, presença, sync, chamadas, newsletter, etc.).
  - Tabela comparativa **Webhook da instância** × **Webhook global** (configuração, filtro, payload com `userID`/`instanceName`, HMAC, retry).
  - Referência à variável `KIRAGO_GLOBAL_WEBHOOK_EVENTS` na tabela de variáveis de ambiente.

- **`kirago/.env.sample`**
  - Chaves `KIRAGO_GLOBAL_WEBHOOK` e `KIRAGO_GLOBAL_WEBHOOK_EVENTS` documentadas no exemplo.

#### Exemplo rápido (`.env`)

```env
KIRAGO_GLOBAL_WEBHOOK=https://seu-servidor.com/hook
KIRAGO_GLOBAL_WEBHOOK_EVENTS=Message,GroupMessage,Connected
```

Equivalente via CLI (quando preferir não usar `.env` para isso):

```text
-globalwebhookevents=Message,GroupMessage,Connected
```

---

### Changed ♻️

- ♻️ Documentação: README passa a ser a referência principal para **nomes e significados** dos eventos em PT‑BR (alinhado ao painel e ao `POST /webhook`).

### Notas para quem faz deploy

- O webhook global continua exigindo `KIRAGO_GLOBAL_WEBHOOK` (URL); o filtro **só** restringe **quais** eventos são enviados.
- HMAC global (`KIRAGO_GLOBAL_HMAC_KEY`) e diferenças de retry em relação ao webhook por instância permanecem como descrito no README.

## [1.4] - 2026-04-14

### Release notes 🚀

**Atualização KiraGo (CRM + Typebot + mídias + dashboard) — Versão 1.4**

Release focada em **integrações** (CRM e Typebot), **mídia/interativos** (incluindo GIF), **enquetes** e **usabilidade do painel**, com melhorias gerais de estabilidade e documentação.

### Added ✅

- **CRM (KiraGo):** configuração por instância (URL + token); envio inbound padronizado (texto, mídias, reação); sincronização de histórico em lotes (`HistoryBatch` / `POST /session/crm/sync-history`).
- **Typebot:** fluxo 1:1 automático (mensagem → Typebot → resposta no WhatsApp); gatilhos por palavra-chave/operador; palavras de parada e controle de sessão.
- **Mídia / interativos:** endpoint `POST /chat/send/gif`; GIF em headers de mensagens interativas (botões/carrossel) com conversão para MP4 quando necessário; melhor compatibilidade para imagem, vídeo, áudio e documento.
- **Enquetes:** envio de poll via helper alinhado ao motor; melhor tratamento de **votos** no webhook.

### Changed ♻️

- ♻️ **Dashboard:** melhorias visuais e responsividade; UX em conexão / pair phone; organização de cards e integrações.
- ♻️ **Webhook / eventos / sessão:** ajustes de fluxo e tratamento de erros.
- ♻️ **Swagger / documentação:** continuidade de melhorias em PT‑BR.

### Notas 🐳

- Imagens Docker publicadas nesta linha: `ggdadds/kirago:1.4`, `ggdadds/kirago:latest` (conforme release no GitHub).

## [1.3] - 2026-01-03

### Release notes 🚀

**Atualização KiraGo (Chatwoot + Typebot + logs + webhook) — Versão 1.3**

Release focada em **automação de conversa** (Chatwoot e Typebot), **painel com logs**, **webhook mais resiliente** e atualização do motor WhatsApp.

### Added ✅

- **Chatwoot:** envio e recebimento de mensagens; reações; respostas (*reply*) com melhor mapeamento de mensagens; suporte a histórico e sincronização.
- **Typebot:** fluxo automático entre WhatsApp e Typebot; configuração pelo painel; `public_id`, `base_url`, token e modo preview; gatilhos por palavra-chave e modos avançados; controle de expiração de sessão; mensagem de fallback; opções para reiniciar fluxo, manter sessão e responder no próprio número; mídia retornada pelo Typebot.
- **Painel:** visualizador de logs de webhook (consulta de eventos, erros e entregas; melhor rastreabilidade).
- **Webhook:** fila interna para entrega; **retry** com *backoff* exponencial; menos perda em falhas temporárias; envio em segundo plano mais controlado.

### Changed ♻️

- ♻️ **Docker:** build corrigido e estabilizado; melhor compatibilidade na geração da imagem.
- ♻️ **Whatsmeow:** motor WhatsApp atualizado (conexão e eventos).
- ♻️ Compatibilidade com banco e inicialização; organização dos fluxos internos de eventos entre painel, webhook, Chatwoot e Typebot.

### Fixed 🛠️

- 🛠️ Ajustes de integração entre painel, webhook, Chatwoot e Typebot (consolidação da linha 1.3).

### Variáveis de ambiente (opcional) ⚙️

- `WEBHOOK_RETRY_ENABLED` (`true` / `false`)
- `WEBHOOK_RETRY_COUNT` (ex.: `5`)
- `WEBHOOK_RETRY_DELAY_SECONDS` (ex.: `30`)
- `WEBHOOK_ERROR_QUEUE_NAME` (ex.: `webhook_errors`)

## [1.2] - 2025-12-24

### Release notes ✨

Atualização KiraGo (Painel + Webhook + Swagger) 🚀 Versão 1.2

- 🔔 Webhook: botão “Ativo/pausado” (para de enviar sem apagar URL/eventos).
- 👥 Webhook: separação de eventos de grupos (ex.: `GroupMessage`, `GroupReadReceipt`) mantendo a mesma URL.
- 🧩 Painel: tela da instância mais moderna e responsiva (cards, ações rápidas e layout melhor).
- 📚 API (/api): Swagger mais amigável em PT‑BR e tema escuro consistente (inclui modal “Autorizar”).
- 🧪 `develop.html`: protótipo de Gerenciador de Intenções (para desenvolvimento).

### Added ✅
- ✅ Webhook: flag `webhook_active` com persistência (pausar/retomar envio sem desconfigurar).
- ✅ Webhook: novos tipos de evento para grupos e campo `isGroup` em confirmações.
- ✅ Swagger: “Ajuda rápida” e melhorias de usabilidade (filtro, persistência de autorização, labels PT‑BR).

### Changed ♻️
- ♻️ Webhook: envio agora respeita `webhook_active` (quando desativado, não dispara).
- ♻️ Painel: reorganização visual da página da instância (mais clean e profissional).

### Fixed 🛠️
- 🛠️ Banco: compatibilidade ao iniciar com banco antigo (cria a coluna `webhook_active` quando faltar).
- 🛠️ Swagger: correções de contraste em modo escuro (“No parameters”, cabeçalhos e modal de autorização).

## [1.1] - 2025-12-20

### Release notes 🚀

Atualização KiraGo (Painel/WhatsApp) — Versão 1.1

- 🔔 Webhook: modal reorganizado (URL em primeiro) e eventos por categorias em checkboxes (bem melhor no tema escuro).
- 🧩 Criar instância: seleção de eventos do webhook também virou checkboxes.
- 🔗 Conexão: botão “Desconectar” funciona mesmo quando está só no QR Code (antes de logar).
- 🗂️ S3/Mídia: corrigido bug de cache que às vezes ignorava `media_delivery` (base64/s3/both).
- 🔒 Licença/Atualizações: validação da janela de updates usando a data do build.

### Changed ♻️
- Dashboard: seleção de eventos do webhook passou de *select* para checkboxes (categorias + tema escuro).
- Modais: reorganização do webhook (URL em primeiro) e ajuste de layout.
- Licença/updates: validação por data do build e janela de updates do plano.

### Fixed 🛠️
- Sessão: desconectar funciona mesmo durante QR/login (pairing).
- Mídia/S3: cache passou a considerar `media_delivery`/`s3_enabled` corretamente.
