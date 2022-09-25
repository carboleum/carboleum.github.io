---
layout: post
title:  "Initiation au backtesting - Implémentation sous Python"
date:   2022-09-25 20:12:04 +0200
categories: trading
---

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
  <script id="MathJax-script" async
          src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
  </script>
  
  
<h3> Installation libs </h3>

<h4> TA-Lib </h4>
```bash
sudo apt install -y build-essential wget
wget http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz
tar -xvf ta-lib-0.4.0-src.tar.gz
cd ta-lib
./configure --prefix=/usr
make
sudo make install
```
<h4> Libs Python </h4>
```bash
sudo pip3 install pandas TA-Lib bokeh
```

<h3> Import libs </h3>
```python
import requests
import numpy as np
import pandas as pd
import talib as ta
from bokeh.plotting import figure,show
#from bokeh.layouts import column,row
```

<h3> Téléchargement des données </h3>
```python
pair = 'btceur'
period = 12*60*60 # en secondes
url = f'https://api.cryptowat.ch/markets/kraken/{pair}/ohlc'
ohlc = requests.get(url).json()['result'][str(period)]
columns = ['time','open','high','low','close','volume','count']
df = pd.DataFrame(ohlc, columns=columns).astype(float)
```

![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot.png){:style="display:block; margin-left:auto; margin-right:auto"}

<h3> Définition stratégie </h3>

\\( SIG_{in} \equiv \bigl( SMA_{14} > SMA_{200} \bigr) \ \&\  \bigl( RSI_{14} > 60 \bigr) \\)

\\( SIG_{out} \equiv \big( RSI_{14} < 40 \big) \\)

```python
# strategy
RSI = ta.RSI(df.close, timeperiod=14)
SMA_200 = ta.SMA(df.close, timeperiod=200)
SMA_14 = ta.SMA(df.close,timeperiod=14)
SIG_in = (RSI > 60) & (SMA_200 < SMA_14)
SIG_out = (RSI < 40)
```

<h3> Signal </h3>

```python
SIG_0 = SIG_in.astype(int) - SIG_out.astype(int)
SIG_1 = SIG_0.where(SIG_0!=0).ffill()
POS = SIG_1 > 0
```

<h3> Calcul du rendement </h3>

```python
r_0 = df.close / df.close.shift()
r_strat = np.where(POS.shift(), r_0, 1)
r_fee = np.where(POS.shift() + POS == 1, 1-0.0025, 1)
```

<h3> Rendement cumulé </h3>

```python
R_net = np.nancumprod(r_strat * r_fee)
```

<h3> Graphique </h3>

```python
# graphique
p = figure(height=300)
p.line(df.time,df.r_0.cumprod(),color='lightgray')
p.line(df.time,df.r_strat.cumprod())
p.line(df.time,df.R_net,color='red')
show(p)
```


![Graph BTC - 12h]({{site.url}}/assets/bokeh_plot-8.png){:style="display:block; margin:auto"}


Avec:
  * en grisé, le rendement cumulé en HOLD
  * en bleu, le rendement brut cumulé de la stratégie
  * en rouge, le rendement net cumulé de la stratégie 

<h3> Implémentation </h3>

[kissbacktest](https://github.com/carboleum/kissbacktest)

