---
output:
  html_document: default
---

## Time to crunch Police outcomes data. 

I shall create a github walkthrough to document my work. 

## Today I need to begin by repeating one simple task: import and bind multiple csv files (lots of them: 3 years worth) without needing to do so manually. 

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

To do this I'm using the following process in command line (terminal), which I saved as a shell script:

pwd
cd (to my designated directory)
cat *.csv > myfilename.csv

## Time to create an object, clean things up, and pipe some magic.

To do this I orignally experimented using the "distinct" function, which I found using [this tutorial](https://www.datasciencemadesimple.com/remove-duplicate-rows-r-using-dplyr-distinct-function/). However in hindsight I decided against this as it may adversely affect my data analysis. This is especially the case if, for example, there are what appear to be double entries with identical data because some of the fields have been left blank (e.g. crime ID / location / etc), but in fact these entries could represent separate crime records.

```{r}
Policedataclean <- read.csv("CombinedPoliceOutcomesdata2017-20raw.csv",stringsAsFactors =FALSE) %>%
  clean_names()
```

## Quick check of the data

```{r}
glimpse(Policedataclean)
head(Policedataclean)
tail(Policedataclean)
```

## Selection & organisation

I want to filter the data and find how many crimes simply aren't being solved since November 2017 until November 2020 so I can then do some more useful statistical analysis / dataviz later. Here we go! 

```{r}
outcomedata <- Policedataclean %>% 
  select("month","falls_within","lsoa_name","outcome_type") %>%
   filter(Policedataclean$outcome_type!="Outcome type",Policedataclean$falls_within!="Falls within")
```
## Time for a simple count of outcomes and police forces to see what some selected data looks like
```{r}
  outcomecount <- count(outcomedata$outcome_type) %>%
  arrange(desc(freq))
```

```{r}
policeforcecount <- count(outcomedata$falls_within) %>%
  arrange(desc(freq))
```

## Filtering time to narrow down "unsolved" crimes (Subset)
```{r}
nosuspect <- subset(outcomedata,outcomedata$outcome_type == "Investigation complete; no suspect identified")
```
Now for unable to prosecute... 
```{r}
unableprosecute <- subset(outcomedata,outcomedata$outcome_type == "Unable to prosecute suspect")
```

I decided to do these separately so that I can compare them to eachother and have them separated for future reference.

## Now to bind these 2 dataframes together
```{r}
unsolvedcrimes <- rbind(nosuspect,unableprosecute) %>% 
  filter(unsolvedcrimes$lsoa_name!="Lsoa name")
tail(unsolvedcrimes)
```
Now to count again... 
```{r}
unsolvedoutcomecount <- count(unsolvedcrimes$outcome_type) %>%
  arrange(desc(freq))
```
### By police force
```{r}
unsolved_bypoliceforce <- count(unsolvedcrimes$falls_within) %>%
  arrange(desc(freq))
```

### By Lsoa name

```{r}
unsolved_bylsoaname <- count(unsolvedcrimes$lsoa_name) %>%
  arrange(desc(freq))
```

### Now to find out how this compares to suspects being charged
```{r}
suspectcharged <- subset(outcomedata,outcomedata$outcome_type == "Suspect charged")
```
Let's try to count this by police force 
```{r}
suspectcharged_policeforcecount <- suspectcharged %>%
  count("falls_within") %>%
  arrange(desc(freq))
```
and then by lsoa name
```{r}
suspectcharged_lsoacount <- suspectcharged %>%
  count("lsoa_name") %>%
  arrange(desc(freq))
```

## I'm now going to create a csv of the master dataset (Policedataraw) for future reference. 

```{r}
write_csv(Policedataclean,"UKcleanpolicedataoutcomes2017_20_master.csv")
```
## Time to split the data into years so I can examine trend lines...

### To do this I used a useful [date-based tutorial](https://blog.exploratory.io/filter-with-date-function-ce8e84be680) which makes use of a package called "lubridate"

Filtered outcome data 2017

NB: I'm calling these dataframes "EnglandWales..." as they do not include data for Northern Ireland or Scotland.

```{r}

EnglandWales_2017outcomes <- Policedataclean %>%
 select(month,falls_within,outcome_type,lsoa_name) %>%
 filter(month <= "2017-12")

```
That worked. Next, 2018... 

```{r}

EnglandWales_2018outcomes <- Policedataclean %>%
 select(month,falls_within,outcome_type,lsoa_name) %>%
 filter(month > "2017-12" & month <="2018-12")
```
Boom. Next step 2019: 

```{r}
EnglandWales_2019outcomes <- Policedataclean %>%
 select(month,falls_within,outcome_type,lsoa_name) %>%
 filter(month > "2018-12" & month <="2019-12")

```
Next and final step for now, 2020:

```{r}
EnglandWales_2020outcomes <- Policedataclean %>%
 select(month,falls_within,outcome_type,lsoa_name) %>%
 filter(month > "2019-12" & month <="2020-12")

```

## Now let's re-filter, count, and sort the data to look at trends year on year.

I probably could have done these two steps in one, but for now I'll split them. This helps me to"

a) re-trace my steps / provide a simple trail for others to follow... 

