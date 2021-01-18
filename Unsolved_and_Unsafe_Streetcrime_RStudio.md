---
output: html_document
---
## Time for my final attempt at crunching UK Police data for the time being. Once again I'm armed with my an expanding methodology encompassing the excellent [Rladiessydney website](https://rladiessydney.org/courses/ryouwithme/), combined with Paul Bradshaw's tutorials, and plenty of Googling to problem solve as required. I've also tapped into Code Academy recently and other tutorials online. 

I shall create a github walkthrough to document my work. 

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

### I originally did this by reusing a magical formula I discovered earlier, which utilises the **Pipe** approach. 

I started by setting my working directory to the folder which has all of the csv files in that I want to combine. I originally used [a magic recipe written by Jonathan NG](https://www.youtube.com/watch?v=HpWce0ovphY)

First step, look in my directory to see what's there (the folder my csv files are in) 
```{r}
setwd("~/Desktop")
mydir = "UK Street 3 Year"
myfiles = list.files(path=mydir, pattern="*.csv", full.names=TRUE)
myfiles
```
That worked really well. I'm impressed. I now have two options. Use Jonathan NG's R function to bind all of these CSV files together, or use command line. I'm going to try the latter as I think it's faster. A lot faster.

## To do this I'm using the following process in command line (terminal), which I've saved as a shell script:

pwd
cd (to my designated directory)
cat *.csv > myfilename.csv

Boom! No need for the "magic" below ... Although it might be useful for smaller projects. It's slow with this much data. Ignore lines 49-53 for now. I've circumvented it and used command line instead.

## (49-53) Time to create a dataframe, clean things up, and pipe some magic. *This is now redundant as I can do this much faster using command line.
```{r}
cleanpolicedata3yr <- dir("~/Desktop/UK Street 3 Year", full.names = T) %>% map_df(read_csv) %>%
  clean_names()
```
## (55) Time to import and clean my data

To do this I orignally experimented using the "distinct" function, which I found using [this tutorial](https://www.datasciencemadesimple.com/remove-duplicate-rows-r-using-dplyr-distinct-function/). However in hindsight I decided against this as it may adversely affect my data analysis. Apparent duplicate entries may be different crimes.

```{r}
Policedataclean <- read.csv("CombinedPoliceStreetdata2017-20raw.csv", stringsAsFactors = FALSE) %>%
  clean_names()
```

### Quick check of the data

```{r}
glimpse(Policedataclean)
head(Policedataclean)
tail(Policedataclean)
```
I now need to remove a column called "context" which has no useful data.
```{r}
Policedataclean$context <- NULL
```

## A quick analysis of the data reminded me that antisocial behaviour needs to be removed from reported crime when dealing with police crime data, as it never has a recorded outcome so it can affect the stats quite dramatically. Let's see how many crimes were recorded as antisocial behaviour and remove that figure from the totals. I'll also remove any blank spaces.

```{r}
Policedata_rm_antisocial <- Policedataclean %>%
  filter(Policedataclean$crime_type!="Anti-social behaviour",Policedataclean$crime_type!="Crime type")
```
## Selection time & organise that selection 

I want to filter the data and find how many crimes simply aren't being solved since October 2017 so I can then do some more useful statistical analysis / dataviz later. Here we go! 

```{r}
outcomedata <- Policedata_rm_antisocial %>% 
  select("month","falls_within","lsoa_name","crime_type","last_outcome_category") %>%
  filter("crime_type" != "","last_outcome_category" != "","lsoa_name" != "")
```
## Time for a simple count of outcomes and crime type to see what the selected data looks like
```{r}
  outcomecount <- count(outcomedata$last_outcome_category) %>%
  arrange(desc(freq))
```

```{r}
crimetypecount <- count(outcomedata$crime_type) %>%
  arrange(desc(freq))
```
### There seems to be some blank space. I'll try to remove this. To do this I'll create a dataframe to see what exactly it is.

```{r}
oddcrimedata <- outcomedata %>%
  filter(outcomedata$crime_type =="Crime type")
```

#Okay so I can see there are 1684 entries with no data except "crime type", which I've now discovered. I'll now filter this out. To do this I'll retrace my steps and add a filter to lines 77-80 above. I'll then re-run my initial sorting functions. 

It worked. Time to move on. Let's see what's going on with my category of "unsolved" crimes. What I really want to see compared to the outcomes dataset, is which crime types are most prevalent across the country, and which crime types are least likely to result in a suspect being charged. Here we go.

## Filtering time (Subset)
```{r}
nosuspect <- subset(outcomedata,outcomedata$last_outcome_category == "Investigation complete; no suspect identified")
```
Boom. Now for unable to prosecute... 
```{r}
unableprosecute <- subset(outcomedata,outcomedata$last_outcome_category == "Unable to prosecute suspect")
```
## Now to bind these 2 dataframes together
```{r}
unsolvedcrimes <- rbind(nosuspect,unableprosecute)
glimpse(unsolvedcrimes)
tail(unsolvedcrimes)
```
This is beautiful. It worked! Now to count again... 
```{r}
  unsolvedcrimetypecount <- count(unsolvedcrimes$crime_type) %>%
  arrange(desc(freq))
```
This is extremely useful. I can now compare the "unsolved count", with the "crime type" count and calculate percentages for each crime type. This will overlap with the outcomes dataset, but as that lacks crime type information, the quickest way to create a data snapshot is to use the "street" data for counting crime types, and the "outcomes" data for counting final outcomes, and be as transparent about this as possible in my story.

### I'm now going to create a csv of the master dataset (Policedataraw) for future reference. 

```{r}
write_csv(Policedataclean, "UKcleanpolicedatastreet2017_20_master.csv")
```

## Location and counting... 

More ambitious project: extract latitude and longitude data alongside freq of unsolved crimes to find national hotspots. So to do this I need to break it down... I've decided to retrace my steps and reuse some adapted code from my earlier steps to create a selection of the master dataset: 

```{r}
locationdata <- Policedata_rm_antisocial %>% 
  select("month","lsoa_name","falls_within","latitude","longitude","crime_type","location","last_outcome_category") 
  glimpse(locationdata)
```

## Counting time

```{r}
locationdatafreq <- count(locationdata,"lsoa_name") %>%
arrange(desc(freq))
locationdatafreq
```
## There's *alot* of missing data here. This could be for various reasons and may form another angle in my story. For now I'll count the locations I do have based on LSOA names.

## Filtering time: Subset 2 - Location of "unsolved" crimes

```{r}
nosuspectlocation <- subset(locationdata,locationdata$last_outcome_category == "Investigation complete; no suspect identified")
```
Boom. Now for unable to prosecute... 
```{r}
unableprosecutelocation <- subset(locationdata,locationdata$last_outcome_category == "Unable to prosecute suspect")
```
## Now to bind these 2 dataframes together
```{r}
unsolvedcrimeslocation <- rbind(nosuspectlocation,unableprosecutelocation)
glimpse(unsolvedcrimeslocation)
```

### Time to save this! 

```{r}
write_csv(unsolvedcrimeslocation,"UK_unsolvedcrimes_location_2017-20.csv")
```

## Counting time 

```{r}
unsolvedlocationcount <- count(unsolvedcrimeslocation,"lsoa_name") %>%
  arrange(desc(freq))
```

```{r}
write_csv(unsolvedlocationcount,"UKunsolvedcrimelocationcount.csv")
```
## Interesting. Missing LSOA data aside, I can now see the most frequent locations with the highest number of "unsolved" crimes. This will tie into my "outcomes" dataset beautifully.

### Leeds 111B, Leicester, Manchester, City of London, Birmingham, Brighton & Hove, Liverpool, and Cambridge all make the top ten!

And in the case of Manchester and Liverpool, there are two separate LSOA codes in the top 15 entries. It would make sense to investigate them as separate locations, and also combine them for a broader, city-wide view, and also to tie in with police forces in general.

### Time to experiment with the location data... (Current: 06/01/2021)

I decided to play around with grep & grepl to see if I could create an aggregated dataset which lacks the granular level data of specific LSOA codes. This would (in theory) allow me to quickly create choropleth maps of each type of unsolved crime.... 

### As Leeds 111B is the top location in my "unsolvedlocationcount", I decided to experiment on this first: 

```{r}
grep("Leeds", unsolvedcrimeslocation$lsoa_name) #This should return the positions of the matches "Leeds"
```
So far so good.
```{r}
grepl("Leeds", unsolvedcrimeslocation$lsoa_name) #This returns a T/F pattern in terms of whether it matches the pattern
```
Next, to create a new object for Leeds...
```{r}
Leedsmatch <- grepl("Leeds", unsolvedcrimeslocation$lsoa_name)
```

So far so good. Next step... 

```{r}
unsolvedcrimeslocation$leeds <- Leedsmatch #Add a new column to the dataframe showing true/false values 
```
### Filtering time
```{r}
leedsonly <- subset(unsolvedcrimeslocation,unsolvedcrimeslocation$leeds == T) 
```
That worked a treat. Now, let's play around ... 

### 1st I want to filter that dataset by Leeds 111B
```{r}
leeds_111B <- leedsonly %>%
  filter(leedsonly$lsoa_name == "Leeds 111B")
```
### Let's count the crimes in that area... 
```{r}
leeds111Bunsolvedcrimetypes <- leeds_111B %>%
  count("crime_type") %>%
  arrange(desc(freq))
```
### Leeds more broadly

```{r}
leedsunsolvedcrimes <- leedsonly %>%
  count("crime_type") %>%
  arrange(desc(freq))
```
### I can now see context within Leeds more broadly for Leeds111B - which means I can calculate percentages. 

### I can also extract location data (lats/longs) and split data into year groups to look at trends.

### This data will allow me to see aggregated data for Leeds as a whole, plus something closer to "granular" data for specific parts of the city. I can repeat this for other areas of interest and cross reference this with my "outcomes" dataset.

I've decided to do an unsolved crimes story which focuses on

a) key locations
b) crime types
c) unsolved crimes per capita, by police force, etc.
d) interesting correlations. E.g. Who & What is in each of the top ten areas? - 
businesses / transport hubs / universities / colleges / schools / etc. What are the demographics like in each area? 
e) people I can find to speak to from each area who might comment on the figures: ideally a mixture of "official" people (MPs/Police/Charities/statisticians/criminologist, etc), and "local" people on the street... 

