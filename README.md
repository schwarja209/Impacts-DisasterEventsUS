# Health Impacts and Economic Costs of Disaster Events in the US
Jacob Schwartz  
August 30, 2017  
  
# Synopsis
  
This project involves exploring the U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database. This database tracks characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property or crop damage. NOAA has maintained this database of storm events since 1950. 


**This analysis will investigate the costs and human impact of these disaster events. We will endeavor to answer the questions:**

**1. Across the United States, which types of events are most harmful with respect to population health?**
**2. Across the United States, which types of events have the greatest economic consequences?**


*Note: Although the data goes as far back as 1950, it wasn't until 1996 that NOAA created a list of 48 standardized event types. Events prior to 1996 were classed into fewer types, and even after 1996 many reports of storm events do not stick to the official list of events. For the purposes of this analysis, we will observe the data both in total and with the data post 1995 separated out, and determine which, if either, is more suited to our needs.*

# Data Processing
  
  
### 1. Loading Libraries
  
In order to process the data and results, we first need to initialize a few libraries. Plyr, dplyr and reshape2 are for data cleaning and processing purposes, while ggplot2 and gridExtra are for presentation of the results.
  

```r
library(plyr) #load necessary libraries
library(dplyr)
library(reshape2)
library(ggplot2)
library(gridExtra)
```
  
  
### 2. Loading Data
  
The next step is to download and extract the data from NOAA. We'll go with a straight extraction for simplicity, though there are potentially faster means of doing so with such heavily compressed data. We'll also move the extracted data to a new variable, in case we do any manipulations but still want to refer back to the original extraction. This prevents having to go back and re-extract the data repeatedly.
  

```r
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2","StormData.csv.bz2")
StormData_RAW<-read.csv("StormData.csv.bz2") #download and load data
StormData<-StormData_RAW #store RAW data for reference 
```
  
Then the question is, what does the structure of the raw data look like.
  

```r
str(StormData) #look at data
```

```
'data.frame':	902297 obs. of  37 variables:
 $ STATE__   : num  1 1 1 1 1 1 1 1 1 1 ...
 $ BGN_DATE  : Factor w/ 16335 levels "1/1/1966 0:00:00",..: 6523 6523 4242 11116 2224 2224 2260 383 3980 3980 ...
 $ BGN_TIME  : Factor w/ 3608 levels "00:00:00 AM",..: 272 287 2705 1683 2584 3186 242 1683 3186 3186 ...
 $ TIME_ZONE : Factor w/ 22 levels "ADT","AKS","AST",..: 7 7 7 7 7 7 7 7 7 7 ...
 $ COUNTY    : num  97 3 57 89 43 77 9 123 125 57 ...
 $ COUNTYNAME: Factor w/ 29601 levels "","5NM E OF MACKINAC BRIDGE TO PRESQUE ISLE LT MI",..: 13513 1873 4598 10592 4372 10094 1973 23873 24418 4598 ...
 $ STATE     : Factor w/ 72 levels "AK","AL","AM",..: 2 2 2 2 2 2 2 2 2 2 ...
 $ EVTYPE    : Factor w/ 985 levels "   HIGH SURF ADVISORY",..: 834 834 834 834 834 834 834 834 834 834 ...
 $ BGN_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
 $ BGN_AZI   : Factor w/ 35 levels "","  N"," NW",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ BGN_LOCATI: Factor w/ 54429 levels "","- 1 N Albion",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ END_DATE  : Factor w/ 6663 levels "","1/1/1993 0:00:00",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ END_TIME  : Factor w/ 3647 levels ""," 0900CST",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
 $ COUNTYENDN: logi  NA NA NA NA NA NA ...
 $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
 $ END_AZI   : Factor w/ 24 levels "","E","ENE","ESE",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ END_LOCATI: Factor w/ 34506 levels "","- .5 NNW",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ LENGTH    : num  14 2 0.1 0 0 1.5 1.5 0 3.3 2.3 ...
 $ WIDTH     : num  100 150 123 100 150 177 33 33 100 100 ...
 $ F         : int  3 2 2 2 2 2 2 1 3 3 ...
 $ MAG       : num  0 0 0 0 0 0 0 0 0 0 ...
 $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
 $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
 $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
 $ PROPDMGEXP: Factor w/ 19 levels "","-","?","+",..: 17 17 17 17 17 17 17 17 17 17 ...
 $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
 $ CROPDMGEXP: Factor w/ 9 levels "","?","0","2",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ WFO       : Factor w/ 542 levels ""," CI","$AC",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ STATEOFFIC: Factor w/ 250 levels "","ALABAMA, Central",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ ZONENAMES : Factor w/ 25112 levels "","                                                                                                               "| __truncated__,..: 1 1 1 1 1 1 1 1 1 1 ...
 $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
 $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
 $ LATITUDE_E: num  3051 0 0 0 0 ...
 $ LONGITUDE_: num  8806 0 0 0 0 ...
 $ REMARKS   : Factor w/ 436781 levels "","-2 at Deer Park\n",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
```
  
There are a few things we can see from the output above:

- First, there are a lot of variables, most of which identify various location references and related data. So most of the variables will be unnecessary in our ultimate analysis.  
- Second, the most important variable, `EVTYPE`, which identifies the different types of events at the heart of our guiding questions, appears to have 985 factors. However, in the documentation linked in the appendix, NOAA only identifies 48 distinct events. So there is definitely some cleaning of this variable that will need to be done.  
- Third, the costs for property damage (`PROPDMG`) and crop damage (`CROPDMG`) are divided into two separate variables a piece. So these will need to be combined at some point for proper analysis. And based on the first several values for `PROPDMGEXP` and `CROPDMEXP`, neither is a straight forward factor we can just multiply by.  
- And finally, at least for now, all of the date and time variables are stored as factors. We'll have to make sure these are reformated to a standard format at some point, or we won't be able to filter the dates consistently (or potentially graph by them continuously).
  
  
### 3. Cleaning Event Types
  
For the moment, we can put the many variables issue aside and focus on the other three cleaning issues. So let's start by cleaning the Event Types.

Since the event types variable appears to be in such dramatic disarray, the first step we'll take is to create a clean vector of the official event types, as a control. We can get this from the data documentation as mentioned earlier.


```r
#load official EVTYPE list for comparison
EVTYPE_official<-c("astronomicallowtide","avalanche","blizzard","coastalflood","coldwindchill", 
                   "debrisflow","densefog","densesmoke","drought","dustdevil","duststorm",   
                   "excessiveheat","extremecoldwindchill","flashflood","flood","freezingfog",
                   "frostfreeze","funnelcloud","hail","heat","heavyrain","heavysnow","highsurf",
                   "highwind","hurricanetyphoon","icestorm","lakeeffectsnow","lakeshoreflood",
                   "lightning","marinehail","marinehighwind","marinestrongwind","marinethunderstormwind",
                   "ripcurrent","seiche","sleet","stormsurgetide","strongwind","thunderstormwind",
                   "tornado","tropicaldepression","tropicalstorm","tsunami","volcanicash",
                   "waterspout","wildfire","winterstorm","winterweather")
```

Then we can begin to clean with some simple steps:

1. First, we can make all of the event types listed in the dataset lower case, to eliminate any variance in capitalization.
2. Then we can remove all not alphabetic characters (`[A-Za-z]`), to remove any variance in punctuation and spaces (using periods or underscores instead of spaces, for instance).
3. And finally, we can replace any string that contains one of the values in our control vector with just that value.

*Note: When we do the last part below, we actually only remove characters after a given official event type insert, not before.  This is because several of the event types in the official list are sub-strings of others, and the sub-string aspect is always the last part of the string. So in order to avoid losing any valid event types, for the time being, we're only overwriting the end of given strings with the official vector values. We'll test if more needs to be done shortly.*


```r
StormData$EVTYPE<-tolower(StormData$EVTYPE) #make EVTYPE lowercase to narrow down options
StormData$EVTYPE<-gsub("[^A-Za-z]","",StormData$EVTYPE) #remove all non alphabetic chars to narrow further
for(i in 1:length(EVTYPE_official)){ #sub in official EVTYPE for all EVTYPES that are similar
    StormData$EVTYPE<-gsub(paste0(EVTYPE_official[i],".*"),paste0(EVTYPE_official[i]),StormData$EVTYPE)
}
```

Now it's time to check the results of our cleaning thus far, and see if we've done enough to make `EVTYPE` a sufficiently useable variable.

In order to do this check, we can create a `check` table that stores a count of all of the remaining `EVTYPE` values that are still not cleaned.  We can then also sort the resulting table, so that the unclean variables that appear most often are right at the top. This will allow us to find where we can get the biggest bang for our buck if we decide to do any further cleaning.


```r
check1<-table(StormData$EVTYPE) #create table of all remaining EVTYPEs
check1<-check1[!(names(check1) %in% EVTYPE_official)] #narrow table to only unofficial EVTYPEs
check1<-check1[order(check1,decreasing=TRUE)] #sort table in order of decreasing EVTYPE frequency
```

Before we see what our `check` table contains, let's first see if we've already done enough cleaning. We can do this by calculating what percentage of the total rows from the original data frame the sum of the `check` table represents.


```r
print(sum(check1)/nrow(StormData)*100) #see what percent of the EVTYPE data is still uncleaned
```

```
[1] 26.62183
```

Based on this calculation, it looks like `EVTYPE` still contains **26.6%** unofficial `EVTYPEs`. That's very high. Losing over a quarter of our data this early would be significant, so we should probably try to clean the variable further.

In order to better direct our continued cleaning, let's see where we can make the most headway by looking at the first few values of our `check` table. We can also then look at the entire list of value names to see if any of those at the top are also similar to any other stragglers (given that this second part is optional and takes up a lot of space, the output is hidden below).


```r
print(head(check1,25)) #see what the top 10 uncleaned EVTYPES are, to check clustering
```

```

            tstmwind       marinetstmwind    urbansmlstreamfld 
              219961                 6175                 3392 
      wildforestfire         tstmwindhail          extremecold 
                1457                 1028                  657 
                snow            landslide                  fog 
                 617                  600                  538 
          urbanflood                 wind           stormsurge 
                 354                  347                  261 
        freezingrain    heavysurfhighsurf     extremewindchill 
                 260                  228                  210 
          riverflood        drymicroburst            lightsnow 
                 202                  192                  176 
           hurricane         recordwarmth     unseasonablywarm 
                 174                  154                  126 
astronomicalhightide     moderatesnowfall            wintrymix 
                 103                  101                   94 
           heavysurf 
                  87 
```

```r
print(sort(names(check1))) #get list of all unofficial EVTYPES remaining <--optional
```