And

b) provide a separate set of processes for future reference... 

I'll do it again here... Shouldn't take too long... 

```{r}
EnglandWales_2017nosuspect <- subset(EnglandWales_2017outcomes,EnglandWales_2017outcomes$outcome_type == "Investigation complete; no suspect identified")
```

Now to repeat with 2018:

```{r}
EnglandWales_2018nosuspect <- subset(EnglandWales_2018outcomes,EnglandWales_2018outcomes$outcome_type == "Investigation complete; no suspect identified")
```

2019:

```{r}
EnglandWales_2019nosuspect <- subset(EnglandWales_2019outcomes,EnglandWales_2019outcomes$outcome_type == "Investigation complete; no suspect identified")
```

2020:
```{r}
EnglandWales_2020nosuspect <- subset(EnglandWales_2020outcomes,EnglandWales_2020outcomes$outcome_type == "Investigation complete; no suspect identified")
```

## Boom. Now for unable to prosecute... 

2017 

```{r}
EnglandWales_2017unabletoprosecute <- subset(EnglandWales_2017outcomes,EnglandWales_2017outcomes$outcome_type == "Unable to prosecute suspect")
```
2018
```{r}
EnglandWales_2018unabletoprosecute <- subset(EnglandWales_2018outcomes,EnglandWales_2018outcomes$outcome_type == "Unable to prosecute suspect")
```
2019
```{r}
EnglandWales_2019unabletoprosecute <- subset(EnglandWales_2019outcomes,EnglandWales_2019outcomes$outcome_type == "Unable to prosecute suspect")
```
2020
```{r}
EnglandWales_2020unabletoprosecute <- subset(EnglandWales_2020outcomes,EnglandWales_2020outcomes$outcome_type == "Unable to prosecute suspect")
```

## Now to bind these 2 dataframes together

2017
```{r}
EnglandWales_2017Unsolved <- rbind(EnglandWales_2017nosuspect,EnglandWales_2017unabletoprosecute)
```
2018
```{r}
EnglandWales_2018Unsolved <- rbind(EnglandWales_2018nosuspect,EnglandWales_2018unabletoprosecute)
```
2019
```{r}
EnglandWales_2019Unsolved <- rbind(EnglandWales_2019nosuspect,EnglandWales_2019unabletoprosecute)
```
2020
```{r}
EnglandWales_2020Unsolved <- rbind(EnglandWales_2020nosuspect,EnglandWales_2020unabletoprosecute)
```
Success! Now I can do some counting... 

## Counting and arranging time

2017

```{r}
Unsolvedoutcomecount2017 <- count(EnglandWales_2017Unsolved$outcome_type) %>%
  arrange(desc(freq))
```

2018

