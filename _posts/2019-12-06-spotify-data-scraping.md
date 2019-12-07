---
title: "Scraping Audio Features From the Spotify API"
date: 2019-12-06
author_profile: true
excerpt: "Scraping Audio Features From the Spotify API"
header:
    image: "/images/spotify_api.png"
---

# Introduction
To fetch our song and playlist data, we started with the “Million Playlist Dataset”. This came from the [2018 Spotify Recsys Challenge](https://recsys-challenge.spotify.com/). Per the website, it contains 1 million playlists created by Spotify users, and contains identifying fields such as `trackuri`, `playlistid`, `track_name` and `artist_name`. 

After loading this data into our [GCP-hosted data platform](https://spottedd-spotify.github.io/data-infrastructure/), we then fetched audio features for each unique track. To do so, we wrote a python script that used the `trackuri` field to access audio data from the [Spotify API](https://developer.spotify.com/documentation/web-api/reference/tracks/get-audio-features/).  

# Spotify-Scraping Code
To fetch our audio features from the Spotify API, we leveraged [Spotipy](https://spotipy.readthedocs.io/en/latest/), a Python library that allows you to easily call the Spotify API. In addition we found the following [Will Soares' Spotify-Genius integration](https://dev.to/willamesoares/how-to-integrate-spotify-and-genius-api-to-easily-crawl-song-lyrics-with-python-4o62) useful when learning to use the Spotipy library. 

## 1. Initialize Spotify connection
First, using Spotify, we initialized a connection to the Spotify API using the [oauth code flow](https://developer.spotify.com/documentation/general/guides/authorization-guide/#authorization-code-flow)

```python
def initialize_spotify_connection():
    '''
    Instantiate spotify client object using authorization code flow:
    https://developer.spotify.com/documentation/general/guides/authorization-guide/#authorization-code-flow
    :return spotify: spotify api client object
    '''
    token = util.oauth2.SpotifyClientCredentials(
        client_id=my_settings.SPOTIPY_CLIENT_ID,
        client_secret=my_settings.SPOTIPY_CLIENT_SECRET
    )
    cache_token = token.get_access_token()
    spotify = spotipy.Spotify(cache_token)

    return spotify
```    

## 2. Extract audio data
The following functions do the bulk of the work. First, we started with a list of unique `trackuris` which we collected and cleaned using our GCP BigQuery instance. Once we had our list in csv form, we extracted all available audio features for each of those songs. 

There were a couple of tricky portions of the code below. The first was data size. By loading the data into BigQuery, and generating a list of _unique_ `trackuris` was huge. It reduced our starting dataset from ~70 million to ~2.2 million records. Nevertheless, loading 2.2 million records into memory was still unteneble. Thus, to prevent memory pressure concerns, we fetched the audio data in batches of 100.

Next, the Spotify authorization token expires every hour. To deal with this, we wrote code that re-instantiated the Spotify token if it expired.    

```python
def read_csv_in_chunks(path, spotify):
    '''
    For a given input csv, fetches song features in batches of 100 tracks at a
    time, then writes to csv by batch to handle memory pressure
    :param path: full filepath of csv location
    :param spotify: spotify client object
    '''
    # Reading by chunk
    # https://stackoverflow.com/questions/42900757/sequentially-read-huge-csv-file-in-python
    chunksize = 100

    for chunk_index, chunk in enumerate(pd.read_csv(path, chunksize=chunksize)):
        try:
            features = extract_song_features_in_chunks(list(chunk["trackid"]),
                                                       spotify)
            for feature in features:
                if len(feature) < 10:
                    print("STOP")
        except:
            spotify = initialize_spotify_connection()
            features = extract_song_features_in_chunks(list(chunk["trackid"]),
                                                       spotify)

        for feature_index, (trackid, song_feature) in enumerate(zip(list(chunk["trackid"]), features)):
            if song_feature is None:
                features[feature_index] = {"trackid": trackid}
            else:
                song_feature["trackid"] = trackid

        write_to_csv(features, chunk_index)
        
        
def extract_song_features_in_chunks(songs, spotify):
    '''
    Calls spotify API to retrieve song audio features either individually or in
    batches up to 100 tracks at a time
    :param songs: either individual trackid or batch (up to 100)
    :param spotify: spotify client object
    :return: dictionary of features for track/tracks
    '''

    features = spotify.audio_features(songs)
    return features
```

## 3. Write to output csv
Finally, the last step was to write the tracks to a csv, with the newly extracted audio features. This step was straighforward. 

```python
def write_to_csv(songs, index=0):
    '''
    Converts song list to a dataframe and writes it to a csv. Includes handling
    for first batch (open in write mode) or subsequent (open in append mode)
    :param songs: song list
    '''
    csv_fullpath = my_settings.OUTPUT_PATH / 'song_attribs_test.csv'
    songs_df = pd.DataFrame(songs)
    songs_df.set_index("trackid")

    if index == 0:
        songs_df.to_csv(csv_fullpath, header=True)
    else:
        with open(csv_fullpath, 'a') as f:
            songs_df.to_csv(f, header=False)
```

# Data Dictionary:
All field descriptions below are from the documentation in the [Spotify API](https://developer.spotify.com/documentation/web-api/reference/tracks/get-audio-features/):
- `trackid (string)`: The Spotify ID for the track.
- `duration_ms (int)`: The duration of the track in milliseconds
- `key (int)`: The estimated overall key of the track. Integers map to pitches using standard Pitch Class notation . E.g. 0 = C, 1 = C♯/D♭, 2 = D, and so on. If no key was detected, the value is -1
- `mode (int)`: Mode indicates the modality (major or minor) of a track, the type of scale from which its melodic content is derived. Major is represented by 1 and minor is 0
- `time_signature (int)`: An estimated overall time signature of a track. The time signature (meter) is a notational convention to specify how many beats are in each bar (or measure)
- `acousticness (float)`: A confidence measure from 0.0 to 1.0 of whether the track is acoustic. 1.0 represents high confidence the track is acoustic
- `danceability (float)`: Danceability describes how suitable a track is for dancing based on a combination of musical elements including tempo, rhythm stability, beat strength, and overall regularity. A value of 0.0 is least danceable and 1.0 is most danceable
- `energy (float)`: Energy is a measure from 0.0 to 1.0 and represents a perceptual measure of intensity and activity. Typically, energetic tracks feel fast, loud, and noisy
- `instrumentalness (float)`: Predicts whether a track contains no vocals. “Ooh” and “aah” sounds are treated as instrumental in this context. Rap or spoken word tracks are clearly “vocal”. The closer the instrumentalness value is to 1.0, the greater likelihood the track contains no vocal content
- `liveness (float)`: Detects the presence of an audience in the recording. Higher liveness values represent an increased probability that the track was performed live
- `loudness (float)`: The overall loudness of a track in decibels (dB). Loudness values are averaged across the entire track and are useful for comparing relative loudness of tracks. Loudness is the quality of a sound that is the primary psychological correlate of physical strength (amplitude)
- `speechiness (float)`: Speechiness detects the presence of spoken words in a track. The more exclusively speech-like the recording (e.g. talk show, audio book, poetry), the closer to 1.0 the attribute value
- `valence (float)`: A measure from 0.0 to 1.0 describing the musical positiveness conveyed by a track. Tracks with high valence sound more positive (e.g. happy, cheerful, euphoric), while tracks with low valence sound more negative (e.g. sad, depressed, angry)
- `tempo (float)`: The overall estimated tempo of a track in beats per minute (BPM). In musical terminology, tempo is the speed or pace of a given piece and derives directly from the average beat duration