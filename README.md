```python
import requests
import pandas as pd
from datetime import datetime, timedelta
from pycoingecko import CoinGeckoAPI
```


```python
cg = CoinGeckoAPI()
price_data = cg.get_coin_market_chart_by_id(id='bitcoin', vs_currency='usd', days=30)

# Extract daily prices from timestamp
prices = []
for entry in price_data['prices']:
    date = datetime.utcfromtimestamp(entry[0] / 1000).date()
    prices.append({'date': date, 'price': entry[1]})

# Group by date (since CoinGecko gives multiple prices per day)
df_prices = pd.DataFrame(prices).groupby('date', as_index=False).mean()
```


```python
# 2. Get 30 news titles from NewsAPI
# -----------------------------
API_KEY = '6ece552a55d7445b81152343964547e2'  # 🔑 Replace with your actual NewsAPI key

end_date = datetime.now()
start_date = end_date - timedelta(days=30)

url = 'https://newsapi.org/v2/everything'
params = {
    'q': 'Bitcoin',
    'from': start_date.strftime('%Y-%m-%d'),
    'to': end_date.strftime('%Y-%m-%d'),
    'sortBy': 'publishedAt',
    'language': 'en',
    'pageSize': 30,
    'apiKey': API_KEY
}

response = requests.get(url, params=params)
news_json = response.json()

# Extract date and title from each article
articles = news_json.get('articles', [])
news_data = [{'date': article['publishedAt'][:10], 'title': article['title']} for article in articles]
df_news = pd.DataFrame(news_data)
df_news['date'] = pd.to_datetime(df_news['date']).dt.date
```


```python
# 3. Merge both DataFrames on 'date'
# -----------------------------
# Merging: for each news title, match with that day's price
df_merged = pd.merge(df_news, df_prices, on='date', how='left')
```


```python
# 4. Save to CSV
# -----------------------------
df_merged.to_csv('bitcoin_news_and_prices_30_days.csv', index=False)
print("✅ Data saved to 'bitcoin_news_and_prices_30_days.csv'")
```

    ✅ Data saved to 'bitcoin_news_and_prices_30_days.csv'



```python

```


```python
# 1. Get historical daily prices from CoinGecko
# -----------------------------
cg = CoinGeckoAPI()
price_data = cg.get_coin_market_chart_by_id(id='bitcoin', vs_currency='usd', days=30)

# Extract daily prices
daily_prices = {}
for entry in price_data['prices']:
    date = datetime.utcfromtimestamp(entry[0] / 1000).date()
    if date not in daily_prices:  # Only get the first price of the day
        daily_prices[date] = entry[1]

# Convert to DataFrame and sort
df_prices = pd.DataFrame([{'date': date, 'price': price} for date, price in daily_prices.items()])
df_prices.sort_values('date', inplace=True)
df_prices.reset_index(drop=True, inplace=True)
```


```python
# 2. Get 30 recent Bitcoin news headlines from NewsAPI
# -----------------------------
API_KEY = '6ece552a55d7445b81152343964547e2'  # Replace with your actual NewsAPI key

end_date = datetime.today()
start_date = end_date - timedelta(days=30)

url = 'https://newsapi.org/v2/everything'
params = {
    'q': 'Bitcoin',
    'from': start_date.strftime('%Y-%m-%d'),
    'to': end_date.strftime('%Y-%m-%d'),
    'sortBy': 'publishedAt',
    'language': 'en',
    'pageSize': 100,
    'apiKey': API_KEY
}

response = requests.get(url, params=params)
news_json = response.json()

# Group news articles by date and pick the first title per day
seen_dates = set()
news_by_date = []
for article in news_json.get("articles", []):
    date = article["publishedAt"][:10]
    if date not in seen_dates:
        news_by_date.append({'date': datetime.strptime(date, "%Y-%m-%d").date(), 'title': article['title']})
        seen_dates.add(date)
    if len(news_by_date) >= 30:
        break

# Convert to DataFrame
df_news = pd.DataFrame(news_by_date)
df_news.sort_values('date', inplace=True)
df_news.reset_index(drop=True, inplace=True)
```


```python
# 3. Merge both into one DataFrame
# -----------------------------
df_combined = pd.merge(df_prices, df_news, on='date', how='inner')  # only keep matching days
```


```python
# Save to CSV
df_combined.to_csv('bitcoin_prices_and_news_30_days.csv', index=False)
print("✅ CSV saved as 'bitcoin_prices_and_news_30_days.csv'")
```

    ✅ CSV saved as 'bitcoin_prices_and_news_30_days.csv'



```python

```


