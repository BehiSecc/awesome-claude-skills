---
name: article-extractor
description: Extract clean article content from URLs (blog posts, articles, tutorials) and save as readable text. Use when user wants to download, extract, or save an article/blog post from a URL without ads, navigation, or clutter.
allowed-tools: Bash,Write
---

# Article Extractor

This skill extracts the main content from web articles and blog posts, removing navigation, ads, newsletter signups, and other clutter. Saves clean, readable text.

## When to Use This Skill

Activate when the user:
- Provides an article/blog URL and wants the text content
- Asks to "download this article"
- Wants to "extract the content from [URL]"
- Asks to "save this blog post as text"
- Needs clean article text without distractions

## How It Works

### Priority Order:
1. **Check if tools are installed** (reader or trafilatura)
2. **Download and extract article** using best available tool
3. **Clean up the content** (remove extra whitespace, format properly)
4. **Save to file** with article title as filename
5. **Confirm location** and show preview

## Installation Check

Check for article extraction tools in this order:

### Option 1: reader (Recommended - Mozilla's Readability)

```bash
command -v reader
```

If not installed:
```bash
npm install -g @mozilla/readability-cli
# or
npm install -g reader-cli
```

### Option 2: trafilatura (Python-based, very good)

```bash
command -v trafilatura
```

If not installed:
```bash
pip3 install trafilatura
```

### Option 3: Fallback (curl + simple parsing)

If no tools available, use basic curl + text extraction (less reliable but works)

## Extraction Methods

### Method 1: Using reader (Best for most articles)

```bash
# Extract article
reader "URL" > article.txt
```

**Pros:**
- Based on Mozilla's Readability algorithm
- Excellent at removing clutter
- Preserves article structure

### Method 2: Using trafilatura (Best for blogs/news)

```bash
# Extract article
trafilatura --URL "URL" --output-format txt > article.txt

# Or with more options
trafilatura --URL "URL" --output-format txt --no-comments --no-tables > article.txt
```

**Pros:**
- Very accurate extraction
- Good with various site structures
- Handles multiple languages

**Options:**
- `--no-comments`: Skip comment sections
- `--no-tables`: Skip data tables
- `--precision`: Favor precision over recall
- `--recall`: Extract more content (may include some noise)

### Method 3: Fallback (curl + basic parsing)

```bash
# Download and extract basic content
curl -s "URL" | python3 -c "
from html.parser import HTMLParser
import sys

class ArticleExtractor(HTMLParser):
    def __init__(self):
        super().__init__()
        self.in_content = False
        self.content = []
        self.skip_tags = {'script', 'style', 'nav', 'header', 'footer', 'aside'}
        self.current_tag = None

    def handle_starttag(self, tag, attrs):
        if tag not in self.skip_tags:
            if tag in {'p', 'article', 'main', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6'}:
                self.in_content = True
        self.current_tag = tag

    def handle_data(self, data):
        if self.in_content and data.strip():
            self.content.append(data.strip())

    def get_content(self):
        return '\n\n'.join(self.content)

parser = ArticleExtractor()
parser.feed(sys.stdin.read())
print(parser.get_content())
" > article.txt
```

**Note:** This is less reliable but works without dependencies.

## Getting Article Title

Extract title for filename:

### Using reader:
```bash
# reader outputs markdown with title at top
TITLE=$(reader "URL" | head -n 1 | sed 's/^# //')
```

### Using trafilatura:
```bash
# Get metadata including title
TITLE=$(trafilatura --URL "URL" --json | python3 -c "import json, sys; print(json.load(sys.stdin)['title'])")
```

### Using curl (fallback):
```bash
TITLE=$(curl -s "URL" | grep -oP '<title>\K[^<]+' | sed 's/ - .*//' | sed 's/ | .*//')
```

## Filename Creation

Clean title for filesystem:

```bash
# Get title
TITLE="Article Title from Website"

# Clean for filesystem (remove special chars, limit length)
FILENAME=$(echo "$TITLE" | tr '/' '-' | tr ':' '-' | tr '?' '' | tr '"' '' | tr '<' '' | tr '>' '' | tr '|' '-' | cut -c 1-100 | sed 's/ *$//')

# Add extension
FILENAME="${FILENAME}.txt"
```

## Complete Workflow

