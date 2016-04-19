---
layout: post
comments: true
title:  "Most harmful weather event types"
date:   2016-04-14 16:09:02 +0000
categories: data-science R dplyr reshape
---

Storms and other severe weather events can cause both public health and economic problems for communities and municipalities. Many severe events can result in fatalities, injuries, and property damage, and preventing such outcomes to the extent possible is a key concern.

In this article, I describe an analysis made over the U.S. National Oceanic and Atmospheric Administration’s (NOAA) storm database to discover which types of events are most harmful with respect to population health and those that have the greatest economic consequences.

This work is an assignment of Reproducible Research course, which is part of John Hopkins Data Science specialization that I am taking at Coursera.

# Downloading and loading NOAA data

```R
download.file('https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2', destfile='stormdata.csv.bz2')
storm_data <- read.csv("stormdata.csv.bz2", strip.white = TRUE)
```

# Exploring the dataframe

This is a relatively big data frame with many columns. So, the first thing to do is explore it and find out the relevant columns:

```R
str(strom_data)
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
 $ BGN_LOCATI: Factor w/ 54429 levels ""," Christiansburg",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ END_DATE  : Factor w/ 6663 levels "","1/1/1993 0:00:00",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ END_TIME  : Factor w/ 3647 levels ""," 0900CST",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
 $ COUNTYENDN: logi  NA NA NA NA NA NA ...
 $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
 $ END_AZI   : Factor w/ 24 levels "","E","ENE","ESE",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ END_LOCATI: Factor w/ 34506 levels ""," CANTON"," TULIA",..: 1 1 1 1 1 1 1 1 1 1 ...
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
 $ WFO       : Factor w/ 542 levels ""," CI","%SD",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ STATEOFFIC: Factor w/ 250 levels "","ALABAMA, Central",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ ZONENAMES : Factor w/ 25112 levels "","                                                                                                                               "| __truncated__,..: 1 1 1 1 1 1 1 1 1 1 ...
 $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
 $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
 $ LATITUDE_E: num  3051 0 0 0 0 ...
 $ LONGITUDE_: num  8806 0 0 0 0 ...
 $ REMARKS   : Factor w/ 436781 levels "","\t","\t\t",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
```

The main goal of this article is compare the event types with each other in respect of how harmful they are. The event type values are stored in the `$EVTYPE` column

To analyze the damage against human health, we need to focus on `$FATALITIES` and `$INJURIES` columns. To think about the damage against economy the focus changes to `$CROPDMGEXP`, `$CROPDMG`, `$PROPDMG` and `$PROPDMGEXP` columns.

But first, I should do some transformations on `$EVTYPE` column.

# Event type transformations

The records from NOAA storm database have 985 distincts event types.

```R
length(levels(storm_data$EVTYPE))
```

```
## [1] 985
```

Many of those are closely related, for example:

```R
hail_idx <- grep('hail', levels(storm_data$EVTYPE),  ignore.case = TRUE)
levels(storm_data$EVTYPE)[hail_idx]
```

```
##  [1] "DEEP HAIL"                  "FUNNEL CLOUD/HAIL"         
##  [3] "GUSTY WIND/HAIL"            "HAIL"                      
##  [5] "HAIL 0.75"                  "HAIL 0.88"                 
##  [7] "HAIL 075"                   "HAIL 088"                  
##  [9] "HAIL 1.00"                  "HAIL 1.75"                 
## [11] "HAIL 1.75)"                 "HAIL 100"                  
## [13] "HAIL 125"                   "HAIL 150"                  
## [15] "HAIL 175"                   "HAIL 200"                  
## [17] "HAIL 225"                   "HAIL 275"                  
## [19] "HAIL 450"                   "HAIL 75"                   
## [21] "HAIL 80"                    "HAIL 88"                   
## [23] "HAIL ALOFT"                 "HAIL DAMAGE"               
## [25] "HAIL FLOODING"              "HAIL STORM"                
## [27] "Hail(0.75)"                 "HAIL/ICY ROADS"            
## [29] "HAIL/WIND"                  "HAIL/WINDS"                
## [31] "HAILSTORM"                  "HAILSTORMS"                
## [33] "LATE SEASON HAIL"           "MARINE HAIL"               
## [35] "NON SEVERE HAIL"            "small hail"                
## [37] "Small Hail"                 "SMALL HAIL"                
## [39] "THUNDERSTORM HAIL"          "THUNDERSTORM WIND/HAIL"    
## [41] "THUNDERSTORM WINDS HAIL"    "THUNDERSTORM WINDS/ HAIL"  
## [43] "THUNDERSTORM WINDS/HAIL"    "THUNDERSTORM WINDSHAIL"    
## [45] "TORNADOES, TSTM WIND, HAIL" "TSTM WIND/HAIL"            
## [47] "WIND/HAIL"
```