To do this I'm going to repeat my filtering using grep & grepl to search through other key areas in my top ten list based on my "unsolvedlocationcount" object which I created.

## I will need to cross-reference this on police "outcomes" data to clarify outcomes. Mustn't forget that these two datasets are best used for different things.

## Next place is Manchester... although this is on the basis that there are two distinct LSOA codes in Manchester with high levels of unsolved crime... a cursory glance suggested combining these might be wise to get an overview of which cities are the real hotspots... This also tallied with my "outcomes" data analysis. 

### Manchester 
```{r}
grep("Manchester", unsolvedcrimeslocation$lsoa_name) #This should return the positions of the matches "Manchester"
```
So far so good.
```{r}
grepl("Manchester", unsolvedcrimeslocation$lsoa_name) #This returns a T/F pattern in terms of whether it matches the pattern
```
Next, to create a new d/f for Manchester
```{r}
Mancmatch <- grepl("Manchester", unsolvedcrimeslocation$lsoa_name)
```

So far so good. Next step... 

```{r}
unsolvedcrimeslocation$Manchester <- Mancmatch #Add a new column to the dataframe showing true/false values 
```
### Filtering time
```{r}
Manchesteronly <- subset(unsolvedcrimeslocation,unsolvedcrimeslocation$Manchester == T) 
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

### Let's count the crimes in those two areas... 
```{r}
Manc_054Ccount <- Manc_054C %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

