# PHBS_MLF_2023
## Homework
Homework 1 is in the folder "give-me-some-credit" \
Homework 2 is in the main branch 

## Project
**The project aims to explore whether combining tweets data with common technical factors can predict BTC price change** 
### Data
#### Tweets Data
1. **The dataset is from huggingface. The link is https://huggingface.co/datasets/StephanAkkerman/financial-tweets-crypto.**
- The dataset is the scraped financial tweets collected from a variety of financial influencers on Twitter
- The original dataset contains: 
  * timestamp: ISO8601 time
  * tweet_text: the text of the tweet
  * tweet_url: the url of the tweet
  * tweet_type: tweet/retweet/quote tweet
  * tickers_mentioned: cryptocurrencies mentioned in the text
  * price_of_ticker: the price of tickers mentioned
  * change_of_ticker: 24h price change of tickers
  * category: whether the tweet contains image
- Keep columns which may be useful and drop data points which timestamp is null (Most of them are pure image)
   <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/tweets_data_nona.png" width="500" height="350">
2. **Group the data on a daily and weekly basis** 
  - Number of dates when tweets amount > 7 is 402 \
    <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/Date_distribution.png" width="600" height="350">
  - Number of weeks when tweets amount > 7 is 71 \
    <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/Week_distribution.png" width="600" height="350">
3. **Sentiment analysis**
  - Use a popular package *vaderSentiment* designed specially for social media text
    * Rule-based
    * Identify punctuation, degree modifier(e.g. very), slang and emoji
    * https://github.com/cjhutto/vaderSentiment
  - Results: \
    Compound is a index averaging the neg, neu and pos scores of the sentence
    <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/tweets_with_sentiment.png" width="800" height="500">
#### Market Data
1. **The dataset is from cryptocompare api.**
- The data is shown on a daily basis
- The original dataset contains:
  * time: unix timestamp
  * high: the highest price of the day
  * low: the lowest price of the day
  * open: the open price of the day
  * volumefrom: trade volume measured by BTC
  * volumeto: trade volume measured by USD
  * close: the close price of the day
2. **Transform unix timestamp into ISO8601 timestamp**
    <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/market_data.png" width="800" height="400">
#### Concat Data
1. **Group tweets data on a daily basis**
- BTC related tweets (Tweets which mentioned BTC)
- Total tweets (Tweets which related to cryptocurrencies)
  <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/tweets_data_all.png" width="800" height="400"> 
2. **Calculate technical factors and target varible**
- MOM_10: $$Close_t - Close_{t-10}$$
- SMA_5 & SMA_10: $$\frac{\displaystyle\sum_{i=0}^{k-1} Close_{t-i}}{k} (k=5, 10)$$
- RSI_14: $$\frac{\text{14 Period Average Gain}}{\text{14 Period Average Gain} + \text{14 Period Average Loss}}$$
          $$\frac{EMA_{(U, 14)}}{EMA_{(U, 14)}+EMA_{(D, 14)}}$$
          $$(U=Close-Open \text{ if } Close > Open \text{ else } 0, D=Open-Close \text{ if } Open > Close \text{ else } 0)$$
- MFI_14(A volume version of RSI_14): $$\frac{\text{14 Period Inflow}}{\text{14 Period Inflow} + \text{14 Period Outflow}}$$
                                      $$\text{Flow}=\text{Typical Price}*Volume$$
                                      $$\text{Typical Price}=\frac{High+Low+Close}{3}$$
- **Next_day_return(Target Varible)**: $$\text{If } Close_{t+1}>Close_t, 1$$
                                       $$\text{Else}, 0$$
### Training and Testing