```bash
ARTICLE_URL="https://example.com/article"

# Check for tools
if command -v reader &> /dev/null; then
    TOOL="reader"
    echo "Using reader (Mozilla Readability)"
elif command -v trafilatura &> /dev/null; then
    TOOL="trafilatura"
    echo "Using trafilatura"
else
    TOOL="fallback"
    echo "Using fallback method (may be less accurate)"
fi

# Extract article
case $TOOL in
    reader)
        # Get content
        reader "$ARTICLE_URL" > temp_article.txt

        # Get title (first line after # in markdown)
        TITLE=$(head -n 1 temp_article.txt | sed 's/^# //')
        ;;

    trafilatura)
        # Get title from metadata
        METADATA=$(trafilatura --URL "$ARTICLE_URL" --json)
        TITLE=$(echo "$METADATA" | python3 -c "import json, sys; print(json.load(sys.stdin).get('title', 'Article'))")

        # Get clean content
        trafilatura --URL "$ARTICLE_URL" --output-format txt --no-comments > temp_article.txt
        ;;

    fallback)
        # Get title
        TITLE=$(curl -s "$ARTICLE_URL" | grep -oP '<title>\K[^<]+' | head -n 1)
        TITLE=${TITLE%% - *}  # Remove site name
        TITLE=${TITLE%% | *}  # Remove site name (alternate)

        # Get content (basic extraction)
        curl -s "$ARTICLE_URL" | python3 -c "
from html.parser import HTMLParser
import sys

class ArticleExtractor(HTMLParser):
    def __init__(self):
        super().__init__()
        self.in_content = False
        self.content = []
        self.skip_tags = {'script', 'style', 'nav', 'header', 'footer', 'aside', 'form'}

    def handle_starttag(self, tag, attrs):
        if tag not in self.skip_tags:
            if tag in {'p', 'article', 'main'}:
                self.in_content = True
        if tag in {'h1', 'h2', 'h3'}:
            self.content.append('\n')

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

# Clean filename
FILENAME=$(echo "$TITLE" | tr '/' '-' | tr ':' '-' | tr '?' '' | tr '"' '' | tr '<>' '' | tr '|' '-' | cut -c 1-80 | sed 's/ *$//' | sed 's/^ *//')
FILENAME="${FILENAME}.txt"

# Move to final filename
mv temp_article.txt "$FILENAME"

# Show result
echo "✓ Extracted article: $TITLE"
echo "✓ Saved to: $FILENAME"
echo ""
echo "Preview (first 10 lines):"
head -n 10 "$FILENAME"
```

## Error Handling

### Common Issues

**1. Tool not installed**
- Try alternate tool (reader → trafilatura → fallback)
- Offer to install: "Install reader with: npm install -g reader-cli"

**2. Paywall or login required**
- Extraction tools may fail
- Inform user: "This article requires authentication. Cannot extract."

**3. Invalid URL**
- Check URL format
- Try with and without redirects

**4. No content extracted**
- Site may use heavy JavaScript
- Try fallback method
- Inform user if extraction fails

**5. Special characters in title**
- Clean title for filesystem
- Remove: `/`, `:`, `?`, `"`, `<`, `>`, `|`
- Replace with `-` or remove

## Output Format

### Saved File Contains:
- Article title (if available)
- Author (if available from tool)
- Main article text
- Section headings
- No navigation, ads, or clutter

### What Gets Removed:
- Navigation menus
- Ads and promotional content
- Newsletter signup forms
- Related articles sidebars
- Comment sections (optional)
- Social media buttons
- Cookie notices

## Tips for Best Results

**1. Use reader for most articles**
- Best all-around tool
- Based on Firefox Reader View
- Works on most news sites and blogs

**2. Use trafilatura for:**
- Academic articles
- News sites
- Blogs with complex layouts
- Non-English content

**3. Fallback method limitations:**
- May include some noise
- Less accurate paragraph detection
- Better than nothing for simple sites

**4. Check extraction quality:**
- Always show preview to user
- Ask if it looks correct
- Offer to try different tool if needed

## Example Usage

**Simple extraction:**
```bash
# User: "Extract https://example.com/article"
reader "https://example.com/article" > temp.txt
TITLE=$(head -n 1 temp.txt | sed 's/^# //')
FILENAME="$(echo "$TITLE" | tr '/' '-').txt"
mv temp.txt "$FILENAME"
echo "✓ Saved to: $FILENAME"
```

**With error handling:**
```bash
if ! reader "$URL" > temp.txt 2>/dev/null; then
    if command -v trafilatura &> /dev/null; then
        trafilatura --URL "$URL" --output-format txt > temp.txt
    else
        echo "Error: Could not extract article. Install reader or trafilatura."
        exit 1
    fi
fi
```

## Best Practices

- ✅ Always show preview after extraction (first 10 lines)
- ✅ Verify extraction succeeded before saving
- ✅ Clean filename for filesystem compatibility
- ✅ Try fallback method if primary fails
- ✅ Inform user which tool was used
- ✅ Keep filename length reasonable (< 100 chars)

## After Extraction

Display to user:
1. "✓ Extracted: [Article Title]"
2. "✓ Saved to: [filename]"
3. Show preview (first 10-15 lines)
4. File size and location

Ask if needed:
- "Would you like me to also create a Ship-Learn-Next plan from this?" (if using ship-learn-next skill)
- "Should I extract another article?"