```python
def get_historical_prices(coin_id, vs_currency='usd', from_date='2023-01-01', to_date='2024-12-31'):
    from_timestamp = int(datetime.strptime(from_date, "%Y-%m-%d").timestamp())
    to_timestamp = int(datetime.strptime(to_date, "%Y-%m-%d").timestamp())
    url = f"https://api.coingecko.com/api/v3/coins/{coin_id}/market_chart/range"
    params = {
        'vs_currency': vs_currency,
        'from': from_timestamp,
        'to': to_timestamp
    }
    response = requests.get(url, params=params)
    data = response.json()
    prices = data['prices']
    df = pd.DataFrame(prices, columns=["timestamp", "price"])
    df['date'] = pd.to_datetime(df['timestamp'], unit='ms').dt.date
    df = df.groupby('date').first().reset_index()
    return df

# Пример для биткоина:
btc_prices = get_historical_prices('bitcoin')
print(btc_prices.head())
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    Cell In[12], line 19
         16     return df
         18 # Пример для биткоина:
    ---> 19 btc_prices = get_historical_prices('bitcoin')
         20 print(btc_prices.head())


    Cell In[12], line 12, in get_historical_prices(coin_id, vs_currency, from_date, to_date)
         10 response = requests.get(url, params=params)
         11 data = response.json()
    ---> 12 prices = data['prices']
         13 df = pd.DataFrame(prices, columns=["timestamp", "price"])
         14 df['date'] = pd.to_datetime(df['timestamp'], unit='ms').dt.date


    KeyError: 'prices'



```python
import requests
import pandas as pd
from datetime import datetime

def get_top_10_coin_ids():
    url = "https://api.coingecko.com/api/v3/coins/markets"
    params = {
        'vs_currency': 'usd',
        'order': 'market_cap_desc',
        'per_page': 10,
        'page': 1
    }
    response = requests.get(url, params=params)
    coins = response.json()
    return [(coin['id'], coin['symbol'].upper()) for coin in coins]

def get_price_data(coin_id, start='2024-12-01', end='2024-12-31'):
    from_ts = int(datetime.strptime(start, "%Y-%m-%d").timestamp())
    to_ts = int(datetime.strptime(end, "%Y-%m-%d").timestamp())
    url = f"https://api.coingecko.com/api/v3/coins/{coin_id}/market_chart/range"
    params = {
        'vs_currency': 'usd',
        'from': from_ts,
        'to': to_ts
    }
    response = requests.get(url, params=params)
    data = response.json()
    prices = data.get('prices', [])
    df = pd.DataFrame(prices, columns=['timestamp', 'price'])
    df['date'] = pd.to_datetime(df['timestamp'], unit='ms').dt.date
    df = df.groupby('date').first().reset_index()
    df['coin_id'] = coin_id
    return df[['date', 'coin_id', 'price']]

# Получаем топ-10 монет
top_coins = get_top_10_coin_ids()
print("Топ-10 монет:", top_coins)

# Собираем данные по всем монетам
all_prices = pd.DataFrame()
for coin_id, symbol in top_coins:
    print(f"Собираем данные для {symbol}...")
    df = get_price_data(coin_id)
    df['symbol'] = symbol
    all_prices = pd.concat([all_prices, df], ignore_index=True)

# Сохраняем в CSV
all_prices.to_csv("top10_crypto_prices_2023_2024.csv", index=False)
print("✅ Готово! Данные сохранены в top10_crypto_prices_2023_2024.csv")
```

    Топ-10 монет: [('bitcoin', 'BTC'), ('ethereum', 'ETH'), ('tether', 'USDT'), ('ripple', 'XRP'), ('binancecoin', 'BNB'), ('usd-coin', 'USDC'), ('solana', 'SOL'), ('tron', 'TRX'), ('dogecoin', 'DOGE'), ('cardano', 'ADA')]
    Собираем данные для BTC...
    Собираем данные для ETH...
    Собираем данные для USDT...
    Собираем данные для XRP...
    Собираем данные для BNB...
    Собираем данные для USDC...
    Собираем данные для SOL...
    Собираем данные для TRX...
    Собираем данные для DOGE...
    Собираем данные для ADA...
    ✅ Готово! Данные сохранены в top10_crypto_prices_2023_2024.csv



```python
import requests
import pandas as pd
from datetime import datetime

CRYPTO_PANIC_API_KEY = 'твой_API_ключ_сюда'

def get_news(symbol, max_pages=3):
    all_news = []
    base_url = "https://cryptopanic.com/api/v1/posts/"
    for page in range(1, max_pages + 1):
        params = {
            'auth_token': CRYPTO_PANIC_API_KEY,
            'currencies': symbol.lower(),
            'public': 'true',
            'page': page
        }
        response = requests.get(base_url, params=params)
        if response.status_code != 200:
            print(f"Ошибка для {symbol} на странице {page}")
            continue
        results = response.json().get('results', [])
        for item in results:
            all_news.append({
                'symbol': symbol.upper(),
                'title': item.get('title'),
                'published_at': item.get('published_at'),
                'domain': item.get('domain'),
                'url': item.get('url')
            })
    return all_news

