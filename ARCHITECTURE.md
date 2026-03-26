# nvnda.dev — Architecture & Documentation

> Breaking things carefully, then writing about it.

A single-page security portfolio and blog hub for Michelle Hartono. Dark theme, interactive terminal footer, featured article showcase, and study notes grid. Designed as a landing page that links out to published work across Medium, Substack, and HackMD.

**Live**: [nvnda.dev](https://nvnda.dev)
**Repo**: [github.com/michelle-hartono/nvnda-dev](https://github.com/michelle-hartono/nvnda-dev)

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Single HTML file (vanilla HTML/CSS/JS) |
| Fonts | Google Fonts (Inter, JetBrains Mono, Space Grotesk) |
| Hosting | Vercel (static deployment) |
| Domain | nvnda.dev (Cloudflare DNS → Vercel) |
| Build | None — static file, no framework |

---

## Architecture

```
┌─────────────────────────────────────┐
│           index.html                │
│                                     │
│  ┌───────────┐  ┌────────────────┐  │
│  │  <style>  │  │   <body>       │  │
│  │  All CSS  │  │   Header       │  │
│  │  inline   │  │   Hero         │  │
│  └───────────┘  │   Proof Strip  │  │
│                 │   Featured     │  │
│  ┌───────────┐  │   Recent       │  │
│  │ <script>  │  │   Study Notes  │  │
│  │ Terminal  │  │   About        │  │
│  │ XSS/SQLi  │  │   Terminal     │  │
│  │ detection │  │   Footer       │  │
│  └───────────┘  └────────────────┘  │
└─────────────────────────────────────┘
         │
         ▼
┌─────────────────┐
│  Vercel (CDN)   │
│  Static hosting │
│  Auto-deploy    │
│  from GitHub    │
└─────────────────┘
         │
         ▼
┌─────────────────┐
│  Cloudflare DNS │
│  nvnda.dev      │
│  A → Vercel IP  │
└─────────────────┘
```

---

## Page Sections

| Section | Purpose | Links to |
|---------|---------|----------|
| **Header** | Sticky nav with terminal cursor | Anchor links (#featured, #recent, #notes, #about) |
| **Hero** | Title + subtitle + credential badges | — |
| **Proof Strip** | CRTP, CEH, Block Magnates, 她Ta Zhi Dao, HackMD | — |
| **Featured** | Web3 smart contract article with code block | [Medium article](https://medium.com/block-magnates/breaking-into-web3-hacking-on-chain-smart-contracts-5c0a09820123) |
| **Recent** | Podcast + Fallback writeup | [Substack](https://nataliefenglin.substack.com/cp/188470043), [Medium](https://medium.com/block-magnates/my-first-on-chain-hacking-fallback-1097fd08af28) |
| **Study Notes** | 3 HackMD cards (CRTP, Databases, ViewState) | [HackMD links](https://hackmd.io/@michellenovenda) |
| **About** | Bio + connect links | LinkedIn, Medium, Email |
| **Terminal** | Interactive footer with command handling | Easter egg: `cd stitch` → needleandstrings.com |

---

## Content Map

### Featured (Block Magnates)
- **Breaking into Web3: Hacking On-Chain Smart Contracts** — Introduction to smart contract security, EVM vulnerabilities, reentrancy exploit breakdown

### Recent Posts
| Title | Date | Type | Platform |
|-------|------|------|----------|
| Cybersecurity, Scammers, and Confidence | Feb 2026 | Podcast | Substack (她Ta Zhi Dao) |
| My First On-Chain Hacking: Fallback | 2025 | Article | Medium (Block Magnates) |

### Study Notes (HackMD)
| Title | Topic |
|-------|-------|
| CRTP Study Notes | AD red team: AMSI bypass, Kerberos attacks, cross-forest trust abuse |
| Exploiting Databases | SQLi reference for Oracle, MySQL, PostgreSQL — extraction to RCE |
| Exploiting ViewState | ASP.NET deserialization via ysoserial.net gadget chains |

---

## Interactive Terminal

The footer contains a functional terminal emulator with:

**Known commands**:
`help`, `whoami`, `ls`, `cat .plan`, `ping`, `history`, `clear`, `sudo`, `exit`,
`cd blog`, `cd notes`, `cd research`, `cd stitch`

**Attack detection** (playful responses):
- **XSS**: Detects `<script>`, `alert()`, `onerror` — shows fake alert modal with the payload
- **SQLi**: Detects `' OR 1=1`, `UNION SELECT`, `DROP TABLE` — contextual responses
- **Command injection**: Detects `;`, `|`, backticks
- **Path traversal**: Detects `../`
- **SSTI**: Detects `{{`, `${`, `<%` — evaluates safe math expressions

**Visitor detection** (on scroll into view):
- IP geolocation via `ipapi.co/json/`
- Browser + OS detection from User-Agent
- Local time with personality (`night owl`, `early bird`)

**Security**:
- All user input sanitized via `esc()` (textContent → innerHTML)
- Known commands whitelist — no dynamic property access
- Math evaluation sandboxed with `Function()` + regex filter
- Input maxlength: 80 characters
- Output capped at 18 lines

**Easter egg**: `stitch/` appears in `ls` output in warm amber (#C4875A). `cd stitch` opens needleandstrings.com.

---

## Design System

### Color Palette

| Name | Hex | Usage |
|------|-----|-------|
| BG Top | `#1A1510` | Page gradient start (darkest) |
| BG Mid | `#2A2218` | Page gradient mid |
| BG Bottom | `#382E22` | Page gradient end |
| BG Raised | `#332A22` | Card backgrounds |
| BG Code | `#1A1C21` | Code block background |
| Text | `#F2EBE0` | Primary text (warm cream) |
| Text Dim | `rgba(242,235,224,0.65)` | Secondary text |
| Text Muted | `rgba(242,235,224,0.32)` | Tertiary/labels |
| Gold | `#D4A55C` | Primary accent |
| Gold Light | `#E4BC72` | Code keywords, highlights |
| Slate | `#A3BFDA` | Code variables, secondary accent |
| Green | `#7A9E7E` | Code strings, success |
| Red | `#C47070` | Code danger, errors |
| Border | `rgba(237,229,216,0.06)` | Subtle borders |
| Border Hover | `rgba(212,165,92,0.2)` | Hover borders |

### Body Gradient
```css
background: linear-gradient(180deg,
  #1A1510 0%,      /* near-black warm */
  #2A2218 35%,     /* dark brown */
  #33291E 70%,     /* warm mid */
  #382E22 100%     /* lighter base */
);
background-attachment: fixed;
```

### Typography

| Role | Font | Weight | Usage |
|------|------|--------|-------|
| Headings | Space Grotesk | 300-700 | Hero title, card titles, section headers |
| Body | Inter | 300-600 | Paragraphs, descriptions, nav |
| Code/UI | JetBrains Mono | 400-700 | Terminal, code blocks, tags, labels |

### Code Syntax Colors
| Token | Color | Hex |
|-------|-------|-----|
| Keyword | Gold Light | `#E4BC72` |
| String | Green | `#7A9E7E` |
| Comment | Slate (dim) | `rgba(163,191,218,0.45)` |
| Danger | Red | `#C47070` |

---

## Ambient Effects

### City Light Bokeh (Hero)
```css
/* Scattered warm radial gradients simulating distant city lights */
background:
  radial-gradient(circle at 8% 30%, rgba(212,165,92,0.07) 0%, transparent 40%),
  radial-gradient(circle at 85% 15%, rgba(196,135,90,0.05) 0%, transparent 35%),
  radial-gradient(circle at 65% 70%, rgba(212,165,92,0.04) 0%, transparent 30%),
  radial-gradient(circle at 20% 80%, rgba(180,140,80,0.03) 0%, transparent 25%),
  radial-gradient(circle at 92% 55%, rgba(142,170,194,0.03) 0%, transparent 30%);
```

### Page Bottom Glow
```css
/* Fixed city-lights glow along the bottom edge */
body::after — warm elliptical gradients at 15%, 50%, 80% horizontal
```

---

## File Structure

```
nvnda/
├── index.html          # Everything — HTML, CSS, JS in one file
└── ARCHITECTURE.md     # This file
```

That's it. One file. 1208 lines.

---

## Related Projects

| Project | Domain | Purpose |
|---------|--------|---------|
| **needle & strings** | [needleandstrings.com](https://needleandstrings.com) | Cross-stitch pattern generator + gallery |
| **WebCenters site** | michellehartono.com (JESD WebCenters) | Personal landing page linking both projects |

### WebCenters (michellehartono.com)

The personal hub site is hosted on JESD MA WebCenters platform. Key files:

| WebCenters File | Content |
|----------------|---------|
| `home-head-section.html` | Google Fonts links |
| `home.css` | Hide platform header/footer |
| `home.html` | Full page content (style + HTML + script) |
| `header.html` | Platform header (hidden via CSS) |
| `footer.html` | Platform footer (hidden via CSS) |

Design mirrors nvnda.dev aesthetics but with a light/dark card layout:
- Dark card: nvnda.dev (security work)
- Light card: needle & strings (cross-stitch)
- Hero quote: *"Breaking what shouldn't break, stitching what I can't forget."*

---

## Deployment

```
Push to main → Vercel auto-deploys → CDN serves index.html
                                    → Cloudflare DNS resolves nvnda.dev
```

No build step. No bundler. No framework. Just HTML.

---

## Future Plans

- **Astro migration**: When self-hosted blog posts begin, convert to Astro with markdown content collections (`/blog/`, `/writeups/`, `/notes/`)
- **HTB writeups**: Dedicated writeups section for HackTheBox and CTF challenges
- **Self-hosted blog**: Posts written directly on nvnda.dev instead of Medium
- **Notes stay on HackMD**: Cards on nvnda.dev link out to HackMD

---

*Last updated: 2026-03-26*
