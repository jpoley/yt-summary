---
description: Download a YouTube video transcript and generate a well-formatted markdown summary with key insights, dialogue, and references.
---

# YouTube Video Summarizer

Download and summarize YouTube videos into structured markdown documents.

## User Input

```text
$ARGUMENTS
```

## Instructions

### Step 1: Download Transcript

First, run the `/youtube:transcript` skill with the provided YouTube URL to download the transcript.

The transcript will be saved to `./yt-transcripts/VIDEO_TITLE.txt`.

### Step 2: Extract Video ID and Get Metadata

Extract the VIDEO_ID directly from the input URL:
- `youtube.com/watch?v=VIDEO_ID` → parse the `v` parameter
- `youtu.be/VIDEO_ID` → parse after the `/`

```bash
# Extract VIDEO_ID from URL (handles both youtube.com and youtu.be formats)
VIDEO_ID=$(echo "YOUTUBE_URL" | sed -E 's/.*[?&]v=([^&]+).*/\1/' | sed -E 's/.*youtu\.be\/([^?]+).*/\1/')

# Get other metadata
VIDEO_TITLE=$(yt-dlp --print "%(title)s" "YOUTUBE_URL")
VIDEO_TITLE_SAFE=$(echo "$VIDEO_TITLE" | tr '/' '_' | tr ':' '-' | tr '?' '' | tr '"' '' | tr ' ' '_')
CHANNEL=$(yt-dlp --print "%(channel)s" "YOUTUBE_URL")
UPLOAD_DATE=$(yt-dlp --print "%(upload_date)s" "YOUTUBE_URL")
DURATION=$(yt-dlp --print "%(duration_string)s" "YOUTUBE_URL")
DESCRIPTION=$(yt-dlp --print "%(description)s" "YOUTUBE_URL")
```

Use `VIDEO_TITLE_SAFE` for filenames (spaces replaced with underscores).

**For the summary header:**
- Thumbnail URL: `https://img.youtube.com/vi/VIDEO_ID/maxresdefault.jpg`
- Video link: `https://www.youtube.com/watch?v=VIDEO_ID`

### Step 3: Extract Links from Description

Parse the description to extract all URLs. Common patterns include:
- Direct GitHub links: `https://github.com/...`
- Shortened links: `https://link.*/...`
- Timestamps with links: `00:00 - Title https://...`
- Social/contact links

Extract and categorize links:
```bash
echo "$DESCRIPTION" | grep -oE 'https?://[^ ]+' | sort -u
```

For each link, preserve the context (what it's for) from the description.

### Step 4: Generate Summary

Read the downloaded transcript and generate a comprehensive markdown summary with this structure:

```markdown
# [Video Title]

[![Video Thumbnail](https://img.youtube.com/vi/VIDEO_ID/maxresdefault.jpg)](YOUTUBE_URL)

**🎬 [Watch on YouTube](YOUTUBE_URL)**

| | |
|---|---|
| **Channel** | [Channel Name] |
| **Date** | [Upload Date formatted as Month Day, Year] |
| **Duration** | [Duration] |

---

## TL;DR

[2-3 sentence executive summary of the main point]

## Key Takeaways

- [Bullet point 1]
- [Bullet point 2]
- [Bullet point 3]
- [etc.]

## Summary

[Detailed summary broken into logical sections with headers]

### [Section 1 Title]

[Content with inline quotes from the speaker where impactful]

### [Section 2 Title]

[Continue with more sections as needed]

## Notable Quotes

> "[Exact quote from transcript]"

> "[Another notable quote]"

## Chapters

[If timestamps are in the description, format them as a table:]

| Time | Topic | Link |
|------|-------|------|
| 00:00 | Introduction | |
| 00:11 | [Topic Name] | [GitHub](https://...) |

## References & Resources

### From Description

[List all links from the video description with their context:]

- **[Project/Resource Name]**: [URL]
- **[Another Resource]**: [URL]

### Mentioned in Video

- [Any additional tools, products, concepts mentioned in the transcript]
- [Technologies discussed]
- [People or companies referenced]

---

## Full Transcript

<details>
<summary>Click to expand full transcript</summary>

[Full transcript text here]

</details>
```

### Step 5: Save Summary

Save the generated markdown summary to:
```
./yt-transcripts/[VIDEO_TITLE_SAFE].summary.md
```

### Guidelines for Summary Generation

1. **Dialogue Format**: If the video is a conversation/interview, preserve the dialogue format with speaker labels when identifiable
2. **Key Concepts**: Bold important terms and concepts on first mention
3. **Code/Technical**: Use code blocks for any code, commands, or technical syntax mentioned
4. **Timestamps**: If specific timestamps are notable, include them in parentheses
5. **Links**: If URLs are mentioned verbally, try to reconstruct them
6. **Actionable Items**: Highlight any actionable advice or steps the viewer should take

### Output

Confirm to the user:
1. Transcript location: `./yt-transcripts/[VIDEO_TITLE_SAFE].txt`
2. Summary location: `./yt-transcripts/[VIDEO_TITLE_SAFE].summary.md`
3. Brief preview of the TL;DR section
