# rePear User Guide (macOS)

This document is a Markdown transcription and expansion of `usage.html`, focused on running rePear on Mac.

- Original author: Martin J. Fiedler
- Original documented version: 0.4.1 (2009-02-25)
- This repository currently runs `repear.py` via Python 3

## For the impatient ...

### Initial setup

1. Initialize your iPod once with iTunes / Music app.
2. Optionally remove existing tracks with Apple software if you want a clean start.
3. Put rePear files on the iPod root or run from this repo with `--root`.
4. Run configuration:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" config
```

5. Optional: import old iTunes-managed tracks into normal folders (dangerous):

```bash
python3 repear.py --root "/Volumes/<iPod Name>" dissect
```

6. Copy music onto the iPod using Finder.
7. Freeze the database:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" freeze
```

8. Wait for completion.
9. Eject safely:

```bash
diskutil eject "/Volumes/<iPod Name>"
```

10. Listen on the iPod.

### Regular maintenance (daily workflow)

1. Connect iPod.
2. Unfreeze:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" unfreeze
```

3. Add, delete, rename, or move music files.
4. Freeze again:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" freeze
```

5. Eject safely.
6. Enjoy.

## rePear Setup - Step by Step

The first setup usually takes 10-30 minutes depending on library size and iPod speed.

### 1. Prepare the iPod

- Initialize once with iTunes / Music app.
- Identify mount point on macOS:
  - `/Volumes/<iPod Name>`
- Back up iPod database files before first use:

```bash
cp -a "/Volumes/<iPod Name>/iPod_Control/iTunes" ~/Desktop/iPod_iTunes_Backup
```

Notes:

- `iPod_Control` is hidden in Finder by default. Use Terminal paths or enable hidden files.
- rePear will also create its own backup of `iTunesDB`, but keep your own backup anyway.

### 2. Install rePear

If you already cloned this repo, you can run in place from your Mac.

Required runtime:

- Python 3

Optional tools (only for specific features):

- `vorbis-tools` (`oggdec`) + `lame` for `.ogg` transcoding
- `Pillow` for artwork generation
- `ffmpeg`/`ffprobe` for `.m4b` conversion/inspection workflows

Install optional dependencies on macOS:

```bash
brew install vorbis-tools lame ffmpeg
python3 -m pip install --upgrade Pillow
```

Run style:

```bash
python3 repear.py [options] [action]
```

Examples:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" help
python3 repear.py --root "/Volumes/<iPod Name>" freeze
```

### 3. Configure rePear

Run the interactive configuration wizard:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" config
```

This includes:

- Model selection (required for artwork generation)
- FWID setup (checksum support for newer classic-era iPods)

### 4. Import tracks (optional)

If your iPod already has tracks managed by iTunes, you can:

- Keep them where they are, or
- Use `dissect` to export into normal folders

Command:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" dissect
```

`dissect` creates `Dissected Tracks/Artist/Album/Track` style folders.

Warning:

- `dissect` changes the existing on-device file layout. Keep your backup first.

### 5. Copy your tracks onto the iPod

You can organize files however you like.

Recommendations:

- Do not manually manage content inside `iPod_Control/Music`.
- Put your library in user folders like `/Music`, `/Audiobooks`, etc. on the iPod volume.

Supported scanned extensions in current code:

- `.mp3`, `.ogg`, `.m4a`, `.m4b`, `.mp4`

### 6. Run rePear

Freeze rebuilds the playable iPod database:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" freeze
```

What happens:

- rePear scans supported files.
- It reads metadata and updates internal cache.
- It moves tracks into iPod-managed storage under `iPod_Control/Music`.
- It rebuilds `iTunesDB`.
- It applies playlists and optional artwork.

Performance:

- Initial freeze is the slowest run.
- Later runs are faster due to cache reuse.

### 7. Disconnect the iPod and listen to your music

Always eject cleanly:

```bash
diskutil eject "/Volumes/<iPod Name>"
```

If playback fails after freeze:

- Inspect `repear.log`.
- Compare generated `iTunesDB` with your backup.
- Verify mount path and no interrupted writes.

## Managing the rePear tracks - Step by Step

### 1. Plug in the iPod and run rePear

If you run without an explicit action, rePear chooses `auto` and typically does the right next step:

```bash
python3 repear.py --root "/Volumes/<iPod Name>"
```

Usually, for a frozen database this means `unfreeze`.

### 2. Manage your music

After `unfreeze`, your files return to original user-visible locations.

You can then:

- Add new albums
- Delete old tracks
- Rename folders/files
- Move content between folders

### 3. Run rePear again

