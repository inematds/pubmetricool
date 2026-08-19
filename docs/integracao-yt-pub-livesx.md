# Integração com o `yt-pub-livesx`

Proposta de desenho. **Nada aqui foi implementado ainda** — o que foi validado é o fluxo manual descrito em [publicar-video.md](publicar-video.md).

Hoje o pipeline corta lives e publica **no YouTube**. A ideia é estender: o mesmo clip vira também Reel e TikTok, via Metricool.

## Duas restrições que decidem o projeto — leia antes de qualquer código

### 1. Daemon não fala MCP

Tudo que foi validado rodou dentro de uma **sessão interativa do Claude Code**, com OAuth já concedido. O `scheduler.py` e o `import_worker.py` são daemons Python — eles **não têm** essa conexão.

Três caminhos, nenhum testado ainda:

| Caminho | Como | Custo | Risco |
|---|---|---|---|
| **A. `claude -p` headless** | O pipeline chama o Claude Code em modo não-interativo, que usa o MCP já autenticado | Consome cota Claude | Saída não-determinística; caro pra tarefa mecânica |
| **B. `mcp-remote` como cliente** | Python fala com `npx mcp-remote https://ai.metricool.com/mcp`. O token OAuth fica em cache (`~/.mcp-auth`) após um grant interativo | Grátis | Renovação do token não é clara; se expirar, o daemon quebra calado |
| **C. API REST** (`X-Mc-Auth`) | HTTP direto do Python, sem MCP | **Exige Advanced (€43+/mês)** | Nenhum — é o caminho projetado pra isso |

**Recomendação:** começar em **B** com prova de conceito curta (autenticar uma vez, verificar se sobrevive a um restart e a 24h). Se o token não se sustentar sozinho, a escolha real é entre A (mais caro por execução) e C (mensalidade). Não escreva o integrador inteiro antes de resolver isso — é o ponto que pode inviabilizar tudo.

### 2. A matemática do volume não fecha no Free

- `publicados` no lives8: **989 linhas**
- Plano Free: **20 posts/mês**, **1 marca**

E mais: cada instância `yt-pub-lives<N>` que publicar numa conta diferente precisa da **própria marca** — o Free só tem uma.

Dois modos possíveis, e é uma decisão de negócio, não técnica:

| Modo | O que é | Plano |
|---|---|---|
| **Curadoria** | ~20 clips/mês escolhidos a dedo (melhores, ou os já validados no YouTube) | Free serve |
| **Volume** | Todo clip publicado no YouTube vira Reel + TikTok | Starter mínimo (€16–29/mês); 5–10 marcas |

Um integrador que ignore isso descreve algo que não roda.

## Onde implementar

Regra do projeto (`CLAUDE.md` do `yt-pub-livesx`): **`yt-pub-livesx` é o template fonte**; `scripts/sync-instances` propaga. Então:

- Código novo em `yt-pub-livesx`
- Config **por instância**, em `config/.env` — nunca sincronizada
- Depois do sync, reiniciar os serviços afetados
- Bumpar a versão no `dashboard/index.html`

```
METRICOOL_ENABLED=false          # desligado por padrão
METRICOOL_BLOG_ID=6745962
METRICOOL_TIMEZONE=America/Sao_Paulo
METRICOOL_NETWORKS=instagram,tiktok
METRICOOL_MAX_PER_MONTH=20       # trava local, espelha a cota do plano
METRICOOL_MIN_DELAY_MIN=15       # nunca agendar pra "já"
MIDIA_REPO=inematds/midia
MIDIA_RELEASE_TAG=v1
```

## Tabela de rastreio

Sem ela não há idempotência — um restart do scheduler reagenda tudo.

