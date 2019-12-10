---
title: "Contents"
permalink: /contents/
header:
    image: "/images/spotify.jpeg"
gallery:
  - url: /images/content-eda/n_cluster_comparison.svg
    image_path: /images/content-eda/n_cluster_comparison.svg
    alt: "n cluster comparison"
    title: "n by cluster"
  - url: /images/content-eda/danceability_cluster_comparison.svg
    image_path: /images/content-eda/danceability_cluster_comparison.svg
    alt: "danceability cluster comparison"
    title: "danceability by cluster"
  - url: /images/content-eda/duration_ms_cluster_comparison.svg
    image_path: /images/content-eda/duration_ms_cluster_comparison.svg
    alt: "duration cluster comparison"
    title: "duration by cluster"
  - url: /images/content-eda/energy_cluster_comparison.svg
    image_path: /images/content-eda/energy_cluster_comparison.svg
    alt: "energy cluster comparison"
    title: "energy by cluster"
  - url: /images/content-eda/instrumentalness_cluster_comparison.svg
    image_path: /images/content-eda/instrumentalness_cluster_comparison.svg
    alt: "instrumentalness cluster comparison"
    title: "instrumentalness by cluster"
  - url: /images/content-eda/key_cluster_comparison.svg
    image_path: /images/content-eda/key_cluster_comparison.svg
    alt: "key cluster comparison"
    title: "key by cluster"
  - url: /images/content-eda/liveness_cluster_comparison.svg
    image_path: /images/content-eda/liveness_cluster_comparison.svg
    alt: "liveness cluster comparison"
    title: "liveness by cluster"
  - url: /images/content-eda/loudness_cluster_comparison.svg
    image_path: /images/content-eda/loudness_cluster_comparison.svg
    alt: "loudness cluster comparison"
    title: "loudness by cluster"           
  - url: /images/content-eda/mode_cluster_comparison.svg
    image_path: /images/content-eda/mode_cluster_comparison.svg
    alt: "mode cluster comparison"
    title: "mode_cluster_comparison.svg by cluster"
  - url: /images/content-eda/mode_cluster_comparison.svg
    image_path: /images/content-eda/mode_cluster_comparison.svg
    alt: "mode cluster comparison"
    title: "mode by cluster"    
  - url: /images/content-eda/speechiness_cluster_comparison.svg
    image_path: /images/content-eda/speechiness_cluster_comparison.svg
    alt: "speechiness cluster comparison"
    title: "speechiness by cluster"
  - url: /images/content-eda/tempo_cluster_comparison.svg
    image_path: /images/content-eda/tempo_cluster_comparison.svg
    alt: "tempo cluster comparison"
    title: "tempo by cluster"
  - url: /images/content-eda/time_signature_cluster_comparison.svg
    image_path: /images/content-eda/time_signature_cluster_comparison.svg
    alt: "time signature cluster comparison"
    title: "time signature by cluster"    
  - url: /images/content-eda/valence_cluster_comparison.svg
    image_path: /images/content-eda/valence_cluster_comparison.svg
    alt: "valence cluster comparison"
    title: "valence by cluster"
images:
  - image_path: /images/content-baseline-modeling/Mozart_PianoConcertoNo.21inCMajor_nearestneighbors.svg
    title: "Mozart: Piano Concerto No.21 in C Major"
  - image_path: /images/content-baseline-modeling/KendrickLamar_alright_nearestneighbors.svg
    title: "Kendrick Lamar: alright"
  - image_path: /images/content-baseline-modeling/Slayer_RainingBlood_nearestneighbors.svg
    title: "Slayer: Raining Blood"   

---