Before unplugging, freeze again:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" freeze
```

### 4. Disconnect the iPod and listen to your music

Eject safely from Finder or Terminal (`diskutil eject`).

## Details on playlists

rePear supports two playlist sources:

1. Standard `.m3u` files found on the iPod.
2. Automatic playlists from rules in `repear_playlists.ini`.

### The master playlist file

File location:

- `repear_playlists.ini` in iPod root (default)

Format:

- INI-style text
- `key = value`
- `;` starts comments
- Sections are `[Playlist Name]`
- Keys before first section are global options

Boolean values accepted:

- `1/0`, `yes/no`, `y/n`, `on/off`, `true/false`, `enable/disable`

### Global playlist options

At top of `repear_playlists.ini` before first section:

- `skip album playlists` (boolean, default enabled)
  - skips playlists that exactly match one album
- `directory playlists` (boolean, default disabled)
  - creates one playlist per directory containing playable files

### Automatic playlist options

Inside each `[PlaylistName]` section:

- `include = <path-or-pattern>`
  - include a directory recursively or a pattern like `*.mp3`
- `exclude = <path-or-pattern>`
  - remove matching files from that playlist selection
- `new = <boolean>`
  - include files new since last freeze
- `changed = <boolean>`
  - include files with changed metadata since last freeze
- `shuffle = <mode>`
  - off: `0/no/off/false/disabled/none`
  - balanced shuffle: `1/yes/on/true/enabled/balanced`
  - random shuffle: `2/random/standard`
- `sort = <criteria>`
  - one or more sort criteria, comma-separated

Sort precedence notes:

- If both shuffle and sort are present, sorting happens after shuffle.
- Multiple `sort` lines are applied in order.

### Sort criteria

Available criteria:

| Criterion | Meaning |
|---|---|
| `title` | Track title |
| `artist` | Track artist |
| `album` | Album title |
| `year` | Release year |
| `compilation` | Compilation flag |
| `rating` | User rating |
| `path` | File path |
| `length` | Track duration |
| `file size` | File size |
| `mtime` | Modification time |
| `bitrate` | Audio bitrate |
| `sample rate` | Audio sample rate |
| `track number` | Track number |
| `disc number` | Disc number |
| `total discs` | Number of discs |
| `artwork count` | Artwork item count |
| `BPM` | Beats per minute |
| `movie flag` | Movie/media flag |
| `play count` | Times fully played |
| `skip count` | Times skipped |
| `start count` | `play count + skip count` |
| `last played time` | Last full play timestamp |
| `last skipped time` | Last skip timestamp |
| `last started time` | Later of played/skipped timestamp |

Sort syntax modifiers:

- `+criterion` ascending (default)
- `-criterion` descending
- `>criterion` place empty values last (default)
- `<criterion` place empty values first

Example:

- `artist, year, album, disc number, track number, title`

### Playlist examples

Example `repear_playlists.ini`:

```ini
[Heavy Metal]
include = /Music/Metal

[Random Soft Songs]
include = /Music
exclude = /Music/Metal
exclude = /Music/Rock
shuffle = 1
sort = artist, album, title

[Audiobooks]
include = *.book.mp3

[Hot New Stuff]
new = 1
changed = 1

[Randomized]
include = /Music
shuffle = 1
sort = start count
```

Interpretation highlights:

- `[Heavy Metal]` includes one directory tree.
- `[Random Soft Songs]` includes `/Music` minus excluded subtrees.
- `[Audiobooks]` pattern-matches filenames globally.
- `[Hot New Stuff]` tracks recent changes since last freeze.
- `[Randomized]` combines balanced shuffle with a follow-up sort.

## Details on artwork

rePear chooses artwork using priority rules. For a track like
`/MyMusic/TheAlbum/TheTrack.mp3`, priority is:

1. `TheTrack.jpg` or `TheTrack.png` in same folder
2. `TheAlbum.jpg`/`.png` in same folder
3. First filename containing `front` in same folder
4. First filename containing `cover` in same folder
5. First image (alphabetic) in same folder
6. Parent-level image named after directory, e.g. `/MyMusic/TheAlbum.jpg`
7. Inherited directory-level artwork from parent folders

Practical macOS tip:

- Keep non-music images out of top-level library folders, or they may be reused broadly by inheritance.

## Details on last.fm scrobbling (removed)

Last.fm scrobbling has been removed from this fork. There is no scrobble action,
no scrobble config file, and no Last.fm submission step during `freeze`/`update`.

## A more detailed look at rePear's options and actions

If no action is provided, rePear defaults to `auto`.

### Actions

- `help`
  - show help text and exit
- `auto`
  - choose between `freeze` and `unfreeze` based on cache state
- `freeze`
  - scan files, move into iPod library area, rebuild `iTunesDB`
- `unfreeze`
  - move tracks back to original locations recorded in cache
- `update`
  - rebuild database from cache without scanning for newly added files
- `dissect`
  - extract existing iTunesDB tracks into `Dissected Tracks/...`
- `reset`
  - clear metadata cache (dangerous if currently frozen)
- `cfg-fwid`
  - detect and store iPod serial/FWID for checksum support
- `cfg-model`
  - interactive model selection for artwork support
- `config`
  - run all config steps (`cfg-fwid`, `cfg-model`)

### Command-line options

General usage:

```bash
python3 repear.py [options] [action]
```

Options:

- `-r, --root PATH`
  - iPod root path (recommended on Mac)
- `-l, --log FILE`
  - output log file path
- `-L, --lameopts CMDLINE`
  - LAME options used for OGG transcoding
- `-m, --model MODEL`
  - iPod model (artwork support)
- `-f, --force`
  - skip dangerous-action confirmation prompts
- `-p, --playlist FILE`
  - override playlist config file
- `--nowait`
  - Windows-only keypress wait toggle

Valid model names:

| Model values | Hardware |
|---|---|
| `nano`, `nano1g`, `nano2g` | iPod nano 1st/2nd gen |
| `4g`, `photo` | iPod photo (4th gen) |
| `5g`, `video` | iPod video (5th gen) |
| `6g`, `classic`, `nano3g` | iPod classic 6th gen / nano 3rd gen |
| `nano4g` | iPod nano 4th gen |

## macOS-specific troubleshooting notes

- "root directory contains no iPod database"
  - Confirm `--root` points to mount containing `iPod_Control/iTunes/iTunesDB`.
- Freezes are very slow
  - First run is always slower; use `update` when adding no new files.
- `.m4b` skipped
  - HE-AAC/AAC+ `.m4b` files are skipped; convert to AAC-LC and re-freeze.
- Playback issues after interruption
  - Reconnect, run `freeze` again, and inspect `repear.log`.