Wow, `tstmwind` is in `EVTYPE` over **200000** times! There are just over **900000** `EVTYPE` values total, so that's extremely significant for just one value (over **24%** of all values!). We can also see that the second most significant remaining unofficial value is `marinetstmwind`, which, while similar, fits into a completely different official event type. This will be important shortly.

Another notable item is that after about 20 values the count for the given values in our `check` table drops below 100. Any count below 100 is only 0.01% of all of the values in `EVTYPE` *at most*, so we can probably ignore anything below this mark. You could alternatively ignore any value with a count below 1000 (0.1% or less of all values), which would still give us quite precise results, but for the sake of the experience, let's dig deeper than that.

In order to fix the top 20-ish remaining values, we'll go through piecemeal, and make sure that we really get them fixed. As mentioned earlier, we can also throw in anything similar to these values (from the full name list), since we'll have a line of code for them anyway.

Here's the process:

1. We can start right at the top with `tstmwind`, but then we could run into a problem with `marinetstmwind`, since `tstmwind` is a sub-string. So let's start with `marine tstmwind` first. This appears to map to the `marinethunderstormwind` value in our official vector, and doesn't appear to have any particularly similar stragglers.
2. Now we can deal with `tstmwind`. But actually, since there's no official value for just `thunderstorm`, we can look for any value that still has `tstm` in it period. And from the full list of names, we can see that we can also include anything that involves `thunder`.
3. Next we have `urbansmlstreamfld`. At first glance this doesn't appear to map to anything in the official vector. But if we go to the documentation and search for the word "urban", we can see that this is only mentioned in the context of the `flood` event type. So we can map anything with `urban` to the `flood` event type, and while we're at it we might as well throw in `riverflood`.
4. Next there's `wildforestfire`, which based on the list of names can be boiled down to just `wild`, and fits into the official category of `wildfire`.
5. Then there's `tstmwindhail`, but we've already handled that with `tstm`.
6. Then `extremecold` is next. This one is pretty easy to fit into `extremecoldwindchill`, and it looks like we can also fit in `extremewindchill` and `extendedcold`. One thing that's different here is that `extremecold` is a sub-string of `extremecoldwindchill`. So to avoid any unnecessary or unwanted replacement, we should anchor the former to the end of the string.
7. ... 23. 

We can continue to go on like this down the list, with a few quirks here and there (like distinguishing what goes into `strongwind` vs `highwind`, using the definitions in the documentation), and once we do, we end up with the series of functions below.


```r
#use data documentation to fit top unofficialEVTYPES (and other similar ones) into official categories
StormData$EVTYPE<-gsub("marinetstm.*","marinethunderstormwind",StormData$EVTYPE)
StormData$EVTYPE<-gsub("tstm.*|thunder.*","thunderstormwind",StormData$EVTYPE)
StormData$EVTYPE<-gsub("urban.*|riverflood.*","flood",StormData$EVTYPE)
StormData$EVTYPE<-gsub("wild.*","wildfire",StormData$EVTYPE) #where we'd cut off with 1000 threshold 
StormData$EVTYPE<-gsub("extremecold$|extremewindchill.*|extendedcold.*","extremecoldwindchill",StormData$EVTYPE)
StormData$EVTYPE<-gsub("^snow.*","heavysnow",StormData$EVTYPE)
StormData$EVTYPE<-gsub("landslide.*|landslump.*","debrisflow",StormData$EVTYPE)
StormData$EVTYPE<-gsub("^fog.*","densefog",StormData$EVTYPE)
StormData$EVTYPE<-gsub("^wind.*","strongwind",StormData$EVTYPE)
StormData$EVTYPE<-gsub("stormsurge.*","stormsurgetide",StormData$EVTYPE)
StormData$EVTYPE<-gsub("freezingrain.*","sleet",StormData$EVTYPE)
StormData$EVTYPE<-gsub("heavysurf.*|.*hightide.*","highsurf",StormData$EVTYPE)
StormData$EVTYPE<-gsub("^drymicroburst.*","highwind",StormData$EVTYPE)
StormData$EVTYPE<-gsub("lightsnow.*|moderatesnow.*|wintrymix.*","winterstorm",StormData$EVTYPE)
StormData$EVTYPE<-gsub("^hurricane.*|^typhoon.*","hurricanetyphoon",StormData$EVTYPE)
StormData$EVTYPE<-gsub("recordw.*|recordh.*","excessiveheat",StormData$EVTYPE)
StormData$EVTYPE<-gsub("unseasonablyw.*|unseasonablyh.*|unusuallyw.*","heat",StormData$EVTYPE)
```

Now that we've done this more in depth cleaning, it's time to check how our efforts to fix things faired.  We can use the same `check` method as before to do this.


```r
check2<-table(StormData$EVTYPE) #create table of all remaining EVTYPEs
check2<-check2[!(names(check2) %in% EVTYPE_official)] #narrow table to only unofficial EVTYPEs
check2<-check2[order(check2,decreasing=TRUE)] #sort table in order of decreasing EVTYPE frequency
print(head(check2,10)) #see if the top 10 uncleaned EVTYPES have under 100 records per EVTYPE
```

```

           cold          freeze      recordcold      gustywinds 
             82              76              68              65 
            ice           frost unseasonablydry       smallhail 
             61              57              56              53 
          other          funnel 
             52              46 
```

```r
print(sum(check2)/nrow(StormData)*100) #see what percent of the EVTYPE data is still uncleaned
```

```
[1] 0.2146743
```

From the results above we can see that the top remaining unofficial event type now has a mere 82 instances in `EVTYPE`, and all of the remaining unofficial event types amount to only **0.2%** of the data. That's a job well done.


