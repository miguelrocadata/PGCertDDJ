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

# Update! Monday Oct 5th: Data analysis #fail! :(

Sadly I discovered over the weekend that I made a fatal error in my precedeing step to arrive at a list of top ten cities per capita. I was investigating a slightly different question: What happens if you combine different census categories & count them all as "pagan"? (paganism seems to encompass other sub
categories like heathenry / wicca / etc) - and to my horror I discovered that somehow I cleaned and/or filtered the data incorrectly when I did my initial 
investigation. 

I think I sorted the towns in cities in order of population first, then in order of pagans per capita (I think). This somehow skewed my results. 
I retraced my steps and did the same process again with the original dataset - clean / filter / calculate. This led me to a revised list as follows:

1. Hastings
2. Stroud
3. Lincoln
4. Eastbourne
5. Lancaster
6. Norwich
7. Worthing
8. Swindon UA
9. Lewes
10. Southend-on-Sea UA

This took a lot less time than my previous efforts, so I guess that's a good sign. I then continued with the following steps:

With some very simple maths this allowed me to work out a rough figure per capita for each town (Pagans/Population x 100000).
Now I had some useful data. So I played around with the Google sheets charts functions until I found some dataviz I was happy with. I also used Datawrapper to create some different angles / perspectives for comparison
e.g. of the top ten towns & cities, what does it look like when you filter them by population size as opposed to number of pagans per capita. It helped me understand the data in different ways and I hope it would also interest my
potential audience (at least briefly).

## Problem 3: combined categories
I finally decided to investigate what would happen if I combined the category "pagan" with others to reflect the way many pagans seem to view themselves & how experts categorise them (pagan = larger category which includes other religions).

By this point I realised that I didn't actually need to use any special functions to calculate this. I just created a new column called "combined" and 
then used the following formula to finalise my investigation: (SUM(Rows of religious categories I wanted to add up)/Row of Population)* 100000 (no space after the * but markup thinks I want to italicise the number!) 

I simply sorted the data by this new row and had a new top ten. Simple! I did learn about how to use GETPIVOTDATA & VLOOKUP through my investigations last week, but in the end I didn't need them for this list and I learnt a much more important lesson: Check & clean your data meticulously (and quickly) before moving to analysis. My missteps led me down totally the wrong track! I even interviewed someone in Southend because my original data was flawed and I thought it had the highest number of pagans per capita! :) Oh dear. 


M
