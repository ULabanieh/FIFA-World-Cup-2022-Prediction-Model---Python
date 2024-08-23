# ⚽Project Overview
---
This is a prediction model project created with Python and its goal is to create a prediction of the results of the World Cup 2022 based on historical data from previous world cups with each team's performance. With Web Scraping, I extracted the historical data of the previous world cups from Wikipedia along with the fixture list of the Qatar World Cup 2022. Then, using the historical data, I created a calculation of each team's strength based on previous results and goals scored. Finally, I carried out a simulation of the 2022 World Cup matches and ended up with a predicted winner: Brazil!

This project is composed of several steps:
- Data Collection
- Data Cleaning and Preparation
- Building the Model

If you want to view the project in full detail, you can download the project files and explore them at will. Here I will outline the most important steps and actions within the project.
# Data Collection
---
0:00-48:15
In this step, I collected two different sets of data using web scraping. First, I extracted the group tables of the FIFA 2022 World Cup and then extracted the list of fixtures of the 2022 World Cup and the results historical data of previous world cups. The source of the data is Wikipedia. I used Pandas, BeautifulSoup for completing this part.

## Group Tables Extraction
---
### Reading in website

```python
all_tables = pd.read_html('https://web.archive.org/web/202211115040351/https://en.wikipedia.org/wiki/2022_FIFA_World_Cup')
```

### Example Table

```python
all_tables [26]
```

![[1446-02-19 12_16_40-Extract WC22 Group Tables.ipynb - World Cup 2022 Prediction Model - Visual Studi.png]]

### Storing group tables in dataframe and assigning a letter to each group

```python
dict_table = {}

for letter, i in zip(alphabet, range(12, 68, 7)):

    df = all_tables [i]

    df.rename(columns = {df.columns[1]: 'Team'}, inplace = True)

    df.pop('Qualification')

    dict_table[f'Group {letter}'] = df

dict_table.keys()
```

![[1446-02-19 12_20_02-Extract WC22 Group Tables.ipynb - World Cup 2022 Prediction Model - Visual Studi.png]]

#### Example Stored Table

```python
dict_table['Group E']
```

![[1446-02-19 12_21_50-Extract WC22 Group Tables.ipynb - World Cup 2022 Prediction Model - Visual Studi.png]]

### Export group tables in a flat file using Pickle

```python
with open('wc22_groups.csv', 'wb') as output:

    pickle.dump(dict_table, output)
```

```python
groups_df_file = open("wc22_groups.csv", "rb")

groups = pickle.load(groups_df_file)

groups_df_file.close()

print(groups)
```

![[1446-02-19 12_24_56-Extract WC22 Group Tables.ipynb - World Cup 2022 Prediction Model - Visual Studi.png]]

## Fixture List and Historical Data Extraction
---
Timestamp: 17:22

Using a Python and the BeautifulSoup library, I scraped the list of matches of the Qatar 2022 FIFA World Cup and the historical data of the previous world cups. Here is how I did it:

### Read in website

```python
web = 'https://en.wikipedia.org/wiki/2014_FIFA_World_Cup'

response = requests.get(web)

content = response.text

soup = BeautifulSoup(content, 'lxml')

print(response.text)
```

![[1446-02-19 15_43_13-Extract WC22 Fixtures + Historical WC Match Data.ipynb - World Cup 2022 Predicti.png]]

### Extract matches and storing results (2014 WC)

```python
matches = soup.find_all('div', class_ = 'footballbox')
```

```python
for match in matches:

    print(match.find('th', class_ = 'fhome').get_text())

    print(match.find('th', class_ = 'fscore').get_text())

    print(match.find('th', class_ = 'faway').get_text())
```

![[1446-02-19 15_44_17-Extract WC22 Fixtures + Historical WC Match Data.ipynb - World Cup 2022 Predicti.png]]

```python
home = []

score = []

away = []
```

```python
for match in matches:

    home.append(match.find('th', class_ = 'fhome').get_text())

    score.append(match.find('th', class_ = 'fscore').get_text())

    away.append(match.find('th', class_ = 'faway').get_text())
```