### 4. Cleaning Property and Crop Damage Values

Next on our list is cleaning the property and crop damage cost variables. In this case, however, we don't really need to worry about the actual `PROPDMG` and `CROPDMG` variables themselves, as they're in decent shape. The issue we saw earlier pertained to `PROPDMGEXP` and `CROPDMGEXP`, which, according to the documentation, are the exponential components of the cost values given.

Let's start off by taking a closer look at the distribution of values in these two factors.


```r
print(table(StormData$PROPDMGEXP)) #check the values and distribution of PROPDMGEXP and CROPDMGEXP
```

```

            -      ?      +      0      1      2      3      4      5 
465934      1      8      5    216     25     13      4      4     28 
     6      7      8      B      h      H      K      m      M 
     4      5      1     40      1      6 424665      7  11330 
```

```r
print(table(StormData$CROPDMGEXP))
```

```

            ?      0      2      B      k      K      m      M 
618413      7     19      1      9     21 281832      1   1994 
```

As we can see in the tables above, we have a variety of values, from random seeming punctuation, to numbers, to letters, to a large number of factors with completely empty values. Referring again to the documentation, it appears that the various factors mean the following:

- `h` or `H` means *10^2*
- `k` or `K` means *10^3*
- `m` or `M` means *10^6*
- `b` or `B` means *10^9*
- The numbers are each their own power of 10
- The symbols are approximation indicators
- And the blanks just mean there's no exponential component.

It looks like what we need to do is standardize this notation as factors of 10 that we can eventually multiply our base cost values by. This will eventually allow us to make direct cost comparisons.

By standardize, we mean replace, an operation that would usually use a base or package function coupled with `mutate`. But there's no base or package function that exists that can navigate the specific quirks we just listed above. So first, we need to create a function that understands the nuances of the replacement operation we'd like to perform.


```r
EXP<-function(x){ #create a function to translate PROPDMGEXP and CROPDMGEXP into usable integers
    if(x %in% 0:9) return(as.numeric(paste0("1e",x)))
    else if(x=="h"|x=="H") return(1e2) 
    else if(x=="k"|x=="K") return(1e3)
    else if(x=="m"|x=="M") return(1e6)
    else if(x=="b"|x=="B") return(1e9)
    else return(1)
}
```

Then, we can plug `PROPDMGEXP` and `CROPDMGEXP` into that function and get them cleaned up.


```r
StormData<-mutate(StormData,PROPDMGEXP=as.numeric(sapply(PROPDMGEXP,FUN=EXP)))%>%
    mutate(CROPDMGEXP=as.numeric(sapply(CROPDMGEXP,FUN=EXP))) #run all EXP data through the function
```

Now, let's check those distributions again, and see how things look.


```r
print(table(StormData$PROPDMGEXP)) #re-check the values and distribution of PROPDMGEXP and CROPDMGEXP
```

```

     1     10    100   1000  10000  1e+05  1e+06  1e+07  1e+08  1e+09 
466164     25     20 424669      4     28  11341      5      1     40 
```

```r
print(table(StormData$CROPDMGEXP))
```

```

     1    100   1000  1e+06  1e+09 
618439      1 281853   1995      9 
```

Looks much better!


### 5. Cleaning Dates

Last on our original list of items to fix were the dates. This one is easy, as we're simply standardizing each of the dates to be `Date` class variables in a basic `"%m/%d/%Y"` format.


```r
StormData$BGN_DATE<-as.Date(StormData$BGN_DATE,format="%m/%d/%Y") #convert dates to date format
StormData$END_DATE<-as.Date(StormData$END_DATE,format="%m/%d/%Y")
```

While we're looking at the dates, though, let's turn for a moment to an side item mentioned in our synopsis. NOAA only finalized their official list of event types after 1995. This could lead us to conclude that we should filter the data down to only include values recorded from 1996 on, to give us a more even playing field for comparison. But that begs the question: if we do filter the data, how much data would we be losing?


```r
#see what percent of dates are before 1996
print(nrow(StormData[StormData$BGN_DATE<"1996-01-01",])/nrow(StormData)*100)
```

```
[1] 27.57041
```

```r
print(summary(StormData$BGN_DATE)) #get a summary of the date range and spread
```

```
        Min.      1st Qu.       Median         Mean      3rd Qu. 
"1950-01-03" "1995-04-20" "2002-03-18" "1998-12-27" "2007-07-28" 
        Max. 
"2011-11-30" 
```

What we can see above is that the data pre-1996 is over **27%** of our values. That's a lot to throw away. The summary of the dates shows this clearly as well, with the first quartile marker being in mid-1995.

We weren't planning to do any filtering right now anyway, but this is going to be important to keep in mind once we start consolodating the data into summaries (which happens to be our next task).


### 6. Narrowing and Summarizing The Data

Now that we're done cleaning the data, it's time to start shaping the storm data into a form that can answer our original questions. Just to refresh, those questions were:

1. Across the United States, which types of events are most harmful with respect to population health?  
2. Across the United States, which types of events have the greatest economic consequences?  

Thinking about these questions, it's clear that there are two roads we can take:

