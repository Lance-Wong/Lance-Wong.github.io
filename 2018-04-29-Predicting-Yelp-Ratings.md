---
layout: post
title: Predicting Yelp Ratings
---
# Overview
Running a restaurant in New York City, or any city for that matter, is exceptionally difficult.

The board of a major restaurant group has approached me and wants to open up a new restaurant in the country. They don't really know what they want to open but are confident that their chefs will create an appropriate menu that will satisfy all palates.

Though they are seasoned veterans in the restaurant business, their presence in media has been less than average, but they know that Yelp Reviews are extremely important. They have created a few Michelin Star restaurants in the past.

They tasked me to identify the most important factors a restaurant should keep in mind to maximize public performance.

#Methodology
The world of restaurants ratings can be broken down into 3 groups:
1. High-tier review guides such as Michelin that come out every year.
2. Mid-tier review publications such as Zagat, Infatuation that review on a rolling basis
3. Low-tier crowd-sourcing ratings and reviews such as Yelp or TripAdvisor, which are posted on a fairly frequent basis.

Taking a top down approach, I will pull my dataset based off of the Michelin Guide, and combine that with Zagat and Yelp reviews.

This is a linear regression problem, so I will establish the dependent continuous variable as an average Yelp Rating.

Tools that will be leveraged include BeautifulSoup, Selenium for data scraping, and statsmodels, Sci-kit learn for regression modeling.

# Obtaining the data

## Michelin Stars

The Michelin Guide is a great website to use BeautifulSoup. After scraping the site for individual restaurant links, I ran a function that goes through each link and outputs a list of metrics I want to use.

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import numpy as np

def michelin_information(master):
    michelin_dictionary=[]
    #Goes through each link and scrapes the key information
    for links in master:
        clean_entry=[]

        restaurant_page=links
        response = requests.get(restaurant_page)
        text = response.text
        soup = BeautifulSoup(text,"html5lib")

        header=header[0:-2]
        cuisine= soup.find(class_="content-header-desc__detail content-header-desc__cuisine").find('a').text
        neighborhood= soup.find(class_="content-header-desc__detail content-header-desc__area").find('a').text

        characteristics = soup.find(class_='v-content__restaurant-criteria').find_all(class_="restaurant-criteria__icon")
        distinction = characteristics[0].text
        classification = characteristics[1].text
        price=characteristics[-1].text


        description = str(soup.find(class_='restaurant-desc').text)
        address= soup.find(class_='location-item__desc').find_all('p')[1].text
        entry=[header, cuisine, neighborhood, distinction, classification, price, description,address]
        michelin_dictionary += [clean_entry]
    return michelin_dictionary
```

## Zagat reviews

Zagat.com has a fairly unwieldy search function and does not work well with BeautifulSoup. Instead, I used Selenium and Google Search to find the Zagat reviews. It should be noted that about 15% of the links did not direct me to the actual restaurant review page, and Selenium would sometimes skip over the entry even though the entry existed. After receiving the first pass of data, I ran the function again to fill up the skipped entries.
