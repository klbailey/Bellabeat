# Bellabeat
echo "# Bellabeat" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/klbailey/Bellabeat.git
git push -u origin main

#The query is used to check for nulls in any column. The DailyActivity does not have any nulls
SELECT *
FROM `bellabeat-99999.Case1.DailyActivity` 
WHERE NOT (`bellabeat-99999.Case1.DailyActivity` IS NOT NULL);

#The query is used to check for nulls in any column. The sleepDay does not have any nulls
SELECT *
FROM `bellabeat-99999.Case1.sleepDay`
WHERE NOT (`bellabeat-99999.Case1.sleepDay` IS NOT NULL);

#The query is used to check for nulls in any column. The WeightLogInfo does not have any nulls
SELECT *
FROM `bellabeat-99999.Case1.weightLogInfo`
WHERE Id || Date || WeightKg || WeightPounds || Fat || BMI || IsManualReport || LogId IS NULL;

#Created a table with the day of the week added
SELECT 
VeryActiveMinutes/60 as very_active_hours,
SedentaryMinutes/60 as sedentary_hours,
Calories,
format_date('%a', ActivityDate) as dayofweek
FROM `bellabeat-99999.Case1.DailyActivity`

#Join Activity and Sleep Activity
SELECT * 
FROM `bellabeat-99999.Case1.DailyActivity`dam
JOIN `bellabeat-99999.Case1.sleepDay` sd
ON dam.Id = sd.Id

#exported to table as Activity_Merge_Sleep

#Average amount of activity that a user is performing 
SELECT 
AVG(TotalSteps) as avg_total_steps,
AVG(TotalDistance) as avg_total_distance,
AVG(VeryActiveMinutes) as avg_very_active_minutes,
AVG(FairlyActiveMinutes) as avg_fairly_active_minutes,
AVG(LightlyActiveMinutes) as avg_lightly_active_minutes,
AVG(SedentaryMinutes) as avg_sedentary_minutes,
AVG(Calories) as avg_calories
FROM
`bellabeat-99999.Case1.DailyActivity`
/*
Analysis:
- More minutes are spent being sedentary > lightly active > fairly active > very active
*/



#Average amount of activity that is performed 
SELECT
DISTINCT Id, 
AVG(TotalSteps) as avg_total_steps,
AVG(TotalDistance) as avg_total_distance,
AVG(VeryActiveMinutes) as avg_very_active_minutes,
AVG(FairlyActiveMinutes) as avg_fairly_active_minutes,
AVG(LightlyActiveMinutes) as avg_lightly_active_minutes,
AVG(SedentaryMinutes) as avg_sedentary_minutes,
AVG(Calories) as avg_calories
FROM
`bellabeat-99999.Case1.DailyActivity`
GROUP BY
Id
ORDER BY
Id

#Average amount of user sleep in minutes and hours
WITH avg_sleep_hours as
(
  SELECT
  DISTINCT Id,
  AVG(TotalMinutesAsleep) as avg_minutes_asleep,
  AVG(TotalTimeInBed) as avg_minutes_in_bed
  FROM `bellabeat-99999.Case1.sleepDay`
  GROUP BY
  Id
)

SELECT 
*,
avg_sleep_hours.avg_minutes_asleep/60 as avg_sleep_hours,
avg_sleep_hours.avg_minutes_in_bed - avg_sleep_hours.avg_minutes_asleep as avg_time_awake
FROM avg_sleep_hours
ORDER BY
Id
/*
Analysis:
- Looking at the average data only 12 out of 24 got the recommended 7 hours of sleep
- 12 other users got an avg of less than 7 hours of sleep 
*/

