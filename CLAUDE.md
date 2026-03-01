# Battle of the Books — CLAUDE.md

## Project
Single `index.html` site celebrating Commander Joe's Battle of the Books reading.
Hosted on GitHub Pages at `https://kyleosullivan.github.io/battle-books`.
No build steps, no npm, no frameworks — everything is embedded in one file.

## Stack
- HTML/CSS/JS only, all in `index.html`
- Google Fonts CDN: Orbitron (headers) + Nunito (body)
- Google Books API (keyless): covers and descriptions fetched at runtime

## Deployment
```bash
git add index.html
git commit -m "..."
git push  # deploys to GitHub Pages automatically
```
Remote: `git@github.com:kyleosullivan/battle-books.git`

## Key Stats (current)
- 18 books, 3,203 pages, ~640,600 words, ~80 hrs reading time
- Comparisons: 457 YouTube videos, 52 ISS orbits, 40 movies

## Book Data
Books are defined in the `const books = [...]` array near the top of the `<script>` block.
Each entry has: `title`, `author`, `genre`, `pages`, `series`, `isbn`, `coverUrl`, `descCache`.

Genre strings use ` / ` as separator (e.g. `'Fantasy / Adventure'`). The genre planets
use keyword matching against these strings — keep them consistent.

## Genre Planets
Defined in `const genres = [...]`. Each entry has `name`, `count`, `color`, `emoji`, `keywords`.
- `count` determines planet size (used for display only)
- `keywords` is what the click panel uses to match books — must match substrings of book `genre` fields
- Update both `count` and `keywords` when adding/changing books

## Google Books API
```js
// Fetch by ISBN (keyless, ~100 req/day per IP)
https://www.googleapis.com/books/v1/volumes?q=isbn:{ISBN}

// Build cover URL from returned volume ID — do NOT use imageLinks directly
// (zoom=0 is invalid and breaks images)
https://books.google.com/books/content?id={volumeId}&printsec=frontcover&img=1&zoom=1
```
Cover URLs and descriptions are prefetched at page load via `prefetchBookData()`,
stored on the book object (`coverUrl`, `descCache`), then the grid is updated in place.

## Architecture Notes

### Section reveals
Uses `IntersectionObserver` with `threshold: 0, rootMargin: '0px 0px -60px 0px'`.
**Do not increase threshold** — tall sections on portrait mobile will never reach a higher
threshold and will stay permanently hidden (opacity: 0).

### Book grid rendering
`renderBooks()` is called immediately on page load (before prefetch) so cards always
appear. After `prefetchBookData()` resolves, cover `<img>` elements are updated in place.
Never make `renderBooks()` wait on a network request.

### Stat counters
Animated via `requestAnimationFrame` + easeOut when `#stats` enters viewport.
`data-target` and `data-suffix` attributes on `.stat-number` elements drive the animation.
When updating stats, change both the HTML `data-target` values AND the comparison card text.

### Modal
Opens on book card click. Description comes from `book.descCache` (populated by prefetch).
Falls back to `"No description available — but it's a great read!"`.

## Common Update Tasks

### Adding/changing books
1. Update `const books` array
2. Update `const genres` array (counts + keywords)
3. Recalculate and update stats: pages, words (×200), hours (pages÷40), movies (hours÷2), ISS orbits (hours×60÷92), YouTube videos (words÷1400)
4. Update hero teaser chips, stat `data-target` values, comparison card text, mission complete copy and badges

### Changing a comparison stat
Update in 3 places: hero teaser chip, comparison card (both emoji AND text), mission complete badge.

## Lessons Learned
- **OpenLibrary** has poor coverage for newer/smaller-press children's books — use Google Books
- **zoom=0** is not a valid Google Books image zoom level — always use `zoom=1`
- **IntersectionObserver threshold** must stay near 0 for very tall sections on mobile
- The stat number font uses `clamp(1.4rem, 3.5vw, 2rem)` — keep the max ≤ 2rem or 6-digit numbers overflow their card
