# pyheartradio

pyheartradio is a Python client for the iHeartRadio public API. It searches and streams live radio stations, podcasts, artists, tracks, and playlists. It needs no API key and no account.

```python
from pyheartradio import IHeartRadio

client = IHeartRadio()
for station in client.search_stations("jazz"):
    print(station.title, station.stream)
```

## Install

```bash
pip install pyheartradio
```

## Features

- Search stations, podcasts, artists, tracks, and playlists
- Get podcast episodes with direct audio stream URLs
- Typed dataclass models, for IDE completion, `dataclasses.asdict()`, and JSON serialization
- Parallel detail fetches. Station and artist lookups run concurrently, not sequentially
- `to_external_ids()` and `to_signals()` on every model, for [metadatarr](https://github.com/LeMetadatarr/metadatarr) integration
- Session reuse and a configurable timeout. Attach any cache adapter through `client.session`

## Quick examples

```python
client = IHeartRadio(timeout=5, max_workers=4)

# Stations: stream URL included
for station in client.search_stations("classic rock", max_results=3):
    print(station.title, "->", station.stream)

# Podcasts + episodes
podcast = next(client.search_podcast("Serial"))
for ep in client.get_podcast_episodes(podcast.id):
    print(ep.title, ep.stream)

# Artists
artist = next(client.search_artist("David Bowie"))
print(artist.title, len(artist.albums), "albums")

# Tracks (metadata only, no stream URL available from the API)
for track in client.search_track("Heroes"):
    print(track.title, track.artist)
```

## metadatarr integration

All models expose `to_external_ids()` and `to_signals()` for drop-in use with `mediavocab.models.ExternalIds` and `Signals`:

```python
from mediavocab.models import ExternalIds

station = next(client.search_stations("WNYC"))
ids = ExternalIds.from_dict(station.to_external_ids())
for stream in ids.streams:
    print(stream.platform, stream.url)   # -> radio  https://...
```

See [docs/metadatarr.md](docs/metadatarr.md) for the full integration guide.


## Caching

The client uses a plain `requests.Session`. Station stream URLs are extremely stable (years-long TTL). To add caching, attach an adapter before searching:

```python
import requests_cache
client = IHeartRadio()
requests_cache.install_cache("iheart", expire_after=3600)
```

Or use `CacheControl` without adding a global side-effect:

```python
from cachecontrol import CacheControl
client.session = CacheControl(client.session)
```

## Documentation

- [Quickstart](docs/quickstart.md)
- [Data models](docs/models.md)
- [API methods](docs/methods.md)
- [metadatarr integration](docs/metadatarr.md)
- [Advanced usage](docs/advanced.md)

## Related projects

- [LeMetadatarr/metadatarr](https://github.com/LeMetadatarr/metadatarr): the entity-resolution layer pyheartradio results plug into
- [TigreGotico/mediavocab](https://github.com/TigreGotico/mediavocab): supplies the `ExternalIds` and `Signals` models used by the metadatarr integration

## License

Apache-2.0