#combined tables AverageActivityOfUSer & AverageSleep_MinutesHours
SELECT 
*
FROM `bellabeat-99999.Case1.AverageActivityOfUSer` aau
JOIN `bellabeat-99999.Case1.AverageSleep_MinutesHours` asu
ON aau.Id = asu.Id
ORDER BY
avg_very_active_minutes DESC
#The average amount of time a user is awake while in bed
SELECT
DISTINCT Id,
avg(TotalTimeInBed - TotalMinutesAsleep) as time_spend_awake_inbed
FROM `bellabeat-99999.Case1.sleepDay`
GROUP BY
id
ORDER BY
time_spend_awake_inbed
/*
Analysis:
- All users have some time in which they are in bed but not sleeping
- 19 out of the 24 users have more than 18 minutes of time in bed not sleeping
- This can be further seen in detail by the query below that shows each days amounts
*/

#The amount of time a user was awake in bed for each day
SELECT
DISTINCT Id,
SleepDay,
TotalTimeInBed - TotalMinutesAsleep as time_spend_awake_inbed
FROM `bellabeat-99999.Case1.sleepDay`
ORDER BY 
id, SleepDay

#Average users user information by day
WITH active_time as
(
  SELECT
  format_date('%a', ActivityDate) as dayofweek, 
  AVG(TotalSteps) as avg_total_steps,
  AVG(TotalDistance) as avg_total_distance,
  AVG(VeryActiveMinutes) as avg_very_active_minutes,
  AVG(FairlyActiveMinutes) as avg_fairly_active_minutes,
  AVG(LightlyActiveMinutes) as avg_lightly_active_minutes,
  AVG(SedentaryMinutes) as avg_sedentary_minutes,
  AVG(Calories) as avg_calories
  FROM `bellabeat-99999.Case1.DailyActivity`
  group by
  dayofweek
  Order by
    CASE
    WHEN dayofweek = 'Sun' THEN 1
    WHEN dayofweek = 'Mon' THEN 2
    WHEN dayofweek = 'Tue' THEN 3
    WHEN dayofweek = 'Wed' THEN 4
    WHEN dayofweek = 'Thu' THEN 5
    WHEN dayofweek = 'Fri' THEN 6
    WHEN dayofweek = 'Sat' THEN 7
      END ASC
)
SELECT 
dayofweek,
active_time.avg_total_steps,
active_time.avg_total_distance,
active_time.avg_very_active_minutes,
active_time.avg_fairly_active_minutes,
active_time.avg_lightly_active_minutes,
active_time.avg_sedentary_minutes,
active_time.avg_calories,
active_time.avg_very_active_minutes + active_time.avg_fairly_active_minutes + active_time.avg_lightly_active_minutes as total_active_time
FROM active_time
/*
saved this query as a table called ActivityByDay
Analysis:
- The most steps happen on Tuesday and Saturday
- The most active day is Saturday with an average of 244 minutes of activity when combining very, faily, lightly
- The least active day is Sunday
- The most sedentary day is Monday with the least being Thursday
*/

#Average users sleep information by day
WITH sleep_time as
(
  SELECT
  format_date('%a', SleepDay) as dayofweek, 
  AVG(TotalMinutesAsleep) as avg_minutes_asleep,
  AVG(TotalTimeInBed) as avg_minutes_in_bed
  FROM `bellabeat-99999.Case1.sleepDay`
  group by
  dayofweek
  Order by
    CASE
    WHEN dayofweek = 'Sun' THEN 1
    WHEN dayofweek = 'Mon' THEN 2
    WHEN dayofweek = 'Tue' THEN 3
    WHEN dayofweek = 'Wed' THEN 4
    WHEN dayofweek = 'Thu' THEN 5
    WHEN dayofweek = 'Fri' THEN 6
    WHEN dayofweek = 'Sat' THEN 7
      END ASC
)
SELECT 
dayofweek,
sleep_time.avg_minutes_asleep,
sleep_time.avg_minutes_in_bed,
avg_minutes_in_bed - avg_minutes_asleep as average_time_awake,
sleep_time.avg_minutes_asleep/60 as avg_hour_asleep,
sleep_time.avg_minutes_in_bed/60 as avg_hour_inbed
FROM sleep_time
/*
Saved this query as a table SleepByDay
Analysis:
- The most hours asleep is Sunday with 7.5 hours
- The most hours in bed is Sunday with 8.3 hours
*/