# Получим список символов топ-10 монет
top_symbols = [symbol for _, symbol in get_top_10_coin_ids()]
all_articles = []

# Собираем заголовки по каждой монете
for symbol in top_symbols:
    print(f"Собираем новости для {symbol}...")
    news = get_news(symbol)
    all_articles.extend(news)

# Сохраняем в DataFrame
news_df = pd.DataFrame(all_articles)
news_df['published_at'] = pd.to_datetime(news_df['published_at'])
news_df.to_csv("top10_crypto_news_titles_2023_2024.csv", index=False)
print("✅ Заголовки сохранены в top10_crypto_news_titles_2023_2024.csv")
```


```python
import requests
import pandas as pd
from datetime import datetime, timedelta

API_KEY = '4a307c61ec5499ce62a8414814438166fa2a3c6e'  # вставь свой токен от Cryptopanic
BASE_URL = 'https://cryptopanic.com/api/v1/posts/'

# Целевые монеты
target_coins = ['bitcoin', 'ethereum', 'tether', 'ripple', 'binancecoin']

# Словарь для хранения результатов: {coin -> {date -> новость}}
filtered_news = {coin: {} for coin in target_coins}

# Пороговая дата (30 дней назад)
date_limit = datetime.utcnow() - timedelta(days=30)

url = BASE_URL
params = {
    'auth_token': API_KEY,
    'kind': 'news',
    'public': 'true'
}

print("🔍 Собираем данные...")

while url:
    response = requests.get(url, params=params)
    data = response.json()

    for post in data.get('results', []):
        published = datetime.strptime(post['published_at'], "%Y-%m-%dT%H:%M:%SZ")
        if published < date_limit:
            url = None  # прекратить сбор, как только новости станут старше 30 дней
            break

        date_str = published.date().isoformat()
        tags = [currency['slug'].lower() for currency in post.get('currencies', [])]

        # Проверка по каждой монете
        for coin in target_coins:
            if coin in tags and date_str not in filtered_news[coin]:
                filtered_news[coin][date_str] = {
                    'date': date_str,
                    'coin': coin,
                    'title': post['title'],
                    'url': post['url']
                }

    # Переход к следующей странице
    url = data.get('next')

print("✅ Сбор завершён!")

# Преобразуем в DataFrame
news_rows = []
for coin_data in filtered_news.values():
    news_rows.extend(coin_data.values())

df = pd.DataFrame(news_rows)
df = df.sort_values(['coin', 'date'])
df.to_csv("crypto_news_last_30_days.csv", index=False)

print("📁 Сохранено в crypto_news_last_30_days.csv")
```

    🔍 Собираем данные...
    ✅ Сбор завершён!
    📁 Сохранено в crypto_news_last_30_days.csv



```python
import requests
import pandas as pd
from datetime import datetime, timedelta

API_KEY = "YOUR_API_KEY"  # 🔁 Вставь свой ключ сюда
BASE_URL = "https://newsapi.org/v2/everything"

coin = "bitcoin"
today = datetime.utcnow().date()
date_list = [today - timedelta(days=i) for i in range(30)]

news_data = []

print("📰 Сбор заголовков новостей по Bitcoin за последние 30 дней:\n")

for date in date_list:
    from_date = date.isoformat()
    to_date = (date + timedelta(days=1)).isoformat()

    params = {
        'q': coin,
        'from': from_date,
        'to': to_date,
        'language': 'en',
        'sortBy': 'relevancy',
        'apiKey': '6ece552a55d7445b81152343964547e2',
        'pageSize': 1
    }

    try:
        response = requests.get(BASE_URL, params=params)
        data = response.json()

        if data.get('status') != 'ok':
            print(f"⚠️ {date}: ошибка запроса — {data.get('message')}")
            continue

        articles = data.get('articles', [])
        if articles:
            article = articles[0]
            news_data.append({
                'date': from_date,
                'coin': coin,
                'title': article['title'],
                'url': article['url']
            })
            print(f"✅ {date}: {article['title'][:60]}...")
        else:
            print(f"❌ {date}: новостей не найдено.")

    except Exception as e:
        print(f"⚠️ {date}: ошибка — {e}")

# Сохраняем результат
df = pd.DataFrame(news_data)
df = df.sort_values(by='date')
df.to_csv("bitcoin_news_last_30_days.csv", index=False)

