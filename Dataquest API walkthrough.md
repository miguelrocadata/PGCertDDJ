---
title: "API Practice Dataquest"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

#### I'm writing this [Dataquest](https://www.dataquest.io/) tutorial up purely for my own memory and to share with my classmates as I learn R Studio for my PGCert in Data Journalism.

### Let's follow the tutorial on [this webpage](https://www.dataquest.io/blog/r-api-tutorial/)

## Time to load packages

I may not need them all but this is good practice for future coding in R. 

```{r}
install.packages("httr")
library(httr)
install.packages("jsonlite")
library(jsonlite)
library(tidyverse)
library(skimr)
library(janitor)
```
Done. Next steps... Time to leave the earth for outer space... 

## NASA API : How many astronauts are there in space?
```{r}
res = GET("http://api.open-notify.org/astros.json")
glimpse(res)
list(res)
```
So far I can't see the list... but I can inspect the result of the request. Great! Onwards...

## Convert raw data to readable content (characters). More magical mysteries!
```{r}
rawToChar(res$content)
```
## Now let's use the jsonlite package to sort this out... 
```{r}
data = fromJSON(rawToChar(res$content))
names(data)
```
## Let's explore more... 
```{r}
data$people
```
The result looked like this:

#name                          #craft
<chr>                           <chr>

- 1	Chris Cassidy	ISS		
- 2	Anatoly Ivanishin	ISS		
- 3	Ivan Vagner	ISS		
- 4	Sergey Ryzhikov	ISS		
- 5	Kate Rubins	ISS		
- 6	Sergey Kud-Sverchkov	ISS

6 rows

# Finito!

And that's it! Very simple workflow, but immensely useful! More to follow... 
M

