---
description: Download YouTube video transcripts using yt-dlp. Use when user provides a YouTube URL and wants the transcript, captions, or subtitles.
---

# YouTube Transcript Downloader

Download transcripts (subtitles/captions) from YouTube videos using yt-dlp.

## User Input

```text
$ARGUMENTS
```

## Output Directory

**IMPORTANT**: Always save transcripts to `./yt-transcripts/` subfolder in the current working directory.

```bash
mkdir -p ./yt-transcripts
```

All transcript files should be saved to this folder, not the current directory.

## Instructions

### Priority Order:
1. **Create output directory** - `mkdir -p ./yt-transcripts`
2. **Check if yt-dlp is installed** - install if needed
3. **List available subtitles** - see what's actually available
4. **Try manual subtitles first** (`--write-sub`) - highest quality
5. **Fallback to auto-generated** (`--write-auto-sub`) - usually available
6. **Last resort: Whisper transcription** - if no subtitles exist (requires user confirmation)
7. **Confirm the download** and show the user where the file is saved
8. **Optionally clean up** the VTT format if the user wants plain text

## Installation Check

**IMPORTANT**: Always check if yt-dlp is installed first:

```bash
which yt-dlp || command -v yt-dlp
```

### If Not Installed

**macOS (Homebrew)**:
```bash
brew install yt-dlp
```

**Alternative (pip)**:
```bash
pip3 install yt-dlp
```

## Check Available Subtitles

**ALWAYS do this first** before attempting to download:

```bash
yt-dlp --list-subs "YOUTUBE_URL"
```

## Download Strategy

### Option 1: Manual Subtitles (Preferred)

```bash
yt-dlp --write-sub --skip-download --output "/tmp/transcript_%(id)s" "YOUTUBE_URL"
```

### Option 2: Auto-Generated Subtitles (Fallback)

```bash
yt-dlp --write-auto-sub --skip-download --output "/tmp/transcript_%(id)s" "YOUTUBE_URL"
```

## Option 3: Whisper Transcription (Last Resort)

**ONLY use this if both manual and auto-generated subtitles are unavailable.**

1. Show file size and ask for user confirmation
2. Check for Whisper installation (`pip3 install openai-whisper`)
3. Download audio: `yt-dlp -x --audio-format mp3 --output "audio_%(id)s.%(ext)s" "YOUTUBE_URL"`
4. Transcribe: `whisper audio_VIDEO_ID.mp3 --model base --output_format vtt`
5. Cleanup audio file after transcription

## Getting Video Information

```bash
yt-dlp --print "%(title)s" "YOUTUBE_URL"
```

## Post-Processing: Convert to Plain Text

YouTube's auto-generated VTT files contain duplicate lines. Always deduplicate and save to `./yt-transcripts/`:

```bash
mkdir -p ./yt-transcripts
VIDEO_TITLE=$(yt-dlp --print "%(title)s" "YOUTUBE_URL" | tr '/' '_' | tr ':' '-' | tr '?' '' | tr '"' '' | tr ' ' '_')
VTT_FILE=$(ls /tmp/*.vtt | head -n 1)

python3 -c "
import sys, re
seen = set()
with open('$VTT_FILE', 'r') as f:
    for line in f:
        line = line.strip()
        if line and not line.startswith('WEBVTT') and not line.startswith('Kind:') and not line.startswith('Language:') and '-->' not in line:
            clean = re.sub('<[^>]*>', '', line)
            clean = clean.replace('&amp;', '&').replace('&gt;', '>').replace('&lt;', '<')
            if clean and clean not in seen:
                print(clean)
                seen.add(clean)
" > "./yt-transcripts/${VIDEO_TITLE}.txt"

rm "$VTT_FILE"
echo "Saved to: ./yt-transcripts/${VIDEO_TITLE}.txt"
```

## Output Formats

- **VTT format** (`.vtt`): Includes timestamps, good for video players
- **Plain text** (`.txt`): Just text content, good for reading/analysis

## Error Handling

| Issue | Solution |
|-------|----------|
| yt-dlp not installed | `brew install yt-dlp` or `pip3 install yt-dlp` |
| No subtitles available | Offer Whisper transcription (with user confirmation) |
| Invalid/private video | Check URL format, inform user of error |
| Multiple languages | Use `--sub-langs en` for English only |

## Best Practices

- Always check what's available before downloading (`--list-subs`)
- Ask user before large downloads (audio files, Whisper models)
- Clean up temporary files after processing
- Provide clear feedback at each stage
