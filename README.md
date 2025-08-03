# How Can a Wellness Tech Company Maximize Its Impact?

## A Google Data Analytics Professional Certificate Capstone Project

[Bellabeat](https://bellabeat.com/) is a brand specializing in health and fitness products designed for women. Their product lineup includes smart water bottles, stylish fitness watches and jewelry, as well as yoga mats. Users can track their wellness data using these products through the Bellabeat app.

Bellabeat’s co-founders want to examine the data from non-Bellabeat fitness trackers to understand how people engage with these devices. The company aims to leverage these insights to refine their marketing strategies.

## Project Objective

### Primary Stakeholders

1. **Urška Sršen**: Co-founder and Chief Creative Officer at Bellabeat.
2. **Sando Mu**: Mathematician and Co-founder at Bellabeat.
3. **Bellabeat Marketing Analytics Team**: A group of data analysts tasked with analyzing and reporting data to shape Bellabeat’s marketing approach.

### Bellabeat Products

* **Bellabeat App**: This app offers users valuable insights into their health data, such as activity, sleep, stress, menstrual cycle, and mindfulness habits. By syncing with Bellabeat’s product line, it helps users make informed health decisions.

* **Leaf**: A versatile wellness tracker that can be worn as a bracelet, necklace, or clip. The Leaf connects with the Bellabeat app to monitor activity, sleep, and stress.

* **Time**: A wellness-focused timepiece blending classic aesthetics with smart tracking capabilities. The Time watch tracks activity, sleep, and stress, providing users with detailed wellness insights.

* **Spring**: A smart water bottle that tracks daily hydration levels, ensuring users maintain proper hydration throughout the day. It syncs with the Bellabeat app for hydration monitoring.

* **Bellabeat Membership**: Bellabeat offers a subscription service that delivers 24/7 personalized wellness advice, covering nutrition, activity, sleep, health, beauty, and mindfulness, tailored to individual goals.

### Business Challenge

Analyze the usage data from non-Bellabeat smart devices and compare it with data from one of Bellabeat's products to derive actionable insights. These findings will assist in developing targeted marketing strategies.

### Key Questions

1. What are the patterns and trends in smart device usage?
2. How can these trends inform Bellabeat’s customer base?
3. How can Bellabeat adjust its marketing approach based on these trends?

## Data Preparation

### Data Source:

The data for this project is sourced from the Fitbit Fitness Tracker dataset on [Kaggle](https://www.kaggle.com/datasets/arashnic/fitbit). This dataset consists of 18 CSV files containing smart health data from 30 Fitbit users. The data, collected via Amazon Mechanical Turk from March to May 2016, includes detailed activity, heart rate, and sleep metrics. It was last updated in August 2022.

### Limitations of the Data:

* **Small Sample Size**: Only 30 users are represented in this dataset.
* **Age of Data**: The dataset is approximately six years old, and newer Fitbit devices might offer improved data accuracy.
* **Survey-Driven Data**: Since the data was collected via a survey, there may be inaccuracies or inconsistencies due to self-reporting biases.
* **Limited Weight Data**: Only eight users provided weight data, with many missing or manually entered values.

### Additional Data (Future Exploration):

Given the limitations of the Fitbit data, exploring additional sources may be valuable. One such source is the [Mi Band fitness tracker dataset (04.2016 - present)](https://www.kaggle.com/datasets/damirgadylyaev/more-than-4-years-of-steps-and-sleep-data-mi-band), which covers data from one user over a period of six years. This dataset includes step and sleep data, though some step data was corrupted for a two-week period and defaulted to zero.

---

## Methodology

### Selecting Data Files

The files `dailyActivity_merged.csv` (providing activity and calories data), `sleepDay_merged.csv` (offering sleep data), and `weightLogInfo_merged.csv` (containing weight information) are selected for analysis, as they provide key data for understanding user activity and health trends.

### Tools Used:

* **Excel** for an initial review and quality check of the data.
* **R** for data transformation and exploration.
* **Tableau** for advanced, interactive data visualization.

### Initial Data Quality Check

1. Remove any blank entries by applying filters.
2. Convert `Id` field to character type as no mathematical operations are required.
3. Change `ActivityDate` to a Date format (since no time data is needed).
4. In `dailyActivity_merged.csv`, there are instances where `TotalSteps` is 0 and `SedentaryMinutes` equals 1440. This may indicate inaccuracies due to user weight or height differences.
5. The `weightLogInfo_merged` file has sparse data for the `Fat` field, with very few entries, so this will not be used in analysis.

### Data Transformation and Exploration

All R code for the analysis can be found [here](https://github.com/ghaithkouki/Bellabeat-data-analysis-case-study/blob/main/BellaBeat_RScript).

1. Import the `tidyverse` package and datasets.
2. Check data integrity by verifying if it loads correctly.
3. Convert `Id` field to character type.
4. Rename columns like `ActivityDate`, `SleepDay`, and `Date` to appropriate date types.

```r
activity <- activity %>%
  mutate_at(vars(Id), as.character) %>%
  mutate_at(vars(ActivityDate), as.Date, format="%m/%d/%y") %>%
  rename("Day"="ActivityDate")
```

5. Combine datasets using `left` and `right` joins.
6. Add a `Weekday` variable to capture the day of the week.

```r
combined_data <- sleep %>%
  right_join(activity, by=c("Id", "Day")) %>%
  left_join(weight, by=c("Id", "Day")) %>%
  mutate(Weekday = weekdays(as.Date(Day, "m/%d/%Y")))
```

7. Filter out duplicates and count missing values, distinct `Id` entries.

```r
combined_data <- combined_data[!duplicated(combined_data), ]
sum(is.na(combined_data))
n_distinct(combined_data$Id)
```

After processing, the final dataset consists of 940 observations and 25 variables. It contains data for 33 unique user IDs, with distinct entries in `dailyActivity`, `sleepDay`, and `weightLogInfo` of 33, 24, and 8 respectively. There are 6893 missing values across the dataset.

---

## Data Analysis

### Summary Statistics and Visualizations

```r
combined_data %>%
  select(TotalMinutesAsleep, TotalSteps, TotalDistance, VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes, Calories, WeightKg, Fat, BMI, IsManualReport) %>%
  summary()
```

**Summary Statistics**: The average user weighs 72.04 kg, has a BMI of 25.19, and spends most of their time in light activity. They average 6.9 hours of sleep, take 7,638 steps, and cover 5.49 km daily.

![Summary Statistics](https://github.com/ghaithkouki/Bellabeat-data-analysis-case-study/blob/main/images/Summary%20Statistics.png)

**Steps by Day**: Users tend to take the most steps on Sundays and the least on Fridays, with fairly consistent activity on other days. This suggests that users find step tracking valuable, which is crucial for Bellabeat’s marketing strategy.

![Steps by Day](https://github.com/ghaithkouki/Bellabeat-data-analysis-case-study/blob/main/images/Total%20steps%20by%20day.png)

**Fairly Active Minutes by Day**: Activity levels drop on Wednesdays, possibly due to midweek fatigue. This could inform Bellabeat’s understanding of user behavior and engagement.

![Fairly Active Minutes by Day](https://github.com/ghaithkouki/Bellabeat-data-analysis-case-study/blob/main/images/Minutes%20of%20moderate%20activity%20per%20day.png)

**Distribution of Sleep Duration**: The majority of users sleep between 312 and 563 minutes, equivalent to 5.2 to 9.4 hours. This data can guide Bellabeat in tailoring sleep-related features in their products.

![Distribution of Total Sleep Time](https://github.com/ghaithkouki/Bellabeat-data-analysis-case-study/blob/main/images/Distribution%20of%20sleep%20time.png)

**Calories vs Sleep Time**: A positive trend emerges between calories burned and sleep duration, suggesting that those who sleep between 5 and 7 hours may burn more calories.

![Calories vs Time Slept](https://github.com/ghaithkouki/Bellabeat-data-analysis-case-study/blob/main/images/Total%20minutes%20Asleep%20vs%20Calories.png)

**Logged Activity Distance by Day**: The logged activity feature was not frequently used. This suggests that users prefer automatic tracking, which could influence Bellabeat’s future product decisions.

![Logged Activity Distance by Day](https://github.com/ghaithkouki/Bellabeat-data-analysis-case-study/blob/main/images/Logged%20Activities%20Distance.png)

---

## Actionable Insights

1. \*\*Encouraging


Weekend Activity\*\*: Users took the least steps on Fridays. Bellabeat could consider sending motivational reminders on Thursday evenings and Friday mornings to encourage continued activity.

2. **Automatic Tracking Preference**: The low usage of the "Logged Distance" feature suggests that users would prefer automatic data collection. Bellabeat should consider prioritizing automatic tracking in future products.

3. **Targeted Weight Management Tools**: Since weight logging was underused, Bellabeat could explore offering automatic tracking solutions, like smart scales, which would appeal to users who are interested in monitoring their weight.

4. **Additional Data Sources**: The [Mi Band fitness tracker data](https://www.kaggle.com/datasets/damirgadylyaev/more-than-4-years-of-steps-and-sleep-data-mi-band) could provide deeper insights, especially over longer periods of time.