I should group those closely related event type values in order to make the comparison practicable.

NOTE: Here, as I am not a specialist on weather issues, I might have done a few wrong guesses about some relations of event types.

```R
levels(storm_data$EVTYPE) <- tolower(levels(storm_data$EVTYPE))
levels(storm_data$EVTYPE)[grep('frost', levels(storm_data$EVTYPE))] = 'ice/snow'
levels(storm_data$EVTYPE)[grep('freez', levels(storm_data$EVTYPE))] = 'ice/snow'
levels(storm_data$EVTYPE)[grep('blizzard', levels(storm_data$EVTYPE))] = 'ice/snow'
levels(storm_data$EVTYPE)[grep('fog', levels(storm_data$EVTYPE))] = 'fog/smoke'
levels(storm_data$EVTYPE)[grep('smoke', levels(storm_data$EVTYPE))] = 'fog/smoke'
levels(storm_data$EVTYPE)[grep('snow', levels(storm_data$EVTYPE))] = 'ice/snow'
levels(storm_data$EVTYPE)[grep('ice', levels(storm_data$EVTYPE))] = 'ice/snow'
levels(storm_data$EVTYPE)[grep('dust', levels(storm_data$EVTYPE))] = 'dustStorm'
levels(storm_data$EVTYPE)[grep('storm', levels(storm_data$EVTYPE))] = 'regular storm'
levels(storm_data$EVTYPE)[grep('volcanic', levels(storm_data$EVTYPE))] = 'volcano'
levels(storm_data$EVTYPE)[grep('urban', levels(storm_data$EVTYPE))] = 'urban stream'
levels(storm_data$EVTYPE)[grep('wet', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('wind', levels(storm_data$EVTYPE))] = 'wind/typhoon/tornado/hurricane'
levels(storm_data$EVTYPE)[grep('tornad', levels(storm_data$EVTYPE))] = 'wind/typhoon/tornado/hurricane'
levels(storm_data$EVTYPE)[grep('typhoon', levels(storm_data$EVTYPE))] = 'wind/typhoon/tornado/hurricane'
levels(storm_data$EVTYPE)[grep('hurricane', levels(storm_data$EVTYPE))] = 'wind/typhoon/tornado/hurricane'
levels(storm_data$EVTYPE)[grep('dry', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('warm', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('dry', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('tstm', levels(storm_data$EVTYPE))] = 'storm'
levels(storm_data$EVTYPE)[grep('cold', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('cool', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('heat', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('hypotherm', levels(storm_data$EVTYPE))] = 'hypothermia'
levels(storm_data$EVTYPE)[grep('hail', levels(storm_data$EVTYPE))] = 'hail'
levels(storm_data$EVTYPE)[grep('hot', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('temperature', levels(storm_data$EVTYPE))] = 'temperature and wet'
levels(storm_data$EVTYPE)[grep('thunder', levels(storm_data$EVTYPE))] = 'storm'
levels(storm_data$EVTYPE)[grep('winter', levels(storm_data$EVTYPE))] = 'winter'
levels(storm_data$EVTYPE)[grep('flood', levels(storm_data$EVTYPE))] = 'flood'
levels(storm_data$EVTYPE)[grep('waterspout', levels(storm_data$EVTYPE))] = 'waterspout'
levels(storm_data$EVTYPE)[grep('fire', levels(storm_data$EVTYPE))] = 'fire'
levels(storm_data$EVTYPE)[grep('rip current', levels(storm_data$EVTYPE))] = 'seas'
levels(storm_data$EVTYPE)[grep('seas', levels(storm_data$EVTYPE))] = 'seas'
levels(storm_data$EVTYPE)[grep('surf', levels(storm_data$EVTYPE))] = 'seas'
levels(storm_data$EVTYPE)[grep('marine', levels(storm_data$EVTYPE))] = 'seas'
levels(storm_data$EVTYPE)[grep('wave', levels(storm_data$EVTYPE))] = 'seas'
levels(storm_data$EVTYPE)[grep('lightning', levels(storm_data$EVTYPE))] = 'lightning'
levels(storm_data$EVTYPE)[grep('rain', levels(storm_data$EVTYPE))] = 'rain'
```

