---
output: html_document
---
## Time for my 2nd serious attempt at crunching UK Police data. Once again I'm armed with my new methodology from the excellent [Rladiessydney website](https://rladiessydney.org/courses/ryouwithme/), combined with Paul Bradshaw's tutorials and plenty of Googling to problem solve as required. I've also tapped into Code Academy recently and other tutorials online. 

I have created a github walkthrough to document my work. 

## Today I need to repeat one simple task: import and bind multiple csv files (lots of them: 3 years worth) without needing to do so manually. 

I know how to to do that thanks to my recent studies & Jonathan NG's magic recipe. Here we go:

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
mydir = "CambsPolicedata2"
myfiles = list.files(path=mydir, pattern="*.csv", full.names=TRUE)
myfiles
```
That worked really well. I'm impressed. Next step:

Use command line to join all the CSVs together...

I used this script to do so:

echo "combining all the data"
cat *.csv > CombinedCambs_streetdata2017-20raw.csv
echo "all done!"

I'll now skip lines 50-56


## Time to create an object, clean things up, and pipe some magic.
```{r}
cleanpolicedata <- dir("~/Desktop/CambsPolicedata2", full.names = T) %>% map_df(read_csv) %>%
  clean_names()
```
## Quick check of the data

```{r}
glimpse(cleanpolicedata)
head(cleanpolicedata)
tail(cleanpolicedata)
```
I now need to remove a column called "context" which has no useful data.
```{r}
cleanpolicedata$context <- NULL
```
[I used this suggestion for a single column](https://stackoverflow.com/questions/4605206/drop-data-frame-columns-by-name)

But!

I still have a useless column filled with N/A ... I forget how to do this... let's select our way out of it!... 

## Selection time

I want to filter the data and find how many crimes simply aren't being solved since September 2017 so I can then do some more useful statistical analysis / dataviz later. Here we go! I'll do two operations for this. One for when no suspect can be identified.

```{r}
selectdata <- cleanpolicedata %>% 
  select("crime_type","last_outcome_category")
```
## Now to organise that selection

```{r}
orderedselectdata <- selectdata %>%
  arrange(desc(crime_type))
```

## Filtering time (Subset)
```{r}
nosuspect <- subset(orderedselectdata,orderedselectdata$last_outcome_category == "Investigation complete; no suspect identified")
```
Boom. Now for unable to prosecute... 
```{r}
unableprosecute <- subset(orderedselectdata,orderedselectdata$last_outcome_category == "Unable to prosecute suspect")
```
## Now to bind these 2 dataframes together
```{r}
unsolvedcrimes <- rbind(nosuspect,unableprosecute)
tail(unsolvedcrimes)
```
This is beautiful. It worked! Now to write... 
```{r}
write_csv(unsolvedcrimes,"cambsunsolvedcrimes_street2017-20.csv") 
```
Boom!!! I have succeeded. Next step... learn more about this data...
```{r}
glimpse(unsolvedcrimes)
summary(unsolvedcrimes)
```
## Okay so I have crunched the "outcomes" dataset. Now what percentage of the original dataset (cleanpolicedata) does this represent? Easy calculation... Let's use the native "count" function to get a sense of the different outcomes here. 

```{r}
crimetypesummary <- unsolvedcrimes %>%
  count("crime_type") %>%
  arrange(desc(freq))
```
Success! 

```{r}
outcomesummary <- count(unsolvedcrimes$last_outcome_category)
```

## Another important data angle: Counting crimes and outcomes by type... 

Outcomes: 

```{r}
cleanoutcomecount <- count(cleanpolicedata$last_outcome_category) %>%
  arrange(desc(freq))
cleanoutcomecount
```

Types:

```{r}
orderedcrimetype <- count(cleanpolicedata$crime_type) %>%
   arrange(desc(freq))
```

Time to write to csv files for these angles!

#outcomes 

```{r}
write_csv(cleanoutcomecount,"cambsoutcomesstreetclean2017_20.csv")
```

#types

```{r}
write_csv(orderedcrimetype, "cambscrimesstreetclean2017_20.csv")
```

Let's experiment...

## Time to pipe some more magic

```{r}
orderedunsolvedcrime <- count(unsolvedcrimes)
```

```{r}
unsolvedcrimesummary <- orderedunsolvedcrime %>% 
 arrange(desc(freq))
