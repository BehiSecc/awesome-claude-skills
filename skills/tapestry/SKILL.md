---
name: tapestry
description: Unified content extraction and action planning. Use when user says "tapestry <URL>", "weave <URL>", "help me plan <URL>", "extract and plan <URL>", "make this actionable <URL>", or similar phrases indicating they want to extract content and create an action plan. Automatically detects content type (YouTube video, article, PDF) and processes accordingly.
allowed-tools: Bash,Read,Write
---

# Tapestry: Unified Content Extraction + Action Planning

This is the **master skill** that orchestrates the entire Tapestry workflow:
1. Detect content type from URL
2. Extract content using appropriate skill
3. Automatically create a Ship-Learn-Next action plan

## When to Use This Skill

Activate when the user:
- Says "tapestry [URL]"
- Says "weave [URL]"
- Says "help me plan [URL]"
- Says "extract and plan [URL]"
- Says "make this actionable [URL]"
- Says "turn [URL] into a plan"
- Provides a URL and asks to "learn and implement from this"
- Wants the full Tapestry workflow (extract → plan)

**Keywords to watch for**: tapestry, weave, plan, actionable, extract and plan, make a plan, turn into action

## How It Works

### Complete Workflow:
1. **Detect URL type** (YouTube, article, PDF)
2. **Extract content** using appropriate skill:
   - YouTube → youtube-transcript skill
   - Article → article-extractor skill
   - PDF → download and extract text
3. **Create action plan** using ship-learn-next skill
4. **Save both** content file and plan file
5. **Present summary** to user

## URL Detection Logic

### YouTube Videos

**Patterns to detect:**
- `youtube.com/watch?v=`
- `youtu.be/`
- `youtube.com/shorts/`
- `m.youtube.com/watch?v=`

**Action:** Use youtube-transcript skill

### Web Articles/Blog Posts

**Patterns to detect:**
- `http://` or `https://`
- NOT YouTube, NOT PDF
- Common domains: medium.com, substack.com, dev.to, etc.
- Any HTML page

**Action:** Use article-extractor skill

### PDF Documents

**Patterns to detect:**
- URL ends with `.pdf`
- URL returns `Content-Type: application/pdf`

**Action:** Download and extract text

### Other Content

**Fallback:**
- Try article-extractor (works for most HTML)
- If fails, inform user of unsupported type

## Step-by-Step Workflow

### Step 1: Detect Content Type

```bash
URL="$1"

# Check for YouTube
if [[ "$URL" =~ youtube\.com/watch || "$URL" =~ youtu\.be/ || "$URL" =~ youtube\.com/shorts ]]; then
    CONTENT_TYPE="youtube"

# Check for PDF
elif [[ "$URL" =~ \.pdf$ ]]; then
    CONTENT_TYPE="pdf"

# Check if URL returns PDF
elif curl -sI "$URL" | grep -i "Content-Type: application/pdf" > /dev/null; then
    CONTENT_TYPE="pdf"

# Default to article
else
    CONTENT_TYPE="article"
fi

echo "📍 Detected: $CONTENT_TYPE"
```

### Step 2: Extract Content (by Type)

#### YouTube Video

```bash
# Use youtube-transcript skill workflow
echo "📺 Extracting YouTube transcript..."

# 1. Check for yt-dlp
if ! command -v yt-dlp &> /dev/null; then
    echo "Installing yt-dlp..."
    brew install yt-dlp
fi

# 2. Get video title
VIDEO_TITLE=$(yt-dlp --print "%(title)s" "$URL" | tr '/' '_' | tr ':' '-' | tr '?' '' | tr '"' '')

# 3. Download transcript
yt-dlp --write-auto-sub --skip-download --sub-langs en --output "temp_transcript" "$URL"

# 4. Convert to clean text (deduplicate)
python3 -c "
import sys, re
seen = set()
vtt_file = 'temp_transcript.en.vtt'
try:
    with open(vtt_file, 'r') as f:
        for line in f:
            line = line.strip()
            if line and not line.startswith('WEBVTT') and not line.startswith('Kind:') and not line.startswith('Language:') and '-->' not in line:
                clean = re.sub('<[^>]*>', '', line)
                clean = clean.replace('&amp;', '&').replace('&gt;', '>').replace('&lt;', '<')
                if clean and clean not in seen:
                    print(clean)
                    seen.add(clean)
except FileNotFoundError:
    print('Error: Could not find transcript file', file=sys.stderr)
    sys.exit(1)
" > "${VIDEO_TITLE}.txt"

# 5. Cleanup
rm -f temp_transcript.en.vtt

CONTENT_FILE="${VIDEO_TITLE}.txt"
echo "✓ Saved transcript: $CONTENT_FILE"
```

#### Article/Blog Post