There are also event types that doesn’t tell anything about an event type, for example:

```R
smridx <- grep('summary', levels(storm_data$EVTYPE),  ignore.case = TRUE)
levels(storm_data$EVTYPE)[smridx]
```

```
##  [1] "summary august 10"      "summary august 11"     
##  [3] "summary august 17"      "summary august 2-3"    
##  [5] "summary august 21"      "summary august 28"     
##  [7] "summary august 4"       "summary august 7"      
##  [9] "summary august 9"       "summary jan 17"        
## [11] "summary july 23-24"     "summary june 18-19"    
## [13] "summary june 5-6"       "summary june 6"        
## [15] "summary of april 12"    "summary of april 13"   
## [17] "summary of april 21"    "summary of april 27"   
## [19] "summary of april 3rd"   "summary of august 1"   
## [21] "summary of july 11"     "summary of july 2"     
## [23] "summary of july 22"     "summary of july 26"    
## [25] "summary of july 29"     "summary of july 3"     
## [27] "summary of june 10"     "summary of june 11"    
## [29] "summary of june 12"     "summary of june 13"    
## [31] "summary of june 15"     "summary of june 16"    
## [33] "summary of june 18"     "summary of june 23"    
## [35] "summary of june 24"     "summary of june 3"     
## [37] "summary of june 30"     "summary of june 4"     
## [39] "summary of june 6"      "summary of march 14"   
## [41] "summary of march 23"    "summary of march 24"   
## [43] "summary of march 24-25" "summary of march 27"   
## [45] "summary of march 29"    "summary of may 10"     
## [47] "summary of may 13"      "summary of may 14"     
## [49] "summary of may 22"      "summary of may 22 am"  
## [51] "summary of may 22 pm"   "summary of may 26 am"  
## [53] "summary of may 26 pm"   "summary of may 31 am"  
## [55] "summary of may 31 pm"   "summary of may 9-10"   
## [57] "summary sept. 25-26"    "summary september 20"  
## [59] "summary september 23"   "summary september 3"   
## [61] "summary september 4"    "summary: nov. 16"      
## [63] "summary: nov. 6-7"      "summary: oct. 20-21"   
## [65] "summary: october 31"    "summary: sept. 18"
```

The records with thoses values as event type are useless for the goal of this article, so it`s reasonable removing them from data frame.

```R
storm_data <- storm_data[!grepl('summary', storm_data$EVTYPE), ]
storm_data <- droplevels(storm_data) 
```

Now, there are 108 distinct event type in the storm_data data frame.

```R
length(levels(storm_data$EVTYPE))
```

```
## [1] 108
```

# Analysis of human health damage by event type

I'll describe in this section the analysis about the human health damage by event types.

I should first arrange the data records, grouping by the event type and ordering by number of fatalities, and showing the number of injuries and number of fatalities.

Here, we can do it easily by chaining operations from **dplyr** library.

```R
fatalities_injuries_by_event <- storm_data  %>% 
  group_by(EVTYPE) %>% 
  summarize(injuries=sum(INJURIES), fatalities=sum(FATALITIES)) %>%
  arrange(desc(fatalities, injuries))