```{r}
Manc_055Bcount <- Manc_055B %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

### Next step: Count all crimes in Manchester... 

```{r}
Manchestercount <- Manchesteronly %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

### Cambridge stats have come back into view as my "outcomes" data analysis highlighted Cambridgeshire constabulary. So, let's crunch Cambridge:

```{r}
grep("Cambridge", unsolvedcrimeslocation$lsoa_name) #This should return the positions of the matches "Cambridge"
```
So far so good.
```{r}
grepl("Cambridge", unsolvedcrimeslocation$lsoa_name) #This returns a T/F pattern in terms of whether it matches the pattern
```
Next, to create a new object for Cambridge
```{r}
Cam_match <- grepl("Cambridge", unsolvedcrimeslocation$lsoa_name)
```

So far so good. Next step... 

```{r}
unsolvedcrimeslocation$Cambridge <- Cam_match #Add a new column to the dataframe showing true/false values 
```
## Filtering time
```{r}
Cambridgeonly <- subset(unsolvedcrimeslocation,unsolvedcrimeslocation$Cambridge == T)
tail(Cambridgeonly)
```

Counting time again

```{r}
Cambridgecrimecount <- Cambridgeonly %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

## Time to split the data into years so I can examine trend lines... UPDATE: I also redid this including the "falls_within" category as this is more useful for measuring unsolved crimes with respect to police forces which leave blanks for some crimes. 

### To do this I used a useful [date-based tutorial](https://blog.exploratory.io/filter-with-date-function-ce8e84be680) which makes use of a package called "lubridate"

"Filter with Date function" 2017

```{r}