```{r}
Unsolvedoutcomecount2018 <- count(EnglandWales_2018Unsolved$outcome_type) %>%
  arrange(desc(freq))
```

2019

```{r}
Unsolvedoutcomecount2019 <- count(EnglandWales_2019Unsolved$outcome_type) %>%
  arrange(desc(freq))
```

2020

```{r}
Unsolvedoutcomecount2020 <- count(EnglandWales_2020Unsolved$outcome_type) %>%
  arrange(desc(freq))
```
### Now by police force

2017

```{r}
Unsolved_policeforce2017 <- count(EnglandWales_2017Unsolved$falls_within) %>%
  arrange(desc(freq))
```

2018

```{r}
Unsolved_policeforce2018 <- count(EnglandWales_2018Unsolved$falls_within) %>%
  arrange(desc(freq))
```

2019

```{r}
Unsolved_policeforce2019 <- count(EnglandWales_2019Unsolved$falls_within) %>%
  arrange(desc(freq))
```

2020

```{r}
Unsolved_policeforce2020 <- count(EnglandWales_2020Unsolved$falls_within) %>%
  arrange(desc(freq))
```
# First I'll look at suspects charged year on year for comparison as this is straighforward

"Filter with Date function"

2017
```{r}
suspectscharged_2017 <- suspectcharged %>%
 filter(month <= "2017-12")
```
2018
```{r}
suspectscharged_2018 <- suspectcharged %>%
 filter(month > "2017-12" & month <="2018-12")
```
2019
```{r}
suspectscharged_2019 <- suspectcharged %>%
 filter(month > "2018-12" & month <="2019-12")
```
2020
```{r}
suspectscharged_2020 <- suspectcharged %>%
 filter(month > "2019-12" & month <="2020-12")
```
### Now for some counting 

Suspects charged 2017 by police force

```{r}
Suspectscharged_byforce2017 <- suspectscharged_2017 %>%
  count("falls_within") %>%
  arrange(desc(freq))
```

Suspects charged 2018 by police force

```{r}
Suspectscharged_byforce2018 <- suspectscharged_2018 %>%
  count("falls_within") %>%
  arrange(desc(freq))
```

Suspects charged 2019 by police force

```{r}
Suspectscharged_byforce2019 <- suspectscharged_2019 %>%
  count("falls_within") %>%
  arrange(desc(freq))
```

Suspects charged 2020 by police force

```{r}
Suspectscharged_byforce2020 <- suspectscharged_2020 %>%
  count("falls_within") %>%
  arrange(desc(freq))
```

##Total recorded crime by police force by year...

2017
```{r}
totalcrime2017 <- outcomedata %>%
  filter(month <= "2017-12")
```

```{r}
totalcrime_byforce2017 <- totalcrime2017 %>%
 count("falls_within") %>%
  arrange(desc(freq))
```
2018
```{r}
totalcrime2018 <- outcomedata %>%
  filter(month > "2017-12" & month <="2018-12")
```

```{r}
totalcrime_byforce2018 <- totalcrime2018 %>%
 count("falls_within") %>%
  arrange(desc(freq))
```
2019
```{r}
totalcrime2019 <- outcomedata %>%
  filter(month > "2018-12" & month <="2019-12")
```

```{r}
totalcrime_byforce2019 <- totalcrime2019 %>%
 count("falls_within") %>%
  arrange(desc(freq))
```
2020
```{r}
totalcrime2020 <- outcomedata %>%
  filter(month > "2019-12" & month <="2020-12")
```

```{r}
totalcrime_byforce2020 <- totalcrime2020 %>%
 count("falls_within") %>%
  arrange(desc(freq))
```

Done. Next ... 

## Time to save some files... 

