# 🧲 Offer Subscriber

**Offer Subscriber** is an automated desktop application that aggregates job offers from email alerts and job portal sitemaps, deduplicates them across multiple layers, and consolidates everything into a single, always up-to-date Excel file — with zero manual effort.

The app registers its own daily unattended run in Windows Task Scheduler on first launch, and can also be triggered on demand from a lightweight desktop GUI.

---

## ✨ Features

- **Dual Extraction**: Pulls offers from Outlook job alerts (Microsoft Graph, OAuth2, read-only `Mail.Read` scope) and directly from configured job portal sitemaps (`requests` + `lxml`).
- **Layered Deduplication**: An exact layer (canonical URL + content hash, zero cost) followed by a fuzzy layer (`RapidFuzz`) that catches republished offers with minor title changes.
- **AI Enrichment (in progress)**: A Gemini-backed layer that classifies offers against a closed `job_group` list, infers the hiring `company`, and flags cross-portal duplicates with a confidence score — it only suggests, never deletes; the final decision stays human.
- **Excel-Native Storage**: Persists results in the format the end user already works in daily (`openpyxl`), keeping the door open to swap in a database later without touching the core.
- **Self-Installing Automation**: On first packaged run, the app registers itself in the Windows Task Scheduler for a daily 08:00 execution — automation is part of the product, not a manual setup step.
- **On-Demand GUI**: A minimal `CustomTkinter` window to pick a lookback window (1–30 days) and re-run the pipeline manually, e.g. after a vacation.
- **Hexagonal Architecture**: `core` defines domain entities and use cases behind ports (extraction, storage, deduplication, enrichment) and never imports a concrete dependency — swapping Gemini, Excel, or the email provider means writing a new adapter, not touching the core or its test suite.
- **Strict Quality Bar**: Pydantic v2 models validated at the system boundary, `mypy --strict` with zero errors in production code, and a pytest suite covering unit and integration layers.

---

## 🏗️ Architecture

The project follows a **ports & adapters (hexagonal)** design:

- `core/domain`: business entities (`JobOffer`, `FilterConfig`), value objects (canonical URL, offer ID, content hash), enums and domain exceptions. Pure Python, zero external dependencies.
- `core/application`: use cases (`ProcessOffersUseCase`, `DeduplicateOffersUseCase`, `EnrichOffersUseCase`) orchestrating the domain through abstract ports — the core declares *what* it needs, never *how* it's implemented.
- `infrastructure/adapters`: concrete implementations of those ports — `OutlookAdapter` and `SitemapAdapter` for extraction, `ExcelStorageAdapter` for storage, `GeminiAdapter` for AI enrichment.
- `interface/gui`: the driving adapter (`CustomTkinter` desktop window) that calls into the same use cases the scheduled run uses.
- `composition_root.py`: the single place where ports and adapters are wired together, shared by both the GUI and the headless entry point.

---

## 🛠️ Tech Stack

- **Language**: Python 3.12
- **Email Extraction**: O365 / Microsoft Graph (OAuth2, `Mail.Read` scope, locally cached token)
- **Web Extraction**: `requests` + `lxml` (sitemap / sitemap-index parsing with cycle detection)
- **Fuzzy Deduplication**: `RapidFuzz`
- **Storage**: `openpyxl`
- **AI Enrichment**: `google-genai` (Gemini 2.5 Flash) — in development
- **Data Validation**: `Pydantic v2` / `pydantic-settings`
- **GUI**: `CustomTkinter`
- **Scheduling**: Windows Task Scheduler (self-registered)
- **Packaging**: `PyInstaller`
- **Testing & Quality**: `pytest` (unit + integration, ~97% core coverage), `mypy --strict`

---

## 📋 Prerequisites

1. **Python 3.12** or higher installed.
2. An **Azure AD app registration** (Client ID) to authorize read-only access to a personal Microsoft/Outlook mailbox.
3. *(Optional)* A **Gemini API Key** if you want to enable the AI enrichment layer. You can get one at [Google AI Studio](https://aistudio.google.com/).

---

## 🚀 Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/tu-usuario/offer_subscriber.git
   cd offer_subscriber
   ```

2. **Create a virtual environment (recommended):**
   ```bash
   python -m venv .venv
   # On Windows:
   .venv\Scripts\activate
   # On Linux/Mac:
   source .venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   # for running the test suite / type checking:
   pip install -r requirements-dev.txt
   ```

4. **Configure environment variables:**
   Copy `.env.example` to `.env` and fill in at least the Outlook Client ID:
   ```env
   OUTLOOK_CLIENT_ID=your_azure_app_client_id
   # SITEMAP_URLS=["https://example.com/sitemap.xml"]
   # GEMINI_API_KEY=your_gemini_api_key
   ```

---

## 🖥️ Usage

The application has two entry points sharing the same underlying pipeline:

- **On-demand (GUI):**
  ```bash
  python gui_main.py
  ```
  Pick how many days to look back (1–30), click **"Recuperar ofertas"**, and open the resulting Excel file directly from the window once it's done. On first packaged run, this same launch also registers the daily scheduled task.

- **Headless (scheduled run):**
  ```bash
  python main.py
  ```
  Runs the full ETL silently — extraction, deduplication and storage — using the configured lookback window. This is the command Windows Task Scheduler invokes automatically every day at 08:00.

---

## 📂 Project Structure

```text
offer_subscriber/
├── core/
│   ├── domain/                  # Entities, value objects, enums, exceptions
│   └── application/
│       ├── ports/               # Abstract interfaces (extraction, storage, dedup, enrichment)
│       └── use_cases/           # ProcessOffers, DeduplicateOffers, EnrichOffers
├── infrastructure/
│   ├── adapters/
│   │   ├── extractors/          # OutlookAdapter, SitemapAdapter
│   │   ├── storage/             # ExcelStorageAdapter
│   │   └── ai/                  # GeminiAdapter
│   ├── config/                  # Pydantic settings, API key store
│   ├── scheduling/               # Windows Task Scheduler self-registration
│   └── service/                  # SimilarityService (fuzzy dedup)
├── interface/
│   └── gui/                      # CustomTkinter desktop window
├── test/
│   ├── unit/                     # Domain, use cases, adapters (mocked)
│   └── integration/               # Outlook, sitemap, Excel, E2E flow
├── composition_root.py            # Wires ports to adapters
├── gui_main.py                    # GUI entry point
├── main.py                        # Headless/scheduled entry point
└── .env.example                   # Configuration template
```

---

## 👤 Author

- **Iván Herrero Galván**
- **Date**: July 2026
- **Status**: Active development — AI enrichment layer in progress (phase 2 of 7 on the roadmap)

---

## 📄 License

This project is for private/educational use. Contact the author for distribution details.