UK2017Streetcrime <- Policedata_rm_antisocial %>%
 select(month,lsoa_name,falls_within,crime_type,last_outcome_category) %>%
 filter(month <= "2017-12")

```
That worked. Next, 2018... 

```{r}

UK2018Streetcrime <- Policedata_rm_antisocial %>%
 select(month,lsoa_name,falls_within,crime_type,last_outcome_category,) %>%
 filter(month > "2017-12" & month <="2018-12")
```
Boom. Next step 2019: 

```{r}
UK2019Streetcrime <- Policedata_rm_antisocial %>%
 select(month,lsoa_name,falls_within,crime_type,last_outcome_category,) %>%
 filter(month > "2018-12" & month <="2019-12")

```
Next and final step for now, 2020:

```{r}
UK2020Streetcrime <- Policedata_rm_antisocial %>%
 select(month,lsoa_name,falls_within,crime_type,last_outcome_category,) %>%
 filter(month > "2019-12" & month <="2020-12")

```

## Now let's re-filter, count, and sort the data to look at trends year on year.

I probably could have done these two steps in one, but for now I'll split them.

### Filtering time: Subset 3 - year by year analysis 

Again, I probably could have done this earlier, for example by integrating it into the previous 2 steps, but for the sake of 

a) re-tracing my steps / providing a simple trail for others to follow... 

And

b) providing a separate set of processes for future reference... 

I'll do it again here... Shouldn't take too long... First crime types...

2017
```{r}
crimetypecount2017 <- count(UK2017Streetcrime$crime_type) %>%
  arrange(desc(freq))
```

2018
```{r}
crimetypecount2018 <- count(UK2018Streetcrime$crime_type) %>%
  arrange(desc(freq))
```

2019
```{r}
crimetypecount2019 <- count(UK2019Streetcrime$crime_type) %>%
  arrange(desc(freq))
```

2020
```{r}
crimetypecount2020 <- count(UK2020Streetcrime$crime_type) %>%
  arrange(desc(freq))
```

## Now to find unsolved crime trends year by year..

### No suspects

```{r}
Nosuspect2017 <- subset(UK2017Streetcrime,UK2017Streetcrime$last_outcome_category == "Investigation complete; no suspect identified")
```

2018:

```{r}
Nosuspect2018 <- subset(UK2018Streetcrime,UK2018Streetcrime$last_outcome_category == "Investigation complete; no suspect identified")
```

2019:

```{r}
Nosuspect2019 <- subset(UK2019Streetcrime,UK2019Streetcrime$last_outcome_category == "Investigation complete; no suspect identified")
```

2020:

```{r}
Nosuspect2020 <- subset(UK2020Streetcrime,UK2020Streetcrime$last_outcome_category == "Investigation complete; no suspect identified")
```

#Unable to prosecute

2017 

```{r}
Unabletoprosecute2017 <- subset(UK2017Streetcrime,UK2017Streetcrime$last_outcome_category == "Unable to prosecute suspect")
```
2018
```{r}
Unabletoprosecute2018 <- subset(UK2018Streetcrime,UK2018Streetcrime$last_outcome_category == "Unable to prosecute suspect")
```
2019
```{r}
Unabletoprosecute2019 <- subset(UK2019Streetcrime,UK2019Streetcrime$last_outcome_category == "Unable to prosecute suspect")
```
2020
```{r}
Unabletoprosecute2020 <- subset(UK2020Streetcrime,UK2020Streetcrime$last_outcome_category == "Unable to prosecute suspect")
```

### Now to bind these 2 dataframes together

2017
```{r}
Unsolved2017 <- rbind(Nosuspect2017,Unabletoprosecute2017)
```
2018
```{r}
Unsolved2018 <- rbind(Nosuspect2018,Unabletoprosecute2018)
```
2019
```{r}
Unsolved2019 <- rbind(Nosuspect2019,Unabletoprosecute2019)
```
2020
```{r}
Unsolved2020 <- rbind(Nosuspect2020,Unabletoprosecute2020)
```
Success! Now I can do some counting... 

### Counting and arranging time

2017

```{r}
Unsolvedcrimecount2017 <- count(Unsolved2017$crime_type) %>%
  arrange(desc(freq))