print("\n✅ Готово! Файл сохранён как bitcoin_news_last_30_days.csv")
```

    📰 Сбор заголовков новостей по Bitcoin за последние 30 дней:
    
    ❌ 2025-04-09: новостей не найдено.
    ✅ 2025-04-08: Bitcoin Plunges Below $75,000 As Tariff Anxiety Creates Fres...
    ✅ 2025-04-07: Trump’s DOJ will no longer prosecute cryptocurrency fraud...
    ✅ 2025-04-06: Stock market today: Dow sinks 350 points, S&P 500 falls for ...
    ✅ 2025-04-05: BitBonds: The $2 Trillion Idea That Could Slash The National...
    ✅ 2025-04-04: Watch Live as SpaceX’s Private Fram2 Crew Returns From Polar...
    ✅ 2025-04-03: Watch Live as SpaceX’s Private Fram2 Crew Returns From Polar...
    ✅ 2025-04-02: Motion Sickness Hits Hard as Space Tourists Describe Polar O...
    ✅ 2025-04-01: Larry Fink Says Bitcoin Could Replace the Dollar as the Worl...
    ✅ 2025-03-31: Watch Live as SpaceX Launches Private Crew to Elusive Polar ...
    ✅ 2025-03-30: Watch Live as SpaceX Launches Private Crew to Elusive Polar ...
    ✅ 2025-03-29: Tech Now...
    ✅ 2025-03-28: Meet the astronauts of SpaceX's Fram2 mission, the 1st to fl...
    ✅ 2025-03-27: Like DOGE, XRP Going Vertical Is a Good Indicator of Market ...
    ✅ 2025-03-26: GameStop to Borrow $1.3 Billion to Fund Bitcoin Buying Spree...
    ✅ 2025-03-25: Bitcoin in the bush - the crypto mine in remote Zambia...
    ✅ 2025-03-24: Bitcoin in the bush - the crypto mine in remote Zambia...
    ✅ 2025-03-23: The Quantum Apocalypse Is Coming. Be Very Afraid...
    ✅ 2025-03-22: The crypto-lending industry, nearly wiped out in the last be...
    ✅ 2025-03-21: The crypto-lending industry, nearly wiped out in the last be...
    ✅ 2025-03-20: Sources: Coinbase is in advanced talks to buy Deribit, the l...
    ✅ 2025-03-19: Cathie Wood says people are going to learn the hard way that...
    ✅ 2025-03-18: Cathie Wood says people are going to learn the hard way that...
    ✅ 2025-03-17: Gemini Credit Card...
    ✅ 2025-03-16: Bitcoin, BNB, XRP, and more cryptocurrencies to watch this w...
    ✅ 2025-03-15: Tesla stock bleeds, recession fears — and Bitcoin $200,000: ...
    ✅ 2025-03-14: Tesla stock bleeds, recession fears — and Bitcoin $200,000: ...
    ✅ 2025-03-13: Think You Know Bitcoin? Here's 1 Little-Known Fact You Can't...
    ✅ 2025-03-12: Think You Know Bitcoin? Here's 1 Little-Known Fact You Can't...
    ✅ 2025-03-11: What Would Make Bitcoin Fall to Zero? 4 Things That Give It ...
    
    ✅ Готово! Файл сохранён как bitcoin_news_last_30_days.csv



```python
import requests
import pandas as pd
from datetime import datetime

def get_historical_prices(coin_id, vs_currency='usd', from_date='2025-03-11', to_date='2025-04-08'):
    from_timestamp = int(datetime.strptime(from_date, "%Y-%m-%d").timestamp())
    to_timestamp = int(datetime.strptime(to_date, "%Y-%m-%d").timestamp())
    url = f"https://api.coingecko.com/api/v3/coins/{coin_id}/market_chart/range"
    params = {
        'vs_currency': vs_currency,
        'from': from_timestamp,
        'to': to_timestamp
    }
    response = requests.get(url, params=params)
    data = response.json()
    prices = data['prices']
    df = pd.DataFrame(prices, columns=["timestamp", "price"])
    df['date'] = pd.to_datetime(df['timestamp'], unit='ms').dt.date
    df = df.groupby('date').first().reset_index()
    return df

# Пример для биткоина:
btc_prices = get_historical_prices('bitcoin')
print(btc_prices.head())
```

             date      timestamp         price
    0  2025-03-10  1741633485249  77494.599945
    1  2025-03-11  1741651486566  78537.058058
    2  2025-03-12  1741738184594  82753.777595
    3  2025-03-13  1741824091524  83678.465524
    4  2025-03-14  1741911282006  80857.267017



```python
import requests
import pandas as pd
from datetime import datetime

def get_bitcoin_price_data(start='2025-03-11', end='2025-04-08'):
    from_ts = int(datetime.strptime(start, "%Y-%m-%d").timestamp())
    to_ts = int(datetime.strptime(end, "%Y-%m-%d").timestamp())
    url = "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart/range"
    params = {
        'vs_currency': 'usd',
        'from': from_ts,
        'to': to_ts
    }
    response = requests.get(url, params=params)
    data = response.json()
    prices = data.get('prices', [])
    df = pd.DataFrame(prices, columns=['timestamp', 'price'])
    df['date'] = pd.to_datetime(df['timestamp'], unit='ms').dt.date
    df = df.groupby('date').first().reset_index()
    df['coin_id'] = 'bitcoin'
    df['symbol'] = 'BTC'
    return df[['date', 'coin_id', 'symbol', 'price']]

