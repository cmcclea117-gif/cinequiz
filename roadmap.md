# CineQuiz — Product Roadmap

Last updated: April 30, 2026

---

## ✅ Shipped

### Core
- 6 quiz categories: Movies, TV, Games, Music, Books, Board Games
- 60 questions per category, 20 randomly selected per quiz
- Question rotation — 3 unique quiz sets before repeating
- AI wrong answer generation (Cloudflare Workers AI / llama-3.3-70b)
- Cover art enrichment (TMDB, RAWG, Last.fm, Open Library, BoardGameGeek)
- GitHub Gist publish + shareable link
- Results card download (canvas PNG with watermark)
- Shareable results page (?results= URL param)
- Play count tracking per quiz
- Access gate — request-based creator approval
- Auto-approve clean emails, flag suspicious ones
- Email notifications via Resend (tuekssietch.fun domain)
- localStorage answer persistence (survives page refresh/Safari kill)
- Quiz data caching (7-day offline resilience)
- Light/dark theme toggle
- "Create your own" CTA on shared quizzes
- AI killswitch (KV flag killswitch:ai = 1)
- Rate limiting (40 AI calls per IP per 30 minutes)
- Dev mode (5x eyebrow click)
- security.txt, DMARC, HTTPS enforced

### Answer Suggestions (April 30, 2026)
- 💡 button per creator form row triggers AI suggestion call on demand
- Returns 4 well-known examples relevant to the question and quiz type
- Per-type prompts with explicit format instructions and hard exclusion rules (no music answers for game questions, etc.)
- Results shown as dismissible chips — click to populate input, × to dismiss individually
- KV cache by question ID + quiz type, 7-day TTL (repeat clicks are free)
- Separate rate limit bucket (80 calls/30 min) independent of wrong-answer generation

### Creator Form UX (April 30, 2026)
- 2-column layout (was 3 columns) — question + answer/hint, no separate hint column
- Search hint moves to row 2 beneath the answer input, spanning the answer column only
- Hint placeholder updated to "Search hint (optional)" — self-explanatory
- Quiz type buttons use `auto-fit` grid — wrap to 2 per row on mobile instead of overflowing

### CSV Import Instructions (April 30, 2026)
- Rewritten from 7 outdated steps to 4 accurate ones
- All 6 quiz categories listed
- 60-question pool and random 20-selection per quiz explained
- Stale Anthropic API key step removed (AI now handled by worker)

### Share Overlay + Social Sharing (April 30, 2026)
- Post-submit overlay appears immediately with score and contextual label
- Single "Share Results" button: publishes to Gist, auto-copies link to clipboard, confirms with "✓ Link copied to clipboard!"
- X Post and WhatsApp buttons appear once link is ready, pre-filled with score + creator name
- Share text: "I just took [Name]'s CineQuiz and scored X/Y (Z%)! How well do you know them?"
- Social share uses spoiler-free ?quiz= link so recipients take the quiz fresh
- "See my answers ↓" dismisses overlay and scrolls to answer review
- Theme toggle hidden while overlay is open, restored on close

---

## 🔜 Next Up

### Answer Autocorrect / Capitalization
**What:** Before generating wrong answers, run the creator's answer through a light normalization pass — fix obvious misspellings, apply correct capitalization (title case for films/games, etc.), and standardize formats like "Character (Actor)".

**Why:** Misspelled or oddly-cased answers are a giveaway to quiz-takers. "the godfather" or "pulp ficton" makes the correct answer obvious from user error alone.

**Tech:** One extra AI call (or a simpler regex+dictionary pass) before the wrong answer generation step. Could be a fire-and-forget suggestion shown to the creator ("Did you mean: The Godfather?") rather than auto-replacing.

**Considerations:**
- Should be transparent to the creator — show the normalized version and let them accept or override
- Title case rules differ by category (films = title case, artists = as-stylized)
- Could fold into the existing generation prompt rather than a separate call

---

### quizType Persistence
Save the selected quiz type to localStorage so switching tabs doesn't reset to Movies.

---

## 🗓 Medium Term

### Aggregate Stats Per Question
**What:** After submitting a quiz, show "X% of CineQuiz users picked this answer" per question.

**Why:** Adds a social/discovery layer. Turns a personal quiz into a cultural conversation. "Only 12% of people picked The Godfather — you're in rare company."

**Tech requirements:**
- Cloudflare D1 (SQLite on Cloudflare) — needed for proper aggregation queries
- Schema: `quiz_answers(question_id, answer_text, quiz_type, submitted_at)`
- Worker route: `POST /record-answer` (fire-and-forget on submit)
- Worker route: `GET /question-stats?id=m01` returns top answers + percentages
- Privacy/consent: opt-in checkbox on submit — "Include my answers in anonymous CineQuiz stats"

**Unlocks with D1:**
- Leaderboards (most played quizzes)
- Trending questions (most answered this week)
- Quiz history per access token
- "Most popular answers" discovery page

**Considerations:**
- Answer normalization is hard — "The Godfather" vs "godfather" need to map together. Use AI to normalize on write.
- Start opt-in only, make it prominent not buried
- Tabled until sharing and engagement features are stable

---

## 🔭 Future / Big Ideas

### More Quiz Categories
Ideas in rough priority order:
- Podcasts
- Theatre / Broadway
- Food and Restaurants
- Travel
- Sports
- Anime
- Art and Design

Each needs: 60 questions, an image API (or graceful placeholder fallback), AI prompt tuning for that category's answer types.

---

### Social Features
- Quiz collections — group multiple quizzes ("My Complete Taste Profile")
- Quiz challenges — send directly to a friend and get notified when they complete it
- Leaderboard — who knows you best across all your quizzes?
- Profile page — public page showing all your published quizzes

---

### Mobile App
- PWA (Progressive Web App) first — add to home screen, offline support
- Native iOS/Android if traction warrants it
- Push notifications for "someone took your quiz!"

---

### Quiz Discovery
- Public gallery of quizzes (opt-in)
- "Take a random quiz" feature
- Category browse — "Movie quizzes this week"
- Featured quizzes curated by CineQuiz

---

### Analytics Dashboard for Quiz Creators
- How many times has your quiz been taken?
- Average score
- Which questions trip people up most?
- Geographic distribution of players

---

### Monetization (if/when needed)
- Free tier: 1 active quiz at a time, basic stats
- Pro tier ($3-5/mo): Unlimited quizzes, detailed analytics, custom branding, remove watermark
- No ads — keep the experience clean

---

## Backlog / Parked Ideas

- Certificate/SSL pinning — not applicable to web apps (browser controls SSL stack). Current protections (HTTPS + APP_SECRET + CORS + rate limiting + killswitch) are the right model.
- Row Level Security — not applicable, no SQL database currently. Revisit when D1 is added.
- Hardcover API for books — switched to Open Library (no key needed). Revisit if cover quality insufficient.
- Per-question answer save indicator — small UI tick showing answer was saved to localStorage
- Quiz expiry — option to make a quiz expire after X days
- Answer suggestions picklist — show AI suggestions as options but make clear user can type anything

---

## Known Issues / Tech Debt

- quizType state not saved to localStorage — switching tabs resets to movies
- BGG API can be slow (2-step XML fetch) — consider caching results in KV
- Last.fm images sometimes return empty string instead of null — causes broken img tags
- Results page hides score (by design) but also hides percentage — reconsider UX
- RAWG (video game images) blocked on some corporate/institutional networks — low priority, graceful fallback already in place

---

*Update this document after each development session.*
