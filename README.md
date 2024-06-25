# Bellabeat-Case-Study
##### Author: Anushka Khanvilkar

##### [Tableau Dashboard]
#
## Introduction
**Bellabeat** is a high-tech manufacturer of health-focused smart products for women.  Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their own health and habits. Since its foundation in 2013, it has grown rapidly and quickly positioned itself as a tech-driven wellness company producing beautifully designed technology that informs and inspires women around the world.
##  Ask
Analyze Fitbit data to gain insight and help guide marketing strategy for Bellabeat to grow as a global player.

Primary stakeholders: Urška Sršen and Sando Mur, executive team members.

Secondary stakeholders: Bellabeat marketing analytics team.
#  The "Prepare" Phase 
Around thirty eligible Fitbit users consented to the submission of personal tracker data.
The  are the **eighteen** CSV documents available to us from the dataset.
 **The dataset has limitations :**
 1)The dataset is not up-to-date, and the survey period was limited to only two months, which could further affect the representativeness and validity of the results.
 2) Upon further investigation with ```n_distinct()``` to check for unique user Id, the set has 33 user data from daily activity, 24 from sleep and only 8 from weight. There are 3 extra users and some users did not record their data for tracking daily activity and sleep. 
 3) Most data is recorded from Tuesday to Thursday, which may not be comprehensive enough to form an accurate analysis. 

#  The "Process" Phase
 Installing relevant packages
 ```
install.packages("tidyverse")
install.packages("here")
install.packages("skimr")
install.packages("janitor")
install.packages("lubridate")

library(tidyverse)
library(here)
library(skimr)
library(janitor)
library(lubridate)
```
 Importing relevant datasets
 * *daily_activity*
* *daily_sleep*
* *hourly_steps*
```
daily_activity <- read.csv("/kaggle/input/fitbit/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")

daily_sleep <- read.csv("/kaggle/input/fitbit/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")

hourly_steps <- read.csv("/kaggle/input/fitbit/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/hourlySteps_merged.csv")
```
## Previewing the selected datasets
We check the content and the structure of the three data frames we selected.
```
head(daily_activity)
str(daily_activity)

head(daily_sleep)
str(daily_sleep)

head(hourly_steps)
str(hourly_steps)
```
##  Cleaning the data
First of all, we determine the number of unique users from each data frame.
```
n_unique(daily_activity$Id)
n_unique(daily_sleep$Id)
n_unique(hourly_steps$Id)
```
<img width="164" alt="image" src="https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/36f9f07c-536e-47f5-b733-4352509f6cb9">

Then, we check for any duplicates, if present.
```
sum(duplicated(daily_activity))
sum(duplicated(daily_sleep))
sum(duplicated(hourly_steps))
```
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/959de12f-8c16-45f9-9dbf-8c247814089f)

We find that there are three duplicate values in the data frame *daily_sleep*. We now remove duplicates and NA values from each data frame.
```
install.packages("dplyr")
library(dplyr)
install.packages("tidyr")
library(tidyr)

daily_activity <- daily_activity %>% 
  distinct() %>% 
  drop_na()
head(daily_activity)

daily_sleep <- daily_sleep %>% 
  distinct() %>% 
  drop_na()
head(daily_sleep)

hourly_steps <- hourly_steps %>% 
  distinct() %>% 
  drop_na()
head(hourly_steps)
```
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/60a89eee-b8c7-4a7f-a3df-acb0dfe2a01f)
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/0679df32-4210-4666-b69d-5bc36861a967)

Next, to enhance consistency and readability, we would like to make sure that the column names in each data frame are unique and consist only of the underscore character, numbers, and letters. We would also like to change the case of each column name into lowercase.

```
library(janitor)

clean_names(daily_activity)
clean_names(daily_sleep)
clean_names(hourly_steps)

daily_activity <- rename_with(daily_activity,tolower)
daily_sleep <- rename_with(daily_sleep,tolower)
hourly_steps <- rename_with(hourly_steps,tolower)
```
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/d5cfd125-06cf-434b-8e80-8547085f84a2)
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/e4680a2e-90e7-40d1-9ce3-92964ceaba9a)

We clean the date-time format for *daily_activity* and *daily_sleep*, for consistency in columns while merging these two data frames.

