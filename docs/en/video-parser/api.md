---
outline: deep
description: "VRChat video API reference: v1 video and music, v3 danmaku and lyrics, v4 unified parsing, cover encoding, playlists, world slots and JSON responses."
---

# VRC Video Parsing API Call and Restriction Documentation

## Project Overview

VRC Video Parsing provides free and fast video and audio parsing for VRChat players.

## Basic Information

### Interface Address
```text
https://api.kipfel.link/
```

#### Alternative Interface Address

```text
https://api.kipfel.vrchat.org.cn/
```

### User-Agent Requirements
::: warning Note
- For v3 / v4, lyrics and danmaku components use a UA containing `Unity`; media players use one without it. These are **two separate requests**, not two UAs in one request. Non-Unity clients can use `json=1` for JSON.
:::

---

## v1 Core Parsing Interface

### Video Parsing

**Interface**: `/v1/vrc?url={video link}`  
**Parameters**:
- `url` (required): Video link

**Response**:
- **Success**: 302 redirect to the direct link address
- **Failure**: Returns JSON formatted error information

**JSON Version**: `/v1/vrc-json` - always returns JSON

**JSON Response Format**:

**Success Response**:
```json
{
  "code": 0,
  "message": "success",
  "log_id": "req_example",
  "data": {
    "video_id": "BV17PTT6FEJJ",
    "avid": "",
    "bvid": "BV17PTT6FEJJ",
    "title": "视频标题内容",
    "desc": "视频的详细描述信息",
    "duration": 2136,
    "cover": "http://example.com/cover.jpg",
    "create_time": 1783075293,
    "update_time": 0,
    "status": 0,
    "category": "",
    "category_name": "",
    "author": {
      "user_id": "",
      "nickname": "作者昵称",
      "avatar": "http://example.com/avatar.jpg"
    },
    "stat": {
      "play_count": 36234,
      "like": 2089,
      "comment": 11,
      "share": 32,
      "favorite": 0
    },
    "content": {
      "play_url": "https://example.com/video_no_watermark.mp4",
      "cover_url": "http://example.com/cover.jpg",
      "hashtags": [],
      "mentions": [],
      "music_info": {}
    }
  }
}
```

Field description:
- `code` (Number): Business status code, `0` indicates success.
- `message` (String): Business response message, usually `"success"`.
- `log_id` (String): Request ID.
- `data.video_id` (String): BV ID extracted from the link; otherwise the request ID.
- `data.avid` (String): AV ID extracted from an av input link; otherwise empty.
- `data.bvid` (String): BV ID extracted from input and uppercased; otherwise empty.
- `data.title` (String): The title of the video or audio.
- `data.desc` (String): The description/introduction of the video or audio.
- `data.duration` (Number): Total media duration, unit: seconds.
- `data.cover` (String): Direct link to the default display cover image.
- `data.create_time` (Number): Timestamp when the work was published (in seconds).
- `data.update_time` (Number): Currently 0.
- `data.status` (Number): Media status code.
- `data.category` (String): Currently an empty string.
- `data.category_name` (String): Currently an empty string.
- `data.author.user_id` (String): Currently an empty string.
- `data.author.nickname` (String): The creator's nickname.
- `data.author.avatar` (String): URL of the creator's avatar image.
- `data.stat.play_count` (Number): Total plays.
- `data.stat.like` (Number): Total likes.
- `data.stat.comment` (Number): Total comments.
- `data.stat.share` (Number): Total shares/forwards.
- `data.stat.favorite` (Number): Currently 0.
- `data.content.play_url` (String): **Core data**: Direct link to the watermark-free media playback.
- `data.content.cover_url` (String): The same cover URL as `data.cover`.
- `data.content.hashtags` (Array): Currently an empty array.
- `data.content.mentions` (Array): Currently an empty array.
- `data.content.music_info` (Object): Currently an empty object.

**Failure Response**:
```json
{
  "error": true,
  "status": 400,
  "code": "PARSE_ERROR",
  "message": "解析失败，请检查链接是否正确"
}
```

Field description:
- `error` (boolean): Whether an error occurred
- `status` (number): HTTP status code
- `code` (string): Error code
- `message` (string): Error message

---

::: tip Response Fields
In v1, `log_id` is the request ID. `video_id` uses an extracted BV ID, otherwise the request ID. `avid` / `bvid` are extracted from input only; original casing and AV / BV conversion are not guaranteed. `author.user_id` and category fields are empty, `favorite` is 0, and hashtags, mentions and music information are empty arrays / objects. Select JSON through `-json`, `json=1` or `Accept: application/json`.
:::

