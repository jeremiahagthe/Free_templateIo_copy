# 🔗 n8n Integration Guide

Complete guide for using the Carousel Generator API as an HTTP Request node in n8n workflows.

## 📋 Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [HTTP Request Node Configuration](#http-request-node-configuration)
4. [Request Parameters](#request-parameters)
5. [Response Format](#response-format)
6. [Common Workflows](#common-workflows)
7. [Error Handling](#error-handling)
8. [Converting Base64 to Binary](#converting-base64-to-binary)
9. [Tips & Best Practices](#tips--best-practices)
10. [Troubleshooting](#troubleshooting)

---

## Overview

The Carousel Generator API is a serverless endpoint that generates social media carousel images with customizable text overlays. It's perfect for automating social media content creation in n8n workflows.

**API Endpoint:** `POST https://your-app.netlify.app/.netlify/functions/carousel`

**Key Features:**
- ✅ Generates multiple carousel slides in parallel (<500ms for 7 slides)
- ✅ Returns base64-encoded images ready for use
- ✅ Optional Google Drive upload integration
- ✅ Supports custom dimensions for different platforms
- ✅ Rate limited (10 requests/minute per IP)

---

## Prerequisites

1. **Deployed Carousel Generator**
   - Deploy to Netlify following the [Deployment Guide](./DEPLOYMENT.md)
   - Note your Netlify URL (e.g., `https://your-app.netlify.app`)

2. **n8n Instance**
   - Self-hosted n8n or n8n Cloud account
   - Access to create workflows

3. **Background Images**
   - Publicly accessible image URLs
   - Google Drive images: Use `https://drive.google.com/uc?id=FILE_ID` format
   - Unsplash, Pexels, or other image services work great

---

## HTTP Request Node Configuration

### Basic Setup

1. **Add HTTP Request Node** to your n8n workflow
2. **Configure the node:**

| Setting | Value |
|---------|-------|
| **Method** | `POST` |
| **URL** | `https://your-app.netlify.app/.netlify/functions/carousel` |
| **Authentication** | `None` |
| **Send Body** | `Yes` |
| **Body Content Type** | `JSON` |
| **Specify Body** | `Using JSON` |

3. **Request Body (JSON):**

```json
{
  "backgrounds": [
    "https://picsum.photos/1080/1080",
    "https://images.unsplash.com/photo-xxx"
  ],
  "slides": [
    {
      "title": "Welcome to AI Marketing",
      "subtitle": "Transform your social media with automated carousel posts",
      "textColor": "#FFFFFF"
    },
    {
      "title": "10x Your Engagement",
      "subtitle": "Generate stunning visuals in seconds, not hours",
      "textColor": "#FFFFFF"
    }
  ],
  "width": 1080,
  "height": 1080
}
```

### Advanced Configuration

**Options Tab:**
- **Timeout:** `15000` (15 seconds recommended)
- **Follow Redirect:** `Yes`
- **Ignore SSL Issues:** `No` (unless testing)

**Headers Tab:**
- Not required for basic usage
- Content-Type is automatically set to `application/json`

---

## Request Parameters

### Required Parameters

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `backgrounds` | `string[]` | Array of publicly accessible image URLs. Must match number of slides. | `["https://picsum.photos/1080/1080"]` |
| `slides` | `object[]` | Array of slide objects with title and subtitle. Must match number of backgrounds. | See below |

### Slide Object Structure

Each slide object in the `slides` array:

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `title` | `string` | ✅ | - | Main heading text |
| `subtitle` | `string` | ✅ | - | Supporting text below title |
| `textColor` | `string` | ❌ | `#FFFFFF` | Hex color code for text |
| `fontFamily` | `string` | ❌ | `Arial` | Font name (system fonts only) |
| `titleSize` | `number` | ❌ | Auto-calculated | Title font size in pixels |
| `subtitleSize` | `number` | ❌ | Auto-calculated | Subtitle font size in pixels |
| `titleX` | `number` | ❌ | Center | Title X position (horizontal) |
| `titleY` | `number` | ❌ | Center | Title Y position (vertical) |
| `subtitleX` | `number` | ❌ | Center | Subtitle X position (horizontal) |
| `subtitleY` | `number` | ❌ | Center | Subtitle Y position (vertical) |
| `maxTitleWidth` | `number` | ❌ | Width - 10% | Maximum title width in pixels (prevents overflow) |
| `maxSubtitleWidth` | `number` | ❌ | Width - 10% | Maximum subtitle width in pixels (prevents overflow) |
| `textAlign` | `string` | ❌ | `center` | Text alignment: `left`, `center`, or `right` |

**Supported Fonts:**
- System fonts: `Arial`, `Helvetica`, `Impact`, `Futura`, `Georgia`, `Times`
- Google Fonts: `Roboto`, `Open Sans`, `Montserrat`, `Bebas Neue` (fallback to Arial in serverless)

**Font Recommendations for Black & White Images:**
- **Impact** - Bold, high contrast, dramatic (best for emotional/moody images)
- **Futura** - Clean, modern, readable
- **Helvetica** - Professional, safe choice
- **Georgia** - Elegant serif, good for emotional content
- **Arial Black** - Very bold, maximum contrast

### Optional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `width` | `number` | `1080` | Image width in pixels (200-4000) |
| `height` | `number` | `1080` | Image height in pixels (200-4000) |
| `uploadToDrive` | `boolean` | `false` | Enable Google Drive upload |
| `driveToken` | `string` | - | Google OAuth access token |
| `driveFolderId` | `string` | - | Google Drive folder ID for uploads |
| `returnUrls` | `boolean` | `false` | Return Drive URLs instead of base64 (reduces payload size, prevents 413 errors) |

### Platform-Specific Dimensions

| Platform | Width | Height | Aspect Ratio |
|----------|-------|--------|--------------|
| Instagram Feed | 1080 | 1080 | 1:1 (Square) |
| Instagram Stories | 1080 | 1920 | 9:16 (Vertical) |
| Facebook Post | 1080 | 1080 | 1:1 (Square) |
| TikTok | 1080 | 1920 | 9:16 (Vertical) |
| LinkedIn | 1200 | 627 | 1.91:1 (Horizontal) |
| Twitter/X | 1200 | 675 | 16:9 (Horizontal) |
| Pinterest | 1000 | 1500 | 2:3 (Vertical) |

---

## Response Format

### Success Response

```json
{
  "success": true,
  "images": [
    {
      "base64": "data:image/png;base64,iVBORw0KGgo...",
      "filename": "carousel-slide-1.png",
      "success": true
    },
    {
      "base64": "data:image/png;base64,iVBORw0KGgo...",
      "filename": "carousel-slide-2.png",
      "success": true
    }
  ],
  "stats": {
    "totalSlides": 2,
    "successful": 2,
    "failed": 0,
    "generationTimeMs": 347,
    "dimensions": {
      "width": 1080,
      "height": 1080
    }
  }
}
```

### Response with Google Drive Uploads

If `uploadToDrive: true`, response includes:

```json
{
  "success": true,
  "images": [...],
  "driveUrls": [
    {
      "success": true,
      "fileId": "abc123xyz",
      "webViewLink": "https://drive.google.com/file/d/abc123xyz/view",
      "webContentLink": "https://drive.google.com/uc?id=abc123xyz"
    }
  ],
  "stats": {...}
}
```

### Partial Failure Response

If some slides fail to generate:

```json
{
  "success": true,
  "images": [
    {
      "base64": "data:image/png;base64,...",
      "filename": "carousel-slide-1.png",
      "success": true
    }
  ],
  "failed": [
    {
      "filename": "carousel-slide-2.png",
      "error": "Failed to download image: HTTP 404",
      "success": false
    }
  ],
  "stats": {
    "totalSlides": 2,
    "successful": 1,
    "failed": 1,
    ...
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": "Error message description"
}
```

**HTTP Status Codes:**
- `200` - Success
- `400` - Invalid request (missing fields, validation errors)
- `429` - Rate limit exceeded (10 requests/minute)
- `500` - Internal server error

---

## Common Workflows

### Workflow 1: Scheduled Instagram Carousel Posts

**Trigger:** Schedule (Cron) - Every Monday at 9am

**Nodes:**
1. **Schedule Trigger** - Weekly trigger
2. **Code Node** - Prepare carousel data
3. **HTTP Request Node** - Generate carousel
4. **Function Node** - Convert base64 to binary
5. **Instagram Node** - Post carousel

**Code Node Example:**
```javascript
// Prepare carousel data from previous nodes or hardcoded
const backgrounds = [
  'https://picsum.photos/id/1015/1080/1080',
  'https://picsum.photos/id/1018/1080/1080',
  'https://picsum.photos/id/1025/1080/1080'
];

const slides = [
  { title: 'Monday Motivation', subtitle: 'Start your week strong with these tips', textColor: '#FFFFFF' },
  { title: 'Tip #1', subtitle: 'Set clear goals for the week ahead', textColor: '#FFFFFF' },
  { title: 'Tip #2', subtitle: 'Plan your most important tasks first', textColor: '#FFFFFF' }
];

return [{
  json: {
    backgrounds,
    slides,
    width: 1080,
    height: 1080
  }
}];
```

### Workflow 2: Google Drive → Carousel → Instagram

**Trigger:** Webhook or Manual

**Nodes:**
1. **Google Drive Node** - List files from folder
2. **HTTP Request Node** - Generate carousel (with Drive URLs as backgrounds)
3. **Function Node** - Convert base64 to binary
4. **Instagram Node** - Post carousel

**HTTP Request Node Body (using expressions):**
```json
{
  "backgrounds": "{{ $json.files.map(f => 'https://drive.google.com/uc?id=' + f.id) }}",
  "slides": [
    {
      "title": "{{ $json.files[0].name }}",
      "subtitle": "Generated carousel from Drive",
      "textColor": "#FFFFFF"
    }
  ],
  "width": 1080,
  "height": 1080
}
```

### Workflow 3: Airtable → Carousel → Save to Drive

**Trigger:** Airtable Trigger (when record created)

**Nodes:**
1. **Airtable Trigger** - New record created
2. **HTTP Request Node** - Generate carousel
   - Use Airtable field values for titles/subtitles
   - Use background image URLs from Airtable
3. **HTTP Request Node** - Upload to Google Drive (optional)
4. **Airtable Node** - Update record with Drive URLs

**HTTP Request Node Body:**
```json
{
  "backgrounds": ["{{ $json.fields.BackgroundURL }}"],
  "slides": [
    {
      "title": "{{ $json.fields.Title }}",
      "subtitle": "{{ $json.fields.Subtitle }}",
      "textColor": "{{ $json.fields.TextColor || '#FFFFFF' }}"
    }
  ],
  "width": {{ $json.fields.Width || 1080 }},
  "height": {{ $json.fields.Height || 1080 }},
  "uploadToDrive": true,
  "driveToken": "{{ $node['Google Drive OAuth'].json.access_token }}",
  "driveFolderId": "your-folder-id",
  "returnUrls": true
}
```

### Workflow 4: CSV Import → Batch Carousel Generation

**Trigger:** Manual or Schedule

**Nodes:**
1. **Read Binary File Node** - Read CSV file
2. **Code Node** - Parse CSV and split into batches
3. **Split in Batches Node** - Process 5 slides at a time
4. **HTTP Request Node** - Generate carousel for batch
5. **Function Node** - Convert and save images

---

## Error Handling

### Handle Rate Limits

Add a **Wait Node** between multiple HTTP Request calls:

```javascript
// Wait 7 seconds between requests (10 requests/minute = 6 seconds minimum)
// Adding buffer: 7 seconds
```

Or use **Try-Catch Node**:

1. **HTTP Request Node** - Generate carousel
2. **IF Node** - Check if status code is 429
3. **Wait Node** - Wait 60 seconds (if rate limited)
4. **HTTP Request Node** - Retry request

### Handle Failed Slides

Add a **Function Node** after HTTP Request to check for failures:

```javascript
const response = $input.item.json;

// Check if any slides failed
if (response.failed && response.failed.length > 0) {
  // Log failures or send notification
  console.error('Failed slides:', response.failed);
  
  // Optionally, filter out failed slides for next node
  return [{
    json: {
      ...response,
      images: response.images.filter(img => img.success)
    }
  }];
}

return $input.all();
```

### Validate Response

```javascript
// Function Node after HTTP Request
const response = $input.item.json;

if (!response.success) {
  throw new Error(`Carousel generation failed: ${response.error}`);
}

if (!response.images || response.images.length === 0) {
  throw new Error('No images were generated');
}

// Check if all slides succeeded
if (response.stats.failed > 0) {
  // Warn but continue with successful slides
  console.warn(`${response.stats.failed} slides failed to generate`);
}

return $input.all();
```

---

## Converting Base64 to Binary

Most n8n nodes (Instagram, Facebook, etc.) require binary image data, not base64 strings. Use a **Function Node** to convert:

### Basic Conversion

```javascript
// Function Node after HTTP Request
const carouselResponse = $input.item.json;

if (!carouselResponse.images || carouselResponse.images.length === 0) {
  throw new Error('No images in response');
}

// Convert each base64 image to binary
const binaryItems = carouselResponse.images.map((image) => {
  // Remove data:image/png;base64, prefix
  const base64Data = image.base64.replace(/^data:image\/\w+;base64,/, '');
  const buffer = Buffer.from(base64Data, 'base64');

  return {
    json: {
      filename: image.filename,
      originalIndex: carouselResponse.images.indexOf(image)
    },
    binary: {
      data: {
        data: buffer.toString('base64'),
        mimeType: 'image/png',
        fileName: image.filename
      }
    }
  };
});

return binaryItems;
```

### Advanced Conversion with Metadata

```javascript
// Function Node: Convert with additional metadata
const carouselResponse = $input.item.json;

if (!carouselResponse.images || carouselResponse.images.length === 0) {
  return [];
}

const binaryItems = carouselResponse.images.map((image, index) => {
  // Extract base64 data
  const base64Match = image.base64.match(/^data:image\/(\w+);base64,(.+)$/);
  if (!base64Match) {
    throw new Error(`Invalid base64 format for ${image.filename}`);
  }

  const mimeType = `image/${base64Match[1]}`;
  const base64Data = base64Match[2];
  const buffer = Buffer.from(base64Data, 'base64');

  return {
    json: {
      // Original image data
      filename: image.filename,
      slideNumber: index + 1,
      
      // Carousel metadata
      totalSlides: carouselResponse.stats.totalSlides,
      dimensions: carouselResponse.stats.dimensions,
      generationTime: carouselResponse.stats.generationTimeMs,
      
      // Binary file info
      fileSize: buffer.length,
      mimeType: mimeType
    },
    binary: {
      data: {
        data: buffer.toString('base64'),
        mimeType: mimeType,
        fileName: image.filename
      }
    }
  };
});

return binaryItems;
```

### Using with Instagram Node

After converting to binary, connect to Instagram node:

1. **HTTP Request Node** - Generate carousel
2. **Function Node** - Convert base64 to binary (use code above)
3. **Instagram Node** - Post carousel
   - **Operation:** Upload Photo/Video
   - **Binary Property:** `data`
   - **Caption:** Use expression like `{{ $json.filename }}`

---

## Tips & Best Practices

### 1. Use n8n Expressions for Dynamic Content

**Instead of hardcoding:**
```json
{
  "title": "Welcome"
}
```

**Use expressions:**
```json
{
  "title": "{{ $json.title || 'Default Title' }}",
  "subtitle": "{{ $json.subtitle }}",
  "width": {{ $json.width || 1080 }}
}
```

### 2. Validate Input Data Before API Call

Add a **Function Node** before HTTP Request to validate:

```javascript
const input = $input.item.json;

// Validate backgrounds
if (!input.backgrounds || !Array.isArray(input.backgrounds) || input.backgrounds.length === 0) {
  throw new Error('backgrounds array is required');
}

// Validate slides
if (!input.slides || !Array.isArray(input.slides) || input.slides.length === 0) {
  throw new Error('slides array is required');
}

// Ensure arrays match
if (input.backgrounds.length !== input.slides.length) {
  throw new Error(`backgrounds (${input.backgrounds.length}) and slides (${input.slides.length}) must have the same length`);
}

// Validate dimensions
const width = input.width || 1080;
const height = input.height || 1080;

if (width < 200 || width > 4000 || height < 200 || height > 4000) {
  throw new Error('Dimensions must be between 200 and 4000 pixels');
}

return [input];
```

### 3. Batch Processing for Large Carousels

For carousels with many slides (>10), split into batches:

```javascript
// Function Node: Split into batches
const backgrounds = $input.item.json.backgrounds;
const slides = $input.item.json.slides;
const batchSize = 7; // Process 7 slides at a time

const batches = [];
for (let i = 0; i < backgrounds.length; i += batchSize) {
  batches.push({
    json: {
      backgrounds: backgrounds.slice(i, i + batchSize),
      slides: slides.slice(i, i + batchSize),
      width: $input.item.json.width || 1080,
      height: $input.item.json.height || 1080,
      batchIndex: Math.floor(i / batchSize)
    }
  });
}

return batches;
```

### 4. Cache Background Image URLs

If using the same backgrounds repeatedly, store URLs in n8n Credentials or environment variables:

```javascript
// Function Node: Get background URLs
const backgrounds = [
  'https://drive.google.com/uc?id=FILE_ID_1',
  'https://drive.google.com/uc?id=FILE_ID_2'
];

return [{
  json: {
    backgrounds,
    // ... rest of data
  }
}];
```

### 5. Error Notifications

Add error handling with notifications:

1. **Try-Catch Node** - Wrap HTTP Request
2. **IF Node** - Check if error occurred
3. **Slack/Discord/Email Node** - Send error notification

```javascript
// Function Node in Catch block
const error = $input.item.json.error;

return [{
  json: {
    message: `Carousel generation failed: ${error.message || error}`,
    timestamp: new Date().toISOString(),
    workflow: 'Carousel Generator',
    severity: 'error'
  }
}];
```

### 6. Optimize Performance

- **Use smaller dimensions** when possible (1080x1080 vs 4000x4000)
- **Limit carousel size** to 7-10 slides for best performance
- **Process in parallel** using Split in Batches node
- **Cache results** if generating the same carousel multiple times

### 7. Google Drive Integration Tips

When using Google Drive upload:

1. **Get OAuth Token:**
   - Use Google Drive OAuth2 API credential in n8n
   - Access token is available via expression: `{{ $node['Google Drive'].json.access_token }}`

2. **Folder ID:**
   - Get folder ID from Drive URL: `https://drive.google.com/drive/folders/FOLDER_ID`
   - Or use Drive node to list/create folders first

3. **Handle Token Expiry:**
   - n8n handles OAuth refresh automatically
   - Ensure credential is configured with proper scopes

---

## Troubleshooting

### Issue: "Rate limit exceeded" Error

**Solution:**
- Wait 60 seconds between requests
- Implement retry logic with exponential backoff
- Split large batches across multiple executions
- Use multiple API endpoints if needed

### Issue: "Failed to download image" Error (HTTP 303, 301, 302)

**Note:** The API now handles redirects automatically. HTTP 303, 301, 302, 307, 308 redirects are followed automatically (e.g., Google Drive links).

**Solutions:**
- Verify image URL is publicly accessible
- For Google Drive: Use `https://drive.google.com/uc?id=FILE_ID` format (works with or without `&export=download`)
- Check if URL requires authentication (API won't work with private URLs)
- Test image URL in browser first
- If you see "Too many redirects" error: URL may have a redirect loop (max 10 redirects supported)

### Issue: Base64 images not displaying/posting

**Solutions:**
- Convert base64 to binary using Function Node (see [Converting Base64 to Binary](#converting-base64-to-binary))
- Verify binary format: `{ data: { data: string, mimeType: string, fileName: string } }`
- Check MIME type is `image/png`
- Ensure Instagram/Facebook nodes are configured with correct binary property

### Issue: Text not visible on images

**Solutions:**
- Use high contrast colors (white text on dark backgrounds)
- Avoid similar text and background colors
- Text automatically has stroke and shadow for readability
- Test with different `textColor` values

### Issue: Slow generation times

**Solutions:**
- Reduce image dimensions (1080x1080 is optimal)
- Use faster image hosting (CDN-backed URLs)
- Limit number of slides per request
- Check background image download speeds

### Issue: Google Drive upload fails

**Solutions:**
- Verify OAuth token is valid and not expired
- Check folder ID is correct and accessible
- Ensure OAuth scope includes `https://www.googleapis.com/auth/drive.file`
- Test token with a simple Drive API call first

### Issue: "Response payload size exceeded" (HTTP 413)

**Symptom:** Error message: "Response payload size exceeded maximum allowed payload size (6291556 bytes)"

**Cause:** Netlify functions have a 6MB response limit. Large carousels with many slides or high-resolution images exceed this limit when returning base64 data.

**Solutions:**
1. **Use `returnUrls: true` with Google Drive upload (Recommended):**
   ```json
   {
     "uploadToDrive": true,
     "driveToken": "{{ $node['Google Drive OAuth'].json.access_token }}",
     "driveFolderId": "your-folder-id",
     "returnUrls": true
   }
   ```
   This returns Drive URLs instead of base64 data, dramatically reducing payload size.

2. **Reduce number of slides:** Generate fewer slides per request (3-5 instead of 10+)

3. **Use smaller dimensions:** Reduce width/height (e.g., 1080x1080 instead of 4000x4000)

4. **Split into batches:** Process carousels in smaller batches using Split in Batches node

**Example with returnUrls:**
```json
{
  "backgrounds": ["https://..."],
  "slides": [{"title": "...", "subtitle": "..."}],
  "uploadToDrive": true,
  "driveToken": "ya29...",
  "driveFolderId": "folder-id",
  "returnUrls": true
}
```

**Response with returnUrls:**
```json
{
  "success": true,
  "images": [
    {
      "filename": "carousel-slide-1.png",
      "success": true,
      "driveUrl": "https://drive.google.com/uc?id=..."
    }
  ],
  "driveUrls": [...],
  "stats": {...}
}
```

---

## Example Workflow JSON

Complete n8n workflow JSON for Instagram carousel automation:

```json
{
  "name": "Instagram Carousel Generator",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "cronExpression",
              "expression": "0 9 * * 1"
            }
          ]
        }
      },
      "name": "Schedule Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "jsCode": "const backgrounds = [\n  'https://picsum.photos/id/1015/1080/1080',\n  'https://picsum.photos/id/1018/1080/1080'\n];\n\nconst slides = [\n  { title: 'Monday Motivation', subtitle: 'Start your week strong', textColor: '#FFFFFF' },\n  { title: 'Tip of the Week', subtitle: 'Stay focused and productive', textColor: '#FFFFFF' }\n];\n\nreturn [{\n  json: {\n    backgrounds,\n    slides,\n    width: 1080,\n    height: 1080\n  }\n}];"
      },
      "name": "Prepare Data",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [450, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://your-app.netlify.app/.netlify/functions/carousel",
        "sendBody": true,
        "bodyContentType": "json",
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify($json) }}",
        "options": {
          "timeout": 15000
        }
      },
      "name": "Generate Carousel",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [650, 300]
    },
    {
      "parameters": {
        "jsCode": "const response = $input.item.json;\n\nif (!response.images || response.images.length === 0) {\n  throw new Error('No images generated');\n}\n\nconst binaryItems = response.images.map((image) => {\n  const base64Data = image.base64.replace(/^data:image\\/\\w+;base64,/, '');\n  const buffer = Buffer.from(base64Data, 'base64');\n\n  return {\n    json: {\n      filename: image.filename\n    },\n    binary: {\n      data: {\n        data: buffer.toString('base64'),\n        mimeType: 'image/png',\n        fileName: image.filename\n      }\n    }\n  };\n});\n\nreturn binaryItems;"
      },
      "name": "Convert to Binary",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [850, 300]
    }
  ],
  "connections": {
    "Schedule Trigger": {
      "main": [[{ "node": "Prepare Data", "type": "main", "index": 0 }]]
    },
    "Prepare Data": {
      "main": [[{ "node": "Generate Carousel", "type": "main", "index": 0 }]]
    },
    "Generate Carousel": {
      "main": [[{ "node": "Convert to Binary", "type": "main", "index": 0 }]]
    }
  }
}
```

---

## Additional Resources

- [Main README](./README.md) - Full project documentation
- [Deployment Guide](./DEPLOYMENT.md) - Deploy to Netlify
- [Quick Start Guide](./QUICKSTART.md) - Get started in 5 minutes
- [n8n Documentation](https://docs.n8n.io/) - n8n workflow platform docs

---

**Need Help?** Open an issue on [GitHub](https://github.com/jeremiahagthe/Free_templateIo_copy/issues)

**Built with ❤️ for the n8n community**