```python
dict_football = {'home': home, 'score': score, 'away': away}

df_football = pd.DataFrame(dict_football)

df_football['year'] = '2014'

print(df_football)
```

![[1446-02-19 15_45_16-Extract WC22 Fixtures + Historical WC Match Data.ipynb - World Cup 2022 Predicti.png]]

### Function to extrat rest of World Cups' historical data

```python
def get_matches(year):

    web = f'https://en.wikipedia.org/wiki/{year}_FIFA_World_Cup'

    response = requests.get(web)

    content = response.text

    soup = BeautifulSoup(content, 'lxml')

    matches = soup.find_all('div', class_ = 'footballbox')

    home = []

    score = []

    away = []

    for match in matches:

        home.append(match.find('th', class_ = 'fhome').get_text())

        score.append(match.find('th', class_ = 'fscore').get_text())

        away.append(match.find('th', class_ = 'faway').get_text())

    dict_football = {'home': home, 'score': score, 'away': away}

    df_football = pd.DataFrame(dict_football)

    df_football['year'] = year

    return df_football
```


Function test:
```python
print(get_matches('2018'))
```

![[1446-02-19 15_46_33-Extract WC22 Fixtures + Historical WC Match Data.ipynb - World Cup 2022 Predicti.png]]

### Extract World Cup 2022 Fixture List and store in CSV
```python
def get_fixtures(year):

    web = 'https://web.archive.org/web/20221111192235/https://en.wikipedia.org/wiki/2022_FIFA_World_Cup'

    response = requests.get(web)

    content = response.text

    soup = BeautifulSoup(content, 'lxml')

    matches = soup.find_all('div', class_ = 'footballbox')

    home = []

    score = []

    away = []

    for match in matches:

        home.append(match.find('th', class_ = 'fhome').get_text())

        score.append(match.find('th', class_ = 'fscore').get_text())

        away.append(match.find('th', class_ = 'faway').get_text())

    dict_football = {'home': home, 'score': score, 'away': away}

    df_football = pd.DataFrame(dict_football)

    df_football['year'] = '2022'

    return df_football
```

```python
df_fixture = get_fixtures(2022)

df_fixture.to_csv('fifa_worldcup_fixture22.csv', index = False)
```

### Missing Data
There were some world cup historical data missing that I could not scrape correctly with the first try due to a difference in formatting of the Wikipedia page. To get the missing data I imported the Selenium library and used the following code:

```python
path = r'C:\Program Files\webdrivers\chromedriver-win64\chromedriver.exe'

service = Service(executable_path=path)

driver = webdriver.Chrome(service=service)
```

```python
def get_misssing_data(year):

    web = f'https://en.wikipedia.org/wiki/{year}_FIFA_World_Cup'

  

    driver.get(web)

    matches = driver.find_elements(by='xpath', value='//td[@align="right"]/.. | //td[@style="text-align:right;"]/..')

    # matches = driver.find_elements(by='xpath', value='//tr[@style="font-size:90%"]')

  

    home = []

    score = []

    away = []

  

    for match in matches:

        home.append(match.find_element(by='xpath', value='./td[1]').text)

        score.append(match.find_element(by='xpath', value='./td[2]').text)

        away.append(match.find_element(by='xpath', value='./td[3]').text)

  

    dict_football = {'home': home, 'score': score, 'away': away}

    df_football = pd.DataFrame(dict_football)

    df_football['year'] = year

    time.sleep(2)

    return df_football
```

```python
years = [1930, 1934, 1938, 1950, 1954, 1958, 1962, 1966, 1970, 1974,

         1978, 1982, 1986, 1990, 1994, 1998, 2002, 2006, 2010, 2014,

         2018]
```

```python
fifa = [get_misssing_data(year) for year in years]

driver.quit()

df_fifa = pd.concat(fifa, ignore_index=True)

df_fifa.to_csv("fifa_worldcup_missing_data.csv", index=False)
```
# Data Cleaning and Preparation
---
48:15-1:18:43

## Cutting extra spaces from df_fixture
```python
df_fixture['home'] = df_fixture['home'].str.strip()

df_fixture['away'] = df_fixture['away'].str.strip()

df_fixture
```

