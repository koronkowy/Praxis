# 🛡️ Praxis: The High-Signal Progressive Job Board

**Praxis** is a curated, minimalist job board designed to filter out ethically compromised and standard corporate noise. It isolates roles specifically within **human rights advocacy, public interest technology, labor organizing, and vetted progressive impact work.**

By surfacing region constraints, work models, and dynamic recency metrics directly in the dashboard, Praxis helps you identify roles that match your specific geographic and authorization requirements instantly.

**Created by [Lacey Kasten](https://laceykasten.com/).**

**Powered by:** GitHub Actions, Gemini AI (Data Clerk), & Vanilla JavaScript.

*Praxis is an open-source project. Follow the project or contribute on [GitHub](https://github.com/koronkowy/Praxis).*

---
## 📥 How to Submit a Position

We welcome community contributions! To propose new roles for the board, follow these steps:

1. **Open an Issue:** Navigate to the GitHub Issues tab and click "New issue."
2. **Batch Your URLs:** Paste the direct URLs of the job postings into the issue body. (Please use *one URL per line* if submitting a batch).
3. **The Gatekeeper Review (Why the delay?):** To protect our backend API quotas from spam and ensure only high-signal progressive roles make it to the board, **automation does not run immediately**. Once you submit your issue, a repository maintainer will manually review the links.
4. **Automation Trigger:** If the links fit our scope, the maintainer will manually apply the `job-submission` label. This label triggers our background "Data Clerk" to extract the job data and create a Pull Request.
5. **Live Deployment:** Once the generated data is merged by the maintainer, the roles will instantly appear live on the board.

> **Note:** Please submit only active job postings relevant to human rights advocacy, public interest technology, or labor organizing. Submitting ethically compromised roles, non-job URLs, or "junk" data may result in submission privileges being revoked.

---

## 🛠️ System Architecture & Workflow

Praxis utilizes GitHub’s native infrastructure for a zero-cost, high-performance pipeline.

### The Ingestion Flow

1. **Trigger:** A labeled GitHub Issue initiates a **GitHub Action** runner.
2. **Extraction:** A Python-based "Data Clerk" (`parser.py`) scrapes the URL and uses **Gemini AI** to extract structured metadata.
3. **Deduplication:** The script validates the URL against `jobs.json`. Duplicate URLs are rejected; new jobs are assigned a sequential ID.
4. **Human Gatekeeper:** A **Pull Request** is generated. This allows for manual verification of scope (progressive relevance), data accuracy, and final approval before hitting `main`.

---

## 🤖 Automation: Daily Health Check

![Daily Health Check](https://github.com/koronkowy/Praxis/actions/workflows/health_check.yml/badge.svg)

To ensure the job board stays relevant, I’ve built an automated **Health Check** pipeline that runs daily to identify expired or broken job postings.

### How it works

The `health_check.py` script performs the following logic on every job in the database:

1. **Maintenance Awareness:** It scrapes page content for keywords (e.g., "maintenance," "scheduled-downtime"). If found, it skips the job, preventing false "expired" statuses during temporary outages.
2. **Deadline Verification:** Any job that has passed its `deadline` date is automatically flagged as `expired`.
3. **Link Integrity:** The script validates that the job URL is still a valid, unique posting. It detects:
   * **HTTP 404/410 Errors:** Confirmed broken links.
   * **Generic Redirects:** If a link redirects to a career portal homepage (instead of a specific job ID), it marks the listing as `expired`.
4. **Skeleton Page & Manual Review Flow:** Some modern job boards (like Workday) use heavy JavaScript, resulting in an empty HTML skeleton (< 200 characters). To avoid falsely marking these as expired, the script extracts structural metadata from `<script type="application/ld+json">` and `<meta>` tags. If the text is still unverifiable (or the request times out), the job is flagged with `needs_manual_check: true`.
5. **Monday Batched Reviews:** Every Monday, the pipeline aggregates any jobs flagged for manual review and creates a single GitHub Issue for human verification. On the frontend board, these unverifiable jobs will display their recency date in red. Once the human verified or the job successfully passes the next automated daily check, this flag auto-clears.
6. **Self-Healing:** Once a job is marked `expired`, the script updates the status in `jobs.json`, which automatically triggers the frontend to dim the listing and update the deadline display to "Expired".

### Automated Pipeline

This process is fully hands-free:

* **GitHub Actions:** A scheduled workflow runs the script every day at midnight (UTC).
* **Auto-Commit:** If any jobs are flagged as expired, the workflow automatically commits the changes to the `main` branch, keeping the live board updated without manual intervention.

---

## 🤖 AI Integration & Gemini API Usage

![Gemini 429 Errors](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/koronkowy/Praxis/main/badge_429.json&logo=googlegemini) ![Gemini 503 Errors](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/koronkowy/Praxis/main/badge_503.json&logo=googlegemini)

> **Note:** If the **Gemini 429 Errors** badge is red, job postings may be delayed as we have hit our daily quota or per-minute rate limits for the Gemini AI. If the **Gemini 503 Errors** badge is red, the Gemini API is currently overloaded or experiencing downtime. Please check back later!

Praxis relies on Google's **Gemini AI** (specifically the `gemini-flash-latest` model) acting as a "Data Clerk" to parse and structure the unstructured text of job descriptions.

This project is built to work fairly well even on **Gemini's Free Tier**. However, the free tier does come with restrictions regarding daily limits and per-minute API calls. If the ingestion pipeline is processing a large batch of jobs, you may encounter temporary rate limits (429 errors).

If you bring your own "beefier" API key (a paid tier), you should not experience these same rate limiting issues and can ingest jobs much faster. However, regardless of the tier, be mindful of your usage to control costs or stay within quotas.

### Secret Management

Your **Gemini API Key** is never hardcoded. It must be stored in **GitHub Repository Secrets** and is injected into the GitHub Actions workflow at runtime, ensuring complete security.

---

## 🗺️ Geographic & Regional Logic

| **Region** | **Mapping Logic** | **Examples / Hubs** |
| :--- | :--- | :--- |
| **Global** | Worldwide / Global remote roles | Global Remote, Worldwide, Anywhere (Maps to all primary regions) |
| **US-All** | Nationwide US remote roles | Remote (US), Nationwide, United States, Americas |
| **US-East** | East Coast & DC Metro (DMV) | VA, DC, MD, NYC, NY, Boston, MA, PA, NC, NJ, SC, FL, GA, OH, CT, ME, DMV, Washington DC |
| **US-West** | West Coast & PNW | WA, Seattle, Portland, OR, CA, SF, LA, San Francisco, UT, NV, AZ, CO, HI, PNW, Bay Area, SOCAL |
| **US-Central** | Midwest & South-Central / TOLA | TX, Austin, Dallas, Houston, IL, Chicago, MN, TN, AL, MO, NE, ND, WI, MI, TOLA, Midwest |
| **Canada** | Canada nationwide & all provinces | Canada, Americas, ON, Toronto, BC, Vancouver, QC, Montreal, AB, Calgary, MB, Winnipeg, SK, Saskatoon, NS, Halifax, NB, PEI, NL |
| **Europe** | UK, EU, Schengen, DACH, Nordics, Baltics, Balkans | Europe, Schengen, UK, London, Dublin, Germany, Berlin, DACH, Paris, Amsterdam, Madrid, Stockholm, Tallinn, Warsaw, Prague, Belgrade, Kyiv |
| **MENA** | Middle East & North Africa | MENA, Levant, GCC, Israel, Tel Aviv, Beirut, Palestine, Ramallah, Amman, Cairo, Dubai, Riyadh, Qatar, Bahrain, Oman, Iraq, Morocco, Tunis |
| **APAC** | Asia/Pacific & Australasia | APAC, ANZ, India, Bangalore, Mumbai, Pakistan, Karachi, Lahore, Tokyo, Singapore, Sydney, Manila, Seoul, Taipei, Jakarta, Bangkok |
| **LATAM** | Central & South America | LATAM, Americas, Central America, Costa Rica, Panama, São Paulo, Mexico City, Bogotá, Buenos Aires, Santiago, Lima |

> **Note:** "Washington" resolves to `US-West` (WA state) unless it explicitly includes "Washington, DC", "Washington, D.C.", "Washington DC", or "D.C." (which resolves to `US-East`).

### Regional Logic Summary

* **Intelligent Expansion:** When you search for a specific hub (e.g., "Seattle"), the engine automatically expands the query to include the parent region (`US-West`) and nationwide remote roles (`US-All`).
* **Satellite Hub Normalization:** The search dictionary automatically maps commuter satellites (e.g., Redmond/Bellevue for Seattle; Reston/McLean for DMV) to their parent region, ensuring relevant roles are never missed due to localized naming.
* **Territory Alias Support:** The engine natively resolves industry shorthand. Typing territory codes like **TOLA**, **DMV**, **DACH**, **ANZ**, or **PNW** into the search bar instantly expands the query to include all associated cities, states, and regional authorization constraints.

---

*Built for the community. High signal, low noise.*
