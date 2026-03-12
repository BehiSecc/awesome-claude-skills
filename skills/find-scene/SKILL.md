---
name: find-scene
description: Search movie and TV show scenes by dialog, time, or visual description. Download video clips, extract frames, find quotes, identify movies from quotes, and query IMDB data. Use when the user wants to find a specific scene, download a clip, search for a quote in a movie/show, extract a frame, or get movie information via the find-scene API.
---

# find-scene API Skill

Search and download movie/TV show scenes by dialog, time, or visual description.

## Authentication

Every API request requires a `_token` field in the JSON body.

```json
{ "_token": "user-api-token", ...other fields }
```

### How to get a token

1. Go to https://find-scene.com and sign in
2. In the chat, ask the bot to "generate an API token"
3. The bot will return a token string — keep it secret
4. Include it as `_token` in every API request body

To revoke a token, ask the bot to "revoke my API token".

## Base URL

```
https://api.find-scene.com
```

All endpoints are `POST` with `Content-Type: application/json`, except `GET /api/operation/{id}`.

## Response Format

All successful responses are wrapped in `{ "result": <structured object> }`. Each endpoint returns a typed JSON object (not a plain string). Error responses use `{ "error": "message" }` at the top level (HTTP 4xx/5xx) or `{ "result": { "error": "message" } }` for domain-level errors within a 200 response.

### Video Source Hash

An internal find-scene ID for a video file. Obtained from `get_best_video_source`. This is NOT an IMDB ID or filename. Required for downloads, frame extraction, and high-accuracy text source lookups.

### Text Source Hash

An internal find-scene ID for a subtitle/text file. Obtained from `get_text_source` or `get_high_accuracy_text_source`. Required for phrase search and subtitle retrieval. NOT a filename or IMDB ID.

### Async Operations

`download_by_time` and `extract_frame` return an operation ID (not a download URL). You must poll `GET /api/operation/{id}` until status is `completed`, then use the `url` field from the response.

**Statuses:** `in_progress`, `completed`, `failed`, `cancelled`

### Video Query Object

Many endpoints accept a video query object with these optional fields:

| Field | Type | Description |
|-------|------|-------------|
| `movieOrTVShowName` | string/null | Movie or TV show name |
| `dubbed` | string/null | Dubbing language code, e.g. "en", "de", "pt-br" |
| `animated` | boolean/null | Is the video animated |
| `blackAndWhite` | boolean/null | Is the video black and white |
| `year` | number/null | Year (for distinguishing remakes) |
| `isSeries` | boolean/null | User explicitly said it's a series |
| `season` | number/null | Season number (series only) |
| `episode` | number/null | Episode number (series only) |

### Time Format

All time parameters use `HH:MM:SS` format, e.g. `"00:01:30"`.

### Display Params Object

Used by `download_by_time` and `extract_frame`:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `removeWatermark` | boolean | `false` | Pro users only |
| `gif` | boolean | `false` | Only if user explicitly asked for GIF |
| `mobile` | boolean | `false` | Crop to mobile format. Only if user asked |

## Typical Workflows

### Workflow 1: Find and download a scene by quote

```
1. quote_to_movie        -> identify which movie contains the quote
2. get_best_video_source -> get videoHash for that movie
3. get_text_source       -> get textSource hash
4. search_phrase         -> find exact timestamp of the quote
5. download_by_time      -> schedule clip download (returns operation ID)
6. GET /api/operation/id -> poll until completed, get download URL
```

### Workflow 2: Download a scene by time

```
1. get_best_video_source -> get videoHash
2. download_by_time      -> schedule download with start/end times
3. GET /api/operation/id -> poll until completed
```

### Workflow 3: Search by visual scene description

```
1. find_by_scene_description -> search by what happens visually
2. get_best_video_source     -> get videoHash for the match
3. download_by_time          -> download the scene
4. GET /api/operation/id     -> poll until completed
```

### Workflow 4: Find which episode contains a quote (TV series)

