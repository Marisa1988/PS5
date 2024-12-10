# Problem Set 5

### Please enter the names of the group members here:
Marisa Morales

## Due date
Problem Set 5 is due on **Friday, November 15th**.

## Information

We've been playing with the `world.sqlite` database in class a lot. These questions make use of this database.
* The database is at `/blue/bsc4452/share/Class_Files/data/world.sqlite`
* It can also be downloaded from: https://github.com/comptoolsres/Class_Files/raw/main/data/world.sqlite

### Questions 1, 2, and 3 are in SQL code, while questions 4 and 5 are in SQLAlchemy code. This repository also contains a .ipynb file with the last questions to view the graphs.
### You can also find the database with the required information for the country 'Serbia' and the cities of 'Belgrade' and 'Novi Sad'.

## Question 1 (5 points):

What is the country in the databese with the latest year (most recent) of independence? 
* Provide your answer (2 points) and the code used to get that answer (3 points). 
* The code can either be SQL or SQLAlchemy code.

### Answer: The country with the most recent year of independence is Palau in 1994.

### SQL code: SELECT Name, IndepYear FROM country WHERE IndepYear = (SELECT MAX(IndepYear) FROM country);

## Question 2 (5 points): 

Refer to this page: [https://www.statista.com/chart/11430/the-worlds-youngest-countries/](https://www.statista.com/chart/11430/the-worlds-youngest-countries/)

According to this, there are several countries that have become independent since the country in your answer to question 1.

Pick one of those newer countries and using Wikipedia, or another source, add as much data to the country table as you can for that country.

Do not spend a lot of time trying to find all the data. One or two additional items beyond Name and IndepYear is fine.

Provide either a SQL INSERT statement or a SQLAlchemy insert statement to add the data for a new country into the database.

### SQL code to add information of Serbia: INSERT INTO country (Code, Name, Continent, Region, SurfaceArea, IndepYear, Population, LifeExpectancy, GNP, GNPOld, LocalName, GovernmentForm, HeadOfState, Capital) VALUES ('SRB', 'Serbia', 'Europe', 'Balkans', 88499, 2006, 6600000, 75, 75.015, 173.075, 'Republika Srbija', 'Unitary parliamentary republic', 'Aleksandar Vucic', 'Belgrade');

### SQL code to show information added: SELECT * FROM country WHERE code = 'SR'

## Question 3 (5 points):

For the country added in question 2, find two cities to add to the `city` table of the database and provide the SQL or SQLAlchemy insert statement to add this data. 
* Again, don't sweat it if you can't find *all* of the data. 
* If it's not in Wikipedia, don't spend time looking for it! 
* A few columns worth of data is sufficient.

### SQL code to add information of Belgrade and Novi Sad: INSERT INTO city (ID, Name, CountryCode, District, Population) VALUES (4080, 'Belgrade', 'SR', 'Belgrade', 1197714), (4081, 'Novi Sad', 'SR', 'Vojvodina', 346773);

### SQL code to show information added: SELECT * FROM city WHERE name IN ('Belgrade', 'Novi Sad');

## Question 4 (5 points):

* **Grad Students Only; Undergrads get 5 points free, but can earn 5 points extra credit points for doing this question**

Using the LifeExpectancy data in the `country` table on the y-axis, plot this data against some other value. 
* Suggestions for the x-axis: GNP, Population or IndepYear could be interesting, but up to you.
* I'd suggest using SQLAlchemy, get the data and make either a dataframe or numpy arrays and then use `matplotlib` to plot.

### Option A: SQLAlchemy code for scatter plot of Life Expectancy vs Population: 

import matplotlib.pyplot as plt
import pandas as pd
from sqlalchemy import create_engine, MetaData, Table

#### Define the database URL
engine = create_engine('sqlite:////blue/bsc4452/Marisa1988/ps5-mmm/world.sqlite')
connection = engine.connect()
metadata = MetaData()

#### Reflect the table
country = Table('country', metadata, autoload_with=engine)

#### Query the data
query = connection.execute(country.select())
data = query.fetchall()

#### Convert the data to a DataFrame
df = pd.DataFrame(data, columns=query.keys())

#### Create a scatter plot with column 6 (Population) on the x-axis and column 7 (Life Expectancy) on the y-axis
plt.scatter(df.iloc[:, 6], df.iloc[:, 7])

#### Set the labels for the axes (x=Population and y=Life Expectancy)
plt.xlabel(df.columns[6])
plt.ylabel(df.columns[7])

#### Set the title of the plot
plt.title('Scatter Plot of Life Expectancy vs Population')

#### Show the plot
plt.show()

### Option B: SQLAlchemy code for bar plot of Life Expectancy vs Continent: 

import matplotlib.pyplot as plt
import pandas as pd
from sqlalchemy import create_engine, MetaData, Table

#### Define the database URL
engine = create_engine('sqlite:////blue/bsc4452/Marisa1988/ps5-mmm/world.sqlite')
connection = engine.connect()
metadata = MetaData()

#### Reflect the table
country = Table('country', metadata, autoload_with=engine)

#### Query the data
query = connection.execute(country.select())
data = query.fetchall()

#### Convert the data to a DataFrame
df = pd.DataFrame(data, columns=query.keys())

#### Create a bar plot with column 2 (Continent) on the x-axis and column 7 (Life Expectancy) on the y-axis
plt.figure(figsize=(10, 6))
plt.bar(df.iloc[:, 2], df.iloc[:, 7])

#### Set the labels for the axes (x=Continent and y=Life Expectancy)
plt.xlabel(df.columns[2])
plt.ylabel(df.columns[7])

#### Set the title of the plot
plt.title('Bar Plot of Life Expectancy vs Continent')

#### Show the plot
plt.show()

## Question 5 (5 points):
## Grad student extra credit (5 points, sorry undergrads, only question 4 counts as extra credit): 

Plot LifeExpectancy vs the ratio of the total population of all the cities in the country divided by the total population of the country. This is an approximation of the % urban population in the country.

### SQLAlchemy code: 

import matplotlib.pyplot as plt
import pandas as pd
from sqlalchemy import create_engine, MetaData, Table

#### Define the database URL
engine = create_engine('sqlite:////blue/bsc4452/Marisa1988/ps5-mmm/world.sqlite')
connection = engine.connect()
metadata = MetaData()

#### Reflect the tables
country = Table('country', metadata, autoload_with=engine)
city = Table('city', metadata, autoload_with=engine)

#### Query the data from the 'country' table
country_query = connection.execute(country.select())
country_data = country_query.fetchall()
country_df = pd.DataFrame(country_data, columns=country_query.keys())

#### Query the data from the 'city' table
city_query = connection.execute(city.select())
city_data = city_query.fetchall()
city_df = pd.DataFrame(city_data, columns=city_query.keys())

#### Calculate the % urban population for each country
urban_population = city_df.groupby('CountryCode')['Population'].sum()
total_population = country_df.set_index('Code')['Population']
urban_percentage = (urban_population / total_population) * 100

#### Create a new DataFrame for the 'urban' table
urban_df = pd.DataFrame({
    'CountryCode': urban_percentage.index,
    'UrbanPercentage': urban_percentage.values
}).reset_index(drop=True)

#### Merge the 'urban' DataFrame with the 'country' DataFrame to get LifeExpectancy
merged_df = pd.merge(urban_df, country_df[['Code', 'LifeExpectancy']], left_on='CountryCode', right_on='Code')

#### Create a scatter plot with LifeExpectancy on the y-axis and % urban population on the x-axis
plt.scatter(merged_df['UrbanPercentage'], merged_df['LifeExpectancy'])

#### Set the labels for the axes
plt.xlabel('% Urban Population')
plt.ylabel('Life Expectancy')

#### Set the title of the plot
plt.title('Scatter Plot of Life Expectancy vs % Urban Population')

#### Show the plot
plt.show()


