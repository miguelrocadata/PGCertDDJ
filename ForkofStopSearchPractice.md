## Fork of using APIs to query police stop and search data and matching it to postcodes: I tried a different way of looking at this problem. It failed but the method could be useful for future reference. 

First, always prep the magic. 

```{r}
library(tidyverse)
library(here)
library(skimr)
library(janitor)
library(readxl)
library(plyr)
library(readr)

```
## Now let's get the big data.
```{r}
mydir = "Stopsearchdata"
list.files(path=mydir, pattern="*.csv", full.names=TRUE)

```
## Dataset creation using [Jonathan NG's sorcery](https://www.google.com/url?q=https://www.youtube.com/watch%3Fv%3DHpWce0ovphY&usd=2&usg=AFQjCNFPI3um0oRzRdeQrOVN8SrUpBWr3w):
```{r}
Oneyearstopsearch <- dir("Stopsearchdata", full.names = T) %>% map_df(read_csv)
```
Let's quickly look at the data...
```{r}
glimpse(Oneyearstopsearch)
```
## Wait! I have a magical idea... 
Maybe I could find a list of postcodes & lats/longs online? 

I googled it and found [this website](https://www.freemaptools.com/download-uk-postcode-lat-lng.htm) with [these files available to download in csv format](https://www.freemaptools.com/download/full-postcodes/ukpostcodes.zip)

Time to import them:
```{r}
read_csv("ukpostcodes.csv")
```
Time to make an object from them:
```{r}
ukpostcodes <- read_csv("ukpostcodes.csv")
```

### This file is large!! Too large for me to get my head around yet...  

Okay plan B: Let's narrow it down to Cambs... [I found some datasets with this data in](https://data.cambridgeshireinsight.org.uk/dataset/cambridgeshire-postcodes). 

So I have imported that csv file. Time to clean it up.

## Cleaning time: Janitor
First I cleaned up the CBArea object using "Janitor's 'clean_names' function". 

[See my other Github Markdown which explains where I got this idea from](https://github.com/miguelrocadata/PGCertDDJ/blob/master/RLadies%20Sydney%20Tutorials.md)

Now to check out the data... 
```{r}
glimpse(CBArea_0_0)
cleanCBArea <-clean_names(CBArea_0_0)
```
Next I shall try to select only what I need from the Cambridge postcode dataset I found. Then I shall pop into a new object called 'cambspostcodes'
```{r}
cambspostcodes <- select(cleanCBArea,postcode,latitude,longitude)
```
Now can I merge these two objects? Let's extract the lat/long from the cambsstopsearch data first
```{r}
camstopslatlong <- select(cleanstopsearch,latitude,longitude)
```
Success. Now I shall try to merge this with the cambspostcode dataset using 
[a new magical incantation I summoned from the ether at midnight over the corpse of a sacrifical goat (see answer no.7)](https://stackoverflow.com/questions/35202380/merging-data-frames-with-different-number-of-rows-and-different-columns): 
```{r}
merge(data.frame(cambspostcodes, row.names=NULL), data.frame(camstopslatlong, row.names=NULL), 
  by = 0, all = TRUE)[-1]
```
Okay I was kidding about the goat. Wow so it worked?!?! Now to make these into a new object called combined data & write to csv.
```{r}
combineddataset <- merge(data.frame(cambspostcodes, row.names=NULL), data.frame(camstopslatlong, row.names=NULL), 
  by = 0, all = TRUE)[-1]
write_csv(combineddataset,"cambsstopspostcodesdata.csv")
```
Okay that worked too. Now let's inspect it in Excel / R, and see if I can match the data... 
```{r}
summary(combineddataset)

```
A quick look at the data shows that the coordinates don't match which means I can't simply match stopsearch lat/long to cambs postcodes. This seems odd but it is what I have in the data. Maybe a bad data moment? Back to the drawing board... :) I shall attempt the API stuff this week, but I thought I'd try another way of problem solving this particular challenge to keep practising R.

M

