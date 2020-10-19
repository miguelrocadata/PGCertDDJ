## Working through some tutorials 
I found [these walkthroughs helpful](https://rladiessydney.org/courses/ryouwithme/01-basicbasics-2/)
First let's install tidyverse:
```{r}
install.packages("tidyverse")
```
Looks like I did this earlier on another project. Now let's try one called "here":
```{r}
install.packages("here")
```
Done. Time to load packages. I also had to install another tool called "skimr" further down in my notebook. I think I'll install & load all packages at the top of each script I write in future. I have moved some of these library commands up to the top of my script to make for good practice in future. 
```{r}
library(tidyverse)
library(here)
library(skimr)
library(janitor)
```
## Reading data time: Syndey beaches!
```{r}
beaches <- read_csv(here("sydneybeaches.csv"))
```
## exploring the data
```{r}
view(beaches)

dim(beaches)

str(beaches)
```
This is fantastic. There's a function called "glimpse" too!
```{r}
glimpse(beaches)
```
Also as we learnt elsewhere... heads or tails? 

```{r}
head(beaches)
tail(beaches)
```
Also the good old summary... 
```{r}
summary(beaches)
```
## New toys...
```{r}
install.packages("skimr")
```
Now let's skim read the data!
```{r}
skim(beaches)
```
## Going to install some extra toys... 
```{r}
install.packages("readxl")
install.packages("haven")
install.packages("googlesheets")
install.packages("datapasta")
```
## Tidying columns 
```{r}
select_all(beaches, toupper)
```
That worked. Now let's change them back!
```{r}
select_all(beaches, tolower)
```
## I need a janitor! (package)
```{r}
install.packages("janitor")
```
Great. Remember to load it!
```{r}
library(janitor)
```
Now to clean up my act.
```{r}
clean_names(beaches)
```
Crucial point made in the tutorials: R doesn't change the dataset when you use functions like clean_names etc. It just produces a result from the data... to change the data you have to assign a new object! E.g. 
```{r}
cleanbeaches <- clean_names(beaches)
```
This has now produced a new object in my Environment window - with the new column names cleaned up by the Janitor package. Really important to remember this point about how R works! 
## Time to rename the awkward "enteroccoci" column
Remember this point she made about the syntax of the code... 
## for rename use newname = oldname
```{r}
rename(cleanbeaches, beachbugs = enterococci_cfu_100ml)
```
Wow this actually worked really well!! I'm amazed.
I'll now check the columns using the names function. 
```{r}
names(cleanbeaches)
```

So I can see that this rename function hasn't overwritten or changed the original data... so let's do that by rewriting the object by simply adding an assignment symbol to the command above. 
```{r}
cleanbeaches <- rename(cleanbeaches, beachbugs = enterococci_cfu_100ml)
```
Time to check... 
```{r}
names(cleanbeaches)
```
## Time to select a subset of data, in this case columns
```{r}
select(cleanbeaches, council, site, beachbugs)
```
Wow this works SO well. I'm feeling empowered! :)
Now I'm going to put these 3 columns back into context using a command called "everything" after the original function. Here we go... 
```{r}
select(cleanbeaches, council, site, beachbugs, everything())
```
So now I've reordered the dataframe with my 3 selected columns at the front. Bliss. Now for a new, powerful tool to string lots of operations together to perform multiple functions...
## Pipe %>%
First I'm going to clean my old "environment", in the top right window, to start over, as it were. I will then run my original "beaches <- read_csv" code to reimport the data I want into an object.

First I'm going to "pipe" the original data into a new object using a series of operations. This will essentially do all of the previous operations I did separately, in one go. Then write the data to a csv file.
```{r}
cleanbeaches <- beaches %>%
  clean_names() %>% 
  rename(beachbugs = enterococci_cfu_100ml) 
  write_csv(cleanbeaches, "cleanbeaches.csv")
```
Success! I now have a new csv file which contains my cleaned up & selected data. Now I'm going to try it on a new dataset.

## Filtering, arranging, grouping by, and summarising data
First we go back over previous steps: load packages, and redo object assigning... and tweak the **pipe** code to utilise all of the data. I'll try this earlier on in the code as before... lines 118-122 are from the latest tutorial.

## Which beach has the most extreme level of bugs? Piping time! 
```{r}
cleanbeaches %>% arrange(desc(beachbugs))
```
I'll now create a new object to help me see the important data more clearly... and also use a simpler way of arranging in descending order which is a '-' symbol:
```{r}
worstbugs <- cleanbeaches %>% arrange(-beachbugs)
```
(I'm not entirely sure why the tutor got us to do this but it's all practice)
Now for a new subset of the data...
```{r}
cleanbeaches %>% filter(site == "Coogee Beach") %>%
  arrange(-beachbugs)
```
```{r}
worstcoogee <- cleanbeaches %>% 
  filter(site == "Coogee Beach") %>%
  arrange(-beachbugs)
```
## Let's compare max bug values across beaches

So this introduced a new idea: '%in%' which allows us to filter by 'instances of'... in other words either/or... so now R looks for both Coogee & Bondi. We also use 'c' after this to create a list of sorts...  I'll then also arrange this in descending order as above...

```{r}
coogee_bondi <- cleanbeaches %>%
  filter(site %in% c("Coogee Beach", "Bondi Beach")) %>%
arrange(-beachbugs)
```
Now to summarise the data... This introduces 'group_by', max/mean/median/sd, and also how to deal with N/A values in the data by using 'na.rm = TRUE'.
```{r}
cleanbeaches %>%
  filter(site %in% c("Coogee Beach", "Bondi Beach")) %>%
group_by(site) %>%
  summarise(maxbug = max(beachbugs,na.rm = TRUE),
            medianbugs = median(beachbugs, na.rm = TRUE),
            meanbugs = mean(beachbugs,na.rm = TRUE),
            sdbugs = sd(beachbugs,na.rm = TRUE))
```
We can also remove the filter or group_by functions and get a different look at the data... 
## No filter
```{r}
cleanbeaches %>%
group_by(site) %>%
  summarise(maxbug = max(beachbugs,na.rm = TRUE),
            medianbugs = median(beachbugs, na.rm = TRUE),
            meanbugs = mean(beachbugs,na.rm = TRUE),
            sdbugs = sd(beachbugs,na.rm = TRUE))
```
## No group_by
```{r}
cleanbeaches %>%
  summarise(maxbug = max(beachbugs,na.rm = TRUE),
            medianbugs = median(beachbugs, na.rm = TRUE),
            meanbugs = mean(beachbugs,na.rm = TRUE),
            sdbugs = sd(beachbugs,na.rm = TRUE))
```
Clearly the last option is not so useful here, but it could be depending on the data? 

## Let's compare councils

First let's use the 'distinct' function to see who is looking after what in our data...
```{r}
cleanbeaches %>% distinct(council)
```
Boom. Useful!
Let's compare councils:
```{r}
councilbysite <- cleanbeaches %>%
  group_by(council, site) %>%
  summarise(meanbugs = mean(beachbugs,na.rm = TRUE),
            medianbugs = median(beachbugs, na.rm = TRUE))
```
Cool. 




