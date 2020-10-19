## More practice with UK Police data ## 

This time I'm armed with my new methodology from the excellent [Rladiessydney website](https://rladiessydney.org/courses/ryouwithme/), 
combined with Paul's tutorials and plenty of Googling to problem solve as required.

I shall create a separate github walkthrough to document my work on the Sydney beaches data by R Ladies Sydney here.

Today I wanted to do one simple editorial task: *import and bind multiple csv files* (lots of them: 2 years worth of Cambridgeshire Police data) without needing to do so manually. 
This is so I could (for example) offer an editor a long view of crime stats in an area to give useful context to a story & do so very, very quickly. It could also reveal more important longer term trends which could illuminate a story.

I have found a way to do just that. 
Here we go:

### First steps: Install & Load all of my potentially useful packages: ###
```{r}
library(tidyverse)
library(here)
library(skimr)
library(janitor)
library(readxl)
library(plyr)
library(readr)

```
### Next I will make a list and import all of the files into my session: ### 

To do that I tried [this approach](https://datascienceplus.com/how-to-import-multiple-csv-files-simultaneously-in-r-and-create-a-data-frame/)

So, I have loaded my packages above. Then... tried his suggestion.

```{r}
setwd("~/Desktop")
mydir = "CambsPolicedata"
myfiles = list.files(path=mydir, pattern="*.csv", full.names=TRUE)
myfiles
```
That worked really well. I'm impressed. Next step:

## Time to create a dataset with these files.

```{r}
dat_csv = ldply(myfiles, read_csv)
dat_csv
```
Hmm, I ran into some inexplicable problems with this so time for option 2 ( I shall figure out what went wrong here another time):

The approach I'm now going to attempt is a magical **super-formula** I discovered earlier, which uses the *Pipe* method. See my other github page using the 
R Ladies Sydney tutorials for more details. I am learning a lot from these walkthroughs! For instance, I didn't know how to use *pipe* before, but now I do. 
Let's see if it helps.

## Time to import multiple csv files and combine them into one.

I started by setting my working directory to the folder which has all of the csv files in that I want to combine. 
Then I'm going to try to use [a magic recipe written by Jonathan NG](https://www.youtube.com/watch?v=HpWce0ovphY)
which should combine my csv files in <3 seconds... 

First step, look in my directory to see what's there (the folder my csv files are in), and then use the *map_df* function to import all of the csv files instantly. 
```{r}
dir("~/Desktop/CambsPolicedata", full.names = T) %>% map_df(read_csv)
```
Wow... that worked! IT ACTUALLY WORKED!!! 
Next step, turn them into an object. 
```{r}
Combinedpolicedata <- dir("~/Desktop/CambsPolicedata", full.names = T) %>% map_df(read_csv)
```
Interesting. It sort of worked but has added my Rmd title as a new column in the dataframe... ?!? Why? Let's delete that... 

[I used this suggestion for how to delete a single column](https://stackoverflow.com/questions/4605206/drop-data-frame-columns-by-name) before discovering that Paul had already mentioned 
this in his *10 more things to do in R tutorial*.
```{r}
Combinedpolicedata$`## More practice with UK Police data. This time I'm armed with my new method. First steps: Load packages.` <- NULL
```
I'll do the same with the column called "context", which is empty.
```{r}
Combinedpolicedata$Context <- NULL
```
Success! At last. Not sure what happened with my 1st effort to import multiple files, but I now have a recipe that works better. And a useful new "NULL" command. Now it's time to ...

## Interrogate my data using the Australian methods I've been learning! ##
```{r}
dim(Combinedpolicedata)

```
This allows me to see the dimensions of the dataset.
```{r}
summary(Combinedpolicedata)
```
As covered by Paul's tutorials & elsewhere. Very useful.
```{r}
str(Combinedpolicedata)
```
```{r}
glimpse(Combinedpolicedata)
```
Really handy tool! Glimpse presents a snapshot of the data in a helpful way which is different to summary / str / dim.
```{r}
head(Combinedpolicedata)
```
```{r}
names(Combinedpolicedata)
```
```{r}
cleanpolicedata <- clean_names(Combinedpolicedata)
names(cleanpolicedata)
```
```{r}
select(cleanpolicedata,lsoa_name,crime_type,last_outcome_category)
```
I'm winning! Must remember to clean my data using Janitor! 
Selection time and reordering time... 
```{r}
select(cleanpolicedata,lsoa_name,crime_type,last_outcome_category,everything())
```
Now I want to filter the data and find how many crimes simply aren't being solved since August 2018 so I can then do some more useful statistical analysis later. Here we go! I'll do two operations for this. One for when no suspect can be identified.
## Filtering time (Subset)
```{r}
unsolved_crimes_2yrs <- subset(cleanpolicedata,cleanpolicedata$last_outcome_category == "Investigation complete; no suspect identified")
```
Then find out which crimes are not being solved because suspects cannot be prosecuted. **NB** I should have named this first object "no suspect"... Can I change that? Let's look and find out... (Google)... Yes I can! Here we go:
```{r}
rm(unsolved_crimes_2yrs)
```
Now let's rerun the code with a more sensible name... 
```{r}
nosuspect <- subset(cleanpolicedata,cleanpolicedata$last_outcome_category == "Investigation complete; no suspect identified")
```
Boom. Now for unable to prosecute... 
```{r}
unableprosecute <- subset(cleanpolicedata,cleanpolicedata$last_outcome_category == "Unable to prosecute suspect")
```
Now to bind them together and write them to a csv... 
```{r}
unsolvedcrimes <- rbind(nosuspect,unableprosecute)
```
This is beautiful. It worked! Now to write... 
```{r}
write_csv(unsolvedcrimes,"cambsunsolvedcrimes2018-20.csv")
```
Boom!!! I have succeeded. Next step... learn more about this data... 
```{r}
glimpse(unsolvedcrimes)
summary(unsolvedcrimes)
```
I'm now going to try some filtering and arranging... using ideas from my Sydney beaches tutorial.
## Filtering by crime type and lsoa name... 
```{r}
violentsexburglary <- unsolvedcrimes %>% 
  filter(crime_type %in% c("Violence and sexual offences", "Burglary")) %>%
  group_by(lsoa_name)
```
Let's summarise this data... 
```{r}
glimpse(violentsexburglary)
summary(violentsexburglary)
```
distinctions? 
```{r}
count(violentsexburglary$crime_type)
```
```{r}
select(violentsexburglary, crime_type, lsoa_name, location) %>%
  arrange(desc(lsoa_name))
```
I'll try some ggplot now... 

## GGPLOT Maiden voyage
```{r}
install.packages("ggplot2")
library(ggplot2)
```
Now let's see what it can do...
```{r}
ggplot(violentsexburglary) +
  geom_bar(aes(x=crime_type))
```
It did produce a column chart which looks like this: 

[Simple column chart](https://github.com/miguelrocadata/PGCertDDJ/blob/Screenshots/Screenshot%202020-10-19%20at%2016.29.28.png)

## Percentages?
I'll try manually first... seems simple enough. I did a count of each type of offence before... so based on this subset:
```{r}
35560+9506
```
```{r}
Burglary = 9506/45066
Violence_and_sexual_offences = 35560/45066
```
Although this is only a fraction of all crimes... let's compute: 
```{r}
Violence_and_sexual_offences = 35560/112717
Burglary = 9506/112717
```
This is clunky but useful to know it can be done if you need a quick calculation. I think I'll stop there for now. Mission accomplished this week!
M