# Получаем данные по Bitcoin
print("📊 Собираем данные по Bitcoin...")
bitcoin_prices = get_bitcoin_price_data()

# Сохраняем в CSV
bitcoin_prices.to_csv("bitcoin_prices_2023_2024.csv", index=False)
print("✅ Готово! Данные сохранены в bitcoin_prices_2023_2024.csv")
```

    📊 Собираем данные по Bitcoin...
    ✅ Готово! Данные сохранены в bitcoin_prices_2023_2024.csv



```python
import pandas as pd

# Загрузка CSV-файлов
news_df = pd.read_csv('bitcoin_news_last_30_days.csv')        # ← Замени на своё имя файла
price_df = pd.read_csv('bitcoin_prices_2023_2024.csv')      # ← Замени на своё имя файла

# Приведение колонок к одному стилю
news_df['date'] = pd.to_datetime(news_df['date']).dt.date
price_df['date'] = pd.to_datetime(price_df['date']).dt.date

# Объединение по дате и coin (coin_id = coin)
merged_df = pd.merge(news_df, price_df, how='inner', left_on=['date', 'coin'], right_on=['date', 'coin_id'])

# Удалим лишнее и упорядочим
merged_df = merged_df[['date', 'coin', 'symbol', 'price', 'title', 'url']]

# Сохраняем в CSV
merged_df.to_csv('bitcoin_news_with_price.csv', index=False)

print("✅ Готово! Файл сохранён как bitcoin_news_with_price.csv")
```

    ✅ Готово! Файл сохранён как bitcoin_news_with_price.csv



```python
# Загружаем объединённый файл
df = pd.read_csv('bitcoin_news_with_price.csv')

# Удаляем столбец 'url'
df = df.drop(columns=['url'])

# Сохраняем новый CSV
df.to_csv('bitcoin_news_with_price_no_url.csv', index=False)

print("✅ Готово! Файл сохранён как bitcoin_news_with_price_no_url.csv (без URL)")
```

    ✅ Готово! Файл сохранён как bitcoin_news_with_price_no_url.csv (без URL)



```python
import pandas as pd
from nltk.sentiment.vader import SentimentIntensityAnalyzer
import nltk

# Скачиваем словарь для анализа тональности
nltk.download('vader_lexicon')

# Инициализация анализатора
sid = SentimentIntensityAnalyzer()

# Загружаем твой CSV-файл
df = pd.read_csv("bitcoin_news_with_price_no_url.csv")  # Замени на своё имя файла

# Функция для оценки тональности
def get_sentiment(text):
    score = sid.polarity_scores(str(text))['compound']
    if score >= 0.05:
        return 'positive'
    elif score <= -0.05:
        return 'negative'
    else:
        return 'neutral'

# Применяем анализ к заголовкам
df['sentiment'] = df['title'].apply(get_sentiment)

# Сохраняем результат
df.to_csv("crypto_with_sentiment.csv", index=False)
print("✅ Готово! Результат сохранён в crypto_with_sentiment.csv")
```

    [nltk_data] Downloading package vader_lexicon to
    [nltk_data]     /Users/kabdolaru/nltk_data...


    ✅ Готово! Результат сохранён в crypto_with_sentiment.csv



```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Загрузим файл
df = pd.read_csv("crypto_with_sentiment.csv")

# Преобразуем колонку 'date' в datetime формат
df['date'] = pd.to_datetime(df['date'])

