# Mama Hattie — 51 Southern Remedies

Landing page de vendas para o ebook **"51 Southern Remedies My Grandmother Taught Me"** (Mama Hattie), com checkout da Hotmart **embutido diretamente na página** — sem redirecionamento, sem modal. O visitante lê a copy e o formulário de compra já está ali, na mesma rolagem.

- **Site ao vivo (produção):** https://mamahattie.divinecalling21.com (alias: https://mamahattie.netlify.app)
- **Produto na Hotmart:** ID 8187731 · link de checkout `https://pay.hotmart.com/V106872827E` · **USD $12.95** (conversão automática de moeda ligada) · order bump "The Simple System to Look & Feel 10 Years Younger" $9.95
- **Repositório:** https://github.com/vodetmor/mamahattie
- **Assets master (PNG):** `D:\Mama Hattie\landing\assets-src\` (no repo ficam só os WebP de `img/`)

## Stack

Página estática, um único arquivo (`index.html`) + `img/*.webp`, zero build step. Hospedada na Netlify.

## Estrutura da página (v3 — ritmo claro/escuro)

1. **Hero** (verde-profundo) — headline + chips de benefício + preço $12.95 + CTA; **livro e tablet 3D em PNG transparente** flutuando sobre glow dourado.
2. **Meet Mama Hattie** (quase-preto) — retrato IA da Mama fundido full-bleed (rosto da oferta, continuidade com os vídeos do orgânico).
3. **Dor** (claro) — "You've tried everything" + 3 cards (diets/supplements/prescriptions).
4. **A Taste of What's Inside** (verde) — triptych full-bleed com fotos IA de 3 receitas (Golden Milk #26, Fire Cider #41, Rice Water #51).
5. **Inside You'll Discover** (claro) — as 9 seções + bônus #51 + "Each Remedy Includes".
6. **Signoff + Value stack + 18 reviews** (claro/faixa tonal).
7. **FAQ** (verde escuro) — acordeão nativo `<details>`, 6 objeções.
8. **Checkout** (seção **branca** inteira) — iframe Hotmart funde no fundo branco; alturas 1950px (desktop) / 2250px (mobile) + **scroll interno habilitado** como válvula: formulário de qualquer país sempre acessível.
9. **Sticky CTA** mobile/desktop com visibilidade por scroll.

## Rastreio (orgânico, sem pixel)

A página propaga `src`, `sck`, `utm_*` e `xcod` da URL para o iframe do checkout. Links dos vídeos do Facebook usam `https://mamahattie.divinecalling21.com/?src=fborg&sck=C{n}` → o relatório da Hotmart mostra qual vídeo gerou cada venda.

## Deploy

```bash
cd C:\dev\mamahattie
git add <arquivos> && git commit && git push   # versionamento
netlify deploy --prod --dir=.                  # deploy (o push NÃO dispara CD sozinho)
```

## Editar

Design system: verde `#1B2619/#2C3A2A/#3C5A3A`, dourado `#B8862F/#E8C468`, pergaminho `#FBF6EA`, serif Georgia/Palatino — mesmo DS do ebook (`D:\Mama Hattie\ebook\assemble.py`). Trocar checkout: buscar `V106872827E` no `index.html` (só na `data-src` do iframe). Ver `AGENTS.md` para a técnica do embed.
