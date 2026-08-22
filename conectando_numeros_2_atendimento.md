# Parte 2 — Guia da equipe de atendimento

**Público:** equipe que vai atender pelo UpWaba CRM.
**Pré-requisito:** o número já foi provisionado na Meta ([Parte 1](conectando_numeros_1_tecnico.md)) e você recebeu o **Phone Number ID**.

---

## O que muda quando um número é conectado

Não é um espelho das conversas — é uma **mudança de ferramenta**.

| Antes | Depois |
|---|---|
| Atendimento pelo **WhatsApp Business App** no celular | Atendimento pelo **Inbox** do UpWaba CRM, no navegador |
| Cada atendente com o número no aparelho | Time inteiro atende pela mesma caixa de entrada |
| Histórico dentro do celular | Histórico registrado na plataforma |

### Três pontos importantes

**1. O número sai do celular.** Depois de conectado, o WhatsApp Business App **não funciona mais** com esse número. Não dá para usar os dois ao mesmo tempo.

**2. O histórico antigo não vem junto.** As conversas que já existem no celular **não são transferidas**. O registro na plataforma começa **do zero**, a partir do momento da conexão.

> Se houver conversas que precisam ser preservadas, **exporte ou salve antes** de conectar o número.

**3. Migre um número por vez.** Conecte um, opere alguns dias, e só então os demais. Assim o time se adapta sem parar o atendimento.

---

## Passo 4 — Cadastrar o número no plugin

1. Acesse **upwaba.upciga.com/wp-admin**
2. Menu **WAS Master → Números**
3. Cadastre um novo número:

| Campo | O que informar |
|---|---|
| **Tenant / Cliente** | A empresa dona do número |
| **Telefone** | O número que foi conectado |
| **Phone Number ID** | O código recebido da Parte 1 |

> 💡 Confirme que o número ficou vinculado à **conta WhatsApp (WABA) correta** — é dela que sai a autorização para enviar mensagens.

---

## Passo 5 — Testar

1. Peça para alguém enviar uma mensagem ao número recém-conectado
2. Abra **WhatsApp SaaS → Inbox**
3. A conversa deve aparecer na lista à esquerda
4. Responda pelo campo de texto
5. Confirme que a resposta chegou no celular de quem enviou

✅ Mensagem apareceu **e** resposta chegou = número operacional.

---

## Como funciona no dia a dia

### Cada conversa responde pelo número certo, sozinha

Se o cliente escreveu para o número A, a resposta sai pelo número A. Você não precisa escolher — o sistema faz isso automaticamente.

Se o mesmo cliente escrever para dois números diferentes da empresa, aparecem **duas conversas separadas** no Inbox. Isso é esperado.

### A janela de 24 horas

O WhatsApp permite resposta livre por **24 horas** contadas a partir da **última mensagem enviada pelo cliente**. O Inbox mostra esse prazo no topo da conversa.

| Situação | O que dá para fazer |
|---|---|
| 🟢 **Dentro da janela** | Responder normalmente, texto livre |
| 🔴 **Fora da janela** | Só enviar **templates aprovados** pela Meta |

**Consequência prática:** a empresa **não pode iniciar** uma conversa com texto livre. O cliente precisa escrever primeiro. Para abordagem ativa, é necessário usar template aprovado.

Cada mensagem nova do cliente **reinicia** as 24 horas.

---

## Se algo não funcionar

| O que você vê | O que provavelmente é | O que fazer |
|---|---|---|
| Mensagem não aparece no Inbox | Webhooks não foram ativados | Acionar o técnico — refazer **Passo 2 da Parte 1** |
| Conversa aparece, mas resposta não chega | Token vencido ou número mal vinculado | Acionar o técnico com o horário do ocorrido |
| "Janela de atendimento encerrada" | Passaram-se mais de 24h da última mensagem do cliente | Usar template aprovado, ou aguardar o cliente escrever |
| Conversa duplicada do mesmo cliente | Ele escreveu para dois números da empresa | Comportamento normal |

### Antes de acionar o suporte

1. Anote **horário** e **número** envolvido
2. Tire **print** da tela
3. Verifique **WhatsApp SaaS → Logs** — se houver um evento `WEBHOOK_EVENT` no horário, a mensagem chegou ao servidor e o problema é interno; se não houver, o problema é na entrega da Meta

Essa informação acelera muito o diagnóstico.

---

## Resumo rápido

- ✅ Atendimento agora é pelo **Inbox**, no navegador
- ✅ Resposta sai pelo número certo automaticamente
- ⏱️ **24 horas** para responder livremente após cada mensagem do cliente
- 🚫 O número **não funciona mais** no app do celular
- 🚫 O histórico anterior **não foi transferido**
