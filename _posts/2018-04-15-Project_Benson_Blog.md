---
layout: post
title: New York City MTA Turnstile Data Analysis, PT1
---
# Overview
As a native New Yorker, I've been riding the MTA for over 20 years. I recently got the opportunity as a beginner data scientist to look at New York subway traffic using publicly available MTA turnstile data. This post is about pulling, cleaning, and ultimately making the data more digestible and understandable.  

Tools leveraged in this analysis include Python, Pandas, and Mathplotlib

======  

## Obtaining the Data

The New York City MTA has turnstile entry and exit data which can be found [here](http://web.mta.info/developers/turnstile.html). Each file is broken out by week, and consists of data from the trailing 7 days excluding the day in the title (For example, a file marked 'Saturday, April 14, 2018' would contain data from Saturday, April 7, 2018 @ 00:00:00 to Friday, April 13, 2018 @ 23:59:59).  

On the site, there are explanations of what individual columns in the file stand for. Here are some of the definitions:  

* STATION  = Represents the station name the device is located at
* ENTRIES  = The cumulative entry register value for a device
* EXITS    = The cumulative exit register value for a device  

------

![MTA Raw File](/images/MTA_Raw_File.png "Raw File")  

After downloading a few files and pulling them together, we're ready to start cleaning. But first, a few notes on the consolidated datafile:  

1. Each row is broken out by a 4 hour timeframe
2. 'ENTRIES' and 'EXITS' are cumulative- this means that the actual amount of traffic in one given time frame must be the difference between the current row and the row preceding it.
3. Different naming conventions- quite egregious in the LINENAME column, there are multiple variations of the same combination of subway lines, i.e. 59 ST- 'NQR456W' is the same as 59 ST- '456WNQR', but creates  separate rows for the two. In addition, there are various subway stations with the same name but different subway lines

======  

## Cleaning the Data

Let's first tidy up the date and time columns, and make it something easier to leverage going forward:

```python
#Consolidates Date and Time Columns into datetime format
df['DateTime']= df['DATE'] + " " + df['TIME']
df.DateTime= pd.to_datetime(df.DateTime)
```

Next, let's consolidate the Stations and subway lines associated with them:

```python
#sorts string in LINENAME and appends new string to STATION
ID=["".join(sorted(x)) for x in df['LINENAME']]
df['STATID']=df['STATION']+" "+(ID)
```

```python
#sorts for most granular combinuations, then takes the difference. Expect to see that the first entry of every combination is NaN
df= df.sort_values(by=['C/A','UNIT','SCP','STATID','DateTime'])
x=df.sort_values(by=['C/A','UNIT','SCP','STATID','DateTime'])[['C/A','UNIT','SCP','STATID','ENTRIES']].groupby(['C/A','UNIT','SCP','STATID']).diff()['ENTRIES']
y=df.sort_values(by=['C/A','UNIT','SCP','STATID','DateTime'])[['C/A','UNIT','SCP','STATID','EXITS']].groupby(['C/A','UNIT','SCP','STATID']).diff()['EXITS']
df['Entry_diff']= list(x)
df['Exit_diff']= list(y)
```


![Description statistics](/images/Raw_Describe.png "Describing Statistics by Station")

By taking a look at the summary statistics across each station, there were a few things that stuck out. There were stations were minimums or maximums were extremely far away from the median, thus presenting an outlier. This is likely due to the way the 'ENTRIES' and 'EXITS' fields are tallied, and we can attribute this to turnstile mechanical resets.

After attempting to isolate these anomalies(will revisit in the future to identify outliers more scientifically and Pythonically), we can start digesting the data.

## Analyzing the Data

![Average Ridership](/images/Daily Average Ridership.png "Average Ridership")

Taking a look at average ridership by station, we can see that the majority of ridership is centered around huge travel hubs which may have non-subway transportation connections (34 ST PENN STATION, Grand Central 42 Street, Times square 42 ST).

![Daily Breakout](/images/Day_by_Day.png "Daily Breakout")

Ridership tends to peak towards the middle of the week, and drops off leading and through the weekend.

![Weekend Neighborhoods](/images/Weekend_Neighborhoods.png "Daily Breakout")

By appending a separate dataset for zipcode and neighborhood, we are able to discover out ridership distribution in neighborhoods over the weekend. We can see that neighborhoods such as 'West Queens','Lower East Side', and 'Greenpoint' are generate the most volume.

## Lessons Learned/ Key Takeaways
1. Data is often times messy.
2. There so many different ways to cut data and draw conclusions.
3. Incorporate exception cases and try/assert functions to properly identify outliers.
