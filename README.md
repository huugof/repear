# rePear

rePear is a command-line tool for managing older Apple iPods without iTunes. It scans audio files on the iPod filesystem, builds `iTunesDB`, and supports freeze/unfreeze workflows so you can organize files directly in Finder.

This fork currently runs as a Python script (`repear.py`).

## macOS Setup

### 1. Requirements

- Python 3
- Optional (only if needed):
  - `ffmpeg` (for converting unsupported `.m4b` files)
  - `vorbis-tools` (`oggdec`) + `lame` (for `.ogg` -> `.mp3` transcoding)
  - `Pillow` (for artwork generation)

Install with Homebrew / pip:

```bash
brew install ffmpeg vorbis-tools lame
python3 -m pip install --upgrade Pillow
```

### 2. Prepare the iPod

1. Initialize the iPod at least once with iTunes/Music app.
2. Mount the iPod (typically `/Volumes/<iPod Name>`).
3. Back up `iPod_Control/iTunes` from the iPod before first use.

### 3. Run rePear against the iPod mount

From this repo:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" help
```

If the root path is omitted, rePear tries to auto-detect it from the current/script directory.

## First-Time Configuration (Mac)

Run:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" config
```

This runs:

- `cfg-fwid` (detect iPod serial/hash info)
- `cfg-model` (needed for artwork support)

## Daily Usage

Typical cycle:

1. Plug in iPod.
2. Unfreeze (or run no action and let `auto` decide):

```bash
python3 repear.py --root "/Volumes/<iPod Name>" unfreeze
```

3. Add/remove/rename music files in Finder.
4. Freeze to rebuild database:

```bash
python3 repear.py --root "/Volumes/<iPod Name>" freeze
```

5. Eject safely:

```bash
diskutil eject "/Volumes/<iPod Name>"
```

## Supported File Types

rePear scans these extensions:

- `.mp3`
- `.ogg` (transcoded to MP3 if `oggdec` + `lame` are available)
- `.m4a`
- `.m4b`
- `.mp4`

## `.m4b` Audiobook Notes and Conversion

rePear imports `.m4b` and flags them as audiobooks (bookmarking enabled, shuffle disabled).

Important: `.m4b` files encoded as **HE-AAC / AAC+** are skipped. Convert them to **AAC-LC** first.

### Check codec/profile

```bash
ffprobe -v error -select_streams a:0 -show_entries stream=codec_name,profile -of default=noprint_wrappers=1 "book.m4b"
```

### Convert one file to AAC-LC `.m4b` (preserve tags + embedded cover)

```bash
ffmpeg -i "book.m4b" \
  -map 0:a:0 -map '0:v:0?' -dn \
  -c:a aac -profile:a aac_low -b:a 96k \
  -c:v copy -disposition:v:0 attached_pic \
  -map_metadata 0 -map_chapters -1 \
  -f ipod "book.aaclc.m4b"
```

### Batch convert all `.m4b` files in a folder

```bash
for f in *.m4b; do
  ffmpeg -i "$f" \
    -map 0:a:0 -map '0:v:0?' -dn \
    -c:a aac -profile:a aac_low -b:a 96k \
    -c:v copy -disposition:v:0 attached_pic \
    -map_metadata 0 -map_chapters -1 \
    -f ipod "${f%.m4b}.aaclc.m4b"
done
```

Notes:
- `-map_metadata 0` keeps title/artist/album/year tags.
- `-map '0:v:0?'` keeps embedded cover art when present.
- `-dn` avoids problematic data/text streams that can fail iPod muxing.

After conversion, copy converted files to the iPod and run `freeze` again.

## Common Actions

```bash
python3 repear.py --root "/Volumes/<iPod Name>" help
python3 repear.py --root "/Volumes/<iPod Name>" freeze
python3 repear.py --root "/Volumes/<iPod Name>" unfreeze
python3 repear.py --root "/Volumes/<iPod Name>" update
python3 repear.py --root "/Volumes/<iPod Name>" dissect
python3 repear.py --root "/Volumes/<iPod Name>" reset
```

## Useful Options

- `-r, --root PATH` iPod root path
- `-l, --log FILE` log file path (default `repear.log`)
- `-m, --model MODEL` iPod model (required for artwork)
- `-L, --lameopts "..."` override LAME options for OGG transcoding
- `-f, --force` skip confirmation prompts
- `-p, --playlist FILE` playlist config path

## Config Files

By default, rePear looks for:

- `repear_playlists.ini`

in the iPod root (or as overridden by CLI options).
