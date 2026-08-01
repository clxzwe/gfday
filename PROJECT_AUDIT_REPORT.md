# LoveArea.in — Complete Project Audit Report

**Audit date:** August 1, 2026  
**Workspace path:** `/home/akuma/.gemini/antigravity/scratch/lovearea/lovearea.in`  
**Live site:** [https://lovearea.in](https://lovearea.in)  
**Git remote:** [https://github.com/clxzwe/gfday.git](https://github.com/clxzwe/gfday.git)  

---

## Critical Finding

> **This workspace contains a production build artifact (Vite** `dist/` **output), not the source codebase.**  
> There is no `package.json`, no `src/` directory, no TypeScript/React source files, and no build configuration. All application logic lives inside a single minified JavaScript bundle (`assets/index-DZYHQvRn.js`, 1.39 MB). The original source was likely built with Lovable.dev (commented references remain in `index.html`).

---



# 1. Project Overview


| Field                    | Value                                                                                                                                                                                                                                  |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Project name**         | LoveArea (`lovearea.in`)                                                                                                                                                                                                               |
| **Purpose**              | SaaS platform for creating personalized, animated romantic web pages for occasions (birthdays, anniversaries, Valentine's Day, proposals, etc.). Includes template editor, user auth, cart/checkout, and Cashfree payment integration. |
| **Current completion %** | **~80% (live product estimate)** — Full SPA with 10 template occasions, auth, payments, and blog routes are bundled. **This local deploy artifact is only ~15% asset-complete** (69 of 455 referenced media files present).            |
| **Overall architecture** | Static SPA frontend (React + Vite) → REST API backend at `https://api.lovearea.in` → Cashfree payments → Google OAuth → Cloudflare Pages hosting                                                                                       |
| **Production ready?**    | **Partially.** The live site at lovearea.in appears deployed and functional. This local copy has critical gaps: 386 missing assets, one 238 MB video, an accidental 10.8 MB duplicate bundle, and no source code for maintenance.      |




### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Cloudflare Pages (CDN)                    │
│  index.html → index-DZYHQvRn.js (React SPA) + CSS + assets  │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTPS
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
   api.lovearea.in   sdk.cashfree.com   fonts.googleapis.com
   (Backend API)     (Payments)         (Web fonts)
          │
          ├── /auth/* (login, signup, Google OAuth)
          ├── /payments/* (create-link, orders)
          ├── /media/upload
          └── /s/:slug (personalized pages)
```

---



# 2. Directory Structure

```
lovearea.in/                          256.63 MB (excl. .git)
├── index.html                        SPA shell / entry point (2,958 bytes)
├── favicon.ico                       Site favicon (2,692 bytes)
├── logo.png                          Logo / OG image (2,692 bytes)
├── logo.webp                         WebP logo variant (21,186 bytes)
├── Perfect.mp3                       Girlfriend Day template BGM (272,091 bytes)
├── .gitignore                        Ignores large videos (136 bytes)
├── assets/                           256.32 MB — all static media + bundles
│   ├── index-DZYHQvRn.js             Main production JS bundle (1,451,616 bytes)
│   ├── index-B30c12lU.css            Main production CSS bundle (251,505 bytes)
│   ├── test_module.mjs               ⚠ Accidental duplicate bundle (10,885,080 bytes, UNTRACKED)
│   ├── cherry-blossom-lake-house-minecraft.mp4  ⚠ 238 MB background video (GITIGNORED)
│   ├── kiss_video.mp4                Unused kiss video (1,881,608 bytes)
│   ├── love-D-moh3Up.mp4             Short love animation (48,316 bytes)
│   ├── couple-CIb7Mo28.gif           Animated couple GIF (294,943 bytes)
│   ├── girlfriend-6K1GTJ07.webp      Hero illustration (219,076 bytes)
│   ├── moment1.jpg – moment15.jpg    15 personal moment photos (~1.2 MB total)
│   ├── mc_*.png/jpg                  Minecraft-themed UI textures
│   ├── bg*.webp, bgr*.webp           Background illustrations (36 WebP files)
│   ├── *.webp (occasion icons)       birthday, anniversary, apology, etc.
│   ├── AngelWithNoWings.mp3          Audio (189,962 bytes, duplicate of DiscSong)
│   ├── DiscSong.mp3                  Minecraft disc audio (189,962 bytes)
│   ├── Minecraft.ttf                 Pixel font (14,488 bytes, duplicated)
│   └── minecraft_font/
│       └── Minecraft.ttf             Same font file (14,488 bytes)
├── template/
│   └── girlfriendday/
│       └── perfect-girlfriend.html   Template preview shell (2,796 bytes)
├── .well-known/
│   └── appspecific/
│       └── com.chrome.devtools.json  Chrome DevTools artifact (2,692 bytes)
└── .git/                             53 MB — git objects
```



### Folder Explanations


| Folder                    | Contents                                                    |
| ------------------------- | ----------------------------------------------------------- |
| `/` (root)                | HTML entry point, logo, favicon, root-level audio           |
| `assets/`                 | All bundled JS/CSS, images, videos, audio, fonts — 81 files |
| `assets/minecraft_font/`  | Minecraft pixel font (duplicate of root-level copy)         |
| `template/girlfriendday/` | Static HTML shell for girlfriend day template preview       |
| `.well-known/`            | Chrome DevTools configuration (should be removed)           |
| `.git/`                   | Git repository (52 MB object store)                         |


**Notable absence:** No `src/`, `components/`, `pages/`, `public/`, `node_modules/`, `package.json`, or any deployment config (`netlify.toml`, `vercel.json`, `_redirects`, `wrangler.toml`).

---



# 3. Tech Stack


| Category               | Technology                                                           | Evidence                                                    |
| ---------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Framework**          | React 18                                                             | 119 references in JS bundle                                 |
| **Language**           | JavaScript (compiled from TypeScript/JSX)                            | Minified `.js` output; no source                            |
| **Runtime**            | Browser (client-side SPA)                                            | `index.html` mounts `#root`                                 |
| **Build tool**         | Vite                                                                 | 20 references; hashed asset filenames (`index-DZYHQvRn.js`) |
| **Bundler**            | Vite (Rollup-based)                                                  | Single JS + CSS output, no code splitting                   |
| **Styling**            | Tailwind CSS                                                         | 4,157 `--tw-` / `@apply` references in CSS                  |
| **UI library**         | Radix UI                                                             | 57 references (dialogs, toasts, tooltips, etc.)             |
| **Icons**              | Lucide React                                                         | 2 references                                                |
| **Animation**          | Framer Motion                                                        | 1 reference; CSS animations in Tailwind                     |
| **State management**   | React hooks (useState, useEffect, useMemo)                           | No Redux/Zustand detected                                   |
| **Routing**            | React Router                                                         | 15 defined routes                                           |
| **HTTP client**        | Axios                                                                | 14 references                                               |
| **Notifications**      | Sonner                                                               | 115 references                                              |
| **Confetti**           | canvas-confetti                                                      | 15 references                                               |
| **Audio**              | Howler.js + native HTML5 Audio                                       | 6 Howler references                                         |
| **QR codes**           | QR code library (likely `qr-code-styling`)                           | QR generator component in bundle                            |
| **3D**                 | Three.js                                                             | 1 reference (minimal usage)                                 |
| **Backend**            | External REST API                                                    | `https://api.lovearea.in`                                   |
| **Database**           | Not in this repo                                                     | Managed by backend API                                      |
| **Authentication**     | Google OAuth + email auth                                            | `/auth/login`, `/auth/signup`, `/auth/google`               |
| **Payments**           | Cashfree                                                             | `sdk.cashfree.com/js/v3/cashfree.js`                        |
| **Hosting**            | Cloudflare Pages                                                     | Analytics beacon token in `index.html`                      |
| **Package manager**    | Unknown (no `package.json`)                                          | Source repo not present                                     |
| **Testing tools**      | None detected                                                        | No test files                                               |
| **Image optimization** | WebP (partial)                                                       | 36 WebP + 25 JPG + 12 PNG coexist                           |
| **Video handling**     | HTML5 `<video>` + MP4                                                | 3 local MP4 files                                           |
| **Fonts**              | Google Fonts (14+ families) + Minecraft.ttf                          | Loaded via CDN and self-hosted                              |
| **Analytics**          | Google Analytics (G-MJ12JW44KZ) + Cloudflare Web Analytics           | Both in `index.html`                                        |
| **SEO**                | Meta tags, OG tags, Twitter cards                                    | Present in `index.html`                                     |
| **CMS**                | None                                                                 | Content hardcoded in React components                       |
| **APIs**               | `api.lovearea.in`, Cashfree, Google OAuth, Unsplash, Mixkit, Spotify | Referenced in bundle                                        |
| **AI integrations**    | None detected                                                        |                                                             |
| **Origin platform**    | Lovable.dev                                                          | Commented OG image URL in `index.html`                      |


---



# 4. Code Location

> **There is no editable source code in this workspace.** Below maps where functionality lives in the build artifact.


| Category            | Location                                                | Notes                                           |
| ------------------- | ------------------------------------------------------- | ----------------------------------------------- |
| **Pages / Routes**  | Embedded in `assets/index-DZYHQvRn.js`                  | 15 React Router paths (see below)               |
| **Components**      | Minified inside JS bundle                               | All React components compiled into single file  |
| **Hooks**           | Inside JS bundle                                        | Standard React hooks                            |
| **Utilities**       | Inside JS bundle                                        | Axios helpers, formatters, etc.                 |
| **API routes**      | External: `https://api.lovearea.in`                     | Not in this repo                                |
| **Assets**          | `assets/` (81 files) + root (`logo.png`, `Perfect.mp3`) | 386 additional assets referenced but missing    |
| **Config files**    | `index.html` only                                       | No `vite.config`, `tsconfig`, `tailwind.config` |
| **Template shells** | `template/girlfriendday/perfect-girlfriend.html`        | Duplicate of `index.html`                       |
| **Styles**          | `assets/index-B30c12lU.css`                             | 251 KB Tailwind output                          |




### Defined Routes (from bundle analysis)


| Route                                                 | Purpose                           |
| ----------------------------------------------------- | --------------------------------- |
| `/`                                                   | Homepage                          |
| `/about`                                              | About page                        |
| `/blog`, `/blog/:slug`                                | Blog                              |
| `/auth`, `/login`, `/signup`                          | Authentication                    |
| `/cart`, `/checkout`                                  | E-commerce flow                   |
| `/payment/callback`                                   | Cashfree payment return           |
| `/s/:slug`                                            | Published personalized love pages |
| `/template/:occasion`                                 | Template gallery by occasion      |
| `/template/:occasion/:templateId`                     | Specific template editor/viewer   |
| `/template/girlfriendday`, `/template/birthday`, etc. | 10 occasion templates             |
| `/privacy-policy`, `/refund-policy`, `/terms`         | Legal pages                       |




### Template Occasions (10)

`anniversary`, `apology`, `birthday`, `cheerup`, `confession`, `girlfriendday`, `holi`, `proposeday`, `roseday`, `valentine`

---



# 5. File Size Analysis


| Metric                              | Value                            |
| ----------------------------------- | -------------------------------- |
| **Total project size (incl. .git)** | 310 MB                           |
| **Total project size (excl. .git)** | 256.63 MB (269,093,572 bytes)    |
| **Total source code size**          | 12.01 MB (HTML + JS + CSS + MJS) |
| **Total assets size (media)**       | 244.62 MB                        |
| **Total files (excl. .git)**        | 90                               |




### Breakdown by Folder


| Folder                    | Size      | % of Total |
| ------------------------- | --------- | ---------- |
| `assets/`                 | 256.32 MB | 99.9%      |
| `/` (root)                | 0.29 MB   | 0.1%       |
| `assets/minecraft_font/`  | 0.01 MB   | <0.1%      |
| `template/girlfriendday/` | 0.003 MB  | <0.1%      |
| `.well-known/`            | 0.003 MB  | <0.1%      |
| `.git/`                   | 53 MB     | (separate) |




### Breakdown by Category


| Category                   | Files | Size      | %     |
| -------------------------- | ----- | --------- | ----- |
| **Videos**                 | 3     | 239.04 MB | 93.1% |
| **Code (JS/CSS/HTML/MJS)** | 6     | 12.01 MB  | 4.7%  |
| **Images**                 | 75    | 4.92 MB   | 1.9%  |
| **Audio**                  | 3     | 0.62 MB   | 0.2%  |
| **Fonts**                  | 2     | 0.03 MB   | <0.1% |
| **Other**                  | 1     | 136 bytes | <0.1% |




### Largest Folders

1. `assets/` — 256.32 MB (dominated by one video file)
2. `/` (root) — 0.29 MB
3. `assets/minecraft_font/` — 0.01 MB

---



# 6. Largest Files

> Only 90 files exist (excluding `.git`). All files listed below, sorted descending.


| #     | Path                                             | Size                     | Type       | Purpose                                                                 |
| ----- | ------------------------------------------------ | ------------------------ | ---------- | ----------------------------------------------------------------------- |
| 1     | `assets/cherry-blossom-lake-house-minecraft.mp4` | 248,723,998 B (237.2 MB) | MP4 video  | Girlfriend Day background video (4K 60fps) — **GITIGNORED**             |
| 2     | `assets/test_module.mjs`                         | 10,885,080 B (10.4 MB)   | JavaScript | ⚠ Accidental duplicate/corrupted bundle — **NOT REFERENCED, UNTRACKED** |
| 3     | `assets/kiss_video.mp4`                          | 1,881,608 B (1.8 MB)     | MP4 video  | Kiss moment video — **UNUSED** (not in bundle refs)                     |
| 4     | `assets/index-DZYHQvRn.js`                       | 1,451,616 B (1.38 MB)    | JavaScript | Main React SPA production bundle                                        |
| 5     | `assets/mc_book_readymade.jpg`                   | 308,568 B (301 KB)       | JPEG       | Minecraft book texture — **DUPLICATE** of #6                            |
| 6     | `assets/mc_book_final.jpg`                       | 308,568 B (301 KB)       | JPEG       | Minecraft book texture — **DUPLICATE** of #5                            |
| 7     | `assets/couple-CIb7Mo28.gif`                     | 294,943 B (288 KB)       | GIF        | Animated couple illustration                                            |
| 8     | `assets/mc_book_final.png`                       | 287,011 B (280 KB)       | PNG        | Minecraft book texture (PNG variant)                                    |
| 9     | `Perfect.mp3`                                    | 272,091 B (266 KB)       | MP3 audio  | Girlfriend Day template background music                                |
| 10    | `assets/index-B30c12lU.css`                      | 251,505 B (246 KB)       | CSS        | Tailwind production stylesheet                                          |
| 11    | `assets/girlfriend-6K1GTJ07.webp`                | 219,076 B (214 KB)       | WebP       | Girlfriend Day hero image                                               |
| 12    | `assets/moment7.jpg`                             | 199,047 B (194 KB)       | JPEG       | Personal moment photo #7                                                |
| 13    | `assets/DiscSong.mp3`                            | 189,962 B (186 KB)       | MP3 audio  | Minecraft disc song                                                     |
| 14    | `assets/AngelWithNoWings.mp3`                    | 189,962 B (186 KB)       | MP3 audio  | **EXACT DUPLICATE** of DiscSong.mp3                                     |
| 15    | `assets/kiss_image.jpg`                          | 153,002 B (149 KB)       | JPEG       | Kiss illustration photo                                                 |
| 16    | `assets/moment13.jpg`                            | 138,458 B (135 KB)       | JPEG       | Personal moment photo #13                                               |
| 17    | `assets/mc_book_readymade.png`                   | 129,723 B (127 KB)       | PNG        | Minecraft book readymade texture                                        |
| 18    | `assets/pixel_love_jar.png`                      | 115,110 B (112 KB)       | PNG        | Pixel art love jar                                                      |
| 19    | `assets/bg0r-BScjokOq.webp`                      | 107,174 B (105 KB)       | WebP       | Background image (rotated)                                              |
| 20    | `assets/bg0-DTqZNEvh.webp`                       | 104,490 B (102 KB)       | WebP       | Background image                                                        |
| 21    | `assets/pixel_love_bucket.png`                   | 104,099 B (102 KB)       | PNG        | Pixel art love bucket                                                   |
| 22    | `assets/moment10.jpg`                            | 102,068 B (100 KB)       | JPEG       | Personal moment photo #10                                               |
| 23    | `assets/moment15.jpg`                            | 101,564 B (99 KB)        | JPEG       | Personal moment photo #15                                               |
| 24    | `assets/fav_pic.jpg`                             | 101,364 B (99 KB)        | JPEG       | Favorite picture                                                        |
| 25–90 | *(remaining 66 files)*                           | < 99 KB each             | various    | Template icons, backgrounds, moments, UI textures                       |


---



# 7. Asset Inventory


| Category               | Count | Total Size               | Largest File                                              |
| ---------------------- | ----- | ------------------------ | --------------------------------------------------------- |
| **Images (PNG)**       | 12    | 854,956 B (835 KB)       | `mc_book_final.png` (287,011 B)                           |
| **Images (JPG)**       | 25    | 2,600,255 B (2.48 MB)    | `mc_book_readymade.jpg` (308,568 B)                       |
| **Images (WebP)**      | 36    | 1,409,030 B (1.34 MB)    | `girlfriend-6K1GTJ07.webp` (219,076 B)                    |
| **Images (GIF)**       | 1     | 294,943 B (288 KB)       | `couple-CIb7Mo28.gif` (294,943 B)                         |
| **Images (AVIF)**      | 0     | —                        | —                                                         |
| **Images (SVG)**       | 0     | —                        | —                                                         |
| **Videos (MP4)**       | 3     | 250,653,922 B (239.0 MB) | `cherry-blossom-lake-house-minecraft.mp4` (248,723,998 B) |
| **Audio (MP3)**        | 3     | 652,015 B (637 KB)       | `Perfect.mp3` (272,091 B)                                 |
| **Audio (WAV/OGG)**    | 0     | —                        | —                                                         |
| **Fonts (TTF)**        | 2     | 28,976 B (28 KB)         | `Minecraft.ttf` (14,488 B, duplicated)                    |
| **Fonts (WOFF/WOFF2)** | 0     | —                        | —                                                         |
| **JSON**               | 1     | 2,692 B                  | `com.chrome.devtools.json`                                |
| **Lottie**             | 0     | —                        | —                                                         |
| **GLB/GLTF**           | 0     | —                        | —                                                         |
| **PDF**                | 0     | —                        | —                                                         |
| **ICO**                | 1     | 2,692 B                  | `favicon.ico`                                             |
| **Other media**        | 0     | —                        | —                                                         |




### Bundle vs. Local Asset Gap


| Metric                                  | Count   |
| --------------------------------------- | ------- |
| Media paths referenced in JS/CSS bundle | 455     |
| Present locally                         | 69      |
| **Missing locally**                     | **386** |


Missing breakdown: 322 WebP, 30 GIF, 14 MP3, 10 PNG, 8 MP4, 2 JPG

---



# 8. Image Audit


| Format   | Count | Total Size |
| -------- | ----- | ---------- |
| **PNG**  | 12    | 835 KB     |
| **JPG**  | 25    | 2.48 MB    |
| **WebP** | 36    | 1.34 MB    |
| **AVIF** | 0     | —          |
| **SVG**  | 0     | —          |
| **GIF**  | 1     | 288 KB     |




### Oversized Images (>100 KB)


| File                       | Size   | Dimensions | Recommendation                                             |
| -------------------------- | ------ | ---------- | ---------------------------------------------------------- |
| `mc_book_final.png`        | 287 KB | 576×704    | Convert to WebP (~200 KB savings)                          |
| `mc_book_final.jpg`        | 301 KB | 827×1024   | Convert to WebP; remove duplicate                          |
| `mc_book_readymade.jpg`    | 301 KB | 827×1024   | Remove (duplicate of above)                                |
| `mc_book_readymade.png`    | 127 KB | 576×704    | Convert to WebP                                            |
| `girlfriend-6K1GTJ07.webp` | 214 KB | 1536×1024  | Recompress or convert to AVIF (~80 KB savings)             |
| `couple-CIb7Mo28.gif`      | 288 KB | 498×498    | Replace with WebP animation or short MP4 (~200 KB savings) |
| `moment7.jpg`              | 194 KB | 768×1024   | Convert to WebP (~80 KB savings)                           |
| `kiss_image.jpg`           | 149 KB | 1024×1024  | Convert to WebP (~60 KB savings)                           |
| `moment13.jpg`             | 135 KB | 576×1024   | Convert to WebP                                            |
| `pixel_love_jar.png`       | 112 KB | 420×680    | Convert to WebP                                            |
| `pixel_love_bucket.png`    | 102 KB | 357×417    | Convert to WebP                                            |
| `bg0-DTqZNEvh.webp`        | 102 KB | 736×1308   | Consider AVIF (~40 KB savings)                             |
| `bg0r-BScjokOq.webp`       | 105 KB | 1308×736   | Consider AVIF (~40 KB savings)                             |
| `moment10.jpg`             | 100 KB | 576×1024   | Convert to WebP                                            |
| `moment15.jpg`             | 99 KB  | 576×1024   | Convert to WebP                                            |
| `fav_pic.jpg`              | 99 KB  | 1024×683   | Convert to WebP                                            |




### Missing Image

`/heart-icon.webp` is referenced in the QR code component but **does not exist** in the workspace.

---



# 9. Video Audit


| Path                                             | Resolution     | Codec                | Size     | Duration | FPS | Bitrate   | Status                               |
| ------------------------------------------------ | -------------- | -------------------- | -------- | -------- | --- | --------- | ------------------------------------ |
| `assets/cherry-blossom-lake-house-minecraft.mp4` | 3840×2160 (4K) | H.264 High + AAC     | 237.2 MB | 49.3 sec | 60  | 40.3 Mbps | Referenced in bundle; **gitignored** |
| `assets/kiss_video.mp4`                          | 624×832        | H.264 High           | 1.8 MB   | 9.0 sec  | 30  | 1.67 Mbps | **Not referenced — dead asset**      |
| `assets/love-D-moh3Up.mp4`                       | 370×282        | H.264 High (libx264) | 47 KB    | 2.0 sec  | 10  | 193 Kbps  | Referenced in bundle                 |




### Missing Videos (referenced in bundle, not present locally)

8 MP4 files including: `surprise2-BPV19uoT.mp4`, `angry-DnOxkSLj.mp4`, and others for birthday/valentine templates.

### Compression Recommendations


| Video                                     | Current                | Recommended                                        | Est. Savings         |
| ----------------------------------------- | ---------------------- | -------------------------------------------------- | -------------------- |
| `cherry-blossom-lake-house-minecraft.mp4` | 4K 60fps H.264, 237 MB | 1080p 30fps H.265/AV1, ~15 MB                      | **~222 MB (93%)**    |
| `cherry-blossom` (alt)                    | —                      | Host on Cloudflare R2/Stream with adaptive bitrate | Offloads from deploy |
| `kiss_video.mp4`                          | 1.8 MB, 624×832        | Delete (unused) or 480p ~600 KB                    | 1.2 MB               |
| `love-D-moh3Up.mp4`                       | 47 KB                  | Already optimal                                    | —                    |


---



# 10. Dependencies

> **No** `package.json` **exists.** Dependencies inferred from production JS bundle analysis.



### Production Dependencies (Inferred)


| Package                    | Purpose                                      | In Bundle             |
| -------------------------- | -------------------------------------------- | --------------------- |
| `react`                    | UI framework                                 | ✓                     |
| `react-dom`                | DOM rendering                                | ✓                     |
| `react-router-dom`         | Client routing                               | ✓                     |
| `axios`                    | HTTP requests                                | ✓                     |
| `sonner`                   | Toast notifications                          | ✓                     |
| `lucide-react`             | Icons                                        | ✓                     |
| `framer-motion`            | Animations                                   | ✓                     |
| `canvas-confetti`          | Celebration effects                          | ✓                     |
| `howler`                   | Audio playback                               | ✓                     |
| `@radix-ui/react-*`        | UI primitives (dialog, toast, tooltip, etc.) | ✓ (57 refs)           |
| `tailwindcss`              | Utility CSS                                  | ✓ (compiled into CSS) |
| `clsx` / `tailwind-merge`  | Class name utilities                         | Likely                |
| `class-variance-authority` | Component variants                           | Likely                |
| QR code library            | QR generation                                | ✓                     |




### Development Dependencies

**None present** — no source repo, no dev tooling.

### Dependency Analysis


| Check                        | Result                                                                                                                                                   |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Unused packages**          | Cannot determine without source; bundle includes all deps inline                                                                                         |
| **Duplicate packages**       | `test_module.mjs` duplicates the entire app bundle (~7.5× concatenated)                                                                                  |
| **Outdated packages**        | Cannot check without `package.json`                                                                                                                      |
| **Unusually large packages** | Entire bundle is 1.38 MB uncompressed / 400 KB gzipped — large for a landing page due to 10 full template implementations bundled without code splitting |


---



# 11. Build Output


| Asset                                               | Uncompressed           | Gzipped            | Brotli (est.)     |
| --------------------------------------------------- | ---------------------- | ------------------ | ----------------- |
| **JS bundle** (`index-DZYHQvRn.js`)                 | 1,451,616 B (1.38 MB)  | 409,295 B (400 KB) | ~350 KB           |
| **CSS bundle** (`index-B30c12lU.css`)               | 251,505 B (246 KB)     | 37,010 B (36 KB)   | ~30 KB            |
| **Accidental MJS** (`test_module.mjs`)              | 10,885,080 B (10.4 MB) | ~3 MB (est.)       | Should be deleted |
| **Total code transfer (gzip)**                      | —                      | **~436 KB**        | —                 |
| **Total deploy (all files)**                        | 256.63 MB              | —                  | —                 |
| **Total deploy (excl. cherry video + test_module)** | **~17.3 MB**           | —                  | —                 |




### Chunks


| Chunk                | Size    | Notes                                           |
| -------------------- | ------- | ----------------------------------------------- |
| `index-DZYHQvRn.js`  | 1.38 MB | **Single monolithic chunk** — no code splitting |
| `index-B30c12lU.css` | 246 KB  | Single CSS file                                 |
| Dynamic imports      | **0**   | All templates loaded upfront                    |




### Largest Bundles

1. `index-DZYHQvRn.js` — 1.38 MB (contains all 10 template implementations)
2. `index-B30c12lU.css` — 246 KB (full Tailwind output)
3. `test_module.mjs` — 10.4 MB (accidental, should not exist)

---



# 12. Performance Audit


| Issue                         | Severity    | Details                                                                   |
| ----------------------------- | ----------- | ------------------------------------------------------------------------- |
| **Monolithic JS bundle**      | 🔴 High     | 1.38 MB single chunk with all 10 templates; no lazy loading               |
| **Missing 386 assets**        | 🔴 High     | 85% of referenced media not in local deploy; broken templates locally     |
| **238 MB background video**   | 🔴 Critical | 4K 60fps video loaded as background; blocks page on slow connections      |
| **10.4 MB accidental file**   | 🔴 High     | `test_module.mjs` is untracked duplicate bundle                           |
| **14+ Google Font families**  | 🟡 Medium   | Render-blocking font loads from multiple CSS URLs                         |
| **No code splitting**         | 🟡 Medium   | 0 dynamic imports detected                                                |
| **Duplicate files**           | 🟡 Medium   | 7 MD5-identical file pairs wasting ~1.2 MB                                |
| **GIF instead of video/WebP** | 🟡 Medium   | 288 KB GIF could be ~50 KB WebP animation                                 |
| **JPG/PNG alongside WebP**    | 🟡 Medium   | 37 non-WebP images that could be WebP-only                                |
| **Missing heart-icon.webp**   | 🟡 Medium   | QR code component references missing file                                 |
| **Audio autoplay**            | 🟡 Medium   | `Perfect.mp3` autoplays on girlfriendday template (browser policy issues) |
| **Unnecessary re-renders**    | ⚪ Unknown   | Cannot audit without source code                                          |
| **Dead code in bundle**       | 🟡 Medium   | Likely unused Radix components bundled but not tree-shaken                |




### Unused Assets (present but not referenced in bundle)


| File                                       | Size    | Notes                              |
| ------------------------------------------ | ------- | ---------------------------------- |
| `assets/test_module.mjs`                   | 10.4 MB | Accidental duplicate               |
| `assets/kiss_video.mp4`                    | 1.8 MB  | Not referenced                     |
| `assets/AngelWithNoWings.mp3`              | 186 KB  | Duplicate of DiscSong.mp3          |
| `assets/mc_book_final.jpg`                 | 301 KB  | Duplicate of mc_book_readymade.jpg |
| `assets/mc_book_gui.png`                   | 31 KB   | Not referenced                     |
| `assets/mc_paper_raw.png`                  | 10 KB   | Not referenced                     |
| `assets/minecraft_cats_us.jpg`             | 59 KB   | Triplicate (3 identical files)     |
| `assets/minecraft_us_cats.jpg`             | 59 KB   | Triplicate                         |
| `assets/final_couple_art.jpg`              | 59 KB   | Triplicate                         |
| `assets/pixel_love_jar.jpg`                | 27 KB   | PNG version exists                 |
| `assets/mc_book_readymade.jpg/png`         | 436 KB  | Superseded by other versions       |
| `.well-known/.../com.chrome.devtools.json` | 2.7 KB  | Dev artifact                       |


**Total unused/duplicate local assets: ~13.2 MB** (excluding cherry video)

---



# 13. Storage Analysis


| Category                   | Bytes           | MB         | % of Total |
| -------------------------- | --------------- | ---------- | ---------- |
| **Videos**                 | 250,653,922     | 239.04     | **93.1%**  |
| **Code (JS/CSS/HTML/MJS)** | 12,596,647      | 12.01      | 4.7%       |
| **Images (all formats)**   | 5,161,876       | 4.92       | 1.9%       |
| **Audio**                  | 652,015         | 0.62       | 0.2%       |
| **Fonts**                  | 28,976          | 0.03       | <0.1%      |
| **Other**                  | 136             | 0.00       | <0.1%      |
| **Total**                  | **269,093,572** | **256.63** | **100%**   |




### Visual Breakdown

```
Videos     ████████████████████████████████████████████████  93.1%
Code       ██                                               4.7%
Images     █                                                1.9%
Audio      ▏                                                0.2%
Fonts      ▏                                                <0.1%
```



### Excluding the 237 MB Cherry Blossom Video


| Category     | MB           | %     |
| ------------ | ------------ | ----- |
| Code         | 12.01        | 69.4% |
| Images       | 4.92         | 28.4% |
| Audio        | 0.62         | 3.6%  |
| Other videos | 1.89         | 10.9% |
| Fonts        | 0.03         | 0.2%  |
| **Total**    | **~17.3 MB** | 100%  |


---



# 14. Deployment Readiness


| Platform             | Suitable?                | Notes                                                                                  |
| -------------------- | ------------------------ | -------------------------------------------------------------------------------------- |
| **Cloudflare Pages** | ✅ **Currently deployed** | Analytics beacon present; ideal for static SPA + API proxy                             |
| **Netlify**          | ✅ Yes                    | Static files; add `_redirects` for SPA routing (`/* /index.html 200`)                  |
| **Vercel**           | ✅ Yes                    | Static export; add `vercel.json` rewrites for SPA                                      |
| **GitHub Pages**     | ⚠️ Partial               | Works for static files; 100 MB file size limit blocks cherry video; no server-side API |
| **Docker**           | ✅ Yes                    | Simple nginx container serving static files; API remains external                      |
| **VPS**              | ✅ Yes                    | `server.py` exists in parent directory for local dev with API proxy                    |




### Deployment Issues

1. **No SPA fallback config** — Missing `_redirects` / `vercel.json` / `_routes.json` for client-side routing
2. **238 MB video exceeds GitHub's 100 MB limit** — Already gitignored (correct)
3. **386 missing assets** — Local deploy incomplete; production likely has more assets on CDN
4. **No build step in repo** — Cannot rebuild from this artifact; need source repo
5. **Modified JS not committed** — `assets/index-DZYHQvRn.js` has uncommitted changes
6. **Untracked 10.4 MB file** — `test_module.mjs` would bloat any deployment if included
7. **No environment variable config** — API URL hardcoded as `https://api.lovearea.in`

---



# 15. Git Repository Health


| Metric                      | Value                                     |
| --------------------------- | ----------------------------------------- |
| **Repository size (.git/)** | 53 MB                                     |
| **Estimated clone size**    | ~52 MB (git count-objects: 51.97 MiB)     |
| **Total blob objects**      | 85                                        |
| **Total blob size**         | 56.3 MB                                   |
| **Tracked files**           | 88                                        |
| **Tracked content size**    | 9.05 MB                                   |
| **Branches**                | `main`, `backup-pre-revert`               |
| **Commits**                 | 1 ("Initial commit: lovearea.in website") |
| **Uncommitted changes**     | Modified `index-DZYHQvRn.js`              |
| **Untracked files**         | `assets/test_module.mjs`                  |




### .gitignore

```
assets/cherry-blossom-lake-house-minecraft.mp4
assets/pink-clouds-train.mp4
```



### Large Objects in Git History


| Object                                    | Size                      | Status                                              |
| ----------------------------------------- | ------------------------- | --------------------------------------------------- |
| `cherry-blossom-lake-house-minecraft.mp4` | 23.9 MB (compressed blob) | Gitignored in working tree but may exist in history |
| `pink-clouds-train.mp4`                   | 15.1 MB                   | Removed from tree, **still in git history**         |
| `index-DZYHQvRn.js` (older version)       | 10.9 MB                   | Previous bundle version in history                  |
| `kiss_video.mp4`                          | 1.8 MB                    | Tracked                                             |




### Recommendations


| Action                                      | Priority                                                             |
| ------------------------------------------- | -------------------------------------------------------------------- |
| Large assets should be excluded from git    | ✅ Already gitignoring cherry video                                   |
| Git LFS needed?                             | **Yes, if videos must be versioned** — otherwise use external CDN/R2 |
| Purge `pink-clouds-train.mp4` from history  | Recommended — saves 15 MB clone size                                 |
| Purge old 10.9 MB JS bundle from history    | Recommended                                                          |
| Remove `backup-pre-revert` branch if unused | Optional                                                             |
| Do NOT commit `test_module.mjs`             | Critical                                                             |


---



# 16. Optimization Opportunities — TOP 50


| #   | Optimization                                                              | Est. Savings                       | Priority    |
| --- | ------------------------------------------------------------------------- | ---------------------------------- | ----------- |
| 1   | Re-host `cherry-blossom-lake-house-minecraft.mp4` to Cloudflare R2/Stream | **237 MB**                         | 🔴 Critical |
| 2   | Re-encode cherry video to 1080p30 H.265/AV1 (~15 MB)                      | **222 MB**                         | 🔴 Critical |
| 3   | Delete `test_module.mjs` (accidental 7× concatenated bundle)              | **10.4 MB**                        | 🔴 Critical |
| 4   | Purge `pink-clouds-train.mp4` from git history                            | **15 MB** (clone)                  | 🔴 High     |
| 5   | Implement route-based code splitting (lazy load templates)                | **~800 KB** initial JS             | 🔴 High     |
| 6   | Delete unused `kiss_video.mp4`                                            | **1.8 MB**                         | 🟡 Medium   |
| 7   | Remove duplicate `mc_book_final.jpg` (= mc_book_readymade.jpg)            | **301 KB**                         | 🟡 Medium   |
| 8   | Remove duplicate `AngelWithNoWings.mp3` (= DiscSong.mp3)                  | **186 KB**                         | 🟡 Medium   |
| 9   | Remove triplicate cat images (keep 1 of 3)                                | **120 KB**                         | 🟡 Medium   |
| 10  | Remove duplicate `Minecraft.ttf` (keep one copy)                          | **14 KB**                          | 🟢 Low      |
| 11  | Remove duplicate `mc_button_bg.png` (= mc_pink_wood_bg.jpg)               | **12 KB**                          | 🟢 Low      |
| 12  | Convert `mc_book_final.png` (287 KB) to WebP                              | **~87 KB**                         | 🟡 Medium   |
| 13  | Convert 15 `moment*.jpg` photos to WebP                                   | **~600 KB**                        | 🟡 Medium   |
| 14  | Convert `kiss_image.jpg` to WebP                                          | **~60 KB**                         | 🟡 Medium   |
| 15  | Convert `fav_pic.jpg` to WebP                                             | **~40 KB**                         | 🟡 Medium   |
| 16  | Convert `pixel_love_jar.png` to WebP                                      | **~50 KB**                         | 🟡 Medium   |
| 17  | Convert `pixel_love_bucket.png` to WebP                                   | **~45 KB**                         | 🟡 Medium   |
| 18  | Convert `gf_award_certificate.png` to WebP                                | **~35 KB**                         | 🟡 Medium   |
| 19  | Replace `couple-CIb7Mo28.gif` with WebP/video                             | **~200 KB**                        | 🟡 Medium   |
| 20  | Remove unused `mc_book_gui.png`, `mc_paper_raw.png`                       | **~41 KB**                         | 🟢 Low      |
| 21  | Remove unused `pixel_love_jar.jpg` (PNG exists)                           | **~27 KB**                         | 🟢 Low      |
| 22  | Remove `logo.png` (duplicate MD5 of favicon; use logo.webp)               | **~3 KB**                          | 🟢 Low      |
| 23  | Delete `.well-known/appspecific/com.chrome.devtools.json`                 | **~3 KB**                          | 🟢 Low      |
| 24  | Recompress `girlfriend-6K1GTJ07.webp` to AVIF                             | **~80 KB**                         | 🟡 Medium   |
| 25  | Recompress `bg0/bg0r` WebP backgrounds to AVIF                            | **~60 KB**                         | 🟡 Medium   |
| 26  | Tree-shake unused Radix UI components                                     | **~50 KB** JS                      | 🟡 Medium   |
| 27  | Purge old JS bundle versions from git history                             | **~10 MB** (clone)                 | 🟡 Medium   |
| 28  | Consolidate to WebP-only image pipeline                                   | **~1.5 MB**                        | 🟡 Medium   |
| 29  | Self-host 2–3 critical fonts instead of 14+ Google Font URLs              | 0 KB (perf gain)                   | 🟡 Medium   |
| 30  | Enable Brotli compression on CDN                                          | 0 KB stored; ~15% transfer savings | 🟡 Medium   |
| 31  | Add responsive `srcset` for moment photos                                 | 0 KB stored; mobile perf gain      | 🟡 Medium   |
| 32  | Lazy-load below-fold images                                               | 0 KB stored; perf gain             | 🟡 Medium   |
| 33  | Preload only critical above-fold assets                                   | 0 KB stored; perf gain             | 🟡 Medium   |
| 34  | Add `loading="lazy"` to non-hero images                                   | 0 KB stored; perf gain             | 🟡 Medium   |
| 35  | Use video poster frames instead of autoplay 4K                            | 0 KB stored; UX gain               | 🟡 Medium   |
| 36  | Reduce `kiss_video.mp4` to 480p if kept                                   | **~1.2 MB**                        | 🟢 Low      |
| 37  | Create missing `heart-icon.webp`                                          | 0 KB (fixes broken QR)             | 🟡 Medium   |
| 38  | Sync/deploy all 386 missing bundle assets                                 | Fixes broken templates             | 🔴 Critical |
| 39  | Recover source repo from Lovable.dev                                      | Enables proper development         | 🔴 Critical |
| 40  | Add `_redirects` for SPA routing on Cloudflare                            | 0 KB; fixes 404 on refresh         | 🟡 Medium   |
| 41  | Remove commented lovable.dev OG image refs                                | ~0 KB                              | 🟢 Low      |
| 42  | Minify `perfect-girlfriend.html` template shell                           | **~0.5 KB**                        | 🟢 Low      |
| 43  | Add cache headers for static assets (1 year)                              | 0 KB; perf gain                    | 🟡 Medium   |
| 44  | Use Git LFS for any remaining large media                                 | Prevents future bloat              | 🟡 Medium   |
| 45  | Remove `backup-pre-revert` branch                                         | **~0 MB**                          | 🟢 Low      |
| 46  | Deduplicate mc_book JPG/PNG pairs (keep WebP only)                        | **~400 KB**                        | 🟡 Medium   |
| 47  | Convert `love-D-moh3Up.mp4` to WebM                                       | **~20 KB**                         | 🟢 Low      |
| 48  | Audit and remove unused CSS (PurgeCSS on rebuild)                         | **~50 KB** CSS                     | 🟡 Medium   |
| 49  | Split vendor chunk (React, Radix) from app code                           | Better caching                     | 🟡 Medium   |
| 50  | Move all audio (MP3) to CDN with streaming                                | **~650 KB** from deploy            | 🟢 Low      |


**Total potential savings: ~700 MB** (dominated by video re-hosting/re-encoding)

---



# 17. Final Recommendation



### Is Netlify appropriate?

**Cloudflare Pages is the better fit** (already deployed). Netlify would also work for the static SPA, but Cloudflare integrates better with R2 for video hosting and provides the analytics already configured. If switching to Netlify, add `_redirects` for SPA routing and host videos externally.

### Should assets remain inside the repository?

**No.** Only code bundles and small icons (<50 KB) should be in git. Specifically:


| Asset Type             | Keep in Repo? | Alternative                        |
| ---------------------- | ------------- | ---------------------------------- |
| JS/CSS bundles         | ✅ Yes         | —                                  |
| WebP icons (<50 KB)    | ✅ Yes         | —                                  |
| Videos (any size)      | ❌ No          | Cloudflare R2 / Stream / Bunny CDN |
| Audio (MP3)            | ❌ No          | CDN or R2                          |
| Large photos (>100 KB) | ❌ No          | CDN with WebP/AVIF variants        |
| Fonts                  | ⚠️ Optional   | Self-host or Google Fonts CDN      |




### Which folders should move to external storage?

1. `assets/*.mp4` → Cloudflare R2 (all videos, especially cherry-blossom)
2. `assets/*.mp3` + `Perfect.mp3` → R2 or CDN
3. `assets/moment*.jpg` → R2 with WebP variants
4. **Missing 386 assets** → Must be deployed to CDN for templates to work



### Which files should be compressed?


| Priority | Files                                      | Action                                    |
| -------- | ------------------------------------------ | ----------------------------------------- |
| 🔴 P0    | `cherry-blossom-lake-house-minecraft.mp4`  | Re-encode to 1080p30 → ~15 MB; host on R2 |
| 🔴 P0    | `test_module.mjs`                          | **Delete immediately**                    |
| 🟡 P1    | 15 `moment*.jpg`                           | Convert to WebP                           |
| 🟡 P1    | `mc_book_final.png`, `couple-CIb7Mo28.gif` | Convert to WebP/video                     |
| 🟡 P1    | `index-DZYHQvRn.js`                        | Code-split by template route              |
| 🟢 P2    | All duplicate files                        | Delete duplicates                         |
| 🟢 P2    | `bg0/bg0r` WebP backgrounds                | Convert to AVIF                           |




### Expected Deployment Size After Optimization


| Scenario                                      | Deploy Size                        | Git Clone Size |
| --------------------------------------------- | ---------------------------------- | -------------- |
| **Current (with cherry video)**               | 256.63 MB                          | 53 MB          |
| **Remove test_module + duplicates + unused**  | ~242 MB                            | 53 MB          |
| **Re-encode cherry video to 1080p30**         | ~20 MB                             | 53 MB          |
| **Move all videos to R2**                     | ~17.3 MB                           | ~9 MB          |
| **Full optimization (WebP, code split, CDN)** | **~8–12 MB**                       | **~5 MB**      |
| **Gzipped transfer per page load**            | **~500 KB** (JS+CSS) + lazy assets | —              |




### Summary Verdict

LoveArea.in is a **functional romantic page builder SaaS** that is live on Cloudflare Pages with a separate API backend. However, **this workspace is an incomplete production build artifact**, not a maintainable codebase. The single biggest issue is a **237 MB 4K background video** consuming 93% of storage. Immediate actions:

1. **Delete** `assets/test_module.mjs` (10.4 MB waste)
2. **Move videos to Cloudflare R2** and re-encode cherry-blossom to 1080p
3. **Recover the source repository** from Lovable.dev for ongoing development
4. **Deploy the 386 missing assets** or verify they exist on the live CDN
5. **Add SPA routing config** (`_redirects` or `_routes.json`)
6. **Implement code splitting** when source is available
7. **Remove all duplicate files** (~1.2 MB immediate savings)

---

*Report generated by automated workspace inspection. All sizes measured directly from filesystem (August 1, 2026).*