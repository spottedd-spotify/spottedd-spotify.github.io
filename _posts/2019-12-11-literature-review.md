---
title: "Spotify Project Literature Review"
date: 2019-12-11
author_profile: true
excerpt: "Spotify Project Literature Review"
---

The following resources helped us in our initial research, data scraping, 
data infrastructure, model building, and final analysis: 

## Data Scraping
- [1 million playlist data set](https://recsys-challenge.spotify.com/): Used to understand our initial playlist data
- [Spotify API documentation](https://developer.spotify.com/documentation/web-api/reference/tracks/get-audio-features/): Used when we built script to extract audio features from the Spotify API
- [Spotify oauth code flow](https://developer.spotify.com/documentation/general/guides/authorization-guide/#authorization-code-flow): We initialized a connection to the Spotify API using the oath code flow documented here
- [Spotipy documentation](https://spotipy.readthedocs.io/en/latest/) Spotipy is a Python library that allows you to easily call the Spotify API. 
- [Will Soares' Spotify-Genius integration](https://dev.to/willamesoares/how-to-integrate-spotify-and-genius-api-to-easily-crawl-song-lyrics-with-python-4o62) We found this useful when learning to use the Spotipy library.

## Data Infrastructure
- [GCP BigQuery documentation](https://cloud.google.com/bigquery/docs/): Used to learn how to load data into Google Cloud Platform's BigQuery  
- [GCP service accounts](https://cloud.google.com/iam/docs/service-accounts): Documentation for service accounts in GCP  
- [GCP python authetication documentation](https://cloud.google.com/docs/authentication/getting-started#auth-cloud-implicit-python): Documentation for authorization handling in Python

## Model Building
### K-Means:
- [University of Washington's k-means course on Coursera](https://www.coursera.org/learn/ml-clustering-and-retrieval): Learned to how implement k-means using this course.
- [Towards Data Science: K-Means](https://towardsdatascience.com/understanding-k-means-clustering-in-machine-learning-6a6e67336aa1): A useful high-level summary of the k-means algorithm

### K-NN:
- [kNN classification](https://github.com/Harvard-IACS/2019-CS109A/tree/master/content/lectures/lecture10/presentation)

### Logistic Regression:
- [Logistic regression](https://github.com/Harvard-IACS/2019-CS109A/tree/master/content/lectures/lecture10/presentation)

### Monte Carlo:
- [Monte Carlo method overview](https://en.wikipedia.org/wiki/Monte_Carlo_method)