![[1446-02-19 16_18_24-Data Cleaning & Transformation.ipynb - World Cup 2022 Prediction Model - Visual .png]]


## Clean df_missing_data and merging with df_historical_data

```python
df_missing_data[df_missing_data['home'].isnull()]
```

```python
df_missing_data.dropna(inplace=True)
```

```python
df_historical_data = pd.concat([df_historical_data, df_missing_data], ignore_index = True)

df_historical_data.drop_duplicates(inplace = True)

df_historical_data.sort_values('year', inplace = True)

df_historical_data
```

![[1446-02-19 16_19_41-Data Cleaning & Transformation.ipynb - World Cup 2022 Prediction Model - Visual .png]]

## Cleaning df_historical_data

```python
# deleting match with walk over
delete_index = df_historical_data[df_historical_data['home'].str.contains('Sweden') &

                                 df_historical_data['away'].str.contains('Austria')].index

df_historical_data.drop(index = delete_index, inplace = True)
```

```python
# columns score with not only digits and "-"  --> [^ ]: Matches characters not in brackets (regex)

df_historical_data[df_historical_data['score'].str.contains('[^\d–]')]

df_historical_data['score'] = df_historical_data['score'].str.replace('[^\d–]', '', regex=True)

df_historical_data['score']
```

![[1446-02-19 16_23_17-Data Cleaning & Transformation.ipynb - World Cup 2022 Prediction Model - Visual .png]]

```python
# splitting score columns into home and away goals and drop the original score column

df_historical_data[['HomeGoals', 'AwayGoals']] = df_historical_data['score'].str.split('–', expand = True)

df_historical_data.drop('score', axis = 1, inplace = True)

df_historical_data
```

![[1446-02-19 16_24_11-Data Cleaning & Transformation.ipynb - World Cup 2022 Prediction Model - Visual .png]]

```python
# rename columns and change data types

df_historical_data .rename(columns ={'home': 'HomeTeam', 'away': 'AwayTeam', 'year': 'Year'}, inplace = True)

  

df_historical_data = df_historical_data.astype({'HomeGoals': int, 'AwayGoals': int, 'Year': int})

df_historical_data
```

![[1446-02-19 16_24_54-Data Cleaning & Transformation.ipynb - World Cup 2022 Prediction Model - Visual .png]]

```python
# creating new column "TotalGoals"

df_historical_data ['TotalGoals'] = df_historical_data['HomeGoals'] + df_historical_data['AwayGoals']

df_historical_data
```

![[1446-02-19 16_25_34-Data Cleaning & Transformation.ipynb - World Cup 2022 Prediction Model - Visual .png]]

## Export clean data

```python
df_historical_data.to_csv('clean_fifa_worldcup_historical_data.csv', index = False)

df_fixture.to_csv('clean_fifa_worldcup_fixture22.csv', index = False)
```

# Building the Model
---
1:18:43-2:14:57

## Calculate Team Strength

```python
# split df into df_home and df_away

  

df_home = df_historical_data[['HomeTeam', 'HomeGoals', 'AwayGoals']]

df_away = df_historical_data[['AwayTeam', 'HomeGoals', 'AwayGoals']]

# rename columns

df_home = df_home.rename(columns = {'HomeTeam': 'Team', 'HomeGoals': 'GoalsScored', 'AwayGoals': 'GoalsConceded'})

df_away = df_away.rename(columns = {'AwayTeam': 'Team', 'HomeGoals': 'GoalsConceded', 'AwayGoals': 'GoalsScored'})

# concat df_home and df_away, group by team and calculate the mean

df_team_strength = pd.concat([df_home, df_away], ignore_index=True).groupby('Team').mean()

df_team_strength
```

