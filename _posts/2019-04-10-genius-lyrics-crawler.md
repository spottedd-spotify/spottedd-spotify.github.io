---
title: "Spotify Project: Data Infrastructure"
date: 2019-12-06
author_profile: true
excerpt: "Building data infrastructure to support the Spotify project"
tags: [Data Science, Data Engineering]
---

[Genius](https://genius.com/) is a music platform that has a rich repository of song lyrics, along with lyric analysis. This post details some work I did scraping the Genius API. In particular, the code below allows you to extract the full discography of a given artist, along with the lyrics for each song. The work was partially inspired by [Will Soares' Genius-Spotify integration](http://willamesoares.com/tech/how-to-integrate-spotify-and-genius-api-to-easily-crawl-song-lyrics-with-python/). You can access the full code for this project [here](https://github.com/austinrochon/song-lyrics-analysis).


## First, retrieve the artist_id, given an artist's name

To retrieve all the songs composed by a given artist, we first need the artist_id, a unique identifier Genius uses to identify artists in its database: 

```python
def get(api_path, data, token):
    '''
    General function that makes api request based on a specified path
    :param api_path: path designating the type of api request
    :param data: data to feed to the request
    :param token: authorization token
    :return response: json response from api
    '''

    headers = {'Authorization': f'''Bearer {token}'''}
    response = requests.get(api_path, params=data, headers=headers).json()

    return response


def get_artist_id(artist_name, base_url):
    '''
    Retrieves artist_id for a given artist_name
    :param artist_name: name of artist to retrieve
    :param base_url: api_path to call
    :return artist_id: id for specified artist
    '''

    data = {'q': artist_name}
    response = get(f'''{base_url}/search''',
                   data,
                   my_settings.GENIUS_TOKEN)

    if len(response['response']['hits']) == 0:
        print(f'''Artist name {artist_name} not found...''')
        exit(0)

    for hit in response['response']['hits']:
        if hit['result']['primary_artist']['name'].lower() == artist_name:
            artist_id = hit['result']['primary_artist']['id']
            break

    return artist_id
```


## Next, retrieve all songs associated with the artist_id

To retrieve the full discography for a given artist, I first created an array containing the song_ids for each song linked to our artist. The logic for paging through all results was inspired by [Jon Evans' work here](https://www.jw.pe/blog/post/quantifying-sufjan-stevens-with-the-genius-api-and-nltk/)

```python
def get_artist_discography(artist_id, base_url):
    '''
    Retrieves list of all song_ids in entire artist discography
    :param artist_id:
    :param base_url:
    :return discography: list of all song_ids in artist discography
    '''

    # Logic for paging through all results inspired by:
    # https://www.jw.pe/blog/post/quantifying-sufjan-stevens-with-the-genius-api-and-nltk/
    current_page = 1
    next_page = True
    discography = []

    while next_page:
        data = {'page': current_page,
                'per_page': 50,
                'sort': 'title'}
        response = get(f'''{base_url}/artists/{artist_id}/songs''',
                       data,
                       my_settings.GENIUS_TOKEN)
        page_songs = response['response']['songs']

        if page_songs:
            for song in page_songs:
                discography.append(song['id'])
            print(f'''Retrieved results for page {current_page}''')
            current_page += 1
        else:
            next_page = False

    return discography

```

Next, I started building out a dictionary containing key fields associated with the each song, including, song names and album names
```python

def get_song_data(song_id_list, artist_name, artist_id, base_url):
    '''
    Retrieves song data for a given list of song_ids
    :param song_id_list:
    :param base_url:
    :return all_songs:
    '''

    all_songs = []

    for song_id in song_id_list:
        print(f'''Retreiving song: {song_id}''')
        song_data = {}
        data = {
            'id': song_id,
            'text_format': 'plain'
        }
        response = get(f'''{base_url}/songs/{song_id}''',
                              data,
                              my_settings.GENIUS_TOKEN)
        song_data['artist_id'] = artist_id
        song_data['artist_name'] = artist_name
        song_data['song_id'] = song_id
        song_data['song_title'] = response['response']['song']['title']
        song_data['song_url'] = response['response']['song']['url']
        if response['response']['song']['album']:
            song_data['album_id'] = response['response']['song']['album']['id']
            song_data['album_name'] = response['response']['song']['album'][
                'name']
        else:
            song_data['album_id'] = None
            song_data['album_name'] = None
        all_songs.append(song_data)

    return all_songs
```


## Finally, retrieve all lyrics, and save output

Next, I fetched the lyrics for each song in the collection. To do so, I used [Will Soares'](http://willamesoares.com/tech/how-to-integrate-spotify-and-genius-api-to-easily-crawl-song-lyrics-with-python/) concise BeautifulSoup implementation to scrape the HTML of the page associated song_url:


```python
def retrieve_song_lyrics(songs):
    '''
    Given a list of songs, append lyrics to each song dict
    :param songs: song list
    :return songs: song list with appended lyrics
    '''

    for song in songs:
        print(f'''Retrieving lyrics for: {song['song_title']}''')
        url = song['song_url']
        page = requests.get(url)
        html = BeautifulSoup(page.text, 'html.parser')
        song['lyrics'] = html.find('div', class_='lyrics').get_text()

    return songs
```

Finally I stored my song dictionary to a pandas DataFrame, then wrote the results to a csv:

```python
def write_to_csv(songs):
    '''
    Converts song list to a dataframe and writes it to a csv
    :param songs: song list
    '''

    songs_df = pd.DataFrame(songs)
    songs_df.to_csv(my_settings.OUTPUT_PATH / 'test.csv')
```

Here's the output of a test run I did with the discography of one of my favorite hip-hop artists, 2pac. I've expanded the lyrics for Keep Ya Head Up:

![alt text](https://raw.githubusercontent.com/austinrochon/austinrochon.github.io/master/images/song_lyrics_2pac_output.PNG "2pac Discography")