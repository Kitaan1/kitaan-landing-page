# Kitaan Coaching Landing Page

A single-page fitness landing page for a powerlifting and physique coaching offer. Goal: get visitors to watch the VSL video and click APPLY NOW, which opens a short application questionnaire that filters out unqualified leads. Built for a solo coach, no team.

## The one rule: keep it simple and fast
This is deliberately ONE static HTML file with no framework, no build step, and no dependencies to install. Do not add React, a bundler, npm, or a CSS framework. If a change can be done in plain HTML, CSS, or vanilla JS, do it that way. Simpler and faster always wins here.

Style note for anyone (or any AI) editing the docs or copy: do not use em dashes anywhere. Use commas, full stops, or the word "and" instead.

## Files
- `index.html` is the entire page. All CSS is in the `<style>` block in the head, all JS is in the `<script>` block at the bottom. Edit this one file for almost everything.
- `Assets/transformation1-4.webp` are the before/after client photos actually used on the page.
- `Assets/transformation1-4.png` are the ORIGINAL photos, no longer used on the page. Kept as backup only. Safe to ignore or delete.
- `Assets/strength1-27.mp4` are client strength/PR clips, portrait phone videos (720x1280).
- `Assets/posters/strength1-27.jpg` are freeze-frame thumbnails shown before each video loads. One per video.
- `Assets/youtube_thumb.jpg` is the thumbnail shown for the main VSL video before it is clicked.

## Page structure (top to bottom)
1. Headline (the offer) plus a sub-headline underneath
2. VSL video (YouTube, click-to-play)
3. APPLY NOW button plus a reassurance line
4. Social proof intro line
5. Client transformation photos (4, WebP)
6. Strength videos (8 shown, 19 more behind a "Show all 27" button)
7. Second APPLY NOW button plus a reassurance line
8. Sticky APPLY NOW bar (mobile only, pinned to the bottom of the screen)
9. Application questionnaire (popup, opened by every APPLY NOW button)

## How the tricky bits work
- The VSL video is click-to-play on purpose. It shows `youtube_thumb.jpg` with a play button, and the real YouTube player only loads when clicked (the `loadVSL()` function). It is still a normal YouTube embed, just loaded on demand. Keep this pattern.
- APPLY NOW opens the application questionnaire as a popup (a native `<dialog id="quiz">`, opened by `openQuiz()`). Every apply button uses this.
- Questions, in order: goal, how serious, biggest struggle, name, email, phone. Choosing "Something else" for the goal or "Just exploring my options" for seriousness ends on a polite not a fit screen and collects no contact details. These options carry `data-dq="1"` in the HTML. To change what disqualifies someone, add or remove that attribute.
- Answers are sent as a text/plain JSON POST (no-cors, so success cannot be read back) to `FORM_ENDPOINT` at the top of the quiz code in the script block. After a successful submit the questionnaire closes and Calendly opens as a popup (`openCalendly()`, name and email prefilled) so qualified leads can book a time slot. The Calendly link is `CALENDLY_URL` in the script. It MUST be set (the Google Apps Script URL that writes into the Leads Google Sheet) or submissions show an error and nothing is saved.
- Videos lazy-load. They only download when you scroll near them (the IntersectionObserver at the bottom of the script). Until then they show their poster image. Keep `preload="none"` on the `<video>` tags.

## Optimizations already applied (what was done, why, and how to redo it)
These were done deliberately to make the page load fast on mobile without losing quality. If you add new media, apply the same steps so the page stays fast.

### 1. YouTube video: click-to-play facade
- Problem: a normal embedded YouTube player downloads roughly 1.3MB of scripts on page load, before anyone even presses play. On mobile that is a major slowdown, and slow pages lose bookings.
- What was done: instead of embedding the live player, the page shows a static image of the video (`Assets/youtube_thumb.jpg`) with a red play button drawn in CSS. The real YouTube iframe is only injected into the page when the visitor clicks it, via the `loadVSL()` function. It is still a genuine YouTube embed.
- How the thumbnail was made: downloaded from YouTube's own thumbnail URL for the video.
  `curl -o Assets/youtube_thumb.jpg "https://img.youtube.com/vi/u7myBLOnoXY/maxresdefault.jpg"`
  (Replace `u7myBLOnoXY` with the new video ID if the video changes. If `maxresdefault.jpg` does not exist for a video, use `hqdefault.jpg` instead.)

