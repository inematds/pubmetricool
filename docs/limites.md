# Limites

Fonte: página de preços da Metricool e central de ajuda, consultadas em 2026-08-19.

## Contas (marcas)

Uma **marca** ("brand"/`blogId`) agrupa as redes de um mesmo negócio.

### Quantos canais cabem numa marca

**Uma conta de cada rede.** Na tabela de preços, o número de conexões de cada rede é sempre igual ao número de marcas do plano — Free tem 1 marca e 1 Instagram, Starter tem 10 e 10, e assim por diante.

Cabem numa marca: Facebook, Instagram, Threads, Bluesky, Pinterest, TikTok, YouTube, Twitch, Google Business Profile, Web/Blog, Facebook Ads, Google Ads, TikTok Ads — **uma conta de cada**.

- **LinkedIn**: não existe no Free; entra a partir do Starter
- **X/Twitter**: add-on pago em qualquer plano, indisponível no Free

Consequência prática: **dois perfis do Instagram exigem duas marcas.** Não há como pendurar uma segunda conta da mesma rede na mesma marca.

| Plano | Marcas | Preço (anual, EUR) |
|---|---|---|
| **Free** | **1** | €0 |
| Starter | 5 | €16/mês |
| Starter | 10 | €29/mês |
| Advanced | 15 | €43/mês |
| Advanced | 25 | €69/mês |
| Advanced | 50 | €130/mês |
| Custom | sob consulta | — |

Preços mensais (sem compromisso anual) são mais altos: Starter a partir de €20, Advanced a partir de €54. Valores sem IVA.

**Free não conecta LinkedIn nem X/Twitter.** X exige add-on mesmo nos planos pagos.

> Cada Google Business Profile precisa de uma marca própria (uma marca = um local).

## Publicação

| Plano | Limite |
|---|---|
| **Free** | **20 posts por mês** |
| Starter+ | Ilimitado, sujeito à *Fair Use Policy* |

**Não há limite diário documentado pela Metricool.** O teto é mensal.

Duas ressalvas honestas:

1. **As redes têm limites próprios.** Instagram e TikTok impõem cotas de publicação via API, do lado deles. Não estão documentados pela Metricool e não foram verificados aqui — se for publicar em volume, testar antes de confiar.
2. **Se um post multi-rede consome 1 ou N da cota mensal, não está documentado.** Um post pra Instagram + TikTok pode contar como 1 ou como 2. **A confirmar empiricamente** ao longo do mês.

## Analytics

| Plano | Histórico |
|---|---|
| **Free** | **30 dias** |
| Starter+ | Ilimitado |

Se uma consulta vier vazia ou "faltando dados" no Free, suspeite da janela de 30 dias antes de tratar como erro.

Concorrentes: 5 perfis no Free, 100 no Starter. Só Instagram, Facebook, Twitch, YouTube, X e Bluesky suportam concorrentes.

## MCP vs API

| | MCP | API REST |
|---|---|---|
| Plano | **Qualquer um, incluindo Free** | **Só Advanced ou Custom** |
| Auth | OAuth no navegador | Token (`X-Mc-Auth`) |
| Uso | Cliente de IA, linguagem natural | Scripts, n8n, Make, Looker, Power BI |
| Token | não precisa | Definições da conta → API |

Os dois são produtos **separados**. Que o MCP funcione no Free não libera a API.

Cuidado com uma armadilha: alguns clientes de IA oferecem conexão "MCP" **por token**, que na verdade usa a API por baixo — **essa exige Advanced**. A conexão OAuth em `https://ai.metricool.com/mcp` é a que funciona no Free.

Docs da API (quando houver plano): `https://app.metricool.com/resources/apidocs/index.html`

## Limites do lado do cliente de IA

Não são cobrança da Metricool:

- **Claude Code**: exige assinatura Claude Pro ou Max
- **Claude Free**: só um conector personalizado por vez
- **ChatGPT**: precisa de acesso a Apps

## Resumindo pro caso INEMA

Hoje: **Free**, 1 marca (`inemafuturos`), **20 posts/mês**, 30 dias de analytics, sem LinkedIn/X.

20/mês ≈ **5 por semana**. Cabe pra curadoria; não cabe pra despejar o backlog do `yt-pub-lives` (989 clips publicados). Ver [integracao-yt-pub-livesx.md](integracao-yt-pub-livesx.md).