```

2018

```{r}
Unsolvedcrimecount2018 <- count(Unsolved2018$crime_type) %>%
  arrange(desc(freq))
```

2019

```{r}
Unsolvedcrimecount2019 <- count(Unsolved2019$crime_type) %>%
  arrange(desc(freq))
```

2020

```{r}
Unsolvedcrimecount2020 <- count(Unsolved2020$crime_type) %>%
  arrange(desc(freq))
```

## Now I can easily calculate year on year unsolved crime by location, from which I can find averages etc... 

2017

```{r}
locationcount2017 <- Unsolved2017 %>%
  count("lsoa_name") %>%
  arrange(desc(freq))
```

2018

```{r}
locationcount2018 <- Unsolved2018 %>%
  count("lsoa_name") %>%
  arrange(desc(freq))
```

2019

```{r}
locationcount2019 <- Unsolved2019 %>%
  count("lsoa_name") %>%
  arrange(desc(freq))
```

2020

```{r}
locationcount2020 <- Unsolved2020 %>%
  count("lsoa_name") %>%
  arrange(desc(freq))
```

That worked a treat. Now, let's play around ... I could have done this with one of the larger datasets but this is all helpful... 

## Let's see if we can determine which areas have blank LSOA names

```{r}
blanklsoa <- Policedata_rm_antisocial %>%
  filter(Policedata_rm_antisocial$lsoa_name == "")
tail(blanklsoa)
```

## Interesting! That's a lot of data... Let's count the "falls within" to get a sense of what's going on and where... 

```{r}
policeforces_missingdata <- blanklsoa %>%
  count("falls_within") %>%
  arrange(desc(freq))
policeforces_missingdata
```
## So lots of missing data here. Let's save this and see what the crime types are... 

```{r}
missing_data_crime_type <- blanklsoa %>%
  count("crime_type") %>%
  arrange(desc(freq))
missing_data_crime_type
```
## I've decided that I need to recompute parts of the other data subsets I've created to take account of this missing data - it might change the figures... especially for location data & ranking "forces" by crime types and outcomes data... I've mentioned this in the earlier part of this notebook retrospectively, and also taken it into account for my "outcomes" data analysis. 

To look at which police forces have the highest rates of unsolved crime etc I will use the "falls within" category for greater clarity.

## Recomputing time: Falls within... 

Let's find out which police forces are most prevalent...

```{r}
fallswithindata <- Policedata_rm_antisocial %>%
  select("month","lsoa_name","latitude","longitude","falls_within","crime_type","location","last_outcome_category") 
  glimpse(fallswithindata)
```

## Counting time

```{r}
fallswithindatafreq <- count(fallswithindata,"falls_within") %>%
arrange(desc(freq))
fallswithindatafreq
```
### Now let's explore this...

### I now want to rank police forces by their rate of unsolved crimes... First I'll create a subset of the data grouped by "falls within", "crime types" and "last outcome category.." 

```{r}
policeforce_subset <- Policedata_rm_antisocial %>%
  select(falls_within,crime_type,last_outcome_category)
```

Then I'll filter this subset by unsolved crimes... repeating a process I've done before in this analysis:

```{r}
policeforce_nosuspect <- policeforce_subset %>%
  filter(policeforce_subset$last_outcome_category =="Investigation complete; no suspect identified")
```

```{r}
policeforce_unabletoprosecute <- policeforce_subset %>%
  filter(policeforce_subset$last_outcome_category =="Unable to prosecute suspect")
```

```{r}
policeforceunsolved <- rbind(policeforce_nosuspect,policeforce_unabletoprosecute)
```

### Counting time... 

```{r}
policeforceunsolvedcount <- policeforceunsolved %>%
  count("falls_within") %>%
  arrange(desc(freq))