### Alternative Video Parsing Interface

**Interface**: `/v1/kfc?url={video link}`  
**Parameters**: Same as `/v1/vrc`

**Response**: Same as `/v1/vrc`

**JSON Version**: `/v1/kfc-json` - response format is the same as `/v1/vrc-json`

---

### Music Parsing

**Interface**: `/v1/music?url={music link}`\
**Parameters**: `url` (required); use individual song links. Fetch playlists through the list interfaces below first.

**Response**: 302 to playable media on success. Add `json=1` or send `Accept: application/json` for JSON in the `/v1/vrc-json` format.

::: warning JSON Address
`/v1/music-json` is not registered. Use `/v1/music?json=1&url=...`; `/v1/musickfc-json` remains available. `i` / `index` / `@N` song selection is not connected.
:::

---

### Alternative Music Parsing Interface

**Interface**: `/v1/musickfc?url={music link}`\
**Parameters**: Same as `/v1/music`

**Response**: Same as `/v1/music`

**JSON Version**: `/v1/musickfc-json` - response format is the same as `/v1/music?json=1`

---


## Music CDN and Accompaniment {#music-playback}

**Endpoints**: `/v1/music`, `/v3/vrc-lyric`, and `/v4/vrc` (or `/v4/url`).

**Parameters**:

| Parameter | Description |
|---|---|
| `cdn` | Optional. Select an available CDN alias, for example `aly`. Omit it to use the default route. |
| `track` | Optional. `original` plays the original track (default). `accompaniment` (alias `acc`) plays the accompaniment track for QQ Music, Kugou, and NetEase Cloud Music. |
| `vocal` | Optional (with `track=accompaniment`). Integer `0`–`100` for vocal volume: `0` pure instrumental, default `18` low guide vocal, `50` medium vocal, `100` full vocal (NetEase & QQ Music; Kugou is always pure instrumental). |
| `url` | Required. The complete song link. Place API parameters such as `cdn`, `track`, `vocal`, and `json` before `url`. |

**Examples**:

```bash
curl -G "https://api.kipfel.link/v1/music" --data-urlencode "cdn=aly" --data-urlencode "url=https://www.kugou.com/mixsong/j2i4p51.html"
curl -G "https://api.kipfel.link/v3/vrc-lyric" --data-urlencode "track=accompaniment" --data-urlencode "json=1" --data-urlencode "url=https://www.kugou.com/mixsong/j2i4p51.html"
curl -G "https://api.kipfel.link/v4/vrc" --data-urlencode "cdn=aly" --data-urlencode "track=accompaniment" --data-urlencode "vocal=0" --data-urlencode "url=https://music.163.com/song?id=1330348068"
curl -G "https://api.kipfel.link/v4/vrc" --data-urlencode "track=acc" --data-urlencode "vocal=18" --data-urlencode "url=https://music.163.com/song?id=1330348068"
```

**JSON Response Example** (with `track=accompaniment` and `json=1`):

```json
{
  "success": true,
  "url": "https://music-api-aly.example/m/track.mp3?t=...",
  "url_expires_at": "2026-11-27T01:00:00Z",
  "data": {
    "platform": "netease",
    "song_name": "起风了",
    "singer": "买辣椒也用券",
    "duration": 325,
    "lyrics": [
      {
        "time": 18.46,
        "text": "这一路上走走停停",
        "words": [[0, 320, 1], [320, 280, 1], [600, 350, 1]]
      }
    ],
    "tlyrics": [],
    "cover_mode": "bc1",
    "cover_encoding": "png",
    "cover_format": "wimbc1png1",
    "cover_url": "https://api.kipfel.link/cover-gpu.png?t=..."
  }
}
```

**Response**: A successful playback request returns a 302 redirect to the selected track. Accompaniment is served as MP3 audio. In V1 JSON, the playback URL is `data.content.play_url`. In V3/V4 lyrics JSON, passing `cdn`, `track`, or `media=1` adds the playback URL as the top-level `url` while preserving full lyrics and cover metadata. If the platform has no accompaniment track for a song (such as purely instrumental pieces), the endpoint returns `422 ACCOMPANIMENT_UNAVAILABLE` (`details.reason: "STEMS_UNAVAILABLE"`).

---

## v3 Advanced Interface

v3 is intended for players that need danmaku or lyrics. **A UA containing `Unity` or `unity`, or `json=1`, returns JSON**; other clients receive a 302 redirect on success.

### Danmaku Interface