```bash
# Use article-extractor skill workflow
echo "📄 Extracting article content..."

# 1. Check for extraction tools
if command -v reader &> /dev/null; then
    TOOL="reader"
elif command -v trafilatura &> /dev/null; then
    TOOL="trafilatura"
else
    TOOL="fallback"
fi

echo "Using: $TOOL"

# 2. Extract based on tool
case $TOOL in
    reader)
        reader "$URL" > temp_article.txt
        ARTICLE_TITLE=$(head -n 1 temp_article.txt | sed 's/^# //')
        ;;

    trafilatura)
        METADATA=$(trafilatura --URL "$URL" --json)
        ARTICLE_TITLE=$(echo "$METADATA" | python3 -c "import json, sys; print(json.load(sys.stdin).get('title', 'Article'))")
        trafilatura --URL "$URL" --output-format txt --no-comments > temp_article.txt
        ;;

    fallback)
        ARTICLE_TITLE=$(curl -s "$URL" | grep -oP '<title>\K[^<]+' | head -n 1)
        ARTICLE_TITLE=${ARTICLE_TITLE%% - *}
        curl -s "$URL" | python3 -c "
from html.parser import HTMLParser
import sys

class ArticleExtractor(HTMLParser):
    def __init__(self):
        super().__init__()
        self.content = []
        self.skip_tags = {'script', 'style', 'nav', 'header', 'footer', 'aside', 'form'}
        self.in_content = False

    def handle_starttag(self, tag, attrs):
        if tag not in self.skip_tags and tag in {'p', 'article', 'main'}:
            self.in_content = True

    def handle_data(self, data):
        if self.in_content and data.strip():
            self.content.append(data.strip())

    def get_content(self):
        return '\n\n'.join(self.content)

parser = ArticleExtractor()
parser.feed(sys.stdin.read())
print(parser.get_content())
" > temp_article.txt
        ;;
esac

# 3. Clean filename
FILENAME=$(echo "$ARTICLE_TITLE" | tr '/' '-' | tr ':' '-' | tr '?' '' | tr '"' '' | cut -c 1-80 | sed 's/ *$//')
CONTENT_FILE="${FILENAME}.txt"
mv temp_article.txt "$CONTENT_FILE"

echo "✓ Saved article: $CONTENT_FILE"
```

#### PDF Document

```bash
# Download and extract PDF
echo "📑 Downloading PDF..."

# 1. Download PDF
PDF_FILENAME=$(basename "$URL")
curl -L -o "$PDF_FILENAME" "$URL"

# 2. Extract text using pdftotext (if available)
if command -v pdftotext &> /dev/null; then
    pdftotext "$PDF_FILENAME" temp_pdf.txt
    CONTENT_FILE="${PDF_FILENAME%.pdf}.txt"
    mv temp_pdf.txt "$CONTENT_FILE"
    echo "✓ Extracted text from PDF: $CONTENT_FILE"

    # Optionally keep PDF
    echo "Keep original PDF? (y/n)"
    read -r KEEP_PDF
    if [[ ! "$KEEP_PDF" =~ ^[Yy]$ ]]; then
        rm "$PDF_FILENAME"
    fi
else
    # No pdftotext available
    echo "⚠️  pdftotext not found. PDF downloaded but not extracted."
    echo "   Install with: brew install poppler"
    CONTENT_FILE="$PDF_FILENAME"
fi
```

### Step 3: Create Ship-Learn-Next Action Plan

**IMPORTANT**: Always create an action plan after extracting content.

```bash
# Read the extracted content
CONTENT_FILE="[from previous step]"

# Invoke ship-learn-next skill logic:
# 1. Read the content file
# 2. Extract core actionable lessons
# 3. Create 5-rep progression plan
# 4. Save as: Ship-Learn-Next Plan - [Quest Title].md

# See ship-learn-next/SKILL.md for full details
```

**Key points for plan creation:**
- Extract actionable lessons (not just summaries)
- Define a specific 4-8 week quest
- Create Rep 1 (shippable this week)
- Design Reps 2-5 (progressive iterations)
- Save plan to markdown file
- Use format: `Ship-Learn-Next Plan - [Brief Quest Title].md`

### Step 4: Present Results

Show user:
```
✅ Tapestry Workflow Complete!

📥 Content Extracted:
   ✓ [Content type]: [Title]
   ✓ Saved to: [filename.txt]
   ✓ [X] words extracted

📋 Action Plan Created:
   ✓ Quest: [Quest title]
   ✓ Saved to: Ship-Learn-Next Plan - [Title].md

🎯 Your Quest: [One-line summary]

📍 Rep 1 (This Week): [Rep 1 goal]

When will you ship Rep 1?
```

## Complete Tapestry Workflow Script