- [Background](https://spottedd-spotify.github.io/contents/#background)
- [Data Engineering](https://spottedd-spotify.github.io/contents/#data-engineering)
- [EDA](https://spottedd-spotify.github.io/contents/#eda)
- [Baseline Modeling](https://spottedd-spotify.github.io/contents/#baseline-modeling)
- [Model Refinement](https://spottedd-spotify.github.io/contents/#model-refinement)
- [Appendixes](https://spottedd-spotify.github.io/contents/#appendixes)

# Background
- Mark write-up 

# Data Engineering 
We encountered multiple issues with our dataset. For one, the “playlist-song” 
dataset comprised of just under 70 million rows total. A dataset like this was 
far too large to manage locally on our machines without being confined to 
importing selected CSVs as a subsample, which would be significantly limiting 
down the road.

Thus, we decided to host our data on Google Cloud Platform (GCP) via their 
distributed big data store, BigQuery. This would allow us to efficiently query 
and manipulate any subset of the million playlist dataset using their SQL-like 
interface. After some research, we were able to configure our setups such that 
we could retrieve the data through BigQuery and store it in our Juptyer 
notebooks to perform the EDAs locally. Ultimately, this setup allowed us to 
analyze larger datasets; instead of confining ourselves to one CSV of ~70,000 
rows, we were able to conduct analysis performantly on data sets of up to 
500,000 and theoretically more, though we decided that this was a decent cutoff
point.

Additionally, moving our data over into GCP proved invaluable during the initial
steps of querying each song through the Spotify features API. Since songs are 
duplicated across playlists, using BigQuery, we were able to extract a uniquified
dataset of just 2 million songs from the 70 million rows and save them to a 
separate table. With this technique, a scraping endeavor that would’ve taken an 
estimated 19+ hours total was reduced to under an hour. From there, it was a 
matter of performing a simple outer join query on the original songs dataset to 
associate the features back to each playlist-song row. Lastly, this ability to 
isolate and query uniquified songs enabled us to sample a truly random subset of 
the song data (without including duplicate songs or introducing bias by 
selecting songs clustered around a set of playlists in a given CSV) for our EDA.

Further challenges arose when we discovered that the playlist IDs were unique 
only to each file, as opposed to being a globally unique playlist ID (each file 
included playlists with a PID of 0 - 999). This would become an issue for any 
analyses we wanted to run on playlist-specific data. To account for this, we 
re-processed each CSV, adding an additional column called “unique_pid” which 
combined the file number with the playlist ID, and re-processed the data in BQ.


# EDA
In this section, we'll briefly recap our EDA from milestone 3. We began by 
examining the correlations of the audio features in our data set. The 
correlogram below depicts correlations that are intuitive. For example, 
acousticness is strongly negatively correlated with loudness and energy. On the 
other hand, energy is positively correlated with loudness and valence (the 
musical “positiveness” associated with a track). In conclusion, although there 
is some multi-collinearity between the features, we can still include them all, 
given that their absolute correlations are under 0.75

![Audio Features Correlogram](/images/content-eda/audio_features_correlogram.svg "Audio features correlogram")

The next step in our analysis was to find similarities between the songs in our 
dataset. To do so, we applied k-means clustering to our data. K-means is an 
unsupervised learning method that groups data into clusters according to the 
mean value of their features. At a high level, it works by initializing a 
user-specified number of k centroids, then assigns each data point to the 
closest centroid [Towards Data Science: K-Means](https://towardsdatascience.com/understanding-k-means-clustering-in-machine-learning-6a6e67336aa1)

We identified 12 salient clusters to group our data into. As you can see in the 
plots below, some features (_speechiness, key_) are quite helpful in 
differentiating our songs, while some are virtually useless (_tempo_).

{% include gallery caption="Spotify audio features by k-means cluster" %}


# Baseline Modeling
In our first iteration, we used a combination of supervised and unsupervised 
learning to generate the initial recommendation approach:
1. **Labeling (K-means)**: cluster universe of all songs in all playlists into 
12 clusters based on spotify music features. This serves as a 1-dimensional 
label with 12 categories which makes computation extremely efficient if we run 
supervised learning over it.
2. **Classification (K-Nearest Neighbors)**: Upon receiving a new song that is 
not labeled with one of the 12 categories, we can use K-nearest neighbors based
on musical features to identify the cluster in which this new song falls. We 
found that the model with `K=100` gave us the highest cross-validation scores 
(`mean=97.8%`) and are confident in its ability to correctly put the new
song in an appropriate cluster.

#### Labeling (K-means) Results
As an example, the below shows the cluster assignment generated by our model 
for the following three songs: 
- Mozart: Piano Concerto No.21 in C Major
- Kendrick Lamar: alright
- Slayer: Raining Blood

![Sample clusters](/images/content-baseline-modeling//sample_clusters.png)

Let's compare the cluster assignments for Slayer's Raining Blood, and Mozart's
Piano Concerto No.21 in C Major. Clearly, these are two songs that couldn't be 
more different. Slayer is a 1980's [thrash metal](https://en.wikipedia.org/wiki/Thrash_metal)
band known for its aggression, "fast, percussive beats, low-register guitar 
riffs, and shredding-style lead guitar work". Mozart on the other hand is, well,
Mozart. 

First, examining Mozart's cluster (F) in the plots above, we can see that this
is the cluster associated with the lowest levels of loudness, energy, tempo, and
speechiness, and the highest levels of acousticness and instrumentalness of all 
our clusters. In short, this Mozart Piano Concerto has been assigned to a quiet, 
slow, calm, and soothing cluster with low levels of vocals.

The cluster for Slayer's Raining Blood (A), on the other hand, is associated 
with the highest levels of loudness, energy, liveness, and tempo in all our 
clusters. In addition, cluster A has some of the lowest levels of acousticness 
and instrumentalness. It also has the second highest level of speechiness. In
summary, Raining Blood's cluster is one associated with loud, fast-paced, highly
energetic music featuring a high level of vocals.

Thus, the qualitative analysis of our cluster assignments validates the 
groupings that k-means created for us. 

#### Classification (K-Nearest Neighbors)
In addition, we can also use our model to make song recommendations. Below, you 
can see the top 10 "closest" songs for each of the following three songs above:

<ul class="photo-gallery">
  {% for image in page.images %}
    <li><img src="{{ image.image_path }}" alt="{{ image.title}}"/></li>
  {% endfor %}
</ul>

As we can see, the top recommendations for Mozart's Piano Concerto are pieces by
composers such as Schubert, Dvorak, Einaudi, and Holst. It also features several
instrumental tracks from movie soundtracks. For Kendrick Lamar's punchy hip-hop anthem, _alright_, we have recommended songs 
by other hip-hop artists such as Lil Jon, Lexxmatiq, and the Notorious B.I.G.
Finally, our recommendations for Slayer's _Raining Blood_ feature songs by 
similar thrash and death metal bands such as Necrophagist, Amon Amarth, 
Carnifex, and Judas Priest.

Interestingly, for both _Raining Blood_ and _alright_, the top recommended song
is the song itself. This is due to the fact that these songs are present in 
different albums by the same artist. As you can see the "distance" for these 
duplicate songs from the target song is zero, which makes sense.


# Model Refinement


# Appendixes
Finally, we've included several [appendixes](https://spottedd-spotify.github.io/appendixes/) that detail the nuts and bolts of our data engineering, infrastructure, and scraping efforts.
- [Data Infrastructure](https://spottedd-spotify.github.io/data-infrastructure/)
- [Data Scraping](https://spottedd-spotify.github.io/spotify-data-scraping/)