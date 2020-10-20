# Tues Oct 20 - Tutorial time again! I will master R... one day! :)

This is a continuation of my efforts to learn R using the amazing [R Ladies of Sydney tutorials: 
This is from the tutorial called "Clean it Up 3"](https://rladiessydney.org/courses/ryouwithme/02-cleanitup-3/)

I am basically following all of the steps in the tutorial to the letter. This github markdown is purely so that:

1. I can remember the logic of how to use R & memorise the functions / workflow based on the R Ladies of Sydney methodology. 

And

2. So I can share this with my PGCert in Data Journalism classmates and anyone else who might find this useful. Enjoy!

### The original dataset is [here](https://raw.githubusercontent.com/rladiessydney/RYouWithMe/master/sydneybeaches.csv)

And the starting point for the tutorials is [here](https://rladiessydney.org/courses/ryouwithme/)

## Time to 'separate', 'unite', and 'mutate' !
We're still using the 'cleanbeaches' dataset. First, let's look at it again... 
```{r}
glimpse(cleanbeaches)
```
## Piping time!! 
We are separating a single column into multiple columns using the 'separate' function and the 'c' function, which creates vectors. You can also keep the original column by appending the 'remove = FALSE' command at the end of the piping hot command:

```{r}
testdate <- cleanbeaches %>% separate(date, c("day", "month", "year"), remove = FALSE)
```
## Now for the opposite! "unite"... 
```{r}
cleanbeaches %>% unite(council_site, council:site)
```
Success! So useful... Next... 

## Mutate to survive!!!

# Use mutate to transform the beachbugs data
Let's remind ourselves what the data looks like:
```{r}
summary(cleanbeaches)
```
Due to the extreme values in the beachbugs data, we want to transform this into "something like a normal distribution". Here we go!

```{r}
cleanbeaches %>% mutate(logbeachbugs = log(beachbugs))
```
Wow, okay so R has a native function to calculate logs which means we can transform the data instantly... 

# Use mutate to compute new numeric variable

Now to use a function called 'lag' which essentially gives you the dataset "one back", so you can see the difference between new variables & previous ones... I think!

```{r}
cleanbeaches %>% mutate(beachbugsdiff = beachbugs - lag(beachbugs))
```
Intriguing. Next step:

# Use mutate to compute new logical variable

```{r}
cleanbeaches %>% mutate(buggier = beachbugs > mean(beachbugs,na.rm = TRUE))
```
Let's check the mean values to see if this is correct... 
```{r}
meanbugs = mean(cleanbeaches$beachbugs,na.rm = TRUE)
```
Brain-meltingly fantastic!! 

## Now to pipe all of these fabulous functions into a new object. SO useful...

```{r}
cleanbeaches_new <- cleanbeaches %>% 
  separate(date, c("day", "month", "year"), remove = FALSE) %>%
  mutate(logbeachbugs = log(beachbugs)) %>%
  mutate(beachbugsdiff = beachbugs - lag(beachbugs)) %>%
  mutate(buggier_all = beachbugs > mean(beachbugs,na.rm = TRUE)) %>%
  group_by(site) %>%
  mutate(buggier_site = beachbugs >mean(beachbugs, na.rm = TRUE))
```
My head is actually going to explode in the best possible way. I think I followed this... wow!! The power of %>% Pipe & mutate!!! 
M
