# AGENTS.md — Checkout embutido via iframe (técnica replicável)

Este documento registra a técnica usada para embutir o checkout da Hotmart **diretamente dentro da landing page**, com scroll único e sem exigir clique para "abrir" o pagamento. A técnica não é específica da Hotmart — qualquer checkout/gateway que não bloqueie iframe pode ser embutido do mesmo jeito. Ler isto antes de tentar replicar para outro produto, outra plataforma de checkout, ou outro site.

## O problema original

Plataformas como **Stan Store** ou **ThriveCart (modo embutido)** têm o formulário de pagamento nativo na própria página — copy e checkout no mesmo template, mesma URL, um scroll só.

Plataformas como **Hotmart, Eduzz, Kiwify, ClickBank** isolam o checkout num domínio próprio (`pay.hotmart.com` etc.) por PCI-DSS, motor de fraude, conversão de moeda centralizada. O padrão dessas plataformas é: página de vendas → **clique** → redireciona ou abre modal/lightbox → só aí aparece o formulário.

O pedido aqui foi eliminar esse clique: o formulário devia **já estar visível**, integrado ao scroll da página, como se fosse nativo.

## A técnica: iframe direto, sem clique

Em vez do botão/lightbox oficial (`hotmart-fb` widget, que abre um modal ao clicar), embutimos a **própria URL do checkout** num `<iframe>` comum:

```html
<iframe
  src="https://pay.hotmart.com/{CODIGO}?checkoutMode=2"
  height="1560"
  scrolling="no"
  loading="eager"
  title="Checkout">
</iframe>
```

Isso só funciona se o servidor do checkout **não enviar** os headers que bloqueiam framing. Testar ANTES de montar qualquer coisa:

```bash
curl -s -D - "https://pay.hotmart.com/{CODIGO}?checkoutMode=2" -o /dev/null \
  | grep -i -E "^x-frame-options|frame-ancestors"
```

- Se não retornar nada → sem bloqueio de header, o iframe deve carregar. **(Foi o caso da Hotmart — testado e confirmado.)**
- Se retornar `X-Frame-Options: DENY` ou `SAMEORIGIN`, ou `Content-Security-Policy` com `frame-ancestors` restritivo → **a técnica não funciona nesse checkout**, ponto final. Não adianta insistir com iframe; nesse caso a alternativa é o widget oficial em modal (clique) ou redirecionamento.

Isso é o primeiro passo obrigatório antes de tentar replicar para qualquer outra plataforma (Eduzz, Kiwify, Stripe Checkout hospedado, etc.) — cada uma pode ter política diferente, e pode mudar sem aviso.

> **ATUALIZAÇÃO 25/07/2026 — decisão superada em produção:** o `scrolling="no"` foi
> REMOVIDO. Motivo: formulários de países diferentes têm alturas diferentes
> (endereço, imposto, order bump, mais métodos de pagamento) e altura fixa sem
> scroll deixava campos INACESSÍVEIS para parte dos compradores — perda direta de
> venda. O modelo atual: seção do checkout em fundo BRANCO (funde com o form da
> Hotmart), altura generosa por breakpoint (1950px desktop / 2250px mobile) para
> que a maioria nunca veja scroll interno, e scroll interno habilitado como
> válvula de segurança para os casos maiores. A seção abaixo fica como registro
> histórico da técnica original.

## Por que `scrolling="no"` + altura fixa generosa (não auto-resize)

Como o iframe é **cross-origin**, o JavaScript da página pai não consegue ler `scrollHeight` do conteúdo interno (bloqueio padrão do navegador, same-origin policy). Isso significa:

- Não dá para redimensionar o iframe dinamicamameteno lado do pai sem cooperação do lado de dentro (a Hotmart não expõe um `postMessage` de altura que a gente possa escutar).
- A solução prática é **medir manualmente** e fixar uma altura alta o bastante para caber todo o conteúdo do formulário **sem gerar scroll interno** — assim existe apenas UM scroll (o da página), que é exatamente o efeito desejado ("a pessoa nem percebe que é outra página dentro da página").

