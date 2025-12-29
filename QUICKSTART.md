# ⚡ Quick Start Guide

Get your Carousel Generator running in 5 minutes!

## 🎯 Goal

Deploy a free, self-hosted carousel generation API that works with n8n automation.

## ✅ What You Need

- GitHub account (free)
- Netlify account (free)
- 5 minutes

## 🚀 Steps

### 1. Get the Code (1 minute)

**Option A: Fork this repo**
1. Click "Fork" on GitHub
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/Free_templateIo_copy.git
   cd Free_templateIo_copy
   ```

**Option B: Download ZIP**
1. Download this repo as ZIP
2. Extract and open terminal in folder

### 2. Deploy to Netlify (2 minutes)

**Option A: Via GitHub (Recommended)**
1. Push code to your GitHub repo
2. Go to [app.netlify.com](https://app.netlify.com)
3. Click "Add new site" → "Import from Git"
4. Select your repo
5. Leave default settings
6. Click "Deploy"

**Option B: Via CLI**
```bash
npm install -g netlify-cli
netlify login
netlify init
netlify deploy --prod
```

### 3. Test It (2 minutes)

**Web UI Test:**
1. Open: `https://YOUR-SITE.netlify.app`
2. Click "Load Sample Data"
3. Click "Generate Carousel"
4. ✅ You should see 2 carousel images!

**API Test:**
```bash
curl -X POST https://YOUR-SITE.netlify.app/.netlify/functions/carousel \
  -H "Content-Type: application/json" \
  -d '{
    "backgrounds": ["https://picsum.photos/1080/1080"],
    "slides": [{"title": "Hello", "subtitle": "This is a test"}]
  }'
```

✅ If you get a JSON response with base64 images, **you're done!**

---

## 📱 Use with n8n

### Step 1: Add HTTP Request Node

In n8n workflow:
1. Add "HTTP Request" node
2. Set method to **POST**
3. Set URL to: `https://YOUR-SITE.netlify.app/.netlify/functions/carousel`

### Step 2: Configure Request Body

```json
{
  "backgrounds": [
    "{{ $json.background_url_1 }}",
    "{{ $json.background_url_2 }}"
  ],
  "slides": [
    {
      "title": "{{ $json.title_1 }}",
      "subtitle": "{{ $json.subtitle_1 }}",
      "textColor": "#FFFFFF"
    },
    {
      "title": "{{ $json.title_2 }}",
      "subtitle": "{{ $json.subtitle_2 }}",
      "textColor": "#FFFFFF"
    }
  ],
  "width": 1080,
  "height": 1080
}
```

### Step 3: Get Results

The response includes:
- `images[]` - Array of base64 images
- `images[].base64` - Image data (use this for uploads)
- `images[].filename` - Suggested filename

### Step 4: Upload to Social Media

Use n8n's Instagram/Facebook nodes:
1. Loop through `images[]` array
2. Convert base64 to binary
3. Upload to platform

**Example n8n function to convert:**
```javascript
const base64Data = $json.images[0].base64.replace(/^data:image\/\w+;base64,/, '');
return [{
  json: {},
  binary: {
    data: {
      data: base64Data,
      mimeType: 'image/png'
    }
  }
}];
```

---

## 🎨 Customize Text Styling

### Change Text Color

```json
{
  "slides": [
    {
      "title": "Red Title",
      "subtitle": "Red subtitle text",
      "textColor": "#FF0000"
    }
  ]
}
```

### Different Dimensions

```json
{
  "width": 1080,
  "height": 1920  // Instagram Stories
}
```

**Common sizes:**
- Instagram Square: 1080 x 1080
- Instagram Stories: 1080 x 1920
- Facebook: 1080 x 1080
- TikTok: 1080 x 1920
- LinkedIn: 1200 x 627

---

## 💡 Tips

### 1. Google Drive Images

Use direct download format:
```
❌ https://drive.google.com/file/d/FILE_ID/view
✅ https://drive.google.com/uc?id=FILE_ID
```

### 2. Fast Generation

- Keep dimensions reasonable (1080x1080 is optimal)
- Use max 10 slides per request
- Compress background images before uploading

### 3. Error Handling

The API gracefully handles errors:
- If one slide fails, others still generate
- Check `failed[]` array in response
- Individual slide errors don't break the request

### 4. Rate Limits

Free tier: 10 requests/minute per IP

For high volume:
- Deploy multiple instances
- Add delays in n8n workflow
- Upgrade to Netlify Pro

---

## 📚 Next Steps

1. **Read the full README:** [README.md](README.md)
2. **Deployment guide:** [DEPLOYMENT.md](DEPLOYMENT.md)
3. **API documentation:** See README.md API Reference section
4. **n8n examples:** See README.md n8n Integration section

---

## 🆘 Troubleshooting

### "Failed to download image"
- Check if URL is public
- Verify Google Drive links use `uc?id=` format
- Test URL in browser first

### "Rate limit exceeded"
- Wait 60 seconds
- Reduce request frequency
- Deploy additional instances

### Images don't have text
- Verify `slides[]` array has `title` and `subtitle`
- Check `textColor` is valid hex code
- Try white (#FFFFFF) or black (#000000) first

### No response from API
- Check Netlify deployment status
- View function logs in Netlify dashboard
- Test with smaller payload first

---

## 💬 Get Help

- **Issues:** [GitHub Issues](https://github.com/jeremiahagthe/Free_templateIo_copy/issues)
- **Discussions:** [GitHub Discussions](https://github.com/jeremiahagthe/Free_templateIo_copy/discussions)

---

**That's it! You now have a free carousel generator API running on Netlify.** 🎉

Save hundreds of dollars by hosting your own instead of paying for template.io or similar services!