policeforceunsolvedcount
```
### Now we have a lot of figures... 

Let's try one last angle before ending data analysis in R for now...I want to know which police forces have the highest rates of unsolved violent crime & sexual offences, as these came out on top as the most common unsolved crime types, and which police forces left location data out of these crimes (missing data)...

### Police forces ranked by violent crimes ... 

```{r}
Violentcrime <- Policedata_rm_antisocial %>%
  filter(Policedata_rm_antisocial$crime_type == "Violence and sexual offences")
```

```{r}
Unsolvedviolentcrime <- policeforceunsolved %>%
  filter(policeforceunsolved$crime_type == "Violence and sexual offences")
```

#Counting time

```{r}
Violentcrimecount <- Violentcrime %>%
  count("falls_within") %>%
  arrange(desc(freq))
```

```{r}
Unsolvedviolentcrimecount <- Unsolvedviolentcrime %>%
  count("falls_within") %>%
  arrange(desc(freq))
```

### Done. I can now move on from my initial data analysis.

I can also use aspects of this method to crunch the police outcomes data.

### Time to save some files... 

```{r}
write_csv(policeforceunsolvedcount,"policeforceunsolvedcount_street_2017-20.csv")
write_csv(policeforceunsolved,"policeforceunsolved_data_2017-20.csv")
write_csv(policeforces_missingdata,"policeforces_missingdata_2017-20.csv")
write_csv(UK2017Streetcrime,"UK2017Streetcrime.csv")
write_csv(UK2018Streetcrime,"UK2018Streetcrime.csv")
write_csv(UK2019Streetcrime,"UK2019Streetcrime.csv")
write_csv(UK2020Streetcrime,"UK2020Streetcrime.csv")
write_csv(Unsolved2017,"Unsolvedcrimestreet2017.csv")
write_csv(Unsolved2018,"Unsolvedcrimestreet2018.csv")
write_csv(Unsolved2019,"Unsolvedcrimestreet2019.csv")
write_csv(Unsolved2020,"Unsolvedcrimestreet2020.csv")
write_csv(Unsolvedcrimecount2017,"Unsolvedcrimecount_street2017.csv")
write_csv(Unsolvedcrimecount2018,"Unsolvedcrimecount_street2018.csv")
write_csv(Unsolvedcrimecount2019,"Unsolvedcrimecount_street2019.csv")
write_csv(Unsolvedcrimecount2020,"Unsolvedcrimecount_street2020.csv")
write_csv(Unsolvedcrimes,"Unsolvedcrimestreet_2017-20.csv")
write_csv(unsolvedcrimeslocation,"Unsolvedcrimestreet_location2017-20.csv")
write_csv(unsolvedcrimetypecount,"Unsolvedcrimecount_street2017-20.csv")
write_csv(unsolvedlocationcount,"Unsolvedcrimelocationcount_street2017-20.csv")
write_csv(Unsolvedviolentcrime,"Unsolvedviolentcrime_street2017-20.csv")
write_csv(Unsolvedviolentcrimecount,"Unsolvedviolentcrimecount_street2017-20.csv")
write_csv(Violentcrime,"Violentcrime_street2017-20.csv")
write_csv(Violentcrimecount,"Violentcrimecount_street2017-20.csv")
write_csv(missing_data_crime_type,"Missingdata_street_crimetype2017-20.csv")
write_csv(outcomecount,"Outcomecount_street2017-20.csv")
write_csv(crimetypecount,"Crimetypecount_street2017-20.csv")
write_csv(crimetypecount2017,"Crimetypecount_street2017.csv")
write_csv(crimetypecount2018,"Crimetypecount_street2018.csv")
write_csv(crimetypecount2019,"Crimetypecount_street2019.csv")
write_csv(crimetypecount2020,"Crimetypecount_street2020.csv")
write_csv(fallswithindatafreq,"Policeforcecount_street2017-20.csv")
write_csv(leedsonly,"Leeds_streetcrime2017.20.csv")
write_csv(leeds_111B,"Leeds111B_streetcrime2017.20.csv")
write_csv(leeds111Bunsolvedcrimetypes,"Leeds111B_unsolvedcrimetypes2017.20.csv")
write_csv(locationdatafreq,"locationdatacount_streetcrime2017-20.csv")
write_csv(Manchesteronly,"Manchesterstreetcrime_2017-20.csv")
write_csv(Manchestercount,"Manchesterstreetcrimecount_2017-20.csv")
write_csv(Manc_054C,"Manc_054CStreetcrime_2017-20.csv")
write_csv(Manc_055B,"Manc_055BStreetcrime_2017-20.csv")
write_csv(Manc_054Ccount,"Manc_054CStreetcrimecount_2017-20.csv")
write_csv(Manc_055Bcount,"Manc_055BStreetcrimecount_2017-20.csv")