**Interface**: `/v3/vrc-danmaku?url={video link}`  
**Parameters**: `url` (required); `json=1` (optional, returns JSON for non-Unity clients too).

**JSON Response Format**:

```json
{
  "success": true,
  "platform": "哔哩哔哩",
  "video_id": "BV1aAhm6vEA6",
  "title": "Video title",
  "has_more": false,
  "comments": [
    {
      "t": 0.5,
      "c": "Comment text",
      "ty": 1,
      "co": 16777215,
      "fs": 25
    }
  ],
  "danmaku": [
    {
      "t": 0.5,
      "c": "Comment text",
      "ty": 1,
      "co": 16777215,
      "fs": 25
    }
  ]
}
```


Field descriptions:
- `success` (boolean): whether the request succeeded; the Unity branch also returns an empty response on parsing failure, as explained below.
- `platform` (string): Chinese platform display name, such as `哔哩哔哩`; `generic` when no result is available.
- `comments` (array): top-level danmaku entries; `[]` when empty, with no `data` wrapper.
- `danmaku` (array): the same entries for older Udon players; read one array to avoid drawing duplicates.
- `title` / `video_id`: title / BV ID extracted from the link; the ID is empty when unavailable.
- `has_more` (boolean): currently always `false`; time-window pagination is not connected.
- `comments[].t` (number): timestamp in seconds.
- `comments[].c` (string): comment text.
- `comments[].ty` (number): type, such as `1` scrolling, `4` bottom or `5` top.
- `comments[].co` (number): decimal RGB color; `16777215` = white.
- `comments[].fs` (number): font size.

::: warning Note
The danmaku interface now uses top-level `comments`, retaining top-level `danmaku` for older players; `data.comments` is no longer returned. Small metadata fields precede the large arrays. `subtitles` / `subtitle_langs` are not returned, and the old `limit`, `start_time`, `duration`, `ai_subtitle`, `lang` / `subtitle_lang` parameters remain unconnected.
:::

### Lyrics Interface

**Interface**: `/v3/vrc-lyric?url={music link}`  
**Parameters**: `url` (required); `json=1` (optional).

**JSON Response Format** (with word-by-word `words` timeline, default Base64 cover mode):

```json
{
  "success": true,
  "data": {
    "platform": "netease",
    "lyrics": [
      {
        "time": 18.466,
        "text": "今天我寒夜里看雪飘过",
        "words": [
          [0, 320, 1],
          [320, 280, 1],
          [600, 410, 1]
        ]
      }
    ],
    "tlyrics": [],
    "song_name": "Song title",
    "singer": "Artist",
    "audio_id": "",
    "album_id": "",
    "duration": 326,
    "singer_id": "",
    "cover_mode": "base64",
    "cover_encoding": "jpeg",
    "cover_origin": "top-left",
    "cover_width": 0,
    "cover_height": 0,
    "cover_palette": [],
    "cover_pixels": "",
    "cover_size": "",
    "cover_base64": ""
  }
}
```


Field descriptions:
- `data.platform` (string): platform key, such as `netease`, `kugou`, `qqmusic`, `kuwo`, or `qishui`.
- `data.lyrics` / `data.tlyrics` (array): original / translated lyrics. Entries use `{ "time": 18.466, "text": "Lyrics", "words": [...] }`; **timestamps are in seconds and may contain decimals**. Missing lyrics produce empty arrays.
- `data.lyrics[].words` (array, optional): **word-by-word timing**, present when the upstream platform provides syllable/word lyrics (NetEase YRC, QQ QRC, Kugou KRC, Qishui); omitted for plain line-level LRC. Each item is `[offsetMs, durationMs, charCount]` relative to the line's `time`.
- `data.song_name` / `data.singer` (string): song title / artist.
- `data.duration` (number): total duration in seconds.
- `data.audio_id` / `data.album_id` / `data.singer_id` (string): currently empty strings in this compatibility response.

::: tip Cover Image
Read cover art according to `cover_mode` / `cover_encoding` in the response:
- `base64` (default): `cover_encoding: "jpeg"`; read `cover_base64`. A successful conversion sets `cover_size: "128x128"`.
- `palette`: `cover_encoding: "palette64"`; read `cover_width`, `cover_height`, `cover_palette` and `cover_pixels`. A successful image is 128×128 with 64 colors and `cover_origin: "top-left"`. Pixels run left to right, top to bottom. Each character's position in `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_` is its palette index; palette colors are decimal RGB. **This pixel string is not a Base64 image**.
- `url`: `cover_encoding: "url"`; download the image from `cover_url`.
- `png`: fetches a remote JPEG, PNG or WebP image and converts it into a 128×128 binary PNG. `cover_encoding: "png"`; download `cover_url`. JSON contains no `cover_base64`. Song slots use a stable `/s/{ID}/cover.png?world=...` URL that can be baked into a world's VRCUrl list.
- `bc1`: download the GPU cover's PNG data atlas from `cover_url`. `cover_encoding` is `png`, and `cover_format: "wimbc1png1"` identifies the BC1 format.