```{r}
write_csv(unsolvedcrimes,"Unsolvedcrimes_outcomes2017-20.csv")
write_csv(Unsolved_policeforce2017,"Unsolved_policeforce2017.csv")
write_csv(Unsolved_policeforce2018,"Unsolved_policeforce2018.csv")
write_csv(Unsolved_policeforce2019,"Unsolved_policeforce2019.csv")
write_csv(Unsolved_policeforce2020,"Unsolved_policeforce2020.csv")
write_csv(suspectcharged,"Suspectscharged2017-20.csv")
write_csv(suspectcharged_lsoacount,"Suspectcharged_lsoa2017-20.csv")
write_csv(suspectcharged_policeforcecount,"Suspectchargedpoliceforce2017-20.csv")
write_csv(unsolved_bylsoaname,"Unsolvedoutcome_LSOA2017-20.csv")
write_csv(unsolved_bypoliceforce,"Unsolvedoutcomes_Policeforce2017-20.csv")
write_csv(policeforcecount,"Policeforcecount_outcomes2017-20.csv")
```

## Now it's time to experiment with location data... 

I decided to play around using two functions called grep & grepl to see if I could create an aggregated dataset which uses the granular level data of specific LSOA codes. This would (in theory) allow me to quickly create choropleth maps of hotspots for unsolved crime by region / police force. It will also provide me with much needed insight into where to focus my investigation for localised stories. 

To do this I decided to start with a simple count of LSOA names... 

## There is a lot of missing data here. As with the "street crime" data. This potentially gives rise to yet another story. I'll address this later, however. 

### As Leeds 111B is the top location in my "unsolvedbylsoaname" count, I decided to experiment on this first: 

```{r}
grep("Leeds", unsolvedcrimes$lsoa_name) #This should return the positions of the matches "Leeds"
```
So far so good.
```{r}
grepl("Leeds", unsolvedcrimes$lsoa_name) #This returns a T/F pattern in terms of whether it matches the pattern
```
Next, to create a new d/f for Leeds...
```{r}
Leedsmatch <- grepl("Leeds", unsolvedcrimes$lsoa_name)
```

So far so good. Next step... 

```{r}
unsolvedcrimes$leeds <- Leedsmatch #Add a new column to the dataframe showing true/false values 
```
## Filtering time
```{r}
leedsonly <- subset(unsolvedcrimes,unsolvedcrimes$leeds == T) 
```
That worked a treat. Now, let's play around ... 

## 1st I want to filter that dataset by Leeds 111B
```{r}
leeds_111B <- leedsonly %>%
  filter(leedsonly$lsoa_name == "Leeds 111B")
```
## Let's count the crimes in that area... 
```{r}
leeds111Bunsolvedcrimecount <- leeds_111B %>%
  count("outcome_type") %>%
  arrange(desc(freq))
```
## Next step: create a table

```{r}
leedsunsolvedcrimes <- leedsonly %>%
  count("outcome_type") %>%
  arrange(desc(freq))
```

### Saving time

```{r}
write_csv(leeds_111B,"Leeds111BUnsolved_2017-20.csv")
```

## *Latest 30/12/20 - This is now mirroring a parallel dataset I've created which looks at reported crime types... This would provide an interesting comparison - and also it begs the question: Which forces have the highest number of different "unsolved" crimes per capita? E.g. unable to prosecute vs no suspect indentified... 

In any case, I now have objects saved which allow me to see a broad range of aggregated data, plus data just for Leeds as a whole, plus something closer to "granular" data for a specific part of the city which is of interest.

As I prepare to write my crime article, I'm therefore looking at:

a) location
b) crime type
c) unsolved crimes per capita (to be computed)
d) interesting correlations. E.g. Who & What is in each of the top ten areas? - 
businesses / transport hubs / universities / colleges / schools / etc. What are the demographics like in each area? 
e) people I can find to speak to from each area who might comment on the figures: ideally a mixture of "official" people (MPs/Police/Charities/statisticians/criminologist, etc), and "local" people on the street... 

Let's press on with other locations of interest...What's great to see in a way, is that my "unsolved by lsoa name" dataset looks very similar to my parallel analysis of "street" crime.

## Next place of interest is Manchester... although this is on the basis that there are two distinct LSOA codes in Manchester with high levels of unsolved crime... a cursory glance suggested combining these might ultimately be a wise way of getting an overview of which cities are the real hotspots... 