### 2. Photos: converted from PNG to WebP
- Problem: the original PNG photos were 730KB to 920KB each. That is heavy, especially on mobile data.
- What was done: converted each to WebP at quality 82, which dropped them to roughly 24KB to 45KB each with no visible quality loss. That is about a 95% size reduction. The page references the `.webp` files. The original `.png` files are kept in `Assets/` as a backup but are not used.
- How to redo it for a new photo (needs Python with Pillow installed, `pip install pillow`):
  `python -c "from PIL import Image; Image.open('Assets/NAME.png').convert('RGB').save('Assets/NAME.webp','WEBP',quality=82,method=6)"`
- Then reference the `.webp` file in the `<img>` tag, not the `.png`.

### 3. Videos: generated poster thumbnails
- Problem: with `preload="none"` and no poster image, each video showed as a black box until you scrolled to it and it loaded. Black boxes look broken and hurt trust.
- What was done: generated a small freeze-frame image for each video (a single frame grabbed about 0.5 seconds in, scaled to 360px wide, roughly 20KB to 45KB each) and set it as the video's `poster`. The video itself still does not download until scrolled near, so this fixes the look without adding load.
- How to redo it for a new video (needs ffmpeg installed):
  `ffmpeg -ss 0.5 -i Assets/strengthN.mp4 -frames:v 1 -vf "scale=360:-2" -q:v 4 Assets/posters/strengthN.jpg`
  (If a clip is shorter than 0.5s, drop the `-ss 0.5` to grab the very first frame instead.)

### 4. Videos: lazy loading
- What was done: videos start with `preload="none"` and no real `src`. The real file path is stored in a `data-src` attribute. An IntersectionObserver (at the bottom of the script) swaps `data-src` into `src` only when the video scrolls near the viewport. So on page load, zero video bytes are downloaded. Keep this pattern for any new video: give it `class="lazy-video"`, `preload="none"`, a `poster`, and a `data-src` (not a `src`).

### 5. Only 8 of 27 videos shown by default
- Too many videos in one grid buries the best ones and adds scroll. The page shows the first 8, with the other 19 inside `<div id="moreVideos">` (hidden by CSS) revealed by a "Show all 27 videos" button. To change which videos show first, reorder the `<video>` lines. The 8 outside the div show, the ones inside it are hidden until the button is clicked.

### 6. Layout stability (no jumping while loading)
- Every video, photo, and the video box has a fixed `aspect-ratio` in CSS (9/16 for the portrait media, 16/9 for the YouTube box). This reserves the right amount of space before the media loads, so the page does not jump around as things come in. Keep an `aspect-ratio` on any new media.

### 7. Mobile usability
- The APPLY NOW buttons are large with generous padding so they are easy to tap. A sticky APPLY NOW bar appears only on screens 640px wide or less (see the `@media (max-width: 640px)` block) so the call to action is always reachable on the long mobile scroll. The headline uses `clamp()` so it scales down cleanly on small screens instead of overflowing.

## Common edits, how to do them
- Change the questions or answer options: edit the `.quiz-step` blocks inside `<dialog id="quiz">`. If you add or remove a step, update the `QUIZ_STEPS` list in the script too.
- Change the Calendly link: edit `CALENDLY_URL` in the script.
- Change where applications go: set `FORM_ENDPOINT` in the script.
- Change the VSL video: replace `u7myBLOnoXY` (the YouTube video ID) in the `loadVSL()` function, and regenerate `Assets/youtube_thumb.jpg` (see optimization 1 above).
- Change the headline colours: edit the `.green` and `.red` spans inside the `<h1>`.
- Add or remove a strength video: copy an existing `<video>` line and change the number, then generate its poster (see optimization 3). The first 8 show by default, the rest go inside `<div id="moreVideos">`.
- Add photo captions: add a caption element under each `<img>` in the image grid. Specific numbers (timeframe, kg gained) are the most persuasive.
- Add a new photo: convert it to WebP first (see optimization 2), then reference the `.webp`.

## Testing it locally
Do NOT just double-click `index.html`. YouTube and the form submission break when the page is opened as a `file://` page. Run a local server instead:
`python -m http.server 8000`
then open `http://localhost:8000/index.html`.
Once hosted on a real domain, everything works normally and the URL is just the clean domain with no `/index.html`.

## Hosting
Any static host works and most are free (Cloudflare Pages, Netlify, GitHub Pages). Upload the folder as-is. The file must stay named `index.html` so the domain root loads it automatically. The page needs a live internet connection for the YouTube video, the form endpoint and the Inter font (loaded from Google Fonts).
