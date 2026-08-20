# Zernio — análise e comparação

Pesquisa de 2026-08-20, a partir de `zernio.com/pricing`, `docs.zernio.com` e do `llms.txt` da documentação.
**Não testada.** O que está aqui é o que a Zernio declara.

## Tem API? Tem MCP?

**As duas coisas, e é o produto inteiro.** A Zernio se apresenta como *"Social Media & Messaging API for Developers & AI Agents"* — não é uma suíte com API pendurada, é uma API.

### API REST

```
https://zernio.com/api/v1
```

- Auth por **API key** (`Bearer`), criada em Settings → API Keys — e gerenciável **pela própria API**
- SDKs oficiais (`zernio-sdk`), CLI, OpenAPI, `llms.txt`
- Endpoints por domínio: Posting, Comments, Messaging, Analytics, Ads
- **Webhooks** de eventos reais (status de post, mensagem, comentário) — *"a gente re-tenta e dispara um webhook com o motivo. Sem falha silenciosa, sem polling"*
- Chaves com escopo e roteamento de webhook por perfil (modelo multi-cliente)
- Bulk por CSV, com `dryRun=true` pra validar sem criar

### MCP hospedado

```
https://mcp.zernio.com/mcp
```

- **Claude Code tem plugin próprio:**
  ```
  /plugin marketplace add zernio-dev/zernio-claude-plugin
  /plugin install zernio@zernio
  ```
  Pede a API key uma vez e guarda no keychain do sistema.
- Claude web/desktop, ChatGPT, Codex, Cursor, Windsurf: conector OAuth, sem API key
- Usa `search_tools` (ferramentas sob demanda) em vez de despejar tudo no contexto

## Preço: por conta conectada, sem plano

Não há plano nem assento. Você paga **por conta social, por mês**, em faixas:

| Faixa | Preço |
|---|---|
| Contas 1–2 | **grátis**, sem cartão |
| Contas 3–10 | $6/mês cada |
| Contas 11–100 | $3/mês cada |
| Contas 101+ | $1/mês cada, sem teto |

**Todo recurso vem em toda conta** — agendamento e publicação ilimitados, analytics, DMs, comentários e avaliações, anúncios, webhooks. Nada atrás de upgrade.

Extras cobrados à parte: X/Twitter repassado ao custo exato da X (sem markup), números de telefone (a partir de $3/mês), chamadas, SMS e WhatsApp.

### No seu caso

Instagram + TikTok + YouTube = 3 contas → 2 grátis + 1 × $6 = **$6/mês**.

| | Custo pro caso INEMA | API | DMs |
|---|---|---|---|
| Metricool Free | €0 | ❌ | ❌ |
| Metricool Advanced | €43/mês | ✅ | ❌ |
| Blotato Starter | $29/mês | ✅ | ✅ |
| **Zernio** | **$6/mês** | ✅ | ✅ |

## O que a Zernio resolve e as outras não

### 1. Hospedagem de mídia — elimina o `inematds/midia`

A Zernio tem endpoint de upload próprio:

- `POST /v1/media` com **presigned URL até 5 GB** — devolve `uploadUrl` e `publicUrl`, você faz `PUT` do arquivo direto
- Upload direto com Bearer token, até 25 MB, pra anexo de mensagem

Ou seja: **manda o arquivo local, recebe URL pública.** Some a etapa "subir pro GitHub Release" descrita em [publicar-video.md](publicar-video.md). Metricool e Blotato exigem URL pública pronta; a Zernio hospeda.

### 2. Webhooks em vez de polling

Post publicado, mensagem recebida, comentário novo → evento HTTP. Nas outras, você teria que ficar consultando. Pro `scheduler.py`, isso é a diferença entre um loop de polling e um handler.

### 3. Escopo bem além de rede social