Processo de calibragem usado aqui:
1. Comecei com `height="720"` → sobrou conteúdo, apareceu barra de scroll *dentro* do iframe (ruim — duas rolagens aninhadas).
2. Subi para `height="1900"` → cabia tudo, mas sobrava um espaço em branco grande no final (iframe maior que o conteúdo real).
3. Ajustei para `height="1560"` → encaixou quase exato, sem scroll interno e sem sobra perceptível.

Isso é **sensível à largura do container** (o checkout é responsivo e reflui para mais alto em containers estreitos) e ao conteúdo específico daquele produto (nome/descrição mais longos empurram a altura). Ao trocar de produto ou mudar a largura do container (`.wrap { max-width }`), repetir a calibragem: carregar a página, rolar até o fim com `End`, e ajustar a altura do iframe até o formulário terminar coincidindo com o fim do card (sem barra de scroll própria, sem espaço vazio grande).

## Por que só UM checkout, no final, com botão-âncora no topo

Primeira versão tinha dois iframes (topo + final) — funcionava, mas dobrava o carregamento e a calibragem de altura. Ajuste pedido pelo usuário: **um único iframe**, no final da página, e no topo apenas um botão-âncora:

```html
<a href="#checkout" class="cta-btn">Get Instant Access — $12</a>
...
<div id="checkout">
  <iframe src="..." height="1560" scrolling="no"></iframe>
</div>
```

Com `html { scroll-behavior: smooth; }` no CSS, o clique no botão do topo rola suavemente até o formulário — sem sair da página, sem abrir nada, sem carregar um segundo iframe.

## Verificação de que os campos realmente funcionam (não só carregam visualmente)

Carregar o iframe não é suficiente para confirmar que o formulário é utilizável — é preciso testar clique + digitação de verdade num campo, porque:
- Alguns checkouts detectam que estão sendo carregados dentro de um iframe (via `window.top !== window.self` no runtime, não via header HTTP) e podem desabilitar campos ou recusar renderizar o formulário mesmo sem bloqueio de header.
- Cliques via automação em iframe cross-origin podem "errar" o alvo se a página tiver rolado entre o cálculo de coordenadas e o clique — sempre tirar screenshot imediatamente antes de clicar para confirmar a posição exata, e um screenshot logo depois para confirmar que o cursor de texto (`|`) apareceu dentro do campo antes de digitar.

Neste projeto, testado e confirmado: cliquei no campo "Nome do titular", vi o cursor piscando, digitei "Test Buyer", o texto apareceu — os campos aceitam input normalmente dentro do iframe embutido.

## Generalização — checklist para replicar em outro checkout/site

1. `curl -I` na URL do checkout, procurar `X-Frame-Options` e `frame-ancestors` na CSP. Sem bloqueio → segue.
2. Montar o `<iframe>` com `scrolling="no"`, altura inicial qualquer (ex: 800).
3. Carregar a página real (servidor local ou já publicada — **iframes de terceiros costumam não carregar bem em `file://`**, sempre testar servido via HTTP).
4. Rolar até o fim com `End`, ver se sobra scroll interno (subir altura) ou espaço vazio (baixar altura). Repetir até bater.
5. Testar um campo de verdade: clicar, confirmar cursor piscando, digitar, confirmar que o texto aparece.
6. Se o produto/preço mudar, ou o container mudar de largura, repetir os passos 4–5 — a altura calibrada não é permanente.

## Limitações conhecidas

- Depende inteiramente da plataforma de checkout **não decidir bloquear iframing no futuro** — se a Hotmart adicionar `X-Frame-Options` um dia, o embed para de carregar sem aviso prévio. Não há como "forçar" contra um bloqueio de header do lado do servidor.
- A conversão de moeda/idioma automática (visitante de outro país vendo preço na moeda local) é um recurso **da Hotmart**, ativo nas configurações do produto ("Conversão de moeda automática") — não é algo controlado por este HTML. A copy ao redor do iframe (textos desta página) é estática e não traduz sozinha.
- Altura fixa em pixels é uma solução pragmática, não elegante — não reage a mudanças dinâmicas dentro do checkout (ex: cupom aplicado que adiciona uma linha). Se isso virar problema real, a alternativa correta seria a Hotmart expor um `postMessage` de altura (não expõe hoje) ou usar uma lib tipo `iframe-resizer` — mas isso exige cooperação do lado servido dentro do iframe, que não temos.
