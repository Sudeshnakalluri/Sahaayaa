# Sahaaya: Voice-First Government Scheme Navigator

Speak. Discover. Benefit.

Sahaaya is a voice-first civic technology web application that helps Indian citizens discover government welfare schemes they may be eligible for. Users describe their situation in their own language, and Sahaaya builds a structured profile, matches it against a curated database of central government schemes using a deterministic rules engine, and produces a plain-language explanation, a document checklist, and an action plan that can be taken to a Common Service Centre (CSC).

Theme: Tech for a Better Tomorrow

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Solution Overview](#solution-overview)
3. [Key Features](#key-features)
4. [Responsible AI Design](#responsible-ai-design)
5. [System Architecture](#system-architecture)
6. [Tech Stack](#tech-stack)
7. [Project Structure](#project-structure)
8. [Supported Schemes](#supported-schemes)
9. [Supported Languages](#supported-languages)
10. [Getting Started](#getting-started)
11. [Environment Variables](#environment-variables)
12. [API Reference](#api-reference)
13. [Testing](#testing)
14. [Deployment](#deployment)
15. [Browser Support and Limitations](#browser-support-and-limitations)
16. [Adding or Updating a Scheme](#adding-or-updating-a-scheme)
17. [Roadmap](#roadmap)
18. [Contributing](#contributing)
19. [Disclaimer](#disclaimer)
20. [License](#license)

---

## Problem Statement

India runs hundreds of welfare schemes across agriculture, education, health, housing, pensions, and employment. Many eligible citizens never claim them because:

- Government portals are text-heavy, English-first, and hard to navigate.
- Eligibility rules are spread across multiple documents and are hard to interpret.
- Low-literacy and rural users, senior citizens, and widows often depend on intermediaries.
- People do not know which documents to carry or where to apply.

## Solution Overview

Sahaaya replaces forms with a conversation:

1. The citizen speaks or types their situation in their regional language.
2. Sahaaya extracts only the facts the user explicitly stated (age, gender, state, occupation, income, and so on) into a structured profile.
3. If important information is missing, Sahaaya asks one targeted follow-up question at a time.
4. A deterministic rules engine evaluates the profile against every scheme in the database.
5. Results are grouped as Likely Eligible, Needs More Info, or Does Not Appear to Match, with a plain-language reason for each.
6. The citizen gets a consolidated document checklist and a step-by-step action plan, which can be read aloud, printed, or shared.

## Key Features

- Voice input using the browser Web Speech API, with full text-input fallback.
- Read-aloud (text-to-speech) for results and action plans in regional Indian voices.
- Support for seven languages: English, Hindi, Tamil, Telugu, Kannada, Bengali, and Marathi.
- Profile extraction that never invents data. Unknown fields stay unknown.
- One-question-at-a-time follow-up flow for missing information.
- Deterministic eligibility engine with three outcome states.
- Estimated total annual benefit shown as a headline figure.
- Per-scheme detail view with benefit, eligibility reason, required documents, application method, official portal, and CSC guidance.
- Consolidated, de-duplicated document checklist across all matched schemes.
- Printable action plan and shareable text summary.
- Offline-resilient mode: the app works fully without any API key using a built-in heuristic NLP engine.
- Optional Google Gemini integration for improved profile extraction.
- Demo personas for quick testing (for example a widow farmer in Tamil Nadu, and a college student in Uttar Pradesh).

## Responsible AI Design

Sahaaya treats eligibility as a rules problem, not a language-model problem.

- The language model, when enabled, is used only to extract structured facts from what the user said. It is never allowed to decide eligibility or to invent benefits.
- All eligibility rules live in `backend/data/schemes.json` and are evaluated in plain code in `backend/services/eligibilityEngine.js`.
- Every scheme record carries an official source, a portal name, and a last-verified date.
- Results are always presented as an initial eligibility check, not a final determination.

Eligibility outcomes:

| Status | Meaning |
| :--- | :--- |
| Likely Eligible | All currently known conditions satisfy the scheme rules. |
| Needs More Info | No known conflict, but one or more required attributes are unknown. |
| Does Not Appear to Match | A known attribute conflicts with a hard requirement of the scheme. |

## System Architecture

```
User (voice or text, regional language)
        |
        v
Browser: Web Speech API (speech-to-text)  <->  Speech Synthesis API (text-to-speech)
        |
        v
React + Vite + Tailwind CSS frontend
        |
        v  HTTP / JSON
Express.js backend
   |-- POST /api/profile/extract    -> NLP service (heuristic engine, optional Gemini)
   |-- POST /api/profile/questions  -> follow-up question generator
   |-- POST /api/schemes/match      -> deterministic eligibility engine
   |-- GET  /api/schemes            -> list all schemes
   |-- GET  /api/schemes/:id        -> single scheme record
   |-- POST /api/explain            -> plain-language scheme explanation
   |-- POST /api/summary            -> action plan and document checklist
   |-- GET  /api/health             -> service status
        |
        v
backend/data/schemes.json (curated scheme database)
```

In production, the Express server also serves the built frontend from `frontend/dist`, so the entire application runs from a single process on a single port.

## Tech Stack

Frontend

- React 19
- Vite
- Tailwind CSS 4 (via `@tailwindcss/vite`)
- lucide-react for icons
- Web Speech API (recognition and synthesis)
- oxlint for linting

Backend

- Node.js (18 or later) with ES modules
- Express 4
- cors
- dotenv
- Optional: Google Gemini API (`gemini-1.5-flash`) for extraction

## Project Structure

```
Sahaaya/
|-- README.md
|-- package.json               Root scripts (start, server, client, build, test)
|-- .env.example               Environment variable template
|-- .gitignore
|-- backend/
|   |-- server.js              Express app and API routes
|   |-- package.json
|   |-- data/
|   |   `-- schemes.json       Curated scheme database
|   |-- services/
|   |   |-- eligibilityEngine.js   Deterministic matching logic
|   |   `-- nlpService.js          Heuristic and Gemini extraction, follow-ups, explanations
|   `-- tests/
|       |-- engine.test.js         Unit tests for the engine
|       |-- personas.test.js       Multi-persona integration tests
|       `-- telugu.test.js         Telugu input end-to-end test
`-- frontend/
    |-- index.html
    |-- vite.config.js         Dev server on port 3000 with /api proxy to port 5000
    |-- package.json
    |-- public/                Static assets (favicon, icons)
    `-- src/
        |-- main.jsx
        |-- App.jsx            Screen flow and application state
        |-- index.css
        |-- components/
        |   |-- Navbar.jsx
        |   |-- ScreenWelcome.jsx
        |   |-- ScreenProfilePreview.jsx
        |   |-- ScreenFollowUp.jsx
        |   |-- ScreenResults.jsx
        |   |-- SchemeDetailModal.jsx
        |   `-- ScreenActionPlan.jsx
        |-- hooks/
        |   `-- useVoice.js    Speech recognition and synthesis hook
        `-- data/
            `-- translations.js   UI strings, languages, demo presets
```

## Supported Schemes

The database currently contains 22 central government schemes.

| Category | Schemes |
| :--- | :--- |
| Agriculture | PM-KISAN, PM Fasal Bima Yojana (PMFBY), Kisan Credit Card (KCC), PM Krishi Sinchayee Yojana |
| Education | PM-USP Central Sector Scholarship, Post-Matric Scholarship for SC/ST/OBC, AICTE Pragati Scholarship for Girls, NMMSS |
| Women and Widows | Indira Gandhi National Widow Pension Scheme, PM Matru Vandana Yojana, Lakhpati Didi (DAY-NRLM), Sukanya Samriddhi Yojana |
| Healthcare | Ayushman Bharat PM-JAY, PM Suraksha Bima Yojana, Janani Shishu Suraksha Karyakram |
| Pension and Elderly | Indira Gandhi National Old Age Pension Scheme, Atal Pension Yojana, PM Vaya Vandana Yojana |
| Housing | PM Awas Yojana Gramin, PM Awas Yojana Urban 2.0 |
| Employment | MGNREGA, PM SVANidhi |

Each record includes the benefit description, an annual benefit estimate where applicable, eligibility rules, required documents, application method, official source, portal name, CSC guidance, and a last-verified date.

## Supported Languages

| Code | Language | Speech locale |
| :--- | :--- | :--- |
| en | English | en-IN |
| hi | Hindi | hi-IN |
| ta | Tamil | ta-IN |
| te | Telugu | te-IN |
| kn | Kannada | kn-IN |
| bn | Bengali | bn-IN |
| mr | Marathi | mr-IN |

## Getting Started

### Prerequisites

- Node.js 18 or later
- npm 9 or later
- A modern Chromium-based browser (Chrome or Edge) for voice input

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/sahaaya.git
cd sahaaya
```

### 2. Install dependencies

```bash
cd backend
npm install
cd ../frontend
npm install
cd ..
```

### 3a. Run in production mode (single server)

```bash
cd frontend
npm run build
cd ..
npm start
```

Open http://localhost:5000 in your browser.

### 3b. Run in development mode (hot reload)

Use two terminals.

Terminal 1, backend:

```bash
cd backend
npm run dev
```

Terminal 2, frontend:

```bash
cd frontend
npm run dev
```

- Frontend dev server: http://localhost:3000
- Backend API: http://localhost:5000

The Vite dev server proxies all `/api` requests to the backend on port 5000.

### 4. Optional: enable Gemini

Copy `.env.example` to `.env` and add your key. See the next section.

## Environment Variables

Create a `.env` file in the project root or in the `backend` folder. Never commit this file.

| Variable | Required | Default | Description |
| :--- | :--- | :--- | :--- |
| PORT | No | 5000 | Port the Express server listens on. |
| GEMINI_API_KEY | No | empty | Google Gemini API key. If empty, the app uses the built-in heuristic NLP engine. |

Example:

```env
PORT=5000
GEMINI_API_KEY=your_gemini_api_key_here
```

Note: dotenv loads `.env` from the directory the server is started in. If you start the server from the project root with `npm start`, place `.env` in the root. If you start it from `backend`, place it there.

## API Reference

Base URL: `http://localhost:5000`

### GET /api/health

Returns service status and the active NLP mode (`gemini_ai_enabled` or `local_heuristic_nlp`).

### POST /api/profile/extract

Extracts structured profile facts from free text.

Request body:

```json
{
  "text": "I am 58 years old. I am a widow. I live in Tamil Nadu. I have a small farm.",
  "currentProfile": {}
}
```

`text` is required. `currentProfile` is optional and is merged with newly extracted facts. Returns HTTP 400 if `text` is missing.

### POST /api/profile/questions

Generates the next follow-up question for missing information.

Request body:

```json
{
  "profile": { "age": 58, "gender": "female" },
  "lang": "en"
}
```

### POST /api/schemes/match

Runs the deterministic eligibility engine against the profile.

Request body:

```json
{
  "profile": { "age": 58, "gender": "female", "state": "Tamil Nadu" }
}
```

Returns grouped results (likely eligible, needs more info, not matching) and a headline estimate of annual support.

### GET /api/schemes

Returns every scheme in the database.

### GET /api/schemes/:id

Returns a single scheme by its id (for example `pm-kisan`).

### POST /api/explain

Returns a plain-language explanation of a scheme for the given profile and language.

Request body:

```json
{
  "scheme": { "id": "pm-kisan" },
  "profile": {},
  "lang": "en"
}
```

### POST /api/summary

Builds the consolidated action plan and document checklist for a profile.

Request body:

```json
{
  "profile": { "age": 58, "gender": "female", "state": "Tamil Nadu" }
}
```

## Testing

Run the unit tests for the eligibility engine:

```bash
cd backend
npm test
```

Run the multi-persona integration test:

```bash
cd backend
node tests/personas.test.js
```

Run the Telugu end-to-end test:

```bash
cd backend
node tests/telugu.test.js
```

The tests are deterministic and require no network access or API key.

Lint the frontend:

```bash
cd frontend
npm run lint
```

## Deployment

Because the Express server serves the built frontend, the simplest deployment is a single Node process.

1. Build the frontend: `cd frontend && npm run build`
2. Set the `PORT` environment variable (and optionally `GEMINI_API_KEY`) on your host.
3. Start the server: `npm start` from the project root.

This works on any Node-capable host such as Render, Railway, Fly.io, or a VPS. Most of these platforms build and run automatically if you configure a build command of `cd frontend && npm install && npm run build` and a start command of `npm start` (after installing backend dependencies).

Alternatively, deploy the `frontend` folder to a static host (Vercel or Netlify) and the `backend` folder to a Node host. In that case, configure the frontend to reach the backend URL instead of relying on the local Vite proxy.

## Browser Support and Limitations

- Voice input relies on the Web Speech API, which is best supported in Chrome and Edge. Firefox has limited support. Users on unsupported browsers can use text input.
- Availability and quality of regional text-to-speech voices depend on the voices installed on the user's device and operating system.
- Some browsers send speech audio to a cloud service for recognition, so voice input generally needs an internet connection.
- The scheme database covers 22 central schemes. State-specific schemes are not yet included.
- Eligibility results are an initial screening based only on what the user told the system.

## Adding or Updating a Scheme

1. Open `backend/data/schemes.json`.
2. Add a new object (or edit an existing one) using the existing fields: `id`, `name`, `short_name`, `category`, `benefit`, `benefit_annual_inr`, `benefit_type`, `description`, `scope`, `eligibility`, `documents`, `application_method`, `official_source`, `portal_name`, `csc_guidance`, `last_verified`.
3. Use the same `eligibility` structure as existing entries so the engine can evaluate it.
4. Update `last_verified` to the date you checked the official source.
5. Add or update a test in `backend/tests/` covering the new rule.
6. Run `npm test` in `backend` to confirm nothing broke.

## Roadmap

- State-specific schemes for more states.
- Additional languages and dialect handling.
- Offline-first progressive web app support.
- Multi-turn conversational memory across sessions.
- Assisted application flows and CSC operator mode.
- Automated freshness checks for scheme data against official portals.

## Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/your-feature`.
3. Make your changes and add tests where relevant.
4. Run the tests and lint checks.
5. Commit with a clear message and open a pull request.

For scheme data changes, please cite the official government source in the pull request.

## Disclaimer

Sahaaya provides an initial, informational eligibility check only. It is not an official government service and does not guarantee eligibility or approval for any scheme. Scheme rules, benefit amounts, and application processes change over time. Always confirm details on the official portal or at your nearest Common Service Centre or Panchayat office before applying.

## License

Released under the MIT License.

Scheme information is compiled from official government sources, including myscheme.gov.in, pmkisan.gov.in, nsap.nic.in, and nha.gov.in.
