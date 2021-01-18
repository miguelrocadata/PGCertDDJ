---
output: html_document
---

# Using an API in R

An API is one way of getting data from a web resource. Typically you get data by forming a URL - the URL is basically your 'question' (**query**), and the webpage that is delivered to you (the **endpoint**) contains the 'response' with the data, often in JSON format.

The [postcodes API](http://api.postcodes.io/), for example, can be queried by putting a postcode (without spaces) at the *end* of this URL:

`http://api.postcodes.io/postcodes/`

To ask about the postcode B42 2SU, then, you would add it to the end to form the URL:

`http://api.postcodes.io/postcodes/b422su`

If you go to that URL you will get a bunch of code in **JSON** - this is the data for that postcode. If you want it to look a bit easier to understand use the browser extension [JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc?hl=en).

Let's store that URL in an object in R - we'll call it `url`:

```{r}
url <- "http://api.postcodes.io/postcodes/b47ap"
```

Note that this is only a string of characters - it is *not* the contents that can be found *at* that URL. But now that we've stored that URL address, we are going to grab some data from it.

## Installing a package

To import JSON files we need to activate a **package** called `jsonlite`. Packages [are](https://www.statmethods.net/interface/packages.html:

> "collections of R functions, data, and compiled code in a well-defined format." 
In other languages like JavaScript and Python, this would be called a library. But in R, confusingly, a *library* is something else: "the directory where packages are stored".

To use a package in R you need to do two things: firstly, make sure the package is *installed*, and secondly, *add it* to your library.

```{r}
install.packages('jsonlite')
library('jsonlite')
```

In some cases, the package does not actually need to be installed because it is *already* installed. `jsonlite` is actually one such package, but I've shown you how to install it anyway above because it's useful to see the two together.

*(To see which packages are installed, you can browse the *Packages* tab in the lower right corner area in RStudio. Here you will see a list of packages, and you can add any to your library by ticking it. )*

Once you've installed a package and added it to your library, you can use **functions** from it. In this case, we need the function `fromJSON`, which we can use to load JSON from our URL like so:

```{r}
jsoneg <- fromJSON(url)
```

Now that new object should appear in the *Environment* area (upper right) in RStudio. The next step is to grab something specific from that JSON object.

## Drilling into the JSON

It's a good idea to have the URL open in a browser at the same time so you can see the structure and work out how to access the bit you're after. Again, you should use Chrome or Firefox with the extension [JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc?hl=en) installed, as this makes it a lot easier to understand. (*Tip: hover over any element to see the 'path' to that element in the bottom left corner of the browser*).

If we print the contents of that object, we get some idea of the branches within it. Look at the dollar sign name before each item. For example the first piece of information - `200` - comes after `$status`:

```{r}
jsoneg
```

You can access individual items by using the word after the dollar sign inside a double square bracket and single commas like so:

```{r}
jsoneg$status
```

Some have more than one - for example `$result$codes$ccg`. To access that just use the dollar signs in the same way to indicate a branch like so:

```{r}
jsoneg$result$codes$ccg
```

So to get the constituency we need:

```{r}
jsoneg$result$parliamentary_constituency
```

That's fine for one JSON file - but how can we do this for dozens or hundreds?

## Creating a loop

To use this more than once - on multiple postcodes - we need to create a **loop** - and we need a *list* of those postcodes to loop through.

The list can be stored in a *vector* object created using the `c()` function like so:

```{r}
postcodes <- c("B47AP","BL23QQ","B735XJ")
```

A loop uses `for`, followed by a reference to the list in parentheses, followed by what we want to happen for each item. It looks like this:

```{r}
for (i in postcodes){
  print(i)
}
```

Note that we create a new object here, each time the loop runs: `i`. This is the name given to each item in that list, each time it lists. So the first time, `i` is "B422SU", the second time `i` is "BL23QQ", and so on. `i` is just an arbitrary name - we could call it 'postcode', for example. Often `i` is used in loops, though, because it's short.

At the moment `i` is just printed - but we can do something more useful with it, like add it to the postcodes URL. A useful function for that is `paste`:

```{r}
paste("http://api.postcodes.io/postcodes/","B47AP")
```

This combines the two strings in parentheses, into one. It can be used with a variable as well:

```{r}
postcode <- "BL23QQ"
paste("http://api.postcodes.io/postcodes/",postcode)
```

Note that there's a space between them, though. To stop that happening, we need to specify what the *separator* should be (it defaults to a space), using `sep=`:

```{r}
paste("http://api.postcodes.io/postcodes/","B422SU", sep="")
```

Now to put that in the loop:

```{r}
for (i in postcodes){
  print(paste("http://api.postcodes.io/postcodes/",i, sep=""))
}
```

And then, let's grab the JSON from each URL - and print something from it:

```{r}
for (i in postcodes){
  #store the full url in an object
  url <- paste("http://api.postcodes.io/postcodes/",i, sep="")
  #grab the JSON
  jsoneg <- fromJSON(url)
  #drill down into one branch and print the value
  print(jsoneg$result$parliamentary_constituency)
}
```

## Storing the results of a loop

To store the results of a list you can add each result to a list as you go. Here's the code:

```{r}
#This creates an empty list called 'resultslist'
resultslist = c()
for (i in postcodes){
  url <- paste("http://api.postcodes.io/postcodes/",i, sep="")
  jsoneg <- fromJSON(url)
  print(jsoneg$result$parliamentary_constituency)
  #This time we store the results of drilling down into part of the JSON
  constituency <- jsoneg$result$parliamentary_constituency
  #Then we add it to the list
  resultslist = c(resultslist, constituency)
}
```

The key line here is:

`resultslist = c(resultslist, constituency)`

The `c()` function here has 2 ingredients: firstly, a vector (`resultslist`) and secondly, a single item (`constituency`). The two are combined to create a new vector, which is stored in `resultslist` again. In other words, the original vector is *overwritten* with its original contents, plus a new item.


## Bonus: store the code in a function 

If you're going to re-use code like this it's often a good idea to store it in a function. You can see that being done with an API in [this example](https://github.com/paulbradshaw/Rintro/blob/master/rAPI/spotify/using_spotify_api.Rmd) - although bear in mind the Spotify API now requires a key, so the query won't work now. But the code still makes sense.
