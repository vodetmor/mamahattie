# Mama Hattie — 51 Southern Remedies

Landing page de vendas para o ebook **"51 Southern Remedies My Grandmother Taught Me"** (Mama Hattie), com checkout da Hotmart **embutido diretamente na página** — sem redirecionamento, sem modal, sem clique para "abrir" o pagamento. O visitante lê a copy e o formulário de compra já está ali, na mesma rolagem.

- **Site ao vivo:** https://mamahattie.netlify.app
- **Produto na Hotmart:** ID 8187731 · link de checkout `https://pay.hotmart.com/V106872827E`
- **Repositório:** https://github.com/vodetmor/mamahattie
- **Backup local:** `D:\Mama Hattie\landing\index.html`

## Stack

Página estática, um único arquivo (`index.html`), zero build step. Hospedada na Netlify com deploy contínuo a partir deste repositório (branch `master` → produção).

## Estrutura da página

1. **Hero** — badge de urgência, capa do ebook, grid de 6 benefícios, preço, botão que rola suavemente até o checkout (`<a href="#checkout">`).
2. **Copy de vendas** — hook, as 9 seções do livro + bônus (#51), "each remedy includes", carta de encerramento da Mama Hattie.
3. **Prova social** — 6 depoimentos.
4. **Checkout embutido** (`#checkout`) — o formulário de pagamento real da Hotmart, carregado via `<iframe>`, aparece diretamente na página. Ver `AGENTS.md` para o detalhe técnico de como isso foi feito e como replicar para qualquer outro checkout.

## Preço

O produto está configurado na Hotmart em **USD $12** (idioma: English, moeda: Dólar Americano, conversão automática de moeda ligada). A copy da página está fixa em inglês — não há tradução automática por país. Para visitantes fora dos EUA, o **checkout embutido converte a moeda sozinho** (recurso nativo da Hotmart, não depende de nada nesta página); a copy ao redor permanece em inglês por design (oferta em mercado americano).

## Deploy

```bash
cd C:\dev\mamahattie
git add -A
git commit -m "mensagem"
git push          # dispara deploy automático na Netlify (se CD estiver linkado)

# ou deploy manual direto:
netlify deploy --prod --dir=.
```

## Editar a página

Arquivo único: `index.html`. Design system (cores, tipografia) segue o mesmo padrão do ebook (`D:\Mama Hattie\ebook\assemble.py`): pergaminho `#FBF6EA`, verde `#3C5A3A`, dourado `#B8862F`, serifado Georgia/Palatino.

Para trocar o link de checkout (ex: outro produto/oferta), procurar `V106872827E` no arquivo e trocar pelo novo código — aparece em dois lugares: no `href` do botão do topo (`#checkout`, não precisa mudar) e na `src` do iframe.
