---
title: "Data Wrangling With R (Bikeshare Data)"
author: "Kristian Murimi"
date: "3/21/2022"
---

```{r setup}
library(tidyverse)
library(janitor)
library(lubridate)

```

## Import Data

```{r import}
jan_2022 <- read_csv("capstone/202201-divvy-tripdata.csv")
dec_2021 <- read_csv("capstone/202112-divvy-tripdata.csv")
nov_2021 <- read_csv("capstone/202111-divvy-tripdata.csv")
oct_2021 <- read_csv("capstone/202110-divvy-tripdata.csv")
sep_2021 <- read_csv("capstone/202109-divvy-tripdata.csv")
aug_2021 <- read_csv("capstone/202108-divvy-tripdata.csv")
jul_2021 <- read_csv("capstone/202107-divvy-tripdata.csv")
jun_2021 <- read_csv("capstone/202106-divvy-tripdata.csv")
may_2021 <- read_csv("capstone/202105-divvy-tripdata.csv")
apr_2021 <- read_csv("capstone/202104-divvy-tripdata.csv")
mar_2021 <- read_csv("capstone/202103-divvy-tripdata.csv")
feb_2021 <- read_csv("capstone/202102-divvy-tripdata.csv")
```

## Merging our data
We will compare our tables to make sure that the column names and coresponding data is the same across all the tables. This is important since the data cannot be merged if there are differences in the data.

```{r comparing column data}
compare_df_cols_same(jan_2022,dec_2021,nov_2021,oct_2021,sep_2021,aug_2021,
                     jul_2021,jun_2021,may_2021,apr_2021,mar_2021,feb_2021)
```
The code gives us a result of TRUE meaning that the data is row-bindable. We can therefore merge the tables into a single dataset.

```{r merge}
tripdata <- do.call("rbind",list(jan_2022,dec_2021,nov_2021,oct_2021,sep_2021,
                                 aug_2021,jul_2021,jun_2021,may_2021,apr_2021,mar_2021,feb_2021))
```

## Looking at our new table                              
Let us take a look at our new table to make sure our data is all there and in the correct data type.
```{r columns}
colnames(tripdata)
```
```{r column structure}
str(tripdata)
```
```{r preview data}
head(tripdata)
```
We can coclude that our data merge was successful and our data is now in a single table making it easier to work with.

## Filtering our data
Now that all our data is in one table, we can filter our data to have only the columns we want to use.

```{r filter}
trip_data <- tripdata[,-c(9,10:12)]
```
The above code has allowed us to get rid of unwanted columns and create a new table with only the columns we want.

```{r new table columns}
colnames(trip_data)
```
## Data transformation
Now we can add new columns that can give use more insight into our data from the columns we have.

We can get the duration of a trip using the start and end times from the table.
```{r trip duration}
trip_data$trip_duration <- round(difftime(trip_data$ended_at,
                                          trip_data$started_at,units = 'mins'),2)
```
Let us make sure we worked it out correctly.

```{r column names}
colnames(trip_data)
```
```{r preview table}
head(trip_data)
```
The new column has been successfully added.
Let us check whether our new column has some null values.

```{r null}
sum(trip_data$trip_duration < 0)
```
There are 145 null or invalid values in the trip_duration column.
Let us remove them as we cannot get further information about those particular trips.

```{r cleanup}
trip_data <- trip_data[!(trip_data$trip_duration < 0),]
```

```{r check for null or invalid}
sum(trip_data$trip_duration < 0)
```
Now our column has no invalid or null values.

Let us add a column that gives us the day of the week each trip was taken.
```{r day of trip}
trip_data$trip_day <- weekdays(trip_data$started_at)
```

```{r preview day_of_trip}
head(trip_data)
```

Let us also group the time of day each trip took place under a new column.
```{r creating categories}
breaks <- hour(hm("00:00","6:00","12:00","18:00","23:59"))
labels <- c("Night","Morning","Afternoon","Evening")

trip_data$time_of_trip<- cut(x=hour(trip_data$started_at),
                             breaks = breaks, labels = labels,include.lowest = TRUE)
```
```{r preview time_of_trip}
head(trip_data)
```


## Exporting data for visualization
With those new columns our data is ready for visualization.
Let us export the data into a csv file.

```{r export, warning=TRUE}
write.csv(trip_data,"C:\\users\\krism\\documents\\trip_data.csv")
```
