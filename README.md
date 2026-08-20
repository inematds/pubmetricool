# pubmetricool

Publicação em redes sociais (Instagram, TikTok, YouTube…) via **MCP da Metricool**, a partir da linha de comando / de um agente.

Documenta o que foi **testado de verdade** em 2026-08-19 — um Reel + TikTok agendados ponta a ponta — e o desenho da integração com o pipeline `yt-pub-livesx`.

## Docs

| Doc | O que responde |
|---|---|
| [`docs/mcp-setup.md`](docs/mcp-setup.md) | Conectar o MCP, as 9 ferramentas disponíveis, dados da marca |
| [`docs/publicar-video.md`](docs/publicar-video.md) | O fluxo validado: master local → host público → agendamento |
| [`docs/limites.md`](docs/limites.md) | Limites de contas, de publicação, janela de analytics, MCP vs API |
| [`docs/recursos.md`](docs/recursos.md) | O que dá e o que **não** dá pra fazer (responder mensagens, etc.) |
| [`docs/integracao-yt-pub-livesx.md`](docs/integracao-yt-pub-livesx.md) | Proposta de integração com o pipeline de clips |
| [`docs/comparativo-blotato.md`](docs/comparativo-blotato.md) | Metricool vs Blotato — quando cada uma vale |
| [`docs/comparativo-zernio.md`](docs/comparativo-zernio.md) | Zernio — API + MCP, $6/mês pro caso INEMA, hospeda mídia |

## O fluxo, em 10 linhas

```bash
# 1. master local (NÃO baixar do YouTube — ver docs/publicar-video.md)
cp /mnt/hd8t/yt-pub-lives/yt-pub-lives8/lives/<live_id>/clips/<arq>.mp4 ./reel.mp4

# 2. host público (o Metricool só aceita URL direta de arquivo)
gh release upload v1 reel.mp4 --repo inematds/midia --clobber
# → https://github.com/inematds/midia/releases/download/v1/reel.mp4

# 3. agendar via MCP (no Claude Code, com o MCP conectado)
#    createScheduledPost → blogId 6745962, providers [instagram, tiktok],
#    media [a URL acima], publicationDate {dateTime, timezone}
```

O Metricool baixa o arquivo da URL e re-hospeda no CDN dele. A URL do GitHub só precisa responder no momento do agendamento.

## Estado

- ✅ **Validado:** conexão MCP, agendamento IG Reel + TikTok, GitHub Release como host, edição de post agendado
- ⚠️ **A validar:** uso do MCP por daemon não-interativo (ver [integração](docs/integracao-yt-pub-livesx.md)), fluxo de aprovação, se post multi-rede consome 1 ou N da cota mensal
- ❌ **Não existe:** responder mensagens/comentários via MCP (ver [recursos](docs/recursos.md))

## Contexto

Marca **inemafuturos** · `blogId 6745962` · plano **Free**.
Mídia hospedada em [`inematds/midia`](https://github.com/inematds/midia) (release `v1`).