```
1. find_episode_by_phrase -> find season/episode for a phrase
2. get_best_video_source  -> get videoHash with season/episode
3. get_text_source        -> get textSource
4. search_phrase          -> get exact timestamp
5. download_by_time       -> download clip
6. GET /api/operation/id  -> poll until completed
```

### Workflow 5: Extract a frame / screenshot

```
1. get_best_video_source -> get videoHash
2. extract_frame         -> schedule frame extraction (returns operation ID)
3. GET /api/operation/id -> poll until completed, get image URL
```

## API Endpoints Reference

### Video Source Tools

#### `POST /api/get_best_video_source`

Get the best video source for a movie or TV show.

```json
{
  "_token": "...",
  "query": {
    "movieOrTVShowName": "The Matrix",
    "year": 1999
  },
  "timeoutSeconds": 60
}
```

- `query` (required): Video query object (see above)
- `timeoutSeconds` (optional): Max wait time. Default 60. Do not manually convert minutes to seconds.
- Returns: `{ "result": { "sources": ["videoHash1", ...], "error": "..." } }` — `sources` is always present; `error` is optional

### Text Source Tools

#### `POST /api/get_text_source`

Get a text/subtitle source hash by movie details. Less accurate timing than `get_high_accuracy_text_source`.

```json
{
  "_token": "...",
  "query": { "movieOrTVShowName": "The Matrix" },
  "language": "en",
  "minDuration": 3600
}
```

- `query` (required): Video query object
- `language` (optional): Subtitle language, e.g. "en", "pt-br", "de"
- `minDuration` (optional): Minimum subtitle file duration in seconds
- Returns: `{ "result": { "textSources": ["hash1", ...] } }` or `{ "result": { "error": "..." } }`

#### `POST /api/get_high_accuracy_text_source`

Get a text source with accurate timing. Requires a videoHash from `get_best_video_source`.

```json
{
  "_token": "...",
  "query": { "movieOrTVShowName": "The Matrix" },
  "videoHash": "abc123...",
  "language": "en"
}
```

- `query` (required): Video query object
- `videoHash` (required): From `get_best_video_source`
- `language` (optional): Subtitle language
- Returns: `{ "result": { "textSources": ["hash1", ...] } }` or `{ "result": { "error": "..." } }`

### Text Search Tools

#### `POST /api/search_phrase`

Search for a phrase/quote in a text source's subtitles.

```json
{
  "_token": "...",
  "textSource": "textSourceHash",
  "phraseSearchParams": {
    "phraseStart": "I know kung fu",
    "phraseEnd": "",
    "nSkip": 0,
    "maxOccurrences": 1
  }
}
```

- `textSource` (required): Text source hash
- `phraseSearchParams.phraseStart` (required): The phrase to search. Do not include speaker names. For long text, split using `phraseEnd`.
- `phraseSearchParams.phraseEnd` (optional): End of phrase if split
- `phraseSearchParams.nSkip` (required): Results to skip (default 0)
- `phraseSearchParams.maxOccurrences` (required): Max results (default 1)
- Returns: `{ "result": { "occurrences": [{ "time": "HH:MM:SS", "rangeStart": "HH:MM:SS", "rangeEnd": "HH:MM:SS", "srt": "subtitle text" }, ...] } }` or `{ "result": { "error": "..." } }`

#### `POST /api/get_srt_entries_around_phrase`

Get subtitle entries in a time window around a phrase occurrence.

```json
{
  "_token": "...",
  "textSource": "textSourceHash",
  "phrase": "I know kung fu",
  "limit": 1,
  "skip": 0,
  "secondsBefore": 5,
  "secondsAfter": 5
}
```

All fields are required.
- Returns: same as `search_phrase` — `{ "result": { "occurrences": [...] } }` or `{ "result": { "error": "..." } }`

#### `POST /api/get_srt_entries_by_time_range`

Get subtitle entries for a video within a time range.

```json
{
  "_token": "...",
  "videoQuery": { "movieOrTVShowName": "The Matrix" },
  "startTime": "00:30:00",
  "endTime": "00:31:00",
  "subsLanguage": "en"
}
```