```bash
#!/bin/bash

# Tapestry: Extract content + create action plan
# Usage: tapestry <URL>

URL="$1"

if [ -z "$URL" ]; then
    echo "Usage: tapestry <URL>"
    exit 1
fi

echo "🧵 Tapestry Workflow Starting..."
echo "URL: $URL"
echo ""

# Step 1: Detect content type
if [[ "$URL" =~ youtube\.com/watch || "$URL" =~ youtu\.be/ || "$URL" =~ youtube\.com/shorts ]]; then
    CONTENT_TYPE="youtube"
elif [[ "$URL" =~ \.pdf$ ]] || curl -sI "$URL" | grep -iq "Content-Type: application/pdf"; then
    CONTENT_TYPE="pdf"
else
    CONTENT_TYPE="article"
fi

echo "📍 Detected: $CONTENT_TYPE"
echo ""

# Step 2: Extract content
case $CONTENT_TYPE in
    youtube)
        echo "📺 Extracting YouTube transcript..."
        # [YouTube extraction code from above]
        ;;

    article)
        echo "📄 Extracting article..."
        # [Article extraction code from above]
        ;;

    pdf)
        echo "📑 Downloading PDF..."
        # [PDF extraction code from above]
        ;;
esac

echo ""

# Step 3: Create action plan
echo "🚀 Creating Ship-Learn-Next action plan..."
# [Plan creation using ship-learn-next skill]

echo ""
echo "✅ Tapestry Workflow Complete!"
echo ""
echo "📥 Content: $CONTENT_FILE"
echo "📋 Plan: Ship-Learn-Next Plan - [title].md"
echo ""
echo "🎯 Next: Review your action plan and ship Rep 1!"
```

## Error Handling

### Common Issues:

**1. Unsupported URL type**
- Try article extraction as fallback
- If fails: "Could not extract content from this URL type"

**2. No content extracted**
- Check if URL is accessible
- Try alternate extraction method
- Inform user: "Extraction failed. URL may require authentication."

**3. Tools not installed**
- Auto-install when possible (yt-dlp, reader, trafilatura)
- Provide install instructions if auto-install fails
- Use fallback methods when available

**4. Empty or invalid content**
- Verify file has content before creating plan
- Don't create plan if extraction failed
- Show preview to user before planning

## Best Practices

- ✅ Always show what was detected ("📍 Detected: youtube")
- ✅ Display progress for each step
- ✅ Save both content file AND plan file
- ✅ Show preview of extracted content (first 10 lines)
- ✅ Create plan automatically (don't ask)
- ✅ Present clear summary at end
- ✅ Ask commitment question: "When will you ship Rep 1?"

## Usage Examples

### Example 1: YouTube Video (using "tapestry")

```
User: tapestry https://www.youtube.com/watch?v=dQw4w9WgXcQ

Claude:
🧵 Tapestry Workflow Starting...
📍 Detected: youtube
📺 Extracting YouTube transcript...
✓ Saved transcript: Never Gonna Give You Up.txt

🚀 Creating action plan...
✓ Quest: Master Video Production
✓ Saved plan: Ship-Learn-Next Plan - Master Video Production.md

✅ Complete! When will you ship Rep 1?
```

### Example 2: Article (using "weave")

```
User: weave https://example.com/how-to-build-saas

Claude:
🧵 Tapestry Workflow Starting...
📍 Detected: article
📄 Extracting article...
✓ Using reader (Mozilla Readability)
✓ Saved article: How to Build a SaaS.txt

🚀 Creating action plan...
✓ Quest: Build a SaaS MVP
✓ Saved plan: Ship-Learn-Next Plan - Build a SaaS MVP.md

✅ Complete! When will you ship Rep 1?
```

### Example 3: PDF (using "help me plan")

```
User: help me plan https://example.com/research-paper.pdf

Claude:
🧵 Tapestry Workflow Starting...
📍 Detected: pdf
📑 Downloading PDF...
✓ Downloaded: research-paper.pdf
✓ Extracted text: research-paper.txt

🚀 Creating action plan...
✓ Quest: Apply Research Findings
✓ Saved plan: Ship-Learn-Next Plan - Apply Research Findings.md

✅ Complete! When will you ship Rep 1?
```

## Dependencies

This skill orchestrates the other skills, so requires:

**For YouTube:**
- yt-dlp (auto-installed)
- Python 3 (for deduplication)

**For Articles:**
- reader (npm) OR trafilatura (pip)
- Falls back to basic curl if neither available

**For PDFs:**
- curl (built-in)
- pdftotext (optional - from poppler package)
  - Install: `brew install poppler` (macOS)
  - Install: `apt install poppler-utils` (Linux)

**For Planning:**
- No additional requirements (uses built-in tools)

## Philosophy

**Tapestry weaves learning content into action.**

The unified workflow ensures you never just consume content - you always create an implementation plan. This transforms passive learning into active building.

Extract → Plan → Ship → Learn → Next.

That's the Tapestry way.
