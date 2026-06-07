# Korean Golf Course Website Scraping Guide

## Common Site Structures

### Type A: Per-hole page (most common)
- Main course page lists all holes or links to individual hole pages
- Each hole page has distance table, par, HDCP, and course image
- URL pattern examples:
  - `/course/hole1`, `/course/hole/01`
  - `/golf/course?hole=1`
  - `/dunes/hole01.asp`

### Type B: Single scorecard page
- One page with a full table of all 18 holes
- Distance data in a grid/table format
- Images may be separate CDN paths

### Type C: JavaScript SPA
- Data embedded in JS variables or `<script>` tags
- Look for JSON arrays with hole data
- May need to inspect the page source for data objects

## Image URL Patterns (Korean golf courses)

Most Korean golf course sites follow predictable image naming:

```
Pattern 1: /image/course/{direction}/con{NN}.png
  - direction: one (OUT), two (IN)
  - NN: zero-padded hole sequence (01-09)
  - Example: /dunescourse/image/course/one/con01.png

Pattern 2: /upload/course/hole{N}.jpg
  - N: hole number 1-18
  - Example: /upload/course/hole1.jpg

Pattern 3: /img/golf/{courseName}/hole_{NN}.png
  - courseName: abbreviated course name
  - Example: /img/golf/dunes/hole_01.png

Pattern 4: /content/course/images/h{NN}.jpg
```

To discover the pattern:
1. Fetch the course detail page for hole 1
2. Look for `<img>` tags with src containing "hole", "con", "course", "map"
3. Try to infer the pattern from hole 1 URL and extrapolate for holes 2-18

## Distance Table Parsing

Korean golf courses typically show distances in a table with columns:
- 블랙/BLACK (longest, championship tee)
- 골드/GOLD
- 화이트/WHITE (standard, most common reference)
- 실버/SILVER
- 레드/RED (shortest, ladies' tee)

Some courses use different color names:
- 챔피언/Champion instead of Black
- 블루/Blue instead of Silver
- 레이디스/Ladies instead of Red

Map extracted labels to the standard keys: bk, gd, wh, sv, rd

## HDCP (Handicap) Extraction

HDCP is usually shown as a number 1-18 in a row labeled "핸디캡", "HDCP", or "H/C".
It represents hole difficulty ranking (1 = hardest, 18 = easiest).

## Course Name and Contact Info

Look for:
- `<h1>` or `<h2>` tags near the top with the club name
- Phone numbers in format: 0XX-XXX-XXXX
- Address in the footer or contact section

## Fallback: When Data Cannot Be Scraped

If the website blocks scraping or has complex JS rendering:
1. Ask the user to provide the scorecard data manually or as a screenshot
2. Use any partial data found (course name, hole count, par totals)
3. Generate placeholder distances if needed, clearly marking them as estimates

## Strategy Text Generation Guidelines

When generating hole strategy text, consider these factors:

**Tee Shot Strategy** (티샷 공략):
- Par 3: Aim for green center, note hazards flanking green
- Par 4: Identify landing zone, avoid OB/water on dominant side
- Par 5: First shot for position, not max distance

**Green Strategy** (그린 공략):
- Slope direction (물이 흐르는 방향)
- Front/back pin position implications
- Bunker positions around green
- Grain direction if known

**Factors from hole stats**:
- Low HDCP (1-6): High difficulty, emphasize conservative play
- High HDCP (13-18): Birdie opportunity, more aggressive line
- Par 3 with distance > 180m: Extra club, wind factor
- Par 5 with distance < 480m: Eagle opportunity for long hitters
