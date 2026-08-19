# Guia de Implantação — WhatsApp Business API (UpCiga)

Documento com o passo a passo executado para colocar o plugin **WAS Master / WhatsApp Business Official SaaS** em produção, integrado à API oficial do WhatsApp (Meta Cloud API), rodando em `https://upwaba.upciga.com`.

> ⚠️ Este guia **não contém segredos** (tokens, senhas, secrets, PINs). Onde eles aparecem, foram substituídos por `[REDACTED]`. Consulte o cofre de credenciais da equipe para os valores reais.

---

## 1. Infraestrutura e domínio

### 1.1 DNS (Cloudflare)
- Criado registro **A** para `upwawp.upciga.com` → `2.25.202.121` (DNS only, sem proxy).
- Posteriormente o domínio foi renomeado para `upwaba.upciga.com` (ver seção 1.3).

### 1.2 HTTPS/SSL
- Certificado gerenciado automaticamente pelo **Traefik** (reverse proxy embutido no Coolify v4.1.2), via Let's Encrypt.
- Bastou alterar o domínio do serviço no Coolify de `http://` para `https://` — o Traefik solicitou o certificado automaticamente.
- Como o Traefik termina o SSL e repassa HTTP internamente ao container, foi necessário detectar o protocolo original via header `X-Forwarded-Proto`.

### 1.3 Renomeação do domínio (`upwawp` → `upwaba`)
Alterado em três camadas:
1. **Cloudflare DNS** — novo registro A para `upwaba.upciga.com` → `2.25.202.121`.
2. **Coolify** — domínio do serviço WordPress atualizado + redeploy.
3. **`wp-config.php`** (dentro do container, via terminal do Coolify) — adicionado após `<?php`:
   ```php
   if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') { $_SERVER['HTTPS'] = 'on'; }
   define('WP_HOME','https://upwaba.upciga.com');
   define('WP_SITEURL','https://upwaba.upciga.com');
   ```
   Necessário porque o banco de dados do WordPress ainda apontava para a URL antiga (sslip.io/domínio anterior), causando redirecionamentos incorretos no login.

### 1.4 Acesso ao terminal do container (Coolify)
- Caminho: **Coolify → Projeto → Serviço wordpress-with-mariadb → Terminal**.
- Selecionar container `wordpress-bnrexxy5bptb2l0zc16soht5 (localhost)` → **Connect**.
- Usado tanto para editar `wp-config.php` quanto para rodar chamadas `curl` à Graph API do lado do servidor (a rede local de desenvolvimento não conseguia alcançar `graph.facebook.com` diretamente — timeouts).

---

## 2. Ajuste visual (logo)

- Localizado o HTML do logo em:
  `wp-content/plugins/public-wa-oficial-wordpress-main/templates/landing-page.php`
- Substituído o SVG + texto "Fornecedor CRM" por:
  ```html
  <a href="#" class="logo"><img src="https://upciga.com/brand/logo-horizontal.png" alt="UpCiga" style="height:40px;"></a>
  ```
- Aplicado em **dois lugares** para persistir em futuras reinstalações do plugin:
  1. Direto no servidor (via `perl -i -0pe`, já que o container não tem Python).
  2. No repositório fonte `https://github.com/fvmaia/public-wa-oficial-wordpress` (clonado, editado, commitado e enviado — commit `12abe22`).

---

## 3. Criação do App na Meta for Developers

