<div align="center">

# AeroDex — Real-Time Airfare Price Index

### A data-driven prototype for monitoring domestic airfare movements in India

**Project theme:** Development of a real-time airfare price index through automated collection of airline and online travel portal fares, with analytical links to the Consumer Price Index (CPI).

[![Python](https://img.shields.io/badge/Python-3.12%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Playwright](https://img.shields.io/badge/Browser%20Automation-Playwright-2EAD33?logo=playwright&logoColor=white)](https://playwright.dev/)
[![SQLite](https://img.shields.io/badge/Database-SQLite-003B57?logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![Scikit-learn](https://img.shields.io/badge/ML-Scikit--learn-F7931E?logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![Status](https://img.shields.io/badge/Status-Working%20Prototype-blue)](#project-status)

</div>

---

## Contents

- [What is AeroDex?](#what-is-aerodex)
- [The Problem](#the-problem)
- [Project Objectives](#project-objectives)
- [How It Works](#how-it-works)
- [Key Features](#key-features)
- [Index Methodology](#index-methodology)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [API Overview](#api-overview)
- [Data Sources and Responsible Collection](#data-sources-and-responsible-collection)
- [Data Quality and Limitations](#data-quality-and-limitations)
- [Testing](#testing)
- [Roadmap](#roadmap)
- [Project Status](#project-status)
- [Author](#author)
- [Disclaimer](#disclaimer)

## What is AeroDex?

**AeroDex** is a software and data-analysis prototype designed to turn individual flight-fare observations into useful indicators of airfare movement in India.

Airfares can change by route, airline, departure date, booking lead time, season, and availability. A single ticket price does not explain whether prices are generally rising or falling. AeroDex explores a more consistent approach: collect fare observations, validate and store them, aggregate comparable fares, and calculate route-level and national-level indicators.

The project combines a fare-data pipeline with an index-calculation engine, an analytical dashboard/API, data-integrity checks, and optional forecasting and AI-generated summaries.

> **Important:** AeroDex is an independent prototype. It is not an official MoSPI or DGCA system, and its outputs should not be interpreted as an official government-published index.

## The Problem

Flight prices are dynamic, while economic indicators are generally compiled and published on a different schedule. Researchers and analysts may therefore find it difficult to compare changing ticket prices across routes and airlines using isolated observations alone.

AeroDex investigates the following question:

**Can automated fare collection and transparent index methodology help summarise domestic airfare movements in a consistent, repeatable way?**

The project also explores whether airfare indicators can be compared with Consumer Price Index (CPI) data and other aviation-related signals. Such comparisons can provide context, but do not by themselves establish causation or an official contribution to inflation.

## Project Objectives

- Collect flight-fare observations from supported airline and online travel sources.
- Store observations in a structured database for analysis and auditing.
- Standardise fare components where source data makes that possible.
- Compare fares across routes, airlines, travel dates, and booking windows.
- Calculate route-level and national-level airfare indicators.
- Compare airfare trends with an available CPI baseline.
- Examine data quality, unusual price movements, and market concentration.
- Explore forecasting, scenario analysis, and readable analytical briefings.
- Clearly distinguish collected live observations from fallback or benchmark estimates.

## How It Works

```text
Airline / Travel-Portal Sources
             |
             v
     Fare Collection Layer
       (browser automation)
             |
             v
   Validation & Data Integrity
             |
             v
        SQLite Database
             |
             v
   Fare Normalisation and
       Route Aggregation
             |
             v
    Route / National Index
             |
      +------+------+
      |             |
      v             v
 Trend Analysis   Dashboard / API
      |
      v
 CPI Comparison, Forecasting
 and Scenario Exploration
```

### Typical workflow

1. **Collect:** The scraper attempts to obtain fare and itinerary details from configured sources.
2. **Validate:** Records are checked for required fields, plausible values, duplicates, fare consistency, and other quality concerns.
3. **Store:** Observations and related baseline data are stored in SQLite.
4. **Aggregate:** Comparable observations are grouped by route, carrier, travel date, and other available dimensions.
5. **Calculate:** The index engine calculates fare and index indicators using the configured methodology.
6. **Present:** The application exposes selected outputs through its dashboard and API.
7. **Interpret:** Analytical components support trend comparisons, quality assessment, and optional scenario or forecasting workflows.

## Key Features

### 1. Multi-source fare collection

The existing project documentation describes integrations or collection workflows involving sources such as Google Flights, EaseMyTrip, Cleartrip, and SpiceJet. Source availability and extraction success can vary over time due to website changes, access controls, location, and rate limits.

The data model supports fields such as carrier, flight number, origin, destination, departure date and time, arrival time, duration, stops, fare components, source portal, and collection timestamp.

### 2. Live data and fallback transparency

The design distinguishes between:
- **Live-collected observations**, when a configured source returns usable fare data.
- **Fallback or benchmark estimates**, when live collection is unavailable and a fallback path is used.

These categories are not interchangeable. A fallback estimate must not be presented as a fare observed live from a website. Always check the source/provenance indicator for a particular result.

### 3. Fare decomposition

Where the source provides the relevant fields, a fare may be represented using components such as:
- Base fare
- Fuel surcharge (YQ)
- Airport fees (for example, UDF/PSF)
- Goods and Services Tax (GST)
- Total fare

Not every source exposes the same breakdown. The application should reconcile components only when the available data supports that calculation.

### 4. Route and national indicators

The index engine includes route-level and national-level indicator concepts. These are intended to summarise movements across observations rather than simply display one ticket price.

### 5. Data integrity checks

The project includes a data-integrity module intended to evaluate aspects such as fare-component reconciliation, unusual observations, source diversity, and freshness. A quality score is a diagnostic measure; it is not proof that every source record is correct.

### 6. CPI baseline comparison

A CPI baseline is stored for analysis alongside airfare observations. This supports comparison and exploratory analysis; it does not automatically establish that a particular airfare change caused a change in headline CPI.

### 7. Forecasting and scenario analysis

The codebase includes forecasting and aviation-shock scenario components. Forecasts and simulated scenarios are analytical outputs, not guarantees of future prices or direct measurements of historical market outcomes.

### 8. Optional AI-generated summaries

The project supports optional Gemini integration for natural-language summaries. A deterministic/offline path is described in the project configuration for environments without an API key. AI-generated text should be checked against the underlying data before being used for decisions.

## Index Methodology

The formulas below explain common index concepts relevant to AeroDex. **The exact implementation, route weights, base period, and missing-data rules must be confirmed in the corresponding source code and methodology documentation before treating any formula here as a complete specification.**

### 1. Total fare

Where all components are separately available and are not already included in one another:

\[
F = B + YQ + A + T
\]

Where:
- \(F\) = total fare
- \(B\) = base fare
- \(YQ\) = fuel surcharge
- \(A\) = applicable airport fees
- \(T\) = applicable taxes

Care is needed to avoid double-counting components that a source already includes in its quoted total.

### 2. Weighted average fare

\[
\bar{F}_w = \frac{\sum_{i=1}^{n} w_i F_i}{\sum_{i=1}^{n} w_i}
\]

Here, \(F_i\) is an observed fare and \(w_i\) is its assigned weight. A higher weight means that an observation contributes more to the average. The meaning of the result depends on how the project defines those weights.

### 3. Fixed-base price index

A basic fixed-base index is:

\[
I_t = \frac{P_t}{P_0}\times 100
\]

Where:
- \(P_t\) = comparable price in the current period
- \(P_0\) = comparable price in the chosen base period

For example, if the base-period fare is ₹5,000 and the comparable current fare is ₹5,500, the index is 110. This indicates a 10% increase relative to the base period.

### 4. Percentage change

\[
\%\Delta P = \frac{P_t-P_{t-1}}{P_{t-1}}\times100
\]

For example, a change from ₹4,000 to ₹4,400 is an increase of 10%.

### 5. Weighted national indicator

A general weighted aggregation of route indices can be written as:

\[
I_{\text{national},t} =
\frac{\sum_{r=1}^{R} w_r I_{r,t}}{\sum_{r=1}^{R} w_r}
\]

Where \(I_{r,t}\) is the index for route \(r\), and \(w_r\) is that route's weight. A defensible national indicator requires documented weights, consistent route definitions, a base period, and rules for routes with missing observations.

### 6. Laspeyres, Paasche, and Fisher indices

The existing project documentation describes Laspeyres and Paasche index concepts and a Fisher-index comparison in its scenario-analysis component. In general:

\[
P_L = \frac{\sum p_t q_0}{\sum p_0 q_0}\times100
\]

\[
P_P = \frac{\sum p_t q_t}{\sum p_0 q_t}\times100
\]

\[
P_F = \sqrt{P_L P_P}
\]

Here \(p_0\) and \(p_t\) represent base-period and current-period prices, while \(q_0\) and \(q_t\) represent base-period and current-period quantities or weights. These formulas illustrate the standard concepts; verify the project's actual implementation and available quantity/weight data before claiming that a particular result is calculated exactly this way.

## Architecture

The repository contains a web-facing application and a fare-processing area. Based on the current project files, important components include:

| Component | Role |
|---|---|
| `airfare_index/live_fetcher/scraper.py` | Fare collection workflow |
| `airfare_index/live_fetcher/robot_guard.py` | Robots.txt and scraping-policy checks |
| `airfare_index/live_fetcher/worker_scraper.py` | Separate/background collection workflow |
| `airfare_index/live_fetcher/index_engine.py` | Index-calculation functionality |
| `airfare_index/live_fetcher/integrity_engine.py` | Data-quality and integrity checks |
| `airfare_index/live_fetcher/forecasting_engine.py` | Forecasting functionality |
| `airfare_index/live_fetcher/shock_replay.py` | Scenario/stress-test functionality |
| `airfare_index/live_fetcher/live_calamity_tracker.py` | External event/weather-related monitoring component |
| `airfare_index/live_fetcher/ai_engine.py` | Optional AI summary and fallback logic |
| `airfare_index/live_fetcher/static/index.html` | Web dashboard interface |
| `airfare_index/airfare_index.db` | SQLite database containing fare and CPI baseline tables |
| `METHODOLOGY.md` | Project methodology documentation |
| `DEMO_SCRIPT.md` | Demonstration walkthrough |

File names and responsibilities may evolve as the project changes. Consult the code for the current runtime behaviour.

## Technology Stack

| Technology | Use in the project |
|---|---|
| **Python** | Application logic, data processing, and index calculations |
| **Playwright** | Browser automation for supported fare-collection workflows |
| **SQLite** | Local structured database |
| **Scikit-learn** | Machine-learning/forecasting components |
| **Gemini API** | Optional natural-language summaries |
| **HTML, CSS, JavaScript** | Dashboard/front-end interface |
| **Git and GitHub** | Version control and project hosting |

## Repository Structure

The project may change as development continues. The main areas and files include:

```text
AeroDex-Real-Time-Airfare-Price-Index/
├── .github/                         # GitHub-related configuration
├── airfare_index/
│   ├── live_fetcher/
│   │   ├── static/
│   │   │   └── index.html           # Dashboard interface
│   │   ├── ai_engine.py             # AI summary logic
│   │   ├── forecasting_engine.py    # Forecasting
│   │   ├── integrity_engine.py      # Data-quality checks
│   │   ├── index_engine.py          # Index calculations
│   │   ├── live_calamity_tracker.py # Event monitoring
│   │   ├── proxy_rotator.py         # Network/proxy support
│   │   ├── robot_guard.py           # Robots.txt checks
│   │   ├── scraper.py               # Fare collection
│   │   ├── shock_replay.py          # Scenario analysis
│   │   └── worker_scraper.py        # Background collection
│   ├── README.md
│   └── airfare_index.db             # SQLite data store
├── Documentation/                   # Project report/documentation
├── tests/                            # Automated tests
├── app.py                            # Application entry point (check configuration)
├── main.py                           # Application entry point (check configuration)
├── DEMO_SCRIPT.md                    # Demo guide
├── METHODOLOGY.md                    # Methodology details
├── requirements.txt                  # Python dependencies
├── render.yaml                       # Deployment configuration
├── railway.json                      # Deployment configuration
├── runtime.txt                       # Runtime configuration
└── README.md                          # Project overview
```

## Getting Started

### Prerequisites

- Git
- Python version compatible with the installed dependencies (the project documentation targets Python 3.12+)
- Internet access for dependency installation and live collection
- Playwright browser binaries if using browser-based collection
- A Gemini API key only if you want to use the optional Gemini functionality

### 1. Clone the repository

```bash
git clone https://github.com/jenilsoni25/AeroDex-Real-Time-Airfare-Price-Index.git
cd AeroDex-Real-Time-Airfare-Price-Index
```

### 2. Create and activate a virtual environment

**Windows PowerShell:**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS/Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

Start with the root dependency file if it exists and matches your current setup:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If the runtime component maintains its own dependency file, install that file instead or as instructed by the component's README.

### 4. Install Playwright Chromium

If browser automation is required:

```bash
python -m playwright install chromium
```

On Linux/cloud hosts, additional system dependencies may be required. See the official [Playwright documentation](https://playwright.dev/python/docs/intro).

### 5. Configure environment variables

Review `airfare_index/.env.example`. If the application expects a local `.env` file in that directory, create it with:

**PowerShell:**

```powershell
Copy-Item .\airfare_index\.env.example .\airfare_index\.env
```

Set `GEMINI_API_KEY` only if you intend to use Gemini features. Keep real credentials in your local environment or hosting provider's secret settings. **Never commit a real `.env` file or API key.**

### 6. Run the application

The repository contains multiple entry-point files and deployment configurations, so the correct command depends on the configured application entry point.

The existing project documentation describes a server workflow similar to:

```bash
cd airfare_index/live_fetcher
python server.py
```

If `server.py` is absent in your checked-out version, use the current entry point documented in that directory or the deployment configuration. Do not assume that every command shown in an older project version applies to the latest version.

When the server starts successfully, open the local address and port printed in the terminal (the project documentation describes port `8000` as the default).

### 7. Run the background scraper (optional)

The project documentation describes a worker interface similar to:

```bash
python worker_scraper.py --routes DEL-BOM,DEL-BLR,BOM-BLR --interval 300
```

Run this from the directory containing `worker_scraper.py`, and verify the supported command-line options in the current version before use. Scraping may be restricted or unavailable for particular sources.

## Configuration

The example environment file documents these settings:

| Variable | Purpose |
|---|---|
| `GEMINI_API_KEY` | Optional credential for Gemini features |
| `GEMINI_MODEL` | Configured model identifier |
| `PORT` | Server port (example default: `8000`) |
| `ENABLE_CLOUD_PLAYWRIGHT` | Controls configured cloud Playwright behaviour |

Environment-variable support depends on the active entry point. Keep the example file populated only with placeholders and safe defaults.

## API Overview

The existing project README documents the following API route families. **Confirm the routes against the running server or route definitions in the current code**, as endpoint availability may change by version or configuration.

| Endpoint | Method | Intended purpose |
|---|---:|---|
| `/api/v1/search` | `POST` | Flight search / fare retrieval |
| `/api/v1/integrity/score` | `GET` | Data-integrity score |
| `/api/v1/shocks/list` | `GET` | Available scenario catalog |
| `/api/v1/shocks/replay` | `GET` | Scenario replay |
| `/api/v1/compliance/sources` | `GET` | Configured source catalog |
| `/api/v1/compliance/robots` | `GET` | Robots.txt compliance status |
| `/api/v1/live/pulse` | `GET` | Current index pulse |
| `/api/v1/index/national` | `GET` | National index indicator |
| `/api/v1/index/weekly` | `GET` | Weekly aggregation timeline |
| `/api/v1/macro/timeline` | `GET` | CPI/aviation-related comparison |
| `/api/v1/routes/weights` | `GET` | Route-weight information |
| `/api/v1/db/quotes` | `GET` | Recent stored fare observations |
| `/api/v1/export/daily` | `GET` | Daily export |
| `/api/v1/export/weekly` | `GET` | Weekly export |
| `/api/v1/export/quotes` | `GET` | Fare-observation export |
| `/api/v1/ai/summary` | `GET` | Optional AI-generated summary |

For example, once the server is running locally, an endpoint can be tested with a browser or an HTTP client. Use the actual host, port, and routes configured in your version.

## Data Sources and Responsible Collection

The existing project documentation names Google Flights, EaseMyTrip, Cleartrip, and SpiceJet among its intended or configured collection sources. Live availability and extraction success are not guaranteed. A source may change its site, block automated requests, require additional verification, or disallow particular automated access.

Responsible collection should include:

- Review the applicable terms of service and access policies for each source.
- Check and respect robots.txt instructions where applicable.
- Use reasonable request rates and backoff after rate limits or service errors.
- Avoid bypassing authentication, CAPTCHAs, paywalls, or other access controls.
- Record collection timestamps and source provenance.
- Clearly identify fallback estimates rather than presenting them as live quotes.
- Prefer official/public datasets for weights and economic baselines, and document their source and reference period.

The project documentation references DGCA-related route/passenger weights and a MoSPI CPI baseline. Before using these in published analysis, document the exact source files, release dates, definitions, coverage, and transformation steps.

## Data Quality and Limitations

AeroDex is a prototype and its results depend on the quality and coverage of the collected data.

- **Coverage bias:** Not all airlines, routes, dates, or booking windows will necessarily be represented equally.
- **Changing fares:** A fare is a time-specific observation and may change shortly after collection.
- **Comparability:** Fare inclusions, baggage rules, fare classes, and source presentation may differ.
- **Missing values:** Missing data can affect route averages and national aggregation.
- **Outliers:** Statistical outlier rules can reduce the influence of unusual values but may also exclude genuine price shocks.
- **Index design:** Base-period selection, weights, route entry/exit, and missing-route treatment materially affect results.
- **CPI interpretation:** An association with CPI is not proof of causation or a verified CPI contribution.
- **Forecast uncertainty:** Model outputs are estimates and should be evaluated against held-out observations and suitable baselines.
- **Live source reliability:** Website changes, anti-bot controls, network issues, and policy restrictions can interrupt collection.

Use the index as an exploratory analytical indicator unless and until its methodology and representativeness have been independently validated.

## Testing

The project's existing documentation describes tests for index calculations, fare-component reconciliation, outlier handling, source parsing, and API behaviour. Run the test suite supported by the checked-out version.

A documented command is:

```bash
python -m unittest discover tests -v
```

Treat test results from your own environment as authoritative for your version. Tests can verify implemented behaviour, but they do not by themselves validate the representativeness of the data or the economic interpretation of an index.

## Roadmap

Potential future improvements include:

- [ ] Publish a clear, versioned index methodology and base-period definition.
- [ ] Document the provenance and reference period for each official baseline dataset.
- [ ] Add reproducible data-quality and index-validation reports.
- [ ] Evaluate index behaviour against independent benchmarks.
- [ ] Improve route coverage and treatment of missing observations.
- [ ] Add monitoring for scraper failures and source changes.
- [ ] Report forecast errors and uncertainty, not only point estimates.
- [ ] Provide sample outputs and dashboard screenshots.
- [ ] Add automated CI checks for tests and code quality.

## Project Status

**Current status: Working prototype / ongoing development.**

Features, source integrations, endpoint availability, and deployment instructions may change. The repository should be treated as a research and engineering project, not a production-grade official statistical system.

## Author

**Jenil Soni**  
M.Sc. Data Science student, GSFC University, Vadodara, Gujarat, India  
GitHub: [@jenilsoni25](https://github.com/jenilsoni25)

This project explores the application of data engineering, web automation, statistical index construction, and machine learning to airfare monitoring and economic analysis.

---

## Acknowledgements and References

- [Python](https://www.python.org/)
- [Playwright documentation](https://playwright.dev/python/docs/intro)
- [SQLite documentation](https://www.sqlite.org/docs.html)
- [scikit-learn documentation](https://scikit-learn.org/stable/)
- [Google AI for Developers](https://ai.google.dev/)
- [MoSPI](https://mospi.gov.in/)
- [Directorate General of Civil Aviation (DGCA)](https://www.dgca.gov.in/)

The exact datasets and versions used for any reported result should be cited in the accompanying methodology and analysis.

---

<div align="center">

**AeroDex — turning fare observations into interpretable airfare indicators.**

</div>