Missing artwork does not prevent lyrics from being returned. The `size` / `cover_size` parameters have no effect on the lyrics endpoint. BC1 clients should read the actual texture dimensions from the PNG data atlas.
:::

```bash
curl -G -A "Unity demo" "https://api.kipfel.link/v3/vrc-danmaku" --data-urlencode "url=https://www.bilibili.com/video/BV1aAhm6vEA6"
curl -G "https://api.kipfel.link/v3/vrc-lyric" --data-urlencode "json=1" --data-urlencode "url=https://music.163.com/song?id=347230"
```

---

### BC1 GPU covers {#bc1-cover}

The BC1 cover URL returns `image/png` containing BC1 blocks for a shader. Use a decoder that supports `wimbc1png1`. The texture preserves the source aspect ratio and defaults to a maximum side of 512 pixels, with top-left origin and no mipmaps.

- Pass an enabled `world` ID in the lyrics request to receive `data.cover_slot`, `data.cover_world`, and a URL such as `https://api.kipfel.link/s/0/cover-gpu.png?world=demo` when artwork is available. Assigning a slot preserves the manual playlist.
- Bake the corresponding HTTPS VRCUrl into the world. Match the full scheme, host, slot ID and `world` parameter if the player reports `COVER_URL_NOT_BAKED`. The slot URL requires no music CDN signature.
- Without `world`, the API returns a dynamic `/cover-gpu.png?url=...` URL when artwork is available. If the requested world is unknown, disabled, or full, the response includes `data.cover_error` and an empty `data.cover_url`. Lyrics remain available.

```bash
curl -G "https://api.kipfel.link/v4/vrc" --data-urlencode "json=1" --data-urlencode "world=demo" --data-urlencode "url=https://music.163.com/song?id=347230"
```

**BC1 Mode JSON Response Example**:

```json
{
  "success": true,
  "data": {
    "platform": "netease",
    "song_name": "海阔天空",
    "singer": "Beyond",
    "duration": 326,
    "lyrics": [
      {
        "time": 18.466,
        "text": "今天我寒夜里看雪飘过",
        "words": [[0, 320, 1], [320, 280, 1], [600, 410, 1]]
      }
    ],
    "tlyrics": [],
    "cover_mode": "bc1",
    "cover_encoding": "png",
    "cover_format": "wimbc1png1",
    "cover_origin": "top-left",
    "cover_mipmaps": false,
    "cover_max_side": 512,
    "cover_slot": 0,
    "cover_world": "demo",
    "cover_url": "https://api.kipfel.link/s/0/cover-gpu.png?world=demo"
  }
}
```

Replace `demo` with your world ID. Add `media=1` to include the audio URL in the top-level `url`.

---

## v4 Merged Endpoint

v4 provides one entry point for video and music, selecting danmaku or lyrics automatically from the link by default.

**Interface**: `/v4/vrc?url={link}`; `/v4/url` is an alias handled by the same function.

| Parameter | Description |
|---|---|
| `url` (required) | Video / music link; use the playlist interfaces below for lists |
| `mode` | Default `auto`; `lyric` / `music` selects the lyrics response, other values select danmaku; auto mode returns lyrics for music |
| `json` | `1` returns the full branch JSON for non-Unity clients too |
| `media` | `1` adds a top-level `url` with the playable direct link to successful JSON responses |
| `world` | World ID; assigning video part / collection IDs requires an existing, enabled world |

::: warning Merged Response
The current implementation does not handle `merge=1`, and `mode=both` does not combine lyrics with danmaku. Responses use the corresponding v3 branch structure. Non-Unity clients with `json=1` also receive the full branch JSON, rather than only `success` / `url`. `Accept` / `format` do not control JSON selection in v3 / v4.
:::

### Call Examples

```bash
curl -G -A "Unity demo" "https://api.kipfel.link/v4/vrc" --data-urlencode "media=1" --data-urlencode "url=https://music.163.com/song?id=347230"
curl -G "https://api.kipfel.link/v4/vrc" --data-urlencode "json=1" --data-urlencode "world=demo" --data-urlencode "url=https://www.bilibili.com/video/BV1aAhm6vEA6?p=2"
curl -I "https://api.kipfel.link/v4/vrc?url=https://music.163.com/song?id=347230"
```


