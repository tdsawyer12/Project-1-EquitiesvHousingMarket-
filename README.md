<a href="https://imgur.com/MpkfU6j"><img src="https://i.imgur.com/MpkfU6j.png" title="source: imgur.com" /></a>

 - The below analysis aims to provide a broad comparison of returns and volatility between the stock market and the housing market over the past decade. 
 - The SPY index is used in our analysis to represent trends in the equity market.
 - The Case Shiller Index is used in our analysis to represent trends in the housing market.
 
 
## First, any relevant libraries are imported here. Libraries for Visualization include Panel, Hvplot, and Matplotlib. API Libraries Alpaca_trade_api, Requests, and Dotenv. Libraries for Calculation/Analytics include Pandas and Numpy. ##

```python
import panel as pn
import os
import pandas as pd
import matplotlib.pyplot as plt
import hvplot.pandas
import plotly.express as px
from dotenv import load_dotenv
from panel.interact import interact
import requests
import numpy as np
import alpaca_trade_api as tradeapi
pn.extension('plotly')
%matplotlib inline
```

## Case Shiller data on 20 of the largest US cities are collected from the Federal Reserve Economic Data (FRED) API and combined into a single data frame.##

```
cities = ['PHXRNSA','ATXRSA','BOXRSA','NYXRSA','DAXRSA','SEXRNSA','CHXRSA','MIXRNSA','POXRSA','CEXRSA','WDXRSA','DNXRSA', 'CSUSHPISA']
dfs = []
for city in cities:
    city_url = f"https://fred.stlouisfed.org/graph/fredgraph.csv?bgcolor=%23e1e9f0&chart_type=line&drp=0&fo=open%20sans&graph_bgcolor=%23ffffff&height=450&mode=fred&recession_bars=on&txtcolor=%23444444&ts=12&tts=12&width=1168&nt=0&thu=0&trc=0&show_legend=yes&show_axis_titles=yes&show_tooltip=yes&id={city}&scale=left&cosd=2010-01-01&coed=2020-05-01&line_color=%234572a7&link_values=false&line_style=solid&mark_type=none&mw=3&lw=2&ost=-99999&oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&fgst=lin&fgsnd=2020-02-01&line_index=1&transformation=lin&vintage_date=2020-08-15&revision_date=2020-08-15&nd=1987-01-01"
    df = pd.read_csv(city_url, index_col='DATE')
    dfs.append(df)
df = pd.concat(dfs, axis=1)
df.rename(columns={'PHXRNSA': 'Phoenix', 'ATXRSA': 'Atlanta', 'BOXRSA': 'Boston', 'NYXRSA': 'New York', 'DAXRSA': 'Dallas', 'SEXRNSA': 'Seattle', 'CHXRSA': 'Chicago', 'MIXRNSA': 'Miami', 'POXRSA': 'Portland', 'CEXRSA': 'Cleveland', 'WDXRSA': 'Washington D.C.', 'DNXRSA': 'Denver', 'CSUSHPISA': 'U.S.A'}, inplace=True)
df
```
<a href="https://imgur.com/RPHsLql"><img src="https://i.imgur.com/RPHsLql.jpg" title="source: imgur.com" /></a>

## The below chart utilizes Hvplot to allow the user to view housing price trends for specific cities using the drop-down menu. 

```python
cities = df.columns.tolist()
def myplot(city):
    return df.hvplot(x='DATE', y=city, rot=90, width = (1000), height = (500), title = 'Housing price data by city over the last 10 years')
```
<a href="https://imgur.com/SEvrVkW"><img src="https://i.imgur.com/SEvrVkW.jpg" title="source: imgur.com" /></a>

## To make an apples to apples comparison between the two markets, we converted price changes in the Case Shiller index into percentage terms. ##

```
usa_housing_market = pd.DataFrame(df, columns=['U.S.A'])
usa_house_pct = usa_housing_market.pct_change()
usa_house_pct.reset_index(inplace = True)
usa_house_pct.head()
```
<a href="https://imgur.com/uFtkZfm"><img src="https://i.imgur.com/uFtkZfm.jpg" title="source: imgur.com" /></a>

## Next, we pulled in stock market (SPY) information using the Alapacas API. ##
- The SPY data is adjusted to a monthly average and set to allign with the Case Shiller's Date/Time intervals above.

```
ticker = ["SPY"]
timeframe = "1D"
start_date = pd.Timestamp('2010-01-01', tz='America/New_York').isoformat()
end_date = pd.Timestamp('2020-05-01', tz='America/New_York').isoformat()

df_spy = api.get_barset(
    ticker,
    timeframe,
    limit=None,
    start=start_date,
    end=end_date,
    after=None,
    until=None,
).df

# Only comparing closing value
df_spy = df_spy.drop(
    columns=['open', 'high', 'low', 'volume'],
    level=1
)
#Converts data to reflect a monthly average for comparison against housing data.
df_spy.index = df_spy.index.date
df_spy.reset_index(inplace = True)
df_spy.rename(columns={'index': 'Date'}, inplace= True)
dfspy = df_spy.set_index('Date')
dfspy.index = pd.to_datetime(dfspy.index)
monthly_spy = dfspy['SPY'].resample('M').mean()
monthly_spy
```
<a href="https://imgur.com/eWxJOyj"><img src="https://i.imgur.com/eWxJOyj.jpg" title="source: imgur.com" /></a>

## The below chart visualizes the returns of the SPY index based on closing prices over the last decade. ##

<a href="https://imgur.com/JP1GcFg"><img src="https://i.imgur.com/JP1GcFg.jpg" title="source: imgur.com" /></a>

## Next, the two data sets are combined to show a side by side comparison.##

```
Housing_vs_Stocks = pd.concat([usa_house_pct.reset_index(drop=True),spy_pct.reset_index(drop=True)], axis=1)
Housing_vs_Stocks = Housing_vs_Stocks.set_index('DATE')
Housing_vs_Stocks.rename(columns={'U.S.A.':'US Housing Maket','close': 'Stock Index'}, inplace= True)
Housing_vs_Stocks.head()
```
<a href="https://imgur.com/Q9MU5ZK"><img src="https://i.imgur.com/Q9MU5ZK.jpg" title="source: imgur.com" /></a>

## How do the two markets compare? ##
-I this comparison, the SPY index seems to outperform the housing market over the long term but has greater volatility. 
-This analysis ignores numerous critical factors involved in making a comprehensive comparison between holding the two types of assets. Performance metrics for both markets ignore transaction costs, cash flow, taxes, and depreciation.
```
cumulative_returns = (1 + Housing_vs_Stocks).cumprod()
cumulative_returns.hvplot(y=['U.S.A', 'Stock Index'],width = (1000), height = (500), xticks = (10),
             value_label='Observed Cumulative Returns')
```
<a href="https://imgur.com/y4JgTV1"><img src="https://i.imgur.com/y4JgTV1.jpg" title="source: imgur.com" /></a>

# In this project, we brought together Python coding, APIs, EDA, Visualization, and Comparison techniques to demonstrate a collection of technical, analytical skills. #
