# Contexto — App Meta de teste para DeskcommCRM

> Documento de handoff. Cole ou referencie este arquivo no início de um novo chat.

## Objetivo

Criar um app Meta separado, chamado **"UpDeskComm teste"**, para avaliar o **DeskcommCRM** recebendo webhooks da WhatsApp Cloud API — **sem tocar** no ambiente de produção nem na verificação de Tech Provider em análise.

---

## Situação atual (produção — NÃO MEXER)

| Item | Valor |
|---|---|
| App Meta | **UpCiga WA** — ID `863103113406438` |
| Portfólio empresarial | Upciga Sistemas — ID `1443690694484256` |
| WABA de produção | `1625887882297398` |
| Número de produção | `+55 83 98843-6603` (Phone Number ID `1320960361099090`) |
| WABA de teste (da Meta) | `1604575307858688` — número `+1 555 651-0349` |
| Webhook atual | `https://upwaba.upciga.com/was-meta-check-99` |
| Plugin em produção | UpWaba CRM (WordPress, `upwaba.upciga.com`) |
| Verificação da empresa | ✅ Aprovada em 18/08/2026 |
| **Tech Provider (Access Verification)** | 🔵 **Em análise** — prazo 20/10/2026 |

Credenciais (App Secret, tokens, PIN, senha do WP) estão no cofre da equipe — **não estão neste documento**.

---

## Comparação: UpWaba CRM × DeskcommCRM

Repositórios:
- Nosso plugin: `https://github.com/fvmaia/public-wa-oficial-wordpress`
- DeskcommCRM: `https://github.com/fvmaia/DeskcommCRM`

São projetos de **natureza diferente**, não concorrentes diretos.

| | **UpWaba CRM** | **DeskcommCRM** |
|---|---|---|
| Tipo | Plugin WordPress (PHP) | App standalone (Next.js 16 + TypeScript) |
| Banco | MySQL/MariaDB do WP | Supabase (Postgres + Auth + Storage) |
| Deploy | Dentro de um WordPress | Docker próprio (VPS, worker, scheduler) |
| Licença | Privado | MIT, open source |
| Canal WhatsApp | Só **Cloud API oficial** | **Cloud API oficial** *e* **WAHA** (não-oficial) |
| Multi-tenant | ✅ Painel Master com tenants | ✅ Organizações |

**Só no DeskcommCRM:** Agentes de IA (proposta central), Kanban, Pipelines, Contatos/Leads, Métricas, Radar, Integrações (Nuvemshop etc.), telas de LGPD e Auditoria, gestão de equipe.

**Só no nosso plugin:** Painel Master de operação SaaS (tenants, onboardings, tokens, checklists, App Review/Compliance), Embedded Signup da Meta, integração com o ecossistema WordPress.

**Em comum:** Inbox, Templates, Webhooks, Configurações.

**Resumo:** DeskcommCRM é um *CRM de vendas com IA*. UpWaba CRM é uma *plataforma SaaS de atendimento multi-empresa*.

---

## Como o DeskcommCRM recebe webhooks da Meta

Rota: `app/api/v1/webhooks/meta/[token]/route.ts`

```
https://{dominio}/api/v1/webhooks/meta/{token}
```

- `{token}` identifica a organização (mapeia para uma "meta session")
- `GET` = handshake: responde `hub.challenge` em **texto puro** (sem wrapper JSON)
- `POST` = valida **HMAC SHA-256** com o App Secret
- Variáveis de ambiente: `META_APP_SECRET`, `META_WEBHOOK_VERIFY_TOKEN`

> Observação do próprio código: existe token no path porque o App Secret é do *app*, e um app serve N WABAs de N organizações. O token amarra o payload a UMA org antes de qualquer escrita.

O repo também tem rota WAHA (`webhooks/waha/[token]`) — canal **não-oficial**, não usar neste teste.

---

## Por que app separado (e não trocar o webhook do app atual)

A Meta permite **uma única URL de callback por app**. Trocar seria exclusivo: enquanto apontasse para o DeskcommCRM, a produção pararia e o histórico do plugin ficaria com buraco (a Meta não reenvia depois).

Além disso, quem determina o destino dos webhooks é a **WABA** (via `subscribed_apps`), não o app. Então o isolamento real exige **recursos separados**:

