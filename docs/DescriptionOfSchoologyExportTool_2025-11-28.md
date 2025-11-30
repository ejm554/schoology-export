---
document-purpose: Comprehensive overview of schoology-export tool
date-created: 2025-11-28
authors:
  human-author: EJ Makela
  ai-author: Claude via Continue.dev extension for Visual Studio Code
---

# Schoology Export Tool - Comprehensive Overview

## Project Purpose and Functionality

This tool automates collecting resources, assignments, and files from Schoology courses. It allows users to save an entire course's content as an organized ZIP file for offline access, backup, or migration.

## Key Features

- Exports all files, folders, and resources from Schoology courses
- Maintains the original organizational structure
- Runs entirely in-browser without server dependencies
- Simple UI with three buttons (Open All Folders, Scrape, Export)
- Rate-limited requests to prevent being blocked

## Technical Implementation

### Deployment Options
1. **Browser Extension**: Packaged as a Manifest V3 browser extension
2. **Bookmarklet**: Can be used as a simple bookmarklet (one-click install)
3. **Manual Paste**: Code can be pasted directly into browser console

### Project File Structure

```
schoology-export/
├─ images/
│  ├─ bookmarklet.gif
│  └─ ui.png
├─ src/
│  ├─ dist/
│  │  ├─ master-fork.min.js
│  │  ├─ master.js
│  │  └─ master.min.js
│  ├─ export.js
│  ├─ inject.js
│  ├─ navigation.js
│  ├─ scraper.js
│  └─ util.js
├─ z-private-notes/
├─ .gitignore
├─ manifest.json
├─ package-lock.json
├─ package.json
└─ README.md
```

### Core Components Architecture

1. **Main Coordinator** (`master.js`):
   - Entry point that loads and coordinates all modules
   - Sets up message passing between components
   - Manages the overall export workflow

2. **UI Injection** (`inject.js`):
   - Creates three buttons in the Schoology course page:
     - Open All Folders: Expands all course folders
     - Scrape Course Materials: Collects all content data
     - Export Course Materials: Downloads as ZIP
   - Handles button states, styles, and click events

3. **Navigation Handler** (`navigation.js`):
   - Recursively opens all folders in the course
   - Ensures all content is visible and accessible for scraping

4. **Content Scraper** (`scraper.js`):
   - Traverses the DOM to extract course content
   - Handles multiple content types:
     - Documents (PDFs, images)
     - Pages with embedded content
     - External links
     - Folders and subfolders
   - Preserves metadata like filenames and structure

5. **Export Engine** (`export.js`):
   - Creates a structured ZIP with all content
   - Processes different file types appropriately:
     - Saves documents with proper extensions
     - Converts links to .url files
     - Preserves HTML content with embedded resources
   - Uses client-zip library for browser-based ZIP creation

6. **Utilities** (`util.js`):
   - Implements rate limiting for requests
   - Provides HTML parsing and extraction helpers
   - Handles network requests with proper error handling

### Data Flow

1. User loads a Schoology course page and activates the tool
2. Tool injects UI into the course page
3. When "Open All Folders" is clicked, all folders are recursively expanded
4. "Scrape" collects all visible content and stores it in session storage
5. "Export" processes the stored data, downloads files, and packages everything as a ZIP
6. User receives a complete, organized backup of the course

## Technical Considerations

- **Browser Compatibility**: Works with modern browsers supporting ES modules and Streams API
- **Performance**: Uses asynchronous processing to handle large courses
- **Rate Limiting**: Implements request throttling to avoid Schoology rate limits
- **Memory Management**: Streams data rather than loading everything in memory
- **Error Handling**: Provides graceful fallbacks if certain content can't be accessed
