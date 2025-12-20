# Changelog

Este repositório público é focado em **documentação** e **notas de versão** do KiraGo.

O formato segue a ideia do *Keep a Changelog*: cada versão lista mudanças em **Added / Changed / Fixed**.


## [1.1] - 2025-12-20

### Release notes

Atualização KiraGo (Painel/WhatsApp) 🚀 Versão 1.1

Pessoal, liberamos uma atualização com melhorias no painel e correções importantes ✅

- Webhook 🔔: modal reorganizado (URL em primeiro) e eventos por categorias em checkboxes (bem melhor no tema escuro 🌙).
- Criar instância 🧩: seleção de eventos do webhook também virou checkboxes.
- Conexão 🔗: botão “Desconectar” agora funciona mesmo quando está só no QR Code 📲 (antes de logar).
- S3/Mídia 🗂️: corrigido um bug de cache que às vezes ignorava o `media_delivery` (agora respeita base64/s3/both certinho).
- Licença/Atualizações 🔒: validação da janela de updates usando a data do build (bloqueia/permite versões conforme o plano).

Para atualizar, use a imagem `ggdadds/kirago:latest` ou `ggdadds/kirago:1.1`.

Se alguém notar algo estranho após atualizar, faz um hard refresh (Ctrl+Shift+R) 🔄 e me chama com print/log 🧾.

### Changed
- Dashboard: seleção de eventos do webhook passou de *select* para checkboxes (categorias + melhor tema escuro).
- Modais: reorganização do webhook (URL em primeiro) e ajuste de layout.
- Licença/updates: validação por data do build e janela de updates do plano.

### Fixed
- Sessão: desconectar funciona mesmo durante QR/login (pairing).
- Mídia/S3: cache passou a considerar `media_delivery`/`s3_enabled` corretamente.
