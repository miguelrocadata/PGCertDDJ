---
output: html_document
---
## Time for my 3rd serious attempt at crunching UK Police data. Once again I'm armed with my new methodology from the excellent [Rladiessydney website](https://rladiessydney.org/courses/ryouwithme/), combined with Paul's tutorials and plenty of Googling to problem solve as required. I've also tapped into Code Academy recently and other tutorials online. 

I will create a github walkthrough to document my work. 

Today I need to repeat one simple task: import and bind multiple csv files (lots of them: 3 years worth of police outcomes data) without needing to do so manually. I know how to to do that thanks to my recent studies & Jonathan NG's magic recipe. Here we go:

## Prep packages

First steps: Install & Load all of my potentially useful packages.
```{r}
library(tidyverse)
library(here)
library(skimr)
library(janitor)
library(readxl)
library(plyr)
library(readr)
library(lubridate)

```
## Next I will simply import multiple csv files and combine them into one.
This reuses a magical formula I discovered earlier, which utilises the **Pipe** approach. 

I start by setting my working directory to the folder which has all of the csv files in that I want to combine. To do this I'm going to try to use [a magic recipe written by Jonathan NG](https://www.youtube.com/watch?v=HpWce0ovphY)

First step, look in my directory to see what's there (the folder my csv files are in) 
```{r}
setwd("~/Desktop")
mydir = "CambsPolicedata"
myfiles = list.files(path=mydir, pattern="*.csv", full.names=TRUE)
myfiles
```
## Update: I'm going to use my new method for joining together multiple .csv files instead... 

I did this by running a saved shell script Paul Bradshaw taught us. I now have a new file. Time to read it... This is the script as plain text:

echo "combining all the data"
cat *.csv > CombinedCambsoutcomedata.csv
echo "all done!"

This was my previous (slower) method... I won't do that this time... So ignore lines 42-46.

## Time to create an object, clean things up, and pipe some magic.
```{r}
cleanpolicedata <- dir("~/Desktop/CambsPolicedata", full.names = T) %>% map_df(read_csv) %>%
  clean_names()
```
## Import new combined police outcome data

```{r}
cleanpolicedata <- read.csv("CombinedCambsoutcomedata.csv",stringsAsFactors = F) %>%
 clean_names()
```

## Quick check of the data

```{r}
glimpse(cleanpolicedata)
tail(cleanpolicedata)
```

## Time to save the master sheet for future reference! 

```{r}
write_csv(cleanpolicedata,"cleanCambspoliceoutcomes_2017-20.csv")
```

## Selection time

I want to filter the data and find how many crimes simply aren't being solved since September 2017 so I can then do some more useful statistical analysis / dataviz later. Here we go! I'll do two operations for this. One for when no suspect can be identified. I added the field "falls within" due to discovering a major discrepancy in UK police datasets when I looked at the national picture: Lots of missing data in the "lsoa name" category. So I decided to add "falls within" as this tends to be complete, and also gives a picture of crime by police force.

```{r}
selectdata <- cleanpolicedata %>% 
  select("lsoa_name","falls_within","outcome_type")
```
## Now to examine that new selection

```{r}
head(selectdata)
tail(selectdata)
skim(selectdata)
summary(selectdata)
```
## I now need to filter by Cambridge only... 

How do I do this?! I did it the long way... but it worked. I'll find a way to speed this up in future. 
```{r}
filterdata <- filter(selectdata,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | 
lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | 
lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |
lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |
lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |
lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |
lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |
lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |
lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |
lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |
lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |
lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |
lsoa_name == "Cambridge 013E")
```
## I'm going to add an additional check of this data to see if there are any blanks in the LSOA name field... 

```{r}
lsoacount <- count(cleanpolicedata,"lsoa_name") %>%
  arrange(desc(freq))

```
There are a lot. This is useful to remember for future reference when I'm creating location data related subsets. There's a lot of missing data here!

## Let's count the most common outcome categories for this filtered dataset

```{r}
Cambridgeoutcomecount <- count(filterdata$outcome_type) %>%
  arrange(desc(freq))
Cambridgeoutcomecount
```
## Let's make sure this count is saved

