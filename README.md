# Penang Weather Report WIP

## Background Overview
*Dataset* : Phyton's open source library - 'Meteostat'

*Records* : 5,840 raw rows

*Scope* : Temperature ranges(minimun|maximum|average), precipitation and wind speed.

| Table of contents|
|------------------|
|1. [Project Overview](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#project-overview)|                
|2. [Data Preparation](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#data-preparations)|        
|3. [Analytical Questions](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#analytical-questions)|  
|4. [Key Findings](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#key-findings)|
|5. [Creating and Testing out the Weather Forecast](https://github.com/leemings98-source/Penang-Weather-Report/edit/main/README.md#creating-and-testing-out-the-weather-forecasttesting-out-the-weather-forecast)|     
|6. [Data Limitations](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#data-limitations)|     

## Project Overview

### Objective

- Identify months where temperatures are highest.
- Have temperatures been increasing as the years goes by?
- Which months have the most rainfall?
- Have rainfall increased/decrease over the years?
- Using all the data obtained, is it able to create a predictive weather report?

### Data Structure Overview
The base dataset is composed of one table, 11 columns, and 5840 rows of data. The columns is as follows:

| *Column* | *Meaning* | *Type* |
|----------|-----------|--------|
|time | Timestamp | Date|              
|tavg | Average Temperature | float |   
|tmin | Minimum Temperature | float |
|tmax | Maximum Temperature | float |
|prcp | Total Precipitation | float |
|snow | Amount of Snowdepth | float |
|wdir | Wind direction | float |
|wspd | Wind speed | float |
|wpgt | Peak wind gust | float |
|pres | Sea level air pressure | float |
|tsun | Total sunshine duration | float |

## Data Preperation
### Data Collection

The data for this project is obtained from Phyton's own open source library - 'Meteostat'. It retrieves historical observations and statistics from Meteostat Datasets, which aggregates information from various public sources primarily governmental agencies.

                              from meteostat import Point, Daily,Station
                              from datetime import datetime
                              Penang  = Point(5.3000, 100.2667)
                              start   = datetime(2010,1,1)
                              end     = datetime(2025,12,31)
                              
                              data = Daily(Penang, start,end)
                              data = data.fetch()
                              data.head()
                              
In this instance, the GPS points for Penang was necessary as Meteostat was unable to zero in to the regions of Malaysia. Penang was chosen due to sentimental values. The time period used for this dataset was a 15 year period starting from 2010 to 2025 allowing for a sufficient amount of data to analyze and predict from.   

- Saving a base file.                         

                              data.to_csv("Penang_15year_weather_info.csv")
                              base_data = pd.read.csv(r"C:\file directory\Penang_15year_weather_info.csv")
                              
### Data Cleaning
During the initial lookover of the dataset, no duplicates were found, instead removal of several columns were done. 

                              base_data.drop(columns = ["snow","wdir","wpgt","tsun","pres"],inplace = True )

Columns snow, wdir, wpgt and tsun was found to have no information recorded in their rows. While pres was not needed in this particular project.  

                              base_data.interpolate(limit = 3, inplace =True)
                              
To handle short data gaps while avoiding artificial smoothing, missing values were interpolated with a maximum gap of three consecutive days

                              Monthly = base_data.resample('ME').mean()
                              Yearly = base_data.resample('YE').mean()
                              
Daily Meteostat observations were aggregated to monthly and yearly means to reduce short-term variability and highlight long-term climatic trends. Yearly for overall changes, Monthly for probability
 
## Analytical Questions
- Identify months where temperatures are highest.
  
<img width="740" height="398" alt="{309E25B6-8846-48A8-BAC0-1EE870D520EB}" src="https://github.com/user-attachments/assets/05d522a0-6e80-4bfc-9f8c-b199a15c2a29" />

Months between February to April during the 15 year period are the warmest compared to the other months. With March 2019 being the month with the highest average temperature across all other months.

- Have temperatures been increasing as the years goes by?

<img width="758" height="304" alt="{56C2721A-5816-44E2-A7D2-29FD44500342}" src="https://github.com/user-attachments/assets/9c42c65a-d31a-4ca7-bf02-e06db8a2cd8c" />

Yes, temperature have been slowly increasing since 2010 in an oscillating fashion. Although the end of 2025's temperature ends in a dip, it's temperature of 28.4 degree Celsius is still a higher average than the start of 2010's 28.3 degree Celsius.

- Which months have the most rainfall?

<img width="748" height="361" alt="{F6EA33C0-423A-46C3-AC4F-71C0385F0222}" src="https://github.com/user-attachments/assets/620bf1e2-0c42-47e3-b1b4-453d501d6cc4" />

-- actually go back into phyton and make a chart for this avg rainfall. I still have no idea how to read a box chart accurately, also, the numbers are way too congregated to actually know  -.-|||


- Have rainfall increased/decrease over the years?

<img width="784" height="358" alt="{050F9FFB-8C1A-4309-9E04-9107DB4D92C0}" src="https://github.com/user-attachments/assets/624efeb9-5099-40bb-a5d8-dbb629cd8be5" />

Apparently not, it seems that the amount of precipitation has been at a constant 6mm not decreasing in volume. 2015's "0mm" precipitation happened due to insufficient data during that year therefore it is safe to disregard the year's data within our analysis.

## Key Findings

## Creating and Testing out the Weather ForecastTesting out the Weather Forecast 

                HOT_TEMP = 34       # °C
                RAINY_PRCP = 10      # mm monthly avg (example)

                def weather_probability_assistant(date_str, Monthly):
                    date = pd.to_datetime(date_str)
                    month = date.month

                    # Filter historical data for the same month
                    month_data = Monthly[Monthly.index.month == month]

                    if month_data.empty:
                        return "Not enough historical data for this month."
  
                    # Probabilities
                    prob_hot = (month_data["tmax"] > 34).mean() * 100
                    prob_rain = (month_data["prcp"] > 10).mean() * 100

                    return {
                        "month": date.strftime("%B"),
                        "prob_hot": round(prob_hot, 1),
                        "prob_rain": round(prob_rain, 1)
                    }
                def weather_recommendation(probs):
                    if probs["prob_hot"] > 50 and probs["prob_rain"] < 15:
                          return "Likely hot and dry — stay hydrated and avoid midday outdoor activity."
    
                    if probs["prob_rain"] > 60:
                        return "High chance of rain — bring rain gear and plan indoor activities."
    
                    if probs["prob_hot"] < 30 and probs["prob_rain"] < 30:
                        return "Generally pleasant conditions — good for outdoor activities."
    
                    return "Mixed conditions — check a short-term weather forecast closer to the date."

## Data Limitations
- Blanks in data may occur 