```
It worked! Bit messy, but I can use this... in hindsight I could have simplified this process quite a bit. But I'm getting used to R slowly! 

## Time to back up my important datasets

```{r}
write_csv(unsolvedcrimesummary, "cambscrimesstreetsummary2017_20.csv")
```

## Lastly, I'm going to create a csv of the master dataset for future reference. 

```{r}
write_csv(cleanpolicedata, "cleanpolicedatastreet2017_20.csv")
```

Let's look at this in Excel / Sheets / Datawrapper etc and do some dataviz!

## Time for another angle: what if we zoom in on "Cambridge" as opposed to Cambs? Which crimes have had which outcomes? 

```{r}
selectdata2 <- cleanpolicedata %>% 
  select("crime_type","lsoa_name","falls_within","last_outcome_category")
```
## Now to examine that new selection

```{r}
head(selectdata)
tail(selectdata)
skim(selectdata)
summary(selectdata)
```
## I now need to filter by Cambridge only... 

How do I do this?! Let's try adapting the sorcery I used on a parallel dataset. Admittedly this is a bit longwinded but at least it's thorough. 
  
```{r}
count(selectdata2$lsoa_name == "Cambridge ")
filterdata <- filter(selectdata2,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |lsoa_name == "Cambridge 013E")
```
## Let's count total reported crime in Cambridge to generate percentages later on

```{r}
orderedcrimetypecambridge <- count(filterdata$crime_type) %>%
   arrange(desc(freq))
orderedcrimetypecambridge
```
## Let's write this to a CSV for later data analysis

```{r}
write_csv(orderedcrimetypecambridge,"cambridgetotalcrime_by_type2017-20.csv")
```


Boom. Now for nosuspect/unable to prosecute... 
```{r}
nosuspectcambridge <- subset(filterdata, filterdata$last_outcome_category == "Investigation complete; no suspect identified")
unableprosecutecambridge <- subset(filterdata,filterdata$last_outcome_category == "Unable to prosecute suspect")
```
## Now to bind these 2 dataframes together
```{r}
unsolvedcrimescambridge <- rbind(nosuspectcambridge,unableprosecutecambridge)
tail(unsolvedcrimes)
```
## Time to count the numbers

```{r}
orderedunsolvedcrimescambridge <- count(unsolvedcrimescambridge$crime_type) %>%
  arrange(desc(freq))
orderedunsolvedcrimescambridge
```
This is beautiful. It worked! Now to write... 
```{r}
write_csv(orderedunsolvedcrimescambridge,"cambridgesunsolvedcrimes_street2017-20.csv") 
```

## Next let's see what the most common outcomes were just for Cambridge

```{r}
orderedoutcomecountcambridge <- count(filterdata$last_outcome_category) %>%
   arrange(desc(freq))
orderedoutcomecountcambridge
tail(orderedoutcomecountcambridge)
```
## As ever, let's save this to a CSV

```{r}
write_csv(orderedoutcomecountcambridge,"cambridgeoutcomecountstreet2017-20.csv")
```

## Let's find location data for a map...

```{r}
locationdata <- cleanpolicedata %>%
  select(month,lsoa_name,location,latitude,longitude,crime_type,last_outcome_category) 
```
Let's see which areas had the most reported crimes... 

```{r}
count(locationdata$lsoa_name) %>%
  arrange(desc(freq))
```
This confirms that I need to focus on "Cambridge 007G"

```{r}
Cambridge007G_locationdata <- filter(locationdata,lsoa_name == "Cambridge 007G")
```

Now to filter by month... 

```{r}
Cambridge007G_Oct <- filter(Cambridge007G_locationdata,month == "2020-10")
```

There are 219 reported crimes so I'll count the types and select a balance of each for my map...

```{r}
Cambridge_007G_Octcrimetype <- count(Cambridge007G_Oct$crime_type) %>%
  arrange(desc(freq))
