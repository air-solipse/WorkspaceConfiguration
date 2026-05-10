# Beet Installation and Configuration

You like music? So do I!

Here is how I manage my personal library.

## Install [beet](https://beets.readthedocs.io/en/stable/)

If you don't have it already, install [Python](https://www.python.org/downloads/). Then, run

```
brew install pipx
pipx ensurepath
pipx install beets
```

## Find and open configuration file

(You might need to quit and reopen the terminal shell)
```
beet config -e
```

## Write this configuration to the file

```
# Root directory for music files
directory: ~/Music/Music/Media.localized/Automatically Add to Music.localized
# Location of the beets database
library: ~/Music/Music/library.db

# Plugins to load
plugins:
  - discogs # pip3 install discogs-client
  - fetchart # pip3 install requests
  - lastgenre # pip3 install pylast
  - inline
  - embedart
  - duplicates

# Use multiple cores for processing
threaded : yes

# User interface
ui:
  color: yes

# Import settings
import:
  write: yes
  copy: no
  move: yes
  resume: ask
  from_scratch: yes
  quiet: no
  default_action: apply
  autotag: yes
  bell: yes
  auto: yes


external_ids:
  discogs: yes
  spotify: yes


# Use the original release date of an album when a re-release is added
original_date: yes
# Start counting track numbers back at one for each disc
per_disc_numbering: yes

# Activate embedded cover art
embedart:
  auto: yes
  remove_art_file: yes

# Custom variables for metadata
# Inline python code using inline plugin
item_fields:
  isMultidisc: 1 if disctotal > 1 else 0

aunique:
  keys: albumartist albumtype year album
  disambuguators: format mastering media label albumdisambig releasegroupdisambig
  bracket: '[]'

paths:
  default: $albumartist/($year) $album/%if{$isMultidisc,$disc - }$track $title

singletons:
  album: true
  default: $albumartist/($year) $album/%if{$isMultidisc,$disc - }$track $title

fetchart:
  auto: yes
  sources: 
    - wikipedia
    - coverart: release
    - coverart: releasegroup
    - albumart
    - amazon
    - google
    - itunes
    - fanarttv

lastgenre:
  count: 5
  separator: ', '
  auto: yes
  source: album
  prefer_specific: yes

match:
  ignored: missing_tracks unmatched_tracks
  strong_rec_thresh: 0.15
  max_rec:
    artist: strong
    album: strong
    year: strong
    country: strong
    album_id: strong
    tracks: strong
    track_title: strong
    track_artist: strong
    track_index: strong
    track_length: strong
    track_id: strong
  preferred:
    original_year: yes

duplicates:
  album: no
  count: no
  delete: yes
  full: yes
  merge: yes
  path: no
  strict: no
  tiebreak:
    items: [bitrate]
```