1. Our first option is to summarize the data as *averages* of the effects on population health and economic consequences, per event type. This kind of summary would give us a good idea of what sorts of disaster events are the most harmful, generally speaking, and therefore are statistically the most potentially harmful.
2. Our second option is to summarize the data as *absolute totals* of the effects on population health and economic consequences, per event type. This kind of summary would paint a picture of which disaster types have historically had the largest impacts en masse, taking into account the effects from the most extreme occurances. For instance, even if only 1 in a thousand hurricanes is category 5, that one category 5 hurricane is going to be in a class of it's own in terms of its consequences.

Rather than choose just one approach, for now, let's copy the data into two separate datasets, one that we'll use for *average* summaries, and one that we'll use for *total* summaries. **However, since not all event types were recorded before 1996, we should only use data from post 1996 in the totals summary, to maintain comparitive equity** (this doesn't matter for the averages summary, as the extra data for the few pre-1996 event types will just help to create more robust means).

Also, while we're splitting our dataset for summarization, we can filter our data down to just the (now cleaned up) official event types, combine the property and crop damage costs with their (now cleaned up) exponential values, and get rid of any variables we won't be using going forward.


```r
#narrow data to only official EVTYPES & condense damage costs
StormData_Complete<-filter(StormData,EVTYPE %in% EVTYPE_official)%>%
    mutate(EVTYPE=as.factor(EVTYPE),PROPDMG=PROPDMG*PROPDMGEXP,CROPDMG=CROPDMG*CROPDMGEXP)%>%
    select(STATE,EVTYPE,BGN_DATE,FATALITIES,INJURIES,PROPDMG,CROPDMG)%>% #narrow variables of interest
    group_by(EVTYPE) #set factors for later summarization

StormData_Modern<-filter(StormData_Complete,BGN_DATE>="1996-01-01")%>% #create separate data set
    group_by(EVTYPE)                                                   #for just post 1995 data
```

Now we can create our average and total summaries. For flexibility going forward, we'll create separate summaries for each of the `FATALITIES`,`INJURIES`,`PROPDMG` and `CROPDMG` variables. We can always recombine them later if we feel we want/need to.


```r
StormData_Fatalities_Avg<-summarize(StormData_Complete,FATALITIES=mean(FATALITIES))%>%
    arrange(desc(FATALITIES)) #get average fatalities
StormData_Injuries_Avg<-summarize(StormData_Complete,INJURIES=mean(INJURIES))%>%
    arrange(desc(INJURIES)) #get average injuries
StormData_PropDMG_Avg<-summarize(StormData_Complete,PROPDMG=mean(PROPDMG))%>%
    arrange(desc(PROPDMG)) #get average property damage
StormData_CropDMG_Avg<-summarize(StormData_Complete,CROPDMG=mean(CROPDMG))%>%
    arrange(desc(CROPDMG)) #get average crop damage

# use modern data set for totals, so it's a fair comparison
StormData_Fatalities_Tot<-summarize(StormData_Modern,FATALITIES=sum(FATALITIES))%>%
    arrange(desc(FATALITIES)) #get total fatalities
StormData_Injuries_Tot<-summarize(StormData_Modern,INJURIES=sum(INJURIES))%>%
    arrange(desc(INJURIES)) #get total injuries
StormData_PropDMG_Tot<-summarize(StormData_Modern,PROPDMG=sum(PROPDMG))%>%
    arrange(desc(PROPDMG)) #get total property damage
StormData_CropDMG_Tot<-summarize(StormData_Modern,CROPDMG=sum(CROPDMG))%>%
    arrange(desc(CROPDMG)) #get total crop damage
```

Now we can finally see what our summaries look like (average of each category comes first, then total).


#### **Fatalities**

```r
print(head(StormData_Fatalities_Avg)) #take a look at the summary results
```

```
# A tibble: 6 x 2
            EVTYPE FATALITIES
            <fctr>      <dbl>
1          tsunami  1.6500000
2             heat  1.1297561
3    excessiveheat  0.9860248
4       ripcurrent  0.7425997
5        avalanche  0.5803109
6 hurricanetyphoon  0.4515050
```

```r
print(head(StormData_Fatalities_Tot)) 
```

```
# A tibble: 6 x 2
         EVTYPE FATALITIES
         <fctr>      <dbl>
1 excessiveheat       1799
2       tornado       1511
3    flashflood        887
4     lightning        651
5    ripcurrent        542
6         flood        444
```

#### **Injuries**

```r
print(head(StormData_Injuries_Avg)) #similar distributions of effects for fatalities and injuries
```

```
# A tibble: 6 x 2
            EVTYPE INJURIES
            <fctr>    <dbl>
1          tsunami 6.450000
2 hurricanetyphoon 4.458194
3    excessiveheat 3.403209
4             heat 2.449756
5          tornado 1.505520
6        duststorm 1.025641
```

```r
print(head(StormData_Injuries_Tot))
```

```
# A tibble: 6 x 2
            EVTYPE INJURIES
            <fctr>    <dbl>
1          tornado    20667
2            flood     6838
3    excessiveheat     6391
4 thunderstormwind     5129
5        lightning     4141
6       flashflood     1674
```

#### **Property Damage Cost**

```r
print(head(StormData_PropDMG_Avg))
```

```
# A tibble: 6 x 2
            EVTYPE   PROPDMG
            <fctr>     <dbl>
1 hurricanetyphoon 285472943
2   stormsurgetide 117273164
3    tropicalstorm  11067992
4          tsunami   7203100
5            flood   4990060
6         wildfire   2006987
```

```r
print(head(StormData_PropDMG_Tot))
```

