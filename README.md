# Penang Weather Report WIP

## Background Overview
*Dataset* : Python's open source library - 'Meteostat'

*Records* : 5,840 raw rows

*Scope* : Temperature ranges(minimun|maximum|average), precipitation and wind speed.

| Table of contents|
|------------------|
|1. [Project Overview](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#project-overview)|                
|2. [Data Preparation](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#data-preparations)|        
|3. [Analytical Questions](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#analytical-questions)|  
|4. [Key Findings](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#key-findings)|
|5. [Creating and Testing the Weather Probability Assistant ](https://github.com/leemings98-source/Penang-Weather-Report/blob/main/README.md#creating-and-testing-the-weather-probability-assistant)|     
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
                              
In this instance, the GPS points for Penang was necessary as Meteostat was unable to zero in to the regions of Malaysia. Penang was chosen due to sentimental values. The dataset covers daily observations from January 2010 through December 2025. Allowing for a sufficient amount of data to analyze and predict from.   

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

Yes, temperature have been slowly increasing since 2010 in an oscillating fashion. Although the end of 2025's temperature ends in a decrease, it's temperature of 28.4°C is still a higher average than the start of 2010's 28.3°C.

- Which months have the most rainfall?

<img width="784" height="358" alt="{050F9FFB-8C1A-4309-9E04-9107DB4D92C0}" src="https://github.com/user-attachments/assets/624efeb9-5099-40bb-a5d8-dbb629cd8be5" />
<img width="784" height="384" alt="{A5637088-011A-4AFE-8E57-8C835FF15921}" src="https://github.com/user-attachments/assets/7f45f821-9037-4bb7-b82c-def917c0102f" /> 

October and November are the months with heavier rainfalls.

- Have rainfall increased/decrease over the years?

<img width="898" height="407" alt="{31BF10CD-09E9-4310-8CC3-526030C70552}" src="https://github.com/user-attachments/assets/e5e5e5a5-8248-407c-bdd4-f53308e93d86" />

Amount of precipitation across the years has been hovering around the 6mm amount with around 2mm max difference in increases and decreases. 2015's 0mm precipitation occurred due to insufficient data collected during the year.

## Key Findings
-

## Creating and Testing the Weather Probability Assistant 
_Aims_ 
- This project focuses on Penang as a case study to ensure data consistency and interpretability
- This assistant is based on historical climatology rather than short-term forecasting
- The assistant estimates conditions based on historical patterns for the selected month rather than making a day-specific weather forecast

### First Version

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

The foundation of the weather probability assistant, base thresholds was set as 34°C for high temperature and 10mm precipitation as a rainy day.

[Preview of Weather Forecast.webm](https://github.com/user-attachments/assets/903b1352-351e-4bef-87cb-ef29a54e0670)

Outputs for the current version seems to predominantly fall under generally pleasant. 

Conjecture: 
- Base threshold was placed too high.
- Variation of recommendations were too little, outputs values were unable to satisfy other outcome requirements. 

### Version 2.Testing and refining the code
- Updated the rainy precipitation amount to 8mm instead of 10mm
- Added several more weather recommendation results.
                     
                def weather_recommendation(probs):
        
                    hot = probs["prob_hot"]
                    rain = probs["prob_rain"]
                    
                    if hot > 50 and rain < 30:
                        return "Likely hot and relatively dry — stay hydrated and avoid midday outdoor activity."
                         
                    elif rain > 60:
                        return "High chance of rain — bring rain gear and plan indoor activities."

                    elif rain > 40:
                        return "Moderate chance of rain — bring an umbrella and keep outdoor plans flexible."

                    elif hot > 40:
                        return "Warm conditions are likely — stay hydrated and consider avoiding midday heat."

                    elif hot < 30 and rain < 30:
                        return "Generally pleasant conditions — good for outdoor activities."

                    else:
                        return "Mixed conditions — check a short-term weather forecast closer to the date."
                        
[Preview of Weather Forecast(revised).webm](https://github.com/user-attachments/assets/68d0777f-7b8f-4935-91b2-fea83966723c)

Lowering the baseline of rainy prcp to 8mm allowed for the results from October to output as 'Moderate Chance of Rain' instead of the initial 'Mixed Condition'. Though outputs from other months still seem to be regarded as 'Generally pleasant conditions'.

## Data Limitations
- As this is real world data, blanks in data collection is sometimes inevitable.

  -- Data collected for prcp(Total Precipitation) was sparse until it reached May of 2022. Many of the row's data was recorded as null values.
 