- `videoQuery` (required): Video query object
- `startTime`, `endTime` (required): Time range in HH:MM:SS
- `subsLanguage` (optional): Subtitle language
- Returns: `{ "result": { "srt": "subtitle text" } }` or `{ "result": { "error": "..." } }`

### Video Download Tools

#### `POST /api/download_by_time`

Schedule a video clip download. Returns an operation ID, NOT a URL.

```json
{
  "_token": "...",
  "videoHash": "abc123...",
  "startTime": "00:30:00",
  "endTime": "00:30:15",
  "textSource": "textSourceHash",
  "srtOffset": 0,
  "displayParams": {
    "removeWatermark": false,
    "gif": false,
    "mobile": false
  }
}
```

- `videoHash` (required): From `get_best_video_source`
- `startTime`, `endTime` (required): Clip bounds in HH:MM:SS
- `displayParams` (required): See display params object above
- `textSource` (optional): To burn subtitles into the clip
- `srtOffset` (optional): Subtitle time correction offset
- Returns: `{ "result": { "operationId": "..." } }` or `{ "result": { "error": "..." } }` — you MUST poll the operationId

#### `POST /api/extract_frame`

Extract a single frame/screenshot from a video. Returns an operation ID.

```json
{
  "_token": "...",
  "videoHash": "abc123...",
  "time": "00:30:05",
  "textSource": "textSourceHash",
  "overrideTextTop": "TOP TEXT",
  "overrideTextBottom": "BOTTOM TEXT",
  "displayParams": {
    "removeWatermark": false,
    "gif": false,
    "mobile": false
  }
}
```

- `videoHash` (required): From `get_best_video_source`
- `time` (required): Frame time in HH:MM:SS
- `textSource` (optional): Overlay subtitles on frame
- `overrideTextTop`, `overrideTextBottom` (optional): Custom text overlay (meme mode)
- `displayParams` (optional): See display params object
- Returns: `{ "result": { "operationId": "..." } }` or `{ "result": { "error": "..." } }` — you MUST poll the operationId

#### `POST /api/cancel_operation`

Cancel a stuck async operation.

```json
{ "_token": "...", "id": "operation-id" }
```

- Returns: `{ "result": { "cancelled": true, "operationId": "..." } }` or `{ "result": { "cancelled": false, "reason": "..." } }`

### Async Operation Polling

#### `GET /api/operation/{id}`

Poll the status of an async operation. No request body. No auth token needed in query (it was provided when creating the operation).

Response:
```json
{
  "status": "completed",
  "progress": 100,
  "result": "...",
  "url": "https://signed-download-url..."
}
```

**Polling strategy:** Wait 2-3 seconds between polls. Typical downloads complete in 10-30 seconds. Give up after ~2 minutes.

### Movie Information Tools

#### `POST /api/query_imdb`

Get movie/show information from IMDB.

```json
{
  "_token": "...",
  "title": "The Matrix",
  "year": 1999,
  "imdbId": "tt0133093",
  "season": 1,
  "episode": 1
}
```

- `title` (required): Movie/show title
- All other fields optional
- Returns: `{ "result": { "name": "...", "imdb": "tt...", "year": 1999, "season": 1, "episode": 1, ... } }` — movie metadata object with available IMDB fields

#### `POST /api/is_string_a_movie_name`

Check if a string is a movie/show name.

```json
{ "_token": "...", "string": "The Matrix" }
```

- Returns: `{ "result": { "isMovieName": true } }` or `{ "result": { "isMovieName": false } }`

#### `POST /api/quote_to_movie`

Identify which movie a quote is from.

```json
{ "_token": "...", "quote": "I know kung fu" }
```

- Returns: `{ "result": { "candidates": ["Movie Name 1", "Movie Name 2", ...] } }`

#### `POST /api/popular_quotes_from_title`

Get popular quotes from a movie or TV show.