### Manchester 
```{r}
grep("Manchester", unsolvedcrimes$lsoa_name) #This should return the positions of the matches "Manchester"
```
So far so good.
```{r}
grepl("Manchester", unsolvedcrimes$lsoa_name) #This returns a T/F pattern in terms of whether it matches the pattern
```
Next, to create a new object for Manchester
```{r}
Mancmatch <- grepl("Manchester", unsolvedcrimes$lsoa_name)
```

So far so good. Next step... 

```{r}
unsolvedcrimes$Manchester <- Mancmatch #Add a new column to the dataframe showing true/false values 
```
### Filtering time
```{r}
Manchesteronly <- subset(unsolvedcrimes,unsolvedcrimes$Manchester == T) 
```
That worked a treat. Now, let's play around ... I could have done this with one of the larger datasets but this is all helpful... 

### 1st I want to filter that dataset by Manchester 054C & 055B
```{r}
Manc_054C <- Manchesteronly %>%
  filter(Manchesteronly$lsoa_name == "Manchester 054C")
```

```{r}
Manc_055B <- Manchesteronly %>%
  filter(Manchesteronly$lsoa_name == "Manchester 055B")
```

### Let's count the outcomes in those two areas... 
```{r}
Manc_054Ccount <- Manc_054C %>%
  count("outcome_type") %>%
  arrange(desc(freq))
```

```{r}
Manc_055Bcount <- Manc_055B %>%
  count("outcome_type") %>%
  arrange(desc(freq))
```

## Next step: Count all crimes in Manchester... 

```{r}
Manchestercount <- Manchesteronly %>%
  count("outcome_type") %>%
  arrange(desc(freq))
```

### Cambridge stats have come back into view as I was working on a story related to these and Cambridge pops up in various ways throughout my data analysis. So, let's crunch Cambridge:

### Cambridge 
```{r}
grep("Cambridge", unsolvedcrimes$lsoa_name) #This should return the positions of the matches "Cambridge"
```
So far so good.
```{r}
grepl("Cambridge", unsolvedcrimes$lsoa_name) #This returns a T/F pattern in terms of whether it matches the pattern
```
Next, to create a new d/f for Cambridge
```{r}
Cammatch <- grepl("Cambridge", unsolvedcrimes$lsoa_name)
```

So far so good. Next step... 

```{r}
unsolvedcrimes$Cambridge <- Cammatch #Add a new column to the dataframe showing true/false values 
```
## Filtering time
```{r}
Cambridgeonly <- subset(unsolvedcrimes,unsolvedcrimes$Cambridge == T) 
```

### Let's see if we can fill out blank space in LSOA names to compare with the street crime dataset 

```{r}
blanklsoa <- Policedataclean %>%
  filter(Policedataclean$lsoa_name == "")
tail(blanklsoa)
```

### Interesting! That's a lot of data... Let's count the "falls within" to get a sense of what's going on and where... 

```{r}
policeforces_missingdata <- blanklsoa %>%
  count("falls_within") %>%
  arrange(desc(freq))
policeforces_missingdata
```
### So lots of missing data here. Let's save this and see what the outcome types are... 

```{r}
missing_data_outcome_type <- blanklsoa %>%
  count("outcome_type") %>%
  arrange(desc(freq))
missing_data_outcome_type
```

## Throughout my analysis I realised that I need to recompute parts of the other data subsets I've created to account for this "missing" data - it might change the figures... especially for location data & ranking "forces" by crime types and outcomes data... 

### Recomputed police force data

```{r}
fallswithindata <- Policedataclean %>%
  select("month","lsoa_name","latitude","longitude","falls_within","outcome_type","location") %>%
  filter(Policedataclean$falls_within!="Falls within")
  glimpse(fallswithindata)
```

### Counting time

```{r}
fallswithindatafreq <- count(fallswithindata,"falls_within") %>%
arrange(desc(freq))
fallswithindatafreq
```

