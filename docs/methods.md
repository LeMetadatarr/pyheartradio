# API methods

All methods live on the `IHeartRadio` class.

```python
from pyheartradio import IHeartRadio
client = IHeartRadio(timeout=10)
```

---

## `IHeartRadio(timeout=10, max_workers=6)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `timeout` | `int` | `10` | HTTP request timeout in seconds |
| `max_workers` | `int` | `6` | Thread pool size for parallel detail fetches (see [Advanced usage](advanced.md)) |

The client creates a single `requests.Session` that is reused for all calls.
To use a custom session (e.g. with proxy settings or retry adapters) assign
it directly after construction:

```python
import requests
from requests.adapters import HTTPAdapter, Retry

client = IHeartRadio()
retry = Retry(total=3, backoff_factor=0.5)
client.session.mount("https://", HTTPAdapter(max_retries=retry))
```

---

## `search_stations(search_term, max_results=10)` → `Iterator[Station]`

Search for live radio stations.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `search_term` | `str` | — | Free-text query, e.g. `"jazz"`, `"WNYC"`, `"90s hits"` |
| `max_results` | `int` | `10` | Maximum number of search results to request |

Each yielded `Station` is guaranteed to have a non-empty `.stream` URL.
Results without a playable stream are silently skipped. `.streams` carries
every format the station broadcasts.

**Two HTTP calls per station result** (search + stream-URL lookup).

```python
for s in client.search_stations("NPR"):
    print(s.title, s.stream)
```

---

## `search_podcast(search_term, max_results=10)` → `Iterator[Podcast]`

Search for podcast shows.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `search_term` | `str` | — | Show name, topic, or host |
| `max_results` | `int` | `10` | Maximum number of search results to request |

**One HTTP call** (search only).

```python
podcast = next(client.search_podcast("Serial"), None)
```

---

## `get_podcast_episodes(podcast_id)` → `Iterator[PodcastEpisode]`

Retrieve episodes for a podcast.

| Parameter | Type | Description |
|---|---|---|
| `podcast_id` | `int` | The numeric iHeartRadio podcast ID |

**One HTTP call per episode** (episode list + per-episode stream URL lookup).

```python
for ep in client.get_podcast_episodes(podcast.id):
    print(ep.title, ep.stream)
```

---

## `search_track(search_term, max_results=10)` → `Iterator[Track]`

Search for music tracks.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `search_term` | `str` | — | Track title, artist, or keyword |
| `max_results` | `int` | `10` | Maximum number of search results to request |

**One HTTP call** (search only).

> Stream URLs are not available for tracks via the public API.

```python
for t in client.search_track("Heroes David Bowie"):
    print(t.title, t.artist)
```

---

## `search_artist(search_term, max_results=10)` → `Iterator[Artist]`

Search for artists, with profile data (albums, top tracks, related artists).

| Parameter | Type | Default | Description |
|---|---|---|---|
| `search_term` | `str` | — | Artist or band name |
| `max_results` | `int` | `10` | Maximum number of search results to request |

**Two HTTP calls per artist result** (search + artist profile).

```python
for a in client.search_artist("The Beatles"):
    print(a.title, len(a.albums), "albums")
```

---

## `search_playlist(search_term, max_results=10)` → `Iterator[Playlist]`

Search for curated playlists.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `search_term` | `str` | — | Mood, genre, or keyword |
| `max_results` | `int` | `10` | Maximum number of search results to request |

**One HTTP call** (search only).

```python
for pl in client.search_playlist("lo-fi study"):
    print(pl.title, pl.url)
```

---

## `search(query, max_results=10)` → `SearchResults`

Unified search across all entity types (stations, podcasts, artists, tracks,
playlists) in a single API call.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `query` | `str` | — | Free-text query |
| `max_results` | `int` | `10` | Maximum results per entity type |

Station stream URLs are fetched in parallel, same as `search_stations()`.
Artist results from this method are stubs (`id`, `title`, `image` only) —
use `search_artist()` for full profiles with albums, tracks, and related
artists.

```python
results = client.search("jazz")
print(len(results.stations), "stations,", len(results.podcasts), "podcasts")
```

---

## `get_now_playing(station_id)` → `NowPlaying`

Return what is currently on air for a live station.

| Parameter | Type | Description |
|---|---|---|
| `station_id` | `int` | The numeric iHeartRadio station ID from a `search_stations()` result |

```python
station = next(client.search_stations("WNYC"))
np = client.get_now_playing(station.id)
print(np.artist, "—", np.title)
```

---

## `get_track(track_id)` → `Track`

Fetch a track directly by its iHeartRadio ID, without running a search.

| Parameter | Type | Description |
|---|---|---|
| `track_id` | `int` | The numeric iHeartRadio track ID |

```python
track = client.get_track(555)
print(track.title, track.artist)
```

---

## `get_artist_albums(artist_id)` → `Iterator[Album]`

Fetch albums for a known artist ID without a full profile fetch.

| Parameter | Type | Description |
|---|---|---|
| `artist_id` | `int` | The numeric iHeartRadio artist ID from a `search_artist()` result |

```python
for album in client.get_artist_albums(100):
    print(album.title, album.year)
```

---

## `get_similar_artists(artist_id)` → `Iterator[Artist]`

Fetch artists similar to a given artist ID.

| Parameter | Type | Description |
|---|---|---|
| `artist_id` | `int` | The numeric iHeartRadio artist ID |

```python
for artist in client.get_similar_artists(100):
    print(artist.title)
```

---

## Error handling

All methods raise standard `requests` exceptions on network or HTTP errors:

| Exception | When |
|---|---|
| `requests.HTTPError` | Server returned 4xx or 5xx |
| `requests.ConnectionError` | Network unavailable |
| `requests.Timeout` | Request exceeded `timeout` seconds |

```python
import requests

try:
    stations = list(client.search_stations("jazz"))
except requests.HTTPError as exc:
    print(f"HTTP {exc.response.status_code}")
except requests.Timeout:
    print("Request timed out")
except requests.ConnectionError:
    print("No network")
```

---
[← Data models](models.md) · [Home](README.md) · [metadatarr integration →](metadatarr.md)
