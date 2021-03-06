# 12. Time Series Data {#time}
Most data collected by water utilities are time series, especially SCADA systems. A time series is any type of data that contains repeated measurements of the same parameter over time. This data could be continuous measurements of the pressure in the water mains, daily consumption, monthly performance or annualised financial results. The water quality data in case study 1 is a time series, but with spares observations. Any data where the x-axis displays time and the y-axis a numeric value is a time series.

Traditionally, water utilities only measure consumption from individual customers when a bill needs to be generated. As reading water meters is a time-consuming task this occurs in most cases less then once every month. This limited information is great for billing, but it does not tell us much about patterns of consumption of water consumers.

New technology enables water meters to transmit data at a much higher frequency so that we can gain deeper insights into how consumer use their water. This data unlocks many benefits, from providing customers with detailed knowledge of their consumption, to leak detection and network optimisation. This case study uses the functionality of the Tidyverse to analyse smart meter data.

This chapter introduces the case study data and explains how the R language manages dates and time. The learning objectives for this chapter are:
1. Understand the principles of how the R language stores time and date variables
2. Use date and time variables in calculations and visualisations
3. Apply the functionality of the Lubridate package to mages dates and times.

The data and code for this session are available in the `chapter_12.R` file in the `casestudy3` folder of your RStudio project.

## Problem Statement
Most water utilities measure the consumption of their customers less than once per month, and some don't measure consumption at all. While this practice might be sufficient for billing, the limited amount of consumption data constrains the knowledge utilities can have about how water moves through the system. This approach is like looking at your bank balance once every month and then try to figure out where your money went.

The Invisible Water Utility has just completed a pilot study of 100 services in the Gormsey system.  Technicians fitted data loggers on existing water meters that provides hourly meter reads. Your task is to explore this data and develop some algorithms to visualise consumption and to find services with leaks.

### Smart Water Meters
The term smart meters is quite common, but it contains a lot of marketing spin. Most customer water meters in this category are standard devices fitted with an electronic data logger and transmitter. These data loggers are an integral part of the meter, or they are retrofitted to the device. These meters are not intrinsically smart but provide the utility with detailed data that allows water professionals to make smarter decisions.

![Figure 12.1: Water meter data logger by Ventia](resources/12_time_series/digital_meter.jpg)

Digital water meters provide data at varying frequencies, from every few seconds to daily reads. Deciding how much data to collect depends on several considerations. Water engineers ideally like a reading at least every five minutes to match their modelling data frequency, while the billing department would be more than happy with one daily reading. The customer service team likes to know whether a property has leaks.

While a high data collection frequency would satisfy everybody's needs, there is also a high cost. The higher the data rate, the higher the cost of collection due to increased transmission bandwidth, reduced battery life and data storage costs. Collecting data every few minutes is most likely unfeasible. Collecting data at high frequencies is  potentially unethical because it reveals too much about the lifestyles of customers. Daily data is insufficient to provide benefits in network design and operation. Hourly reads seems a good compromise because it allows for most of the sought benefits, doesn’t significantly impact the privacy of customers, and is within reasonable reach for the current level of technology.

### Simulating water consumption
The data for this case study is synthetic and based on assumptions and stochastic variables. This data was created in a first step to develop a reporting system for smart meter data. Using simulated data enabled the development of the software before the actual data was available.

Consumption data is simulated for two reasons. Firstly, detailed information about the water consumption of consumers reveals information about their lifestyle. This type of data reveals how many people live in the house, their nightly toilet habits, when they are on holidays, and so on. Using real data could thus cause a breach of privacy. Secondly, simulating data is an effective method to test computational methods because we know the expected outcomes.