# Быстрый просмотр
df.head(28)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>coin</th>
      <th>symbol</th>
      <th>price</th>
      <th>title</th>
      <th>sentiment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2025-03-11</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>78537.058058</td>
      <td>What Would Make Bitcoin Fall to Zero? 4 Things...</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2025-03-12</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>82753.777595</td>
      <td>Think You Know Bitcoin? Here's 1 Little-Known ...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2025-03-13</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83678.465524</td>
      <td>Think You Know Bitcoin? Here's 1 Little-Known ...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2025-03-14</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>80857.267017</td>
      <td>Tesla stock bleeds, recession fears — and Bitc...</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2025-03-15</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83955.003789</td>
      <td>Tesla stock bleeds, recession fears — and Bitc...</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2025-03-16</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>84354.471914</td>
      <td>Bitcoin, BNB, XRP, and more cryptocurrencies t...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2025-03-17</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>82617.733124</td>
      <td>Gemini Credit Card</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2025-03-18</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83920.307213</td>
      <td>Cathie Wood says people are going to learn the...</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2025-03-19</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>82780.030487</td>
      <td>Cathie Wood says people are going to learn the...</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2025-03-20</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>86815.441095</td>
      <td>Sources: Coinbase is in advanced talks to buy ...</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2025-03-21</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>84326.234608</td>
      <td>The crypto-lending industry, nearly wiped out ...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2025-03-22</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>84009.532918</td>
      <td>The crypto-lending industry, nearly wiped out ...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2025-03-23</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83823.012451</td>
      <td>The Quantum Apocalypse Is Coming. Be Very Afraid</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2025-03-24</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>86114.333705</td>
      <td>Bitcoin in the bush - the crypto mine in remot...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2025-03-25</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>87404.876491</td>
      <td>Bitcoin in the bush - the crypto mine in remot...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2025-03-26</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>87424.068658</td>
      <td>GameStop to Borrow $1.3 Billion to Fund Bitcoi...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2025-03-27</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>86913.685786</td>
      <td>Like DOGE, XRP Going Vertical Is a Good Indica...</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2025-03-28</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>87212.974683</td>
      <td>Meet the astronauts of SpaceX's Fram2 mission,...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2025-03-29</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>84359.469155</td>
      <td>Tech Now</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2025-03-30</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>82603.846087</td>
      <td>Watch Live as SpaceX Launches Private Crew to ...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2025-03-31</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>82355.188245</td>
      <td>Watch Live as SpaceX Launches Private Crew to ...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2025-04-01</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>82553.236236</td>
      <td>Larry Fink Says Bitcoin Could Replace the Doll...</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2025-04-02</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>85151.383599</td>
      <td>Motion Sickness Hits Hard as Space Tourists De...</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2025-04-03</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>82526.422153</td>
      <td>Watch Live as SpaceX’s Private Fram2 Crew Retu...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2025-04-04</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83208.527965</td>
      <td>Watch Live as SpaceX’s Private Fram2 Crew Retu...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2025-04-05</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83733.529312</td>
      <td>BitBonds: The $2 Trillion Idea That Could Slas...</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2025-04-06</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83514.338244</td>
      <td>Stock market today: Dow sinks 350 points, S&amp;P ...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2025-04-07</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>78366.832124</td>
      <td>Trump’s DOJ will no longer prosecute cryptocur...</td>
      <td>negative</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.isnull().sum()
```




    date         0
    coin         0
    symbol       0
    price        0
    title        0
    sentiment    0
    dtype: int64




```python
df['price'].describe()
```




    count       28.000000
    mean     83781.108866
    std       2287.433904
    min      78366.832124
    25%      82614.261365
    50%      83778.270882
    75%      84557.447766
    max      87424.068658
    Name: price, dtype: float64




```python
plt.figure(figsize=(12, 6))
sns.lineplot(data=df, x='date', y='price', marker='o')
plt.title('📈 Bitcoin Price by day')
plt.xlabel('Date')
plt.ylabel('Price USD')
plt.grid(True)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

    /var/folders/_3/pr9tldy13qq0z72_jf81hfm80000gn/T/ipykernel_1336/3550104963.py:8: UserWarning: Glyph 128200 (\N{CHART WITH UPWARDS TREND}) missing from current font.
      plt.tight_layout()
    /Users/kabdolaru/anaconda3/lib/python3.11/site-packages/IPython/core/pylabtools.py:152: UserWarning: Glyph 128200 (\N{CHART WITH UPWARDS TREND}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)



    
![png](README_files/README_24_1.png)
    



```python
# Количество новостей по тональности
sentiment_counts = df['sentiment'].value_counts()

plt.figure(figsize=(6, 4))
sns.barplot(x=sentiment_counts.index, y=sentiment_counts.values, palette='pastel')
plt.title('📊 Sentiment Distribution of news')
plt.xlabel('Sentiment')
plt.ylabel('Quantity of news')
plt.grid(True, axis='y')
plt.show()
```

    /Users/kabdolaru/anaconda3/lib/python3.11/site-packages/IPython/core/pylabtools.py:152: UserWarning: Glyph 128202 (\N{BAR CHART}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)



    
![png](README_files/README_25_1.png)
    



```python
# Группировка по тональности
avg_price_by_sentiment = df.groupby('sentiment')['price'].mean().reset_index()

plt.figure(figsize=(6, 4))
sns.barplot(data=avg_price_by_sentiment, x='sentiment', y='price', palette='Set2')
plt.title('💰 The average price of Bitcoin according to the tone of the news')
plt.xlabel('Sentiment')
plt.ylabel('Average Price (USD)')
plt.grid(True, axis='y')
plt.show()
```

    /Users/kabdolaru/anaconda3/lib/python3.11/site-packages/IPython/core/pylabtools.py:152: UserWarning: Glyph 128176 (\N{MONEY BAG}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)



    
![png](README_files/README_26_1.png)
    



