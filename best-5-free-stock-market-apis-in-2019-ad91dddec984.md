Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m100[39m, end: [33m110[39m }

# Best 5 free stock market APIs in 2020

Photo by Chris Li on Unsplash

The financial APIs market grows so quickly that last year’s post or platform is not a good choice this year. So in this story, I will show you the best 5 stock market APIs that I use in 2019.

## What is stock market data API?

Stock market data APIs offer real-time or historical data on financial assets that are currently being traded in the markets. These APIs usually offer prices of public stocks, ETFs, ETNs.

These data can be used for generating technical indicators which are the foundation to build trading strategies and monitor the market.

## Data

In this story, I mainly care about price information. For other data, there are some other APIs mainly for that use cases which will not be covered here.

I will talk about the following APIs and where they can be used:

* Yahoo Finance

* Google Finance in Google Sheets

* IEX Cloud

* AlphaVantage

* World trading data

* Other APIs (Polygon.io, Intrinio, Quandl)

## 1. Yahoo Finance

**Docs: [yfinance](https://github.com/ranaroussi/yfinance)**

Yahoo Finance API was shut down in 2017. So you can see a lot of posts about alternatives for Yahoo Finance. However, it went back sometime in 2019. So you can still use Yahoo Finance to get free stock market data. Yahoo’s API was the gold standard for stock-data APIs employed by both individual and enterprise-level users.

Yahoo Finance provides access to more than 5 years of daily OHLC price data. And it’s free and reliable.

There’s a new python module [**yfinance](https://github.com/ranaroussi/yfinance)** that wraps the new Yahoo Finance API, and you can just use it.

    # To install yfinance before you use it.
    > pip install yfinance

Below is an example of how to use the API. Check out the Github link above to see the full document, and you are good to go.

<iframe src="https://medium.com/media/4b6bafc825e05f80c577a6fa88740402" frameborder=0></iframe>

## 2. [GOOGLEFINANCE](https://support.google.com/docs/answer/3093281?hl=en)

Google Finance is deprecated in 2012. However, it doesn’t shut down all the features. There’s a feature in Google Sheets that support you get stock marketing data. And it’s called [GOOGLEFINANCE](https://support.google.com/docs/answer/3093281?hl=en) in Google Sheets.

The way it works is to type something like below and you will get the last stock price.

    GOOGLEFINANCE("GOOG", "price")

Syntax is:

    GOOGLEFINANCE(ticker, [attribute], [start_date], [end_date|num_days], [interval])

* **ticker:** The ticker symbol for the security to consider.

* **attribute**(Optional,"price" by default ): The attribute to fetch about ticker from Google Finance.

* **start_date**(Optional): The start date when fetching historical data.

* **end_date|num_days**(Optional): The end date when fetching historical data, or the number of days from start_date for which to return data.

* **interval**(Optional): The frequency of returned data; either "DAILY" or "WEEKLY".

An example of use is attached.

![](https://cdn-images-1.medium.com/max/2724/1*Y0HdXc3PNA8oMTlAUEMwPg.png)

## 3. IEX Cloud

**Website: [https://iexcloud.io/](https://iexcloud.io/)**

![](https://cdn-images-1.medium.com/max/6240/1*sieqti5CJ3nTGDLho0KCXQ.png)

IEX Cloud is a new financial service just released this year. It’s an independent business separate from IEX Group’s flagship stock exchange, is a high-performance, financial data platform that connects developers and financial data creators.

It’s very cheap compared to other subscription services. $9/month you almost can get all the data you need. Also, the basic free trial, you already get 500,000 core message free for each month.

There’s a python module to wrap their APIs. You can easily check it out: [**iexfinance](https://addisonlynch.github.io/iexfinance/stable/)**

## 4. Alphavantage

**Website: [https://www.alphavantage.co/](https://www.alphavantage.co/)**

![](https://cdn-images-1.medium.com/max/6240/1*xpC1NyaRfntW7umuhBK8JQ.png)

Alpha Vantage Inc. is a leading provider of various free APIs. It provides APIs to gain access to historical and real-time stock data, FX-data, and cryptocurrency data.

With Alphavantage you can perform up to 5 API-requests per minute and 500 API requests per day. 30 API requests per minute with $29.9/month.

## 5. World trading data

**Website: [https://www.worldtradingdata.com/](https://www.worldtradingdata.com/)**

![](https://cdn-images-1.medium.com/max/6240/1*4JfeP_kXdadouZhZ9xhrug.png)

Also, full intraday data API and currency API access are given. For those who need more data points, plans from $8 per month to $ 32 per month are available.

Right now there are four different plans available. For free access, you can get up to 5 stocks per request (real-time API). Up to 250 total requests per day. The subscription plan is not that expensive, and you can get a

They provide URL and your response will be JSON format. There’s currently no available python module to wrap their API yet. So you have to use [**requests](https://requests.readthedocs.io/en/master/) **or other web modules to wrap their APIs.

## 6. Other APIs

**Website: [https://polygon.io](https://polygon.io/pricing)**

![](https://cdn-images-1.medium.com/max/2996/1*mdbl86j_ybpnrzmPU4recQ.jpeg)

It’s $199/month only for the US stock market. This is might be not a good choice for beginners.

**Website: [https://intrinio.com](https://intrinio.com/)**

![](https://cdn-images-1.medium.com/max/2786/1*pZXkNWDFm5hrQjZVLQhkMw.jpeg)

It’s $75/month only for the realtime stock market. Also, for EOD price data, it’s $40/month. You can get EOD price data almost free from other APIs I suggest. Even though they have 206 pricing feeds, ten financial data feeds and tons of other data to subscribe. The price is not that friendly for independent traders.

**Website: [https://www.quandl.com/](https://www.quandl.com/)**

![](https://cdn-images-1.medium.com/max/5792/1*eyqW218ON8TZnMDZWTbn4w.png)

Quandl is an aggregated marketplace for financial, economic and other related APIs. Quandl aggregates APIs from third-party marketplaces as services for users to purchase whatever APIs they want to use.

So you need to subscribe to the different marketplace to get different financial data. And different APIs will have different price systems. Some are free and others are subscription-based or one-time-purchase based.

Also, Quandl has an analysis tool inside its website.

Quandl is a good platform if you don’t care about money.

## Wrap Up

Learning and building a trading system is not easy. But the financial data is the foundation of all. If you have any questions, please ask them below.
