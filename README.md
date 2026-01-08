# yt-summary

A Claude AI plugin that downloads YouTube video transcripts and generates well-structured markdown summaries with key insights, timestamps, and references.

## Features

- Downloads transcripts using `yt-dlp` with automatic fallback strategies
- Extracts video metadata (title, channel, duration, upload date)
- Parses video descriptions for referenced links and resources
- Generates comprehensive markdown summaries with:
  - TL;DR executive summary
  - Key takeaways
  - Detailed section-by-section breakdown
  - Notable quotes with timestamps
  - Categorized references and resources
  - Collapsible full transcript

## Requirements

**Required:**
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) - YouTube video/audio downloader

**Optional:**
- [openai-whisper](https://github.com/openai/whisper) - For audio transcription when subtitles are unavailable

## Installation

### 1. Install Dependencies

```bash
# macOS
brew install yt-dlp

# All platforms
pip3 install yt-dlp

# Optional: Whisper for audio transcription
pip3 install openai-whisper
```

### 2. Install the Plugin

Add this plugin to your Claude Code configuration. The plugin will be available from the GitHub repository `jpoley/yt-summary`.

## Usage

### Download Transcript Only

```
/youtube:transcript https://www.youtube.com/watch?v=VIDEO_ID
```

Output: `./yt-transcripts/VIDEO_TITLE.txt`

### Download and Summarize

```
/yt-summarize https://www.youtube.com/watch?v=VIDEO_ID
```

Output: `./yt-transcripts/VIDEO_TITLE.summary.md`

Both commands support standard YouTube URLs:
- `https://www.youtube.com/watch?v=VIDEO_ID`
- `https://youtu.be/VIDEO_ID`

## How It Works

### Transcript Download Strategy

The transcript command attempts to fetch subtitles in priority order:

1. **Manual subtitles** - Highest quality, human-created
2. **Auto-generated subtitles** - YouTube's automatic captions
3. **Whisper transcription** - Last resort, requires user confirmation

Downloaded VTT files are processed to remove duplicates, clean HTML entities, and convert to plain text.

### Summary Generation

The summarizer extracts metadata and generates a structured markdown document:

```markdown
# Video Title

**Channel:** Channel Name | **Date:** YYYY-MM-DD | **Duration:** HH:MM:SS

## TL;DR
Brief 2-3 sentence executive summary.

## Key Takeaways
- Main insight 1
- Main insight 2
- ...

## Summary
Detailed breakdown organized by topic or timestamp.

## Notable Quotes
> "Exact quote from the video" - Speaker (timestamp)

## References & Resources
- [Resource Name](url) - Description

<details>
<summary>Full Transcript</summary>
Complete transcript text...
</details>
```

## Output Directory

All output files are saved to `./yt-transcripts/` in the current working directory:

```
yt-transcripts/
├── Video Title.txt           # Raw transcript
└── Video_Title.summary.md    # Generated summary
```

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.
