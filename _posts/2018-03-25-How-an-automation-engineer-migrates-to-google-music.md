---
title: "How an automation engineer migrates to Google Music"
layout: post
tags:
- Automation
- Python 3
category: [Automation, Python]
excerpt: "I recently posted on Facebook that I was ditching Facebook and all of the apps dependent on it. While Spotify isn't inherently dependent on it, I also wanted the opportunity to use Google Music for some of my favorite podcasts. Thus, I made the decision that Spotify was going down too.

Migrating from Spotify to Google Play Music is already kind of a difficult task. I've had years of being on Spotify, to the point where I have 850 tracks on playlists and my library. I wanted a way to move my playlists I've built over the years over to Google Music. I found a service called [Soundiiz](soundiiz.com) that performs some needed functions. First, it matches Spotify tracks to Google Play tracks and copies your playlist over. Unfortunately, I found that this only worked for playlists and not my music library. Thus, I took things into my own hands."
---

I recently posted on Facebook that I was ditching Facebook and all of the apps dependent on it. While Spotify isn't inherently dependent on it, I also wanted the opportunity to use Google Music for some of my favorite podcasts. Thus, I made the decision that Spotify was going down too.

Migrating from Spotify to Google Play Music is already kind of a difficult task. I've had years of being on Spotify, to the point where I have 850 tracks on playlists and my library. I wanted a way to move my playlists I've built over the years over to Google Music. I found a service called [Soundiiz](soundiiz.com) that performs a very nifty function at a price of $4.95. It matches Spotify tracks to Google Play tracks and copies your playlist over. Unfortunately, I found that this only worked for playlists and not my music library. Thus, I took things into my own hands.

If you're going to do this I would make sure that you are using 2-factor-auth on Google, setup an app password for Soundiiz, and create one for the script you're going to write. Soundiiz guides you through setting up your auth with Spotify and Google Music, so I'll leave that up to you. Once you're done with the initial setup go ahead and run your platform sync.

You'll notice that playlists are now populated but not the music library :(

## Code

{% highlight python linenos %}
from gmusicapi.clients import Mobileclient

# init
gmusic = Mobileclient()

track_ids_to_import = []

def extract_track(playlists):
    """Extract the store ID so we can add it to the library"""
    for playlist in playlists:
        for track in playlist['tracks']:
            track_id = track['trackId']
            store_id = track['track']['storeId']
            # some sanity checking because I don't know the API
            if track_id != store_id:
                print (f'Track Id ({track_id}) does not match Store Id ({store_id})')
            if track_id not in track_ids_to_import:
                print(f'Importing {track_id} (StoreId: {store_id})')
                track_ids_to_import.append(track_id)


# login
gmusic.login('<email>', '<app_password>', gmusic.FROM_MAC_ADDRESS, 'en_US')
# let me know we're authenticated
print(f'Authenticated: {gmusic.is_authenticated()}')
# extract the tracks
extract_track(gmusic.get_all_user_playlist_contents())
# produce an import report
print(f'Imported {len(track_ids_to_import)} tracks')
# add songs to the library
library_tracks_imported = gmusic.add_store_tracks(track_ids_to_import)

if len(library_tracks_imported) == len(track_ids_to_import):
    print('Successfully imported all tracks.')
else:
    print('Failures occurred while importing...')
    print(f'Number of tracks: {len(track_ids_to_import)}')
    print(f'Number of tracks imported: {len(library_tracks_imported)}')
{% endhighlight %}

First, update the email and app password you generated for line 23. There is no storage of your credentials here, it uses the [*gmusicapi*](https://pypi.python.org/pypi/gmusicapi) library which you can find on PyPi. You're now safe to run the script. If you have PyCharm you can actually browse the playlists and be a bit more choosey about what gets imported.

## Conclusion

There are some other scripts out there that are supposed to do this, even one in Python. I tried them all and they didn't work. I thought Soundiiz would accomplish what I needed, however, when I was stuck with a half imported library this is what I resorted to. I'd recommend someone update [PyPortify](https://github.com/rckclmbr/pyportify).