```

## Pause 06/01/21 11:53
-------------------------------------------------------------------------------------------------

## Unused cuts / edits to notebook / additional computation

```{r}
cambridge007G <- Policedata_rm_antisocial %>%
  filter(Policedataclean$lsoa_name == "Cambridge 007G")
```

Crime types

```{r}
cambridge007Gcrimetypecount <- cambridge007G %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

Outcome count

```{r}
cambridge007Goutcomecount <- cambridge007G %>%
  count("last_outcome_category") %>%
  arrange(desc(freq)) 
```

## Now I'm going to look at Leeds 111B unsolved crime just in 2019, as this was the year with the highest total number of unsolved crimes over the past 3 years.

```{r}
Leeds111B2019Unsolved <- leeds_111B %>%
 select(month,lsoa_name,latitude,longitude,location,crime_type,last_outcome_category) %>%
 filter(month <= "2019-12" & month > "2018-12")
```

```{r}
write_csv(Leeds111B2019Unsolved,"Leeds111B_2019_Unsolved.csv")
```

## Another angle has come to mind - Birmingham - As part of the city centre also popped up in the locationdatafreq count and West Midlands Police have featured in my article. 

```{r}
grep("Birmingham", unsolvedcrimeslocation$lsoa_name) #This should return the positions of the matches "Birmingham"
```
So far so good.
```{r}
grepl("Birmingham", unsolvedcrimeslocation$lsoa_name) #This returns a T/F pattern in terms of whether it matches the pattern
```
Next, to create a new object for Birmingham
```{r}
Brummatch <- grepl("Birmingham", unsolvedcrimeslocation$lsoa_name)
```

So far so good. Next step... 

```{r}
unsolvedcrimeslocation$Birmingham <- Brummatch #Add a new column to the dataframe showing true/false values 
```
### Filtering time
```{r}
Brumonly <- subset(unsolvedcrimeslocation,unsolvedcrimeslocation$Birmingham == T) 
```
That worked a treat. Now, let's play around ... 

### 1st I want to filter that dataset by Birmingham 138A
```{r}
Brum_138A <- Brumonly %>%
  filter(Brumonly$lsoa_name == "Birmingham 138A")
```
### Let's count the crimes in that area... 
```{r}
Brum_138Aunsolvedcrimetypes <- Brum_138A %>%
  count("crime_type") %>%
  arrange(desc(freq))
```
### And year by year for trend analysis... 

"Filter with Date function" 2017

```{r}

Brum_138A_2017 <- Brum_138A %>%
 select(month,lsoa_name,crime_type,last_outcome_category) %>%
 filter(month <= "2017-12")

```
That worked. Next, 2018... 

```{r}

Brum_138A_2018 <- Brum_138A %>%
 select(month,lsoa_name,crime_type,last_outcome_category) %>%
 filter(month > "2017-12" & month <="2018-12")

```
Boom. Next step 2019: 

```{r}
Brum_138A_2019 <- Brum_138A %>%
 select(month,lsoa_name,crime_type,last_outcome_category) %>%
 filter(month > "2018-12" & month <="2019-12")

```
Next and final step for now, 2020:

```{r}
Brum_138A_2020 <- Brum_138A %>%
 select(month,lsoa_name,crime_type,last_outcome_category) %>%
 filter(month > "2019-12" & month <="2020-12")

```
#Saving time... 

```{r}
write_csv(Brum_138A,"Birmingham_138A_street2017-20.csv")
write_csv(Brum_138A_2017,"Birmingham_138A_street2017.csv")
write_csv(Brum_138A_2018,"Birmingham_138A_street2018.csv")
write_csv(Brum_138A_2019,"Birmingham_138A_street2019.csv")
write_csv(Brum_138A_2020,"Birmingham_138A_street2020.csv")
write_csv(Brum_138Aunsolvedcrimetypes,"Birmingham_138A_crimetypecount2017-20.csv")
write_csv(Brumonly,"Birmingham_street2017-20.csv")
```

## Trend analysis for crime types / police forces by year

"Filter with Date function" Outcomes data

