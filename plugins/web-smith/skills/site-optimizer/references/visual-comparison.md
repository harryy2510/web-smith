# Visual Comparison (for clone projects)

When comparing against an original site, use element-level screenshot diffs for pixel-perfect feedback.

## Step 1: Element-level screenshots

```bash
# Original
shot-scraper "https://ORIGINAL.com" -s "nav" -o compare/original-nav.png
shot-scraper "https://ORIGINAL.com" -s "main > section:first-child" -o compare/original-hero.png
shot-scraper "https://ORIGINAL.com" -s "footer" -o compare/original-footer.png
shot-scraper "https://ORIGINAL.com" -o compare/original-full.png --full-page --width 1280

# Clone
shot-scraper "http://localhost:UNIQUE_PORT" -s "nav" -o compare/clone-nav.png
shot-scraper "http://localhost:UNIQUE_PORT" -s "main > section:first-child" -o compare/clone-hero.png
shot-scraper "http://localhost:UNIQUE_PORT" -s "footer" -o compare/clone-footer.png
shot-scraper "http://localhost:UNIQUE_PORT" -o compare/clone-full.png --full-page --width 1280
```

## Step 2: Quick diff score per section

```bash
odiff compare/original-nav.png compare/clone-nav.png compare/diff-nav.png --parsable-stdout
odiff compare/original-hero.png compare/clone-hero.png compare/diff-hero.png --parsable-stdout
odiff compare/original-footer.png compare/clone-footer.png compare/diff-footer.png --parsable-stdout
odiff compare/original-full.png compare/clone-full.png compare/diff-full.png --parsable-stdout
# Output: diffPixels;diffPercentage  (e.g., "2400;2.00")
```

If any section > 2% diff, fix it. Read the diff PNG to see exactly what's wrong.

## Step 3: Extract specific color differences

```bash
magick compare/original-hero.png -crop 1x1+200+165 txt:
# Output: 0,0: (238,0,0,255) #EE0000FF

magick compare/clone-hero.png -crop 1x1+200+165 txt:
# Output: 0,0: (255,0,0,255) #FF0000FF
```

## Step 4: Read diff images visually

Read diff PNGs with the Read tool. Produce specific feedback:
- "Nav height is 4px taller than original"
- "Hero heading font-weight is 600 but should be 700"
- "Button background is #FF0000 but should be #EE0000"

## Step 5: Fix and re-diff

Repeat until all sections < 1% diff.

**Fonts will cause differences.** If using different fonts, focus on layout, spacing, colors, and structure instead of text rendering.
