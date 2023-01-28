# C-h-R-y-S-Google_data_analytics_case_study__Cyclistic
This is the markdown file for the Google data analytics case: Cyclistic

---
title: "Google_data_analytics_Capstone_Project_Cyclistics"
author: "Chrysophyllus Paler"
date: "2023-01-22"
output: html_document
---

#Cyclistics: A Bike Sharing Company


##A markdown file for the Google data analytics capstone project


Load all the packages and libraries. Installation packages were commented out because of a known R error.

```{r}

#load packages and libraries
#install.packages('tidyverse')
#install.packages("dplyr")
library(tidyverse)
library(dplyr)
library(lubridate)
library(readr)
library(janitor)

```


Data assignment

```{r}
#assign csv data to corresponding months
Jan <- read_csv('Jan.csv')
Feb <- read_csv('Feb.csv')
Mar <- read_csv('Mar.csv')
Apr <- read_csv('Apr.csv')
May <- read_csv('May.csv')
Jun <- read_csv('Jun.csv')
Jul <- read_csv('Jul.csv')
Aug <- read_csv('Aug.csv')
Sep <- read_csv('Sep.csv')
Oct <- read_csv('Oct.csv')
Nov <- read_csv('Nov.csv')
Dec <- read_csv('Dec.csv')
```


Verify that all the columns are consistent throughout all the months

```{r}
#compare columns before combining
compare_df_cols(Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec, return = "mismatch")
```


Creation of the data frame

```{r}
#combine all months to Bike_Trips data frame
Bike_Trips <- bind_rows(Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec)
```


Create an unaltered copy which helps when comparing data before and after cleaning and format conversion for accuracy

```{r}
#create an unaltered data copy to Orig_Bike_Trips before cleaning for easy reference
Orig_Bike_Trips <- Bike_Trips
```


Data cleaning and manipulation

```{r}
#remove rows with NA values for consistency of data
Bike_Trips <- drop_na(Bike_Trips)


#remove start and end (longitudes/latitudes) and station ids data because they are not needed
Bike_Trips = subset(Bike_Trips,select = -c(start_lat,end_lat,start_lng,end_lng,start_station_id,end_station_id))


#select rows where started_at < ended_at to correctly calculate trip_length
Bike_Trips <- filter(Bike_Trips,started_at < ended_at)


#convert and calculate trip_length
Bike_Trips$trip_length <- difftime(Bike_Trips$ended_at,Bike_Trips$started_at, units = 'hours')


#convert trip_length to numeric and remove those that are less that 1 min or more than 1 day
Bike_Trips <- filter(Bike_Trips, Bike_Trips$trip_length <= 24 & Bike_Trips$trip_length >= .0166667)


#get actual day
Bike_Trips$day <- strftime(Bike_Trips$started_at, format = "%a")


#get actual month
Bike_Trips$month <- strftime(Bike_Trips$started_at, format = "%b")


#convert into date time
Bike_Trips$time <- ymd_hms(Bike_Trips$started_at)


#break time into 4 groups
breaks <- hour(hms("00:00:00","06:00:00","12:00:00","18:00:00","23:59:59"))


#create labels
labels <- c("Night","Morning","Afternoon","Evening")


#assign labels into Bike_Trips
Bike_Trips$time_of_day <- cut(x=hour(Bike_Trips$time),breaks = breaks, labels = labels, include.lowest=TRUE)


#get actual date and convert it as real date
Bike_Trips$date <- strftime(Bike_Trips$started_at, format = "%D")
Bike_Trips$date <- as.Date(Bike_Trips$date, "%m/%d/%y")


#verify all removed and needed columns and check the structure of the data before saving
str(Bike_Trips)
```