fatalities_injuries_by_event
```

```
## Source: local data frame [108 x 3]
## 
##                            EVTYPE injuries fatalities
##                            (fctr)    (dbl)      (dbl)
## 1  wind/typhoon/tornado/hurricane   101741       7004
## 2             temperature and wet     9550       3402
## 3                           flood     8602       1524
## 4                       lightning     5231        817
## 5                            seas      797        760
## 6                   regular storm     4260        522
## 7                        ice/snow     4166        379
## 8                       avalanche      170        224
## 9                            rain      280        100
## 10                           fire     1608         90
## ..                            ...      ...        ...
```

As we can see, the “wind/typhoon/tornado/hurricane” event type has a much higher number of injuries then other event types. That is suspicious. Let`s investigate a bit more.

## Arranging data by decades

Let’s see how these records are arranged by decades, focusing only on the top event types.

```R
top_evtypes <- fatalities_injuries_by_event$EVTYPE[1:7]
n_storm_data <- storm_data[storm_data$EVTYPE %in% top_evtypes,]
n_storm_data$YEAR = as.numeric(format(as.Date(n_storm_data$BGN_DATE, format = "%m/%d/%Y %H:%M:%S"), '%Y'))
n_storm_data$DECADE = as.integer(n_storm_data$YEAR /10) * 10

fatalities_injuries_by_event_decade <- n_storm_data  %>% 
       group_by(EVTYPE, DECADE) %>% 
       summarize(injuries=sum(INJURIES), fatalities=sum(FATALITIES)) %>%
       arrange(desc(fatalities, injuries))
```

Now, let’s reshape data for better visualization using **reshape** library.

First showing injuries values.

```R
melted_injuries_by_event_decade <- melt(fatalities_injuries_by_event_decade, c("EVTYPE", 'DECADE'), 'injuries')
injuries_by_event_decade_wide <- dcast(melted_injuries_by_event_decade, EVTYPE ~ DECADE)
injuries_by_event_decade_wide
```

```
##                           EVTYPE  1950  1960  1970  1980  1990  2000 2010
## 1                           seas    NA    NA    NA    NA   192   502  103
## 2                          flood    NA    NA    NA    NA  7473   773  356
## 3                      lightning    NA    NA    NA    NA  2238  2617  376
## 4 wind/typhoon/tornado/hurricane 14470 17265 21640 13232 15768 12407 6959
## 5            temperature and wet    NA    NA    NA    NA  4598  4101  851
## 6                       ice/snow    NA    NA    NA    NA  3707   440   19
## 7                  regular storm    NA    NA    NA    NA  2122  1421  717
```

and fatalities data:

```R
melted_fatalities_by_event_decade <- melt(fatalities_injuries_by_event_decade, c("EVTYPE", 'DECADE'), 'fatalities')
fatalities_by_event_decade_wide <- dcast(melted_fatalities_by_event_decade, EVTYPE ~ DECADE)
fatalities_by_event_decade_wide
```

```
##                           EVTYPE 1950 1960 1970 1980 1990 2000 2010
## 1                           seas   NA   NA   NA   NA  165  489  106
## 2                          flood   NA   NA   NA   NA  654  635  235
## 3                      lightning   NA   NA   NA   NA  351  411   55
## 4 wind/typhoon/tornado/hurricane 1419  942  998  699 1074 1157  715
## 5            temperature and wet   NA   NA   NA   NA 2117 1103  182
## 6                       ice/snow   NA   NA   NA   NA  294   81    4
## 7                  regular storm   NA   NA   NA   NA  215  224   83
```

We can see there are data only about wind/typhoon/tornado/hurricane event types before de 1990s. 

Probably, NOAA wasn't prepared to collect data about the other event types before the 1990s. This fact makes the comparison unfair. 

I should consider only data from after the 1990s (more specifically 1993, but we will omit this analysis for simplicity))

So, let us subset the data frame.

```R
storm_data$YEAR = as.numeric(format(as.Date(storm_data$BGN_DATE, format = "%m/%d/%Y %H:%M:%S"), '%Y'))
new_storm_data <- storm_data[storm_data$YEAR>1993,]	
```

