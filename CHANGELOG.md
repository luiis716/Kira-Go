## O que há de novo 🚀

**KiraGo 1.8.2 — Motor WhatsApp atualizado: criar grupo com LID, receipts e estabilidade**

### Principais melhorias

- **Motor WhatsApp atualizado** — criar grupo mais estável quando o contato usa LID, evento de entrada em grupo mais preciso e melhor tratamento de confirmações de entrega.
- **Versão do cliente WhatsApp** alinhada à mais recente.

### Novidades ✅

#### Grupos
- **`POST /group/create`** — funciona melhor com participantes em formato LID.
- Evento webhook **`JoinedGroup`** — informação de endereço do grupo mais completa.

#### Entrega de mensagens
- Confirmações de entrega / retry mais confiáveis.

### Como atualizar ♻️

```bash
docker pull ggdadds/kirago:1.8.2
# ou
docker pull ggdadds/kirago:latest