```
# A tibble: 6 x 2
            EVTYPE      PROPDMG
            <fctr>        <dbl>
1            flood 144129590200
2 hurricanetyphoon  81718889010
3   stormsurgetide  47834724000
4          tornado  24616945710
5       flashflood  15222258910
6             hail  14595143420
```

#### **Crop Damage Cost**

```r
print(head(StormData_CropDMG_Avg)) #more prop dmg from droughts, more prop dmg from storm surges
```

```
# A tibble: 6 x 2
                EVTYPE    CROPDMG
                <fctr>      <dbl>
1     hurricanetyphoon 18448554.5
2              drought  5586794.0
3             icestorm  2499807.6
4        tropicalstorm   996981.3
5          frostfreeze   814126.5
6 extremecoldwindchill   703343.7
```

```r
print(head(StormData_CropDMG_Tot))
```

```
# A tibble: 6 x 2
                EVTYPE     CROPDMG
                <fctr>       <dbl>
1              drought 13367566000
2     hurricanetyphoon  5350107800
3                flood  5013161500
4                 hail  2476029450
5           flashflood  1334901700
6 extremecoldwindchill  1326023000
```

There's a lot of useful information here. In fact, there's so much useful information that we could just end our analysis here and answer our original questions. However, there may be ways to make more direct and easily understandable comparisons if we push a bit further, so we'll keep goin.

We'll can start by breaking down some of the information gleaned from the above tibbles:

- First of all, we can clearly see that assessing both the averages and the totals in our summaries was a good idea. This is because the top event types for any given impact category are quite different between the average datasets and the total datasets. So this is a valuable distinction to be able to make.
- Second, we can see that for the human impact categories (`FATALITIES` and `INJURIES`), the averages are fairly similar (and small) across the board, while the totals only have slight outliers. However, for the economic impact categories (`PROPMG` and `CROPDMG`), there are large outliers in both cases. The outliers for the average datasets, in particular, are **huge**. This is extremely important, as it means we should probably do some scaling on these values, or any graphs we put together are going to be extremely hard to read.
- Finally, it's clear that the values for `FATALITIES` and `INJURIES`, and the values for `PROPMG` and `CROPDMG` are within very similar ranges respectively, for both their averages and their totals. Since the questions we're trying to answer deal with overall human and economic impacts, it wouldn't hurt to further condense our data, and combine each of these pairs of summaries.

Let's tackle the last point first, as it'll give us slightly less to work with going forward.

To do the condensing of the datasets, we need to be careful. While we can add *absolute totals* together, we can't just do that for *averages* or they won't actually be averages anymore. So for each of the condensed *average* datasets, we should rerun the same summary code we just did, only this time the first step we'll take is to combine each of the respective impacts. For the condensed *total* datasets we can do a simple merge of the summaries we already have calculated and then combine the impacts after the fact.

As a side note, we can also divide all the costs by a billion, to bring their absolute scale down a bit. This doesn't solve our scaling problem, but it will be useful for potential graphing purposes later, in terms of saving space and making things easier to read. We'll deal with the actual scaling problem in a moment.


```r
#Note: the difference in averages is much more extreme than the difference in totals
HumanImpact_Avg<-mutate(StormData_Complete,HealthImpacts=FATALITIES+INJURIES)%>% #condense for emphasis
    summarize(HealthImpacts=mean(HealthImpacts))%>%rename(Disaster_Type=EVTYPE)%>% #make nicer for graph
    arrange(desc(HealthImpacts))
HumanImpact_Tot<-merge(StormData_Fatalities_Tot,StormData_Injuries_Tot,all=TRUE)%>%
    mutate(HealthImpacts=FATALITIES+INJURIES)%>%rename(Fatalitities=FATALITIES,Injuries=INJURIES)%>%
    arrange(desc(HealthImpacts))
EconomicCost_Avg<-mutate(StormData_Complete,TotalCost=PROPDMG+CROPDMG)%>%
    summarize(TotalCost=mean(TotalCost)/1000000000)%>%rename(Disaster_Type=EVTYPE)%>% #scale down cost
    arrange(desc(TotalCost))
EconomicCost_Tot<-merge(StormData_PropDMG_Tot,StormData_CropDMG_Tot,all=TRUE)%>%
    mutate(TotalCost=(PROPDMG+CROPDMG)/1000000000,PROPDMG=PROPDMG/1000000000,CROPDMG=CROPDMG/1000000000)%>%
    rename(Property=PROPDMG,Crop=CROPDMG)%>%arrange(desc(TotalCost))
```

So now we have our data summaries all set. Our next step in making our analysis more generally understandable will be to turn our summaries into visual representations.

Thinking about the breadth of our data, one thing is immediately clear: in order to have visually compelling graphs we're goin to need to narrow down our summaries a bit. Fitting **48** different disaster event types onto a single graph would be extremely busy (recall from earlier that that's the number of official event types that we narrowed our data down to). But we don't want to cut out data arbitrarily, and potentially lose an important part of the narrative.

First we'll need to determine what a representative sample of our summaries looks like. Let's see what percentage the **top 10** event types for each summary represent.


```r
# What is a representative slice of each category?
print(sum(HumanImpact_Avg$HealthImpacts[1:10])/sum(HumanImpact_Avg$HealthImpacts)*100) #top 10 is not
```

```
[1] 85.16509
```

```r
print(sum(HumanImpact_Tot$HealthImpacts[1:10])/sum(HumanImpact_Tot$HealthImpacts)*100) #for human impacts
```

```
[1] 85.49317
```

