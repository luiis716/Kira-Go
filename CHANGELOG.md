# Changelog 📌

Formato inspirado no *Keep a Changelog*: cada versão lista mudanças em **Added / Changed / Fixed**.

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
