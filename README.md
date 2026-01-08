# 🎨 Carousel Generator - Free Template.io Alternative

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Node](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen.svg)
![Netlify](https://img.shields.io/badge/netlify-ready-00C7B7.svg)

**A completely FREE, self-hosted serverless API for generating stunning social media carousels using Sharp.** No external services required. Zero monthly costs. Deploy to Netlify in 2 minutes.

## ✨ Features

- 🚀 **Blazing Fast** - Generate 7 carousel slides in <500ms using parallel processing
- 💰 **100% Free** - Netlify free tier: 125K requests/month, 100GB bandwidth
- 🎨 **Beautiful Text Overlays** - Title + subtitle with stroke, shadow, and custom colors
- 📱 **Any Platform** - Custom dimensions for Instagram, Facebook, TikTok, LinkedIn
- 🔗 **n8n Ready** - Perfect integration with automation workflows
- 🌐 **Simple Web UI** - Test and generate carousels directly in your browser
- 📤 **Google Drive Integration** - Optionally upload generated images to Drive
- 🛡️ **Secure** - Rate limiting, input validation, graceful error handling
- 🎯 **Zero Dependencies** on external image services

## 🎯 Why This vs Template.io?

| Feature | This Tool | Template.io |
|---------|-----------|-------------|
| **Cost** | FREE (forever) | $29-99/month |
| **Hosting** | Self-hosted (Netlify) | SaaS |
| **Customization** | Full control | Limited |
| **API Access** | Unlimited | Paid plans only |
| **Privacy** | Your data stays yours | 3rd party service |
| **n8n Integration** | Built-in support | Complex |

## 🚀 Quick Start

### 1. Deploy to Netlify (2 minutes)

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy)

Or manually:

```bash
# Clone repository
git clone https://github.com/jeremiahagthe/Free_templateIo_copy.git
cd Free_templateIo_copy

# Install dependencies
npm install

# Deploy to Netlify
npm run deploy
```

### 2. Test with Web UI

After deployment, open your Netlify URL (e.g., `https://your-app.netlify.app`) in a browser.

1. Add background image URLs (Google Drive, Unsplash, etc.)
2. Create slides with titles and subtitles
3. Set dimensions (1080x1080 for Instagram, 1080x1920 for Stories)
4. Click "Generate Carousel"
5. Download your images!

### 3. Use the API

**Endpoint:** `POST https://your-app.netlify.app/.netlify/functions/carousel`

**Request:**
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

**Response:**
```json
{
  "success": true,
  "images": [
    {
      "base64": "data:image/png;base64,iVBORw0KGgo...",
      "filename": "carousel-slide-1.png",
      "success": true
    }
  ],
  "stats": {
    "totalSlides": 2,
    "successful": 2,
    "failed": 0,
    "generationTimeMs": 347,
    "dimensions": { "width": 1080, "height": 1080 }
  }
}
```

## 🔗 n8n Integration

Perfect for automated social media workflows!

### Example Workflow: Google Drive → Carousel Generator → Instagram

1. **Trigger:** Schedule (e.g., every Monday at 9am)
2. **Google Drive Node:** List files from your templates folder
3. **HTTP Request Node:** Call carousel API
4. **Instagram Node:** Post carousel to Instagram

### n8n HTTP Request Node Setup

```javascript
// Node: HTTP Request
{
  "method": "POST",
  "url": "https://your-app.netlify.app/.netlify/functions/carousel",
  "authentication": "none",
  "jsonParameters": true,
  "options": {
    "timeout": 15000
  },
  "bodyParametersJson": {
    "backgrounds": [
      "{{ $json.background1 }}",
      "{{ $json.background2 }}"
    ],
    "slides": [
      {
        "title": "{{ $json.title1 }}",
        "subtitle": "{{ $json.subtitle1 }}",
        "textColor": "#FFFFFF"
      },
      {
        "title": "{{ $json.title2 }}",
        "subtitle": "{{ $json.subtitle2 }}",
        "textColor": "#FFFFFF"
      }
    ],
    "width": 1080,
    "height": 1080,
    "uploadToDrive": true,
    "driveToken": "{{ $node['Google Drive OAuth'].json.access_token }}",
    "driveFolderId": "your-drive-folder-id"
  }
}
```

### Converting Base64 to Image Files in n8n

```javascript
// Node: Function
// Convert base64 to binary for Instagram upload

const images = items[0].json.images;
const binaries = [];

for (let i = 0; i < images.length; i++) {
  const base64Data = images[i].base64.replace(/^data:image\/\w+;base64,/, '');
  const buffer = Buffer.from(base64Data, 'base64');

  binaries.push({
    json: { filename: images[i].filename },
    binary: {
      data: {
        data: buffer.toString('base64'),
        mimeType: 'image/png',
        fileName: images[i].filename
      }
    }
  });
}

return binaries;
```

## 📐 Supported Dimensions

