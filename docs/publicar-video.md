# Publicar vídeo — o fluxo validado

Testado ponta a ponta em 2026-08-19: Reel + TikTok agendados a partir de um master local.

## Regra que governa tudo: `media` só aceita URL pública de arquivo

O Metricool **baixa** o arquivo da URL que você passar e re-hospeda no CDN dele (`static.metricool.com/planner/...`). Consequências:

- ❌ Link de página do YouTube (`youtube.com/watch`, `/shorts/`) — é HTML, não vídeo
- ❌ URL de stream `googlevideo.com` — **expira em horas e é amarrada ao IP/sessão**. Pode funcionar no teste e quebrar antes da hora do agendamento
- ✅ URL que responde os bytes do arquivo
- ✅ Google Drive / Dropbox (o Metricool resolve; Drive exige a conta vinculada no Metricool)

A URL só precisa estar viva **no momento do agendamento** — depois disso o Metricool já tem a cópia dele.

## Host adotado: GitHub Release

Repo [`inematds/midia`](https://github.com/inematds/midia), público, release `v1` como balde de assets.

```bash
gh release upload v1 arquivo.mp4 --repo inematds/midia --clobber
```

```
https://github.com/inematds/midia/releases/download/v1/arquivo.mp4
```

- Até **2 GB** por arquivo
- **Não entra no histórico do git** — quem clonar não baixa os vídeos
- `--clobber` sobrescreve arquivo de mesmo nome
- O GitHub serve como `application/octet-stream`; **testado, o Metricool não se importa**
- **Nome sem espaço nem acento** — vira parte da URL
- O repo precisa ser **público**: asset de repo privado exige token na URL

### Por que não as alternativas

| Opção | Problema |
|---|---|
| `raw.githubusercontent.com` | Teto de 100 MB, incha o histórico, serve como octet-stream |
| Arquivo commitado + GitHub Pages | Binário fica no histórico pra sempre; cada versão é uma cópia inteira |
| Hugging Face | Funciona igual de bem (`hf upload`, token já disponível). Escolha foi ficar no git |
| Backblaze B2 | Há um remote `b2` em `~/.config/rclone/rclone.conf`, mas o binário `rclone` não está instalado |
| Cloudflare R2 | Só existe `CLOUDFLARE_API_TOKEN`; falta account ID + chave S3 |

## Codec: H.264 obrigatório

Instagram e TikTok exigem **H.264**. E aqui mora uma armadilha:

**`yt-dlp` devolve AV1** nos formatos modernos do YouTube. Se você baixar e mandar direto, quebra.

```bash
ffmpeg -y -i entrada.mp4 -c:v libx264 -profile:v high -pix_fmt yuv420p \
  -crf 20 -preset medium -r 30 -c:a aac -b:a 192k -ar 48000 \
  -movflags +faststart saida.mp4
```

> **Melhor ainda: não baixe do YouTube.** Se o vídeo saiu do pipeline `yt-pub-lives`, o master local existe, já é H.264, e não passou por duas gerações de recompressão. Ver [integracao-yt-pub-livesx.md](integracao-yt-pub-livesx.md).

Formato de Reel/Short/TikTok: **1080×1920**, `yuv420p`, áudio AAC.

## Publicar direto ou tem que agendar?

**Não existe "publicar agora" no MCP.** `createScheduledPost` exige `date`. Mas dá pra chegar perto:

| Modo | Como | Efeito |
|---|---|---|
| Quase imediato | `date` = agora + 2–5 min, `autoPublish: true` | Publica sozinho, na prática "agora" |
| Agendado | `date` futura, `autoPublish: true` | Publica sozinho na hora marcada |
| Manual assistido | `autoPublish: false` | Manda **push pro app do celular**; você finaliza na mão |
| Rascunho | `draft: true` | Não publica; fica no planner |

`autoPublish: false` **não** pula validação nem aprovação — só troca o método de entrega.

Enquanto está `PENDING`, dá pra editar ou cancelar.

## Anatomia do payload

```jsonc
{
  "text": "legenda...",              // obrigatório, exceto Stories
  "media": ["https://.../reel.mp4"],
  "providers": [{"network":"instagram"},{"network":"tiktok"}],
  "publicationDate": {"dateTime":"2026-08-19T12:00:00","timezone":"America/Sao_Paulo"},
  "autoPublish": true,
  "draft": false,
  "instagramData": {"type":"REEL","showReelOnFeed":true},
  "tiktokData": {"privacyOption":"PUBLIC_TO_EVERYONE","title":"..."}
}
```

- `date` (parâmetro, ISO com offset) e `publicationDate` (dentro do `info`, com IANA) são **os dois necessários**
- Só inclua o `<rede>Data` das redes que estão em `providers`
- O fuso vem do `getBrandSettings`

### Requisitos por rede

| Rede | Obrigatório | Observações |
|---|---|---|
| **Instagram** | imagem ou vídeo | `type`: `POST`, `REEL`, `STORY`, `TRIAL_REEL`. Reel exige vídeo. Colaboradores, áudio do catálogo (só Business + Página do FB) |
| **TikTok** | imagem ou vídeo | `privacyOption`, comentários/duet/stitch. Capa personalizada **só em conta Business** |
| **YouTube** | vídeo + `title` + `madeForKids` | `type`: `video` ou `short`; privacidade, tags, categoria |
| **Facebook** | Reel exige vídeo; Story, imagem ou vídeo | `type`: `POST`, `REEL`, `STORY` |
| **Bluesky** | — | máx. 300 caracteres |
| **X/Twitter** | — | um post só, nunca thread; limite varia com a assinatura |
| **Pinterest** | imagem + board | `boardId` numérico como string |

### Capa de vídeo

`videoThumbnailUrl` (URL de imagem) ou `videoCoverMilliseconds` (frame por offset). Só valem em IG Reel/Trial Reel, TikTok Business, YouTube vídeo (não Short), LinkedIn, Facebook Post/Reel. **Fora disso a requisição inteira é rejeitada** com `VIDEO_THUMBNAIL_NOT_APPLICABLE` — não mande "no chute".

## Editar um post agendado

`updateScheduledPost` exige **`id` e `uuid`** (pegue com `getScheduledPosts`) e o **payload completo**: o que você não mandar é apagado. Não há confirmação nem undo.

Detalhe observado: **o `id` muda** depois do update; o `uuid` permanece. Use o `uuid` como chave estável.

## Checklist

- [ ] Master H.264, 1080×1920
- [ ] Nome sem espaço/acento
- [ ] Upload no release + URL conferida
- [ ] `blogId` e `timezone` vindos do `getBrandSettings`
- [ ] `<rede>Data` só das redes em `providers`
- [ ] YouTube: conferir se não duplica no canal de origem
- [ ] Primeira vez: `draft` ou `autoPublish: false`