### Video Parts and Collections

Video JSON from v4 adds top-level `playback`. Basic fields are `source`, `title`, `author`, `duration`, `bvid`, `cid`, `page`, `hasParts` and `hasCollection`. Bilibili videos also trigger part and collection lookup.

- `currentId` (number, optional): the current video's / part's video pool ID; play it through `/sell/{currentId}?world={world ID}`.
- `list` (object, optional): the current video's parts, using the video pool structure below.
- `collection` (object, optional): the collection list; `collectionError` contains a collection lookup error code when available.
- `error` (string, optional): examples include `WORLD_REQUIRED`, `WORLD_UNAVAILABLE`, `INVALID_WORLD`, `PART_POOL_UNAVAILABLE` and `PART_METADATA_ERROR`. These describe incomplete metadata or ID assignment and do not necessarily mean the danmaku response failed.

Pass `world` when you need parts / collections. IDs assigned here **do not replace the list manually loaded through `/p`**.

---

## Playlist Parsing Interface

**Interface**: `/v1/playlist?url={playlist link}` or `/v4/playlist?url={playlist link}`; always returns JSON.

NetEase playlists use the compatibility structure below. Use `/p` for Bilibili parts / collections rather than reading video lists from this compatibility endpoint.

```json
{
  "success": true,
  "platform": "网易云音乐",
  "platform_key": "netease",
  "playlist": {
    "id": "https://music.163.com/playlist?id=3778678",
    "name": "Playlist name",
    "creator": "",
    "creator_id": "",
    "cover_url": "",
    "track_count": 1,
    "count": 1,
    "tracks": [
      {
        "index": 1,
        "id": "347230",
        "title": "Song title",
        "name": "Song title",
        "artists": "Artist",
        "album": "Album name",
        "duration": 326,
        "url": "https://api.kipfel.link/v4/vrc?url=https%3A%2F%2Fmusic.163.com%2Fsong%3Fid%3D347230"
      }
    ]
  }
}
```


- `platform_key` (string): platform key; `platform` is the compatibility display name.
- `playlist.id` (string): currently the source URL; `name` is the playlist name.
- `playlist.track_count` / `playlist.count` (number): total tracks / tracks in this response.
- `playlist.tracks[].index` (number): list position starting from 1, **not a world slot ID**.
- `tracks[].id` / `title` / `name` / `artists` / `album` / `duration`: song ID, title, title alias, artist, album and duration in seconds.
- `tracks[].url` (string): a `/v4/vrc?url=...` entry for this song, returning lyrics JSON or a 302 based on the UA.
- `creator`, `creator_id` and `cover_url` are currently empty strings.

The playlist endpoint does not support `nocache=1`. Select songs using their individual links. The old `i` / `index` / `@N` selection methods are not connected.

---

## Slot Relay Interface (VRChat world song requests)

### Background

VRChat Udon cannot construct `VRCUrl` at runtime. A world can bake fixed addresses in advance, with the server assigning the songs or videos behind those IDs.

::: warning World ID
Every interface below requires exactly one `world`: 1–64 English letters, digits or underscores. Duplicate parameters and percent-encoded IDs are rejected. Use an enabled world ID.
:::

### `GET /poll?world={world ID}`

`/slots` is an alias. Omitting `pool` reads the music pool. `/pocl?world=...` always reads the video pool.

```json
{
  "type": "music_playlist",
  "pool": "music",
  "world": "demo",
  "max": 100000,
  "ok": true,
  "name": "Playlist name",
  "source": "https://music.163.com/playlist?id=3778678",
  "platform": "netease",
  "updatedAt": "2026-10-30T08:00:00Z",
  "mode": "idpool",
  "songs": [
    {
      "i": 0,
      "id": "347230",
      "title": "Song title",
      "artist": "Artist",
      "album": "Album name",
      "duration": 326,
      "songUrl": "https://music.163.com/song?id=347230"
    }
  ],
  "total": 1
}
```

#### List fields

| Field | Description |
| --- | --- |
| `world` / `max` | World ID / pool capacity. |
| `mode` | Always `idpool`. |
| `pool` | `music`, `video` (video parts), or `collection` (collections and favorites). |
| `type` | `music_playlist`, `bilibili_parts`, `bilibili_collection` or `bilibili_collections`. |
| `total` | Current list length. The response has no `count` field. |
| `updatedAt` | RFC 3339 timestamp string, for example `2026-10-30T08:00:00Z`. |

