# PRD Reviewer

An AI-powered product requirements auditor built as a premium Apple-style productivity app. Paste a PRD, get a structured analysis of ambiguities, missing edge cases, scope creep, and inconsistencies — with copy-ready rewrites for every finding.

**[Live Demo →](https://claude.ai/public/artifacts/8bb816ff-071d-4028-84db-0be25edf9145)**

<img width="583" height="706" alt="Screenshot 2026-05-28 at 8 00 30 PM" src="https://github.com/user-attachments/assets/9d2a52bd-f4ff-486f-bbfb-818f5d9cee69" />
<img width="583" height="390" alt="Screenshot 2026-05-28 at 8 01 23 PM" src="https://github.com/user-attachments/assets/ada95e03-a104-4fa8-b79e-ed3a3f9910b9" />
<img width="583" height="390" alt="Screenshot 2026-05-28 at 8 01 38 PM 1" src="https://github.com/user-attachments/assets/16db8c48-bfac-4f99-a4f9-67010d2c987d" />

---

## What it does

Product managers and engineers drop in a requirements doc (text or PDF) and get back:

- **Categorised findings** — ambiguous specs, missing edge cases, scope expansion, requirement conflicts
- **Severity levels** — critical / major / minor / suggestion
- **Quoted source text** — exact lines from the PRD that triggered each finding
- **Suggested rewrites** — concrete, copy-ready replacements
- **Live analysis preview** — heuristic signals appear as you type, before the full API call
- **Ask AI** — follow-up chat with the full PRD and findings in context

---

## Technical highlights

### AI & prompt engineering
- Calls the **Anthropic Claude API** (`claude-sonnet-4`) with a carefully engineered system prompt that forces structured JSON output — category, severity, quoted source text, explanation, and rewrite for every finding
- **Robust JSON parser** with five fallback layers: handles truncated responses, markdown fence wrapping, model preamble, partial arrays, and regex-extracted objects — so the UI never crashes on a malformed API response
- **Heuristic live preview** runs six regex-based detectors client-side as the user types (vague language, passive requirements, no measurable metrics, scope qualifiers, missing error states, no ownership) — streams findings one-by-one with 260ms intervals before the full analysis runs

### React architecture
- Single-file React component (`~800 lines`) with clean separation: design tokens → primitives → domain components → app shell
- `useMotionValue` + `useSpring` + `useTransform` for the parallax hero icon — mouse position tracked via `getBoundingClientRect`, spring-interpolated to rotation and specular sheen
- Progressive disclosure pattern: `focusedCard` state causes inactive cards to animate to `opacity: 0.52, scale: 0.988` while the active card elevates — spatial hierarchy through motion, not color
- PDF input via `FileReader` → base64 → Anthropic document block

### Motion design
- Eight named spring configs tuned to match UIKit/SwiftUI physics (`stiffness`, `damping`, `mass`)
- `AnimatePresence` with `mode="popLayout"` for finding cards — exit animations run before new cards enter
- Streaming reveal: findings animate in sequentially with staggered `delay: i * 0.04` rather than all at once
- `height: 0 → "auto"` spring expansion for progressive disclosure — no hardcoded heights

### Visual design system
- Full Apple HIG implementation: SF Pro Display (≥20px) / SF Pro Text (<20px) via `-apple-system`, 8px spacing grid, iOS semantic color tokens
- Three-layer material system: `MatUltra` (nav, `blur(26px)`, `opacity: 0.58`) → `Glass` (panels, `blur(18px)`, `opacity: 0.76`) → floating cards (`blur(20px)`, `opacity: 0.46–0.82` by severity)
- SVG noise texture (`feTurbulence`, `baseFrequency: 0.75`) applied at `opacity: 0.022` on all glass surfaces — matches Apple's Ultra Thin material grain
- Ambient background: three radial gradient orbs (blue, blue, green) behind all content — same technique as macOS Sonoma wallpapers

---

## Stack

| Layer | Technology |
|---|---|
| UI | React 18 + Framer Motion |
| AI | Anthropic Claude API (`claude-sonnet-4`) |
| Styling | CSS-in-JS (inline styles + injected `<style>`) |
| Build | Vite |
| Deploy | Vercel |

---

## Local setup

```bash
git clone https://github.com/zcher-20/prd-reviewer.git
cd prd-reviewer
npm install
npm run dev
```

The app calls the Anthropic API directly from the client. In a production deployment, proxy the call through a serverless function and set `ANTHROPIC_API_KEY` as an environment variable.

---

## Design decisions worth noting

**Why a single JSX file?** The entire app — tokens, components, state, API logic — lives in one file intentionally. This is a portfolio artifact: reviewers can read the full implementation top-to-bottom without jumping between modules. A production version would split into `components/`, `hooks/`, `lib/`.

**Why inline styles over Tailwind/CSS modules?** The design system is expressed as JS objects (`T.D1`, `C.blue`, `SP.card`), which makes the design tokens directly inspectable and composable. Framer Motion's `animate` prop requires JS values, not class names — keeping everything in JS avoids the friction of translating between two systems.

**Prompt engineering rationale:** The system prompt forces the model to quote exact text from the PRD (`≤30 words`) rather than paraphrasing. This makes findings immediately actionable — the PM can search-and-replace the quoted text directly rather than hunting for what the model is referring to.

---

*Built by Zayneb Cherif*