```python
plt.figure(figsize=(12, 6))
sns.scatterplot(data=df, x='date', y='price', hue='sentiment', palette='coolwarm')
plt.title('🔍 Bitcoin price and News Sentiment')
plt.xlabel('Date')
plt.ylabel('Price (USD)')
plt.xticks(rotation=45)
plt.grid(True)
plt.tight_layout()
plt.show()
```

    /var/folders/_3/pr9tldy13qq0z72_jf81hfm80000gn/T/ipykernel_1336/2034730470.py:8: UserWarning: Glyph 128269 (\N{LEFT-POINTING MAGNIFYING GLASS}) missing from current font.
      plt.tight_layout()
    /Users/kabdolaru/anaconda3/lib/python3.11/site-packages/IPython/core/pylabtools.py:152: UserWarning: Glyph 128269 (\N{LEFT-POINTING MAGNIFYING GLASS}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)



    
![png](README_files/README_27_1.png)
    



```python

# Загружаем словарь для анализа тональности
nltk.download('vader_lexicon')

# Инициализируем анализатор
sid = SentimentIntensityAnalyzer()

# Загружаем CSV-файл
df = pd.read_csv("crypto_with_sentiment.csv")  # Заменить на имя твоего файла

# Функция для получения числового значения тональности
def get_sentiment_score(text):
    return sid.polarity_scores(text)['compound']

# Применяем функцию к заголовкам
df['sentiment_score'] = df['title'].apply(get_sentiment_score)

# Сохраняем результат
df.to_csv("crypto_with_sentiment_score.csv", index=False)
print("✅ Готово! Файл сохранён как crypto_with_sentiment_score.csv")
```

    ✅ Готово! Файл сохранён как crypto_with_sentiment_score.csv


    [nltk_data] Downloading package vader_lexicon to
    [nltk_data]     /Users/kabdolaru/nltk_data...
    [nltk_data]   Package vader_lexicon is already up-to-date!



```python
plt.figure(figsize=(12, 6))
sns.scatterplot(data=df, x='sentiment_score', y='price', palette='coolwarm')
plt.title('🔍 Bitcoin price and News Sentiment')
plt.xlabel('Date')
plt.ylabel('Price (USD)')
plt.xticks(rotation=45)
plt.grid(True)
plt.tight_layout()
plt.show()
```

    /var/folders/_3/pr9tldy13qq0z72_jf81hfm80000gn/T/ipykernel_1336/2184576013.py:2: UserWarning: Ignoring `palette` because no `hue` variable has been assigned.
      sns.scatterplot(data=df, x='sentiment_score', y='price', palette='coolwarm')
    /var/folders/_3/pr9tldy13qq0z72_jf81hfm80000gn/T/ipykernel_1336/2184576013.py:8: UserWarning: Glyph 128269 (\N{LEFT-POINTING MAGNIFYING GLASS}) missing from current font.
      plt.tight_layout()
    /Users/kabdolaru/anaconda3/lib/python3.11/site-packages/IPython/core/pylabtools.py:152: UserWarning: Glyph 128269 (\N{LEFT-POINTING MAGNIFYING GLASS}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)



    
![png](README_files/README_29_1.png)
    


# Here we see that correation is very weak


```python
# Кодируем тональность: positive=1, neutral=0, negative=-1
sentiment_map = {'positive': 1, 'neutral': 0, 'negative': -1}
df['sentiment_score'] = df['sentiment'].map(sentiment_map)

# Корреляция
correlation = df['price'].corr(df['sentiment_score'])
print(f"📌 Correlation between prices and sentiment: {correlation:.3f}")
```

    📌 Correlation between prices and sentiment: 0.279



```python
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
from sklearn.preprocessing import StandardScaler

# Загрузка данных
df = pd.read_csv("crypto_with_sentiment_score.csv")
df['date'] = pd.to_datetime(df['date'])

# Добавим временные признаки
df['day'] = df['date'].dt.day
df['month'] = df['date'].dt.month

# Добавим лаг: цену предыдущего дня
df['price_prev'] = df['price'].shift(1)

# Удалим первую строку с NaN
df = df.dropna()

# Входные данные
X = df[['sentiment_score', 'day', 'month', 'price_prev']]
y = df['price']

# Масштабируем
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Делим на обучающую и тестовую выборки
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, test_size=0.2, shuffle=False)

# Обучаем модель
model = LinearRegression()
model.fit(X_train, y_train)

# Предсказание и оценка
y_pred = model.predict(X_test)
mse = mean_squared_error(y_test, y_pred)

print(f"✅ MSE модели: {mse:.2f}")
```

    ✅ MSE модели: 3498745.53



