---
title: "Contents"
permalink: /contents/
header:
    image: "/images/spotify.jpeg"
---

- [Background](https://spottedd-spotify.github.io/contents/#background)
- [Data Engineering](https://spottedd-spotify.github.io/contents/#data-engineering)

# Background
- Mark write-up 

# Data Engineering 
We encountered multiple issues with our dataset. For one, the “playlist-song” dataset comprised of just
under 70 million rows total. A dataset like this was far too large to manage locally on our machines without
being confined to importing selected CSVs as a subsample, which would be significantly limiting down the
road.

Thus, we decided to host our data on Google Cloud Platform (GCP) via their distributed big data store,
BigQuery. This would allow us to efficiently query and manipulate any subset of the million playlist dataset
using their SQL-like interface. After some research, we were able to configure our setups such that we could
retrieve the data through BigQuery and store it in our Jupyer notebooks to perform the EDAs locally.
Ultimately, this setup allowed us to analyze larger datasets; instead of confining ourselves to one CSV of
~70,000 rows, we were able to conduct analysis performantly on data sets of up to 500,000––and
theoretically more, though we decided that this was a decent cutoff point.
Additionally, moving our data over into GCP proved invaluable during the initial steps of querying each song
through the Spotify features API. Since songs are duplicated across playlists, using BigQuery, we were able
to extract a uniquified dataset of just 2 million songs from the 70 million rows and save them to a separate
table. With this technique, a scraping endeavor that would’ve taken an estimated 19+ hours total was
reduced to under an hour. From there, it was a matter of performing a simple outer join query on the original
songs dataset to associate the features back to each playlist-song row. Lastly, this ability to isolate and
query uniquified songs enabled us to sample a truly random subset of the song data (without including
duplicate songs or introducing bias by selecting songs clustered around a set of playlists in a given CSV) for
our EDA.

Further challenges arose when we discovered that the playlist IDs were unique only to each file, as opposed
to being a globally unique playlist ID (each file included playlists with a PID of 0 - 999). This would become
an issue for any analyses we wanted to run on playlist-specific data. To account for this, we re-processed
each CSV, adding an additional column called “unique_pid” which combined the file number with the playlist
ID, and re-processed the data in BQ