```r
print(sum(EconomicCost_Avg$TotalCost[1:10])/sum(EconomicCost_Avg$TotalCost)*100) #top 10 is
```

```
[1] 98.84623
```

```r
print(sum(EconomicCost_Tot$TotalCost[1:10])/sum(EconomicCost_Tot$TotalCost)*100) #for economic losses
```

```
[1] 95.32703
```

For economic cost, for both the average and total summaries, the top 10 look to be **very** representative (**98.8%** and **93.3%** respectively). However, for the human impact summaries, the numbers are not quite as sterling (around **85%** a piece).

Let's try sampling the **top 20** from the human impacts summaries instead.


```r
print(sum(HumanImpact_Avg$HealthImpacts[1:20])/sum(HumanImpact_Avg$HealthImpacts)*100) #top 20 is
```

```
[1] 96.71101
```

```r
print(sum(HumanImpact_Tot$HealthImpacts[1:20])/sum(HumanImpact_Tot$HealthImpacts)*100) #for human impacts
```

```
[1] 96.36688
```

As we can see, they are both over **96%** now, which is much better.

Now that we have our representative sample size set, we can finally perform the scaling of the economic costs mentioned earlier. To do this we can use the `log10` function on the original data values, and then re-summarize.  However, we only really need to do this for the average cost.


```r
EconomicCost_Avg<-mutate(StormData_Complete,TotalCost=PROPDMG+CROPDMG)%>% #use log10 for avg costs
    summarize(TotalCost=log10(mean(TotalCost)))%>%rename(Disaster_Type=EVTYPE)%>% #for extreme range
    arrange(desc(TotalCost))
```

One last thing: While our average summaries have the different impact types (`FATALITIES`,`PROPDMG`,etc) fully embedded in their values, the data in the total summaries still has the values for each impact type separate and distinct. This gives us an opportunity. If we can turn the impact types into factors themselves, we could plot stacked graphs. These would not only show the overall impacts and costs, but also the degree to which each impact type individually contributed, adding a whole other dimension to our analysis.

To get these new factors, we're goin to have to "melt" our representative sample from our total summaries, and then get rid of the added totals (since we'll be stacking the factors to get the added totals instead). The resulting "molten" summaries will be what we use in our graphs.


```r
#make type of impact/cost a factor
HumanImpactTot_Molten<-melt(HumanImpact_Tot[1:20,])%>%arrange(variable,desc(value)) 
HumanImpactTot_Molten<-HumanImpactTot_Molten[1:40,]%>%rename(Types_of_Impact=variable) #remove total damage
EconomicCostTot_Molten<-melt(EconomicCost_Tot[1:10,])%>%arrange(variable,desc(value)) 
EconomicCostTot_Molten<-EconomicCostTot_Molten[1:20,]%>%rename(Types_of_Damage=variable) #remove total cost
```

And that's it. We could continue to manipulate the data in a million other ways (such as figuring out the top disaster type in each state), but at this point we have plenty of material to answer the questions at hand.  It's time to see our results.


# Results

When we started this analysis of the storm database from NOAA, we posed two questions:

**1. Across the United States, which types of events are most harmful with respect to population health?**
**2. Across the United States, which types of events have the greatest economic consequences?**

After thoroughly analyzing and summarizing the data, we've decided to approach both questions from two angles. The results of these approaches are below.

### Average Impacts and Costs


```r
g1<-ggplot(data=HumanImpact_Avg[1:20,],aes(x=Disaster_Type,y=HealthImpacts))+ #create avg aesthetics
    geom_bar(stat="identity",fill="forestgreen")+
    labs(x="Event Type",y="Health Impacts",title="Top 20 Natural Disasters by Avg Health Impact")+
    theme(axis.text.x=element_text(angle=90,size=12,hjust=1,vjust=-.1),
          axis.title=element_text(size=15),title=element_text(size=15))

g2<-ggplot(data=EconomicCost_Avg[1:10,],aes(x=Disaster_Type,y=TotalCost))+
    geom_bar(stat="identity",fill="firebrick1")+
    labs(x="Event Type",y="log10(Cost ($))",title="Top 10 Natural Disasters by Avg Economic Cost")+
    theme(axis.text.x=element_text(angle=90,size=12,hjust=1,vjust=-.1),
          axis.title=element_text(size=15),title=element_text(size=15))

grid.arrange(g1,g2,ncol=2) #print first set of graphs together
```

![](ReproducibleResearch_Assign2_files/figure-html/graphing1-1.png)<!-- -->

Our first approach was to look at the average human impacts (from fatalities and injuries combined) and average economic costs (from both property and crop damage). This approach is shown above.

1. The first graph clearly shows that, on average, **tsunamis** are the most harmful disaster events, with respect to population health. However, the next three most harmful disasters (which all have fairly similar effects on average) are **hurricanes**, **excessive heat** and just **heat**. So one could say that heat in general is collectively more harmful to population health than even tsunamis.

2. The second graph is slightly less clear, as it is a scaled graph of log10(cost). With this in mind, however, it quickly becomes evident that, on average, **hurricanes** have costs that are literal orders of magnitude higher than other disaster events. That being said, **storm surges** are not too too far behind. One thing to note, though, is that storm surges generally accompany hurricanes. So in reality the top two disasters on this graph are essentially one and the same, and their economic consequences are simply amplified further.


### Total Impacts and Costs


