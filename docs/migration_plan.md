# marisabassoon.com — Migration Plan
## Moving from single HTML file to GitHub Pages with separate image assets

---

## Background

The current site is a single `index.html` file (~60MB) with all images embedded as base64 strings directly in the HTML. This caused Netlify's free plan to hit its bandwidth limit and pause the site.

The fix is to extract all images into separate files, shrink the HTML to ~50KB, and host everything on GitHub Pages, which handles this kind of static site well on the free tier.

---

## Current file location

The current `index.html` is available in this conversation's outputs. It contains:
- All site HTML, CSS, and JavaScript
- 14 images embedded as base64 strings (hero photo, gallery photos, Music with Marisa photo, etc.)
- 2 SoundCloud embeds
- 3 YouTube embeds
- A working EN/DE language toggle

---

## Goal

Produce a proper static site folder structure:

```
marisa-website/
├── index.html          # ~50KB instead of 60MB
├── images/
│   ├── hero.jpg
│   ├── mwm-photo.jpg
│   ├── gallery-01.jpg
│   ├── gallery-02.jpg
│   ├── gallery-03.jpg
│   ├── gallery-04.jpg
│   ├── gallery-05.jpg
│   ├── gallery-06.jpg
│   ├── gallery-07.jpg
│   ├── gallery-08.jpg
│   ├── gallery-enescu-01.jpg
│   ├── gallery-enescu-02.jpg
│   └── gallery-enescu-03.jpg
└── README.md
```

---

## Step-by-step plan for Claude Code

### Step 1 — Extract images from the HTML

The HTML file contains base64-encoded images in the following locations:

| Variable name | Location in HTML | Description |
|---|---|---|
| `hero.jpg` | Inside `<section class="hero">`, first `<img>` tag | Outdoor portrait with bassoon |
| `mwm-photo.jpg` | Inside `class="music-with-marisa"`, `class="mwm-photo"` | Ukulele photo for Music with Marisa |
| `gallery-01` through `gallery-08` | Inside `class="gallery-grid"`, `onclick="openLightbox(0)"` through `openLightbox(7)"` | Spisak competition photos |
| `gallery-enescu-01` through `gallery-enescu-03` | `onclick="openLightbox(8)"` through `openLightbox(10)"` | Enescu Festival photos |

Also inside the `<script>` block at the bottom, the `galleryImages` JavaScript array contains all 11 gallery images as base64 strings — these need to be replaced with file paths too.

**Task:** Write a Python script that:
1. Reads `index.html`
2. Finds each `data:image/jpeg;base64,...` or `data:image/png;base64,...` string
3. Decodes it and saves it as a `.jpg` file in an `images/` folder
4. Replaces the base64 string in the HTML with the relative file path (e.g. `images/hero.jpg`)
5. Does the same replacement inside the `galleryImages` JS array

### Step 2 — Update the HTML image references

All `src="data:image/..."` attributes should become `src="images/filename.jpg"`.

The `galleryImages` JS array currently looks like:
```javascript
const galleryImages = [
  { src: "data:image/jpeg;base64,/9j/...", caption: "..." },
  ...
];
```

It should become:
```javascript
const galleryImages = [
  { src: "images/gallery-01.jpg", caption: "Award ceremony · 13th Michał Spisak Competition · 2021" },
  { src: "images/gallery-02.jpg", caption: "Receiving the prize · 13th Michał Spisak Competition · 2021" },
  { src: "images/gallery-03.jpg", caption: "Laureates on stage · 13th Michał Spisak Competition · 2021" },
  { src: "images/gallery-04.jpg", caption: "With fellow laureates · 13th Michał Spisak Competition · 2021" },
  { src: "images/gallery-05.jpg", caption: "Competition performance · 13th Michał Spisak Competition · 2021" },
  { src: "images/gallery-06.jpg", caption: "On stage with piano · 13th Michał Spisak Competition · 2021" },
  { src: "images/gallery-07.jpg", caption: "Solo performance · 13th Michał Spisak Competition · 2021" },
  { src: "images/gallery-08.jpg", caption: "Before the jury · 13th Michał Spisak Competition · 2021" },
  { src: "images/gallery-enescu-01.jpg", caption: "Würth Philharmoniker · woodwinds · George Enescu Festival, Bucharest · 2023" },
  { src: "images/gallery-enescu-02.jpg", caption: "Würth Philharmoniker on stage · George Enescu Festival, Bucharest · 2023" },
  { src: "images/gallery-enescu-03.jpg", caption: "Romanian Athenaeum · George Enescu Festival · 2023" },
];
```

### Step 3 — Set up GitHub repository

1. Create a new **private** GitHub repository called `marisabassoon-site`
2. Push the `marisa-website/` folder contents to the `main` branch
3. Go to **Settings → Pages**
4. Set source to **Deploy from branch → main → / (root)**
5. GitHub Pages will publish the site at `https://yourusername.github.io/marisabassoon-site`

### Step 4 — Connect custom domain

1. In GitHub Pages settings, add custom domain: `marisabassoon.com`
2. In Namecheap **Advanced DNS**, update the A records to point to GitHub Pages IPs:
   - `185.199.108.153`
   - `185.199.109.153`
   - `185.199.110.153`
   - `185.199.111.153`
3. Add a CNAME record:
   - Host: `www`
   - Value: `yourusername.github.io`
4. Back in GitHub Pages, click **Enforce HTTPS** once DNS propagates

### Step 5 — Future updates workflow

When Marisa wants to update the site:
1. Make changes to `index.html` (or images) locally
2. Commit and push to `main` branch
3. GitHub Pages automatically redeploys within a minute or two

---

## Notes for Claude Code

- The `index.html` file is in the outputs of the Claude.ai conversation. It can be copy-pasted or re-uploaded.
- Image quality: when extracting and re-saving base64 images, use JPEG quality 85 to keep file sizes reasonable without visible quality loss.
- The hero image is a PNG originally — save it as JPG for consistency and smaller file size.
- The MwM photo was converted from a PDF to JPEG during the conversation — it should already be a proper JPEG in the base64.
- The language toggle (EN/DE) uses `data-en` and `data-de` attributes with a small vanilla JS function — no frameworks, no build step needed.
- The lightbox uses vanilla JS with a `galleryImages` array — once the src values are updated to file paths it will work identically.
- No npm, no bundler, no build step required — this is a pure HTML/CSS/JS static site.