The R language includes many functions to generate patterns of random numbers to simulate stochastic processes. Simulating a process can provide information about the distribution of possible outcomes through a [Monte Carlo simulation](https://www.investopedia.com/terms/m/montecarlosimulation.asp). This type of simulation is often applied to probabilistic cost estimates. The method used to simulate the data for this case study is explained in an R Markdown file in the `casestudy3` folder.

## Working with Dates and Times {#posix}
Digital Metering data is a time series because each data point is indexed by the time of the measurement. The data in this case study is an equally-spaced time series, which means that all time intervals are the same, besides some exceptions. In reality, most time-series data is irregular, which complicates the analysis.

Analysing a time series is a specialised task for which many packages exist in R. One of the necessary skills you need to analyse a time series is to work with time and date variables. The R language has extensive functionality to work with dates and times, and the Tidyverse contains the *lubridate* package to 'lubricate' working with these complex data types.

### Dates
Computers store times and dates as a number of seconds from a point of origin. Unix systems start counting at 1 January 1970. R uses Unix time, also called POSIX. The underlying data for all time and date variables is thus a large integer from the starting point. Unix time also has a millennium problem. At 03:14:08 UTC on Tuesday, 19 January 2038, 32-bit versions of the Unix timestamp clock over back to zero.

POSIX is a standard for computer data that includes dates and times. A POSIX variable includes a date, time and timezone, e.g. "15 Jul 2019 13:41:36 AEST".

The function `Sys.Date()` gives the current date. The variable type is a date, which means that it is displayed as a human-readable date, but underneath the display are the number of days in the Unix epoch.

{format: r, line-numbers: false}
```
Sys.Date()
as.numeric(Sys.Date())
```

You can format this date any way you like with the `format()` function. The default format is `YYYY-MM-DD`. To change the format, you need to use a special code, for example `format(Sys.Date(), "%A %d %B %Y")` shows the current date as “Tuesday 15 June 2021”. 

The codes start with a percent sign, followed by an indicator. Some of the most commonly used ones are:

* `%A`: Name of the week.
* `%B`: Name of the month
* `%d`: Day of the month
* `%m`: Number of the month
* `%Y`: Year with century

Q> Display the current date in American format, e.g. June 15, 2021.

I> You can add characters in your format definition: `format(Sys.Date(), "%B %d, %Y")`.

You can create a date variable by converting a string. The conversion variables ensure that all common date formats can be converted to a date variable.

Q> Review the examples below in RStudio. Make sure you review the error message and how it is resolved.

{format: r, line-numbers: false}
```
as.Date("2020-07-01")

as.Date("1 July 2020")

as.Date("1 July 2020", format = "%d %B %Y")
```

When a variable is registered as a date, you can also use arithmetic functions to calculate time differences.

Q> How many days has somebody who was born on 13 March 1977 lived? Use the `as.Date()` function to create two variables.

Please note that the results of a calculation with dates and times is of a special variable class. To use this result in further calculations you need to convert it to a numerical value by using the `as.numeric()` function.

T> Try to take the square root of the time difference and review the result. Then convert it with `as.numeric()` and try again.

{format: r, line-numbers: false}
```
d <- Sys.Date() - as.Date("1977-03-13")

sqrt(d)

sqrt(as.numeric(d))
```

### Time
In R, time is stored as the number of seconds in the Unix epoch. The current time is available with the `Sys.time()` function, which includes the date and timezone in the standard formatting, e.g. “2021-06-15 20:22:11 AEST”.

Q> Convert the current system time to a number and divide it by the number of seconds in a day. Does it match the number of days in the Unix epoch?

You can change the format of a time (POSIXct) variable in the same way as dates, using some additional modifiers.

* `%H`: Number of hours (00--23) 
* `%I`: Number of hours (01--12)
* `%M`: Minutes (00-59)
* `%p`: AM/PM indicator
* `%S`: Seconds (00-61)

Many other options are available, which you can read in the help entry for the `strptime()` function.

Q> Read the `strptime()` help page and display the current date and time as “08:20:33 PM”, or equivalent.

I> You can use one of the several shortcuts: `format(Sys.time(), "%r")`.

You can create POSIX date/time variables using the same principles as with dates, for example:

{format: r, line-numbers: false}
```
as.POSIXct("21 December 2020 12:23", format = "%d %B %Y %H:%M")
```

Note that this variable will be converted to your current timezone. The next section shows how to include a timezone in your variables.

### Timezones
The different timezones across the globe can be confusing, especially when confounded by daylight savings. timezones can also suddenly change for political reasons. For this reason, time series data is often provided in Coordinated Universal Time (UTC). This timezone corresponds with the 0 meridian over Greenwich near London. 

Computers synchronise to this time which is measured with atomic clocks. Computers can access these clocks (NTP servers) to synchronise with UTC. The fact that the name of this timezone and its abbreviation don't match is a result of political compromise between different countries.

When you ask for the current time, the timezone is displayed in a standard abbreviation. In my case the timezone is AEST (Australian Eastern Standard Time). The `Sys.timezone()` function displays the timezone your computer is configured at.

You can easily display any POSIX variable in another timezone with the `format()` function.

{format: r, line-numbers: false}
```
format(Sys.time(), tz = "UTC")
format(Sys.time(), tz = "NZ")
```

This function only displays a new timezone, it does not change the underlying variable. To change the timezone and reset the time, you need to change the variable’s attribute. The code below sets `t` to the current system time and converts this to New Zealand time.

R uses the international standard IANA time zones. These use a consistent naming scheme “/”, typically in the form `<continent>/<city>` (there are a few exceptions because not every country lies on a continent). To find other timezone designations, review the output of the `OlsonNames()` function.

{format: r, line-numbers: false}
```
nzt <- Sys.time()
attr(nzt, "tzone") <- "NZ"
nzt
```

Q> What is the local time in “Europe/Ljubljana”?

To create a POSIX variable with a specific timezone, you need to add another option:

{format: r, line-numbers: false}
```
as.POSIXct("21 December 2020 12:23", format = "%d %B %Y %H:%M", tz = "Israel")
```

### The Lubridate package
The [Lubridate package](https://lubridate.tidyverse.org/) forms part of the Tidyverse and provides syntactic sugar (simplified functions) to work with date and time variables. This package uses the same underlying data types, but provides some additional functionality and simplifies some tasks of the basic R function set. The table below shows some base R functions and their simplified Lubridate alternative

| Base R                                                         | Lubridate                                |
|----------------------------------------------------------------|------------------------------------------|
| `Sys.Date()`                                                   | `today()`                                |
| `as.numeric(format(Sys.Date(), "%m"))`                         | `month(Sys.Date())`                      |
| `as.Date(paste(format(Sys.Date(), "%Y-%m"), "01", sep = "-"))` | `round_date(Sys.Date(), unit = "month")` |

Explore these bits of code and review the outputs. The code in the remainder of this case study extensively uses lubridate functionality without explaining it in detail. The functions are self-explanatory and I leave it to you to reverse-engineer the code.

## Exploring the digital metering data
The `meter_reads.csv` file contains the simulated digital metering data with three fields:
* `DevicdID`: Unique id for the data logger
* `TimeStamp`: Date and time the measurement was taken
* `Count`: Meter read.

The code below reads the digital metering data and filters the data for one data logger between two dates. The graph provides some insight on how the data is collected.

Using the `glimpse(reads)` expression, we find out that the data has three variables with 876,000 observations (reads).

{line-numbers: false}
```
Observations: 876,000
Variables: 3
$ DeviceID   <chr> "RTU-2378716", "RTU-2378716", "RTU-2378716", "RTU-2378716"…
$ TimeStamp <dttm> 2069-06-30 14:09:08, 2069-06-30 15:09:08, 2069-06-30 16:0…
$ Count      <dbl> 2, 4, 5, 7, 10, 12, 17, 24, 30, 35, 39, 43, 47, 51, 54, 58…
```

{format: r, line-numbers: false}
```
meters <- read_csv("casestudy3/meter_reads.csv")
glimpse(reads)
meters_subset <- filter(meters, TimeStamp >= "2069-05-01" & 
                          TimeStamp < "2069-05-10" & 
                          DeviceID == "RTU-640893")
ggplot(meters_subset, aes(TimeStamp, Count * 5)) + 
  geom_line() + 
  labs(title = "Digital Meter Reads",
       subtitle = "RTU-640893",
       y = "Cumulative consumption [litres]")
```

The `Count` variable is the cumulative read of the meter. It represents the number of revolutions of the dial on the physical meter. Each rotation represents five litres. This number depends on the brand and type of physical meter. For this case study, all meters are assumed to have the same displacement per rotation.

The graph shows the data for a random meter in the dataset. You can see that consumption increases during the day and stays flat during the night.

![Digital meter data extract](resources/12_dates_times/meter-reads.png)

You will notice that the first entry of the filtered data frame has a timestamp of 30 April because a timezone conflict. The collected data is in UTC format, while we are filtering in the local Australian timezone. Midnight of 1 May in Australia is 14:00 on the previous dat in the UTC zone.

To prevent confusion, it is best to create a new variable with the local time using the `with_tz()` function from the Lubridate package. This code adds a new variable with a timestamp in the Australian timezone.

{format: r, line-numbers: false}
```
library(lubridate)
meters_au <- meters  %>% 
  mutate(TimeStampAU = with_tz(TimeStamp, tzone = "Australia/Melbourne"))
```

### Aggregating Water consumption

{format: r, line-numbers: false}
```
weekly <- meters_au %>% 
  mutate(Week = ceiling_date(TimeStampAU, unit = "week")) %>% 
  group_by(DeviceID, Week) %>% 
  summarise(Consumption = (max(Count) - min(Count)) * 5) %>% 
  group_by(Week) %>% 
  summarise(Weekly_Consumption = sum(Consumption) / 1000)

ggplot(weekly, aes(Week, Weekly_Consumption)) + 
  geom_col(fill = "dodgerblue4") + 
  labs(title = "Weekly Consumption",
       subtitle = "Digital Meters",
       y = "Cumulative consumption [litres]") + 
  theme_minimal()
  ```
X> Reverse-engineer this code to understand how it functions.

## Analysing water consumption


To determine the level of consumption between two reads in litres per hour, we need to subtract two consecutive reads from each other. Since there is no missing data in this case study, the period between all reads is precisely one hour. In real life, there are missing data points, which means you also need to determine the time difference between subsequent reads.

A fast way to determine the difference between consecutive numbers in a vector is the `diff()` function. An example illustrates the principle. The code below results in a vector with four times the number 1: (`2 - 1, 3 - 2, 4 - 3, 5 - 4`$). Note that this new vector is one element shorter than the original.

{format: r, line-numbers: false}
```
v <- c(1, 2, 3, 4, 5)
diff(v)
```

However, if we apply this function to the whole data frame, we get in trouble when we move from one device to the next. Taking the difference between rows 2 and 3 leads to negative consumption. You can use `summary(diff(reads$Count))` to see the negative values.

{format: r, line-numbers: false}
```
reads[(24 * 365 - 1):(24 * 365 + 2), ]

  DeviceID    TimeStamp           Count
  <chr>       <dttm>              <dbl>
1 RTU-6408930 2070-06-30 12:15:35 12410
2 RTU-6408930 2070-06-30 13:15:35 12410
3 RTU-1300375 2069-06-30 14:57:49     5
4 RTU-1300375 2069-06-30 15:57:49    10
```

We can solve this problem by applying the `diff()` function to the grouped data. However, because the result is one shorter than the original vector, we need to add an `NA` value to the result. Without this addition, we cannot fill every value in the data frame, which is a requirement. The difference between consecutive values is multiplied by five to get litres per hour. The `mutate()` function assigns the new variable to the data frame.

After we have the flow, the `Count` variable can be ditched, and we remove all `NA` values in `Flow` (the very first read). This data frame is the basis of all further analysis, so we save it to disk.

{format: r, line-numbers: false}
```
flow <- reads %>%
    group_by(DeviceID) %>%
    arrange(TimeStamp) %>%
    mutate(Flow = c(NA, diff(Count) * 5)) %>%
    select(-Count) %>%
    filter(!is.na(Flow))    

write_csv(flow, "casestudy3/flow.csv")
```

## Visualising Consumption
Now that we have the flow in litres per hour for each service it is pretty easy to visualise consumption for individual properties.

Q> Recreate the graph in figure 12.3. Tip: First use the `slice()` function to select the first 48 rows and filter the data to only show the device with serial number RTU-210156. Make sure the y-axis scales between 0 by including `ylim(0, 100)` in your call of the ggplot function.

![Figure 12.3: ](resources/session7/flow-profile.png)

D> What story does the graph in figure 12.3 tell?

### Top Ten Users
The previous section visualises flow for one specific service. While this is interesting, in any real-life situation, you need to review the flow of thousands or even millions of services. Also, viewing data in detail from individual services should only be done for operational reasons because this information is privacy-sensitive.

Instead then reviewing the individual performance of each service, we need to look at anomalies. The `group_by()` function summarises data for groups of services over time.

This following example shows how to find the top ten users in the available data. The code groups the flow data by device ID and summarises the consumption for each service by summing the flows and converting this to cubic meters (or kilolitres for Australians).

To find the top ten users, we first arrange the data by consumption. By default, the `arrange()` function sorts data in ascending order (low to high). Adding the `desc()` function sorts the data in descending order.

The last step uses the `top_n()` function to select the top ten users.

{format: r, line-numbers: false}
```
top10 <- flow %>%
    group_by(DeviceID) %>%
    summarise(Consumption = sum(Flow) / 1000) %>%
    arrange(desc(Consumption)) %>%
    top_n(10)
```

Q> Reverse-engineer this code to understand how it works. Use the `slice()` function to achieve the same result. Visualise the results to replicate Figure 12.4.

![Figure 12.4: Top ten water users.](resources/session7/top10.png)




## Further Study
https://r4ds.had.co.nz/dates-and-times.html
