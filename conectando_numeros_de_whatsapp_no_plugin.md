# Como conectar um número de WhatsApp ao UpWaba CRM

Guia para conectar um número de atendimento à plataforma, de modo que as conversas passem a ficar registradas no **Inbox** do plugin.

---

## ⚠️ Leia antes de começar

Conectar um número à API oficial **não é adicionar um espelho** das conversas. É uma **migração**, e ela muda como a equipe trabalha:

| O que muda | Detalhe |
|---|---|
| 📱 O app sai do celular | O número é desconectado do **WhatsApp Business App**. O atendente deixa de usá-lo no aparelho. |
| 💬 O atendimento passa para o plugin | Todas as conversas passam a ser lidas e respondidas pelo **Inbox** do UpWaba CRM. |
| 🕓 O histórico antigo **não vem junto** | Conversas anteriores à conexão **não são transferidas**. O registro no plugin começa do zero, a partir do momento da conexão. |
| 🔒 A migração não é reversível na prática | Voltar o número para o app exige desconectá-lo da API e reconfigurar tudo. |

**Recomendação:** conecte **um número primeiro**, opere alguns dias e só depois migre os demais. Assim a equipe se adapta sem parar o atendimento.

---

## Antes de conectar — checklist

- [ ] O número **não pode** estar ativo em outro WhatsApp (comum ou Business) no momento da conexão
- [ ] Alguém precisa **atender uma ligação** nesse número para receber o código de verificação
- [ ] Definir um **PIN de 6 dígitos** e guardá-lo em local seguro (será pedido se o número precisar ser reconectado no futuro)
- [ ] Avisar a equipe que usa o número: a partir da migração, o atendimento é pelo plugin

---

## Passo 1 — Registrar o número na Meta

1. Acesse **developers.facebook.com** e entre com a conta vinculada ao negócio
2. Abra o app **UpCiga WA**
3. No menu lateral: **WhatsApp → Etapa 2. Configuração da produção**
4. Expanda **"Registre seu número de telefone do WhatsApp"**
5. Clique em **Adicionar novo número**

Preencha o perfil da empresa:

| Campo | O que informar |
|---|---|
| Nome de exibição | Nome comercial (é o que o cliente vê no WhatsApp) |
| Categoria | Ramo de atuação |
| Fuso horário | `America/Belem (GMT-03:00)` ou o correspondente |

Depois informe o telefone com DDI e DDD e escolha o método de verificação.

> 💡 **Prefira "Ligação telefônica".** A verificação por SMS costuma falhar com o erro `#2388002 — Falha ao verificar a qualificação do número`. Por ligação funciona normalmente.

Atenda a ligação, anote o **código de 6 dígitos** e informe na tela.

Ao final, a Meta pede um **PIN de 6 dígitos** que você define. **Anote esse PIN** — ele é exigido para reinscrever o número mais tarde.

✅ O número deve ficar com o status **"Inscrito"**.

---

## Passo 2 — Ativar os webhooks

Ainda na mesma tela, ao lado do nome da conta, ative o botão **"Assinar webhooks"**.

> ⚠️ **Este passo é o que faz as mensagens chegarem no plugin.** Sem ele, o número funciona na Meta mas **nada aparece no Inbox**. É o erro mais comum.

✅ Deve aparecer a confirmação *"Assinatura de webhooks bem-sucedida"*.

---

## Passo 3 — Confirmar que o app certo está inscrito

A Meta às vezes deixa **outro app** inscrito na conta (por exemplo, um app padrão dela). Se isso acontecer, as mensagens vão para o lugar errado.

Quem tiver acesso ao servidor deve rodar:

```bash
curl -X GET 'https://graph.facebook.com/v19.0/{WABA_ID}/subscribed_apps' \
  -H 'Authorization: Bearer {TOKEN}'
```

A resposta precisa citar **`UpCiga WA`** (ID `863103113406438`).

Se aparecer outro app, inscreva o correto:

```bash
curl -X POST 'https://graph.facebook.com/v19.0/{WABA_ID}/subscribed_apps' \
  -H 'Authorization: Bearer {TOKEN}'
```

✅ Deve retornar `{"success":true}`.

---

## Passo 4 — Cadastrar o número no plugin

1. Entre no painel: **upwaba.upciga.com/wp-admin**
2. Menu **WAS Master → Números**
3. Clique em cadastrar novo número
4. Preencha:
   - **Tenant / Cliente** — a empresa dona do número
   - **Telefone** — o número conectado
   - **Phone Number ID** — o código que a Meta gerou no Passo 1

> 💡 Confira que o número ficou vinculado à **conta WhatsApp (WABA) correta** — é dela que sai a autorização de envio.

---

## Passo 5 — Testar

1. Peça para alguém enviar uma mensagem ao número recém-conectado
2. Abra **WhatsApp SaaS → Inbox**
3. A conversa deve aparecer na lista
4. Responda pelo campo de texto e confirme que a resposta chegou no celular de quem enviou

✅ Se a mensagem apareceu e a resposta chegou, o número está operacional.

---

## Se algo não funcionar

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| Mensagem não aparece no Inbox | Webhooks não assinados | Refaça o **Passo 2** |
| Mensagem não aparece, webhook assinado | App errado inscrito na conta | Refaça o **Passo 3** |
| Erro ao enviar: "Account not registered" | Número não finalizou o registro | Confirme o status **"Inscrito"** no Passo 1 |
| Erro `#131058` ao testar envio | Uso do template `hello_world` | Esse template só funciona em número de teste. Em número real, **o cliente precisa escrever primeiro** |
| Nada aparece e nenhum erro claro | — | Veja **WhatsApp SaaS → Logs**. Se houver evento `WEBHOOK_EVENT`, a mensagem chegou ao servidor e o problema é interno |

---

## Como funciona no dia a dia

**Cada conversa fica presa ao número que a recebeu.** Se o cliente escreveu para o número A, a resposta sai pelo número A — automaticamente, sem precisar escolher. Se o mesmo cliente escrever para dois números diferentes, aparecem duas conversas separadas no Inbox.

### A janela de 24 horas

O WhatsApp permite resposta livre por **24 horas** a partir da última mensagem do cliente. O Inbox mostra esse prazo no topo da conversa.

- **Dentro da janela:** responda o que quiser, texto livre
- **Fora da janela:** só é possível enviar **templates aprovados** pela Meta

Por isso o cliente precisa iniciar a conversa — a empresa não pode abordar alguém do nada com mensagem livre.

---

## Precisa de ajuda?

Registre o que aconteceu (print da tela e horário) e verifique **WhatsApp SaaS → Logs** antes de acionar o suporte técnico — os logs costumam mostrar exatamente onde parou.