```json
{
  "_token": "...",
  "name": "The Matrix",
  "limit": 10,
  "imdb": "tt0133093"
}
```

- `name` (required): Movie/show name
- `limit` (required): Max number of quotes
- `imdb` (optional): IMDB ID
- Returns: `{ "result": { "quotes": ["quote 1", "quote 2", ...] } }`

#### `POST /api/compute_running_time`

Get running time of a movie/show.

```json
{ "_token": "...", "imdbId": "tt0133093" }
```

- Returns: `{ "result": { "runtimeSeconds": 8160 } }` or `{ "result": { "runtimeSeconds": null } }` if not found

### TV Series Tools

#### `POST /api/find_episode_by_phrase`

Find which episode of a TV series contains a phrase.

```json
{
  "_token": "...",
  "phrase": "We were on a break",
  "videoQuery": {
    "name": "Friends",
    "season": 3
  },
  "season": 3,
  "limit": 5
}
```

- `phrase` (required): The phrase to search
- `videoQuery` (required): Must include `name`. Can include `imdb`, `season`, `episode`, `year`, `seasonEnd`, `episodeEnd`, `dubbed`, `animated`, `blackAndWhite`
- `season` (optional): Limit search to specific season
- `limit` (optional): Max results
- Returns: `{ "result": { "episodes": [{ "season": 3, "episode": 15, "context": ["surrounding subtitle lines", ...] }, ...] } }`

### Scene Description Search

#### `POST /api/find_by_scene_description`

Search for a scene by visual description (not dialog).

```json
{
  "_token": "...",
  "description": "Neo dodges bullets on a rooftop",
  "video": { "movieOrTVShowName": "The Matrix" },
  "nResults": 3,
  "nSkip": 0,
  "scoreThreshold": 0.4
}
```

- `description` (required): What happens in the scene visually. Do NOT include the movie name here.
- `video` (optional): Video query object to narrow search to a specific title
- `nResults` (required): Number of results to return
- `nSkip` (optional): Skip results (for pagination / "show me another")
- `scoreThreshold` (optional): Minimum similarity score 0-1. Use ~0.6 for specific scenes, ~0.3 for vague descriptions.
- Returns: `{ "result": { "results": [{ "query": { "movieOrTVShowName": "...", ... }, "time": "HH:MM:SS", "score": 0.85 }, ...], "warning": "..." } }` or `{ "result": { "error": "..." } }`

#### `POST /api/request_indexing_for_scene_description`

Request that a movie/show be indexed for scene description search. Use when `find_by_scene_description` returns no results for a known title.

```json
{
  "_token": "...",
  "video": {
    "name": "The Matrix",
    "season": 1,
    "episode": 1
  }
}
```

- `video.name` (required): Movie/show name
- Other fields optional: `imdb`, `season`, `episode`, `year`, `seasonEnd`, `episodeEnd`, `dubbed`, `animated`, `blackAndWhite`
- Returns: `{ "result": { "requested": true } }`

### Transcription

### Account

#### `POST /api/check_quota`

Check remaining search credits for the current month.

```json
{ "_token": "..." }
```

- Returns: `{ "result": { "creditsRemaining": 42 } }`

## Error Handling

- **400**: Invalid parameters (check required fields)
- **401**: Invalid or missing `_token`
- **500**: Internal server error (retry or report)

## Tips

- Always get the video source hash first before attempting downloads or text source lookups.
- Use `get_high_accuracy_text_source` (with videoHash) over `get_text_source` when you have a video source, for better subtitle timing alignment.
- The `download_by_time` and `extract_frame` results are operation IDs. Never return these to the user as download links. Always poll until you get the actual URL.
- Keep clip durations reasonable (under 60 seconds) to avoid long processing times.
- For TV series, use `find_episode_by_phrase` first to identify the episode before searching within it.
- The `find_by_scene_description` endpoint requires the video to have been indexed. If it returns no results, use `request_indexing_for_scene_description` and try again later.
- OpenAPI spec is available at `https://api.find-scene.com/api/openapi.json` for machine-readable schema details.