16 plataformas, incluindo Discord, WhatsApp, SMS, chamadas telefônicas, Google Business Profile, e anúncios em Meta, TikTok, LinkedIn, Pinterest, Google — **e OpenAI Ads (anúncios no ChatGPT)**.

## Comparação das três

| | Metricool | Blotato | Zernio |
|---|---|---|---|
| Natureza | suíte de analytics | API pra agentes | **API pra devs e agentes** |
| Grátis | ✅ permanente (1 conta) | ❌ trial 7 dias | ✅ **2 contas permanentes** |
| Custo p/ 3 contas | €0 (só 1 conta) | $29/mês | **$6/mês** |
| API REST | Advanced (€43) | Starter ($29) | **incluída sempre** |
| API no grátis | ❌ | ❌ | ✅ |
| MCP | ✅ | ✅ | ✅ + plugin do Claude Code |
| Plugin p/ Claude Code | ❌ | ❌ | ✅ |
| Modelo de conta | 1 por rede × marcas | 20 no total | **por conta, sem teto** |
| DMs / comentários | ❌ | ✅ | ✅ |
| Webhooks | ❌ | ? | ✅ |
| **Hospeda mídia** | ❌ exige URL | ❌ exige URL | ✅ **presigned 5 GB** |
| Analytics profundo | ✅ **melhor** | básico | intermediário |
| Concorrentes / hashtags | ✅ | ❌ | ❌ |
| Relatório PDF/PPT/Looker | ✅ | ❌ | ❌ |
| Geração de IA inclusa | ❌ | ✅ créditos | ❌ |
| Maturidade | ✅ consolidada | nova | **nova, sem histórico verificado** |

## Leitura

Pro problema concreto — automatizar publicação a partir do `yt-pub-livesx` — a **Zernio é a mais adequada e a mais barata, com folga**:

- API key estática, que é o que um daemon precisa (resolve o bloqueio do [doc de integração](integracao-yt-pub-livesx.md))
- $6/mês contra €43 da Metricool Advanced
- Hospeda a mídia, encurtando o pipeline
- Webhooks em vez de polling
- SDK, CLI e OpenAPI — dá pra gerar cliente

**A Metricool continua fazendo o que a Zernio não faz:** concorrentes, hashtags, relatório de cliente, analytics histórico profundo. E é grátis. Não há motivo pra desconectar.

Arranjo sugerido: **Zernio publica, Metricool Free mede.** Custo total ~$6/mês.

## Ressalvas honestas

- **Nada foi testado.** Confiabilidade de publicação, qualidade do analytics e suporte são desconhecidos.
- **Produto novo, sem histórico verificado.** Metricool tem anos de mercado e clientes grandes; a Zernio, não que eu tenha confirmado. Depender dela pra publicar é assumir risco de fornecedor.
- Rotatividade de API de rede social é alta (Instagram e TikTok mudam regra o tempo todo). Fornecedor pequeno pode demorar mais pra acompanhar.
- O modelo por uso é ótimo em escala pequena, mas **cobrança por uso surpreende** — X, SMS e chamadas são metered. Ler o doc de Billing antes de ligar qualquer coisa de telefonia.
- "Todo recurso incluído" é afirmação de marketing; verificar na prática o que o analytics realmente entrega.

## Teste sugerido

O tier grátis permite validar **sem pagar nada**:

1. Criar conta, conectar **2 contas** (Instagram + TikTok) — grátis, sem cartão
2. Instalar o plugin do Claude Code
3. Publicar o mesmo clip já validado no Metricool (`Ed9HFMSe0o4`), usando o upload de mídia da Zernio em vez do GitHub Release
4. Comparar: chegou certo? Reel virou Reel? Quanto tempo? Erro é legível?
5. Testar um webhook de status
6. Se passar, conectar a 3ª conta ($6/mês) e portar o `scheduler.py`

Diferente da Blotato (cuja API fica de fora do trial), aqui **dá pra testar a API de graça**. Comece por aqui.