| Platform | Recommended Size | Aspect Ratio |
|----------|-----------------|--------------|
| **Instagram Feed** | 1080 x 1080 | 1:1 (Square) |
| **Instagram Stories** | 1080 x 1920 | 9:16 (Vertical) |
| **Facebook Post** | 1080 x 1080 | 1:1 (Square) |
| **TikTok** | 1080 x 1920 | 9:16 (Vertical) |
| **LinkedIn** | 1200 x 627 | 1.91:1 (Horizontal) |
| **Twitter/X** | 1200 x 675 | 16:9 (Horizontal) |
| **Pinterest** | 1000 x 1500 | 2:3 (Vertical) |

Custom dimensions: Min 200x200, Max 4000x4000

## 🎨 Text Styling

The API automatically applies professional text styling with automatic text wrapping:

- **Title:** ~60pt bold font (customizable via `titleSize`)
- **Subtitle:** ~36pt regular font (customizable via `subtitleSize`)
- **Effects:** White text with dark stroke + shadow for readability
- **Text Wrapping:** Automatically wraps long text to prevent overflow
- **Custom Positioning:** Control exact X/Y positions with `titleX`, `titleY`, `subtitleX`, `subtitleY`
- **Width Control:** Set `maxTitleWidth` and `maxSubtitleWidth` to control text boundaries
- **Text Alignment:** Choose `left`, `center`, or `right` alignment
- **Responsive:** Font sizes scale with image dimensions (or use custom sizes)
- **Safe Area:** Text limited to 20% of image area by default (customizable)
- **Font Selection:** Choose from popular fonts (Arial, Helvetica, Impact, Futura, Georgia, Times, etc.)
- **⚠️ Font Limitation:** Only system fonts (Arial, Helvetica, Impact, Futura, Georgia, Times) work reliably in serverless environments. Google Fonts (Roboto, Montserrat, Bebas Neue, Open Sans) will fall back to Arial unless fonts are embedded as base64 data URIs.

## 📤 Google Drive Upload (Optional)

Upload generated carousel images directly to Google Drive:

### Setup

1. **Get OAuth Token in n8n:**
   - Add "Google Drive" credential
   - Use the "Google Drive OAuth2 API" authentication
   - n8n handles the token refresh automatically

2. **Pass Token in API Call:**

```json
{
  "backgrounds": [...],
  "slides": [...],
  "uploadToDrive": true,
  "driveToken": "ya29.a0AfH6SMB...",
  "driveFolderId": "1a2b3c4d5e6f",
  "returnUrls": true
}
```

**Note:** Setting `returnUrls: true` returns Drive URLs instead of base64 data, which significantly reduces response payload size (useful for large carousels).

3. **Response includes Drive URLs:**

```json
{
  "images": [...],
  "driveUrls": [
    {
      "success": true,
      "fileId": "abc123",
      "webViewLink": "https://drive.google.com/file/d/abc123/view",
      "webContentLink": "https://drive.google.com/uc?id=abc123"
    }
  ]
}
```

### Using Google Drive Images as Backgrounds

To use Google Drive images as carousel backgrounds:

```
❌ Wrong: https://drive.google.com/file/d/1a2b3c4d5e6f/view
✅ Correct: https://drive.google.com/uc?id=1a2b3c4d5e6f
```

## ⚙️ API Reference

### POST /.netlify/functions/carousel

#### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `backgrounds` | string[] | ✅ | Array of image URLs |
| `slides` | object[] | ✅ | Array of slide objects |
| `slides[].title` | string | ✅ | Slide title |
| `slides[].subtitle` | string | ✅ | Slide subtitle |
| `slides[].textColor` | string | ❌ | Hex color (default: #FFFFFF) |
| `slides[].fontFamily` | string | ❌ | Font name (default: Arial) |
| `slides[].titleSize` | number | ❌ | Title font size in px (auto-calculated if not provided) |
| `slides[].subtitleSize` | number | ❌ | Subtitle font size in px (auto-calculated if not provided) |
| `slides[].titleX` | number | ❌ | Title X position (default: center) |
| `slides[].titleY` | number | ❌ | Title Y position (default: center) |
| `slides[].subtitleX` | number | ❌ | Subtitle X position (default: center) |
| `slides[].subtitleY` | number | ❌ | Subtitle Y position (default: center) |
| `slides[].maxTitleWidth` | number | ❌ | Max title width in px (default: width - 10% padding) |
| `slides[].maxSubtitleWidth` | number | ❌ | Max subtitle width in px (default: width - 10% padding) |
| `slides[].textAlign` | string | ❌ | Text alignment: 'left', 'center', 'right' (default: 'center') |
| `width` | number | ❌ | Image width (default: 1080) |
| `height` | number | ❌ | Image height (default: 1080) |
| `uploadToDrive` | boolean | ❌ | Upload to Google Drive |
| `driveToken` | string | ❌ | Google OAuth token |
| `driveFolderId` | string | ❌ | Drive folder ID |
| `returnUrls` | boolean | ❌ | Return Drive URLs instead of base64 (reduces payload size) |

#### Response

```typescript
{
  success: boolean;
  images: Array<{
    base64: string;        // data:image/png;base64,...
    filename: string;      // carousel-slide-1.png
    success: boolean;
  }>;
  failed?: Array<{         // Only if some slides failed
    filename: string;
    error: string;
    success: false;
  }>;
  driveUrls?: Array<{      // Only if uploadToDrive: true
    success: boolean;
    fileId?: string;
    webViewLink?: string;
    webContentLink?: string;
    error?: string;
  }>;
  stats: {
    totalSlides: number;
    successful: number;
    failed: number;
    generationTimeMs: number;
    dimensions: { width: number; height: number; }
  };
}
```

#### Error Responses

| Status | Error | Description |
|--------|-------|-------------|
| 400 | Invalid request | Missing required fields |
| 413 | Payload too large | Response exceeds 6MB limit (use `returnUrls: true` with Drive upload) |
| 429 | Rate limit exceeded | Max 10 requests/minute per IP |
| 500 | Internal error | Server error (check logs) |

## 🛡️ Security & Limits

- ✅ **Rate Limiting:** 10 requests per minute per IP
- ✅ **Timeout:** 10 seconds per request
- ✅ **Input Validation:** Sanitized text, validated URLs
- ✅ **Dimension Limits:** 200x200 to 4000x4000 pixels
- ✅ **Response Size Limit:** 6MB maximum (use `returnUrls: true` for large carousels)
- ✅ **CORS Enabled:** Works from any domain
- ✅ **Graceful Failures:** Individual slide errors don't break the entire request

## 🚨 Common Issues

### 1. Image Download Fails

**Symptom:** Slide generation fails with "Failed to download image"

**Solutions:**
- Verify image URL is publicly accessible
- For Google Drive: Use `https://drive.google.com/uc?id=FILE_ID` format
- For Unsplash: Use direct image URL, not page URL
- Check if URL requires authentication

### 2. Rate Limit Exceeded

**Symptom:** 429 error

**Solutions:**
- Wait 1 minute between requests
- Upgrade to Netlify Pro for higher limits
- Deploy multiple instances with load balancing

### 3. Text Not Visible

**Symptom:** Generated images don't show text

**Solutions:**
- Use light text color (#FFFFFF) on dark backgrounds
- Use dark text color (#000000) on light backgrounds
- Text has stroke/shadow for better contrast

### 4. Slow Generation

**Symptom:** Takes > 2 seconds for small carousels

**Solutions:**
- Use smaller images (1080x1080 vs 4000x4000)
- Reduce number of slides
- Check background image server speed
- Consider Netlify Pro for more resources

## 📊 Performance

**Benchmarks (Netlify Free Tier):**

| Slides | Dimensions | Avg Time | P95 Time |
|--------|-----------|----------|----------|
| 1 | 1080x1080 | 180ms | 250ms |
| 3 | 1080x1080 | 280ms | 380ms |
| 7 | 1080x1080 | 420ms | 580ms |
| 10 | 1080x1080 | 650ms | 850ms |
| 7 | 1080x1920 | 580ms | 780ms |

**Monthly Limits (Netlify Free Tier):**
- 125,000 function requests
- 100 hours function runtime
- 100GB bandwidth
- **≈ 100 carousels per day** (7 slides each)

## 🔧 Local Development

```bash
# Install dependencies
npm install

# Start local development server
npm run dev

# Test locally
curl -X POST http://localhost:8888/.netlify/functions/carousel \
  -H "Content-Type: application/json" \
  -d '{
    "backgrounds": ["https://picsum.photos/1080/1080"],
    "slides": [{"title": "Test", "subtitle": "Local test"}]
  }'

# Deploy to production
npm run deploy
```

## 📦 Tech Stack

- **Sharp** - High-performance image processing
- **Netlify Functions** - Serverless hosting
- **Google APIs** - Optional Drive integration
- **Node.js 18+** - Runtime environment

## 📝 License

MIT License - feel free to fork, modify, and use commercially!

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repo
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 💡 Ideas for Enhancement

- [ ] Support for bullet points/lists
- [ ] Multiple text layers
- [ ] Custom font uploads
- [ ] Image filters (blur, brightness, etc.)
- [ ] Template library (pre-made designs)
- [ ] Batch processing via CSV
- [ ] Webhook triggers
- [ ] Analytics dashboard
- [ ] AI-generated copy suggestions

## 🌟 Star History

If this tool saves you money, please star the repo!

## 📧 Support

- **Issues:** [GitHub Issues](https://github.com/jeremiahagthe/Free_templateIo_copy/issues)
- **Discussions:** [GitHub Discussions](https://github.com/jeremiahagthe/Free_templateIo_copy/discussions)

---

**Built with ❤️ as a free alternative to expensive SaaS tools**

*Save your money. Host your own. Own your data.*
