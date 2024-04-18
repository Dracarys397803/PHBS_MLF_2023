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
**Compare models with and without tweets data**
**Use Grid search (scoring=acc) to find best hyper-parameters**
#### Preprocess
- Drop datapoints which contain no more than 7 tweets, **402** datapoints left
- Split the dataset into train and test
  * Test size=0.3
  * Shuffle=False to keep the time order of the data(Train with old data and test with relatively new data)
  * In train and test dataset, the frequencies of y=1 are equal(0.438)
 - Standardize the data
#### Single Model
1. Logistic Regression
- C=0.0001, penalty='l1' in both models
- Training acc = testing acc = 0.562, f1 score=0 in both models \
<img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/lr_without.png" width="300" height="450"> <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/lr_with.png" width="300" height="450">
2. SVM
- C=0.0001, kernel='linear'
- Training acc = testing acc = 0.562, f1 score=0 in both models \
<img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/svm_without.png" width="300" height="450"> <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/svm_with.png" width="300" height="450"> \
**Single model does not show any prediction ability**
#### Ensemble Learning
1. Random Forest
- Base model: max_depth=1, max_features='log2', n_estimators=110, traing acc=0.555
- Full model: max_depth=1, max_features='sqrt', n_estimators=160, traing acc=0.562
- Testing acc = 0.562, f1 score=0 in both models \
<img src="https://github.com/Dracarys397803/PHBS_MLF_2023/assets/160571976/0d386a30-1626-42b1-8195-12c6a69acd57" width="300" height="450"> <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/assets/160571976/4b794613-1e1b-4f90-af29-8b0f113d1237" width="300" height="450"> \
2. Adaboost (Tree)
- Base model: learning_rate=0.1, n_estimators=10, training acc=0.495, testing acc=0.529, f1 score=0.374
- Full model: learning_rate=0.7, n_estimators=10, training acc=0.498, testing_acc=0.479, f1 score=0.364 \
<img src="https://github.com/Dracarys397803/PHBS_MLF_2023/assets/160571976/289d654c-4e1c-4785-bb51-2fcede29d1de" width="300" height="450"> <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/assets/160571976/57f7130c-d9e5-4e4a-a16c-3527750cc93c" width="300" height="450"> \
#### PCA
- Use all predictors and only involve ensemble models\
  <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/pca.png" width="450" height="400"> 
- PCA results show 5 components are enough
- **Random Forest**: max_depth=1, max_features='sqrt', n_estimators=50, training acc=0.566, testing acc=0.562, f1 score=0
  <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/assets/160571976/c4083033-1b0c-40b7-aaae-d25f72114720" width="300" height="450">
- **Adaboost**: learning_rate=0.6, n_estimators=80, training acc=0.526, testing_acc=0.521, f1 score=0.408
  <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/assets/160571976/89d16c42-44a9-4b11-aea7-0817ab60622f" width="300" height="450">
#### Kernel PCA
- Use all predictors and only involve ensemble models\
  <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/blob/main/Image/kernel_pca.png" width="450" height="400">
- 50 components are enough
- **Random Forest**: max_depth=1, max_features='sqrt', n_estimators=20, training acc=0.562, testing acc=0.562, f1 score=0
  <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/assets/160571976/20ca5d18-7556-48c3-b6a8-71214333ce49" width="450" height="800"> 
- **Adaboost**: learning_rate=1, n_estimators=190, training acc=0.548, testing_acc=0.529, f1 score=0.278
  <img src="https://github.com/Dracarys397803/PHBS_MLF_2023/assets/160571976/a924beb0-f398-44c9-83aa-81fb254f1bd1" width="450" height="800">
### Conclusion
|  Model   |Training Acc|Testing Acc| F1 score | ROC AUC |
|---------:|-----------:|----------:|---------:|--------:|
| LR_Base  |    0.562   |   0.562   |    0     |    0.5  |
| LR_Full  |    0.562   |   0.562   |    0     |    0.5  |
| SVM_Base |    0.562   |   0.562   |    0     |    0.5  |
| SVM_Full |    0.562   |   0.562   |    0     |    0.5  |
| RF_Base  |    0.555   |   0.562   |    0     |    0.52 |
| RF_Full  |    0.562   |   0.562   |    0     |    0.5  |
| Ada_Base |    0.495   |   0.529   |    0.374 |    0.51 |
| Ada_Full |    0.498   |   0.479   |    0.364 |    0.46 |
| PCA_RF   |    0.566   |   0.562   |    0     |    0.48 |
| PCA_Ada  |    0.526   |   0.521   |    0.408 |    0.50 |
| KPCA_RF  |    0.576   |   0.562   |    0     |    0.54 |
| KPCA_Ada |    0.548   |   0.529   |    0.278 |    0.49 |
1. Both base and full models do not perform well in predicting price change of BTC
2. Ensemble learning performs slightly better than single model

**Interpretation**
1. For technical factors, I ignore some BTC-specific factors due to data availability (e.g. Bitcoin Difficulty, Active Addresses, Network Value to Transactions Ratio etc.).
2. For tweets data, the data only focuses on financial influencers' tweets. In a highly decentralized market, using all tweets related to cryptocurrency may have a better prediction power. Besides, it is also possible that the sentiment effect appears in a different time range (i.e. Minute, Hour, Week, Month)
3. The result may also simply suggest tweeters have less power in influencing the future price of BTC than they and I believe.
