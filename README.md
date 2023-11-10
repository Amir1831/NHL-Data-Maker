# NHL News Dataset

This project fetches data from the NHL website using the corresponding API to create a dataset of news related to teams that have played a game in the last 120 days. This dataset can be used for natural language processing (NLP) tasks.

## Fetching Data

To fetch the data, the code uses the `fetch_Data` function, which makes a GET request to the API endpoint. The API key is passed as a parameter in the headers. The response is then converted to a JSON object.

```python
def fetch_Data(url , key="Use-your-key"):
    headers = {"Ocp-Apim-Subscription-Key": key, 'Content-Type': 'application/json'}
    res = requests.get(url, headers=headers)
    response = json.loads(res.text)
    return response

```

## Fetching News by Date

The code fetches news by a specific date using the `NewsByDate` API endpoint. The data is then converted to a DataFrame.

```python
url = "https://api.sportsdata.io/v3/nhl/scores/json/NewsByDate/2023-11-01"
data_2 = fetch_Data(url)
df_2 = pd.DataFrame(data_2)
```

## Creating the Dataset

The code creates a dataset by fetching game data and news data for the last 120 days. It iterates through a range of 1 to 120 (representing the number of days in the past) and fetches game data and news data for each day. If there is no game or news data, it is skipped. The game data and news data are appended to separate dataframes, `df_Game` and `df_news`, respectively. The `fetch_Data` function is used to fetch the data from the API endpoints.

```python
df_Game = []
df_news = []
days = 120
for i in range(1,days):
    date_Game = (now - timedelta(days=i)).strftime("%Y-%m-%d")
    date_News = (now - timedelta(days=i+1)).strftime("%Y-%m-%d")
    url = f"https://api.sportsdata.io/v3/nhl/scores/json/GamesByDate/{date_Game}"
    data_Game = fetch_Data(url)
    url = f"https://api.sportsdata.io/v3/nhl/scores/json/NewsByDate/{date_News}"
    data_News = fetch_Data(url)
    if len(data_Game) and len(data_News): 
        df_Game.append(pd.DataFrame(data_Game)[["GameID","DateTime","IsClosed","AwayTeamID","HomeTeamID","AwayTeamScore","HomeTeamScore"]])
        for dic in data_News:
            dic["DateTime"] = datetime.strptime(date_News, '%Y-%m-%d')
        df_news.append(pd.DataFrame(data_News)[["TeamID" , "Content" , "DateTime"]])

df_Game = pd.concat(df_Game,ignore_index=True)
df_news = pd.concat(df_news,ignore_index=True)

df_Game["DateTime"] = df_Game["DateTime"].apply(lambda x : datetime.strptime(x.split("T")[0], '%Y-%m-%d'))
```

## Merging Data

The game data (`df_Game`) and news data (`df_news`) are joined based on the away team ID and home team ID using the `merge` function. Two separate dataframes are created for away team news (`final_Away_df`) and home team news (`final_Home_df`). The news articles are filtered based on their date, where only articles between 0 and 7 days before the game date are included.

```python
Away_df = pd.merge(df_Game , df_news ,how="inner" , left_on="AwayTeamID" , right_on="TeamID")
Home_df = pd.merge(df_Game , df_news ,how="inner" , left_on="HomeTeamID" , right_on="TeamID")
zero_days = pd.Timedelta(days=0)
seven_days = pd.Timedelta(days=7)
mask_Away = (zero_days < (Away_df["DateTime_x"] - Away_df["DateTime_y"])) & ((Away_df["DateTime_x"] - Away_df["DateTime_y"]) < seven_days)
mask_home = (zero_days < (Home_df["DateTime_x"] - Home_df["DateTime_y"])) & ((Home_df["DateTime_x"] - Home_df["DateTime_y"]) < seven_days)
final_Away_df = Away_df[mask_Away].sort_values(by="DateTime_x",ascending=False).reset_index()
final_Home_df = Home_df[mask_home].sort_values(by="DateTime_x",ascending=False).reset_index()
```

## Save Data

The final datasets (`final_Away_df` and `final_Home_df`) are saved as CSV files.

```python
final_Away_df.to_csv("./Away_data.csv")
final_Home_df.to_csv("./Home_data.csv")
```

Feel free to modify and use this code for creating your own NHL news dataset for NLP tasks!
