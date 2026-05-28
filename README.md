# Close Quarters

Cinematic one-page site for a reality-based self-defense brand built around two instructors — white and blue. Framed-parallax video hero with the headline and UI layered in front, plus a pinned scroll-scrub "method" section where the clip seeks frame-by-frame and slow-zooms while three principles cross-fade.

Single zero-build HTML file. No bundler, no framework. Drop it on any static host.

## Structure

```
close-quarters/
├── index.html          # the whole site (HTML + CSS + JS inline)
├── assets/
│   ├── hero.mp4         # 1280px, h264, dense keyframes (scrub-friendly), ~1 MB
│   ├── hero.webm        # VP9 fallback
│   └── poster.jpg       # first-frame poster (also reused for instructor cards)
├── _headers            # Cloudflare Pages cache rules
└── .gitignore
```

## Local preview

Browsers behave best serving video over HTTP rather than `file://`, so run a tiny static server from the repo root:

```
python3 -m http.server 8080
# then open http://localhost:8080
```

## Deploy to Cloudflare Pages

1. Push this folder to a new repo in the KLINEKRAFT GitHub org (drag-and-drop in the GitHub web UI is fine — there's nothing to build).
2. Cloudflare dashboard → Workers & Pages → Create → Pages → Connect to Git → pick the repo.
3. Build settings:
   - Framework preset: **None**
   - Build command: **(leave empty)**
   - Build output directory: **/** (repo root)
4. Save and Deploy. Every push to the main branch redeploys automatically.

## Customizing

- **Brand name** — find/replace `CLOSE QUARTERS` (and the legal line in the footer).
- **The video** — replace the three files in `assets/`. To regenerate from a new master clip named `source.mp4`:

  ```
  # scrub-friendly mp4 (frequent keyframes so scroll-seeking is smooth)
  ffmpeg -i source.mp4 -an -vf "scale=1280:-2" -c:v libx264 -profile:v high \
    -pix_fmt yuv420p -crf 30 -preset slow -g 8 -keyint_min 8 -sc_threshold 0 \
    -movflags +faststart assets/hero.mp4

  # webm fallback
  ffmpeg -i source.mp4 -an -vf "scale=1280:-2" -c:v libvpx-vp9 -crf 34 -b:v 0 -row-mt 1 assets/hero.webm

  # poster (grab a strong frame; -ss is the timestamp in seconds)
  ffmpeg -ss 2 -i source.mp4 -frames:v 1 -vf "scale=1280:-2" -q:v 3 assets/poster.jpg
  ```

  The instructor cards reuse `poster.jpg` and crop left (white) / right (blue) via `background-position` in CSS — adjust `22% center` / `80% center` to fit a different frame, or point them at real headshots.
- **Copy** — instructor bios (and Michael's last name), the three method principles, programs, stat numbers, and the contact line / booking buttons are all placeholders.
- **Scrub length** — the pinned section is set to `end:'+=2600'` in `index.html`. Increase it for a slower, more deliberate read.
- **KLINEKRAFT mark** — the footer shows a text placeholder. Swap in `klinekraft_logo_white.png` (white wordmark for the dark footer); there's a commented `<img>` line right next to it.

## Notes

- Motion runs on GSAP + ScrollTrigger (CDN). If GSAP is ever blocked, the page degrades gracefully — the method section becomes a normal looping section and reveals fall back to an IntersectionObserver.
- Honors `prefers-reduced-motion`: parallax, scrubbing, and tilt are disabled and the section reads as plain stacked content.
- No emojis anywhere, per house style.