```{r}

Outcomedata_2017 <- outcomedata %>%
 filter(month <= "2017-12")

```
That worked. Next, 2018... 

```{r}

Outcomedata_2018 <- outcomedata %>%
 filter(month > "2017-12" & month <="2018-12")

```
Boom. Next step 2019: 

```{r}
Outcomedata_2019 <- outcomedata %>%
 filter(month > "2018-12" & month <="2019-12")

```
Next and final step for now, 2020:

```{r}
Outcomedata_2020 <- outcomedata %>%
 filter(month > "2019-12" & month <="2020-12")

```

#Selection and counting time

```{r}
crimetype_2017 <- Outcomedata_2017 %>%
  select(falls_within,crime_type)
```

```{r}
crimetype_2018 <- Outcomedata_2018 %>%
  select(falls_within,crime_type)
```

```{r}
crimetype_2019 <- Outcomedata_2019 %>%
  select(falls_within,crime_type)
```

```{r}
crimetype_2020 <- Outcomedata_2020 %>%
  select(falls_within,crime_type)
```

```{r}
policeforcecountoutcomes <- outcomedata %>%
  count("falls_within") %>%
  arrange(desc(freq))
```


### Saving time

```{r}
write_csv(Outcomedata_2017,"outcome_datastreet2017.csv")
write_csv(Outcomedata_2018,"outcome_datastreet2018.csv")
write_csv(Outcomedata_2019,"outcome_datastreet2019.csv")
write_csv(Outcomedata_2020,"outcome_datastreet2020.csv")
write_csv(crimetype_2017,"crimetype_2017street.csv")
write_csv(crimetype_2018,"crimetype_2018street.csv")
write_csv(crimetype_2019,"crimetype_2019street.csv")
write_csv(crimetype_2020,"crimetype_2020street.csv")
```

# The metropolitan police are another definite areas of interest. Let's explore them... 

```{r}
grep("Metropolitan Police Service", unsolvedcrimeslocation$falls_within) #This should return the positions of the matches for the Met
```
So far so good.
```{r}
grepl("Metropolitan Police Service", unsolvedcrimeslocation$falls_within) #This returns a T/F pattern in terms of whether it matches the pattern
```
Next, to create a new object for Birmingham
```{r}
Metonly <- grepl("Metropolitan Police Service", unsolvedcrimeslocation$falls_within)
```

```{r}
unsolvedcrimeslocation$Metpolice <- Metonly #Add a new column to the dataframe showing true/false values 
```
## Filtering time
```{r}
Metpoliceunsolved <- subset(unsolvedcrimeslocation,unsolvedcrimeslocation$Metpolice == T) 
```

Year by year

```{r}
Metunsolved2017 <- Metpoliceunsolved %>%
  filter(month <= "2017-12")
```

```{r}
Metunsolved2018 <- Metpoliceunsolved %>%
  filter(month > "2017-12" & month <="2018-12")
```

```{r}
Metunsolved2019 <- Metpoliceunsolved %>%
  filter(month > "2018-12" & month <="2019-12")
```

```{r}
Metunsolved2020 <- Metpoliceunsolved %>%
   filter(month > "2019-12" & month <="2020-12")
```

```{r}
Metunsolvedcrimetype2017 <- Metunsolved2017 %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

```{r}
Metunsolvedcrimetype2018 <- Metunsolved2018 %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

```{r}
Metunsolvedcrimetype2019 <- Metunsolved2019 %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

```{r}
Metunsolvedcrimetype2020 <- Metunsolved2020 %>%
  count("crime_type") %>%
  arrange(desc(freq))
```

### Saving time

```{r}
write_csv(Metpoliceunsolved,"Metpoliceunsolved_2017-20.csv")
write_csv(Metunsolved2017,"Metunsolved2017.csv")
write_csv(Metunsolved2018,"Metunsolved2018.csv")
write_csv(Metunsolved2019,"Metunsolved2019.csv")
write_csv(Metunsolved2020,"Metunsolved2020.csv")
write_csv(Metunsolvedcrimetype2017,"Metunsolvedcrimetype2017.csv")
write_csv(Metunsolvedcrimetype2018,"Metunsolvedcrimetype2018.csv")
write_csv(Metunsolvedcrimetype2019,"Metunsolvedcrimetype2019.csv")
write_csv(Metunsolvedcrimetype2020,"Metunsolvedcrimetype2020.csv")
```