```
library(lubridate)

daily_activity <- daily_activity %>% 
  rename(date = activitydate) %>% 
  mutate(date = as_date(date,format="%m/%d/%Y"))
head(daily_activity)

daily_sleep <- daily_sleep %>% 
  rename(date = sleepday) %>% 
  mutate(date = as_date(date,format="%m/%d/%Y %I:%M:%S %p",tz=Sys.timezone()))
head(daily_sleep)
```
Likewise, we do the following for the *hourly_steps* data frame:
```
hourly_steps <- hourly_steps %>% 
  rename(date_time = activityhour) %>% 
  mutate(date_time = as.POSIXct(date_time,format="%m/%d/%Y %I:%M:%S %p",tz=Sys.timezone()))
head(hourly_steps)
```
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/9e744ed3-47ba-4cf8-b9ba-a2ceab9a6ba7)

## Merging the required data frames
​
Now, we merge the two data frames *daily_activity* and *daily_sleep*, using *id* and *date* as primary keys. 

```
daily_activity_sleep <- merge(daily_activity, daily_sleep, by=c("id","date"))
head(daily_activity_sleep)
```
#  The "Analyse + Share" Phase
After a thorough exploration and preprocessing of the dataset, we now move on to the core analysis and visualization phase of this case study.
First, let's take a look at a summary for daily data:
```
library(dplyr)

# Assuming both data frames have 'id' column, adjust the inner join
activity_sleep <- inner_join(daily_activity, daily_sleep, by = "id")

# Select specific columns and perform operations
activity_sleep_summary <- activity_sleep %>%
  select(totalsteps, calories,
         veryactiveminutes, fairlyactiveminutes, lightlyactiveminutes, sedentaryminutes,
         totalsleeprecords, totalminutesasleep, totaltimeinbed) %>%
  drop_na() %>%
  summary()

# Print the summary
print(activity_sleep_summary)
```
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/19d8b4d6-2d48-4567-ab3e-6bf21bb0409f)

We want to find out if there are correlations between these variables:

Daily Steps vs Calories
Daily Steps vs.Daily Sleep
Let's make a plot for steps vs. calories:

```
ggplot(activity_sleep, aes(x = totalsteps, y = calories)) +
  geom_jitter(alpha = 0.5) +  # Jittered points with transparency
  geom_rug(position = "jitter", size = 0.08) +  # Rug plot
  geom_smooth(method = "auto", se = FALSE, size = 0.6) +  # Smoothed line without confidence interval
  geom_text(aes(label = paste("r =", round(cor(totalsteps, calories), 2))), x = 20000, y = 2300, size = 4, color = "blue") +  # Pearson correlation annotation
  labs(title = "Daily Steps vs. Calories", x = "Daily Steps", y = "Calories") +  # Title and axis labels
  theme_minimal()  # Minimal theme
```
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/f4105cca-664f-4ed6-b180-4e5793107408)
```

activity_sleep <- activity_sleep[complete.cases(activity_sleep[, c("totalsteps", "totalminutesasleep")]), ]
library(ggplot2)
ggplot(data = activity_sleep, aes(x = totalsteps, y = totalminutesasleep)) +
  geom_point(position = "jitter", alpha = 0.5, size = 2) + 
  geom_smooth(method = "auto", color = "blue", size = 0.6) +  
  labs(title = "Daily steps vs. sleep", x = "Daily steps", y = "Minutes asleep") +
  theme_minimal()
```
![image](https://github.com/Anushka69124/Bellabeat-Case-Study/assets/130921214/61da9e2a-99c9-473f-87bd-e5bb03931562)

 To find out the days of the week on which the users are more active, and also those on which they sleep more
 ```
weekdays_steps_sleep <- daily_activity_sleep %>% 
  mutate(day_of_week=weekdays(date))

weekdays_steps_sleep$day_of_week <- ordered(weekdays_steps_sleep$day_of_week,
levels=c("Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"))

weekdays_steps_sleep <- weekdays_steps_sleep %>% 
  group_by(day_of_week) %>% 
  summarize(daily_steps=mean(totalsteps),daily_sleep=mean(totalminutesasleep))
head(weekdays_steps_sleep)
```






​





