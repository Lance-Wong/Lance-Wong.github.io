---
layout: post
title: ChooseBrew
description: Creating a hybrid recommender using Natural Language Processing and user reviews
img: beer-flight-samples-stock-photo-free-image-social-media-instagram.jpg
---

# Overview
Picture this: It's Friday night. You promised that you would hang out with some of old college buddies who you haven't seen for years (and frankly, you had wanted to keep that way), and they suggest a beer night. So, you leave your apartment anticipating the nightmares of Keystone Light.  

But wait? What's that? You're going to Other Half out in Brooklyn? Seems like your friends are now beer snobs, and you have some catching up to do. But when every beer has notes of this, a hint of that, how can you even begin to navigate the world of beer?

ChooseBrew is an application that I built that uses natural language processing to extract a beer's intrinsic flavor profile, and then leverages user ratings to create a collaborative filter recommender similar beers.

# Obtaining and Cleaning the data

In order to get started, I needed to find reviews to create my corpus. Using BeerAdvocate.com as my site to scrape, I used BeautifulSoup to scrape metadata and ratings from over 9000 beers.  

After a bit of filtering, I was able to get 9156 beers with 130 ratings or more. This came out to 1.2 million ratings total, but only 25% of the ratings had actual reviews. At the end of the day, my corpus had 311K documents, where each document was a review for a given beer.

# TF-IDF
After lemmatizing my corpus and removing for stopwords, I used a combination of tuning my TF-IDF Vectorizer and Non-negative Matrix Factorization hyperparameters to find the most succinct topics in my corpus. I relied on *min_df* and *max_df* parameters to pare down the amount of fluffy words (think "beer","hops","drink") and unique-but-not-useful words in my corpus. 


I also played around with the number of topics (n_components) through NMF to make my topics as succinct as possible. I ended up with these topics after many iterations:

![flavor profiles](/assets/img/flavors.png)