| | Produção (intocada) | Teste DeskcommCRM |
|---|---|---|
| App | UpCiga WA | UpDeskComm teste |
| WABA | `1625887882297398` | WABA nova do app |
| Número | `+55 83 98843-6603` | número de teste da Meta (gratuito) |
| Webhook | plugin WordPress | DeskcommCRM |

Cada app novo ganha da Meta um **número de teste gratuito** com WABA própria.

---

## A verificação de Tech Provider NÃO fica comprometida

A Verificação de Acesso é feita **no nível do portfólio empresarial**, não do app:

- URL usada: `developers.facebook.com/**1443690694484256**/access-verification/` — é o **business ID**
- Cabeçalho mostrava "Upciga Sistemas" com ícone de empresa
- Texto: *"Preencha esse formulário somente para a sua empresa, Upciga Sistemas"*
- A menção a *"evitar restrições a 1 app"* é sobre o que fica protegido **se concluir** — não é o objeto da análise

**Criar outro app não reinicia, não invalida e não altera a análise em curso.**

**Efeito colateral menor:** se o app novo ficar sob o mesmo portfólio, o contador pode virar "2 apps" e o novo passaria a depender da mesma verificação. Para isolamento total, **criar o app sem vincular portfólio empresarial** (o campo é opcional).

---

## Passo a passo a executar

**1. Criar o app**
`developers.facebook.com/apps/create` → tipo **Empresa** → nome **`UpDeskComm teste`** → **deixar o portfólio empresarial vazio**

> O nome é editável depois em Configurações do app → Básico → *Nome de exibição*. O **App ID nunca muda**.

**2. Adicionar o produto WhatsApp**
Painel do app → **Adicionar produto → WhatsApp → Configurar**
Gera automaticamente WABA de teste, número de teste e Phone Number ID. **Anotar os três.**

**3. Coletar credenciais**
- **App ID** e **App Secret** → Configurações do app → Básico (revelar exige senha do Facebook — o usuário faz)
- **Token** → Graph API Explorer, permissões `whatsapp_business_messaging` e `whatsapp_business_management`

**4. Configurar o DeskcommCRM**
```
META_APP_SECRET=<App Secret do app novo>
META_WEBHOOK_VERIFY_TOKEN=<escolher um valor>
```
Criar a sessão Meta da organização para obter o `{token}` da URL.

**5. Apontar o webhook do app novo**
App novo → **Webhooks → Whatsapp Business Account**:

| Campo | Valor |
|---|---|
| URL de callback | `https://{dominio}/api/v1/webhooks/meta/{token}` |
| Verificar token | o mesmo do `META_WEBHOOK_VERIFY_TOKEN` |

Ativar o campo **`messages`**.

**6. Cadastrar destinatário de teste**
App novo → WhatsApp → **Etapa 1. Experimente** → adicionar número.

**7. Testar** enviando mensagem ao número de teste do app novo.

---

## Armadilhas conhecidas (vividas na implantação de produção)

| Armadilha | Como lidar |
|---|---|
| Verificação por **SMS falha** com `#2388002` | Usar **ligação telefônica** |
| Webhook assinado mas nada chega | Conferir `subscribed_apps` da WABA — a Meta às vezes deixa um app padrão dela inscrito |
| Template `hello_world` dá erro `#131058` | Só funciona em número de **teste**; em número real o cliente precisa escrever primeiro |
| Recarregar após salvar mostra valor antigo | É render defasado da UI da Meta, **não** falha de gravação. Conferir em outra sessão/navegador |
| Rede local não alcança `graph.facebook.com` | Rodar `curl` a partir do servidor |
| Popups bloqueados quebram botões da Meta | Alguns botões (ex.: gerar token) abrem popup; usar o Graph API Explorer como alternativa |

---

## Repositório com a documentação completa

`https://github.com/fvmaia/public-wa-oficial-wordpress`

- `guia_de_implantacao.md` — implantação de produção completa, com seções 11 (Embedded Signup bloqueado) e 12 (Tech Provider)
- `conectando_numeros_1_tecnico.md` — provisionamento de número (técnico)
- `conectando_numeros_2_atendimento.md` — guia da equipe de atendimento