And now, let`s see a fair comparison.

```R
fatalities_injuries_by_event <- new_storm_data  %>% 
  group_by(EVTYPE) %>% 
  summarize(injuries=sum(INJURIES), fatalities=sum(FATALITIES)) %>%
  arrange(desc(fatalities, injuries))

fatalities_injuries_by_event
```

```
## Source: local data frame [104 x 3]
## 
##                            EVTYPE injuries fatalities
##                            (fctr)    (dbl)      (dbl)
## 1             temperature and wet     9544       3378
## 2  wind/typhoon/tornado/hurricane    29392       2633
## 3                           flood     8560       1471
## 4                       lightning     5117        795
## 5                            seas      796        758
## 6                   regular storm     4025        477
## 7                        ice/snow     3597        333
## 8                       avalanche      170        224
## 9                            rain      266        100
## 10                           fire     1458         87
## ..                            ...      ...        ...
```

Those may be the most harmful to human health event types from 1993 to today.

## Event types with the greatest economic consequences

The columns $PROPDMG represents values of damage on properties, and $CROPDMG reprepresents values of damage on crops.

Both values have to be multiplied by `10 ^ x` where `x` is the correspondent exponential value  (`$PROPDMGEXP`or `$CROPDMGEXP`)

Let's see the levels of both columns

```R
levels(new_storm_data$PROPDMGEXP)
```

```R
levels(new_storm_data$CROPDMGEXP)
```

I should convert the values of `$PROPDMGEXP` and `$CROPDMGEXP` from literal to numeric.

```R
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='B']="9"
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='K']="3"
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='m']="6"
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='M']="6"
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='k']="3"
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='K']="3"
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='m']="6"
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='M']="6"
levels(new_storm_data$PROPDMGEXP)[levels(new_storm_data$PROPDMGEXP)=='']="0"

levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='B']="9"
levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='K']="3"
levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='m']="6"
levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='M']="6"
levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='k']="3"
levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='K']="3"
levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='m']="6"
levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='M']="6"
levels(new_storm_data$CROPDMGEXP)[levels(new_storm_data$CROPDMGEXP)=='']="0"

dmg_storm_data <- new_storm_data[new_storm_data$PROPDMGEXP %in% c('0', '1', '2', '3', '4', '6', '7', '8', '9'),]
dmg_storm_data <- dmg_storm_data[dmg_storm_data$CROPDMGEXP %in% c('0', '1', '2', '3', '4', '6', '7', '8', '9'),]

dmg_storm_data$PROPDMGEXP <- as.numeric(levels(dmg_storm_data$PROPDMGEXP))[dmg_storm_data$PROPDMGEXP]
dmg_storm_data$CROPDMGEXP <- as.numeric(levels(dmg_storm_data$CROPDMGEXP))[dmg_storm_data$CROPDMGEXP]
```
Now, we can calculate the actual damage and put that value in a new column `$TOTAL_DMG`.

```R
dmg_storm_data$TOTAL_DMG = dmg_storm_data$CROPDMG*(10^dmg_storm_data$CROPDMGEXP) + dmg_storm_data$PROPDMG*(10^dmg_storm_data$PROPDMGEXP)
```

Now, let's group, summarize and arrange data.

```R
dmg_by_event <- dmg_storm_data  %>% 
  group_by(EVTYPE) %>% 
  summarize(damage=sum(TOTAL_DMG)) %>%
  arrange(desc(damage))

dmg_by_event
```

```
## Source: local data frame [104 x 2]
## 
##                            EVTYPE       damage
##                            (fctr)        (dbl)
## 1                           flood 169161879613
## 2  wind/typhoon/tornado/hurricane 128450147639
## 3                           storm  64965912883
## 4                            hail  18314402084
## 5                         drought  14968172000
## 6                        ice/snow  12268074410
## 7                            fire   8285910130
## 8                            rain   3989680990
## 9             temperature and wet   2601044530
## 10                      lightning    888767867
## ..                            ...          ...
```

Those are the most harmful event types regarding damages to economy. 