![[1446-02-19 17_48_40-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]
## Function predict_points

```python
def predict_points(home, away):

    if home in df_team_strength.index and away in df_team_strength.index:

        # goals_scored * goals_conceded

        lamb_home = df_team_strength.at[home, 'GoalsScored'] * df_team_strength.at[away,'GoalsConceded']

        lamb_away = df_team_strength.at[away, 'GoalsScored'] * df_team_strength.at[home, 'GoalsConceded']

        prob_home, prob_away, prob_draw = 0, 0, 0

        for x in range(0, 11): #number of goals home team

            for y in range (0, 11): #number of goals away team

                p = poisson.pmf(x, lamb_home) * poisson.pmf(y, lamb_away)

                if x == y:

                    prob_draw += p

                elif x > y:

                    prob_home += p

                else:

                    prob_away += p

        points_home = 3 * prob_home + prob_draw

        points_away = 3 * prob_away + prob_draw

        return (points_home, points_away)

    else:

        return(0, 0)
```

## Predict the World Cup

### Group Stage

```python
# splitting fixture into group, knockout, quarter, semi and final

df_fixture_group_48 = df_fixture[:48].copy()

df_fixture_knockout = df_fixture[48:56].copy()

df_fixture_quarter = df_fixture[56:60].copy()

df_fixture_semi = df_fixture[60:62].copy()

df_fixture_final = df_fixture[62:].copy()

# run all the matches in the group stage and update group tables

for group in wc22_groups:

    teams_in_group = wc22_groups[group]['Team'].values

    df_fixture_group_6 = df_fixture_group_48[df_fixture_group_48['home'].isin(teams_in_group)]

    for index, row in df_fixture_group_6.iterrows():

        home, away = row['home'], row['away']

        points_home, points_away = predict_points(home, away)

        wc22_groups[group].loc[wc22_groups[group]['Team'] == home, 'Pts'] += points_home

        wc22_groups[group].loc[wc22_groups[group]['Team'] == away, 'Pts'] += points_away

  

    wc22_groups[group] = wc22_groups[group].sort_values('Pts', ascending=False).reset_index()

    wc22_groups[group] = wc22_groups[group][['Team', 'Pts']]

    wc22_groups[group] = wc22_groups[group].round(0)
```

Sample Group:

```python
wc22_groups['Group A']
```

![[1446-02-19 17_52_21-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]
### Round of 16

```python
df_fixture_knockout
```

```python
# update the knockout fixture with group winners and runners up

for group in wc22_groups:

    group_winner = wc22_groups[group].loc[0, 'Team']

    runners_up = wc22_groups[group].loc[1, 'Team']

    df_fixture_knockout.replace({f'Winners {group}': group_winner, f'Runners-up {group}': runners_up}, inplace = True)

df_fixture_knockout['winner'] = '?'

df_fixture_knockout
```

![[1446-02-19 17_53_08-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]

```python
# create get_winner function

def get_winner(df_fixture_updated):

    for index, row in df_fixture_updated.iterrows():

        home, away = row['home'], row['away']

        points_home, points_away = predict_points(home, away)

        if points_home > points_away:

            winner = home

        else:

            winner = away

        df_fixture_updated.loc[index, 'winner'] = winner

    return df_fixture_updated

get_winner(df_fixture_knockout)
```

![[1446-02-19 17_53_47-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]
### Quarter-finals

```python
def update_table(df_fixture_round_1, df_fixture_round_2):

    for index, row in df_fixture_round_1.iterrows():

        winner = df_fixture_round_1.loc[index, 'winner']

        match = df_fixture_round_1.loc[index, 'score']

        df_fixture_round_2.replace({f'Winners {match}':winner}, inplace=True)

    df_fixture_round_2['winner'] = '?'

    return df_fixture_round_2
```

```python
update_table(df_fixture_knockout, df_fixture_quarter)
```

![[1446-02-19 17_54_38-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]

```python
get_winner(df_fixture_quarter)
```

![[1446-02-19 17_55_06-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]
### Semi-finals

```python
update_table(df_fixture_quarter, df_fixture_semi)
```

![[1446-02-19 17_55_38-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]


```python
get_winner(df_fixture_semi)
```

![[1446-02-19 17_55_44-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]
### Final

```python
update_table(df_fixture_semi, df_fixture_final)
```

![[1446-02-19 17_56_19-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]

```python
get_winner(df_fixture_final)
```


![[1446-02-19 17_56_42-Model for Prediction.ipynb - World Cup 2022 Prediction Model - Visual Studio Cod.png]]