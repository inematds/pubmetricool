# O que dá e o que não dá

## Pelo MCP (as 9 ferramentas)

### ✅ Dá

**Publicação**
- Agendar post em qualquer rede conectada, single ou multi-rede
- Editar e cancelar post agendado
- Rascunho, publicação automática, ou push pro app pra finalizar na mão
- Threads (`descendants`), primeiro comentário (`firstCommentText`), alt text
- Enquete no LinkedIn, board no Pinterest, colaborador no Instagram
- Áudio do catálogo do Instagram em Reels (só conta Business + Página do Facebook)
- Melhor horário pra postar, por rede

**Análise**
- Métricas de Instagram, TikTok, YouTube, LinkedIn, Facebook, X, Threads, Bluesky, Pinterest, Twitch, Google Business Profile
- Posts, reels, stories, evolução de seguidores, alcance
- Demografia: idade, gênero, país
- **Concorrentes** (IG, FB, Twitch, YouTube, X, Bluesky), inclusive posts deles
- **Hashtags**
- **Anúncios**: Meta Ads, Google Ads, TikTok Ads

### ❌ Não dá

**Responder mensagens e comentários — não existe no MCP.** Nenhuma das 9 ferramentas toca Inbox, DM ou comentário. Não é limitação de plano: a superfície do MCP simplesmente não expõe isso.

O **Inbox da Metricool existe e está no plano Free** — Instagram, Facebook, TikTok e Google Business Profile em todos os planos (X exige add-on). Mas **só pelo app web/celular**, na mão.

Também fora do MCP:
- Upload de arquivo local (só URL pública — ver [publicar-video.md](publicar-video.md))
- Listar posts **já publicados** (`getScheduledPosts` só traz os pendentes; o histórico vem pelos conectores de analytics)
- Link-in-bio / Smartlinks, relatórios PDF/PPT, gestão de time e papéis
- Criar ou conectar marca nova

## Se você precisa responder mensagens automaticamente

O MCP não serve. As opções reais:

1. **Inbox da Metricool na mão** — grátis, já disponível, cobre IG/FB/TikTok/GBP
2. **API REST da Metricool** — exige Advanced (€43+/mês); confirmar nos [apidocs](https://app.metricool.com/resources/apidocs/index.html) se há endpoint de inbox, não verificado aqui
3. **API nativa da rede** — Instagram Graph API / TikTok API direto, sem Metricool no meio
4. **Zapier / Make** — conectores prontos, mas o conector Metricool usa a API (Advanced)

Pra responder DM do Instagram por automação, o caminho mais direto é a **Graph API do Instagram**, não a Metricool.

## Redes suportadas pelo MCP

Instagram (posts, reels, stories) · TikTok · LinkedIn · Facebook (posts, reels, stories) · X/Twitter · YouTube · Pinterest · Threads · Bluesky · Twitch · Google Business Profile · Meta Ads · Google Ads · TikTok Ads

No **Free**, LinkedIn e X não conectam.

## Fluxo de aprovação

`createScheduledPostForReview` e `sendScheduledPostForReview` existem no MCP, mas o sistema de aprovação é listado como recurso **Advanced**. **Não testado** — provavelmente falha no Free.