```sql
CREATE TABLE IF NOT EXISTS metricool_posts (
  id               INTEGER PRIMARY KEY AUTOINCREMENT,
  publicado_id     INTEGER NOT NULL,      -- FK -> publicados.id
  clip_video_id    TEXT,                  -- id do YouTube, pra rastrear
  metricool_id     INTEGER,               -- muda a cada update
  metricool_uuid   TEXT,                  -- estável: use como chave
  networks         TEXT,                  -- "instagram,tiktok"
  media_url        TEXT,                  -- URL do release
  scheduled_at     TEXT,                  -- ISO com fuso
  status           TEXT,                  -- pending|scheduled|published|failed|skipped
  erro             TEXT,
  criado_em        TEXT DEFAULT (datetime('now')),
  UNIQUE(publicado_id, networks)
);
```

> **`metricool_id` muda a cada `updateScheduledPost`; o `uuid` não.** Guarde os dois, mas case pelo `uuid`.

O `UNIQUE(publicado_id, networks)` é a trava contra republicação dupla.

## Fluxo

```
publicados (novo registro)
      ↓
[1] resolve o master local
      lives.db: publicados.live_video_id
      → /mnt/hd8t/yt-pub-lives/yt-pub-lives<N>/lives/<live_video_id>/clips/
      → clips_manifest.json dá o nome real do arquivo + título + descrição
      ↓
[2] valida  H.264? 1080x1920? < 2 GB?   (transcodifica se preciso)
      ↓
[3] gh release upload <tag> <arquivo> --repo inematds/midia --clobber
      ↓
[4] checa cota do mês na metricool_posts  → estourou? status=skipped, para
      ↓
[5] createScheduledPost   (delay mínimo, nunca "agora")
      ↓
[6] grava id + uuid + status=scheduled
```

### Passo 1 — o detalhe que economiza tempo

**O `filename` da tabela `publicados` NÃO é o nome do arquivo no disco.**

Exemplo real: `publicados.filename = "import_20260804_videos_1"`, mas o arquivo é `reel-263-16.mp4`. Quem dá o nome verdadeiro é o **`clips_manifest.json`** na pasta da live.

O manifesto também traz `title` e `description` — **use a `description` como legenda**, é texto seu, já escrito, melhor que qualquer coisa gerada na hora.

### Passo 2 — codec

Master do pipeline já é H.264 (verificado). A validação é rede de proteção pra clips vindos de outras fontes — e é obrigatória se a origem for `yt-dlp`, que devolve **AV1**.

### Passo 5 — nome do arquivo

`publicados.clip_video_id` é único e ASCII. Use `clip-<clip_video_id>.mp4` — resolve o problema de espaço e acento de graça.

## YouTube: não republique

O `youtubeData` da marca `inemafuturos` aponta pro **`UC2QbQDyPKuHk93dwo5iq3Sw` = INEMA TDS**, que é o canal de **origem** das lives. O pipeline já publica os clips no **INEMA TIA** (`UCavuQHkxBSAZbzRoOm6Gq4g`), o canal de destino.

Ou seja: agendar YouTube pelo Metricool publicaria o clip **de novo, no canal de onde ele saiu**.

`METRICOOL_NETWORKS` deve ficar em `instagram,tiktok`. Se um dia quiser YouTube pelo Metricool, primeiro **reaponte a marca pro INEMA TIA** e desligue a publicação nativa do pipeline — as duas juntas duplicam.

## Ordem de implementação

1. **Resolver o item 1 (auth do daemon).** Prova de conceito do caminho B, 30 minutos. Sem isso, nada depois vale
2. Tabela `metricool_posts` + migração no `db.py`
3. Resolvedor de master (banco + manifesto → caminho no disco), com teste no clip `Ed9HFMSe0o4`
4. Uploader pro release, com `--clobber` e verificação da URL
5. Agendador, atrás de `METRICOOL_ENABLED=false`, começando em `draft=true`
6. Coluna no dashboard mostrando status Metricool
7. Só então ligar em uma instância, em modo curadoria

## Em aberto

- Auth do daemon (bloqueante)
- Post multi-rede consome 1 ou 2 da cota? Muda o dimensionamento em ~2×
- Precisa de capa/thumbnail própria pro Reel, ou o primeiro frame serve?
- Curadoria: manual, ou automática pelas métricas do YouTube (clip que foi bem vira Reel)?
- Limites de publicação do lado do Instagram/TikTok em volume — não documentados, não testados
