---
name: golf-course-guide
description: |
  Creates a beautiful hole-by-hole golf course strategy guide as an HTML file with one-click PDF export from a golf course website URL. Use this skill immediately when: the user provides any golf course URL, asks to create a 골프 공략 가이드 / 홀별 공략법 / 코스 가이드, wants to generate a scorecard document, or mentions 골프장 공략 / PDF 출력 / 코스별 공략. Don't wait for explicit "skill" keywords — if someone pastes a golf course URL alongside words like "공략", "가이드", "만들어줘", trigger this skill right away. Also trigger when the user wants to document an already-played course or prepare for an upcoming round with a strategy guide.
---

# Golf Course Strategy Guide Generator

골프장 URL을 분석하여 전 코스 홀별 공략 가이드를 자동 생성하고, 브라우저 인쇄로 PDF 저장이 가능한 HTML 파일을 만드는 스킬.

## Overview

Output: A single self-contained HTML file (e.g. `{course-name}-guide.html`) that:
- Displays all 18 holes with course map images, distances by tee, and strategy text
- Has tab navigation (전체 / OUT / IN / 스코어카드)
- Has a "PDF 저장" button (browser print → Save as PDF)
- Is print-optimized: 1 hole per A4 page

## Step 1: Research the Golf Course

Read `references/scraping-guide.md` first for website structure patterns and image URL conventions.

### 1a. Fetch the main URL

Use WebFetch on the provided URL. Look for:
- Course name (골프장/클럽 이름)
- Number of courses and holes
- Links to individual course/hole pages
- Navigation menus pointing to course info, hole guides, scorecard

### 1b. Find hole data pages

Korean golf course sites commonly structure hole info at:
- `/course/hole01` ~ `/hole18`
- `/golf/info/course?no=1` ~ `?no=18`
- A single scorecard/거리표 page with a full table

Fetch 2–3 hole pages to confirm the pattern, then extrapolate for all 18.

### 1c. Extract per-hole data

For each of the 18 holes, collect:
```
n       — hole number (1–18)
course  — 'OUT' (holes 1–9) or 'IN' (holes 10–18)
seq     — sequence within course (1–9)
par     — par value (3, 4, or 5)
hdcp    — handicap ranking (1–18)
d       — distances by tee: { bk, gd, wh, sv, rd } (all in meters)
imgUrl  — course map image URL (see image URL patterns in scraping-guide.md)
```

**Distance units**: Korean courses use meters. If a site shows yards, convert: yards × 0.9144.

**Missing tees**: If some tee colors are absent, omit them from `d`. The template handles variable tee counts.

**Image discovery**: Find one hole's image URL then infer the pattern for all holes. Test the URL before assuming it works — try fetching it. If images are not accessible (403, CORS), set `imgUrl: ''`; the template shows a placeholder gracefully.

### 1d. Collect course metadata

- Full club name (e.g. "라비에벨 골프 앤 리조트")
- Course name (e.g. "듄스코스")
- Phone number
- Address
- Total par (usually 72 = 36+36)
- White tee total distance

## Step 2: Generate Strategy Content

For each hole, write two short paragraphs in Korean:

**strategy** (코스 공략, 3–4 sentences):
- Tee shot target and key hazards to avoid
- Recommended landing zone or angle
- Second shot positioning consideration
- Club selection hint if relevant

**green** (그린 공략, 3–4 sentences):
- Approach direction and recommended landing spot
- Green slope/break direction
- Bunker or hazard positions around green
- Putting line advice

Base the text on:
- The hole's par, HDCP, and distances
- Any visual or descriptive information found on the website (hole descriptions, tips, photos)
- General golf knowledge about typical hole designs for that par/distance

For low HDCP holes (1–5): emphasize conservative, risk-averse play.
For high HDCP holes (14–18): note birdie opportunities, slightly more aggressive.

Keep each paragraph concise and actionable — 60–100 characters per sentence.

## Step 3: Build the HTML File

### 3a. Read the template

Read `assets/guide-template.html` — this is the complete styled template. You will replace the placeholders.

### 3b. Fill in placeholders

| Placeholder | Value |
|---|---|
| `{{COURSE_NAME}}` | Club name (e.g. 라비에벨 CC) |
| `{{COURSE_SUBTITLE}}` | Course + subtitle (e.g. 듄스코스 홀별 공략법) |
| `{{COURSE_STATS_CHIPS}}` | HTML chips: 18홀, 파72, White XXXXm, phone |
| `{{TOOLBAR_TITLE}}` | e.g. ⚪ 화이트 티 기준 · 18홀 공략 가이드 |
| `{{HOLES_JSON}}` | JSON array of all 18 hole objects |
| `{{FOOTER_TEXT}}` | Club name · Address · Phone · disclaimer |

### 3c. HOLES_JSON format

```javascript
[
  {
    "n": 1,
    "course": "OUT",
    "seq": 1,
    "par": 4,
    "hdcp": 17,
    "imgUrl": "https://...",
    "d": { "bk": 339, "gd": 324, "wh": 301, "sv": 272, "rd": 240 },
    "strategy": "V자 형태의 페어웨이...",
    "green": "그린이 왼쪽으로 경사져..."
  },
  ...
]
```

### 3d. Write the output file

Save as `{slug}-course-guide.html` in the current working directory.
- slug: lowercase, hyphens, no spaces (e.g. `lavieestbelle-dunes`, `sky72-ocean`)

## Step 4: Report to User

Tell the user:
1. The output file name and path
2. How to open and use it (open in browser, click PDF 저장)
3. A brief summary: "18홀 / 파XX / White XXXXm" stats
4. Note any holes where data was missing or estimated

**Example response:**
> 라비에벨 CC 듄스코스 공략 가이드를 생성했습니다.
> 📄 파일: `lavieestbelle-dunes-course-guide.html`
> 브라우저에서 열고 우측 상단 **PDF 저장** 버튼을 클릭하면 A4 PDF로 저장됩니다. (홀당 1페이지)
> 18홀 · 파72 · White 5,814m

## Troubleshooting

**Site blocks WebFetch / returns 403:**
Ask the user: "사이트가 자동 접근을 차단하고 있습니다. 각 홀의 거리표를 복사해서 붙여넣어 주시면 가이드를 생성할 수 있습니다."

**Images not loading in output HTML:**
If fetched URLs return 403 in browser (hotlink protection), add a note in the footer: "코스 이미지는 골프장 홈페이지에서 직접 확인하세요." and set all imgUrl to ''.

**Partial data (some holes missing):**
Generate strategy text anyway. Mark missing distances as 0 or omit the tee key. Add a note at the top of the file.

**Non-standard hole count (9-hole course, 27-hole, 36-hole):**
- 9홀: Use only OUT, no IN tab
- 27/36홀: Ask which 18 holes (or which course) to generate the guide for. Generate per course.

## Reference Files

- `assets/guide-template.html` — Complete HTML/CSS/JS template with all placeholders
- `references/scraping-guide.md` — Website structure patterns, image URL patterns, distance table parsing