#### Reading each pool

| Pool | List field | Entry and response fields |
| --- | --- | --- |
| Music | `songs` | Each `i` is a server-assigned record ID. |
| Video | `items` | Entries include `bvid`, `cid` and `page`. Multi-part entries also have `parts`. The response includes `playableTotal` for playable entries and `collectionId` when a collection is present. |
| Collections and favorites | `items` | Some sources also return `folders`. |

Songs, videos and parts share `id`, `title`, `artist`, `album`, `duration` and `songUrl`. Durations are in seconds.

::: tip List fields and playback IDs
The video pool also serializes the base `songs` field. Read `items` for video lists and treat a missing field as an empty list. Use each entry's `i` to build playback URLs, since server-assigned IDs can be nonconsecutive.
:::

### Playback and details by record ID

| Content | Current list | Playback and details |
| --- | --- | --- |
| Music playlist | `/poll?world=Lucent_01&pool=music`, read `songs` | `/s/{i}?world=Lucent_01`. A Unity UA (case-insensitive) or `json=1` returns lyrics JSON for one song. Other clients receive a 302. |
| Video parts | `/poll?world=Lucent_01&pool=video`, read `items` | `/sell/{i}?world=Lucent_01` returns a 302. For details JSON, use `/video-info/{i}?world=Lucent_01`. |
| Collections and favorites | `/poll?world=Lucent_01&pool=collection`, read `items` | `/sell/{i}?world=Lucent_01&pool=collection` returns details JSON for a Unity UA or `json=1` and a 302 for other clients. `/video-info/{i}?world=Lucent_01&pool=collection` also returns details. |

Use the chosen entry's `i`. If a video entry contains `parts`, use the selected part's `parts[].i`. Record IDs are independent across pools. The details endpoint returns `{ "success": true, "playback": { ... } }` on success.

Use `/p` below to load a new playlist or set of video parts. Use `/poll` for the current list in the selected pool.

### `GET /p?world={world ID}&u={playlist ID or link}`

Loads a player-side list and returns the same kind of structure as `/poll`. `url` is an alias for `u`; a bare number is treated as a NetEase playlist ID. Supports NetEase playlists and Bilibili parts / collections.

Optional `scope`: `auto` (default), `parts` or `collection`, selecting the Bilibili list scope.

```bash
curl -G "https://api.kipfel.link/p" --data-urlencode "world=demo" --data-urlencode "u=3778678"
curl "https://api.kipfel.link/poll?world=demo&pool=music"
curl "https://api.kipfel.link/poll?world=demo&pool=video"
curl "https://api.kipfel.link/poll?world=demo&pool=collection"
curl "https://api.kipfel.link/pocl?world=demo"
curl -G "https://api.kipfel.link/p" --data-urlencode "world=demo" --data-urlencode "u=https://www.bilibili.com/video/BV1aAhm6vEA6" --data-urlencode "scope=parts"
curl -I "https://api.kipfel.link/s/0?world=demo"
curl -I "https://api.kipfel.link/sell/0?world=demo"
curl "https://api.kipfel.link/video-info/0?world=demo"
```


::: warning Side Effects
`/p` replaces the current list for **the same world and pool**. Rooms sharing a world share that list; different worlds and the music, video and collection pools are independent. IDs remain reserved during the protection period, but expired records can be recycled when the pool fills. Do not treat a record ID as a permanent song ID.
:::

---

## Multi-Platform Search & BC1 Covers {#search}

Supports single-platform and mixed search across music platforms (NetEase, Kugou, QQ Music, Kuwo) and video platforms (Bilibili, AcFun). Every item identifies its source platform and provides a `wimbc1png1` BC1 GPU texture cover identical to lyric covers.

**Endpoint**: `GET /search` (alias `GET /v4/search`)

::: tip VRChat Parameter Order
For VRChat worlds, place all fixed parameters (`w`, `p`, `pl`, `m`, `f`, `st`, `dm`, `n`, `pg`) first and put **`q=` last**, so players' keywords can be appended directly. Even when the trailing keyword contains `&`, it will not be truncated.
:::

### Query Parameters (short names also accept full names)