```r
#create total impact aesthetic
g3<-ggplot(data=HumanImpactTot_Molten,aes(x=EVTYPE,y=value,fill=Types_of_Impact))+ 
    geom_bar(stat="identity")+
    labs(x="Event Type",y="Count of Impacts",title="Top 20 Natural Disasters in US by Total Human Impact since 1996")+
    theme(axis.text.x=element_text(angle=90,size=12,hjust=1,vjust=-.1),
          axis.title=element_text(size=15),title=element_text(size=15),legend.text=element_text(size=12))

#create total cost aesthetic
g4<-ggplot(data=EconomicCostTot_Molten,aes(x=EVTYPE,y=value,fill=Types_of_Damage))+ 
    geom_bar(stat="identity")+
    labs(x="Event Type",y="Cost (billion $)",title="Top 10 Natural Disasters in US by Total Economic Cost since 1996")+
    theme(axis.text.x=element_text(angle=90,size=12,hjust=1,vjust=-.1),
          axis.title=element_text(size=15),title=element_text(size=15),legend.text=element_text(size=12))

print(g3) #print second set of graphs separately
```

![](ReproducibleResearch_Assign2_files/figure-html/graphing2-1.png)<!-- -->

```r
print(g4)
```

![](ReproducibleResearch_Assign2_files/figure-html/graphing2-2.png)<!-- -->

Our second approach wass to look at the total human impacts (from fatalities and injuries) and total economic costs (from property and crop damage). This approach is shown in our final two figures.

1. In terms of population health, **tornados** have clearly caused by far the most harm. However, from the stacked figure, it appears that **excessive heat** has actually caused slightly more fatalities. Also of note is the fact that, here, tsunamis aren't even on the list, in contrast to the average impact. This is likely due to tsunamis being a rare occurence in the US, as opposed to tornados which happen quite frequently.

2. As for economic consequences, this time **floods** cost the US the most by a long shot (the y axis is in terms of billions of dollars). In this case, though, **hurricanes** (which had the highest average costs) do at least come in second, at about half the total cost of floods. This is again likely do to the frequency of floods versus hurricanes. One other very notable item is that **draughts** are far an away the most costly disasters in terms of crop damage, as can be seen from the stacked bar on the left. In fact, while property damage is clearly quite a bit more costly than crop damage generally, the amount of crop damage caused by draughts makes it chart at high property damage levels.


### Overall Assessment

**Overall, we can probably state that excessive heat is the most dangerous disaster for population health, as it's present in the top 3 for *both* the average and the absolute approaches. Similarly we can safely say that hurricanes have the greatest economic consequences from a general standpoint.**


#### Additional Notes

It would not be wise to put too much weight on this analysis. Though we captured more than 99% of the values recorded, it is still limited in many ways. For one things, the recording of event types is extremely imprecise, and therefore some of our re-workin of the `EVTYPE` data may have been faulty. And considering the level of disarray of this variable, there are no doubt many other errors that we didn't see in this brief exercise. The improper coding of even just one crop or property damage exponent from a K or M to a B would throw out the entire analysis. Additionally, it is possible that one of the very few event types we ignored was a single record with *huge* human or economic impacts. Such an occurence could severely skew our results.

Another thing that would be worth considering is inflation adjusted figures in the economic analysis. Or we could observe the data year by year, to see the effects of climate change on natural disasters over time. Speaking of climate, it could be more useful to look at the results on a region by region or state by state basis, since the types of events in any particular region are heavily influenced by geography. The original storm dataset from NOAA has many additional variables to help with this kind of mapping.

Suffice it to say that while we can draw broad conclusions from our analysis, this is only a first step in really answering our core questions about natural disasters in the United State.


## Appendix

#### Data:

The data itself comes in the form of a comma-separated-value file compressed via the bzip2 algorithm to reduce its size. You can download the file here:

- [Storm Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2)

There is also documentation of the database available, where you can find how some of the variables are constructed/defined:

- [National Weather Service Storm Data Documentation](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf)  
- [National Climatic Data Center Storm Events FAQ](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2FNCDC%20Storm%20Events-FAQ%20Page.pdf)


#### Session info for reproducibility:


```r
print(sessionInfo())
```

```
R version 3.4.1 (2017-06-30)
Platform: x86_64-w64-mingw32/x64 (64-bit)
Running under: Windows 10 x64 (build 15063)

Matrix products: default

locale:
[1] LC_COLLATE=English_United States.1252 
[2] LC_CTYPE=English_United States.1252   
[3] LC_MONETARY=English_United States.1252
[4] LC_NUMERIC=C                          
[5] LC_TIME=English_United States.1252    

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base     

other attached packages:
[1] bindrcpp_0.2   gridExtra_2.3  ggplot2_2.2.1  reshape2_1.4.2
[5] dplyr_0.7.3    plyr_1.8.4    

loaded via a namespace (and not attached):
 [1] Rcpp_0.12.12     knitr_1.17       bindr_0.1        magrittr_1.5    
 [5] munsell_0.4.3    colorspace_1.3-2 R6_2.2.2         rlang_0.1.2     
 [9] stringr_1.2.0    tools_3.4.1      grid_3.4.1       gtable_0.2.0    
[13] htmltools_0.3.6  lazyeval_0.2.0   yaml_2.1.14      rprojroot_1.2   
[17] digest_0.6.12    assertthat_0.2.0 tibble_1.3.4     codetools_0.2-15
[21] glue_1.1.1       evaluate_0.10.1  rmarkdown_1.6    stringi_1.1.5   
[25] compiler_3.4.1   scales_0.5.0     backports_1.1.0  pkgconfig_2.0.1 
```