```

Okay what I shall try to do, is calculate a percentage for each type of crime and then reduce the dataset for Oct to 100 markers which reflects that... I'll do this in Sheets using the CSV I generated.

```{r}
write_csv(Cambridge007G_Oct,"Cambridge007G_Crimetype_Oct20.csv")
```

## Let's try one more angle for mapping... location data filtered by all "Cambridge" LSOAs

```{r}
locationdata_Cambridge <- filter(locationdata,lsoa_name == "Cambridge 001A" | lsoa_name == "Cambridge 001B" | lsoa_name == "Cambridge 001C" | lsoa_name == "Cambridge 001D" | lsoa_name == "Cambridge 001E"| lsoa_name == "Cambridge 001F" |lsoa_name == "Cambridge 002A" | lsoa_name == "Cambridge 002B" | lsoa_name == "Cambridge 002C" |lsoa_name == "Cambridge 002D" | lsoa_name == "Cambridge 002E" | lsoa_name == "Cambridge 002F" |lsoa_name == "Cambridge 003B" | lsoa_name == "Cambridge 003C" | lsoa_name == "Cambridge 003D" | lsoa_name == "Cambridge 003E" | lsoa_name == "Cambridge 003F"| lsoa_name == "Cambridge 003G" | lsoa_name == "Cambridge 004A" | lsoa_name == "Cambridge 004B" | lsoa_name == "Cambridge 004C" | lsoa_name == "Cambridge 004D" | lsoa_name == "Cambridge 004E" | lsoa_name == "Cambridge 005A" | lsoa_name == "Cambridge 005B" | lsoa_name == "Cambridge 005C" | lsoa_name == "Cambridge 005D" | lsoa_name == "Cambridge 006A" | lsoa_name == "Cambridge 006B" | lsoa_name == "Cambridge 006C" | lsoa_name == "Cambridge 006D" | lsoa_name == "Cambridge 006E" | lsoa_name == "Cambridge 006F" | lsoa_name == "Cambridge 007C" | lsoa_name == "Cambridge 007D" | lsoa_name == "Cambridge 007E" | lsoa_name == "Cambridge 007F" | lsoa_name == "Cambridge 007G" |lsoa_name == "Cambridge 008A" | lsoa_name == "Cambridge 008B" | lsoa_name == "Cambridge 008C" |lsoa_name == "Cambridge 008D" | lsoa_name == "Cambridge 008E" | lsoa_name == "Cambridge 009A" |lsoa_name == "Cambridge 009B" | lsoa_name == "Cambridge 009C" | lsoa_name == "Cambridge 009D" |lsoa_name == "Cambridge 009E" | lsoa_name == "Cambridge 010A" | lsoa_name == "Cambridge 010B" | lsoa_name == "Cambridge 010C" | lsoa_name == "Cambridge 010D" | lsoa_name == "Cambridge 010E" |lsoa_name == "Cambridge 011A" | lsoa_name == "Cambridge 011B" | lsoa_name == "Cambridge 011C" |lsoa_name == "Cambridge 011D" | lsoa_name == "Cambridge 011E" | lsoa_name == "Cambridge 011F" |lsoa_name == "Cambridge 012A" | lsoa_name == "Cambridge 012B" | lsoa_name == "Cambridge 012D" |lsoa_name == "Cambridge 012E" | lsoa_name == "Cambridge 012F" | lsoa_name == "Cambridge 013A" |lsoa_name == "Cambridge 013B" | lsoa_name == "Cambridge 013C" | lsoa_name == "Cambridge 013D" |lsoa_name == "Cambridge 013E")
```

## Now by the most recent month... Oct 2020

```{r}
Cambridgelocationdata_Oct <- filter(locationdata_Cambridge,month == "2020-10")
```
Where did most crime take place then?

```{r}
Cambridgelocationdatacount_Oct <- Cambridgelocationdata_Oct %>%
  count("lsoa_name") %>%
  arrange(desc(freq))
```

Interesting. Let's save and visualize... 

```{r}
write_csv(Cambridgelocationdata_Oct,"Cambridgelocationdata_Oct2020.csv")
```

M



