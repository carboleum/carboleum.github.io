---
layout: post
title:  "Introduction au backtesting"
date:   2022-08-10 20:12:04 +0200
categories: jekyll
---

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
  <script id="MathJax-script" async
          src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
  </script>

 Soit le cours d'un actif (ici BTC au 24/07/2022 - périodes de 12h) : 
 
 ![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot.png)
 
 <h3> Signaux et positions </h3>
 
 La stratégie consiste à déterminer des situations favorables à l'achat et des situations favorables à la vente. Entre le \\(1^{er}\\) signal d'achat et le \\(1^{er}\\) signal de vente suivant, on est en position.
 
\\[ SIG_{achat}(t_n) = \begin{cases} 1 & \text{si situation favorable à l'achat} \\\\ 0 & \text{sinon} \end{cases} \\]

\\[ SIG_{vente}(t_n) = \begin{cases} 1 & \text{si situation favorable à la vente} \\\\ 0 & \text{sinon} \end{cases} \\]

\\[ POS(t_n) = \begin{cases} 1 & \text{si } SIG_{achat}(t_n) = 1\\\\ 0 & \text{si } SIG_{vente}(t_n) = 1 \\\\ POS(t_{n-1}) & \text{sinon} \end{cases} \\]

\\(SIG_{achat}\\) :

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-2.png)

\\(SIG_{vente}\\) :

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-3.png)

\\(POS\\) :

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-6.png)

L'implémentation de cette proposition impose une étape supplémentaire :

\\[ SIG_0(t_n) = SIG_{achat}(t_n) - SIG_{vente}(t_n) \\]

\\[ SIG_1(t_n) = \begin{cases} SIG_0(t_n) & \text{si } SIG_0(t_n) \ne 0 \\\\ SIG_0(t_{n-1}) & \text{sinon} \end{cases} \\]

$$ POS \equiv SIG_1 > 0 $$

\\(SIG_0\\) :

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-4.png)

\\(SIG_1\\) :

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-5.png)

Cas particulier :

$$ SIG_{vente} = 1 - SIG_{achat} \ \rightarrow\  POS \equiv SIG_{achat} $$


<h3> Rendement </h3>

$$ r_0(t_n) = { Prix(t_n)\over Prix(t_{n-1}) } $$

Interprétation du signal $POS$:

  * \\( POS(t_{n-1}) = 0 \space \& \space POS(t_n) = 0 \rightarrow \\) Hors position de \\(t_{n-1}\\) à \\(t_n\\) (\\(\rightarrow r_{strat}(t_n) = 1\\))
  * \\( POS(t_{n-1}) = 0 \space \& \space POS(t_n) = 1 \rightarrow \\) Hors position de \\(t_{n-1}\\) à \\(t_n\\) (\\(\rightarrow r_{strat}(t_n) = 1\\)) et achat à l'instant \\(t_n\\) 
  * \\( POS(t_{n-1}) = 1 \space \& \space POS(t_n) = 1 \rightarrow \\) En position de \\(t_{n-1}\\) à \\(t_n\\) (\\(\rightarrow r_{strat}(t_n) = r_0(t_n) \\))
  * \\( POS(t_{n-1}) = 1 \space \& \space POS(t_n) = 0 \rightarrow \\) En position de \\(t_{n-1}\\) à \\(t_n\\) (\\(\rightarrow r_{strat}(t_n) = r_0(t_n) \\)) et vente à l'instant \\(t_n\\)

\\( \Rightarrow r_{strat}(t_n) \\) est conditionné par \\( POS(t_{n-1}) \\)

$$ r_{strat}(t_n) = \begin{cases} { Prix(t_n)\over Prix(t_{n-1}) } & \text{si } POS(t_{n-1}) = 1\\ 1 & \text{sinon} \end{cases}  $$

Si, en clôture de la bougie précédente, on a une position ouverte (\\(POS(t_{n-1})=1\\)), alors le rendement de la bougie actuelle vaut le rapport entre le prix de clôture de la bougie actuelle et celui de la précédente, sinon, le rendement vaut 1.

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-7.png)

Lors de chaque transaction (achat et vente), la plateforme prend un fee équivalent à \\(fee \%\\):

Même raisonnement que plus haut:

$$ r_{fee}(t_n) = \begin{cases} 1-fee & \text{si } POS(t_{n-1}) + POS(t_n) = 1 \\ 1 & \text{sinon} \end{cases} $$

<h3> Rendement cumulé </h3>

$$ R(t_n) = \prod_{i=1}^{t_n} \biggl( r_{strat}(i) \times r_{fee}(i) \biggr) $$


![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-8.png)


Avec:
  * en grisé, le rendement cumulé en HOLD
  * en bleu, le rendement brut cumulé de la stratégie
  * en rouge, le rendement net cumulé de la stratégie 