#combine data form the activity by day and sleep by day tables
SELECT
*
FROM `bellabeat-99999.Case1.ActivityByDay` abd
JOIN `bellabeat-99999.Case1.SleepByDay` sbd
ON abd.dayofweek = sbd.dayofweek

#This query gets us all the values together with total values. 
SELECT
abd.dayofweek,
abd.avg_very_active_minutes,
abd.avg_fairly_active_minutes,
abd.avg_lightly_active_minutes,
abd.avg_sedentary_minutes,
sbd.average_time_awake,
sbd.avg_minutes_asleep,
abd.total_active_time+ abd.avg_sedentary_minutes + sbd.avg_minutes_asleep + sbd.average_time_awake as all_total_time
FROM `bellabeat-99999.Case1.ActivityByDay` abd
JOIN `bellabeat-99999.Case1.SleepByDay` sbd
ON abd.dayofweek = sbd.dayofweek
Order by
  CASE
  WHEN dayofweek = 'Sun' THEN 1
  WHEN dayofweek = 'Mon' THEN 2
  WHEN dayofweek = 'Tue' THEN 3
  WHEN dayofweek = 'Wed' THEN 4
  WHEN dayofweek = 'Thu' THEN 5
  WHEN dayofweek = 'Fri' THEN 6
  WHEN dayofweek = 'Sat' THEN 7
    END ASC

#Find the SUM of active minutes and sedentary minutes
WITH active_time as
(
  SELECT
  format_date('%a', ActivityDate) as dayofweek, 
  SUM(TotalSteps) as sum_total_steps,
  SUM(TotalDistance) as sum_total_distance,
  SUM(VeryActiveMinutes) as sum_very_active_minutes,
  SUM(FairlyActiveMinutes) as sum_fairly_active_minutes,
  SUM(LightlyActiveMinutes) as sum_lightly_active_minutes,
  SUM(SedentaryMinutes) as sum_sedentary_minutes,
  SUM(Calories) as sum_calories
  FROM `bellabeat-99999.Case1.DailyActivity`
  group by
  dayofweek
  Order by
    CASE
    WHEN dayofweek = 'Sun' THEN 1
    WHEN dayofweek = 'Mon' THEN 2
    WHEN dayofweek = 'Tue' THEN 3
    WHEN dayofweek = 'Wed' THEN 4
    WHEN dayofweek = 'Thu' THEN 5
    WHEN dayofweek = 'Fri' THEN 6
    WHEN dayofweek = 'Sat' THEN 7
      END ASC
)
SELECT 
dayofweek,
active_time.sum_total_steps,
active_time.sum_total_distance,
active_time.sum_very_active_minutes as very_active_minutes,
active_time.sum_fairly_active_minutes as fairly_active_minutes,
active_time.sum_lightly_active_minutes as lightly_active_minutes,
active_time.sum_sedentary_minutes as sedentary_hours,
active_time.sum_calories,
active_time.sum_very_active_minutes + active_time.sum_fairly_active_minutes + active_time.sum_lightly_active_minutes as total_active_time,
active_time.sum_very_active_minutes + active_time.sum_fairly_active_minutes + active_time.sum_lightly_active_minutes + active_time.sum_sedentary_minutes as total_amount_of_time
FROM active_time
#exported as SumTotalTimes

#Find Percentage of time spent Sedentary & active per type
SELECT
avg(very_active_minutes)/avg(total_amount_of_time) * 100 as very_active_percentage,
avg(fairly_active_minutes)/avg(total_amount_of_time) * 100 as fairly_active_percentage,
avg(lightly_active_minutes)/avg(total_amount_of_time) * 100 as lightly_active_percentage,
avg(total_active_time/total_amount_of_time) * 100 as total_active_time_percentage,
avg(sedentary_hours/total_amount_of_time) * 100 as total_sedentary_hours_percentage
FROM `bellabeat-99999.Case1.SumTotalTimes`
#exported as ActivityPercentage