| Short | Full | Default | Description |
| --- | --- | --- | --- |
| `w` | `world` | Optional | Enabled world ID for slot allocation; omit for dynamic BC1 cover URLs |
| `p` | `platform` | `a` | `wy` (NetEase), `kg` (Kugou), `qq` (QQ Music), `kw` (Kuwo), `bv` (Bilibili), `ac` (AcFun), `m` (all music), `v` (all video), `a` (mixed), or comma-separated list e.g. `wy,kg,bv` |
| `pl` | `pool` | Auto from `p` | Pool override: `m` (`search_music`), `v` (`search_video`), `a` (`search_all`) |
| `m` | `preset` | `d` | Field preset: `c` (BC1 cover only), `s` (cover + title), `d` (default, stats off), `f` (full with stats) |
| `f` | `fields` | From `m` | Custom comma-separated field codes (overrides `m`), see below |
| `st` | `stats` | `0` | Pass `1` to enable all engagement stats (views, likes, danmaku, comments, shares, favorites) |
| `dm` | `desc_max` | `80` | Maximum description length (`0` omits description, max `300`) |
| `n` | `limit` | `10` | Result count (`1`–`30`) |
| `pg` | `page` | `1` | Page number (starting at `1`) |
| `q` | `keyword` | **Required (last)** | Search keyword |

### Custom Field Codes (`f=`) & Compact JSON Keys

Engagement stats are off by default to keep payloads compact for VRChat. Enable only the fields you need:

| `f=` Code | Full Alias | JSON Key | Default | Description |
| --- | --- | --- | --- | --- |
| `c` | `cover` | `cv` | On | BC1 texture cover URL |
| `t` | `title` | `t` | On | Title |
| `d` | `desc` | `d` | On | Description (truncated to `dm`) |
| `a` | `artist` / `author` | `a` | On | Artist / uploader |
| `du` | `duration` | `du` | On | Duration in seconds |
| `u` | `play` | `u` | On | Playback / details URL |
| `al` | `album` | `al` | Off | Album name |
| `id` | `id` / `bvid` | `id`, `bv` | Off | Platform item ID / BVID |
| `src` | `url` / `source` | `src` | Off | Source URL |
| `st` | `stats` | `st` | Off | All engagement stats |
| `vw` | `views` | `st.vw` | Off | Views |
| `lk` | `likes` | `st.lk` | Off | Likes |
| `dm` | `danmaku` | `st.dm` | Off | Danmaku count |
| `cm` | `comments` | `st.cm` | Off | Comments |
| `sh` | `shares` | `st.sh` | Off | Shares |
| `fv` | `favorites` | `st.fv` | Off | Favorites |

Each item includes `i` (slot ID when `w` is set), `p` (source platform key), `pn` (platform display name), and `k` (`m` music / `v` video). In `m=c` cover-only mode, only `i`, `p`, and `cv` are returned.

### 3 Independent Search Pools & Fixed Slot URLs

Search uses 3 dedicated pools independent of `music`, `video`, and `collection`:

| Search Pool | Short Path | Fixed BC1 Cover URL | Fixed Play / Details URL |
| --- | --- | --- | --- |
| `search_music` | `m` | `/search/m/{i}/cover-gpu.png?w={world}` | `/search/m/{i}?w={world}` |
| `search_video` | `v` | `/search/v/{i}/cover-gpu.png?w={world}` | `/search/v/{i}?w={world}` |
| `search_all` | `a` | `/search/a/{i}/cover-gpu.png?w={world}` | `/search/a/{i}?w={world}` |

- `/search/{m|v|a}/{i}/cover-gpu.png?w=...` returns `image/png` with `X-Texture-Format: wimbc1png1`, `X-Texture-Origin: top-left`, and `X-Texture-Mipmaps: 0`.
- `/search/{m|v|a}/{i}?w=...` redirects ordinary players (302); Unity UA or `json=1` returns lyrics JSON for music or danmaku JSON for video.
- Current search pool snapshots can also be read via `/poll?world={world}&pool=search_music` (or `search_video`, `search_all`).

### Examples

```bash
# Single music platform (search_music pool)
curl "https://api.kipfel.link/search?w=demo&p=wy&q=Beyond"

# Music-only mixed search (NetEase + Kugou + QQ Music + Kuwo -> search_music pool)
curl "https://api.kipfel.link/search?w=demo&p=m&n=4&q=Beyond"

# Video-only mixed search (Bilibili + AcFun -> search_video pool)
curl "https://api.kipfel.link/search?w=demo&p=v&n=4&q=VRChat"

# BC1 cover only (m=c)
curl "https://api.kipfel.link/search?w=demo&p=bv&m=c&n=10&q=VRChat"

# Custom fields: cover + title + views + likes + danmaku
curl "https://api.kipfel.link/search?w=demo&p=bv&f=c,t,vw,lk,dm&n=8&q=VRChat"

# All-platform mixed search (search_all pool)
curl "https://api.kipfel.link/search?w=demo&p=a&n=10&q=VRChat"
```

