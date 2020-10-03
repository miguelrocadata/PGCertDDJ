# Top ten pagan towns and cities in England and Wales
I decided to investigate which [cities in England and Wales had the highest number of pagans based on ONS census data (2011)](https://miguelrocawrites.uk/2020/10/02/top-ten-pagan-towns-cities-in-england-wales/). I excluded Scotland and Northern Ireland (for now) as they have separate statistics agencies. This posed a number of small challenges such as:
+ Sourcing the right dataset (ONS has lots!) 
+ Interrogating the data to find leads and / or an answer to my question 
+ Producing some dataviz to represent the data from different angles so I could investigate it further.
To do this I began by using the 3 chords of datajournalism approach as described by Paul Bradshaw: Clean/Sort and/or Filter / Percentages.
## Steps 
1. Download [this dataset from ONS](https://www.nomisweb.co.uk/census/2011/qs210ew) & work with it to clean it:    
  + make a copy of the data & label it "working data"
  + remove empty spaces / pointless text / grand totals / etc.
2. Sort data into gapless columns by size (Z-A). 
3. Filter the data & calculate a percentage - the question is which one?
## Problem 1
I ran into varous issues such as:
+ deciding which percentages to calculate at first. 
+ finding a working definition of a "city" (surprisingly hard as the ONS data had lots of different subcategories and also listed the data by country / region).
+ realising that I needed to simplify the dataset to make it easier to handle.

After several web searches and cross references I finally settled on a simple list of "towns & cities". I switched to Google Sheets & used the "ImportXML" function to scrape some data on UK cities straight into a new spreadsheet.
The basic pattern I've found online to do this (when you want to scrape table data which is already in this structure on a website) is: 
+ =IMPORTXML("web address", "//tr").

I then manually selected areas or districts from the original ONS dataset which aren't "towns" or "cities" per se, but which were listed on the census.
I created a pivot table to strip out the other religious / non-religious categories to explore this more carefully and ended up with a list like this, based on the number of pagans listed in each city filtered in descending order: 
#### Area / No. of pagans	      
1. NG - Nottingham	1470
2. B - Birmingham	1325
3. S - Sheffield	1290
4. BS - Bristol	1145
5. LE - Leicester	980
6. CF - Cardiff	920
7. NE - Newcastle upon Tyne	850
8. M - Manchester	835
9. SE - London SE	810
10. E - London E	605

One of the big problems I noticed immediately in the data was the London was split up into regions. I created a new pivot table to interrogate this data & come up with a combined figure for all London regions. What was interesting was that the proportion of pagans in the South East was over double that of London despite the fact that both areas had very similar populations. I investigated further. 
I then realised that this was problematic because it had no context (percentage) with respect to how many pagans were in each town or city relative to their estimated population. This is where Callum Thompson helped me out by sending me an updated dataset from the ONS which had population per area on too.
## Problem 2: Population Data
Once I combined population data with the data on the number of pagans I was able to get a new filtered list (Z-A) of towns and cities which caught me by surprise, as none of the "big hitters" I'd expected were there (no London / Manchester / Birmingham, for example).
The new list looked like this: 
### City/Town
1. Southend on Sea
2. Portsmouth       
3. Milton Keynes
4. Southampton
5. Nottingham
6. Derby       
7. Stoke-on-Trent
8. Hull
9. Coventry
10. Swansea

I found this out by simply copying 4 columns into Google Sheets (I prefer the visual layout of this sometimes to Excel), and labelling the data as follows: 
+ City/Town
+ Per Capita
+ Population
+ Pagans (total number)

With some very simple maths this allowed me to work out a rough figure per capita for each town (Pagans/Population x 100000).
Now I had some useful data. So I played around with the Google sheets charts functions until I found some dataviz I was happy with. I also used Datawrapper to create some different angles / perspectives for comparison
e.g. of the top ten towns & cities, what does it look like when you filter them by population size as opposed to number of pagans per capita. It helped me understand the data in different ways and I hope it would also interest my
potential audience (at least briefly).
## Problem 3: Population change
I finally decided to investigate what updated figures since 2011 could tell me about how these towns & cities have changed. To do this I used a dataset Paul Bradshaw used in our workshop which was called "uk mid-year estimates...", and I created yet *another spreadsheet* to work with this data.

Briefly, I cleaned, sorted, and filtered the data and then created a pivot table to find just the top ten towns & cities I was interested in. This allowed me to isolate the new population estimates for each place very quickly. 
I created new sheets in my spreadsheet and now had a new list called "population 2019 (date of census estimates)" which had the following headers:
+ Town/City	
+ Population (2019)	
+ Total change since 2011	
+ % change	
+ Estimated Pagans 2019

Now I got to use two functions to help me quickly move data around in my sheet to create comparisons so I could produce my final two data visualisations on Google Sheets. 
### Get Pivot Data

First I used GETPIVOTDATA to move the data from my pivot table into the column called "Population (2019)". This was a case of typing in the formula and using the cursor to select the area I wanted to see in the other sheet. I then hovered the cursor over the first cell I had pulled up until the black cross appeared.
I the dragged the cross down to fill in the empty cells in this column.

### VLOOKUP

Now the tricky part. I wanted to quickly fill the "Estimated pagans" column in using data from another sheet which I had copied from my first dataset based on the 2011 census.
I had called this sheet "Top ten pagan towns 2011". This had the census data on, which I wanted in my new sheet to create a quick comparison. 
At this point I had to look up (pun intended) a VLOOKUP tutorial on the web and follow along. [I found a great one here](https://www.youtube.com/watch?v=d3BYVQ6xIE4). **Update: I caught up on some reading this weekend and discovered that Paul addresses this in one of his books (*finding stories in spreadsheets*)! Whoops!** :)

Following the steps I had learnt, I ended up with the following formula: 

=VLOOKUP('Population 2019'!C2,'Top ten pagan towns 2011'!$A$2:$D$11,2,FALSE)

This did exactly what I had hoped except that there were some errors. I followed the video tutorial to understand how to fix these. I discovered that (of course)
if excel is looking for an *exact match* for a value (which is what the FALSE command does), then names (i.e. of cities) have to be exactly the same - including not having any 
unnecessary spaces or unusual features like hyphens (One of my sheets had *Southend-on-Sea*, the other had *Southend on Sea*, for example). 

Once this was cleared up, I had my new figures. Finally, to calculate the "Estimated Pagans 2019" I Googled how to express this kind of sum in excel/Google Sheets and simply added 
the following addendum to my VLOOKUP formula: 
+ *(1+F2)

So this obviously multiplied the census data which I pulled in using VLOOKUP by 1 plus whatever percentage was in my "% change" column, increasing the number of pagans in 2011 by the same percentage as the change in population based on 2019's figures. 
(It's F2 because I had some other geographical area code/postcode columns in my sheet: I did toy with trying to create an interactive map using this data but my current skillset wouldn't allow it as yet!) :)

Phew! I could add more details about various steps I've taken this week but I think that satisfies the criteria for our directed learning task. I now have a reasonably good working 
knowledge of how, and why to use three functions: IMPORTXML, GETPIVOTDATA, and VLOOKUP. I also have a better idea of how to create and use pivot tables.

And I also have a much better idea of how to use Github Markup!

I hope that all makes sense!

M
