---
layout: post
title:  "Initiation au backtesting - Approche mathématique"
date:   2022-08-10 20:12:04 +0200
categories: jekyll
---

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
  <script id="MathJax-script" async
          src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
  </script>
  
Soit le cours d'un actif (ici BTC au 24/07/2022 - périodes de 12h) : 
 
 ![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot.png){:style="display:block; margin-left:auto; margin-right:auto"}
 
Dans l'interval \\([t_{n-1},t_n]\\), le rendement vaut :
 
 $$ r_0([t_{n-1},t_n]) = { Prix(t_n)\over Prix(t_{n-1}) } $$
 
![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-1.png){:style="display:block; margin-left:auto; margin-right:auto"}
 
 Cette définition du rendement n'est pas des plus orthodoxes. Elle en est une simplification qui, dans le cadre du développement qui suit, est parfaitement satisfaisante. Petite particularité: le neutre est 1. \\(P(t_n) = P(t_{n-1} \rightarrow r_0 = 1\\)
 
 <h3> Signaux et positions </h3>
 
 La stratégie consiste à déterminer selon certains critères établis préalablement, les instants \\(t_n\\) pendant lesquels le marché est favorable à l'achat (\\(SIG_{achat}(t_n) = 1\\)) et/ou à la vente (\\(SIG_{vente}(t_n) = 1\\)). Entre le \\(1^{er}\\) signal d'achat et le \\(1^{er}\\) signal de vente suivant, on est en position.

\\(SIG_{achat}\\) :

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-2.png){:style="display:block; margin-left:auto; margin-right:auto"}

\\(SIG_{vente}\\) :

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-3.png){:style="display:block; margin-left:auto; margin-right:auto"}

\\(POS\\) :

\\[ POS(t_n) = \begin{cases} 1 & \text{si } SIG_{achat}(t_n) = 1\\\\ 0 & \text{si } SIG_{vente}(t_n) = 1 \\\\ POS(t_{n-1}) & \text{sinon} \end{cases} \\]

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-6.png){:style="display:block; margin-left:auto; margin-right:auto"}

Si, sur le papier, cette proposition est séduisante par sa simlicité, l'implémentation demande une étape intermédiaire:

\\(SIG_0\\) :

\\[ SIG_0(t_n) = SIG_{achat}(t_n) - SIG_{vente}(t_n) \\]

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-4.png){:style="display:block; margin-left:auto; margin-right:auto"}

\\(SIG_1\\) :

\\[ SIG_1(t_n) = \begin{cases} SIG_0(t_n) & \text{si } SIG_0(t_n) \ne 0 \\\\ SIG_0(t_{n-1}) & \text{sinon} \end{cases} \\]

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-5.png){:style="display:block; margin-left:auto; margin-right:auto"}

\\(POS\\) :

$$ POS \equiv SIG_1 > 0 $$

Cas particulier : le signal de vente est l'opposé du signal d'achat

$$ SIG_{vente} = 1 - SIG_{achat} \ \rightarrow\  POS \equiv SIG_{achat} $$

<h3> Rendement </h3>

Interprétation du signal \\(POS\\):

$$
\begin{array}{cc|c}
POS(t_{n-1}) & POS(t_n) & r_{strat}([t_{n-1},t_n]) \\ 
\hline
0 & 0 & 1 \\
0 & 1 & 1 \\
1 & 1 & r_0([t_{n-1},t_n]) \\
1 & 0 & r_0([t_{n-1},t_n]) \\
\end{array}
$$

Rendement de la statégie:

$$ r_{strat}([t_{n-1},t_n]) = \begin{cases} { Prix(t_n)\over Prix(t_{n-1}) } & \text{si } POS(t_{n-1}) = 1\\ 1 & \text{sinon} \end{cases}  $$

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-7.png){:style="display:block; margin-left:auto; margin-right:auto"}

Lors de chaque transaction (achat et vente), la plateforme prend un fee équivalent à \\(fee \%\\):

Même raisonnement que plus haut:

$$ r_{fee}(t_n) = \begin{cases} 1-fee & \text{si } POS(t_{n-1}) + POS(t_n) = 1 \\ 1 & \text{sinon} \end{cases} $$

<h3> Rendement cumulé </h3>

$$ R(t_n) = \prod_{i=1}^{t_n} \biggl( r_{strat}(i) \times r_{fee}(i) \biggr) $$


![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-8.png){:style="display:block; margin:auto"}


Avec:
  * en grisé, le rendement cumulé en HOLD
  * en bleu, le rendement brut cumulé de la stratégie
  * en rouge, le rendement net cumulé de la stratégie 