1. Acessar [developers.facebook.com](https://developers.facebook.com/apps/create/), login com a conta Facebook vinculada ao negócio.
2. **Criar app** → tipo **Empresa** → nome `UpCiga WA` → email de contato `upciga.app@gmail.com`.
   - **App ID gerado:** `863103113406438`
3. No painel do app → **Adicionar produto** → **WhatsApp** → Configurar.
4. Na tela de boas-vindas do WhatsApp, selecionar o portfólio empresarial (**Upciga Sistemas**) → Continuar.
   - Isso reivindica automaticamente um **número de teste gratuito** da Meta (`+1 555 651-0349`), com WABA de teste própria.

### 3.1 Credenciais coletadas (App)
| Campo | Onde encontrar |
|---|---|
| App ID | Configurações do app → Básico |
| App Secret | Configurações do app → Básico → "Mostrar" (exige senha do Facebook) |
| WABA ID (teste) | WhatsApp → Etapa 1. Experimente |
| Phone Number ID (teste) | WhatsApp → Etapa 1. Experimente |

---

## 4. Configuração das credenciais no WordPress

- Painel: **WP Admin → WhatsApp SaaS → Configurações Meta** (slug: `admin.php?page=was-settings-meta`).
- Campos preenchidos: App ID, App Secret, WABA ID, Phone Number ID Principal, Meta Access Token, Graph API Version.
- Os campos sensíveis (App Secret, Access Token, Webhook Verify Token) ficam bloqueados por padrão — usar o botão **Desbloquear** para editá-los.
- **Webhook Verify Token**: gerado automaticamente pelo botão **Gerar Novo** dentro da própria tela (não precisa ser copiado de lugar nenhum — o plugin já expõe o valor certo para colar na Meta).
- URLs de callback exibidas automaticamente pela tela (para copiar na Meta):
  - Webhook: `https://upwaba.upciga.com/was-meta-check-99`
  - OAuth Redirect: `https://upwaba.upciga.com/wp-json/was/v1/meta/oauth/callback`
  - Deauthorize: `https://upwaba.upciga.com/wp-json/was/v1/meta/deauthorize`
  - Data Deletion: `https://upwaba.upciga.com/wp-json/was/v1/meta/data-deletion`

---

## 5. Geração de token de acesso temporário (fase de testes)

- Usado o **Graph API Explorer** ([developers.facebook.com/tools/explorer](https://developers.facebook.com/tools/explorer/)) com o app `UpCiga WA` selecionado.
- Permissões: `whatsapp_business_management`, `whatsapp_business_messaging`.
- Token de usuário gerado ali dura **1–2 horas** — usado apenas para testes iniciais, depois substituído por token permanente (seção 9).

---

## 6. Configuração do Webhook na Meta

1. **Meta for Developers → App → Webhooks** (menu lateral, fora do produto WhatsApp).
2. Selecionar o produto **Whatsapp Business Account**.
3. Preencher:
   - **URL de callback:** `https://upwaba.upciga.com/was-meta-check-99`
   - **Verificar token:** o valor gerado na tela de Configurações Meta do WordPress (seção 4).
4. Clicar **Verificar e salvar** — a Meta faz um GET de validação na URL; sucesso confirmado nos **Logs** do plugin (`WEBHOOK_VERIFICATION`, response_code 200).
5. Na tabela de **Campos do webhook**, localizar a linha **`messages`** e ativar o toggle (**Assinado**) — é o campo que entrega mensagens recebidas.

---

## 7. Teste ponta a ponta (número de teste)

### 7.1 Cadastro de destinatário de teste
- Em **WhatsApp → Etapa 1. Experimente**, adicionar o número pessoal como destinatário (até 5 permitidos).
- Fluxo: **Gerenciar lista de números** → código de país `BR +55` → número → verificação por **SMS ou ligação telefônica** (código de 6 dígitos).

### 7.2 Envio de mensagem (outbound)
- Enviado template padrão `hello_world` via `curl` diretamente para a Graph API (a UI da Meta apresentou bugs de renderização/popup bloqueado):
  ```bash
  curl -X POST 'https://graph.facebook.com/v19.0/{PHONE_NUMBER_ID}/messages' \
    -H 'Authorization: Bearer [REDACTED]' \
    -H 'Content-Type: application/json' \
    -d '{
      "messaging_product": "whatsapp",
      "to": "{NUMERO_DESTINATARIO}",
      "type": "template",
      "template": { "name": "hello_world", "language": { "code": "en_US" } }
    }'
  ```
- **Pré-requisito descoberto:** número de teste precisa ser **registrado** na Cloud API antes do primeiro envio:
  ```bash
  curl -X POST 'https://graph.facebook.com/v19.0/{PHONE_NUMBER_ID}/register' \
    -H 'Authorization: Bearer [REDACTED]' \
    -H 'Content-Type: application/json' \
    -d '{"messaging_product": "whatsapp", "pin": "[REDACTED]"}'
  ```

### 7.3 Recebimento de mensagem (inbound) — troubleshooting
- Primeiro teste: mensagem enviada pelo WhatsApp do usuário não apareceu no **Inbox** do plugin (`WhatsApp SaaS → Inbox`).
- **Causa raiz:** o app `UpCiga WA` não estava **inscrito (subscribed)** na WABA — por padrão, um app diferente da própria Meta ("WA DevX Webhook Events 1P App") vinha inscrito no número de teste.
- **Correção** (via terminal do Coolify, chamando a Graph API do lado do servidor):
  ```bash
  # Verificar app inscrito atualmente
  curl -X GET 'https://graph.facebook.com/v19.0/{WABA_ID}/subscribed_apps' \
    -H 'Authorization: Bearer [REDACTED]'

  # Inscrever o app correto
  curl -X POST 'https://graph.facebook.com/v19.0/{WABA_ID}/subscribed_apps' \
    -H 'Authorization: Bearer [REDACTED]'
  ```
- Após a correção, mensagens passaram a aparecer corretamente no Inbox do WordPress, com janela de atendimento de 24h exibida.

---

## 8. Configuração de produção — número de telefone real

> A WhatsApp Cloud API não permite reaproveitar um número que já tem WhatsApp Business App ativo no celular sem perder histórico e desconectar o app do celular. Por isso foi usado um **número novo e dedicado**: `+55 83 98843-6603`.

### 8.1 Etapa 2 (Meta) — Configuração da produção
Local: **App → WhatsApp → Etapa 2. Configuração da produção**.

1. **Configurar webhooks** — já estava concluído (herdado da configuração de teste).
2. **Registrar número de telefone:**
   - **Adicionar novo número** → criar perfil do WhatsApp Business:
     - Nome de exibição: `UpCiga Sistemas`
     - Categoria: `Serviços profissionais`
     - Fuso horário: `America/Belem (GMT-03:00)`
   - Telefone: `+55 83 98843-6603`, verificação por **ligação telefônica** (SMS falhou com erro `#2388002 — Falha ao verificar a qualificação do número`, resolvido usando ligação em vez de SMS).
   - Código de verificação de 6 dígitos recebido por voz.
   - **Nova WABA gerada:** `1625887882297398`
   - **Novo Phone Number ID:** `1320960361099090`
   - Registro na Cloud API com **PIN de 6 dígitos** definido pelo usuário (necessário para reinscrever o número no futuro — guardar em local seguro).
   - Ativado o toggle **Assinar webhooks** para essa WABA (inscreve o app `UpCiga WA` automaticamente — confirmado depois via `subscribed_apps`).
3. **Adicionar informações de pagamento:**
   - **Billing Hub** dentro da própria tela → **Adicionar forma de pagamento**.
   - Localização: Brasil · Moeda: **Dólar americano (USD)** — `Real brasileiro` não estava disponível na lista de moedas suportadas pelo Billing Hub; a cobrança em USD é convertida para BRL automaticamente pela operadora do cartão.
   - Dados do cartão preenchidos **manualmente pelo dono da conta** (nome, número, validade, CVV) — nunca inseridos por automação, por política de segurança.
4. **Enviar mensagem** — validado manualmente fora do fluxo guiado da Meta (ver seção 8.3).

### 8.2 Atualização das credenciais de produção no WordPress
- Voltar em **Configurações Meta**, desbloquear e atualizar:
  - `WABA ID`: `1625887882297398`
  - `Phone Number ID Principal`: `1320960361099090`
  - `Meta Access Token`: token permanente (seção 9)
- Salvar configurações.

### 8.3 Teste do número de produção
- Tentativa de enviar `hello_world` pelo número de produção **falhou de propósito** — esse template só funciona nos números de teste públicos da Meta (erro `#131058`).
- Fluxo correto para número real: **o cliente inicia a conversa primeiro**, abrindo a janela de atendimento de 24h; a partir daí qualquer mensagem de sessão (texto livre, sem template) pode ser enviada em resposta.
- Teste realizado: usuário enviou "Oi" para `+55 83 98843-6603` → mensagem recebida no Inbox do WordPress (confirmado via **Logs** do plugin, evento `WEBHOOK_EVENT` com `phone_number_id` de produção) → resposta enviada pelo campo de texto do Inbox → confirmado recebimento no WhatsApp do cliente.
- Observação: como o número de teste e o de produção têm o mesmo contato de destino (número pessoal do usuário), as conversas de ambos os números ficaram agrupadas na mesma thread do Inbox (o plugin agrupa por contato do cliente, não por número de negócio).

---

## 9. Token de acesso permanente (System User)

O token gerado pelo Graph API Explorer expira em 1–2h — inadequado para produção. Substituído por um token de **System User**, que não expira.

1. **Meta Business Suite → Configurações → Usuários do sistema** ([business.facebook.com/settings/system-users](https://business.facebook.com/settings/system-users)).
2. **Adicionar** → aceitar Política de Não Discriminação.
3. Criar usuário do sistema:
   - Nome: `Upciga integracao` (nomes com maiúscula excessiva ou espaços foram rejeitados pela validação da Meta)
   - Cargo: **Admin**
4. **Atribuir ativos** ao usuário do sistema:
   - **Apps** → selecionar `UpCiga WA` → permissão **Gerenciar app** (acesso total).
   - **Contas do WhatsApp** → selecionar **ambas** as WABAs (teste e produção) → permissão **Tudo** (acesso total).
5. **Gerar token**:
   - App: `UpCiga WA`
   - Expiração: **Nunca**
   - Permissões: `whatsapp_business_messaging`, `whatsapp_business_management`, `whatsapp_business_manage_events`
   - Token copiado imediatamente (a Meta só exibe uma vez).
6. Token atualizado nas Configurações Meta do WordPress (substituindo o temporário) — ver seção 8.2.

---

## 10. Checklist final de produção

| Item | Status |
|---|---|
| App Meta criado e configurado | ✅ |
| Webhook configurado e verificado | ✅ |
| App inscrito na WABA de teste (`subscribed_apps`) | ✅ |
| App inscrito na WABA de produção (`subscribed_apps`) | ✅ |
| Número de produção registrado (`+55 83 98843-6603`) | ✅ |
| Perfil WhatsApp Business criado (nome, categoria) | ✅ |
| Forma de pagamento cadastrada (Billing Hub) | ✅ |
| Token de acesso permanente (System User) | ✅ |
| Credenciais de produção salvas no WordPress | ✅ |
| Teste de envio (outbound) — número de teste | ✅ |
| Teste de recebimento (inbound) — número de teste | ✅ |
| Teste de recebimento (inbound) — número de produção | ✅ |
| Teste de envio (outbound) — número de produção | ✅ |

### Pendências / próximos passos sugeridos
- **Etapa 3 — Verificação da empresa** na Meta (aumenta limites de mensagens/dia, remove restrições de escala).
- Validar o fluxo de **Embedded Signup** (auto-cadastro de números de clientes) logado como usuário-tenant comum, sem privilégio de administrador do WordPress, para confirmar se está liberado para uso self-service pelos clientes finais.
- Repetir a atribuição de ativos (`subscribed_apps` + System User) para cada novo tenant/WABA que for onboarded.

---

## Referências de arquivos tocados no repositório do plugin

- `templates/landing-page.php` — logo do header.
- `includes/Admin/Menu.php` — registro de menus admin (`was-settings-meta`, `was-master-*`).
- `includes/Tenants/TenantRepository.php`, `TenantUserRepository.php` — modelo multi-tenant.
- `includes/Auth/TenantContext.php` — resolução do tenant ativo.
- `includes/WhatsApp/WebhookProcessor.php` — roteamento de webhook por `phone_number_id`/`waba_id`.
- `includes/Router/MetaAppResolver.php` — validação de assinatura HMAC por app.
- `includes/REST/EmbeddedSignupController.php`, `includes/Router/OnboardingService.php` — fluxo de Embedded Signup.
