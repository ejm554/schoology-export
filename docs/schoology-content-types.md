# Schoology Content Types Reference

## Overview

The Schoology export tool encounters six distinct content types when scraping course materials:

- **[document](#document)** - Actual files uploaded to Schoology (PDFs, Word docs, images, etc.)
- **[page](#page)** - Text content created within Schoology's editor
- **[link](#link)** - External URLs
- **[assignment](#assignment)** - Assignment pages with instructions and attached materials
- **[embedded_page](#embedded_page)** - Content embedded in Schoology interface (typically iframes)
- **[other](#other)** - Catch-all for unrecognized content (discussion boards, etc.)

## Summary Table

| Type | Offline Access | Handler | Post-Graduation Access |
|------|---------------|---------|----------------------|
| `document` | ✅ Full | Downloads actual file | ✅ Yes |
| `page` | ⚠️ Partial | Saves HTML (text yes, embeds no) | ⚠️ Text only |
| `link` | ❌ None | Creates `.url` shortcut | ⚠️ Depends on URL |
| `assignment` | ❌ None | Creates `.url` shortcut | ❌ No (Schoology access required) |
| `embedded_page` | ⚠️ Partial | Saves HTML with iframe | ⚠️ Depends on embedded content |
| `other` | ❌ None | Creates `.url` shortcut | ⚠️ Depends on URL |

**Key takeaway:** Only `document` type provides reliable long-term offline access. Most other types require internet, and some require Schoology login depending on whether URLs are public or use Schoology wrapper URLs.

---

## document

**What it is:**  
Actual files uploaded to Schoology by instructors - PDFs, Word documents, images, spreadsheets, etc.

**How the tool handles it:**
- Fetches the file via HTTP request using the `href` URL
- Stores the binary blob directly in the ZIP archive
- Preserves original filename and folder structure

**Offline access:** ✅ Yes - fully functional offline

**Code snippet:**
```javascript
const response = await fetch(href);
const blob = await response.blob();
files.push({
    name: `${directory ? directory + '/' : ''}${name}`,
    input: blob
});
```

**Example:** Teacher uploads `Chapter_1_Notes.pdf` to course materials folder

**Manifest entry:**
- Type: `DOCUMENT`
- Access Note: `Offline`

---

## page

**What it is:**  
Text content created using Schoology's built-in page editor. May contain formatted text, images, and embedded content (like Google Slides iframes).

**How the tool handles it:**
- Extracts HTML content from `.s-page-content` div in Schoology DOM
- Saves as `.html` file in ZIP
- If page has no content in that div, skips it with console warning (your null check fix)

**Offline access:** ⚠️ Partial
- Text and images: Yes
- Embedded content (iframes): Requires internet

**Code snippet:**
```javascript
if (!content || !material.content) {
    console.warn(`Page "${name}" has no content, skipping.`);
    continue;
}
files.push({
    name: `${directory ? directory + '/' : ''}${name}.html`,
    input: new Blob([content], { type: 'text/html' })
});
```

**Example:** Lesson overview page with text instructions and embedded Google Slides presentation

**Manifest entry:**
- Type: `PAGE`
- Access Note: `Offline`

---

## link

**What it is:**  
External URLs added to course materials - could be websites, Google Docs, YouTube videos, etc.

**How the tool handles it:**
- Creates Windows `.url` shortcut file
- File contains the `href` from Schoology (often a Schoology wrapper URL like `schoology.isd623.org/attachment/...`)
- Double-clicking the `.url` file opens the URL in your browser

**Offline access:** ❌ No
- Requires internet connection
- **Post-graduation access:** Depends on URL type
  - Public URLs (YouTube, news sites, etc.): ✅ Continue to work
  - Schoology wrapper URLs (`schoology.isd623.org/attachment/...`): ❌ Stop working when access is revoked
  - Google Docs/Slides via Schoology wrapper: ❌ May lose access

**Code snippet:**
```javascript
const urlFileContent = `[InternetShortcut]\nURL=${href}`;
files.push({
    name: `${directory ? directory + '/' : ''}${name}.url`,
    input: new Blob([urlFileContent], { type: 'text/plain' })
});
```

**Example:** Link to external article or Google Doc

**Manifest entry:**
- Type: `LINK`
- Access Note: `May require Schoology access`

---

## assignment

**What it is:**  
Assignment pages containing instructions, descriptions, due dates, and often attached files (PDFs, worksheets, rubrics).

**How the tool handles it:**
- Creates `.url` shortcut file prefixed with `[ASSIGNMENT]`
- Links to the Schoology assignment page
- **Does NOT fetch the assignment page content or download attached files**

**Offline access:** ❌ No
- Requires Schoology login to view assignment page
- Attached PDFs not automatically downloaded

**The problem:**  
This is the biggest gap in the export tool. Your manual testing found 89% of assignments had downloadable PDFs, but the scraper only saves a link to the assignment page, not the attachments themselves.

**Your workaround:**  
Use URL opener tool to batch-open assignment URLs, manually download PDFs, manually save to archive.

**Code snippet:**
```javascript
const urlFileContent = `[InternetShortcut]\nURL=${href}`;
files.push({
    name: `${directory ? directory + '/' : ''}[ASSIGNMENT] ${name}.url`,
    input: new Blob([urlFileContent], { type: 'text/plain' })
});
```

**Example:** "Unit 1 Test Review" assignment with attached PDF study guide

**Manifest entry:**
- Type: `ASSIGNMENT`
- Access Note: `May require Schoology access`

---

## embedded_page

**What it is:**  
Content embedded directly in the Schoology interface, typically using iframes. Common for Google Slides, videos, or interactive content.

**How the tool handles it:**
- Saves raw HTML including iframe tags
- HTML file can be opened offline but iframe content requires internet
- Similar to `page` type but specifically for embedded content

**Offline access:** ⚠️ Partial
- HTML structure: Yes
- Iframe content: Requires internet
- **Post-graduation access:** Depends on embedded content
  - Public embeds (YouTube, etc.): ✅ Continue to work
  - Schoology-wrapped content: ❌ May lose access

**Code snippet:**
```javascript
files.push({
    name: `${directory ? directory + '/' : ''}${name}.html`,
    input: new Blob([content], { type: 'text/html' })
});
```

**Example:** Embedded Google Slides presentation viewer

**Manifest entry:**
- Type: `EMBEDDED_PAGE`
- Access Note: `May require Schoology access`

---

## other

**What it is:**  
Catch-all category for content that doesn't match the other types. Commonly includes discussion boards, quizzes, or unrecognized content types.

**How the tool handles it:**
- Creates `.url` shortcut file prefixed with `[OTHER]`
- Same behavior as `link` type
- Fallback handler for unknown content

**Offline access:** ❌ No
- Requires Schoology login for most content types in this category
- Discussion content not scraped
- **Post-graduation access:** Varies by content
  - Discussion boards: ❌ Require Schoology access
  - External links: ⚠️ Depends on whether URL is public or Schoology wrapper

**Code snippet:**
```javascript
const urlFileContent = `[InternetShortcut]\nURL=${href}`;
files.push({
    name: `${directory ? directory + '/' : ''}[OTHER] ${name}.url`,
    input: new Blob([urlFileContent], { type: 'text/plain' })
});
```

**Example:** Discussion board thread, quiz, poll

**Manifest entry:**
- Type: `OTHER`
- Access Note: `May require Schoology access`