### JSON Response Example

Default mode (stats off; includes source platform and BC1 cover URL):

```json
{
  "ok": true,
  "pl": "search_all",
  "w": "demo",
  "ce": "png",
  "cf": "wimbc1png1",
  "co": "top-left",
  "cm": false,
  "cs": 512,
  "total": 2,
  "items": [
    {
      "i": 0,
      "p": "netease",
      "pn": "网易云音乐",
      "k": "m",
      "t": "Sunny Day",
      "a": "Artist",
      "du": 269,
      "cv": "https://api.kipfel.link/search/a/0/cover-gpu.png?w=demo",
      "u": "https://api.kipfel.link/search/a/0?w=demo"
    },
    {
      "i": 1,
      "p": "bilibili",
      "pn": "哔哩哔哩",
      "k": "v",
      "t": "VRChat Showcase",
      "a": "Creator",
      "d": "Short description...",
      "du": 185,
      "cv": "https://api.kipfel.link/search/a/1/cover-gpu.png?w=demo",
      "u": "https://api.kipfel.link/search/a/1?w=demo"
    }
  ]
}
```

In `m=c` cover-only mode, each item contains only `i`, `p`, and `cv`. When stats are requested (e.g. `f=c,t,vw,lk,dm`), items include `"st": { "vw": 139021, "lk": 6119, "dm": 145 }`.

---

## General Response Format

### Error Response Format

Legacy v1 / v3 / v4 parameter validation and ordinary playback parsing errors use this structure:

```json
{
  "error": true,
  "status": 400,
  "code": "PARSE_ERROR",
  "message": "链接解析失败，请检查链接是否正确"
}
```


Business errors from `/p`, `/poll`, `/slots`, `/pocl`, `/s/{n}`, `/sell/{n}` and `/video-info/{n}` contain `error`, `code`, `message` and optional `details`, **without a `status` field**. Read the HTTP status from the response itself.

| code | HTTP | Description |
|---|---|---|
| `METHOD_NOT_ALLOWED` | 405 | Missing url parameter |
| `IM_A_TEAPOT` | 418 | Empty url |
| `INVALID_URL / UNSUPPORTED_PLATFORM` | 422 | Invalid URL / unsupported platform |
| `CONTENT_RESTRICTED` | 451 | Restricted content |
| `PARSE_ERROR` | 400 / 403 | Playback / non-Unity JSON parsing failure; rule blocks use 403 |
| `INVALID_WORLD / INVALID_POOL` | 400 | Invalid world or pool parameter |
| `INVALID_INPUT / INVALID_LIST_SCOPE` | 400 | Missing list source or invalid scope |
| `UNSUPPORTED_PLAYLIST` | 400 | Unsupported platform for lists |
| `WORLD_OR_SLOT_NOT_FOUND` | 404 | World or record not found |
| `WORLD_DISABLED` | 403 | World disabled |
| `SLOT_POOL_FULL` | 409 | Pool full and remaining records protected |
| `SLOT_MEDIA_ERROR` | 502 | Record media URL parsing failed |

::: tip Soft Failures for Lyrics / Danmaku
After parameter validation, parsing failures in the Unity branch of v3 / v4 still return HTTP 200 and an empty `success: true` response. Danmaku uses `platform: "generic"` and `comments: []`; lyrics use `generic`, empty lyrics and empty cover fields. These empty responses may omit `cover_mode`. Missing parameters or invalid URLs still return validation errors.

The v1 JSON branch instead returns `{ "code": -1, "message": "...", "log_id": "req_...", "data": {} }` on parsing failure, usually with HTTP 200, or 403 for a rule block. Do not use HTTP status alone to decide whether parsing succeeded.
:::

---

## Restricted Content

Content from the following platforms is not supported for parsing:

- 腾讯视频 (Tencent Video)
- 爱奇艺 (iQiyi)
- 优酷 (Youku)
- 芒果TV (Mango TV)
- 哔哩哔哩番剧 (Bilibili Anime)
- 西瓜视频 (Xigua Video)
- 搜狐视频 (Sohu Video)

---

## Related Pages

- [Usage Guide](/en/video-parser/guide) — how to call each endpoint and the supported link formats
- [FAQ](/en/video-parser/faq) — troubleshooting playback and parsing failures, plus error codes
- [CDN Domain List](/en/video-parser/cdn) — domains to add to your VRChat map whitelist
