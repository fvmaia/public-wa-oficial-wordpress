# Parte 1 — Provisionamento de número na Meta (técnico)

**Público:** quem tem acesso ao painel Meta for Developers e ao servidor.
**Resultado:** número registrado e entregando mensagens ao servidor, pronto para ser cadastrado no plugin.

> Depois de concluir esta parte, repasse o **Phone Number ID** para quem fará a [Parte 2](conectando_numeros_2_atendimento.md).

---

## Antes de começar

| Requisito | Por quê |
|---|---|
| Número **sem WhatsApp ativo** (comum ou Business) | A Cloud API recusa números já em uso |
| Alguém disponível para **atender uma ligação** no número | O código de verificação vem por voz |
| **PIN de 6 dígitos** definido e guardado | Exigido para reinscrever o número no futuro |
| Verificação da empresa aprovada | Sem ela o limite é de 2 números; com ela, até 20 |

> ⚠️ Se o número estiver em uso hoje por um atendente, esta operação **tira o WhatsApp do celular dele** e **não migra o histórico**. Alinhe antes com a equipe — ver Parte 2.

---

## Passo 1 — Registrar o número

**developers.facebook.com** → app **UpCiga WA** → **WhatsApp → Etapa 2. Configuração da produção** → *Registre seu número de telefone* → **Adicionar novo número**

**1.1 Perfil do WhatsApp Business**

| Campo | Valor |
|---|---|
| Nome de exibição | Nome comercial (deve corresponder ao nome real da empresa) |
| Categoria | Ramo de atuação |
| Fuso horário | `America/Belem (GMT-03:00)` ou equivalente |

**1.2 Telefone e verificação**

Informe DDI + DDD + número e escolha o método.

> 💡 **Use "Ligação telefônica".** O SMS falha com `#2388002 — Falha ao verificar a qualificação do número`. Por ligação funciona.

Atenda, anote o código de 6 dígitos e informe.

**1.3 Registro na Cloud API**

A Meta pedirá um **PIN de 6 dígitos** que você define. Guarde-o no cofre de credenciais.

✅ Status deve ficar **"Inscrito"**.
📋 **Anote o `Phone Number ID`** gerado — é o que a Parte 2 precisa.
📋 Anote também o `WABA ID`, se for uma conta nova.

---

## Passo 2 — Assinar webhooks

Na mesma tela, ative o toggle **"Assinar webhooks"** ao lado do nome da conta WhatsApp Business.

> ⚠️ **Sem este passo o número funciona na Meta mas nenhuma mensagem chega ao plugin.** É a falha mais comum.

✅ Confirmação: *"Assinatura de webhooks bem-sucedida"*.

**Verificar o campo `messages`:** em **Webhooks → Whatsapp Business Account**, a linha `messages` precisa estar **Assinado**. É o campo que entrega mensagens recebidas.

---

## Passo 3 — Conferir qual app está inscrito

A Meta pode deixar um app padrão dela inscrito na WABA (ex.: *WA DevX Webhook Events 1P App*), o que desvia as mensagens.

```bash
curl -X GET 'https://graph.facebook.com/v19.0/{WABA_ID}/subscribed_apps' \
  -H 'Authorization: Bearer {TOKEN}'
```

A resposta deve conter **`UpCiga WA`** / `863103113406438`.

Se vier outro app, inscreva o correto:

```bash
curl -X POST 'https://graph.facebook.com/v19.0/{WABA_ID}/subscribed_apps' \
  -H 'Authorization: Bearer {TOKEN}'
```

✅ Deve retornar `{"success":true}`.

---

## Notas de ambiente

**A rede local pode não alcançar `graph.facebook.com`** (timeouts). Rode os `curl` a partir do servidor: **Coolify → serviço wordpress → Terminal**, container `wordpress-…`.

**WP-CLI não está instalado no container.** Para operações no WordPress, use o PHP com o bootstrap do WP:

```bash
php -r 'require_once "/var/www/html/wp-load.php"; /* ... */'
```

**Se o Coolify estiver fora do ar,** arquivos do plugin podem ser editados por **WP Admin → Ferramentas → Editor de arquivos de plugin** (plugin principal: `whatsapp-saas-core.php` — não `whatsapp-saas.php`, que dá `invalid_plugin`).

---

## Onde o roteamento acontece (referência)

- **Entrada:** `WebhookProcessor::resolve_tenant()` identifica o tenant por `phone_number_id`, com fallback por `waba_id`
- **Assinatura HMAC:** `MetaAppResolver` seleciona o `app_secret` correto antes de validar
- **Saída:** `OutboundMessageService` usa o `phone_number_id` gravado na conversa; o "Phone Number ID Principal" é só fallback para conversas legadas

---

## Checklist de entrega

- [ ] Número com status **Inscrito**
- [ ] Toggle **Assinar webhooks** ativo
- [ ] Campo `messages` assinado
- [ ] `subscribed_apps` retornando `UpCiga WA`
- [ ] PIN guardado no cofre
- [ ] **Phone Number ID** e **WABA ID** repassados para a Parte 2