```python
# sentiment в числовой вид
df['sentiment_num'] = df['sentiment'].map({'positive': 1, 'neutral': 0, 'negative': -1})

# процентное изменение цены
df['price_change_pct'] = df['price'].pct_change() * 100

# скользящее среднее
df['rolling_mean_3'] = df['price'].rolling(window=3).mean()

# выходной?
df['is_weekend'] = df['date'].dt.weekday >= 5

# обновим данные
df.dropna(inplace=True)

# новые признаки
X = df[['sentiment_score', 'sentiment_num', 'day', 'month', 'price_prev',
        'price_change_pct', 'rolling_mean_3', 'is_weekend']]
y = df['price']

# Масштабируем
X_scaled = scaler.fit_transform(X)

# Делим и обучаем как раньше
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, test_size=0.2, shuffle=False)
model = LinearRegression()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

# Метрики
print("✅ Multiple Regression с расширенными признаками")
print(f"MSE: {mean_squared_error(y_test, y_pred):.2f}")
print(f"R²: {r2_score(y_test, y_pred):.4f}")
```

    ✅ Multiple Regression с расширенными признаками
    MSE: 1073.31



    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[22], line 33
         31 print("✅ Multiple Regression с расширенными признаками")
         32 print(f"MSE: {mean_squared_error(y_test, y_pred):.2f}")
    ---> 33 print(f"R²: {r2_score(y_test, y_pred):.4f}")


    NameError: name 'r2_score' is not defined



```python
df.head()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>coin</th>
      <th>symbol</th>
      <th>price</th>
      <th>title</th>
      <th>sentiment</th>
      <th>sentiment_score</th>
      <th>day</th>
      <th>month</th>
      <th>price_prev</th>
      <th>sentiment_num</th>
      <th>price_change_pct</th>
      <th>rolling_mean_3</th>
      <th>is_weekend</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>2025-03-14</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>80857.267017</td>
      <td>Tesla stock bleeds, recession fears — and Bitc...</td>
      <td>negative</td>
      <td>-0.6808</td>
      <td>14</td>
      <td>3</td>
      <td>83678.465524</td>
      <td>-1</td>
      <td>-3.371475</td>
      <td>82429.836712</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2025-03-15</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83955.003789</td>
      <td>Tesla stock bleeds, recession fears — and Bitc...</td>
      <td>negative</td>
      <td>-0.6808</td>
      <td>15</td>
      <td>3</td>
      <td>80857.267017</td>
      <td>-1</td>
      <td>3.831117</td>
      <td>82830.245443</td>
      <td>True</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2025-03-16</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>84354.471914</td>
      <td>Bitcoin, BNB, XRP, and more cryptocurrencies t...</td>
      <td>neutral</td>
      <td>0.0000</td>
      <td>16</td>
      <td>3</td>
      <td>83955.003789</td>
      <td>0</td>
      <td>0.475812</td>
      <td>83055.580907</td>
      <td>True</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2025-03-17</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>82617.733124</td>
      <td>Gemini Credit Card</td>
      <td>positive</td>
      <td>0.3818</td>
      <td>17</td>
      <td>3</td>
      <td>84354.471914</td>
      <td>1</td>
      <td>-2.058858</td>
      <td>83642.402942</td>
      <td>False</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2025-03-18</td>
      <td>bitcoin</td>
      <td>BTC</td>
      <td>83920.307213</td>
      <td>Cathie Wood says people are going to learn the...</td>
      <td>negative</td>
      <td>-0.6705</td>
      <td>18</td>
      <td>3</td>
      <td>82617.733124</td>
      <td>-1</td>
      <td>1.576628</td>
      <td>83630.837417</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>




```python
predictPrice = model.predict([[0.7, 1, 15, 5, 80000, -2, 82000, False]])
print(predictPrice)
```

    [1.40765274e+08]



```python
print(data.columns)
```

    Index(['coin', 'symbol', 'price', 'title', 'sentiment', 'sentiment_score'], dtype='object')



```python
# Example: Adding a date column if you have date information
import pandas as pd

# If you have timestamps or can generate dates from the data's order
data['date'] = pd.date_range(start='2025-03-11', periods=len(data), freq='D')

# Convert 'date' column to datetime
data['date'] = pd.to_datetime(data['date'])

# Set the 'date' column as the index
data.set_index('date', inplace=True)
```


```python
# Check the length of the prices_scaled array
print(f"Length of prices_scaled: {len(prices_scaled)}")

# Create a dataset with X_train and y_train
X, y = [], []
window_size = 20  # Number of past time steps to predict the next one
for i in range(window_size, len(prices_scaled)):
    X.append(prices_scaled[i-window_size:i, 0])  # Use the previous 'window_size' prices
    y.append(prices_scaled[i, 0])  # Predict the next price

# Check the shape of X and y
print(f"Shape of X: {len(X)}")
print(f"Shape of y: {len(y)}")

X, y = np.array(X), np.array(y)

# Check again the shape of X and y
print(f"Shape of X (after conversion): {X.shape}")
print(f"Shape of y (after conversion): {y.shape}")

# Reshape X for LSTM input: [samples, time steps, features]
X = np.reshape(X, (X.shape[0], X.shape[1], 1))
```

    Length of prices_scaled: 28
    Shape of X: 8
    Shape of y: 8
    Shape of X (after conversion): (8, 20)
    Shape of y (after conversion): (8,)



```python

```
