# Changelog 📌

Notas de versão do KiraGo para **assinantes** — o que mudou no produto, no painel e na API do dia a dia.

Cada release também pode ser consultada em [GitHub — Kira-Go](https://github.com/luiis716/Kira-Go/releases).

## [1.7] - 2026-05-28

### O que há de novo 🚀

**KiraGo 1.7 — Observação por instância**

- **Observação interna** em cada instância (cliente, proxy, lembrete, etc.) — só no painel, não vai pro WhatsApp.
- **Lista:** chip **Nota** e prévia no card quando houver texto.
- **Dentro da instância:** **Ferramentas → Observação** para editar.
- **Nova instância:** campo opcional na aba **Essencial**.

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