```{r}
write_csv(Cambridgeoutcomecount,"Cambridgeoutcomecount2017-20.csv")
```

Boom. Now for unable to prosecute... 
```{r}
nosuspect <- subset(filterdata, filterdata$outcome_type == "Investigation complete; no suspect identified")
unableprosecute <- subset(filterdata,filterdata$outcome_type == "Unable to prosecute suspect")
```
## Now to bind these 2 dataframes together
```{r}
unsolvedcrimes <- rbind(nosuspect,unableprosecute)
tail(unsolvedcrimes)
```
This is beautiful. It worked! Now to write... 
```{r}
write_csv(unsolvedcrimes,"cambridgesunsolvedcrimes_outcomes2017-20.csv") 
```
Boom!!! I have succeeded. Next step... learn more about this data...
```{r}
glimpse(unsolvedcrimes)
summary(unsolvedcrimes)
```
## Okay so I have crunched the "outcomes" dataset. Now what percentage of the original dataset (cleanpolicedata) does this represent? Easy calculation... Let's use the native "count" function to get a sense of the different outcomes here. 

Success! Next step...
```{r}
Cambsoutcomesummary <- count(unsolvedcrimes$outcome_type)
```

Let's experiment...
## Time to pipe some more magic
```{r}
orderedunsolvedcrime <- count(unsolvedcrimes)
```

```{r}
unsolvedcrimesummary <- orderedunsolvedcrime %>% arrange(desc(freq))
```
It worked! Bit messy, but I can use this... 
```{r}
write_csv(unsolvedcrimesummary, "cambridgescrimeoutcomesummary2017_20.csv")
```
Lastly, I'm going to create a csv of the master dataset for future reference. 
```{r}
write_csv(filterdata, "cambridgefilterdataoutcomes2017_20.csv")
```
Let's look at this... 

## Let's drill down and look at a year by year comparison

To do this I'm using the [following tutorial](https://blog.exploratory.io/filter-with-date-function-ce8e84be680)

## I've adapted the tutorial to match my dataset... 

"Filter with Date function"

```{r}

Cambridge2017outcomes <- cleanpolicedata %>%
 select(month,lsoa_name,falls_within,outcome_type) %>%
 filter(month <= "2017-12")
glimpse(Cambridge2017outcomes)
tail(Cambridge2017outcomes)

```
That worked. Next, 2018... 

```{r}

Cambridge2018outcomes <- cleanpolicedata %>%
 select(month,lsoa_name,falls_within,outcome_type) %>%
 filter(month > "2017-12" & month <="2018-12")
glimpse(Cambridge2018outcomes)
tail(Cambridge2018outcomes)
```
Boom. Next step 2019: 

```{r}
Cambridge2019outcomes <- cleanpolicedata %>%
 select(month,lsoa_name,falls_within,outcome_type) %>%
 filter(month > "2018-12" & month <="2019-12")
glimpse(Cambridge2019outcomes)
head(Cambridge2019outcomes)
tail(Cambridge2019outcomes)

```
Next and final step for now, 2020:

```{r}
Cambridge2020outcomes <- cleanpolicedata %>%
 select(month,lsoa_name,falls_within,outcome_type) %>%
 filter(month > "2019-12" & month <="2020-10")
glimpse(Cambridge2020outcomes)
head(Cambridge2020outcomes)
tail(Cambridge2020outcomes)

```

## Now let's filter, count, and sort the data by Cambridge and by year to look at trends.

I now need to filter by Cambridge only... 

How do I do this?!
```{r}
filterdata2017 <- filter(Cambridge2017outcomes,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | 
lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | 
lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |
lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |
lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |
lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |
lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |
lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |
lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |
lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |
lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |
lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |
lsoa_name == "Cambridge 013E")
```
## Let's count the most common outcome categories for this filtered dataset and repeat year on year

2017:

```{r}
Cambridgeoutcomecount2017 <- count(filterdata2017$outcome_type) %>%
  arrange(desc(freq))
Cambridgeoutcomecount2017
```
2018: 