### Saving time again.. 

```{r}
write_csv(blanklsoa,"BlankLSOA_outcomedata2017-20.csv")
write_csv(Cambridgeonly,"Cambridge_outcomesdata2017-20.csv")
write_csv(EnglandWales_2017nosuspect,"Nosuspect_outcomes2017.csv")
write_csv(EnglandWales_2018nosuspect,"Nosuspect_outcomes2018.csv")
write_csv(EnglandWales_2019nosuspect,"Nosuspect_outcomes2019.csv")
write_csv(EnglandWales_2020nosuspect,"Nosuspect_outcomes2020.csv")
write_csv(EnglandWales_2017unabletoprosecute,"Noprosecute_outcomes2017.csv")
write_csv(EnglandWales_2018unabletoprosecute,"Noprosecute_outcomes2018.csv")
write_csv(EnglandWales_2019unabletoprosecute,"Noprosecute_outcomes2019.csv")
write_csv(EnglandWales_2020unabletoprosecute,"Noprosecute_outcomes2020.csv")
write_csv(leeds_111B,"Leeds111B_outcomes2017-20.csv")
write_csv(leeds111Bunsolvedcrimecount,"Leeds111BUnsolved_outcomes2017-20.csv")
write_csv(leedsonly,"Leeds_outcomes2017-20.csv")
write_csv(Manc_054C,"Manc_054C_outcomes2017-20.csv")
write_csv(Manc_054Ccount,"Manc_054Ccount_outcomes2017-20.csv")
write_csv(Manc_055B,"Manc_055B_outcomes2017-20.csv")
write_csv(Manc_055Bcount,"Manc_055Bcount_outcomes2017-20.csv")
write_csv(Manchestercount,"Manchesteroutcomecount_2017-20.csv")
write_csv(Manchesteronly,"Manchester_outcomes2017-20.csv")
write_csv(missing_data_outcome_type,"Missingdata_outcomes2017-20.csv")
write_csv(outcomedata,"Outcomedata_2017-20.csv")
write_csv(outcomecount,"Outcomedatacount_2017-20.csv")
write_csv(policeforces_missingdata,"Missingpoliceforceoutcomedatacount_2017-20.csv")
write_csv(suspectcharged,"Suspectscharged_outcomes2017-20.csv")
write_csv(suspectscharged_2017,"Suspectscharged2017_outcomes.csv")
write_csv(suspectscharged_2018,"Suspectscharged2018_outcomes.csv")
write_csv(suspectscharged_2019,"Suspectscharged2019_outcomes.csv")
write_csv(suspectscharged_2020,"Suspectscharged2020_outcomes.csv")
write_csv(Suspectscharged_byforce2017,"Suspectscharged_byforce2017_outcomes.csv")
write_csv(Suspectscharged_byforce2018,"Suspectscharged_byforce2018_outcomes.csv")
write_csv(Suspectscharged_byforce2019,"Suspectscharged_byforce2019_outcomes.csv")
write_csv(Suspectscharged_byforce2020,"Suspectscharged_byforce2020_outcomes.csv")
write_csv(totalcrime_byforce2017,"Totalcrimebyforce_outcomes2017.csv")
write_csv(totalcrime_byforce2018,"Totalcrimebyforce_outcomes2018.csv")
write_csv(totalcrime_byforce2019,"Totalcrimebyforce_outcomes2019.csv")
write_csv(totalcrime_byforce2020,"Totalcrimebyforce_outcomes2020.csv")
write_csv(Unsolvedoutcomecount2017,"Unsolvedoutcomecount_2017.csv")
write_csv(Unsolvedoutcomecount2018,"Unsolvedoutcomecount_2018.csv")
write_csv(Unsolvedoutcomecount2019,"Unsolvedoutcomecount_2019.csv")
write_csv(Unsolvedoutcomecount2020,"Unsolvedoutcomecount_2020.csv")
```

#Preliminary data analysis complete. 06/01/2021