```{r}
filterdata2018 <- filter(Cambridge2018outcomes,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | 
lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | 
lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |
lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |
lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |
lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |
lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |
lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |
lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |
lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |
lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |
lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |
lsoa_name == "Cambridge 013E")
```
```{r}
Cambridgeoutcomecount2018 <- count(filterdata2018$outcome_type) %>%
  arrange(desc(freq))
Cambridgeoutcomecount2018
```
2019: 

```{r}
filterdata2019 <- filter(Cambridge2019outcomes,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | 
lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | 
lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |
lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |
lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |
lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |
lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |
lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |
lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |
lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |
lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |
lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |
lsoa_name == "Cambridge 013E")
```
```{r}
Cambridgeoutcomecount2019 <- count(filterdata2019$outcome_type) %>%
  arrange(desc(freq))
Cambridgeoutcomecount2019
```
Last one: 2020

```{r}
filterdata2020 <- filter(Cambridge2020outcomes,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | 
lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | 
lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |
lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |
lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |
lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |
lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |
lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |
lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |
lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |
lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |
lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |
lsoa_name == "Cambridge 013E")
```
Time for the final countdown
```{r}
Cambridgeoutcomecount2020 <- count(filterdata2020$outcome_type) %>%
  arrange(desc(freq))
Cambridgeoutcomecount2020
```
## Let's save these counts for now. 

```{r}
write_csv(Cambridgeoutcomecount2020,"Cambridgeoutcomecount2020.csv")
write_csv(Cambridgeoutcomecount2019,"Cambridgeoutcomecount2019.csv")
write_csv(Cambridgeoutcomecount2018,"Cambridgeoutcomecount2018.csv")
write_csv(Cambridgeoutcomecount2017,"Cambridgeoutcomecount2017.csv")
```

## Let's create some location data to do some mapping.

```{r}
locationdata <- cleanpolicedata %>%
  select(lsoa_name,longitude,latitude,outcome_type)
```
Great, now let's count, filter, and prep for a map...

```{r}
locationoutcomecount <- locationdata %>%
  count() %>%
  arrange(desc(freq))
```
 
## Wow, so this reveals that an enormous number of crimes have no recorded location data. This is surely a "hidden crimes" story. 

However, let's now filter the location data we do have to create mapping data. Let's focus on "Cambridge"

```{r}
filteredlocationdata <- filter(locationoutcomecount,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | 
lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | 
lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |
lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |
lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |
lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |
lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |
lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |
lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |
lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |
lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |
lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |
lsoa_name == "Cambridge 013E")
```

# Let's save this

```{r}
write_csv(filteredlocationdata,"Cambridgelocationdata_outcomes2017-20.csv")
```
#My preferred free mapping software limits the number of markers you can use in a data viz. So I'm going to zoom in and look at 2020 crimes in Cambridge only...

```{r}
locationdata2020 <-  cleanpolicedata %>%
  select(month,lsoa_name,latitude,longitude,outcome_type) %>%
 filter(month > "2019-12" & month <="2020-10")
```

Now to filter by Cambridge
```{r}
Cambridgelocationdata2020 <- filter(locationdata2020,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | 
lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | 
lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |
lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |
lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |
lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |
lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |
lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |
lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |
lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |
lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |
lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |
lsoa_name == "Cambridge 013E")
```

Counting time... 

```{r}
Cambridgelocationdata2020_count <- Cambridgelocationdata2020 %>%
  count() %>%
  arrange(desc(freq))
```
More counting...

```{r}
Cambridge_lsoacount_2020 <- Cambridgelocationdata2020 %>%
  count("lsoa_name") %>%
  arrange(desc(freq))
```
## Just like in my national study, Cambridge 007G - city centre - is unsurprisingly the place with the highest number of reported crimes. Let's filter down to just this place... 

```{r}
Cambridge007G_locationdata <- filter(Cambridgelocationdata2020,lsoa_name == "Cambridge 007G")
```

Now for the most recent events... 

```{r}
Cambridge007G_locationdata_Oct <- filter(Cambridge007G_locationdata,month == "2020-10")
```
Okay I'll save this and remove the last 78 points... 

```{r}
write_csv(Cambridge007G_locationdata_Oct,"Cambridge007G_Oct2020.csv")
```
So far good. But I decided that repeating this process for crime types would be more interesting. Time to switch project. 
